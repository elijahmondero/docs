# Running Containers on Windows Using PodMan or Docker (Not Docker Desktop)

## Introduction

This guide is about running Linux containers on Windows without using Docker Desktop. 🚀🐧

## Overview

To run Linux containers on Windows, we need a VM that will run these containers. Windows offers a way to use Hyper-V. We can either use WSL (Windows Subsystem for Linux), which uses Hyper-V in the background, or use Hyper-V directly. The first is our primary choice as WSL has a bunch of optimisations, including a special kernel, that allow the Linux VM running in WSL to access Windows facilities in a more performant way. Using Hyper-V directly means giving up these optimisations in favour of a more traditional networking-based communication between the Linux VM and the Windows Host.

In essence, we aim to replicate what Docker Desktop does on Windows. This involves installing WSL, a Linux VM, setting up the Docker daemon, and then installing a Docker CLI on the Windows host. This setup makes it seem from a Windows user’s perspective that the containers are running on Windows.

## Using WSL

### Install WSL
Use either the Microsoft Store or the command line.

From the Microsoft store install WSL.

### Install a Linux Machine and Distro
Use the Store or the command line.

From the Microsoft store install Ubuntu v22 or greater.

Set up user-id/pw if the setup hasn’t already asked you for one.

Reboot your Windows machine.

### Setup WSL DNS
You fix this by changing the default resolver address in the `/etc/resolv.conf` file. However, there are a few steps you need to do first.

First, create/update your `/etc/wsl.conf` file to stop the auto-creation of the `/etc/resolv.conf`. You can use `sudo nano /etc/wsl.conf`:

```ini
[boot]
systemd=true
[network]
generateResolvConf = false
```
Next, remove the existing symlinked /etc/resolv.conf file:

```bash
unlink /etc/resolv.conf
```
Next, create a new /etc/resolv.conf with the contents of a public DNS (e.g., 8.8.8.8 from google.com). You can use sudo nano /etc/resolv.conf:
```bash
nameserver 8.8.8.8
```
You will need to restart WSL now (or restart your Windows machine if you like).

Confirm the system is running:
```bash
systemctl --no-pager status user.slice > /dev/null 2>&1 && echo 'OK: Systemd is running' || echo 'FAIL: Systemd not running'
```

### Install Docker
Follow these steps to install Docker:
```bash
# from: https://docs.docker.com/engine/install/ubuntu/
sudo apt-get update
# Install Docker’s certs, gpg key and add Docker’s repo to the apt repo list
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
sudo mkdir -m 0755 -p /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Install Docker tools 
sudo apt-get update 
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
# Install Docker Compose
sudo ln -s /usr/libexec/docker/cli-plugins/docker-compose /usr/bin/docker-compose
# Add yourself to the Docker group (https://askubuntu.com/a/477554/87625)
sudo usermod -aG docker $USER
# Add the Docker group if it doesn't already exist:
sudo groupadd docker
# Add the connected user "$USER" to the Docker group. Change the user name to match your preferred user if you do not want to use your current user:
sudo gpasswd -a $USER docker
# Apply changes (or log out/in to activate the change)
sudo newgrp docker
```

Test Docker in Linux with:

```bash
docker run --rm -it hello-world
```
(This will only work once the DNS is properly set up.)

### Using Docker in WSL
Follow the instructions for installing Docker in WSL Ubuntu 22.0 and then accessing it via the Docker CLI on Windows. Ensure SSH is set up so that the Docker CLI on Windows can connect to the daemon. This step is unnecessary if you use Docker Desktop as it handles this connectivity.

### Exposing DOCKER daemon over tcp and not SSH (RECOMMENDED)

#### On Linux: Expose the daemon using tcp
Summary: Override your systemd docker.service startup from default parameters, and then set the daemon options to expose both the UNIX socket (optional) and a TCP port (in this case 2375). This doesn’t use TLS, but you can add the TLS by setting up some certs (see docs: ​ )

Reference: [How do I expose the docker API over TCP?](https://serverfault.com/questions/843296/how-do-i-expose-the-docker-api-over-tcp) 

Override your systemd docker.service:

```bash
# add the override
sudo mkdir /etc/systemd/system/docker.service.d/
sudo echo [Service] >> /etc/systemd/system/docker.service.d/override.conf
sudo echo ExecStart= >> /etc/systemd/system/docker.service.d/override.conf 
sudo echo ExecStart=/usr/bin/dockerd >> /etc/systemd/system/docker.service.d/override.conf
# exit daemon config to expose unix socket and tcp port
# you need to add 
# "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"]
# to the json file
sudo nano /etc/docker/daemon.json 
# restart the docker service
sudo systemctl daemon-reload
sudo systemctl restart docker.service
# check you can access docker through the env var override
DOCKER_HOST='tcp://0.0.0.0:2375' docker ps -a
```

#### On the Client
On the client side, you need to set up a docker context that defaults the communication from the docker client to the docker daemon using TCP or use an override using either the environment variable or the command line.

##### Mac (parallels)

Setup a docker context (Mac to parallels machine):
```bash
docker context create daemon-over-tcp --description "connect to docker using tcp"  --docker "host=tcp://10.211.55.5:2375"
```

You can make the context the default using:

```bash
docker context use daemon-over-tcp
```

Use an environment variable:
```bash
DOCKER_HOST='tcp://10.211.55.5:2375' ; docker ps -a
```

Use the command line:
```bash
docker -H 10.211.55.5 ps -a
```
##### Windows (WSL)

Setup a docker context (Windows to WSL machine):

```bash
docker context create daemon-over-tcp --description "connect to docker using tcp"  --docker "host=tcp://127.0.0.1:2375"
```

Use an environment variable (Windows):
```powershell
$env:DOCKER_HOST='tcp://localhost:2375' ; docker ps -a
```

Use the command line (Windows):
```bash
docker -H 'tcp://localhost:2375' ps -a
```

To avoid having to specify it always, you can either create docker context for it, or (recommended) make environment variable global:

```powershell
[Environment]::SetEnvironmentVariable('DOCKER_HOST', 'tcp://localhost:2375', 'Machine')
```

Download docker client [Index of win/static/stable/x86_64/](https://download.docker.com/win/static/stable/x86_64/) 

### Exposing DOCKER daemon SSH (not recommended but available option)

You need to be in an administrator PowerShell session.

```powershell
# make sure ssh agent service is enabled and set to automatic start up
Set-Service   ssh-agent -StartupType Automatic
Start-Service ssh-agent
# ensure you have a .ssh folder (should be true already)
md $env:USERPROFILE\.ssh -ErrorAction SilentlyContinue
cd $env:USERPROFILE\.ssh 
# generate a key pair with a password.
# The name of the key can be changed from windowssh (e.g. my key-name is windowsmac)
ssh-keygen -C windowssh -f $env:USERPROFILE\.ssh\id_windowssh
# add the private from the key pair to the ssh-agent
ssh-add id_windowssh
```
Setup SSH Server in Linux
```bash
# install sshd on wsl 
sudo apt-get install ssh
# start ssh 
sudo systemctl start ssh
# confirm you can ssh localhost on wsl 
ssh $USER@localhost
# append public key from the key pair into ~/.ssh/authorized_keys , e.g. 
cat /mnt/c/Users/$USERPROFILE/.ssh/id_windowssh.pub >> ~/.ssh/authorized_keys
```
Confirm SSH in Windows

ou need to be in an administrator PowerShell session.

Remember to replace your_linux_user_name with your actual Linux username.

Check you can access the VM:
```bash
# Now try ssh into remote machine using ssh with the private key with the -i option
ssh your_linux_user_name@localhost -i $env:USERPROFILE\.ssh\id_windowssh
```
Install the docker client binaries
```bash
gsudo choco install --yes docker-cli
```
Of course, there’s no engine connected yet, so you should get this when testing it.

##### Connect a context to the WSL instance
Remember to replace your_linux_user_name with your actual Linux username.

```bash
# this is the docker context for a linux machine
docker context create wsl-ubuntu --description "wsl-ubuntu" --docker "host=ssh://your_linux_user_name@localhost"
docker context use wsl-ubuntu
```

Test docker in Windows with:
```bash
docker run --rm -it hello-world
```


### Common Issues

#### Connectivity Issues
Once you’ve got connectivity from the Windows Docker CLI to the WSL-based Docker daemon, you might find that you cannot run even the basic `docker run hello-world`. This is because, by default, WSL VMs run on a separate network from your Windows machine. This can be fixed by setting up WSL DNS.

#### DNS Issues
Download Docker CLI from [Index of win/static/stable/x86_64/ (docker.com)](https://download.docker.com/win/static/stable/x86_64/). You should now be able to run `docker run hello-world`.

This is a limitation of how WSL defaults on Windows 10. In Windows 11, you can easily use a WSL Linux machine that uses systemd and change the networking to be external so that it uses the external DNS in the same way as Windows.

## Using Podman in WSL

You can use Podman to replace Docker in WSL. Though Podman won’t have a daemon (by default), it will still use the Linux VM to run the containers. You will still need to set up the WSL machine to resolve DNS address correction, similar to the Docker VM setup. After this, Podman should work just as well as Docker.

**Note:** Podman creates a Fedora-based Linux VM by default, so you’ll need to learn `yum` instead of `apt`.

## Not Using WSL

You can replicate the above setup without using WSL by creating Linux VMs directly in Hyper-V. If you do this, you can choose to use an external network by default and thus avoid the DNS issue. However, you lose the easy management of the VM within WSL.

## Resolving DNS Issue on the VPN

1. Ensure that the VPN is disconnected.
2. In the new VPN software, click on settings, go to AnyConnect and select preferences, and tick the "Allow local (LAN) access when using VPN (if configured)".
3. Go to a WSL command prompt (e.g., open Windows Terminal and select an Ubuntu tab), and edit your `/etc/resolv.conf` file to ensure you have the following entries:

```bash
    nameserver 10.60.152.4 
    nameserver 8.8.8.8
```
4. You can edit the file using:
```bash
    sudo nano /etc/resolv.conf
```
5. You can now test this by connecting to the VPN and then in the WSL command line, typing:
```bash
    sudo apt update
```

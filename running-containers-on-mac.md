# macOS - running in Docker in Colima

## Colima 🚀
Colima manages the underlying Linux VMs so the docker CLI can connect to them via the docker socket. You can set up machines that either run on the native platform ARM64, in the case of modern macOS, or another, e.g. AMD64 (X86_64) using an emulator like QEMU. See Colima CPU architecture emulation for more info.

Since Lima is aka Linux on Mac. By transitivity, Colima can also mean Containers on Linux on Mac.

## Profiles 📂
Colima has profiles for each VM you want to run docker on and thus keep separate.

### List profiles 📋

```sh
colima list
```

### Starting a profile VM ▶️

```sh
colima start --profile <profile_name>
```

### Editing a profile ✏️

```bash
# NB. you can omit --editor to use the default editor ($EDITOR) 
colima edit --profile <profile_name> -edit --editor code
```

### Creating a profile ➕

```bash
# here is create a profile with 4 CPUs and 6MB memory called my_profile 
colima start --profile my_profile -c 4 -m 6
# you can either specify a new profile or an existing one in the start command
```

### Deleting a profile ❌

```bash
# NB Use with caution
colima delete --profile <the_profile_to_delete> 
```

## Using colima/qemu 🖥️
1. Install colima (see [GitHub - abiosoft/colima](https://github.com/abiosoft/colima): Container runtimes on macOS (and Linux) with minimal setup) and the QEMU emulator. This might take a few minutes as it’s got a tonne of dependencies

```bash
brew install colima
```

Install the docker CLI (note using `--cask` will install the docker desktop and that’s not what we want!)

```bash
brew install docker
```

### Start Colima ▶️
```bash
❯ colima start                                                                                                                                                                            ─╯
INFO[0000] starting colima
INFO[0000] runtime: docker
INFO[0000] preparing network ...                         context=vm
INFO[0000] starting ...                                  context=vm
INFO[0020] provisioning ...                              context=docker
INFO[0020] starting ...                                  context=docker
INFO[0026] done
```

### Confirm ✅

```bash
❯ docker ps                                                                                                                                                                               ─╯
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
  ~ ········································································································································································  17:54:52 ─╮
❯ docker run hello-world                                                                                                                                                                  ─╯
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
7050e35b49f5: Pull complete
Digest: sha256:c77be1d3a47d0caf71a82dd893ee61ce01f32fc758031a6ec4cf1389248bb833
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
This message shows that your installation appears to be working correctly.
To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash
Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/
For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Colima FAQ: [colima/docs/FAQ.md at main · abiosoft/colima](https://github.com/abiosoft/colima/blob/main/docs/FAQ.md) 

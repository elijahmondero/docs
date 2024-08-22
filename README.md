# README.md

## 📄 Overview

This repository contains documentation on various topics related to containerization and running containers on different platforms. The documentation is written in Markdown format for easy readability and accessibility.

## 📚 Table of Contents

1. [🛡️ Writing Secure Container Images](#writing-secure-container-images)
2. [🪟 Running Containers on Windows](#running-containers-on-windows)
3. [🍏 Running Containers on Mac](#running-containers-on-mac)

## 🛡️ Writing Secure Container Images

This guide provides best practices for writing secure container images for .NET Core web applications and Python applications/APIs. It covers topics such as:

- Using official base images
- Minimizing image size
- Running applications as non-root users
- Regularly updating base images
- Scanning images for vulnerabilities

For more details, refer to the [Writing Secure Container Images](writing-secure-container-images.md) document.

## 🪟 Running Containers on Windows

This guide explains how to run Linux containers on Windows without using Docker Desktop. It covers:

- Installing and setting up WSL (Windows Subsystem for Linux)
- Configuring DNS for WSL
- Installing Docker within WSL
- Exposing the Docker daemon over TCP
- Using Docker CLI on Windows to interact with the Docker daemon running in WSL

For more details, refer to the [Running Containers on Windows](running-containers-on-windows.md) document.

## 🍏 Running Containers on Mac

This guide introduces Colima, a tool for managing Linux VMs on macOS to run Docker containers. It includes:

- Installing Colima and QEMU emulator
- Managing Colima profiles
- Starting and confirming Colima setup
- Running Docker containers using Colima

For more details, refer to the [Running Containers on Mac](running-containers-on-mac.md) document.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a pull request or open an issue if you have any suggestions or improvements.

## 📜 License

This repository is licensed under the MIT License. See the LICENSE file for more information.

+++
date = 2022-08-22T21:50:30+02:00
title = "Lima quick-start"
description = "When you need Docker Desktop alternative"
tags = ["lima", "docker", "networking", "dns", "containerisation"]
categories = ["devops", "tools"]
+++

## Motivation

Since January 31st, 2022 Docker Desktop cannot be commercially used for free in bigger organisations.
Until now several tools may fill the gap, if you, my Dear Developer, forced to uninstall Docker Desktop.
Docker Desktop is a complete, self-sufficient package ready to use tools including UI to run containers.
To replace Docker Desktop you will need to build a custom setup comprising set of tools.
Installing and configuring different tools may be complex and time-consuming.
On the other hand, you have more control and flexibility.

In this article I will describe simplistic approach to run Lima-VM on macOS combined with standard Docker CLI, 
which is still free to use for everyone.
I will cover also configuration changes I applied to make Lima working for me. I would like to emphasize
network setup for DNS and cross-arch support for new Apple Silicon M1 processors.

## What is Lima?

Lima stands for "Linux machines". It was created to run Linux virtual machines as guests on macOS hosts.
Although, Lima can be used on Linux hosts as well.

It is built on top QEMU, quick emulator.
It makes possible running multiple operating systems on the same physical machine. 

## Installing Lima and Starting VM

To install Lima on macOS it is enough to handover it to brew.

```bash
brew install lima
```

Another way is to install Lima from sources, but in 99% cases you don't gonna need it.
I do not cover installation from sources in this article. 

After installation, you may want to check the version of Lima.

```bash
lima --version
```

Now it is time to start first Virtual Machine with Lima.
There are preconfigured templates (configurations) you can use out of the box.

```bash
limactl start template://docker
```

Here `docker` is a template name.

To check available templates, run

```bash
limactl start --list-templates
```

You can prepare your own configuration in local yaml file or tweak existing examples.

```bash
cp /usr/local/share/doc/lima/examples/docker.yaml .
# do some changes in docker.yaml here
limactl start ./docker.yaml
```

You can also start lima passing `name` parameter.

```bash
limactl start --name=default template://docker
```

By default, the name of the VM instance is the same as template name or name of local file without `.yaml` extensions.

To stop and delete instance, refer to its name.

```bash
limactl stop docker
limactl delete docker
```

If the instance name is `default` you can omit name in the command.
Or you can set `LIMA_INSTANCE` env variable and `limactl` commands will refer to instance name defined in the variable.

```bash
export LIMA_INSTANCE="docker"
```

## Configuration

I won't cover all possible config options here. Rather than that I will describe two aspects which I had to change, 
to make Lima usable on my end.

After creating VM, a folder is created under `~/.lima` named the same as VM.
You can find `lima.yaml` there, your VM configuration file.

### Cross-arch

You may have new, shiny mac with Apple Silicon M1 microchip.
Despite the fact, it may be a good idea to switch back to x86 64-bit architecture.
It will save you troubles with running containers not available for new Apple architecture.

Just put one line in your `lima.yaml` and restart a machine.

```yaml
arch: "x86_64"
```

### Network

By default, guest OS works in so called user-mode network, and generally it has limitations.
There are many issues regarding network support in Lima's GitHub. Not all of them are already closed.

Personally, I had a problem with domain names resolution. I am connected to corporate network. Most often I use VPN.
In my case DNS hosts were incorrectly passed from host to guest OS.

Fortunately, there is a way to enforce DNS IPs in guest. I had to add following config to `lima.yaml`.

```yaml
dns:
- 10.a.b.c
- 10.x.y.z
- 1.1.1.1
- 1.0.0.1
useHostResolver: false
```

First two addresses indicate corporate/VPN network DNS hosts. Next pair is primary and secondary servers of Cloudflare.
They can be replaced by any other publicly accessible DNS servers.
They are used as a fallback when you outside corporate/VPN network.

Don't forget to change `useHostResolver` to false. Otherwise, VM won't start.

## Running a container

Install docker CLI if not done already.

```bash
brew install docker docker-compose docker-credential-helper
```

The last one is a tool to use mac keychain for storing container registry passwords after doing `docker login ...`.

After starting Lima VM from docker template, message is printed with info how to run a container with docker CLI.

```bash
docker context create lima-docker --docker "host=unix://<DIRECTORY>/sock/docker.sock"
docker context use lima-docker
docker run hello-world
```

## Next steps

To unlock full potential of Lima, you probably what to use it with Containerd.
Containerd has certain advanced features when comes to running containers.
Moreover, if you're working in cross-arch environment, 
having Containerd enabled is more performant than solution presented in the article.

## Sources

* https://github.com/lima-vm
* https://www.qemu.org/
* https://wiki.qemu.org/Documentation/Networking

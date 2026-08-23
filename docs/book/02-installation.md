
# Chapter 2: Docker CE Installation on CentOS 9 Stream

## Overview

This chapter covers the complete installation process of Docker Community Edition (CE) on CentOS 10 Stream, including rootless mode configuration.

---

## 2.1 Remove Old Docker Versions

Before installing Docker CE, remove any older versions that may conflict:

```bash
sudo dnf remove -y docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine
```

![[install-docker1.png]]

Result:  most of package in "No match" because we start clean installation
![[install-docker2.png]]

tip  type ``clear`` , or ``Ctrl+l``  to clear screen

---

## 2.2 Install dnf-plugins-core

Install the `dnf-plugins-core` package which provides the `yum-config-manager` utility:

```bash
sudo dnf -y install dnf-plugins-core
```

![[install-docker4.png]]
The `dnf-plugins-core` package is the official collection of **essential plugins for the DNF package manager**. It extends DNF's basic functionality with a wide range of useful commands for advanced package management tasks.

### What it Adds

This package provides several new subcommands to DNF, turning it into a much more powerful tool. Here are some of the key plugins it includes, along with what they do:

*   **`builddep`**: Installs all the build dependencies for a given source package, making it easier to compile software from source.
*   **`config-manager`**: Lets you easily manage your repository configurations (like adding, enabling, or disabling repos).
*   **`copr`**: Enables you to work with Copr repositories, which are a popular way to get additional or newer software on Fedora and RHEL-based systems.
*   **`download`**: Downloads RPM package files to your current directory without installing them.
*   **`reposync`**: Downloads an entire remote repository to your local machine for offline use or for creating a local mirror.
*   **`needs-restarting`**: Checks which system services or the kernel itself need to be restarted after updates.
*   **`changelog`**: Displays the changelog for a package, showing a history of its updates.

### Summary

In short, `dnf-plugins-core` is a set of powerful, essential tools that most advanced users and system administrators will find invaluable. The package you're installing **is not the plugins themselves**, but the **framework that enables you to use the `dnf` commands listed above**.

---

## 2.3 Add Docker CE Repository

Add the official Docker CE repository for CentOS/RHEL:
- we use plugin-manager to install docker-ce (Community version Free) repository

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

sudo dnf update -y
```

![[install-docker3.png]]

Verify the repository was added:

```bash
sudo dnf repolist | grep docker-ce
```

![[install-docker5.png]]

**Short summary:**

`sudo dnf repolist` lists all **enabled software repositories** on your system. It shows their repository ID, name, and the number of packages available in each. 

- Without `sudo`, it works for most users, but `sudo` ensures you see system‑wide configurations.
- To see **disabled** repos as well, use `sudo dnf repolist --all`.

---

## 2.4 Install lastest Docker CE Packages

Good time to Install Docker CE, CLI, containerd.io, Buildx, and Compose plugin:

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
![[install-docker6.png]]
- If prompted, verify the GPG key fingerprint matches _060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35_ and accept it.
---
Result
![[install-docker7.png]]

**Package Summary:**
Here is a summary table for the installed Docker-related packages you listed:

| Package | Version | Architecture | Purpose / Function |
| :--- | :--- | :--- | :--- |
| **containerd.io** | `2.3.3-1` | `x86_64` | Core container runtime that manages the container lifecycle (runs containers, handles images, storage, etc.). |
| **docker-buildx-plugin** | `0.36.1-1` | `x86_64` | Extends `docker build` with advanced features like multi-platform builds, parallel builds, and custom build outputs. |
| **docker-ce** | `3:29.7.2-1` | `x86_64` | The Docker Community Edition **engine (daemon)** – the core service that builds, runs, and manages containers. |
| **docker-ce-cli** | `1:29.7.2-1` | `x86_64` | The command-line interface (`docker` command) used to interact with the Docker daemon. |
| **docker-ce-rootless-extras** | `29.7.2-1` | `x86_64` | Additional tools and configurations to run the Docker daemon and containers without `root` privileges (improves security). |
| **docker-compose-plugin** | `5.5.0-1` | `x86_64` | Docker Compose integration (via `docker compose`) to define and run multi-container applications using a YAML file. |

## How to check any package install what command in linux

Run:
```bash
sudo rpm -ql docker-ce
```

![[install-docker8.png]]

- We focus on 2 file   ``docker.service and docker.socket`` this 2 file will work together 
- We run command ``cat /usr/lib/systemd/system/docker.service``  to explore what service file will run when we start docker

![[install-docker9.png]]

you can  step  [2.5 Start and Enable docker Service](#start-docker)


> [!INFO]
> **Explanation of Docker Socket Security and Architecture**
> 
> This infographic provides a detailed explanation of the relationship and security implications of connecting a Docker Service with the Docker Socket. Here is a concise summary of the key concepts presented:
> 
> 1. **Standard Service Management:** In a typical setup, an Orchestrator (like Docker Swarm or Compose) functions as the Service Manager. It uses the host's primary **Docker Socket** (`/var/run/docker.sock`) as an API gateway. The manager sends lifecycle commands (e.g., to create or scale a service) through this socket to the **Docker Daemon** (or Engine). The Daemon, in turn, processes these requests and manages the individual, identical container instances that form the service.
> 2. **The "Inside-Out" Connection (Mounting the Socket):** This is an advanced, high-privilege configuration where the host’s Docker Socket is mounted directly into a service container (e.g., `-v /var/run/docker.sock:/var/run/docker.sock`). This powerful technique changes the direction of control: an application running _inside_ the container can now use its own internal Docker client to communicate _back through_ the mounted socket to the _host’s_ Docker Daemon. This means the container can manage and control other elements on the host system.
> 3. **Critical Security Warning:** The infographic places immense emphasis on the security risk of mounting the Docker Socket. By doing so, the container is effectively granted **full root privileges** on the host Docker engine. It has the ability to manage, create, and delete _any_ image or container on the host system, which could lead to container escape or total control over the host. This configuration is highly dangerous and should be used with extreme caution.
> 
> ---
> 
> The `/var/run/docker.sock` file is a fundamental component of the Docker architecture, specifically acting as the primary communication link on a local host.
> 
> Here is a detailed breakdown of where it resides in the OS space and how it facilitates the creation of containers using low-level runtimes like `crun` or `runc` on a system like CentOS Stream 10.
> 
> ### Part 1: Kernel Space or User Space?
> 
> The `/var/run/docker.sock` file exists as a **Unix Domain Socket (UDS)**. Its presence and function bridge both User and Kernel space. To give a precise answer, we must distinguish between the socket _representation_, the _endpoints_, and the _communication mechanism_:
> 
> | **Component** | **Location** | **Description** |
> | --- | --- | --- |
> | **Processes using the Socket** (`docker` CLI, `dockerd` daemon) | **User Space** | These are regular application programs running with user-specific or root privileges. |
> | **The Socket File** (`/var/run/docker.sock`) | **Filesystem Space** (Managed by Kernel) | It appears on the file system like a regular file, but it is a "special file" acting as an endpoint reference. User-space processes open this file to establish communication. |
> | **Communication Mechanism** (Data Transfer & Buffering) | **Kernel Space** | When the Docker CLI writes a command to the socket, the data does not go to a network card. It is buffered in kernel memory and delivered directly to the listening Docker Daemon process. |
> 
> #### Summary:
> 
> The **mechanism** of communication (local Inter-Process Communication, or IPC) is handled entirely by the **Linux Kernel** for efficiency. However, the **processes** sending and receiving the data are in **User Space**.
> 
> ### Part 2: How it Connects to `crun` or `runc`
> 
> The connection is not a direct "socket connection" from the socket file itself to the low-level runtime. Instead, `/var/run/docker.sock` is the entryway into a tiered management stack.
> 
> Low-level runtimes like **`runc`** (the traditional OCI default) or **`crun`** (a faster, C-based alternative often preferred in CentOS/RHEL 10) are short-lived utility binaries, not persistent daemons.
> 
> Here is the flow of a standard container creation command (e.g., `docker run`), showing where the socket fits and how the low-level runtimes are invoked:
> 
> #### The Communication Flow on CentOS Stream 10
> 
> 4. **CLI to Daemon (via Socket):**
>     - You type `docker run nginx` in user space.
>     - The Docker CLI opens `/var/run/docker.sock` and sends a REST API request (e.g., `POST /containers/create`).
>     - The **Linux Kernel** delivers this data buffer to the **Docker Daemon** (`dockerd`), which is listening on the other side of the socket.
> 5. **Daemon to `containerd` (Internal IPC):**
>     - `dockerd` processes the high-level request but does not create the container directly.
>     - It communicates with a high-level manager called **`containerd`** (another persistent user-space daemon) over a _different_, internal socket (e.g., `/run/containerd/containerd.sock`).
> 6. **`containerd` to Low-Level Runtime (Binary Invocation):**
>     - `containerd` pulls the Nginx image and prepares the root filesystem bundle.
>     - **This is the connection point to `crun`/`runc`:** `containerd` refers to its configuration to see which OCI-compliant runtime is default (on CentOS Stream 10, this is often pre-configured to `crun`).
>     - `containerd` does **not** make a socket connection to `crun`. Instead, it **spawns `crun` as a subprocess binary** (e.g., by executing `/usr/bin/crun create`). It passes `crun` a path to the configuration file (`config.json`).
> 7. **Low-Level Runtime to Kernel (Container Creation):**
>     - The `crun` (or `runc`) binary reads the configuration and executes the actual Linux Kernel system calls to create the isolated environment (namespaces, cgroups, seccomp).
>     - Once the container process is launched and running, the `crun` binary exits.
> 8. **Monitoring (Shim):**
>     - A small process called `containerd-shim` remains to monitor the container’s process ID (PID) and report status back to `containerd`.
> 
> ### Final Architectural Visual
> 
> Plaintext
> ```
>        USER SPACE                   |      KERNEL SPACE
>                                     |
>  [Docker CLI] --(REST API)--> /var/run/docker.sock --> [Docker Daemon (dockerd)]
>                                     |                      |
>                                     |                      | (Internal IPC Socket)
>                                     |                      v
>                                     |             [containerd (High-Level Runtime)]
>                                     |                      |
>                                     |                      | (Executes Subprocess)
>                                     |                      v
>                                     |            [crun or runc (Low-Level Runtime)]
>                                     |                      |
>                                     |                      | (System Calls)
>                                     |                      v
>                                     |          (Namespaces, Cgroups, FS Isolation)
>                                     |                      |
>                                     |                      v
>                                     |              [RUNNING CONTAINER]
> ```
> 
> ---
> 
> **Overall Summary:**  
> You have a **complete Docker environment** installed, covering the runtime (`containerd`), the engine (`docker-ce`), the CLI, Compose, Buildx, and rootless support. Everything matches the `el10` (RHEL 10 / CentOS Stream 10) distribution

![[install-docker10.png]]


[Explaination from the notebook lm](https://notebook.google.com/notebook/b928fda3-b19d-4fc9-9b78-a1695c862178/artifact/a3cce97f-1f62-4642-94bf-18f3e9865ff7?utm_source=nlm_web_share&utm_medium=google_oo&utm_campaign=art_share_1&utm_content=&utm_smc=nlm_web_share_google_oo_art_share_1_)
<a id="start-docker"></a>
## 2.5 Start and Enable Docker Service

Start the Docker daemon and enable it to start on boot:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

![[install-docker11.png]]
Verify the service is running:

```bash
sudo systemctl status docker
```

![[install-docker12.png]]

Run: command below check who connect to socker
```
sudo lsof /var/run/docker.sock
```

![[install-docker13.png]]
---

## 2.6 Verify Installation

Check the Docker version:

```bash
sudo docker --version
sudo docker compose version
sudo docker ps
```

![[install-docker15.png]]
Run the hello-world container to verify Docker is working correctly:

```bash
sudo docker run hello-world
```

![[install-docker14.png]]
There are 3 step:
- Try to fine docker image locally
- if can not fine. it will go to remote repository of docker hub and pull down image ***
- docker will run docker image which download from repository. There result is docker containers *** 


### The Two Different Things  dockerd and docker socket

|Aspect|**dockerd (The Daemon)**|**Docker Socket (The File)**|
|---|---|---|
|**What it is**|A **running process** (a background service).|A **special file** on the filesystem (`/var/run/docker.sock`).|
|**Purpose**|Actually _does_ the work (creates containers, manages networks, pulls images).|Acts as the **doorway** or **phone line** to send commands to `dockerd`.|
|**Location**|Lives in **User Space** (as a running PID, e.g., PID 8432).|Lives in the **Filesystem**, but its communication mechanism is managed by the **Kernel**.|
|**Analogy**|The **factory worker** who physically builds the container.|The **intercom / telephone** you use to tell the worker what to build.|
|**Without it**|Docker stops working. The socket file has nothing to connect to.|You cannot send commands to `dockerd` locally (you'd have to use a TCP API).|

---

## 2.7 Add Current User to Docker Group

Avoid using `sudo` for every Docker command by adding your user to the `docker` group:

```bash
echo $USER
getend group docker
sudo usermod -aG docker $USER
newgrp docker

cat /etc/group
```

![[install-docker16.png]]
- restart server 
- When Server is back to online Verify you can run Docker without sudo:
- run docker command again but this time with out sudo
```bash
docker run hello-world
```

![[install-docker17.png]]
---

## Check container status
Run:
```
docker ps -a
```
- **docker ps -a** will see all docker both running (active in background) and stop (exit) docker
![[install-docker18.png]] 
![[install-docker19.png]]
## 2.8 Rootless Mode Setup (Optional)

Docker can run in rootless mode for enhanced security. Here is very importance step to incress security of docker:

### Prerequisites

```bash
sudo dnf install -y shadow-utils
sudo dnf install -y fuse-overlayfs slirp4netns 
```
check:
```
rpm -qa shadow-utils
rpm -qa fuse-overlayfs
rpm -qa slirp4netns
```

![[install-docker20.png]]
### Install Docker in Rootless Mode  
docker provide script ``dockerd-rootless-setuptool.sh``  Out of the box

```bash
# Stop docker service
sudo systemctl stop docoker.service
# Disable root Docker first
sudo systemctl disable --now docker.service

# Install rootless Docker
dockerd-rootless-setuptool.sh install --force
```

![[install-docker21.png]]

Scroll down to end of result command

![[install-docker22.png]]
### Set Environment Variables
- from the [info] of script we need add 2 line to  ``~/.bashrc`` 
```bash
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock
```
- how to do use command below
```
echo 'export PATH=/usr/bin:$PATH' >> ~/.bashrc
echo 'export DOCKER_HOST=unix:///run/user/1000/docker.sock' >> ~/.bashrc
```

![[install-docker23.png]]

### Start Rootless Docker
- Run 3 command below.  we don't use sudo start  docker because i run in user mode

```bash
systemctl --user start docker
systemctl --user enable docker
systemctl --user status docker
```

![[install-docker24.png]]
### Verify Rootless Mode

```bash
docker run hello-world
```

![[install-docker25.png]]

- docker download image again because the previous image is difference name space

## 3 Play with Container
- Download official image and create create container 

```bash
# pull image
docker pull quay.io/centos/centos:stream10
```
![[install-docker26.png]]

```bash
# Run image or start container from image then run /bin/bash command in that container
# -it simply get terminal from container
docker run -it quay.io/centos/centos:stream10 /bin/bash
```

![[install-docker27.png]]

-  docker container use container id as  hostname
- we use exit command to exit from running container. it will totally exit and running container will exit.

## We will exit  from docker container and container will still runing

- Run docker container from images again
- stop with Ctrl + p then  ``Ctrl+q``

```
docker run -it quay.io/centos/centos:stream10 /bin/bash
Ctrl +p  then Ctrl+q
```

![[install-docker28.png]]

![[install-docker29.png]]

## Attach to running container
- Now we can reuse process that running in background 

```bash
## Attach is simple
docker attach <containerid>

Ctrl + p then Ctrl+q  
```

![[install-docker30.png]]

## Kill running Container and remove container

![[install-docker31.png]]


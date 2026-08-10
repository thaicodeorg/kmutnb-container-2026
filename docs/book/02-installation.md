!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/02-installation.html)

# Chapter 2: Docker CE Installation on CentOS 9 Stream

## Overview

This chapter covers the complete installation process of Docker Community Edition (CE) on CentOS 9 Stream, including rootless mode configuration.

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

---

## 2.2 Install yum-utils

Install the `yum-utils` package which provides the `yum-config-manager` utility:

```bash
sudo dnf install -y yum-utils
```

---

## 2.3 Add Docker CE Repository

Add the official Docker CE repository for CentOS/RHEL:

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Verify the repository was added:

```bash
dnf repolist | grep docker-ce
```

---

## 2.4 Install Docker CE Packages

Install Docker CE, CLI, containerd.io, Buildx, and Compose plugin:

```bash
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## 2.5 Start and Enable Docker Service

Start the Docker daemon and enable it to start on boot:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Verify the service is running:

```bash
sudo systemctl status docker
```

---

## 2.6 Verify Installation

Run the hello-world container to verify Docker is working correctly:

```bash
sudo docker run hello-world
```

Check the Docker version:

```bash
docker --version
docker compose version
```

---

## 2.7 Add Current User to Docker Group

Avoid using `sudo` for every Docker command by adding your user to the `docker` group:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Verify you can run Docker without sudo:

```bash
docker run hello-world
```

---

## 2.8 Rootless Mode Setup (Optional)

Docker can run in rootless mode for enhanced security:

### Prerequisites

```bash
sudo dnf install -y fuse-overlayfs slirp4netns uidmap
```

### Install Docker in Rootless Mode

```bash
# Disable root Docker first
sudo systemctl disable --now docker

# Install rootless Docker
dockerd-rootless-setuptool.sh install
```

### Set Environment Variables

```bash
export PATH=/usr/bin:$PATH
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock
```

### Start Rootless Docker

```bash
systemctl --user start docker
systemctl --user enable docker
```

### Verify Rootless Mode

```bash
docker run hello-world
```

---

## 2.9 Configure Docker Daemon

Edit the Docker daemon configuration file:

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "storage-driver": "overlay2"
}
EOF
```

Restart Docker to apply changes:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

## Summary

| Step | Command |
|------|---------|
| Remove old versions | `dnf remove docker*` |
| Install yum-utils | `dnf install yum-utils` |
| Add repository | `yum-config-manager --add-repo ...` |
| Install Docker CE | `dnf install docker-ce ...` |
| Start service | `systemctl start docker` |
| Enable on boot | `systemctl enable docker` |
| Add user to group | `usermod -aG docker $USER` |

---

## Quiz

??? question "Question 1: What command removes old Docker versions before a fresh install?"
    **Answer:**
    
    `sudo dnf remove docker docker-client docker-engine`

??? question "Question 2: Which utility package provides yum-config-manager?"
    **Answer:**
    
    `yum-utils` — install with `sudo dnf install -y yum-utils`

??? question "Question 3: What is the URL of the official Docker CE repository for CentOS?"
    **Answer:**
    
    `https://download.docker.com/linux/centos/docker-ce.repo`

??? question "Question 4: Name the five main Docker CE packages to install."
    **Answer:**
    
    `docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin`, `docker-compose-plugin`

??? question "Question 5: What two commands start and enable the Docker service on boot?"
    **Answer:**
    
    `sudo systemctl start docker` and `sudo systemctl enable docker`

??? question "Question 6: How do you add your current user to the docker group to avoid using sudo?"
    **Answer:**
    
    `sudo usermod -aG docker $USER` then `newgrp docker`

??? question "Question 7: What prerequisite packages are needed for rootless Docker mode?"
    **Answer:**
    
    `fuse-overlayfs`, `slirp4netns`, and `uidmap`

??? question "Question 8: What is the path to the Docker daemon configuration file?"
    **Answer:**
    
    `/etc/docker/daemon.json`

??? question "Question 9: Which log rotation settings should be configured in daemon.json?"
    **Answer:**
    
    `"max-size": "10m"` and `"max-file": "3"` under the `log-opts` key

??? question "Question 10: What command verifies Docker is installed and working correctly?"
    **Answer:**
    
    `sudo docker run hello-world` — it pulls and runs the hello-world image

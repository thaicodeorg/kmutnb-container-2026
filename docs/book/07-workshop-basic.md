!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/07-workshop-basic.html)

# Chapter 7: Workshop - Basic Operations

## Overview

This hands-on workshop covers basic Docker operations using CentOS 9 Stream containers.

---

## Lab 1: Run CentOS Container Interactively

### Step 1: Pull the CentOS 9 Stream Image

```bash
docker pull quay.io/centos/centos:stream9
```

### Step 2: Run Interactive Container

```bash
docker run -it --name mycentos quay.io/centos/centos:stream9 /bin/bash
```

### Step 3: Explore the Container

Inside the container, run these commands:

```bash
# Check OS version
cat /etc/os-release

# Check kernel
uname -a

# Check hostname
hostname

# Check environment variables
env

# Check current user
whoami
id
```

### Step 4: Exit the Container

```bash
exit
```

---

## Lab 2: Check OS Inside Container

### Step 1: Run Container with Command

```bash
docker run --rm quay.io/centos/centos:stream9 cat /etc/os-release
```

### Step 2: Check CentOS Version

```bash
docker run --rm quay.io/centos/centos:stream9 rpm -q centos-stream-release
```

### Step 3: List Installed Packages

```bash
docker run --rm quay.io/centos/centos:stream9 rpm -qa | head -20
```

---

## Lab 3: List, Stop, and Remove Containers

### Step 1: List All Containers

```bash
docker ps -a
```

### Step 2: Stop Running Container

```bash
docker stop mycentos
```

### Step 3: Verify Container is Stopped

```bash
docker ps -a
```

### Step 4: Start Container Again

```bash
docker start mycentos
docker ps
```

### Step 5: Remove the Container

```bash
docker stop mycentos
docker rm mycentos
docker ps -a
```

---

## Lab 4: Container Lifecycle

### Step 1: Create Container Without Starting

```bash
docker create --name test-container quay.io/centos/centos:stream9 echo "Hello from container"
```

### Step 2: Inspect Container

```bash
docker inspect test-container
```

### Step 3: Start Container

```bash
docker start test-container
docker logs test-container
```

### Step 4: Remove Container

```bash
docker rm test-container
```

---

## Lab 5: Execute Commands in Running Container

### Step 1: Run Container in Background

```bash
docker run -d --name webserver nginx:latest
```

### Step 2: Execute Command

```bash
docker exec webserver cat /etc/nginx/nginx.conf
```

### Step 3: Open Interactive Shell

```bash
docker exec -it webserver /bin/bash
```

Inside the container:

```bash
ls /etc/nginx/
cat /etc/nginx/conf.d/default.conf
exit
```

### Step 4: Clean Up

```bash
docker stop webserver
docker rm webserver
```

---

## Lab 6: Port Mapping

### Step 1: Run Nginx with Port Mapping

```bash
docker run -d --name mynginx -p 8080:80 nginx:latest
```

### Step 2: Test Access

```bash
curl http://localhost:8080
```

### Step 3: Check Port Mapping

```bash
docker port mynginx
```

### Step 4: Clean Up

```bash
docker stop mynginx
docker rm mynginx
```

---

## Lab 7: Container with Resource Limits

### Step 1: Run Container with Memory Limit

```bash
docker run -d --name limited --memory=100m quay.io/centos/centos:stream9 sleep 3600
```

### Step 2: Check Resource Usage

```bash
docker stats limited --no-stream
```

### Step 3: Clean Up

```bash
docker stop limited
docker rm limited
```

---

## Summary of Commands Used

| Command | Description |
|---------|-------------|
| `docker run -it` | Run interactive container |
| `docker ps -a` | List all containers |
| `docker stop` | Stop a container |
| `docker rm` | Remove a container |
| `docker exec -it` | Execute command in container |
| `docker logs` | View container logs |
| `docker inspect` | Inspect container details |
| `docker port` | Show port mappings |
| `docker stats` | View resource usage |

---

## Quiz

??? question "Question 1: What flag allows interactive access to a container?"
    **Answer:**
    
    `-it` — combined flags for interactive terminal and pseudo-TTY

??? question "Question 2: How do you list all containers including stopped ones?"
    **Answer:**
    
    `docker ps -a`

??? question "Question 3: What command removes a stopped container?"
    **Answer:**
    
    `docker rm <container_name>`

??? question "Question 4: How do you run a command inside a running container?"
    **Answer:**
    
    `docker exec <container_name> <command>`

??? question "Question 5: What does the `--rm` flag do when running a container?"
    **Answer:**
    
    Automatically removes the container when it exits

??? question "Question 6: How do you map host port 8080 to container port 80?"
    **Answer:**
    
    `docker run -d -p 8080:80 nginx:latest`

??? question "Question 7: What command shows real-time resource usage of containers?"
    **Answer:**
    
    `docker stats`

??? question "Question 8: How do you view the logs of a running container?"
    **Answer:**
    
    `docker logs <container_name>`

??? question "Question 9: What is the difference between `docker stop` and `docker rm`?"
    **Answer:**
    
    `stop` halts a running container; `rm` deletes a stopped container

??? question "Question 10: How do you check the OS version inside a container?"
    **Answer:**
    
    `docker run --rm centos:stream9 cat /etc/os-release`

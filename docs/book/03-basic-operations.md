
# Chapter 3: Docker Basic Operations

## Overview

This chapter covers the fundamental Docker operations: managing images, containers, networks, and volumes.

---

## 3.1 Docker Images

### List Images

```bash
docker images
docker image ls
```

### Pull an Image

```bash
docker pull centos:stream10
docker pull nginx:latest
docker pull python:3.11-slim
```

### Search for Images

```bash
docker search centos
docker search --limit 5 nginx
```

### Build an Image

```bash
docker build -t myapp:1.0 .
docker build -f Dockerfile.prod -t myapp:prod .
```

### Remove an Image

```bash
docker rmi <image_id>
docker rmi -f <image_id>
docker image prune -a
```

---

## 3.2 Docker Containers

### Run a Container

```bash
# Interactive terminal
docker run -it centos:stream9 /bin/bash

# Detached mode
docker run -d nginx:latest

# With a name
docker run -d --name webserver nginx:latest

# Remove container after exit
docker run --rm centos:stream9 cat /etc/os-release
```

### List Containers

```bash
docker ps              # Running containers
docker ps -a           # All containers
docker ps -q           # Container IDs only
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

### Stop and Start Containers

```bash
docker stop <container_id>
docker stop webserver
docker start <container_id>
docker restart <container_id>
```

### Remove Containers

```bash
docker rm <container_id>
docker rm -f <container_id>
docker container prune
```

### Execute Commands in Running Container

```bash
docker exec -it <container_id> /bin/bash
docker exec -it webserver /bin/sh
docker exec webserver cat /etc/nginx/nginx.conf
```

### View Container Logs

```bash
docker logs <container_id>
docker logs -f <container_id>     # Follow logs
docker logs --tail 100 webserver  # Last 100 lines
docker logs --since 30m webserver # Last 30 minutes
```

---

## 3.3 Port Mapping

### Basic Port Mapping

```bash
# Host port 8080 -> Container port 80
docker run -d -p 8080:80 nginx:latest

# Map to specific network interface
docker run -d -p 127.0.0.1:8080:80 nginx:latest

# Random host port
docker run -d -p 80 nginx:latest

# Multiple port mappings
docker run -d -p 8080:80 -p 8443:443 nginx:latest
```

### Check Port Mappings

```bash
docker port <container_id>
docker inspect --format='{{.NetworkSettings.Ports}}' <container_id>
```

---

## 3.4 Docker Networks

### List Networks

```bash
docker network ls
docker network ls --format "table {{.ID}}\t{{.Name}}\t{{.Driver}}"
```

### Create a Network

```bash
# Bridge network
docker network create mybridge

# Custom subnet
docker network create --subnet=172.20.0.0/16 mynetwork

# With driver options
docker network create --driver bridge --opt com.docker.network.bridge.name=br-mynet mynet
```

### Connect Container to Network

```bash
docker network connect mybridge <container_id>
docker network connect --ip 172.20.0.10 mybridge webserver
```

### Disconnect Container from Network

```bash
docker network disconnect mybridge <container_id>
```

### Inspect Network

```bash
docker network inspect mybridge
```

### Remove Network

```bash
docker network rm mybridge
docker network prune
```

---

## 3.5 Docker Volumes

### Create a Volume

```bash
docker volume create mydata
docker volume create --driver local --opt type=tmpfs --opt device=tmpfs mytmp
```

### List Volumes

```bash
docker volume ls
docker volume ls --format "table {{.Name}}\t{{.Driver}}"
```

### Use Volumes with Containers

```bash
# Named volume
docker run -d -v mydata:/var/lib/data centos:stream9

# Bind mount
docker run -d -v /host/path:/container/path nginx:latest

# Read-only mount
docker run -d -v /host/config:/etc/config:ro nginx:latest
```

### Inspect Volume

```bash
docker volume inspect mydata
```

### Remove Volumes

```bash
docker volume rm mydata
docker volume prune
```

---

## Summary

| Operation | Command |
|-----------|---------|
| Pull image | `docker pull <image>` |
| Run container | `docker run -d <image>` |
| List containers | `docker ps -a` |
| Stop container | `docker stop <id>` |
| Exec into container | `docker exec -it <id> /bin/bash` |
| Create network | `docker network create <name>` |
| Create volume | `docker volume create <name>` |
| Port mapping | `-p host:container` |

---

## Quiz

??? question "Question 1: What flag runs a container in detached (background) mode?"
    **Answer:**
    
    The `-d` flag, as in `docker run -d nginx`

??? question "Question 2: How do you list all containers including stopped ones?"
    **Answer:**
    
    `docker ps -a`

??? question "Question 3: What command executes an interactive shell inside a running container?"
    **Answer:**
    
    `docker exec -it <container_id> /bin/bash`

??? question "Question 4: How do you map host port 8080 to container port 80?"
    **Answer:**
    
    `docker run -d -p 8080:80 nginx:latest`

??? question "Question 5: What are the three types of Docker network drivers?"
    **Answer:**
    
    `bridge`, `host`, and `overlay` (plus `none` and `macvlan`)

??? question "Question 6: What is the difference between a named volume and a bind mount?"
    **Answer:**
    
    Named volumes are managed by Docker (`-v mydata:/path`), bind mounts use a host path (`-v /host:/container`)

??? question "Question 7: How do you view the last 50 log lines from a container?"
    **Answer:**
    
    `docker logs --tail 50 <container_id>`

??? question "Question 8: What command removes all unused Docker volumes?"
    **Answer:**
    
    `docker volume prune`

??? question "Question 9: How do you connect a running container to an existing network?"
    **Answer:**
    
    `docker network connect <network_name> <container_id>`

??? question "Question 10: What does `docker container prune` do?"
    **Answer:**
    
    It removes all stopped containers from the system


# Fundamentals

## What are Containers?

Containers are lightweight, standalone, executable packages that include everything needed to run a piece of software, including:

- Code
- Runtime
- System tools
- System libraries
- Settings

Containers are isolated from each other and bundle their own software, libraries, and configuration files. They communicate through well-defined channels.

## VMs vs Containers

| Feature | Virtual Machines | Containers |
|---------|------------------|------------|
| **Isolation** | Hardware-level | OS kernel-level |
| **Size** | Gigabytes | Megabytes |
| **Startup** | Minutes | Seconds |
| **Performance** | Near native | Near native |
| **OS** | Each VM has full OS | Shared host OS kernel |

### Key Differences

**Virtual Machines:**
- Virtualize hardware
- Each VM runs a complete OS
- Require more resources
- Slower to start

**Containers:**
- Virtualize OS kernel
- Share host OS kernel
- Lightweight and fast
- Efficient resource usage

## Docker Architecture

Docker uses a client-server architecture with three main components:

### Docker Client

The Docker client is the primary way users interact with Docker. It communicates with the Docker daemon.

```bash
docker run hello-world
```

### Docker Daemon (dockerd)

The Docker daemon listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.

### Docker Registry

A Docker registry stores Docker images. Docker Hub is a public registry that anyone can use.

```
┌─────────────────┐
│  Docker Client  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Docker Daemon  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Docker Registry│
└─────────────────┘
```

## Images and Containers

### Images

An image is a read-only template with instructions for creating a Docker container. Images are built from a Dockerfile.

### Containers

A container is a runnable instance of an image. You can create, start, stop, move, or delete a container.

```bash
# Create and run a container
docker run -d --name my-nginx nginx

# List running containers
docker ps

# Stop a container
docker stop my-nginx
```

---

## Quiz

??? question "Question 1: What is a container?"
    **Answer:**
    
    A lightweight, standalone, executable package that includes everything needed to run a piece of software, including code, runtime, system tools, libraries, and settings.

??? question "Question 2: What is the main difference between a VM and a container?"
    **Answer:**
    
    VMs virtualize hardware and each runs a complete OS, while containers virtualize the OS kernel and share the host OS kernel, making them lighter and faster.

??? question "Question 3: What are the three main components of Docker architecture?"
    **Answer:**
    
    Docker Client, Docker Daemon (dockerd), and Docker Registry.

??? question "Question 4: What is Docker Hub?"
    **Answer:**
    
    Docker Hub is a public Docker registry where users can store and share Docker images.

??? question "Question 5: What command lists all running containers?"
    **Answer:**
    
    `docker ps`

??? question "Question 6: What is an image in Docker?"
    **Answer:**
    
    An image is a read-only template with instructions for creating a Docker container. Images are built from a Dockerfile.

??? question "Question 7: How do you stop a running container?"
    **Answer:**
    
    Using the `docker stop` command followed by the container name or ID.

??? question "Question 8: Why are containers faster to start than VMs?"
    **Answer:**
    
    Because containers share the host OS kernel and don't need to boot a complete operating system, they can start in seconds compared to minutes for VMs.

??? question "Question 9: What is the Docker daemon?"
    **Answer:**
    
    The Docker daemon (dockerd) is a service that listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes.

??? question "Question 10: Can containers communicate with each other?"
    **Answer:**
    
    Yes, containers can communicate through well-defined channels, typically using Docker networks.

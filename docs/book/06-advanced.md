!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/06-advanced.html)

# Chapter 6: Advanced Docker Topics

## Overview

This chapter covers Docker security, logging, monitoring, Docker Swarm, and an introduction to Kubernetes.

---

## 6.1 Docker Security

### Rootless Mode

Run Docker daemon as a non-root user:

```bash
# Install prerequisites
sudo dnf install -y fuse-overlayfs slirp4netns uidmap

# Disable root Docker
sudo systemctl disable --now docker

# Install rootless mode
dockerd-rootless-setuptool.sh install

# Set environment
export DOCKER_HOST=unix:///run/user/$(id -u)/docker.sock
```

### User Namespaces

Remap container root to unprivileged host user:

```json
// /etc/docker/daemon.json
{
    "userns-remap": "default"
}
```

### Seccomp Profiles

限制容器系统调用:

```json
{
    "defaultAction": "SCMP_ACT_ERRNO",
    "architectures": ["SCMP_ARCH_X86_64"],
    "syscalls": [
        {
            "names": ["read", "write", "open", "close"],
            "action": "SCMP_ACT_ALLOW"
        }
    ]
}
```

Apply seccomp profile:

```bash
docker run --security-opt seccomp=profile.json centos:stream9
```

### AppArmor

```bash
docker run --security-opt apparmor=docker-default centos:stream9
```

### Read-Only Filesystem

```bash
docker run --read-only --tmpfs /tmp centos:stream9
```

### Resource Limits

```bash
docker run -d \
    --memory=512m \
    --cpus=1.5 \
    --pids-limit=100 \
    centos:stream9
```

---

## 6.2 Logging Drivers

### JSON File (Default)

```json
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    }
}
```

### Syslog

```bash
docker run -d \
    --log-driver=syslog \
    --log-opt syslog-address=tcp://logserver:514 \
    nginx:latest
```

### Fluentd

```bash
docker run -d \
    --log-driver=fluentd \
    --log-opt fluentd-address=localhost:24224 \
    nginx:latest
```

### View Container Logs

```bash
docker logs <container_id>
docker logs -f <container_id>
docker logs --since 1h <container_id>
docker logs --tail 50 <container_id>
```

---

## 6.3 Monitoring with Docker

### docker stats

Real-time container resource usage:

```bash
docker stats
docker stats --no-stream
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### Inspect Container Details

```bash
docker inspect <container_id>
docker inspect --format='{{.State.Status}}' <container_id>
docker inspect --format='{{.NetworkSettings.IPAddress}}' <container_id>
```

### System Information

```bash
docker system df
docker system df -v
docker info
```

### Clean Up Resources

```bash
docker system prune -a
docker system prune --volumes
```

---

## 6.4 Docker Swarm

### Initialize Swarm

```bash
docker swarm init --advertise-addr 192.168.1.100
```

### Join Swarm

```bash
# On worker node
docker swarm join --token <token> 192.168.1.100:2377
```

### Deploy Service

```bash
# Create service
docker service create --name web --replicas 3 -p 80:80 nginx:latest

# List services
docker service ls

# Scale service
docker service scale web=5

# Update service
docker service update --image nginx:1.25 web

# Remove service
docker service rm web
```

### Swarm Stack

```yaml
# docker-stack.yml
version: "3.8"
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```

```bash
docker stack deploy -c docker-stack.yml myapp
docker stack ls
docker stack services myapp
```

---

## 6.5 Kubernetes Overview

### Key Concepts

| Concept | Description |
|---------|-------------|
| Pod | Smallest deployable unit, one or more containers |
| Service | Exposes Pods to network traffic |
| Deployment | Manages ReplicaSets and Pods |
| Namespace | Virtual cluster isolation |
| ConfigMap | Non-confidential configuration data |
| Secret | Confidential data (base64 encoded) |

### Pod Example

```yaml
# pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

### Deployment Example

```yaml
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
```

### Service Example

```yaml
# service.yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

### kubectl Commands

```bash
# Apply configuration
kubectl apply -f deployment.yml

# Get resources
kubectl get pods
kubectl get services
kubectl get deployments

# Describe resource
kubectl describe pod nginx-pod

# View logs
kubectl logs nginx-pod

# Delete resource
kubectl delete -f deployment.yml
```

---

## Summary

| Topic | Key Command/Concept |
|-------|---------------------|
| Rootless Docker | `dockerd-rootless-setuptool.sh install` |
| Seccomp | `--security-opt seccomp=profile.json` |
| Resource Limits | `--memory=512m --cpus=1.5` |
| Logging | `--log-driver=syslog` |
| Monitoring | `docker stats` |
| Swarm Init | `docker swarm init` |
| K8s Deploy | `kubectl apply -f deploy.yml` |

---

## Quiz

??? question "Question 1: What is Docker rootless mode?"
    **Answer:**
    
    Running the Docker daemon as a non-root user, improving security by reducing the attack surface

??? question "Question 2: What does the `--memory` flag do when running a container?"
    **Answer:**
    
    Sets the maximum memory limit the container can use, e.g., `--memory=512m`

??? question "Question 3: What is the default Docker logging driver?"
    **Answer:**
    
    `json-file`

??? question "Question 4: How do you view real-time container resource usage?"
    **Answer:**
    
    `docker stats` shows CPU, memory, network, and disk usage

??? question "Question 5: What command initializes a Docker Swarm cluster?"
    **Answer:**
    
    `docker swarm init --advertise-addr <IP_ADDRESS>`

??? question "Question 6: What is the difference between a Pod and a Deployment in Kubernetes?"
    **Answer:**
    
    A Pod is the smallest deployable unit; a Deployment manages Pods and provides rollback, scaling, and updates

??? question "Question 7: What does `docker system prune -a` do?"
    **Answer:**
    
    Removes all unused containers, networks, images (dangling and unreferenced), and optionally build cache

??? question "Question 8: What is a Seccomp profile in Docker?"
    **Answer:**
    
    A security profile that restricts which system calls a container can make

??? question "Question 9: How do you scale a Docker Swarm service?"
    **Answer:**
    
    `docker service scale <service_name>=<replica_count>`

??? question "Question 10: What Kubernetes resource exposes Pods to network traffic?"
    **Answer:**
    
    A `Service` — it provides stable network access to a set of Pods

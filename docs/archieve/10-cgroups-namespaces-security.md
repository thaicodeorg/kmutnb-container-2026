!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/10-cgroups-namespaces.html)

# Cgroups, Namespaces & Security

## Linux Kernel Features

Docker containers are built on two fundamental Linux kernel features:

### Control Groups (cgroups)

Control groups limit and isolate resource usage (CPU, memory, disk I/O, network) of a collection of processes.

```bash
# View cgroup version
cat /proc/filesystems | grep cgroup

# View cgroup v2 mount
mount | grep cgroup
```

### Namespaces

Namespaces provide isolated views of system resources (PID, network, mount, user, hostname).

```bash
# View namespaces
ls -la /proc/self/ns/

# View process namespaces
lsns
```

---

## Control Groups (cgroups)

### What are Cgroups?

Cgroups control how much CPU, memory, and I/O a container can use.

```bash
# Create a cgroup
sudo mkdir /sys/fs/cgroup/mycontainer

# Limit CPU (10% = 10000/100000)
echo "10000 100000" | sudo tee /sys/fs/cgroup/mycontainer/cpu.max

# Limit memory (100MB)
echo "104857600" | sudo tee /sys/fs/cgroup/mycontainer/memory.max

# Run container with resource limits
docker run -d --cpus="0.1" --memory="100m" nginx
```

### Cgroup Resources

| Resource | Control | Example |
|----------|---------|---------|
| CPU | `--cpus` | `--cpus="0.5"` (50% of one core) |
| Memory | `--memory` | `--memory="512m"` |
| Disk I/O | `--device-read-bps` | `--device-read-bps=/dev/sda:10mb` |
| Network | `--net` | `--net=host` or custom network |
| PIDs | `--pids-limit` | `--pids-limit=100` |

---

## Namespaces

### Types of Namespaces

| Namespace | Isolates |
|-----------|----------|
| **PID** | Process IDs |
| **NET** | Network interfaces, IPs, routes |
| **MNT** | Filesystem mount points |
| **UTS** | Hostname and domain name |
| **IPC** | Inter-process communication |
| **USER** | User and group IDs |
| **CGROUP** | Cgroup root directory |

### View Namespaces

```bash
# List all namespaces
lsns

# List specific namespace type
lsns -t pid
lsns -t net

# View process namespaces
cat /proc/self/status | grep NSpid
```

### PID Namespace Example

```bash
# Run container and see PID 1 inside
docker run --rm -it centos:stream9 ps aux

# Compare with host
ps aux | head -5
```

---

## Container Security

### Security Layers

```
┌─────────────────────────────┐
│     Application Security    │
├─────────────────────────────┤
│     Container Security      │
├─────────────────────────────┤
│     Image Security          │
├─────────────────────────────┤
│     Host Security           │
├─────────────────────────────┤
│     Kernel Security         │
└─────────────────────────────┘
```

### Docker Security Features

```bash
# Run as non-root user
docker run --user 1000:1000 nginx

# Read-only filesystem
docker run --read-only nginx

# Drop all capabilities
docker run --cap-drop=ALL nginx

# Add specific capability
docker run --cap-add=NET_ADMIN nginx

# No new privileges
docker run --security-opt=no-new-privileges nginx
```

---

## Advanced Security

### Seccomp Profiles

Seccomp (Secure Computing Mode) filters system calls a container can make.

```bash
# Run with default seccomp profile
docker run --security-opt seccomp=default nginx

# Run with custom profile
docker run --security-opt seccomp=myprofile.json nginx

# Disable seccomp (not recommended)
docker run --security-opt seccomp=unconfined nginx
```

### AppArmor Profiles

AppArmor restricts program capabilities with per-program profiles.

```bash
# Run with AppArmor profile
docker run --security-opt apparmor=myprofile nginx

# View container security options
docker inspect --format='{{.HostConfig.SecurityOpt}}' <container>
```

### User Namespaces

```bash
# Enable user namespace remapping
# Edit /etc/docker/daemon.json
{
    "userns-remap": "default"
}

# Restart Docker
sudo systemctl restart docker

# Verify
docker info | grep -i "user namespace"
```

---

## Image Security

### Best Practices

```dockerfile
# Use specific version tags
FROM python:3.11.5-slim

# Run as non-root user
RUN useradd -r -s /bin/false appuser
USER appuser

# Use COPY instead of ADD
COPY --chown=appuser:appuser . .

# Scan for vulnerabilities
# docker scout cves <image>
```

### Docker Scout

```bash
# Enable Docker Scout
docker scout quickview <image>

# View CVEs
docker scout cves <image>

# Compare images
docker scout compare <image1> <image2>
```

---

## Security Scanning Tools

| Tool | Purpose |
|------|---------|
| **Docker Scout** | Built-in vulnerability scanning |
| **Trivy** | Open-source vulnerability scanner |
| **Grype** | Vulnerability scanner for containers |
| **Snyk** | Developer-first security |
| **Clair** | Static vulnerability analysis |

```bash
# Scan with Trivy
trivy image nginx:latest

# Scan with Grype
grype nginx:latest

# Scan with Docker Scout
docker scout cves nginx:latest
```

---

## Runtime Security

### Docker Content Trust

```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Push signed image
docker push myregistry/myimage:latest

# Pull only signed images
docker pull myregistry/myimage:latest
```

### Audit Logging

```bash
# Enable audit logging
sudo auditctl -w /usr/bin/docker -p rwxa -k docker

# View audit logs
sudo ausearch -k docker
```

### Resource Limits

```bash
# Set memory limit
docker run -d --memory=512m nginx

# Set CPU limit
docker run -d --cpus=1.5 nginx

# Set PID limit
docker run -d --pids-limit=100 nginx

# Combine limits
docker run -d \
    --memory=256m \
    --cpus=0.5 \
    --pids-limit=50 \
    nginx
```

---

## Best Practices Summary

| Area | Practice |
|------|----------|
| **Images** | Use minimal base images, scan for CVEs |
| **Runtime** | Run as non-root, drop capabilities |
| **Network** | Use bridge network, expose only needed ports |
| **Storage** | Use read-only where possible |
| **Secrets** | Use Docker secrets or external vault |
| **Updates** | Regularly rebuild images with latest patches |

---

## Quiz

??? question "Question 1: What do cgroups control?"
    **Answer:**
    
    Cgroups control and limit resource usage including CPU, memory, disk I/O, and network bandwidth.

??? question "Question 2: What is the purpose of PID namespace?"
    **Answer:**
    
    PID namespace isolates the process ID number space, so processes in different namespaces can have the same PID.

??? question "Question 3: What is the difference between --cap-add and --security-opt?"
    **Answer:**
    
    --cap-add adds Linux capabilities to a container, while --security-opt configures security policies like AppArmor and seccomp.

??? question "Question 4: Why should you run containers as non-root?"
    **Answer:**
    
    Running as non-root limits the container's ability to access host resources and reduces the attack surface if the container is compromised.

??? question "Question 5: What is Docker Content Trust?"
    **Answer:**
    
    Docker Content Trust ensures image integrity by using digital signatures to verify the publisher and content of images.

??? question "Question 6: What is seccomp in Docker?"
    **Answer:**
    
    Seccomp (Secure Computing Mode) filters the system calls a container can make to the host kernel, limiting potential attacks.

??? question "Question 7: How do you enable user namespace remapping?"
    **Answer:**
    
    Add "userns-remap": "default" to /etc/docker/daemon.json and restart Docker.

??? question "Question 8: What is the purpose of --read-only flag?"
    **Answer:**
    
    --read-only makes the container's filesystem read-only, preventing write operations except for explicitly mounted volumes.

??? question "Question 9: Name three security scanning tools for Docker images."
    **Answer:**
    
    Docker Scout, Trivy, Grype, Snyk, or Clair.

??? question "Question 10: What does --pids-limit restrict?"
    **Answer:**
    
    --pids-limit restricts the number of processes that can be created inside a container, preventing fork bombs.

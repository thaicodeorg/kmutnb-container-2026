# Chapter 6: Docker Advanced (Progressive)

## Overview
### Objectives
- Build optimized Docker images with multi-stage builds
- Understand Docker overlay filesystem and layer management
- Configure container resource limits
- Monitor containers with logging and inspection
- Configure Docker networking
- Deploy to private registry with security scanning

---

## Assignment: Create Student's Submission along tutorial
- Student will submit Screen of Docker installation process according to step-by-step from manual.
- Student can use Basic Template [Download](../assets/dockerbasic-advanced-progressive-submission.docx) or generate own version.

---

## Workshop 1: Optimized Builds

### Lab 1.1: Multi-stage Build - Python Application

#### Step 1: Create project directory
```bash
mkdir ~/python-multistage
cd ~/python-multistage
```

#### Step 2: Create app.py and requirements.txt
```bash
cat > app.py << 'EOF'
from flask import Flask
import os
import platform

app = Flask(__name__)

@app.route("/")
def hello():
    return f"""
    <h1>Hello from Python container!</h1>
    <p>Python Version: {platform.python_version()}</p>
    <p>Hostname: {os.uname().nodename}</p>
    <p>OS: {os.uname().sysname} {os.uname().release}</p>
    """

@app.route("/health")
def health():
    return "OK", 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
EOF

cat > requirements.txt << 'EOF'
flask==3.0.0
gunicorn==21.2.0
EOF
```

#### Step 3: Create single-stage Dockerfile
```bash
cat > Dockerfile.single << 'EOF'
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
EOF
```

#### Step 4: Build and check image size
```bash
docker build -t python-single -f Dockerfile.single .
docker images python-single
```
> **Note:** Single-stage image is ~900MB with full Python distribution

#### Step 5: Create multi-stage Dockerfile
```bash
cat > Dockerfile << 'EOF'
# Build stage
FROM python:3.11 AS builder

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Final stage
FROM python:3.11-slim

WORKDIR /app

COPY --from=builder /install /usr/local
COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
EOF
```

#### Step 6: Build and compare sizes
```bash
docker build -t python-multi .
docker images | grep python-
```
> **Note:** Multi-stage image is ~150MB - much smaller!

#### Step 7: Verify application works
```bash
docker run -d --name python-app -p 5000:5000 python-multi
curl http://localhost:5000
docker stop python-app
docker rm python-app
```

---

### Lab 1.2: BuildKit Advanced Features

#### Step 1: Enable BuildKit
```bash
export DOCKER_BUILDKIT=1
```

#### Step 2: Create Dockerfile with cache mounts
```bash
cat > Dockerfile.buildkit << 'EOF'
FROM python:3.11 AS builder

WORKDIR /app

COPY requirements.txt .

# Use cache mount for pip
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --prefix=/install -r requirements.txt

FROM python:3.11-slim

COPY --from=builder /install /usr/local
COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
EOF
```

#### Step 3: Build with BuildKit
```bash
time docker build -t python-buildkit -f Dockerfile.buildkit .
time docker build -t python-buildkit-cache -f Dockerfile.buildkit .
```
> **Note:** Second build is faster due to pip cache mount

#### Step 4: Use secret mounts
```bash
echo "secret-api-key-12345" > secrets.txt

cat > Dockerfile.secret << 'EOF'
FROM python:3.11-slim

# Copy secret during build (not in final image)
RUN --mount=type=secret,id=api_key \
    cat /run/secrets/api_key > /tmp/secret.txt && \
    rm /tmp/secret.txt

CMD ["echo", "Secret build completed"]
EOF

DOCKER_BUILDKIT=1 docker build --secret id=api_key,src=secrets.txt -t python-secret .
```

#### Step 5: Verify secret not in image
```bash
docker history python-secret
```

---

### Lab 1.3: Image Optimization

#### Step 1: Create .dockerignore file
```bash
cat > .dockerignore << 'EOF'
.git
.gitignore
*.md
*.txt
!requirements.txt
secrets.txt
Dockerfile*
docker-compose*
.vscode
.idea
__pycache__
*.pyc
*.pyo
.env
.venv
venv
EOF
```

#### Step 2: Order Dockerfile instructions
```bash
cat > Dockerfile.optimized << 'EOF'
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install dependencies first (better cache utilization)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code last (changes most frequently)
COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
EOF
```

#### Step 3: Combine RUN commands
```bash
cat > Dockerfile.combined << 'EOF'
FROM python:3.11-slim

WORKDIR /app

# Combine multiple RUN commands into one
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
EOF
```

#### Step 4: Use minimal base images
```bash
# Compare different base images
docker pull python:3.11
docker pull python:3.11-slim
docker pull python:3.11-alpine
docker images | grep python
```

#### Step 5: Analyze with docker history
```bash
docker build -t python-optimized -f Dockerfile.optimized .
docker history python-optimized
```

---

## Workshop 2: Runtime Management

### Lab 2.1: Overlay Filesystem Deep Dive

#### Step 1: Run container and examine /proc/mounts
```bash
docker run -it --name overlay-deep quay.io/centos/centos:stream10 /bin/bash
cat /proc/mounts | grep overlay
mount | grep overlay
```

#### Step 2: View overlay layers
```bash
# Inside container
df -h
ls -la /
```

#### Step 3: Create and modify files
```bash
# Inside container
mkdir -p /data
echo "Original content" > /data/file.txt
cat /data/file.txt
echo "Modified content" >> /data/file.txt
cat /data/file.txt
```

#### Step 4: Exit and examine layer changes
```bash
exit

# On host
docker diff overlay-deep
```

#### Step 5: Understand Copy-on-Write
```bash
# Create snapshot
docker commit overlay-deep overlay-snapshot

# Compare layers
docker history overlay-deep
docker history overlay-snapshot
```

---

### Lab 2.2: Resource Constraints

#### Step 1: Set memory limits with --memory
```bash
# Run container with 256MB memory limit
docker run -d --name mem-limit \
  --memory=256m \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

# Monitor memory usage
docker stats mem-limit --no-stream
```

#### Step 2: Set CPU limits with --cpus
```bash
# Run container with 0.5 CPU limit
docker run -d --name cpu-limit \
  --cpus=0.5 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

# Monitor CPU usage
docker stats cpu-limit --no-stream
```

#### Step 3: Set PID limits with --pids-limit
```bash
# Run container with 50 process limit
docker run -d --name pid-limit \
  --pids-limit=50 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

# Check PID limit
docker inspect --format='{{.HostConfig.PidsLimit}}' pid-limit
```

#### Step 4: Monitor with docker stats
```bash
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.PIDs}}"
```

#### Step 5: Use cgroup v2 features
```bash
# View cgroup information
docker inspect --format='{{.HostConfig.Cgroupns}}' mem-limit

# Check cgroup version on host
cat /proc/filesystems | grep cgroup
```

---

### Lab 2.3: Container Logging

#### Step 1: View container logs with docker logs
```bash
# Run container with output
docker run -d --name log-test quay.io/centos/centos:stream10 sh -c 'for i in $(seq 1 10); do echo "Log entry $i"; sleep 1; done'

# View logs
docker logs log-test
docker logs --tail 5 log-test
docker logs -f log-test  # Follow logs
```

#### Step 2: Configure logging drivers
```bash
# Run with json-file driver (default)
docker run -d --name log-json \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  quay.io/centos/centos:stream10 tail -f /dev/null

# Check logging driver
docker inspect --format='{{.HostConfig.LogConfig.Type}}' log-json
```

#### Step 3: Use syslog driver
```bash
# Run with syslog driver (requires syslog running)
docker run -d --name log-syslog \
  --log-driver=syslog \
  quay.io/centos/centos:stream10 tail -f /dev/null
```

#### Step 4: Centralize logs
```bash
# View log location on host
docker inspect --format='{{.LogPath}}' log-test

# View log file
sudo tail -f $(docker inspect --format='{{.LogPath}}' log-test)
```

---

## Workshop 3: Production Deployment

### Lab 3.1: Docker Networking

#### Step 1: List and inspect networks
```bash
docker network ls
docker network inspect bridge
docker network inspect host
docker network inspect none
```

#### Step 2: Create custom bridge network
```bash
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --ip-range 172.20.240.0/20 \
  my-custom-network

docker network inspect my-custom-network
```

#### Step 3: Run containers on custom network
```bash
docker run -d --name web1 --network my-custom-network quay.io/sclorg/httpd-24-c10s
docker run -d --name web2 --network my-custom-network quay.io/sclorg/httpd-24-c10s
```

#### Step 4: Test DNS resolution
```bash
# Test DNS between containers
docker exec web1 ping web2
docker exec web1 curl http://web2
```

#### Step 5: Configure network aliases
```bash
docker network disconnect my-custom-network web2
docker run -d --name web3 \
  --network my-custom-network \
  --network-alias myweb \
  quay.io/sclorg/httpd-24-c10s

# Test alias
docker exec web1 ping myweb
```

---

### Lab 3.2: Private Registry

#### Step 1: Run local registry container
```bash
docker run -d --name local-registry \
  -p 5000:5000 \
  --restart always \
  -v registry-data:/var/lib/registry \
  registry:2
```

#### Step 2: Configure registry storage
```bash
# Check registry is running
docker ps | grep registry
curl http://localhost:5000/v2/
```

#### Step 3: Tag and push images
```bash
# Tag image for local registry
docker tag python-multi localhost:5000/python-app:latest
docker tag python-multi localhost:5000/python-app:v1.0

# Push to registry
docker push localhost:5000/python-app:latest
docker push localhost:5000/python-app:v1.0
```

#### Step 4: Pull from registry
```bash
# Remove local images
docker rmi localhost:5000/python-app:latest localhost:5000/python-app:v1.0

# Pull from registry
docker pull localhost:5000/python-app:latest
docker pull localhost:5000/python-app:v1.0
```

#### Step 5: Enable authentication
```bash
# Create password file
mkdir -p auth
docker run --entrypoint htpasswd httpd:2 -Bbn admin password123 > auth/htpasswd

# Stop old registry
docker stop local-registry
docker rm local-registry

# Run registry with authentication
docker run -d --name local-registry \
  -p 5000:5000 \
  --restart always \
  -v registry-data:/var/lib/registry \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2

# Login to registry
docker login localhost:5000
# Enter admin / password123

# Push with authentication
docker push localhost:5000/python-app:latest
```

---

### Lab 3.3: Security Scanning

#### Step 1: Scan with Docker Scout
```bash
# Quick view
docker scout quickview python-multi

# View CVEs
docker scout cves python-multi
```

#### Step 2: Scan with Trivy
```bash
# Install Trivy (if not installed)
# curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh

# Scan image
trivy image python-multi
```

#### Step 3: Review vulnerabilities
```bash
# Compare scan results
docker scout recommendations python-multi
trivy image --severity HIGH,CRITICAL python-multi
```

#### Step 4: Fix high-severity issues
```bash
# Rebuild with updated base image
docker pull python:3.11-slim
docker build -t python-secure .
docker scout cves python-secure
```

#### Step 5: Create secure deployment
```bash
# Create docker-compose with security options
cat > docker-compose.secure.yaml << 'EOF'
services:
  secure-app:
    image: python-secure
    ports:
      - "5000:5000"
    read_only: true
    security_opt:
      - no-new-privileges:true
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.5'
    restart: unless-stopped
EOF

docker compose -f docker-compose.secure.yaml up -d
docker compose -f docker-compose.secure.yaml ps
```

---

## Reading Note: Docker Advanced Commands

### Advanced Docker Commands Cheat Sheet

| Command | Purpose |
|---------|---------|
| `docker buildx build` | Build with BuildKit builder |
| `docker buildx ls` | List available builders |
| `docker system df` | Show Docker disk usage |
| `docker system prune` | Remove unused data |
| `docker system prune -a` | Remove all unused data |
| `docker inspect` | Return container/image details |
| `docker stats` | Display resource usage |
| `docker top` | Display running processes |
| `docker diff` | Inspect changes in container |
| `docker logs` | Fetch container logs |
| `docker port` | List port mappings |
| `docker network ls` | List networks |
| `docker network inspect` | Inspect network details |
| `docker scout cves` | Scan for vulnerabilities |
| `docker scout quickview` | Quick vulnerability overview |

---

## Quiz

??? question "Question 1: Why is python:3.11-slim smaller than python:3.11?"
    **Answer:**
    
    The `-slim` variant includes only essential packages needed to run Python, removing build tools, documentation, and other unnecessary packages. This reduces the image size significantly.

??? question "Question 2: What is Copy-on-Write (CoW) in overlay filesystem?"
    **Answer:**
    
    Copy-on-Write is a technique where files are only copied when they are modified. When a container modifies a file from an image layer, the overlay filesystem copies the file to the container's upper layer before modifying it, preserving the original image layer.

??? question "Question 3: How do you check container memory limits?"
    **Answer:**
    
    Use `docker inspect --format='{{.HostConfig.Memory}}' container_name` or `docker stats` to see real-time memory usage.

??? question "Question 4: What is the difference between bridge and host networks?"
    **Answer:**
    
    Bridge network provides isolated network with NAT, while host network shares the host's network stack directly, giving maximum performance but less isolation.

??? question "Question 5: How do you enable logging in Docker containers?"
    **Answer:**
    
    Use `--log-driver` flag to specify logging driver and `--log-opt` for options. Default is json-file.

??? question "Question 6: What is Docker Scout used for?"
    **Answer:**
    
    Docker Scout scans Docker images for known vulnerabilities (CVEs) and provides security recommendations.

??? question "Question 7: How do you create a private Docker registry?"
    **Answer:**
    
    Run `docker run -d -p 5000:5000 registry:2` to start a local registry, then tag and push images to it.

??? question "Question 8: What does --no-new-privileges do?"
    **Answer:**
    
    It prevents processes inside the container from gaining additional privileges through setuid or setgid bits.

??? question "Question 9: How do you view container process tree?"
    **Answer:**
    
    Use `docker top container_name` to see running processes inside a container.

??? question "Question 10: What is the benefit of combining RUN commands?"
    **Answer:**
    
    Combining RUN commands reduces the number of layers in the image, making it smaller and reducing potential attack surface.

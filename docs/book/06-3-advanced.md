# Chapter 6: Docker Advanced (Practical)

## Overview
### Objectives
- Build production-ready Docker images
- Optimize container resource usage
- Deploy to private Docker registry
- Implement security best practices

---

## Assignment: Create Student's Submission along tutorial
- Student will submit Screen of Docker installation process according to step-by-step from manual.
- Student can use Basic Template [Download](../assets/dockerbasic-advanced-practical-submission.docx) or generate own version.

---

## Workshop 1: Build a Production App

### Scenario: Deploy a Production Node.js Application

### Lab 1.1: Project Setup

#### Step 1: Create project directory
```bash
mkdir ~/nodejs-production
cd ~/nodejs-production
```

#### Step 2: Create package.json and app.js
```bash
cat > package.json << 'EOF'
{
  "name": "production-app",
  "version": "1.0.0",
  "description": "Production-ready Node.js application",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

cat > app.js << 'EOF'
const express = require('express');
const os = require('os');

const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from production Node.js app!',
    hostname: os.hostname(),
    platform: os.platform(),
    nodeVersion: process.version,
    uptime: process.uptime()
  });
});

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy' });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF
```

#### Step 3: Create .dockerignore
```bash
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.dockerignore
Dockerfile*
docker-compose*
EOF
```

#### Step 4: Create single-stage Dockerfile
```bash
cat > Dockerfile.single << 'EOF'
FROM node:18

WORKDIR /app

COPY package.json .
RUN npm install

COPY app.js .

EXPOSE 3000

CMD ["npm", "start"]
EOF
```

---

### Lab 1.2: Optimize with Multi-stage Build

#### Step 1: Create multi-stage Dockerfile
```bash
cat > Dockerfile << 'EOF'
# Build stage
FROM node:18 AS builder

WORKDIR /app

COPY package.json .
RUN npm install --only=production

# Final stage
FROM node:18-slim

WORKDIR /app

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

COPY --from=builder /app/node_modules ./node_modules
COPY app.js .

# Set ownership
RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 3000

CMD ["node", "app.js"]
EOF
```

#### Step 2: Build and compare sizes
```bash
docker build -t nodejs-single -f Dockerfile.single .
docker build -t nodejs-multi .
docker images | grep nodejs
```

#### Step 3: Use BuildKit cache mounts
```bash
cat > Dockerfile.buildkit << 'EOF'
FROM node:18 AS builder

WORKDIR /app

COPY package.json .

# Use cache mount for npm
RUN --mount=type=cache,target=/root/.npm \
    npm install --only=production

FROM node:18-slim

WORKDIR /app

RUN groupadd -r appuser && useradd -r -g appuser appuser

COPY --from=builder /app/node_modules ./node_modules
COPY app.js .

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 3000

CMD ["node", "app.js"]
EOF

DOCKER_BUILDKIT=1 docker build -t nodejs-buildkit .
```

#### Step 4: Add health check
```bash
cat > Dockerfile.healthcheck << 'EOF'
FROM node:18-slim

WORKDIR /app

RUN groupadd -r appuser && useradd -r -g appuser appuser

COPY package.json .
RUN npm install --only=production

COPY app.js .

RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "app.js"]
EOF

docker build -t nodejs-healthcheck .
```

#### Step 5: Test application
```bash
docker run -d --name nodejs-app -p 3000:3000 nodejs-multi
curl http://localhost:3000
curl http://localhost:3000/health
docker stop nodejs-app
docker rm nodejs-app
```

---

### Lab 1.3: Production Hardening

#### Step 1: Run as non-root user
```bash
# Verify running as non-root
docker run --rm nodejs-multi whoami
docker run --rm nodejs-multi id
```

#### Step 2: Set resource limits
```bash
docker run -d --name nodejs-limited \
  -p 3000:3000 \
  --memory=256m \
  --cpus=0.5 \
  --pids-limit=100 \
  nodejs-multi

docker stats nodejs-limited --no-stream
```

#### Step 3: Add logging configuration
```bash
docker run -d --name nodejs-logged \
  -p 3000:3000 \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nodejs-multi
```

#### Step 4: Create docker-compose.yaml
```bash
cat > docker-compose.yaml << 'EOF'
services:
  nodejs-app:
    build: .
    ports:
      - "3000:3000"
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.5'
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
EOF
```

#### Step 5: Deploy and verify
```bash
docker compose up -d
docker compose ps
docker compose logs
curl http://localhost:3000
docker compose down
```

---

## Workshop 2: Manage Container Resources

### Scenario: Set Limits and Monitor Containers

### Lab 2.1: Resource Limit Configuration

#### Step 1: Create container with memory limits
```bash
docker run -d --name mem-demo \
  --memory=128m \
  --memory-swap=256m \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

docker stats mem-demo --no-stream
```

#### Step 2: Create container with CPU limits
```bash
# Limit to 25% of one CPU
docker run -d --name cpu-demo \
  --cpus=0.25 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

# Limit to specific CPUs
docker run -d --name cpu-set \
  --cpuset-cpus="0,1" \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

docker stats --no-stream
```

#### Step 3: Create container with PID limits
```bash
docker run -d --name pid-demo \
  --pids-limit=25 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

docker inspect --format='{{.HostConfig.PidsLimit}}' pid-demo
```

#### Step 4: Monitor resource usage
```bash
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.PIDs}}"
```

#### Step 5: Compare limited vs unlimited
```bash
# Run unlimited container
docker run -d --name unlimited \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

# Run limited container
docker run -d --name limited \
  --memory=64m \
  --cpus=0.1 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

docker stats --no-stream
```

---

### Lab 2.2: Container Inspection

#### Step 1: Use docker inspect for details
```bash
docker inspect limited

# Get specific values
docker inspect --format='{{.HostConfig.Memory}}' limited
docker inspect --format='{{.HostConfig.NanoCpus}}' limited
docker inspect --format='{{.HostConfig.PidsLimit}}' limited
```

#### Step 2: View container logs
```bash
docker logs limited
docker logs --tail 5 limited
docker logs -f limited  # Follow logs
docker logs --since 10m limited  # Logs from last 10 minutes
```

#### Step 3: Use docker stats
```bash
# Real-time stats
docker stats limited

# Single snapshot
docker stats --no-stream limited

# Custom format
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"
```

#### Step 4: Use docker top
```bash
docker top limited
docker top limited -o pid,ppid,comm
```

#### Step 5: Use docker diff to see changes
```bash
# Create some files in container
docker exec limited touch /tmp/test.txt
docker exec limited mkdir -p /data
docker exec limited echo "test" > /data/file.txt

# View changes
docker diff limited
```

---

### Lab 2.3: Overlay Filesystem Management

#### Step 1: Examine container filesystem
```bash
docker run -it --name fs-demo quay.io/centos/centos:stream10 /bin/bash

# Inside container
df -h
mount | grep overlay
ls -la /
```

#### Step 2: View layer information
```bash
# Inside container
cat /proc/mounts | grep overlay

# Exit and check on host
exit
docker inspect fs-demo | grep -i "upperdir\|merged"
```

#### Step 3: Understand Copy-on-Write
```bash
# Create container from image
docker run -d --name cow-demo quay.io/centos/centos:stream10 tail -f /dev/null

# Modify file in container
docker exec cow-demo sh -c 'echo "modified" > /etc/hostname'

# Check diff
docker diff cow-demo
```

#### Step 4: Manage container storage
```bash
# Check container size
docker system df -v

# Check specific container
docker inspect cow-demo | grep -i "size"
```

#### Step 5: Clean up unused layers
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Check disk usage
docker system df
```

---

## Workshop 3: Deploy to Private Registry

### Scenario: Set Up and Use Local Docker Registry

### Lab 3.1: Registry Setup

#### Step 1: Run local registry container
```bash
docker run -d --name local-registry \
  -p 5000:5000 \
  --restart always \
  registry:2
```

#### Step 2: Configure storage backend
```bash
# Check registry is running
docker ps | grep registry
curl http://localhost:5000/v2/

# Create custom config
mkdir -p /tmp/registry
cat > /tmp/registry/config.yml << 'EOF'
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
EOF

# Restart with custom config
docker stop local-registry
docker rm local-registry

docker run -d --name local-registry \
  -p 5000:5000 \
  --restart always \
  -v /tmp/registry/config.yml:/etc/docker/registry/config.yml \
  registry:2
```

#### Step 3: Enable authentication
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
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2
```

#### Step 4: Configure TLS
```bash
# Generate self-signed certificate
mkdir -p certs
openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout certs/domain.key \
  -x509 -days 365 \
  -out certs/domain.crt \
  -subj "/CN=localhost"

# Stop old registry
docker stop local-registry
docker rm local-registry

# Run registry with TLS
docker run -d --name local-registry \
  -p 5000:5000 \
  --restart always \
  -v $(pwd)/certs:/certs \
  -e "REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt" \
  -e "REGISTRY_HTTP_TLS_KEY=/certs/domain.key" \
  registry:2
```

#### Step 5: Test registry access
```bash
# Login to registry
docker login localhost:5000
# Enter admin / password123

# Test registry
curl -k https://localhost:5000/v2/
```

---

### Lab 3.2: Push and Pull Images

#### Step 1: Build application image
```bash
cd ~/nodejs-production
docker build -t nodejs-app .
```

#### Step 2: Tag for local registry
```bash
docker tag nodejs-app localhost:5000/nodejs-app:latest
docker tag nodejs-app localhost:5000/nodejs-app:v1.0
```

#### Step 3: Push to registry
```bash
docker push localhost:5000/nodejs-app:latest
docker push localhost:5000/nodejs-app:v1.0
```

#### Step 4: Pull from registry
```bash
# Remove local images
docker rmi localhost:5000/nodejs-app:latest localhost:5000/nodejs-app:v1.0

# Pull from registry
docker pull localhost:5000/nodejs-app:latest
docker pull localhost:5000/nodejs-app:v1.0
```

#### Step 5: Verify image integrity
```bash
# Check registry contents
curl -k https://localhost:5000/v2/_catalog
curl -k https://localhost:5000/v2/nodejs-app/tags/list

# Run from registry
docker run -d --name from-registry -p 3000:3000 localhost:5000/nodejs-app:latest
curl http://localhost:3000
```

---

### Lab 3.3: Security Scanning

#### Step 1: Scan image with Docker Scout
```bash
docker scout quickview nodejs-app
docker scout cves nodejs-app
```

#### Step 2: Scan with Trivy
```bash
# Install Trivy
# curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh

# Scan image
trivy image nodejs-app
```

#### Step 3: Review vulnerabilities
```bash
# View high and critical vulnerabilities
trivy image --severity HIGH,CRITICAL nodejs-app

# View all vulnerabilities
trivy image --severity MEDIUM,HIGH,CRITICAL nodejs-app
```

#### Step 4: Fix high-severity issues
```bash
# Update base image
docker pull node:18-slim

# Rebuild application
docker build -t nodejs-secure .

# Scan again
trivy image --severity HIGH,CRITICAL nodejs-secure
```

#### Step 5: Create secure deployment
```bash
cat > docker-compose.secure.yaml << 'EOF'
services:
  secure-app:
    image: localhost:5000/nodejs-app:latest
    ports:
      - "3000:3000"
    read_only: true
    security_opt:
      - no-new-privileges:true
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.5'
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
EOF

docker compose -f docker-compose.secure.yaml up -d
docker compose -f docker-compose.secure.yaml ps
docker compose -f docker-compose.secure.yaml logs
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
| `docker trust` | Content trust |
| `docker manifest` | Inspect image manifests |

---

## Quiz

??? question "Question 1: What is the purpose of a health check in Docker?"
    **Answer:**
    
    A health check allows Docker to automatically monitor container health and restart unhealthy containers. It's defined with `HEALTHCHECK` instruction in Dockerfile or `healthcheck` in docker-compose.

??? question "Question 2: How do you run a container as non-root?"
    **Answer:**
    
    Use the `USER` instruction in Dockerfile or `--user` flag in docker run. Example: `USER appuser` or `docker run --user 1000:1000 image`

??? question "Question 3: What is the benefit of read-only containers?"
    **Answer:**
    
    Read-only containers prevent write operations, improving security by reducing attack surface. Use `--read-only` flag.

??? question "Question 4: How do you configure logging in Docker?"
    **Answer:**
    
    Use `--log-driver` to specify driver and `--log-opt` for options. Example: `docker run --log-driver=json-file --log-opt max-size=10m image`

??? question "Question 5: What is the difference between --memory and --memory-swap?"
    **Answer:**
    
    `--memory` sets the memory limit, while `--memory-swap` sets the total memory + swap limit. Swap = --memory-swap - --memory.

??? question "Question 6: How do you scan Docker images for vulnerabilities?"
    **Answer:**
    
    Use Docker Scout (`docker scout cves image`) or Trivy (`trivy image image`) to scan for known vulnerabilities.

??? question "Question 7: What is a Docker private registry?"
    **Answer:**
    
    A private registry is a self-hosted Docker image storage service that allows you to store and distribute Docker images privately.

??? question "Question 8: How do you enable authentication for a registry?"
    **Answer:**
    
    Use htpasswd to create credentials, then configure `REGISTRY_AUTH` environment variables with the htpasswd file path.

??? question "Question 9: What is the purpose of .dockerignore?"
    **Answer:**
    
    `.dockerignore` excludes files from the build context, reducing build time and preventing sensitive files from being copied into the image.

??? question "Question 10: How do you clean up unused Docker resources?"
    **Answer:**
    
    Use `docker system prune` to remove unused data, `docker container prune` for stopped containers, `docker image prune -a` for unused images.

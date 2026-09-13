# Chapter 6: Docker Advanced

## Overview
### Objectives
- Understand multi-stage builds and image optimization
- Learn Docker BuildKit features
- Understand Docker overlay filesystem
- Master container resource management
- Configure Docker networking
- Set up and use Docker private registries

---

## Assignment: Create Student's Submission along tutorial
- Student will submit Screen of Docker installation process according to step-by-step from manual.
- Student can use Basic Template [Download](../assets/dockerbasic-advanced-submission.docx) or generate own version.

## Ref Reading
- [docker officail reference](https://docs.docker.com/build/building/multi-stage/)
---


![](../assets/images/Multi-Stage_Builds2.png)

### 1. Key Advantages of the Overlay Filesystem (OverlayFS / overlay2)
The Overlay Filesystem is a modern union filesystem that combines multiple underlying directories into a single, unified filesystem view. It plays a crucial role in making Docker builds fast, storage-efficient, and lightweight:
- **Union Directory Layering (lowerdir, upperdir, merged):**
    - **lowerdir:**  Read-only base layers (such as OS images or pre-installed SDKs)

    - **upperdir:** The read-write layer where new file changes from a build step are written

    - **merged:** The unified mount point where Docker presents the combined filesystem 

- **Copy-on-Write (CoW) Efficiency:** If a build instruction (RUN, COPY) modifies an existing file, OverlayFS copies the file from lowerdir to upperdir before editing. Unchanged files are never duplicated, drastically saving disk I/O and storage space

- **Instant Layer Mounts:** Rather than extracting heavy tarballs or copying full directory trees for each Dockerfile line, Docker simply mounts existing layer directories on host storage (/var/lib/docker/overlay2/)


## Workshop 1: Image Optimization

![](../assets/images/Multi-Stage_Builds1.png)

### Lab 1.1: Multi-stage Build - Go Application

#### Step 1: Create project directory
```bash
mkdir ~/go-multistage
cd ~/go-multistage
```

#### Step 2: Create main.go application
```bash
cat > main.go << 'EOF'
package main

import (
    "fmt"
    "net/http"
    "os"
    "runtime"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        hostname, _ := os.Hostname()
        fmt.Fprintf(w, "Hello from Go container!\n")
        fmt.Fprintf(w, "Hostname: %s\n", hostname)
        fmt.Fprintf(w, "Go Version: %s\n", runtime.Version())
        fmt.Fprintf(w, "OS/Arch: %s/%s\n", runtime.GOOS, runtime.GOARCH)
    })

    http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, "OK")
    })

    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}
EOF
```

#### Step 3: Create single-stage Dockerfile (show problem)
```bash
cat > Dockerfile.single << 'EOF'
FROM golang:1.21

WORKDIR /app

COPY main.go .

RUN go build -o server main.go

EXPOSE 8080

CMD ["./server"]
EOF
```

#### Step 4: Build and check image size
```bash
# Switch from lagacy build to Buildkit
export DOCKER_BUILDKIT=1
docker build -t go-single -f Dockerfile.single .
docker images go-single
```
> **Note:** Single-stage image is ~800MB because it includes Go compiler and all build tools

#### Step 5: Create multi-stage Dockerfile
```bash
cat > Dockerfile << 'EOF'
# Build stage
FROM golang:1.21 AS builder

WORKDIR /app

COPY main.go .

RUN CGO_ENABLED=0 GOOS=linux go build -o server main.go

# Final stage
FROM alpine:3.18

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=builder /app/server .

EXPOSE 8080

CMD ["./server"]
EOF
```

#### Step 6: Build and compare image sizes
```bash
docker build -t go-multi .
docker images | grep go-
```
> **Note:** Multi-stage image is ~15MB - over 50x smaller!

#### Step 7: Verify application works
```bash
docker run -d --name go-app -p 8080:8080 go-multi
curl http://localhost:8080
docker stop go-app
docker rm go-app
```

---

### Lab 1.2: BuildKit Features

#### Step 1: Enable BuildKit
```bash
export DOCKER_BUILDKIT=1
docker build --version
```

#### Step 2: Create Dockerfile with cache mounts
```bash
cat > Dockerfile.buildkit << 'EOF'
FROM golang:1.21 AS builder

WORKDIR /app

# Use cache mount for Go module cache
COPY main.go .

RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    CGO_ENABLED=0 GOOS=linux go build -o server main.go

FROM alpine:3.18

COPY --from=builder /app/server /server

EXPOSE 8080

CMD ["/server"]
EOF
```

#### Step 3: Build with BuildKit caching
```bash
DOCKER_BUILDKIT=1 docker build -t go-buildkit -f Dockerfile.buildkit .
time docker build -t go-buildkit-cache -f Dockerfile.buildkit .
```
> **Note:** Second build is faster due to BuildKit cache mounts

#### Step 4: Use build secrets with --mount=type=secret
```bash
echo "my-api-key-12345" > api-key.txt

cat > Dockerfile.secret << 'EOF'
FROM alpine:3.18

# Copy secret during build (not in final image)
RUN --mount=type=secret,id=apikey \
    cat /run/secrets/apikey > /tmp/key.txt

# Verify secret is not in final layer
RUN rm /tmp/key.txt

CMD ["echo", "Secret build completed"]
EOF

DOCKER_BUILDKIT=1 docker build --secret id=apikey,src=api-key.txt -t go-secret .
```

#### Step 5: Verify secret not in image
```bash
docker history go-secret
```
> **Note:** Secret is not visible in image history

---

### Lab 1.3: Image Analysis

#### Step 1: Use docker history to inspect layers
```bash
docker history go-multi
docker history go-single
```

#### Step 2: Analyze image with dive tool (if installed)
```bash
# Install dive (optional)
# brew install dive  (macOS)
# sudo apt-get install dive  (Ubuntu)

# Analyze image
dive go-multi
```

#### Step 3: Optimize .dockerignore
```bash
cat > .dockerignore << 'EOF'
.git
.gitignore
*.md
*.txt
api-key.txt
Dockerfile*
docker-compose*
.vscode
.idea
node_modules
__pycache__
*.pyc
EOF
```

#### Step 4: Rebuild with optimized .dockerignore
```bash
DOCKER_BUILDKIT=1 docker build -t go-optimized .
docker images go-optimized
```

---

## Workshop 2: Container Internals

### Lab 2.1: Understanding Overlay Filesystem

#### Step 1: Run container and examine filesystem
```bash
docker run -it --name overlay-test quay.io/centos/centos:stream10 /bin/bash
```

#### Step 2: View overlay mount points (inside container)
```bash
mount | grep overlay
cat /proc/mounts | grep overlay
```

#### Step 3: Create files in container
```bash
mkdir -p /test
echo "container file" > /test/file.txt
cat /test/file.txt
```

#### Step 4: Exit and examine layers (on host)
```bash
exit

# Find container layer (on host)
docker inspect overlay-test | grep -i "upperdir\|merged"
```

#### Step 5: View layer contents
```bash
# Find the overlay directory
sudo ls -la /var/lib/docker/overlay2/
```

#### Step 6: Compare container vs image layers
```bash
docker diff overlay-test
docker commit overlay-test overlay-test-snapshot
docker history overlay-test-snapshot
```

---

### Lab 2.2: Resource Limits

#### Step 1: Run container with memory limit
```bash
docker run -d --name memory-limit \
  --memory=256m \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

docker stats memory-limit --no-stream
```

#### Step 2: Run container with CPU limit
```bash
docker run -d --name cpu-limit \
  --cpus=0.5 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

docker stats cpu-limit --no-stream
```

#### Step 3: Run container with PID limit
```bash
docker run -d --name pid-limit \
  --pids-limit=50 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

# Test PID limit (optional - may fail)
docker exec pid-limit sh -c 'for i in $(seq 1 100); do sleep 100 & done'
```

#### Step 4: Monitor resource usage
```bash
docker stats --no-stream
```

#### Step 5: Combine resource limits
```bash
docker run -d --name combined-limit \
  --memory=128m \
  --cpus=0.25 \
  --pids-limit=25 \
  quay.io/centos/centos:stream10 \
  tail -f /dev/null

docker stats combined-limit --no-stream
```

---

### Lab 2.3: Container Inspection

#### Step 1: Use docker inspect
```bash
docker inspect combined-limit
docker inspect --format='{{.HostConfig.Memory}}' combined-limit
docker inspect --format='{{.HostConfig.NanoCpus}}' combined-limit
```

#### Step 2: View container logs
```bash
docker logs combined-limit
docker logs -f combined-limit  # Follow logs
docker logs --tail 10 combined-limit  # Last 10 lines
```

#### Step 3: Use docker stats
```bash
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

#### Step 4: Use docker top
```bash
docker top combined-limit
docker top combined-limit -o pid,comm
```

#### Step 5: Use docker diff
```bash
docker exec combined-limit touch /tmp/test-file
docker diff combined-limit
```

---

## Workshop 3: Advanced Networking

### Lab 3.1: Docker Network Types

#### Step 1: List default networks
```bash
docker network ls
docker network inspect bridge
```

#### Step 2: Create bridge network
```bash
docker network create --driver bridge my-bridge
docker network ls | grep my-bridge
```

#### Step 3: Create host network
```bash
docker network create --driver host my-host
docker network ls | grep my-host
```

#### Step 4: Connect containers to custom network
```bash
# Run containers on custom network
docker run -d --name net-test-1 --network my-bridge quay.io/centos/centos:stream10 tail -f /dev/null
docker run -d --name net-test-2 --network my-bridge quay.io/centos/centos:stream10 tail -f /dev/null
```

#### Step 5: Test inter-container communication
```bash
# Test DNS resolution
docker exec net-test-1 ping net-test-2

# Test connectivity
docker exec net-test-1 curl net-test-2
```

---

### Lab 3.2: Port Mapping and Exposing

#### Step 1: Run container with -p flag
```bash
docker run -d --name port-test -p 8080:80 quay.io/sclorg/httpd-24-c10s
curl http://localhost:8080
```

#### Step 2: Run container with -P flag
```bash
docker run -d --name port-random -P quay.io/sclorg/httpd-24-c10s
docker port port-random
```

#### Step 3: Use docker port command
```bash
docker port port-test
docker port port-test 80
```

#### Step 4: Configure port forwarding
```bash
# Bind to specific interface
docker run -d --name port-bind -p 127.0.0.1:9090:80 quay.io/sclorg/httpd-24-c10s
curl http://127.0.0.1:9090
```

---

### Lab 3.3: Private Registry Setup

#### Step 1: Run local registry container
```bash
docker run -d --name registry -p 5000:5000 --restart always registry:2
```

#### Step 2: Tag image for local registry
```bash
docker tag go-multi localhost:5000/go-app:latest
```

#### Step 3: Push to local registry
```bash
docker push localhost:5000/go-app:latest
```

#### Step 4: Pull from local registry
```bash
docker rmi localhost:5000/go-app:latest
docker pull localhost:5000/go-app:latest
```

#### Step 5: Verify registry contents
```bash
curl http://localhost:5000/v2/_catalog
curl http://localhost:5000/v2/go-app/tags/list
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

---

## Quiz

??? question "Question 1: What is the benefit of multi-stage builds?"
    **Answer:**
    
    Multi-stage builds reduce final image size by separating build dependencies from runtime dependencies. The build stage contains compilers and tools, while the final stage contains only the application and minimal runtime.

??? question "Question 2: How do you enable BuildKit?"
    **Answer:**
    
    Set the environment variable `DOCKER_BUILDKIT=1` or configure Docker daemon with BuildKit enabled.

??? question "Question 3: What is the overlay filesystem in Docker?"
    **Answer:**
    
    Overlay filesystem (overlay2) is a union filesystem that combines multiple directories (layers) into a single unified view. Docker uses it to efficiently store container layers with Copy-on-Write (CoW).

??? question "Question 4: How do you limit container memory?"
    **Answer:**
    
    Use the `--memory` flag: `docker run --memory=256m image_name`

??? question "Question 5: What is the difference between -p and -P?"
    **Answer:**
    
    `-p` maps specific host port to container port (e.g., `-p 8080:80`), while `-P` publishes all exposed ports to random host ports.

??? question "Question 6: How do you create a custom Docker network?"
    **Answer:**
    
    Use `docker network create --driver bridge network_name`

??? question "Question 7: What is the purpose of .dockerignore?"
    **Answer:**
    
    `.dockerignore` excludes files and directories from the build context, reducing build time and preventing sensitive files from being copied into the image.

??? question "Question 8: How do you run a container with CPU limits?"
    **Answer:**
    
    Use the `--cpus` flag: `docker run --cpus=0.5 image_name` (limits to 50% of one CPU core)

??? question "Question 9: What is a Docker private registry?"
    **Answer:**
    
    A private registry is a self-hosted Docker image storage service that allows you to store and distribute Docker images privately within your organization.

??? question "Question 10: How do you inspect container resource limits?"
    **Answer:**
    
    Use `docker inspect --format='{{.HostConfig.Memory}}' container_name` or `docker stats` for real-time monitoring.

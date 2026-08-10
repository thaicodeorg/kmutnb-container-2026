!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/04-dockerfile.html)

# Chapter 4: Dockerfile

## Overview

This chapter covers Dockerfile instructions, multi-stage builds, and best practices for creating optimized Docker images.

---

## 4.1 Dockerfile Instructions

### FROM

Sets the base image:

```dockerfile
FROM centos:stream9
FROM python:3.11-slim
FROM scratch
```

### WORKDIR

Sets the working directory inside the container:

```dockerfile
WORKDIR /app
WORKDIR /usr/src/app
```

### COPY

Copies files from build context to the image:

```dockerfile
COPY . .
COPY requirements.txt .
COPY src/ /app/src/
COPY --chown=user:user file.txt /app/
```

### RUN

Executes commands during the build:

```dockerfile
RUN dnf install -y nginx
RUN pip install -r requirements.txt
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*
```

### CMD

Default command when the container starts (can be overridden):

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
CMD ["python", "app.py"]
CMD echo "Hello"
```

### ENTRYPOINT

Defines the main executable (not easily overridden):

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### ENV

Sets environment variables:

```dockerfile
ENV APP_HOME=/app
ENV PYTHONUNBUFFERED=1
ENV NODE_ENV=production
```

### EXPOSE

Documents which ports the container listens on:

```dockerfile
EXPOSE 80
EXPOSE 443
EXPOSE 8080/tcp
```

### ARG

Defines build-time variables:

```dockerfile
ARG PYTHON_VERSION=3.11
FROM python:${PYTHON_VERSION}-slim
ARG BUILD_DATE
LABEL build-date="${BUILD_DATE}"
```

---

## 4.2 Multi-Stage Builds

Multi-stage builds reduce the final image size by separating build and runtime stages:

### Python Example

```dockerfile
# Stage 1: Build
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

### Node.js Example

```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Go Example

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o server .

# Stage 2: Runtime
FROM scratch
COPY --from=builder /app/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
```

---

## 4.3 .dockerignore

Create a `.dockerignore` file to exclude files from the build context:

```
.git
.gitignore
.env
node_modules
__pycache__
*.pyc
.venv
venv
dist
build
*.md
LICENSE
Dockerfile
.dockerignore
docker-compose.yml
.vscode
.idea
```

---

## 4.4 Best Practices

### Layer Caching

Order instructions from least to most frequently changing:

```dockerfile
# Good: Dependencies first, code last
FROM node:18-slim
WORKDIR /app
COPY package*.json ./          # Rarely changes
RUN npm ci                     # Cached until package*.json changes
COPY . .                       # Changes often
```

### Use Minimal Base Images

```dockerfile
# Prefer slim or alpine variants
FROM python:3.11-slim
FROM node:18-alpine
FROM golang:1.21-alpine
```

### Run as Non-Root User

```dockerfile
FROM centos:stream9
RUN useradd -r -s /bin/false appuser
USER appuser
WORKDIR /app
COPY --chown=appuser:appuser . .
CMD ["./start.sh"]
```

### Combine RUN Commands

```dockerfile
# Bad: Multiple layers
RUN dnf install -y curl
RUN dnf install -y git
RUN dnf clean all

# Good: Single layer
RUN dnf install -y curl git && \
    dnf clean all && \
    rm -rf /var/cache/dnf
```

### Use Specific Tags

```dockerfile
# Bad: Mutable tag
FROM python:latest

# Good: Specific version
FROM python:3.11.5-slim

# Best: Digest pinning
FROM python:3.11.5-slim@sha256:abc123...
```

---

## Summary

| Instruction | Purpose | Overridable |
|-------------|---------|-------------|
| `FROM` | Base image | — |
| `WORKDIR` | Set working directory | — |
| `COPY` | Copy files to image | — |
| `RUN` | Execute build commands | — |
| `CMD` | Default start command | Yes |
| `ENTRYPOINT` | Main executable | With `--entrypoint` |
| `ENV` | Environment variables | Yes |
| `EXPOSE` | Document port | — |
| `ARG` | Build-time variable | — |

---

## Quiz

??? question "Question 1: What is the difference between CMD and ENTRYPOINT?"
    **Answer:**
    
    `CMD` provides default arguments that can be overridden at runtime; `ENTRYPOINT` defines the main executable and is harder to override.

??? question "Question 2: What does a multi-stage build accomplish?"
    **Answer:**
    
    It separates build-time dependencies from the runtime image, resulting in a smaller final image.

??? question "Question 3: What goes in a .dockerignore file?"
    **Answer:**
    
    Files and directories that should not be sent to the Docker build context, like `.git`, `node_modules`, and `.env`

??? question "Question 4: How do you set an environment variable in a Dockerfile?"
    **Answer:**
    
    Using the `ENV` instruction, e.g., `ENV APP_ENV=production`

??? question "Question 5: What is the correct way to combine RUN commands?"
    **Answer:**
    
    Chain them with `&&` and clean up in the same layer: `RUN dnf install -y pkg && dnf clean all`

??? question "Question 6: How do you copy files from a previous build stage?"
    **Answer:**
    
    Using `COPY --from=builder <source> <destination>`

??? question "Question 7: Why should you order COPY instructions from least to most frequently changing?"
    **Answer:**
    
    To take advantage of Docker's layer caching mechanism — unchanged layers are cached

??? question "Question 8: How do you run a container as a non-root user in a Dockerfile?"
    **Answer:**
    
    Create a user with `RUN useradd` and then use the `USER` instruction

??? question "Question 9: What is the purpose of the ARG instruction?"
    **Answer:**
    
    To define build-time variables that can be passed during `docker build` with `--build-arg`

??? question "Question 10: Why are slim/alpine base images preferred?"
    **Answer:**
    
    They produce smaller final images with fewer vulnerabilities and faster pull times

!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/08-workshop-webserver.html)

# Chapter 8: Workshop - Web Server with Nginx

## Overview

This workshop covers building a custom Nginx Docker image on CentOS 9 Stream with SSL support.

---

## Lab 1: Project Setup

### Step 1: Create Project Directory

```bash
mkdir -p ~/nginx-centos && cd ~/nginx-centos
```

### Step 2: Create Directory Structure

```bash
mkdir -p html certs conf
```

---

## Lab 2: Create Custom Nginx Dockerfile

### Step 1: Create Dockerfile

```dockerfile
# Dockerfile
FROM quay.io/centos/centos:stream9

ARG NGINX_VERSION=1.24.0

# Install dependencies
RUN dnf install -y \
    nginx \
    openssl \
    openssl-devel \
    && dnf clean all \
    && rm -rf /var/cache/dnf

# Create nginx user
RUN useradd -r -s /bin/false nginx

# Create directories
RUN mkdir -p /var/cache/nginx && \
    mkdir -p /var/log/nginx && \
    mkdir -p /etc/nginx/conf.d

# Copy configuration
COPY conf/nginx.conf /etc/nginx/nginx.conf
COPY html/index.html /usr/share/nginx/html/index.html

# Set permissions
RUN chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

### Step 2: Create Custom nginx.conf

```nginx
# conf/nginx.conf
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 4096;
    client_max_body_size 16M;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    server {
        listen 80;
        server_name localhost;

        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }

        location /api {
            proxy_pass http://backend:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

### Step 3: Create index.html

```html
<!-- html/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Nginx on CentOS 9 Stream</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }
        .container {
            text-align: center;
            padding: 2rem;
            background: rgba(0,0,0,0.2);
            border-radius: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Nginx on CentOS 9 Stream</h1>
        <p>Docker Container Workshop</p>
        <p>Server is running successfully!</p>
    </div>
</body>
</html>
```

---

## Lab 3: Build and Run

### Step 1: Build the Image

```bash
docker build -t mynginx:centos9 .
```

### Step 2: Run the Container

```bash
docker run -d \
    --name nginx-server \
    -p 8080:80 \
    mynginx:centos9
```

### Step 3: Test Access

```bash
curl http://localhost:8080
```

### Step 4: View Logs

```bash
docker logs nginx-server
docker logs -f nginx-server
```

### Step 5: Inspect Container

```bash
docker inspect nginx-server
```

---

## Lab 4: SSL Certificate Setup

### Step 1: Generate Self-Signed Certificate

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout certs/server.key \
    -out certs/server.crt \
    -subj "/C=TH/ST=Bangkok/L=Bangkok/O=DevOps/CN=localhost"
```

### Step 2: Update nginx.conf for SSL

Add to the server block:

```nginx
server {
    listen 80;
    server_name localhost;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Step 3: Rebuild and Run with SSL

```bash
docker stop nginx-server
docker rm nginx-server

docker build -t mynginx:ssl .

docker run -d \
    --name nginx-ssl \
    -p 443:443 \
    -p 80:80 \
    -v $(pwd)/certs:/etc/nginx/certs:ro \
    mynginx:ssl
```

### Step 4: Test SSL Access

```bash
curl -k https://localhost
curl -I http://localhost
```

---

## Lab 5: Multi-Stage Build for Frontend

### Step 1: Create Multi-Stage Dockerfile

```dockerfile
# Dockerfile.multistage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build

FROM quay.io/centos/centos:stream9
RUN dnf install -y nginx && dnf clean all
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Step 2: Build and Run

```bash
docker build -t myapp:latest -f Dockerfile.multistage .
docker run -d -p 3000:80 myapp:latest
```

---

## Lab 6: Clean Up

```bash
# Stop all containers
docker stop nginx-server nginx-ssl myapp

# Remove all containers
docker rm nginx-server nginx-ssl myapp

# Remove images
docker rmi mynginx:centos9 mynginx:ssl myapp:latest

# Verify cleanup
docker ps -a
docker images
```

---

## Summary

| Step | Command |
|------|---------|
| Build image | `docker build -t mynginx:centos9 .` |
| Run with port | `docker run -d -p 8080:80 mynginx:centos9` |
| Generate SSL | `openssl req -x509 ...` |
| Test HTTP | `curl http://localhost:8080` |
| Test HTTPS | `curl -k https://localhost` |
| View logs | `docker logs -f nginx-server` |

---

## Quiz

??? question "Question 1: What is the base image used in this workshop?"
    **Answer:**
    
    `quay.io/centos/centos:stream9` — the official CentOS 9 Stream image

??? question "Question 2: How do you install Nginx in a CentOS 9 Stream container?"
    **Answer:**
    
    `dnf install -y nginx` inside a RUN instruction in the Dockerfile

??? question "Question 3: What does `daemon off;` do in the CMD instruction?"
    **Answer:**
    
    Keeps Nginx running in the foreground so Docker can track the process

??? question "Question 4: How do you generate a self-signed SSL certificate?"
    **Answer:**
    
    `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt`

??? question "Question 5: What volume mount makes SSL certificates available inside the container?"
    **Answer:**
    
    `-v $(pwd)/certs:/etc/nginx/certs:ro` — mounts the certs directory as read-only

??? question "Question 6: What is the purpose of a multi-stage Docker build?"
    **Answer:**
    
    To separate build dependencies from the final runtime image, reducing image size

??? question "Question 7: How do you redirect HTTP to HTTPS in Nginx?"
    **Answer:**
    
    Using `return 301 https://$host$request_uri;` in the HTTP server block

??? question "Question 8: What does the `-k` flag do with curl?"
    **Answer:**
    
    Disables SSL certificate verification, allowing testing with self-signed certificates

??? question "Question 9: How do you view real-time logs from a container?"
    **Answer:**
    
    `docker logs -f <container_name>` — the `-f` flag follows the log output

??? question "Question 10: What user should Nginx run as inside the container?"
    **Answer:**
    
    A non-root user like `nginx` — created with `useradd -r -s /bin/false nginx`

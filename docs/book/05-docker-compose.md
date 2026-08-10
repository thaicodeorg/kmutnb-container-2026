!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/05-docker-compose.html)

# Chapter 5: Docker Compose

## Overview

This chapter covers Docker Compose for defining and running multi-container applications.

---

## 5.1 YAML Syntax Basics

Docker Compose files use YAML format. Key rules:

```yaml
# Indentation matters (2 spaces)
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

# Lists use dashes
environment:
  - APP_ENV=production
  - DEBUG=false

# Strings with special characters need quotes
command: "echo 'hello world'"
```

---

## 5.2 Services Definition

### Basic Service

```yaml
services:
  web:
    image: nginx:latest
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
    environment:
      - NGINX_HOST=localhost
    volumes:
      - ./html:/usr/share/nginx/html
```

### Build from Dockerfile

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        PYTHON_VERSION: "3.11"
    image: myapp:1.0
```

### Service Dependencies

```yaml
services:
  backend:
    depends_on:
      - database
      - redis

  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
```

### Service with Health Check

```yaml
services:
  web:
    image: nginx:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

---

## 5.3 Networks Configuration

### Default Network

```yaml
services:
  web:
    image: nginx:latest
  backend:
    image: python:3.11-slim
# Automatically creates a default network
```

### Custom Networks

```yaml
services:
  web:
    image: nginx:latest
    networks:
      - frontend

  backend:
    image: python:3.11-slim
    networks:
      - frontend
      - backend

  database:
    image: postgres:15
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

---

## 5.4 Volumes Configuration

### Named Volumes

```yaml
services:
  database:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
    driver: local
```

### Bind Mounts

```yaml
services:
  web:
    image: nginx:latest
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./config:/etc/nginx/conf.d
```

### Volume with Options

```yaml
volumes:
  pgdata:
    driver: local
    driver_opts:
      type: none
      device: /data/postgres
      o: bind
```

---

## 5.5 Environment Variables

### Using Environment Key

```yaml
services:
  app:
    image: myapp:1.0
    environment:
      - APP_ENV=production
      - DATABASE_URL=postgresql://db:5432/mydb
      - REDIS_URL=redis://redis:6379
```

### Using Environment File

```yaml
services:
  app:
    image: myapp:1.0
    env_file:
      - .env
      - .env.production
```

### .env File Example

```bash
# .env
APP_ENV=development
POSTGRES_USER=admin
POSTGRES_PASSWORD=secret
POSTGRES_DB=myapp
```

---

## 5.6 Common Commands

### Start Services

```bash
# Start in foreground
docker compose up

# Start in background
docker compose up -d

# Build before starting
docker compose up --build

# Force recreate containers
docker compose up -d --force-recreate
```

### Stop Services

```bash
# Stop services
docker compose stop

# Stop and remove containers
docker compose down

# Stop and remove everything (including volumes)
docker compose down -v
```

### View Status

```bash
# List running services
docker compose ps

# List all services
docker compose ps -a

# View logs
docker compose logs
docker compose logs -f web
docker compose logs --tail=100
```

### Execute Commands

```bash
# Run one-off command
docker compose run --rm web echo "hello"

# Execute in running service
docker compose exec web bash
docker compose exec database psql -U admin
```

---

## 5.7 Complete Example

```yaml
# docker-compose.yml
version: "3.8"

services:
  web:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - api
    networks:
      - frontend

  api:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - DATABASE_URL=postgresql://admin:secret@db:5432/myapp
    networks:
      - frontend
      - backend

  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    networks:
      - backend

  redis:
    image: redis:7-alpine
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  pgdata:
```

---

## Summary

| Feature | Syntax |
|---------|--------|
| Service | `services: web: image: nginx` |
| Port | `ports: "8080:80"` |
| Volume | `volumes: ./src:/app` |
| Network | `networks: frontend:` |
| Environment | `environment: - KEY=value` |
| Start | `docker compose up -d` |
| Stop | `docker compose down` |

---

## Quiz

??? question "Question 1: What is the default filename for a Docker Compose file?"
    **Answer:**
    
    `docker-compose.yml` (or `compose.yml` in newer versions)

??? question "Question 2: How do you start services in detached mode?"
    **Answer:**
    
    `docker compose up -d`

??? question "Question 3: What does the `depends_on` key do?"
    **Answer:**
    
    Defines startup order — dependent services wait for their dependencies to start first

??? question "Question 4: How do you specify an environment file for a service?"
    **Answer:**
    
    Using `env_file:` directive, e.g., `env_file: .env`

??? question "Question 5: What command creates and starts containers with a fresh build?"
    **Answer:**
    
    `docker compose up --build`

??? question "Question 6: How do you view logs for a specific service?"
    **Answer:**
    
    `docker compose logs <service_name>`, with `-f` to follow

??? question "Question 7: What does `docker compose down -v` do?"
    **Answer:**
    
    Stops containers, removes them, and also removes named volumes

??? question "Question 8: How do you run a one-off command in a service?"
    **Answer:**
    
    `docker compose run --rm <service> <command>`

??? question "Question 9: What does `internal: true` do on a network?"
    **Answer:**
    
    Prevents external access to the network — containers can only communicate with each other

??? question "Question 10: How do you access the database service interactively?"
    **Answer:**
    
    `docker compose exec database psql -U admin` (or bash for interactive shell)

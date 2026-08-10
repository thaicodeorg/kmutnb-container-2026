# Docker Container Technology Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a complete MkDocs educational resource with 9 book chapters, 9 reveal.js presentations, and 10 quizzes per chapter for teaching Docker Container Technology to CS students.

**Architecture:** Flat file structure with 1:1 mapping between book chapters and presentations. Each book chapter includes a callout linking to its presentation, main content, and 10 quiz questions. Presentations are standalone reveal.js HTML files.

**Tech Stack:** MkDocs, Material for MkDocs theme, reveal.js, Markdown, HTML, TypeScript (workshop example), Python FastAPI (workshop example), PostgreSQL (workshop example)

## Global Constraints

- Base OS for examples: CentOS 9 Stream
- Language: English
- Theme: Material for MkDocs
- Presentations: reveal.js (standalone HTML)
- Quiz format: Collapsible `<details>` blocks
- Links: `[[wiki-link]]` format in index.md

---

## File Structure

```
docs/
├── index.md                         # Table of contents with wiki-links
├── book/
│   ├── 01-fundamentals.md
│   ├── 02-installation.md
│   ├── 03-basic-operations.md
│   ├── 04-dockerfile.md
│   ├── 05-docker-compose.md
│   ├── 06-advanced.md
│   ├── 07-workshop-basic.md
│   ├── 08-workshop-webserver.md
│   └── 09-workshop-fullstack.md
├── presents/
│   ├── 01-fundamentals.html
│   ├── 02-installation.html
│   ├── 03-basic-operations.html
│   ├── 04-dockerfile.html
│   ├── 05-docker-compose.html
│   ├── 06-advanced.html
│   ├── 07-workshop-basic.html
│   ├── 08-workshop-webserver.html
│   └── 09-workshop-fullstack.html
└── assets/
    └── logo.png
```

---

### Task 1: Create index.md with wiki-links

**Files:**
- Modify: `docs/index.md`

**Interfaces:**
- Consumes: None
- Produces: Table of contents linking to all book chapters and presentations

- [ ] **Step 1: Write index.md content**

```markdown
## Docker Technology

### Book

- [[book/01-fundamentals|1. Fundamentals]]
- [[book/02-installation|2. Installation]]
- [[book/03-basic-operations|3. Basic Operations]]
- [[book/04-dockerfile|4. Dockerfile]]
- [[book/05-docker-compose|5. Docker Compose]]
- [[book/06-advanced|6. Advanced]]
- [[book/07-workshop-basic|7. Workshop: Basic]]
- [[book/08-workshop-webserver|8. Workshop: Webserver]]
- [[book/09-workshop-fullstack|9. Workshop: Fullstack]]

### Presentations

- [[presents/01-fundamentals|1. Fundamentals Slides]]
- [[presents/02-installation|2. Installation Slides]]
- [[presents/03-basic-operations|3. Basic Operations Slides]]
- [[presents/04-dockerfile|4. Dockerfile Slides]]
- [[presents/05-docker-compose|5. Docker Compose Slides]]
- [[presents/06-advanced|6. Advanced Slides]]
- [[presents/07-workshop-basic|7. Workshop: Basic Slides]]
- [[presents/08-workshop-webserver|8. Workshop: Webserver Slides]]
- [[presents/09-workshop-fullstack|9. Workshop: Fullstack Slides]]
```

- [ ] **Step 2: Verify links work**

Run: `mkdocs serve` and click each link in browser

- [ ] **Step 3: Commit**

```bash
git add docs/index.md
git commit -m "feat: add index.md with wiki-links to all chapters"
```

---

### Task 2: Create book chapter template

**Files:**
- Create: `docs/book/01-fundamentals.md` (first chapter as template)

**Interfaces:**
- Consumes: None
- Produces: Chapter template with callout, content structure, quiz format

- [ ] **Step 1: Create chapter with callout and quiz structure**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/01-fundamentals.html)

# Fundamentals

## What are Containers?

[Content here]

## VMs vs Containers

[Content here]

## Docker Architecture

[Content here]

---

## Quiz

??? question "Question 1: What is a container?"
    **Answer:**
    
    A lightweight, standalone, executable package that includes everything needed to run a piece of software.

??? question "Question 2: What is the difference between a VM and a container?"
    **Answer:**
    
    VMs virtualize hardware, containers virtualize the OS kernel.

[Continue for 10 questions]
```

- [ ] **Step 2: Verify rendering**

Run: `mkdocs serve` and check callout, collapsible quizzes render correctly

- [ ] **Step 3: Commit**

```bash
git add docs/book/01-fundamentals.md
git commit -m "feat: add chapter template with callout and quiz structure"
```

---

### Task 3: Create reveal.js presentation template

**Files:**
- Create: `docs/presents/01-fundamentals.html`

**Interfaces:**
- Consumes: None
- Produces: reveal.js HTML template for all presentations

- [ ] **Step 1: Create reveal.js HTML template**

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Docker Fundamentals</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@4.6.0/dist/reveal.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@4.6.0/dist/theme/white.css">
</head>
<body>
    <div class="reveal">
        <div class="slides">
            <section>
                <h1>Docker Fundamentals</h1>
                <p>Container Technology for Computer Science</p>
            </section>
            <section>
                <h2>What are Containers?</h2>
                <ul>
                    <li>Lightweight</li>
                    <li>Standalone</li>
                    <li>Executable</li>
                </ul>
            </section>
            <section>
                <h2>VMs vs Containers</h2>
                <p>[Content]</p>
            </section>
            <section>
                <h2>Docker Architecture</h2>
                <p>[Content]</p>
            </section>
            <section>
                <h1>Thank You</h1>
                <p><a href="../book/01-fundamentals.html">Back to Book</a></p>
            </section>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/reveal.js@4.6.0/dist/reveal.js"></script>
    <script>
        Reveal.initialize({
            hash: true,
            slideNumber: true
        });
    </script>
</body>
</html>
```

- [ ] **Step 2: Verify presentation opens**

Run: `mkdocs serve` and open `http://127.0.0.1:8000/presents/01-fundamentals.html`

- [ ] **Step 3: Commit**

```bash
git add docs/presents/01-fundamentals.html
git commit -m "feat: add reveal.js presentation template"
```

---

### Task 4: Create all book chapters (02-09)

**Files:**
- Create: `docs/book/02-installation.md`
- Create: `docs/book/03-basic-operations.md`
- Create: `docs/book/04-dockerfile.md`
- Create: `docs/book/05-docker-compose.md`
- Create: `docs/book/06-advanced.md`
- Create: `docs/book/07-workshop-basic.md`
- Create: `docs/book/08-workshop-webserver.md`
- Create: `docs/book/09-workshop-fullstack.md`

**Interfaces:**
- Consumes: Template from Task 2
- Produces: All 9 book chapters with content and quizzes

- [ ] **Step 1: Create 02-installation.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/02-installation.html)

# Installation

## Prerequisites

- CentOS 9 Stream
- Root access or sudo privileges

## Install Docker CE

```bash
# Remove old versions
sudo dnf remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine

# Install yum-utils
sudo dnf install -y yum-utils

# Add Docker repository
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
```

## Rootless Mode

[Content here]

---

## Quiz

[10 questions about installation]
```

- [ ] **Step 2: Create 03-basic-operations.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/03-basic-operations.html)

# Basic Operations

## Images

```bash
# List images
docker images

# Pull image
docker pull centos:stream9

# Search images
docker search nginx
```

## Containers

```bash
# Run container
docker run -d --name my-nginx -p 8080:80 nginx

# List containers
docker ps -a

# Stop container
docker stop my-nginx

# Remove container
docker rm my-nginx
```

## Networking

[Content here]

## Volumes

[Content here]

---

## Quiz

[10 questions about basic operations]
```

- [ ] **Step 3: Create 04-dockerfile.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/04-dockerfile.html)

# Dockerfile

## Basic Instructions

```dockerfile
FROM centos:stream9
WORKDIR /app
COPY . .
RUN dnf install -y python3
CMD ["python3", "app.py"]
```

## Multi-stage Builds

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

## Best Practices

[Content here]

---

## Quiz

[10 questions about Dockerfile]
```

- [ ] **Step 4: Create 05-docker-compose.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/05-docker-compose.html)

# Docker Compose

## YAML Syntax

```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
```

## Services, Networks, Volumes

[Content here]

---

## Quiz

[10 questions about Docker Compose]
```

- [ ] **Step 5: Create 06-advanced.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/06-advanced.html)

# Advanced Topics

## Security

[Content here]

## Logging & Monitoring

[Content here]

## Docker Swarm

[Content here]

## Kubernetes Overview

[Content here]

---

## Quiz

[10 questions about advanced topics]
```

- [ ] **Step 6: Create 07-workshop-basic.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/07-workshop-basic.html)

# Workshop: Basic Operations

## Objectives

- Run a container
- Exec into a container
- Stop and remove containers

## Lab Exercise

```bash
# Step 1: Run CentOS container
docker run -it --name lab-centos centos:stream9 /bin/bash

# Step 2: Inside container, check OS
cat /etc/os-release

# Step 3: Exit container
exit

# Step 4: List containers
docker ps -a

# Step 5: Stop container
docker stop lab-centos

# Step 6: Remove container
docker rm lab-centos
```

---

## Quiz

[10 questions about basic operations]
```

- [ ] **Step 7: Create 08-workshop-webserver.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/08-workshop-webserver.html)

# Workshop: Web Server

## Objectives

- Deploy Nginx on CentOS 9 Stream
- Custom configuration
- SSL setup

## Lab Exercise

```bash
# Step 1: Create project directory
mkdir -p ~/nginx-docker && cd ~/nginx-docker

# Step 2: Create Dockerfile
cat > Dockerfile << 'EOF'
FROM centos:stream9
RUN dnf install -y nginx
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF

# Step 3: Create custom nginx.conf
cat > nginx.conf << 'EOF'
events {}
http {
    server {
        listen 80;
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
}
EOF

# Step 4: Build and run
docker build -t my-nginx .
docker run -d -p 8080:80 my-nginx
```

---

## Quiz

[10 questions about web server]
```

- [ ] **Step 8: Create 09-workshop-fullstack.md**

```markdown
!!! tip "Slides Available"
    📊 **View Presentation** → [Open Slides](../presents/09-workshop-fullstack.html)

# Workshop: Fullstack Application

## Architecture

| Component | Technology |
|-----------|------------|
| Frontend | ReactJS + TypeScript |
| Backend | FastAPI |
| Database | PostgreSQL |
| Proxy | Nginx |

## Project Structure

```
fullstack-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
└── nginx/
    └── nginx.conf
```

## docker-compose.yml

```yaml
version: '3.8'
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
  
  backend:
    build: ./backend
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://user:secret@db:5432/appdb
  
  frontend:
    build: ./frontend
    depends_on:
      - backend
  
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    depends_on:
      - frontend
      - backend

volumes:
  pgdata:
```

## Frontend (React + TypeScript)

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
RUN npm run build
FROM nginx:alpine
COPY --from=0 /app/dist /usr/share/nginx/html
```

## Backend (FastAPI)

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Running the Application

```bash
docker-compose up -d
```

---

## Quiz

[10 questions about fullstack workshop]
```

- [ ] **Step 9: Commit all chapters**

```bash
git add docs/book/
git commit -m "feat: add all book chapters with content and quizzes"
```

---

### Task 5: Create all presentations (02-09)

**Files:**
- Create: `docs/presents/02-installation.html`
- Create: `docs/presents/03-basic-operations.html`
- Create: `docs/presents/04-dockerfile.html`
- Create: `docs/presents/05-docker-compose.html`
- Create: `docs/presents/06-advanced.html`
- Create: `docs/presents/07-workshop-basic.html`
- Create: `docs/presents/08-workshop-webserver.html`
- Create: `docs/presents/09-workshop-fullstack.html`

**Interfaces:**
- Consumes: Template from Task 3
- Produces: All 9 reveal.js presentations

- [ ] **Step 1: Create 02-installation.html**

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Docker Installation</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@4.6.0/dist/reveal.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@4.6.0/dist/theme/white.css">
</head>
<body>
    <div class="reveal">
        <div class="slides">
            <section>
                <h1>Docker Installation</h1>
                <p>CentOS 9 Stream Setup</p>
            </section>
            <section>
                <h2>Prerequisites</h2>
                <ul>
                    <li>CentOS 9 Stream</li>
                    <li>Root/sudo access</li>
                    <li>Internet connection</li>
                </ul>
            </section>
            <section>
                <h2>Install Docker CE</h2>
                <pre><code>sudo dnf install -y docker-ce</code></pre>
            </section>
            <section>
                <h1>Thank You</h1>
                <p><a href="../book/02-installation.html">Back to Book</a></p>
            </section>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/reveal.js@4.6.0/dist/reveal.js"></script>
    <script>
        Reveal.initialize({
            hash: true,
            slideNumber: true
        });
    </script>
</body>
</html>
```

- [ ] **Step 2: Create remaining presentations (03-09)**

Repeat similar structure for each presentation, adapting content to match book chapters.

- [ ] **Step 3: Commit all presentations**

```bash
git add docs/presents/
git commit -m "feat: add all reveal.js presentations"
```

---

### Task 6: Update mkdocs.yml for navigation

**Files:**
- Modify: `mkdocs.yml`

**Interfaces:**
- Consumes: All created files
- Produces: Updated MkDocs configuration

- [ ] **Step 1: Update mkdocs.yml**

```yaml
site_name: Kmutnb modern infrastructure
theme:
  logo: assets/logo.png
  name: material
  font:
    text: IBM Plex Sans Thai Looped
  features:
    - navigation.top
    - navigation.footer
    
nav:
  - Home: index.md
  - Book:
    - 1. Fundamentals: book/01-fundamentals.md
    - 2. Installation: book/02-installation.md
    - 3. Basic Operations: book/03-basic-operations.md
    - 4. Dockerfile: book/04-dockerfile.md
    - 5. Docker Compose: book/05-docker-compose.md
    - 6. Advanced: book/06-advanced.md
    - 7. Workshop Basic: book/07-workshop-basic.md
    - 8. Workshop Webserver: book/08-workshop-webserver.md
    - 9. Workshop Fullstack: book/09-workshop-fullstack.md

markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences

copyright: Copyright &copy; 2025 sawangpong muadphet
```

- [ ] **Step 2: Verify navigation works**

Run: `mkdocs serve` and check sidebar navigation

- [ ] **Step 3: Commit**

```bash
git add mkdocs.yml
git commit -m "feat: update mkdocs.yml with navigation and extensions"
```

---

### Task 7: Final verification

**Files:**
- None (verification only)

**Interfaces:**
- Consumes: All created files
- Produces: Verified working documentation site

- [ ] **Step 1: Build site**

Run: `mkdocs build`

- [ ] **Step 2: Serve and test all links**

Run: `mkdocs serve`

- [ ] **Step 3: Verify all wiki-links work**

Click each link in index.md

- [ ] **Step 4: Verify all presentation links work**

Click each "View Presentation" callout in book chapters

- [ ] **Step 5: Verify quizzes render**

Check collapsible quiz sections in each chapter

- [ ] **Step 6: Final commit**

```bash
git add -A
git commit -m "docs: complete Docker Container Technology documentation"
```

---

## Implementation Order

1. Task 1: index.md
2. Task 2: Chapter template
3. Task 3: Presentation template
4. Task 4: All book chapters
5. Task 5: All presentations
6. Task 6: mkdocs.yml
7. Task 7: Final verification

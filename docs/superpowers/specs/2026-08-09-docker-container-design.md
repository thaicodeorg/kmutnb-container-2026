# Docker Container Technology - Design Spec

## Overview

A MkDocs-based educational resource for Computer Science students covering Docker container technology. Includes book chapters with quizzes and matching reveal.js presentations.

**Target Audience:** Computer Science students  
**Base OS:** CentOS 9 Stream  
**Language:** English  

## Structure

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

## Book Chapters

| # | Chapter | File | Topics |
|---|---------|------|--------|
| 1 | Fundamentals | `01-fundamentals.md` | What are containers, VMs vs containers, Docker architecture, container ecosystem |
| 2 | Installation | `02-installation.md` | Docker CE on CentOS 9 Stream, rootless mode, configuration |
| 3 | Basic Operations | `03-basic-operations.md` | Images, containers, networking, volumes, port mapping |
| 4 | Dockerfile | `04-dockerfile.md` | Instructions, multi-stage builds, best practices, .dockerignore |
| 5 | Docker Compose | `05-docker-compose.md` | YAML syntax, services, networks, volumes, environment variables |
| 6 | Advanced | `06-advanced.md` | Security, logging, monitoring, Docker Swarm intro, K8s overview |
| 7 | Workshop: Basic | `07-workshop-basic.md` | Hands-on: run, exec, stop, remove containers |
| 8 | Workshop: Webserver | `08-workshop-webserver.md` | Nginx/Apache on CentOS 9, custom config, SSL |
| 9 | Workshop: Fullstack | `09-workshop-fullstack.md` | React+TypeScript frontend, FastAPI backend, PostgreSQL database |

## Chapter Format

Each book chapter includes:

1. **Callout banner** - Links to matching presentation
   ```markdown
   !!! tip "Slides Available"
       📊 **View Presentation** → [Open Slides](../presents/XX-name.html)
   ```

2. **Main content** - Explanations, code examples, diagrams

3. **Quiz section** - 10 questions at end (collapsible `<details>` blocks)

## Presentations

- **Format:** reveal.js HTML files
- **Mapping:** 1:1 with book chapters
- **Content:** Slide summaries of book content for classroom teaching

## index.md

Table of contents using `[[wiki-link]]` format:
```markdown
## Docker Technology

### Book
- [[book/01-fundamentals|1. Fundamentals]]
- [[book/02-installation|2. Installation]]
...

### Presentations
- [[presents/01-fundamentals|1. Fundamentals Slides]]
...
```

## Workshop 9: Fullstack Application

| Component | Technology |
|-----------|------------|
| Frontend | ReactJS + TypeScript |
| Backend | FastAPI (Python) |
| Database | PostgreSQL |
| Web Server | Nginx (reverse proxy) |

## Implementation Order

1. Create `index.md` with wiki-links
2. Create book chapters (01-09) with content + quizzes
3. Create reveal.js presentations (01-09)
4. Test all links and navigation

# Docker Container Technology

## Why Learn Docker?

> "The future of software development is containerized. Every developer, every team, every company needs to understand containers."

---

## The World is Changing

### Traditional Software Development
- "It works on my machine" - the most common excuse
- Different environments for development, testing, and production
- Hours wasted debugging environment issues
- Deployment nightmares across different servers

### Modern Software Development with Docker
- Build once, run anywhere
- Consistent environments from laptop to cloud
- Deploy in seconds, not hours
- Scale effortlessly with container orchestration

---

## Career Opportunities

### Docker Skills = High Demand Jobs

| Role | Average Salary (USD) | Demand |
|------|---------------------|--------|
| DevOps Engineer | $120,000 - $160,000 | 🔥🔥🔥🔥🔥 |
| Cloud Engineer | $115,000 - $155,000 | 🔥🔥🔥🔥🔥 |
| Software Engineer | $100,000 - $150,000 | 🔥🔥🔥🔥 |
| Site Reliability Engineer | $130,000 - $170,000 | 🔥🔥🔥🔥🔥 |
| Platform Engineer | $125,000 - $165,000 | 🔥🔥🔥🔥🔥 |

**Source: Glassdoor, Indeed, LinkedIn Jobs (2024-2025)**

---

## Companies Using Docker

### Tech Giants
- **Google** - Kubernetes (born from Google's container orchestration)
- **Amazon** - AWS ECS, EKS, Fargate
- **Microsoft** - Azure Container Instances, AKS
- **Netflix** - Thousands of microservices in containers
- **Spotify** - Deploy 800+ microservices daily
- **Twitter** - Containerized infrastructure

### Startups & Enterprise
- **Airbnb** - Microservices architecture
- **Uber** - Containerized ride-sharing platform
- **PayPal** - Migrated to containers for scalability
- **Samsung** - IoT and edge computing with containers

---

## What You'll Learn

### Module 1: Fundamentals
- What are containers and why they matter
- Docker architecture and components
- VMs vs Containers comparison

### Module 2: Installation & Setup
- Docker CE on CentOS 9 Stream
- Rootless mode and security configuration
- Post-installation best practices

### Module 3: Basic Operations
- Container lifecycle management
- Image management and optimization
- Networking and storage fundamentals

### Module 4: Dockerfile Mastery
- Writing efficient Dockerfiles
- Multi-stage builds for production
- Security best practices

### Module 5: Docker Compose
- Multi-container applications
- Service orchestration
- Environment management

### Module 6: Advanced Topics
- Docker security hardening
- Docker Swarm and Kubernetes intro
- Monitoring and logging

### Module 7-9: Hands-on Workshops
- Build real-world applications
- Deploy fullstack apps with React, FastAPI, PostgreSQL
- Nginx web server configuration

### Module 10: Deep Dive
- Linux kernel features (cgroups, namespaces)
- Container security internals
- Advanced security scanning tools

---

## Real-World Projects You'll Build

### Workshop 1: Basic Operations
```bash
# Run your first container
docker run -d --name my-app nginx

# See it in action
docker ps
curl http://localhost:80
```

### Workshop 2: Web Server
- Configure Nginx on CentOS 9 Stream
- Custom configuration and SSL setup
- Reverse proxy configuration

### Workshop 3: Fullstack Application
```
┌─────────────────────────────────────┐
│           Frontend                  │
│       React + TypeScript            │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│           Backend                   │
│          FastAPI (Python)           │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│           Database                  │
│          PostgreSQL                 │
└─────────────────────────────────────┘
```

---

## Why This Course?

### 1. Industry-Relevant Skills
- Learn technologies used by top companies
- Build a portfolio of real projects
- Prepare for DevOps and cloud roles

### 2. Hands-on Learning
- 3 practical workshops
- Real code, real deployments
- Step-by-step guidance

### 3. Complete Coverage
- From basics to advanced security
- Linux kernel internals
- Production best practices

### 4. Career Ready
- Skills matching job market demands
- Portfolio projects for interviews
- Certification preparation

---

## Learning Path

```
Beginner ──────────────────────────────────────────────> Advanced

[1. Fundamentals] → [2. Installation] → [3. Operations]
        │                   │                   │
        ▼                   ▼                   ▼
[4. Dockerfile] ──> [5. Compose] ──> [6. Advanced]
        │                   │                   │
        ▼                   ▼                   ▼
[7-9. Workshops] ──────────────────────────> [10. Security]
```

---

## Student Testimonials

> "Docker changed how I think about software deployment. No more 'it works on my machine' excuses!" - Computer Science Student

> "After learning Docker, I got interviews at three cloud companies. The demand is real!" - Recent Graduate

> "The workshops were amazing. I built a fullstack app and deployed it in containers. My portfolio looks professional now!" - CS Student

---

## Getting Started

### Prerequisites
- Basic Linux command line knowledge
- Text editor (VS Code recommended)
- Computer with 4GB+ RAM

### Quick Start
```bash
# Install Docker on CentOS 9 Stream
sudo dnf install -y docker-ce docker-ce-cli containerd.io

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
docker run hello-world
```

---

## Course Resources

- **Book**: 10 comprehensive chapters
- **Presentations**: Slide decks for each topic
- **Workshops**: 3 hands-on labs
- **PDF Summary**: Quick reference guide

---

## The Future is Containers

### Industry Trends (2024-2025)
- 92% of organizations use containers in production
- Container market growing 25% annually
- Kubernetes adoption up 67% year-over-year
- Cloud-native skills most in-demand

### Your Advantage
By completing this course, you'll have:
- ✅ In-demand technical skills
- ✅ Real project experience
- ✅ Understanding of modern architecture
- ✅ Foundation for Kubernetes and cloud

---

## Ready to Start?

**Open the first chapter and begin your Docker journey!**

[1. Fundamentals →](book/01-fundamentals.md)

---

## Contact & Support

**Instructor**: Sawangpong Muadphet
**Institution**: KMUTNB (King Mongkut's University of Technology North Bangkok)
**Department**: Computer Science

---

## License

Copyright © 2025 Sawangpong Muadphet. All rights reserved.

---

*Last updated: August 2025*

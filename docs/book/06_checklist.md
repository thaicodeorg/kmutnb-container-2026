# ✅ Chapter 6: Docker Advanced — Student Self-Verification Checklist

> **How to use this checklist:**
> - Tick `☐` in the **Done** column as you complete each step.
> - Take a **screenshot** for every step marked with 📸 — required for submission.
> - Fill in the **Notes** column if you hit errors or made changes.
> - Complete the **Concept Mastery** and **Quiz Self-Check** sections at the end.

---

## 🏭 Workshop 1: Image Optimization

### 📘 Lab 1.1: Multi-stage Build – Go Application

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Create project dir | `mkdir ~/go-multistage && cd ~/go-multistage` | Directory exists | ☐ | ☐ | |
| 2 | Create `main.go` | `cat > main.go << 'EOF' ... EOF` | Go HTTP server on :8080 | ☐ | ☐ | |
| 3 | Create `Dockerfile.single` | Single-stage `golang:1.21` build | File exists | ☐ | ☐ | |
| 4 | Build single-stage | `docker build -t go-single -f Dockerfile.single .` | Image built | ☐ | ☐ | |
| 4b | Check size | `docker images go-single` | ~1.28 GB disk / 316 MB content | ☐ | ☐ | |
| 5 | Create multi-stage `Dockerfile` | `builder` + `alpine:3.18` final | File exists | ☐ | ☐ | |
| 6 | Build multi-stage | `docker build -t go-multi .` | ~23 MB disk / 7.61 MB content | ☐ | ☐ | |
| 6b | Compare | `docker images \| grep go-` | Both images listed | ☐ | ☐ | |
| 7 | Verify app works | `docker run -d --name go-app -p 8080:8080 go-multi` + `curl localhost:8080` | "Hello from Go container!" | ☐ | ☐ | |
| 7b | Cleanup | `docker stop go-app && docker rm go-app` | Container removed | ☐ | ☐ | |

---

### 📘 Lab 1.2: BuildKit Features

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Enable BuildKit | `export DOCKER_BUILDKIT=1` | Env var set | ☐ | ☐ | |
| 2 | Create `Dockerfile.buildkit` | With `--mount=type=cache` for Go mod/build | File exists | ☐ | ☐ | |
| 3 | Build with BuildKit | `DOCKER_BUILDKIT=1 docker build -t go-buildkit -f Dockerfile.buildkit .` | Image built | ☐ | ☐ | |
| 3b | Time cached rebuild | `time docker build -t go-buildkit-cache -f Dockerfile.buildkit .` | Second build faster | ☐ | ☐ | |
| 4 | Create secret file | `echo "my-api-key-12345" > api-key.txt` | File exists | ☐ | ☐ | |
| 4b | Create `Dockerfile.secret` | Uses `--mount=type=secret,id=apikey` | File exists | ☐ | ☐ | |
| 4c | Build with secret | `docker build --secret id=apikey,src=api-key.txt -t go-secret .` | Image built | ☐ | ☐ | |
| 5 | Verify secret NOT in image | `docker history go-secret` | No `apikey` layer visible | ☐ | ☐ | |

---

### 📘 Lab 1.3: Image Analysis

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Inspect layers | `docker history go-multi` and `docker history go-single` | Layer sizes shown | ☐ | ☐ | |
| 2 | Install `dive` | `sudo dnf install -y https://github.com/wagoodman/dive/releases/download/v0.13.1/dive_0.13.1_linux_amd64.rpm` | `dive --version` works | ☐ | ☐ | |
| 2b | Analyze image | `dive go-multi` | Two-pane UI, efficiency score | ☐ | ☐ | |
| 3 | Create `.dockerignore` | `cat > .dockerignore << 'EOF' ... EOF` | File exists | ☐ | ☐ | |
| 4 | Rebuild optimized | `DOCKER_BUILDKIT=1 docker build -t go-optimized .` | Image built faster | ☐ | ☐ | |
| 4b | Compare | `docker images go-optimized` | Size close to `go-multi` | ☐ | ☐ | |

---

## 🔬 Workshop 2: Container Internals

### 📘 Lab 2.1: Understanding Overlay Filesystem

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Run CentOS container | `docker run -it --name overlay-test quay.io/centos/centos:stream10 /bin/bash` | Shell inside container | ☐ | ☐ | |
| 2 | View overlay mount | `mount \| grep overlay` and `cat /proc/mounts \| grep overlay` | Shows `lowerdir`, `upperdir`, `workdir` | ☐ | ☐ | |
| 2b | Break down output | Identify each field | Can explain each | ☐ | ☐ | |
| 3 | Create files | `mkdir -p /test && echo "container file" > /test/file.txt && cat /test/file.txt` | File created & displayed | ☐ | ☐ | |
| 4 | Exit & restart | `exit` then `docker start overlay-test && docker ps` | Container running again | ☐ | ☐ | |
| 4b | Find `upperdir` | `cat /proc/$(docker inspect -f '{{.State.Pid}}' overlay-test)/mountinfo \| grep overlay` | Paths shown | ☐ | ☐ | |
| 5 | View layer contents | **Rootless:** `ls ~/.local/share/docker/containerd/.../snapshots/` **Rootful:** `sudo ls /var/lib/docker/overlay2/` | Layer dirs listed | ☐ | ☐ | |
| 6 | Compare & commit | `docker diff overlay-test` → `docker commit overlay-test overlay-test-snapshot` → `docker history overlay-test-snapshot` | Diff shows `/test`, new image created | ☐ | ☐ | |

---

### 📘 Lab 2.2: Resource Limits

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Memory limit | `docker run -d --name memory-limit --memory=256m quay.io/centos/centos:stream10 tail -f /dev/null` | `docker stats` shows 256 MB limit | ☐ | ☐ | |
| 2 | CPU limit | `docker run -d --name cpu-limit --cpus=0.5 ...` | 0.5 CPU limit in `stats` | ☐ | ☐ | |
| 3 | PID limit | `docker run -d --name pid-limit --pids-limit=50 ...` + fork test | PID fork fails after 50 | ☐ | ☐ | |
| 4 | Monitor all | `docker stats --no-stream` | Table of running containers | ☐ | ☐ | |
| 5 | Combined limits | `docker run -d --name combined-limit --memory=128m --cpus=0.25 --pids-limit=25 ...` | All limits applied | ☐ | ☐ | |

---

### 📘 Lab 2.3: Container Inspection

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Inspect container | `docker inspect combined-limit` | Full JSON config | ☐ | ☐ | |
| 1b | Format queries | `docker inspect --format='{{.HostConfig.Memory}}' combined-limit` | `134217728` (128 MB) | ☐ | ☐ | |
| 1c | CPU format | `docker inspect --format='{{.HostConfig.NanoCpus}}' combined-limit` | `250000000` (0.25 CPU) | ☐ | ☐ | |
| 2 | View logs | `docker logs overlay-test`, `docker logs -f ...`, `docker logs --tail 10 ...` | Logs shown (empty if idle) | ☐ | ☐ | |
| 3 | Stats table | `docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"` | Formatted table | ☐ | ☐ | |
| 4 | Process list | `docker top combined-limit` and `-o pid,comm` | Processes shown | ☐ | ☐ | |
| 5 | Diff changes | `docker exec combined-limit touch /tmp/test-file` then `docker diff combined-limit` | Shows `A /tmp/test-file` | ☐ | ☐ | |

---

## 🌐 Workshop 3: Advanced Networking

### 📘 Lab 3.1: Docker Network Types

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | List networks | `docker network ls`, `docker network inspect bridge` | Default bridge shown | ☐ | ☐ | |
| 2 | Create bridge | `docker network create --driver bridge my-bridge` | Network created | ☐ | ☐ | |
| 3 | Create host network | `docker network create --driver host my-host` | ⚠️ May error (`host` is built-in) | ☐ | ☐ | |
| 4 | Run containers | `docker run -d --name net-test-1 --network my-bridge ...` and `net-test-2` | Both running | ☐ | ☐ | |
| 4b | Install `ping` | `docker exec -it net-test-1 bash -c "dnf install -y iputils && ping -c 3 net-test-2"` | Ping succeeds | ☐ | ☐ | |
| 5 | Test communication | `docker exec net-test-1 ping net-test-2` and `curl net-test-2` | DNS resolves, ping succeeds | ☐ | ☐ | |

---

### 📘 Lab 3.2: Port Mapping and Exposing

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Map port | `docker run -d --name port-test -p 8080:80 quay.io/sclorg/httpd-24-c10s` + `curl localhost:8080` | HTTP response | ☐ | ☐ | |
| 2 | Random ports | `docker run -d --name port-random -P quay.io/sclorg/httpd-24-c10s` + `docker port port-random` | Random host port assigned | ☐ | ☐ | |
| 3 | Query ports | `docker port port-test` and `docker port port-test 80` | Mapping shown | ☐ | ☐ | |
| 4 | Bind to interface | `docker run -d --name port-bind-2 -p 127.0.0.1:9092:80 ...` + `curl 127.0.0.1:9092` | Only localhost works | ☐ | ☐ | |

---

### 📘 Lab 3.3: Upload Image to Docker Hub

| # | Step | Command / Action | Expected Result | Done | 📸 | Notes |
|---|------|------------------|-----------------|:----:|:--:|-------|
| 1 | Create Docker Hub account | Sign up at https://hub.docker.com | Account created | ☐ | ☐ | |
| 2 | Login from CLI | `docker login` | "Login Succeeded" | ☐ | ☐ | |
| 2b | Login to website | Browser login | Dashboard visible | ☐ | ☐ | |
| 3 | Tag image | `docker tag go-multi <username>/go-app:latest` and `:v1.0` | Tags visible in `docker images` | ☐ | ☐ | |
| 4 | Push image | `docker push <username>/go-app:latest` and `:v1.0` | Push succeeds | ☐ | ☐ | |
| 5 | Verify on Docker Hub | https://hub.docker.com/repositories | Image + both tags shown | ☐ | ☐ | |
| 6 | Pull image | `docker rmi ...` then `docker pull <username>/go-app:latest` | Pull succeeds | ☐ | ☐ | |
| 6b | Run pulled image | `docker run -d --name hub-app -p 8080:8080 <username>/go-app:latest` + `curl localhost:8080` | Response works | ☐ | ☐ | |
| 6c | Cleanup | `docker stop hub-app && docker rm hub-app` | Container removed | ☐ | ☐ | |
| 7 | Push private image | Create private repo, tag, push | Private repo visible | ☐ | ☐ | |
| 8 | Logout | `docker logout` | Logged out | ☐ | ☐ | |

---

## 📋 Overall Progress Summary

| Workshop | Labs | Total Steps | Completed |
|----------|------|:-----------:|:---------:|
| **1 – Image Optimization** | 1.1, 1.2, 1.3 | 24 | ☐ |
| **2 – Container Internals** | 2.1, 2.2, 2.3 | 17 | ☐ |
| **3 – Advanced Networking** | 3.1, 3.2, 3.3 | 22 | ☐ |
| **TOTAL** | **9 labs** | **63 steps** | ☐ |

---

## 🧠 Concept Mastery Self-Check

Tick only if you can **explain out loud** without looking:

| # | Concept | Can Explain? |
|---|---------|:------------:|
| 1 | Difference between DISK USAGE and CONTENT SIZE | ☐ |
| 2 | Why multi-stage builds shrink images | ☐ |
| 3 | How BuildKit cache mounts speed up rebuilds | ☐ |
| 4 | How `--mount=type=secret` keeps secrets out of layers | ☐ |
| 5 | What `dive` shows that `docker history` cannot | ☐ |
| 6 | What `.dockerignore` optimizes (context, cache, security) | ☐ |
| 7 | OverlayFS: `lowerdir`, `upperdir`, `workdir`, `merged` | ☐ |
| 8 | Copy-on-Write (CoW) behavior | ☐ |
| 9 | `--memory`, `--cpus`, `--pids-limit` and verification | ☐ |
| 10 | Difference between `-p` and `-P` | ☐ |
| 11 | DNS-based discovery on custom bridge networks | ☐ |
| 12 | Push/pull from Docker Hub (public + private) | ☐ |

---

## 📚 Quiz Self-Check (Chapter 6)

| # | Question | Answer Understood? |
|---|----------|:------------------:|
| 1 | Benefit of multi-stage builds? | ☐ |
| 2 | How to enable BuildKit? | ☐ |
| 3 | What is the overlay filesystem? | ☐ |
| 4 | How to limit container memory? | ☐ |
| 5 | Difference between `-p` and `-P`? | ☐ |
| 6 | How to create a custom Docker network? | ☐ |
| 7 | Purpose of `.dockerignore`? | ☐ |
| 8 | How to run with CPU limits? | ☐ |
| 9 | What is a Docker private registry? | ☐ |
| 10 | How to inspect container resource limits? | ☐ |

---

## 🔑 Troubleshooting Quick Reference

| Symptom | Fix |
|---|---|
| `ping: command not found` | `docker exec -it <container> dnf install -y iputils` |
| `docker logs` shows nothing | Container's PID 1 produces no stdout (e.g., `tail -f /dev/null`) — normal |
| `/var/lib/docker/overlay2` not found | Rootless Docker — use `~/.local/share/docker/containerd/.../snapshots/` |
| `docker inspect` has no `GraphDriver` | Modern containerd snapshotter hides it — use `mountinfo` or `docker diff` |
| `docker create --driver host` fails | `host` network already exists — skip or use `none`/`bridge` |
| Name conflict on `docker run` | `docker rm <name>` first |

---

## ✅ Submission Checklist

Before submitting, confirm:

- ☐ All **63 steps** completed and ticked
- ☐ **Screenshots** taken for every 📸 row
- ☐ **Concept Mastery** self-check completed (12 concepts)
- ☐ **Quiz** answers understood (10 questions)
- ☐ Containers cleaned up (`docker rm -f $(docker ps -aq)`) — optional but tidy
- ☐ This checklist included with your submission document

---

**Student Name:** ________________________
**Student ID:** __________________________
**Date Submitted:** ______________________
**Instructor Signature:** ________________
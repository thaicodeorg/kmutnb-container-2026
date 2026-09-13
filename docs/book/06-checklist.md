# ✅ Student Self-Verification Checklist: Docker Advanced (Chapter 6)

Use this table to tick off each **Workshop → Lab → Step** as you complete it. Add ✅ in the "Done" column and note any errors in the "Notes" column.

---

## 🏭 Workshop 1: Image Optimization

### Lab 1.1: Multi-stage Build – Go Application

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Create project dir | `mkdir ~/go-multistage && cd ~/go-multistage` | Directory exists | ☐ | |
| 2 | Create `main.go` | `cat > main.go << 'EOF' ... EOF` | File with HTTP server on :8080 | ☐ | |
| 3 | Create `Dockerfile.single` | Single-stage `golang:1.21` build | File exists | ☐ | |
| 4 | Build single-stage | `docker build -t go-single -f Dockerfile.single .` | Image built, ~1.28 GB disk / 316 MB content | ☐ | |
| 5 | Create `Dockerfile` (multi-stage) | Builder + `alpine:3.18` final | File exists | ☐ | |
| 6 | Build multi-stage | `docker build -t go-multi .` | Image ~23 MB disk / 7.61 MB content | ☐ | |
| 7 | Verify app works | `docker run -d --name go-app -p 8080:8080 go-multi` then `curl localhost:8080` | "Hello from Go container!" response | ☐ | |
| 7b | Cleanup | `docker stop go-app && docker rm go-app` | Container removed | ☐ | |

---

### Lab 1.2: BuildKit Features

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Enable BuildKit | `export DOCKER_BUILDKIT=1` | Env var set | ☐ | |
| 2 | Create `Dockerfile.buildkit` | With `--mount=type=cache` for Go mod/build | File exists | ☐ | |
| 3 | Build with BuildKit | `DOCKER_BUILDKIT=1 docker build -t go-buildkit -f Dockerfile.buildkit .` | Image built | ☐ | |
| 3b | Time cached rebuild | `time docker build -t go-buildkit-cache -f Dockerfile.buildkit .` | Second build faster (cache hit) | ☐ | |
| 4 | Create secret file | `echo "my-api-key-12345" > api-key.txt` | File exists | ☐ | |
| 4b | Create `Dockerfile.secret` | Uses `--mount=type=secret,id=apikey` | File exists | ☐ | |
| 4c | Build with secret | `DOCKER_BUILDKIT=1 docker build --secret id=apikey,src=api-key.txt -t go-secret .` | Image built | ☐ | |
| 5 | Verify secret NOT in image | `docker history go-secret` | No `apikey` layer visible | ☐ | |

---

### Lab 1.3: Image Analysis

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Inspect layers | `docker history go-multi` and `docker history go-single` | go-single shows ~13 layers, go-multi shows ~5 | ☐ | |
| 2 | Install `dive` | `sudo dnf install -y https://github.com/wagoodman/dive/releases/download/v0.13.1/dive_0.13.1_linux_amd64.rpm` | `dive --version` works | ☐ | |
| 3 | Analyze image | `dive go-multi` | Two-pane UI opens, efficiency score shown | ☐ | |
| 3b | Answer Q1 & Q2 | See "Try to Answer Questions" section | Concepts understood | ☐ | |
| 4 | Create `.dockerignore` | `cat > .dockerignore << 'EOF' ... EOF` | File exists | ☐ | |
| 5 | Rebuild optimized | `DOCKER_BUILDKIT=1 docker build -t go-optimized .` | Image built faster, similar size | ☐ | |
| 5b | Compare images | `docker images go-optimized` | Size close to `go-multi` | ☐ | |

---

## 🔬 Workshop 2: Container Internals

### Lab 2.1: Understanding Overlay Filesystem

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Run CentOS container | `docker run -it --name overlay-test quay.io/centos/centos:stream10 /bin/bash` | Shell inside container | ☐ | |
| 2 | View overlay mount | `mount \| grep overlay` and `cat /proc/mounts \| grep overlay` | Shows `lowerdir`, `upperdir`, `workdir` | ☐ | |
| 2b | Break down output | Identify each field (rw, lowerdir, upperdir, workdir, userxattr) | Can explain each | ☐ | |
| 3 | Create files | `mkdir -p /test && echo "container file" > /test/file.txt && cat /test/file.txt` | File created & displayed | ☐ | |
| 4 | Exit & inspect | `exit` then `docker inspect overlay-test \| grep -i "upperdir\|merged"` | Paths shown | ☐ | |
| 5 | View layer contents | `sudo ls -la /var/lib/docker/overlay2/` | Layer dirs listed | ☐ | |
| 6 | Compare & commit | `docker diff overlay-test`, `docker commit overlay-test overlay-test-snapshot`, `docker history overlay-test-snapshot` | Diff shows `/test`, new image created | ☐ | |

---

### Lab 2.2: Resource Limits

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Memory limit | `docker run -d --name memory-limit --memory=256m quay.io/centos/centos:stream10 tail -f /dev/null` | Container runs, limit shown in `stats` | ☐ | |
| 2 | CPU limit | `docker run -d --name cpu-limit --cpus=0.5 ...` | CPU limited to 50% | ☐ | |
| 3 | PID limit | `docker run -d --name pid-limit --pids-limit=50 ...` then fork test | PID fork fails after 50 | ☐ | |
| 4 | Monitor all | `docker stats --no-stream` | Table of running containers | ☐ | |
| 5 | Combined limits | `docker run -d --name combined-limit --memory=128m --cpus=0.25 --pids-limit=25 ...` | All limits applied | ☐ | |

---

### Lab 2.3: Container Inspection

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Inspect container | `docker inspect combined-limit` | Full JSON config | ☐ | |
| 1b | Format queries | `docker inspect --format='{{.HostConfig.Memory}}' combined-limit` | `134217728` (128 MB) | ☐ | |
| 2 | View logs | `docker logs combined-limit`, `docker logs --tail 10 ...` | Logs shown | ☐ | |
| 3 | Stats table | `docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"` | Formatted table | ☐ | |
| 4 | Process list | `docker top combined-limit` and `-o pid,comm` | Processes shown | ☐ | |
| 5 | Diff changes | `docker exec combined-limit touch /tmp/test-file` then `docker diff combined-limit` | Shows `A /tmp/test-file` | ☐ | |

---

## 🌐 Workshop 3: Advanced Networking

### Lab 3.1: Docker Network Types

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | List networks | `docker network ls`, `docker network inspect bridge` | Default bridge shown | ☐ | |
| 2 | Create bridge | `docker network create --driver bridge my-bridge` | Network created | ☐ | |
| 3 | Create host network | `docker network create --driver host my-host` | ⚠️ May fail (host is built-in) | ☐ | |
| 4 | Run containers on net | `docker run -d --name net-test-1 --network my-bridge ...` and `net-test-2` | Both running | ☐ | |
| 5 | Test communication | `docker exec net-test-1 ping net-test-2` | DNS resolves, ping succeeds | ☐ | |

---

### Lab 3.2: Port Mapping and Exposing

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Map port | `docker run -d --name port-test -p 8080:80 quay.io/sclorg/httpd-24-c10s` then `curl localhost:8080` | HTTP response | ☐ | |
| 2 | Random ports | `docker run -d --name port-random -P quay.io/sclorg/httpd-24-c10s` then `docker port port-random` | Random host port assigned | ☐ | |
| 3 | Query ports | `docker port port-test` and `docker port port-test 80` | Mapping shown | ☐ | |
| 4 | Bind to interface | `docker run -d --name port-bind -p 127.0.0.1:9090:80 ...` then `curl 127.0.0.1:9090` | Only localhost works | ☐ | |

---

### Lab 3.3: Private Registry Setup

| # | Step | Command / Action | Expected Result | Done | Notes |
|---|------|------------------|-----------------|------|-------|
| 1 | Run registry | `docker run -d --name registry -p 5000:5000 --restart always registry:2` | Registry container running | ☐ | |
| 2 | Tag image | `docker tag go-multi localhost:5000/go-app:latest` | Tag created | ☐ | |
| 3 | Push | `docker push localhost:5000/go-app:latest` | Push succeeds | ☐ | |
| 4 | Pull | `docker rmi localhost:5000/go-app:latest` then `docker pull localhost:5000/go-app:latest` | Pull succeeds | ☐ | |
| 5 | Verify | `curl http://localhost:5000/v2/_catalog` and `curl http://localhost:5000/v2/go-app/tags/list` | JSON with image name & tag | ☐ | |

---

## 📋 Final Self-Check Summary

| Workshop | Labs | Total Steps | Completed |
|----------|------|-------------|-----------|
| **1 – Image Optimization** | 1.1, 1.2, 1.3 | ~22 | ☐ |
| **2 – Container Internals** | 2.1, 2.2, 2.3 | ~17 | ☐ |
| **3 – Advanced Networking** | 3.1, 3.2, 3.3 | ~15 | ☐ |
| **TOTAL** | **9 labs** | **~54 steps** | ☐ |

---

## 🧠 Concept Mastery Self-Check

Tick these only if you can **explain out loud** without looking:

| # | Concept | Can Explain? |
|---|---------|--------------|
| 1 | Difference between DISK USAGE and CONTENT SIZE | ☐ |
| 2 | Why multi-stage builds shrink images (builder vs final) | ☐ |
| 3 | How BuildKit cache mounts speed up rebuilds | ☐ |
| 4 | How `--mount=type=secret` keeps secrets out of image layers | ☐ |
| 5 | What `dive` shows that `docker history` cannot | ☐ |
| 6 | What `.dockerignore` optimizes (context, cache, security) | ☐ |
| 7 | OverlayFS: `lowerdir`, `upperdir`, `workdir`, `merged` | ☐ |
| 8 | Copy-on-Write (CoW) behavior when modifying lower-layer files | ☐ |
| 9 | `--memory`, `--cpus`, `--pids-limit` and how to verify them | ☐ |
| 10 | Difference between `-p` and `-P` port mapping | ☐ |
| 11 | How custom bridge networks enable DNS-based container discovery | ☐ |
| 12 | How to push/pull from a local private registry | ☐ |

---

## 🔑 Quick Tips for Verification

- **Screenshots required?** The assignment asks for screenshots of each step — take one for every ☐ row.
- **If a command fails**, check:
  - Container name already in use → `docker rm <name>`
  - Image not found → check tag spelling
  - Permission denied → prefix with `sudo` (or add user to `docker` group)
- **Cleanup between labs:** `docker system prune -a` (⚠️ removes all unused images/containers).

Want me to convert this into a printable PDF-style format or a Markdown file you can drop into your submission?
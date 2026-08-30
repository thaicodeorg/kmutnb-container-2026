# Chapter 4: Docker Basics && Dockerfile Workshop (Using CentOS Stream 10)

## Reviews: 
### What is Docker?
Docker is a platform that uses OS-level virtualization to package applications into **containers**. Think of it as lightweight, portable units that include everything needed to run software—code, libraries, dependencies, and configuration.

**Key concepts:**
- **Image**: A read-only template or "blueprint" containing everything needed to run an application
- **Container**: A runnable instance of an image—an isolated environment where the application actually runs

### Docker vs. Virtual Machines
Unlike VMs that require a full operating system per instance, Docker containers share the host OS kernel. This makes containers:
- **Lighter** (megabytes vs. gigabytes for VMs)
- **Faster** to start (seconds vs. minutes)
- **More efficient** with system resources

### Why Use Docker?
- **Environment consistency**: Eliminates "it works on my machine" problems by providing identical environments everywhere
- **Portability**: Run containers on any system with Docker installed—from laptops to cloud servers
- **Isolation**: Containers run independently without interfering with each other

---

## Assignment: Create Student's Submission along tutorial
- Student will submit Screen of Docker installation process according to step-by-step from manual.
- Student can use Basic Template [Download](../assets/dockerbasic-dockerfile-submission.docx)  or generate own version.


### Using CentOS Stream 10 with Docker

If you want to use CentOS Stream 10 as your base image, here's how to find and use it:

#### Pre-built Images on Quay.io

visit [Ref https://quay.io/organization/sclorg ](https://quay.io/organization/sclorg) to review docker image. 

![](../assets/images/basic-docker1.png)

CentOS Stream 10 images are available on **Quay.io** (Red Hat's container registry). Examples include:

| Image Type | Registry Address |
|------------|------------------|
| Apache HTTP Server | `quay.io/sclorg/httpd-24-c10s` |
| PostgreSQL 16 | `quay.io/sclorg/postgresql-16-c10s` |
| Node.js 22 | `quay.io/sclorg/nodejs-22-c10s` |
| Node.js 22 Minimal | `quay.io/sclorg/nodejs-22-minimal-c10s` |
| PostgreSQL 18 | `quay.io/sclorg/postgresql-18-c10s` |
 
You can **pull** all of these images in table using:
- use command format ``docker pull <repo name/owner/image name>``
```bash
docker pull quay.io/sclorg/postgresql-16-c10s
```
![](../assets/images/basic-docker2.png)

> **Note**: *** pull all image with list in above tables and copy screen in templage.

- run command ``docker image ls`` to verify successfully image from table

![](../assets/images/basic-docker5.png)

#### Next Learn building from a Dockerfile
- create  basicdocker1 folder as project folder (Remeber there  is only one Dockerfile in each of project)
- project structure:  create file Dockerfile, index.html, app.js
    ```
    your-project/
    ├── Dockerfile
    ├── index.html
    └── app.js
    ```

```bash
# Create project folder holding your project
mkdir basicdocker1
cd basicdocker1
```
- use structure to generate file (Linux Admin skill call heredoc)
```
cat > Dockerfile << 'EOF'

## content will go here
EOF
```

- 1 generate ``Dockerfile`` to create docker image
```bash
cat > Dockerfile  << 'EOF'
FROM quay.io/sclorg/httpd-24-c10s

# --- Switch to root to install packages ---
USER root

# Install additional packages
RUN dnf install -y curl nginx

# --- Create log directories and set permissions for the 'default' user ---
# Added /var/log/nginx to the list of directories created and chowned
RUN mkdir -p /app/logs /run/nginx /var/lib/nginx/tmp /var/log/nginx && \
    chown -R default:root /app/logs /run/nginx /var/lib/nginx /var/log/nginx

# --- Switch back to the default non-root user ---
USER default

# Set working directory
WORKDIR /app

# --- Copy Nginx configuration ---
COPY nginx.conf /etc/nginx/nginx.conf

# --- Copy application files ---
COPY index.html app.js /app/

# Expose the unprivileged port
EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
EOF
```
output:
![](../assets/images/basic-docker6.png)

- 2 generate ``index.html``
```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Web is Running</title>
    <script src="app.js"></script>
</head>
<body>
    <h1>🚀 Web is Running!</h1>
    <p>Nginx is successfully serving files from <code>/app</code>.</p>
    <p>Open your browser console (F12) to see the message from <code>app.js</code>.</p>
</body>
</html>
EOF
```
output:
![](../assets/images/basic-docker7.png)

- 3 generate ``app.js`` 
```bash
cat > app.js << 'EOF'
console.log("✅ Web is running! App.js loaded successfully.");
alert("Hello from app.js!");
EOF
```
output:
![](../assets/images/basic-docker8.png)

- 4 generate ``nginx.conf``
```bash
cat > nginx.conf << 'EOF'
# --- Tell Nginx to write the PID file in the writable directory ---
pid /run/nginx/nginx.pid;
error_log /app/logs/error.log;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    access_log /app/logs/access.log;
    
    server {
        # Listen on an unprivileged port since we are running as a non-root user
        listen 8080;
        server_name localhost;

        root /app;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}
EOF
```
output:  
![](../assets/images/basic-docker9.png)

### List output files in project folder 

![](../assets/images/basic-docker10.png)


## Explaination
```dockerfile
cat > Dockerfile  << 'EOF'
FROM quay.io/sclorg/httpd-24-c10s

# --- Switch to root to install packages ---
USER root

# Install additional packages
RUN dnf install -y curl nginx

# --- Create log directories and set permissions for the 'default' user ---
# Added /var/log/nginx to the list of directories created and chowned
RUN mkdir -p /app/logs /run/nginx /var/lib/nginx/tmp /var/log/nginx && \
    chown -R default:root /app/logs /run/nginx /var/lib/nginx /var/log/nginx

# --- Switch back to the default non-root user ---
USER default

# Set working directory
WORKDIR /app

# --- Copy Nginx configuration ---
COPY nginx.conf /etc/nginx/nginx.conf

# --- Copy application files ---
COPY index.html app.js /app/

# Expose the unprivileged port
EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
EOF

```

#### Enable firewall 
```bash
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```


#### Build and Run
```bash
# Build the image (do not 'dot' end of command)
docker build -t my-http-app .
```
output: 
![](../assets/images/basic-docker11.png)

#### list your image
```
docker image  ls
```
![](../assets/images/basic-docker12.png)
```

mkdir -p ./host_logs
sudo chmod 777 ./host_logs
docker run  -p 8080:80 --name my-http-container  -v "$(pwd)/host_logs:/app/logs" my-http-app

# after run terminal will hold session - not return terminal to user
```
- open browser and view index.html

![](../assets/images/basic-docker13.png)

#### Open open new terminal
```bash
# Run a container from the image
docker exec -it my-http-container /bin/bash
```

![](../assets/images/basic-docker14.png)



## Try to understand most core basic of Docker Technology
When the **Docker daemon (service)** processes a `Dockerfile` during `docker build`, it reads the instructions **top-down**, executing each statement sequentially. Each instruction (except `CMD`) creates a new **read-only layer** in the image filesystem, which is cached for future builds.

Here is a comprehensive summary in **table format**, covering when, how, and what each of your specified commands does:

| Instruction | Execution Phase | Primary Purpose | Creates a Layer? | Key Behavior & Details |
| :--- | :--- | :--- | :--- | :--- |
| **`FROM`** | **Build Time** | Sets the base image for subsequent instructions. | **Yes** (pulls parent layers) | The service checks the local cache; if missing, it pulls the image from the registry. Every valid Dockerfile must start with this. |
| **`RUN`** | **Build Time** | Executes commands (e.g., `dnf install`) in a new shell. | **Yes** | The daemon runs the command, commits the result to a new layer, and saves it. This is where you install system packages and modify files inside the image. |
| **`WORKDIR`** | **Build Time**<br>*(affects Runtime)* | Sets the current working directory for **subsequent** `RUN`, `CMD`, `COPY`, and `ENTRYPOINT` instructions. | **No** (metadata only) | If the directory does not exist inside the temporary container, the daemon automatically creates it. It changes the filesystem path context for the next commands. |
| **`COPY`** | **Build Time** | Copies new files/directories from the **build context** (your host machine) into the container's filesystem. | **Yes** | The daemon reads the source files from the host, streams them into the build container, and commits the new filesystem state as a layer. *Note: It does NOT support URL fetching (use `ADD` for that).* |
| **`CMD`** | **Runtime**<br>(Container Start) | Provides default arguments/command for the running container. | **No** (config metadata) | The daemon **does not execute** this during `docker build`. Instead, it stores this command as metadata in the image config. When you run `docker run` (or the service starts a container), the daemon executes this. **Important:** If the container runs with a different command (e.g., `docker run <image> /bin/bash`), this `CMD` is entirely **overridden**. |

---

###  Additional Context from the Daemon's Perspective

1.  **Layer Caching**: The daemon uses a cache to speed up builds. If it finds an unchanged instruction (e.g., `COPY . .` with unchanged files) and a cached layer exists, it will reuse it instead of rebuilding.
2.  **Build Context**: Before processing `COPY`, the daemon receives a compressed "build context" (the directory where you run `docker build`). `COPY` can only access files within this context for security reasons.
3.  **Shell Form vs. Exec Form**: 
    - `RUN dnf install -y curl` uses **shell form** (runs `/bin/sh -c`).
    - `CMD ["nginx", "-g", "daemon off;"]` uses **exec form** (runs directly without a shell). The daemon prefers exec form because it handles signals (like `SIGTERM`) properly, allowing for graceful shutdowns.
4.  **Docker Service (Swarm/Compose)**: If you deploy this image via **Docker Service** (`docker service create`), the service manager ignores the `CMD` if you specify a command in the service definition, exactly like `docker run`.



### Basic Docker Commands Cheat Sheet

| Command | Purpose |
|---------|---------|
| `docker pull IMAGE` | Download an image from a registry |
| `docker build -t NAME .` | Build an image from a Dockerfile |
| `docker run IMAGE` | Create and start a container from an image |
| `docker run -it IMAGE /bin/bash` | Run interactively with a shell |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker stop CONTAINER_ID` | Stop a running container |
| `docker rm CONTAINER_ID` | Remove a stopped container |
| `docker image ls` | List downloaded images |

### Important Notes
- Docker commands typically require `sudo` permission
- You can add your user to the `docker` group to avoid using `sudo`
- CentOS Stream 10 images are available primarily through **Quay.io**, not Docker Hub




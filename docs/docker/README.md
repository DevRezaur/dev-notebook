# Docker Knowledge Base

## Table of Contents

1. [Introduction to Docker](#1-introduction-to-docker)
2. [Docker Installation & Setup](#2-docker-installation--setup)
3. [Docker Images](#3-docker-images)
4. [Docker Containers](#4-docker-containers)
5. [Docker Volumes & Mounts](#5-docker-volumes--mounts)
6. [Networking in Docker](#6-networking-in-docker)
7. [Docker Compose](#7-docker-compose)
8. [Docker Registry & Repositories](#8-docker-registry--repositories)
9. [Docker Security](#9-docker-security)
10. [Docker Swarm (Optional)](#10-docker-swarm-optional)
11. [Docker in CI/CD](#11-docker-in-cicd)
12. [Troubleshooting & Debugging](#12-troubleshooting--debugging)
13. [Docker Best Practices](#13-docker-best-practices)

---

## 1. Introduction to Docker

### What is Docker?
Docker is an _open-source containerization platform_ that packages an application and **all of its dependencies** (libraries, runtime, configuration files) into an isolated, lightweight unit called a **container**. These containers run on any host with the Docker Engine, guaranteeing that "_it works on my machine_" also means "_it works everywhere._"

Key characteristics:

* **Process-level isolation** using kernel features such as namespaces and cgroups.
* **Immutability**—an image (the template used to instantiate containers) is read-only once built.
* **Layered file system** for space-efficient storage and caching.
* **Portable runtime** across Linux, macOS, and Windows.

### Docker vs Virtual Machines

|                       | Docker Container | Virtual Machine |
|-----------------------|------------------|-----------------|
| Abstraction Layer     | OS **kernel**    | **Hardware** via hypervisor |
| Guest OS required?    | No               | Yes             |
| Startup time          | Seconds          | Minutes         |
| Resource overhead     | Low (shared OS)  | High (full OS per VM) |
| Isolation strength    | Process-level    | Full OS boundary |

`TL;DR` – Containers trade a bit of isolation for massive gains in density and speed.

### Benefits and Use Cases

1. **Consistent Dev-Prod parity**—same image runs on laptop, CI, and production.
2. **Microservices**—each service ships in its own container image.
3. **CI/CD**—ephemeral, reproducible build environments.
4. **Legacy apps**—wrap an old binary in a modern deployment unit.
5. **Education & Experimentation**—quickly spin up databases, message queues, etc.

### Docker Architecture

**High-level components**

Docker follows a _client ↔ daemon_ model:

* **docker CLI / Docker Compose** — sends REST commands via a Unix socket (or TCP) to the daemon.
* **Docker Daemon (`dockerd`)** — builds images, runs containers, manages networks & volumes.
* **Images** — immutable, layered filesystem snapshots stored locally under `/var/lib/docker`.
* **Containers** — runtime processes created from images with an extra writable layer.
* **Volumes** — host-side directories managed by Docker and mounted into containers for persistent or shared data.
* **Registries** — remote stores such as Docker Hub, GHCR, ECR where images are pushed & pulled.

The CLI communicates with `dockerd`; the daemon pulls images from registries, instantiates containers, attaches them to networks, and mounts volumes as requested.

---

## 2. Docker Installation & Setup

### Platform-specific Installers

| OS      | Method | Notes |
|---------|--------|-------|
| **Linux** | `apt`, `yum`, or `dnf` repos provided by Docker Inc. | Kernel ≥ 3.10 |
| **macOS** | [Docker Desktop](https://www.docker.com/products/docker-desktop/) `.dmg` | Bundles Docker Engine inside an xhyve/Apple HV VM |
| **Windows 10/11** | Docker Desktop (WSL 2 backend preferred) | Requires WSL 2 feature enabled |

```bash
# Example: Ubuntu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER  # run docker without sudo (logout/login required)
```

### Docker Desktop

GUI that wraps Docker Engine, Compose, Kubernetes (optional) and provides resource controls and automatic updates.

### Docker CLI vs Docker Engine

* **Docker Engine** – The daemon (`dockerd`) that exposes a REST API over a Unix socket/Windows named pipe.
* **Docker CLI** – The `docker` command that talks to that API. You can swap the CLI for other tools or script directly against the socket.

### Installing Docker Compose

Compose V2 is now bundled with recent Docker Desktop or packaged as a CLI plugin:

```bash
# Linux standalone binary (if not using Docker Desktop)
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.24.6/docker-compose-linux-x86_64 \
    -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

Verify:
```bash
docker compose version
```

---

## 3. Docker Images

### What is a Docker Image?
A snapshot of a filesystem _plus_ metadata (environment variables, default command, exposed ports). Images are **immutable**—every change creates a new image or layer.

### Image Layers & Union File System
Each `RUN`, `COPY`, or `ADD` instruction in a **Dockerfile** produces a new layer. At runtime, the container mounts a **union view** where lower layers are read-only and the top writable layer captures changes.

```text
my-image:latest
├─ Layer 4  <-- newest (e.g., application code)
├─ Layer 3  <-- dependencies
├─ Layer 2  <-- OS packages
└─ Layer 1  <-- base image (scratch/alpine/ubuntu)
```

### Official vs Custom Images
* **Official** – Curated by Docker (e.g., `python`, `nginx`). Security-scanned, minimal tags.
* **Custom** – Your own images, typically starting `FROM` an official base.

### Building Images with Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt --no-cache-dir
COPY . .
CMD ["python", "main.py"]
```

Build & tag:
```bash
docker build -t myapp:1.0 .
```

### Common Dockerfile Instructions
| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image |
| `RUN` | Execute command during build |
| `COPY` / `ADD` | Copy files |
| `CMD` | Default command (can be overridden) |
| `ENTRYPOINT` | Hard-wired command (arguments can be appended) |
| `ENV` | Set environment variables |
| `ARG` | Build-time variables |
| `EXPOSE` | Document the port |
| `LABEL` | Metadata |

`ENTRYPOINT` + `CMD` pattern:
```dockerfile
ENTRYPOINT ["/usr/bin/git"]
CMD ["--help"]  # default args
```

### Multi-stage Builds
Combine multiple `FROM` statements to build artifacts in one stage and copy into a minimal runtime stage.

```dockerfile
# 1️⃣ Build stage
FROM golang:1.21-bullseye AS builder
WORKDIR /src
COPY . .
RUN go build -o server

# 2️⃣ Minimal runtime image (~10 MB)
FROM gcr.io/distroless/base-debian12
COPY --from=builder /src/server /server
ENTRYPOINT ["/server"]
```

### .dockerignore
Works like `.gitignore`; prevents unnecessary files (e.g., `.git`, `node_modules`) from being sent to the daemon.

### Image Tagging & Versioning
Semantic version (`1.2.3`), date (`2024-06-25`), or Git SHA. Avoid `latest` in production.

```bash
docker build -t ghcr.io/acme/myapp:1.2.3 .
```

### Image Optimization Tips
* Start from a small base (`alpine`, `scratch`, `distroless`).
* Merge `RUN` steps and clean package caches.
* Leverage `.dockerignore` to exclude dev files.
* Use multi-stage builds to omit build-time dependencies.

---

## 4. Docker Containers

### What is a Container?
A _live instance_ of an image, with its own PID namespace, network stack, and a thin writable layer.

### Container Lifecycle Commands

```bash
docker run --name web -d -p 80:80 nginx:alpine  # create + start
docker stop web
docker start web
docker rm web        # remove (must be stopped)
```

### Executing Commands Inside a Container

```bash
docker exec -it web bash        # open interactive shell
docker logs -f web              # follow logs
docker attach web               # attach STDIN/OUT/ERR
```

### Container vs Image
Images are **static templates**; containers are **runtime processes** based on those templates.

---

## 5. Docker Volumes & Mounts

### What is a Volume?
A Docker-managed directory on the host designed for **persistent, independent data**.

### Types of Storage
| Type | Created by | Typical Use | Example |
|------|------------|-------------|---------|
| Anonymous Volume | Docker | Ephemeral data | `docker run -v /var/lib/mysql mysql` |
| Named Volume | User | Persisted across runs | `docker volume create pgdata` |
| Bind Mount | User | Tight coupling to host path | `docker run -v "$PWD":/app node` |

#### Choosing the right mount type

* **Anonymous volumes** are ideal for quick experiments or disposable data; Docker cleans them with `docker container prune`.
* **Named volumes** should be the default for durable data (e.g., Postgres clusters). They survive container re-creates and are host-agnostic.
* **Bind mounts** map an exact host path into the container—great for live-coding during development, but they tie the image to host paths and permissions.

#### Other volume flavours

| Mode | Flag | When to use |
|------|------|-------------|
| Read-only | `:ro` suffix | Share configuration files safely |
| tmpfs | `--tmpfs` | Keep secrets or high-churn files in RAM only |
| SELinux relabel | `:z` / `:Z` | Fix permission issues on SELinux-enforcing hosts |

#### Backing up & restoring volumes

```bash
# Backup named volume to current directory
docker run --rm -v pgdata:/volume -v "$PWD":/backup alpine \
  tar czf /backup/pgdata-$(date +%F).tgz -C /volume .

# Restore the archive into the volume
docker run --rm -v pgdata:/volume -v "$PWD":/backup alpine \
  tar xzf pgdata-2024-06-25.tgz -C /volume
```

### Creating & Using Volumes

```bash
docker volume create pgdata
docker run -d -v pgdata:/var/lib/postgresql/data postgres:15
```

### Inspecting Volumes
```bash
docker volume ls
docker volume inspect pgdata
docker volume rm pgdata
```

### Volume Drivers
Plugins that redirect volume storage to NFS, S3, Azure Files, etc.

---

## 6. Networking in Docker

### Built-in Network Drivers
| Driver | Scope | Description |
|--------|-------|-------------|
| `bridge` | Per-host | Default for containers without `--network` |
| `host` | Per-host | Shares host network namespace |
| `none` | Per-host | Isolated, no networking |
| `overlay` | Swarm | Multi-host virtual network |
| `macvlan` | Per-host | Assigns MAC address on physical network |

### Common Commands
```bash
docker network create --driver bridge mynet
docker run -d --network=mynet --name db postgres

docker network inspect mynet
```

### Port Publishing vs Exposing
* `EXPOSE 8080` in Dockerfile **documents** a port.
* `-p 8080:80` on `docker run` **publishes** a port to the host.

---

## 7. Docker Compose

### What is Docker Compose?
A declarative tool for defining and running **multi-container** applications via a `docker-compose.yml` (v2.x schema).

Example `docker-compose.yml`:
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/app
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

Lifecycle:
```bash
docker compose up -d --build
docker compose logs -f
docker compose down --volumes
```

Environment overrides: `docker-compose.override.yml`, `--profile`, or `--env-file`.

### Multi-Container Builds with Buildx & Compose
Compose v2 + BuildKit's **buildx** let you build images for **every service in your compose file** with dependency awareness, caching, and multi-platform support.

```bash
# Enable BuildKit (Docker 23+)
export DOCKER_BUILDKIT=1

# Build all services defined in compose.yml
# The --builder flag lets you pick a multi-platform builder instance
# The --push flag pushes to the registry directly after build

docker buildx bake --file docker-compose.yml \
  --builder default --push
```

Tips:
* Define a `target` per service (`target: prod`) and build only what you need: `docker buildx bake --set *.target=prod`.
* Use BuildKit's `--mount=type=cache` to persist `npm` / `pip` caches between builds:
  ```dockerfile
  RUN --mount=type=cache,target=/root/.cache/pip pip install -r requirements.txt
  ```
* Cross-compile images in one shot: `docker buildx bake --platform linux/amd64,linux/arm64`.

### Advanced Docker Compose Patterns

| Pattern | Why it matters | Snippet |
|---------|---------------|---------|
| **Healthchecks** | Wait for DB to be ready before starting the app | `healthcheck: { test: ["CMD", "pg_isready", "-U", "postgres"], interval: 5s }` |
| **depends_on + condition** | Declarative start-order based on health | `depends_on: db: { condition: service_healthy }` |
| **Profiles** | Enable optional services (e.g., monitoring) | `profiles: ["observability"]` |
| **Resource limits** | Prevent noisy neighbours | `deploy: resources: limits: { cpus: '0.50', memory: 256M }` |
| **Named volumes + init containers** | Seed databases | `init: true` in Compose 2.20+ |
| **Extension fields (`x-`)** | Re-use common configuration | `x-common-env: &env
  environment:
    - TZ=UTC` |

Best practice: keep `docker-compose.override.yml` for dev-only tweaks (bind-mount source, expose debuggers) and commit a production-ready `compose.yml` that relies solely on images.

---

## 8. Docker Registry & Repositories

### Docker Hub & Private Registries
* **Docker Hub** – Public registry at `hub.docker.com`.
* **GHCR**, **ECR**, **GCR** – Vendor-specific private registries.

Login & Push:
```bash
docker login ghcr.io -u <user>
docker tag myapp:1.0 ghcr.io/<user>/myapp:1.0
docker push ghcr.io/<user>/myapp:1.0
```

### Docker Content Trust (Notary)
Enable signed pulls/pushes:
```bash
export DOCKER_CONTENT_TRUST=1
docker pull alpine:3.19
```

---

## 9. Docker Security

### Image Scanning
* `docker scan <image>` (powered by Snyk)
* Third-party: Trivy, Clair, Grype.

### Least-Privilege Containers
* Run as non-root: `USER 1001` in Dockerfile.
* Drop capabilities: `docker run --cap-drop all ...`.
* Read-only root FS: `--read-only`.

### Kernel Security Modules
* **Seccomp** – Default profile blocks dangerous syscalls.
* **AppArmor / SELinux** – MAC policies for containers.

### Secrets Management
* `docker secret` (Swarm), or mount files via `--secret` in Compose v3.9+.

---

## 10. Docker Swarm (Optional)

### Swarm vs Kubernetes
Swarm offers a simple, Docker-native clustering solution; fewer features than Kubernetes but faster to learn.

### Getting Started
```bash
docker swarm init               # manager node
docker swarm join --token <…>   # worker node
```

Deploy a service:
```bash
docker service create --name web --replicas 3 -p 80:80 nginx:alpine
docker service ls
docker service scale web=5
```

---

## 11. Docker in CI/CD

### Docker-in-Docker (dind)
Useful for GitLab CI or GitHub Actions when the runner itself needs to build images.

```yaml
# .gitlab-ci.yml
build:
  image: docker:25.0
  services:
    - docker:25.0-dind
  script:
    - docker build -t myapp:$CI_COMMIT_SHA .
    - docker push registry.example.com/myapp:$CI_COMMIT_SHA
```

### Caching Layers
* Use `--cache-from` or GitHub Action `docker/build-push-action@v5` with `cache-from`/`cache-to` options.

---

## 12. Troubleshooting & Debugging

| Command | Purpose |
|---------|---------|
| `docker logs <c>` | View STDOUT/ERR of a container |
| `docker inspect <c or img>` | Low-level JSON metadata |
| `docker top <c>` | Processes inside container |
| `docker events` | Real-time daemon events |
| `docker system df` | Disk usage by images/volumes/containers |

Cleanup helpers:
```bash
docker container prune -f
docker image prune -af
docker volume prune -f
docker system prune -af --volumes  # nuke everything
```

---

## 13. Docker Best Practices

1. **Minimize Image Size** – Smaller attack surface & faster pulls.
2. **Leverage Layer Caching** – Order Dockerfile steps from least to most frequently changing.
3. **Keep Secrets Out of Images** – Use runtime env vars or secrets APIs.
4. **Graceful Shutdown** – Trap `SIGTERM` and forward to child processes.
5. **Avoid `latest` in Production** – Pin versions for reproducibility.
6. **One Process per Container** – Simpler resource & failure isolation.

#### Deep-Dive: Build & Image Optimisation

* **Choose the right base image**: start from `alpine` or Google's *distroless* when you only need a glibc-less runtime; fall back to `debian-slim` when you require system libraries.
* **Multi-stage builds**: compile/bundle assets in a heavy stage and `COPY --from` the final artefacts into a minimal stage. Example for Node:
  ```dockerfile
  FROM node:22-bullseye AS build
  WORKDIR /app
  COPY package*.json ./
  RUN npm ci --production=false --cache /tmp/npm
  COPY . .
  RUN npm run build

  FROM node:22-alpine AS runtime
  WORKDIR /app
  ENV NODE_ENV=production
  COPY --from=build /app/dist /app/dist
  CMD ["node", "dist/index.js"]
  ```
* **Enable BuildKit**: gives parallel build graph, automatic mount caches, and secret/ssh forwarding. Add `# syntax=docker/dockerfile:1.7` and export `DOCKER_BUILDKIT=1`.
* **Inline caches**: push cache metadata to the registry so CI can pull it: `docker buildx build --build-arg BUILDKIT_INLINE_CACHE=1 --cache-to=type=registry,...`.
* **Filesystem trims**: Delete package managers' cache (`apt-get clean`, `npm ci --production --ignore-scripts`) and remove docs/locales using `rm -rf /var/lib/apt/lists/* /usr/share/doc`.
* **Consolidate RUN steps**: combine related commands with `&&` to avoid extra layers, but **keep readability**.
* **Use `.dockerignore` aggressively**: exclude tests, docs, and local build artefacts to cut context size.
* **Leverage `COPY --link` (Docker 25+)**: avoids an extra copy for unchanged files and speeds up incremental rebuilds.
* **Target-specific images**: build separate images for dev, test, and prod rather than a one-size-fits-all blob.

---

### Further Reading
* Official docs: <https://docs.docker.com/>
* "Docker Deep Dive" by Nigel Poulton.
* "The Pragmatic Programmer's Guide to Docker" (ebook).

--- 
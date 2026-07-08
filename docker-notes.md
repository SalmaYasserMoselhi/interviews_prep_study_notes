# Docker Learning Notes
> Combined revision document — Phases 0 through 9

---

## Table of Contents

**Part 1 — Concepts**
- [Phase 0 — The Pre-Docker Era](#phase-0--the-pre-docker-era)
- [Phase 1 — Core Concepts](#phase-1--core-concepts)
- [Phase 2 — Dockerfile](#phase-2--dockerfile)
- [Phase 3 — Docker Commands](#phase-3--docker-commands)
- [Phase 4 — Volumes & Bind Mounts](#phase-4--volumes--bind-mounts)
- [Phase 5 — Networking](#phase-5--networking)
- [Phase 6 — Docker Compose](#phase-6--docker-compose)
- [Phase 8 — Real World Docker](#phase-8--real-world-docker)
- [Phase 9 — Bridge to Orchestration](#phase-9--bridge-to-orchestration)

**Part 2 — Hands-On Tutorial**
- [Project Scenario](#the-project-were-building)
- [Phase 1 — Project Architecture & Requirements](#phase-1--understanding-the-project-architecture-and-requirements)
- [Phase 2 — What to Containerize](#phase-2--deciding-what-should-and-shouldnt-be-containerized)
- [Phase 3 — Preparing Repositories](#phase-3--preparing-each-repository-for-docker)
- [Phase 4 — Backend Dockerfile](#phase-4--writing-the-backend-dockerfile-from-scratch)
- [Phase 5 — Frontend Dockerfile](#phase-5--writing-the-frontend-dockerfile-from-scratch)
- [Phase 6 — .dockerignore](#phase-6--understanding-and-writing-dockerignore)
- [Phase 7 — Building & Testing Images](#phase-7--building-and-testing-docker-images)
- [Phase 8 — Running Containers Individually](#phase-8--running-containers-individually)
- [Phase 9 — Docker Networking](#phase-9--connecting-containers-with-docker-networking)
- [Phase 10 — Environment Variables](#phase-10--managing-environment-variables-for-development-and-production)
- [Phase 11 — Volumes](#phase-11--using-volumes-when-to-and-when-not-to)
- [Phase 12 — docker-compose.yml (Dev)](#phase-12--writing-a-complete-docker-composeyml-for-local-development)
- [Phase 13 — Running the Full App](#phase-13--running-the-complete-application-with-docker-compose)
- [Phase 14 — Tests Inside Containers](#phase-14--running-automated-tests-inside-containers)
- [Phase 15 — Debugging & Troubleshooting](#phase-15--common-debugging-techniques-and-troubleshooting)
- [Phase 16 — Optimizing the Dev Workflow](#phase-16--optimizing-the-development-workflow)
- [Phase 17 — Production-Ready Images](#phase-17--preparing-production-ready-docker-images)
- [Phase 18 — Production Docker Compose](#phase-18--writing-a-production-docker-compose)
- [Phase 19 — Complete CI/CD Pipeline](#phase-19--building-a-complete-cicd-pipeline)
- [Phase 20 — Deploying to AWS](#phase-20--deploying-the-application-to-aws)
- [Phase 21 — Verifying the Deployment](#phase-21--verifying-the-deployment)
- [Phase 22 — Monitoring & Maintenance](#phase-22--monitoring-updating-and-maintaining-the-deployed-application)

---

## Phase 0 — The Pre-Docker Era

### 📖 The Story
Companies bought physical servers — one server per app. Hardware got incredibly powerful (64 cores, 512GB RAM) but was running at 5% capacity. A brilliant engineer asked: *"What if one machine could pretend to be many?"* That question gave birth to VMs.

VMs solved hardware waste. But engineers soon noticed a new problem — the **VM tax**. Every VM carried a full OS (GBs of RAM) just to run a 50MB app. Licensing costs piled up. VM templates were huge and painful to share. "Works on my machine" still existed between VMs.

Then someone asked: *"Do we actually need a full OS per app? Or only the parts the app uses?"* That question gave birth to containers. Linux already had the tools — cgroups and namespaces. Docker in 2013 packaged them into something any developer could use.

### Physical Machines
- One server = one app = one OS
- Hardware was underutilized (often < 10% usage)
- Need a new app? Buy a new server → weeks + thousands of dollars
- Security risk: apps sharing a server share risk
- "Works on my machine" problem was already alive here

### Virtual Machines (VMs)
- Appeared to solve hardware waste
- One physical machine → multiple isolated OSes
- Key question that started it: *"What if one machine could pretend to be many?"*

#### How VMs Work

```
Physical Hardware (64 cores, 256GB RAM)
          │
  HYPERVISOR LAYER (VMware / Hyper-V / KVM / VirtualBox)
  "I divide the hardware and lie to each VM"
          │
  ┌───────┴──────────┬──────────────────┐
  │                  │                  │
[VM 1: Ubuntu]  [VM 2: Windows]  [VM 3: CentOS]
 Web App          .NET API          Database
```

- **VM = a complete fake computer inside a real computer**
- Each VM has its own: virtual CPU, virtual RAM, virtual disk, full OS
- The VM has no idea it is sharing physical hardware
- **VMs provided hardware isolation → containers provide app isolation**

#### Two Types of Hypervisor

| Type 1 (Bare Metal) | Type 2 (Hosted) |
|---|---|
| Runs directly on hardware | Runs on top of an OS |
| VMware ESXi, AWS Nitro, Hyper-V | VirtualBox, VMware Workstation |
| Fast, used in production | Slower, used by developers locally |

> When you use AWS EC2 or DigitalOcean — you are using a VM.

#### Dev / Ops Split (VM Era)
- **Dev** → builds the VM (OS, dependencies, app)
- **Ops** → administrator (start, backup, stop, monitor)
- Problem: "works on my machine" vs "breaks in production" → **The Wall of Confusion**

---

### The VM Tax — Why VMs Were Not Enough

| Pain | Description |
|---|---|
| Resource Tax | Every VM carries a full OS. Before your app runs, the OS consumed GBs of RAM. |
| Licensing Cost | Windows Server ~$1,000+. Red Hat ~$800/year. 50 VMs = $50,000+ in licenses alone. |
| Template Headache | VM templates are huge, hard to share, need a DevOps expert to build. |
| Still not portable | A VM template built on VMware doesn't run on AWS or a teammate's laptop. |

**The key insight:**

> *"Do we actually need a full OS for every app? Or only the parts the app actually uses?"*

```
VM  = carries a FULL HOUSE to run one application
Container = carries only the ROOM the application needs
```

---

### OS Layers — The Key to Understanding Containers

```
┌──────────────────────────┐
│       USER MODE          │  → GUI, apps, utilities → humans use this
│  Containers DON'T need   │
├──────────────────────────┤
│      KERNEL MODE         │  → CPU, memory, filesystem, networking
│  Containers SHARE this   │  → programs use this
└──────────────────────────┘
```

- **VMs virtualize hardware**
- **Containers virtualize the OS user space**
- Containers share the host kernel → that's why they are so much lighter

---

### Linux Kernel Features Behind Containers

Both existed in Linux before Docker. Docker just packaged them.

**cgroups (Control Groups) — 2006, by Google**
- Purpose: resource **limiting** and **accounting**
- "Container A: max 2 CPU cores and 512MB RAM"
- Think: a **budget system** for processes

**namespaces — 2002–2013**
- Purpose: **isolation** and **visibility** control
- Each container gets its own view of: processes, filesystem, network, hostname, users
- Container A does not know Container B exists
- Think: **blinders** for processes

---

### Evolution Timeline

```
Physical Machines (1 server = 1 app, wasteful)
    ↓
Virtual Machines (1 machine, many OSes via hypervisor)
    ↓
cgroups + namespaces added to Linux kernel (2006–2013)
    ↓
LXC — Linux Containers (2008, powerful but hard to use)
    ↓
Docker (2013 — made containers accessible to every developer)
    ↓
Docker Compose (2014)
    ↓
Kubernetes (2014, open-sourced by Google)
```

---

### Common Mistakes — Phase 0

- ❌ Thinking VMs solved all problems → they created the VM tax
- ❌ Confusing the hypervisor with the OS → hypervisor *manages* OSes, it's not one
- ❌ Thinking containers replaced VMs → in production, containers often **run inside VMs**

### 📝 What Should I Remember
- Physical servers wasted hardware → VMs fixed that → VMs created the VM tax → containers fixed that
- A VM is a complete fake computer inside a real computer — has its own OS, virtual CPU, virtual RAM
- The hypervisor is the software layer that lies to each VM and divides the physical hardware
- Containers share the **host kernel** — no full OS copy → lightweight, fast, cheap
- Two Linux features make containers possible: **cgroups** (resource limits) and **namespaces** (isolation) — Docker did not invent these, it packaged them

---

## Phase 1 — Core Concepts

### 📖 The Story
Docker launched in 2013. Containers were powerful. But developers needed vocabulary before they could use them. What is an image exactly? What is a container? How do they relate? Where do images come from?

The core confusion that trips every beginner: an image and a container look similar but are completely different things. One is static (on disk), one is dynamic (running). One is a blueprint, one is a running instance. Getting this wrong leads to hours of confusion.

Also missing from the picture: how does a Docker command actually reach a running container? The answer is the architecture — client, daemon, registry — three pieces that work together on every single `docker` command you type.

### "Works on My Machine" — The Developer Problem

```
Developer Laptop    CI Server       Production
────────────────    ──────────      ────────────
Python 3.11         Python 3.9 ❌   Python 3.10 ❌
Ubuntu 22.04        CentOS 7        Ubuntu 20.04
Works ✅            Breaks ❌       Breaks ❌
```

- Nobody changed the code. The **environment** is different.
- Docker's answer: the environment travels **with** the application

```
Without Docker:  Code travels. Environment stays behind.
With Docker:     Code AND environment travel together.
```

---

### Docker Architecture

Every `docker` command you type goes through this:

```
DOCKER CLIENT           (what you interact with)
docker build / run / pull...
        │
        │  sends commands via REST API
        ▼
DOCKER DAEMON  (dockerd — the engine doing all the work)
• builds images
• runs containers
• manages volumes, networks
        │
        │  pulls from / pushes to
        ▼
DOCKER REGISTRY         (where images live)
Docker Hub (public) / GHCR / ECR (private)
```

- **Client** → just a CLI. Sends instructions, does nothing itself.
- **Daemon** → background service. Does everything.
- **Registry** → storage for images. Docker Hub is the default.

---

### Image vs Container

| Image | Container |
|---|---|
| Static, on disk | Dynamic, in memory |
| Read-only, layered | Has one writable layer on top |
| Like a class definition | Like a running object instance |
| Like a git repository | Like a working directory |
| Never changes | Can write data (but only temporarily) |

**The correct flow:**
```
Dockerfile → [docker build] → Image → [docker run] → Container
```

- Dockerfile = recipe on paper
- Image = sealed finished dish
- Container = dish being consumed right now

> A container **never** produces an image. An image produces containers.

---

### Image Layers

```
Layer 4: Your App Code        ← you add this
Layer 3: App Dependencies     ← you add this
Layer 2: Runtime (Node/Python/etc)
Layer 1: Base OS Libraries (Ubuntu minimal / Alpine)
─────────────────────────────
READ-ONLY IMAGE

When container runs:
┌─────────────────────────────┐
│  Writable Container Layer   │  ← only this can change
├─────────────────────────────┤
│  All image layers below     │  ← read-only, shared
└─────────────────────────────┘
```

- Stop/remove container → writable layer gone
- Image layers → untouched, reusable
- 10 containers from same image → 10 separate writable layers, **one** image on disk

---

### Container Lifecycle

```
IMAGE ──docker run──► RUNNING ──docker stop──► STOPPED
                                                    │
                                               docker start
                                                    │
                                                RUNNING
                                                    │
                                               docker rm
                                                    │
                                               DELETED ← gone forever
```

- **Stopped ≠ Deleted**
- Stopped container still exists, can restart
- `docker ps` → running only | `docker ps -a` → all including stopped
- Stopped containers accumulate silently and waste disk

---

### Docker on Windows and Mac

> Linux containers need a Linux kernel.

```
Windows/Mac
    │
Docker Desktop creates a hidden lightweight Linux VM
    │
Linux Kernel (inside the VM)
    │
Your Containers
```

- You don't see this VM — Docker Desktop manages it silently
- 99% of real-world usage uses Linux containers, even on Windows/Mac

---

### Docker Hub

- Docker Hub = GitHub but for Docker images
- `docker pull nginx` → goes to Docker Hub automatically
- **Official images**: no username prefix → `nginx`, `postgres`, `node`
- **Community images**: have username prefix → `bitnami/nginx`, `johndoe/myapp`

**Image Tags:**
```
nginx:latest        → most recent (not always stable)
nginx:1.25          → specific version
nginx:1.25-alpine   → specific version on Alpine Linux
```

> ⚠️ Never use `latest` in production. Pin to a specific version.

---

### Container Image = VM Template

| VM Template | Container Image |
|---|---|
| Pre-built snapshot | Pre-built snapshot |
| Huge size (GBs) | Lightweight (MBs–~1GB) |
| Hard to share | Push/pull via registry |
| Needs DevOps expert to create | Any developer writes a Dockerfile |
| Stored on company file server | Stored on Docker Hub |

### 📝 What Should I Remember
- **Image** = static, read-only, on disk, a blueprint. **Container** = dynamic, running, in memory, an instance
- The correct flow: `Dockerfile → [docker build] → Image → [docker run] → Container` — a container never produces an image
- Images are made of **layers** — each layer is read-only and cached. Containers add one **writable layer** on top that dies when the container is removed
- Docker architecture: **Client** (sends commands) → **Daemon** (does the work) → **Registry** (stores images)
- One image can run as **many containers simultaneously** — the image is never consumed or changed

---

## Phase 2 — Dockerfile

### 📖 The Story
You understand what an image is. But where do images come from? Who builds them? How?

Imagine you built a NestJS API. It works on your machine. You want your teammate to run it. They need to: install the right Node version, install dependencies, set environment variables, run the right start command. If they get one step wrong — nothing works. Now multiply this across 5 developers, a staging server, a production server, and a CI/CD pipeline. That is 9 environments where a human must follow the same steps perfectly.

Docker engineers needed a format that was simple to write, precise to follow, versionable in git, and reproducible every time. Their answer: a plain text file. They called it the Dockerfile. The setup document becomes **executable code**.

### What Is a Dockerfile?

- A plain text file with **ordered instructions**
- Docker follows them top to bottom to build an image
- Lives in your repo next to your code (in git)
- **Your environment as code**

```
Dockerfile → [docker build] → Image → [docker run] → Container
```

---

### Core Instructions

#### `FROM` — Starting Point
```dockerfile
FROM node:20-alpine
```
- Every image starts from another image — nobody starts from absolute zero
- `node` = software, `20` = version, `alpine` = OS variant (tiny, ~5MB)

```
FROM node:20           → ~350MB base image
FROM node:20-slim      → ~200MB base image
FROM node:20-alpine    → ~50MB base image
```

#### `WORKDIR` — Working Directory Inside Image
```dockerfile
WORKDIR /app
```
- Like `cd /app` but permanent
- All following instructions happen relative to `/app`
- Docker creates the folder if it doesn't exist

#### `COPY` — Bring Code Inside
```dockerfile
COPY package*.json ./   # copy dependency file first
COPY . .                # copy all code after
```
- `COPY [source on your machine] [destination inside image]`
- Two separate COPY instructions = **cache optimization** (explained below)

#### `RUN` — Execute During Build
```dockerfile
RUN npm install
RUN npm run build
```
- Runs during `docker build`, result is frozen into the layer
- Use for: installing dependencies, compiling code, running build scripts
- **NOT** for starting the application

Chain commands to reduce layers:
```dockerfile
# ❌ Two layers
RUN apt-get update
RUN apt-get install -y curl

# ✅ One layer
RUN apt-get update && apt-get install -y curl
```

#### `EXPOSE` — Document the Port
```dockerfile
EXPOSE 3000
```
- Documents which port the app uses
- Does **NOT** open the port or make it accessible
- Actual port mapping happens with `-p` at runtime

**Port concept:**
```
Your laptop (port 8080) ←──bridge──► Container (port 3000)
                    docker run -p 8080:3000
```
- Left = YOUR machine, Right = INSIDE the container
- Memory trick: `-p ME:CONTAINER`

#### `CMD` — What Runs When Container Starts
```dockerfile
CMD ["node", "dist/main.js"]
```
- Runs when `docker run` is called
- Use **exec form** (array) — more precise, handles signals correctly
- Only one CMD per Dockerfile — last one wins

```
docker build → image created (all RUN commands happened here)
docker run   → CMD executes → your app is running
```

#### `ENV` — Environment Variables at Runtime
```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```
- Available inside the running container
- ⚠️ Never hardcode secrets in ENV — baked into image permanently

#### `ARG` — Build-Time Variables Only
```dockerfile
ARG NODE_VERSION=20
FROM node:${NODE_VERSION}-alpine
```
- Available only during `docker build`, not at runtime
- Different from `ENV` which persists into the container

```
ARG → docker build only
ENV → build AND runtime inside container
```

---

### Build Caching — Most Important Practical Concept

- Docker caches each layer
- If a layer's **input hasn't changed**, Docker reuses the cache
- If a layer changes → **all layers below it are invalidated**

```
FROM node:20-alpine          → cached ✅
WORKDIR /app                 → cached ✅
COPY package*.json ./        → cached ✅ (packages unchanged)
RUN npm install              → cached ✅
COPY . .                     → INVALIDATED ❌ (code changed)
RUN npm run build            → must re-run ❌
```

**Golden Rule:**
```
Put instructions that change RARELY → at the TOP
Put instructions that change OFTEN  → at the BOTTOM
```

```dockerfile
# ✅ SMART ORDER
COPY package*.json ./     # rarely changes
RUN npm install           # only re-runs when packages change
COPY . .                  # changes every time you write code
RUN npm run build

# ❌ NAIVE ORDER
COPY . .                  # changes every time
RUN npm install           # re-runs on EVERY build unnecessarily
```

---

### `docker build` Command

```bash
docker build -t myapp:1.0 .
```

```
docker build   -t myapp:1.0   .
      │              │         │
      │              │         └── build context (current directory)
      │              └── tag: name:version for the image
      └── build command
```

**What happens internally:**
1. Docker client sends current directory to the daemon
2. Daemon reads Dockerfile line by line
3. For each instruction: check cache → reuse or execute
4. Final image tagged and stored locally

**Useful flags:**
```bash
docker build --no-cache -t myapp:1.0 .      # force full rebuild
docker build -f Dockerfile.prod -t myapp .  # different Dockerfile name
docker build --progress=plain -t myapp .    # verbose output
```

---

### What a Dockerfile Is NOT

- ❌ Not a shell script — runs in a controlled build environment
- ❌ `RUN` ≠ `CMD` → RUN is build time, CMD is runtime
- ❌ `EXPOSE` does not open a port — only documents intent
- ❌ Not tied to one language — works for Node, Python, Ruby, PHP, Go...

---

### Complete Dockerfile Example (Django)

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Tell Python not to write .pyc files
ENV PYTHONDONTWRITEBYTECODE=1
# Don't buffer logs — appear immediately in docker logs
ENV PYTHONUNBUFFERED=1

# Copy dependency file first (cache optimization)
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Copy app code after (this changes more often)
COPY . .

EXPOSE 8000

# 0.0.0.0 = accept connections from anywhere (needed outside container)
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

> ⚠️ Use `0.0.0.0` not `127.0.0.1` — `127.0.0.1` inside a container means "only the container itself". Nothing outside can reach it.

---

### Common Dockerfile Mistakes

```dockerfile
# ❌ Wrong COPY order
COPY . .
RUN pip install -r requirements.txt   # re-runs on every code change

# ✅ Right COPY order
COPY requirements.txt ./
RUN pip install -r requirements.txt   # cached unless requirements change
COPY . .
```

```dockerfile
# ❌ CMD for build steps
CMD ["npm", "install"]

# ✅ RUN for build, CMD for start
RUN npm install
CMD ["node", "dist/main.js"]
```

```dockerfile
# ❌ Secrets in Dockerfile
ENV DATABASE_PASSWORD=supersecret   # baked into image forever

# ✅ Inject at runtime, not build time
# (via docker run -e or Docker Compose)
```

---

### Practice — Pushing to Docker Hub

The full flow from building to sharing an image:

```bash
docker build -t myapp:1.0 .         # build the image
docker login                         # authenticate with Docker Hub

# Docker Hub requires your username in the tag
# because it's a shared public space — your username is your storage room
docker tag myapp:1.0 yourusername/myapp:1.0

docker images                        # both tags point to the same image layers
docker push yourusername/myapp:1.0  # upload to Docker Hub

docker rmi yourusername/myapp:1.0   # remove local copy to test
docker pull yourusername/myapp:1.0  # pull back from Hub
docker run -p 8080:3000 yourusername/myapp:1.0
```

> The username in the tag is required. Without it, Docker does not know which account on Docker Hub to push to.

**Ruby Dockerfile example (from practice):**
```dockerfile
FROM ruby:3.2-bookworm

WORKDIR /app

COPY Gemfile* ./
RUN bundle install

COPY . .

EXPOSE 3000

# -b 0.0.0.0 binds to all interfaces so the port is reachable outside the container
CMD ["bundle", "exec", "rails", "s", "-b", "0.0.0.0"]
```

> ⚠️ Note: deleting an image while a container uses it fails. You must `docker rm` the container first, then `docker rmi` the image — just like you cannot delete a class definition while an instance of it is still running.

### 📝 What Should I Remember
- A Dockerfile is a plain text file with **ordered instructions** — Docker follows them top to bottom to build an image. It is your environment as code
- **Order matters for caching** — put rarely-changing steps at the top (dependency files, `RUN install`), frequently-changing steps at the bottom (your code)
- `RUN` = build time (frozen into the image). `CMD` = runtime (when the container starts). Mixing these is the most common mistake
- `EXPOSE` only **documents** a port — it does not open it. Real port mapping happens with `docker run -p`
- Always use `0.0.0.0` as bind address in CMD — `127.0.0.1` inside a container means "only the container itself"

---

## Phase 3 — Docker Commands

### 📖 The Story
You know what an image is and how to build one. But an image sitting on disk does nothing. You need to run it, manage it, inspect it, stop it, clean it up.

Think of your daily git workflow. You do not just know what git is conceptually — you have commands you reach for constantly: `git add`, `git commit`, `git push`, `git log`. Docker is the same. There are 15 commands you will use every day and 30 more for edge cases.

The goal of this phase is not memorization. It is understanding what each command does internally, why it exists, and which category it belongs to — image commands (static, on disk) or container commands (dynamic, running). Confusing the two categories causes real pain.

### Two Categories — Never Mix Them

```
IMAGES (static, on disk)        CONTAINERS (dynamic, running)
────────────────────────        ──────────────────────────────
docker pull                     docker run
docker images                   docker ps / ps -a
docker rmi         ←──────────► docker rm
docker tag                      docker stop / start
docker push                     docker exec
docker image prune              docker logs
                                docker inspect
```

> `docker rmi` = remove IMAGE | `docker rm` = remove CONTAINER

---

### Image Commands

```bash
docker pull nginx                    # download image from Docker Hub
docker pull nginx:1.25-alpine        # specific version

docker images                        # list all local images

docker rmi nginx                     # remove image (no container must use it)
docker rmi nginx:1.25-alpine         # remove specific tag

docker tag myapp:1.0 myusername/myapp:1.0   # rename/version image

docker push myusername/myapp:1.0     # upload to Docker Hub (login first)
docker login                         # log into Docker Hub

docker image prune                   # remove dangling images
docker image prune -a                # remove ALL unused images
```

**Rule:** Cannot remove an image if a container is using it (even stopped).
```
# Correct order:
docker rm <container>    # first
docker rmi <image>       # then
```

---

### `docker run` — Most Important Command

```bash
docker run -d -p 8080:80 --name my-nginx nginx:latest
```

```
docker run  = docker create + docker start

-d              → detached (background), terminal stays free
-p 8080:80      → port mapping: HOST:CONTAINER (left=you, right=container)
--name my-nginx → readable name (without this, Docker picks a random name)
-e KEY=VALUE    → set environment variable
-v vol:/path    → mount volume or bind mount
--rm            → auto-delete container when it stops
-it             → interactive terminal (keep stdin open + allocate terminal)
--network name  → connect to a specific Docker network
```

**Without `-d`:** container runs in terminal, locks it, Ctrl+C to stop.  
**With `-d`:** runs in background, terminal is free.

---

### Port Mapping

```
Your laptop port 8080 ──bridge──► Container port 3000
                  docker run -p 8080:3000

-p  LEFT : RIGHT
    ────   ──────
    YOUR   CONTAINER
  machine   (what the app listens on inside)
```

> Memory trick: `-p ME:CONTAINER`

---

### `-it` Flag — Get Inside a Container

```bash
docker run -it ubuntu bash
```
- Opens an interactive shell **inside** the container
- `-i` = keep stdin open | `-t` = allocate a terminal
- `exit` to leave (container stops)

---

### Container Commands

```bash
docker ps                            # running containers
docker ps -a                         # all containers (including stopped)

docker stop my-nginx                 # graceful stop (sends SIGTERM, waits 10s)
docker kill my-nginx                 # immediate kill (SIGKILL) — last resort
docker start my-nginx                # restart a stopped container

docker rm my-nginx                   # remove stopped container
docker rm -f my-nginx                # force remove running container
docker rm $(docker ps -aq)           # remove ALL stopped containers

docker exec -it my-nginx bash        # shell inside RUNNING container
docker exec my-app npm run migrate   # run command inside running container

docker logs my-nginx                 # see container output
docker logs -f my-nginx              # follow logs in real time
docker logs --tail 50 my-nginx       # last 50 lines

docker inspect my-nginx              # full JSON details of container or image
```

**`docker exec` vs `docker run -it`:**
```
docker run -it image bash     → creates NEW container, dies on exit
docker exec -it container bash → enters RUNNING container, keeps running on exit
```

---

### `docker ps` Output Explained

```
CONTAINER ID  → first 12 chars of full ID
IMAGE         → which image this came from
STATUS        → Up 5 minutes / Exited (0) / Exited (1) / Restarting
PORTS         → 0.0.0.0:8080->80/tcp = your port 8080 → container port 80
NAMES         → container name
```

**Exit codes:** `Exited (0)` = clean exit | `Exited (1)` = something went wrong

---

### `docker stop` vs `docker rm`

```
docker stop  → pauses container (can come back with docker start)
docker rm    → destroys container (cannot come back, starts fresh from image)

Stopped ≠ Deleted
```

> Stopped containers accumulate silently. Run `docker container prune` periodically.

---

### Logging to stdout

- `docker logs` captures **stdout and stderr** only
- If your app logs to a file → `docker logs` shows nothing
- Always log to stdout in containerized apps

---

### Practical Workflow — PostgreSQL

```bash
docker pull postgres:16-alpine
docker run -d --name my-postgres \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp_db \
  -p 5432:5432 \
  postgres:16-alpine

docker ps
docker logs my-postgres
docker exec -it my-postgres psql -U admin -d myapp_db

docker stop my-postgres       # done for the day
docker start my-postgres      # next day
docker stop my-postgres && docker rm my-postgres  # done with project
```

---

### Full Command Map

```
REGISTRY (Docker Hub)
        │ docker pull / push
        ▼
LOCAL IMAGE STORE
docker images / rmi / tag / build / image prune
        │ docker run
        ▼
CONTAINERS
docker ps / stop / start / rm / exec / logs / inspect
```

### Other Useful Commands

```bash
docker info           # full info about your Docker installation and daemon state
docker system df      # how much disk Docker is using (images, containers, volumes)
docker system prune   # clean up stopped containers, unused networks, dangling images
```

### 📝 What Should I Remember
- Two categories that never mix: **image commands** (`docker rmi`, `docker images`, `docker pull`) and **container commands** (`docker rm`, `docker ps`, `docker stop`)
- `docker run` is the most important command — it creates AND starts a container. Key flags: `-d` (background), `-p HOST:CONTAINER` (left=you, right=container), `--name`, `-e`, `-it`
- **Stopped ≠ Deleted** — `docker stop` pauses (can come back). `docker rm` destroys permanently. Stopped containers accumulate silently
- `docker exec -it <name> bash` = get inside a running container to debug. This is your SSH equivalent
- `docker logs -f <name>` streams live output — your app must log to **stdout** for this to work

---

## Phase 4 — Volumes & Bind Mounts

### 📖 The Story
You have a running PostgreSQL container. You spend the afternoon building your database schema — tables, relationships, seed data. Everything looks perfect.

You stop the container. You run `docker rm my-postgres`. You start a new container from the same postgres image.

Everything is gone. Every table. Every row. Every migration. The container is running. The image is intact. But the database is completely empty. Back to zero.

This is not a bug. This is Docker working exactly as designed. Containers are ephemeral — disposable by design. The writable layer that held your data was part of the container. It died with it.

The solution: move data outside the container before it runs. Docker gives you two ways — Volumes (Docker manages the location) and Bind Mounts (you choose the location on your machine).

### The Problem — Why Data Dies

When a container runs, Docker adds a **writable layer** on top of the read-only image.

```
CONTAINER
┌──────────────────────────┐  ← Writable layer (created fresh on run)
│  database files          │    ALL writes go here
│  uploaded images         │
│  generated logs          │
├──────────────────────────┤
│  Image layers (read-only)│
└──────────────────────────┘

docker rm my-postgres
→ container deleted
→ writable layer deleted → ALL data gone
→ image untouched
→ next container starts empty
```

This is called **container ephemerality** — containers are designed to be disposable.

**Why ephemerality is a feature:**
- Containers are predictable (always start clean)
- Easy to scale and update
- No state drift
- Easy crash recovery

**The problem it creates:**
- Database files, uploads, logs disappear on removal
- You cannot lose your production database on every deploy

**Solution:** move data **outside** the container.

---

### Mental Model

```
Container = HOTEL ROOM
Guest = your application
Room = container's writable layer → cleaned when you check out

VOLUME = personal storage unit → stays when you leave the hotel
BIND MOUNT = your laptop from home → you bring it, it's always yours
```

---

### Type 1 — Docker Volumes

- Docker-managed storage, lives outside the container filesystem
- Docker decides the location, you choose the name
- **Volume lifecycle is independent of container lifecycle**

```
/var/lib/docker/volumes/
    ├── my-postgres-data/   ← your named volume
    │       └── _data/      ← actual data here
    └── my-redis-data/
```

**Volume commands:**
```bash
docker volume create my-postgres-data
docker volume ls
docker volume inspect my-postgres-data
docker volume rm my-postgres-data
docker volume prune          # ⚠️ PERMANENT — removes all unused volumes
```

**Using a volume:**
```bash
docker run -d \
  --name my-postgres \
  -v my-postgres-data:/var/lib/postgresql/data \
  postgres:16-alpine
```

```
-v  my-postgres-data  :  /var/lib/postgresql/data
    ────────────────     ─────────────────────────
    name of volume       path INSIDE the container
    (on your machine)    where postgres stores files
```

Docker creates the volume automatically if it doesn't exist.

**Common database data paths:**
```
PostgreSQL  → /var/lib/postgresql/data
MySQL       → /var/lib/mysql
MongoDB     → /data/db
Redis       → /data
```

---

### Type 2 — Bind Mounts

- You choose the exact folder on your machine
- Both sides stay in sync instantly — changes appear immediately in both directions
- Used for **live code reload in development**

```
YOUR MACHINE                  CONTAINER
/home/mo/projects/myapp  ──►  /app
src/main.ts     ═══════════►  src/main.ts (same file, live sync)
```

```bash
docker run -d -p 3000:3000 \
  -v /home/mo/projects/myapp:/app \
  myapp:dev

# Or use $(pwd) when already in the project directory
docker run -d -p 3000:3000 -v $(pwd):/app myapp:dev
```

```
-v  /absolute/path/on/machine  :  /app
    ─────────────────────────     ────
    must be ABSOLUTE path         inside container
```

**Development workflow with bind mount:**
```
Without bind mount:  change code → docker build → docker run → test → repeat
With bind mount:     change code → instantly reflected → test → repeat
```

---

### Type 3 — tmpfs Mounts

- Stored in host RAM only — never written to disk
- Gone when container stops
- Used for: sensitive temporary data, session tokens, decrypted secrets

```bash
docker run -d --tmpfs /tmp myapp
```

Rarely used in regular development. Know it exists.

---

### The `--mount` Flag (Modern Alternative to `-v`)

```bash
# Volume
docker run -d \
  --mount type=volume,source=my-postgres-data,target=/var/lib/postgresql/data \
  postgres:16-alpine

# Bind mount
docker run -d --mount type=bind,source=$(pwd),target=/app myapp:dev

# tmpfs
docker run -d --mount type=tmpfs,target=/tmp myapp
```

- `-v` → shorter, more common
- `--mount` → longer, more explicit, preferred in scripts
- Both work identically

---

### Storage Visualization

```
HOST MACHINE
┌──────────────────────────────────────────────────────────┐
│  /home/mo/projects/myapp  ← YOUR code directory          │
│  /var/lib/docker/volumes/ ← DOCKER manages this          │
│  RAM                      ← TMPFS lives here             │
│                                                          │
│  ┌──────────────────────────────────────────────────┐    │
│  │               CONTAINER                          │    │
│  │  /app      ◄──── BIND MOUNT ── /home/mo/myapp    │    │
│  │  /var/lib/ ◄──── VOLUME ────── docker volumes/   │    │
│  │  /tmp      ◄──── TMPFS ─────── RAM only          │    │
│  └──────────────────────────────────────────────────┘    │
│                                                          │
│  Container removed:                                      │
│  ✅ Volume data survives                                 │
│  ✅ Bind mount survives (it was your files all along)    │
│  ❌ Container writable layer destroyed                   │
│  ❌ tmpfs gone (was only in RAM)                         │
└──────────────────────────────────────────────────────────┘
```

---

### Volume vs Bind Mount Decision

```
Development?
  ├── Need live code changes?  → BIND MOUNT  (-v $(pwd):/app)
  └── Need persistent data?    → VOLUME      (-v my-db:/var/lib/...)

Production?
  └── Almost always            → VOLUME
```

---

### Common Mistakes — Volumes

```bash
# ❌ No volume = data dies
docker run -d --name my-db postgres:16

# ✅ Named volume = data survives
docker run -d --name my-db \
  -v my-db-data:/var/lib/postgresql/data postgres:16
```

```bash
# ❌ Relative path fails
docker run -v ./myapp:/app nginx

# ✅ Absolute path required
docker run -v $(pwd):/app nginx
```

```bash
# ❌ Mounts everything including host node_modules (macOS binaries)
docker run -v $(pwd):/app node-app

# ✅ Mount only source code, protect node_modules with a volume
docker run \
  -v $(pwd)/src:/app/src \
  -v app-node-modules:/app/node_modules \
  node-app
```

```bash
# ⚠️ PERMANENT — check first
docker volume prune

# ✅ Check before pruning
docker volume ls
docker volume rm specific-volume-name
```

### 📝 What Should I Remember
- Containers are **ephemeral by design** — everything written inside dies when the container is removed. This is a feature, not a bug
- **Volumes** = Docker-managed storage outside the container. Data survives container removal. Use for databases and file uploads. Syntax: `-v volume-name:/path/in/container`
- **Bind Mounts** = map a folder from your machine into the container. Changes sync instantly. Use for live code reload in development. Syntax: `-v $(pwd):/app`
- **Never bind mount `node_modules`** (or runtime equivalent) — platform-specific binaries will break. Use a named volume for it instead
- `docker volume prune` is **permanent and irreversible** — always check `docker volume ls` first

---

## Phase 5 — Networking

### 📖 The Story
You now have two containers running — a NestJS API and a PostgreSQL database. Both are running. Both have volumes. Both are healthy.

You make a request that needs the database. It fails. Connection refused.

You check PostgreSQL — running fine. You check NestJS — running fine. You check port mapping — correct. The problem is not in any container individually. The problem is **between** them.

By design, containers are islands. Each lives in its own isolated world with its own IP address assigned at runtime. That IP can change on every restart. Hardcoding it would break randomly.

The solution: custom networks with automatic DNS. Put containers on the same custom network and they find each other by **container name** — a stable DNS entry that never changes regardless of what IP they get.

### The Problem

Containers are **isolated by default**. Each container gets:
- Its own network interface
- Its own IP address (assigned at runtime, may change on restart)
- Its own DNS configuration

```
Container A (NestJS)     IP: 172.17.0.2
Container B (PostgreSQL) IP: 172.17.0.3

Problem: How does NestJS know the postgres IP?
- IPs are assigned at runtime → may change on restart
- Hardcoding IPs = breaks randomly
```

**The core question:** How do containers find each other without hardcoding fragile IPs?  
**The answer:** Custom networks with automatic DNS resolution.

---

### Mental Model

```
Docker networks = PRIVATE OFFICE FLOORS

Each floor (network) is completely separate.
People (containers) on the same floor can talk freely.
People on different floors cannot hear each other.

The floor directory (DNS) knows everyone by NAME.
You say "connect me to postgres" → directory finds them automatically.
```

---

### Three Network Drivers

#### 1. Bridge — Your Daily Tool

A private internal network on your machine, managed by Docker.

**Default Bridge vs Custom Bridge:**

| Default Bridge | Custom Bridge |
|---|---|
| Created automatically | Created by you |
| All containers join by default | Only explicitly added containers |
| ❌ No automatic DNS | ✅ Full automatic DNS |
| Find each other by IP only | Find each other by **container name** |
| Avoid it | Always use this |

> **Most important networking fact:** Default bridge = no DNS. Custom bridge = full DNS. **Always create custom networks.**

#### 2. Host — Share Machine Network

```bash
docker run -d --network host nginx
```

- Container has NO separate IP
- No port mapping needed (or possible)
- Container port 80 **is** machine port 80
- Linux only (doesn't work as expected on Mac/Windows)
- Use only for performance-critical scenarios

#### 3. None — Complete Isolation

```bash
docker run -d --network none myapp
```

- No network access whatsoever
- Use for: batch jobs, security-sensitive computation

---

### Automatic DNS on Custom Networks

```
docker network create my-app-network

Container: --name nestjs-api  → DNS entry: nestjs-api
Container: --name postgres-db → DNS entry: postgres-db
Container: --name redis-cache → DNS entry: redis-cache

From inside nestjs-api:
ping postgres-db → resolves automatically
DATABASE_URL: postgresql://user:pass@postgres-db:5432/mydb
                                      ───────────
                                      container name = hostname
```

IP addresses change on restart. Container names never change. **Always use names, not IPs.**

---

### Network Commands

```bash
docker network ls                                  # list all networks
docker network create my-app-network               # create custom network
docker network inspect my-app-network              # see connected containers
docker network connect my-app-network my-container # add container to network
docker network disconnect my-app-network my-container
docker network rm my-app-network
docker network prune                               # remove unused networks
```

**`docker network ls` output:**
```
NETWORK ID     NAME              DRIVER    SCOPE
───────────    ──────────────    ──────    ──────
a1b2c3...      bridge            bridge    local    ← default (avoid)
b2c3d4...      host              host      local
c3d4e5...      none              null      local
d4e5f6...      my-app-network    bridge    local    ← your custom ✅
```

---

### Connecting Containers — Full Example

```bash
docker network create my-app-network

docker run -d \
  --name postgres-db \
  --network my-app-network \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -v pg-data:/var/lib/postgresql/data \
  postgres:16-alpine

docker run -d \
  --name nestjs-api \
  --network my-app-network \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://admin:secret@postgres-db:5432/myapp \
  nestjs-app:latest
```

Note: `@postgres-db:5432` — container name used as hostname. Docker DNS resolves it automatically.

---

### Port Mapping vs Networking — Critical Distinction

```
PORT MAPPING (-p)                    NETWORKING (--network)
══════════════════                   ══════════════════════
Purpose: YOUR machine → container    Purpose: container → container
Direction: OUTSIDE → CONTAINER       Direction: CONTAINER → CONTAINER

-p 3000:3000                         --network my-app-network
Browser → NestJS container           NestJS → postgres-db:5432

These are INDEPENDENT of each other.
PostgreSQL does NOT need -p for NestJS to reach it.
PostgreSQL DOES need to be on the same network as NestJS.
```

> Use `-p` to expose to the world. Use `--network` for container-to-container.

---

### Security with Networking

```
Public-facing containers  → use -p → internet can reach
Internal containers       → no -p  → only reachable within network
Database containers       → NEVER use -p → never exposed to internet
```

---

### Network Isolation Between Projects

```
PROJECT A NETWORK               PROJECT B NETWORK
┌──────────────────┐           ┌──────────────────┐
│  [api-a] [db-a]  │           │  [api-b] [db-b]  │
│  can talk ✅     │           │  can talk ✅      │
└──────────────────┘           └──────────────────┘
         CANNOT reach each other ❌
         Different networks = complete isolation
```

This is exactly how Docker Compose isolates projects from each other.

---

### Debugging Network Issues

```bash
docker ps                                          # both running?
docker network inspect my-app-network             # both on same network?
docker exec -it nestjs-api sh                     # get inside

# From inside:
ping postgres-db                                  # found by DNS?
nslookup postgres-db                              # should return an IP

# If ping fails → containers not on same network
# If ping works but app fails → check DATABASE_URL format/credentials
```

---

### Common Mistakes — Networking

```bash
# ❌ Hardcoding IP
DATABASE_URL=postgresql://user:pass@172.17.0.3:5432/db  # breaks on restart

# ✅ Use container name
DATABASE_URL=postgresql://user:pass@postgres-db:5432/db  # works forever
```

```bash
# ❌ Using default bridge (no DNS)
docker run -d --name postgres-db postgres:16-alpine
docker run -d --name nestjs-api nestjs-app
# ping postgres-db → fails

# ✅ Custom network (DNS works)
docker network create my-network
docker run -d --name postgres-db --network my-network postgres:16-alpine
docker run -d --name nestjs-api --network my-network nestjs-app
# ping postgres-db → works ✅
```

### 📝 What Should I Remember
- Containers are **network-isolated by default** — they cannot reach each other unless explicitly put on the same network
- **Always use custom bridge networks** — not the default bridge. Default bridge = no DNS. Custom bridge = full automatic DNS by container name
- **Port mapping (`-p`) and networking (`--network`) are completely different** — `-p` exposes to your machine, `--network` enables container-to-container communication
- **Never hardcode container IPs** — IPs change on restart. Container names on a custom network are permanent DNS entries
- Internal services (databases, caches) should have **no `-p` flag** unless you specifically need local access — the network is your security boundary

---

## Phase 6 — Docker Compose

### 📖 The Story
End of Phase 5. You have a working NestJS + PostgreSQL + Redis stack. You got there by running five separate commands, in the correct order, with 30+ flags, remembering every environment variable, every volume name, every network name.

Then your teammate messages you: *"Hey, can you send me the setup so I can run it locally?"*

You stare at your terminal history. Five commands. Thirty flags. Specific startup order. What do you send them? A screenshot? A text file with commands? Whatever you send, they will get something wrong.

Then your tech lead says: *"We need to spin this up in CI/CD for every pull request."*

Now you need a script that runs all five commands in a CI environment with no human interaction every time someone opens a PR.

Docker Compose answers everything: write the entire stack once as a YAML file, commit it to git, and share it. `docker compose up` — done. `docker compose down` — done. Every time. On any machine.

### The Problem

After Phase 5, your full stack required 5 manual commands, 30+ flags, correct startup order, scattered environment variables. Give this to a teammate? They'll get something wrong. Every time.

```bash
# Everything you had to run manually:
docker network create myapp-network
docker volume create pg-data
docker volume create redis-data
docker run -d --name postgres-db --network myapp-network ...
docker run -d --name redis-cache --network myapp-network ...
docker run -d --name nestjs-api --network myapp-network -p 3000:3000 ...
```

**Docker Compose replaces all of this with one file.**

---

### Mental Model

```
docker-compose.yml = ARCHITECT'S BLUEPRINT for your entire stack

It defines:
• What services exist        → services:
• How they are built         → build: or image:
• What ports they expose     → ports:
• What environment they need → environment:
• What data they persist     → volumes:
• What order they start in   → depends_on:

docker compose up   = bring the blueprint to life
docker compose down = demolish everything cleanly
```

**Everything you did manually maps to YAML:**
```
docker network create      → networks: section
docker volume create       → volumes: section
docker run --name          → service name
docker run --network       → automatic (Compose creates it)
docker run -p              → ports:
docker run -e              → environment:
docker run -v              → volumes: under each service
starting in correct order  → depends_on:
```

---

### docker-compose.yml Structure

```yaml
services:        # containers that make up your application
  service-a:     # each service = one container
    ...
  service-b:
    ...

volumes:         # named volumes used by services
  my-volume:

networks:        # custom networks (Compose creates one automatically)
  my-network:
```

---

### Key Service Options

#### `image` vs `build`
```yaml
services:
  postgres-db:
    image: postgres:16-alpine      # pull from Docker Hub

  nestjs-api:
    build: .                       # build from Dockerfile in current dir
    # or more control:
    build:
      context: .
      dockerfile: Dockerfile.dev
```

#### `ports`
```yaml
ports:
  - "3000:3000"   # always quote — YAML misreads bare numbers
  - "9229:9229"
```

#### `environment`
```yaml
environment:
  NODE_ENV: production
  DATABASE_URL: postgresql://admin:secret@postgres-db:5432/appdb
  # or list format:
  - NODE_ENV=production
  - DATABASE_URL=postgresql://...
```

#### `volumes`
```yaml
services:
  postgres-db:
    volumes:
      - pg-data:/var/lib/postgresql/data     # named volume
  nestjs-api:
    volumes:
      - .:/app                               # bind mount (dev)
      - /app/node_modules                    # protect node_modules

volumes:
  pg-data:      # must declare at top level
  redis-data:
```

#### `depends_on`
```yaml
nestjs-api:
  depends_on:
    - postgres-db
    - redis-cache
```

> ⚠️ `depends_on` controls **start order only** — NOT readiness. Postgres might start but not yet accept connections.

**Fix with health checks:**
```yaml
postgres-db:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U admin -d appdb"]
    interval: 5s
    timeout: 5s
    retries: 5
    start_period: 10s

nestjs-api:
  depends_on:
    postgres-db:
      condition: service_healthy   # waits until postgres is READY ✅
```

#### `restart`
```yaml
restart: unless-stopped   # restart if crashes, not if manually stopped
```
- `"no"` → never restart (default)
- `always` → always restart
- `on-failure` → only on error
- `unless-stopped` → most common in production

---

### Networks in Compose

**Compose automatically creates a default network for the project.**  
Every service joins it automatically. Every service finds every other by **service name**.

```yaml
services:
  nestjs-api:
    # No network config needed
    # Automatically reaches postgres-db by name

  postgres-db:
    # Same — automatic
```

Default network name: `<project-folder-name>_default`

For custom network segmentation (optional):
```yaml
services:
  nestjs-api:
    networks:
      - backend
      - frontend
  postgres-db:
    networks:
      - backend          # only backend, not exposed to frontend
  react-frontend:
    networks:
      - frontend         # only frontend, no direct DB access

networks:
  backend:
  frontend:
```

---

### Environment Variables — Three Approaches

**Approach 1: Inline (fine for non-sensitive config)**
```yaml
environment:
  NODE_ENV: development
  PORT: 3000
```

**Approach 2: `.env` file (standard professional approach)**
```bash
# .env (never commit to git)
POSTGRES_USER=admin
POSTGRES_PASSWORD=secret
API_PORT=3000
```

```yaml
# docker-compose.yml (safe to commit)
environment:
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
  DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres-db:5432/${POSTGRES_DB}
```

```bash
# .env.example (commit this)
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=
```

**Approach 3: `env_file` directive**
```yaml
nestjs-api:
  env_file:
    - .env
    - .env.local    # optional overrides
```

---

### `docker compose down` — Cleanup Behavior

```
docker compose stop
→ Containers STOPPED ✅  Containers EXIST ✅  Volumes INTACT ✅

docker compose down
→ Containers STOPPED ✅  Containers REMOVED ✅  Network REMOVED ✅  Volumes INTACT ✅

docker compose down --volumes
→ Everything above + Volumes REMOVED ⚠️  DATA GONE — use with caution
```

---

### Core Compose Commands

```bash
docker compose up -d                  # start all services in background
docker compose up -d --build          # rebuild images then start
docker compose up nestjs-api          # start only one service

docker compose down                   # stop + remove containers + networks
docker compose down --volumes         # also delete volumes (⚠️ data gone)
docker compose down --rmi all         # also remove built images

docker compose build                  # build images without starting
docker compose build --no-cache       # force full rebuild

docker compose logs                   # all service logs
docker compose logs -f                # follow in real time
docker compose logs nestjs-api        # one service only

docker compose ps                     # containers for this project only
docker compose exec nestjs-api sh     # shell inside service container
docker compose exec postgres-db psql -U admin -d appdb

docker compose restart                # restart all services
docker compose pull                   # pull latest images
```

---

### Complete docker-compose.yml Example

```yaml
services:
  postgres-db:
    image: postgres:16-alpine
    container_name: myapp-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pg-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  redis-cache:
    image: redis:7-alpine
    container_name: myapp-redis
    restart: unless-stopped
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  nestjs-api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp-api
    restart: unless-stopped
    ports:
      - "${API_PORT:-3000}:3000"
    environment:
      NODE_ENV: ${NODE_ENV:-development}
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres-db:5432/${POSTGRES_DB}
      REDIS_URL: redis://redis-cache:6379
    volumes:
      - .:/app                  # bind mount for live code reload
      - /app/node_modules       # protect node_modules
    depends_on:
      postgres-db:
        condition: service_healthy
      redis-cache:
        condition: service_healthy

volumes:
  pg-data:
  redis-data:
```

---

### What Compose Is NOT

- ❌ Not a production orchestrator — that's Kubernetes
- ❌ `docker compose up` ≠ `docker compose up --build` → always `--build` after code changes
- ❌ `docker compose down` ≠ `docker compose down --volumes` — one keeps data, one deletes it
- ❌ `docker compose` (space) ≠ `docker-compose` (hyphen) — use the new one with space

---

### Forgetting `--build` — Most Common Compose Mistake

```bash
# ❌ Code changed, old image still running
docker compose up -d
# Your changes aren't there

# ✅ Always rebuild after code or Dockerfile changes
docker compose up -d --build
```

### 📝 What Should I Remember
- `docker-compose.yml` is your entire application infrastructure as code — replaces every manual `docker run`, `docker network create`, and `docker volume create`
- **`docker compose up -d --build`** is your most-used command — `-d` for background, `--build` to rebuild after code changes. Forgetting `--build` is the #1 Compose mistake
- **`docker compose down`** is safe. **`docker compose down --volumes`** deletes all data permanently — never use it unless you want a clean slate
- Use **`.env` files** for all values. Never hardcode secrets in `docker-compose.yml`. Always add `.env` to `.gitignore`. Commit `.env.example` instead
- `depends_on` controls **start order only**, not readiness — use `healthcheck` + `condition: service_healthy` when a service must wait for another to be truly ready

---

## Phase 8 — Real World Docker

### 📖 The Story
You can spin up a full NestJS + PostgreSQL + Redis stack locally with one command. Your teammates can clone the repo and be running in two minutes. Local development is smooth.

Then your tech lead says: *"Great. Now let's ship it."*

And you realize local development is only half the story. The other half is everything that happens between writing code on your laptop and that code running in production for real users.

How does the image get from your machine to a server? Who builds the image when you push code? Where do images live between build and deployment? How do you keep secrets out of your images and out of your git history?

The server is just another machine you do not control. Docker Hub is public and slow for teams. Manually building and pushing images does not scale. Secrets cannot be in Dockerfiles or git.

This phase is where Docker stops being a local tool and becomes a professional engineering workflow.

### The Gap Between Local and Production

```
Works locally ✅  ...now what?

PROBLEM 1: IMAGE DISTRIBUTION
  Your image is on your laptop. Server has never seen it. How does it get there?

PROBLEM 2: AUTOMATED BUILDING
  Cannot manually build + push every code change.
  Who builds the image? When? Triggered by what?

PROBLEM 3: SECRET MANAGEMENT
  Passwords can't be in Dockerfile, can't be in git.
  Where do they live?

PROBLEM 4: DEPLOYMENT CONSISTENCY
  Which version runs in production? How do you roll back?
```

---

### Mental Model — Postal System

```
Your code     = the letter you write
Docker image  = letter sealed in an envelope
CI/CD         = the post office
Registry      = the sorting warehouse
Production    = the final destination

You don't hand-deliver every letter.
You drop it at the post office. The system handles the rest.
```

> In professional Docker workflows, **no human manually builds or deploys images.** Code pushed to git triggers the pipeline. The pipeline does the work.

---

### "Works on My Machine" Is Still a Problem in Production

```
YOUR MACHINE     TEAMMATE'S MACHINE    PRODUCTION SERVER
────────────     ──────────────────    ─────────────────
Node 20          Node 20               Node 18 ❌
Ubuntu 22        Ubuntu 22             CentOS 7 ❌
Works ✅         Works ✅              Breaks ❌
```

The server is just another machine you don't control. Docker solves this:

```
Without Docker: Code travels → environment stays behind → breaks on server
With Docker:    Image travels → image carries its own environment → works always
```

---

### Railway and Vercel

They do Docker behind the scenes — you just never see it:

```
YOUR PROJECT (what you saw)     WHAT ACTUALLY HAPPENED
──────────────────────────      ─────────────────────────────────
git push                        git push
Railway/Vercel "just works" →   Detects push → clones repo
Secrets in dashboard            Builds image (uses your Dockerfile,
                                or auto-detects stack)
                                Injects secrets at runtime
                                Deploys the container
```

- Have a Dockerfile? → they use it
- No Dockerfile? → they auto-detect: `package.json` → Node image, `requirements.txt` → Python image

**Without Railway/Vercel (raw server):**
```
WITH PLATFORMS              WITHOUT (AWS, VPS)
────────────────────        ─────────────────────────────────
Push code                   Push code
They build image       →    YOU build image (GitHub Actions)
They store image       →    YOU push to registry (GHCR, ECR)
They run container     →    YOU pull + run on your server
They manage secrets    →    YOU manage secrets yourself
```

---

### Private Docker Registries

**Why not Docker Hub for company images:**
- ❌ Images are public by default — code exposed
- ❌ Rate limits — CI/CD pipelines hit pull limits
- ❌ Slow — Docker Hub servers far from your infrastructure
- ❌ No fine-grained team access control

**Private registry options:**

| Registry | Best For |
|---|---|
| AWS ECR | Teams on AWS |
| Google Artifact Registry | Teams on GCP |
| Azure Container Registry | Teams on Azure |
| GitHub Container Registry (GHCR) | Teams using GitHub (free for public repos, good starting point) |
| GitLab Registry | Teams using GitLab |
| Self-hosted (Harbor) | Maximum control |

The pull/push mechanics are identical — only the URL changes:
```
Docker Hub:      docker pull nginx
GHCR:            docker pull ghcr.io/myusername/myapp:latest
AWS ECR:         docker pull 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

---

### Image Tagging Strategy

| Tag | Example | When |
|---|---|---|
| Git SHA (short) | `myapp:a3f8c12` | Every commit, fully traceable |
| Semantic version | `myapp:v1.4.2` | Release versions |
| `latest` | `myapp:latest` | ❌ Never in production |

```bash
IMAGE=ghcr.io/mycompany/myapp
TAG=$(git rev-parse --short HEAD)   # short commit SHA

docker build -t $IMAGE:$TAG .
docker build -t $IMAGE:latest .
docker push $IMAGE:$TAG
docker push $IMAGE:latest

# Roll back: pull previous version by its SHA
docker pull ghcr.io/mycompany/myapp:a3f8c11
```

---

### CI/CD — What It Means

**CI (Continuous Integration):**
- Every push to git → build image, run tests, report pass/fail on PR

**CD (Continuous Deployment):**
- Code merged to main → build prod image, push to registry, deploy to server

**Done via GitHub Actions** — YAML files in `.github/workflows/`

Each workflow has:
- **Triggers** → when to run (push, PR, tag)
- **Jobs** → main tasks (test, build, deploy)
- **Steps** → individual commands per job

---

### Complete GitHub Actions Pipeline

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]       # runs on merge to main
  pull_request:
    branches: [main]       # runs on PR (test job only)

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker compose -f docker-compose.test.yml up -d --build
      - run: docker compose -f docker-compose.test.yml exec -T api npm run test
      - run: docker compose -f docker-compose.test.yml down --volumes
        if: always()

  build-and-push:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'   # only on merge, not PRs
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USER }}
          key: ${{ secrets.PROD_SSH_PRIVATE_KEY }}
          script: |
            docker pull ghcr.io/${{ env.IMAGE_NAME }}:latest
            cd /opt/myapp
            docker compose pull
            docker compose up -d --no-build
            docker compose ps
```

**How triggers and `if:` conditions work:**
```
                  PULL REQUEST OPENED    MERGE TO MAIN
                  ───────────────────    ──────────────────────
test:             ✅ runs                ✅ runs
build-and-push:   ❌ skipped (if: gate) ✅ runs
deploy:           ❌ skipped (if: gate) ✅ runs
```

- Triggers at the top decide **when the workflow starts**
- `if: github.ref == 'refs/heads/main'` is the **gate** inside each job

---

### The Full Deployment Flow

```
DEVELOPER'S MACHINE
git push → triggers CI
        ↓
GITHUB ACTIONS (CI — temporary Linux VM)
1. Checkout code
2. docker compose -f docker-compose.test.yml up -d --build
3. Run tests inside containers
4. docker compose down --volumes
5. ✅ or ❌ reported on PR
        ↓ (PR merged to main)
GITHUB ACTIONS (CD)
1. docker build -t ghcr.io/mycompany/myapp:a3f8c12 .
2. docker push ghcr.io/mycompany/myapp:a3f8c12
3. docker push ghcr.io/mycompany/myapp:latest
        ↓ (image pushed)
PRIVATE REGISTRY (GHCR)
ghcr.io/mycompany/myapp:a3f8c12  ← new version
ghcr.io/mycompany/myapp:a3f8c11  ← previous (rollback available)
ghcr.io/mycompany/myapp:latest   ← points to a3f8c12
        ↓ (CD SSHes to your server)
PRODUCTION SERVER
docker compose pull
docker compose up -d
Secrets: injected from .env or secrets manager
Running: myapp:a3f8c12 ✅
```

**Key:** Your EC2 never talks to GitHub's build machine directly.
```
GitHub's temp machine → pushes image → GHCR (warehouse)
Your EC2 server       → pulls image  → GHCR (warehouse)
They only meet at GHCR — in the middle.
```

---

### docker-compose.yml vs deploy.yml

| | `docker-compose.yml` | `deploy.yml` (GitHub Actions) |
|---|---|---|
| **What** | Defines HOW to run containers | Defines WHEN and HOW to automate |
| **Who reads it** | Docker | GitHub Actions |
| **Lives in** | Root of repo | `.github/workflows/` |
| **Push to git?** | ✅ Yes (no real values) | ✅ Yes |

```
deploy.yml wakes up (push triggers it)
    ↓
deploy.yml SSHes into your server
    ↓
deploy.yml runs: docker compose up -d
    ↓
docker-compose.yml does the actual container work
```

**Two repos = two pipelines:**
```
frontend-repo/
├── Dockerfile
├── docker-compose.yml
└── .github/workflows/deploy.yml

backend-repo/
├── Dockerfile
├── docker-compose.yml
└── .github/workflows/deploy.yml
```

Each repo deploys independently. They connect only through a URL at runtime (API_URL env var), not through Docker.

---

### Secrets — Where They Cannot Live

```
❌ Hardcoded in Dockerfile
   ENV DATABASE_PASSWORD=supersecret
   → baked into image forever, visible via docker history

❌ In .env committed to git
   → in git history FOREVER even after deletion

❌ Via --build-arg
   → visible in docker history, same problem
```

---

### Secrets — Where They Should Live

**For CI/CD pipelines (GitHub Actions Secrets):**
```
GitHub repo → Settings → Secrets and variables → Actions → New secret
```
```yaml
- name: Deploy
  env:
    DB_PASSWORD: ${{ secrets.PROD_DATABASE_PASSWORD }}
```
- Encrypted at rest
- Never appear in logs (shown as `***`)
- Only available during pipeline run

**For containers at runtime (injection):**
```bash
# ❌ Build time — secret baked in
docker build --build-arg DB_PASSWORD=secret .

# ✅ Runtime — secret injected when container starts
docker run -d -e DATABASE_PASSWORD=$DATABASE_PASSWORD myapp:latest
```

**The correct mental model:**
```
WRONG: "My image has all the config it needs"
       → secrets baked in, image is dangerous if it leaks

CORRECT: "My image knows NOTHING about its environment"
         Image = code + dependencies only
         Secrets = provided at runtime by the environment

Same image runs in:
→ Local dev    (dev secrets from .env)
→ CI testing   (secrets from GitHub Actions)
→ Production   (secrets from secret manager / server .env)
```

---

### Secrets Methods (Not Required Levels — Just Options)

| Method | Use When |
|---|---|
| GitHub Actions Secrets | Secrets needed in CI/CD pipeline |
| Runtime env injection | Secrets passed to containers at start |
| Cloud Secret Managers (AWS Secrets Manager, HashiCorp Vault) | Production at scale |
| Docker Secrets | Only in Docker Swarm mode |

The one rule that applies to all: **secret never touches the Dockerfile, never touches git.**

---

### Secrets Are Injected at Runtime — Not Build Time

```
BUILD PHASE                     RUN PHASE
──────────────────              ─────────────────────────────────
Railway reads your code         Railway starts the container
Builds the image                Injects your secrets as env vars
NO secrets here ✅              Container sees them at runtime ✅
```

Change a secret? Restart the container — no rebuild needed.

---

### Your EC2 Workflow (Git-Pull Pattern)

```yaml
# Your actual deploy.yml pattern
- name: Deploy on EC2 over SSH
  uses: appleboy/ssh-action@v1.0.3
  with:
    script: |
      cd "${{ secrets.EC2_PROJECT_PATH }}"
      git -C crowdfund-backend fetch origin dev
      git -C crowdfund-backend reset --hard origin/dev
      docker compose -p crowdfund up -d --build --no-deps backend
      docker builder prune -af || true
      docker image prune -f || true
```

**This is valid but different from the registry pattern:**

| Your EC2 Pattern | Registry Pattern |
|---|---|
| Simpler, fewer steps ✅ | More setup required ❌ |
| Build happens ON EC2 ❌ | Build happens in CI ✅ |
| Uses EC2 CPU for building ❌ | EC2 only runs containers ✅ |
| Harder to roll back ❌ | Roll back = pull old image tag ✅ |

> ⚠️ `docker volume prune -f` in that script runs on every deploy. Risk of deleting your database volume if it's temporarily unused during deploy. Be specific about what you prune.

---

### Maintenance Commands

```bash
docker system df               # see how much disk Docker uses
docker system prune            # remove stopped containers, unused networks, dangling images
docker system prune -a --volumes  # ⚠️ nuclear — removes everything unused
```

### GitHub Actions — Triggers vs Job Conditions

A common confusion from our discussion:

```
on:                                ← triggers: when does the workflow START?
  push:
    branches: [main]               ← starts on merge to main
  pull_request:
    branches: [main]               ← starts on PR opened to main

jobs:
  test:
    # no if: condition             ← runs on BOTH triggers
  build-and-push:
    if: github.ref == 'refs/heads/main'   ← gate: only TRUE on real merge
    # on a PR: github.ref = refs/pull/123/merge → condition FALSE → job skipped
  deploy:
    if: github.ref == 'refs/heads/main'   ← same gate
```

```
PULL REQUEST OPENED        MERGE TO MAIN
───────────────────        ──────────────
test          ✅ runs      test          ✅ runs
build-push    ❌ skipped   build-push    ✅ runs
deploy        ❌ skipped   deploy        ✅ runs
```

> Triggers decide **when the workflow starts**. `if:` conditions decide **which jobs actually run** inside it.

### 📝 What Should I Remember
- **No human manually builds or deploys images** in professional workflows — a `git push` triggers the pipeline, the pipeline does everything. You define it once
- Use a **private registry** (GHCR, ECR) for your own images — tag with **git SHA** for traceability and easy rollback. Never use `latest` in production
- **Secrets never touch Dockerfiles, Compose files, or git** — images contain zero secrets. Secrets are injected at runtime. The image is built blind; secrets are handed to the container when it starts
- `docker-compose.yml` (defines HOW to run containers) and `deploy.yml` (defines WHEN to run them automatically) are two different files — both safe to commit, both live in every repo
- The image is built on a **temporary GitHub machine**, stored in GHCR/ECR, then your production server independently pulls it from there — the server never talks directly to GitHub's build machine

---

## Phase 9 — Bridge to Orchestration

### 📖 The Story
Your application is in production. Docker Compose on a single server. Everything works. Users are happy.

Then your app gets featured on a popular website. Traffic spikes from 100 to 10,000 requests per minute. Your server's CPU hits 100%. Response times climb from 50ms to 8 seconds. Your single NestJS container is choking under the load.

You think about running more containers — but they all live on the same overwhelmed machine. You are just splitting the same broken pie into more slices.

Then at 3am that same night, the server's hard drive fails. Everything goes down. The database. The API. The frontend. One hardware failure. Total outage. You spend four hours restoring from backup.

Two problems Docker Compose on one machine cannot solve: **scale** (you need more machines) and **availability** (when a machine dies, containers must move automatically). These two problems are exactly what container orchestration was built to solve.

### The Problem — When One Machine Is Not Enough

**Scale problem:** Traffic spikes → you need more machines, not just more containers on the same machine.

**Availability problem:** Server's hard drive fails → everything dies. You need containers to automatically move to another machine.

```
DOCKER COMPOSE ON ONE SERVER
❌ Cannot span multiple machines
❌ Cannot self-heal if server fails
❌ Cannot auto-scale on load
❌ Cannot do zero-downtime rolling deploys automatically
❌ Cannot balance load across instances automatically
```

These are not bugs — Compose is scoped to one machine by design.

---

### Orchestration — Shift in Thinking

```
WITHOUT ORCHESTRATION:
"This container runs on server-03"
→ You manage servers, containers follow

WITH ORCHESTRATION:
"I want 5 replicas of this container. Figure it out."
→ You declare desired state
→ Orchestrator manages which machine runs what
→ You stop caring about specific machines
```

---

### What an Orchestrator Does

```
YOU DECLARE:                          ORCHESTRATOR DOES:
─────────────────────────             ──────────────────────────────────
nestjs-api: 5 replicas                Places containers on best machines
postgres-db: 1 replica                Monitors all containers constantly
redis-cache: 2 replicas               Restarts any that crash
All healthy and reachable             Replaces containers on failed machines
                                      Distributes network traffic
                                      Handles deployments without downtime
                                      Scales up/down based on load
```

**Problems orchestration solves:**
- **Scheduling** → which machine runs which container
- **Health monitoring** → crashed container replaced in seconds, no human needed
- **Service discovery** → containers find each other by name even when they move machines
- **Load balancing** → traffic distributed across healthy replicas automatically
- **Rolling deployments** → update one replica at a time, no downtime

---

### Docker Swarm

- Docker's **built-in** orchestration — no extra installation needed
- Turns a group of Docker machines into one virtual Docker host
- Uses familiar docker-compose.yml format

```
WITHOUT SWARM:
Machine A → manages only Machine A
Machine B → manages only Machine B

WITH SWARM:
Machine A (manager) → controls containers on ALL machines
Machine B (worker)  → receives and executes instructions
Machine C (worker)  → receives and executes instructions
```

```
DOCKER SWARM CLUSTER
┌─────────────────────────────────────────┐
│  MANAGER NODE                           │
│  • Accepts commands                     │
│  • Schedules containers                 │
│  • Monitors cluster health              │
│       │ orchestrates                    │
│  ┌────┴────┐   ┌────────────┐           │
│  │ WORKER  │   │ WORKER     │           │
│  │Container│   │Container   │           │
│  │Container│   │Container   │           │
│  └─────────┘   └────────────┘           │
└─────────────────────────────────────────┘
```

**Core Swarm commands:**
```bash
docker swarm init                                    # initialize swarm on manager
docker swarm join --token <token> <manager-ip>:2377  # join as worker
docker node ls                                       # list all machines
docker service create --name api --replicas 5 myapp:latest
docker service scale api=10
docker stack deploy -c docker-compose.yml myapp
docker service ls
docker stack rm myapp
```

**Swarm strengths vs weaknesses:**

| ✅ Strengths | ❌ Weaknesses |
|---|---|
| 5 minutes from zero to cluster | Smaller ecosystem |
| Uses docker-compose.yml format | Less powerful autoscaling |
| Built into Docker | Cloud providers built Kubernetes, not Swarm |
| Good for simple multi-machine needs | Lost the orchestration war |

---

### Why Kubernetes Won

When AWS, Google, and Azure all launched **managed Kubernetes** (EKS, GKE, AKS) — not managed Swarm — the industry followed. Kubernetes became the universal standard.

---

### Kubernetes (K8s)

- Open-sourced by Google in 2014
- Originally built as "Borg" — Google's internal system for running billions of containers
- K8s = K + 8 letters + s

**Kubernetes Architecture:**
```
KUBERNETES CLUSTER
┌──────────────────────────────────────────────────────────┐
│  CONTROL PLANE (the brain)                               │
│  API Server  → entry point for all commands              │
│  Scheduler   → decides which node runs what              │
│  Controller  → watches state, fixes drift from desired   │
│  etcd        → stores ALL cluster state                  │
└──────────────────────────┬───────────────────────────────┘
                           │ manages
          ┌────────────────┼──────────────────┐
          │                │                  │
   WORKER NODE      WORKER NODE        WORKER NODE
   kubelet          kubelet            kubelet
   [Pod][Pod]       [Pod][Pod]         [Pod][Pod][Pod]
```

---

### Kubernetes Vocabulary — Mapped to What You Know

| Docker/Compose | Kubernetes | Description |
|---|---|---|
| Container | Pod | Wrapper around 1+ containers (usually 1) |
| `--replicas 5` | Deployment | Maintains desired number of Pods |
| `-p port` / Service in Compose | Service | Stable network access + load balancing |
| `.env` / `environment:` | ConfigMap + Secret | Non-sensitive / sensitive config |
| Volume | PersistentVolumeClaim | Persistent storage outside Pods |
| `docker-compose.yml` | Multiple YAML manifests | Desired state as code |
| `docker compose up` | `kubectl apply -f` | Apply desired state |
| `docker ps` | `kubectl get pods` | List running things |
| `docker logs` | `kubectl logs pod-name` | See output |
| `docker exec -it` | `kubectl exec -it pod-name -- bash` | Get inside |

---

### `kubectl` — The Kubernetes CLI

```bash
kubectl apply -f deployment.yaml          # create/update resources
kubectl get pods                          # list Pods
kubectl get deployments                   # list Deployments
kubectl get services                      # list Services
kubectl describe pod my-pod               # detailed info
kubectl logs my-pod                       # see logs
kubectl logs -f my-pod                    # follow logs
kubectl exec -it my-pod -- bash           # get inside
kubectl scale deployment api --replicas=10
kubectl set image deployment/api api=myapp:v2.0   # rolling deploy
kubectl rollout undo deployment/api               # roll back
kubectl rollout status deployment/api             # watch update
kubectl delete -f deployment.yaml
```

---

### What Kubernetes Is NOT

- ❌ Not a replacement for Docker → they work together (Docker creates containers, K8s orchestrates them)
- ❌ Not only for huge companies → managed services (EKS, GKE, AKS) make it accessible
- ❌ Kubernetes YAML ≠ docker-compose.yml → different format, more verbose
- ❌ Not just about scale → high availability (surviving machine failures) matters equally
- ❌ Docker Swarm is not dead → still a valid choice for simpler needs

---

### Compose vs Swarm vs Kubernetes

```
How many servers do you need?

ONE SERVER
  └── Docker Compose ✅  (simple, sufficient, no overhead)

MULTIPLE SERVERS (2–10), simple needs
  └── Docker Swarm ✅  (easy, built into Docker, familiar format)

MULTIPLE SERVERS, full orchestration
  └── Kubernetes ✅  (managed: EKS, GKE, AKS)

MANY SERVERS (10+) or cloud-native
  └── Kubernetes ✅  (industry standard)
```

> **Most applications never need Kubernetes.** A well-configured single server with Docker Compose can handle millions of requests per day. Add orchestration only when you genuinely feel the pain of single-machine limits.

---

### Your Docker Knowledge Transfers to Kubernetes

| What You Learned | How It Maps |
|---|---|
| Dockerfile | Kubernetes uses the **exact same images** — no changes needed |
| docker push/pull | Kubernetes pulls from the **same registries** |
| Volumes | PersistentVolumes work the same — data outside containers |
| Custom networks / DNS | Services provide the same DNS resolution |
| docker-compose.yml | Kubernetes YAML manifests (more files, same concept) |
| CI/CD pipelines | Build step is **identical** — only deploy step changes |

---

### The Complete Container Ecosystem

```
DEVELOPMENT
You write code → Dockerfile packages it → docker compose up (local dev)
        ↓ git push
CI/CD PIPELINE (GitHub Actions)
docker build → docker push → registry
Tests run inside containers → Image tagged with git SHA
        ↓ image pushed
CONTAINER REGISTRY (GHCR / ECR / GCR)
myapp:a3f8c12  myapp:v1.4.2  myapp:latest
        ↓ pulled by
┌──────────────────────────┐    ┌───────────────────────────────┐
│  SINGLE SERVER (Compose) │    │  CLUSTER (Kubernetes / Swarm) │
│  docker compose up -d    │    │  kubectl apply                │
│  [NestJS][Postgres][Redis]│    │  Node1:[Pod][Pod]             │
│                          │    │  Node2:[Pod][Pod]             │
│  Small-medium apps ✅    │    │  Node3:[Pod][Pod][Pod]        │
│  Simple ops ✅           │    │  High availability ✅         │
│  Single server ✅        │    │  Auto-scaling ✅              │
└──────────────────────────┘    └───────────────────────────────┘
```

---

### The Thread Connecting Everything

Every concept in this course traces to one problem:

> **"Make software run the same way, everywhere, reliably, at scale."**

```
Dockerfile    → same environment, always
Volumes       → data survives, always
Networks      → services communicate, always
Compose       → full stack starts, one command, always
CI/CD         → deploys correctly, automatically, always
Orchestration → runs reliably across any number of machines
```

### 📝 What Should I Remember
- Orchestration solves two things Compose cannot: **horizontal scale** (more machines when traffic grows) and **high availability** (automatic recovery when a machine fails)
- **Docker Swarm** is Docker's built-in orchestration — easy to set up, uses familiar format, good for simple multi-machine needs, but lost the industry standard war
- **Kubernetes won** — every major cloud has a managed service (EKS, GKE, AKS). Your Docker images work in Kubernetes without modification — the container format is universal
- Kubernetes vocabulary maps directly to what you know: Pod = container, Deployment = desired replicas, Service = networking/load balancing, PVC = volume, ConfigMap/Secret = environment config
- **Most applications do not need Kubernetes** — Compose on a well-configured server handles enormous scale. Only add orchestration when you genuinely feel the pain of single-machine limits

---


---
---

# Hands-On Tutorial: Dockerizing, Testing, CI/CD & Deploying a Real Project

> This tutorial assumes you understand the Docker concepts from the notes above but have never applied them to a real project. We build one, together, from an empty folder to a live production deployment on AWS.

## The Project We're Building

Two separate Git repositories, matching how real teams actually work:

```
crowdfund-backend/     ← NestJS API + PostgreSQL
crowdfund-frontend/    ← React app

Each repo:
- Has its own Dockerfile
- Has its own docker-compose.yml
- Has its own CI/CD pipeline
- Deploys independently
```

**Why this stack:** NestJS and React are the technologies you already know from your background — this tutorial applies concepts, it doesn't teach new frameworks.

```
FULL ARCHITECTURE
┌─────────────┐      HTTP/JSON      ┌──────────────┐      SQL      ┌────────────┐
│   React     │ ──────────────────► │   NestJS     │ ────────────► │ PostgreSQL │
│  Frontend   │ ◄────────────────── │   Backend    │ ◄──────────── │  Database  │
└─────────────┘                     └──────────────┘               └────────────┘
  Own repo                            Own repo                     Container only
  Own Dockerfile                      Own Dockerfile                (no repo needed)
```

### Core Path vs Advanced/Optional

Not every phase carries equal weight. Some are essential to get a working deployment. Others are real-world upgrades you can add later, once the basics work. This distinction is marked throughout with a badge next to the phase title.

```
🟢 CORE — do these in order, this is the minimum working deployment
🟡 ADVANCED / OPTIONAL — valuable, but skip on a first pass without guilt

🟢 Phase 1   Architecture & requirements
🟢 Phase 2   What to containerize
🟢 Phase 3   Preparing repositories
🟢 Phase 4   Backend Dockerfile
🟢 Phase 5   Frontend Dockerfile (simple version first, multi-stage is the 🟡 upgrade)
🟢 Phase 6   .dockerignore
🟢 Phase 7   Building & testing images
🟢 Phase 8   Running containers individually
🟢 Phase 9   Networking
🟢 Phase 10  Environment variables
🟢 Phase 11  Volumes
🟢 Phase 12  docker-compose.yml (dev)
🟢 Phase 13  Running the full app
🟡 Phase 14  Tests inside containers (dedicated test compose file is 🟡)
🟢 Phase 15  Debugging & troubleshooting
🟡 Phase 16  Optimizing the dev workflow
🟢 Phase 17  Production-ready images (multi-stage is the 🟡 upgrade)
🟢 Phase 18  Production docker-compose
🟢 Phase 19  CI/CD pipeline
🟢 Phase 20  Deploying to AWS (plain HTTP first — HTTPS/reverse proxy is 🟡)
🟢 Phase 21  Verifying the deployment
🟡 Phase 22  Monitoring & maintenance
```

> If this is your first real Docker project: work through every 🟢 phase in order, get the app live over plain HTTP, and stop there. Come back for the 🟡 phases once that's working and feels solid — they are genuine best practices, but they are not what stands between you and a working deployment.

---

## Phase 1 — Understanding the Project Architecture and Requirements 🟢

**Goal:** Know what you're building before writing a single Dockerfile.

**Why this step exists:** Jumping straight into Dockerfiles without mapping the architecture leads to containers that don't talk to each other correctly, wrong ports, and confusing rebuilds later.

### The Three Services

| Service | Technology | Talks To | Exposed to Internet? |
|---|---|---|---|
| Frontend | React (Vite) | Backend API (via URL) | ✅ Yes |
| Backend | NestJS | PostgreSQL (via network) | ✅ Yes (API only) |
| Database | PostgreSQL | Nothing (receives only) | ❌ Never |

### Request Flow

```
Browser
   │
   │ 1. Loads React app (static files)
   ▼
React Frontend (served by nginx in production)
   │
   │ 2. JS makes fetch() calls to backend API
   ▼
NestJS Backend (port 3000)
   │
   │ 3. Backend queries the database
   ▼
PostgreSQL (port 5432, internal only)
```

**Key decision made here:** Frontend and backend are **separate repos** — this means separate Dockerfiles, separate CI/CD pipelines, separate deployments. They only connect through a URL at runtime (the frontend calls the backend's public API address), never through Docker networking directly, since in production they may not even be on the same server.

### Common Mistakes
- ❌ Designing the Dockerfiles before knowing which service talks to which
- ❌ Assuming frontend and backend must be in the same repo because they're "one project"

### Best Practices
- ✅ Draw the request flow before writing any code
- ✅ Decide early: does each service need its own repo, or can it be a monorepo? (We use separate repos here because it's the more common real-world pattern — matches your own EC2 project structure)

---

## Phase 2 — Deciding What Should and Shouldn't Be Containerized 🟢

**Goal:** Understand which parts of your stack belong in a container and which don't.

**Why this step exists:** Beginners often try to containerize everything, including things that don't need it, or things that actively make development harder inside a container.

```
SHOULD BE CONTAINERIZED                  SHOULDN'T BE CONTAINERIZED
─────────────────────────                ─────────────────────────
✅ NestJS backend (the app itself)       ❌ Your IDE / code editor
✅ React build process                   ❌ Git itself
✅ PostgreSQL database                   ❌ node_modules on your HOST
✅ Redis (if used for caching)              (lives inside the container,
✅ Any background workers                   not bind-mounted raw)
                                          ❌ .env files with real secrets
                                             (never baked into an image)
```

### Decision Framework

```
Does it need to run as a process?
  │
  ├── YES → Does it need a consistent environment across machines?
  │           │
  │           ├── YES → Containerize it ✅
  │           └── NO  → Maybe not worth the overhead
  │
  └── NO (it's a file, a secret, a tool you run once) → Don't containerize
```

**Applied to our project:**
- NestJS backend → containerized (runs as a process, needs consistent Node version)
- React frontend → containerized (build process needs consistent Node/npm version, and nginx serves the built files)
- PostgreSQL → containerized (official image, no need to install Postgres locally)
- `.env` files → **never** containerized — injected at runtime (Phase 10 covers this)

### Common Mistakes
- ❌ Containerizing your database migrations as a separate always-running container (they should run once, not stay running)
- ❌ Trying to put your code editor's config inside the image

### Best Practices
- ✅ If it "runs and stays running" → container
- ✅ If it's a one-time or occasional command → run it via `docker exec` or `docker compose run`, not as a persistent service


---

## Phase 3 — Preparing Each Repository for Docker 🟢

**Goal:** Set up the folder structure so Docker has what it needs in each repo.

**Prerequisites:** A working NestJS app (`crowdfund-backend/`) and a working React app (`crowdfund-frontend/`), each already runnable with `npm run start:dev` / `npm run dev` locally (outside Docker) — Docker doesn't fix a broken app, it packages a working one.

### Backend Repo Structure

```
crowdfund-backend/
├── src/
│   └── ... (your NestJS source code)
├── package.json
├── package-lock.json
├── Dockerfile              ← we write this next (Phase 4)
├── .dockerignore           ← Phase 6
├── docker-compose.yml      ← Phase 12
├── .env.example            ← committed
├── .env                    ← NOT committed
└── .gitignore
```

### Frontend Repo Structure

```
crowdfund-frontend/
├── src/
│   └── ... (your React source code)
├── package.json
├── package-lock.json
├── Dockerfile              ← Phase 5
├── .dockerignore           ← Phase 6
├── docker-compose.yml      ← optional, mostly for local full-stack testing
├── .env.example
├── .env
└── .gitignore
```

### `.gitignore` — What Belongs Here (Not Docker-Specific, But Critical)

```gitignore
node_modules/
dist/
.env
*.log
```

> `node_modules` is git-ignored because dependencies are reproducible from `package.json`. This same reasoning is *why* we don't bind-mount it raw into containers either (Phase 11 explains this fully).

### Verification Steps
```bash
# In each repo, confirm the app runs WITHOUT Docker first
cd crowdfund-backend && npm run start:dev   # should work
cd crowdfund-frontend && npm run dev        # should work
```

### Common Mistakes
- ❌ Writing a Dockerfile for an app that doesn't run locally yet — debug the app first, then containerize it
- ❌ Committing `.env` before adding it to `.gitignore`

---

## Phase 4 — Writing the Backend Dockerfile From Scratch 🟢

**Goal:** Package the NestJS API into a portable image.

**Why this step exists:** This is where "works on my machine" for the backend officially ends.

**Files to create:** `crowdfund-backend/Dockerfile`

### Building It Instruction by Instruction

```dockerfile
# ── Base Image ───────────────────────────────────────────────
# Use Node LTS on Alpine — small, fast to pull, good for CI/CD
FROM node:20-alpine

# ── Working Directory ────────────────────────────────────────
WORKDIR /app

# ── Dependencies First (cache optimization) ──────────────────
# Copy ONLY package files — this layer is cached unless
# dependencies change, so rebuilds after code edits are fast
COPY package*.json ./
RUN npm ci

# ── Application Code ─────────────────────────────────────────
COPY . .

# ── Build the NestJS app (TypeScript → JavaScript) ───────────
RUN npm run build

# ── Port Documentation ───────────────────────────────────────
EXPOSE 3000

# ── Startup Command ──────────────────────────────────────────
# Run the compiled output, not the TypeScript source
CMD ["node", "dist/main.js"]
```

### Why `npm ci` Instead of `npm install`

```
npm install → may update package-lock.json, slightly non-deterministic
npm ci      → installs EXACTLY what's in package-lock.json, faster, deterministic

Inside Docker: always prefer npm ci.
Reproducibility matters more than convenience in a build environment.
```

### Command Explanation — Building the Image

```bash
docker build -t crowdfund-backend:dev .
```

```
docker build   -t crowdfund-backend:dev   .
      │               │                   │
      │               │                   └── build context = current directory
      │               └── name:tag you choose
      └── the build command
```

### Expected Output

```
[+] Building 24.3s (12/12) FINISHED
 => [1/6] FROM docker.io/library/node:20-alpine
 => [2/6] WORKDIR /app
 => [3/6] COPY package*.json ./
 => [4/6] RUN npm ci
 => [5/6] COPY . .
 => [6/6] RUN npm run build
 => exporting to image
 => naming to docker.io/library/crowdfund-backend:dev
```

### Verification Steps
```bash
docker images                       # confirm crowdfund-backend:dev exists
docker run -p 3000:3000 crowdfund-backend:dev
# then in another terminal:
curl http://localhost:3000          # should get a response (even an error is fine —
                                     # it proves the server is running)
```

### Common Mistakes
- ❌ Forgetting `RUN npm run build` — the container tries to run TypeScript source directly and crashes
- ❌ Using `CMD ["npm", "run", "start:dev"]` in production — that's the dev watcher, slower and not meant for production
- ❌ Copying `.env` into the image with `COPY . .` before adding it to `.dockerignore` (Phase 6 fixes this)

### Best Practices
- ✅ Always copy `package*.json` before the rest of the code (cache optimization from the notes above)
- ✅ Use the compiled `dist/main.js` as your CMD, not the raw TypeScript


---

## Phase 5 — Writing the Frontend Dockerfile From Scratch 🟢 (multi-stage upgrade is 🟡)

**Goal:** Package the React app so it builds and serves correctly in a container.

**Prerequisites:** Comfortable with `FROM` / `WORKDIR` / `COPY` / `RUN` / `CMD` from Phase 4 — this phase adds one new idea on top of those (multi-stage), nothing else.

**Files to create:** `crowdfund-frontend/Dockerfile`

### Step 1 — The Simple Version (Get This Working First)

React apps have a build step: your JSX/TypeScript compiles into plain HTML/CSS/JS files. The simplest possible Dockerfile just does that build and serves the result with `nginx`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Install nginx to serve the built files
RUN apk add --no-cache nginx

EXPOSE 80

# Copy build output to where nginx expects it, then start nginx
CMD sh -c "cp -r dist/* /usr/share/nginx/html/ && nginx -g 'daemon off;'"
```

This works and is easy to reason about: one `FROM`, top to bottom, nothing hidden.

### Command Explanation

```bash
docker build -t crowdfund-frontend:dev .
```

### Verification Steps
```bash
docker images                          # check crowdfund-frontend:dev exists
docker run -p 8080:80 crowdfund-frontend:dev
# open http://localhost:8080 — React app should load
```

### The One Real Problem With the Simple Version

```bash
docker images
# crowdfund-frontend:dev   ~400MB
```

That 400MB contains your entire `node_modules`, the npm build tools, and your source code — none of which are needed once the static files exist. Only the small `dist/` folder (a few MB) actually needs to ship. This is the exact problem multi-stage builds solve — worth knowing once the simple version works, but not required to get started.

---

### Step 2 (🟡 Advanced/Optional) — Multi-Stage Upgrade

**Prerequisites:** The simple version above is already working for you locally.

The idea: use one temporary stage with all the build tools, then throw that stage away and keep only the output in a fresh, tiny image.

```dockerfile
# ── STAGE 1: Build ───────────────────────────────────────────
# This stage has all the tools needed to build the app.
# It will NOT be part of the final image.
FROM node:20-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# ── STAGE 2: Serve ───────────────────────────────────────────
# Fresh, tiny image — only nginx + the built files.
# None of the build tools, none of node_modules, land here.
FROM nginx:alpine AS production

# Copy ONLY the built static files from the build stage
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Why Multi-Stage Matters (Visualized)

```
SIMPLE VERSION (Step 1)              MULTI-STAGE (Step 2)
────────────────────────             ──────────────────────
FROM node:20-alpine                  STAGE 1: FROM node:20-alpine
  + all your source code               + all your source code
  + full node_modules                  + full node_modules
  + build tools                        + build tools
  + nginx installed on top             + built dist/ files
                                        ↓ only dist/ is copied out
Final image: ~400MB                  STAGE 2: FROM nginx:alpine
                                        + only dist/ files
                                      Final image: ~25MB ✅
```

Same `docker build` command as before — Docker automatically detects the multi-stage syntax and only keeps the final stage (`production`) in the resulting image.

### Expected Output

```
[+] Building 31.8s (14/14) FINISHED
 => [build 1/5] FROM docker.io/library/node:20-alpine
 => [build 4/5] RUN npm ci
 => [build 5/5] RUN npm run build
 => [production 1/2] FROM docker.io/library/nginx:alpine
 => [production 2/2] COPY --from=build /app/dist /usr/share/nginx/html
 => exporting to image
```

### Common Mistakes
- ❌ Forgetting `AS build` / `AS production` stage names — makes `COPY --from=` fail
- ❌ Using `FROM node:20-alpine` for the final image instead of `nginx:alpine` — ships unnecessary Node.js runtime for what is now just static files
- ❌ Not setting the `VITE_API_URL` (or equivalent) environment variable at build time — the built JS bundle needs to know the backend URL (React env vars are baked in at *build* time, unlike backend env vars which are runtime — an important distinction covered in Phase 10)

### Best Practices
- ✅ Always multi-stage build frontend apps — the size difference (400MB vs 25MB) directly affects deploy speed and registry costs
- ✅ Use `nginx:alpine` as the serving image — it's the industry standard for static file serving

---

## Phase 6 — Understanding and Writing `.dockerignore` 🟢

**Goal:** Prevent unnecessary or sensitive files from being sent to the Docker daemon during build.

**Why this step exists:** `COPY . .` copies *everything* in your build context unless told otherwise. Without a `.dockerignore`, you accidentally: send `node_modules` (huge, and platform-specific — see Phase 4 notes on this), send `.env` (a secrets leak straight into your image), and slow down every build.

```
.dockerignore = .gitignore, but for the Docker build context
```

### Backend `.dockerignore`

```dockerignore
node_modules
dist
.env
.env.local
*.log
.git
.gitignore
README.md
.vscode
```

### Frontend `.dockerignore`

```dockerignore
node_modules
dist
build
.env
.env.local
*.log
.git
.gitignore
README.md
.vscode
```

### Visualizing the Effect

```
WITHOUT .dockerignore                  WITH .dockerignore
──────────────────────                 ──────────────────────
Build context sent to daemon:          Build context sent to daemon:
├── src/                               ├── src/
├── node_modules/  (300MB!) ❌         ├── package.json
├── .env  (SECRETS!) ❌                ├── package-lock.json
├── .git/  (huge history) ❌           └── Dockerfile
├── package.json
└── Dockerfile                         Build: fast, secure ✅

Build: slow, and .env risks
being COPYed into a layer ❌
```

### Verification Steps
```bash
docker build --progress=plain -t crowdfund-backend:dev . 2>&1 | head -5
# "Sending build context to Docker daemon" — this line's SIZE tells you
# if .dockerignore is doing its job (should be a few MB, not hundreds)
```

### Common Mistakes
- ❌ Adding `.dockerignore` after already building images that contain secrets — old images still have the secret baked in, must rebuild AND remove the old ones
- ❌ Forgetting `.env` in `.dockerignore` — the single most dangerous omission

### Best Practices
- ✅ Write `.dockerignore` **before** your first `docker build`, not after
- ✅ Treat it with the same seriousness as `.gitignore`


---

## Phase 7 — Building and Testing Docker Images 🟢

**Goal:** Confirm both images build cleanly and produce the expected result before wiring anything together.

**Why this step exists:** Debugging is far easier one image at a time than after everything is connected through Compose. Isolate problems early.

### Build Both Images

```bash
cd crowdfund-backend
docker build -t crowdfund-backend:dev .

cd ../crowdfund-frontend
docker build -t crowdfund-frontend:dev .
```

### Testing Checklist Per Image

```
FOR EACH IMAGE, VERIFY:
┌─────────────────────────────────────────────────┐
│ ✅ docker build completes with no errors         │
│ ✅ docker images shows a reasonable size          │
│ ✅ docker run starts without immediately exiting │
│ ✅ docker logs shows expected startup messages   │
│ ✅ The exposed port responds to a request        │
└─────────────────────────────────────────────────┘
```

### Backend Test

```bash
docker run -d --name test-backend -p 3000:3000 crowdfund-backend:dev
docker logs test-backend
# Expected: "Nest application successfully started" (or similar)
curl http://localhost:3000
docker rm -f test-backend
```

### Frontend Test

```bash
docker run -d --name test-frontend -p 8080:80 crowdfund-frontend:dev
docker logs test-frontend
# Expected: nginx startup logs, no errors
curl http://localhost:8080
docker rm -f test-frontend
```

### Common Mistakes
- ❌ Testing both containers wired together before confirming each works alone — makes it hard to know which one is broken
- ❌ Ignoring `docker logs` when a container exits immediately — the reason is always in there

### Best Practices
- ✅ Always test in isolation first (this phase), then integrate (Phase 9+)
- ✅ Use `--rm` when testing so you don't accumulate leftover containers: `docker run --rm -p 3000:3000 crowdfund-backend:dev`

---

## Phase 8 — Running Containers Individually 🟢

**Goal:** Understand the full manual `docker run` flow before Compose automates it — this cements *why* Compose exists (from Phase 6 of the notes above).

**Files needed:** Both images already built (Phase 7).

### Manual Run — Full Flags

```bash
# Backend, with environment variables set manually
docker run -d \
  --name crowdfund-backend \
  -p 3000:3000 \
  -e NODE_ENV=development \
  -e DATABASE_URL=postgresql://admin:secret@localhost:5432/crowdfund \
  crowdfund-backend:dev

# Frontend
docker run -d \
  --name crowdfund-frontend \
  -p 8080:80 \
  crowdfund-frontend:dev
```

### What You'll Notice (The Problem This Phase Reveals)

```
Backend tries to reach "localhost:5432" for the database.
"localhost" INSIDE the container refers to the CONTAINER ITSELF,
not your host machine, and not another container.

Result: connection refused ❌

This is intentional — it's the exact isolation-by-default behavior
from Phase 5 of the notes. Solved next, in Phase 9.
```

### Verification Steps
```bash
docker ps                    # both running?
docker logs crowdfund-backend  # look for the connection error described above
```

### Common Mistakes
- ❌ Expecting `localhost` to magically mean "my other container" — it never does
- ❌ Forgetting to clean up test containers between attempts (`docker rm -f crowdfund-backend crowdfund-frontend`)

### Best Practices
- ✅ Run this phase specifically to *feel* the networking problem before jumping to the solution — understanding beats memorizing


---

## Phase 9 — Connecting Containers With Docker Networking 🟢

**Goal:** Fix the connection problem from Phase 8 using a custom network — and understand exactly when networking is needed and when it isn't.

**Why this step exists:** Backend and database must talk to each other. Frontend and backend, in this architecture, do **not** need Docker networking between them — the browser (not the container) is what calls the backend, over a public URL.

### When You DO Need Docker Networking

```
Backend ──► Database
(NestJS container needs to reach PostgreSQL container directly)
→ YES, needs a shared custom network
```

### When You DON'T Need Docker Networking

```
Browser ──► Backend (via public port mapping, e.g. api.mycompany.com)
Browser ──► Frontend (via public port mapping, e.g. mycompany.com)

The frontend's JAVASCRIPT (running in the USER'S BROWSER, not in
a container) calls the backend's PUBLIC URL. The browser doesn't
know or care that Docker exists. No Docker network involved here.
```

```
VISUALIZING THE DISTINCTION
┌──────────────────────────────────────────────────────────────┐
│  User's Browser                                               │
│      │                                                        │
│      │  HTTPS to api.mycompany.com  (public internet)         │
│      ▼                                                        │
│  ┌─────────────┐        Docker custom network         ┌────┐ │
│  │  Backend    │ ─────────────────────────────────────►│ DB │ │
│  │  container  │  DATABASE_URL uses container name     │    │ │
│  └─────────────┘                                       └────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Setting Up the Backend ↔ Database Network

```bash
docker network create crowdfund-network

docker run -d \
  --name crowdfund-db \
  --network crowdfund-network \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=crowdfund \
  -v crowdfund-pg-data:/var/lib/postgresql/data \
  postgres:16-alpine

docker run -d \
  --name crowdfund-backend \
  --network crowdfund-network \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://admin:secret@crowdfund-db:5432/crowdfund \
  crowdfund-backend:dev
```

Note: `@crowdfund-db:5432` — the **container name**, not `localhost`, not an IP. This is exactly the DNS behavior from Phase 5 of the notes above.

### Verification Steps
```bash
docker network inspect crowdfund-network   # both containers listed?
docker exec -it crowdfund-backend sh
# from inside:
ping crowdfund-db     # should resolve and respond
exit
docker logs crowdfund-backend   # should show a successful DB connection now
```

### Common Mistakes
- ❌ Putting the frontend on the same Docker network as the backend and database, thinking it's "more connected" — unnecessary, the browser talks to the backend over its public URL, not through Docker's internal network
- ❌ Using the default bridge network instead of a custom one — no DNS, `crowdfund-db` wouldn't resolve

### Best Practices
- ✅ Only put containers that need direct container-to-container communication on the same custom network
- ✅ Database containers get **no `-p` flag** — only reachable from the backend via the network, never exposed to your machine or the internet directly (unless you specifically need a local DB GUI connection, in which case add `-p` temporarily for development only)

---

## Phase 10 — Managing Environment Variables for Development and Production 🟢

**Goal:** Handle configuration and secrets correctly across environments — critical, high-risk area from Phase 8 of the notes above.

**Files to create:** `.env`, `.env.example` in both repos.

### Backend `.env.example` (committed to git)

```bash
NODE_ENV=development
PORT=3000
POSTGRES_USER=
POSTGRES_PASSWORD=
POSTGRES_DB=crowdfund
DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@crowdfund-db:5432/${POSTGRES_DB}
JWT_SECRET=
```

### Backend `.env` (never committed — real values)

```bash
NODE_ENV=development
PORT=3000
POSTGRES_USER=admin
POSTGRES_PASSWORD=dev_only_secret_123
POSTGRES_DB=crowdfund
DATABASE_URL=postgresql://admin:dev_only_secret_123@crowdfund-db:5432/crowdfund
JWT_SECRET=dev_jwt_secret_change_in_prod
```

### The Frontend Env Variable Distinction (Important, Often Missed)

```
BACKEND env vars               FRONTEND env vars
──────────────────             ──────────────────
Read at RUNTIME                Baked in at BUILD TIME
Can change without rebuild     MUST rebuild image to change value
docker run -e / compose        Passed as --build-arg during docker build
  environment: works fine

Why: React code compiles into static JS files. Whatever URL
was set during `npm run build` is now PERMANENTLY inside those
JS files — there's no "runtime" for a static file to read from.
```

```dockerfile
# Frontend Dockerfile — accepting a build-time variable
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

# ARG must be declared before it's used
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL

RUN npm run build
# ... (production stage same as Phase 5)
```

```bash
# Building with a specific backend URL baked in
docker build --build-arg VITE_API_URL=https://api.crowdfund.com -t crowdfund-frontend:prod .
```

### `.gitignore` Addition (Both Repos)

```gitignore
.env
.env.local
.env.production
```

### Verification Steps
```bash
# Confirm .env is never tracked by git
git status   # .env should NOT appear
git check-ignore .env   # should print ".env" confirming it's ignored
```

### Common Mistakes
- ❌ Trying to change the frontend's API URL with a runtime `docker run -e` — has zero effect, because it's already baked into the JS bundle at build time
- ❌ Committing `.env` even once — it stays in git history forever even after later deletion (from Phase 8 notes)

### Best Practices
- ✅ Backend: runtime env vars, injected via `-e` or Compose `environment:`
- ✅ Frontend: build-time env vars, injected via `--build-arg` or Compose `build.args:`


---

## Phase 11 — Using Volumes (When To, and When Not To) 🟢

**Goal:** Apply Phase 4 of the notes above to this real project — persist the database, live-reload the backend in dev, and correctly avoid volumes where they'd cause harm.

### Decision Table for This Project

| Data / Folder | Use a Volume? | Why |
|---|---|---|
| PostgreSQL data | ✅ Named volume | Must survive container restarts — it's the actual user data |
| Backend `src/` (dev only) | ✅ Bind mount | Live reload while coding |
| Backend `node_modules` | ✅ Named volume (protects it) | Prevents host OS binaries overwriting container's Linux binaries |
| Frontend `dist/` (production) | ❌ No volume | Built once during image build, baked into the image — nothing to persist |
| Backend `dist/` (production) | ❌ No volume | Same — compiled once at build time |
| `.env` files | ❌ Never a volume, never in the image | Injected as environment variables, not files |

### Applying It

```bash
# Database — named volume, survives everything
docker volume create crowdfund-pg-data

docker run -d \
  --name crowdfund-db \
  --network crowdfund-network \
  -v crowdfund-pg-data:/var/lib/postgresql/data \
  postgres:16-alpine

# Backend in DEV — bind mount for live reload + protected node_modules
docker run -d \
  --name crowdfund-backend \
  --network crowdfund-network \
  -p 3000:3000 \
  -v $(pwd)/src:/app/src \
  -v crowdfund-backend-node-modules:/app/node_modules \
  crowdfund-backend:dev
```

### Why NOT Bind-Mount the Whole Project in Dev

```
❌ -v $(pwd):/app
   Overwrites the container's Linux node_modules with your
   host machine's node_modules (built for macOS/Windows) —
   native modules break immediately.

✅ -v $(pwd)/src:/app/src
   Only your actual source code syncs. node_modules stays
   a separate, protected, Linux-native volume.
```

### Verification Steps
```bash
docker volume ls                     # crowdfund-pg-data exists
docker exec -it crowdfund-db psql -U admin -d crowdfund -c "\dt"   # tables listed
docker rm -f crowdfund-db
docker run -d --name crowdfund-db --network crowdfund-network \
  -v crowdfund-pg-data:/var/lib/postgresql/data postgres:16-alpine
docker exec -it crowdfund-db psql -U admin -d crowdfund -c "\dt"
# Same tables still there → volume worked ✅
```

### Common Mistakes
- ❌ Bind-mounting the entire backend folder including `node_modules` in dev
- ❌ Forgetting a volume for PostgreSQL — losing the entire database on every `docker compose down`

### Best Practices
- ✅ Volumes for anything that must survive a container's death
- ✅ Bind mounts only for source code you're actively editing
- ✅ No volumes at all in the production image — it's baked and immutable


---

## Phase 12 — Writing a Complete `docker-compose.yml` for Local Development 🟢

**Goal:** Replace every manual command from Phases 8–11 with a single file (applies Phase 6 of the notes above).

**Files to create:** `crowdfund-backend/docker-compose.yml`

> Since frontend and backend are separate repos and don't need Docker networking between them (Phase 9), the backend's Compose file includes the database. The frontend gets its own, simpler Compose file.

### Backend `docker-compose.yml` (Development)

```yaml
services:
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pg-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s
    # No ports: exposed — only the backend needs to reach it

  backend:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      JWT_SECRET: ${JWT_SECRET}
    volumes:
      - ./src:/app/src               # bind mount for live reload
      - backend-node-modules:/app/node_modules   # protect from host overwrite
    depends_on:
      db:
        condition: service_healthy   # wait until postgres is truly ready

volumes:
  pg-data:
  backend-node-modules:
```

### Frontend `docker-compose.yml` (Development)

```yaml
services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        VITE_API_URL: ${VITE_API_URL}
    restart: unless-stopped
    ports:
      - "8080:80"
    # No volumes in this simple setup — a rebuild is cheap for a static frontend.
    # For live reload during frontend dev, many teams instead run
    # `npm run dev` directly on the host and only use Docker for the
    # production nginx build — a valid and common approach.
```

### Why the Frontend Compose File Is Simpler

```
Backend needs:                     Frontend needs:
- Its database (in the same        - Nothing else in the stack
  Compose file, same network)      - Just itself, built and served
- Live reload via bind mount       - A rebuild per change is fine
- depends_on + healthcheck            (or run `npm run dev` on host
                                        for the fastest dev loop)
```

### Verification Steps
```bash
cd crowdfund-backend
docker compose config    # validates the YAML, shows resolved values — catch typos early
```

### Common Mistakes
- ❌ Forgetting `condition: service_healthy` — backend starts before Postgres is ready to accept connections (Phase 6 notes warning)
- ❌ Quoting port numbers incorrectly, or forgetting quotes — `"3000:3000"` needs the quotes in YAML

### Best Practices
- ✅ Keep each repo's Compose file scoped to what that repo actually needs
- ✅ Use `.env` + `${VARIABLE}` syntax — never hardcode passwords in the YAML itself

---

## Phase 13 — Running the Complete Application With Docker Compose 🟢

**Goal:** Bring the full backend + database stack up with one command, and run the frontend alongside it.

### Backend + Database

```bash
cd crowdfund-backend
cp .env.example .env        # fill in real dev values
docker compose up -d --build
```

### Expected Output

```
[+] Running 3/3
 ✔ Network crowdfund-backend_default   Created
 ✔ Container crowdfund-backend-db-1    Healthy
 ✔ Container crowdfund-backend-backend-1  Started
```

### Frontend

```bash
cd ../crowdfund-frontend
cp .env.example .env
docker compose up -d --build
```

### Full System Diagram (What's Now Running)

```
┌──────────────────────────────────────────────────────────┐
│  crowdfund-backend_default network                       │
│  ┌───────────┐         ┌────────────┐                    │
│  │  backend  │────────►│     db     │                    │
│  │  :3000    │         │  (internal)│                    │
│  └─────┬─────┘         └────────────┘                    │
└────────┼───────────────────────────────────────────────┘
         │ exposed via -p 3000:3000
         ▼
   http://localhost:3000  ← your browser/Postman hits this

┌──────────────────────────────────────────────────────────┐
│  crowdfund-frontend_default network                      │
│  ┌───────────┐                                            │
│  │ frontend  │                                            │
│  │  :80      │                                            │
│  └─────┬─────┘                                            │
└────────┼───────────────────────────────────────────────┘
         │ exposed via -p 8080:80
         ▼
   http://localhost:8080  ← your browser loads the React app,
                              which then calls localhost:3000 directly
```

### Verification Steps
```bash
docker compose ps                       # (in each repo) all services "Up" / "healthy"
curl http://localhost:3000/health       # or whatever your health endpoint is
open http://localhost:8080              # frontend loads, and successfully calls backend
```

### Common Mistakes
- ❌ Running `docker compose up` without `--build` after changing the Dockerfile — old image keeps running (Phase 6 notes, #1 mistake)
- ❌ Forgetting `cp .env.example .env` — Compose fails with empty variable warnings


---

## Phase 14 — Running Automated Tests Inside Containers 🟢 (dedicated test stack is 🟡)

**Goal:** Run your NestJS test suite inside the same environment it will run in production — eliminating "tests pass locally, fail in CI" surprises.

**Prerequisites:** The dev stack from Phase 13 is up and running (`docker compose up -d`), and you already have `test` / `test:e2e` scripts in your `package.json` (standard in any NestJS project).

**Why this step exists:** If tests run on your host machine but the app runs in a container, you're testing two different environments. Testing inside the container closes that gap completely.

### Unit Tests — Using the Stack You Already Have

```bash
docker compose exec backend npm run test
```

```
docker compose exec  backend  npm run test
       │              │            │
       │              │            └── your existing NestJS test script
       │              └── the service name from docker-compose.yml
       └── run a command inside an ALREADY RUNNING container
```

### Integration Tests (Against the Real Database)

```bash
docker compose exec backend npm run test:e2e
```

This is meaningful specifically *because* the backend container is already networked to the real Postgres container (Phase 9) — integration tests hit an actual database, not a mock. For local development, running against your existing dev database this way is enough — just be aware it will contain whatever dev data you've already created.

### Verifying Frontend-Backend Communication

```bash
# With both stacks running (Phase 13):
curl -X POST http://localhost:3000/api/campaigns \
  -H "Content-Type: application/json" \
  -d '{"title":"Test Campaign"}'

# Then confirm the frontend can see it:
# open http://localhost:8080 and check the campaign appears in the UI
```

### Verifying Database Connectivity

```bash
docker compose exec backend sh
# inside the container:
node -e "require('./dist/main.js')"   # or a small script that queries the DB
exit
```

### Expected Output (Test Run)

```
PASS src/app.controller.spec.ts
PASS src/campaigns/campaigns.service.spec.ts

Test Suites: 2 passed, 2 total
Tests:       8 passed, 8 total
```

### Common Mistakes
- ❌ Confusing `docker compose exec` (needs a running container) with `docker compose run` (starts a new one) — `exec` is what you want here since the stack is already up

---

### 🟡 Advanced/Optional — A Dedicated, Ephemeral Test Stack

**Prerequisites:** The commands above already work for you locally.

The limitation of testing against your dev database: results depend on whatever data already exists there, and tests could corrupt it. CI/CD pipelines solve this with a separate, throwaway database that starts empty every single run. You don't need this for local development — it becomes genuinely useful in Phase 19, where it runs automatically on every push with zero manual setup. The full example is built there, in context, rather than duplicated here.

### Best Practices
- ✅ For local dev: testing against your existing dev stack (as shown above) is completely fine
- ✅ For CI/CD: a dedicated ephemeral test database (Phase 19) avoids flaky, data-dependent results

---

## Phase 15 — Common Debugging Techniques and Troubleshooting 🟢

**Goal:** Build a systematic debugging workflow instead of guessing.

### The Debugging Decision Tree

```
Container won't start / exits immediately
  │
  ▼
docker compose logs backend
  │
  ├── Shows a clear error (missing env var, syntax error)
  │       → fix the code/config, rebuild
  │
  └── No useful output / exits before logging
          → docker compose run --rm backend sh
            (overrides CMD, drops you into a shell instead)
            → manually run the start command to see the real error:
              node dist/main.js
```

```
Container runs but app doesn't respond
  │
  ▼
docker compose ps
  │
  ├── Status shows "Up" → check port mapping
  │       docker port backend   (confirms host:container mapping)
  │       curl the correct port
  │
  └── Status shows "Restarting" → crash loop
          → docker compose logs backend --tail 50
```

```
Backend can't reach database
  │
  ▼
docker compose exec backend sh
  │
  ▼
ping db          (does DNS resolve? — confirms same network)
  │
  ├── Fails → not on the same Compose network (shouldn't happen if
  │           using one docker-compose.yml, but check docker network ls)
  │
  └── Works → check DATABASE_URL credentials/format next
```

### Quick Reference Commands

```bash
docker compose logs -f backend           # live logs, most common first move
docker compose ps                        # status of every service
docker compose exec backend sh           # get inside a running container
docker compose run --rm backend sh       # start fresh, override CMD, debug startup itself
docker inspect crowdfund-backend-backend-1 --format '{{json .State}}'  # exact state/exit code
```

### Common Mistakes
- ❌ Rebuilding repeatedly without reading `docker compose logs` first — the answer is almost always in the logs
- ❌ Debugging networking issues by checking IPs instead of testing DNS resolution directly (`ping <service-name>`)

### Best Practices
- ✅ `docker compose logs` is always your first move, before anything else
- ✅ `docker compose run --rm <service> sh` is the most powerful tool for "why won't this even start" problems — it drops you into the container's shell so you can run the failing command manually and see the real error

---

## Phase 16 — Optimizing the Development Workflow 🟡

**Goal:** Reduce daily friction so Docker feels fast, not like overhead.

### Speeding Up Rebuilds

```
Apply the caching order from Phase 2 of the notes:
COPY package*.json first, install, THEN copy source code.
Already done in our Dockerfiles (Phase 4, 5) — verify it stayed that way.
```

### A Helper Script (Optional but Practical)

**Files to create:** `crowdfund-backend/scripts/dev.sh`

```bash
#!/bin/bash
# Quick daily dev commands, wrapped for convenience

case "$1" in
  up)
    docker compose up -d --build
    ;;
  logs)
    docker compose logs -f backend
    ;;
  test)
    docker compose exec backend npm run test
    ;;
  shell)
    docker compose exec backend sh
    ;;
  down)
    docker compose down
    ;;
  *)
    echo "Usage: ./scripts/dev.sh {up|logs|test|shell|down}"
    ;;
esac
```

```bash
chmod +x scripts/dev.sh
./scripts/dev.sh up
./scripts/dev.sh logs
```

### Rebuilding Only What Changed

```bash
docker compose up -d --build backend   # only rebuilds the backend service,
                                        # db keeps running untouched
```

### Common Mistakes
- ❌ Running `docker compose down` (full teardown) for every small code change — `--build` on the specific service is faster and keeps the database running
- ❌ Not using `.dockerignore` correctly — slow build context transfer on every single build (Phase 6)

### Best Practices
- ✅ Keep the database running between backend rebuilds — `docker compose up -d --build backend` instead of tearing everything down
- ✅ A short wrapper script (or Makefile) removes the need to remember long commands daily


---

## Phase 17 — Preparing Production-Ready Docker Images 🟢

**Goal:** Turn the development Dockerfiles into lean, secure, production-grade images.

**Why this step exists:** The dev Dockerfile prioritizes fast iteration (bind mounts, dev dependencies). Production prioritizes small size, security, and immutability.

### Production Backend Dockerfile (Multi-Stage)

```dockerfile
# ── STAGE 1: Build ───────────────────────────────────────────
FROM node:20-alpine AS build

WORKDIR /app

COPY package*.json ./
# Install ALL dependencies (including devDependencies) — needed to build
RUN npm ci

COPY . .
RUN npm run build

# ── STAGE 2: Production ──────────────────────────────────────
FROM node:20-alpine AS production

WORKDIR /app

# Only production dependencies — smaller, faster, safer
COPY package*.json ./
RUN npm ci --omit=dev

# Copy ONLY the compiled output from the build stage
COPY --from=build /app/dist ./dist

ENV NODE_ENV=production
EXPOSE 3000

# Run as a non-root user for security
USER node

CMD ["node", "dist/main.js"]
```

### Why Each Production Change Matters

```
CHANGE                          WHY
─────────────────────────────   ──────────────────────────────────
Multi-stage build               Build tools + devDependencies never
                                 reach the final image (smaller, safer)

npm ci --omit=dev               Skips packages only needed for
                                 building/testing, not running

USER node                       Container runs as a non-root user —
                                 if compromised, attacker has limited
                                 permissions inside the container

No bind mounts                  Code is baked into the image —
                                 immutable, exactly what was tested
```

### Production Frontend Dockerfile

Already production-ready from Phase 5 (multi-stage, nginx-served). One addition — a custom nginx config for a Single Page App:

**Files to create:** `crowdfund-frontend/nginx.conf`

```nginx
server {
    listen 80;

    root /usr/share/nginx/html;
    index index.html;

    # React Router needs this: any unmatched route serves index.html
    # so client-side routing works on page refresh
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

```dockerfile
# Add to the production stage of the frontend Dockerfile:
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

### Tagging for Production

```bash
# Applying Phase 8 notes: git SHA tagging, never "latest" in production
TAG=$(git rev-parse --short HEAD)

docker build -t crowdfund-backend:$TAG .
docker build --build-arg VITE_API_URL=https://api.crowdfund.com \
  -t crowdfund-frontend:$TAG ./crowdfund-frontend
```

### Verification Steps
```bash
docker images | grep crowdfund   # compare sizes: production images should be
                                  # noticeably smaller than :dev versions
docker run --rm crowdfund-backend:$TAG node -e "console.log('works')"
```

### Common Mistakes
- ❌ Using the dev Dockerfile in production — ships devDependencies and bind-mount assumptions that don't apply
- ❌ Running the production container as root — unnecessary security risk
- ❌ Forgetting `try_files` in nginx config — React Router routes 404 on direct URL access/refresh

### Best Practices
- ✅ Always multi-stage in production
- ✅ Always run as a non-root user when the base image supports it
- ✅ Tag with git SHA, never `latest` (Phase 8 notes)

---

## Phase 18 — Writing a Production Docker Compose 🟢

**Goal:** Define how the production stack runs — used for single-server deployment (matches your EC2 setup pattern).

**Files to create:** `crowdfund-backend/docker-compose.prod.yml`

```yaml
services:
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pg-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      retries: 5
    # Never exposed with -p in production — internal only

  backend:
    image: ghcr.io/mycompany/crowdfund-backend:${IMAGE_TAG:-latest}
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      db:
        condition: service_healthy
    # No volumes: code is baked into the image, immutable

volumes:
  pg-data:
```

### Key Differences From the Dev Compose File

```
DEV                                   PRODUCTION
──────────────────────────            ──────────────────────────
build: .                              image: ghcr.io/.../backend:$TAG
  (builds locally every time)           (pulls a pre-built, tested image)

Bind mount for live reload            No bind mounts — code is fixed
  -v ./src:/app/src                     inside the image

No specific tag                       Specific git-SHA tag, traceable
```

### Running It on the Server

```bash
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
```

### Common Mistakes
- ❌ Using `build:` in the production Compose file — forces the server to build the image itself instead of pulling a pre-tested one from the registry (Phase 8 notes: this is the git-pull EC2 pattern trade-off)
- ❌ Forgetting `restart: unless-stopped` — a server reboot won't bring the app back up automatically

### Best Practices
- ✅ Production Compose pulls a pre-built image — the server's only job is running containers, not building them
- ✅ `${IMAGE_TAG:-latest}` pattern lets your CI/CD pipeline control exactly which version deploys


---

## Phase 19 — Building a Complete CI/CD Pipeline 🟢

**Goal:** Automate everything from Phases 4–18 so it happens on every push, with zero manual steps — the full application of Phase 8 of the notes above.

**Prerequisites:**
- A GitHub repository for each project, code already pushed
- A GitHub Container Registry (GHCR) — nothing to set up in advance, it activates automatically the first time you push an image
- An EC2 server reachable over SSH (built in Phase 20 — if you haven't done that yet, you can still write and test the `test` job below; the `deploy` job needs the server to exist first)

**Why CI/CD matters here specifically:** Without it, every deploy means: you manually build the backend image, manually build the frontend image, manually SSH into the server, manually pull, manually restart, and hope you didn't forget a step. One missed step = broken production.

### The Overall Workflow

```
Developer pushes code
        │
        ▼
GitHub Actions triggers automatically
        │
        ├── LINT & TEST (every push and PR)
        │     Run ESLint
        │     Run unit tests inside a container
        │     Run integration tests against a real ephemeral DB
        │
        ├── BUILD & PUSH (only on merge to main)
        │     Build the production image (multi-stage, Phase 17)
        │     Tag with git SHA
        │     Push to GitHub Container Registry
        │
        └── DEPLOY (only after successful push)
              SSH into the production server
              Pull the new image
              Restart with zero-downtime intent
              Verify it's healthy
```

### The Ephemeral Test Stack This Pipeline Uses

Referenced back in Phase 14 as a 🟡 advanced pattern — here is where it earns its keep, because CI needs a database that starts empty on every single run, with zero manual setup.

**Files to create:** `crowdfund-backend/docker-compose.test.yml`

```yaml
services:
  db-test:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_pass
      POSTGRES_DB: test_db
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U test_user -d test_db"]
      interval: 5s
      retries: 5

  backend-test:
    build: .
    environment:
      NODE_ENV: test
      DATABASE_URL: postgresql://test_user:test_pass@db-test:5432/test_db
    depends_on:
      db-test:
        condition: service_healthy
    command: npm run test:e2e
```

> No named volumes here on purpose — the database is meant to be thrown away after each run (Phase 6 notes guidance on ephemeral vs persistent data).

### Backend CI/CD — `.github/workflows/backend.yml`

**Files to create:** `crowdfund-backend/.github/workflows/backend.yml`

```yaml
name: Backend CI/CD

# Triggers: what starts this workflow
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:

  # ── JOB 1: Lint and Test ─────────────────────────────────────
  # Runs on EVERY push and EVERY pull request
  test:
    name: Lint & Test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Start test stack
        run: docker compose -f docker-compose.test.yml up -d --build

      - name: Wait for database to be healthy
        run: docker compose -f docker-compose.test.yml ps

      - name: Run lint
        run: docker compose -f docker-compose.test.yml exec -T backend-test npm run lint

      - name: Run unit tests
        run: docker compose -f docker-compose.test.yml exec -T backend-test npm run test

      - name: Run integration tests
        run: docker compose -f docker-compose.test.yml exec -T backend-test npm run test:e2e

      - name: Tear down (always, even on failure)
        if: always()
        run: docker compose -f docker-compose.test.yml down --volumes

  # ── JOB 2: Build and Push ────────────────────────────────────
  # Only runs after tests pass, and only on a real merge to main
  build-and-push:
    name: Build & Push Image
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push production image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ghcr.io/${{ env.IMAGE_NAME }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ── JOB 3: Deploy ─────────────────────────────────────────────
  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy over SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USER }}
          key: ${{ secrets.PROD_SSH_PRIVATE_KEY }}
          script: |
            set -e
            cd /opt/crowdfund-backend
            export IMAGE_TAG=${{ github.sha }}
            docker compose -f docker-compose.prod.yml pull
            docker compose -f docker-compose.prod.yml up -d
            docker compose -f docker-compose.prod.yml ps
            docker image prune -f
```

### Explaining Every Section

```
on: push/pull_request      → triggers: when the workflow STARTS
env:                       → shared values every job can reference
jobs.test                  → no "if:" — runs on both PRs and merges
jobs.build-and-push        → "needs: test" — waits for tests to pass
                              "if: github.ref == main" — skips on PRs
jobs.deploy                → "needs: build-and-push" — waits for the image
                              to actually exist in the registry first
```

This chain — `test → build-and-push → deploy` — is the exact `needs:` dependency pattern that guarantees nothing broken ever reaches production.

### Frontend CI/CD — `.github/workflows/frontend.yml`

**Files to create:** `crowdfund-frontend/.github/workflows/frontend.yml`

```yaml
name: Frontend CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name: Lint & Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run lint
      - run: npm run test

  build-and-push:
    name: Build & Push Image
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          build-args: |
            VITE_API_URL=${{ secrets.PROD_API_URL }}
          tags: |
            ghcr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ghcr.io/${{ env.IMAGE_NAME }}:latest

  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_SERVER_HOST }}
          username: ${{ secrets.PROD_SERVER_USER }}
          key: ${{ secrets.PROD_SSH_PRIVATE_KEY }}
          script: |
            cd /opt/crowdfund-frontend
            export IMAGE_TAG=${{ github.sha }}
            docker compose -f docker-compose.prod.yml pull
            docker compose -f docker-compose.prod.yml up -d
```

### Note: `VITE_API_URL` as a Build Argument in CI

This directly applies Phase 10 of this tutorial — the frontend URL must be baked in *during the build*, so it's passed as a `build-args` value in the GitHub Actions build step, sourced from a GitHub Actions secret, not a runtime environment variable.

### Setting Up GitHub Actions Secrets

```
For EACH repo, go to:
GitHub repo → Settings → Secrets and variables → Actions → New repository secret

Required secrets (backend repo):
  PROD_SERVER_HOST         → your EC2 public IP or domain
  PROD_SERVER_USER         → SSH username (e.g. ubuntu)
  PROD_SSH_PRIVATE_KEY     → the private key matching the server's authorized_keys

Required secrets (frontend repo):
  PROD_SERVER_HOST
  PROD_SERVER_USER
  PROD_SSH_PRIVATE_KEY
  PROD_API_URL             → e.g. https://api.crowdfund.com

GITHUB_TOKEN is provided automatically — never add it manually.
```

### Rolling Out Updates

```
Every merge to main = a new deploy, automatically:
1. New image built and tagged with the new commit SHA
2. Server pulls that specific tag
3. Old container replaced with new one
4. docker image prune -f cleans up the old image afterward
```

### Rollback Basics

```bash
# If a deployment breaks something, roll back to the previous known-good SHA:
ssh user@server
cd /opt/crowdfund-backend
export IMAGE_TAG=<previous-working-git-sha>
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
```

```
WHY THIS WORKS
──────────────────────────────────────────────────────
Every image is tagged with its exact git commit SHA (Phase 17).
The registry keeps every previously pushed version.
Rolling back is just: point the server at an OLDER tag.
No rebuilding. No guessing. Seconds, not hours.
```

### Verification Steps
```bash
# After pushing to a feature branch and opening a PR:
# → Check the "Checks" tab on the PR — test job should run and pass

# After merging to main:
# → Check the "Actions" tab — all 3 jobs should run in sequence
# → Check GHCR (Packages tab on your GitHub profile/org) —
#   new image tag should appear

# On the server:
ssh user@server "docker compose -f /opt/crowdfund-backend/docker-compose.prod.yml ps"
```

### Common Mistakes
- ❌ Missing `needs:` between jobs — jobs could run out of order or in parallel when they shouldn't
- ❌ Forgetting `if: always()` on the test teardown step — a failed test leaves containers running forever in the CI runner
- ❌ Putting the frontend's API URL as a runtime secret instead of a build arg — has no effect, per Phase 10

### Best Practices
- ✅ Separate `test`, `build-and-push`, and `deploy` into distinct jobs with `needs:` — a failure at any stage stops the pipeline immediately
- ✅ Tag every image with the git SHA — this is what makes rollback possible in seconds
- ✅ Never let PRs trigger a deploy — only merges to `main` should reach production (`if: github.ref == 'refs/heads/main'`)


---

## Phase 20 — Deploying the Application to AWS 🟢 (HTTPS/reverse proxy is 🟡)

**Goal:** Get the app running on a real server, reachable over plain HTTP. That's a complete, working deployment on its own — HTTPS is a valuable upgrade you add once this works, not a requirement to get there.

**Prerequisites:**
- An AWS account with a payment method attached (EC2 isn't free past the trial tier)
- The images from Phase 17 already being pushed successfully to GHCR by your CI/CD pipeline, or pushed manually once to confirm the registry works
- An SSH key pair you can use to access the instance (AWS generates one for you when you launch it)

### Choosing the AWS Service

```
OPTION                    WHEN TO USE
────────────────────      ──────────────────────────────────────────
EC2 (what we use)         Full control, matches your existing EC2
                           experience, simplest mental model for a
                           single-server Docker Compose deployment

ECS/Fargate                When you outgrow one server — managed
                           container orchestration, more moving parts

Elastic Beanstalk          Simpler PaaS-style deploys, less control

App Runner                 Simplest option, but less flexible for
                           multi-container setups like ours
```

**We use EC2** — it directly matches Phase 9 of the notes ("Docker Compose is for one machine") and is the most beginner-friendly path to understanding what's actually happening, rather than hiding it behind a managed service.

### Server Setup

```
STEP-BY-STEP
──────────────────────────────────────────────────────
1. Launch an EC2 instance
   - Ubuntu 22.04 LTS
   - t3.small or larger (t3.micro often too small for Postgres + Node)
   - Open ports in the Security Group: 22 (SSH), 80 (HTTP)
     — 443 only needed later, if/when you add HTTPS

2. SSH into it
   ssh -i your-key.pem ubuntu@<ec2-public-ip>
```

### Installing Docker and Docker Compose

```bash
sudo apt update && sudo apt upgrade -y

curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

sudo usermod -aG docker $USER
newgrp docker

docker --version
docker compose version
```

### Directory Setup on the Server

```bash
sudo mkdir -p /opt/crowdfund-backend /opt/crowdfund-frontend
sudo chown -R $USER:$USER /opt/crowdfund-backend /opt/crowdfund-frontend

cd /opt/crowdfund-backend
git clone https://github.com/yourorg/crowdfund-backend.git .
# (or just copy docker-compose.prod.yml — the server only needs
#  the Compose file, since it PULLS pre-built images, never builds)
```

### Managing Environment Variables Securely on the Server

```bash
# On the server — NOT in git, created manually
cd /opt/crowdfund-backend
nano .env
```

```bash
# .env on the SERVER (real production secrets)
POSTGRES_USER=produser
POSTGRES_PASSWORD=<strong-random-password>
POSTGRES_DB=crowdfund
JWT_SECRET=<strong-random-secret>
IMAGE_TAG=latest
```

```
This matches Phase 8 of the notes exactly:
Secrets live on the SERVER's .env, never in git, never in the image.
docker compose -f docker-compose.prod.yml up -d reads this file
and injects the values into the running containers.
```

### First Manual Deployment (Before CI/CD Takes Over)

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u yourusername --password-stdin

cd /opt/crowdfund-backend
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d

cd /opt/crowdfund-frontend
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
```

After this first manual run works, every future deploy is handled automatically by the GitHub Actions pipeline from Phase 19 — this manual step is done once to confirm the server itself is correctly configured.

### Verification Steps — Plain HTTP (This Is a Working Deployment)

```bash
curl http://<ec2-public-ip>:3000/health     # backend reachable
curl http://<ec2-public-ip>:8080            # frontend reachable
docker compose -f docker-compose.prod.yml ps   # all services healthy
```

At this point the application is live and working. Everything below is a genuine improvement, not a missing piece.

---

### 🟡 Advanced/Optional — Domain Name and HTTPS

**Prerequisites:** The plain-HTTP deployment above is already working and verified.

**Domain:** buy one (Route 53, Namecheap, etc.), then point an A record at your EC2 instance's public IP — `crowdfund.com → <EC2 IP>`.

**HTTPS, at a glance:** browsers expect the padlock, and some frontend features require it. The concept is simple even though the setup has a few moving parts — a small proxy sits in front of your app, handles the certificate, and forwards plain HTTP internally:

```
Internet ──HTTPS──► Reverse Proxy ──HTTP (internal)──► Your Containers
                     (handles the cert)
```

The beginner-friendly option is **Caddy** — it's a single container that fetches and renews Let's Encrypt certificates automatically, needing only a couple of lines of config (unlike nginx, which needs manual certificate setup). When you're ready to add it: add a `caddy:alpine` service to your production Compose file, point it at your domain in a short `Caddyfile`, and open port 443 in your Security Group. This is a self-contained upgrade you can pursue later — it doesn't change anything about the backend, frontend, or database containers themselves.

---

### Updating With Minimal Downtime

```
The CI/CD pipeline (Phase 19) already does this correctly:
  docker compose pull    → downloads new image WHILE old one still runs
  docker compose up -d   → Compose stops old container, starts new one
                            (a few seconds of downtime — acceptable for
                             a single-server setup; true zero-downtime
                             requires orchestration, Phase 9 of the notes)
```

### Basic Monitoring and Log Inspection

```bash
docker compose -f docker-compose.prod.yml logs -f backend   # live logs
docker compose -f docker-compose.prod.yml ps                # health status
docker stats                                                  # live CPU/RAM per container
df -h                                                          # disk space on the server
```

### Common Production Mistakes

```
❌ Using a t3.micro for Postgres + Node — runs out of RAM under load
❌ Never rotating logs — docker logs grow unbounded, fill the disk
❌ Storing the SSH private key insecurely instead of in GitHub Secrets
❌ Running database migrations manually and forgetting to automate them
   in the CI/CD pipeline for future deploys
❌ Not setting resource limits — one runaway container can starve others
   on the same server (this is exactly what cgroups solve — Phase 0 notes)
```

---

## Phase 21 — Verifying the Deployment 🟢

**Goal:** Confirm the entire system works end-to-end in production, not just that containers are "Up".

> URLs below assume plain HTTP (`http://<ec2-ip>`). If you've added the optional domain + HTTPS from Phase 20, swap in `https://yourdomain.com` instead — everything else is identical.

### The Full Verification Checklist

```
┌──────────────────────────────────────────────────────────────┐
│ ✅ Backend container is "Up" and healthy                      │
│     docker compose -f docker-compose.prod.yml ps              │
│                                                                │
│ ✅ Database container is "Up" and healthy                     │
│     Same command — check the healthcheck status               │
│                                                                │
│ ✅ Backend responds                                           │
│     curl http://<ec2-ip>:3000/health                          │
│                                                                │
│ ✅ Frontend loads                                             │
│     curl -I http://<ec2-ip>:8080  (expect 200 OK)              │
│                                                                │
│ ✅ Frontend can successfully call the backend                 │
│     Open the site in a real browser, check Network tab        │
│     for successful API calls (not CORS errors, not 502s)      │
│                                                                │
│ ✅ Data actually persists                                     │
│     Create a record via the UI, restart the backend            │
│     container, confirm the record is still there               │
│                                                                │
│ ✅ CI/CD pipeline shows green on the latest commit             │
│     Check the Actions tab — all 3 jobs passed                  │
└──────────────────────────────────────────────────────────────┘
```

### End-to-End Test Script (🟡 Optional but Useful)

```bash
#!/bin/bash
# verify-deployment.sh — run after every deploy

set -e
HOST=${1:-http://<ec2-ip>}   # pass your domain here once you have one

echo "Checking backend health..."
curl -sf $HOST:3000/health || { echo "❌ Backend down"; exit 1; }

echo "Checking frontend..."
curl -sf -o /dev/null $HOST:8080 || { echo "❌ Frontend down"; exit 1; }

echo "✅ All checks passed"
```

You can add this as a final step in the `deploy` job of Phase 19's pipeline — if it fails, you know immediately, from inside the pipeline itself, rather than discovering it from a user complaint.

### Common Mistakes
- ❌ Only checking `docker compose ps` and assuming "Up" means "working" — a container can be running while the app inside it is broken
- ❌ Never testing the actual browser experience — CORS issues and frontend-backend miscommunication only show up here

---

## Phase 22 — Monitoring, Updating, and Maintaining the Deployed Application 🟡

**Goal:** Keep the application healthy long after the first successful deploy.

### Daily/Weekly Habits

```bash
# Check disk usage — Docker silently accumulates unused layers (Phase 8 notes)
docker system df

# Clean up periodically
docker system prune -f

# Check running containers and their resource usage
docker stats --no-stream

# Tail recent logs for errors
docker compose -f docker-compose.prod.yml logs --tail 100 backend
```

### Handling Database Migrations on Deploy

```
Add a migration step to the deploy job (Phase 19), run BEFORE
restarting the backend with new code:

script: |
  cd /opt/crowdfund-backend
  docker compose -f docker-compose.prod.yml pull
  docker compose -f docker-compose.prod.yml run --rm backend npm run migration:run
  docker compose -f docker-compose.prod.yml up -d
```

```
WHY "run --rm" AND NOT "exec"?
────────────────────────────────────────────────────
"exec" requires the container to ALREADY be running.
"run --rm" starts a temporary, one-off container from the
NEW image, runs the migration, then removes itself —
perfect for a task that should run once, not persist
(this connects back to Phase 2 of this tutorial: migrations
are a "run once" task, not a persistent service).
```

### Backup Strategy for the Database

```bash
# Simple daily backup via cron on the server
docker compose -f docker-compose.prod.yml exec -T db \
  pg_dump -U produser crowdfund > /opt/backups/crowdfund-$(date +%F).sql

# Restore, if ever needed
cat /opt/backups/crowdfund-2026-06-01.sql | \
  docker compose -f docker-compose.prod.yml exec -T db psql -U produser crowdfund
```

> Remember from Phase 4 of the notes: **volumes are not a backup strategy on their own.** If the server's disk fails, the volume is gone too. Regular `pg_dump` backups, stored off the server (e.g., S3), are what actually protects your data.

### Monitoring Beyond the Basics (Where to Go Next)

```
This tutorial covers manual log/stat inspection — enough for a
single-server, small-to-medium project. As the project grows,
the natural next steps (not required to start) are:

- Centralized logging (e.g., shipping docker logs to a service)
- Uptime monitoring (an external service pinging your health endpoint)
- Alerting (get notified before users report problems)

These are additions on TOP of what you've built here — the
foundation (Docker, Compose, CI/CD, AWS) doesn't change.
```

### Common Mistakes
- ❌ Never running `docker system prune` — the server's disk eventually fills up entirely
- ❌ No backup strategy — treating the Postgres volume as if it were indestructible
- ❌ Running migrations manually and forgetting to automate them — the next deploy overwrites code but the database schema is now out of sync

### Best Practices
- ✅ Automate migrations as a `run --rm` step in the deploy pipeline
- ✅ Schedule regular database backups, stored somewhere other than the server itself
- ✅ Periodic `docker system prune` — either manually or as a scheduled cron job

---

## Tutorial Summary — What You Just Built

```
crowdfund-backend/                    crowdfund-frontend/
├── Dockerfile (multi-stage)          ├── Dockerfile (multi-stage)
├── docker-compose.yml (dev)          ├── docker-compose.yml (dev)
├── docker-compose.test.yml           ├── nginx.conf
├── docker-compose.prod.yml           ├── docker-compose.prod.yml
├── .dockerignore                     ├── .dockerignore
├── .env.example                      ├── .env.example
└── .github/workflows/backend.yml     └── .github/workflows/frontend.yml

           │                                      │
           └──────────────┬───────────────────────┘
                          ▼
              Both deploy independently to
              the same EC2 server, via their
              own CI/CD pipelines, pulling
              from their own registry images
```

### The Full Journey, End to End

```
Write code
    ↓
Dockerize it (Dockerfile, .dockerignore)
    ↓
Run it locally (docker run, then docker compose)
    ↓
Connect services correctly (networking, only where needed)
    ↓
Persist what must survive (volumes, only where needed)
    ↓
Test inside containers (same environment as production)
    ↓
Build lean production images (multi-stage)
    ↓
Automate everything (CI/CD: test → build → push → deploy)
    ↓
Deploy to a real server (AWS EC2, Docker Compose)
    ↓
Secure it (HTTPS, reverse proxy, non-root containers)
    ↓
Verify it actually works end-to-end
    ↓
Maintain it (monitoring, backups, migrations, cleanup)
```

You now have a complete, real, production-deployed full-stack application — and every decision along the way traces back to a concept explained earlier in these notes. Nothing here was memorized command syntax; every step answered a "why" before showing the "how."

*End of Hands-On Tutorial*


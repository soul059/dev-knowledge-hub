# Chapter 1: Docker Engine Components

> **Unit 2: Docker Architecture** | [Course Home](../README.md)

---

## Overview

Docker Engine is the core software that runs and manages containers. Understanding its **component architecture** is essential for troubleshooting, performance tuning, and making informed decisions about container infrastructure.

---

## 1.1 Docker Engine — High-Level Architecture

Docker Engine follows a **client-server architecture** with three major components:

```
+-------------------------------------------------------------------+
|                        DOCKER ENGINE                               |
|                                                                    |
|  +------------------+         +-------------------------------+    |
|  |   Docker CLI     |  REST   |         Docker Daemon         |    |
|  |   (docker)       | ------> |          (dockerd)            |    |
|  |                  |   API   |                               |    |
|  |  User interface  |         |  +----------+ +-----------+   |    |
|  |  to Docker       |         |  |  Image   | | Container |   |    |
|  +------------------+         |  | Managmnt | | Managmnt  |   |    |
|                               |  +----------+ +-----------+   |    |
|  +------------------+         |  +-----------+ +----------+   |    |
|  |   Docker API     |         |  |  Network  | |  Volume  |   |    |
|  |  (REST / SDK)    |         |  | Managmnt  | | Managmnt |   |    |
|  +------------------+         |  +-----------+ +----------+   |    |
|                               |                               |    |
|                               |  +---------------------------+|    |
|                               |  |       containerd          ||    |
|                               |  |  (container lifecycle)    ||    |
|                               |  +-------------+-------------+|    |
|                               |                |              |    |
|                               |  +-------------v-------------+|    |
|                               |  |          runc             ||    |
|                               |  |  (OCI runtime - creates   ||    |
|                               |  |   containers using kernel)||    |
|                               |  +---------------------------+|    |
|                               +-------------------------------+    |
+-------------------------------------------------------------------+
|                      Host Operating System                         |
|              (Linux Kernel: namespaces, cgroups, UnionFS)          |
+-------------------------------------------------------------------+
```

---

## 1.2 Component Breakdown

### The Three Major Components

```
  1. Docker CLI         2. Docker Daemon        3. Container Runtime
  (Client)              (Server)                (Execution)
  
  +----------+          +-------------+         +-------------+
  | docker   |  REST    |  dockerd    |         | containerd  |
  | commands | -------> |             | ------> |             |
  |          |   API    | Orchestrates|  gRPC   | Manages     |
  | build    |          | everything  |         | containers  |
  | run      |          |             |         |             |
  | pull     |          | Images      |         +------+------+
  | push     |          | Networking  |                |
  | ...      |          | Volumes     |         +------v------+
  +----------+          | Build       |         |    runc     |
                        | Security    |         | Creates     |
                        +-------------+         | containers  |
                                                +-------------+
```

| Component | Binary | Role |
|-----------|--------|------|
| **Docker CLI** | `docker` | User-facing command-line interface |
| **Docker Daemon** | `dockerd` | Server that manages Docker objects |
| **containerd** | `containerd` | Industry-standard container runtime |
| **runc** | `runc` | OCI-compliant low-level container creator |

---

## 1.3 Request Flow — What Happens When You Type `docker run`

```
  $ docker run -d nginx
  
  Step 1: CLI parses command
  +----------+
  | docker   | "Run nginx image in detached mode"
  | CLI      |
  +----+-----+
       |
       | REST API call: POST /containers/create
       v
  Step 2: Daemon processes request
  +----------+
  | dockerd  | - Checks if nginx image exists locally
  |          | - If not, pulls from Docker Hub
  |          | - Creates container configuration
  |          | - Sets up networking
  |          | - Prepares volumes
  +----+-----+
       |
       | gRPC call
       v
  Step 3: containerd manages container lifecycle
  +------------+
  | containerd | - Receives container spec
  |            | - Creates container bundle
  |            | - Manages container state
  +----+-------+
       |
       | Fork/exec
       v
  Step 4: runc creates the actual container
  +----------+
  |   runc   | - Sets up namespaces
  |          | - Configures cgroups
  |          | - Pivots root filesystem
  |          | - Starts the container process
  |          | - Exits (container runs independently)
  +----------+
       |
       v
  Step 5: Container is running
  +--------------+
  | nginx:latest |  PID 1 = nginx master process
  | Container    |  Isolated network, filesystem, PID space
  +--------------+
```

---

## 1.4 Communication Protocols

```
  Docker CLI  <-- REST API (HTTP/Unix Socket) -->  Docker Daemon
  Docker Daemon  <-- gRPC -->  containerd
  containerd  <-- OCI Runtime Spec (fork/exec) -->  runc
  
  +------------------------------------------------------------------+
  |  Communication Channels                                           |
  |                                                                   |
  |  CLI ──> Daemon:                                                  |
  |    Unix socket: /var/run/docker.sock  (default, local)            |
  |    TCP socket:  tcp://host:2376      (remote, TLS secured)        |
  |                                                                   |
  |  Daemon ──> containerd:                                           |
  |    gRPC over Unix socket: /run/containerd/containerd.sock         |
  |                                                                   |
  |  containerd ──> runc:                                             |
  |    Direct process execution (fork/exec with OCI bundle)           |
  +------------------------------------------------------------------+
```

---

## 1.5 Docker Desktop vs Docker Engine

```
  Docker Desktop (Windows/Mac)        Docker Engine (Linux)
  +----------------------------+      +----------------------------+
  |  Docker Desktop App        |      |                            |
  |  +----------------------+  |      |  Docker CLI (docker)       |
  |  | GUI / Dashboard      |  |      |                            |
  |  +----------------------+  |      |  Docker Daemon (dockerd)   |
  |  | Docker CLI           |  |      |                            |
  |  +----------------------+  |      |  containerd                |
  |  | Docker Daemon        |  |      |                            |
  |  +----------------------+  |      |  runc                      |
  |  | Linux VM             |  |      |                            |
  |  | (WSL2 or HyperKit)   |  |      |  Linux Kernel             |
  |  |  +----------------+  |  |      |  (native)                  |
  |  |  | containerd     |  |  |      +----------------------------+
  |  |  | runc           |  |  |
  |  |  | Linux Kernel   |  |  |      No VM needed!
  |  |  +----------------+  |  |      Native performance.
  |  +----------------------+  |
  +----------------------------+
  
  Runs Linux in a lightweight VM
  because Docker needs a Linux kernel.
```

---

## 1.6 Verifying Components

```bash
# Check Docker version (client + server)
$ docker version
Client:
 Version:           24.0.7
 API version:       1.43
 Go version:        go1.21.3
 OS/Arch:           linux/amd64

Server:
 Engine:
  Version:          24.0.7
  API version:      1.43
  containerd:
   Version:         1.7.6
  runc:
   Version:         1.1.10
  docker-init:
   Version:         0.19.0

# Check Docker system info
$ docker info
Server Version: 24.0.7
Storage Driver: overlay2
Logging Driver: json-file
Cgroup Driver: systemd
Kernel Version: 6.2.0
Operating System: Ubuntu 22.04.3 LTS
CPUs: 4
Total Memory: 7.764GiB

# Check individual components
$ containerd --version
$ runc --version

# Check Docker daemon status
$ systemctl status docker
$ systemctl status containerd
```

---

## 1.7 Docker Engine Editions

| Edition | Abbreviation | Target | Support |
|---------|-------------|--------|---------|
| Docker Engine (Community) | CE | Developers, small teams | Community |
| Docker Desktop | — | Individual developers | Free for personal/small biz |
| Docker Business | — | Enterprise | Commercial support |
| Mirantis Container Runtime | MCR | Enterprise (formerly Docker EE) | Commercial |

---

## Summary Table

| Component | Binary | Role | Communicates Via |
|-----------|--------|------|-----------------|
| Docker CLI | `docker` | User commands | REST API (Unix/TCP socket) |
| Docker Daemon | `dockerd` | Orchestrates everything | Listens on socket; calls containerd via gRPC |
| containerd | `containerd` | Container lifecycle management | gRPC; calls runc via fork/exec |
| runc | `runc` | Creates containers (OCI runtime) | OS kernel (namespaces, cgroups) |
| docker-init | `docker-init` | Lightweight init for containers | Runs as PID 1 when `--init` used |

---

## Quick Revision Questions

1. **What are the three major components of Docker Engine?**
   - Docker CLI (client), Docker Daemon (server/dockerd), and container runtime (containerd + runc)

2. **What protocol does the Docker CLI use to communicate with the Docker Daemon?**
   - REST API over a Unix socket (`/var/run/docker.sock`) or TCP socket

3. **What is the role of containerd in the Docker architecture?**
   - containerd manages the complete container lifecycle — image transfer, container execution, storage, and networking — acting as a bridge between dockerd and runc

4. **Why does Docker Desktop on Windows/Mac need a Linux VM?**
   - Docker containers require Linux kernel features (namespaces, cgroups). On non-Linux systems, Docker Desktop runs a lightweight Linux VM (via WSL2 or HyperKit) to provide this kernel

5. **What happens to runc after it creates a container?**
   - runc exits after creating the container. The container process runs independently, managed by containerd's shim process

6. **How can you verify the versions of all Docker components?**
   - Run `docker version` to see client, server, containerd, and runc versions all at once

---

[← Previous Unit: Container Use Cases](../01-Container-Concepts/05-container-use-cases.md) | [Next: Docker Daemon →](02-docker-daemon.md)

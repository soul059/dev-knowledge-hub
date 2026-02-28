# Chapter 4: containerd and runc

> **Unit 2: Docker Architecture** | [Course Home](../README.md)

---

## Overview

**containerd** and **runc** are the lower-level components that actually create and manage containers. Docker donated them to the Cloud Native Computing Foundation (CNCF) and OCI respectively, making them industry-standard components used far beyond Docker.

---

## 4.1 Where They Fit in the Stack

```
  +------------------------------------------------------------+
  |  High Level (User-Facing)                                   |
  |  +---------------------------+                              |
  |  |  Docker CLI / Kubernetes  |  <-- Users interact here    |
  |  +------------+--------------+                              |
  |               |                                             |
  |  +------------v--------------+                              |
  |  |  Docker Daemon / kubelet  |  <-- Orchestration layer    |
  |  +------------+--------------+                              |
  |               |                                             |
  |  Mid Level (Container Management)                           |
  |  +------------v--------------+                              |
  |  |       containerd          |  <-- Lifecycle management   |
  |  |  (CNCF graduated project) |                              |
  |  +------------+--------------+                              |
  |               |                                             |
  |  Low Level (Container Creation)                             |
  |  +------------v--------------+                              |
  |  |          runc             |  <-- Kernel interaction     |
  |  |   (OCI reference runtime) |                              |
  |  +---------------------------+                              |
  |                                                             |
  +------------------------------------------------------------+
  |  Linux Kernel (namespaces, cgroups, capabilities)           |
  +------------------------------------------------------------+
```

---

## 4.2 containerd — In Detail

### What containerd Does

containerd is a **container lifecycle manager**. It handles everything between the daemon and the low-level runtime.

```
+----------------------------------------------------------------+
|                         containerd                              |
|                                                                 |
|  +------------------+    +------------------+                   |
|  | Image Service    |    | Container Service|                   |
|  | - Pull images    |    | - Create         |                   |
|  | - Push images    |    | - Start          |                   |
|  | - Unpack layers  |    | - Stop           |                   |
|  | - Store content  |    | - Delete         |                   |
|  +------------------+    +------------------+                   |
|                                                                 |
|  +------------------+    +------------------+                   |
|  | Snapshot Service |    | Task Service     |                   |
|  | - Manage layers  |    | - Exec processes |                   |
|  | - overlay2       |    | - Attach I/O     |                   |
|  | - snapshots      |    | - Manage signals |                   |
|  +------------------+    +------------------+                   |
|                                                                 |
|  +------------------+    +------------------+                   |
|  | Content Store    |    | Events Service   |                   |
|  | - Blobs          |    | - Container events|                  |
|  | - Manifests      |    | - Image events   |                   |
|  | - Configs        |    | - Task events    |                   |
|  +------------------+    +------------------+                   |
|                                                                 |
|  +---------------------------------------------------------+   |
|  |                    containerd-shim                       |   |
|  |  - Runs between containerd and runc                      |   |
|  |  - Allows containerd to restart without killing containers|  |
|  |  - Manages STDIO and keeps container STDIN/STDOUT open   |   |
|  |  - Reports exit status back to containerd                |   |
|  +---------------------------------------------------------+   |
+----------------------------------------------------------------+
```

### The Shim Process — Key Innovation

```
  WITHOUT shim (old model):
  dockerd ──> container process
  If dockerd crashes, container dies!
  
  WITH shim (current model):
  
  dockerd ──> containerd ──> containerd-shim-runc-v2 ──> container
  
  +----------+     +-----------+     +------------------+     +----------+
  | dockerd  | --> | containerd| --> | containerd-shim  | --> | container|
  +----------+     +-----------+     | -runc-v2         |     | process  |
                                     +------------------+     +----------+
                                          |
  dockerd can restart  ✓                  Stays alive!
  containerd can restart  ✓               Keeps container running!
  Only shim needs to stay                 via pipe/fifo
```

### containerd CLI (ctr)

```bash
# containerd has its own CLI for direct interaction
# (rarely needed — Docker CLI is preferred)

# List containers
sudo ctr containers list

# List images
sudo ctr images list

# Pull an image
sudo ctr images pull docker.io/library/nginx:latest

# Run a container
sudo ctr run docker.io/library/nginx:latest my-nginx

# List tasks (running processes)
sudo ctr tasks list

# Enter a container namespace
sudo ctr tasks exec --exec-id shell1 my-nginx sh
```

---

## 4.3 runc — In Detail

### What runc Does

runc is the **low-level container runtime** that creates containers by interacting directly with the Linux kernel.

```
  runc receives:
  +----------------------------------+
  |  OCI Bundle                       |
  |  ├── config.json  (runtime spec)  |
  |  └── rootfs/      (filesystem)    |
  |      ├── bin/                     |
  |      ├── etc/                     |
  |      ├── lib/                     |
  |      └── ...                      |
  +----------------------------------+
  
  runc does:
  1. Parse config.json
  2. Create Linux namespaces (PID, NET, MNT, UTS, IPC, USER)
  3. Set up cgroups (CPU, memory, I/O limits)
  4. Configure capabilities (drop unnecessary privileges)
  5. Set up seccomp profile (restrict system calls)
  6. Pivot root (change to container's rootfs)
  7. Execute the container process
  8. EXIT (runc is no longer running!)
  
  +----------------------------------+
  |  RESULT: Running Container       |
  |  - Isolated PID namespace        |
  |  - Own network stack             |
  |  - Own mount namespace           |
  |  - Resource limits applied       |
  |  - Restricted capabilities       |
  |  - Managed by containerd-shim    |
  +----------------------------------+
```

### Using runc Directly

```bash
# runc can be used standalone (advanced usage)

# Create an OCI bundle
mkdir -p mycontainer/rootfs

# Export a Docker image to use as rootfs
docker export $(docker create busybox) | tar -C mycontainer/rootfs -xf -

# Generate default OCI config
cd mycontainer
runc spec  # Creates config.json

# Run with runc
sudo runc run my-container

# List runc containers
sudo runc list

# Check container state
sudo runc state my-container

# Delete container
sudo runc delete my-container
```

---

## 4.4 Complete Container Creation Flow

```
  docker run -d --name web -p 80:80 --memory 256m nginx
  
  =====================================================
  
  1. Docker CLI
     └── POST /containers/create {...}
         POST /containers/{id}/start
  
  2. Docker Daemon (dockerd)
     ├── Pull nginx image (if needed)
     ├── Create container record
     ├── Set up networking (bridge, port mapping)
     ├── Prepare mount points
     └── Call containerd via gRPC
  
  3. containerd
     ├── Create OCI bundle from image
     │   ├── Unpack image layers → rootfs
     │   └── Generate config.json
     ├── Create containerd-shim-runc-v2
     └── Tell shim to create container
  
  4. containerd-shim-runc-v2
     ├── Fork runc with OCI bundle
     ├── runc creates namespaces + cgroups
     ├── runc starts container process
     ├── runc EXITS
     └── Shim keeps running, manages container I/O
  
  5. Container Process
     ├── nginx master process (PID 1 in container)
     ├── Isolated in own namespaces
     ├── Limited to 256 MB memory
     └── Port 80 mapped to host port 80

  =====================================================
  
  PROCESS TREE ON HOST:
  
  systemd (PID 1)
  ├── dockerd
  ├── containerd
  │   └── containerd-shim-runc-v2
  │       └── nginx (master)
  │           ├── nginx (worker)
  │           └── nginx (worker)
  └── ...
```

---

## 4.5 containerd Beyond Docker

containerd is used by many platforms, not just Docker:

```
  +------------------+
  |   Docker Engine  | ----+
  +------------------+     |
                           |     +--------------+     +------+
  +------------------+     +---->|              |---->|      |
  |   Kubernetes     | -------->| containerd   |     | runc |
  |   (via CRI)      | -------->|              |---->|      |
  +------------------+     +---->|              |     +------+
                           |     +--------------+
  +------------------+     |
  |   AWS Fargate    | ----+     Used by all major
  +------------------+          container platforms!
  
  +------------------+
  |   Google GKE     | ----+
  +------------------+     |
                            → containerd
  +------------------+     |
  |   Azure AKS      | ----+
  +------------------+
```

### Kubernetes CRI Integration

```
  Kubernetes Node:
  
  +----------------------------------------------------------+
  |  kubelet                                                  |
  |    |                                                      |
  |    | CRI (Container Runtime Interface) gRPC               |
  |    v                                                      |
  |  +------------------+                                     |
  |  |   containerd     |                                     |
  |  |  (CRI plugin)    |                                     |
  |  +--------+---------+                                     |
  |           |                                               |
  |  +--------v---------+                                     |
  |  |      runc        |                                     |
  |  +------------------+                                     |
  |                                                           |
  |  Docker is NOT needed for Kubernetes!                     |
  |  Kubernetes talks directly to containerd.                 |
  +----------------------------------------------------------+
```

---

## 4.6 Alternative Runtimes

| Runtime | Written In | Key Feature | Used By |
|---------|-----------|-------------|---------|
| **runc** | Go | OCI reference implementation | Docker (default) |
| **crun** | C | Faster, lower memory | Podman (default) |
| **kata-runtime** | Go | Lightweight VM per container | Security teams |
| **runsc (gVisor)** | Go | User-space kernel sandbox | Google Cloud Run |
| **youki** | Rust | Memory safety | Experimental |
| **runwasi** | Rust | Run WASM workloads | Edge/serverless |

---

## Summary Table

| Component | containerd | runc |
|-----------|-----------|------|
| **Role** | Container lifecycle management | Container creation |
| **Level** | Mid-level | Low-level |
| **Project** | CNCF graduated | OCI reference runtime |
| **Written in** | Go | Go |
| **Stays running?** | Yes (daemon) | No (exits after creating container) |
| **CLI** | `ctr` | `runc` |
| **Used by** | Docker, K8s, Fargate, GKE, AKS | Docker, Podman, containerd |
| **Key innovation** | containerd-shim (daemon-less containers) | Direct kernel interaction |

---

## Quick Revision Questions

1. **What is the role of containerd in Docker's architecture?**
   - containerd manages the complete container lifecycle: image pulling/unpacking, container creation, execution management, and storage — acting as the bridge between dockerd and runc

2. **What is the containerd-shim and why is it important?**
   - The shim is a lightweight process that sits between containerd and the container process. It allows containerd (and dockerd) to restart without killing running containers

3. **Does runc keep running after creating a container?**
   - No. runc creates the container (sets up namespaces, cgroups, rootfs) then exits. The container process is then managed by the containerd-shim

4. **Can containerd be used without Docker?**
   - Yes. Kubernetes uses containerd directly via CRI, and platforms like AWS Fargate and Google GKE also use containerd without Docker

5. **What is an OCI bundle that runc receives?**
   - An OCI bundle consists of a `config.json` (runtime specification with namespaces, cgroups, capabilities) and a `rootfs` directory (the container's filesystem)

6. **Name two alternative OCI runtimes to runc and their distinguishing features.**
   - crun (written in C, faster and lower memory usage) and Kata Containers (uses lightweight VMs for stronger isolation)

---

[← Previous: Docker CLI](03-docker-cli.md) | [Next: Docker Objects →](05-docker-objects.md)

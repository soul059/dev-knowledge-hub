# Chapter 4: OCI Standards

> **Unit 1: Container Concepts** | [Course Home](../README.md)

---

## Overview

The **Open Container Initiative (OCI)** defines industry standards for container formats and runtimes. These standards ensure that containers built with one tool can run on any OCI-compliant runtime, preventing vendor lock-in and fostering interoperability.

---

## 4.1 Why Standards Were Needed

Before OCI, Docker's proprietary format was the de facto standard. This created concerns:

```
  BEFORE OCI (2013-2015):
  
  Docker format ──> Only runs on Docker runtime
  rkt format    ──> Only runs on rkt runtime
  LXC format    ──> Only runs on LXC runtime
  
  Problem: VENDOR LOCK-IN
  
  ============================================
  
  AFTER OCI (2015+):
  
  Any OCI-compliant ──> Any OCI-compliant
  image builder          runtime
  
  +----------+           +----------+
  | Docker   |──+    +──>| Docker   |
  | BuildKit |  |    |   | (runc)   |
  +----------+  |    |   +----------+
                v    v
  +----------+  +----+   +----------+
  | Podman   |──| OCI|──>| crun     |
  | Buildah  |  |spec|   +----------+
  +----------+  +----+
                ^    ^   +----------+
  +----------+  |    +──>| Kata     |
  | Kaniko   |──+        | Containers|
  +----------+           +----------+
  
  Result: INTEROPERABILITY
```

---

## 4.2 What Is the OCI?

The **Open Container Initiative** was established in June 2015 by Docker and other industry leaders under the Linux Foundation. Key founding members:

- Docker
- CoreOS
- Google  
- Microsoft
- Amazon
- IBM
- Red Hat

### OCI Defines Three Specifications

```
+-------------------------------------------------------+
|              OCI Specifications                        |
|                                                        |
|  +------------------+  +-------------------+           |
|  |  Runtime Spec    |  |   Image Spec      |           |
|  |  (runtime-spec)  |  |   (image-spec)    |           |
|  |                  |  |                   |           |
|  | How to RUN a     |  | How to BUILD and  |           |
|  | container        |  | PACKAGE an image  |           |
|  +------------------+  +-------------------+           |
|                                                        |
|  +------------------+                                  |
|  | Distribution Spec|                                  |
|  | (dist-spec)      |                                  |
|  |                  |                                  |
|  | How to PUSH/PULL |                                  |
|  | images to/from   |                                  |
|  | registries       |                                  |
|  +------------------+                                  |
+-------------------------------------------------------+
```

---

## 4.3 OCI Runtime Specification

The runtime spec defines the **lifecycle and configuration** of a container.

### Container Lifecycle (per OCI Spec)

```
  +---------+     +---------+     +---------+     +----------+
  | Creating| --> | Created | --> | Running | --> | Stopped  |
  +---------+     +---------+     +---------+     +----------+
                       |                               |
                       v                               v
                  +---------+                    +---------+
                  |  Start  |                    | Deleted |
                  +---------+                    +---------+
```

### Runtime Config (config.json)

Every OCI container has a `config.json` that defines:

```json
{
  "ociVersion": "1.0.2",
  "process": {
    "terminal": true,
    "user": { "uid": 0, "gid": 0 },
    "args": ["sh"],
    "env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ],
    "cwd": "/"
  },
  "root": {
    "path": "rootfs",
    "readonly": true
  },
  "hostname": "my-container",
  "linux": {
    "namespaces": [
      { "type": "pid" },
      { "type": "network" },
      { "type": "ipc" },
      { "type": "uts" },
      { "type": "mount" }
    ],
    "resources": {
      "memory": { "limit": 536870912 },
      "cpu": { "shares": 1024 }
    }
  }
}
```

### OCI-Compliant Runtimes

| Runtime | Description | Use Case |
|---------|------------|----------|
| **runc** | Reference implementation (by Docker/OCI) | Default Docker runtime |
| **crun** | Written in C, faster than runc | Podman default, performance |
| **Kata Containers** | Lightweight VMs for containers | Strong isolation |
| **gVisor (runsc)** | Google's sandboxed runtime | Security-sensitive workloads |
| **youki** | Written in Rust | Experimental, safety |

---

## 4.4 OCI Image Specification

The image spec defines how container images are **built, structured, and distributed**.

### Image Structure

```
OCI Image Layout:
  
  +---------------------------------------------------+
  |  Image Index (manifest list)                       |
  |  Points to platform-specific manifests             |
  |                                                    |
  |  +---------------------------------------------+  |
  |  |  Image Manifest                              |  |
  |  |  +---------+  +---------+  +---------+      |  |
  |  |  | Config  |  | Layer 1 |  | Layer 2 |      |  |
  |  |  | (JSON)  |  | (tar.gz)|  | (tar.gz)|      |  |
  |  |  +---------+  +---------+  +---------+      |  |
  |  +---------------------------------------------+  |
  +---------------------------------------------------+
  
  Config JSON contains:
  - Architecture (amd64, arm64)
  - OS (linux, windows)
  - Environment variables
  - Entrypoint / Cmd
  - Exposed ports
  - Volumes
  - Labels
```

### Multi-Platform Images

```
  Image Index (fat manifest)
  +------------------------------------------+
  |                                          |
  |  linux/amd64 ──> Manifest ──> Layers     |
  |  linux/arm64 ──> Manifest ──> Layers     |
  |  linux/arm/v7 ─> Manifest ──> Layers     |
  |  windows/amd64 > Manifest ──> Layers     |
  |                                          |
  +------------------------------------------+
  
  docker pull nginx
  --> Automatically pulls the correct architecture!
```

---

## 4.5 OCI Distribution Specification

Defines the **API for pushing and pulling** images to/from registries.

### Registry API Flow

```
  Client                          Registry
    |                                |
    |  GET /v2/                      |
    |------------------------------->|  Check API version
    |  200 OK                        |
    |<-------------------------------|
    |                                |
    |  HEAD /v2/<name>/manifests/<ref>|
    |------------------------------->|  Check if image exists
    |  200 OK (or 404)               |
    |<-------------------------------|
    |                                |
    |  GET /v2/<name>/manifests/<ref> |
    |------------------------------->|  Pull manifest
    |  200 OK + manifest JSON        |
    |<-------------------------------|
    |                                |
    |  GET /v2/<name>/blobs/<digest>  |
    |------------------------------->|  Pull each layer
    |  200 OK + layer data           |
    |<-------------------------------|
```

### OCI-Compliant Registries

| Registry | Provider | Features |
|----------|----------|----------|
| Docker Hub | Docker Inc. | Largest public registry |
| GitHub Container Registry | GitHub | Tied to GitHub repos |
| Amazon ECR | AWS | Integrated with AWS services |
| Azure Container Registry | Microsoft | Integrated with Azure |
| Google Artifact Registry | Google | Integrated with GCP |
| Harbor | CNCF | Open-source, self-hosted |
| Quay.io | Red Hat | Security scanning built-in |

---

## 4.6 OCI in Practice — How Docker Uses OCI

```
+-------------------------------------------------------+
|  docker build / docker run                             |
|                                                        |
|  +--------------------------------------------------+ |
|  |  Docker Engine (dockerd)                          | |
|  |                                                   | |
|  |  BuildKit ─────────> OCI Image Spec               | |
|  |  (builds images)     (image format)               | |
|  |                                                   | |
|  |  containerd ─────────> OCI Runtime Spec           | |
|  |  (manages lifecycle)   (container config)         | |
|  |                                                   | |
|  |  runc ───────────────> OCI Runtime Spec           | |
|  |  (creates containers)  (process execution)        | |
|  |                                                   | |
|  |  Registry push/pull ──> OCI Distribution Spec     | |
|  |                         (API protocol)            | |
|  +--------------------------------------------------+ |
+-------------------------------------------------------+
```

---

## 4.7 Why OCI Matters to You

1. **No vendor lock-in** — images built with Docker work with Podman, containerd, CRI-O
2. **Tool flexibility** — choose the best tool for each job
3. **Future-proof** — industry-standard won't disappear
4. **Multi-platform** — single image supports multiple architectures
5. **Security** — standardized image signing and verification

```
  Build with:         Run with:           Store in:
  +----------+        +----------+        +-----------+
  | Docker   |        | Docker   |        | Docker Hub|
  | BuildKit |  OCI   | Podman   |  OCI   | ECR       |
  | Buildah  | =====> | CRI-O    | <===== | ACR       |
  | Kaniko   | Image  | containerd| Image  | GCR       |
  | Pack     |        | Kata     |        | Harbor    |
  +----------+        +----------+        +-----------+
```

---

## Summary Table

| OCI Spec | Purpose | Key Content |
|----------|---------|-------------|
| Runtime Spec | How to run containers | Lifecycle states, config.json, namespaces, cgroups |
| Image Spec | How to package images | Layers, manifest, config, multi-platform support |
| Distribution Spec | How to share images | Registry API for push/pull operations |

| Term | Definition |
|------|-----------|
| OCI | Open Container Initiative — industry standards body |
| runc | Reference OCI runtime implementation |
| Manifest | JSON describing image layers and config |
| Image Index | Multi-platform manifest list |
| config.json | Runtime configuration for a container |

---

## Quick Revision Questions

1. **What is the OCI and why was it created?**
   - The Open Container Initiative is a standards body under the Linux Foundation, created to prevent vendor lock-in by defining open standards for container runtimes, images, and distribution

2. **Name the three OCI specifications and what each defines.**
   - Runtime Spec (how to run containers), Image Spec (how to package images), Distribution Spec (how to push/pull images to/from registries)

3. **What is runc and what is its relationship to OCI?**
   - runc is the reference implementation of the OCI Runtime Specification, originally developed by Docker, used as the default low-level container runtime

4. **How do OCI standards benefit developers in practice?**
   - Images built with any OCI-compliant builder (Docker, Buildah, Kaniko) can run on any OCI-compliant runtime (Docker, Podman, CRI-O) and be stored in any OCI-compliant registry

5. **What is an Image Index (fat manifest) used for?**
   - An Image Index points to platform-specific manifests, enabling a single image reference (like `nginx:latest`) to automatically resolve to the correct architecture (amd64, arm64, etc.)

6. **Name three OCI-compliant container runtimes besides runc.**
   - crun (C-based, faster), Kata Containers (VM-based isolation), gVisor/runsc (Google's sandboxed runtime)

---

[← Previous: Container History](03-container-history.md) | [Next: Container Use Cases →](05-container-use-cases.md)

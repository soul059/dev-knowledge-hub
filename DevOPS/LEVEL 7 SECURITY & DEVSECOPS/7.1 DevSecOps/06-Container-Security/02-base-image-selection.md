# Chapter 6.2: Base Image Selection

## Overview

The base image is the foundation of every container. Choosing the right base image dramatically impacts security posture — a full Ubuntu image has 400+ packages (many unnecessary), while a distroless image has only what your app needs. This chapter covers selection criteria, hardened images, and migration strategies.

---

## Base Image Comparison

```
BASE IMAGE SECURITY COMPARISON:

  ┌──────────────────┬────────┬──────────┬──────────┬───────────┐
  │ Base Image        │ Size   │ Packages │ Vulns*   │ Shell?    │
  ├──────────────────┼────────┼──────────┼──────────┼───────────┤
  │ ubuntu:22.04     │ 77 MB  │ 400+     │ 50-100   │ Yes       │
  │ debian:12        │ 124 MB │ 300+     │ 40-80    │ Yes       │
  │ python:3.11      │ 920 MB │ 600+     │ 100-200  │ Yes       │
  │ node:20          │ 1.1 GB │ 700+     │ 150-250  │ Yes       │
  ├──────────────────┼────────┼──────────┼──────────┼───────────┤
  │ python:3.11-slim │ 120 MB │ 100+     │ 10-30    │ Yes       │
  │ node:20-slim     │ 200 MB │ 100+     │ 10-30    │ Yes       │
  │ alpine:3.19      │ 7 MB   │ 15       │ 0-5      │ Yes (ash) │
  │ python:3.11-alpine│ 50 MB │ 30       │ 0-10     │ Yes (ash) │
  ├──────────────────┼────────┼──────────┼──────────┼───────────┤
  │ distroless/python│ 50 MB  │ ~10      │ 0-3      │ No        │
  │ distroless/nodejs│ 40 MB  │ ~10      │ 0-3      │ No        │
  │ scratch          │ 0 MB   │ 0        │ 0        │ No        │
  └──────────────────┴────────┴──────────┴──────────┴───────────┘
  * Approximate typical vulnerability count
```

---

## Image Selection Decision Tree

```
CHOOSING A BASE IMAGE:

                        Start
                          │
                          ▼
                ┌───────────────────┐
                │ Language/Runtime?  │
                └────────┬──────────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Compiled       Interpreted    Static
       (Go, Rust,     (Python, Node, (HTML, config)
        C++)           Ruby, Java)
          │              │              │
          ▼              ▼              ▼
     ┌────────┐    ┌──────────┐    ┌────────┐
     │scratch │    │Need debug│    │ nginx  │
     │or      │    │shell?    │    │ alpine │
     │distro- │    └──┬───┬──┘    └────────┘
     │less    │   Yes │   │ No
     └────────┘       ▼   ▼
               slim/     distroless
               alpine

  RECOMMENDATION BY USE CASE:
  ┌────────────────┬──────────────────────────────┐
  │ Use Case        │ Recommended Base              │
  ├────────────────┼──────────────────────────────┤
  │ Go microservice│ scratch or distroless/static │
  │ Python API     │ python:3.11-slim             │
  │ Node.js app    │ node:20-slim                 │
  │ Java (Spring)  │ eclipse-temurin:21-jre-alpine│
  │ Static website │ nginx:alpine                  │
  │ Maximum security│ gcr.io/distroless/...       │
  └────────────────┴──────────────────────────────┘
```

---

## Distroless Images

```
WHAT IS DISTROLESS?

  Traditional Image:
  ┌──────────────────────────────────────────────┐
  │ App binary                                    │
  │ App dependencies                              │
  │ Package manager (apt, yum)     ← NOT NEEDED  │
  │ Shell (bash, sh)                ← NOT NEEDED  │
  │ Utilities (curl, wget, vi)     ← NOT NEEDED  │
  │ System libraries                               │
  │ OS userspace                    ← BLOAT       │
  └──────────────────────────────────────────────┘

  Distroless Image:
  ┌──────────────────────────────────────────────┐
  │ App binary                                    │
  │ App dependencies                              │
  │ Minimal system libraries (glibc, libssl)     │
  │ CA certificates                               │
  │ (Nothing else — no shell, no package mgr)    │
  └──────────────────────────────────────────────┘

  SECURITY BENEFIT:
  - No shell = attacker can't get interactive access
  - No package manager = can't install attack tools
  - Minimal libraries = fewer vulnerabilities
  - Smaller attack surface
```

```dockerfile
# Distroless Python example
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --target=/app/deps -r requirements.txt
COPY . .

FROM gcr.io/distroless/python3-debian12
WORKDIR /app
COPY --from=builder /app /app
ENV PYTHONPATH=/app/deps
CMD ["main.py"]
```

```dockerfile
# Distroless Go example (scratch also works)
FROM golang:1.22 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /server .

FROM gcr.io/distroless/static-debian12
COPY --from=builder /server /server
USER nonroot:nonroot
ENTRYPOINT ["/server"]
```

---

## Alpine vs Slim vs Distroless

```
COMPARISON FOR PYTHON:

  ┌────────────────────┬────────────┬─────────────┬──────────────┐
  │ Aspect              │ Alpine     │ Slim         │ Distroless   │
  ├────────────────────┼────────────┼─────────────┼──────────────┤
  │ Size                │ ~50 MB     │ ~120 MB     │ ~50 MB       │
  │ Shell access        │ Yes (ash)  │ Yes (bash)  │ No           │
  │ Package manager    │ apk        │ apt         │ None          │
  │ Debugging           │ Easy       │ Easy        │ Hard          │
  │ C extension compat │ Complex*   │ Full        │ Full          │
  │ musl vs glibc      │ musl       │ glibc       │ glibc         │
  │ Security            │ Very good  │ Good        │ Best          │
  │ Community support   │ Large      │ Large       │ Growing       │
  └────────────────────┴────────────┴─────────────┴──────────────┘

  * Alpine uses musl libc, which can break Python packages with
    C extensions (numpy, pandas, cryptography). Use slim for
    these packages or install build dependencies.
```

---

## Hardening Base Images

```dockerfile
# HARDENED BASE IMAGE TEMPLATE

FROM python:3.11-slim AS base

# 1. Update system packages (patch known vulns)
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        tini && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 2. Create non-root user
RUN groupadd -r appuser && \
    useradd -r -g appuser -d /app -s /sbin/nologin appuser

# 3. Set filesystem permissions
WORKDIR /app
COPY --chown=appuser:appuser . .

# 4. Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# 5. Remove unnecessary tools
RUN apt-get purge -y --auto-remove gcc g++ make && \
    rm -rf /tmp/* /root/.cache

# 6. Switch to non-root user
USER appuser

# 7. Use tini as init process
ENTRYPOINT ["tini", "--"]
CMD ["python", "app.py"]
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Python package fails on Alpine | C extension incompatible with musl | Use slim image instead; or install build-base + compile |
| Distroless: Can't debug container | No shell available | Use debug variant: `gcr.io/distroless/python3:debug` temporarily |
| Base image has unfixed CVEs | Upstream hasn't patched | Use alternative base; accept risk with documentation; build custom base |
| Image too large (1GB+) | Using full runtime image | Switch to slim/alpine; use multi-stage builds |
| Alpine DNS issues | musl DNS resolution differences | Use `RUN apk add --no-cache libc6-compat` or switch to slim |

---

## Summary Table

| Image Type | Size | Security | Debugging | Best For |
|------------|------|----------|-----------|----------|
| **Full (ubuntu/debian)** | Large | Low | Easy | Development only |
| **Slim** | Medium | Good | Easy | Production (most apps) |
| **Alpine** | Small | Very Good | Easy | Small images, Go, simple apps |
| **Distroless** | Small | Best | Hard | Production (max security) |
| **Scratch** | Zero | Best | None | Static Go/Rust binaries |

---

## Quick Revision Questions

1. **Why is base image selection the most impactful container security decision?**
   > 60-80% of container vulnerabilities come from the base OS layer. A full Ubuntu image has 400+ packages, most unnecessary for your app but each a potential vulnerability. Slim/distroless images eliminate hundreds of vulnerabilities by removing unneeded packages.

2. **What is a distroless image and why is it more secure?**
   > Distroless images contain only your app, its runtime dependencies, and minimal system libraries — no shell, no package manager, no utilities. If an attacker compromises the app, they can't get an interactive shell or install attack tools, significantly limiting post-exploitation.

3. **When should you NOT use Alpine?**
   > When your application uses Python packages with C extensions (numpy, pandas, cryptography) or when you need glibc compatibility. Alpine uses musl libc instead of glibc, which can cause compilation failures or runtime issues with certain libraries.

4. **What is a multi-stage build and how does it improve security?**
   > Multi-stage builds use separate stages for building (with compilers, dev tools) and running (minimal runtime only). Build tools never end up in the final image, reducing the attack surface and image size.

5. **What five steps harden a base image?**
   > (1) Update all system packages, (2) Create and use a non-root user, (3) Remove unnecessary tools (gcc, make, package managers), (4) Use tini as init process, (5) Set proper file permissions and remove temp/cache files.

---

[← Previous: 6.1 Image Security Scanning](01-image-security-scanning.md) | [Next: 6.3 Dockerfile Best Practices →](03-dockerfile-best-practices.md)

[Back to Table of Contents](../README.md)

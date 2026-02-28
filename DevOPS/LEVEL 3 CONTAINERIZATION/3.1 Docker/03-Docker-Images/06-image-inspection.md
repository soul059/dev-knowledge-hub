# Chapter 6: Image Inspection

> **Unit 3: Docker Images** | [Course Home](../README.md)

---

## Overview

Inspecting Docker images lets you understand their **structure, configuration, layers, and metadata**. This is essential for debugging, security auditing, and optimizing images.

---

## 6.1 Inspection Commands

```bash
# Full JSON inspection
docker image inspect nginx:latest

# Specific fields with Go templates
docker inspect --format='{{.Id}}' nginx
docker inspect --format='{{.Config.Cmd}}' nginx
docker inspect --format='{{.Config.Env}}' nginx
docker inspect --format='{{json .Config}}' nginx | jq .
docker inspect --format='{{.Os}}/{{.Architecture}}' nginx

# Image history (layers)
docker image history nginx:latest
docker image history --no-trunc nginx:latest   # Full commands

# Image size
docker images --format "{{.Repository}}:{{.Tag}} {{.Size}}" nginx

# Manifest inspection (multi-platform)
docker manifest inspect nginx:latest
docker manifest inspect --verbose nginx:latest
```

---

## 6.2 What docker inspect Reveals

```bash
docker image inspect nginx:latest | jq '.[0]'
```

```
  docker inspect output structure:
  
  +---------------------------------------------------+
  |  Image Inspect JSON                                |
  |                                                    |
  |  Id: "sha256:a1b2c3..."                           |
  |  RepoTags: ["nginx:latest", "nginx:1.25"]         |
  |  RepoDigests: ["nginx@sha256:32fdf..."]            |
  |  Created: "2024-01-15T10:30:00Z"                   |
  |  Size: 187432960                                   |
  |                                                    |
  |  Config:                                           |
  |  ├── Hostname: ""                                  |
  |  ├── User: ""                                      |
  |  ├── Env:                                          |
  |  │   ├── PATH=/usr/local/sbin:...                  |
  |  │   ├── NGINX_VERSION=1.25.3                      |
  |  │   └── NJS_VERSION=0.8.2                         |
  |  ├── Cmd: ["nginx", "-g", "daemon off;"]           |
  |  ├── ExposedPorts: {"80/tcp": {}}                  |
  |  ├── WorkingDir: ""                                |
  |  ├── Entrypoint: ["/docker-entrypoint.sh"]         |
  |  ├── Labels: {"maintainer": "NGINX..."}            |
  |  └── StopSignal: "SIGQUIT"                         |
  |                                                    |
  |  RootFS:                                           |
  |  ├── Type: "layers"                                |
  |  └── Layers:                                       |
  |      ├── "sha256:aaa..."                           |
  |      ├── "sha256:bbb..."                           |
  |      └── "sha256:ccc..."                           |
  +---------------------------------------------------+
```

---

## 6.3 Common Inspection Queries

```bash
# What command does this image run?
docker inspect --format='CMD: {{.Config.Cmd}}' nginx
docker inspect --format='ENTRYPOINT: {{.Config.Entrypoint}}' nginx

# What ports are exposed?
docker inspect --format='{{json .Config.ExposedPorts}}' nginx
# {"80/tcp":{}}

# What environment variables are set?
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' nginx
# PATH=/usr/local/sbin:...
# NGINX_VERSION=1.25.3

# What user does it run as?
docker inspect --format='User: {{.Config.User}}' nginx
# User:   (empty means root!)

# How many layers?
docker inspect --format='{{len .RootFS.Layers}} layers' nginx
# 7 layers

# Image creation date
docker inspect --format='{{.Created}}' nginx

# Labels
docker inspect --format='{{json .Config.Labels}}' nginx | jq .

# Architecture
docker inspect --format='{{.Architecture}}' nginx
# amd64
```

---

## 6.4 Image History — Layer Analysis

```bash
$ docker image history nginx:latest

IMAGE          CREATED      CREATED BY                                 SIZE
a1b2c3d4e5f6   2 days ago   CMD ["nginx" "-g" "daemon off;"]           0B
<missing>      2 days ago   STOPSIGNAL SIGQUIT                         0B
<missing>      2 days ago   EXPOSE map[80/tcp:{}]                      0B
<missing>      2 days ago   ENTRYPOINT ["/docker-entrypoint.sh"]       0B
<missing>      2 days ago   COPY file:xxx in /docker-entrypoint.d      4.62kB
<missing>      2 days ago   COPY file:xxx in /docker-entrypoint.d      1.04kB
<missing>      2 days ago   RUN /bin/sh -c set -x && addgroup...       61.1MB
<missing>      2 days ago   ENV NGINX_VERSION=1.25.3 NJS_VER...        0B
<missing>      3 days ago   /bin/sh -c #(nop) ADD file:xxx in /        74.8MB
```

```
  READING THE HISTORY:
  
  Bottom-up = Build order (first instruction at bottom)
  
  +----------------------------------------------------------+
  | CMD (0B)           <-- Metadata only, no filesystem change |
  | EXPOSE (0B)        <-- Metadata only                       |
  | ENTRYPOINT (0B)    <-- Metadata only                       |
  | COPY scripts (5KB) <-- Small files copied                  |
  | RUN install (61MB) <-- Biggest layer! nginx installed here |
  | ENV (0B)           <-- Metadata only                       |
  | ADD base (75MB)    <-- Base Debian filesystem              |
  +----------------------------------------------------------+
  
  To optimize: Focus on the largest layers.
  Can the 61MB layer be smaller? (use alpine base?)
```

---

## 6.5 Third-Party Inspection Tools

### dive — Layer Explorer

```bash
# Install dive (popular image layer explorer)
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive nginx:latest
```

```
  dive output:
  
  +--LEFT PANE (Layers)--------+--RIGHT PANE (Files)----------+
  |                             |                               |
  | Layer 1: 74.8 MB           | /bin/                         |
  | ├── ADD base filesystem    | /boot/                        |
  |                             | /dev/                         |
  | Layer 2: 61.1 MB           | /etc/                         |
  | ├── RUN apt-get install    | /etc/nginx/  (Added)          |
  |     nginx                  | /usr/sbin/nginx  (Added)     |
  |                             |                               |
  | Layer 3: 1 KB              | /docker-entrypoint.d/         |
  | ├── COPY 10-listen.sh      |   10-listen.envsubst.sh      |
  |                             |                               |
  +-----------------------------+-------------------------------+
  
  Image efficiency score: 97%
  Wasted space: 1.2 MB
```

### skopeo — Remote Inspection

```bash
# Inspect image WITHOUT pulling it
skopeo inspect docker://nginx:latest

# Compare images across registries
skopeo inspect docker://docker.io/nginx:latest
skopeo inspect docker://ghcr.io/owner/nginx:latest

# Copy between registries without pulling locally
skopeo copy docker://source/image:tag docker://dest/image:tag
```

---

## 6.6 Security Inspection

```bash
# Check for known vulnerabilities
docker scout cve nginx:latest        # Docker Scout
docker scout quickview nginx         # Quick overview

# Trivy (popular open-source scanner)
docker run --rm aquasec/trivy image nginx:latest

# Grype
docker run --rm anchore/grype nginx:latest

# Check if image runs as root
docker inspect --format='{{.Config.User}}' nginx:latest
# Empty = runs as root (security risk!)

# Check for secrets in image layers
docker history --no-trunc myapp:latest | grep -i "password\|secret\|key\|token"
```

---

## 6.7 Practical Inspection Scenarios

### Scenario 1: Debug why a container won't start

```bash
# Check the default command
docker inspect --format='{{.Config.Cmd}}' myapp
docker inspect --format='{{.Config.Entrypoint}}' myapp

# Check environment variables
docker inspect --format='{{json .Config.Env}}' myapp | jq .

# Check working directory
docker inspect --format='{{.Config.WorkingDir}}' myapp

# Check exposed ports
docker inspect --format='{{json .Config.ExposedPorts}}' myapp
```

### Scenario 2: Audit an image before using it

```bash
# Who built it? When?
docker inspect --format='Created: {{.Created}}' suspicious-image
docker inspect --format='{{json .Config.Labels}}' suspicious-image

# What user does it run as?
docker inspect --format='User: "{{.Config.User}}"' suspicious-image

# What does it execute?
docker inspect --format='Entrypoint: {{.Config.Entrypoint}}' suspicious-image
docker inspect --format='Cmd: {{.Config.Cmd}}' suspicious-image

# What's in each layer?
docker history --no-trunc suspicious-image
```

---

## Summary Table

| Command | Purpose |
|---------|---------|
| `docker inspect` | Full JSON metadata of an image |
| `docker history` | Show layers and build instructions |
| `docker manifest inspect` | View manifest (multi-platform info) |
| `docker scout cve` | Scan for vulnerabilities |
| `dive` | Interactive layer file explorer |
| `skopeo inspect` | Inspect remote images without pulling |
| `trivy` / `grype` | Open-source vulnerability scanners |

---

## Quick Revision Questions

1. **How do you find out what command an image runs by default?**
   - `docker inspect --format='{{.Config.Cmd}}' image` and `docker inspect --format='{{.Config.Entrypoint}}' image`

2. **How can you check if an image runs as root?**
   - `docker inspect --format='{{.Config.User}}' image` — if empty or `"0"`, the image runs as root

3. **What does `docker image history` show?**
   - The layers of an image, including the Dockerfile instruction that created each layer, when it was created, and its size

4. **How can you inspect an image in a remote registry without pulling it?**
   - Use `skopeo inspect docker://registry/image:tag` or `docker manifest inspect image:tag`

5. **What is the `dive` tool used for?**
   - dive is an interactive tool that lets you explore each layer of an image, see which files were added/modified/deleted, and identify wasted space

6. **How do you check an image for known vulnerabilities?**
   - Use `docker scout cve image:tag`, or third-party tools like `trivy image image:tag` or `grype image:tag`

---

[← Previous: Docker Hub and Registries](05-docker-hub-and-registries.md) | [Next Unit: Dockerfile →](../04-Dockerfile/01-dockerfile-instructions.md)

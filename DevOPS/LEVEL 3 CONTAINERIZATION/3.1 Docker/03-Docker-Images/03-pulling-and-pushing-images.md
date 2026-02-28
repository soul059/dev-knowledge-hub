# Chapter 3: Pulling and Pushing Images

> **Unit 3: Docker Images** | [Course Home](../README.md)

---

## Overview

Pulling and pushing images is how you **download** images from registries and **upload** your own images to share with others. This is the foundation of Docker's distribution model.

---

## 3.1 Pull Flow

```
  $ docker pull nginx:1.25
  
  +----------+                          +----------------+
  | Docker   |  GET /v2/nginx/manifests | Docker Hub     |
  | Client   | -----------------------> | (Registry)     |
  |          |                          |                |
  |          | <---- manifest JSON ---- |                |
  |          |                          |                |
  |          | GET /v2/nginx/blobs/sha  |                |
  |          | -----------------------> | Layer 1        |
  |          | <---- layer data ------- |                |
  |          |                          |                |
  |          | GET /v2/nginx/blobs/sha  |                |
  |          | -----------------------> | Layer 2        |
  |          | <---- layer data ------- |                |
  |          |                          |                |
  |          | GET /v2/nginx/blobs/sha  |                |
  |          | -----------------------> | Layer 3        |
  |          | <---- layer data ------- |                |
  +----------+                          +----------------+
  
  Layers already present locally are SKIPPED!
```

### Pull Commands

```bash
# Pull from Docker Hub (default registry)
docker pull nginx                     # Defaults to :latest tag
docker pull nginx:1.25                # Specific version
docker pull nginx:1.25-alpine         # Specific variant

# Pull by digest (immutable — always gets exact same image)
docker pull nginx@sha256:32fdf92db...

# Pull from other registries
docker pull ghcr.io/owner/image:tag         # GitHub Container Registry
docker pull 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:v1  # AWS ECR
docker pull myregistry.azurecr.io/myapp:v1  # Azure Container Registry
docker pull gcr.io/my-project/myapp:v1      # Google Container Registry

# Pull all tags of an image
docker pull --all-tags nginx

# Pull for a specific platform
docker pull --platform linux/arm64 nginx:latest
```

### Understanding Pull Output

```
$ docker pull nginx:1.25
1.25: Pulling from library/nginx
a803e7c4b030: Already exists       <-- Layer cached locally
5fb8b62b6f10: Pull complete        <-- New layer downloaded
08a81e3bad7c: Pull complete
46bb67e7bb50: Pull complete
Digest: sha256:32fdf92db...        <-- Content hash
Status: Downloaded newer image for nginx:1.25
docker.io/library/nginx:1.25       <-- Full image reference
```

---

## 3.2 Push Flow

```
  $ docker push myregistry/myapp:v1
  
  +----------+                          +----------------+
  | Docker   |  POST /v2/myapp/blobs/  | Registry       |
  | Client   | -----------------------> |                |
  |          | ---- upload Layer 1 ---> |  Stored!       |
  |          |                          |                |
  |          | ---- upload Layer 2 ---> |  Stored!       |
  |          |                          |                |
  |          | ---- upload Layer 3 ---> |  Already exists|
  |          |                          |  (skipped!)    |
  |          |                          |                |
  |          | PUT /v2/myapp/manifests/ |                |
  |          | ---- upload manifest --> |  Stored!       |
  +----------+                          +----------------+
  
  Layers already in registry are SKIPPED!
```

### Push Commands

```bash
# Step 1: Tag image for your registry
docker tag myapp:v1 myregistry.com/username/myapp:v1

# Step 2: Login to registry
docker login myregistry.com
# Enter username and password

# Step 3: Push
docker push myregistry.com/username/myapp:v1

# Push to Docker Hub
docker tag myapp:v1 username/myapp:v1
docker login
docker push username/myapp:v1

# Push all tags
docker push --all-tags username/myapp
```

---

## 3.3 Registry Authentication

```
  docker login flow:
  
  +----------+     +----------------+     +------------------+
  | docker   | --> | Enter username | --> | Credentials      |
  | login    |     | and password   |     | stored in        |
  |          |     |                |     | ~/.docker/config |
  +----------+     +----------------+     +------------------+
  
  ~/.docker/config.json:
  {
    "auths": {
      "https://index.docker.io/v1/": {
        "auth": "dXNlcm5hbWU6cGFzc3dvcmQ="    <-- base64 encoded
      },
      "ghcr.io": {
        "auth": "Z2hwX3h4eDp4eHg="
      }
    },
    "credHelpers": {
      "gcr.io": "gcloud",                      <-- uses gcloud CLI
      "*.dkr.ecr.*.amazonaws.com": "ecr-login" <-- uses AWS CLI
    }
  }
```

### Login to Various Registries

```bash
# Docker Hub
docker login
docker login -u username

# GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# AWS ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Azure Container Registry
az acr login --name myregistry

# Google Container Registry
gcloud auth configure-docker

# Logout
docker logout
docker logout ghcr.io
```

---

## 3.4 Image Naming Convention

```
  FULL IMAGE REFERENCE:
  
  registry.example.com/namespace/repository:tag@sha256:digest
  ├──────────────────┤ ├───────┤ ├────────┤ ├─┤ ├──────────┤
  │                    │         │          │   │
  Registry hostname    Namespace Repository Tag Digest
  (optional, defaults  (user or  (image    (version)
  to docker.io)        org name) name)
  
  EXAMPLES:
  ┌────────────────────────────────────────────────────────────┐
  │ nginx                                                      │
  │ = docker.io/library/nginx:latest                          │
  │                                                            │
  │ username/myapp:v2                                          │
  │ = docker.io/username/myapp:v2                             │
  │                                                            │
  │ ghcr.io/owner/repo:main                                   │
  │ = ghcr.io/owner/repo:main                                │
  │                                                            │
  │ 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:latest       │
  │ = (full AWS ECR reference)                                │
  └────────────────────────────────────────────────────────────┘
```

---

## 3.5 Saving and Loading Images (Offline Transfer)

```bash
# Save an image to a tar file
docker save nginx:latest -o nginx.tar
docker save nginx:latest | gzip > nginx.tar.gz

# Load an image from a tar file
docker load -i nginx.tar
docker load < nginx.tar.gz

# Export a container's filesystem (not an image!)
docker export mycontainer -o container-fs.tar

# Import a filesystem as an image
docker import container-fs.tar myimage:imported
```

```
  SAVE vs EXPORT:
  
  docker save (IMAGE)              docker export (CONTAINER)
  +-----------------+              +------------------+
  | All layers      |              | Flat filesystem  |
  | Metadata        |              | No layers        |
  | Tags            |              | No metadata      |
  | Full image      |              | Just files       |
  +-----------------+              +------------------+
  Use: Transfer images             Use: Extract filesystem
  Restore: docker load             Restore: docker import
```

---

## 3.6 Real-World Workflow

```
  Developer Workflow:
  
  1. Write Dockerfile
  2. Build image locally
     $ docker build -t myapp:v1 .
  
  3. Test locally
     $ docker run -p 3000:3000 myapp:v1
  
  4. Tag for registry
     $ docker tag myapp:v1 ghcr.io/myteam/myapp:v1
  
  5. Push to registry
     $ docker push ghcr.io/myteam/myapp:v1
  
  6. Deploy on server
     $ docker pull ghcr.io/myteam/myapp:v1
     $ docker run -d ghcr.io/myteam/myapp:v1
  
  +--------+    +--------+    +---------+    +--------+
  | Build  | -> | Test   | -> | Push to | -> | Deploy |
  | Image  |    | Local  |    | Registry|    | Server |
  +--------+    +--------+    +---------+    +--------+
```

---

## Summary Table

| Operation | Command | Description |
|-----------|---------|-------------|
| Pull | `docker pull image:tag` | Download image from registry |
| Push | `docker push image:tag` | Upload image to registry |
| Login | `docker login registry` | Authenticate to registry |
| Logout | `docker logout registry` | Remove credentials |
| Tag | `docker tag src dst` | Create new reference for image |
| Save | `docker save -o file.tar` | Export image to tar (with layers) |
| Load | `docker load -i file.tar` | Import image from tar |
| Export | `docker export container` | Export container filesystem (flat) |
| Import | `docker import file.tar` | Create image from flat filesystem |

---

## Quick Revision Questions

1. **What is the default registry when you run `docker pull nginx`?**
   - Docker Hub (`docker.io`). The full reference is `docker.io/library/nginx:latest`

2. **How can you guarantee pulling the exact same image every time?**
   - Use the image digest instead of a tag: `docker pull nginx@sha256:32fdf92db...`

3. **What is the difference between `docker save` and `docker export`?**
   - `docker save` exports a full image with all layers and metadata. `docker export` exports a container's filesystem as a flat tarball without layers

4. **Why are some layers shown as "Already exists" during a pull?**
   - Those layers are already cached locally from a previous pull of the same or another image that shares those layers

5. **What must you do before pushing an image to a private registry?**
   - Tag the image with the registry hostname (`docker tag myapp registry.com/myapp:v1`), authenticate (`docker login registry.com`), then push

6. **Where does Docker store registry credentials locally?**
   - In `~/.docker/config.json` — credentials are base64 encoded (not encrypted!). Use credential helpers for secure storage

---

[← Previous: Image Layers and Caching](02-image-layers-and-caching.md) | [Next: Image Tagging →](04-image-tagging.md)

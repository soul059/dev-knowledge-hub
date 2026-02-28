# Chapter 5: Docker Hub and Registries

> **Unit 3: Docker Images** | [Course Home](../README.md)

---

## Overview

A **container registry** is a storage and distribution system for Docker images. Docker Hub is the default public registry, but many private and self-hosted options exist for enterprise use.

---

## 5.1 Registry Architecture

```
  +------------------------------------------------------------+
  |                    Container Registry                       |
  |                                                            |
  |  +------------------+    +------------------+              |
  |  |  API Server       |    |  Storage Backend |              |
  |  |  (HTTP/HTTPS)     |    |  (S3, GCS, Azure |              |
  |  |                   |    |   Blob, local FS) |              |
  |  |  /v2/             |    |                   |              |
  |  |  ├── manifests    |    |  +-----------+    |              |
  |  |  ├── blobs        |--->|  | Manifests |    |              |
  |  |  └── tags         |    |  | Layers    |    |              |
  |  |                   |    |  | Configs   |    |              |
  |  +------------------+    |  +-----------+    |              |
  |                           +------------------+              |
  |                                                            |
  |  +------------------+    +------------------+              |
  |  | Authentication   |    | Garbage          |              |
  |  | (tokens, OAuth)  |    | Collection       |              |
  |  +------------------+    +------------------+              |
  +------------------------------------------------------------+
```

---

## 5.2 Docker Hub

Docker Hub is the world's largest container image library and the default registry.

```
  Docker Hub:
  +------------------------------------------------------+
  |                                                      |
  |  Official Images         Verified Publisher           |
  |  +-------------+        +------------------+         |
  |  | nginx       |        | bitnami/nginx    |         |
  |  | postgres    |        | grafana/grafana  |         |
  |  | node        |        | hashicorp/vault  |         |
  |  | python      |        | datadog/agent    |         |
  |  | redis       |        +------------------+         |
  |  | ubuntu      |                                     |
  |  +-------------+        Community Images              |
  |  Curated, reviewed      +------------------+         |
  |  by Docker Inc.         | user/myapp       |         |
  |                          | team/project     |         |
  |                          +------------------+         |
  |                          Anyone can publish           |
  +------------------------------------------------------+
```

### Docker Hub Features

| Feature | Free | Pro | Team | Business |
|---------|------|-----|------|----------|
| Public repos | Unlimited | Unlimited | Unlimited | Unlimited |
| Private repos | 1 | Unlimited | Unlimited | Unlimited |
| Pulls/6hrs | 100 (anon), 200 (auth) | 5,000 | 5,000 | Unlimited |
| Automated builds | No | Yes | Yes | Yes |
| Vulnerability scanning | No | Yes | Yes | Yes |
| Team management | No | No | Yes | Yes |

### Using Docker Hub

```bash
# Search for images
docker search nginx
docker search --filter is-official=true nginx

# Pull official image
docker pull nginx
docker pull nginx:1.25-alpine

# Push to Docker Hub
docker login
docker tag myapp:v1 username/myapp:v1
docker push username/myapp:v1

# Create automated build (link to GitHub/GitLab)
# Configured in Docker Hub web UI
```

---

## 5.3 Registry Comparison

| Registry | Provider | Best For | Pricing |
|----------|----------|----------|---------|
| **Docker Hub** | Docker Inc. | Public images, getting started | Free tier + paid |
| **GitHub GHCR** | GitHub | GitHub-integrated projects | Free for public |
| **AWS ECR** | Amazon | AWS-deployed applications | Pay per storage/transfer |
| **Azure ACR** | Microsoft | Azure-deployed applications | Pay per tier |
| **Google AR** | Google | GCP-deployed applications | Pay per storage/transfer |
| **Harbor** | CNCF | Self-hosted, air-gapped | Free (open-source) |
| **Quay.io** | Red Hat | OpenShift, security scanning | Free + paid |
| **GitLab CR** | GitLab | GitLab CI/CD integrated | Part of GitLab |
| **JFrog Artifactory** | JFrog | Universal artifact management | Paid |

---

## 5.4 Running Your Own Registry

Docker provides an official registry image for self-hosting.

```bash
# Start a local registry
docker run -d -p 5000:5000 --name registry \
  -v registry-data:/var/lib/registry \
  registry:2

# Tag and push to local registry
docker tag myapp:v1 localhost:5000/myapp:v1
docker push localhost:5000/myapp:v1

# Pull from local registry
docker pull localhost:5000/myapp:v1

# List images in registry (API)
curl http://localhost:5000/v2/_catalog
# {"repositories":["myapp"]}

# List tags
curl http://localhost:5000/v2/myapp/tags/list
# {"name":"myapp","tags":["v1"]}
```

### Production Registry Setup

```yaml
# docker-compose.yml for production registry
services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: "Registry Realm"
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      REGISTRY_HTTP_TLS_KEY: /certs/domain.key
    volumes:
      - registry-data:/var/lib/registry
      - ./auth:/auth
      - ./certs:/certs
    restart: always

volumes:
  registry-data:
```

```bash
# Create htpasswd file for authentication
docker run --rm --entrypoint htpasswd \
  httpd:2 -Bbn admin secretpassword > auth/htpasswd
```

---

## 5.5 Registry Mirrors and Proxies

```
  Without Mirror:
  Client ──────────────────────> Docker Hub (slow, rate-limited)
  
  With Mirror/Proxy:
  Client ── check ──> Local Mirror ── cache miss ──> Docker Hub
                           |
                      cache hit ──> Return immediately (fast!)
  
  +----------------------------------------------------------+
  |  Internal Network                                         |
  |                                                           |
  |  +--------+     +----------+     +------------------+     |
  |  | Dev 1  | --> |          |     |                  |     |
  |  +--------+     |  Pull-   | --> | Docker Hub       |     |
  |  +--------+     |  through |     | (only on cache   |     |
  |  | Dev 2  | --> |  Cache   |     |  miss)           |     |
  |  +--------+     |  (mirror)|     +------------------+     |
  |  +--------+     |          |                              |
  |  | CI/CD  | --> +----------+                              |
  |  +--------+       Serves cached images locally            |
  +----------------------------------------------------------+
```

```json
// /etc/docker/daemon.json — configure mirror
{
  "registry-mirrors": ["https://mirror.mycompany.com"]
}
```

---

## 5.6 Image Trust and Signing

```
  Docker Content Trust (DCT):
  
  Publisher:                          Consumer:
  +----------------+                 +----------------+
  | docker push    |                 | docker pull    |
  | (signs image   |                 | (verifies      |
  |  with private  |                 |  signature     |
  |  key)          |                 |  with public   |
  +-------+--------+                 |  key)          |
          |                          +-------+--------+
          v                                  ^
  +-------+--------+                         |
  |  Registry +    |                         |
  |  Notary Server |  ───────────────────────+
  |  (stores sigs) |
  +----------------+
```

```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Now all push/pull operations are signed/verified
docker push myapp:v1    # Signs automatically
docker pull myapp:v1    # Verifies signature

# Disable for unsigned images
export DOCKER_CONTENT_TRUST=0
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Registry | Storage and distribution service for container images |
| Docker Hub | Default public registry by Docker Inc. |
| Official Image | Curated image maintained by Docker and upstream maintainers |
| Private Registry | Self-hosted registry (using `registry:2` image) |
| Registry Mirror | Local cache/proxy reducing pulls from remote registries |
| Content Trust | Signing mechanism ensuring image integrity and publisher identity |
| Rate Limiting | Docker Hub limits: 100 pulls/6hrs anonymous, 200 authenticated |

---

## Quick Revision Questions

1. **What is the default registry when no registry is specified in an image name?**
   - Docker Hub (`docker.io`). For example, `nginx` resolves to `docker.io/library/nginx`

2. **What is the difference between an Official Image and a Verified Publisher image?**
   - Official Images are curated and reviewed by Docker Inc. Verified Publisher images come from verified organizations but are not curated by Docker

3. **How do you set up a private Docker registry?**
   - Run the official `registry:2` image: `docker run -d -p 5000:5000 registry:2`, then push images to `localhost:5000/image:tag`

4. **What is Docker Hub rate limiting and how do you work around it?**
   - Anonymous users: 100 pulls/6hrs, authenticated: 200 pulls/6hrs. Work around it by authenticating, using registry mirrors, or using a paid plan

5. **What is Docker Content Trust (DCT)?**
   - A signing mechanism (using Notary) that ensures image integrity and authenticity. When enabled, pushes are signed and pulls verify signatures

6. **Why would a company run a registry mirror?**
   - To reduce external network traffic, avoid rate limits, speed up pulls with local caching, and ensure image availability even if external registries are down

---

[← Previous: Image Tagging](04-image-tagging.md) | [Next: Image Inspection →](06-image-inspection.md)

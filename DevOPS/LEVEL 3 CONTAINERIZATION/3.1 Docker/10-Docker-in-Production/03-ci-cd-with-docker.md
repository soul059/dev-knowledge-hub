# Chapter 3: CI/CD with Docker

> **Unit 10: Docker in Production** | [Course Home](../README.md)

---

## Overview

Docker transforms CI/CD pipelines by providing consistent build environments, cacheable layers, and portable artifacts. This chapter covers building, testing, and deploying Docker images through automated pipelines.

---

## 3.1 Docker in the CI/CD Pipeline

```
  CI/CD Pipeline with Docker:
  
  Developer            CI Server              Registry         Production
  +-------+     +--------------------+     +---------+     +-----------+
  |  git  | --> | 1. Checkout code   |     |         |     |           |
  | push  |     | 2. Lint + Test     |     |         |     |           |
  +-------+     | 3. Build image     | --> | Push    | --> | Pull +    |
                | 4. Scan image      |     | tagged  |     | Deploy    |
                | 5. Push to registry|     | image   |     | container |
                +--------------------+     +---------+     +-----------+
                         |
                    +----v----+
                    | Staging |  (optional deploy + test)
                    +---------+
```

---

## 3.2 GitHub Actions Pipeline

```yaml
# .github/workflows/docker.yml
name: Build and Deploy

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
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests in Docker
        run: |
          docker compose -f docker-compose.test.yml up \
            --build --abort-on-container-exit --exit-code-from tests

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha
            type=semver,pattern={{version}}
            type=ref,event=branch

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/myapp
            docker compose pull
            docker compose up -d --remove-orphans
            docker image prune -f
```

---

## 3.3 GitLab CI Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - scan
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  DOCKER_IMAGE_LATEST: $CI_REGISTRY_IMAGE:latest

test:
  stage: test
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker compose -f docker-compose.test.yml up
        --build --abort-on-container-exit

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build
        --cache-from $DOCKER_IMAGE_LATEST
        --tag $DOCKER_IMAGE
        --tag $DOCKER_IMAGE_LATEST
        .
    - docker push $DOCKER_IMAGE
    - docker push $DOCKER_IMAGE_LATEST

scan:
  stage: scan
  image:
    name: aquasec/trivy:latest
    entrypoint: [""]
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $DOCKER_IMAGE
  allow_failure: true

deploy_staging:
  stage: deploy
  environment: staging
  script:
    - ssh deploy@staging "cd /app && IMAGE=$DOCKER_IMAGE docker compose up -d"
  only:
    - main

deploy_production:
  stage: deploy
  environment: production
  script:
    - ssh deploy@prod "cd /app && IMAGE=$DOCKER_IMAGE docker compose up -d"
  when: manual
  only:
    - main
```

---

## 3.4 Build Optimisation for CI

```dockerfile
# Optimised Dockerfile for CI caching
# syntax=docker/dockerfile:1

# Stage 1: Dependencies (cached unless package files change)
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --production=false

# Stage 2: Build (cached unless source changes)
FROM deps AS build
COPY . .
RUN npm run build

# Stage 3: Production dependencies only
FROM node:20-alpine AS prod-deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --production

# Stage 4: Final image
FROM node:20-alpine AS runtime
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=prod-deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
USER app
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

```
  CI Cache Strategy:
  
  Layer 1: Base image (node:20-alpine)     → rarely changes
  Layer 2: COPY package*.json              → weekly changes
  Layer 3: RUN npm ci                      → weekly changes  
  Layer 4: COPY source code                → every commit
  Layer 5: RUN npm run build               → every commit
  
  Cache hits on Layers 1-3 save 60-90% build time!
  
  Cache backends:
  +-----------+----------------------------------+
  |  Backend  |  Best For                        |
  +-----------+----------------------------------+
  | local     | Single machine builders          |
  | registry  | Shared CI, multiple runners       |
  | gha       | GitHub Actions (built-in cache)   |
  | s3        | AWS-based CI environments          |
  +-----------+----------------------------------+
```

```bash
# BuildKit cache mount for package managers
# syntax=docker/dockerfile:1
FROM python:3.12-slim
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# BuildKit inline cache (share cache via registry)
docker build \
  --build-arg BUILDKIT_INLINE_CACHE=1 \
  --cache-from myregistry/myapp:latest \
  -t myregistry/myapp:v1.2.3 .
```

---

## 3.5 Testing with Docker in CI

```yaml
# docker-compose.test.yml
services:
  # System under test
  api:
    build:
      context: .
      target: build
    environment:
      - DATABASE_URL=postgres://test:test@db:5432/testdb
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  # Test runner
  tests:
    build:
      context: .
      dockerfile: Dockerfile.test
    environment:
      - API_URL=http://api:3000
    depends_on:
      api:
        condition: service_healthy

  # Test dependencies
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U test"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
```

```bash
# Run tests and get exit code
docker compose -f docker-compose.test.yml up \
  --build \
  --abort-on-container-exit \
  --exit-code-from tests

# Clean up after tests
docker compose -f docker-compose.test.yml down -v
```

---

## 3.6 Image Scanning in CI

```bash
# Trivy - popular open-source scanner
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  myapp:v1.2.3

# Docker Scout (Docker's native scanner)
docker scout cves myapp:v1.2.3
docker scout recommendations myapp:v1.2.3

# Grype
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  anchore/grype myapp:v1.2.3 --fail-on high
```

```
  Scan Integration Pipeline:
  
  Build Image
       |
       v
  +----------+
  | Scan for |──── CRITICAL found? ──── FAIL build
  | CVEs     |                          (block merge)
  +----+-----+
       |
       v (no critical)
  +----------+
  | Scan for |──── HIGH found? ──── WARN
  | HIGH     |                      (allow, create ticket)
  +----+-----+
       |
       v (clean)
  +----------+
  | Push to  |
  | Registry |
  +----------+
```

---

## 3.7 Deployment Strategies

```
  Blue-Green Deployment:
  
  Load Balancer
       |
  +----+----+
  |         |
  v         v
  BLUE      GREEN
  (v1.0)    (v1.1)
  ACTIVE    STANDBY
  
  Step 1: Deploy v1.1 to GREEN
  Step 2: Run smoke tests on GREEN
  Step 3: Switch LB to GREEN
  Step 4: GREEN becomes ACTIVE
  Step 5: Keep BLUE for quick rollback
```

```bash
# Blue-Green with Docker Compose
# Step 1: Deploy new version
docker compose -f docker-compose.green.yml up -d

# Step 2: Health check new version
curl -f http://localhost:8081/health

# Step 3: Switch traffic (update nginx/LB config)
cp nginx-green.conf /etc/nginx/conf.d/default.conf
nginx -s reload

# Step 4: Remove old version (after validation)
docker compose -f docker-compose.blue.yml down
```

```
  Rolling Update:
  
  Time ──────────────────────────────>
  
  Instance 1: [  v1.0  ][  v1.1  ]
  Instance 2: [    v1.0    ][  v1.1  ]
  Instance 3: [       v1.0     ][  v1.1  ]
  
  Always N-1 instances available
  Gradual traffic shift
```

```bash
# Rolling update with Compose (scale)
docker compose up -d --scale api=4 --no-recreate
# Deploy new version one at a time
docker compose up -d --no-deps api
```

---

## 3.8 Rollback Strategy

```bash
# Rollback using image tags
# Current: v1.2.3, Rollback to: v1.2.2

# Option 1: Update compose and redeploy
sed -i 's/v1.2.3/v1.2.2/' docker-compose.yml
docker compose up -d

# Option 2: Direct image change
docker compose pull   # pulls pinned version
docker compose up -d

# Option 3: Keep previous containers
docker stop api-new
docker start api-old

# Verify rollback
curl -f http://localhost:3000/health
docker logs api --tail 50
```

---

## Summary Table

| CI/CD Aspect | Tool / Approach |
|-------------|----------------|
| **Build** | Multi-stage Dockerfile + BuildKit |
| **Cache** | Layer caching, registry cache, GHA cache |
| **Test** | docker-compose.test.yml with test dependencies |
| **Scan** | Trivy, Docker Scout, Grype |
| **Push** | Private registry (GHCR, ECR, ACR) |
| **Deploy** | docker compose pull + up -d |
| **Blue-Green** | Two compose files, LB switch |
| **Rolling** | Scale up new, drain old |
| **Rollback** | Redeploy previous image tag |
| **Secrets** | CI secrets injected at deploy time |

---

## Quick Revision Questions

1. **Why should you use multi-stage builds in CI/CD pipelines?**
   - They separate build-time dependencies from runtime, producing smaller images. CI benefits from caching each stage independently, and the final image contains only what's needed to run

2. **How does BuildKit cache improve CI build times?**
   - BuildKit can push/pull cache to a registry or CI-native store (e.g., GHA cache). Unchanged layers are reused between CI runs, skipping dependency installation and compilation

3. **Explain the difference between blue-green and rolling deployments.**
   - Blue-green runs two full environments and switches traffic atomically — instant rollback but requires double resources. Rolling updates replace instances one at a time — uses fewer resources but has a mixed-version window

4. **Why should image scanning be a blocking step in CI?**
   - Critical vulnerabilities in production images can be exploited. Blocking the pipeline on CRITICAL CVEs prevents knowingly deploying vulnerable code. HIGH severity can be a warning

5. **How do you ensure test environments are consistent in CI?**
   - Use `docker-compose.test.yml` to define the exact versions of databases, caches, and services. Every CI run gets identical infrastructure regardless of which runner executes it

---

[← Previous: Monitoring and Observability](02-monitoring-and-observability.md) | [Next: Container Orchestration →](04-container-orchestration.md)

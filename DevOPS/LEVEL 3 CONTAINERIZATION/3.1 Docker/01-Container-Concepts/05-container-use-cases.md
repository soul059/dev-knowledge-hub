# Chapter 5: Container Use Cases

> **Unit 1: Container Concepts** | [Course Home](../README.md)

---

## Overview

Containers have transformed how software is built, shipped, and operated. This chapter explores the **real-world scenarios** where containers shine, across development, testing, deployment, and operations.

---

## 5.1 Microservices Architecture

The most impactful use case. Containers let you decompose a monolithic application into independent, deployable services.

```
  MONOLITH:                        MICROSERVICES (Containerized):
  
  +---------------------+         +--------+ +--------+ +--------+
  |    Single App        |         |  Auth  | |  User  | | Order  |
  |  +-----+  +------+  |         | Service| | Service| | Service|
  |  | Auth |  | User |  |         +--------+ +--------+ +--------+
  |  +-----+  +------+  |         +--------+ +--------+ +--------+
  |  +------+  +------+  |         |Payment | |  Cart  | | Email  |
  |  | Order|  | Cart |  |  ===>   | Service| | Service| | Service|
  |  +------+  +------+  |         +--------+ +--------+ +--------+
  |  +-------+ +------+  |
  |  |Payment| | Email|  |         Each service:
  |  +-------+ +------+  |         - Own container
  |                       |         - Own language/framework
  |  Single deployment    |         - Independent scaling
  |  Single language      |         - Independent deployment
  +---------------------+         - Own database (optional)
```

### Example: E-Commerce Platform

```yaml
# Microservices running as containers
services:
  frontend:     # React app — 3 replicas
  auth-api:     # Node.js — 2 replicas
  product-api:  # Python Flask — 2 replicas
  order-api:    # Java Spring — 3 replicas
  payment-api:  # Go — 2 replicas
  notification: # Python — 1 replica
  postgres:     # Database
  redis:        # Cache
  rabbitmq:     # Message queue
```

---

## 5.2 CI/CD Pipelines

Containers provide **consistent, reproducible** build and test environments.

```
  +-------+     +----------+     +--------+     +---------+     +------+
  | Code  | --> |  Build   | --> |  Test  | --> | Stage   | --> | Prod |
  | Push  |     | Container|     |Container|    |Container|    |Container
  +-------+     +----------+     +--------+     +---------+     +------+
      |              |               |               |              |
      |         Dockerfile       Same image      Same image     Same image!
      |         builds app       runs tests     tested in       deployed to
      |                                         staging         production
      
  GUARANTEE: What you tested is EXACTLY what you deploy
```

### Real CI/CD Example (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy
on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Run tests in container
        run: docker run myapp:${{ github.sha }} npm test

      - name: Push to registry
        run: |
          docker tag myapp:${{ github.sha }} registry.io/myapp:latest
          docker push registry.io/myapp:latest

      - name: Deploy to production
        run: kubectl set image deployment/myapp myapp=registry.io/myapp:latest
```

---

## 5.3 Development Environments

Containers eliminate "works on my machine" by providing identical environments for every developer.

```
  WITHOUT CONTAINERS:                WITH CONTAINERS:
  
  Developer A:                       All Developers:
  - Node 18.2                        +-------------------+
  - PostgreSQL 14                    | docker compose up  |
  - Redis 6                          | +------+ +------+ |
  - Python 3.10                      | |Node18| |PG 15 | |
                                     | +------+ +------+ |
  Developer B:                       | +------+ +------+ |
  - Node 20.1       CONFLICTS!      | |Redis7| |Py3.11| |
  - PostgreSQL 15                    | +------+ +------+ |
  - Redis 7                          +-------------------+
  - Python 3.11                      
                                     IDENTICAL for everyone
  Developer C:                       Single command setup
  - Node 16.3                        Version-controlled
  - PostgreSQL 13                    Works on any OS
  - Redis 5
  - Python 3.9
```

### docker-compose.yml for Development

```yaml
services:
  app:
    build: .
    volumes:
      - .:/app              # Hot reload
    ports:
      - "3000:3000"
    depends_on:
      - db
      - cache

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: devpass
    volumes:
      - pgdata:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine

  mailhog:
    image: mailhog/mailhog   # Fake SMTP for testing
    ports:
      - "8025:8025"

volumes:
  pgdata:
```

---

## 5.4 Application Isolation and Multi-Tenancy

Run multiple versions or client instances on the same infrastructure without conflicts.

```
  MULTI-TENANT SaaS PLATFORM:
  
  +--------------------------------------------------+
  |  Host Server                                      |
  |                                                   |
  |  Client A:           Client B:          Client C: |
  |  +-----------+       +-----------+     +----------+
  |  | App v2.1  |       | App v2.0  |     | App v2.2||
  |  | DB: PG 15 |       | DB: PG 14 |     | DB:PG 15||
  |  | Config: A |       | Config: B |     | Config:C||
  |  +-----------+       +-----------+     +----------+
  |                                                   |
  |  Each client gets their own isolated stack        |
  |  Different versions, different configs            |
  +--------------------------------------------------+
```

---

## 5.5 Legacy Application Modernization

Containerize legacy apps without rewriting them to gain deployment benefits.

```
  LEGACY MIGRATION STRATEGY:
  
  Phase 1: Lift and Shift
  +------------------+          +--------------------+
  |  Old VM           |   ==>   |  Docker Container   |
  |  Apache + PHP 5.6 |         |  Apache + PHP 5.6   |
  |  MySQL 5.5        |         |  MySQL 5.5           |
  +------------------+          +--------------------+
  
  Phase 2: Decompose
  +-------------------+         +-------+ +-------+
  |  Monolith in      |   ==>   | PHP   | | MySQL |
  |  Single Container |         | App   | |  DB   |
  +-------------------+         +-------+ +-------+
  
  Phase 3: Modernize
  +-------+ +-------+          +------+ +------+ +------+
  | PHP   | | MySQL |   ==>   |Node.js| | Vue  | | PG   |
  | App   | |  DB   |         | API   | | SPA  | | 15   |
  +-------+ +-------+         +------+ +------+ +------+
```

---

## 5.6 Testing and QA

Spin up complete environments for testing, then tear them down instantly.

### Test Scenarios

```bash
# Integration testing with real dependencies
docker compose -f docker-compose.test.yml up -d
docker compose exec app pytest tests/integration/
docker compose down -v  # Clean teardown

# Browser testing with Selenium
docker run -d --name selenium selenium/standalone-chrome
docker run --network=host myapp:test npm run e2e

# Load testing
docker run --rm -i grafana/k6 run - < loadtest.js

# Security testing
docker run --rm -v $(pwd):/zap/wrk owasp/zap2docker-stable zap-baseline.py \
  -t http://target-app:8080
```

### Parallel Test Execution

```
  +--------------------------------------------------+
  |  CI Server                                        |
  |                                                   |
  |  +----------+  +----------+  +----------+        |
  |  | Test     |  | Test     |  | Test     |        |
  |  | Suite A  |  | Suite B  |  | Suite C  |        |
  |  | (own DB) |  | (own DB) |  | (own DB) |        |
  |  +----------+  +----------+  +----------+        |
  |       |              |              |             |
  |  All running simultaneously in separate           |
  |  containers with isolated databases               |
  +--------------------------------------------------+
```

---

## 5.7 Edge Computing and IoT

Run containers on resource-constrained edge devices.

```
  Cloud                        Edge Devices
  +------------+               +----------+
  |  Registry  | -- Pull -->   | Raspberry|
  |  (images)  |               | Pi + k3s |
  +------------+               +--+-------+
       |                          |
       |                       +--+-------+
       +---- Pull ----------> | IoT      |
       |                       | Gateway  |
       |                       +----------+
       |
       +---- Pull ----------> +----------+
                               | Retail   |
                               | Kiosk    |
                               +----------+
```

---

## 5.8 Database Management

Run databases in containers for development and testing (and increasingly for production).

```bash
# Spin up any database instantly
docker run -d --name postgres  -e POSTGRES_PASSWORD=pass postgres:15
docker run -d --name mysql     -e MYSQL_ROOT_PASSWORD=pass mysql:8
docker run -d --name mongodb   mongo:7
docker run -d --name redis     redis:7-alpine
docker run -d --name cassandra cassandra:4

# Multiple versions side by side (for migration testing)
docker run -d --name pg14 -p 5432:5432 postgres:14
docker run -d --name pg15 -p 5433:5432 postgres:15
docker run -d --name pg16 -p 5434:5432 postgres:16
```

---

## 5.9 Machine Learning and Data Science

Containers solve the notorious **dependency problem** in ML/DS projects.

```
  ML PIPELINE (Containerized):
  
  +------------+    +-------------+    +-----------+    +----------+
  |  Data      | -> | Training    | -> | Model     | -> | Serving  |
  |  Pipeline  |    | Container   |    | Registry  |    | Container|
  |            |    |             |    |           |    |          |
  | Python 3.11|    | GPU-enabled |    | MLflow    |    | FastAPI  |
  | Pandas     |    | PyTorch 2.x |    |           |    | ONNX     |
  | Spark      |    | CUDA 12     |    |           |    | TF Serve |
  +------------+    +-------------+    +-----------+    +----------+
  
  Each step has EXACT reproducible dependencies
```

---

## 5.10 Use Case Summary Matrix

| Use Case | Key Benefit | Technologies |
|----------|------------|--------------|
| Microservices | Independent deployment and scaling | Docker, Kubernetes |
| CI/CD | Consistent build/test environments | Docker, GitHub Actions, Jenkins |
| Dev Environments | "Works on every machine" | Docker Compose |
| Multi-tenancy | Isolated customer instances | Docker, Swarm/K8s |
| Legacy Modernization | Incremental migration path | Docker, Compose |
| Testing/QA | Fast setup and teardown | Docker Compose, Testcontainers |
| Edge/IoT | Lightweight deployment | k3s, Docker |
| Databases | Instant provisioning, version testing | Docker, volumes |
| ML/Data Science | Reproducible experiments | Docker, NVIDIA Container Toolkit |
| Disaster Recovery | Fast environment recreation | Docker images, registries |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Microservices | Decompose monoliths into independently deployable container services |
| CI/CD | Same image from build → test → staging → production |
| Dev Environments | docker compose up gives every developer identical setup |
| Multi-tenancy | Isolate customer workloads on shared infrastructure |
| Legacy Modernization | Containerize first, then decompose and modernize |
| Testing | Spin up full environments instantly, tear down after tests |
| Edge Computing | Run containers on constrained devices at the edge |
| ML Pipelines | Reproducible training, serving, and inference environments |

---

## Quick Revision Questions

1. **How do containers enable a microservices architecture?**
   - Each microservice runs in its own container with its own dependencies, language/runtime, and can be deployed, scaled, and updated independently

2. **What CI/CD guarantee do containers provide?**
   - The exact same image that passed tests is deployed to production — eliminating environment-related deployment failures

3. **How do containers solve the "works on my machine" problem for development teams?**
   - All developers use the same docker-compose.yml, which defines exact versions of every service and dependency, run with a single command

4. **Why are containers useful for database management in development?**
   - You can instantly spin up any database (Postgres, MySQL, MongoDB) at any version, run multiple versions simultaneously, and discard them when done

5. **How does containerization help with legacy application modernization?**
   - You can first containerize the monolith as-is (lift and shift), then gradually decompose it into separate containers, and finally modernize each service

6. **What makes containers suitable for ML/Data Science workflows?**
   - Containers capture the exact Python version, library versions, CUDA drivers, and configurations needed — ensuring experiments are reproducible across machines and over time

---

[← Previous: OCI Standards](04-oci-standards.md) | [Next Unit: Docker Architecture →](../02-Docker-Architecture/01-docker-engine-components.md)

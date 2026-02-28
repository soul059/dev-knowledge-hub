# Chapter 8.3: GitLab Runners

[← Previous: Stages & Jobs](02-stages-jobs.md) | [Back to README](../README.md) | [Next: Variables & Secrets →](04-variables-secrets.md)

---

## Overview

**GitLab Runners** are agents that pick up and execute CI/CD jobs. GitLab provides shared runners (SaaS), and you can register your own runners for custom needs.

---

## Runner Types

```
  RUNNER TYPES
  ════════════

  1. SHARED RUNNERS (gitlab.com)
     ├── Managed by GitLab
     ├── Available to all projects
     ├── Linux (Docker Machine)
     └── Free: 400 min/month

  2. GROUP RUNNERS
     ├── Shared across group projects
     ├── You manage infrastructure
     └── Scoped to GitLab group

  3. PROJECT RUNNERS (Specific)
     ├── Dedicated to one project
     ├── Useful for special hardware
     └── Most restrictive scope
```

---

## Executor Types

```
  ┌──────────────┬────────────────────────────────────────────────┐
  │ Executor     │ Description                                    │
  ├──────────────┼────────────────────────────────────────────────┤
  │ Docker       │ Each job runs in a fresh container (RECOMMENDED)│
  │ Kubernetes   │ Jobs run as K8s pods (auto-scaling)            │
  │ Shell        │ Runs directly on runner machine (least isolated)│
  │ Docker Machine│ Auto-creates VMs with Docker (auto-scaling)   │
  │ SSH          │ Runs commands on remote machine via SSH        │
  │ VirtualBox   │ Runs in VirtualBox VMs                        │
  └──────────────┴────────────────────────────────────────────────┘

  RECOMMENDED:
  ├── Docker executor   → Best for most teams
  └── Kubernetes        → Best for auto-scaling
```

---

## Installing & Registering a Runner

### Linux Installation

```bash
# Install GitLab Runner
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt install gitlab-runner

# Register runner
sudo gitlab-runner register \
  --url https://gitlab.com/ \
  --registration-token YOUR_TOKEN \
  --executor docker \
  --docker-image node:20-alpine \
  --description "Docker Runner" \
  --tag-list "docker,linux,node" \
  --run-untagged=true
```

### Docker Installation

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest

docker exec -it gitlab-runner gitlab-runner register
```

### Kubernetes (Helm)

```bash
helm repo add gitlab https://charts.gitlab.io
helm install gitlab-runner gitlab/gitlab-runner \
  --set gitlabUrl=https://gitlab.com/ \
  --set runnerRegistrationToken=YOUR_TOKEN \
  --set runners.executor=kubernetes \
  --set runners.config="
    [[runners]]
      [runners.kubernetes]
        namespace = \"gitlab-ci\"
        cpu_request = \"500m\"
        memory_request = \"1Gi\"
  "
```

---

## Runner Configuration

```toml
# /etc/gitlab-runner/config.toml
concurrent = 10
check_interval = 3

[[runners]]
  name = "Docker Runner"
  url = "https://gitlab.com/"
  token = "RUNNER_TOKEN"
  executor = "docker"

  [runners.docker]
    image = "node:20-alpine"
    privileged = false
    volumes = ["/cache"]
    pull_policy = ["if-not-present"]
    shm_size = 268435456         # 256MB shared memory

  [runners.cache]
    Type = "s3"
    Shared = true
    [runners.cache.s3]
      ServerAddress = "s3.amazonaws.com"
      BucketName = "gitlab-runner-cache"
      BucketLocation = "us-east-1"
```

---

## Tags & Job Assignment

```yaml
# .gitlab-ci.yml
build:
  tags:
    - docker          # Runs on runner tagged "docker"
  script: npm run build

deploy:
  tags:
    - shell
    - production      # Runs on runner with BOTH tags
  script: ./deploy.sh

gpu-training:
  tags:
    - gpu
    - cuda            # Runs on GPU runner
  script: python train.py
```

```
  JOB → RUNNER MATCHING
  ═════════════════════

  Job tags: [docker, linux]
  
  Runner A tags: [docker, linux, node]     ← MATCH ✓
  Runner B tags: [docker, windows]         ← NO MATCH ✕
  Runner C tags: [shell, linux]            ← NO MATCH ✕
  
  Rule: Runner must have ALL job tags
```

---

## Services (Sidecar Containers)

```yaml
# Database service for integration tests
integration-test:
  stage: test
  image: node:20
  services:
    - name: postgres:16
      alias: db
      variables:
        POSTGRES_DB: test_db
        POSTGRES_USER: test
        POSTGRES_PASSWORD: test_pass
    - name: redis:7-alpine
      alias: cache
  variables:
    DATABASE_URL: "postgres://test:test_pass@db:5432/test_db"
    REDIS_URL: "redis://cache:6379"
  script:
    - npm ci
    - npm run test:integration
```

---

## Auto-Scaling Runners

```
  AUTO-SCALING WITH DOCKER MACHINE
  ═════════════════════════════════

  Idle state: 1 runner
            │
  Jobs queued ──► Spin up VMs (max 10)
            │        │
  Jobs complete ──► Scale down after idle timeout
            │
  Back to: 1 runner

  BEST FOR:
  • Variable workload (busy during work hours)
  • Cost optimization (pay only when building)
```

```toml
# config.toml — Docker Machine autoscaling
[[runners]]
  executor = "docker+machine"
  limit = 10

  [runners.machine]
    IdleCount = 1
    IdleTime = 1800          # 30 min before removing idle machines
    MaxBuilds = 50           # Recreate after 50 builds
    MachineDriver = "amazonec2"
    MachineName = "gitlab-runner-%s"
    MachineOptions = [
      "amazonec2-instance-type=m5.large",
      "amazonec2-region=us-east-1",
      "amazonec2-vpc-id=vpc-xxx",
      "amazonec2-subnet-id=subnet-xxx",
    ]

    [[runners.machine.autoscaling]]
      Periods = ["* * 9-17 * * mon-fri *"]   # Work hours
      IdleCount = 5
      IdleTime = 3600
      Timezone = "America/New_York"

    [[runners.machine.autoscaling]]
      Periods = ["* * * * * sat,sun *"]       # Weekends
      IdleCount = 0
      IdleTime = 300
```

---

## Real-World Scenario

> **Scenario**: A company needs runners that scale up during work hours and scale down overnight/weekends to control costs.
>
> **Solution**: Use Docker Machine executor with AWS EC2. Configure autoscaling periods: 5 idle machines during business hours (9-17 Mon-Fri), 0 idle machines on weekends. Max 10 concurrent machines. Each machine handles 50 builds before replacement (prevents drift). S3-based distributed cache for fast builds across machines.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Job stuck "pending" | No runner with matching tags | Check job tags match runner tags; register new runner |
| "No runner available" | Runner offline | Check `gitlab-runner status`; restart service |
| Docker pull fails | Network or auth issue | Check Docker login; use `pull_policy: if-not-present` |
| Service container unreachable | Wrong hostname | Use `alias:` as hostname; check service health |
| Build cache not working | Cache not configured | Configure S3/GCS cache in `config.toml` |
| "too many requests" (Docker Hub) | Rate limit | Use mirror registry or authenticated pulls |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **Shared Runners** | GitLab-managed, available to all projects |
| **Group Runners** | Self-managed, scoped to GitLab group |
| **Project Runners** | Dedicated to a single project |
| **Docker Executor** | Isolated containers per job (recommended) |
| **Kubernetes Executor** | Jobs as K8s pods (auto-scaling) |
| **Tags** | Match jobs to specific runners |
| **Services** | Sidecar containers (databases, caches) |
| **Auto-scaling** | Docker Machine or Kubernetes for elastic capacity |

---

## Quick Revision Questions

1. **What are the three scopes of GitLab Runners?** *Shared (all projects), Group (group projects), and Project (single project).*
2. **Which executor is recommended for most teams?** *Docker executor — it provides isolation, reproducibility, and supports any Docker image.*
3. **How do job tags work?** *Jobs specify required tags; a runner must have ALL listed tags to pick up the job.*
4. **What are services in GitLab CI?** *Sidecar containers (databases, caches) that run alongside the job container, accessible by hostname/alias.*
5. **How does auto-scaling work with Docker Machine?** *GitLab Runner creates/destroys VMs on-demand based on job queue length, with configurable idle counts and time periods.*
6. **What is the `config.toml` file?** *The GitLab Runner configuration file that defines executor settings, concurrency, cache, and autoscaling parameters.*

---

[← Previous: Stages & Jobs](02-stages-jobs.md) | [Back to README](../README.md) | [Next: Variables & Secrets →](04-variables-secrets.md)

# Chapter 4: Container Orchestration

> **Unit 10: Docker in Production** | [Course Home](../README.md)

---

## Overview

Container orchestration automates the deployment, scaling, networking, and management of containerised applications across clusters of machines. This chapter explains why orchestration is essential for production and compares the major platforms.

---

## 4.1 Why Orchestration Is Needed

```
  Single Host vs Orchestrated Cluster:
  
  Single Docker Host:              Orchestrated Cluster:
  +----------------+              +------+  +------+  +------+
  | web  api  db   |              |Node 1|  |Node 2|  |Node 3|
  |                |              | web  |  | web  |  | api  |
  | Single point   |              | api  |  | api  |  | api  |
  | of failure!    |              +------+  +------+  +------+
  +----------------+                   |        |        |
                                  +----+--------+--------+----+
                                  |     Orchestrator           |
                                  | Scheduling, Scaling,       |
                                  | Self-healing, Networking   |
                                  +----------------------------+
```

**Problems orchestration solves:**

| Problem | Single Host | With Orchestration |
|---------|------------|-------------------|
| Host failure | Application down | Auto-reschedule to healthy node |
| Scaling | Manual docker run | `replicas: 10` → auto-distributed |
| Load balancing | External setup needed | Built-in service discovery + LB |
| Updates | Manual stop/start | Rolling updates with zero downtime |
| Networking | Single bridge network | Multi-host overlay networking |
| Secrets | Files or env vars | Encrypted, distributed secret store |

---

## 4.2 Core Orchestration Concepts

```
  Orchestration Components:
  
  +===============================================+
  |              Control Plane                     |
  |  +----------+  +---------+  +-------------+   |
  |  | Scheduler |  | API     |  | State Store |   |
  |  | (place    |  | Server  |  | (desired vs |   |
  |  |  workload)|  | (entry) |  |  actual)    |   |
  |  +----------+  +---------+  +-------------+   |
  +====================+=========================+=+
                       |                         |
           +-----------v-----------+  +----------v----------+
           |      Worker Node 1    |  |     Worker Node 2   |
           | +------+  +--------+ |  | +------+ +--------+ |
           | | Pod/ |  | Pod/   | |  | | Pod/ | | Pod/   | |
           | | Task |  | Task   | |  | | Task | | Task   | |
           | +------+  +--------+ |  | +------+ +--------+ |
           | Agent | Runtime      |  | Agent | Runtime      |
           +-----------------------+  +----------------------+
```

**Key terms across platforms:**

| Concept | Docker Swarm | Kubernetes | ECS |
|---------|-------------|------------|-----|
| **Cluster** | Swarm | Cluster | Cluster |
| **Manager** | Manager node | Control plane | Control plane (managed) |
| **Worker** | Worker node | Worker node | Container instance |
| **Work unit** | Task | Pod | Task |
| **Grouping** | Service | Deployment | Service |
| **Config file** | Compose / Stack | YAML Manifests | Task Definition |
| **Networking** | Overlay | CNI plugins | awsvpc |

---

## 4.3 Desired State and Reconciliation

```
  Desired State Reconciliation Loop:
  
  User declares:                   Orchestrator:
  "I want 3 replicas              Continuously compares
   of my web service"             desired vs actual state
       |                               |
       v                               v
  +-----------+                  +-----------+
  | Desired   |   !=             | Actual    |
  | State: 3  | --------+       | State: 2  |
  +-----------+         |       +-----------+
                        v
                  +-----------+
                  | Schedule  |
                  | 1 more    |
                  | container |
                  +-----------+
                        |
                        v
                  +-----------+
                  | Actual    |
                  | State: 3  | ← matches desired
                  +-----------+
  
  This loop runs CONTINUOUSLY:
  - Container crashes → restart or reschedule
  - Node dies → reschedule all tasks to other nodes
  - Scale request → add/remove containers
```

---

## 4.4 Orchestration Platforms Compared

```
  +==================+==================+=================+
  |  Docker Swarm    |  Kubernetes (K8s)|  Amazon ECS     |
  +==================+==================+=================+
  | Built into       | Industry         | AWS-managed     |
  | Docker Engine    | standard         | control plane   |
  +------------------+------------------+-----------------+
  | Easy to set up   | Complex but      | Deep AWS        |
  | (1 command)      | very powerful    | integration     |
  +------------------+------------------+-----------------+
  | Docker Compose   | YAML manifests   | Task            |
  | file compatible  | (many resources) | Definitions     |
  +------------------+------------------+-----------------+
  | Good for small/  | Enterprise-grade | Best for AWS-   |
  | medium clusters  | any scale        | native workloads|
  +------------------+------------------+-----------------+
  | Limited          | Huge ecosystem   | Limited to      |
  | ecosystem        | (Helm, Operators)| AWS ecosystem   |
  +==================+==================+=================+
```

### Other Platforms

| Platform | Key Strength |
|----------|-------------|
| **HashiCorp Nomad** | Multi-workload (containers, VMs, batch jobs), lightweight |
| **Docker Swarm** | Simplicity, built into Docker, uses Compose files |
| **Kubernetes** | Industry standard, massive ecosystem, any cloud |
| **Amazon ECS** | Tight AWS integration, Fargate serverless option |
| **Azure Container Apps** | Managed Kubernetes with simplified UX |
| **Google Cloud Run** | Serverless containers, scale-to-zero |

---

## 4.5 Choosing the Right Platform

```
  Decision Tree:
  
  Start
    |
    v
  Small team (< 10 devs)?
    |           |
    Yes         No
    |           |
    v           v
  < 20        Multi-cloud or
  containers?  on-prem needed?
    |    |       |        |
    Yes  No      Yes      No
    |    |       |        |
    v    v       v        v
  Docker Docker  Kubernetes  AWS-only?
  Compose Swarm  (managed     |     |
  (single (small  or self-    Yes   No
   host)  cluster) hosted)    |     |
                              v     v
                            ECS   Kubernetes
                           +Fgate  (managed)
```

```
  Complexity vs Capability:
  
  Capability
  ^
  |                                    * Kubernetes
  |                              *  
  |                        * ECS / ACA
  |                  *
  |            * Docker Swarm
  |      *
  | * Docker Compose
  +---------------------------------->
                              Complexity
  
  Choose the SIMPLEST tool that meets your needs!
```

---

## 4.6 Common Orchestration Features

```yaml
# Scaling
replicas: 5             # Run 5 instances

# Health checks
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/health"]
  interval: 30s
  retries: 3

# Update strategy
update_config:
  parallelism: 2          # Update 2 at a time
  delay: 10s              # Wait between batches
  failure_action: rollback
  order: start-first      # Start new before stopping old

# Rollback
rollback_config:
  parallelism: 1
  delay: 5s

# Placement constraints
placement:
  constraints:
    - node.role == worker
    - node.labels.zone == eu-west-1a

# Resource limits
resources:
  limits:
    cpus: "2.0"
    memory: 1G
  reservations:
    cpus: "0.5"
    memory: 256M
```

---

## Summary Table

| Feature | Docker Compose | Docker Swarm | Kubernetes |
|---------|---------------|-------------|------------|
| **Scope** | Single host | Multi-host | Multi-host |
| **Setup** | Instant | 1 command | Complex |
| **Scaling** | Manual | Built-in | Built-in + auto |
| **Self-healing** | Restart only | Reschedule | Reschedule + custom |
| **Networking** | Bridge | Overlay | CNI (flexible) |
| **Load balancing** | None built-in | Built-in (L4) | Ingress (L7) |
| **Secrets** | File-based | Encrypted store | Encrypted store |
| **Rolling updates** | Manual | Built-in | Built-in |
| **Config format** | compose.yml | Stack file | YAML manifests |
| **Learning curve** | Low | Medium | High |

---

## Quick Revision Questions

1. **What problem does container orchestration solve that Docker alone cannot?**
   - Docker manages containers on a single host. Orchestration distributes containers across multiple hosts, handles node failures, provides service discovery, enables rolling updates, and automates scaling

2. **Explain the desired state reconciliation loop.**
   - You declare the desired state (e.g., 3 replicas). The orchestrator continuously compares it against actual state. If they differ (container crash, node failure), it takes action to reach the desired state

3. **When would you choose Docker Swarm over Kubernetes?**
   - Small to medium deployments where simplicity matters, teams already using Docker Compose, when you need quick cluster setup without the operational complexity of Kubernetes

4. **What is the difference between a control plane and worker nodes?**
   - The control plane runs the scheduler, API server, and state store — it decides where to run workloads. Worker nodes run the actual container workloads, reporting status back to the control plane

5. **Why is the advice "choose the simplest tool that meets your needs" important?**
   - Orchestration platforms add operational complexity. Running Kubernetes for 5 containers is overkill. Docker Compose or Swarm may be sufficient and much simpler to operate, saving engineering time

---

[← Previous: CI/CD with Docker](03-ci-cd-with-docker.md) | [Next: Docker Swarm →](05-docker-swarm.md)

# Microservices

## Unit 2: Web Application Architecture — Topic 6

## 🎯 Overview

Microservices architecture decomposes applications into small, independent services communicating via APIs. While this offers scalability and flexibility, it dramatically expands the attack surface. Penetration testers must understand inter-service communication, service discovery, container security, and the unique vulnerabilities of distributed systems.

---

## 1. Microservices Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                  MICROSERVICES ARCHITECTURE                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client ──→ API Gateway ──┬──→ Auth Service (:8001)         │
│                           ├──→ User Service (:8002)         │
│                           ├──→ Order Service (:8003)        │
│                           ├──→ Payment Service (:8004)      │
│                           ├──→ Notification Service (:8005) │
│                           └──→ Search Service (:8006)       │
│                                                              │
│             ┌──────────┐                                     │
│             │ Message   │                                     │
│             │ Queue     │ (RabbitMQ, Kafka)                  │
│             │ ┌──┐ ┌──┐│                                     │
│             │ │Q1│ │Q2││                                     │
│             │ └──┘ └──┘│                                     │
│             └──────────┘                                     │
│                                                              │
│  Each service has its own:                                   │
│  • Database       • Deployment     • Tech stack              │
│  • API            • Scaling        • Team                    │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Attack Surface Expansion

```
┌──────────────────────────────────────────────────────────────┐
│           MONOLITH vs MICROSERVICES ATTACK SURFACE            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Monolith:                   Microservices:                  │
│  ┌──────────────┐           ┌───┐ ┌───┐ ┌───┐ ┌───┐       │
│  │              │           │ S1│─│ S2│─│ S3│─│ S4│       │
│  │   Single     │           └─┬─┘ └─┬─┘ └─┬─┘ └─┬─┘       │
│  │   App        │             │     │     │     │           │
│  │              │           ┌─┴─┐ ┌─┴─┐ ┌─┴─┐ ┌─┴─┐       │
│  │  1 entry     │           │DB1│ │DB2│ │DB3│ │DB4│       │
│  │  point       │           └───┘ └───┘ └───┘ └───┘       │
│  └──────────────┘                                           │
│                                                              │
│  Attack surface:             Attack surface:                 │
│  • 1 application             • N services                   │
│  • 1 database                • N databases                  │
│  • 1 network boundary        • N×N inter-service comms      │
│  • 1 auth system             • Multiple auth mechanisms     │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Inter-Service Communication Attacks

### Service-to-Service Trust Issues

```bash
# Internal services often trust each other without auth
# If attacker gains access to one service, lateral movement is easy

# SSRF to reach internal services
curl "https://target.com/api/proxy?url=http://user-service:8002/admin"
curl "https://target.com/api/proxy?url=http://payment-service:8004/refund"

# Internal APIs often lack auth
# External: POST /api/transfer → requires Bearer token
# Internal: POST http://payment-service:8004/transfer → no auth!

# Service discovery enumeration
# Kubernetes DNS: service-name.namespace.svc.cluster.local
curl http://user-service.default.svc.cluster.local:8002/users
```

### Message Queue Attacks

```bash
# If message queues are accessible:
# RabbitMQ management (default: guest/guest)
curl http://target:15672/api/queues

# Kafka — consume messages
kafka-console-consumer --bootstrap-server target:9092 \
  --topic user-events --from-beginning

# Message injection — send malicious messages
# If input isn't validated at the consumer side
```

---

## 4. Container Security

```
┌──────────────────────────────────────────────────────────────┐
│                  CONTAINER ATTACK SURFACE                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Host OS                                                     │
│  ┌────────────────────────────────────────────────────┐      │
│  │  Docker Engine                                     │      │
│  │  ┌──────────────┐  ┌──────────────┐               │      │
│  │  │  Container 1 │  │  Container 2 │               │      │
│  │  │  App + Deps  │  │  App + Deps  │               │      │
│  │  │              │  │              │               │      │
│  │  │ Vuln image?  │  │ Secrets in   │               │      │
│  │  │ Root user?   │  │ env vars?    │               │      │
│  │  │ Mounted sock?│  │ Privesc?     │               │      │
│  │  └──────────────┘  └──────────────┘               │      │
│  └────────────────────────────────────────────────────┘      │
│                                                              │
│  Attack vectors:                                              │
│  • Vulnerable base images (outdated packages)                │
│  • Docker socket mounted (/var/run/docker.sock)              │
│  • Running as root inside container                          │
│  • Secrets in environment variables or images                │
│  • Container escape via kernel exploits                      │
│  • Privileged containers                                      │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Check if inside a container
cat /proc/1/cgroup | grep docker
ls /.dockerenv

# Docker socket mounted? → container escape
ls -la /var/run/docker.sock
# If accessible:
curl --unix-socket /var/run/docker.sock http://localhost/containers/json

# Check environment for secrets
env | grep -iE "(pass|secret|key|token|api)"

# Kubernetes service account token
cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

---

## 5. API Gateway Security

```
┌──────────────────────────────────────────────────────────────┐
│                   API GATEWAY                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client ──→ ┌────────────────────────┐ ──→ Services         │
│             │      API Gateway       │                       │
│             │  • Authentication      │                       │
│             │  • Rate limiting       │                       │
│             │  • Request routing     │                       │
│             │  • SSL termination     │                       │
│             │  • Logging/monitoring  │                       │
│             └────────────────────────┘                       │
│                                                              │
│  Common gateways: Kong, NGINX, AWS API Gateway, Traefik     │
│                                                              │
│  Testing:                                                    │
│  • Bypass gateway (direct service access)                    │
│  • Path traversal through routing rules                      │
│  • Rate limit bypass via different paths                     │
│  • Header manipulation for auth bypass                       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Bypass API gateway — access services directly
nmap -p 8001-8010 target.com  # Find exposed service ports

# Path confusion
curl https://target.com/api/admin/../users  # Routing bypass
curl https://target.com/API/admin           # Case sensitivity
```

---

## 6. Service Mesh and Kubernetes

```bash
# Kubernetes dashboard (if exposed)
curl https://target.com:443/api/v1/namespaces
curl https://target.com:10250/pods  # Kubelet API

# etcd (key-value store) — contains all cluster state
curl https://target.com:2379/v2/keys/?recursive=true

# Helm Tiller (legacy) — deploy arbitrary workloads
curl http://target.com:44134

# Common Kubernetes misconfigurations:
# • Dashboard exposed without auth
# • RBAC overly permissive
# • Pod security policies missing
# • Network policies not configured (all pods can communicate)
```

---

## 📊 Summary Table

| Component | Attack Vector | Testing Approach |
|-----------|-------------|-----------------|
| API Gateway | Bypass, path confusion | Direct service access |
| Inter-service comm | SSRF, missing auth | Internal endpoint access |
| Message queues | Message injection | Queue enumeration |
| Containers | Escape, secrets, vulns | Environment inspection |
| Service discovery | Internal endpoint enum | DNS/API enumeration |
| Kubernetes | Dashboard, etcd, RBAC | Port scanning, API access |

---

## ❓ Revision Questions

1. How does microservices architecture expand the attack surface compared to monoliths?
2. What is the risk of SSRF in a microservices environment?
3. How can a mounted Docker socket lead to container escape?
4. What role does an API gateway play in security, and how can it be bypassed?
5. What Kubernetes components are commonly exposed and exploitable?
6. Why might internal microservices lack proper authentication?

---

*Previous: [05-api-architectures.md](05-api-architectures.md) | Next: [07-cdn-and-caching.md](07-cdn-and-caching.md)*

---

*[Back to README](../README.md)*

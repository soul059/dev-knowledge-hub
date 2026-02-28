# Chapter 3: Load Balancing Concepts

## Overview

Load balancing distributes incoming network traffic across multiple backend servers to ensure high availability, reliability, and performance. It's one of the most critical components in production infrastructure. Every DevOps engineer must understand load balancer types, algorithms, health checks, and when to use each type.

---

## 3.1 Why Load Balancing?

```
┌──────────────────────────────────────────────────────────────┐
│              WITHOUT LOAD BALANCER                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  All traffic → Single Server                                 │
│  ┌────────┐        ┌──────────┐                              │
│  │ Users  │───────▶│ Server 1 │  ← Single point of failure! │
│  └────────┘        │ (100%    │  ← Overloaded!              │
│                    │  load)   │                              │
│                    └──────────┘                              │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│              WITH LOAD BALANCER                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────┐   ┌──────────────┐   ┌──────────┐               │
│  │        │   │              │──▶│ Server 1 │ (33% load)    │
│  │ Users  │──▶│ Load         │   └──────────┘               │
│  │        │   │ Balancer     │   ┌──────────┐               │
│  │        │   │              │──▶│ Server 2 │ (33% load)    │
│  │        │   │              │   └──────────┘               │
│  └────────┘   │              │   ┌──────────┐               │
│               │              │──▶│ Server 3 │ (33% load)    │
│               └──────────────┘   └──────────┘               │
│                                                              │
│  ✓ High availability  ✓ No single point of failure           │
│  ✓ Horizontal scaling ✓ Zero-downtime deployments            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 Types of Load Balancers

```
┌──────────────────────────────────────────────────────────────┐
│              LOAD BALANCER TYPES BY OSI LAYER                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 4 (Transport) Load Balancer:                          │
│  ┌─────────────────────────────────────────────┐             │
│  │ ├── Routes based on: IP + Port              │             │
│  │ ├── Does NOT inspect content                │             │
│  │ ├── Very fast, low latency                  │             │
│  │ ├── TCP/UDP level                           │             │
│  │ ├── Examples: AWS NLB, HAProxy (TCP mode)   │             │
│  │ └── Use: Databases, TCP services, gaming    │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  Layer 7 (Application) Load Balancer:                        │
│  ┌─────────────────────────────────────────────┐             │
│  │ ├── Routes based on: URL, headers, cookies  │             │
│  │ ├── Inspects HTTP content                   │             │
│  │ ├── More features, slightly more latency    │             │
│  │ ├── HTTP/HTTPS level                        │             │
│  │ ├── Examples: AWS ALB, Nginx, HAProxy       │             │
│  │ ├── TLS termination                         │             │
│  │ └── Use: Web apps, APIs, microservices      │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### AWS Load Balancer Comparison

```
┌──────────────────┬──────────────┬──────────────┬──────────────┐
│ Feature          │ ALB          │ NLB          │ CLB (Legacy) │
├──────────────────┼──────────────┼──────────────┼──────────────┤
│ Layer            │ 7 (HTTP)     │ 4 (TCP/UDP)  │ 4 & 7        │
│ Protocol         │ HTTP/HTTPS   │ TCP/UDP/TLS  │ TCP/HTTP     │
│ Path-based       │ Yes          │ No           │ No           │
│ Host-based       │ Yes          │ No           │ No           │
│ WebSocket        │ Yes          │ Yes          │ No           │
│ Static IP        │ No           │ Yes          │ No           │
│ Performance      │ Good         │ Millions rps │ Moderate     │
│ TLS termination  │ Yes          │ Yes          │ Yes          │
│ Health checks    │ HTTP/HTTPS   │ TCP/HTTP     │ TCP/HTTP     │
│ Sticky sessions  │ Yes          │ Yes          │ Yes          │
│ gRPC             │ Yes          │ Yes          │ No           │
└──────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 3.3 Load Balancing Algorithms

```
┌──────────────────────────────────────────────────────────────┐
│              LOAD BALANCING ALGORITHMS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Round Robin (Default)                                    │
│     S1 → S2 → S3 → S1 → S2 → S3 → ...                     │
│     Equal distribution, ignores server capacity              │
│                                                              │
│  2. Weighted Round Robin                                     │
│     S1(w=3) → S1 → S1 → S2(w=1) → S1 → S1 → S1 → S2      │
│     More powerful servers get more traffic                   │
│                                                              │
│  3. Least Connections                                        │
│     Routes to server with fewest active connections          │
│     Best for: long-lived connections, WebSockets             │
│                                                              │
│  4. IP Hash                                                  │
│     hash(client_IP) → consistent server assignment           │
│     Same client always goes to same server                   │
│     Best for: session persistence without cookies            │
│                                                              │
│  5. Least Response Time                                      │
│     Routes to server with fastest response time              │
│     Best for: variable workloads                             │
│                                                              │
│  6. Random                                                   │
│     Random server selection                                  │
│     Simple but unpredictable                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.4 Health Checks

```
┌──────────────────────────────────────────────────────────────┐
│              HEALTH CHECK FLOW                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Load Balancer periodically checks each backend:             │
│                                                              │
│  LB ──── GET /health ────▶ Server 1 → 200 OK ✓ (healthy)   │
│  LB ──── GET /health ────▶ Server 2 → 200 OK ✓ (healthy)   │
│  LB ──── GET /health ────▶ Server 3 → timeout ✗ (unhealthy)│
│                                                              │
│  ┌────────────────────────────────────────┐                  │
│  │  Server 3 removed from rotation!       │                  │
│  │  Traffic only goes to Server 1 & 2     │                  │
│  │                                        │                  │
│  │  Once Server 3 passes health checks    │                  │
│  │  again, it's added back automatically  │                  │
│  └────────────────────────────────────────┘                  │
│                                                              │
│  Health Check Parameters:                                    │
│  ├── Path: /health or /healthz                               │
│  ├── Interval: 30 seconds                                    │
│  ├── Timeout: 5 seconds                                      │
│  ├── Healthy threshold: 3 consecutive successes              │
│  ├── Unhealthy threshold: 2 consecutive failures             │
│  └── Success codes: 200-299                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.5 Layer 7 Routing (Path & Host-Based)

```
┌──────────────────────────────────────────────────────────────┐
│        ALB PATH-BASED AND HOST-BASED ROUTING                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                    ┌──────────┐                               │
│  Users ──────────▶│   ALB    │                               │
│                    └────┬─────┘                               │
│                         │                                    │
│         ┌───────────────┼───────────────┐                    │
│         │               │               │                    │
│    /api/*          /images/*        /*  (default)             │
│         │               │               │                    │
│    ┌────▼────┐    ┌─────▼────┐    ┌─────▼────┐              │
│    │ API     │    │ Static   │    │ Frontend │              │
│    │ Servers │    │ Content  │    │ Servers  │              │
│    │(Target  │    │(S3/CDN)  │    │(Target   │              │
│    │ Group 1)│    │          │    │ Group 2) │              │
│    └─────────┘    └──────────┘    └──────────┘              │
│                                                              │
│  Host-based:                                                 │
│  api.example.com → API servers                               │
│  www.example.com → Frontend servers                          │
│  admin.example.com → Admin panel                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.6 Configuration Examples

### Nginx Load Balancer

```nginx
# upstream block defines backend servers
upstream backend_servers {
    least_conn;                    # Algorithm: least connections
    
    server 10.0.1.10:8080 weight=3;  # More traffic
    server 10.0.1.11:8080 weight=1;
    server 10.0.1.12:8080 backup;    # Only if others are down
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Health check (Nginx Plus only)
        # health_check interval=10 fails=3 passes=2;
    }
}
```

### Terraform: AWS ALB

```hcl
resource "aws_lb" "app" {
  name               = "app-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = aws_subnet.public[*].id
}

resource "aws_lb_target_group" "app" {
  name     = "app-targets"
  port     = 8080
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id

  health_check {
    path                = "/health"
    healthy_threshold   = 3
    unhealthy_threshold = 2
    timeout             = 5
    interval            = 30
    matcher             = "200"
  }
}

resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.app.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = aws_acm_certificate.main.arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

---

## 3.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| All targets unhealthy | Health check path wrong, app not running | Verify health endpoint, check app logs |
| Uneven traffic distribution | Sticky sessions, client-side caching | Review algorithm, check cookie settings |
| 502 Bad Gateway | Backend crashed or unreachable | Check backend servers, security groups |
| 504 Gateway Timeout | Backend too slow | Increase timeout, optimize backend |
| Connection draining issues | Old connections persist | Configure deregistration delay |
| High latency | TLS termination + complex routing | Check ALB metrics, optimize |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Load Balancer | Distributes traffic across multiple servers |
| Layer 4 (NLB) | TCP/UDP level, very fast, static IP |
| Layer 7 (ALB) | HTTP level, path/host routing, TLS termination |
| Round Robin | Default algorithm, equal distribution |
| Least Connections | Best for variable-duration requests |
| Health Checks | Automatically remove unhealthy servers |
| Sticky Sessions | Same client → same server (via cookies) |
| TLS Termination | LB handles SSL, backend gets plain HTTP |

---

## Quick Revision Questions

1. **What's the difference between Layer 4 and Layer 7 load balancing?**
2. **When would you choose an NLB over an ALB in AWS?**
3. **Explain how health checks ensure high availability.**
4. **What load balancing algorithm would you use for WebSocket connections? Why?**
5. **How does path-based routing work in an ALB? Give an example.**
6. **What does "TLS termination at the load balancer" mean, and why is it beneficial?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Routing Basics](02-routing-basics.md) | [README](../README.md) | [Firewalls & Security Groups →](04-firewalls-security-groups.md) |

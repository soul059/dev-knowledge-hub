# Chapter 2.5: Elastic Load Balancing (ALB, NLB)

## Overview

Elastic Load Balancing (ELB) distributes incoming traffic across multiple targets (EC2, containers, Lambda, IPs) in one or more Availability Zones. It's essential for **high availability**, **fault tolerance**, and **scalability**. AWS offers three types — ALB, NLB, and GLB.

---

## Load Balancer Types

```
┌─────────────────────────────────────────────────────────────────┐
│                  AWS LOAD BALANCER TYPES                        │
│                                                                 │
│  ┌───────────────────┐  ┌───────────────────┐  ┌────────────┐ │
│  │  ALB              │  │  NLB              │  │  GLB       │ │
│  │  (Application)    │  │  (Network)        │  │ (Gateway)  │ │
│  │  Layer 7 (HTTP)   │  │  Layer 4 (TCP)    │  │ Layer 3    │ │
│  │                   │  │                   │  │ (IP)       │ │
│  │  ✓ Path routing   │  │  ✓ Ultra-low      │  │            │ │
│  │  ✓ Host routing   │  │    latency        │  │ ✓ Firewall│ │
│  │  ✓ HTTP headers   │  │  ✓ Static IP      │  │ ✓ IDS/IPS │ │
│  │  ✓ Query strings  │  │  ✓ Millions of    │  │ ✓ Deep    │ │
│  │  ✓ Redirects      │  │    requests/sec   │  │   inspect  │ │
│  │  ✓ Fixed response │  │  ✓ TCP/UDP/TLS    │  │            │ │
│  │  ✓ WebSocket      │  │  ✓ WebSocket      │  │ 3rd party │ │
│  │  ✓ Lambda targets │  │                   │  │ appliances│ │
│  └───────────────────┘  └───────────────────┘  └────────────┘ │
│                                                                 │
│  USE:  90% of cases     High-perf/gaming     Security virtual  │
│        Web apps         IoT, UDP             appliances         │
└─────────────────────────────────────────────────────────────────┘
```

### Comparison Table

| Feature | ALB | NLB | GLB |
|---------|-----|-----|-----|
| **OSI Layer** | 7 (HTTP/HTTPS) | 4 (TCP/UDP/TLS) | 3 (IP) |
| **Protocol** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | IP (GENEVE) |
| **Routing** | Path, host, header, query | Port-based | Transparent |
| **Static IP** | ✗ (use Global Accelerator) | ✓ | ✗ |
| **Latency** | ~400ms | ~100μs | N/A |
| **WebSocket** | ✓ | ✓ | ✗ |
| **Lambda Targets** | ✓ | ✗ | ✗ |
| **SSL Termination** | ✓ | ✓ | ✗ |
| **Sticky Sessions** | ✓ | ✓ | ✗ |
| **Cross-Zone LB** | ✓ (default on) | ✓ (default off) | ✓ |
| **Pricing** | $$  | $$ | $$ |

---

## Application Load Balancer (ALB) — Deep Dive

```
┌──────────────────────────── ALB Architecture ──────────────────┐
│                                                                 │
│  Internet ──► ALB ──► Listener (Port 443) ──► Rules ──► TG    │
│                                                                 │
│  ┌─────────────┐    ┌──────────────────────────────────────┐   │
│  │  LISTENERS  │    │           ROUTING RULES              │   │
│  │  ─────────  │    │                                      │   │
│  │  Port 80    │    │  IF path = /api/*                    │   │
│  │  Port 443   │    │  THEN → Target Group: api-tg         │   │
│  │             │    │                                      │   │
│  │  Protocol:  │    │  IF host = admin.app.com             │   │
│  │  HTTP/HTTPS │    │  THEN → Target Group: admin-tg       │   │
│  └─────────────┘    │                                      │   │
│                     │  IF header X-Custom = special        │   │
│                     │  THEN → Target Group: special-tg     │   │
│                     │                                      │   │
│                     │  DEFAULT → Target Group: web-tg      │   │
│                     └──────────────────────────────────────┘   │
│                                                                 │
│  ┌──────────────── Target Groups ───────────────────────────┐  │
│  │                                                           │  │
│  │  web-tg (EC2)        api-tg (ECS)       lambda-tg        │  │
│  │  ┌────┐ ┌────┐      ┌────┐ ┌────┐     ┌────────────┐   │  │
│  │  │EC2 │ │EC2 │      │Task│ │Task│     │  Lambda    │   │  │
│  │  │ :80│ │ :80│      │:3000│:3000│     │  Function  │   │  │
│  │  └────┘ └────┘      └────┘ └────┘     └────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Create an ALB

```bash
# 1. Create the ALB
aws elbv2 create-load-balancer \
  --name webapp-alb \
  --subnets subnet-aaa subnet-bbb subnet-ccc \
  --security-groups sg-alb \
  --scheme internet-facing \
  --type application

# 2. Create Target Group
aws elbv2 create-target-group \
  --name webapp-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-xxx \
  --target-type instance \
  --health-check-path "/health" \
  --health-check-interval-seconds 30 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3

# 3. Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:...:targetgroup/webapp-tg/xxx \
  --targets Id=i-aaa Id=i-bbb

# 4. Create HTTPS Listener with SSL certificate
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/webapp-alb/xxx \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:...:certificate/xxx \
  --default-actions Type=forward,TargetGroupArn=arn:...:targetgroup/webapp-tg/xxx

# 5. HTTP to HTTPS redirect
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:...:loadbalancer/app/webapp-alb/xxx \
  --protocol HTTP \
  --port 80 \
  --default-actions '[{
    "Type": "redirect",
    "RedirectConfig": {
      "Protocol": "HTTPS",
      "Port": "443",
      "StatusCode": "HTTP_301"
    }
  }]'
```

### ALB Path-Based Routing

```bash
# Route /api/* to API target group
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/xxx \
  --conditions '[{
    "Field": "path-pattern",
    "PathPatternConfig": {"Values": ["/api/*"]}
  }]' \
  --actions '[{
    "Type": "forward",
    "TargetGroupArn": "arn:...:targetgroup/api-tg/xxx"
  }]' \
  --priority 10

# Route based on hostname
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:...:listener/xxx \
  --conditions '[{
    "Field": "host-header",
    "HostHeaderConfig": {"Values": ["admin.myapp.com"]}
  }]' \
  --actions '[{
    "Type": "forward",
    "TargetGroupArn": "arn:...:targetgroup/admin-tg/xxx"
  }]' \
  --priority 20
```

---

## Network Load Balancer (NLB) — Deep Dive

```
┌──────────────────────── NLB Architecture ─────────────────────┐
│                                                                │
│  Client ──TCP──► NLB (Static IP per AZ) ──TCP──► Target      │
│                                                                │
│  Key Differences from ALB:                                    │
│  • Operates at Layer 4 (TCP/UDP)                              │
│  • Static IP per AZ (or Elastic IP)                           │
│  • Handles millions of requests per second                    │
│  • Ultra-low latency (~100 microseconds)                      │
│  • Preserves source IP (no X-Forwarded-For needed)           │
│  • TLS termination supported                                  │
│  • No Security Groups on NLB (control at target level)       │
│                                                                │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐              │
│  │  NLB     │     │  NLB     │     │  NLB     │              │
│  │  AZ-1a   │     │  AZ-1b   │     │  AZ-1c   │              │
│  │ 10.0.1.5 │     │ 10.0.2.5 │     │ 10.0.3.5 │              │
│  │(static)  │     │(static)  │     │(static)  │              │
│  └────┬─────┘     └────┬─────┘     └────┬─────┘              │
│       │                │                │                     │
│       ▼                ▼                ▼                     │
│  ┌────────┐       ┌────────┐       ┌────────┐               │
│  │ EC2    │       │ EC2    │       │ EC2    │               │
│  │ :3000  │       │ :3000  │       │ :3000  │               │
│  └────────┘       └────────┘       └────────┘               │
└────────────────────────────────────────────────────────────────┘
```

### When to Use NLB over ALB

| Use Case | Use NLB |
|----------|---------|
| TCP/UDP protocols (not HTTP) | ✓ |
| Need static IP | ✓ |
| Need ultra-low latency | ✓ |
| Millions of connections | ✓ |
| Gaming, IoT, real-time | ✓ |
| Source IP preservation | ✓ |
| PrivateLink (expose service) | ✓ |

---

## Target Groups

```
┌──────────────────────────────────────────────────────────┐
│                  TARGET GROUP TYPES                       │
│                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐    │
│  │  Instance   │  │     IP      │  │   Lambda     │    │
│  │  (EC2 ID)   │  │  (Private)  │  │  (Function)  │    │
│  │             │  │             │  │              │    │
│  │ Most common │  │ Cross-VPC   │  │ ALB only     │    │
│  │ Auto-reg    │  │ On-prem via │  │ Serverless   │    │
│  │ with ASG    │  │ Direct Conn │  │              │    │
│  └─────────────┘  └─────────────┘  └──────────────┘    │
│                                                          │
│  Health Checks:                                          │
│  • Protocol: HTTP, HTTPS, TCP                           │
│  • Path: /health (for HTTP/HTTPS)                       │
│  • Interval: 30s (ALB) / 10-30s (NLB)                  │
│  • Healthy threshold: 2-10 consecutive successes        │
│  • Unhealthy threshold: 2-10 consecutive failures       │
└──────────────────────────────────────────────────────────┘
```

---

## SSL/TLS Termination

```
┌──────────────────────────────────────────────────────────────┐
│              SSL/TLS TERMINATION                             │
│                                                              │
│  Option 1: Terminate at ALB (most common)                   │
│                                                              │
│  Client ══HTTPS══► ALB ──HTTP──► EC2 (port 80)              │
│                    │                                         │
│                    └── ACM Certificate attached              │
│                        (Free, auto-renewed)                  │
│                                                              │
│  Option 2: End-to-end encryption                            │
│                                                              │
│  Client ══HTTPS══► ALB ══HTTPS══► EC2 (port 443)            │
│                    │              │                           │
│                    └── ACM Cert   └── Self-signed cert       │
│                                                              │
│  Option 3: Passthrough (NLB only)                           │
│                                                              │
│  Client ══TLS═══► NLB ══TLS═══► EC2 (port 443)              │
│                   │              │                            │
│                   No decryption  └── App handles TLS         │
└──────────────────────────────────────────────────────────────┘
```

### Request SSL Certificate from ACM

```bash
# Request a public certificate
aws acm request-certificate \
  --domain-name myapp.com \
  --subject-alternative-names "*.myapp.com" \
  --validation-method DNS

# List certificates
aws acm list-certificates
```

---

## Cross-Zone Load Balancing

```
┌──────────────────────────────────────────────────────────────┐
│          CROSS-ZONE LOAD BALANCING                           │
│                                                              │
│  WITHOUT Cross-Zone:            WITH Cross-Zone:            │
│                                                              │
│  AZ-a (2 instances)             AZ-a (2 instances)          │
│  50% traffic ÷ 2 = 25% each    50% ÷ 4 = 12.5% each       │
│                                                              │
│  AZ-b (2 instances)             AZ-b (2 instances)          │
│  50% traffic ÷ 2 = 25% each    50% ÷ 4 = 12.5% each       │
│                                                              │
│  ⚠ Uneven if AZs differ        ✓ Even distribution         │
│                                                              │
│  ALB: ON by default (no extra charge)                       │
│  NLB: OFF by default (inter-AZ data charges apply)          │
└──────────────────────────────────────────────────────────────┘
```

---

## Connection Draining (Deregistration Delay)

```
┌─────────────────────────────────────────────────────┐
│        CONNECTION DRAINING                          │
│                                                     │
│  instance marked unhealthy/deregistering            │
│       │                                             │
│       ▼                                             │
│  ┌──────────────┐                                   │
│  │ Draining     │  ← No NEW connections            │
│  │ (300s default│  ← Existing connections finish   │
│  │  configurable│  ← Then instance removed         │
│  └──────────────┘                                   │
│                                                     │
│  For fast deployments: set to 30-60 seconds        │
│  For long requests: increase to match max duration │
│  For WebSocket: increase significantly             │
└─────────────────────────────────────────────────────┘
```

---

## Sticky Sessions

```bash
# Enable cookie-based sticky sessions (ALB)
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:...:targetgroup/webapp-tg/xxx \
  --attributes \
    Key=stickiness.enabled,Value=true \
    Key=stickiness.type,Value=lb_cookie \
    Key=stickiness.lb_cookie.duration_seconds,Value=86400
```

---

## Real-World Scenarios

### Scenario 1: Microservices Routing with ALB
```
ALB ──► /api/users/*   ──► user-service-tg (ECS)
    ──► /api/orders/*  ──► order-service-tg (ECS)
    ──► /api/payments/*──► payment-service-tg (ECS)
    ──► /* (default)   ──► frontend-tg (S3/CloudFront)
```

### Scenario 2: Multi-Protocol Application
- ALB for HTTP/HTTPS traffic (web UI, REST API)
- NLB for TCP traffic (database connections, MQTT IoT)

### Scenario 3: Blue/Green Deployment
- Two target groups: Blue (current) and Green (new version)
- ALB listener rules point 100% to Blue
- Deploy new version to Green
- Shift traffic: 10% → 50% → 100% to Green
- Roll back by shifting back to Blue

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| 502 Bad Gateway | Target returning invalid response | Check target app is running; check security group |
| 503 Service Unavailable | No healthy targets | Check target health; check health check path |
| 504 Gateway Timeout | Target too slow to respond | Increase ALB idle timeout; optimize app |
| Uneven traffic distribution | Cross-zone disabled (NLB) | Enable cross-zone; balance instances across AZs |
| SSL certificate error | Cert not matching domain | Check ACM cert covers all domains/subdomains |
| Health checks fail | SG blocks health check traffic | Allow ALB SG ingress to target SG on health port |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **ALB** | Layer 7. HTTP routing by path/host/header. Best for web apps and microservices. |
| **NLB** | Layer 4. TCP/UDP. Static IP. Ultra-low latency. Best for gaming, IoT, real-time. |
| **Target Groups** | EC2 instances, IPs, or Lambda functions. Health checks determine routing. |
| **Listeners** | Define port + protocol. Rules route to target groups based on conditions. |
| **SSL/TLS** | Terminate at ALB with free ACM certificates. Auto-renewed. |
| **Cross-Zone** | Distributes traffic evenly across all targets in all AZs. ALB default on. |
| **Draining** | Graceful shutdown. Existing connections finish before target removed. |
| **Sticky Sessions** | Route same client to same target. Use for stateful apps (avoid if possible). |

---

## Quick Revision Questions

1. **What's the main difference between ALB and NLB?**
   > ALB operates at Layer 7 (HTTP) with content-based routing. NLB operates at Layer 4 (TCP/UDP) with ultra-low latency and static IPs.

2. **How does path-based routing work in ALB?**
   > Listener rules match the URL path (e.g., `/api/*`) and forward matching requests to specific target groups.

3. **What is connection draining and why is it important?**
   > When a target is deregistered, existing connections are allowed to complete (default 300s). Prevents dropped requests during deployments or scale-in.

4. **How do you get a free SSL certificate for your ALB?**
   > Use AWS Certificate Manager (ACM) to request a public certificate. It's free, auto-renewed, and attaches directly to ALB listeners.

5. **What happens if all targets in a target group are unhealthy?**
   > ALB returns 503 (Service Unavailable). Optionally, ALB can use "fail-open" behavior on health checks.

6. **Why would you use NLB instead of ALB for a microservice?**
   > When you need static IPs, TCP/UDP protocol support, ultra-low latency, or need to expose the service via AWS PrivateLink.

---

[← Previous: 2.4 Auto Scaling Groups](04-auto-scaling-groups.md) | [Next: 2.6 Lambda (Serverless) →](06-lambda-serverless.md)

[← Back to README](../README.md)

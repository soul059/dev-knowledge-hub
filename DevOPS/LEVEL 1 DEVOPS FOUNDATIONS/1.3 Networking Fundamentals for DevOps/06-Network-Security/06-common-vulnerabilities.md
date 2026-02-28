# Chapter 6: Common Network Vulnerabilities

## Overview

Understanding common network vulnerabilities is critical for DevOps engineers because you build and maintain the infrastructure attackers target. This chapter covers the most frequent attack vectors, how they work, how to detect them, and — most importantly — how to harden your systems against them.

---

## 6.1 Attack Landscape for DevOps

```
┌──────────────────────────────────────────────────────────────┐
│          COMMON ATTACK VECTORS IN DEVOPS                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│     │                                                        │
│     ├─ DDoS Attacks ────────────▶ Load Balancer / CDN        │
│     ├─ Port Scanning ──────────▶ Public-facing servers       │
│     ├─ SSL Stripping ─────────▶ Unprotected HTTP             │
│     ├─ DNS Hijacking ─────────▶ DNS resolvers                │
│     │                                                        │
│  Internal                                                    │
│     ├─ MITM (Man-in-the-Middle) ▶ Unencrypted traffic       │
│     ├─ ARP Spoofing ──────────▶ Local network               │
│     ├─ Lateral Movement ──────▶ Flat networks                │
│     ├─ Credential Theft ──────▶ Leaked secrets/tokens        │
│     │                                                        │
│  Application                                                 │
│     ├─ SSRF ───────────────────▶ Internal endpoints          │
│     ├─ DNS Rebinding ─────────▶ Internal services            │
│     └─ Container Escape ──────▶ Host system                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.2 Man-in-the-Middle (MITM)

```
┌──────────────────────────────────────────────────────────────┐
│           MAN-IN-THE-MIDDLE ATTACK                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal Communication:                                       │
│  ┌────────┐ ─────────────────────────▶ ┌────────┐            │
│  │ Client │                            │ Server │            │
│  └────────┘ ◀───────────────────────── └────────┘            │
│                                                              │
│  MITM Attack:                                                │
│  ┌────────┐    ┌──────────┐    ┌────────┐                    │
│  │ Client │───▶│ Attacker │───▶│ Server │                    │
│  │        │◀───│ (reads/  │◀───│        │                    │
│  └────────┘    │ modifies)│    └────────┘                    │
│                └──────────┘                                   │
│  Client thinks it's talking to Server directly               │
│                                                              │
│  Attack Methods:                                             │
│  - ARP spoofing (local network)                              │
│  - DNS spoofing (redirect to attacker's server)              │
│  - Rogue Wi-Fi (evil twin access point)                      │
│  - BGP hijacking (internet routing)                          │
│  - SSL stripping (downgrade HTTPS → HTTP)                    │
│                                                              │
│  Prevention:                                                 │
│  ✅ Use TLS/HTTPS everywhere                                  │
│  ✅ Enable HSTS (prevents SSL stripping)                      │
│  ✅ Use mTLS for service-to-service                           │
│  ✅ Certificate pinning for critical APIs                     │
│  ✅ DNSSEC to prevent DNS spoofing                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.3 DDoS (Distributed Denial of Service)

```
┌──────────────────────────────────────────────────────────────┐
│           DDoS ATTACK TYPES                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 3/4 (Volume-Based):                                   │
│  ┌────────┐                                                  │
│  │Botnet  │══╗                                               │
│  │ 1000s  │  ║  Flood with traffic                           │
│  │machines│  ║  (UDP, SYN, ICMP)                             │
│  └────────┘  ║                                               │
│              ╠══════▶ ┌────────┐  OVERWHELMED                │
│  ┌────────┐  ║        │ Target │  Bandwidth saturated        │
│  │ More   │══╝        │ Server │                             │
│  │ Bots   │           └────────┘                             │
│  └────────┘                                                  │
│                                                              │
│  Layer 7 (Application):                                      │
│  ┌────────┐                                                  │
│  │Attacker│                                                  │
│  │        │──── Slowloris (hold connections open)             │
│  │        │──── HTTP flood (valid-looking requests)           │
│  │        │──── API abuse (expensive queries)                │
│  └────────┘                                                  │
│                                                              │
│  DNS Amplification:                                          │
│  Attacker → small query → DNS resolver → LARGE response     │
│  (spoofed source IP = victim's IP)                           │
│  Amplification factor: 50-100x                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### DDoS Protection Layers

| Layer | Protection | Tool/Service |
|-------|-----------|-------------|
| Edge/CDN | Absorb traffic globally | CloudFront, Cloudflare, Akamai |
| AWS Shield | Auto L3/L4 protection | Shield Standard (free), Advanced ($) |
| WAF | L7 filtering, rate limiting | AWS WAF, Cloudflare WAF |
| Load Balancer | Distribute + absorb | ALB/NLB with auto-scaling |
| Rate Limiting | Limit requests per IP | Nginx `limit_req`, API Gateway throttle |
| Auto Scaling | Scale up during attacks | ASG, ECS auto-scaling |

```nginx
# Nginx rate limiting
http {
    # Define rate limit zone: 10 requests/second per IP
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    server {
        location /api/ {
            # Allow burst of 20, delay after 10
            limit_req zone=api burst=20 delay=10;
            limit_req_status 429;  # Return 429 Too Many Requests

            proxy_pass http://backend;
        }
    }
}
```

---

## 6.4 Port Scanning and Reconnaissance

```
┌──────────────────────────────────────────────────────────────┐
│          PORT SCANNING RECONNAISSANCE                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Attacker scans for open ports:                              │
│                                                              │
│  nmap -sV -sC target.com                                     │
│                                                              │
│  PORT     STATE    SERVICE                                   │
│  22/tcp   open     OpenSSH 8.9                               │
│  80/tcp   open     nginx 1.24                                │
│  443/tcp  open     nginx 1.24                                │
│  3306/tcp open     MySQL 8.0     ← Database exposed!         │
│  6379/tcp open     Redis 7.0     ← Cache exposed!            │
│  8080/tcp open     Jenkins 2.4   ← CI/CD exposed!            │
│  9090/tcp open     Prometheus    ← Monitoring exposed!       │
│                                                              │
│  ⚠️ Every open port is an attack surface                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Hardening Checklist

```bash
# Scan your own infrastructure (know what's exposed)
nmap -sV -p- your-server.com

# Check what's listening
ss -tlnp    # TCP listeners
ss -ulnp    # UDP listeners

# Only expose required ports in Security Groups
# ✅ Good: 443 from 0.0.0.0/0 (HTTPS only)
# ❌ Bad:  0-65535 from 0.0.0.0/0 (everything open)

# Block common dangerous ports
# 22   - SSH (restrict to VPN/bastion IP only)
# 3306 - MySQL (never expose to internet)
# 5432 - PostgreSQL (never expose to internet)
# 6379 - Redis (never expose to internet, no default auth!)
# 9200 - Elasticsearch (never expose to internet)
# 27017 - MongoDB (never expose to internet)
```

### Terraform — Restrictive Security Group

```hcl
resource "aws_security_group" "web" {
  name_prefix = "web-"
  vpc_id      = aws_vpc.main.id

  # Only HTTPS from internet
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS from internet"
  }

  # SSH only from VPN
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]  # Internal VPN range only
    description = "SSH from VPN only"
  }

  # No other inbound ports!

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    description = "HTTPS outbound"
  }

  tags = { Name = "web-restricted" }
}
```

---

## 6.5 DNS Attacks

```
┌──────────────────────────────────────────────────────────────┐
│           DNS ATTACK TYPES                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. DNS SPOOFING / CACHE POISONING                           │
│  ┌────────┐     ┌─────────┐     ┌──────────┐                │
│  │ Client │────▶│ DNS     │     │ Attacker │                │
│  │        │     │ Resolver│◀────│ Injects  │                │
│  │        │◀────│ (cached │     │ fake     │                │
│  └────────┘     │  poison)│     │ records  │                │
│                 └─────────┘     └──────────┘                │
│  bank.com → attacker's IP (phishing site)                   │
│                                                              │
│  Prevention: DNSSEC, DoH (DNS over HTTPS)                    │
│                                                              │
│  2. DNS HIJACKING                                            │
│  Attacker gains control of domain registrar account          │
│  Changes nameservers to attacker-controlled ones             │
│                                                              │
│  Prevention: Registrar lock, MFA on domain accounts          │
│                                                              │
│  3. DNS TUNNELING                                            │
│  Encode data in DNS queries to exfiltrate data               │
│  ┌────────┐  DNS query: data.encoded.evil.com                │
│  │Malware │──────────────────────────────────▶ C2 Server     │
│  └────────┘  Bypasses firewalls (DNS = port 53 = allowed)    │
│                                                              │
│  Prevention: Monitor DNS query patterns, block excessive     │
│              DNS queries, use DNS firewall (Route 53 RFW)    │
│                                                              │
│  4. SUBDOMAIN TAKEOVER                                       │
│  app.example.com CNAME → old-service.herokuapp.com           │
│  Service deleted but DNS record remains                      │
│  Attacker claims old-service.herokuapp.com                   │
│  Now controls app.example.com!                               │
│                                                              │
│  Prevention: Audit stale DNS records, remove unused CNAMEs   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Check for subdomain takeover risks
# Look for CNAME records pointing to decommissioned services
dig app.example.com CNAME

# Check if the target is unclaimed
curl -I https://old-service.herokuapp.com
# If 404 / not found → takeover risk!

# Audit all DNS records
aws route53 list-resource-record-sets \
  --hosted-zone-id Z1234567890 \
  --query "ResourceRecordSets[?Type=='CNAME']"
```

---

## 6.6 SSRF (Server-Side Request Forgery)

```
┌──────────────────────────────────────────────────────────────┐
│          SSRF ATTACK (Cloud Metadata Theft)                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal:                                                     │
│  User → App: "Fetch https://example.com/image.png"           │
│  App fetches image and returns it                            │
│                                                              │
│  SSRF Attack:                                                │
│  Attacker → App: "Fetch http://169.254.169.254/..."          │
│                                                              │
│  ┌──────────┐                    ┌──────────────────┐        │
│  │ Attacker │──── POST ─────────▶│ Vulnerable App   │        │
│  │          │  url=http://       │                  │        │
│  │          │  169.254.169.254/  │ Fetches internal │        │
│  │          │  latest/meta-data/ │ metadata!        │        │
│  │          │◀──────────────────│                  │        │
│  └──────────┘  Returns IAM      └──────────────────┘        │
│                credentials!                                  │
│                                                              │
│  Impact: Steal AWS IAM role credentials, access S3,          │
│          databases, internal services                        │
│                                                              │
│  Real-World: Capital One breach (2019) — SSRF → metadata    │
│              → IAM creds → 100M customer records            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### SSRF Prevention

```hcl
# 1. Use IMDSv2 (requires token for metadata access)
resource "aws_launch_template" "secure" {
  name_prefix = "secure-"

  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"  # IMDSv2 — blocks SSRF!
    http_put_response_hop_limit = 1           # Prevent container escape
  }

  # Also useful: disable metadata entirely if not needed
  # http_endpoint = "disabled"
}
```

```bash
# Verify IMDSv2 is enforced on existing instances
aws ec2 describe-instances \
  --query "Reservations[].Instances[].{ID:InstanceId,IMDS:MetadataOptions.HttpTokens}" \
  --output table

# Enforce IMDSv2 organization-wide with SCP
# (Service Control Policy)
```

---

## 6.7 Container and Kubernetes Security

```
┌──────────────────────────────────────────────────────────────┐
│        CONTAINER NETWORK SECURITY RISKS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Risk 1: CONTAINER ESCAPE                                    │
│  ┌─────────────────────────────────────────────────────┐     │
│  │ Container (compromised)                             │     │
│  │    │                                                │     │
│  │    │ Escape via kernel exploit / mount /             │     │
│  │    ▼                                                │     │
│  │ Host System (full access) ← GAME OVER               │     │
│  └─────────────────────────────────────────────────────┘     │
│  Prevention: Non-root containers, read-only rootfs,          │
│              seccomp profiles, AppArmor                       │
│                                                              │
│  Risk 2: EXPOSED KUBERNETES API                              │
│  kubectl accessible from internet → cluster takeover         │
│  Prevention: Private API endpoint, RBAC, network policy      │
│                                                              │
│  Risk 3: PRIVILEGED CONTAINERS                               │
│  Container with --privileged flag = host-level access         │
│  Prevention: PodSecurityStandards, OPA Gatekeeper            │
│                                                              │
│  Risk 4: IMAGE VULNERABILITIES                               │
│  Base image contains CVEs → compromised at runtime           │
│  Prevention: Image scanning (Trivy, Snyk), minimal images   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# Kubernetes PodSecurityContext — hardened pod
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault

  containers:
    - name: app
      image: myapp:latest
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]

      # Limit resources to prevent resource abuse
      resources:
        limits:
          cpu: "500m"
          memory: "256Mi"
        requests:
          cpu: "100m"
          memory: "128Mi"
```

```bash
# Scan container images for vulnerabilities
trivy image myapp:latest

# Scan Kubernetes cluster for misconfigurations
trivy k8s --report summary cluster

# Check for privileged containers
kubectl get pods -A -o json | \
  jq '.items[] | select(.spec.containers[].securityContext.privileged==true) | .metadata.name'
```

---

## 6.8 Hardening Checklist for DevOps

```
┌──────────────────────────────────────────────────────────────┐
│         NETWORK HARDENING CHECKLIST                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ENCRYPTION                                                  │
│  □ TLS 1.2+ everywhere (disable 1.0/1.1)                    │
│  □ HSTS enabled with preload                                 │
│  □ mTLS for service-to-service                               │
│  □ Encrypt data at rest (EBS, S3, RDS)                       │
│                                                              │
│  ACCESS CONTROL                                              │
│  □ Security groups: least privilege (no 0.0.0.0/0 on SSH)    │
│  □ NACLs on sensitive subnets                                │
│  □ SSH key-based auth only (disable password)                │
│  □ MFA on all admin accounts                                 │
│  □ VPN or bastion for management access                      │
│                                                              │
│  MONITORING & DETECTION                                      │
│  □ VPC Flow Logs enabled                                     │
│  □ CloudTrail logging all API calls                          │
│  □ DNS query logging                                         │
│  □ GuardDuty / IDS enabled                                   │
│  □ Alerts on unusual traffic patterns                        │
│                                                              │
│  INFRASTRUCTURE                                              │
│  □ No public IPs on databases/caches                         │
│  □ IMDSv2 enforced (prevent SSRF)                            │
│  □ Network segmentation (public/private/isolated)            │
│  □ Egress filtering (restrict outbound traffic)              │
│  □ Regular port scan audits                                  │
│                                                              │
│  CONTAINERS                                                  │
│  □ Non-root containers                                       │
│  □ Read-only root filesystem                                 │
│  □ Drop all Linux capabilities                               │
│  □ Image scanning in CI/CD pipeline                          │
│  □ Kubernetes NetworkPolicies applied                        │
│                                                              │
│  DNS                                                         │
│  □ Remove stale CNAME records (subdomain takeover)           │
│  □ DNSSEC enabled where possible                             │
│  □ DNS firewall rules (Route 53 Resolver Firewall)           │
│  □ Monitor for DNS tunneling                                 │
│                                                              │
│  SECRETS                                                     │
│  □ No hardcoded credentials                                  │
│  □ Use OIDC/workload identity (no static keys)               │
│  □ Rotate secrets automatically (≤90 days)                   │
│  □ Secrets Manager / Vault for storage                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.9 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Unexplained high traffic | DDoS or traffic amplification | Enable WAF, Shield, rate limiting |
| Services responding to unknown IPs | Overly permissive security groups | Audit and restrict SG rules |
| Metadata credentials leaked | SSRF via IMDSv1 | Enforce IMDSv2 across all instances |
| Stale DNS pointing to wrong service | Subdomain takeover risk | Audit and remove unused CNAME records |
| Container running as root | Missing security context | Add `runAsNonRoot: true` to pod spec |
| Unencrypted internal traffic | No mTLS between services | Deploy service mesh with strict mTLS |

---

## Summary Table

| Vulnerability | Impact | Prevention |
|--------------|--------|------------|
| MITM | Data interception/modification | TLS, HSTS, mTLS |
| DDoS | Service unavailability | CDN, WAF, Shield, rate limiting |
| Port Scanning | Reconnaissance for attacks | Minimize open ports, SG hardening |
| DNS Spoofing | Traffic redirection | DNSSEC, DoH |
| Subdomain Takeover | Domain hijacking | Audit stale CNAMEs |
| SSRF | Cloud credential theft | IMDSv2, input validation |
| Container Escape | Host compromise | Non-root, drop capabilities |
| DNS Tunneling | Data exfiltration | DNS monitoring, firewall |

---

## Quick Revision Questions

1. **How does a Man-in-the-Middle attack work, and what prevents it?**
2. **What's the difference between Layer 3/4 and Layer 7 DDoS attacks?**
3. **Why is SSRF especially dangerous in cloud environments? How does IMDSv2 help?**
4. **What is a subdomain takeover and how do you prevent it?**
5. **List five hardening measures you'd apply to a production Kubernetes cluster's network.**
6. **Why should you never expose database ports (3306, 5432, 6379) to the internet?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Zero Trust Networking](05-zero-trust-networking.md) | [README](../README.md) | [Diagnostic Tools →](../07-Network-Troubleshooting/01-diagnostic-tools.md) |

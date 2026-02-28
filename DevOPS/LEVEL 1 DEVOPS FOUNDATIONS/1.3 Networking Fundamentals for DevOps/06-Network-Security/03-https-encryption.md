# Chapter 3: HTTPS Encryption

## Overview

**HTTPS** (HTTP Secure) encrypts all HTTP communication using TLS (Transport Layer Security). For DevOps engineers, understanding HTTPS isn't just about buying a certificate — it's about choosing cipher suites, configuring TLS versions, deciding where to terminate SSL, and ensuring end-to-end security across your infrastructure.

---

## 3.1 How HTTPS Works

```
┌──────────────────────────────────────────────────────────────┐
│               HTTPS = HTTP over TLS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Client                          Server                     │
│   ┌──────┐                        ┌──────┐                   │
│   │      │──── TCP Handshake ────▶│      │  Port 443         │
│   │      │◀─── SYN-ACK ──────────│      │                   │
│   │      │──── ACK ──────────────▶│      │                   │
│   │      │                        │      │                   │
│   │      │──── TLS ClientHello ──▶│      │  TLS 1.3          │
│   │      │◀─── ServerHello ───────│      │  + Certificate    │
│   │      │◀─── Certificate ───────│      │  + Key Share      │
│   │      │──── Key Exchange ─────▶│      │                   │
│   │      │──── Finished ─────────▶│      │                   │
│   │      │◀─── Finished ─────────│      │                   │
│   │      │                        │      │                   │
│   │      │════ ENCRYPTED ════════▶│      │  HTTP Request     │
│   │      │◀═════ DATA ═══════════│      │  HTTP Response    │
│   └──────┘                        └──────┘                   │
│                                                              │
│   TLS 1.2: 2 round trips (slower)                            │
│   TLS 1.3: 1 round trip  (faster, more secure)               │
│   TLS 1.3 0-RTT: 0 round trips (resumed sessions)           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 TLS Version Comparison

| Feature | TLS 1.0 | TLS 1.1 | TLS 1.2 | TLS 1.3 |
|---------|---------|---------|---------|---------|
| Status | Deprecated | Deprecated | Active | Recommended |
| Handshake | 2 RTT | 2 RTT | 2 RTT | 1 RTT |
| Cipher Suites | Many (weak) | Many (weak) | Many | Only 5 (strong) |
| 0-RTT Resume | No | No | No | Yes |
| PFS Required | No | No | Optional | Mandatory |
| Key Exchange | RSA/DHE | RSA/DHE | RSA/ECDHE | ECDHE only |
| Compliance | PCI non-compliant | PCI non-compliant | PCI compliant | PCI compliant |

> **DevOps Rule:** Disable TLS 1.0 and 1.1 everywhere. Prefer TLS 1.3.

---

## 3.3 Cipher Suites

```
┌──────────────────────────────────────────────────────────────┐
│         CIPHER SUITE ANATOMY                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TLS 1.2 Example:                                            │
│  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384                       │
│  ───── ──── ───      ─── ─── ───  ──────                     │
│    │    │    │        │   │   │     │                         │
│    │    │    │        │   │   │     └─ Hash: SHA-384          │
│    │    │    │        │   │   └─ Mode: Galois/Counter         │
│    │    │    │        │   └─ Key size: 256-bit                │
│    │    │    │        └─ Encryption: AES                      │
│    │    │    └─ Authentication: RSA certificate               │
│    │    └─ Key Exchange: ECDHE (Elliptic Curve Diffie-Hellman)│
│    └─ Protocol: TLS                                          │
│                                                              │
│  TLS 1.3 Example (simplified):                               │
│  TLS_AES_256_GCM_SHA384                                      │
│  (Key exchange is always ECDHE — no need to specify)         │
│                                                              │
│  RECOMMENDED TLS 1.3 Cipher Suites:                          │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ TLS_AES_256_GCM_SHA384       (strongest)              │  │
│  │ TLS_CHACHA20_POLY1305_SHA256 (fastest on mobile)      │  │
│  │ TLS_AES_128_GCM_SHA256       (good balance)           │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Key Concepts

| Term | Meaning |
|------|---------|
| **PFS** (Perfect Forward Secrecy) | Even if the server's private key is compromised, past sessions can't be decrypted |
| **ECDHE** | Ephemeral key exchange — new keys per session (enables PFS) |
| **GCM** | Authenticated encryption — encrypts + verifies integrity in one step |
| **RSA** (for key exchange) | Deprecated — no PFS, avoid |

---

## 3.4 SSL Termination Patterns

```
┌──────────────────────────────────────────────────────────────┐
│          SSL TERMINATION ARCHITECTURES                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Pattern 1: TERMINATE AT LOAD BALANCER (Most Common)         │
│  ┌────────┐  HTTPS   ┌─────────┐  HTTP   ┌────────────┐     │
│  │ Client │─────────▶│ ALB/    │────────▶│ Backend    │     │
│  │        │◀─────────│ Nginx   │◀────────│ (port 80)  │     │
│  └────────┘  :443    └─────────┘  :80    └────────────┘     │
│                       Cert here    Unencrypted               │
│  ✅ Simple, offloads CPU from backends                       │
│  ⚠️ Internal traffic unencrypted                             │
│                                                              │
│  Pattern 2: SSL PASSTHROUGH                                  │
│  ┌────────┐  HTTPS   ┌─────────┐ HTTPS   ┌────────────┐     │
│  │ Client │─────────▶│ LB (L4) │────────▶│ Backend    │     │
│  │        │◀─────────│ TCP only │◀────────│ (port 443) │     │
│  └────────┘  :443    └─────────┘  :443   └────────────┘     │
│               LB can't inspect traffic    Cert here          │
│  ✅ End-to-end encryption                                    │
│  ⚠️ No L7 routing, each backend needs cert                   │
│                                                              │
│  Pattern 3: RE-ENCRYPTION (Most Secure)                      │
│  ┌────────┐  HTTPS   ┌─────────┐ HTTPS   ┌────────────┐     │
│  │ Client │─────────▶│ ALB/    │────────▶│ Backend    │     │
│  │        │◀─────────│ Nginx   │◀────────│ (port 443) │     │
│  └────────┘  :443    └─────────┘  :443   └────────────┘     │
│            Public cert  Decrypts/   Internal cert            │
│                        re-encrypts                           │
│  ✅ End-to-end encryption + L7 routing                       │
│  ⚠️ More complex, two sets of certs                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.5 Nginx HTTPS Configuration

```nginx
# /etc/nginx/conf.d/example.conf

# Redirect HTTP → HTTPS
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

# HTTPS Server
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # Certificate and key
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # TLS versions (disable 1.0 and 1.1)
    ssl_protocols TLSv1.2 TLSv1.3;

    # Cipher suites (strong only)
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305';
    ssl_prefer_server_ciphers on;

    # OCSP Stapling (faster certificate validation)
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

    # SSL session caching
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;  # Disable for PFS

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    # DH parameters (TLS 1.2 only, not needed for 1.3)
    ssl_dhparam /etc/nginx/dhparam.pem;

    location / {
        proxy_pass http://backend:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Generate strong DH parameters (one-time, takes a few minutes)
openssl dhparam -out /etc/nginx/dhparam.pem 2048

# Test configuration
nginx -t

# Reload Nginx
systemctl reload nginx
```

---

## 3.6 AWS ALB with HTTPS (Terraform)

```hcl
# ACM Certificate
resource "aws_acm_certificate" "main" {
  domain_name               = "example.com"
  subject_alternative_names = ["*.example.com"]
  validation_method         = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}

# DNS Validation Record
resource "aws_route53_record" "cert_validation" {
  for_each = {
    for dvo in aws_acm_certificate.main.domain_validation_options : dvo.domain_name => {
      name   = dvo.resource_record_name
      type   = dvo.resource_record_type
      record = dvo.resource_record_value
    }
  }

  zone_id = aws_route53_zone.main.zone_id
  name    = each.value.name
  type    = each.value.type
  ttl     = 60
  records = [each.value.record]
}

# Wait for validation
resource "aws_acm_certificate_validation" "main" {
  certificate_arn         = aws_acm_certificate.main.arn
  validation_record_fqdns = [for r in aws_route53_record.cert_validation : r.fqdn]
}

# ALB HTTPS Listener
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"  # TLS 1.3 + 1.2
  certificate_arn   = aws_acm_certificate_validation.main.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

# HTTP → HTTPS Redirect
resource "aws_lb_listener" "http_redirect" {
  load_balancer_arn = aws_lb.main.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type = "redirect"
    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}
```

### AWS SSL Policies

| Policy | TLS Versions | Use Case |
|--------|-------------|----------|
| `ELBSecurityPolicy-TLS13-1-2-2021-06` | 1.3 + 1.2 | **Recommended** |
| `ELBSecurityPolicy-TLS-1-2-2017-01` | 1.2 only | Legacy compatibility |
| `ELBSecurityPolicy-FS-1-2-2019-08` | 1.2 + PFS | Forward secrecy required |
| `ELBSecurityPolicy-2016-08` | 1.0+ | **Avoid** (insecure) |

---

## 3.7 Testing HTTPS Configuration

```bash
# Test TLS handshake
openssl s_client -connect example.com:443

# Check TLS version negotiated
openssl s_client -connect example.com:443 -tls1_3

# Show certificate chain
openssl s_client -connect example.com:443 -showcerts

# Test specific cipher
openssl s_client -connect example.com:443 -cipher ECDHE-RSA-AES256-GCM-SHA384

# Verify TLS 1.0/1.1 is disabled (should fail)
openssl s_client -connect example.com:443 -tls1
openssl s_client -connect example.com:443 -tls1_1

# Check HSTS header
curl -sI https://example.com | grep -i strict

# Full SSL test with curl
curl -vI https://example.com 2>&1 | grep -E 'SSL|TLS|subject|issuer'

# Use nmap to scan SSL/TLS
nmap --script ssl-enum-ciphers -p 443 example.com

# Online tools
# - https://www.ssllabs.com/ssltest/  (SSL Labs — A+ rating target)
# - https://observatory.mozilla.org/   (Mozilla Observatory)
```

---

## 3.8 HSTS (HTTP Strict Transport Security)

```
┌──────────────────────────────────────────────────────────────┐
│                  HSTS FLOW                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  First Visit:                                                │
│  Browser ──HTTP──▶ Server                                    │
│  Browser ◀──301 Redirect to HTTPS── Server                   │
│  Browser ──HTTPS──▶ Server                                   │
│  Browser ◀──200 + HSTS Header── Server                       │
│                                                              │
│  Header: Strict-Transport-Security: max-age=63072000;        │
│          includeSubDomains; preload                          │
│                                                              │
│  Subsequent Visits:                                          │
│  Browser ──HTTPS──▶ Server  (browser auto-upgrades to HTTPS) │
│                                                              │
│  max-age=63072000  → Browser remembers for 2 years           │
│  includeSubDomains → Applies to all subdomains               │
│  preload           → Submit to browser preload list          │
│                    → hstspreload.org                          │
│                                                              │
│  ⚠️  Once preloaded, very hard to undo!                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.9 mTLS (Mutual TLS)

```
┌──────────────────────────────────────────────────────────────┐
│          STANDARD TLS vs MUTUAL TLS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Standard TLS (one-way):                                     │
│  ┌────────┐              ┌────────┐                          │
│  │ Client │──────────────│ Server │                          │
│  │        │  Verifies    │        │                          │
│  │        │  server cert │  Cert  │                          │
│  └────────┘              └────────┘                          │
│  Client trusts server. Server trusts anyone.                 │
│                                                              │
│  Mutual TLS (two-way):                                       │
│  ┌────────┐              ┌────────┐                          │
│  │ Client │──────────────│ Server │                          │
│  │  Cert  │  Both verify │  Cert  │                          │
│  │        │  each other  │        │                          │
│  └────────┘              └────────┘                          │
│  Both parties authenticated. Service-to-service.             │
│                                                              │
│  Used in:                                                    │
│  - Kubernetes service mesh (Istio, Linkerd)                  │
│  - API Gateway client authentication                         │
│  - Microservices communication                               │
│  - Zero trust architecture                                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Nginx mTLS Configuration

```nginx
server {
    listen 443 ssl;
    server_name api.internal.example.com;

    # Server certificate
    ssl_certificate     /etc/nginx/certs/server.crt;
    ssl_certificate_key /etc/nginx/certs/server.key;

    # Client certificate verification (mTLS)
    ssl_client_certificate /etc/nginx/certs/ca.crt;  # CA that signed client certs
    ssl_verify_client on;                             # Require client cert
    ssl_verify_depth 2;

    location / {
        # Pass client cert info to backend
        proxy_set_header X-Client-CN $ssl_client_s_dn_cn;
        proxy_set_header X-Client-Verify $ssl_client_verify;
        proxy_pass http://backend:8080;
    }
}
```

---

## 3.10 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| ERR_SSL_VERSION_OR_CIPHER_MISMATCH | No common cipher between client/server | Update cipher list, check TLS version |
| Mixed content warnings | HTTP resources on HTTPS page | Use relative URLs or upgrade all to HTTPS |
| SSL Labs grade below A | Weak ciphers or old TLS enabled | Disable TLS 1.0/1.1, remove weak ciphers |
| HSTS causing issues in dev | Browser cached HSTS for domain | Use different domain for dev, clear HSTS in browser |
| Certificate chain incomplete | Missing intermediate cert | Use fullchain.pem, not just cert.pem |
| TLS handshake timeout | Firewall blocking port 443 | Check security group / firewall rules |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| HTTPS | HTTP encrypted with TLS on port 443 |
| TLS 1.3 | 1-RTT handshake, mandatory PFS, strongest |
| TLS 1.2 | Still acceptable, configure strong ciphers |
| TLS 1.0/1.1 | Deprecated — disable everywhere |
| PFS | Ephemeral keys protect past sessions |
| SSL Termination | Decrypt at LB (most common), passthrough, or re-encrypt |
| HSTS | Forces browser to always use HTTPS |
| mTLS | Both client and server present certificates |
| OCSP Stapling | Server provides certificate status (faster) |

---

## Quick Revision Questions

1. **What's the difference between TLS 1.2 and TLS 1.3 in terms of handshake performance?**
2. **Explain the three SSL termination patterns and when to use each.**
3. **What does Perfect Forward Secrecy (PFS) protect against?**
4. **How does HSTS prevent SSL stripping attacks?**
5. **When would you use mTLS instead of standard TLS?**
6. **What AWS SSL policy should you use for an ALB in production, and why?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Certificate Authorities](02-certificate-authorities.md) | [README](../README.md) | [Network Segmentation →](04-network-segmentation.md) |

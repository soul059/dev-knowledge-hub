# Cloud Infrastructure Discovery

## Unit 4 - Topic 5 | Network Reconnaissance

---

## Overview

**Cloud infrastructure discovery** identifies cloud services, storage buckets, serverless functions, and virtual machines used by the target. As organizations migrate to cloud, a significant portion of the attack surface exists in AWS, Azure, GCP, and other cloud platforms.

---

## 1. Cloud Fingerprinting

| Indicator | Cloud Provider |
|-----------|:---:|
| CNAME `*.amazonaws.com` | AWS |
| CNAME `*.azurewebsites.net` | Azure |
| CNAME `*.cloudapp.azure.com` | Azure |
| CNAME `*.appspot.com` | GCP |
| CNAME `*.firebaseapp.com` | GCP/Firebase |
| Headers `x-amz-*` | AWS |
| Headers `x-ms-*` | Azure |
| Headers `x-goog-*` | GCP |
| IP range 52.x.x.x, 54.x.x.x | AWS |
| IP range 13.x.x.x, 20.x.x.x | Azure |
| IP range 34.x.x.x, 35.x.x.x | GCP |

---

## 2. Cloud Storage Discovery

```bash
# === AWS S3 BUCKET ENUMERATION ===
# Common naming patterns:
# target-backup, target-assets, target-data, target-prod

# Check if bucket exists and is public
aws s3 ls s3://target-backup --no-sign-request
aws s3 ls s3://target-data --no-sign-request
aws s3 ls s3://target-assets --no-sign-request

# Automated S3 enumeration
# cloud_enum
cloud_enum -k target -k "target corp" -k "targetcorp"

# S3Scanner
s3scanner scan --bucket-file buckets.txt

# GrayhatWarfare (web)
# buckets.grayhatwarfare.com — searchable S3/Azure/GCP buckets

# === AZURE BLOB STORAGE ===
# Format: https://<account>.blob.core.windows.net/<container>
curl https://targetstorage.blob.core.windows.net/public?restype=container&comp=list

# === GCP STORAGE ===
# Format: https://storage.googleapis.com/<bucket>
curl https://storage.googleapis.com/target-bucket/
```

---

## 3. Cloud Service Enumeration

```bash
# === DNS-BASED DISCOVERY ===
# Check for cloud service subdomains
dig api.target.com CNAME +short
dig app.target.com CNAME +short
dig cdn.target.com CNAME +short

# Common cloud patterns:
# *.elasticbeanstalk.com    → AWS Elastic Beanstalk
# *.s3.amazonaws.com        → AWS S3
# *.execute-api.*.amazonaws.com → AWS API Gateway
# *.azurewebsites.net       → Azure App Service
# *.database.windows.net    → Azure SQL
# *.run.app                 → GCP Cloud Run
# *.cloudfunctions.net      → GCP Cloud Functions

# === CLOUD METADATA ===
# If you have SSRF vulnerability:
# AWS: http://169.254.169.254/latest/meta-data/
# Azure: http://169.254.169.254/metadata/instance?api-version=2021-02-01
# GCP: http://169.254.169.254/computeMetadata/v1/
# ⚠️ Metadata endpoint reveals credentials and config!

# === IP RANGE LOOKUP ===
# AWS IP ranges: https://ip-ranges.amazonaws.com/ip-ranges.json
# Azure IP ranges: Published by Microsoft
# GCP IP ranges: Published by Google
```

---

## 4. Misconfigured Cloud Services

| Misconfiguration | Risk | Detection |
|-----------------|------|-----------|
| **Public S3 bucket** | Data exposure | `aws s3 ls --no-sign-request` |
| **Open Azure blob** | Data exposure | Direct URL access |
| **Exposed metadata** | Credential theft | SSRF to 169.254.169.254 |
| **Default security groups** | Unauthorized access | Port scan cloud IPs |
| **Public snapshots** | Data exposure | AWS console/API search |
| **Exposed .git** | Source code leak | `curl target.com/.git/HEAD` |

---

## 5. Cloud Recon Workflow

```
CLOUD INFRASTRUCTURE DISCOVERY:

STEP 1: Identify cloud provider
├── DNS CNAME records
├── HTTP headers
├── IP range lookup
└── Technology detection (Wappalyzer)

STEP 2: Enumerate cloud services
├── S3/Blob/GCS bucket names
├── Serverless functions
├── Databases
└── API gateways

STEP 3: Check for misconfigurations
├── Public storage
├── Open security groups
├── Exposed metadata
└── Default credentials

STEP 4: Document findings
├── All cloud assets found
├── Misconfigurations identified
├── Data exposure risk
└── Recommendations
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Cloud detection** | DNS CNAME, HTTP headers, IP ranges |
| **S3 buckets** | Common misconfiguration — public access |
| **Metadata** | 169.254.169.254 — reveals credentials |
| **Tools** | cloud_enum, S3Scanner, GrayhatWarfare |
| **Risk** | Data exposure, credential theft |
| **Multi-cloud** | Check AWS, Azure, and GCP |

---

## Quick Revision Questions

1. **How do you identify which cloud provider a target uses?**
2. **What is the cloud metadata endpoint and why is it important?**
3. **How do you enumerate S3 buckets?**
4. **What are common cloud misconfigurations?**
5. **Name 2 tools for cloud storage discovery.**

---

[← Previous: CDN Identification](04-cdn-identification.md)

---

*Unit 4 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Web Reconnaissance →](../05-Web-Reconnaissance/01-website-analysis.md)*

# Chapter 4.1: S3 — Simple Storage Service

## Overview

Amazon S3 is an **object storage** service offering unlimited scalability, 99.999999999% (11 nines) durability, and a rich feature set for DevOps workflows including artifact storage, static website hosting, log archiving, and backup/restore.

---

## S3 Architecture

```
┌─────────────────── S3 Service (Regional) ─────────────────────┐
│                                                                 │
│  ┌──── Bucket: my-app-artifacts ──────────────────────────┐   │
│  │  Globally unique name                                    │   │
│  │  Region: us-east-1                                       │   │
│  │                                                          │   │
│  │  ┌── Object ─────────────────────────────────────────┐  │   │
│  │  │  Key:    builds/v1.2.3/app.zip                    │  │   │
│  │  │  Value:  <binary data up to 5 TB>                 │  │   │
│  │  │  Metadata: Content-Type, custom headers           │  │   │
│  │  │  Version ID: (if versioning enabled)              │  │   │
│  │  │  Tags: {Environment: prod, Team: backend}         │  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  │                                                          │   │
│  │  ┌── Object ─────────────────────────────────────────┐  │   │
│  │  │  Key:    logs/2024/01/15/access.log.gz            │  │   │
│  │  │  Value:  <compressed log data>                    │  │   │
│  │  └───────────────────────────────────────────────────┘  │   │
│  │                                                          │   │
│  │  Features:                                               │   │
│  │  ├── Versioning (enabled/suspended)                     │   │
│  │  ├── Encryption (SSE-S3, SSE-KMS, SSE-C)               │   │
│  │  ├── Lifecycle Rules                                     │   │
│  │  ├── Replication (CRR / SRR)                            │   │
│  │  ├── Access Logging                                      │   │
│  │  └── Event Notifications (Lambda, SQS, SNS)            │   │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

Key Concepts:
  Bucket  = Container (globally unique name, region-specific)
  Object  = File (key + value + metadata)
  Key     = Full path (prefix/filename, no real "folders")
  Value   = Content (0 bytes to 5 TB)
```

---

## S3 Operations (CLI)

```bash
# Create bucket
aws s3 mb s3://my-devops-artifacts-2024

# Upload a file
aws s3 cp app.zip s3://my-devops-artifacts-2024/builds/v1.2.3/

# Upload a directory recursively
aws s3 sync ./dist s3://my-devops-artifacts-2024/website/ --delete

# Download a file
aws s3 cp s3://my-devops-artifacts-2024/builds/v1.2.3/app.zip ./

# List objects with prefix
aws s3 ls s3://my-devops-artifacts-2024/builds/ --recursive

# Copy between buckets
aws s3 cp s3://source-bucket/file.zip s3://dest-bucket/file.zip

# Remove objects
aws s3 rm s3://my-devops-artifacts-2024/builds/old/ --recursive

# Presigned URL (temporary access, 1 hour)
aws s3 presign s3://my-devops-artifacts-2024/builds/v1.2.3/app.zip \
  --expires-in 3600
```

### S3 API Commands (Lower Level)

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-devops-artifacts-2024 \
  --versioning-configuration Status=Enabled

# Enable default encryption (SSE-S3)
aws s3api put-bucket-encryption \
  --bucket my-devops-artifacts-2024 \
  --server-side-encryption-configuration '{
    "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "AES256"}}]
  }'

# Block all public access (best practice)
aws s3api put-public-access-block \
  --bucket my-devops-artifacts-2024 \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# Enable access logging
aws s3api put-bucket-logging \
  --bucket my-devops-artifacts-2024 \
  --bucket-logging-status '{
    "LoggingEnabled": {
      "TargetBucket": "my-access-logs-bucket",
      "TargetPrefix": "s3-logs/"
    }
  }'
```

---

## S3 Versioning

```
┌──── Bucket with Versioning Enabled ──────────────────┐
│                                                       │
│  Key: config.json                                     │
│                                                       │
│  ┌──────────────────────────────────────────────┐    │
│  │ Version 3 (current)  ID: abc789  Latest      │    │
│  │ Uploaded: 2024-01-15  Size: 2.1 KB           │    │
│  └──────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────┐    │
│  │ Version 2             ID: abc456             │    │
│  │ Uploaded: 2024-01-10  Size: 1.8 KB           │    │
│  └──────────────────────────────────────────────┘    │
│  ┌──────────────────────────────────────────────┐    │
│  │ Version 1             ID: abc123             │    │
│  │ Uploaded: 2024-01-05  Size: 1.5 KB           │    │
│  └──────────────────────────────────────────────┘    │
│                                                       │
│  DELETE config.json → adds Delete Marker (recoverable)│
│  DELETE specific version → permanent deletion         │
└───────────────────────────────────────────────────────┘
```

---

## Bucket Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCodeBuildAccess",
      "Effect": "Allow",
      "Principal": {
        "Service": "codebuild.amazonaws.com"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-devops-artifacts-2024/builds/*"
    },
    {
      "Sid": "DenyUnencryptedUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-devops-artifacts-2024/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    },
    {
      "Sid": "EnforceTLSOnly",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-devops-artifacts-2024",
        "arn:aws:s3:::my-devops-artifacts-2024/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

---

## S3 Event Notifications

```
┌─────────────────── S3 Event Flow ──────────────────────┐
│                                                         │
│   Upload to S3                                          │
│       │                                                 │
│       ▼                                                 │
│   s3:ObjectCreated:Put                                  │
│       │                                                 │
│       ├──► Lambda → Process image / scan for virus      │
│       │                                                 │
│       ├──► SQS → Queue for batch processing             │
│       │                                                 │
│       └──► SNS → Notify team of new artifact            │
│                                                         │
│   Event types:                                          │
│   • s3:ObjectCreated:*      (Put, Post, Copy, MPU)     │
│   • s3:ObjectRemoved:*      (Delete, DeleteMarker)     │
│   • s3:ObjectRestore:*      (from Glacier)             │
│   • s3:Replication:*        (replication events)       │
└─────────────────────────────────────────────────────────┘
```

---

## Multipart Upload

```
File: 10 GB database backup
┌───────────────────────────────────────────────────┐
│  Part 1: 100 MB  ──────────► ┐                   │
│  Part 2: 100 MB  ──────────► │                   │
│  Part 3: 100 MB  ──────────► ├── Parallel upload │
│  ...                          │   + auto-retry    │
│  Part 100: 100 MB ─────────► ┘                   │
│                                                   │
│  Complete Multipart Upload → S3 assembles parts   │
│                                                   │
│  Rules:                                           │
│  • Required for files > 5 GB                      │
│  • Recommended for files > 100 MB                 │
│  • Each part: 5 MB to 5 GB                        │
│  • Max 10,000 parts                               │
│  • aws s3 cp uses multipart automatically         │
└───────────────────────────────────────────────────┘
```

---

## S3 Encryption Options

| Method | Key Management | Use Case |
|--------|---------------|----------|
| **SSE-S3** | AWS manages keys (AES-256) | Default, simplest |
| **SSE-KMS** | AWS KMS manages keys | Audit trail, key rotation, compliance |
| **SSE-C** | Customer provides keys | Full key control, keys sent with each request |
| **Client-Side** | Encrypt before upload | Maximum control, zero trust |

---

## Real-World DevOps Scenario: CI/CD Artifact Store

```bash
#!/bin/bash
# Build pipeline artifact management

BUCKET="my-company-artifacts"
APP_NAME="web-app"
VERSION=$(git describe --tags)
BUILD_DIR="./dist"

# Upload build artifacts
aws s3 sync $BUILD_DIR s3://$BUCKET/$APP_NAME/$VERSION/ \
  --sse aws:kms \
  --storage-class STANDARD_IA

# Tag the artifact
aws s3api put-object-tagging \
  --bucket $BUCKET \
  --key "$APP_NAME/$VERSION/app.zip" \
  --tagging 'TagSet=[{Key=Environment,Value=staging},{Key=BuildId,Value='"$BUILD_NUMBER"'}]'

# Generate deployment URL (valid 1 hour)
DEPLOY_URL=$(aws s3 presign s3://$BUCKET/$APP_NAME/$VERSION/app.zip \
  --expires-in 3600)
echo "Deploy URL: $DEPLOY_URL"
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| 403 Forbidden | Bucket policy or IAM denies access | Check bucket policy, IAM policy, and public access block |
| 404 Not Found | Wrong key/prefix or wrong region | Verify exact key path and bucket region |
| Slow uploads | Large file, single stream | Use multipart upload or `aws s3 cp` (auto-multipart) |
| Accidental deletion | No versioning | Enable versioning + MFA Delete for critical buckets |
| High costs | Infrequent data in STANDARD | Use lifecycle rules to transition to cheaper classes |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **S3 Object** | Key (path) + Value (data, up to 5 TB) + Metadata |
| **Bucket** | Globally unique name, region-specific, flat namespace |
| **Versioning** | Keep all versions; delete adds marker (recoverable) |
| **Encryption** | SSE-S3 (default), SSE-KMS (audit), SSE-C (your keys) |
| **Bucket Policy** | JSON resource-based policy for access control |
| **Events** | Trigger Lambda/SQS/SNS on object changes |
| **Multipart** | Required >5 GB, recommended >100 MB, parallel upload |

---

## Quick Revision Questions

1. **What is the maximum size of a single S3 object?**
   > 5 TB. For uploads larger than 5 GB, multipart upload is required.

2. **What happens when you delete an object in a versioned bucket?**
   > S3 adds a "delete marker" as the current version. The object can be recovered by removing the delete marker.

3. **What is the difference between SSE-S3 and SSE-KMS?**
   > SSE-S3 uses AWS-managed keys with no audit trail. SSE-KMS uses KMS keys with CloudTrail audit logging and fine-grained access control.

4. **How do you enforce encryption on all uploads?**
   > Add a bucket policy that denies `s3:PutObject` when the encryption header is missing.

5. **What is a presigned URL?**
   > A time-limited URL that grants temporary access to a private S3 object without requiring AWS credentials.

---

[Next: 4.2 S3 Storage Classes & Lifecycle →](02-s3-storage-classes-lifecycle.md)

[← Back to README](../README.md)

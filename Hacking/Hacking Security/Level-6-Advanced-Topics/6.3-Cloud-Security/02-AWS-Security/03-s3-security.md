# Unit 2: AWS Security — Topic 3: S3 Security (Bucket Policies, ACLs)

## Overview

**Amazon S3 (Simple Storage Service)** is one of the most widely used and most frequently misconfigured AWS services. S3 stores everything from application assets to database backups and log files. Misconfigured S3 buckets have led to some of the largest cloud data breaches in history. Understanding S3 security — bucket policies, ACLs, public access blocks, and encryption — is critical for cloud security.

---

## 1. S3 Security Model

```
S3 ACCESS CONTROL LAYERS:

  ┌──────────────────────────────────────┐
  │       S3 PUBLIC ACCESS BLOCK         │  ← Account/Bucket level
  │       (Override all below)           │
  ├──────────────────────────────────────┤
  │       BUCKET POLICY                  │  ← Bucket level (JSON)
  │       (Resource-based)               │
  ├──────────────────────────────────────┤
  │       ACLs (Legacy)                  │  ← Bucket/Object level
  │       (Access Control Lists)         │
  ├──────────────────────────────────────┤
  │       IAM POLICIES                   │  ← User/Role level
  │       (Identity-based)               │
  ├──────────────────────────────────────┤
  │       VPC ENDPOINTS                  │  ← Network level
  │       (Restrict to VPC)              │
  └──────────────────────────────────────┘

ACCESS EVALUATION:
  1. Is Public Access Block enabled? → Block public access
  2. Explicit Deny in any policy? → DENIED
  3. Allow in bucket policy OR IAM policy? → ALLOWED
  4. Otherwise → DENIED

BUCKET NAMING:
  → Globally unique names
  → DNS-compatible (lowercase, no underscores)
  → 3-63 characters
  → Bucket name is part of URL:
    https://bucket-name.s3.amazonaws.com/key
    https://s3.amazonaws.com/bucket-name/key
```

---

## 2. S3 Bucket Policies

```
BUCKET POLICY EXAMPLES:

PUBLIC READ (DANGEROUS!):
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicRead",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*"
  }]
}
→ Principal: "*" = ANYONE on the internet!
→ This is what causes data breaches

RESTRICT TO SPECIFIC IP:
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "IPRestrict",
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": [
      "arn:aws:s3:::my-bucket",
      "arn:aws:s3:::my-bucket/*"
    ],
    "Condition": {
      "NotIpAddress": {
        "aws:SourceIp": "10.0.0.0/8"
      }
    }
  }]
}

ENFORCE ENCRYPTION:
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyUnencrypted",
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "StringNotEquals": {
        "s3:x-amz-server-side-encryption": "aws:kms"
      }
    }
  }]
}

ENFORCE HTTPS:
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyHTTP",
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "Bool": {
        "aws:SecureTransport": "false"
      }
    }
  }]
}

CROSS-ACCOUNT ACCESS:
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "CrossAccount",
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123456789012:root"
    },
    "Action": [
      "s3:GetObject",
      "s3:ListBucket"
    ],
    "Resource": [
      "arn:aws:s3:::my-bucket",
      "arn:aws:s3:::my-bucket/*"
    ]
  }]
}
```

---

## 3. Public Access Block

```
S3 BLOCK PUBLIC ACCESS:

SETTINGS (4 independent controls):
  1. BlockPublicAcls:
     → Block PUT requests with public ACLs
     → Prevents new public ACLs being set
  
  2. IgnorePublicAcls:
     → Ignore existing public ACLs
     → Existing public ACLs have no effect
  
  3. BlockPublicPolicy:
     → Block PUT of bucket policies that grant public access
     → Prevents new public policies
  
  4. RestrictPublicBuckets:
     → Restrict public bucket policies
     → Only AWS services and authorized users can access

RECOMMENDATION: ENABLE ALL FOUR — Always!

ACCOUNT LEVEL:
  # Block all public access for entire account
  aws s3control put-public-access-block \
    --account-id 123456789012 \
    --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,\
    BlockPublicPolicy=true,RestrictPublicBuckets=true

BUCKET LEVEL:
  # Block public access for specific bucket
  aws s3api put-public-access-block \
    --bucket my-bucket \
    --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,\
    BlockPublicPolicy=true,RestrictPublicBuckets=true

CHECK STATUS:
  # Account level
  aws s3control get-public-access-block --account-id 123456789012
  
  # Bucket level
  aws s3api get-public-access-block --bucket my-bucket
```

---

## 4. S3 Encryption

```
ENCRYPTION OPTIONS:

SERVER-SIDE ENCRYPTION (SSE):

SSE-S3 (Amazon managed keys):
  → AWS manages encryption keys
  → AES-256 encryption
  → Simplest option
  → Default encryption option
  → Header: x-amz-server-side-encryption: AES256

SSE-KMS (AWS KMS managed keys):
  → Customer Managed Key (CMK) in KMS
  → Key usage logged in CloudTrail
  → Access control via KMS key policy
  → Key rotation support
  → Header: x-amz-server-side-encryption: aws:kms

SSE-C (Customer provided keys):
  → Customer provides encryption key with each request
  → AWS encrypts/decrypts but doesn't store key
  → Customer manages keys entirely
  → Must provide key for every read/write

CLIENT-SIDE ENCRYPTION:
  → Encrypt before uploading
  → AWS never sees plaintext
  → Customer manages encryption entirely
  → Most secure but most complex

DEFAULT ENCRYPTION:
  # Enable default encryption on bucket
  aws s3api put-bucket-encryption \
    --bucket my-bucket \
    --server-side-encryption-configuration '{
      "Rules": [{
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "aws:kms",
          "KMSMasterKeyID": "arn:aws:kms:..."
        },
        "BucketKeyEnabled": true
      }]
    }'
```

---

## 5. S3 Security Assessment

```
FINDING MISCONFIGURATIONS:

AWS CLI CHECKS:
  # List all buckets
  aws s3 ls
  
  # Check bucket ACL
  aws s3api get-bucket-acl --bucket my-bucket
  
  # Check bucket policy
  aws s3api get-bucket-policy --bucket my-bucket
  
  # Check public access block
  aws s3api get-public-access-block --bucket my-bucket
  
  # Check encryption
  aws s3api get-bucket-encryption --bucket my-bucket
  
  # Check versioning
  aws s3api get-bucket-versioning --bucket my-bucket
  
  # Check logging
  aws s3api get-bucket-logging --bucket my-bucket

EXTERNAL ENUMERATION:
  # Try to list bucket contents (unauthenticated)
  curl https://bucket-name.s3.amazonaws.com/
  aws s3 ls s3://bucket-name --no-sign-request
  
  # Try to download objects
  curl https://bucket-name.s3.amazonaws.com/secret.txt
  
  # Bucket enumeration tools
  → S3Scanner
  → cloud_enum
  → bucket_finder

NOTABLE S3 BREACHES:
  → Capital One (2019): 100M+ records via SSRF
  → Twitch (2021): Source code via misconfigured server
  → Accenture (2017): 4 public buckets with client data
  → US Military (2017): Intelligence data in public bucket

SECURITY CHECKLIST:
  [ ] Public Access Block enabled (account + bucket)
  [ ] No public bucket policies
  [ ] No public ACLs
  [ ] Default encryption enabled (SSE-KMS preferred)
  [ ] HTTPS enforced via bucket policy
  [ ] Versioning enabled
  [ ] Access logging enabled
  [ ] Lifecycle policies configured
  [ ] MFA Delete enabled for critical buckets
  [ ] VPC endpoints for private access
```

---

## Summary Table

| Feature | Purpose | Recommendation |
|---------|---------|---------------|
| Public Access Block | Prevent public exposure | Enable all 4 settings |
| Bucket Policy | Resource-based access | Least privilege, no Principal: * |
| ACLs | Legacy access control | Disable (use policies instead) |
| SSE-KMS | Encryption at rest | Enable as default |
| HTTPS enforcement | Encryption in transit | Deny non-SSL via policy |
| Versioning | Data protection | Enable on all buckets |

---

## Revision Questions

1. What are the layers of S3 access control?
2. What makes a bucket policy dangerous with Principal: "*"?
3. How does S3 Block Public Access work?
4. What are the differences between SSE-S3, SSE-KMS, and SSE-C?
5. How can you assess S3 security from both internal and external perspectives?

---

*Previous: [02-iam-users-roles-policies.md](02-iam-users-roles-policies.md) | Next: [04-vpc-security.md](04-vpc-security.md)*

---

*[Back to README](../README.md)*

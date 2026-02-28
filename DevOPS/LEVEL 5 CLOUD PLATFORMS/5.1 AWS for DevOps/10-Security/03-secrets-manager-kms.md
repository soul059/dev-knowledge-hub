# Chapter 10.3: AWS Secrets Manager & KMS

## Overview

**AWS Secrets Manager** stores, rotates, and retrieves credentials (database passwords, API keys, tokens). **AWS Key Management Service (KMS)** creates and manages encryption keys used to encrypt data across AWS services. Together, they form the core of AWS secrets and encryption management for DevOps.

---

## Secrets Manager Architecture

```
┌──── Secrets Manager ──────────────────────────────────────┐
│                                                            │
│  Application / Lambda / ECS Task                          │
│       │                                                    │
│       │ GetSecretValue("prod/db/mysql")                   │
│       ▼                                                    │
│  ┌──────────────────────┐                                  │
│  │   Secrets Manager    │────── Encrypted with KMS key    │
│  │                      │                                  │
│  │  Secret Versions:    │     ┌─────────────────────┐     │
│  │   AWSCURRENT  ───────┼────►│ user: admin         │     │
│  │   AWSPREVIOUS ───────┼──┐  │ pass: P@ssw0rd2024  │     │
│  │   AWSPENDING  ──┐    │  │  │ host: rds.endpoint  │     │
│  │                  │    │  │  └─────────────────────┘     │
│  │                  ▼    │  │  ┌─────────────────────┐     │
│  │   (during rotation)  │  └─►│ (previous version)  │     │
│  └──────────────────────┘     └─────────────────────┘     │
│                                                            │
│  Automatic Rotation:                                      │
│  ┌────────┐    ┌────────────┐    ┌──────────────────┐    │
│  │Schedule │───►│ Lambda     │───►│ Update DB Pass   │    │
│  │(30 days)│    │ Rotator    │    │ + Secret Value   │    │
│  └────────┘    └────────────┘    └──────────────────┘    │
└────────────────────────────────────────────────────────────┘
```

---

## Secrets Manager CLI

```bash
# Create a secret
aws secretsmanager create-secret \
  --name "prod/db/mysql" \
  --description "Production MySQL credentials" \
  --secret-string '{"username":"admin","password":"P@ssw0rd2024","host":"mydb.cluster-abc.us-east-1.rds.amazonaws.com","port":3306}'

# Retrieve secret value
aws secretsmanager get-secret-value \
  --secret-id "prod/db/mysql" \
  --query 'SecretString' --output text

# Update secret value
aws secretsmanager put-secret-value \
  --secret-id "prod/db/mysql" \
  --secret-string '{"username":"admin","password":"NewP@ss2025"}'

# Enable automatic rotation (every 30 days)
aws secretsmanager rotate-secret \
  --secret-id "prod/db/mysql" \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:SecretsRotator \
  --rotation-rules '{"AutomaticallyAfterDays":30}'

# Tag a secret
aws secretsmanager tag-resource \
  --secret-id "prod/db/mysql" \
  --tags Key=Environment,Value=Production

# Delete secret (with recovery window)
aws secretsmanager delete-secret \
  --secret-id "prod/db/mysql" \
  --recovery-window-in-days 7
```

---

## Accessing Secrets in Code (Python)

```python
import json
import boto3

def get_db_credentials(secret_name):
    client = boto3.client('secretsmanager', region_name='us-east-1')
    
    response = client.get_secret_value(SecretId=secret_name)
    secret = json.loads(response['SecretString'])
    
    return {
        'host': secret['host'],
        'username': secret['username'],
        'password': secret['password'],
        'port': secret.get('port', 3306)
    }

# Lambda handler example
def handler(event, context):
    creds = get_db_credentials('prod/db/mysql')
    # Use creds to connect to database
```

---

## Secrets in ECS Task Definitions

```json
{
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/app:latest",
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/db/mysql:password::"
        },
        {
          "name": "API_KEY",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/api-key"
        }
      ]
    }
  ]
}
```

---

## AWS KMS Architecture

```
┌──── KMS Key Hierarchy ───────────────────────────────────┐
│                                                           │
│  ┌───────────────────────────────────────┐               │
│  │  KMS Customer Master Key (CMK)        │               │
│  │  (never leaves KMS in plaintext)     │               │
│  └──────────────┬────────────────────────┘               │
│                 │                                         │
│       ┌─────────┴─────────┐                              │
│       ▼                   ▼                              │
│  ┌──────────┐      ┌──────────┐                         │
│  │ Encrypt  │      │ Generate │                         │
│  │ ≤4 KB    │      │ Data Key │                         │
│  │ directly │      │ (for >4KB│                         │
│  └──────────┘      │  data)   │                         │
│                    └─────┬────┘                          │
│                          │                               │
│            ┌─────────────┴──────────────┐               │
│            ▼                            ▼               │
│  ┌──────────────────┐   ┌────────────────────┐         │
│  │ Plaintext Data   │   │ Encrypted Data Key │         │
│  │ Key (use & del)  │   │ (store with data)  │         │
│  └────────┬─────────┘   └────────────────────┘         │
│           │                                              │
│           ▼                                              │
│  ┌────────────────────────────────────┐                 │
│  │  Envelope Encryption:             │                 │
│  │  1. Generate Data Key via KMS     │                 │
│  │  2. Encrypt data with data key    │                 │
│  │  3. Store encrypted data +        │                 │
│  │     encrypted data key            │                 │
│  │  4. Discard plaintext data key    │                 │
│  └────────────────────────────────────┘                 │
└───────────────────────────────────────────────────────────┘
```

---

## KMS Key Types

```
┌──── Key Types ───────────────────────────────────────────┐
│                                                           │
│  1. AWS Managed Keys (aws/s3, aws/ebs, aws/rds)         │
│     - Created automatically by AWS services             │
│     - Rotated every year automatically                  │
│     - Cannot manage policy or delete                    │
│                                                           │
│  2. Customer Managed Keys (CMK)                          │
│     - You create and manage                             │
│     - Full control over key policy                      │
│     - Optional automatic rotation (every year)          │
│     - Can be disabled or scheduled for deletion         │
│                                                           │
│  3. AWS Owned Keys                                       │
│     - Used internally by AWS                            │
│     - Not visible in your account                       │
│     - Free                                              │
│                                                           │
│  Key Material Origin:                                    │
│  ├── AWS_KMS    → AWS generates (default)               │
│  ├── EXTERNAL   → You import key material               │
│  └── AWS_CLOUDHSM → Generated in CloudHSM cluster      │
└───────────────────────────────────────────────────────────┘
```

---

## KMS CLI Commands

```bash
# Create a symmetric encryption key
aws kms create-key \
  --description "Application encryption key" \
  --key-usage ENCRYPT_DECRYPT \
  --origin AWS_KMS

# Create an alias for easy reference
aws kms create-alias \
  --alias-name alias/app-encryption \
  --target-key-id <key-id>

# Encrypt data (up to 4 KB directly)
aws kms encrypt \
  --key-id alias/app-encryption \
  --plaintext fileb://plaintext.txt \
  --output text --query CiphertextBlob | base64 --decode > encrypted.bin

# Decrypt data
aws kms decrypt \
  --ciphertext-blob fileb://encrypted.bin \
  --output text --query Plaintext | base64 --decode > decrypted.txt

# Generate a data key (for envelope encryption)
aws kms generate-data-key \
  --key-id alias/app-encryption \
  --key-spec AES_256

# Enable automatic key rotation
aws kms enable-key-rotation --key-id <key-id>

# Schedule key deletion (7-30 day waiting period)
aws kms schedule-key-deletion \
  --key-id <key-id> \
  --pending-window-in-days 7

# List keys
aws kms list-keys

# Describe key
aws kms describe-key --key-id alias/app-encryption
```

---

## KMS Key Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RootAccountFullAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "KeyAdministrators",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/KeyAdmin"
      },
      "Action": [
        "kms:Create*",
        "kms:Describe*",
        "kms:Enable*",
        "kms:List*",
        "kms:Put*",
        "kms:Update*",
        "kms:Revoke*",
        "kms:Disable*",
        "kms:Get*",
        "kms:Delete*",
        "kms:ScheduleKeyDeletion",
        "kms:CancelKeyDeletion"
      ],
      "Resource": "*"
    },
    {
      "Sid": "KeyUsers",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/AppRole"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:GenerateDataKey",
        "kms:DescribeKey"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Encryption Across AWS Services

```
┌──── Service Encryption Integration ──────────────────────┐
│                                                           │
│  Service         Encryption Type     KMS Integration     │
│  ─────────────   ────────────────    ─────────────────   │
│  S3              SSE-S3/SSE-KMS/     ✓ Default or CMK   │
│                  SSE-C/Client-side                       │
│  EBS             AES-256             ✓ Default or CMK   │
│  RDS             AES-256             ✓ At creation only  │
│  DynamoDB        AES-256             ✓ AWS or CMK       │
│  EFS             AES-256             ✓ At creation only  │
│  Lambda (env)    AES-256             ✓ AWS or CMK       │
│  Secrets Mgr     AES-256             ✓ Always encrypted │
│  CloudWatch Logs AES-256             ✓ Optional CMK     │
│  SQS             AES-256             ✓ Optional         │
│  SNS             AES-256             ✓ Optional         │
│                                                           │
│  ⚠ Some services: encryption at rest cannot be disabled  │
│     once enabled (EBS snapshots, RDS)                    │
└───────────────────────────────────────────────────────────┘
```

---

## SSM Parameter Store vs Secrets Manager

```
┌──── Comparison ──────────────────────────────────────────┐
│                                                           │
│  Feature            Parameter Store   Secrets Manager    │
│  ──────────────     ─────────────     ────────────────   │
│  Cost               Free (standard)   $0.40/secret/mo   │
│  Max size           8 KB (adv)        64 KB              │
│  Auto rotation      ✗ No             ✓ Yes (Lambda)    │
│  Cross-account      ✗ Limited        ✓ Resource policy  │
│  RDS integration    ✗ Manual         ✓ Built-in         │
│  Versioning         ✓ Labels         ✓ Staging labels  │
│  Hierarchy          ✓ Paths (/a/b)   ✗ Names only      │
│  KMS encryption     ✓ Optional       ✓ Always          │
│  CloudFormation     ✓ Dynamic ref    ✓ Dynamic ref     │
│                                                           │
│  Use Parameter Store for: config values, feature flags  │
│  Use Secrets Manager for: DB credentials, API keys,     │
│                            anything needing rotation     │
└───────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: RDS Secret Rotation

```bash
# 1. Store RDS credentials in Secrets Manager
aws secretsmanager create-secret \
  --name "prod/rds/postgres" \
  --secret-string '{"username":"dbadmin","password":"InitPass123","engine":"postgres","host":"mydb.cluster-abc.us-east-1.rds.amazonaws.com","port":5432,"dbname":"appdb"}'

# 2. Enable rotation with managed Lambda
aws secretsmanager rotate-secret \
  --secret-id "prod/rds/postgres" \
  --rotation-lambda-arn arn:aws:lambda:us-east-1:123456789012:function:SecretsManagerRDSPostgreSQLRotationSingleUser \
  --rotation-rules '{"ScheduleExpression":"rate(30 days)"}'
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `AccessDeniedException` on decrypt | Missing `kms:Decrypt` permission | Add KMS decrypt to IAM role + key policy |
| Secret rotation fails | Lambda can't reach RDS or Secrets Manager | Ensure Lambda has VPC config + SM VPC endpoint |
| `KMSInvalidStateException` | Key is pending deletion or disabled | Re-enable key or cancel deletion |
| Cross-account secret access denied | No resource policy on secret | Add resource policy allowing cross-account `GetSecretValue` |
| High KMS costs | Too many API requests | Use data key caching (encryption SDK) |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Secrets Manager** | Stores credentials. Auto rotation with Lambda. Always encrypted. |
| **KMS** | Managed encryption keys. Envelope encryption for large data. |
| **Key Types** | AWS Managed, Customer Managed (CMK), AWS Owned |
| **Envelope Encryption** | Generate data key → encrypt data → store encrypted key + data |
| **Key Policy** | Controls who can manage vs use the key. Root account anchors. |
| **Rotation** | Secrets: configurable schedule. KMS CMK: annual automatic. |

---

## Quick Revision Questions

1. **What is envelope encryption?**
   > KMS generates a data key (plaintext + encrypted). You encrypt data with the plaintext key, discard it, and store the encrypted data key alongside the encrypted data. To decrypt, KMS decrypts the data key first.

2. **Can you encrypt more than 4 KB directly with KMS?**
   > No. For data > 4 KB, you must use envelope encryption (generate a data key and encrypt locally).

3. **Do SCPs affect the management account?**
   > No. SCPs never apply to the management account — only to member accounts.

4. **What is the difference between Secrets Manager and Parameter Store?**
   > Secrets Manager supports automatic rotation and is always encrypted ($0.40/secret/month). Parameter Store is free (standard tier), supports hierarchical paths, but has no built-in rotation.

5. **What happens when you schedule a KMS key for deletion?**
   > The key enters a waiting period (7-30 days) during which it cannot be used. If not cancelled, it is permanently deleted along with all data encrypted with it.

6. **How do you use Secrets Manager in ECS?**
   > Reference the secret ARN in the task definition's `secrets` block. ECS injects the value as an environment variable at container startup.

---

[← Previous: 10.2 Organizations & SCPs](02-organizations-scps.md) | [Next: 10.4 WAF & Shield →](04-waf-shield.md)

[← Back to README](../README.md)

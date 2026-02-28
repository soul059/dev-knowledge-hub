# Chapter 1.3: AWS Console, CLI, and SDK

## Overview

AWS provides **three primary interfaces** to interact with its services. Every action in AWS is ultimately an **API call** — the Console, CLI, and SDK are simply different ways to make those calls. Understanding all three is essential for DevOps engineers who need to automate everything.

---

## The Three Interfaces

```
┌─────────────────────────────────────────────────────────────────┐
│                  AWS API (The Foundation)                        │
│                                                                 │
│   Everything is an API call signed with your credentials        │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────────────────┐  │
│  │   CONSOLE   │  │     CLI     │  │         SDK            │  │
│  │  (Browser)  │  │  (Terminal) │  │  (Code: Python/JS/etc) │  │
│  │             │  │             │  │                        │  │
│  │  • Visual   │  │  • Scripts  │  │  • Applications        │  │
│  │  • Manual   │  │  • Automtn  │  │  • Custom tools        │  │
│  │  • Learning │  │  • CI/CD    │  │  • Lambda functions    │  │
│  └──────┬──────┘  └──────┬──────┘  └───────────┬────────────┘  │
│         │               │                     │                │
│         └───────────────┼─────────────────────┘                │
│                         ▼                                       │
│              ┌──────────────────┐                               │
│              │   AWS API        │                               │
│              │   (HTTPS REST)   │                               │
│              │   SigV4 Signing  │                               │
│              └──────────────────┘                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1. AWS Management Console

The web-based graphical interface at [console.aws.amazon.com](https://console.aws.amazon.com).

### Key Features

| Feature | Description |
|---------|-------------|
| **Service Search** | Search bar to quickly find any service |
| **Region Selector** | Top-right dropdown to switch regions |
| **CloudShell** | Browser-based terminal with pre-installed CLI |
| **Resource Groups** | Organize resources with tags |
| **Favorites** | Pin frequently used services to the navbar |

### Console Best Practices

```
┌─────────────────────────────────────────────────────┐
│           CONSOLE BEST PRACTICES                    │
│                                                     │
│  ✓ Use MFA on root account (ALWAYS)                │
│  ✓ Create IAM users; don't use root for daily work │
│  ✓ Use tag-based resource groups for navigation     │
│  ✓ Check region before creating resources           │
│  ✗ Don't rely on console for production changes     │
│  ✗ Don't share console passwords                    │
└─────────────────────────────────────────────────────┘
```

### CloudShell

AWS CloudShell provides a **browser-based terminal** with:
- Pre-installed AWS CLI, Python, Node.js, git
- 1 GB persistent storage per region
- No extra charge (included with AWS account)
- Automatic credential management

```
    ┌──────────────── Browser ────────────────────┐
    │  AWS Console                                 │
    │  ┌────────────────────────────────────────┐  │
    │  │  $ aws s3 ls                           │  │
    │  │  2026-01-15 my-bucket                  │  │
    │  │  2026-02-20 logs-bucket                │  │
    │  │  $ aws ec2 describe-instances \        │  │
    │  │      --query 'Reservations[*]...'      │  │
    │  │  ▐                                     │  │
    │  │                                        │  │
    │  │  CloudShell — No setup required!       │  │
    │  └────────────────────────────────────────┘  │
    └──────────────────────────────────────────────┘
```

---

## 2. AWS CLI (Command Line Interface)

The CLI is a **unified tool to manage all AWS services** from the terminal.

### Installation

```bash
# Windows (MSI installer or chocolatey)
choco install awscli

# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
# aws-cli/2.x.x Python/3.x.x ...
```

### Configuration

```bash
# Interactive setup (stores in ~/.aws/credentials and ~/.aws/config)
aws configure

# AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name [None]: us-east-1
# Default output format [None]: json
```

### Configuration Files

```
~/.aws/
├── credentials          # Access keys (NEVER commit to git!)
│   [default]
│   aws_access_key_id = AKIA...
│   aws_secret_access_key = wJal...
│
│   [dev-account]
│   aws_access_key_id = AKIA...
│   aws_secret_access_key = wJal...
│
└── config               # Region, output format, profiles
    [default]
    region = us-east-1
    output = json

    [profile dev-account]
    region = eu-west-1
    output = table
```

### Named Profiles

```bash
# Configure a named profile
aws configure --profile production

# Use a specific profile
aws s3 ls --profile production

# Set profile for entire session
export AWS_PROFILE=production       # Linux/Mac
$env:AWS_PROFILE = "production"     # PowerShell

# List configured profiles
aws configure list-profiles
```

### Essential CLI Commands

```bash
# ──────────── IDENTITY ────────────
aws sts get-caller-identity              # Who am I?

# ──────────── S3 ────────────
aws s3 ls                                # List all buckets
aws s3 cp file.txt s3://my-bucket/       # Upload file
aws s3 sync ./folder s3://my-bucket/     # Sync directory
aws s3 mb s3://new-bucket                # Create bucket

# ──────────── EC2 ────────────
aws ec2 describe-instances               # List instances
aws ec2 start-instances --instance-ids i-xxx
aws ec2 stop-instances --instance-ids i-xxx
aws ec2 run-instances \
  --image-id ami-0abcdef \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-xxx \
  --subnet-id subnet-xxx

# ──────────── IAM ────────────
aws iam list-users
aws iam create-user --user-name devuser
aws iam attach-user-policy \
  --user-name devuser \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess

# ──────────── LAMBDA ────────────
aws lambda list-functions
aws lambda invoke --function-name myFunc output.json
```

### CLI Output Formats

```bash
# JSON (default, best for scripting)
aws ec2 describe-instances --output json

# Table (human-readable)
aws ec2 describe-instances --output table

# Text (tab-separated, good for Unix tools)
aws ec2 describe-instances --output text
```

### Filtering with --query (JMESPath)

```bash
# Get only instance IDs
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].InstanceId" \
  --output text

# Get instance ID, type, and state
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]" \
  --output table

# Filter running instances only
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType}" \
  --output table

# Get specific S3 bucket
aws s3api list-buckets \
  --query "Buckets[?Name=='my-bucket']"
```

### CLI Pagination

```bash
# Some commands return paginated results
# Use --no-paginate to get everything (small datasets)
aws s3api list-objects --bucket my-bucket --no-paginate

# Or control page size
aws ec2 describe-instances --page-size 10 --max-items 50
```

### CLI Waiter Commands

```bash
# Wait until an instance is running before proceeding
aws ec2 wait instance-running --instance-ids i-xxx

# Wait until RDS instance is available
aws rds wait db-instance-available --db-instance-identifier mydb

# Useful in scripts!
echo "Launching instance..."
aws ec2 run-instances --image-id ami-xxx --instance-type t3.micro
aws ec2 wait instance-running --instance-ids i-xxx
echo "Instance is running!"
```

---

## 3. AWS SDKs

SDKs let you interact with AWS **programmatically** from your application code or automation scripts.

### Available SDKs

| Language | Package | Install |
|----------|---------|---------|
| Python | `boto3` | `pip install boto3` |
| JavaScript/Node.js | `@aws-sdk/client-*` | `npm install @aws-sdk/client-s3` |
| Java | `software.amazon.awssdk` | Maven/Gradle |
| Go | `github.com/aws/aws-sdk-go-v2` | `go get` |
| .NET | `AWSSDK.*` | NuGet |
| Ruby | `aws-sdk-*` | `gem install aws-sdk-s3` |

### Python (boto3) Examples

```python
import boto3

# ──── S3 Operations ────
s3 = boto3.client('s3')

# List buckets
response = s3.list_buckets()
for bucket in response['Buckets']:
    print(f"  {bucket['Name']}")

# Upload a file
s3.upload_file('local-file.txt', 'my-bucket', 'remote-file.txt')

# Download a file
s3.download_file('my-bucket', 'remote-file.txt', 'downloaded.txt')

# ──── EC2 Operations ────
ec2 = boto3.resource('ec2')

# List all running instances
instances = ec2.instances.filter(
    Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
)
for instance in instances:
    print(f"  {instance.id} — {instance.instance_type}")

# ──── DynamoDB Operations ────
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

# Put item
table.put_item(Item={
    'user_id': '123',
    'name': 'Alice',
    'email': 'alice@example.com'
})

# Get item
response = table.get_item(Key={'user_id': '123'})
print(response['Item'])
```

### Client vs Resource (boto3)

```python
# CLIENT — low-level, mirrors API 1:1
s3_client = boto3.client('s3')
response = s3_client.list_objects_v2(Bucket='my-bucket')

# RESOURCE — high-level, object-oriented (more Pythonic)
s3_resource = boto3.resource('s3')
bucket = s3_resource.Bucket('my-bucket')
for obj in bucket.objects.all():
    print(obj.key)
```

| Aspect | Client | Resource |
|--------|--------|----------|
| Level | Low-level | High-level |
| Returns | Dictionaries | Python objects |
| Coverage | All services | Subset of services |
| Use when | Need full API control | Want simpler code |

---

## Authentication Flow

```
┌──────────────────────────────────────────────────────────────┐
│               AWS CREDENTIAL RESOLUTION ORDER                │
│                                                              │
│  1. ──► Environment Variables                                │
│         AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY             │
│         │                                                    │
│  2. ──► Shared Credentials File (~/.aws/credentials)         │
│         │                                                    │
│  3. ──► AWS Config File (~/.aws/config)                      │
│         │                                                    │
│  4. ──► Container Credentials (ECS task role)                │
│         │                                                    │
│  5. ──► Instance Profile (EC2 instance role)                 │
│         │                                                    │
│  6. ──► SSO / Web Identity Token                             │
│                                                              │
│  ★ Best Practice: Use IAM Roles (not access keys)           │
│    for EC2, Lambda, ECS, etc.                                │
└──────────────────────────────────────────────────────────────┘
```

### Using IAM Roles Instead of Keys

```bash
# On EC2: Attach IAM role to instance (no keys needed!)
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxx \
  --iam-instance-profile Name=MyInstanceRole

# The SDK/CLI automatically uses the instance role
# No credentials in code or config files!
```

### Assume Role (Cross-Account Access)

```bash
# Assume a role in another account
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/CrossAccountRole \
  --role-session-name my-session

# Use the returned credentials
export AWS_ACCESS_KEY_ID=ASIAxxxx
export AWS_SECRET_ACCESS_KEY=yyyy
export AWS_SESSION_TOKEN=zzzz
```

---

## CLI Automation Recipes

### Script: Find All Untagged EC2 Instances

```bash
#!/bin/bash
echo "=== Untagged EC2 Instances ==="
aws ec2 describe-instances \
  --query "Reservations[*].Instances[?!Tags].[InstanceId,InstanceType,State.Name]" \
  --output table
```

### Script: Cleanup Old EBS Snapshots

```bash
#!/bin/bash
DAYS=30
DATE=$(date -d "-${DAYS} days" +%Y-%m-%d)
echo "Deleting snapshots older than $DATE..."

aws ec2 describe-snapshots \
  --owner-ids self \
  --query "Snapshots[?StartTime<='${DATE}'].SnapshotId" \
  --output text | tr '\t' '\n' | while read snap_id; do
    echo "Deleting $snap_id"
    aws ec2 delete-snapshot --snapshot-id "$snap_id"
done
```

### Script: Tag All Resources in a Region

```bash
#!/bin/bash
# Tag all EC2 instances with environment
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].InstanceId" \
  --output text | tr '\t' '\n' | while read id; do
    aws ec2 create-tags --resources "$id" \
      --tags Key=Environment,Value=Production
    echo "Tagged $id"
done
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `Unable to locate credentials` | No credentials configured | Run `aws configure` or attach IAM role |
| `ExpiredTokenException` | Temporary credentials expired | Re-assume role or refresh SSO login |
| `AccessDeniedException` | Missing IAM permissions | Check IAM policy; use `aws sts get-caller-identity` to verify identity |
| Wrong region results | CLI using wrong region | Check `aws configure get region`; use `--region` flag |
| `SignatureDoesNotMatch` | Clock skew on machine | Sync system clock with NTP |

---

## Summary Table

| Interface | Best For | Key Command / Feature |
|-----------|----------|----------------------|
| **Console** | Learning, visual exploration, one-off tasks | Browser at console.aws.amazon.com |
| **CLI** | Scripting, automation, CI/CD pipelines | `aws <service> <action> --parameters` |
| **SDK** | Application integration, Lambda code, custom tools | `boto3` (Python), `@aws-sdk` (JS) |
| **CloudShell** | Quick CLI access without local setup | Browser terminal with pre-installed tools |
| **Credentials** | Auth for all interfaces | Env vars → credentials file → instance role |
| **--query** | Filter CLI output | JMESPath expressions |

---

## Quick Revision Questions

1. **What are the three ways to interact with AWS services?**
   > Console (browser GUI), CLI (command line), and SDK (programmatic via code libraries).

2. **In what order does the AWS SDK resolve credentials?**
   > Environment variables → credentials file → config file → container credentials → instance profile → SSO/web identity token.

3. **What is the --query flag in AWS CLI used for?**
   > It filters JSON output using JMESPath expressions, letting you extract specific fields from API responses.

4. **Why should you prefer IAM Roles over access keys for EC2 instances?**
   > Roles provide temporary, automatically-rotated credentials. Access keys are long-lived secrets that can be leaked.

5. **What is AWS CloudShell?**
   > A browser-based terminal in the AWS Console with pre-installed CLI, Python, Node.js, and git. No local setup needed.

6. **What's the difference between boto3 Client and Resource?**
   > Client is low-level (returns dictionaries, covers all services). Resource is high-level (returns Python objects, more Pythonic, covers subset of services).

---

[← Previous: 1.2 Regions and Availability Zones](02-regions-and-availability-zones.md) | [Next: 1.4 IAM Basics →](04-iam-basics.md)

[← Back to README](../README.md)

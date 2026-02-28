# Chapter 2.1: EC2 Instances

## Overview

Amazon Elastic Compute Cloud (EC2) is the **backbone of AWS compute**. It provides resizable virtual servers (instances) in the cloud, giving you full control over the operating system, networking, and storage. For DevOps, EC2 is where most applications run.

---

## EC2 Architecture

```
┌─────────────────────── AWS Region ───────────────────────┐
│                                                           │
│  ┌─────── Availability Zone ──────────────────────────┐  │
│  │                                                     │  │
│  │  ┌──────────── EC2 Instance ────────────────────┐  │  │
│  │  │                                              │  │  │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │  │  │
│  │  │  │   vCPU   │  │  Memory  │  │ Instance │  │  │  │
│  │  │  │  (cores) │  │  (GiB)   │  │  Store   │  │  │  │
│  │  │  └──────────┘  └──────────┘  └──────────┘  │  │  │
│  │  │                                              │  │  │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │  │  │
│  │  │  │   ENI    │  │   EBS    │  │ Security │  │  │  │
│  │  │  │(Network) │  │ (Disk)   │  │  Group   │  │  │  │
│  │  │  └──────────┘  └──────────┘  └──────────┘  │  │  │
│  │  │                                              │  │  │
│  │  │  OS: Amazon Linux / Ubuntu / Windows / etc   │  │  │
│  │  └──────────────────────────────────────────────┘  │  │
│  │                                                     │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────┘
```

---

## Launching EC2 Instances

### What You Need to Decide

```
┌─────────────────────────────────────────────────────────┐
│           EC2 LAUNCH DECISION CHECKLIST                  │
│                                                          │
│  1. AMI ──────────► Which operating system / image?     │
│  2. Instance Type ► How much CPU, RAM, network?         │
│  3. Key Pair ─────► SSH key for access                  │
│  4. VPC / Subnet ─► Which network / AZ?                │
│  5. Security Group► Firewall rules (ports)              │
│  6. Storage ──────► EBS volume size and type             │
│  7. IAM Role ─────► Permissions for the instance        │
│  8. User Data ────► Bootstrap script (runs at launch)   │
│  9. Tags ─────────► Name, Environment, Team, etc.       │
└─────────────────────────────────────────────────────────┘
```

### Launch via CLI

```bash
# Launch a basic EC2 instance
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --iam-instance-profile Name=EC2-SSM-Role \
  --tag-specifications \
    'ResourceType=instance,Tags=[{Key=Name,Value=WebServer},{Key=Env,Value=Dev}]' \
  --user-data file://bootstrap.sh \
  --block-device-mappings '[{
    "DeviceName": "/dev/xvda",
    "Ebs": {
      "VolumeSize": 20,
      "VolumeType": "gp3",
      "Encrypted": true
    }
  }]'
```

### User Data (Bootstrap Script)

User data runs **once at first boot** (by default).

```bash
#!/bin/bash
# bootstrap.sh — runs as root at instance launch

# Update system
yum update -y

# Install web server
yum install -y httpd
systemctl start httpd
systemctl enable httpd

# Install CloudWatch agent
yum install -y amazon-cloudwatch-agent

# Install CodeDeploy agent
yum install -y ruby wget
cd /home/ec2-user
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
chmod +x ./install
./install auto

# Create health check page
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```

---

## Instance Lifecycle

```
┌──────────┐     Start      ┌──────────┐    Terminate   ┌──────────────┐
│ PENDING  │ ──────────────► │ RUNNING  │ ─────────────► │ SHUTTING-DOWN│
└──────────┘                 └────┬─────┘                └──────┬───────┘
                                  │                             │
                              Stop│                             ▼
                                  ▼                      ┌──────────────┐
                             ┌──────────┐                │ TERMINATED   │
                             │ STOPPING │                │ (gone in ~1h)│
                             └────┬─────┘                └──────────────┘
                                  │
                                  ▼
                             ┌──────────┐    Start
                             │ STOPPED  │ ──────────► PENDING ──► RUNNING
                             └──────────┘

  BILLING:
  • RUNNING  = charged per second (minimum 60 seconds)
  • STOPPED  = NO compute charge (EBS still charged)
  • TERMINATED = NO charges (EBS deleted if configured)
```

### Instance Lifecycle Commands

```bash
# Start a stopped instance
aws ec2 start-instances --instance-ids i-0123456789abcdef0

# Stop a running instance
aws ec2 stop-instances --instance-ids i-0123456789abcdef0

# Terminate (permanently delete)
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0

# Reboot (no downtime charge, keeps public IP)
aws ec2 reboot-instances --instance-ids i-0123456789abcdef0

# Check instance state
aws ec2 describe-instance-status --instance-ids i-0123456789abcdef0
```

---

## EC2 Networking

```
┌──────────────────── VPC ─────────────────────────────┐
│                                                       │
│  ┌──────── Public Subnet ──────┐                     │
│  │                              │                     │
│  │  ┌──── EC2 Instance ─────┐  │                     │
│  │  │                       │  │                     │
│  │  │  Private IP: 10.0.1.5│  │  Public IP:         │
│  │  │  (always assigned)    │  │  54.23.x.x          │
│  │  │                       │  │  (optional, changes │
│  │  │  ENI (eth0)           │  │   on stop/start)    │
│  │  └───────────────────────┘  │                     │
│  │                              │  Elastic IP:        │
│  │  Security Group:             │  (static, persists) │
│  │  ┌─ Inbound ─────────────┐  │                     │
│  │  │ SSH (22) from my IP   │  │                     │
│  │  │ HTTP (80) from 0/0    │  │                     │
│  │  │ HTTPS (443) from 0/0  │  │                     │
│  │  └───────────────────────┘  │                     │
│  └──────────────────────────────┘                     │
└───────────────────────────────────────────────────────┘
```

### IP Address Types

| Type | Persistence | Cost | Use Case |
|------|-------------|------|----------|
| **Private IP** | Always assigned, persists | Free | Internal communication |
| **Public IP** | Changes on stop/start | Free while attached | Internet access |
| **Elastic IP** | Static, you control | $0.005/hr if NOT attached | Stable endpoint |

```bash
# Allocate an Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with instance
aws ec2 associate-address \
  --instance-id i-xxx \
  --allocation-id eipalloc-xxx

# Release (IMPORTANT: release when done to avoid charges)
aws ec2 release-address --allocation-id eipalloc-xxx
```

---

## EC2 Storage Options

```
┌────────────────────────────────────────────────────────────┐
│               EC2 STORAGE OPTIONS                          │
│                                                            │
│  ┌──────────────────┐  ┌──────────────────┐               │
│  │   EBS Volumes    │  │  Instance Store  │               │
│  │  (Persistent)    │  │  (Ephemeral)     │               │
│  │                  │  │                  │               │
│  │  • Network-based │  │  • Physically    │               │
│  │  • Persists on   │  │    attached disk │               │
│  │    stop/terminate│  │  • LOST on stop  │               │
│  │    (configurable)│  │    or terminate  │               │
│  │  • Snapshots     │  │  • Highest IOPS  │               │
│  │  • Encryptable   │  │  • Best for temp │               │
│  │  • Multiple types│  │    data/cache    │               │
│  └──────────────────┘  └──────────────────┘               │
│                                                            │
│  ┌──────────────────┐                                     │
│  │   EFS            │  Mount NFS to multiple EC2         │
│  │  (Shared)        │  instances across AZs              │
│  └──────────────────┘                                     │
└────────────────────────────────────────────────────────────┘
```

---

## Connecting to EC2

### SSH (Linux)

```bash
# Connect with SSH key
ssh -i "my-key.pem" ec2-user@54.23.x.x

# For Ubuntu AMI
ssh -i "my-key.pem" ubuntu@54.23.x.x

# Fix permission error
chmod 400 my-key.pem
```

### Session Manager (Recommended for DevOps)

No SSH key needed, no port 22 open, fully audited.

```bash
# Install SSM plugin locally
# Then connect:
aws ssm start-session --target i-0123456789abcdef0

# Run a command remotely
aws ssm send-command \
  --instance-ids i-0123456789abcdef0 \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["uptime","df -h","free -m"]'
```

### EC2 Instance Connect

```bash
# Push temporary SSH key and connect (no permanent key needed)
aws ec2-instance-connect send-ssh-public-key \
  --instance-id i-xxx \
  --availability-zone us-east-1a \
  --instance-os-user ec2-user \
  --ssh-public-key file://my-temp-key.pub
```

---

## EC2 Metadata Service

Every EC2 instance can query its own metadata:

```bash
# IMDSv2 (recommended — requires token)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Get instance ID
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id

# Get instance type
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-type

# Get public IP
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/public-ipv4

# Get IAM role credentials
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/MyRole

# Get user data (bootstrap script)
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/user-data
```

---

## Real-World Scenarios

### Scenario 1: Web Application Server
- **Instance:** t3.medium (2 vCPU, 4 GiB RAM)
- **AMI:** Amazon Linux 2023
- **Storage:** 20 GB gp3 root + 100 GB gp3 data
- **Security Group:** HTTP/HTTPS from ALB, SSH from bastion only
- **User Data:** Install nginx, CodeDeploy agent, CloudWatch agent

### Scenario 2: Build Agent (CI/CD)
- **Instance:** c5.xlarge (4 vCPU, 8 GiB RAM) — Spot Instance
- **AMI:** Custom with Docker, build tools pre-installed
- **Lifecycle:** Launch on-demand, terminate after build
- **Cost:** ~70-90% less than on-demand

### Scenario 3: Database Server
- **Instance:** r5.2xlarge (8 vCPU, 64 GiB RAM)
- **Storage:** io2 EBS (high IOPS)
- **Placement:** Multi-AZ for standby
- **Note:** Consider RDS instead for managed experience

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| `InsufficientInstanceCapacity` | No capacity in AZ for that type | Try different AZ or instance type |
| `InstanceLimitExceeded` | Account limit reached | Request limit increase via Support |
| SSH timeout | Security group port 22 not open | Add inbound rule for SSH |
| Can't reach instance | No public IP or wrong route table | Check subnet has IGW route; assign public IP |
| Instance immediately terminates | AMI issue or EBS volume limit | Check system log: `aws ec2 get-console-output` |
| User data not running | Script errors or wrong shebang | Check `/var/log/cloud-init-output.log` |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **EC2** | Virtual servers in the cloud. Full OS control. Choose AMI, type, network, storage. |
| **Launch Config** | AMI + Instance Type + Key Pair + VPC/Subnet + SG + EBS + IAM Role + User Data. |
| **Lifecycle** | Pending → Running → Stopped → Terminated. Billed per-second when running. |
| **Networking** | Private IP (always), Public IP (optional, changes), Elastic IP (static). |
| **Storage** | EBS (persistent), Instance Store (ephemeral), EFS (shared filesystem). |
| **Access** | SSH, Session Manager (recommended), EC2 Instance Connect. |
| **Metadata** | Query at 169.254.169.254 for instance info, role credentials, user data. |

---

## Quick Revision Questions

1. **What happens to the public IP when you stop and start an EC2 instance?**
   > It changes. Use an Elastic IP if you need a static public IP.

2. **What is User Data and when does it run?**
   > A bootstrap script that runs as root at first launch. Used to install software, configure services, etc.

3. **What is the difference between stopping and terminating an EC2 instance?**
   > Stopped: instance preserved, can be restarted (EBS data retained, no compute charge). Terminated: permanently deleted.

4. **Why is Session Manager preferred over SSH for DevOps?**
   > No SSH keys to manage, no port 22 needed, all sessions logged to CloudTrail/S3, works through NAT.

5. **What is IMDSv2 and why is it recommended?**
   > Instance Metadata Service v2 requires a session token, preventing SSRF attacks from accessing metadata.

6. **What is an Elastic Network Interface (ENI)?**
   > A virtual network card attached to an EC2 instance. Contains private IP, public IP, MAC address, security groups.

---

[← Previous: 1.5 AWS Pricing Model](../01-AWS-Fundamentals/05-aws-pricing-model.md) | [Next: 2.2 EC2 Instance Types and Sizing →](02-ec2-instance-types-and-sizing.md)

[← Back to README](../README.md)

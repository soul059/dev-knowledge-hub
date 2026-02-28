# Chapter 2.3: AMIs and Launch Templates

## Overview

An **Amazon Machine Image (AMI)** is a pre-configured template for EC2 instances containing the OS, software, and configuration. **Launch Templates** define the complete configuration to launch instances consistently. Together, they're essential for DevOps automation and immutable infrastructure.

---

## AMI Concepts

```
┌────────────────────────── AMI ──────────────────────────────┐
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Root Volume │  │  Data Volumes│  │   Metadata       │  │
│  │  Snapshot    │  │  Snapshots   │  │                  │  │
│  │              │  │  (optional)  │  │  • ami-id        │  │
│  │  OS +        │  │              │  │  • Architecture  │  │
│  │  Software +  │  │  /data       │  │  • Permissions   │  │
│  │  Config      │  │  /logs       │  │  • Region        │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                              │
│  AMI = Snapshot(s) + Launch Permissions + Block Device Map  │
└──────────────────────────────────────────────────────────────┘
```

### AMI Types

| Type | Description | Example |
|------|-------------|---------|
| **AWS Provided** | Official Amazon/Ubuntu/Windows AMIs | Amazon Linux 2023, Ubuntu 24.04 |
| **Marketplace** | Third-party vendor AMIs (may have cost) | Fortinet, Splunk, WordPress |
| **Community** | Shared by other users (use with caution) | Custom community builds |
| **Custom (yours)** | Created from your configured instances | Your hardened, pre-baked AMIs |

### Golden AMI Pattern (DevOps Best Practice)

```
┌────────────────────────────────────────────────────────────────┐
│                 GOLDEN AMI PIPELINE                            │
│                                                                │
│  Base AMI ──► Configure ──► Test ──► Create AMI ──► Deploy   │
│  (Amazon      (Install       (Run     (Snapshot     (Use in   │
│   Linux)       agents,       tests,    the          Auto      │
│               harden OS,    scan      instance)    Scaling)  │
│               pre-bake      for                              │
│               software)     vulns)                            │
│                                                                │
│  ┌──────┐   ┌─────────┐   ┌──────┐   ┌────────┐  ┌───────┐ │
│  │ Base │──►│Configure│──►│ Test │──►│Create  │─►│Deploy │ │
│  │ AMI  │   │Instance │   │ AMI  │   │Golden  │  │  ASG  │ │
│  └──────┘   └─────────┘   └──────┘   │  AMI   │  └───────┘ │
│                                       └────────┘             │
│                                                                │
│  Automated with: EC2 Image Builder or Packer (HashiCorp)     │
└────────────────────────────────────────────────────────────────┘
```

### Creating a Custom AMI

```bash
# 1. Launch and configure a base instance
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t3.micro \
  --key-name my-key

# 2. SSH in and install/configure software
ssh -i my-key.pem ec2-user@<public-ip>
sudo yum update -y
sudo yum install -y docker nginx aws-cli
sudo systemctl enable docker nginx
# ... configure, harden, test

# 3. Create AMI from the instance
aws ec2 create-image \
  --instance-id i-0123456789abcdef0 \
  --name "webapp-golden-v1.2" \
  --description "Web app with nginx, docker, agents" \
  --no-reboot  # Use --reboot for consistent snapshot

# 4. Wait for AMI to be available
aws ec2 wait image-available --image-ids ami-xxx

# 5. Use AMI to launch new instances
aws ec2 run-instances --image-id ami-xxx --instance-type t3.medium
```

### AMI Cross-Region Copy

```bash
# Copy AMI to another region (for DR or multi-region deployment)
aws ec2 copy-image \
  --source-image-id ami-xxx \
  --source-region us-east-1 \
  --region eu-west-1 \
  --name "webapp-golden-v1.2-eu"

# Share AMI with another AWS account
aws ec2 modify-image-attribute \
  --image-id ami-xxx \
  --launch-permission "Add=[{UserId=123456789012}]"
```

---

## EC2 Image Builder

Automate AMI creation with a managed pipeline.

```
┌────────────────────────────────────────────────────────────┐
│              EC2 IMAGE BUILDER PIPELINE                     │
│                                                            │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌────────┐│
│  │  Source   │──►│  Build   │──►│  Test    │──►│Output  ││
│  │  Image   │   │Components│   │Components│   │  AMI   ││
│  └──────────┘   └──────────┘   └──────────┘   └────────┘│
│                                                            │
│  Components (YAML):                                        │
│  • Install packages (yum, apt)                            │
│  • Copy files                                              │
│  • Run scripts                                             │
│  • CIS benchmark hardening                                │
│  • Vulnerability scanning                                  │
│                                                            │
│  Schedule: CRON, manual, or on dependency update          │
│  Distribution: Multi-region, multi-account                │
└────────────────────────────────────────────────────────────┘
```

### Packer (HashiCorp Alternative)

```hcl
# packer-template.pkr.hcl
source "amazon-ebs" "webapp" {
  ami_name      = "webapp-golden-{{timestamp}}"
  instance_type = "t3.micro"
  region        = "us-east-1"
  
  source_ami_filter {
    filters = {
      name                = "amzn2-ami-hvm-*-x86_64-gp2"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    owners      = ["amazon"]
    most_recent = true
  }
  
  ssh_username = "ec2-user"
}

build {
  sources = ["source.amazon-ebs.webapp"]

  provisioner "shell" {
    inline = [
      "sudo yum update -y",
      "sudo yum install -y docker nginx",
      "sudo systemctl enable docker nginx"
    ]
  }

  provisioner "file" {
    source      = "configs/nginx.conf"
    destination = "/tmp/nginx.conf"
  }

  provisioner "shell" {
    inline = [
      "sudo cp /tmp/nginx.conf /etc/nginx/nginx.conf"
    ]
  }
}
```

```bash
# Build with Packer
packer init .
packer validate packer-template.pkr.hcl
packer build packer-template.pkr.hcl
```

---

## Launch Templates

A **Launch Template** captures all the parameters needed to launch an EC2 instance — like a reusable "recipe."

```
┌──────────────── Launch Template ─────────────────────────┐
│                                                           │
│  Name: webapp-template                                   │
│  Version: 3 (Latest)                                     │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  AMI:            ami-0abc123 (Golden AMI v1.2)      │ │
│  │  Instance Type:  t3.medium                          │ │
│  │  Key Pair:       webapp-key                         │ │
│  │  Security Groups: sg-web, sg-ssh                    │ │
│  │  IAM Role:       WebAppRole                         │ │
│  │  User Data:      #!/bin/bash ...                    │ │
│  │  EBS:            20GB gp3 (encrypted)               │ │
│  │  Tags:           Env=Prod, App=WebApp               │ │
│  │  Monitoring:     Detailed (1-min CloudWatch)        │ │
│  │  Network:        Subnet auto-assigned               │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                           │
│  Used by: Auto Scaling Groups, EC2 Fleet, Spot Fleet    │
└───────────────────────────────────────────────────────────┘
```

### Create a Launch Template

```bash
aws ec2 create-launch-template \
  --launch-template-name webapp-template \
  --version-description "v1 - initial" \
  --launch-template-data '{
    "ImageId": "ami-0abc123def456",
    "InstanceType": "t3.medium",
    "KeyName": "webapp-key",
    "SecurityGroupIds": ["sg-0123456789abcdef0"],
    "IamInstanceProfile": {
      "Name": "WebAppRole"
    },
    "BlockDeviceMappings": [
      {
        "DeviceName": "/dev/xvda",
        "Ebs": {
          "VolumeSize": 20,
          "VolumeType": "gp3",
          "Encrypted": true
        }
      }
    ],
    "TagSpecifications": [
      {
        "ResourceType": "instance",
        "Tags": [
          {"Key": "Name", "Value": "WebApp"},
          {"Key": "Environment", "Value": "Production"}
        ]
      }
    ],
    "UserData": "'$(base64 -w0 bootstrap.sh)'",
    "Monitoring": {"Enabled": true},
    "MetadataOptions": {
      "HttpTokens": "required",
      "HttpEndpoint": "enabled"
    }
  }'
```

### Launch Template Versioning

```bash
# Create new version (update AMI)
aws ec2 create-launch-template-version \
  --launch-template-name webapp-template \
  --version-description "v2 - new AMI" \
  --source-version 1 \
  --launch-template-data '{"ImageId": "ami-0newversion123"}'

# Set default version
aws ec2 modify-launch-template \
  --launch-template-name webapp-template \
  --default-version "2"

# List all versions
aws ec2 describe-launch-template-versions \
  --launch-template-name webapp-template

# Launch from specific version
aws ec2 run-instances \
  --launch-template LaunchTemplateName=webapp-template,Version=2

# Launch from latest version
aws ec2 run-instances \
  --launch-template LaunchTemplateName=webapp-template,Version='$Latest'
```

### Launch Templates vs Launch Configurations

| Feature | Launch Template | Launch Configuration |
|---------|----------------|---------------------|
| Versioning | ✓ Multiple versions | ✗ Immutable |
| Mixed instances | ✓ Multiple types in ASG | ✗ Single type |
| Spot + On-Demand | ✓ Mixed | ✗ One or other |
| Inheritance | ✓ From existing template | ✗ No |
| T2/T3 unlimited | ✓ Supported | ✗ Not supported |
| Status | **Current (recommended)** | **Legacy (deprecated)** |

---

## Real-World Scenarios

### Scenario 1: Immutable Infrastructure Deployment
1. Build Golden AMI with Packer (weekly)
2. Store AMI ID in SSM Parameter Store
3. Update Launch Template with new AMI
4. Auto Scaling Group performs rolling update
5. Old instances replaced with new AMI instances

```
┌──────┐    ┌───────┐    ┌──────────┐    ┌─────┐    ┌─────┐
│Packer│───►│ AMI   │───►│ Launch   │───►│ ASG │───►│EC2s │
│Build │    │ Store │    │ Template │    │     │    │(new)│
└──────┘    └───────┘    └──────────┘    └─────┘    └─────┘
```

### Scenario 2: Multi-Architecture Support
- Build AMI for both x86 and ARM (Graviton)
- Launch Template specifies mixed instance types
- ASG uses both for cost optimization

### Scenario 3: Disaster Recovery
- Copy Golden AMI to DR region nightly
- Launch Template in DR region references local AMI
- On failover, ASG in DR region scales up

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| AMI creation takes too long | Large EBS volumes | Use `--no-reboot` for faster creation |
| AMI not found in other region | AMIs are region-specific | Copy AMI to target region |
| Launch template version conflict | ASG using wrong version | Set `$Latest` or `$Default` explicitly |
| User data not executing | Base64 encoding issue | Ensure script is base64-encoded in template |
| Packer build fails | Security group missing egress | Allow outbound internet in builder SG |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **AMI** | Template for EC2: OS + software + config. Region-specific. Can copy cross-region. |
| **Golden AMI** | Pre-baked, tested, hardened AMI. Built via pipeline (Image Builder/Packer). |
| **AMI Types** | AWS-provided, Marketplace, Community, Custom. |
| **Launch Template** | Reusable instance config. Versioned. Required for modern ASG features. |
| **Versioning** | Templates support versions. Use `$Latest` or `$Default` in ASG. |
| **Image Builder** | AWS-managed AMI pipeline: build → test → distribute. |
| **Packer** | HashiCorp tool for multi-cloud image building. HCL config. |

---

## Quick Revision Questions

1. **What is a Golden AMI and why is it important for DevOps?**
   > A pre-configured, tested, hardened AMI used as the standard base image. Ensures consistency across all instances and enables immutable infrastructure.

2. **Are AMIs global or regional?**
   > Regional. To use an AMI in another region, you must copy it with `aws ec2 copy-image`.

3. **What is the advantage of Launch Templates over Launch Configurations?**
   > Templates support versioning, mixed instance types, Spot + On-Demand, and T3 unlimited. Launch Configurations are legacy and immutable.

4. **How does EC2 Image Builder differ from Packer?**
   > Image Builder is AWS-managed (no server to run). Packer is multi-cloud, open-source, runs from your machine/CI. Both automate AMI creation.

5. **What does `--no-reboot` do when creating an AMI?**
   > Creates the AMI without stopping the instance. Faster, but may result in inconsistent filesystem state. Use `--reboot` for production AMIs.

6. **How would you automate AMI updates across an Auto Scaling Group?**
   > Update the Launch Template with the new AMI ID → Trigger an ASG instance refresh → Old instances are gradually replaced with new ones.

---

[← Previous: 2.2 EC2 Instance Types](02-ec2-instance-types-and-sizing.md) | [Next: 2.4 Auto Scaling Groups →](04-auto-scaling-groups.md)

[← Back to README](../README.md)

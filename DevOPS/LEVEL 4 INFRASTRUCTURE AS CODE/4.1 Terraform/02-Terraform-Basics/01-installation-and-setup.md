# Chapter 1: Installation and Setup

[← Previous: Terraform Overview](../01-IaC-Concepts/05-terraform-overview.md) | [Back to README](../README.md) | [Next: Provider Configuration →](02-provider-configuration.md)

---

## Overview

This chapter covers how to install Terraform on different operating systems, configure cloud provider credentials, set up your development environment, and verify everything works end to end.

---

## 1.1 Installation Methods

```
┌─────────────────────────────────────────────────────────────┐
│              TERRAFORM INSTALLATION OPTIONS                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│   │   Manual     │  │  Package Mgr │  │   Docker     │     │
│   │  Download    │  │  (Recommended)│  │  Container   │     │
│   ├──────────────┤  ├──────────────┤  ├──────────────┤     │
│   │ Download zip │  │ brew install │  │ docker pull  │     │
│   │ Unzip binary │  │ choco install│  │ hashicorp/   │     │
│   │ Add to PATH  │  │ apt install  │  │ terraform    │     │
│   └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 1.2 Installation by OS

### Windows

```powershell
# Method 1: Chocolatey (Recommended)
choco install terraform

# Method 2: Scoop
scoop install terraform

# Method 3: Manual Download
# 1. Go to https://developer.hashicorp.com/terraform/downloads
# 2. Download Windows AMD64 zip
# 3. Extract terraform.exe
# 4. Add to System PATH:
#    Settings → System → About → Advanced → Environment Variables
#    → Path → Add folder containing terraform.exe
```

### macOS

```bash
# Method 1: Homebrew (Recommended)
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Update later
brew upgrade hashicorp/tap/terraform
```

### Linux (Ubuntu/Debian)

```bash
# Add HashiCorp GPG key and repository
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt-get update && sudo apt-get install terraform
```

### Linux (CentOS/RHEL)

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum install terraform
```

### Verify Installation

```bash
# Check version
terraform -version
# Terraform v1.7.x
# on windows_amd64

# Check available commands
terraform -help
```

---

## 1.3 Version Management with tfenv

Managing multiple Terraform versions is common (different projects may need different versions):

```bash
# Install tfenv (macOS/Linux)
brew install tfenv               # macOS
git clone https://github.com/tfutils/tfenv.git ~/.tfenv   # Linux

# List available versions
tfenv list-remote

# Install specific version
tfenv install 1.7.0
tfenv install 1.6.6

# Switch versions
tfenv use 1.7.0

# Pin version for a project (creates .terraform-version file)
echo "1.7.0" > .terraform-version

# tfenv auto-switches when entering the directory
```

```
  ┌─────────────────────────────────────────────────┐
  │            VERSION PINNING FLOW                 │
  ├─────────────────────────────────────────────────┤
  │                                                  │
  │   Project A/                Project B/          │
  │   ├── .terraform-version    ├── .terraform-version
  │   │   "1.7.0"               │   "1.6.6"        │
  │   ├── main.tf               ├── main.tf        │
  │   └── ...                   └── ...            │
  │                                                  │
  │   cd project-a → tfenv auto-selects 1.7.0      │
  │   cd project-b → tfenv auto-selects 1.6.6      │
  │                                                  │
  └─────────────────────────────────────────────────┘
```

---

## 1.4 Cloud Provider Credentials Setup

### AWS Credentials

```bash
# Method 1: AWS CLI (Recommended)
aws configure
# AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region: us-east-1
# Default output format: json

# Method 2: Environment Variables
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="us-east-1"

# Method 3: Shared credentials file (~/.aws/credentials)
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# Method 4: IAM Instance Profile (for EC2 — BEST for production)
# No keys needed — role attached to EC2 instance
```

### Azure Credentials

```bash
# Login with Azure CLI
az login

# Set subscription
az account set --subscription "My Subscription"

# Service Principal for automation
az ad sp create-for-rbac --name "terraform" --role="Contributor" \
  --scopes="/subscriptions/SUBSCRIPTION_ID"

# Set environment variables
export ARM_CLIENT_ID="app-id"
export ARM_CLIENT_SECRET="password"
export ARM_SUBSCRIPTION_ID="subscription-id"
export ARM_TENANT_ID="tenant-id"
```

### GCP Credentials

```bash
# Login with gcloud CLI
gcloud auth application-default login

# Service Account for automation
gcloud iam service-accounts create terraform \
  --description="Terraform service account" \
  --display-name="Terraform"

# Create and download key
gcloud iam service-accounts keys create key.json \
  --iam-account=terraform@PROJECT_ID.iam.gserviceaccount.com

# Set environment variable
export GOOGLE_CREDENTIALS=$(cat key.json)
```

### Credential Priority Order

```
┌─────────────────────────────────────────────────────────┐
│          TERRAFORM CREDENTIAL RESOLUTION ORDER          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   1. Provider block (AVOID — hardcoded secrets!)        │
│      provider "aws" { access_key = "..." }              │
│                          │                               │
│   2. Environment variables (GOOD for CI/CD)             │
│      AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY           │
│                          │                               │
│   3. Shared credentials file (GOOD for local dev)       │
│      ~/.aws/credentials                                  │
│                          │                               │
│   4. Instance profile / managed identity (BEST)         │
│      IAM role attached to the machine                    │
│                                                          │
│   RULE: Never hardcode credentials in .tf files!        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 1.5 IDE Setup

### VS Code (Recommended)

```
  ┌────────────────────────────────────────────────────┐
  │           VS CODE EXTENSIONS FOR TERRAFORM         │
  ├────────────────────────────────────────────────────┤
  │                                                     │
  │  1. HashiCorp Terraform                            │
  │     • Syntax highlighting                          │
  │     • Auto-completion                              │
  │     • Go-to-definition                             │
  │     • Format on save                               │
  │                                                     │
  │  2. Terraform doc snippets                         │
  │     • Quick reference snippets                     │
  │                                                     │
  │  3. GitLens (optional)                             │
  │     • See who changed what in .tf files            │
  │                                                     │
  └────────────────────────────────────────────────────┘
```

**VS Code Settings (settings.json):**

```json
{
  "[terraform]": {
    "editor.defaultFormatter": "hashicorp.terraform",
    "editor.formatOnSave": true,
    "editor.formatOnSaveMode": "file"
  },
  "[terraform-vars]": {
    "editor.defaultFormatter": "hashicorp.terraform",
    "editor.formatOnSave": true
  }
}
```

---

## 1.6 First Project Setup — Step by Step

```bash
# 1. Create project directory
mkdir my-first-terraform && cd my-first-terraform

# 2. Create configuration file
cat > main.tf << 'EOF'
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example" {
  bucket = "my-tf-test-bucket-unique-name-12345"

  tags = {
    Name        = "My First Bucket"
    Environment = "learning"
    ManagedBy   = "terraform"
  }
}

output "bucket_arn" {
  value = aws_s3_bucket.example.arn
}
EOF

# 3. Initialize
terraform init

# 4. Format code
terraform fmt

# 5. Validate configuration
terraform validate

# 6. Preview changes
terraform plan

# 7. Apply (create resources)
terraform apply

# 8. View outputs
terraform output

# 9. Cleanup
terraform destroy
```

### What Happens During `terraform init`

```
  $ terraform init
  
  Initializing the backend...          ← Sets up state storage
  
  Initializing provider plugins...     ← Downloads providers
  - Finding hashicorp/aws versions matching "~> 5.0"...
  - Installing hashicorp/aws v5.31.0...
  - Installed hashicorp/aws v5.31.0 (signed by HashiCorp)
  
  Terraform has created a lock file .terraform.lock.hcl
  
  Terraform has been successfully initialized!
  
  Generated files:
  ┌── .terraform/              ← Provider binaries
  │   └── providers/
  │       └── registry.terraform.io/
  │           └── hashicorp/aws/5.31.0/
  └── .terraform.lock.hcl     ← Version lock file
```

---

## 1.7 .gitignore for Terraform

```gitignore
# .gitignore for Terraform projects

# Local .terraform directories
**/.terraform/*

# .tfstate files (use remote state instead)
*.tfstate
*.tfstate.*

# Crash log files
crash.log
crash.*.log

# Variable files with secrets
*.tfvars
*.tfvars.json

# Override files
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# CLI configuration files
.terraformrc
terraform.rc

# Lock file (usually committed — check team conventions)
# .terraform.lock.hcl
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `terraform: command not found` | Binary not in PATH | Add terraform binary location to system PATH |
| `Error: No valid credential sources found` | AWS credentials not configured | Run `aws configure` or set environment variables |
| `Error: Failed to query available provider packages` | No internet / firewall | Check connectivity, or use provider mirror |
| `terraform init` is slow | Large providers downloading | Normal first time; `.terraform/` is cached |
| Version mismatch between team members | No version pinning | Use `required_version` in config + tfenv |
| `Error: Unsupported Terraform Core version` | Project needs different version | Install required version via tfenv |

---

## Summary Table

| Topic | Key Points |
|-------|------------|
| **Installation** | Use package manager (brew, choco, apt) for easiest setup |
| **Version Management** | Use tfenv to manage multiple Terraform versions per project |
| **AWS Credentials** | Use `aws configure`, env vars, or IAM roles — never hardcode |
| **Azure Credentials** | Use `az login` or service principal with env vars |
| **GCP Credentials** | Use `gcloud auth` or service account key file |
| **IDE** | VS Code + HashiCorp Terraform extension for best experience |
| **Project Init** | `terraform init` downloads providers and sets up backend |
| **Version Pinning** | Always set `required_version` and provider version constraints |
| **.gitignore** | Exclude `.terraform/`, `*.tfstate`, `*.tfvars` |

---

## Quick Revision Questions

1. **What is the command to verify Terraform is installed correctly?**
   > `terraform -version` — shows the installed version and platform.

2. **Why should you never hardcode cloud credentials in `.tf` files?**
   > Credentials in code can be committed to Git, exposing secrets. Use environment variables, CLI profiles, or instance roles instead.

3. **What does `terraform init` do, and when must you run it?**
   > It downloads providers, initializes the backend, and sets up the working directory. Run it when starting a new project or after changing providers/backend.

4. **What is tfenv, and why is it useful?**
   > tfenv is a version manager for Terraform. It lets you install and switch between multiple versions per project using `.terraform-version` files.

5. **Which files should be excluded from version control in a Terraform project?**
   > `.terraform/` directory, `*.tfstate` files, `*.tfvars` (may contain secrets), and crash logs.

6. **What is the `.terraform.lock.hcl` file?**
   > It locks provider versions to ensure everyone on the team uses the same provider versions. It should usually be committed to Git.

---

[← Previous: Terraform Overview](../01-IaC-Concepts/05-terraform-overview.md) | [Back to README](../README.md) | [Next: Provider Configuration →](02-provider-configuration.md)

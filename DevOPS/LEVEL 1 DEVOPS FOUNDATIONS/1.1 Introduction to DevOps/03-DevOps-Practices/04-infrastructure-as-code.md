# Chapter 3.4 — Infrastructure as Code (IaC)

[← Previous: Continuous Deployment](03-continuous-deployment.md) | [Next: Configuration Management →](05-configuration-management.md)

---

## Overview

**Infrastructure as Code (IaC)** is the practice of managing and provisioning infrastructure through machine-readable definition files rather than manual processes. Instead of clicking through cloud consoles, you write code that describes your desired infrastructure state.

---

## Manual vs IaC

```
┌──────────────────────────────────────────────────────────┐
│  MANUAL (Click-Ops)              IaC (Code-Ops)          │
│  ══════════════════              ══════════════           │
│                                                          │
│  1. Log into AWS Console         1. Write Terraform file │
│  2. Click "Launch Instance"      2. git commit & push    │
│  3. Select AMI...                3. PR review            │
│  4. Choose instance type...      4. terraform apply      │
│  5. Configure security group..   5. Done ✅               │
│  6. Add tags...                                          │
│  7. Review and launch...                                 │
│                                                          │
│  Problems:                       Benefits:               │
│  ├── Not reproducible            ├── 100% reproducible   │
│  ├── No audit trail              ├── Full git history    │
│  ├── Human errors                ├── Peer-reviewed       │
│  ├── Slow and tedious            ├── Fast and automated  │
│  ├── Can't version control       ├── Version controlled  │
│  └── Config drift over time      └── Consistent state    │
└──────────────────────────────────────────────────────────┘
```

---

## Declarative vs Imperative IaC

```
┌──────────────────────────────────────────────────────────┐
│  TWO APPROACHES TO IaC                                   │
│                                                          │
│  DECLARATIVE (WHAT)              IMPERATIVE (HOW)        │
│  ══════════════════              ════════════════         │
│                                                          │
│  "I want 3 web servers"         "Create server 1,        │
│                                  create server 2,        │
│  resource "aws_instance" {       create server 3"        │
│    count = 3                                             │
│    ami   = "ami-xxx"            aws ec2 run-instances    │
│    type  = "t3.medium"            --count 3              │
│  }                                --image-id ami-xxx     │
│                                   --instance-type t3.med │
│  Tools:                                                  │
│  ├── Terraform                   Tools:                  │
│  ├── CloudFormation              ├── AWS CLI scripts     │
│  ├── Pulumi                      ├── Ansible             │
│  └── Kubernetes YAML             └── Bash/Python scripts │
│                                                          │
│  ✅ Idempotent                   ⚠️ Must handle state    │
│  ✅ Self-documenting             ⚠️ Order matters        │
│  ✅ Drift detection              ⚠️ No built-in drift    │
└──────────────────────────────────────────────────────────┘
```

---

## Terraform: The Most Popular IaC Tool

### Basic Terraform Workflow

```
┌──────────────────────────────────────────────────────────┐
│  TERRAFORM WORKFLOW                                      │
│                                                          │
│  ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐             │
│  │Write │──►│ Init │──►│ Plan │──►│Apply │             │
│  │ .tf  │   │      │   │      │   │      │             │
│  └──────┘   └──────┘   └──────┘   └──────┘             │
│     │          │          │          │                   │
│     ▼          ▼          ▼          ▼                   │
│  Define     Download   Preview   Execute                │
│  desired    providers  changes   changes                │
│  state      & modules  (dry run) (real)                 │
│                                                          │
│  terraform init  → terraform plan → terraform apply      │
└──────────────────────────────────────────────────────────┘
```

### Complete Terraform Example

```hcl
# providers.tf
terraform {
  required_version = ">= 1.7"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "production/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
}

# variables.tf
variable "aws_region" {
  default = "us-east-1"
}

variable "environment" {
  default = "production"
}

# main.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "${var.environment}-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = false
}

resource "aws_instance" "web" {
  count         = 3
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.medium"
  subnet_id     = module.vpc.private_subnets[count.index]

  tags = {
    Name        = "web-${count.index + 1}"
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

### Terraform Commands

```bash
# Initialize working directory (download providers)
terraform init

# Preview changes (dry run — ALWAYS do this first!)
terraform plan -out=tfplan

# Apply changes (create/modify infrastructure)
terraform apply tfplan

# Show current state
terraform show

# Destroy all managed infrastructure
terraform destroy

# Format code
terraform fmt

# Validate configuration
terraform validate

# Import existing resources into Terraform state
terraform import aws_instance.web i-1234567890abcdef0
```

---

## IaC Architecture Patterns

### Module Structure

```
infrastructure/
├── modules/                    # Reusable modules
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── eks/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── rds/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── environments/               # Environment-specific configs
│   ├── dev/
│   │   ├── main.tf            # Uses modules with dev params
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── production/
│       ├── main.tf
│       └── terraform.tfvars
└── README.md
```

### State Management

```
┌──────────────────────────────────────────────────────────┐
│  TERRAFORM STATE                                         │
│                                                          │
│  State File = Terraform's knowledge of real-world infra  │
│                                                          │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐        │
│  │ .tf Code │     │ State    │     │ Real     │        │
│  │ (desired)│ ──► │ File     │ ──► │ Infra    │        │
│  │          │     │ (known)  │     │ (actual) │        │
│  └──────────┘     └──────────┘     └──────────┘        │
│       │                │                │               │
│       └────terraform plan──────────────┘               │
│            (compares desired vs actual)                 │
│                                                          │
│  ⚠️ State should be stored REMOTELY:                    │
│  ├── AWS S3 + DynamoDB (locking)                        │
│  ├── Azure Blob Storage                                 │
│  ├── Google Cloud Storage                               │
│  └── Terraform Cloud                                    │
│                                                          │
│  ❌ NEVER store state locally or in Git                  │
│  (contains secrets, causes conflicts)                   │
└──────────────────────────────────────────────────────────┘
```

---

## IaC Tools Comparison

| Feature | Terraform | CloudFormation | Pulumi | Ansible |
|---------|-----------|---------------|--------|---------|
| **Type** | Declarative | Declarative | Declarative | Procedural/Declarative |
| **Language** | HCL | JSON/YAML | Python/TS/Go | YAML |
| **Cloud** | Multi-cloud | AWS only | Multi-cloud | Multi-cloud |
| **State** | State file | Managed by AWS | State file | Stateless |
| **Best For** | Multi-cloud IaC | AWS-native | Dev-friendly IaC | Config + IaC |
| **Learning Curve** | Medium | Medium | Low (if you know Python) | Low |

---

## Real-World Scenario: Disaster Recovery with IaC

### 🏢 Entire Region Goes Down

```
WITHOUT IaC:
├── "What servers did we have?"
├── "What were the configurations?"
├── "Who set up the database?"
├── Rebuild manually from memory + old docs
├── Recovery time: 2-5 DAYS
└── Some configs permanently lost

WITH IaC:
├── git clone infrastructure-repo
├── cd environments/production
├── Update region in terraform.tfvars: "us-west-2"
├── terraform init
├── terraform apply
├── Recovery time: 30-60 MINUTES
└── Exact replica of original infrastructure
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| State drift | Manual changes outside Terraform | Run `terraform plan` regularly; use drift detection |
| State file conflicts | Multiple people applying at once | Use remote state with locking (S3 + DynamoDB) |
| Destroy accident | `terraform destroy` run accidentally | Use `prevent_destroy` lifecycle rule + plan reviews |
| Secrets in state | State file contains sensitive data | Encrypt state at rest (S3 encryption), restrict access |
| Slow plans | Large state file, many resources | Split into smaller state files (workspaces / separate projects) |
| Module version mismatch | Teams using different module versions | Pin module versions, use a module registry |

---

## Summary Table

| IaC Concept | Description |
|------------|-------------|
| **Core Idea** | Define infrastructure in code, not through GUIs |
| **Declarative** | Describe WHAT you want (Terraform, CloudFormation) |
| **Imperative** | Describe HOW to do it (scripts, Ansible) |
| **Idempotent** | Running the same code gives the same result |
| **State** | Tracks what Terraform manages vs what exists |
| **Modules** | Reusable infrastructure components |
| **Workflow** | Write → Init → Plan → Apply |
| **Key Benefit** | Reproducible, version-controlled, reviewable infrastructure |

---

## Quick Revision Questions

1. **What is Infrastructure as Code, and what problem does it solve?**
   <details><summary>Answer</summary>IaC is the practice of defining infrastructure (servers, networks, databases) in code files instead of manual processes. It solves problems of inconsistency, lack of reproducibility, configuration drift, no audit trail, and the inability to version control or peer review infrastructure changes.</details>

2. **What is the difference between declarative and imperative IaC?**
   <details><summary>Answer</summary>Declarative (e.g., Terraform): you describe the desired end state ("I want 3 servers"), and the tool figures out how to achieve it. Imperative (e.g., Bash scripts): you describe the step-by-step process ("create server 1, create server 2, create server 3"). Declarative is preferred because it's idempotent and handles state management.</details>

3. **Why should Terraform state be stored remotely?**
   <details><summary>Answer</summary>Remote state: 1) Enables team collaboration (shared access). 2) Supports locking (prevents concurrent modifications). 3) Can be encrypted at rest (protection for sensitive data). 4) Prevents loss (backed up). Local state causes conflicts, can be lost, and contains unencrypted secrets.</details>

4. **What is idempotency, and why is it important for IaC?**
   <details><summary>Answer</summary>Idempotency means applying the same configuration multiple times produces the same result. If you run `terraform apply` twice with no code changes, nothing happens the second time. This is critical because it means you can safely re-run IaC without worrying about creating duplicate resources or unintended changes.</details>

5. **How does IaC enable disaster recovery?**
   <details><summary>Answer</summary>With IaC, your entire infrastructure is defined in code stored in Git. If a region goes down, you can recreate the entire environment by changing the region variable and running `terraform apply`. Recovery takes minutes instead of days, and you get an exact replica of the original infrastructure.</details>

6. **What is configuration drift, and how does IaC prevent it?**
   <details><summary>Answer</summary>Configuration drift occurs when the actual state of infrastructure diverges from the intended state — usually due to manual changes. IaC prevents it by: 1) Making all changes through code (no manual edits). 2) Running `terraform plan` to detect differences. 3) Using GitOps to ensure the code always reflects the desired state.</details>

---

[← Previous: Continuous Deployment](03-continuous-deployment.md) | [Next: Configuration Management →](05-configuration-management.md)

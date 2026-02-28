# Chapter 8.4: Terraform with AWS

## Overview

Terraform by HashiCorp is an **open-source, multi-cloud IaC tool** that uses HCL (HashiCorp Configuration Language) to define infrastructure. Unlike CloudFormation (AWS-only), Terraform manages resources across AWS, Azure, GCP, and hundreds of other providers.

---

## Terraform Architecture

```
┌──── Terraform Workflow ──────────────────────────────────┐
│                                                           │
│  .tf Files         Terraform CLI        Cloud Provider   │
│  (HCL)                                                    │
│  ┌──────────┐    ┌──────────────┐    ┌────────────────┐  │
│  │ main.tf  │    │ terraform    │    │ AWS            │  │
│  │ vars.tf  │──→ │ plan         │──→ │ Azure          │  │
│  │ output.tf│    │ apply        │    │ GCP            │  │
│  └──────────┘    │ destroy      │    │ Kubernetes     │  │
│                  └──────┬───────┘    └────────────────┘  │
│                         │                                 │
│                  ┌──────▼───────┐                         │
│                  │ State File   │                         │
│                  │ terraform    │                         │
│                  │ .tfstate     │                         │
│                  │ (S3 + Dynamo │                         │
│                  │  for remote) │                         │
│                  └──────────────┘                         │
│                                                           │
│  Plan: Preview changes | Apply: Execute | Destroy: Tear  │
└───────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
my-infra/
├── main.tf              # Provider config + main resources
├── variables.tf         # Input variable declarations
├── outputs.tf           # Output values
├── terraform.tfvars     # Variable values (don't commit secrets)
├── providers.tf         # Provider configuration
├── backend.tf           # Remote state configuration
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── ecs/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── environments/
    ├── dev.tfvars
    ├── staging.tfvars
    └── prod.tfvars
```

---

## Provider and Backend Configuration

```hcl
# providers.tf
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "terraform"
      Project     = var.project_name
    }
  }
}

# backend.tf — Remote state in S3
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

---

## Example: VPC + EC2

```hcl
# variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.medium"
}

# main.tf
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.environment}-vpc"
  }
}

resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.environment}-public-${count.index + 1}"
  }
}

data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  count                  = var.environment == "prod" ? 2 : 1
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public[count.index % length(aws_subnet.public)].id
  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    echo "<h1>Server ${count.index + 1}</h1>" > /var/www/html/index.html
  EOF

  tags = {
    Name = "${var.environment}-web-${count.index + 1}"
  }
}

resource "aws_security_group" "web" {
  name_prefix = "${var.environment}-web-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# outputs.tf
output "instance_ips" {
  description = "Public IPs of web servers"
  value       = aws_instance.web[*].public_ip
}

output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}
```

---

## Terraform CLI Commands

```bash
# Initialize (download providers, configure backend)
terraform init

# Format code
terraform fmt -recursive

# Validate configuration
terraform validate

# Plan (preview changes)
terraform plan -var-file=environments/prod.tfvars

# Apply (create/update resources)
terraform apply -var-file=environments/prod.tfvars

# Apply with auto-approve (CI/CD)
terraform apply -auto-approve -var-file=environments/prod.tfvars

# Destroy all resources
terraform destroy -var-file=environments/prod.tfvars

# Show current state
terraform show

# List resources in state
terraform state list

# Import existing resource into state
terraform import aws_instance.web i-1234567890abcdef0

# Remove resource from state (without destroying)
terraform state rm aws_instance.web
```

---

## Remote State & Locking

```
┌──── Remote State (S3 + DynamoDB) ────────────────────────┐
│                                                          │
│  terraform apply                                         │
│       │                                                  │
│       ▼                                                  │
│  ┌─────────────┐    ┌────────────────────┐               │
│  │ DynamoDB    │    │ S3 Bucket          │               │
│  │ Lock Table  │    │ terraform.tfstate  │               │
│  │             │    │                    │               │
│  │ Acquire ────│───→│ Read state         │               │
│  │ lock        │    │ Plan changes       │               │
│  │             │    │ Apply changes      │               │
│  │             │    │ Write new state    │               │
│  │ Release ────│───→│ State updated     │               │
│  │ lock        │    │                    │               │
│  └─────────────┘    └────────────────────┘               │
│                                                          │
│  ✓ Prevents concurrent modifications                    │
│  ✓ State encrypted at rest (S3 SSE)                     │
│  ✓ Versioning for state history                         │
└──────────────────────────────────────────────────────────┘
```

```bash
# Create DynamoDB lock table
aws dynamodb create-table \
  --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

---

## Modules

```hcl
# Using a module
module "vpc" {
  source = "./modules/vpc"
  # or: source = "terraform-aws-modules/vpc/aws"

  environment = var.environment
  cidr_block  = "10.0.0.0/16"
  azs         = ["us-east-1a", "us-east-1b"]
}

module "ecs" {
  source = "./modules/ecs"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| State lock error | Previous apply crashed | `terraform force-unlock <LOCK_ID>` |
| Provider version conflict | Incompatible provider version | Pin version in `required_providers`; run `terraform init -upgrade` |
| Resource already exists | Importing existing infrastructure | Use `terraform import` to bring into state |
| Circular dependency | Resources depend on each other | Refactor using `depends_on` or separate resources |
| Plan shows unwanted changes | State drift from manual changes | Import changes or run `terraform apply` to reconcile |
| Destroy fails | Dependencies prevent deletion | Destroy dependent resources first or use `-target` |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Terraform** | Multi-cloud IaC using HCL. Plan → Apply → Destroy. |
| **State** | Tracks real infrastructure. Store remotely in S3 + DynamoDB. |
| **Providers** | Plugins for AWS, Azure, GCP, K8s, etc. |
| **Modules** | Reusable infrastructure components. Local or registry. |
| **Data Sources** | Query existing resources (AMIs, AZs, etc.) |
| **Workspaces** | Manage multiple environments with same config |

---

## Quick Revision Questions

1. **What is Terraform state and why is it important?**
   > The state file maps Terraform config to real infrastructure. It tracks resource IDs, dependencies, and metadata needed for planning and applying changes.

2. **Why use remote state with S3 and DynamoDB?**
   > S3 stores the state file (encrypted, versioned). DynamoDB provides state locking to prevent concurrent modifications by multiple users.

3. **What is the difference between `terraform plan` and `terraform apply`?**
   > `plan` shows what changes will be made without executing them. `apply` executes the changes to create, update, or delete resources.

4. **How do you manage multiple environments?**
   > Use separate `.tfvars` files per environment (dev/staging/prod) or Terraform workspaces.

5. **What advantage does Terraform have over CloudFormation?**
   > Multi-cloud support, a single language (HCL) for all providers, and a rich module ecosystem. CloudFormation is AWS-only but has tighter AWS integration.

6. **How do you bring existing resources under Terraform management?**
   > Use `terraform import <resource_address> <resource_id>` to add the resource to state, then write the matching configuration.

---

[← Previous: 8.3 SAM](03-sam.md) | [Next: 9.1 CloudWatch Logs →](../09-Monitoring-and-Logging/01-cloudwatch-logs.md)

[← Back to README](../README.md)

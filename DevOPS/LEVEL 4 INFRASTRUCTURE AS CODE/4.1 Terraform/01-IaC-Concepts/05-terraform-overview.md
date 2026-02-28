# Chapter 5: Terraform Overview

[← Previous: IaC Tools Comparison](04-iac-tools-comparison.md) | [Back to README](../README.md) | [Next: Installation and Setup →](../02-Terraform-Basics/01-installation-and-setup.md)

---

## Overview

HashiCorp Terraform is the **most widely adopted** Infrastructure as Code tool. This chapter covers its architecture, core concepts, workflow, and ecosystem to give you a solid foundation before diving into hands-on usage.

---

## 5.1 What is Terraform?

Terraform is an **open-source, declarative Infrastructure as Code tool** that lets you define cloud and on-premises resources in human-readable configuration files (HCL). You can version, reuse, and share these configurations.

```
┌─────────────────────────────────────────────────────────────┐
│                    TERRAFORM AT A GLANCE                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Created by:   HashiCorp (2014)                             │
│  Written in:   Go                                           │
│  Config Lang:  HCL (HashiCorp Configuration Language)       │
│  License:      BSL 1.1 (v1.6+), MPL 2.0 (earlier)          │
│  Approach:     Declarative, State-based                     │
│  Providers:    3,000+ (AWS, Azure, GCP, K8s, GitHub...)     │
│  Registry:     registry.terraform.io                        │
│  Enterprise:   Terraform Cloud / Terraform Enterprise       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 5.2 Terraform Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                   TERRAFORM ARCHITECTURE                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ┌────────────────┐                                             │
│   │  .tf Config    │  You write HCL configuration files          │
│   │  Files         │                                             │
│   └───────┬────────┘                                             │
│           │                                                       │
│           ▼                                                       │
│   ┌────────────────┐                                             │
│   │  Terraform     │  The CLI tool processes your config         │
│   │  Core (CLI)    │                                             │
│   └───────┬────────┘                                             │
│           │                                                       │
│           ├──────────────┬──────────────┐                        │
│           ▼              ▼              ▼                         │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐                │
│   │  AWS       │  │  Azure     │  │  GCP       │  Providers     │
│   │  Provider  │  │  Provider  │  │  Provider  │  (plugins)     │
│   └─────┬──────┘  └─────┬──────┘  └─────┬──────┘                │
│         │               │               │                         │
│         ▼               ▼               ▼                         │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐                │
│   │  AWS APIs  │  │ Azure APIs │  │  GCP APIs  │  Cloud APIs    │
│   └────────────┘  └────────────┘  └────────────┘                │
│                                                                   │
│           ┌────────────────────┐                                  │
│           │   State File       │  Tracks current infrastructure  │
│           │  (terraform.tfstate)│  Maps config → real resources  │
│           └────────────────────┘                                  │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Terraform Core** | The CLI binary that parses HCL, builds dependency graph, manages state |
| **Providers** | Plugins that interface with cloud APIs (AWS, Azure, GCP, etc.) |
| **State File** | JSON file mapping your config to real-world resources |
| **HCL Config** | `.tf` files where you define your infrastructure |
| **Registry** | Public repository of modules and providers |

---

## 5.3 How Terraform Works

```
  ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐
  │ INIT │────▶│ PLAN │────▶│APPLY │────▶│  USE │
  └──────┘     └──────┘     └──────┘     └──────┘
     │            │            │            │
     │            │            │            │
     ▼            ▼            ▼            ▼
  Download     Compare      Execute     Infrastructure
  providers    desired vs   changes     is running!
  + modules    current      against
               state →      cloud APIs
               show diff
```

### The Terraform Workflow in Detail

```
  ┌─────────────────────────────────────────────────────────────┐
  │                TERRAFORM WORKFLOW                           │
  ├─────────────────────────────────────────────────────────────┤
  │                                                              │
  │  1. WRITE (.tf files)                                       │
  │     ┌─────────────────────────────────────────┐             │
  │     │ resource "aws_instance" "web" {          │             │
  │     │   ami           = "ami-abc123"           │             │
  │     │   instance_type = "t3.micro"             │             │
  │     │ }                                        │             │
  │     └─────────────────────────────────────────┘             │
  │                       │                                      │
  │  2. INIT (terraform init)                                   │
  │     • Downloads AWS provider plugin                         │
  │     • Initializes backend (state storage)                   │
  │     • Downloads referenced modules                          │
  │                       │                                      │
  │  3. PLAN (terraform plan)                                   │
  │     • Reads current state                                   │
  │     • Compares with desired config                          │
  │     • Shows: "Will create 1 EC2 instance"                   │
  │                       │                                      │
  │  4. APPLY (terraform apply)                                 │
  │     • Calls AWS API to create instance                      │
  │     • Updates state file with resource ID                   │
  │     • Reports: "Apply complete! 1 added"                    │
  │                       │                                      │
  │  5. DESTROY (terraform destroy) — when done                 │
  │     • Reads state to find all managed resources             │
  │     • Deletes them via APIs                                 │
  │     • Clears state file                                     │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘
```

---

## 5.4 Terraform File Structure

```
  my-project/
  ├── main.tf              # Primary resource definitions
  ├── variables.tf         # Input variable declarations
  ├── outputs.tf           # Output value declarations
  ├── providers.tf         # Provider configurations
  ├── terraform.tfvars     # Variable values (not in git)
  ├── versions.tf          # Required provider versions
  │
  ├── modules/             # Reusable modules
  │   ├── vpc/
  │   │   ├── main.tf
  │   │   ├── variables.tf
  │   │   └── outputs.tf
  │   └── ec2/
  │       ├── main.tf
  │       ├── variables.tf
  │       └── outputs.tf
  │
  ├── .terraform/          # Downloaded providers (auto-generated)
  ├── .terraform.lock.hcl  # Provider version lock file
  └── terraform.tfstate    # State file (auto-generated)
```

### File Naming Convention

| File | Purpose |
|------|---------|
| `main.tf` | Primary resource definitions |
| `variables.tf` | Input variable declarations |
| `outputs.tf` | Output values to expose |
| `providers.tf` | Provider configuration and versions |
| `terraform.tfvars` | Actual variable values (keep out of version control) |
| `locals.tf` | Local computed values |
| `data.tf` | Data source definitions |
| `versions.tf` | Required Terraform and provider versions |

---

## 5.5 Core Concepts

### Resources — The Building Blocks

```hcl
# A resource is a single piece of infrastructure
resource "aws_instance" "web" {
  #       ▲              ▲
  #       │              └── Local name (your label)
  #       └── Resource type (provider_resource)

  ami           = "ami-0c55b159cbfafe1f0"   # Arguments
  instance_type = "t3.micro"
  
  tags = {
    Name = "web-server"
  }
}

# Reference this resource elsewhere:
# aws_instance.web.id
# aws_instance.web.public_ip
```

### Providers — Cloud Connectors

```hcl
# Providers connect Terraform to cloud APIs
provider "aws" {
  region = "us-east-1"
}

provider "azurerm" {
  features {}
}

provider "google" {
  project = "my-project"
  region  = "us-central1"
}
```

### State — The Source of Truth

```
  ┌─────────────────────────────────────────────────────┐
  │                 STATE FILE ROLE                     │
  ├─────────────────────────────────────────────────────┤
  │                                                      │
  │   Config (.tf):     "I want a t3.micro instance"    │
  │   State (.tfstate): "Instance i-abc123 exists"      │
  │   Real World:        EC2 instance running on AWS    │
  │                                                      │
  │   terraform plan compares:                          │
  │                                                      │
  │   Config ──▶ "I want X"                             │
  │                  ↕  compare                         │
  │   State  ──▶ "X already exists as i-abc123"         │
  │                  ↕  verify                          │
  │   AWS API ──▶ "i-abc123 is running"                 │
  │                                                      │
  │   Result: No changes needed ✓                       │
  │                                                      │
  └─────────────────────────────────────────────────────┘
```

---

## 5.6 Dependency Graph

Terraform automatically determines the order to create/destroy resources:

```
  ┌────────────────────────────────────────────────────────┐
  │             DEPENDENCY GRAPH (auto-resolved)          │
  ├────────────────────────────────────────────────────────┤
  │                                                         │
  │              ┌──────────┐                              │
  │              │   VPC    │  Created FIRST               │
  │              └────┬─────┘                              │
  │                   │                                     │
  │            ┌──────┴──────┐                             │
  │            ▼             ▼                              │
  │      ┌──────────┐  ┌──────────┐                       │
  │      │ Subnet A │  │ Subnet B │  Created SECOND       │
  │      └────┬─────┘  └────┬─────┘                       │
  │           │              │                              │
  │           ▼              ▼                              │
  │      ┌──────────┐  ┌──────────┐                       │
  │      │ EC2 in A │  │ EC2 in B │  Created THIRD        │
  │      └──────────┘  └──────────┘                       │
  │                                                         │
  │   Destroy order: REVERSE (EC2 → Subnets → VPC)        │
  │                                                         │
  └────────────────────────────────────────────────────────┘
```

```hcl
# Terraform resolves dependencies from references
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "a" {
  vpc_id     = aws_vpc.main.id        # ← depends on VPC
  cidr_block = "10.0.1.0/24"
}

resource "aws_instance" "web" {
  subnet_id = aws_subnet.a.id          # ← depends on Subnet
  ami       = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}
```

---

## 5.7 Terraform Ecosystem

```
┌──────────────────────────────────────────────────────────────────┐
│                   TERRAFORM ECOSYSTEM                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ┌────────────────┐   ┌─────────────────┐   ┌───────────────┐  │
│   │  Terraform CLI │   │ Terraform Cloud │   │  TF Enterprise│  │
│   │  (Free, local) │   │ (SaaS, teams)   │   │  (Self-hosted)│  │
│   └────────────────┘   └─────────────────┘   └───────────────┘  │
│                                                                   │
│   ┌────────────────┐   ┌─────────────────┐   ┌───────────────┐  │
│   │  Registry      │   │   Sentinel      │   │   Vault       │  │
│   │  (Modules +    │   │  (Policy as     │   │  (Secrets     │  │
│   │   Providers)   │   │   Code)         │   │   mgmt)       │  │
│   └────────────────┘   └─────────────────┘   └───────────────┘  │
│                                                                   │
│   ┌────────────────┐   ┌─────────────────┐   ┌───────────────┐  │
│   │  Terragrunt    │   │   Checkov       │   │   tflint      │  │
│   │  (Wrapper tool)│   │  (Security scan)│   │  (Linter)     │  │
│   └────────────────┘   └─────────────────┘   └───────────────┘  │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5.8 First Terraform Configuration

Here's a complete, minimal example:

```hcl
# versions.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# providers.tf
provider "aws" {
  region = "us-east-1"
}

# main.tf
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket-name-12345"

  tags = {
    Name        = "My First Terraform Bucket"
    Environment = "learning"
  }
}

# outputs.tf
output "bucket_name" {
  value = aws_s3_bucket.my_bucket.id
}
```

### Running It

```bash
# Step 1: Initialize
terraform init
# Downloading provider plugin: hashicorp/aws v5.x...

# Step 2: Preview
terraform plan
# Plan: 1 to add, 0 to change, 0 to destroy.

# Step 3: Apply
terraform apply
# Apply complete! Resources: 1 added.
# Outputs: bucket_name = "my-unique-bucket-name-12345"

# Step 4: Destroy (cleanup)
terraform destroy
# Destroy complete! Resources: 1 destroyed.
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `terraform init` fails | No internet or wrong provider source | Check connectivity, verify `required_providers` block |
| Provider authentication errors | Credentials not configured | Set `AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY` or use `aws configure` |
| State file conflicts | Multiple people running apply | Use remote state with locking (S3 + DynamoDB) |
| "Resource already exists" | Resource was created outside Terraform | Use `terraform import` to bring it under management |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Terraform** | HashiCorp's declarative IaC tool for multi-cloud provisioning |
| **HCL** | HashiCorp Configuration Language — Terraform's DSL |
| **Provider** | Plugin that interfaces with a cloud API (AWS, Azure, GCP, etc.) |
| **Resource** | A single piece of infrastructure (EC2 instance, S3 bucket, etc.) |
| **State File** | JSON file that maps config to real-world resource IDs |
| **Dependency Graph** | Auto-resolved order of resource creation/destruction |
| **terraform init** | Downloads providers and initializes the working directory |
| **terraform plan** | Previews changes without applying them |
| **terraform apply** | Executes changes to create/modify/destroy resources |
| **terraform destroy** | Removes all resources managed by the configuration |
| **Registry** | Public repository of reusable modules and providers |

---

## Quick Revision Questions

1. **What are the three core components of Terraform's architecture?**
   > Terraform Core (CLI), Providers (plugins), and State File.

2. **What language does Terraform use for configuration, and what does it stand for?**
   > HCL — HashiCorp Configuration Language. It's a declarative DSL designed for infrastructure.

3. **Explain the purpose of the `terraform plan` command.**
   > It compares the desired state (config) with the current state (state file + real infrastructure) and shows what changes will be made without executing them.

4. **How does Terraform determine the order to create resources?**
   > Through automatic dependency analysis — when one resource references another (e.g., `aws_vpc.main.id`), Terraform builds a dependency graph and creates resources in the correct order.

5. **What is the state file and why is it important?**
   > The state file (`terraform.tfstate`) is a JSON file that maps your configuration to real-world resource IDs. It's essential for Terraform to know what exists and what needs to change.

6. **List the four main Terraform commands in their typical workflow order.**
   > `terraform init` → `terraform plan` → `terraform apply` → `terraform destroy`.

---

[← Previous: IaC Tools Comparison](04-iac-tools-comparison.md) | [Back to README](../README.md) | [Next: Installation and Setup →](../02-Terraform-Basics/01-installation-and-setup.md)

# Chapter 4: Provider Aliases

[← Previous: Multi-Provider](03-multi-provider.md) | [Back to README](../README.md) | [Next: Custom Providers →](05-custom-providers.md)

---

## Overview

**Provider aliases** allow you to create multiple configurations of the same provider. This is essential for multi-region deployments, cross-account access, and resources that need different provider settings within the same configuration.

---

## 4.1 Why Aliases?

```
┌──────────────────────────────────────────────────────────┐
│          PROVIDER ALIAS USE CASES                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Multi-Region                                             │
│  ├── Primary infrastructure in us-east-1                 │
│  ├── DR infrastructure in us-west-2                      │
│  └── Global resources (CloudFront, IAM, Route53)         │
│                                                           │
│  Multi-Account                                            │
│  ├── Shared services account                             │
│  ├── Application account                                  │
│  └── Security/audit account                              │
│                                                           │
│  Different Credentials                                    │
│  ├── Admin role for security resources                   │
│  ├── Limited role for app resources                      │
│  └── Read-only role for data sources                     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 Alias Syntax

```hcl
# ─── Default provider (no alias) ─────────────────
provider "aws" {
  region = "us-east-1"
}

# ─── Aliased provider ────────────────────────────
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

provider "aws" {
  alias  = "eu"
  region = "eu-west-1"
}

# ─── Using aliased providers in resources ─────────
resource "aws_instance" "primary" {
  # Uses default provider (us-east-1)
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

resource "aws_instance" "dr" {
  # Explicitly uses the "west" alias (us-west-2)
  provider      = aws.west
  ami           = "ami-0892d3c7ee96c0bf7"
  instance_type = "t3.micro"
}

resource "aws_instance" "eu_app" {
  # Uses the "eu" alias (eu-west-1)
  provider      = aws.eu
  ami           = "ami-0d75513e7706cf2d9"
  instance_type = "t3.micro"
}
```

```
┌──────────────────────────────────────────────────────────┐
│         ALIAS PROVIDER SELECTION                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  provider "aws" { }           ← DEFAULT (no alias)       │
│  provider "aws" { alias="w" } ← NAMED alias              │
│                                                           │
│  resource "aws_instance" "a" { }                         │
│    └── Uses DEFAULT provider automatically               │
│                                                           │
│  resource "aws_instance" "b" {                           │
│    provider = aws.w                                      │
│  }                                                       │
│    └── Uses NAMED alias explicitly                       │
│                                                           │
│  RULE: Resources without provider = ... use default      │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.3 Multi-Region Deployment

```hcl
# ─── Provider configurations ─────────────────────
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "dr"
  region = "us-west-2"
}

# ─── Primary region: VPC + App ───────────────────
resource "aws_vpc" "primary" {
  cidr_block = "10.0.0.0/16"
  tags       = { Name = "primary-vpc" }
}

resource "aws_instance" "primary" {
  ami           = data.aws_ami.primary.id
  instance_type = "m5.large"
  subnet_id     = aws_subnet.primary.id
  tags          = { Name = "primary-app" }
}

# ─── DR region: VPC + App ────────────────────────
resource "aws_vpc" "dr" {
  provider   = aws.dr
  cidr_block = "10.1.0.0/16"
  tags       = { Name = "dr-vpc" }
}

resource "aws_instance" "dr" {
  provider      = aws.dr
  ami           = data.aws_ami.dr.id
  instance_type = "m5.large"
  subnet_id     = aws_subnet.dr.id
  tags          = { Name = "dr-app" }
}

# ─── Data sources also need provider ─────────────
data "aws_ami" "primary" {
  most_recent = true
  owners      = ["099720109477"]  # Canonical
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

data "aws_ami" "dr" {
  provider    = aws.dr
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}

# ─── Cross-region VPC peering ────────────────────
resource "aws_vpc_peering_connection" "primary_to_dr" {
  vpc_id      = aws_vpc.primary.id
  peer_vpc_id = aws_vpc.dr.id
  peer_region = "us-west-2"
  auto_accept = false
}

resource "aws_vpc_peering_connection_accepter" "dr_accept" {
  provider                  = aws.dr
  vpc_peering_connection_id = aws_vpc_peering_connection.primary_to_dr.id
  auto_accept               = true
}
```

---

## 4.4 Multi-Account Deployment

```hcl
# ─── Provider: Management account ────────────────
provider "aws" {
  region = "us-east-1"
  # Uses default credentials (management account)
}

# ─── Provider: Production account ────────────────
provider "aws" {
  alias  = "production"
  region = "us-east-1"
  assume_role {
    role_arn = "arn:aws:iam::111111111111:role/TerraformRole"
  }
}

# ─── Provider: Staging account ───────────────────
provider "aws" {
  alias  = "staging"
  region = "us-east-1"
  assume_role {
    role_arn = "arn:aws:iam::222222222222:role/TerraformRole"
  }
}

# ─── Provider: Security account ──────────────────
provider "aws" {
  alias  = "security"
  region = "us-east-1"
  assume_role {
    role_arn = "arn:aws:iam::333333333333:role/SecurityRole"
  }
}

# ─── Resources in different accounts ─────────────
resource "aws_s3_bucket" "prod_data" {
  provider = aws.production
  bucket   = "prod-app-data"
}

resource "aws_s3_bucket" "staging_data" {
  provider = aws.staging
  bucket   = "staging-app-data"
}

resource "aws_cloudtrail" "audit" {
  provider       = aws.security
  name           = "org-audit-trail"
  s3_bucket_name = aws_s3_bucket.audit_logs.bucket
}
```

```
┌──────────────────────────────────────────────────────────┐
│       MULTI-ACCOUNT ARCHITECTURE                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Management Account (default provider)                    │
│  ├── Organizations config                                │
│  └── IAM roles for cross-account                         │
│                                                           │
│       ┌──────────────┬──────────────┬──────────────┐     │
│       │  assume_role  │  assume_role  │  assume_role  │    │
│       ▼              ▼              ▼              │     │
│  ┌─────────┐   ┌──────────┐  ┌──────────┐              │
│  │ prod    │   │ staging  │  │ security │              │
│  │ (aws.   │   │ (aws.    │  │ (aws.    │              │
│  │ produc- │   │ staging) │  │ security)│              │
│  │ tion)   │   │          │  │          │              │
│  └─────────┘   └──────────┘  └──────────┘              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.5 Passing Aliases to Modules

```hcl
# ─── Root module: pass providers to child module ──
provider "aws" {
  region = "us-east-1"
}

provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

module "multi_region_app" {
  source = "./modules/app"

  providers = {
    aws          = aws          # Default → module's default
    aws.secondary = aws.west    # Named → module's named
  }

  name = "myapp"
}

# ─── Child module: modules/app/main.tf ────────────
# Declare required providers in the module
terraform {
  required_providers {
    aws = {
      source                = "hashicorp/aws"
      version               = "~> 5.0"
      configuration_aliases = [aws.secondary]
    }
  }
}

# Uses default aws provider
resource "aws_instance" "primary" {
  ami           = var.primary_ami
  instance_type = "t3.micro"
}

# Uses the secondary alias
resource "aws_instance" "dr" {
  provider      = aws.secondary
  ami           = var.secondary_ami
  instance_type = "t3.micro"
}
```

---

## 4.6 Global Resources Pattern (AWS)

```hcl
# Some AWS resources MUST be in us-east-1
provider "aws" {
  region = "eu-west-1"   # Main region
}

provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"   # For global resources
}

# CloudFront requires ACM cert in us-east-1
resource "aws_acm_certificate" "cdn_cert" {
  provider          = aws.us_east_1
  domain_name       = "example.com"
  validation_method = "DNS"
}

# CloudFront distribution (global, but cert from us-east-1)
resource "aws_cloudfront_distribution" "cdn" {
  # Uses default provider
  origin {
    domain_name = aws_s3_bucket.origin.bucket_regional_domain_name
    origin_id   = "s3-origin"
  }

  viewer_certificate {
    acm_certificate_arn = aws_acm_certificate.cdn_cert.arn
    ssl_support_method  = "sni-only"
  }
  # ...
}

# WAF v2 WebACL for CloudFront also needs us-east-1
resource "aws_wafv2_web_acl" "cdn_waf" {
  provider = aws.us_east_1
  name     = "cdn-waf"
  scope    = "CLOUDFRONT"
  # ...
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `provider not found: aws.west` | Alias not declared | Add `provider "aws" { alias = "west" ... }` |
| Module doesn't accept alias | Missing `configuration_aliases` | Add to module's `required_providers` |
| Wrong region for resource | Missing `provider = aws.alias` | Explicitly set `provider` on every aliased resource |
| `AssumeRole` permission denied | Trust policy doesn't allow | Update IAM trust policy in target account |
| AMI not found in region | AMI IDs are region-specific | Use data source with provider alias to find AMI |

---

## Summary Table

| Concept | Syntax | Purpose |
|---------|--------|---------|
| **Default provider** | `provider "aws" { }` | Used when no `provider` attr specified |
| **Aliased provider** | `provider "aws" { alias = "x" }` | Additional config of same provider |
| **Resource binding** | `provider = aws.x` | Assign aliased provider to resource |
| **Module providers** | `providers = { aws.y = aws.x }` | Pass aliases to child modules |
| **Config aliases** | `configuration_aliases = [aws.y]` | Declare expected aliases in module |

---

## Quick Revision Questions

1. **What is a provider alias?**
   > A named, additional configuration of the same provider type. It allows multiple instances of a provider with different settings (region, account, credentials).

2. **How do you assign an aliased provider to a resource?**
   > Add `provider = aws.alias_name` to the resource block. Without this, the resource uses the default (non-aliased) provider.

3. **How do you pass provider aliases to modules?**
   > Use the `providers` map in the module block: `providers = { aws.secondary = aws.west }`. The module must declare `configuration_aliases` in its `required_providers`.

4. **Why would you use provider aliases for AWS?**
   > Multi-region deployments, cross-account access via assume_role, and global resources that require `us-east-1` (ACM for CloudFront, WAF for CloudFront).

5. **What happens if you forget to add `provider = aws.alias` to a resource?**
   > The resource uses the default (non-aliased) provider, which may create it in the wrong region or account.

---

[← Previous: Multi-Provider](03-multi-provider.md) | [Back to README](../README.md) | [Next: Custom Providers →](05-custom-providers.md)

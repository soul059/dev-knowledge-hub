# Chapter 2: Multi-Region Deployment

[← Previous: Multi-Account Architecture](01-multi-account-architecture.md) | [Back to README](../README.md) | [Next: Disaster Recovery →](03-disaster-recovery.md)

---

## Overview

Multi-region deployments provide **low-latency access** for global users, **disaster recovery**, and **high availability**. This chapter covers Terraform patterns for deploying and managing infrastructure across multiple cloud regions.

---

## 2.1 Multi-Region Architecture

```
┌──────────────────────────────────────────────────────────┐
│           MULTI-REGION ARCHITECTURE                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌───────────────┐         ┌───────────────┐             │
│  │  us-east-1    │         │  eu-west-1    │             │
│  │  (primary)    │         │  (secondary)  │             │
│  │               │         │               │             │
│  │ ┌───────────┐ │         │ ┌───────────┐ │             │
│  │ │    ALB    │ │         │ │    ALB    │ │             │
│  │ └─────┬─────┘ │         │ └─────┬─────┘ │             │
│  │ ┌─────┴─────┐ │         │ ┌─────┴─────┐ │             │
│  │ │  ECS/EKS  │ │         │ │  ECS/EKS  │ │             │
│  │ └─────┬─────┘ │         │ └─────┬─────┘ │             │
│  │ ┌─────┴─────┐ │  Repl.  │ ┌─────┴─────┐ │             │
│  │ │  RDS      │◄├────────►├─│  RDS      │ │             │
│  │ │  Primary  │ │         │ │  Read     │ │             │
│  │ └───────────┘ │         │ └───────────┘ │             │
│  └───────┬───────┘         └───────┬───────┘             │
│          │                         │                      │
│          └──────────┬──────────────┘                      │
│                     │                                     │
│              ┌──────┴──────┐                              │
│              │  Route 53   │  (latency / failover)        │
│              │  DNS        │                              │
│              └─────────────┘                              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 Provider Configuration for Multi-Region

```hcl
# ─── Provider aliases per region ─────────────────
provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"
}

provider "aws" {
  alias  = "eu_west_1"
  region = "eu-west-1"
}

provider "aws" {
  alias  = "ap_southeast_1"
  region = "ap-southeast-1"
}

# ─── Module per region ──────────────────────────
module "us_east" {
  source = "./modules/regional-stack"
  providers = {
    aws = aws.us_east_1
  }

  region      = "us-east-1"
  environment = var.environment
  is_primary  = true
  vpc_cidr    = "10.1.0.0/16"
}

module "eu_west" {
  source = "./modules/regional-stack"
  providers = {
    aws = aws.eu_west_1
  }

  region      = "eu-west-1"
  environment = var.environment
  is_primary  = false
  vpc_cidr    = "10.2.0.0/16"
}
```

---

## 2.3 Dynamic Multi-Region with for_each

```hcl
# ─── Region configuration map ───────────────────
variable "regions" {
  type = map(object({
    vpc_cidr    = string
    is_primary  = bool
    az_count    = number
  }))

  default = {
    "us-east-1" = {
      vpc_cidr   = "10.1.0.0/16"
      is_primary = true
      az_count   = 3
    }
    "eu-west-1" = {
      vpc_cidr   = "10.2.0.0/16"
      is_primary = false
      az_count   = 2
    }
    "ap-southeast-1" = {
      vpc_cidr   = "10.3.0.0/16"
      is_primary = false
      az_count   = 2
    }
  }
}

# ─── Dynamic providers not supported in for_each ─
# Must use separate module calls per region
# OR use Terragrunt for dynamic provider generation

# ─── Global resources (us-east-1 only) ──────────
resource "aws_acm_certificate" "global" {
  provider          = aws.us_east_1   # CloudFront requires us-east-1
  domain_name       = var.domain_name
  validation_method = "DNS"

  subject_alternative_names = ["*.${var.domain_name}"]

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_wafv2_web_acl" "global" {
  provider = aws.us_east_1   # CloudFront WAF must be in us-east-1
  name     = "${var.project}-global-waf"
  scope    = "CLOUDFRONT"
  # ...
}
```

---

## 2.4 Cross-Region Networking

```hcl
# ─── VPC Peering across regions ──────────────────
resource "aws_vpc_peering_connection" "us_to_eu" {
  provider    = aws.us_east_1
  vpc_id      = module.us_east.vpc_id
  peer_vpc_id = module.eu_west.vpc_id
  peer_region = "eu-west-1"

  tags = { Name = "us-east-1-to-eu-west-1" }
}

resource "aws_vpc_peering_connection_accepter" "eu_accept" {
  provider                  = aws.eu_west_1
  vpc_peering_connection_id = aws_vpc_peering_connection.us_to_eu.id
  auto_accept               = true
}

# ─── Transit Gateway (hub-and-spoke) ────────────
resource "aws_ec2_transit_gateway" "main" {
  provider    = aws.us_east_1
  description = "Multi-region transit gateway"

  default_route_table_association = "disable"
  default_route_table_propagation = "disable"

  tags = { Name = "${var.project}-tgw" }
}

# Peer transit gateways across regions
resource "aws_ec2_transit_gateway_peering_attachment" "us_eu" {
  provider                = aws.us_east_1
  peer_region             = "eu-west-1"
  transit_gateway_id      = aws_ec2_transit_gateway.main.id
  peer_transit_gateway_id = aws_ec2_transit_gateway.eu.id

  tags = { Name = "tgw-peer-us-eu" }
}
```

---

## 2.5 Cross-Region Data Replication

```hcl
# ─── RDS Cross-Region Read Replica ──────────────
resource "aws_db_instance" "primary" {
  provider             = aws.us_east_1
  identifier           = "${var.project}-primary"
  engine               = "postgres"
  engine_version       = "15.4"
  instance_class       = "db.r6g.xlarge"
  allocated_storage    = 100
  backup_retention_period = 7

  tags = { Name = "${var.project}-db-primary" }
}

resource "aws_db_instance" "replica" {
  provider             = aws.eu_west_1
  identifier           = "${var.project}-replica-eu"
  replicate_source_db  = aws_db_instance.primary.arn
  instance_class       = "db.r6g.xlarge"
  # Storage inherited from primary

  tags = { Name = "${var.project}-db-replica-eu" }
}

# ─── S3 Cross-Region Replication ────────────────
resource "aws_s3_bucket" "primary" {
  provider = aws.us_east_1
  bucket   = "${var.project}-primary-${var.environment}"
}

resource "aws_s3_bucket_versioning" "primary" {
  provider = aws.us_east_1
  bucket   = aws_s3_bucket.primary.id
  versioning_configuration { status = "Enabled" }
}

resource "aws_s3_bucket_replication_configuration" "primary" {
  provider = aws.us_east_1
  bucket   = aws_s3_bucket.primary.id
  role     = aws_iam_role.replication.arn

  rule {
    id     = "replicate-all"
    status = "Enabled"

    destination {
      bucket        = aws_s3_bucket.replica.arn
      storage_class = "STANDARD_IA"
    }
  }

  depends_on = [aws_s3_bucket_versioning.primary]
}

resource "aws_s3_bucket" "replica" {
  provider = aws.eu_west_1
  bucket   = "${var.project}-replica-${var.environment}"
}
```

---

## 2.6 DNS-Based Routing

```hcl
# ─── Route 53: Latency-based routing ────────────
resource "aws_route53_record" "app_us" {
  zone_id = data.aws_route53_zone.main.zone_id
  name    = "app.${var.domain_name}"
  type    = "A"

  set_identifier = "us-east-1"
  
  latency_routing_policy {
    region = "us-east-1"
  }

  alias {
    name                   = module.us_east.alb_dns_name
    zone_id                = module.us_east.alb_zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "app_eu" {
  zone_id = data.aws_route53_zone.main.zone_id
  name    = "app.${var.domain_name}"
  type    = "A"

  set_identifier = "eu-west-1"

  latency_routing_policy {
    region = "eu-west-1"
  }

  alias {
    name                   = module.eu_west.alb_dns_name
    zone_id                = module.eu_west.alb_zone_id
    evaluate_target_health = true
  }
}

# ─── Route 53: Failover routing ────────────────
resource "aws_route53_health_check" "primary" {
  fqdn              = module.us_east.alb_dns_name
  port               = 443
  type               = "HTTPS"
  resource_path      = "/health"
  failure_threshold  = 3
  request_interval   = 10
}

resource "aws_route53_record" "failover_primary" {
  zone_id = data.aws_route53_zone.main.zone_id
  name    = "app.${var.domain_name}"
  type    = "A"

  set_identifier = "primary"

  failover_routing_policy {
    type = "PRIMARY"
  }

  health_check_id = aws_route53_health_check.primary.id

  alias {
    name                   = module.us_east.alb_dns_name
    zone_id                = module.us_east.alb_zone_id
    evaluate_target_health = true
  }
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| CloudFront cert must be us-east-1 | AWS requirement | Use `provider = aws.us_east_1` for ACM certs |
| Cross-region peering pending | Accepter not configured | Add `vpc_peering_connection_accepter` resource |
| Replica lag too high | Insufficient instance size | Scale up replica instance class |
| DNS not routing correctly | Health checks failing | Verify health check endpoint and thresholds |
| CIDR overlap between regions | Poor planning | Use non-overlapping CIDRs (10.1.0.0/16, 10.2.0.0/16) |

---

## Summary Table

| Pattern | Use Case | DNS Routing | Data Sync |
|---------|----------|------------|-----------|
| **Active-Active** | Global users, HA | Latency-based | Bi-directional replication |
| **Active-Passive** | DR, compliance | Failover | Primary → replica |
| **Read-local** | Read-heavy apps | Latency-based | Read replicas |
| **Write-primary** | Write consistency | Failover to primary | Single write region |

---

## Quick Revision Questions

1. **How do you configure Terraform for multiple AWS regions?**
   > Use provider aliases with different `region` values, then pass specific providers to modules or resources using the `providers` argument.

2. **What DNS routing policy sends users to the closest region?**
   > Latency-based routing in Route 53 — it routes queries to the region with the lowest latency for the user.

3. **Why must CloudFront ACM certificates be in us-east-1?**
   > AWS requires that ACM certificates used with CloudFront distributions must be created in the us-east-1 (N. Virginia) region.

4. **How do you replicate RDS across regions?**
   > Create a cross-region read replica using `replicate_source_db` pointing to the primary database's ARN, deployed via a provider in the target region.

5. **What is the difference between active-active and active-passive multi-region?**
   > Active-active serves traffic from all regions simultaneously (latency routing). Active-passive keeps one region on standby, only routing traffic there during failover.

---

[← Previous: Multi-Account Architecture](01-multi-account-architecture.md) | [Back to README](../README.md) | [Next: Disaster Recovery →](03-disaster-recovery.md)

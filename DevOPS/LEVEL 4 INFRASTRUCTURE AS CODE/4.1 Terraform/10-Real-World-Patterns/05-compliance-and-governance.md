# Chapter 5: Compliance and Governance

[← Previous: Cost Optimization](04-cost-optimization.md) | [Back to README](../README.md) | [Next: Migration Strategies →](06-migration-strategies.md)

---

## Overview

Compliance and governance ensure infrastructure meets **regulatory requirements**, **organizational policies**, and **security standards**. Terraform enables automated compliance through policy-as-code, tagging enforcement, and audit trails.

---

## 5.1 Governance Framework

```
┌──────────────────────────────────────────────────────────┐
│       TERRAFORM GOVERNANCE FRAMEWORK                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌───────────────────────────────────────────────┐       │
│  │ Layer 1: Preventive Controls                   │       │
│  │ Stop non-compliant resources from being created│       │
│  │ → SCPs, Sentinel, OPA, variable validation     │       │
│  └───────────────────────────────────────────────┘       │
│                                                           │
│  ┌───────────────────────────────────────────────┐       │
│  │ Layer 2: Detective Controls                    │       │
│  │ Detect drift and policy violations             │       │
│  │ → AWS Config, CloudTrail, drift detection      │       │
│  └───────────────────────────────────────────────┘       │
│                                                           │
│  ┌───────────────────────────────────────────────┐       │
│  │ Layer 3: Corrective Controls                   │       │
│  │ Auto-remediate non-compliant resources         │       │
│  │ → Config Rules auto-remediation, re-apply      │       │
│  └───────────────────────────────────────────────┘       │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Tagging Enforcement

```hcl
# ─── Required tags via provider default_tags ─────
provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = var.project
      Environment = var.environment
      Team        = var.team
      ManagedBy   = "terraform"
      CostCenter  = var.cost_center
      Compliance  = var.compliance_framework  # e.g., "hipaa", "pci", "sox"
    }
  }
}

# ─── Tag validation via variable ────────────────
variable "required_tags" {
  type = map(string)

  validation {
    condition = alltrue([
      for key in ["Project", "Environment", "Team", "CostCenter"] :
      contains(keys(var.required_tags), key)
    ])
    error_message = "Missing required tags: Project, Environment, Team, CostCenter."
  }
}

# ─── AWS Organizations Tag Policy ───────────────
resource "aws_organizations_policy" "tagging" {
  name    = "required-tags"
  type    = "TAG_POLICY"
  content = jsonencode({
    tags = {
      Environment = {
        tag_key = {
          "@@assign" = "Environment"
        }
        tag_value = {
          "@@assign" = ["dev", "staging", "prod"]
        }
        enforced_for = {
          "@@assign" = [
            "ec2:instance",
            "ec2:volume",
            "rds:db",
            "s3:bucket"
          ]
        }
      }
    }
  })
}
```

---

## 5.3 AWS Config Rules

```hcl
# ─── AWS Config for compliance monitoring ────────
resource "aws_config_configuration_recorder" "main" {
  name     = "config-recorder"
  role_arn = aws_iam_role.config.arn

  recording_group {
    all_supported                 = true
    include_global_resource_types = true
  }
}

resource "aws_config_delivery_channel" "main" {
  name           = "config-delivery"
  s3_bucket_name = aws_s3_bucket.config.id
  depends_on     = [aws_config_configuration_recorder.main]
}

# ─── Managed Config Rules ──────────────────────
resource "aws_config_config_rule" "s3_encryption" {
  name = "s3-bucket-server-side-encryption-enabled"

  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"
  }

  depends_on = [aws_config_configuration_recorder.main]
}

resource "aws_config_config_rule" "encrypted_volumes" {
  name = "encrypted-volumes"

  source {
    owner             = "AWS"
    source_identifier = "ENCRYPTED_VOLUMES"
  }
}

resource "aws_config_config_rule" "rds_multi_az" {
  name = "rds-multi-az-support"

  source {
    owner             = "AWS"
    source_identifier = "RDS_MULTI_AZ_SUPPORT"
  }
}

resource "aws_config_config_rule" "required_tags" {
  name = "required-tags"

  source {
    owner             = "AWS"
    source_identifier = "REQUIRED_TAGS"
  }

  input_parameters = jsonencode({
    tag1Key   = "Environment"
    tag2Key   = "Project"
    tag3Key   = "ManagedBy"
  })
}

# ─── Auto-remediation ──────────────────────────
resource "aws_config_remediation_configuration" "s3_encryption" {
  config_rule_name = aws_config_config_rule.s3_encryption.name
  target_type      = "SSM_DOCUMENT"
  target_id        = "AWS-EnableS3BucketEncryption"

  parameter {
    name           = "BucketName"
    resource_value = "RESOURCE_ID"
  }

  parameter {
    name         = "SSEAlgorithm"
    static_value = "aws:kms"
  }

  automatic                  = true
  maximum_automatic_attempts = 3
  retry_attempt_seconds      = 60
}
```

---

## 5.4 Service Control Policies (SCPs)

```hcl
# ─── Deny unencrypted resources ─────────────────
resource "aws_organizations_policy" "require_encryption" {
  name    = "require-encryption"
  type    = "SERVICE_CONTROL_POLICY"
  content = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid       = "DenyUnencryptedS3"
        Effect    = "Deny"
        Action    = "s3:PutObject"
        Resource  = "*"
        Condition = {
          StringNotEquals = {
            "s3:x-amz-server-side-encryption" = ["aws:kms", "AES256"]
          }
        }
      },
      {
        Sid       = "DenyUnencryptedEBS"
        Effect    = "Deny"
        Action    = "ec2:CreateVolume"
        Resource  = "*"
        Condition = {
          Bool = {
            "ec2:Encrypted" = "false"
          }
        }
      }
    ]
  })
}

# ─── Restrict regions ──────────────────────────
resource "aws_organizations_policy" "region_restriction" {
  name    = "allowed-regions"
  type    = "SERVICE_CONTROL_POLICY"
  content = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Sid       = "DenyNonApprovedRegions"
      Effect    = "Deny"
      Action    = "*"
      Resource  = "*"
      Condition = {
        StringNotEquals = {
          "aws:RequestedRegion" = var.allowed_regions
        }
        # Exclude global services
        "ForAnyValue:StringNotLike" = {
          "aws:PrincipalServiceName" = [
            "iam.amazonaws.com",
            "organizations.amazonaws.com",
            "cloudfront.amazonaws.com"
          ]
        }
      }
    }]
  })
}

resource "aws_organizations_policy_attachment" "region_workloads" {
  policy_id = aws_organizations_policy.region_restriction.id
  target_id = aws_organizations_organizational_unit.workloads.id
}
```

---

## 5.5 Compliance Frameworks in Terraform

```
┌──────────────────────────────────────────────────────────┐
│       COMPLIANCE MAPPING TO TERRAFORM                     │
├─────────────┬────────────────────────────────────────────┤
│ Framework   │ Terraform Implementation                    │
├─────────────┼────────────────────────────────────────────┤
│ HIPAA       │ • Encryption at rest (KMS)                  │
│             │ • Encryption in transit (TLS/SSL)            │
│             │ • Access logging (CloudTrail)                │
│             │ • VPC isolation (private subnets)            │
│             │ • Backup/retention policies                  │
│             │                                             │
│ PCI-DSS     │ • Network segmentation (SGs, NACLs)         │
│             │ • Encryption everywhere                      │
│             │ • WAF for web apps                           │
│             │ • Multi-factor auth (IAM)                    │
│             │ • Log retention (90+ days)                   │
│             │                                             │
│ SOC 2       │ • Change management (IaC + GitOps)          │
│             │ • Access controls (IAM least privilege)      │
│             │ • Monitoring and alerting                    │
│             │ • Incident response automation               │
│             │                                             │
│ GDPR        │ • Data residency (region restrictions)       │
│             │ • Encryption (KMS + CMK)                     │
│             │ • Data deletion capabilities                 │
│             │ • Access audit trails                        │
└─────────────┴────────────────────────────────────────────┘
```

```hcl
# ─── HIPAA-compliant RDS example ────────────────
resource "aws_db_instance" "hipaa" {
  identifier     = "${var.project}-hipaa-db"
  engine         = "postgres"
  instance_class = "db.r6g.xlarge"

  # Encryption at rest
  storage_encrypted = true
  kms_key_id        = aws_kms_key.database.arn

  # Network isolation
  db_subnet_group_name   = aws_db_subnet_group.private.name
  publicly_accessible    = false
  vpc_security_group_ids = [aws_security_group.database.id]

  # Backup and retention
  backup_retention_period   = 35
  delete_automated_backups  = false
  copy_tags_to_snapshot     = true
  deletion_protection       = true
  skip_final_snapshot       = false
  final_snapshot_identifier = "${var.project}-final-${formatdate("YYYY-MM-DD", timestamp())}"

  # Audit logging
  enabled_cloudwatch_logs_exports = ["postgresql", "upgrade"]

  # HA
  multi_az = true

  lifecycle {
    prevent_destroy = true
  }

  tags = merge(local.common_tags, {
    Compliance = "HIPAA"
    DataClass  = "PHI"
  })
}
```

---

## 5.6 Audit Trail

```hcl
# ─── CloudTrail for all API activity ────────────
resource "aws_cloudtrail" "org_trail" {
  name                       = "organization-trail"
  s3_bucket_name             = aws_s3_bucket.audit_logs.id
  is_organization_trail      = true
  is_multi_region_trail      = true
  enable_log_file_validation = true
  kms_key_id                 = aws_kms_key.audit.arn

  cloud_watch_logs_group_arn = "${aws_cloudwatch_log_group.trail.arn}:*"
  cloud_watch_logs_role_arn  = aws_iam_role.cloudtrail.arn

  event_selector {
    read_write_type           = "All"
    include_management_events = true

    data_resource {
      type   = "AWS::S3::Object"
      values = ["arn:aws:s3:::"]   # All S3 objects
    }
  }

  tags = {
    Compliance = "required"
    ManagedBy  = "terraform"
  }
}

# Log retention
resource "aws_cloudwatch_log_group" "trail" {
  name              = "/aws/cloudtrail/org"
  retention_in_days = 365   # 1 year for compliance
  kms_key_id        = aws_kms_key.audit.arn
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SCP blocks Terraform | Policy too restrictive | Add exceptions for Terraform role principal |
| Config rules non-compliant | Resources created before rules | Run remediation or re-apply with correct config |
| Tag enforcement breaks deploys | Missing required tags | Use `default_tags` in provider for baseline |
| Audit logs incomplete | CloudTrail not multi-region | Enable `is_multi_region_trail = true` |
| Compliance drift | Manual console changes | Enable drift detection, restrict console access |

---

## Summary Table

| Control Type | Tool | Enforcement | Scope |
|-------------|------|------------|-------|
| **Preventive** | SCPs | Block at API level | Organization/OU |
| **Preventive** | Sentinel/OPA | Block at plan time | Workspace/pipeline |
| **Preventive** | Variable validation | Block invalid input | Module/config |
| **Detective** | AWS Config | Monitor compliance | Account/Org |
| **Detective** | CloudTrail | Audit all actions | Account/Org |
| **Corrective** | Config Remediation | Auto-fix violations | Account |
| **Corrective** | Terraform re-apply | Restore desired state | Workspace |

---

## Quick Revision Questions

1. **What are the three types of governance controls?**
   > Preventive (stop violations before they happen — SCPs, Sentinel), detective (find violations after they occur — AWS Config, CloudTrail), and corrective (auto-fix violations — Config remediation, Terraform re-apply).

2. **How do you enforce required tags across all resources?**
   > Use `default_tags` in the provider block for Terraform-managed resources, AWS Organizations Tag Policies for organizational enforcement, and AWS Config rules for detection.

3. **What is a Service Control Policy (SCP)?**
   > An organization-level IAM guardrail that restricts maximum permissions for all principals in member accounts. Even account admins cannot exceed SCP boundaries.

4. **How does Terraform support compliance frameworks?**
   > By codifying compliance requirements as resource configurations (encryption, network isolation, logging), enforcing them with validation rules, and providing an auditable change history via Git.

5. **Why is CloudTrail important for governance?**
   > It records every API call made in the account, providing a complete audit trail of who did what and when — essential for compliance, security investigations, and change tracking.

---

[← Previous: Cost Optimization](04-cost-optimization.md) | [Back to README](../README.md) | [Next: Migration Strategies →](06-migration-strategies.md)

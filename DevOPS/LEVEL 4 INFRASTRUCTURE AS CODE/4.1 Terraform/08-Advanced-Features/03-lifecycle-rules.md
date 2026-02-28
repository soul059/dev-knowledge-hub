# Chapter 3: Lifecycle Rules

[← Previous: Terraform Functions](02-terraform-functions.md) | [Back to README](../README.md) | [Next: Moved and Removed Blocks →](04-moved-and-removed.md)

---

## Overview

**Lifecycle rules** customize how Terraform manages resource creation, updates, and deletion. They override defaults to handle scenarios like zero-downtime deployments, preventing accidental deletions, and ignoring external changes.

---

## 3.1 Lifecycle Meta-Argument

```hcl
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    # Available lifecycle arguments:
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags, ami]
    replace_triggered_by  = [terraform_data.trigger.output]

    # TF 1.5+:
    precondition { ... }
    postcondition { ... }
  }
}
```

```
┌──────────────────────────────────────────────────────────┐
│           LIFECYCLE RULES OVERVIEW                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  create_before_destroy                                    │
│  └── Create new resource BEFORE destroying old one       │
│                                                           │
│  prevent_destroy                                          │
│  └── Block any plan that would destroy this resource     │
│                                                           │
│  ignore_changes                                           │
│  └── Don't detect drift on specified attributes          │
│                                                           │
│  replace_triggered_by                                     │
│  └── Force replacement when referenced values change     │
│                                                           │
│  precondition / postcondition                             │
│  └── Custom validation before/after operations           │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 create_before_destroy

```hcl
# DEFAULT behavior: Destroy → Create (causes downtime)
# ┌─────┐          ┌─────┐
# │ Old │ ─destroy─►│     │ ─create─► │ New │
# └─────┘          └─────┘           └─────┘
#                  ↑ DOWNTIME ↑

# WITH create_before_destroy: Create → Destroy (zero downtime)
# ┌─────┐                    ┌─────┐
# │ Old │ ─────────────────►│ Old │ ─destroy─►
# └─────┘                   └─────┘
#          ┌─────┐
#          │ New │ ─create──►│ New │
#          └─────┘           └─────┘

resource "aws_launch_template" "app" {
  name_prefix   = "myapp-"
  image_id      = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    create_before_destroy = true
  }
}

# Common with resources that must always exist:
resource "aws_security_group" "app" {
  name_prefix = "myapp-sg-"
  vpc_id      = var.vpc_id

  lifecycle {
    create_before_destroy = true
  }
}

# SSL certificates (important: no downtime)
resource "aws_acm_certificate" "main" {
  domain_name       = "example.com"
  validation_method = "DNS"

  lifecycle {
    create_before_destroy = true
  }
}
```

> **Note:** `create_before_destroy` may fail if resources have unique constraints (e.g., exact `name` instead of `name_prefix`).

---

## 3.3 prevent_destroy

```hcl
# Protects critical resources from accidental deletion
resource "aws_db_instance" "production" {
  identifier     = "production-db"
  instance_class = "db.r6g.large"
  engine         = "postgres"
  # ...

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket" "data" {
  bucket = "company-critical-data"

  lifecycle {
    prevent_destroy = true
  }
}
```

```
┌──────────────────────────────────────────────────────────┐
│          prevent_destroy BEHAVIOR                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  terraform destroy                                       │
│       │                                                   │
│       ▼                                                   │
│  Plan: destroy aws_db_instance.production                │
│       │                                                   │
│       ▼                                                   │
│  ERROR: Instance cannot be destroyed                     │
│  Resource has lifecycle.prevent_destroy set               │
│                                                           │
│  To actually destroy:                                     │
│  1. Remove prevent_destroy from config                   │
│  2. Run terraform apply                                  │
│  3. Run terraform destroy                                │
│                                                           │
│  NOTE: Removing the resource block entirely from         │
│  config will ALSO trigger the protection!                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.4 ignore_changes

```hcl
# ─── Ignore specific attributes ──────────────────
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name = "myapp"
  }

  lifecycle {
    # Ignore tags changed outside Terraform (e.g., AWS auto-tagging)
    ignore_changes = [tags]
  }
}

# ─── Ignore nested attributes ────────────────────
resource "aws_autoscaling_group" "app" {
  name         = "myapp"
  min_size     = 2
  max_size     = 10
  desired_capacity = 3

  lifecycle {
    # Ignore desired_capacity — managed by autoscaling policies
    ignore_changes = [desired_capacity]
  }
}

# ─── Ignore ALL changes ──────────────────────────
resource "aws_instance" "legacy" {
  ami           = "ami-initial"
  instance_type = "t3.micro"

  lifecycle {
    # Terraform manages creation only; never updates
    ignore_changes = all
  }
}

# ─── Common use cases ────────────────────────────
resource "aws_ecs_service" "app" {
  name            = "myapp"
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 3

  lifecycle {
    # Task definition updated by CI/CD pipeline, not Terraform
    # Desired count managed by autoscaling
    ignore_changes = [task_definition, desired_count]
  }
}

resource "azurerm_kubernetes_cluster" "main" {
  name = "myaks"
  # ...

  default_node_pool {
    name       = "default"
    node_count = 3
  }

  lifecycle {
    # Node count managed by cluster autoscaler
    ignore_changes = [default_node_pool[0].node_count]
  }
}
```

---

## 3.5 replace_triggered_by

```hcl
# Force resource replacement when related values change
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    replace_triggered_by = [
      # Replace instance when certificate changes
      aws_acm_certificate.main.id,

      # Replace when trigger data changes
      terraform_data.deployment.output,
    ]
  }
}

# Trigger resource for manual replacements
resource "terraform_data" "deployment" {
  input = var.deployment_version
}

# Another example: Replace ECS service when task def changes
resource "aws_ecs_service" "app" {
  name            = "myapp"
  task_definition = aws_ecs_task_definition.app.arn

  lifecycle {
    replace_triggered_by = [
      aws_ecs_task_definition.app,
    ]
  }
}
```

---

## 3.6 Preconditions and Postconditions

```hcl
# ─── Preconditions: Validate BEFORE resource action ─
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type

  lifecycle {
    precondition {
      condition     = contains(["t3.micro", "t3.small", "m5.large"], var.instance_type)
      error_message = "Instance type must be t3.micro, t3.small, or m5.large."
    }

    precondition {
      condition     = var.environment != "prod" || var.instance_type == "m5.large"
      error_message = "Production must use m5.large instances."
    }
  }
}

# ─── Postconditions: Validate AFTER resource created ─
data "aws_ami" "app" {
  most_recent = true
  owners      = ["self"]

  filter {
    name   = "name"
    values = ["myapp-*"]
  }

  lifecycle {
    postcondition {
      condition     = self.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture."
    }
  }
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.app.id
  instance_type = "t3.micro"

  lifecycle {
    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must have a public IP address."
    }
  }
}

# ─── Output validation ──────────────────────────
output "api_url" {
  value = "https://${aws_lb.main.dns_name}"

  precondition {
    condition     = aws_lb.main.dns_name != ""
    error_message = "Load balancer DNS name is not available."
  }
}
```

---

## 3.7 Combining Lifecycle Rules

```hcl
# Real-world example with multiple lifecycle rules
resource "aws_db_instance" "production" {
  identifier     = "prod-db"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.r6g.large"

  lifecycle {
    # Don't delete the production database
    prevent_destroy = true

    # Create new instance before destroying old (for engine upgrades)
    create_before_destroy = true

    # Ignore password changes managed externally
    ignore_changes = [password]

    # Validate before creating
    precondition {
      condition     = var.environment == "prod"
      error_message = "This resource is for production only."
    }

    postcondition {
      condition     = self.multi_az == true
      error_message = "Production database must be multi-AZ."
    }
  }
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Can't destroy protected resource | `prevent_destroy = true` | Remove it from config, apply, then destroy |
| Drift not detected | `ignore_changes` too broad | Narrow the list; avoid `all` unless intentional |
| `create_before_destroy` fails | Unique name constraint | Use `name_prefix` instead of `name` |
| `replace_triggered_by` causes unexpected replacement | Referenced resource attribute changed | Reference only stable attributes |
| Precondition error on plan | Validation caught invalid input | Fix the variable values before applying |

---

## Summary Table

| Rule | Purpose | Common Use |
|------|---------|------------|
| `create_before_destroy` | Zero-downtime replacement | Certs, SGs, launch templates |
| `prevent_destroy` | Block accidental deletion | Databases, S3 buckets, critical infra |
| `ignore_changes` | Skip drift on attributes | ASG desired_count, external tags |
| `replace_triggered_by` | Force replacement on change | Linked resource updates |
| `precondition` | Validate before action | Input validation, env checks |
| `postcondition` | Validate after creation | Ensure resource properties |

---

## Quick Revision Questions

1. **What does `create_before_destroy` do?**
   > Creates the new resource before destroying the old one, enabling zero-downtime replacements. Useful for resources that must always exist.

2. **Can you override `prevent_destroy` to delete a resource?**
   > Yes. Remove `prevent_destroy = true` from the config, run `terraform apply`, then `terraform destroy`.

3. **When would you use `ignore_changes = all`?**
   > When Terraform should only manage resource creation but never update it — for example, legacy resources managed externally after initial creation.

4. **What is `replace_triggered_by`?**
   > A lifecycle rule that forces resource replacement when specified referenced resource attributes change, even if the resource's own attributes haven't changed.

5. **What is the difference between precondition and postcondition?**
   > Preconditions validate before a resource is created/updated (input validation). Postconditions validate after creation using `self` to check the resulting resource state.

---

[← Previous: Terraform Functions](02-terraform-functions.md) | [Back to README](../README.md) | [Next: Moved and Removed Blocks →](04-moved-and-removed.md)

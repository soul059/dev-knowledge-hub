# Chapter 5: Writing Reusable Modules

[← Previous: Public Registry Modules](04-registry-modules.md) | [Back to README](../README.md) | [Next: Module Testing →](06-module-testing.md)

---

## Overview

Writing **reusable modules** requires careful API design, sensible defaults, and clear documentation. A good module is like a well-designed function — it does one thing well and is easy to use correctly.

---

## 5.1 Module Design Principles

```
┌──────────────────────────────────────────────────────────┐
│              MODULE DESIGN PRINCIPLES                     │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. SINGLE RESPONSIBILITY                                 │
│     └── Each module manages one logical component         │
│         ✓ "VPC module"  ✗ "Everything module"            │
│                                                           │
│  2. SENSIBLE DEFAULTS                                     │
│     └── Work out of the box with minimal inputs           │
│         Required: only what varies per deployment         │
│         Optional: everything else with good defaults      │
│                                                           │
│  3. COMPOSABILITY                                         │
│     └── Modules should work together via outputs/inputs   │
│         Output everything consumers might need            │
│                                                           │
│  4. FLEXIBILITY without COMPLEXITY                        │
│     └── Feature flags for optional components             │
│         Don't expose every possible setting               │
│                                                           │
│  5. DOCUMENTATION                                         │
│     └── README, variable descriptions, examples           │
│         Users shouldn't need to read your main.tf         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 5.2 Complete Reusable Module Example

```hcl
# ─── modules/web-app/variables.tf ──────────────────────

variable "name" {
  description = "Application name, used as prefix for all resources"
  type        = string

  validation {
    condition     = can(regex("^[a-z][a-z0-9-]{1,28}[a-z0-9]$", var.name))
    error_message = "Name must be 3-30 chars, lowercase, alphanumeric and hyphens."
  }
}

variable "environment" {
  description = "Deployment environment"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

variable "vpc_id" {
  description = "VPC ID where resources will be created"
  type        = string
}

variable "subnet_ids" {
  description = "List of subnet IDs for the application"
  type        = list(string)
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"
}

variable "min_size" {
  description = "Minimum number of instances in ASG"
  type        = number
  default     = 1
}

variable "max_size" {
  description = "Maximum number of instances in ASG"
  type        = number
  default     = 3
}

variable "enable_https" {
  description = "Whether to create HTTPS listener (requires certificate_arn)"
  type        = bool
  default     = false
}

variable "certificate_arn" {
  description = "ACM certificate ARN for HTTPS (required if enable_https = true)"
  type        = string
  default     = null
}

variable "health_check_path" {
  description = "Path for ALB health checks"
  type        = string
  default     = "/health"
}

variable "tags" {
  description = "Additional tags for all resources"
  type        = map(string)
  default     = {}
}


# ─── modules/web-app/locals.tf ─────────────────────────

locals {
  common_tags = merge(var.tags, {
    Application = var.name
    Environment = var.environment
    ManagedBy   = "terraform"
  })

  name_prefix = "${var.name}-${var.environment}"
}


# ─── modules/web-app/main.tf ───────────────────────────

resource "aws_security_group" "alb" {
  name_prefix = "${local.name_prefix}-alb-"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  dynamic "ingress" {
    for_each = var.enable_https ? [443] : []
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = local.common_tags

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group" "app" {
  name_prefix = "${local.name_prefix}-app-"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = 8080
    to_port         = 8080
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = local.common_tags

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_lb" "this" {
  name               = local.name_prefix
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = var.subnet_ids

  tags = local.common_tags
}

resource "aws_lb_target_group" "this" {
  name     = local.name_prefix
  port     = 8080
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  health_check {
    path                = var.health_check_path
    healthy_threshold   = 2
    unhealthy_threshold = 3
    interval            = 30
    timeout             = 5
  }

  tags = local.common_tags
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type = var.enable_https ? "redirect" : "forward"

    dynamic "redirect" {
      for_each = var.enable_https ? [1] : []
      content {
        port        = "443"
        protocol    = "HTTPS"
        status_code = "HTTP_301"
      }
    }

    dynamic "forward" {
      for_each = var.enable_https ? [] : [1]
      content {
        target_group {
          arn = aws_lb_target_group.this.arn
        }
      }
    }
  }
}


# ─── modules/web-app/outputs.tf ────────────────────────

output "alb_dns_name" {
  description = "DNS name of the Application Load Balancer"
  value       = aws_lb.this.dns_name
}

output "alb_zone_id" {
  description = "Zone ID of the ALB (for Route53 alias)"
  value       = aws_lb.this.zone_id
}

output "alb_arn" {
  description = "ARN of the ALB"
  value       = aws_lb.this.arn
}

output "target_group_arn" {
  description = "ARN of the target group"
  value       = aws_lb_target_group.this.arn
}

output "app_security_group_id" {
  description = "Security group ID for application instances"
  value       = aws_security_group.app.id
}

output "url" {
  description = "Application URL"
  value       = var.enable_https ? "https://${aws_lb.this.dns_name}" : "http://${aws_lb.this.dns_name}"
}
```

---

## 5.3 Usage of the Module

```hcl
# Simple usage (dev)
module "app_dev" {
  source = "./modules/web-app"

  name        = "myapp"
  environment = "dev"
  vpc_id      = module.vpc.vpc_id
  subnet_ids  = module.vpc.public_subnets
}

# Full usage (prod)
module "app_prod" {
  source = "./modules/web-app"

  name            = "myapp"
  environment     = "prod"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.public_subnets
  instance_type   = "m5.large"
  min_size        = 3
  max_size        = 10
  enable_https    = true
  certificate_arn = aws_acm_certificate.main.arn

  tags = {
    CostCenter = "engineering"
    Team       = "platform"
  }
}
```

---

## 5.4 Anti-Patterns to Avoid

```
┌──────────────────────────────────────────────────────────┐
│              MODULE ANTI-PATTERNS                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ✗ GOD MODULE                                            │
│    Module that does everything — hard to understand       │
│    Solution: Split into focused, composable modules       │
│                                                           │
│  ✗ HARDCODED VALUES                                      │
│    Embedding environment-specific values                  │
│    Solution: Use variables with validation                │
│                                                           │
│  ✗ PROVIDER CONFIGURATION IN MODULE                      │
│    Declaring provider blocks inside child modules         │
│    Solution: Inherit provider from root module            │
│                                                           │
│  ✗ EXPOSING EVERYTHING                                   │
│    100+ variables for every possible setting              │
│    Solution: Sensible defaults; expose what's needed      │
│                                                           │
│  ✗ TIGHT COUPLING                                        │
│    Module depends on specific resource names              │
│    Solution: Pass dependencies as variables               │
│                                                           │
│  ✗ NO OUTPUTS                                            │
│    Module creates resources but exposes nothing           │
│    Solution: Output IDs, ARNs, and key attributes         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Module works in dev, fails in prod | Hardcoded assumptions | Use variables for all environment-specific values |
| Provider errors in child module | Provider configured in module | Remove provider block from module; inherit from root |
| Can't compose modules together | Missing outputs | Add outputs for all attributes consumers might need |
| Module too complex to use | Too many required variables | Add sensible defaults; use optional object attributes |
| Breaking changes when updating | No version pinning | Pin versions; follow semver; maintain changelog |

---

## Summary Table

| Principle | Implementation |
|-----------|---------------|
| **Single Responsibility** | One logical component per module |
| **Sensible Defaults** | `default = value` on optional variables |
| **Validation** | `validation {}` blocks for constraints |
| **Feature Flags** | `enable_*` boolean variables |
| **Composability** | Rich outputs; dependencies via variables |
| **Documentation** | Descriptions on all variables and outputs |
| **No Provider Config** | Inherit providers from root module |
| **Tags Pattern** | `merge(defaults, var.tags)` |

---

## Quick Revision Questions

1. **What is the "single responsibility" principle for modules?**
   > Each module should manage one logical component (e.g., VPC, compute, database). Avoid "god modules" that do everything.

2. **Why should you NOT configure providers inside child modules?**
   > Child modules should inherit provider configuration from the root module to maintain flexibility and avoid conflicts.

3. **How do feature flags work in modules?**
   > Boolean variables (e.g., `enable_https`) combined with conditional expressions (`count = var.enable_https ? 1 : 0`) to include/exclude optional resources.

4. **What makes a module composable?**
   > Accepting dependencies as variables (not hardcoding them) and exposing sufficient outputs for other modules to connect.

5. **What is the recommended tagging pattern?**
   > Define default tags in `locals` and merge with user-provided tags: `merge(local.default_tags, var.tags)`.

---

[← Previous: Public Registry Modules](04-registry-modules.md) | [Back to README](../README.md) | [Next: Module Testing →](06-module-testing.md)

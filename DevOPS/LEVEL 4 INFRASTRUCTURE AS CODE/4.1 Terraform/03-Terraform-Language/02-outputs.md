# Chapter 2: Outputs

[← Previous: Variables](01-variables.md) | [Back to README](../README.md) | [Next: Local Values →](03-local-values.md)

---

## Overview

**Output values** expose information about your infrastructure after `terraform apply`. They're used to display important data (like IP addresses), pass information between modules, and share data with external tools and scripts.

---

## 2.1 Output Declaration

```hcl
# outputs.tf

# Basic output
output "instance_ip" {
  description = "Public IP of the web server"
  value       = aws_instance.web.public_ip
}

# Output with sensitive data
output "db_connection_string" {
  description = "Database connection string"
  value       = "postgresql://${var.db_user}:${var.db_pass}@${aws_db_instance.main.endpoint}/appdb"
  sensitive   = true
}

# Output with precondition
output "lb_dns" {
  description = "Load balancer DNS name"
  value       = aws_lb.web.dns_name

  precondition {
    condition     = aws_lb.web.dns_name != ""
    error_message = "Load balancer DNS is empty — deployment may have failed."
  }
}
```

```
  ┌──────────────────────────────────────────────────────────┐
  │               OUTPUT BLOCK STRUCTURE                     │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │  output "name" {                                         │
  │    description = "..."     ← Human-readable purpose     │
  │    value       = expr      ← The data to expose         │
  │    sensitive   = true      ← Hide from CLI output       │
  │    depends_on  = [...]     ← Explicit dependencies      │
  │                                                           │
  │    precondition {          ← Validation (TF 1.2+)       │
  │      condition     = ...                                 │
  │      error_message = "..."                               │
  │    }                                                      │
  │  }                                                        │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 2.2 Viewing Outputs

```bash
# Show all outputs after apply
terraform output

# Show specific output
terraform output instance_ip

# Get raw value (no quotes) — useful in scripts
terraform output -raw instance_ip

# JSON format — for programmatic consumption
terraform output -json

# Example script usage
SERVER_IP=$(terraform output -raw instance_ip)
ssh ubuntu@$SERVER_IP
```

---

## 2.3 Outputs Between Modules

```
  ┌──────────────────────────────────────────────────────────┐
  │            OUTPUTS CONNECTING MODULES                    │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │   Root Module                                            │
  │   ┌──────────────────────────────────────────────┐      │
  │   │                                               │      │
  │   │  module "vpc" {              module "app" {   │      │
  │   │    source = "./modules/vpc"    source = "..." │      │
  │   │  }                            vpc_id =       │      │
  │   │        │                    module.vpc.vpc_id │      │
  │   │        │         ┌─────────────▶  │           │      │
  │   │        ▼         │                            │      │
  │   │  VPC Module      │          App Module        │      │
  │   │  output "vpc_id" │                            │      │
  │   │     ─────────────┘                            │      │
  │   │                                               │      │
  │   └──────────────────────────────────────────────┘      │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

```hcl
# modules/vpc/outputs.tf
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of public subnets"
  value       = aws_subnet.public[*].id
}

# root main.tf — consuming module outputs
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}

module "app" {
  source     = "./modules/app"
  vpc_id     = module.vpc.vpc_id              # ← Module output
  subnet_ids = module.vpc.public_subnet_ids   # ← Module output
}
```

---

## 2.4 Common Output Patterns

```hcl
# Multiple resource outputs
output "instance_ids" {
  description = "IDs of all EC2 instances"
  value       = aws_instance.web[*].id
}

# Computed output using functions
output "instance_summary" {
  description = "Summary of deployed instances"
  value = {
    count      = length(aws_instance.web)
    public_ips = aws_instance.web[*].public_ip
    region     = var.region
  }
}

# Conditional output
output "bastion_ip" {
  description = "Bastion host IP (if enabled)"
  value       = var.enable_bastion ? aws_instance.bastion[0].public_ip : "N/A"
}

# Formatted output
output "ssh_command" {
  description = "SSH command to connect"
  value       = "ssh -i ${var.key_name}.pem ubuntu@${aws_instance.web.public_ip}"
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Output value `(known after apply)` | Resource not yet created | Run `terraform apply` first |
| Output shows `(sensitive value)` | Output marked `sensitive = true` | Use `terraform output -raw name` or `-json` |
| "Output refers to resource that doesn't exist" | Typo or resource not defined | Check resource name and type |
| Can't access module output | Output not declared in module | Add `output` block in the module's `outputs.tf` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Output Value** | Exposes data after apply for display or cross-module use |
| **value** | Expression that computes the output data |
| **sensitive** | Hides value from CLI; still in state file |
| **depends_on** | Explicit dependency for output evaluation |
| **precondition** | Validates output value meets expectations |
| **terraform output** | CLI command to view all outputs |
| **-raw** | Returns raw value without formatting |
| **-json** | Returns outputs in JSON format |
| **Module outputs** | Enable data flow between modules |

---

## Quick Revision Questions

1. **What are the three main uses of Terraform output values?**
   > Display information to users, pass data between modules, and provide data to external scripts/tools.

2. **How do you get an output value without quotes for use in a shell script?**
   > `terraform output -raw output_name`.

3. **How are outputs used to connect modules?**
   > A child module declares `output` blocks, and the parent module references them as `module.module_name.output_name`.

4. **What does `sensitive = true` do on an output?**
   > It hides the value from CLI display during plan/apply. The value is still stored in the state file.

5. **When would you use a `precondition` in an output block?**
   > To validate that the output value meets expectations, e.g., ensuring an IP address or DNS name is not empty after deployment.

6. **What is the command to view outputs in JSON format?**
   > `terraform output -json`.

---

[← Previous: Variables](01-variables.md) | [Back to README](../README.md) | [Next: Local Values →](03-local-values.md)

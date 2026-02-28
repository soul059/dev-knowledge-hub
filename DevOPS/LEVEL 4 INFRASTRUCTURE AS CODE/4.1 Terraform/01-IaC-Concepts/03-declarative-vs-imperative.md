# Chapter 3: Declarative vs Imperative

[← Previous: IaC Benefits](02-iac-benefits.md) | [Back to README](../README.md) | [Next: IaC Tools Comparison →](04-iac-tools-comparison.md)

---

## Overview

The two fundamental approaches to Infrastructure as Code are **Declarative** ("tell me WHAT you want") and **Imperative** ("tell me HOW to do it"). Understanding this distinction is critical for choosing the right tools and writing effective infrastructure code.

---

## 3.1 Declarative Approach

The declarative approach describes the **desired end state** of your infrastructure. The tool figures out how to get there.

```
  ┌──────────────────────────────────────────────────────────┐
  │                  DECLARATIVE APPROACH                    │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │   YOU SAY:           "I want 3 web servers"              │
  │                                                           │
  │   ┌──────────┐       ┌──────────┐       ┌──────────┐    │
  │   │  Config  │──────▶│  Engine  │──────▶│  Result  │    │
  │   │ (WHAT)   │       │ (HOW)    │       │  3 VMs   │    │
  │   └──────────┘       └──────────┘       └──────────┘    │
  │                           │                               │
  │                      Tool decides:                        │
  │                      • API calls order                    │
  │                      • Dependencies                       │
  │                      • Create/update/delete              │
  │                                                           │
  │   EXAMPLES: Terraform, CloudFormation, Kubernetes YAML   │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

### Declarative Example (Terraform)

```hcl
# "I want 3 web servers behind a load balancer"
# Terraform figures out the rest

resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

resource "aws_lb" "web" {
  name               = "web-lb"
  load_balancer_type = "application"
}

# Run 1: Creates 3 instances + 1 LB
# Run 2: No changes (already matches desired state)
# Run 3: No changes (still matches)
```

### How Declarative Handles Changes

```
  Current State: 3 servers          Desired State: 5 servers
  ┌───┐ ┌───┐ ┌───┐                ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
  │ 1 │ │ 2 │ │ 3 │          ───▶  │ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │
  └───┘ └───┘ └───┘                └───┘ └───┘ └───┘ └───┘ └───┘
                                                       ▲     ▲
  Change count = 3 to count = 5                  Only these are
  Terraform adds 2 MORE                          created (smart!)
```

---

## 3.2 Imperative Approach

The imperative approach describes the **exact steps** to achieve the desired outcome. You tell the tool HOW to do it.

```
  ┌──────────────────────────────────────────────────────────┐
  │                  IMPERATIVE APPROACH                     │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │   YOU SAY:           "Create server A, then B, then C"   │
  │                                                           │
  │   ┌──────────┐  Step 1  ┌──────┐                        │
  │   │  Script  │─────────▶│ VM A │                        │
  │   │ (HOW)    │  Step 2  ┌──────┐                        │
  │   │          │─────────▶│ VM B │                        │
  │   │          │  Step 3  ┌──────┐                        │
  │   │          │─────────▶│ VM C │                        │
  │   └──────────┘          └──────┘                        │
  │                                                           │
  │   YOU control:                                           │
  │   • Exact order of operations                            │
  │   • Every API call                                       │
  │   • Error handling                                       │
  │   • Conditional logic                                    │
  │                                                           │
  │   EXAMPLES: Bash scripts, Ansible playbooks, AWS CLI     │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

### Imperative Example (Bash/AWS CLI)

```bash
#!/bin/bash
# "Here's exactly HOW to create 3 web servers"

# Step 1: Create VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)

# Step 2: Create subnet
SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --query 'Subnet.SubnetId' --output text)

# Step 3: Create 3 instances
for i in 1 2 3; do
  aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t3.micro \
    --subnet-id $SUBNET_ID \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=web-$i}]"
done

# Run this again? You get 3 MORE servers (6 total)! NOT idempotent!
```

### The Idempotency Problem

```
  IMPERATIVE — Running the script 3 times:
  
  Run 1: Creates 3 servers    → Total: 3 servers ✓
  Run 2: Creates 3 MORE       → Total: 6 servers ✗
  Run 3: Creates 3 MORE       → Total: 9 servers ✗
  
  DECLARATIVE — Running apply 3 times:
  
  Run 1: Creates 3 servers    → Total: 3 servers ✓
  Run 2: No changes needed    → Total: 3 servers ✓
  Run 3: No changes needed    → Total: 3 servers ✓
```

---

## 3.3 Side-by-Side Comparison

```
  ┌─────────────────────────┬─────────────────────────┐
  │      DECLARATIVE        │      IMPERATIVE          │
  ├─────────────────────────┼─────────────────────────┤
  │                         │                          │
  │  "I want 3 servers"    │  "Create server 1"      │
  │                         │  "Create server 2"      │
  │                         │  "Create server 3"      │
  │                         │                          │
  │  Tool manages HOW      │  YOU manage HOW          │
  │  Idempotent by default │  Must code idempotency   │
  │  State tracking built-in│  You track state         │
  │  Easier to maintain    │  Complex at scale        │
  │  Less flexible         │  Maximum flexibility     │
  │                         │                          │
  │  ┌─────────┐           │  ┌─────────┐            │
  │  │ Terraform│           │  │  Bash   │            │
  │  │ CloudForm│           │  │ Ansible │            │
  │  │ Pulumi * │           │  │ Chef    │            │
  │  └─────────┘           │  │ Puppet  │            │
  │                         │  └─────────┘            │
  │  * Pulumi is hybrid    │                          │
  │                         │                          │
  └─────────────────────────┴─────────────────────────┘
```

### Detailed Comparison Table

| Aspect | Declarative | Imperative |
|--------|-------------|------------|
| **Focus** | WHAT (desired state) | HOW (steps) |
| **Idempotency** | Built-in | Must implement manually |
| **State Awareness** | Tracks current state automatically | Developer must track |
| **Learning Curve** | Must learn DSL (HCL) | Use existing programming skills |
| **Flexibility** | Constrained by DSL | Full programming power |
| **Drift Detection** | Automatic | Must build detection logic |
| **Ordering** | Automatic dependency resolution | Manual ordering |
| **Error Recovery** | Built-in rollback/retry | Custom error handling |
| **Readability** | Easy to understand desired state | Logic can be complex |
| **Best For** | Provisioning cloud resources | Configuration management, scripting |

---

## 3.4 Real-World Decision Flow

```
  ┌──────────────────────────────────────────────────────────────┐
  │              WHICH APPROACH SHOULD I USE?                    │
  ├──────────────────────────────────────────────────────────────┤
  │                                                               │
  │   Need to provision cloud infrastructure?                    │
  │   ├── YES → DECLARATIVE (Terraform)                         │
  │   │                                                           │
  │   Need to configure software on existing servers?            │
  │   ├── YES → IMPERATIVE (Ansible) or HYBRID                  │
  │   │                                                           │
  │   Need complex logic/conditionals?                           │
  │   ├── YES → Consider IMPERATIVE or Pulumi (hybrid)          │
  │   │                                                           │
  │   Need drift detection & state management?                   │
  │   ├── YES → DECLARATIVE (Terraform)                         │
  │   │                                                           │
  │   One-time migration script?                                 │
  │   ├── YES → IMPERATIVE (Bash/Python)                        │
  │   │                                                           │
  │   Most common answer: Use BOTH together!                     │
  │   Terraform (provision) + Ansible (configure)               │
  │                                                               │
  └──────────────────────────────────────────────────────────────┘
```

### Best of Both Worlds

```
  ┌─────────────────────────────────────────────────┐
  │             COMMON PRODUCTION PATTERN           │
  ├─────────────────────────────────────────────────┤
  │                                                  │
  │   Phase 1: DECLARATIVE (Terraform)              │
  │   ┌──────────────────────────────────┐          │
  │   │ Create VPC, subnets, security    │          │
  │   │ groups, EC2 instances, RDS, S3   │          │
  │   └──────────────┬───────────────────┘          │
  │                  │                               │
  │                  ▼                               │
  │   Phase 2: IMPERATIVE (Ansible/Scripts)         │
  │   ┌──────────────────────────────────┐          │
  │   │ Install packages, configure      │          │
  │   │ services, deploy application     │          │
  │   └──────────────────────────────────┘          │
  │                                                  │
  └─────────────────────────────────────────────────┘
```

---

## 3.5 Handling Edge Cases

### Declarative Limitations and Workarounds

```hcl
# Terraform is declarative, but has escape hatches for imperative needs

# 1. Provisioners (imperative within declarative)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # Run script AFTER instance is created (imperative step)
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx"
    ]
  }
}

# 2. null_resource for arbitrary scripts
resource "null_resource" "setup" {
  provisioner "local-exec" {
    command = "bash scripts/setup.sh"
  }
}

# NOTE: HashiCorp recommends AVOIDING provisioners when possible
# Use cloud-init, user_data, or image builders instead
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Script runs twice, creates duplicates | Imperative scripts aren't idempotent | Switch to declarative (Terraform) or add existence checks |
| Terraform can't handle complex logic | Declarative DSL is limited | Use `for_each`, `dynamic` blocks, or Pulumi for complex cases |
| Resources created in wrong order | Imperative scripts don't resolve deps | Use declarative tools that auto-resolve dependency graphs |
| Can't install packages via Terraform | Terraform provisions infra, not config | Use Ansible or cloud-init for config management |

---

## Summary Table

| Concept | Declarative | Imperative |
|---------|-------------|------------|
| **Definition** | Describe desired end state | Describe step-by-step instructions |
| **Analogy** | "I want a pizza with mushrooms" (restaurant) | "Roll dough, add sauce, add cheese..." (recipe) |
| **Idempotent** | Yes, by default | No, must be coded |
| **State Management** | Automatic | Manual |
| **Example Tools** | Terraform, CloudFormation, Pulumi | Bash, Ansible, Chef, AWS CLI |
| **Terraform's Approach** | Primarily declarative | Provisioners for imperative escape hatch |
| **Best For** | Infrastructure provisioning | Software configuration |
| **Common Pattern** | Terraform + Ansible together | Provision then configure |

---

## Quick Revision Questions

1. **What is the key difference between declarative and imperative IaC?**
   > Declarative describes WHAT you want (desired state). Imperative describes HOW to achieve it (step-by-step commands).

2. **Why is idempotency important, and which approach provides it by default?**
   > Idempotency ensures running code multiple times produces the same result. Declarative tools like Terraform provide this by default.

3. **Give a real-world analogy for declarative vs imperative.**
   > Declarative: Ordering food at a restaurant ("I want a burger"). Imperative: Following a recipe step by step ("grill patty 4 min per side, toast bun...").

4. **Can Terraform perform imperative operations? If so, how?**
   > Yes, using provisioners (`local-exec`, `remote-exec`), but HashiCorp recommends avoiding them in favor of cloud-init or image builders.

5. **Why might you use BOTH declarative and imperative tools together?**
   > Terraform (declarative) excels at provisioning infrastructure, while Ansible (imperative) excels at configuring software on that infrastructure.

6. **What happens if you run an imperative script that creates 3 servers a second time?**
   > It creates 3 MORE servers (6 total) because imperative scripts don't track state. A declarative tool would recognize 3 already exist and make no changes.

---

[← Previous: IaC Benefits](02-iac-benefits.md) | [Back to README](../README.md) | [Next: IaC Tools Comparison →](04-iac-tools-comparison.md)

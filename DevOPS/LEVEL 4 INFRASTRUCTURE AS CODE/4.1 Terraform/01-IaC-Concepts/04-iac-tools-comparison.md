# Chapter 4: IaC Tools Comparison

[← Previous: Declarative vs Imperative](03-declarative-vs-imperative.md) | [Back to README](../README.md) | [Next: Terraform Overview →](05-terraform-overview.md)

---

## Overview

The IaC ecosystem has several mature tools, each with different strengths. This chapter compares **Terraform, AWS CloudFormation, Pulumi, Ansible, and others** to help you understand where Terraform fits and when to use alternatives.

---

## 4.1 The IaC Landscape

```
┌──────────────────────────────────────────────────────────────────┐
│                    IaC TOOLS LANDSCAPE                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   PROVISIONING (Create Infrastructure)                           │
│   ┌────────────┐  ┌──────────────┐  ┌────────┐  ┌───────────┐  │
│   │ Terraform  │  │CloudFormation│  │ Pulumi │  │  Bicep    │  │
│   │ (Multi)    │  │ (AWS only)   │  │(Multi) │  │(Azure only)│  │
│   └────────────┘  └──────────────┘  └────────┘  └───────────┘  │
│                                                                   │
│   CONFIGURATION (Configure Software)                             │
│   ┌────────────┐  ┌──────────────┐  ┌────────┐  ┌───────────┐  │
│   │  Ansible   │  │    Chef      │  │ Puppet │  │ SaltStack │  │
│   │ (Agentless)│  │  (Agent)     │  │(Agent) │  │  (Agent)  │  │
│   └────────────┘  └──────────────┘  └────────┘  └───────────┘  │
│                                                                   │
│   CONTAINERS / ORCHESTRATION                                     │
│   ┌────────────┐  ┌──────────────┐  ┌─────────────────────┐    │
│   │ Dockerfile │  │  Kubernetes  │  │  Docker Compose     │    │
│   │            │  │   Manifests  │  │                     │    │
│   └────────────┘  └──────────────┘  └─────────────────────┘    │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Terraform

```
┌────────────────────────────────────────────────────┐
│                  TERRAFORM                         │
├────────────────────────────────────────────────────┤
│  Vendor:    HashiCorp                              │
│  Language:  HCL (HashiCorp Configuration Language) │
│  Type:      Declarative                            │
│  Scope:     Multi-cloud (AWS, Azure, GCP, etc.)   │
│  State:     Yes (local or remote)                  │
│  License:   BSL 1.1 (Terraform 1.6+)              │
│  Alt Fork:  OpenTofu (MPL 2.0)                     │
└────────────────────────────────────────────────────┘
```

```hcl
# Terraform Example — AWS EC2 Instance
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name = "web-server"
  }
}
```

**Strengths:**
- Multi-cloud support (700+ providers)
- Mature ecosystem and large community
- Excellent state management
- Module registry for reusable components
- `plan` command for safety

**Weaknesses:**
- HCL has limited programming constructs
- State file management adds complexity
- BSL license concerns (see OpenTofu)

---

## 4.3 AWS CloudFormation

```
┌────────────────────────────────────────────────────┐
│             AWS CLOUDFORMATION                     │
├────────────────────────────────────────────────────┤
│  Vendor:    Amazon Web Services                    │
│  Language:  JSON or YAML                           │
│  Type:      Declarative                            │
│  Scope:     AWS ONLY                               │
│  State:     Managed by AWS (stacks)                │
│  License:   Proprietary (free to use with AWS)     │
└────────────────────────────────────────────────────┘
```

```yaml
# CloudFormation Example — AWS EC2 Instance
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t3.micro
      Tags:
        - Key: Name
          Value: web-server
```

**Strengths:**
- Native AWS integration (day-1 support)
- No state file to manage (AWS manages it)
- Free to use (no extra cost)
- Drift detection built-in
- Stack sets for multi-account deployment

**Weaknesses:**
- AWS only — **vendor lock-in**
- Verbose JSON/YAML templates
- Limited logic capabilities
- Slow updates for new features
- Difficult rollbacks on some resources

---

## 4.4 Pulumi

```
┌────────────────────────────────────────────────────┐
│                    PULUMI                           │
├────────────────────────────────────────────────────┤
│  Vendor:    Pulumi Corp                            │
│  Language:  Python, TypeScript, Go, C#, Java       │
│  Type:      Imperative / Declarative (hybrid)      │
│  Scope:     Multi-cloud                            │
│  State:     Pulumi Cloud or self-managed           │
│  License:   Apache 2.0 (engine)                    │
└────────────────────────────────────────────────────┘
```

```python
# Pulumi Example — AWS EC2 Instance (Python)
import pulumi
import pulumi_aws as aws

web_server = aws.ec2.Instance("web-server",
    ami="ami-0c55b159cbfafe1f0",
    instance_type="t3.micro",
    tags={"Name": "web-server"}
)

pulumi.export("public_ip", web_server.public_ip)
```

**Strengths:**
- Use real programming languages (Python, TS, Go, etc.)
- Full programming constructs (loops, conditionals, classes)
- Multi-cloud support
- Familiar for developers
- Strong typing and IDE support

**Weaknesses:**
- Smaller community than Terraform
- State management via Pulumi Cloud (or S3)
- Less mature module ecosystem
- Debugging infrastructure code is harder

---

## 4.5 Azure Bicep

```
┌────────────────────────────────────────────────────┐
│                  AZURE BICEP                       │
├────────────────────────────────────────────────────┤
│  Vendor:    Microsoft                              │
│  Language:  Bicep (DSL)                            │
│  Type:      Declarative                            │
│  Scope:     Azure ONLY                             │
│  State:     Managed by Azure (deployments)         │
│  License:   MIT                                    │
└────────────────────────────────────────────────────┘
```

```bicep
// Bicep Example — Azure VM
resource webServer 'Microsoft.Compute/virtualMachines@2021-03-01' = {
  name: 'web-server'
  location: 'eastus'
  properties: {
    hardwareProfile: {
      vmSize: 'Standard_B1s'
    }
  }
}
```

**Strengths:**
- Native Azure support (transpiles to ARM JSON)
- Cleaner syntax than ARM templates
- No state file management
- First-class VS Code support
- Free to use

**Weaknesses:**
- Azure only — **vendor lock-in**
- Relatively new (fewer resources/examples)
- Limited community compared to Terraform

---

## 4.6 Ansible (Configuration Management)

```
┌────────────────────────────────────────────────────┐
│                   ANSIBLE                          │
├────────────────────────────────────────────────────┤
│  Vendor:    Red Hat                                │
│  Language:  YAML (Playbooks)                       │
│  Type:      Imperative (procedural)                │
│  Scope:     Multi-cloud + on-premises              │
│  State:     Stateless (no state file)              │
│  Agent:     Agentless (SSH-based)                  │
│  License:   GPL v3                                 │
└────────────────────────────────────────────────────┘
```

```yaml
# Ansible Example — Install Nginx
- hosts: webservers
  become: yes
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

**Best used alongside Terraform:**
```
Terraform → Provisions infrastructure (VMs, networks, databases)
Ansible   → Configures software on that infrastructure
```

---

## 4.7 Comprehensive Comparison

### Feature Matrix

```
┌──────────────────┬───────────┬────────────┬────────┬───────┬─────────┐
│     Feature      │ Terraform │ CloudForm  │ Pulumi │ Bicep │ Ansible │
├──────────────────┼───────────┼────────────┼────────┼───────┼─────────┤
│ Multi-Cloud      │    ✓      │     ✗      │   ✓    │  ✗    │   ✓     │
│ Language         │   HCL     │ YAML/JSON  │ Python │ Bicep │  YAML   │
│                  │           │            │ TS/Go  │       │         │
│ Declarative      │    ✓      │     ✓      │ Hybrid │  ✓    │  Mostly │
│                  │           │            │        │       │  Imper. │
│ State Mgmt       │   Self    │  Managed   │ Cloud  │Managed│  None   │
│ Drift Detection  │    ✓      │     ✓      │   ✓    │  ✓    │   ✗     │
│ Plan/Preview     │    ✓      │  Changeset │   ✓    │What-If│   ✓‡    │
│ Module Registry  │    ✓      │     ✗*     │   ✓    │  ✗    │ Galaxy  │
│ Learning Curve   │  Medium   │   Medium   │  Low†  │ Medium│  Low    │
│ Community Size   │  Largest  │   Large    │ Growing│ Medium│  Large  │
│ Free/OSS         │  BSL 1.1  │    Free    │Apache2 │  MIT  │  GPL3   │
│ CI/CD Support    │   ✓✓✓     │    ✓✓      │  ✓✓    │  ✓✓   │  ✓✓     │
│ Enterprise       │ TF Cloud  │  StackSets │ Cloud  │  N/A  │  Tower  │
└──────────────────┴───────────┴────────────┴────────┴───────┴─────────┘

  ✗*  CloudFormation has modules/nested stacks but no public registry
  †   Low if you already know Python/TypeScript
  ✓‡  Ansible has --check mode (dry run)
```

### When to Use Each Tool

```
┌─────────────────────────────────────────────────────────────────┐
│                DECISION MATRIX                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "I need multi-cloud infrastructure"                            │
│   └──▶  Terraform or Pulumi                                    │
│                                                                  │
│  "I'm AWS-only and want zero state management"                  │
│   └──▶  CloudFormation                                          │
│                                                                  │
│  "I'm Azure-only and want clean syntax"                         │
│   └──▶  Bicep                                                   │
│                                                                  │
│  "I want to use Python/TypeScript for infra"                    │
│   └──▶  Pulumi                                                  │
│                                                                  │
│  "I need to configure software on servers"                      │
│   └──▶  Ansible                                                 │
│                                                                  │
│  "I need both provisioning AND configuration"                   │
│   └──▶  Terraform + Ansible                                    │
│                                                                  │
│  "I want the largest community and ecosystem"                   │
│   └──▶  Terraform                                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4.8 Real-World Scenario: Choosing the Right Tool

### Scenario: E-commerce Company Migration

```
  Company: MegaShop Inc.
  Current: On-premises data center
  Target:  AWS + some Azure services
  Team:    10 engineers (mix of Python and Ops backgrounds)
  
  ┌─────────────────────────────────────────────────────────┐
  │                    TOOL SELECTION                        │
  ├─────────────────────────────────────────────────────────┤
  │                                                          │
  │  Requirement             →  Tool Selected               │
  │  ─────────────────────       ──────────────             │
  │  Multi-cloud (AWS+Azure) →  Terraform ✓                │
  │  CloudFormation?         →  ✗ (Azure not supported)    │
  │  Pulumi?                 →  Considered, smaller         │
  │                              community for support      │
  │  Configuration mgmt?    →  Ansible (alongside TF)      │
  │                                                          │
  │  DECISION: Terraform + Ansible                          │
  │                                                          │
  │  Terraform: VPCs, EC2, RDS, S3, Azure VNets, Cosmos DB │
  │  Ansible:   Install apps, configure services            │
  │                                                          │
  └─────────────────────────────────────────────────────────┘
```

---

## 4.9 OpenTofu — The Open-Source Fork

After HashiCorp changed Terraform's license from MPL 2.0 to BSL 1.1 (Aug 2023), the community created **OpenTofu** — a truly open-source fork:

```
┌──────────────────────────────────────────────────────┐
│              TERRAFORM vs OPENTOFU                   │
├──────────────────────────────────────────────────────┤
│                                                       │
│  Terraform (HashiCorp)     OpenTofu (Linux Foundation)│
│  ├── BSL 1.1 License       ├── MPL 2.0 License       │
│  ├── HashiCorp controlled  ├── Community governed     │
│  ├── terraform CLI          ├── tofu CLI              │
│  ├── registry.terraform.io ├── Compatible registries │
│  └── Terraform Cloud       └── No cloud service      │
│                                                       │
│  Both use HCL, both share the same concepts.         │
│  Migration: rename binary, minor config changes.     │
│                                                       │
└──────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| CloudFormation can't manage Azure | CloudFormation is AWS-only | Use Terraform for multi-cloud |
| Terraform HCL feels limiting | HCL is a DSL, not a full language | Use Pulumi or Terraform CDK for programming language support |
| Too many tools in the stack | Over-engineering | Start with Terraform; add Ansible only if you configure servers |
| Team split on tool choice | Different preferences | Evaluate based on requirements matrix above, not preferences |

---

## Summary Table

| Tool | Best For | Cloud Support | Language | State |
|------|----------|---------------|----------|-------|
| **Terraform** | Multi-cloud provisioning | All clouds | HCL | Self-managed |
| **CloudFormation** | AWS-native provisioning | AWS only | YAML/JSON | AWS-managed |
| **Pulumi** | Developer-centric IaC | All clouds | Python/TS/Go/C# | Pulumi Cloud |
| **Bicep** | Azure-native provisioning | Azure only | Bicep DSL | Azure-managed |
| **Ansible** | Configuration management | All + on-prem | YAML | Stateless |
| **Chef** | Config management (Ruby) | All + on-prem | Ruby DSL | Stateful |
| **Puppet** | Config management (enterprise) | All + on-prem | Puppet DSL | Stateful |
| **OpenTofu** | Open-source Terraform alternative | All clouds | HCL | Self-managed |

---

## Quick Revision Questions

1. **Name three declarative IaC provisioning tools and their primary cloud support.**
   > Terraform (multi-cloud), CloudFormation (AWS only), Bicep (Azure only).

2. **Why would you choose Terraform over CloudFormation?**
   > Terraform supports multiple clouds (AWS, Azure, GCP), has a module registry, and avoids vendor lock-in. CloudFormation is AWS-only.

3. **What makes Pulumi different from Terraform?**
   > Pulumi uses real programming languages (Python, TypeScript, Go) instead of a DSL, offering full programming constructs like loops, classes, and error handling.

4. **When would you use Ansible alongside Terraform?**
   > Terraform provisions infrastructure (VMs, networks). Ansible configures software on that infrastructure (install packages, configure services).

5. **What is OpenTofu, and why was it created?**
   > OpenTofu is an open-source fork of Terraform created after HashiCorp changed to BSL 1.1 license. It's MPL 2.0 licensed and community-governed.

6. **What is vendor lock-in, and which tools help avoid it?**
   > Vendor lock-in means being tied to one cloud provider. Terraform, Pulumi, and Ansible support multi-cloud, helping avoid lock-in.

---

[← Previous: Declarative vs Imperative](03-declarative-vs-imperative.md) | [Back to README](../README.md) | [Next: Terraform Overview →](05-terraform-overview.md)

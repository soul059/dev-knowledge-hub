# Chapter 4.5 — IaC Tools Deep Dive

[← Previous: Container Orchestration](04-container-orchestration.md) | [Next: Monitoring Tools →](06-monitoring-tools.md)

---

## Overview

This chapter compares the major Infrastructure as Code tools in depth — **Terraform, AWS CloudFormation, Pulumi, and Crossplane** — helping you choose the right tool for different scenarios.

---

## IaC Tools at a Glance

```
┌──────────────────────────────────────────────────────────┐
│  IaC TOOLS SPECTRUM                                      │
│                                                          │
│  Declarative / HCL          Declarative / YAML/JSON      │
│  ┌─────────────┐            ┌────────────────┐          │
│  │  Terraform  │            │ CloudFormation │          │
│  │ (multi-cloud)│           │  (AWS only)    │          │
│  └─────────────┘            └────────────────┘          │
│                                                          │
│  Declarative / Real Code    Declarative / K8s-native     │
│  ┌─────────────┐            ┌────────────────┐          │
│  │   Pulumi    │            │   Crossplane   │          │
│  │(Python/TS/Go)│           │(K8s CRDs/YAML) │          │
│  └─────────────┘            └────────────────┘          │
│                                                          │
│  CDK (AWS)                  Bicep (Azure)                │
│  ┌─────────────┐            ┌────────────────┐          │
│  │   AWS CDK   │            │     Bicep      │          │
│  │(TS/Python)  │            │  (Azure-native)│          │
│  └─────────────┘            └────────────────┘          │
└──────────────────────────────────────────────────────────┘
```

---

## Terraform (HashiCorp)

### When to Use Terraform

- Multi-cloud or hybrid cloud environments
- Team wants a domain-specific language (HCL)
- Large ecosystem of providers and modules needed
- Organization standardizing on one IaC tool

### Advanced Terraform Example

```hcl
# main.tf — EKS Cluster with Terraform

terraform {
  required_version = ">= 1.7"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Use a module for VPC
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.5.0"

  name = "${var.project}-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
}

# Use a module for EKS
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.0.0"

  cluster_name    = "${var.project}-cluster"
  cluster_version = "1.29"

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  eks_managed_node_groups = {
    general = {
      desired_size = 3
      min_size     = 2
      max_size     = 10
      instance_types = ["t3.medium"]
    }
  }
}

# Outputs
output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}
```

---

## AWS CloudFormation

### When to Use CloudFormation

- AWS-only environment
- Organization mandates AWS-native tooling
- Need integration with AWS Service Catalog
- Want managed state (no state file to manage)

### CloudFormation Example

```yaml
# template.yaml — CloudFormation Stack
AWSTemplateFormatVersion: '2010-09-09'
Description: Web Application Infrastructure

Parameters:
  Environment:
    Type: String
    Default: production
    AllowedValues: [development, staging, production]
  InstanceType:
    Type: String
    Default: t3.medium

Resources:
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web server security group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0

  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: ami-0c55b159cbfafe1f0
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub "${Environment}-web-server"
        - Key: Environment
          Value: !Ref Environment

Outputs:
  WebServerPublicIP:
    Description: Public IP of web server
    Value: !GetAtt WebServer.PublicIp
```

---

## Pulumi

### When to Use Pulumi

- Team prefers real programming languages over HCL/YAML
- Need loops, conditionals, and testing with familiar tools
- Want to use existing IDEs and package managers
- Building complex infrastructure with reusable components

### Pulumi Example (Python)

```python
# __main__.py — Pulumi program
import pulumi
import pulumi_aws as aws

config = pulumi.Config()
environment = config.get("environment") or "production"

# Create VPC
vpc = aws.ec2.Vpc(
    "main-vpc",
    cidr_block="10.0.0.0/16",
    enable_dns_hostnames=True,
    tags={"Name": f"{environment}-vpc", "Environment": environment}
)

# Create subnets using a loop (real Python!)
azs = ["us-east-1a", "us-east-1b"]
public_subnets = []
for i, az in enumerate(azs):
    subnet = aws.ec2.Subnet(
        f"public-subnet-{i}",
        vpc_id=vpc.id,
        cidr_block=f"10.0.{i+1}.0/24",
        availability_zone=az,
        map_public_ip_on_launch=True,
        tags={"Name": f"public-{az}"}
    )
    public_subnets.append(subnet)

# Create security group
sg = aws.ec2.SecurityGroup(
    "web-sg",
    vpc_id=vpc.id,
    ingress=[
        {"protocol": "tcp", "from_port": 80, "to_port": 80, "cidr_blocks": ["0.0.0.0/0"]},
        {"protocol": "tcp", "from_port": 443, "to_port": 443, "cidr_blocks": ["0.0.0.0/0"]},
    ],
    egress=[
        {"protocol": "-1", "from_port": 0, "to_port": 0, "cidr_blocks": ["0.0.0.0/0"]},
    ]
)

# Export outputs
pulumi.export("vpc_id", vpc.id)
pulumi.export("subnet_ids", [s.id for s in public_subnets])
```

---

## Head-to-Head Comparison

| Feature | Terraform | CloudFormation | Pulumi |
|---------|-----------|---------------|--------|
| **Language** | HCL | YAML/JSON | Python, TypeScript, Go, C# |
| **Cloud Support** | 3000+ providers | AWS only | Multi-cloud |
| **State Mgmt** | Self-managed (S3) | AWS-managed | Pulumi Cloud or self |
| **Learning Curve** | Medium (new language) | Medium | Low (existing language) |
| **Testing** | Terratest, tftest | CFN Lint, TaskCat | Standard unit tests |
| **Drift Detection** | `terraform plan` | Drift detection | `pulumi preview` |
| **Rollback** | Manual (reapply old) | Automatic | Manual |
| **Community** | Massive | Large (AWS) | Growing |
| **Cost** | Free (OSS) + Cloud paid | Free | Free (OSS) + Cloud paid |
| **Best For** | Multi-cloud standard | AWS shops | Developer-centric teams |

---

## Decision Framework

```
┌──────────────────────────────────────────────────────────┐
│  CHOOSING AN IaC TOOL                                    │
│                                                          │
│  "AWS only?"                                             │
│  ├── YES → CloudFormation (or CDK for code)              │
│  └── NO  → "Team prefers real languages?"                │
│            ├── YES → Pulumi                              │
│            └── NO  → Terraform (industry standard)       │
│                                                          │
│  Additional considerations:                              │
│  ├── Azure only? → Bicep                                 │
│  ├── K8s-centric? → Crossplane                           │
│  ├── Existing Terraform? → Stay with Terraform           │
│  └── Startup / small team? → Pulumi or Terraform         │
│                                                          │
│  When in doubt → Terraform                               │
│  (most job postings, biggest community, multi-cloud)     │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Multi-Cloud Strategy

```
COMPANY REQUIREMENT:
├── Primary infrastructure on AWS
├── DR (Disaster Recovery) region on GCP
├── CDN on Cloudflare
├── DNS on Route53 + Cloudflare
└── Need ONE tool to manage everything

SOLUTION: Terraform with multiple providers

terraform/
├── aws/
│   ├── production/
│   │   ├── main.tf        ← AWS EKS, RDS, S3
│   │   └── providers.tf   ← provider "aws" {}
│   └── dr/
│       ├── main.tf        ← DR resources
│       └── providers.tf
├── gcp/
│   └── dr/
│       ├── main.tf        ← GCP GKE, Cloud SQL
│       └── providers.tf   ← provider "google" {}
├── cloudflare/
│   ├── main.tf            ← CDN, WAF rules
│   └── providers.tf       ← provider "cloudflare" {}
└── modules/
    ├── k8s-cluster/       ← Works on AWS + GCP
    └── database/          ← Abstracted DB module

RESULT:
├── Single workflow: terraform plan/apply
├── State per environment (separate S3 buckets)
├── Consistent patterns across clouds
└── DR failover tested monthly with terraform apply
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Provider not found" | Missing `terraform init` | Run `terraform init` after adding new providers |
| Circular dependency | Resources referencing each other | Use `depends_on` or restructure resources |
| State lock error | Previous apply didn't release lock | Use `terraform force-unlock <lock-id>` carefully |
| CloudFormation rollback | Resource creation failed | Check Events tab; fix the failing resource; re-deploy |
| Pulumi stack conflict | Multiple users same stack | Use Pulumi Cloud for state locking |
| Quota exceeded | Too many resources for account | Request quota increase from cloud provider |

---

## Summary Table

| IaC Tool | Key Characteristics |
|----------|-------------------|
| **Terraform** | HCL, multi-cloud, largest ecosystem, state file management |
| **CloudFormation** | AWS-native, YAML/JSON, managed state, automatic rollback |
| **Pulumi** | Real programming languages, multi-cloud, developer-friendly |
| **AWS CDK** | TypeScript/Python generating CloudFormation, AWS only |
| **Bicep** | Azure-native DSL, compiles to ARM templates |
| **Crossplane** | Kubernetes-native IaC using CRDs and controllers |

---

## Quick Revision Questions

1. **When would you choose Terraform over CloudFormation?**
   <details><summary>Answer</summary>Choose Terraform when: 1) Using multiple cloud providers (AWS + GCP + Azure). 2) Want infrastructure portability. 3) Need non-AWS resources (Cloudflare, Datadog, GitHub). 4) Team wants a consistent workflow across clouds. Choose CloudFormation when AWS-only and you want managed state with automatic rollback.</details>

2. **What advantage does Pulumi have over Terraform?**
   <details><summary>Answer</summary>Pulumi uses real programming languages (Python, TypeScript, Go) instead of a domain-specific language (HCL). Benefits: 1) Use familiar IDEs and linters. 2) Write standard unit tests. 3) Use loops, conditionals, and abstractions naturally. 4) Import existing packages. 5) Lower learning curve for developers already familiar with those languages.</details>

3. **What is Terraform state, and why must it be carefully managed?**
   <details><summary>Answer</summary>Terraform state is a file that maps your configuration to real-world resources. It contains resource IDs, attributes, and sometimes secrets. Must be managed carefully because: 1) Lost state = Terraform loses track of resources. 2) Corrupted state = incorrect changes. 3) Contains sensitive data = must be encrypted. 4) Concurrent access = needs locking. Store in S3 with DynamoDB locking.</details>

4. **What is CloudFormation's advantage with automatic rollback?**
   <details><summary>Answer</summary>If a CloudFormation stack update fails, AWS automatically rolls back to the last known good state. All resources are reverted, no manual intervention needed. Terraform doesn't do this — if an apply fails halfway, you have a partially-applied state and must manually fix forward or revert.</details>

5. **What is a Terraform module, and why use modules?**
   <details><summary>Answer</summary>A module is a reusable package of Terraform configuration. Benefits: 1) DRY — define once, use across environments. 2) Encapsulation — hide complexity behind simple inputs/outputs. 3) Standardization — enforce organizational patterns. 4) Versioning — pin module versions for stability. Example: a "VPC module" that creates VPC, subnets, NAT gateways with one block of configuration.</details>

6. **How would you handle a multi-cloud infrastructure requirement?**
   <details><summary>Answer</summary>Use Terraform or Pulumi (both are multi-cloud). Strategy: 1) Create provider blocks for each cloud. 2) Build abstraction modules that work across providers. 3) Use separate state files per cloud/environment. 4) Define shared patterns (naming, tagging) across clouds. 5) Use workspaces or directory structure to organize. Terraform is the most common choice due to its massive provider ecosystem.</details>

---

[← Previous: Container Orchestration](04-container-orchestration.md) | [Next: Monitoring Tools →](06-monitoring-tools.md)

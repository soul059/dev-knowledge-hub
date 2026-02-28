# Chapter 3: Multi-Provider

[← Previous: Provider Configuration](02-provider-configuration.md) | [Back to README](../README.md) | [Next: Provider Aliases →](04-provider-aliases.md)

---

## Overview

Real-world infrastructure often spans **multiple cloud providers** or services. Terraform can use many providers simultaneously in a single configuration, enabling multi-cloud deployments, hybrid architectures, and integrated stacks.

---

## 3.1 Why Multiple Providers?

```
┌──────────────────────────────────────────────────────────┐
│          MULTI-PROVIDER USE CASES                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Multi-Cloud                                              │
│  ├── AWS for compute + GCP for ML/AI                     │
│  ├── Azure for apps + Cloudflare for CDN                 │
│  └── Avoid vendor lock-in                                │
│                                                           │
│  Supporting Services                                      │
│  ├── AWS + Datadog (monitoring)                          │
│  ├── AWS + PagerDuty (alerting)                          │
│  └── AWS + GitHub (repos, teams)                         │
│                                                           │
│  Infrastructure + Application                             │
│  ├── AWS + Kubernetes (deploy apps)                      │
│  ├── AWS + Helm (install charts)                         │
│  └── AWS + Docker (build images)                         │
│                                                           │
│  DNS + CDN                                                │
│  ├── AWS + Cloudflare                                    │
│  ├── AWS + Route53 + CloudFront                          │
│  └── Any cloud + external DNS provider                   │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Multi-Provider Declaration

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    cloudflare = {
      source  = "cloudflare/cloudflare"
      version = "~> 4.0"
    }
    datadog = {
      source  = "DataDog/datadog"
      version = "~> 3.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
}

# ─── Configure each provider ─────────────────────
provider "aws" {
  region = "us-east-1"
}

provider "azurerm" {
  features {}
  subscription_id = var.azure_subscription_id
}

provider "cloudflare" {
  api_token = var.cloudflare_api_token
}

provider "datadog" {
  api_key = var.datadog_api_key
  app_key = var.datadog_app_key
}
```

---

## 3.3 AWS + Cloudflare (DNS + CDN)

```hcl
# ─── AWS: Application infrastructure ─────────────
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.small"

  tags = {
    Name = "web-server"
  }
}

resource "aws_eip" "web" {
  instance = aws_instance.web.id
  domain   = "vpc"
}

# ─── Cloudflare: DNS + CDN ───────────────────────
resource "cloudflare_record" "web" {
  zone_id = var.cloudflare_zone_id
  name    = "app"
  content = aws_eip.web.public_ip    # Cross-provider reference!
  type    = "A"
  ttl     = 300
  proxied = true                      # Enable Cloudflare CDN
}

resource "cloudflare_page_rule" "https" {
  zone_id = var.cloudflare_zone_id
  target  = "*.example.com/*"
  actions {
    always_use_https = true
  }
}
```

```
┌──────────────────────────────────────────────────────────┐
│         CROSS-PROVIDER DATA FLOW                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  AWS Provider           Cloudflare Provider               │
│  ┌────────────┐         ┌──────────────────┐             │
│  │ aws_eip    │  ──────►│ cloudflare_record │             │
│  │  .public_ip│  output │  .content        │             │
│  └────────────┘         └──────────────────┘             │
│                                                           │
│  Terraform handles cross-provider dependencies           │
│  automatically — creates EIP first, then DNS record      │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.4 AWS + Kubernetes + Helm

```hcl
# ─── AWS: Create EKS Cluster ─────────────────────
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = "my-cluster"
  cluster_version = "1.28"
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets
}

# ─── Kubernetes: Configure using EKS credentials ─
provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)

  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
  }
}

# ─── Helm: Install charts on the cluster ─────────
provider "helm" {
  kubernetes {
    host                   = module.eks.cluster_endpoint
    cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)

    exec {
      api_version = "client.authentication.k8s.io/v1beta1"
      command     = "aws"
      args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
    }
  }
}

# ─── Deploy Kubernetes resources ──────────────────
resource "kubernetes_namespace" "app" {
  metadata {
    name = "myapp"
  }
}

resource "helm_release" "nginx_ingress" {
  name       = "ingress-nginx"
  repository = "https://kubernetes.github.io/ingress-nginx"
  chart      = "ingress-nginx"
  namespace  = kubernetes_namespace.app.metadata[0].name

  set {
    name  = "controller.replicaCount"
    value = "3"
  }
}
```

---

## 3.5 AWS + Datadog (Monitoring)

```hcl
# ─── AWS: Infrastructure ─────────────────────────
resource "aws_autoscaling_group" "app" {
  name                = "myapp-asg"
  min_size            = 2
  max_size            = 10
  desired_capacity    = 3
  vpc_zone_identifier = var.subnet_ids
  # ...
}

# ─── Datadog: Monitoring for AWS resources ────────
resource "datadog_monitor" "high_cpu" {
  name    = "High CPU on myapp ASG"
  type    = "metric alert"
  message = "CPU usage is above 80% on myapp. @pagerduty-myapp"
  query   = "avg(last_5m):avg:aws.ec2.cpuutilization{autoscaling_group:myapp-asg} > 80"

  monitor_thresholds {
    critical = 80
    warning  = 70
  }

  tags = ["env:${var.environment}", "service:myapp"]
}

resource "datadog_dashboard" "app" {
  title       = "MyApp Dashboard"
  description = "Application overview"
  layout_type = "ordered"

  widget {
    timeseries_definition {
      title = "CPU Utilization"
      request {
        q            = "avg:aws.ec2.cpuutilization{autoscaling_group:myapp-asg}"
        display_type = "line"
      }
    }
  }
}
```

---

## 3.6 Multi-Cloud Architecture

```hcl
# ─── AWS: Primary compute ────────────────────────
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "primary" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "m5.large"
  tags = { Name = "primary-app" }
}

# ─── Azure: DR/Failover ──────────────────────────
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "dr" {
  name     = "myapp-dr"
  location = "West Europe"
}

resource "azurerm_linux_virtual_machine" "dr" {
  name                = "dr-app"
  resource_group_name = azurerm_resource_group.dr.name
  location            = azurerm_resource_group.dr.location
  size                = "Standard_D2s_v3"
  # ...
}

# ─── Cloudflare: Global load balancing ────────────
provider "cloudflare" {
  api_token = var.cloudflare_token
}

resource "cloudflare_load_balancer" "app" {
  zone_id          = var.zone_id
  name             = "app.example.com"
  fallback_pool_id = cloudflare_load_balancer_pool.secondary.id
  default_pool_ids = [cloudflare_load_balancer_pool.primary.id]
}

resource "cloudflare_load_balancer_pool" "primary" {
  name = "aws-primary"
  origins {
    name    = "aws"
    address = aws_instance.primary.public_ip
    enabled = true
  }
}

resource "cloudflare_load_balancer_pool" "secondary" {
  name = "azure-dr"
  origins {
    name    = "azure"
    address = azurerm_linux_virtual_machine.dr.public_ip_address
    enabled = true
  }
}
```

```
┌──────────────────────────────────────────────────────────┐
│          MULTI-CLOUD ARCHITECTURE                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│              Cloudflare (Global LB + DNS)                 │
│              ┌──────────────────────┐                    │
│              │ app.example.com      │                    │
│              └──────┬───────┬───────┘                    │
│                     │       │                            │
│          Primary    │       │  Failover                  │
│              ┌──────▼──┐ ┌──▼──────────┐                │
│              │  AWS     │ │  Azure       │               │
│              │ us-east-1│ │ West Europe  │               │
│              │  (m5.lg) │ │ (D2s_v3)    │               │
│              └─────────┘ └──────────────┘                │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Cross-provider dependency fails | Provider B needs output from A | Ensure resource A is created first (implicit dependency) |
| Multiple provider auth failures | Missing credentials for one | Set env vars for each provider separately |
| Slow plan with many providers | Each provider refreshes state | Use `-target` or split into separate configs |
| Provider version conflict | Shared dependency versions | Check compatible version ranges |
| K8s provider fails with EKS | Cluster not ready yet | Use `depends_on` or separate apply steps |

---

## Summary Table

| Pattern | Providers | Use Case |
|---------|-----------|----------|
| **DNS + Cloud** | AWS + Cloudflare | External DNS management |
| **Cloud + K8s** | AWS + Kubernetes + Helm | EKS app deployment |
| **Cloud + Monitoring** | AWS + Datadog | Infrastructure monitoring |
| **Multi-Cloud** | AWS + Azure + Cloudflare | DR/failover across clouds |
| **Cloud + VCS** | AWS + GitHub | GitOps infrastructure |

---

## Quick Revision Questions

1. **Can Terraform use multiple providers in one configuration?**
   > Yes. You declare multiple providers in `required_providers` and configure each with its own `provider` block. Resources reference providers automatically by type prefix.

2. **How does Terraform handle dependencies across providers?**
   > Through implicit dependencies via resource references. If a Cloudflare DNS record references an AWS EIP, Terraform creates the EIP first automatically.

3. **Give an example of a multi-provider use case.**
   > AWS for compute infrastructure + Cloudflare for DNS/CDN + Datadog for monitoring — all managed in one Terraform configuration.

4. **What challenge arises with AWS + Kubernetes providers?**
   > The Kubernetes provider needs the EKS cluster endpoint and credentials, which aren't available until the cluster is created. This requires careful dependency management.

5. **Why might you split multi-provider configs into separate states?**
   > To reduce blast radius, speed up plan/apply times, and allow different teams to manage their provider-specific infrastructure independently.

---

[← Previous: Provider Configuration](02-provider-configuration.md) | [Back to README](../README.md) | [Next: Provider Aliases →](04-provider-aliases.md)

# Terraform - Comprehensive Study Notes

```
  ╔════════════════════════════════════════════════════════════════════╗
  ║                                                                    ║
  ║        ████████╗███████╗██████╗ ██████╗  █████╗ ███████╗ ██████╗  ║
  ║        ╚══██╔══╝██╔════╝██╔══██╗██╔══██╗██╔══██╗██╔════╝██╔═══██╗║
  ║           ██║   █████╗  ██████╔╝██████╔╝███████║█████╗  ██║   ██║║
  ║           ██║   ██╔══╝  ██╔══██╗██╔══██╗██╔══██║██╔══╝  ██║   ██║║
  ║           ██║   ███████╗██║  ██║██║  ██║██║  ██║██║     ╚██████╔╝║
  ║           ╚═╝   ╚══════╝╚═╝  ╚═╝╚═╝  ╚═╝╚═╝  ╚═╝╚═╝      ╚═════╝ ║
  ║                                                                    ║
  ║           Infrastructure as Code — Study Guide                     ║
  ╚════════════════════════════════════════════════════════════════════╝
```

## Course Overview

This comprehensive study guide covers **HashiCorp Terraform** from foundational Infrastructure as Code (IaC) concepts through advanced real-world deployment patterns. It is structured into **10 units** with clear explanations, ASCII architecture diagrams, hands-on configuration examples, best practices, and revision questions.

### Who Is This For?

- DevOps Engineers looking to master Terraform
- Cloud Engineers adopting Infrastructure as Code
- Developers preparing for the **HashiCorp Terraform Associate** certification
- Anyone managing infrastructure on AWS, Azure, or GCP

### How to Use These Notes

1. Follow units **sequentially** for the best learning experience
2. Practice every configuration example in a **sandbox environment**
3. Use the **Quick Revision Questions** at the end of each chapter to test yourself
4. Refer to the **Summary Tables** for rapid review before exams

---

## Complete Table of Contents

### Unit 1: Infrastructure as Code Concepts
| # | Chapter | File |
|---|---------|------|
| 1 | What is IaC? | [01-what-is-iac.md](01-IaC-Concepts/01-what-is-iac.md) |
| 2 | IaC Benefits | [02-iac-benefits.md](01-IaC-Concepts/02-iac-benefits.md) |
| 3 | Declarative vs Imperative | [03-declarative-vs-imperative.md](01-IaC-Concepts/03-declarative-vs-imperative.md) |
| 4 | IaC Tools Comparison | [04-iac-tools-comparison.md](01-IaC-Concepts/04-iac-tools-comparison.md) |
| 5 | Terraform Overview | [05-terraform-overview.md](01-IaC-Concepts/05-terraform-overview.md) |

### Unit 2: Terraform Basics
| # | Chapter | File |
|---|---------|------|
| 1 | Installation and Setup | [01-installation-and-setup.md](02-Terraform-Basics/01-installation-and-setup.md) |
| 2 | Provider Configuration | [02-provider-configuration.md](02-Terraform-Basics/02-provider-configuration.md) |
| 3 | HCL Syntax Basics | [03-hcl-syntax-basics.md](02-Terraform-Basics/03-hcl-syntax-basics.md) |
| 4 | Resources and Data Sources | [04-resources-and-data-sources.md](02-Terraform-Basics/04-resources-and-data-sources.md) |
| 5 | Terraform Workflow | [05-terraform-workflow.md](02-Terraform-Basics/05-terraform-workflow.md) |

### Unit 3: Terraform Language
| # | Chapter | File |
|---|---------|------|
| 1 | Variables (Input Variables) | [01-variables.md](03-Terraform-Language/01-variables.md) |
| 2 | Outputs | [02-outputs.md](03-Terraform-Language/02-outputs.md) |
| 3 | Local Values | [03-local-values.md](03-Terraform-Language/03-local-values.md) |
| 4 | Data Types | [04-data-types.md](03-Terraform-Language/04-data-types.md) |
| 5 | Expressions and Functions | [05-expressions-and-functions.md](03-Terraform-Language/05-expressions-and-functions.md) |
| 6 | Dynamic Blocks | [06-dynamic-blocks.md](03-Terraform-Language/06-dynamic-blocks.md) |
| 7 | Conditionals and Loops | [07-conditionals-and-loops.md](03-Terraform-Language/07-conditionals-and-loops.md) |

### Unit 4: State Management
| # | Chapter | File |
|---|---------|------|
| 1 | What is Terraform State? | [01-what-is-terraform-state.md](04-State-Management/01-what-is-terraform-state.md) |
| 2 | Local vs Remote State | [02-local-vs-remote-state.md](04-State-Management/02-local-vs-remote-state.md) |
| 3 | Backend Configuration | [03-backend-configuration.md](04-State-Management/03-backend-configuration.md) |
| 4 | State Locking | [04-state-locking.md](04-State-Management/04-state-locking.md) |
| 5 | State Commands | [05-state-commands.md](04-State-Management/05-state-commands.md) |
| 6 | Importing Existing Resources | [06-importing-existing-resources.md](04-State-Management/06-importing-existing-resources.md) |
| 7 | State File Security | [07-state-file-security.md](04-State-Management/07-state-file-security.md) |

### Unit 5: Modules
| # | Chapter | File |
|---|---------|------|
| 1 | Module Concepts | [01-module-concepts.md](05-Modules/01-module-concepts.md) |
| 2 | Creating Modules | [02-creating-modules.md](05-Modules/02-creating-modules.md) |
| 3 | Using Modules | [03-using-modules.md](05-Modules/03-using-modules.md) |
| 4 | Module Sources | [04-module-sources.md](05-Modules/04-module-sources.md) |
| 5 | Module Versioning | [05-module-versioning.md](05-Modules/05-module-versioning.md) |
| 6 | Module Best Practices | [06-module-best-practices.md](05-Modules/06-module-best-practices.md) |
| 7 | Public Module Registry | [07-public-module-registry.md](05-Modules/07-public-module-registry.md) |

### Unit 6: Workspaces and Environments
| # | Chapter | File |
|---|---------|------|
| 1 | Workspace Concepts | [01-workspace-concepts.md](06-Workspaces-and-Environments/01-workspace-concepts.md) |
| 2 | Managing Multiple Environments | [02-managing-multiple-environments.md](06-Workspaces-and-Environments/02-managing-multiple-environments.md) |
| 3 | Environment Separation Strategies | [03-environment-separation-strategies.md](06-Workspaces-and-Environments/03-environment-separation-strategies.md) |
| 4 | Variable Files per Environment | [04-variable-files-per-environment.md](06-Workspaces-and-Environments/04-variable-files-per-environment.md) |

### Unit 7: Providers
| # | Chapter | File |
|---|---------|------|
| 1 | Provider Configuration | [01-provider-configuration.md](07-Providers/01-provider-configuration.md) |
| 2 | Provider Versioning | [02-provider-versioning.md](07-Providers/02-provider-versioning.md) |
| 3 | Multiple Provider Instances | [03-multiple-provider-instances.md](07-Providers/03-multiple-provider-instances.md) |
| 4 | Provider Aliases | [04-provider-aliases.md](07-Providers/04-provider-aliases.md) |
| 5 | Custom Providers | [05-custom-providers.md](07-Providers/05-custom-providers.md) |

### Unit 8: Advanced Features
| # | Chapter | File |
|---|---------|------|
| 1 | Provisioners | [01-provisioners.md](08-Advanced-Features/01-provisioners.md) |
| 2 | Terraform Cloud/Enterprise | [02-terraform-cloud-enterprise.md](08-Advanced-Features/02-terraform-cloud-enterprise.md) |
| 3 | Sentinel Policies | [03-sentinel-policies.md](08-Advanced-Features/03-sentinel-policies.md) |
| 4 | Remote Operations | [04-remote-operations.md](08-Advanced-Features/04-remote-operations.md) |
| 5 | State Manipulation | [05-state-manipulation.md](08-Advanced-Features/05-state-manipulation.md) |
| 6 | Terraform CDK | [06-terraform-cdk.md](08-Advanced-Features/06-terraform-cdk.md) |

### Unit 9: Best Practices
| # | Chapter | File |
|---|---------|------|
| 1 | Code Organization | [01-code-organization.md](09-Best-Practices/01-code-organization.md) |
| 2 | Naming Conventions | [02-naming-conventions.md](09-Best-Practices/02-naming-conventions.md) |
| 3 | Module Structure | [03-module-structure.md](09-Best-Practices/03-module-structure.md) |
| 4 | Version Control Integration | [04-version-control-integration.md](09-Best-Practices/04-version-control-integration.md) |
| 5 | CI/CD for Terraform | [05-cicd-for-terraform.md](09-Best-Practices/05-cicd-for-terraform.md) |
| 6 | Security Best Practices | [06-security-best-practices.md](09-Best-Practices/06-security-best-practices.md) |
| 7 | Code Review Guidelines | [07-code-review-guidelines.md](09-Best-Practices/07-code-review-guidelines.md) |

### Unit 10: Real-World Patterns
| # | Chapter | File |
|---|---------|------|
| 1 | VPC and Networking | [01-vpc-and-networking.md](10-Real-World-Patterns/01-vpc-and-networking.md) |
| 2 | Compute Resources (EC2, VMs) | [02-compute-resources.md](10-Real-World-Patterns/02-compute-resources.md) |
| 3 | Managed Services (RDS, S3) | [03-managed-services.md](10-Real-World-Patterns/03-managed-services.md) |
| 4 | Kubernetes Clusters | [04-kubernetes-clusters.md](10-Real-World-Patterns/04-kubernetes-clusters.md) |
| 5 | Multi-Region Deployments | [05-multi-region-deployments.md](10-Real-World-Patterns/05-multi-region-deployments.md) |
| 6 | Disaster Recovery Setup | [06-disaster-recovery-setup.md](10-Real-World-Patterns/06-disaster-recovery-setup.md) |

---

## Quick Reference

### Essential Terraform Commands

```bash
terraform init          # Initialize working directory
terraform plan          # Preview changes
terraform apply         # Apply changes
terraform destroy       # Destroy all resources
terraform fmt           # Format code
terraform validate      # Validate configuration
terraform state list    # List resources in state
terraform import        # Import existing resource
terraform output        # Show output values
terraform workspace     # Manage workspaces
```

### Key Concepts Map

```
┌─────────────────────────────────────────────────────────────────┐
│                    TERRAFORM ECOSYSTEM                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐                  │
│   │   HCL    │───▶│ Providers│───▶│ Resources│                  │
│   │  Config  │    │ (plugins)│    │ (infra)  │                  │
│   └──────────┘    └──────────┘    └──────────┘                  │
│        │                               │                         │
│        ▼                               ▼                         │
│   ┌──────────┐                   ┌──────────┐                   │
│   │ Modules  │                   │  State   │                   │
│   │(reusable)│                   │  File    │                   │
│   └──────────┘                   └──────────┘                   │
│        │                               │                         │
│        ▼                               ▼                         │
│   ┌──────────┐                   ┌──────────┐                   │
│   │ Registry │                   │ Backends │                   │
│   │ (share)  │                   │ (storage)│                   │
│   └──────────┘                   └──────────┘                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Resources

- [Official Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [HashiCorp Learn](https://developer.hashicorp.com/terraform/tutorials)
- [Terraform GitHub Repository](https://github.com/hashicorp/terraform)
- [Terraform Associate Certification](https://www.hashicorp.com/certification/terraform-associate)

---

> **Start your journey:** [Unit 1 — What is IaC?](01-IaC-Concepts/01-what-is-iac.md)

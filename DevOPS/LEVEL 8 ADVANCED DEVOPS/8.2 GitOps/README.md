# 🚀 GitOps — Comprehensive Study Notes

> **Level 8 — Advanced DevOps | Module 8.2**
> Complete guide to GitOps concepts, tools, workflows, and best practices.

---

## Course Overview

GitOps is a modern operational framework that takes DevOps best practices — version control, collaboration, compliance, and CI/CD — and applies them to infrastructure automation. In GitOps, the entire system is described declaratively, Git is the single source of truth, and automated agents ensure the actual state matches the desired state.

This course covers GitOps from foundational concepts through advanced tooling (ArgoCD, Flux CD), real-world workflows, and production best practices.

### What You'll Learn

- Core GitOps principles and how they differ from traditional CI/CD
- Push vs Pull deployment models
- Using Git as the single source of truth for infrastructure and applications
- Hands-on with **ArgoCD** and **Flux CD**
- Designing GitOps workflows for real-world environments
- Security, secrets management, drift detection, and self-healing
- Best practices for repository organization, testing, and monitoring

### Prerequisites

- Solid understanding of Git (branching, merging, PRs)
- Kubernetes fundamentals (Pods, Deployments, Services, Namespaces)
- Basic CI/CD concepts
- Familiarity with YAML and Helm charts

---

## Complete Table of Contents

### Unit 1: GitOps Concepts
| # | Chapter | File |
|---|---------|------|
| 1.1 | What is GitOps? | [01-what-is-gitops.md](01-GitOps-Concepts/01-what-is-gitops.md) |
| 1.2 | GitOps Principles | [02-gitops-principles.md](01-GitOps-Concepts/02-gitops-principles.md) |
| 1.3 | Push vs Pull Model | [03-push-vs-pull-model.md](01-GitOps-Concepts/03-push-vs-pull-model.md) |
| 1.4 | Desired State vs Actual State | [04-desired-vs-actual-state.md](01-GitOps-Concepts/04-desired-vs-actual-state.md) |
| 1.5 | GitOps Benefits | [05-gitops-benefits.md](01-GitOps-Concepts/05-gitops-benefits.md) |

### Unit 2: Git as Single Source of Truth
| # | Chapter | File |
|---|---------|------|
| 2.1 | Repository Structure | [01-repository-structure.md](02-Git-Source-Of-Truth/01-repository-structure.md) |
| 2.2 | Branch Strategies | [02-branch-strategies.md](02-Git-Source-Of-Truth/02-branch-strategies.md) |
| 2.3 | Change Management | [03-change-management.md](02-Git-Source-Of-Truth/03-change-management.md) |
| 2.4 | Audit and Compliance | [04-audit-and-compliance.md](02-Git-Source-Of-Truth/04-audit-and-compliance.md) |
| 2.5 | Version Control Best Practices | [05-version-control-best-practices.md](02-Git-Source-Of-Truth/05-version-control-best-practices.md) |

### Unit 3: ArgoCD
| # | Chapter | File |
|---|---------|------|
| 3.1 | ArgoCD Architecture | [01-argocd-architecture.md](03-ArgoCD/01-argocd-architecture.md) |
| 3.2 | Installation | [02-argocd-installation.md](03-ArgoCD/02-argocd-installation.md) |
| 3.3 | Application Definitions | [03-application-definitions.md](03-ArgoCD/03-application-definitions.md) |
| 3.4 | Sync Policies | [04-sync-policies.md](03-ArgoCD/04-sync-policies.md) |
| 3.5 | Rollouts and Rollbacks | [05-rollouts-and-rollbacks.md](03-ArgoCD/05-rollouts-and-rollbacks.md) |
| 3.6 | Multi-Cluster Management | [06-multi-cluster-management.md](03-ArgoCD/06-multi-cluster-management.md) |
| 3.7 | RBAC in ArgoCD | [07-rbac-in-argocd.md](03-ArgoCD/07-rbac-in-argocd.md) |
| 3.8 | ArgoCD with Helm | [08-argocd-with-helm.md](03-ArgoCD/08-argocd-with-helm.md) |

### Unit 4: Flux CD
| # | Chapter | File |
|---|---------|------|
| 4.1 | Flux Architecture | [01-flux-architecture.md](04-FluxCD/01-flux-architecture.md) |
| 4.2 | Installation | [02-flux-installation.md](04-FluxCD/02-flux-installation.md) |
| 4.3 | GitRepository and Kustomization | [03-gitrepository-and-kustomization.md](04-FluxCD/03-gitrepository-and-kustomization.md) |
| 4.4 | HelmRelease | [04-helmrelease.md](04-FluxCD/04-helmrelease.md) |
| 4.5 | Image Automation | [05-image-automation.md](04-FluxCD/05-image-automation.md) |
| 4.6 | Multi-Tenancy | [06-multi-tenancy.md](04-FluxCD/06-multi-tenancy.md) |

### Unit 5: GitOps Workflows
| # | Chapter | File |
|---|---------|------|
| 5.1 | Application Deployment | [01-application-deployment.md](05-GitOps-Workflows/01-application-deployment.md) |
| 5.2 | Infrastructure Management | [02-infrastructure-management.md](05-GitOps-Workflows/02-infrastructure-management.md) |
| 5.3 | Environment Promotion | [03-environment-promotion.md](05-GitOps-Workflows/03-environment-promotion.md) |
| 5.4 | Secrets Management | [04-secrets-management.md](05-GitOps-Workflows/04-secrets-management.md) |
| 5.5 | Drift Detection | [05-drift-detection.md](05-GitOps-Workflows/05-drift-detection.md) |
| 5.6 | Self-Healing | [06-self-healing.md](05-GitOps-Workflows/06-self-healing.md) |

### Unit 6: GitOps Best Practices
| # | Chapter | File |
|---|---------|------|
| 6.1 | Repository Organization | [01-repository-organization.md](06-GitOps-Best-Practices/01-repository-organization.md) |
| 6.2 | Branch Protection | [02-branch-protection.md](06-GitOps-Best-Practices/02-branch-protection.md) |
| 6.3 | PR Workflows | [03-pr-workflows.md](06-GitOps-Best-Practices/03-pr-workflows.md) |
| 6.4 | Testing GitOps | [04-testing-gitops.md](06-GitOps-Best-Practices/04-testing-gitops.md) |
| 6.5 | Security Considerations | [05-security-considerations.md](06-GitOps-Best-Practices/05-security-considerations.md) |
| 6.6 | Monitoring GitOps | [06-monitoring-gitops.md](06-GitOps-Best-Practices/06-monitoring-gitops.md) |

---

## How to Use These Notes

1. **Sequential Study** — Follow the units in order for a structured learning path
2. **Reference** — Jump to any chapter using the links above
3. **Quick Revision** — Use the summary tables and revision questions at the end of each chapter
4. **Hands-On** — Follow the configuration examples and commands in a lab environment

---

## Architecture Overview — The GitOps Model

```
 ┌─────────────────────────────────────────────────────────────────┐
 │                      GITOPS WORKFLOW                            │
 │                                                                 │
 │  ┌──────────┐    ┌──────────┐    ┌──────────────┐    ┌───────┐ │
 │  │Developer │───>│  Git     │───>│ GitOps Agent │───>│Cluster│ │
 │  │  Push    │    │  Repo    │    │ (ArgoCD/Flux)│    │  K8s  │ │
 │  └──────────┘    └────┬─────┘    └──────┬───────┘    └───┬───┘ │
 │                       │                 │                │     │
 │                       │    ┌─────────┐  │   Reconcile    │     │
 │                       └───>│ Desired │<─┘   Loop         │     │
 │                            │  State  │<──────────────────┘     │
 │                            └─────────┘  Compare & Sync         │
 └─────────────────────────────────────────────────────────────────┘
```

---

## Key Tools Covered

| Tool | Purpose | Website |
|------|---------|---------|
| **ArgoCD** | Kubernetes-native GitOps CD tool | argoproj.github.io |
| **Flux CD** | GitOps toolkit for Kubernetes | fluxcd.io |
| **Kustomize** | Kubernetes manifest customization | kustomize.io |
| **Helm** | Kubernetes package manager | helm.sh |
| **Sealed Secrets** | Encrypted secrets for Git | github.com/bitnami-labs/sealed-secrets |
| **SOPS** | Secrets OPerationS | github.com/getsops/sops |

---

*Begin your journey → [Unit 1: What is GitOps?](01-GitOps-Concepts/01-what-is-gitops.md)*

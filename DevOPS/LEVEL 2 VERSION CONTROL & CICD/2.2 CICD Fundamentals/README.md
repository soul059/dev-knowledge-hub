# CI/CD Fundamentals — Comprehensive Study Notes

> **Course Level:** Intermediate | **Prerequisites:** Basic Git, Linux CLI, Software Development Basics  
> **Last Updated:** February 2026

---

## Course Overview

**Continuous Integration / Continuous Delivery (CI/CD)** is the backbone of modern software delivery. This course covers the complete CI/CD landscape — from foundational concepts and pipeline design through build automation, testing strategies, and deployment patterns — culminating in hands-on coverage of the most widely used CI/CD platforms: **Jenkins**, **GitHub Actions**, **GitLab CI/CD**, and more.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        CI/CD FUNDAMENTALS                              │
│                                                                         │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│   │  COMMIT  │─▶│  BUILD   │─▶│  TEST    │─▶│  DEPLOY  │              │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘              │
│        │             │             │              │                     │
│        ▼             ▼             ▼              ▼                     │
│   Version Ctrl   Automation    Quality Gate   Production               │
│                                                                         │
│   Units 1-2       Unit 3        Unit 4        Unit 5                   │
│   Concepts &      Build         Testing       Deployment               │
│   Pipeline        Automation    Strategies    Strategies               │
│   Design                                                               │
│                                                                         │
│   Units 6-9: Jenkins | GitHub Actions | GitLab CI | Other Tools        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Complete Table of Contents

### Unit 1: CI/CD Concepts
| # | Chapter | File |
|---|---------|------|
| 1.1 | What is Continuous Integration? | [01-continuous-integration.md](01-CICD-Concepts/01-continuous-integration.md) |
| 1.2 | What is Continuous Delivery? | [02-continuous-delivery.md](01-CICD-Concepts/02-continuous-delivery.md) |
| 1.3 | What is Continuous Deployment? | [03-continuous-deployment.md](01-CICD-Concepts/03-continuous-deployment.md) |
| 1.4 | CI/CD Benefits | [04-cicd-benefits.md](01-CICD-Concepts/04-cicd-benefits.md) |
| 1.5 | CI/CD Pipeline Components | [05-pipeline-components.md](01-CICD-Concepts/05-pipeline-components.md) |

### Unit 2: CI/CD Pipeline Design
| # | Chapter | File |
|---|---------|------|
| 2.1 | Pipeline Stages (Build, Test, Deploy) | [01-pipeline-stages.md](02-Pipeline-Design/01-pipeline-stages.md) |
| 2.2 | Pipeline as Code | [02-pipeline-as-code.md](02-Pipeline-Design/02-pipeline-as-code.md) |
| 2.3 | Artifact Management | [03-artifact-management.md](02-Pipeline-Design/03-artifact-management.md) |
| 2.4 | Environment Promotion | [04-environment-promotion.md](02-Pipeline-Design/04-environment-promotion.md) |
| 2.5 | Pipeline Triggers | [05-pipeline-triggers.md](02-Pipeline-Design/05-pipeline-triggers.md) |
| 2.6 | Parallelization and Caching | [06-parallelization-caching.md](02-Pipeline-Design/06-parallelization-caching.md) |

### Unit 3: Build Automation
| # | Chapter | File |
|---|---------|------|
| 3.1 | Build Tools Overview | [01-build-tools-overview.md](03-Build-Automation/01-build-tools-overview.md) |
| 3.2 | Compilation and Packaging | [02-compilation-packaging.md](03-Build-Automation/02-compilation-packaging.md) |
| 3.3 | Dependency Management | [03-dependency-management.md](03-Build-Automation/03-dependency-management.md) |
| 3.4 | Build Optimization | [04-build-optimization.md](03-Build-Automation/04-build-optimization.md) |
| 3.5 | Multi-Stage Builds | [05-multi-stage-builds.md](03-Build-Automation/05-multi-stage-builds.md) |
| 3.6 | Build Caching Strategies | [06-build-caching-strategies.md](03-Build-Automation/06-build-caching-strategies.md) |

### Unit 4: Testing in CI/CD
| # | Chapter | File |
|---|---------|------|
| 4.1 | Test Pyramid | [01-test-pyramid.md](04-Testing-In-CICD/01-test-pyramid.md) |
| 4.2 | Unit Testing | [02-unit-testing.md](04-Testing-In-CICD/02-unit-testing.md) |
| 4.3 | Integration Testing | [03-integration-testing.md](04-Testing-In-CICD/03-integration-testing.md) |
| 4.4 | End-to-End Testing | [04-end-to-end-testing.md](04-Testing-In-CICD/04-end-to-end-testing.md) |
| 4.5 | Performance Testing | [05-performance-testing.md](04-Testing-In-CICD/05-performance-testing.md) |
| 4.6 | Security Testing (SAST, DAST) | [06-security-testing.md](04-Testing-In-CICD/06-security-testing.md) |
| 4.7 | Test Automation Best Practices | [07-test-automation-best-practices.md](04-Testing-In-CICD/07-test-automation-best-practices.md) |

### Unit 5: Deployment Strategies
| # | Chapter | File |
|---|---------|------|
| 5.1 | Rolling Deployment | [01-rolling-deployment.md](05-Deployment-Strategies/01-rolling-deployment.md) |
| 5.2 | Blue-Green Deployment | [02-blue-green-deployment.md](05-Deployment-Strategies/02-blue-green-deployment.md) |
| 5.3 | Canary Deployment | [03-canary-deployment.md](05-Deployment-Strategies/03-canary-deployment.md) |
| 5.4 | A/B Testing | [04-ab-testing.md](05-Deployment-Strategies/04-ab-testing.md) |
| 5.5 | Feature Flags | [05-feature-flags.md](05-Deployment-Strategies/05-feature-flags.md) |
| 5.6 | Rollback Strategies | [06-rollback-strategies.md](05-Deployment-Strategies/06-rollback-strategies.md) |
| 5.7 | Zero-Downtime Deployments | [07-zero-downtime-deployments.md](05-Deployment-Strategies/07-zero-downtime-deployments.md) |

### Unit 6: Jenkins
| # | Chapter | File |
|---|---------|------|
| 6.1 | Jenkins Architecture | [01-jenkins-architecture.md](06-Jenkins/01-jenkins-architecture.md) |
| 6.2 | Installation and Configuration | [02-installation-configuration.md](06-Jenkins/02-installation-configuration.md) |
| 6.3 | Freestyle vs Pipeline Jobs | [03-freestyle-vs-pipeline.md](06-Jenkins/03-freestyle-vs-pipeline.md) |
| 6.4 | Jenkinsfile (Declarative vs Scripted) | [04-jenkinsfile.md](06-Jenkins/04-jenkinsfile.md) |
| 6.5 | Shared Libraries | [05-shared-libraries.md](06-Jenkins/05-shared-libraries.md) |
| 6.6 | Plugins Ecosystem | [06-plugins-ecosystem.md](06-Jenkins/06-plugins-ecosystem.md) |
| 6.7 | Jenkins Agents and Scaling | [07-agents-and-scaling.md](06-Jenkins/07-agents-and-scaling.md) |

### Unit 7: GitHub Actions
| # | Chapter | File |
|---|---------|------|
| 7.1 | Actions Concepts | [01-actions-concepts.md](07-GitHub-Actions/01-actions-concepts.md) |
| 7.2 | Workflow Syntax | [02-workflow-syntax.md](07-GitHub-Actions/02-workflow-syntax.md) |
| 7.3 | Events and Triggers | [03-events-and-triggers.md](07-GitHub-Actions/03-events-and-triggers.md) |
| 7.4 | Jobs and Steps | [04-jobs-and-steps.md](07-GitHub-Actions/04-jobs-and-steps.md) |
| 7.5 | Actions Marketplace | [05-actions-marketplace.md](07-GitHub-Actions/05-actions-marketplace.md) |
| 7.6 | Secrets Management | [06-secrets-management.md](07-GitHub-Actions/06-secrets-management.md) |
| 7.7 | Self-Hosted Runners | [07-self-hosted-runners.md](07-GitHub-Actions/07-self-hosted-runners.md) |
| 7.8 | Matrix Builds | [08-matrix-builds.md](07-GitHub-Actions/08-matrix-builds.md) |

### Unit 8: GitLab CI/CD
| # | Chapter | File |
|---|---------|------|
| 8.1 | GitLab CI Concepts | [01-gitlab-ci-concepts.md](08-GitLab-CICD/01-gitlab-ci-concepts.md) |
| 8.2 | .gitlab-ci.yml Structure | [02-gitlab-ci-yml-structure.md](08-GitLab-CICD/02-gitlab-ci-yml-structure.md) |
| 8.3 | Stages and Jobs | [03-stages-and-jobs.md](08-GitLab-CICD/03-stages-and-jobs.md) |
| 8.4 | Variables and Secrets | [04-variables-and-secrets.md](08-GitLab-CICD/04-variables-and-secrets.md) |
| 8.5 | Runners Configuration | [05-runners-configuration.md](08-GitLab-CICD/05-runners-configuration.md) |
| 8.6 | Artifacts and Caching | [06-artifacts-and-caching.md](08-GitLab-CICD/06-artifacts-and-caching.md) |
| 8.7 | Environments and Deployments | [07-environments-and-deployments.md](08-GitLab-CICD/07-environments-and-deployments.md) |
| 8.8 | GitLab DevOps Platform | [08-gitlab-devops-platform.md](08-GitLab-CICD/08-gitlab-devops-platform.md) |

### Unit 9: Other CI/CD Tools
| # | Chapter | File |
|---|---------|------|
| 9.1 | Azure DevOps Pipelines | [01-azure-devops-pipelines.md](09-Other-CICD-Tools/01-azure-devops-pipelines.md) |
| 9.2 | CircleCI | [02-circleci.md](09-Other-CICD-Tools/02-circleci.md) |
| 9.3 | Travis CI | [03-travis-ci.md](09-Other-CICD-Tools/03-travis-ci.md) |
| 9.4 | ArgoCD | [04-argocd.md](09-Other-CICD-Tools/04-argocd.md) |
| 9.5 | Tekton | [05-tekton.md](09-Other-CICD-Tools/05-tekton.md) |
| 9.6 | Tool Comparison and Selection | [06-tool-comparison-selection.md](09-Other-CICD-Tools/06-tool-comparison-selection.md) |

---

## How to Use These Notes

1. **Sequential Study:** Follow units 1 → 9 for a structured learning path
2. **Quick Reference:** Jump to any specific chapter using the TOC above
3. **Revision:** Use the summary tables and quick revision questions at the end of each chapter
4. **Hands-On:** Follow the examples and commands in each chapter using your own lab environment

## Key Symbols Used

| Symbol | Meaning |
|--------|---------|
| 💡 | Key Concept / Tip |
| ⚠️ | Warning / Common Pitfall |
| 🔧 | Hands-On / Command |
| 📋 | Best Practice |
| 🔄 | Relates to Another Topic |

---

> **Next:** [Unit 1 — CI/CD Concepts →](01-CICD-Concepts/01-continuous-integration.md)

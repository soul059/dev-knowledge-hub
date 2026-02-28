# Chapter 9.5: CI/CD Tool Comparison Matrix

[← Previous: Tekton](04-tekton.md) | [Back to README](../README.md) | [Next: Choosing a CI/CD Tool →](06-choosing-a-tool.md)

---

## Overview

This chapter provides a comprehensive feature comparison of the major CI/CD tools covered in this course: **Jenkins**, **GitHub Actions**, **GitLab CI/CD**, **Azure Pipelines**, **CircleCI**, **ArgoCD**, and **Tekton**.

---

## Platform Comparison

```
  HOSTING MODELS
  ══════════════

  Cloud-Only:       GitHub Actions, CircleCI
  Cloud + Self:     GitLab CI, Azure Pipelines
  Self-Hosted Only: Jenkins, Tekton, ArgoCD

  ┌─────────────┬─────────┬──────────┬──────────────┐
  │ Tool        │ Cloud   │ Self-Host│ K8s-Native   │
  ├─────────────┼─────────┼──────────┼──────────────┤
  │ Jenkins     │    ✗    │    ✓     │ Via plugins  │
  │ GitHub Act. │    ✓    │    ✓*    │ Via ARC      │
  │ GitLab CI   │    ✓    │    ✓     │ Via runners  │
  │ Azure Pipes │    ✓    │    ✓     │ Via Scale Set│
  │ CircleCI    │    ✓    │    ✓*    │ Runner agent │
  │ ArgoCD      │    ✗    │    ✓     │ Native       │
  │ Tekton      │    ✗    │    ✓     │ Native       │
  └─────────────┴─────────┴──────────┴──────────────┘

  * = Self-hosted runners, managed by platform
```

---

## Feature Matrix

```
  ┌────────────────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐
  │ Feature            │ Jenkins │ GH Act. │ GitLab  │ Azure   │ CircleCI│ ArgoCD  │ Tekton  │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Config Format      │Groovy/  │ YAML    │ YAML    │ YAML    │ YAML    │ YAML    │ K8s YAML│
  │                    │YAML     │         │         │         │         │ (CRDs)  │ (CRDs)  │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Pipeline-as-Code   │Jenkinsf.│workflow/│.gitlab- │azure-   │.circle  │App CRD  │Pipeline │
  │                    │         │*.yml    │ci.yml   │pipe.yml │ci/conf  │         │CRD      │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Reusable Config    │Shared   │Reusable │include/ │Templates│Orbs     │App of   │Catalog  │
  │                    │Library  │Workflow │extends  │         │         │Apps     │Tasks    │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Parallelism        │ ✓       │ Matrix  │ parallel│ Matrix  │parallel │ N/A     │ runAfter│
  │                    │         │ strategy│ keyword │ strategy│ keyword │         │         │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Built-in Security  │ Plugins │ Actions │ Native  │ Tasks   │ Orbs    │ RBAC    │ K8s RBAC│
  │ Scanning           │         │         │ SAST/   │         │         │         │         │
  │                    │         │         │ DAST    │         │         │         │         │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Container Registry │ Plugin  │ GHCR    │ Native  │ ACR     │ Orbs    │ Any     │ Any     │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Secret Management  │Credent. │Secrets +│Variables│Variable │Contexts │K8s      │K8s      │
  │                    │Store    │OIDC     │+ Vault  │Groups   │         │Secrets  │Secrets  │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ GitOps Support     │ Plugin  │ Via CD  │ Agent   │ Via CD  │ Via CD  │ Native  │ Via CD  │
  ├────────────────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┼─────────┤
  │ Cost Model         │ Free    │Freemium │Freemium │Freemium │Freemium │ Free    │ Free    │
  │                    │(self)   │+minutes │+minutes │+minutes │+minutes │ (self)  │ (self)  │
  └────────────────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘
```

---

## Configuration Syntax Comparison

### Simple Build + Test + Deploy

```yaml
# GITHUB ACTIONS
name: CI/CD
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - run: npm test
      - run: ./deploy.sh
```

```yaml
# GITLAB CI
stages: [build, test, deploy]
build:
  stage: build
  script: npm ci && npm run build
test:
  stage: test
  script: npm test
deploy:
  stage: deploy
  script: ./deploy.sh
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

```yaml
# AZURE PIPELINES
trigger:
  branches:
    include: [main]
pool:
  vmImage: ubuntu-latest
steps:
  - script: npm ci && npm run build
  - script: npm test
  - script: ./deploy.sh
```

```yaml
# CIRCLECI
version: 2.1
jobs:
  build-test-deploy:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm ci && npm run build
      - run: npm test
      - run: ./deploy.sh
workflows:
  main:
    jobs:
      - build-test-deploy
```

```groovy
// JENKINS (Declarative)
pipeline {
    agent { docker { image 'node:20' } }
    stages {
        stage('Build') {
            steps { sh 'npm ci && npm run build' }
        }
        stage('Test') {
            steps { sh 'npm test' }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps { sh './deploy.sh' }
        }
    }
}
```

---

## Strengths & Weaknesses

```
  JENKINS
  ├── ✓ Extremely flexible (1800+ plugins)
  ├── ✓ Full control (self-hosted)
  ├── ✓ Mature ecosystem
  ├── ✗ High operational overhead
  ├── ✗ Plugin dependency management
  └── ✗ UI/UX dated

  GITHUB ACTIONS
  ├── ✓ Native GitHub integration
  ├── ✓ Huge marketplace (20K+ actions)
  ├── ✓ Easy to learn
  ├── ✗ GitHub-only (vendor lock-in)
  ├── ✗ Limited debugging (no SSH)
  └── ✗ Minutes-based pricing

  GITLAB CI/CD
  ├── ✓ All-in-one platform (SCM+CI+CD+Registry)
  ├── ✓ Built-in security scanning
  ├── ✓ Self-hosted option
  ├── ✗ Advanced features need Premium/Ultimate
  ├── ✗ Large config files for complex pipelines
  └── ✗ Runner management overhead

  AZURE PIPELINES
  ├── ✓ Deep Azure integration
  ├── ✓ Multi-platform (Windows, Linux, macOS)
  ├── ✓ Powerful template system
  ├── ✗ Complex YAML syntax
  ├── ✗ Azure DevOps UI complexity
  └── ✗ Free tier limitations

  CIRCLECI
  ├── ✓ Fast builds + Docker layer caching
  ├── ✓ Excellent test splitting
  ├── ✓ Orbs ecosystem
  ├── ✗ Limited executor types
  ├── ✗ Pricing at scale
  └── ✗ Smaller community than competitors

  ARGOCD
  ├── ✓ True GitOps model
  ├── ✓ Drift detection + self-heal
  ├── ✓ Kubernetes-native
  ├── ✗ CD only (needs separate CI)
  ├── ✗ Kubernetes required
  └── ✗ Learning curve for GitOps

  TEKTON
  ├── ✓ Cloud-agnostic, K8s-native
  ├── ✓ No vendor lock-in (CNCF)
  ├── ✓ Highly composable
  ├── ✗ Verbose YAML configuration
  ├── ✗ Steep learning curve
  └── ✗ Minimal built-in UI
```

---

## Ecosystem Comparison

```
  MARKETPLACE / EXTENSIONS
  ═════════════════════════

  Jenkins:        1,800+ plugins (plugins.jenkins.io)
  GitHub Actions: 20,000+ actions (marketplace)
  GitLab CI:      Templates + CI catalog
  Azure:          VS Marketplace extensions
  CircleCI:       Orb registry (3,000+)
  ArgoCD:         Resource hooks + generators
  Tekton:         Tekton Hub catalog tasks
```

---

## Real-World Scenario

> **Scenario**: A CTO is evaluating CI/CD tools for a series B startup with 30 engineers, using GitHub for source code, deploying to AWS EKS, and building a mix of Node.js and Python services.
>
> **Solution**: GitHub Actions is the strongest fit — native GitHub integration eliminates context switching, the marketplace has actions for ECR/EKS, and the free tier covers the team size. For production CD, pair with ArgoCD for GitOps-based Kubernetes deployments. GitHub Actions handles CI (build, test, scan, push image) while ArgoCD handles CD (watch config repo, sync to cluster). This combination is increasingly the industry standard for GitHub+Kubernetes shops.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Can't decide between tools | Too many options | Use the decision framework in Chapter 9.6 |
| Migration complexity | Tight coupling to current tool | Abstract CI config into scripts, reducing tool-specific logic |
| Multi-tool overhead | Using Jenkins CI + ArgoCD CD | Expected — CI and CD tools serve different roles |
| Vendor lock-in concerns | Deep tool-specific features | Minimize tool-specific config; keep logic in scripts |
| Cost unpredictable | Minutes-based pricing | Monitor usage; consider self-hosted runners |
| Team resistance | Familiarity with current tool | Incremental migration; pilot on one project |

---

## Summary Table

| Dimension | Best Tool(s) |
|---|---|
| GitHub-native teams | GitHub Actions |
| All-in-one platform | GitLab CI/CD |
| Maximum flexibility | Jenkins |
| Azure/Microsoft shops | Azure Pipelines |
| Fast test feedback | CircleCI |
| GitOps / Kubernetes CD | ArgoCD |
| Cloud-agnostic K8s CI | Tekton |
| Enterprise compliance | GitLab Ultimate, Azure Pipelines |
| Open source / no lock-in | Jenkins, Tekton, ArgoCD |

---

## Quick Revision Questions

1. **Which tools are Kubernetes-native?** *ArgoCD and Tekton are fully Kubernetes-native, running as CRDs. GitLab and GitHub Actions have Kubernetes runner support but aren't K8s-native themselves.*
2. **Which tool is best for an all-in-one platform?** *GitLab CI/CD — it combines source control, CI/CD, container registry, security scanning, and project management in one platform.*
3. **How do Jenkins and GitHub Actions compare?** *Jenkins offers maximum flexibility with plugins and self-hosting but has high operational overhead. GitHub Actions is easier to learn with native GitHub integration but creates vendor lock-in.*
4. **What's the difference between ArgoCD and Tekton?** *ArgoCD is a CD-only GitOps tool that syncs Kubernetes state from Git. Tekton is a CI/CD framework that runs build/test/deploy tasks as Kubernetes pods.*
5. **Which tools support multi-cloud deployments?** *Jenkins, Tekton, and ArgoCD are cloud-agnostic. GitHub Actions, GitLab CI, Azure Pipelines, and CircleCI support multi-cloud through integrations but are hosted on specific platforms.*
6. **How do you minimize vendor lock-in?** *Keep CI/CD logic in shell scripts or Makefiles that any tool can call. Use tool-specific YAML only for orchestration (triggers, ordering, environment). This makes migration between tools easier.*

---

[← Previous: Tekton](04-tekton.md) | [Back to README](../README.md) | [Next: Choosing a CI/CD Tool →](06-choosing-a-tool.md)

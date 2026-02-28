# Chapter 8.7: GitLab Auto DevOps

[← Previous: Environments & Review Apps](06-environments-review-apps.md) | [Back to README](../README.md) | [Next: GitLab CI/CD Best Practices →](08-gitlab-cicd-best-practices.md)

---

## Overview

**Auto DevOps** is GitLab's pre-configured CI/CD pipeline that automatically detects, builds, tests, and deploys applications with zero configuration. It provides opinionated defaults that follow best practices.

---

## Auto DevOps Pipeline Stages

```
  AUTO DEVOPS PIPELINE
  ═════════════════════

  ┌─────────┐   ┌──────────┐   ┌───────────┐   ┌──────────┐
  │  Auto    │──▶│  Auto    │──▶│   Auto    │──▶│  Auto    │
  │  Build   │   │  Test    │   │   SAST    │   │  Deploy  │
  └─────────┘   └──────────┘   └───────────┘   └──────────┘
       │              │               │               │
  Buildpacks     Herokuish      Security scans   Helm deploy
  or Dockerfile  test runner    (SAST, DAST,     to Kubernetes
                                 Secrets, Deps)

  Additional stages:
  ┌───────────────┐  ┌────────────────┐  ┌──────────────┐
  │ Code Quality  │  │ Container      │  │ License       │
  │ Analysis      │  │ Scanning       │  │ Compliance    │
  └───────────────┘  └────────────────┘  └──────────────┘
```

---

## Enabling Auto DevOps

### Project Level

```
  Settings → CI/CD → Auto DevOps
  ┌────────────────────────────────────────────┐
  │ ☑ Default to Auto DevOps pipeline          │
  │                                            │
  │ Deployment strategy:                       │
  │ ○ Continuous deployment to production      │
  │ ● Continuous deployment (manual to prod)   │
  │ ○ Timed incremental rollout                │
  │                                            │
  │ Deployment platform:                       │
  │ Kubernetes (cluster integration required)  │
  └────────────────────────────────────────────┘
```

### Instance Level

```yaml
# Admin Area → Settings → CI/CD → Auto DevOps
# Enable for all projects that don't have CI config

# Or use API:
# PUT /api/v4/application/settings
# auto_devops_enabled: true
```

---

## Auto DevOps Stages Explained

### Auto Build

```
  DETECTION ORDER
  ═══════════════

  1. Dockerfile found?
     ├── YES → docker build
     └── NO  → Continue

  2. Heroku buildpacks detection:
     ├── package.json → Node.js buildpack
     ├── Gemfile      → Ruby buildpack
     ├── requirements.txt → Python buildpack
     ├── pom.xml      → Java (Maven) buildpack
     ├── build.gradle → Java (Gradle) buildpack
     ├── go.mod       → Go buildpack
     └── composer.json → PHP buildpack

  Result: Container image pushed to GitLab Registry
```

### Auto Test

```
  ┌────────────────┬───────────────────────────┐
  │ Language       │ Test Command              │
  ├────────────────┼───────────────────────────┤
  │ Ruby           │ rake test / rspec         │
  │ Python         │ python setup.py test      │
  │ Node.js        │ npm test                  │
  │ Java           │ mvn test / gradle test    │
  │ Go             │ go test                   │
  │ PHP            │ phpunit                   │
  └────────────────┴───────────────────────────┘
```

### Auto Security Scanning

```yaml
# Auto DevOps includes these automatically:
# - SAST (Static Application Security Testing)
# - DAST (Dynamic Application Security Testing)
# - Dependency Scanning
# - Container Scanning
# - Secret Detection
# - License Scanning

# Results appear in:
# • MR widgets (new vulnerabilities)
# • Security Dashboard (project/group/instance)
# • Vulnerability reports
```

### Auto Deploy

```
  AUTO DEPLOY TO KUBERNETES
  ══════════════════════════

  Kubernetes Cluster
  ┌─────────────────────────────────────────┐
  │                                         │
  │  Namespace: $KUBE_NAMESPACE             │
  │                                         │
  │  ┌─────────────────────────────┐        │
  │  │ Helm Release: $CI_PROJECT   │        │
  │  │ ├── Deployment (replicas=1) │        │
  │  │ ├── Service (ClusterIP)     │        │
  │  │ ├── Ingress (if domain set) │        │
  │  │ └── HPA (if configured)     │        │
  │  └─────────────────────────────┘        │
  │                                         │
  └─────────────────────────────────────────┘
```

---

## Customizing Auto DevOps

### Using CI/CD Variables

```yaml
# These variables customize Auto DevOps behavior:

variables:
  # Build
  AUTO_DEVOPS_BUILD_IMAGE_CNB_ENABLED: "true"  # Use Cloud Native Buildpacks
  AUTO_DEVOPS_BUILD_IMAGE_EXTRA_ARGS: "--build-arg NODE_ENV=production"

  # Test
  TEST_DISABLED: "true"                # Skip auto test
  CODE_QUALITY_DISABLED: "true"        # Skip code quality

  # Security
  SAST_DISABLED: "true"                # Skip SAST
  DAST_DISABLED: "true"               # Skip DAST
  DEPENDENCY_SCANNING_DISABLED: "true"  # Skip dependency scan
  CONTAINER_SCANNING_DISABLED: "true"   # Skip container scan

  # Deploy
  STAGING_ENABLED: "true"              # Enable staging environment
  CANARY_ENABLED: "true"               # Enable canary deployments
  INCREMENTAL_ROLLOUT_MODE: "manual"   # manual or timed
  ROLLOUT_RESOURCE_TYPE: "deployment"  # deployment or statefulset

  # Replicas
  REPLICAS: "3"
  PRODUCTION_REPLICAS: "5"
  CANARY_REPLICAS: "1"
```

### Partial Override with `.gitlab-ci.yml`

```yaml
# Include Auto DevOps template, then override specific jobs:

include:
  - template: Auto-DevOps.gitlab-ci.yml

# Override the test stage with custom tests
test:
  stage: test
  script:
    - npm ci
    - npm run test:ci
    - npm run test:e2e

# Add a custom job without replacing defaults
custom-lint:
  stage: test
  script:
    - npm run lint
```

### Custom Helm Chart

```yaml
# By default, Auto DevOps uses gitlab/auto-deploy-app chart
# Override with your own:

variables:
  AUTO_DEVOPS_CHART: "./my-chart"              # Local chart
  # OR
  AUTO_DEVOPS_CHART_REPOSITORY: "https://charts.example.com"
  AUTO_DEVOPS_CHART: "my-org/my-app"

# Customize values:
  HELM_UPGRADE_EXTRA_ARGS: "--values custom-values.yaml"
```

---

## Deployment Strategies with Auto DevOps

```
  STRATEGY OPTIONS
  ════════════════

  1. Continuous deployment:
     main → staging → production (automatic)

  2. Manual to production (default):
     main → staging (auto) → production (manual click)

  3. Timed incremental rollout:
     main → staging → 10% → 25% → 50% → 100%
                       ↑     ↑     ↑      ↑
                    (each step waits a period)

  4. Canary deployment:
     main → staging → canary (small %) → full rollout
```

```yaml
# Incremental rollout example
variables:
  INCREMENTAL_ROLLOUT_MODE: "timed"
  ROLLOUT_PERCENTAGE: "10"    # Start at 10%
  # Rolls out: 10% → 25% → 50% → 100%
  # Each step triggers after configured delay
```

---

## When to Use Auto DevOps

```
  ┌─────────────────────────┬──────────────────────────────┐
  │ Good Fit                │ Not a Good Fit               │
  ├─────────────────────────┼──────────────────────────────┤
  │ Standard web apps       │ Complex multi-service apps   │
  │ Rapid prototyping       │ Non-standard build systems   │
  │ Small teams             │ Teams with custom processes  │
  │ Standard languages      │ Embedded / IoT systems       │
  │ Quick Kubernetes deploy │ Non-Kubernetes deployments   │
  │ Compliance scanning     │ Highly regulated industries  │
  │ Getting started quickly │ with custom audit needs      │
  └─────────────────────────┴──────────────────────────────┘
```

---

## Real-World Scenario

> **Scenario**: A startup wants to ship features fast without spending time on CI/CD setup. They have 10 microservices, each in its own repo, all Node.js apps deploying to Kubernetes.
>
> **Solution**: Enable Auto DevOps at the group level. Each project automatically gets build, test, security scanning, and Kubernetes deployment. Override `test` stage in two repos that need custom test setups. Set `STAGING_ENABLED: "true"` at group level so all services get a staging environment. Use incremental rollout for production. This gives the team enterprise-grade CI/CD immediately, with the flexibility to customize individual repos as they mature.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Build stage fails | Language not detected | Add Dockerfile or ensure build file exists |
| "No buildpack found" | Unsupported language or missing manifest | Create `Dockerfile` for unsupported languages |
| Deploy fails | No Kubernetes cluster connected | Connect cluster via Settings → Kubernetes |
| Tests not running | Test framework not detected | Override `test` stage with explicit commands |
| Security scans missing | Ultimate license required for some scans | Check GitLab tier; SAST works on all tiers |
| Review apps not created | Missing `REVIEW_DISABLED` not set | Ensure Kubernetes cluster and domain configured |

---

## Summary Table

| Feature | Description |
|---|---|
| Auto Build | Detects language, builds container image |
| Auto Test | Runs language-appropriate tests |
| Auto SAST | Static security analysis |
| Auto DAST | Dynamic security testing against running app |
| Auto Deploy | Helm-based Kubernetes deployment |
| Container Scanning | Scans built images for vulnerabilities |
| Dependency Scanning | Checks dependencies for known CVEs |
| Incremental Rollout | Gradual traffic percentage increase |
| Customization | Override stages, variables, Helm chart |

---

## Quick Revision Questions

1. **What is Auto DevOps?** *A pre-configured CI/CD pipeline that auto-detects, builds, tests, scans, and deploys applications with zero configuration.*
2. **How does Auto Build detect the language?** *First checks for a Dockerfile. If none exists, uses Cloud Native Buildpacks (or Heroku buildpacks) to detect language from manifest files (package.json, pom.xml, etc.).*
3. **What security scans does Auto DevOps include?** *SAST, DAST, Dependency Scanning, Container Scanning, Secret Detection, and License Compliance.*
4. **How do you customize Auto DevOps without replacing it entirely?** *Include the Auto-DevOps template in `.gitlab-ci.yml` and override specific jobs, or use CI/CD variables to enable/disable features.*
5. **What deployment strategies does Auto DevOps support?** *Continuous deployment, manual-to-production, timed incremental rollout, and canary deployments.*
6. **When should you avoid Auto DevOps?** *Complex multi-service applications, non-standard build systems, non-Kubernetes deployments, or teams with highly customized CI/CD requirements.*

---

[← Previous: Environments & Review Apps](06-environments-review-apps.md) | [Back to README](../README.md) | [Next: GitLab CI/CD Best Practices →](08-gitlab-cicd-best-practices.md)

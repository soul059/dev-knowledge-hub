# Chapter 4.2 — CI/CD Tools

[← Previous: Version Control Systems](01-version-control-systems.md) | [Next: Containerization →](03-containerization.md)

---

## Overview

CI/CD tools automate the process of building, testing, and deploying software. They are the engines that power the DevOps pipeline — taking code from a developer's commit all the way to production without manual intervention.

---

## CI/CD Tools Landscape

```
┌──────────────────────────────────────────────────────────┐
│  CI/CD TOOLS ECOSYSTEM                                   │
│                                                          │
│  CLOUD-NATIVE                    SELF-HOSTED             │
│  ════════════                    ═══════════             │
│  ┌────────────────┐              ┌────────────────┐     │
│  │ GitHub Actions │              │    Jenkins     │     │
│  │ (GitHub-native)│              │ (open-source)  │     │
│  └────────────────┘              └────────────────┘     │
│  ┌────────────────┐              ┌────────────────┐     │
│  │  GitLab CI/CD  │              │    TeamCity    │     │
│  │ (GitLab-native)│              │   (JetBrains)  │     │
│  └────────────────┘              └────────────────┘     │
│  ┌────────────────┐              ┌────────────────┐     │
│  │ Azure Pipelines│              │  Bamboo        │     │
│  │  (Microsoft)   │              │  (Atlassian)   │     │
│  └────────────────┘              └────────────────┘     │
│  ┌────────────────┐              ┌────────────────┐     │
│  │   CircleCI     │              │    Drone CI    │     │
│  └────────────────┘              │  (container)   │     │
│  ┌────────────────┐              └────────────────┘     │
│  │    AWS          │                                     │
│  │ CodePipeline   │              ┌────────────────┐     │
│  └────────────────┘              │    ArgoCD      │     │
│                                  │  (GitOps/K8s)  │     │
│                                  └────────────────┘     │
└──────────────────────────────────────────────────────────┘
```

---

## GitHub Actions

### Pipeline as Code (YAML)

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ═══ BUILD & TEST ═══
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run unit tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.node-version }}
          path: coverage/

  # ═══ SECURITY SCAN ═══
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  # ═══ BUILD DOCKER IMAGE ═══
  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  # ═══ DEPLOY ═══
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/web-app \
            web-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

### GitHub Actions Architecture

```
┌──────────────────────────────────────────────────────────┐
│  GITHUB ACTIONS CONCEPTS                                 │
│                                                          │
│  Workflow (.yml file)                                    │
│  └── Triggered by: push, PR, schedule, manual            │
│      │                                                   │
│      ├── Job: "test"          (runs on: ubuntu-latest)  │
│      │   ├── Step: Checkout code                        │
│      │   ├── Step: Install deps                         │
│      │   ├── Step: Run tests                            │
│      │   └── Step: Upload coverage                      │
│      │                                                   │
│      ├── Job: "security"      (runs in parallel)        │
│      │   └── Step: Snyk scan                            │
│      │                                                   │
│      └── Job: "deploy"        (needs: test, security)   │
│          └── Step: Deploy to K8s                         │
│                                                          │
│  Key Terms:                                              │
│  ├── Workflow = entire pipeline definition               │
│  ├── Job = set of steps running on same runner           │
│  ├── Step = individual task (action or shell command)    │
│  ├── Action = reusable packaged step (from Marketplace) │
│  └── Runner = machine that executes jobs                 │
└──────────────────────────────────────────────────────────┘
```

---

## Jenkins

### Jenkinsfile (Declarative Pipeline)

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'web-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} ."
                sh "docker push ${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} -n staging"
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            input { message "Deploy to production?" }
            steps {
                sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} -n production"
            }
        }
    }

    post {
        success { slackSend channel: '#deploys', message: "✅ ${APP_NAME} deployed" }
        failure { slackSend channel: '#deploys', message: "❌ ${APP_NAME} failed" }
    }
}
```

---

## GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE

unit-test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: test-results.xml

sast:
  stage: security
  include:
    - template: Security/SAST.gitlab-ci.yml

deploy-production:
  stage: deploy
  environment:
    name: production
    url: https://app.example.com
  script:
    - kubectl set image deployment/app app=$DOCKER_IMAGE
  only:
    - main
  when: manual  # Manual approval gate
```

---

## CI/CD Tools Comparison

| Feature | GitHub Actions | Jenkins | GitLab CI/CD | CircleCI |
|---------|---------------|---------|-------------|----------|
| **Hosting** | Cloud (GitHub) | Self-hosted | Cloud + Self | Cloud |
| **Config** | YAML | Groovy/YAML | YAML | YAML |
| **Ease of Setup** | Very Easy | Complex | Easy | Easy |
| **Plugin Ecosystem** | Marketplace (large) | Plugins (huge) | Built-in | Orbs |
| **Free Tier** | 2000 min/month | Free (OSS) | 400 min/month | 6000 min/month |
| **Docker Support** | Native | Plugin | Native | Native |
| **Self-Hosted Runners** | ✅ | Default | ✅ | ✅ |
| **Best For** | GitHub repos | Enterprise, legacy | Full platform | Startups |

---

## Real-World Scenario: Migrating from Jenkins to GitHub Actions

```
SITUATION:
├── Company has 50+ Jenkins pipelines
├── Jenkins server requires constant maintenance
├── Plugin version conflicts causing weekly outages
├── Team spending 20% of time on Jenkins maintenance
└── All code already on GitHub

MIGRATION PLAN:
1. Audit existing Jenkins pipelines
   └── Map each Jenkinsfile to GitHub Actions YAML

2. Start with low-risk pipelines (docs, linting)
   └── Run both systems in parallel for 2 weeks

3. Migrate build + test pipelines
   └── Verify same artifacts produced

4. Migrate deployment pipelines
   └── Use GitHub Environments for approval gates

5. Decommission Jenkins
   └── Archive Jenkinsfiles in repo for reference

RESULTS:
├── Pipeline maintenance: 20% → 2% of team time
├── Build times: 12 min → 6 min (GitHub runners)
├── Developer happiness: Significantly improved
└── Cost: $0 (within GitHub Actions free tier)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Pipeline fails on "works on my machine" | Different environments | Use Docker containers as CI runners for consistency |
| Secrets exposed in logs | Printing env vars | Use secrets masking; never `echo $SECRET` |
| Flaky tests | Non-deterministic tests | Isolate tests, mock external deps, retry with limits |
| Slow pipelines | Sequential steps | Parallelize jobs, use caching (npm cache, Docker layers) |
| Jenkins plugin conflicts | Incompatible versions | Pin plugin versions, test upgrades in staging first |
| Rate limiting | Too many API calls | Use caching, batch operations, respect rate limits |

---

## Summary Table

| CI/CD Concept | Description |
|--------------|-------------|
| **GitHub Actions** | GitHub-native CI/CD with YAML workflows and Marketplace |
| **Jenkins** | Open-source, self-hosted CI/CD with massive plugin ecosystem |
| **GitLab CI/CD** | Built into GitLab with auto-DevOps capabilities |
| **Pipeline as Code** | CI/CD configuration stored as code in the repository |
| **Runners / Agents** | Machines that execute pipeline jobs |
| **Artifacts** | Build outputs passed between pipeline stages |
| **Environments** | Deployment targets with approval gates (staging, prod) |
| **Secrets Management** | Encrypted variables for API keys and credentials |

---

## Quick Revision Questions

1. **What is "Pipeline as Code," and why is it important?**
   <details><summary>Answer</summary>Pipeline as Code means defining CI/CD pipelines in configuration files (YAML or Groovy) stored in the repository alongside application code. Benefits: 1) Version controlled — changes are tracked. 2) Peer-reviewed — PRs review pipeline changes. 3) Reproducible — any team member can understand the pipeline. 4) Portable — move between projects or teams easily.</details>

2. **What are the main differences between GitHub Actions and Jenkins?**
   <details><summary>Answer</summary>GitHub Actions: cloud-hosted, YAML config, easy setup, native GitHub integration, Marketplace for actions, free tier available. Jenkins: self-hosted, Groovy config, complex setup, requires maintenance, huge plugin ecosystem, fully customizable. GitHub Actions is simpler; Jenkins offers more control but higher operational overhead.</details>

3. **What is a CI/CD runner, and how do self-hosted runners differ from cloud runners?**
   <details><summary>Answer</summary>A runner is the machine that executes pipeline jobs. Cloud runners: provided by the platform (ephemeral, pre-configured, no maintenance needed). Self-hosted runners: your own machines (persistent, customizable hardware/software, you maintain them). Use self-hosted for: specialized hardware, private network access, or cost optimization at scale.</details>

4. **How do you handle secrets in CI/CD pipelines?**
   <details><summary>Answer</summary>1) Store secrets in the platform's secrets manager (GitHub Secrets, GitLab Variables). 2) Never hardcode secrets in pipeline files. 3) Use secret masking to prevent accidental logging. 4) Scope secrets to specific environments. 5) Rotate secrets regularly. 6) Use OIDC for cloud provider authentication instead of long-lived credentials.</details>

5. **What causes flaky tests in CI/CD, and how do you fix them?**
   <details><summary>Answer</summary>Flaky tests pass sometimes and fail others. Causes: 1) Timing/race conditions. 2) External service dependencies. 3) Shared state between tests. 4) Date/time sensitivity. Fixes: mock external dependencies, isolate test state, use deterministic data, run flaky tests in quarantine, and add retry logic with a limit.</details>

6. **What is the role of artifacts in CI/CD pipelines?**
   <details><summary>Answer</summary>Artifacts are files produced during a pipeline run (compiled binaries, Docker images, test reports, coverage reports). They serve to: 1) Pass build outputs between stages. 2) Store evidence of test results. 3) Provide deployable packages. 4) Enable audit trails. Artifacts are typically stored temporarily and promoted through environments.</details>

---

[← Previous: Version Control Systems](01-version-control-systems.md) | [Next: Containerization →](03-containerization.md)

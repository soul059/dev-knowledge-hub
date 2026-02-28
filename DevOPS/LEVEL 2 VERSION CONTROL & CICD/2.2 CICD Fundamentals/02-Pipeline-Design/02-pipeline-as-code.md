# Chapter 2.2: Pipeline as Code

[← Previous: Pipeline Stages](01-pipeline-stages.md) | [Back to README](../README.md) | [Next: Artifact Management →](03-artifact-management.md)

---

## Overview

**Pipeline as Code** means defining your CI/CD pipeline in a version-controlled file stored alongside your application source code, rather than configuring it through a GUI. This is a cornerstone of modern CI/CD — treating your pipeline with the same rigor as your application code.

---

## GUI vs Pipeline as Code

```
GUI-BASED PIPELINE (Legacy)                PIPELINE AS CODE (Modern)
═══════════════════════════                ═══════════════════════════

  ┌──────────────────┐                     ┌──────────────────┐
  │  CI/CD Server UI │                     │  Git Repository  │
  │                  │                     │                  │
  │  Click           │                     │  Jenkinsfile     │
  │  Configure       │                     │  .github/        │
  │  Save            │                     │    workflows/    │
  │                  │                     │      ci.yml      │
  │  ❌ No history   │                     │                  │
  │  ❌ No review    │                     │  ✅ Versioned    │
  │  ❌ Not portable │                     │  ✅ Reviewable   │
  │  ❌ Hard to      │                     │  ✅ Portable     │
  │     replicate    │                     │  ✅ Reproducible │
  └──────────────────┘                     └──────────────────┘
```

---

## Benefits of Pipeline as Code

```
┌──────────────────────────────────────────────────────────────────┐
│              PIPELINE AS CODE BENEFITS                            │
│                                                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  VERSION        │  │  CODE REVIEW    │  │  REPRODUCIBLE   │  │
│  │  CONTROLLED     │  │                 │  │                 │  │
│  │                 │  │  Pipeline       │  │  Same file =    │  │
│  │  Full history   │  │  changes go     │  │  same pipeline  │  │
│  │  of pipeline    │  │  through PR     │  │  on any server  │  │
│  │  changes        │  │  review         │  │                 │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │  BRANCH-AWARE   │  │  TESTABLE       │  │  SELF-DOCUMENTING│ │
│  │                 │  │                 │  │                 │  │
│  │  Each branch    │  │  Test pipeline  │  │  Pipeline file  │  │
│  │  can have its   │  │  changes in a   │  │  documents the  │  │
│  │  own pipeline   │  │  feature branch │  │  build process  │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Pipeline as Code — Comparison Across Tools

### Jenkins (Jenkinsfile)

```groovy
// Jenkinsfile (Declarative Pipeline)
pipeline {
    agent {
        docker { image 'node:20-alpine' }
    }
    
    environment {
        NPM_TOKEN = credentials('npm-token')
    }
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps { sh 'npm test' }
                }
                stage('Lint') {
                    steps { sh 'npm run lint' }
                }
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh './deploy.sh production'
            }
        }
    }
    
    post {
        failure {
            slackSend channel: '#alerts', message: "Build Failed!"
        }
    }
}
```

### GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - run: ./deploy.sh production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

test:
  stage: test
  image: node:${NODE_VERSION}-alpine
  script:
    - npm ci
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: reports/junit.xml

build:
  stage: build
  image: node:${NODE_VERSION}-alpine
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

deploy:
  stage: deploy
  only:
    - main
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://app.example.com
```

### Azure Pipelines

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include: [main]

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Test
    jobs:
      - job: UnitTests
        steps:
          - task: NodeTool@0
            inputs: { versionSpec: '20.x' }
          - script: npm ci && npm test

  - stage: Build
    dependsOn: Test
    jobs:
      - job: BuildApp
        steps:
          - script: npm ci && npm run build
          - publish: dist/
            artifact: app-build

  - stage: Deploy
    dependsOn: Build
    condition: eq(variables['Build.SourceBranch'], 'refs/heads/main')
    jobs:
      - deployment: Production
        environment: production
        strategy:
          runOnce:
            deploy:
              steps:
                - script: ./deploy.sh
```

---

## Pipeline as Code Best Practices

### 1. Keep Pipelines DRY

```yaml
# GitHub Actions — Reusable workflows
# .github/workflows/reusable-test.yml
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci && npm test

# .github/workflows/ci.yml  (calling the reusable workflow)
jobs:
  test-node-20:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
```

### 2. Validate Pipeline Files Locally

```bash
# Jenkins
# Install jenkinsfile-runner for local testing

# GitHub Actions
act                          # Run workflows locally with 'act'

# GitLab CI
gitlab-ci-lint               # Lint .gitlab-ci.yml
gitlab-runner exec docker test  # Run a job locally

# CircleCI
circleci config validate     # Validate config
circleci local execute       # Run locally
```

### 3. Use Templates and Anchors (YAML)

```yaml
# GitLab CI — YAML anchors for reuse
.node-setup: &node-setup
  image: node:20-alpine
  before_script:
    - npm ci

test:
  <<: *node-setup
  stage: test
  script:
    - npm test

lint:
  <<: *node-setup
  stage: test
  script:
    - npm run lint
```

---

## Pipeline as Code Anti-Patterns

```
┌──────────────────────────────────────────────────────────────┐
│           ANTI-PATTERNS TO AVOID                              │
│                                                               │
│  ❌ Hardcoding secrets in pipeline files                     │
│     → Use secret management (Vault, CI secrets)              │
│                                                               │
│  ❌ Copy-pasting pipeline code across repos                  │
│     → Use shared libraries / reusable workflows              │
│                                                               │
│  ❌ Putting too much logic in pipeline YAML                  │
│     → Move complex logic to scripts (build.sh, Makefile)     │
│                                                               │
│  ❌ Not reviewing pipeline changes                           │
│     → Require PR reviews for pipeline file changes           │
│                                                               │
│  ❌ Ignoring pipeline file in .gitignore                     │
│     → Pipeline files MUST be version controlled              │
│                                                               │
│  ❌ One massive pipeline file                                │
│     → Split into reusable components/templates               │
└──────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Pipeline as Code

| Issue | Cause | Fix |
|---|---|---|
| YAML syntax error | Indentation or formatting | Use YAML linter, validate locally |
| Pipeline doesn't trigger | Wrong branch filter or event | Check `on:` / `only:` / `trigger:` config |
| Can't test pipeline locally | Tool-specific syntax | Use `act` (GHA), `gitlab-runner exec` |
| Duplicated code across repos | No shared templates | Create reusable workflows / shared libraries |
| Secrets not available | Wrong scope or name | Check secret name, environment scope |

---

## Summary Table

| Aspect | Description |
|---|---|
| **Definition** | Defining CI/CD pipelines in version-controlled files |
| **File Location** | Root of repository (alongside source code) |
| **Key Benefit** | Version history, code review, reproducibility |
| **Jenkins** | `Jenkinsfile` (Groovy — Declarative or Scripted) |
| **GitHub Actions** | `.github/workflows/*.yml` (YAML) |
| **GitLab CI** | `.gitlab-ci.yml` (YAML) |
| **Azure Pipelines** | `azure-pipelines.yml` (YAML) |
| **Best Practice** | DRY, validate locally, use templates, review changes |
| **Anti-Pattern** | Hardcoded secrets, copy-pasta, too much inline logic |

---

## Quick Revision Questions

1. **What does "Pipeline as Code" mean?**  
   *Defining CI/CD pipelines in version-controlled configuration files stored in the same repository as the application code.*

2. **Name three advantages of Pipeline as Code over GUI-configured pipelines.**  
   *Version history (audit trail), code review via PRs, reproducibility across environments.*

3. **What file does each major CI/CD tool use for Pipeline as Code?**  
   *Jenkins: Jenkinsfile, GitHub Actions: .github/workflows/*.yml, GitLab CI: .gitlab-ci.yml, Azure Pipelines: azure-pipelines.yml.*

4. **How can you avoid duplicating pipeline code across multiple repositories?**  
   *Use reusable workflows (GitHub Actions), shared libraries (Jenkins), YAML anchors/templates (GitLab CI).*

5. **Why should you move complex logic out of pipeline YAML files?**  
   *YAML isn't a programming language — complex logic is hard to read, test, and debug in YAML. Move it to shell scripts or Makefiles.*

6. **How can you validate pipeline files before pushing?**  
   *Use tools like `act` (GitHub Actions), `gitlab-ci-lint` (GitLab), `circleci config validate` (CircleCI) to validate locally.*

---

[← Previous: Pipeline Stages](01-pipeline-stages.md) | [Back to README](../README.md) | [Next: Artifact Management →](03-artifact-management.md)

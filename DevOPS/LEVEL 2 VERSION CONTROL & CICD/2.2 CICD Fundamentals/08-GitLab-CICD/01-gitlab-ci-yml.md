# Chapter 8.1: GitLab CI/CD Configuration (.gitlab-ci.yml)

[← Previous: GitHub Actions Best Practices](../07-GitHub-Actions/08-github-actions-best-practices.md) | [Back to README](../README.md) | [Next: Stages & Jobs →](02-stages-jobs.md)

---

## Overview

GitLab CI/CD uses a single `.gitlab-ci.yml` file in the repository root to define the entire pipeline. It supports stages, jobs, variables, caching, artifacts, and more — tightly integrated with the GitLab platform.

---

## GitLab CI/CD Architecture

```
  GITLAB CI/CD ARCHITECTURE
  ═════════════════════════

  Developer pushes code
       │
       ▼
  ┌─────────────────────────┐
  │  GitLab Server           │
  │  ├── Reads .gitlab-ci.yml│
  │  ├── Creates pipeline    │
  │  └── Queues jobs         │
  └──────────┬──────────────┘
             │
             ▼
  ┌─────────────────────────┐
  │  GitLab Runner(s)       │
  │  ├── Picks up jobs      │
  │  ├── Executes in        │
  │  │   Docker / Shell /   │
  │  │   Kubernetes         │
  │  └── Reports results    │
  └─────────────────────────┘
```

---

## Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  NODE_VERSION: "20"

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm test -- --ci --coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      junit: reports/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

deploy:
  stage: deploy
  script:
    - ./deploy.sh production
  environment:
    name: production
    url: https://app.example.com
  only:
    - main
  when: manual
```

---

## Pipeline Structure

```
  PIPELINE → STAGES → JOBS
  ═════════════════════════

  Pipeline (one per push/MR)
  ├── Stage: build
  │   └── Job: build         (runs in parallel within stage)
  ├── Stage: test
  │   ├── Job: unit-tests    (parallel)
  │   ├── Job: lint          (parallel)
  │   └── Job: security-scan (parallel)
  ├── Stage: staging
  │   └── Job: deploy-staging
  └── Stage: production
      └── Job: deploy-prod   (manual trigger)

  RULES:
  • Jobs in the same stage run in PARALLEL
  • Stages run SEQUENTIALLY
  • Next stage starts only if ALL jobs in previous stage pass
```

---

## Key Configuration Elements

### Image

```yaml
# Global image (default for all jobs)
image: node:20-alpine

# Job-specific image
test:
  image: node:20
  script: npm test

build-java:
  image: maven:3.9-eclipse-temurin-21
  script: mvn package
```

### Variables

```yaml
# Global variables
variables:
  DATABASE_URL: "postgres://localhost:5432/test"
  DOCKER_DRIVER: overlay2

# Job variables (override global)
test:
  variables:
    NODE_ENV: test
  script: npm test

# Predefined variables (available automatically):
# CI_COMMIT_SHA       → Full commit hash
# CI_COMMIT_REF_NAME  → Branch or tag name
# CI_PIPELINE_ID      → Pipeline ID
# CI_PROJECT_NAME     → Project name
# CI_MERGE_REQUEST_IID→ MR number
# CI_JOB_TOKEN        → Auth token for GitLab APIs
```

### Before/After Scripts

```yaml
# Global before_script (runs before every job)
default:
  before_script:
    - echo "Starting job in $(pwd)"
    - npm ci

# Job-specific override
deploy:
  before_script:
    - apt-get update && apt-get install -y awscli
  script:
    - aws deploy push
  after_script:
    - echo "Cleanup complete"  # Runs even if script fails
```

---

## Rules (Modern Conditional Execution)

```yaml
# rules: replaces only:/except: (deprecated)
deploy-staging:
  stage: deploy
  script: ./deploy.sh staging
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: always
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
    - when: never    # Default: don't run

deploy-production:
  stage: deploy
  script: ./deploy.sh production
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual
      allow_failure: false

# Path-based rules
test-frontend:
  script: cd frontend && npm test
  rules:
    - changes:
        - frontend/**/*
        - package.json

# Variable existence
notify:
  script: ./notify.sh
  rules:
    - if: $SLACK_WEBHOOK
      exists:
        - scripts/notify.sh
```

---

## Includes (DRY Configuration)

```yaml
# Include templates from various sources
include:
  # Local file
  - local: '/ci/templates/docker.yml'

  # From another project
  - project: 'my-org/ci-templates'
    ref: 'main'
    file:
      - '/templates/node.yml'
      - '/templates/docker.yml'

  # Remote URL
  - remote: 'https://example.com/ci/template.yml'

  # GitLab managed templates
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
```

---

## GitLab vs GitHub Actions vs Jenkins

```
  ┌──────────────────┬───────────────────┬────────────────────┬──────────────┐
  │ Feature          │ GitLab CI/CD      │ GitHub Actions     │ Jenkins      │
  ├──────────────────┼───────────────────┼────────────────────┼──────────────┤
  │ Config file      │ .gitlab-ci.yml    │ .github/workflows/ │ Jenkinsfile  │
  │ Config language  │ YAML              │ YAML               │ Groovy       │
  │ Execution        │ Runners           │ Runners            │ Agents       │
  │ Parallelism      │ Same stage        │ Jobs (default)     │ parallel {}  │
  │ Reusability      │ include + extends │ Reusable workflows │ Shared Libs  │
  │ Secrets          │ CI/CD Variables   │ Secrets & Vars     │ Credentials  │
  │ Container reg    │ Built-in          │ GHCR               │ Plugin       │
  │ Free tier        │ 400 min/month     │ 2000 min/month     │ Self-hosted  │
  └──────────────────┴───────────────────┴────────────────────┴──────────────┘
```

---

## Real-World Scenario

> **Scenario**: A team wants to set up CI/CD for a Node.js project with lint, test, build, and deploy stages.
>
> **Solution**: Create `.gitlab-ci.yml` with 4 stages. Use `node:20` image globally. Lint and test run in parallel in the `test` stage. Build produces artifacts passed to deploy. Deploy job uses `rules:` to run only on `main` with manual trigger. Include GitLab SAST template for security scanning.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Pipeline not created | Invalid YAML syntax | Use CI Lint tool (CI/CD → Editor → Lint) |
| Job stuck "pending" | No available runner | Register a runner or check runner tags |
| Variables not expanding | Wrong syntax or scope | Use `$VARIABLE` in scripts, `${VAR}` in YAML values |
| `only/except` ignored | `rules:` takes precedence | Use `rules:` exclusively (don't mix with only/except) |
| Artifacts not passed | Different stages or expired | Check `expire_in` and ensure jobs are in sequential stages |
| Include file not found | Wrong path or permissions | Verify project access and file path |

---

## Summary Table

| Element | Purpose |
|---|---|
| `stages:` | Define pipeline stage order |
| `image:` | Docker image for job execution |
| `variables:` | Environment variables (global or per-job) |
| `script:` | Commands to execute in the job |
| `rules:` | Conditional job execution (replaces only/except) |
| `artifacts:` | Files to pass between stages |
| `include:` | Import external CI configuration |
| `before_script:` / `after_script:` | Pre/post job commands |

---

## Quick Revision Questions

1. **Where is the GitLab CI/CD configuration stored?** *In `.gitlab-ci.yml` at the repository root.*
2. **How do jobs within the same stage execute?** *In parallel — all jobs in a stage run simultaneously, and the next stage starts only when all pass.*
3. **What replaced `only:` and `except:`?** *`rules:` — more powerful conditional execution with `if:`, `changes:`, `exists:`, and `when:` keywords.*
4. **How do you share configuration across projects?** *Use `include:` with `project:` to import CI templates from another GitLab repository.*
5. **What is a GitLab Runner?** *An agent that picks up and executes CI/CD jobs — can run in Docker, Kubernetes, shell, or virtual machine.*
6. **How do you validate `.gitlab-ci.yml` before pushing?** *Use the CI Lint tool in GitLab (CI/CD → Editor) or the API endpoint `/ci/lint`.*

---

[← Previous: GitHub Actions Best Practices](../07-GitHub-Actions/08-github-actions-best-practices.md) | [Back to README](../README.md) | [Next: Stages & Jobs →](02-stages-jobs.md)

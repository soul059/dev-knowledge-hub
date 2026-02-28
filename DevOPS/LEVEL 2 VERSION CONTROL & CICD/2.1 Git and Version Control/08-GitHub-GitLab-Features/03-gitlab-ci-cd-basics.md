# Chapter 8.3: GitLab CI/CD Basics

[← Previous: GitHub Actions Basics](02-github-actions-basics.md) | [Back to README](../README.md) | [Next: Wikis and Documentation →](04-wikis-and-documentation.md)

---

## Overview

**GitLab CI/CD** is a built-in continuous integration and delivery platform. Pipelines are defined in a single `.gitlab-ci.yml` file at the root of your repository. When code is pushed or a merge request is created, GitLab automatically runs the pipeline.

---

## Core Concepts

```
┌──────────────────────────────────────────────────────────────┐
│       GITLAB CI/CD ARCHITECTURE                             │
│                                                              │
│  .gitlab-ci.yml (Pipeline Definition)                       │
│    │                                                        │
│    ├── STAGE: build                                         │
│    │     └── JOB: compile                                   │
│    │           ├── script: npm ci                            │
│    │           └── script: npm run build                     │
│    │                                                        │
│    ├── STAGE: test                                          │
│    │     ├── JOB: unit-tests                                │
│    │     │     └── script: npm test                          │
│    │     └── JOB: lint                                      │
│    │           └── script: npm run lint                      │
│    │                                                        │
│    └── STAGE: deploy                                        │
│          └── JOB: deploy-prod                               │
│                └── script: ./deploy.sh                       │
│                                                              │
│  STAGES run sequentially; JOBS within a stage run parallel  │
│  Jobs run on RUNNERS (shared, group, or project-specific)   │
└──────────────────────────────────────────────────────────────┘
```

| Concept | Description |
|---------|-------------|
| **Pipeline** | Top-level container for all stages and jobs |
| **Stage** | Group of jobs that run in sequence |
| **Job** | Individual unit of work with scripts |
| **Runner** | Agent that executes jobs (shared or self-hosted) |
| **Artifact** | Files produced by jobs, passed between stages |

---

## Your First Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

unit-tests:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+\.?\d*)%/'

lint:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm run lint

deploy-production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - ./deploy.sh
  environment:
    name: production
    url: https://myapp.com
  only:
    - main
  when: manual
```

---

## Pipeline Flow

```
┌──────────────────────────────────────────────────────────────┐
│  PIPELINE EXECUTION                                         │
│                                                              │
│  Stage: build          Stage: test          Stage: deploy   │
│  ┌──────────────┐      ┌──────────────┐     ┌────────────┐ │
│  │   compile    │      │  unit-tests  │     │  deploy    │ │
│  │   ✅ 45s      │ ──►  │  ✅ 2m        │ ──► │  ✅ 30s     │ │
│  └──────────────┘      │              │     └────────────┘ │
│                        │  lint        │                     │
│                        │  ✅ 30s       │                     │
│                        └──────────────┘                     │
│                        (parallel within                     │
│                         same stage)                         │
│                                                              │
│  If ANY job in a stage fails, the pipeline stops.           │
│  Later stages will NOT run.                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Key Configuration Options

### Variables

```yaml
variables:
  NODE_ENV: production
  DATABASE_URL: $CI_DATABASE_URL    # CI/CD variable (Settings)

build:
  script:
    - echo "Building for $NODE_ENV"
```

### Rules and Conditions

```yaml
deploy-staging:
  stage: deploy
  script: ./deploy.sh staging
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"
      when: always
    - if: $CI_MERGE_REQUEST_ID
      when: never

deploy-production:
  stage: deploy
  script: ./deploy.sh production
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual                    # Requires manual click
    - when: never
```

### Caching

```yaml
build:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run build
```

### Artifacts (Passing data between stages)

```yaml
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:
  stage: deploy
  script:
    - ls dist/           # dist/ is available from build stage!
    - ./deploy.sh
```

---

## Docker-Based Jobs

```yaml
# Use any Docker image as the job environment
test-python:
  image: python:3.12
  script:
    - pip install -r requirements.txt
    - pytest

test-go:
  image: golang:1.22
  script:
    - go test ./...

# Using Docker-in-Docker for building images
build-image:
  image: docker:24
  services:
    - docker:24-dind
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  script:
    - docker build -t myapp:$CI_COMMIT_SHA .
    - docker push myapp:$CI_COMMIT_SHA
```

---

## Environments and Deployment

```yaml
deploy-staging:
  stage: deploy
  environment:
    name: staging
    url: https://staging.myapp.com
  script:
    - ./deploy.sh staging

deploy-production:
  stage: deploy
  environment:
    name: production
    url: https://myapp.com
  when: manual                # Manual approval required
  only:
    - main
```

```
┌──────────────────────────────────────────────────────────────┐
│  GITLAB ENVIRONMENTS                                        │
│                                                              │
│  Operations → Environments                                  │
│                                                              │
│  ┌─────────────┬────────────┬─────────────────────────────┐ │
│  │ Environment │ Status     │ Last Deployment             │ │
│  ├─────────────┼────────────┼─────────────────────────────┤ │
│  │ production  │ ✅ Active   │ v1.2.0 — 2h ago by @alice  │ │
│  │ staging     │ ✅ Active   │ v1.3.0-rc — 30m ago        │ │
│  │ review/42   │ 🔄 Active   │ PR #42 — auto-deployed     │ │
│  └─────────────┴────────────┴─────────────────────────────┘ │
│                                                              │
│  Features: deployment history, rollback, monitoring URL    │
└──────────────────────────────────────────────────────────────┘
```

---

## Predefined CI/CD Variables

| Variable | Description |
|----------|-------------|
| `CI_COMMIT_SHA` | Full commit SHA |
| `CI_COMMIT_BRANCH` | Branch name |
| `CI_COMMIT_TAG` | Tag name (if triggered by tag) |
| `CI_MERGE_REQUEST_ID` | MR ID (if triggered by MR) |
| `CI_PIPELINE_ID` | Pipeline ID |
| `CI_PROJECT_NAME` | Project name |
| `CI_REGISTRY_IMAGE` | Container registry image path |
| `CI_ENVIRONMENT_NAME` | Current environment name |

---

## GitHub Actions vs GitLab CI/CD

| Feature | GitHub Actions | GitLab CI/CD |
|---------|---------------|-------------|
| Config file | `.github/workflows/*.yml` | `.gitlab-ci.yml` |
| Multiple files | Yes (one per workflow) | Single file (with includes) |
| Execution model | Jobs in parallel by default | Stages sequential, jobs in stage parallel |
| Marketplace | GitHub Marketplace | No marketplace (use scripts/images) |
| Runner types | GitHub-hosted, self-hosted | Shared, group, project runners |
| Docker support | Via actions | Native (image: per job) |
| Manual approval | Environments | `when: manual` |
| Included CI templates | — | Auto DevOps, templates library |

---

## Real-World Scenarios

### Scenario: Node.js Application Pipeline

```yaml
stages:
  - install
  - quality
  - build
  - deploy

install:
  stage: install
  image: node:20
  script:
    - npm ci
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths: [node_modules/]

lint:
  stage: quality
  image: node:20
  script: npm run lint

test:
  stage: quality
  image: node:20
  script: npm test -- --coverage
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: node:20
  script: npm run build
  artifacts:
    paths: [dist/]

deploy:
  stage: deploy
  script: ./deploy.sh
  environment: { name: production, url: https://myapp.com }
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Pipeline not triggering | Check `rules:` / `only:` / `except:` conditions |
| YAML syntax errors | Use GitLab's CI Lint tool (CI/CD → Editor → Validate) |
| Job stuck "pending" | No available runner; check runner registration |
| Artifacts not found in next stage | Ensure `artifacts: paths:` is set; check expiration |
| Cache not working | Verify `cache: key:` is consistent; check runner cache config |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| `.gitlab-ci.yml` | Pipeline definition file (repo root) |
| Stage | Sequential phase (build → test → deploy) |
| Job | Parallel unit within a stage |
| Runner | Agent executing jobs |
| `image:` | Docker image for job environment |
| `artifacts:` | Files passed between stages |
| `cache:` | Reused files across pipelines |
| `rules:` / `only:` | Conditional job execution |
| `when: manual` | Require manual trigger |
| `environment:` | Deployment target tracking |

---

## Quick Revision Questions

1. **What is the execution order in a GitLab CI/CD pipeline?**
   <details><summary>Answer</summary>Stages run sequentially in the order defined. Jobs within the same stage run in parallel. If any job in a stage fails, subsequent stages do not run (by default). This is different from GitHub Actions where jobs are parallel by default.</details>

2. **Where is the GitLab CI/CD pipeline defined?**
   <details><summary>Answer</summary>In a `.gitlab-ci.yml` file at the root of the repository. Unlike GitHub Actions (which uses multiple YAML files in `.github/workflows/`), GitLab uses a single file (though it supports `include:` to split configuration).</details>

3. **How do you pass files between stages?**
   <details><summary>Answer</summary>Use `artifacts:` to define files produced by a job. These are automatically available to jobs in later stages. Set `paths:` to specify which files/directories to keep, and optionally `expire_in:` to set retention. Cache is for dependencies, artifacts for build outputs.</details>

4. **What does `when: manual` do?**
   <details><summary>Answer</summary>It makes a job require manual trigger (a button click in the GitLab UI) instead of running automatically. This is commonly used for production deployments to require human approval before deploying.</details>

5. **How is GitLab CI/CD different from GitHub Actions?**
   <details><summary>Answer</summary>GitLab uses a single `.gitlab-ci.yml` with stages (sequential) containing jobs (parallel). GitHub Actions uses multiple workflow files with jobs (parallel by default, `needs:` for sequential) containing steps. GitLab has native Docker support (`image:` per job), while GitHub Actions uses a Marketplace of reusable actions.</details>

---

[← Previous: GitHub Actions Basics](02-github-actions-basics.md) | [Back to README](../README.md) | [Next: Wikis and Documentation →](04-wikis-and-documentation.md)

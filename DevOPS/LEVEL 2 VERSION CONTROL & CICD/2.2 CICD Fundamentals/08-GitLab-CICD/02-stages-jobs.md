# Chapter 8.2: GitLab CI/CD Stages & Jobs

[← Previous: GitLab CI Configuration](01-gitlab-ci-yml.md) | [Back to README](../README.md) | [Next: GitLab Runners →](03-gitlab-runners.md)

---

## Overview

**Stages** define the order of pipeline execution, while **jobs** are the individual tasks that run within each stage. Understanding their relationship is key to building efficient GitLab pipelines.

---

## Stage Execution Flow

```
  STAGE EXECUTION MODEL
  ═════════════════════

  Stage: build          Stage: test          Stage: deploy
  ┌──────────────┐      ┌──────────────┐     ┌──────────────┐
  │  compile     │      │  unit-test   │     │  staging     │
  │  (parallel)  │      │  (parallel)  │     │  (sequential)│
  ├──────────────┤  ──► ├──────────────┤ ──► ├──────────────┤
  │  build-image │      │  lint        │     │  production  │
  │  (parallel)  │      │  (parallel)  │     │  (manual)    │
  └──────────────┘      ├──────────────┤     └──────────────┘
                        │  security    │
                        │  (parallel)  │
                        └──────────────┘

  ✓ Jobs in same stage → PARALLEL
  ✓ Stages → SEQUENTIAL (next starts when all jobs in current pass)
  ✓ One job fails → Stage fails → Pipeline fails (unless allow_failure)
```

---

## Defining Stages

```yaml
# Custom stage order
stages:
  - install
  - lint
  - test
  - build
  - deploy

# If stages: is omitted, defaults are:
# - .pre (always first)
# - build
# - test
# - deploy
# - .post (always last)
```

---

## Job Configuration

```yaml
unit-test:
  stage: test
  image: node:20
  tags:
    - docker                    # Runner tag requirement
  timeout: 15 minutes           # Job-level timeout
  retry:
    max: 2                      # Retry on failure
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
  variables:
    NODE_ENV: test
  before_script:
    - npm ci
  script:
    - npm test -- --ci --coverage
  after_script:
    - echo "Tests completed"    # Runs even on failure
  artifacts:
    paths:
      - coverage/
    reports:
      junit: reports/junit.xml
    expire_in: 7 days
  coverage: '/Statements\s*:\s*(\d+\.?\d*)%/'
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
```

---

## Job Dependencies

### `needs:` (DAG — Directed Acyclic Graph)

```yaml
stages:
  - build
  - test
  - deploy

build-frontend:
  stage: build
  script: npm run build
  artifacts:
    paths: [dist/]

build-backend:
  stage: build
  script: mvn package
  artifacts:
    paths: [target/]

test-frontend:
  stage: test
  needs: [build-frontend]      # Starts as soon as build-frontend finishes
  script: npm test              # Does NOT wait for build-backend

test-backend:
  stage: test
  needs: [build-backend]       # Starts as soon as build-backend finishes
  script: mvn test

deploy:
  stage: deploy
  needs: [test-frontend, test-backend]
  script: ./deploy.sh
```

```
  WITHOUT needs: (SEQUENTIAL STAGES)
  ═══════════════════════════════════
  build-frontend ─┐
                   ├─ WAIT ─► test-frontend ─┐
  build-backend  ─┘           test-backend  ─┴─► deploy
  Time: ████████████████████████████████████████████

  WITH needs: (DAG)
  ═════════════════
  build-frontend ──► test-frontend ──┐
                                      ├──► deploy
  build-backend  ──► test-backend  ──┘
  Time: ██████████████████████████
        (faster — parallel paths)
```

---

## Extends & YAML Anchors (DRY)

### `extends:`

```yaml
# Template (hidden job — starts with .)
.node-defaults:
  image: node:20
  before_script:
    - npm ci
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

# Jobs inherit from template
lint:
  extends: .node-defaults
  stage: lint
  script: npm run lint

test:
  extends: .node-defaults
  stage: test
  script: npm test

build:
  extends: .node-defaults
  stage: build
  script: npm run build
```

### YAML Anchors

```yaml
# Define reusable blocks
.deploy-template: &deploy-template
  image: alpine:3.19
  before_script:
    - apk add --no-cache curl
  script:
    - ./deploy.sh $ENVIRONMENT

deploy-staging:
  <<: *deploy-template
  stage: deploy
  variables:
    ENVIRONMENT: staging
  environment:
    name: staging

deploy-production:
  <<: *deploy-template
  stage: deploy
  variables:
    ENVIRONMENT: production
  environment:
    name: production
  when: manual
```

---

## Parallel Execution

### `parallel:` Keyword

```yaml
rspec-test:
  stage: test
  parallel: 5                  # Creates 5 parallel job instances
  script:
    - |
      # Split tests across parallel instances
      bundle exec rspec \
        --format progress \
        $(circleci tests glob "spec/**/*_spec.rb" | \
          circleci tests split --split-by=timings \
          --index=$CI_NODE_INDEX --total=$CI_NODE_TOTAL)
```

### `parallel: matrix:`

```yaml
test:
  stage: test
  parallel:
    matrix:
      - PLATFORM: [linux, windows, macos]
        NODE_VERSION: ['18', '20', '22']
  image: node:${NODE_VERSION}
  script:
    - echo "Testing on ${PLATFORM} with Node ${NODE_VERSION}"
    - npm test
  tags:
    - ${PLATFORM}
```

---

## Manual & Delayed Jobs

```yaml
deploy-staging:
  stage: deploy
  script: ./deploy.sh staging
  when: manual                  # Requires click in UI to run
  allow_failure: false          # Pipeline BLOCKED until clicked

deploy-canary:
  stage: deploy
  script: ./deploy.sh canary
  when: delayed                 # Auto-starts after delay
  start_in: 30 minutes
```

```
  MANUAL JOB STATES
  ═════════════════

  when: manual + allow_failure: true (default)
  └── Pipeline continues, job is optional "play" button

  when: manual + allow_failure: false
  └── Pipeline BLOCKED — waits for manual trigger
      (acts like an approval gate)
```

---

## Resource Groups

```yaml
# Prevent concurrent deployments to same environment
deploy-staging:
  stage: deploy
  script: ./deploy.sh staging
  resource_group: staging       # Only one job at a time
  environment:
    name: staging

deploy-production:
  stage: deploy
  script: ./deploy.sh production
  resource_group: production
  environment:
    name: production
```

---

## Real-World Scenario

> **Scenario**: A full-stack app (frontend + backend) wants fast CI without waiting for unrelated builds.
>
> **Solution**: Use `needs:` for DAG pipelines. `build-frontend` and `build-backend` run in parallel. `test-frontend needs: [build-frontend]` starts immediately when frontend build completes, without waiting for backend build. Same for `test-backend needs: [build-backend]`. Deploy requires both tests. Total pipeline time reduced by ~40%.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Jobs not running in parallel | Different stages | Jobs must be in the SAME stage for parallel execution |
| Pipeline stuck on manual job | `allow_failure: false` on manual job | Click "Play" button or set `allow_failure: true` |
| Artifacts not available | Job not in `needs:` list | Add dependency to `needs:` or rely on stage order |
| `extends:` not merging correctly | Deep nesting conflict | Check merge order — deeper keys override |
| DAG cycle detected | Circular `needs:` dependencies | Remove circular references in job graph |
| `parallel:` jobs all fail | CI_NODE_INDEX not handled | Use parallel index variables to split work |

---

## Summary Table

| Feature | Purpose |
|---|---|
| `stages:` | Define execution order |
| `stage:` (on job) | Assign job to a stage |
| `needs:` | DAG dependencies (skip waiting for full stage) |
| `extends:` | Inherit from template jobs |
| `parallel:` | Run multiple instances of a job |
| `when: manual` | Require manual trigger |
| `when: delayed` | Auto-run after a time delay |
| `resource_group:` | Prevent concurrent runs |
| `allow_failure:` | Let pipeline continue on job failure |

---

## Quick Revision Questions

1. **How do jobs within the same stage execute?** *In parallel — GitLab runs all jobs in a stage simultaneously.*
2. **What does `needs:` do?** *Creates a DAG (Directed Acyclic Graph) — jobs start as soon as their specific dependencies finish, without waiting for the entire previous stage.*
3. **What is a hidden job in GitLab CI?** *A job name starting with `.` (e.g., `.template`) — it's not executed but can be referenced with `extends:`.*
4. **How do you create an approval gate?** *Use `when: manual` with `allow_failure: false` — the pipeline blocks until someone clicks the play button.*
5. **What does `resource_group:` prevent?** *Concurrent execution — only one job with the same resource group name runs at a time (e.g., preventing parallel deployments).*
6. **How does `parallel: 5` work?** *Creates 5 instances of the job with `CI_NODE_INDEX` (0-4) and `CI_NODE_TOTAL` (5) variables for splitting work.*

---

[← Previous: GitLab CI Configuration](01-gitlab-ci-yml.md) | [Back to README](../README.md) | [Next: GitLab Runners →](03-gitlab-runners.md)

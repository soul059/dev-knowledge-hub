# Chapter 8.8: GitLab CI/CD Best Practices

[← Previous: Auto DevOps](07-auto-devops.md) | [Back to README](../README.md) | [Next: Azure Pipelines →](../09-Other-CICD-Tools/01-azure-pipelines.md)

---

## Overview

This chapter consolidates GitLab CI/CD best practices for pipeline optimization, maintainability, security, and team collaboration.

---

## Pipeline Architecture

### Use DAG Over Sequential Stages

```
  SEQUENTIAL (SLOW)              DAG WITH needs: (FAST)
  ═════════════════              ══════════════════════

  build-frontend ─┐              build-frontend ──▶ test-frontend
  build-backend  ─┤ (wait)       build-backend  ──▶ test-backend
                   ▼                                     │
  test-frontend  ─┐              deploy ◀────────────────┘
  test-backend   ─┤ (wait)
                   ▼
  deploy
  
  Total: sum of all stages       Total: longest path only
```

```yaml
# Bad: Sequential stages — each waits for all previous
stages: [build, test, deploy]

build-frontend:
  stage: build
  script: npm run build

build-backend:
  stage: build
  script: mvn package

test-frontend:
  stage: test
  script: npm test

test-backend:
  stage: test
  script: mvn test

# Good: DAG — explicit dependencies
build-frontend:
  stage: build
  script: npm run build

test-frontend:
  stage: test
  script: npm test
  needs: [build-frontend]          # Starts as soon as frontend build finishes

build-backend:
  stage: build
  script: mvn package

test-backend:
  stage: test
  script: mvn test
  needs: [build-backend]           # Independent of frontend

deploy:
  stage: deploy
  needs: [test-frontend, test-backend]
```

---

## DRY Pipelines with Templates

### Use `extends` and Hidden Jobs

```yaml
# Hidden job as template (starts with .)
.deploy-template:
  image: bitnami/kubectl:latest
  before_script:
    - kubectl config use-context $KUBE_CONTEXT
  script:
    - kubectl set image deployment/$APP $APP=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
    - kubectl rollout status deployment/$APP

deploy-staging:
  extends: .deploy-template
  variables:
    KUBE_CONTEXT: staging
    APP: myapp-staging
  environment:
    name: staging

deploy-production:
  extends: .deploy-template
  variables:
    KUBE_CONTEXT: production
    APP: myapp-production
  environment:
    name: production
  when: manual
```

### Use `include` for Shared Templates

```yaml
# ci-templates/security.yml (shared repo)
.security-scan:
  stage: test
  image: security-scanner:latest
  script:
    - scan --project $CI_PROJECT_DIR
  artifacts:
    reports:
      sast: gl-sast-report.json

# Project .gitlab-ci.yml
include:
  - project: 'devops/ci-templates'
    ref: main
    file:
      - '/templates/docker-build.yml'
      - '/templates/security.yml'
      - '/templates/deploy.yml'
```

---

## Caching & Artifacts Strategy

```
  OPTIMAL STRATEGY
  ════════════════

  Cache (speed up builds):
  ├── node_modules/, .npm/
  ├── .m2/repository/
  ├── pip cache
  └── Key by lock file hash

  Artifacts (pass between jobs):
  ├── Build outputs (dist/, target/)
  ├── Test reports (JUnit XML)
  └── Expire after 1 week max
```

```yaml
# Efficient caching pattern
variables:
  NPM_CONFIG_CACHE: "$CI_PROJECT_DIR/.npm"

.node-cache:
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - .npm/
    policy: pull-push

build:
  extends: .node-cache
  script:
    - npm ci --cache .npm
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour              # Short-lived — only for downstream jobs

test:
  extends: .node-cache
  cache:
    policy: pull                    # Don't write back — just consume
  script: npm test
  artifacts:
    reports:
      junit: test-results.xml
    expire_in: 30 days             # Keep test reports longer
```

---

## Rules Best Practices

```yaml
# BAD: Using deprecated only/except
deploy:
  only:
    - main
  except:
    - tags

# GOOD: Using rules
deploy:
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_TAG
      when: never

# GOOD: Workflow rules for pipeline-level control
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_TAG

# Prevents duplicate pipelines:
# Both MR pipeline and branch pipeline would run
# workflow:rules ensures only one pipeline type
```

### Avoid Duplicate Pipelines

```yaml
# Problem: MR and branch pipelines both trigger
# Solution: Use workflow:rules

workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS
      when: never                    # Skip branch pipeline if MR exists
    - if: $CI_COMMIT_BRANCH

# OR use the simpler approach:
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```

---

## Security Best Practices

```yaml
# 1. Use protected variables for secrets
# Settings → CI/CD → Variables → Protected: Yes, Masked: Yes

# 2. Pin image versions
# Bad:
image: node:latest

# Good:
image: node:20.11.0-alpine@sha256:abc123...

# 3. Restrict runner access
# Use tags to route sensitive jobs to secure runners
production-deploy:
  tags:
    - secure-runner
    - production

# 4. Use OIDC for cloud auth (no long-lived credentials)
deploy-aws:
  id_tokens:
    AWS_TOKEN:
      aud: https://aws.example.com
  script:
    - >
      STS_CREDENTIALS=$(aws sts assume-role-with-web-identity
        --role-arn $ROLE_ARN
        --web-identity-token $AWS_TOKEN
        --role-session-name gitlab-ci)

# 5. Scan dependencies and containers
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml
```

---

## Pipeline Performance

```
  OPTIMIZATION CHECKLIST
  ══════════════════════

  ✓ Use needs: for DAG (reduce wait time)
  ✓ Use cache with file-based keys
  ✓ Use small base images (alpine)
  ✓ Set artifact expire_in (reduce storage)
  ✓ Use interruptible: true (cancel outdated)
  ✓ Use rules: changes: (skip unchanged)
  ✓ Parallelize tests with parallel: keyword
  ✓ Use resource_group for deploy serialization
```

```yaml
# Optimized job template
.optimized-job:
  interruptible: true               # Cancel if new commit pushed
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure

# Skip jobs when files unchanged
test-frontend:
  rules:
    - changes:
        - "frontend/**"
        - "package*.json"

test-backend:
  rules:
    - changes:
        - "backend/**"
        - "pom.xml"

# Parallel test splitting
test:
  parallel: 4
  script:
    - TESTS=$(find tests/ -name "*.test.js" | sort | awk "NR % $CI_NODE_TOTAL == $CI_NODE_INDEX - 1")
    - npx jest $TESTS
```

---

## Monitoring & Observability

```yaml
# Use CI/CD analytics
# Analyze → CI/CD Analytics shows:
# • Pipeline success rate
# • Pipeline duration trends
# • Job failure breakdown

# Structured logging in pipelines
.logging:
  before_script:
    - echo "PIPELINE_ID=$CI_PIPELINE_ID"
    - echo "JOB_ID=$CI_JOB_ID"
    - echo "COMMIT=$CI_COMMIT_SHORT_SHA"
    - echo "BRANCH=$CI_COMMIT_REF_NAME"
    - echo "STARTED_AT=$(date -u +%Y-%m-%dT%H:%M:%SZ)"

# Notification on failure
notify-slack:
  stage: .post                      # Runs after all stages
  script:
    - |
      curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"Pipeline failed: $CI_PROJECT_NAME ($CI_COMMIT_REF_NAME)\"}" \
        $SLACK_WEBHOOK_URL
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: on_failure
```

---

## Project Organization

```
  RECOMMENDED STRUCTURE
  ═════════════════════

  project/
  ├── .gitlab-ci.yml                # Main pipeline (includes only)
  ├── .gitlab/
  │   ├── ci/
  │   │   ├── build.yml             # Build stage jobs
  │   │   ├── test.yml              # Test stage jobs
  │   │   ├── security.yml          # Security scan jobs
  │   │   └── deploy.yml            # Deploy stage jobs
  │   └── CODEOWNERS                # Protect CI config
  ├── src/
  └── tests/
```

```yaml
# .gitlab-ci.yml — clean top-level config
stages:
  - build
  - test
  - security
  - deploy

include:
  - local: '.gitlab/ci/build.yml'
  - local: '.gitlab/ci/test.yml'
  - local: '.gitlab/ci/security.yml'
  - local: '.gitlab/ci/deploy.yml'

# Protect CI config with CODEOWNERS
# .gitlab/CODEOWNERS:
# .gitlab-ci.yml @devops-team
# .gitlab/ci/ @devops-team
```

---

## Anti-Patterns to Avoid

```
  ╔══════════════════════════════════════════════════════════╗
  ║ Anti-Pattern           │ Better Approach                ║
  ╠══════════════════════════════════════════════════════════╣
  ║ Giant single job       │ Split into focused stages/jobs ║
  ║ only/except keywords   │ Use rules: with if/changes     ║
  ║ No artifact expiry     │ Set expire_in: on all artifacts║
  ║ Secrets in scripts     │ Use CI/CD variables (masked)   ║
  ║ latest image tags      │ Pin image versions + digest    ║
  ║ No caching             │ Cache dependencies by lock file║
  ║ Script in YAML         │ Move complex logic to scripts/ ║
  ║ No interruptible       │ Set interruptible: true        ║
  ║ Manual copy-paste      │ Use extends/include templates  ║
  ║ No pipeline analytics  │ Monitor duration + failure rate║
  ╚══════════════════════════════════════════════════════════╝
```

---

## Real-World Scenario

> **Scenario**: A team has 50+ GitLab projects with duplicated pipeline configs. Pipelines are slow (20+ minutes) and hard to maintain.
>
> **Solution**: Create a shared `ci-templates` project with standardized templates for build, test, security, and deploy stages. Each project includes templates via `include: project:`. Switch from sequential stages to DAG with `needs:`. Add file-based cache keys and `interruptible: true`. Use `rules: changes:` in the monorepo to skip unchanged services. Result: pipelines drop to 8 minutes, config is DRY across all projects, and security scans are enforced organization-wide.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Duplicate pipelines | Both MR and branch triggers | Use `workflow:rules` to deduplicate |
| Slow pipelines | Sequential stages, no caching | Use DAG (`needs:`), caching, small images |
| Config duplication | Copy-paste across projects | Use `include:` with shared template repo |
| Failed deploys block pipeline | No serialization | Use `resource_group` for deploy jobs |
| Stale artifacts filling storage | No `expire_in` | Set `expire_in` on all artifact definitions |
| Flaky test failures | Infrastructure issues | Use `retry: max: 2` with specific `when:` |

---

## Summary Table

| Best Practice | Implementation |
|---|---|
| DAG pipelines | `needs:` to define job dependencies |
| DRY templates | `extends:` + `include:` with shared repos |
| Smart caching | File-based keys, pull/pull-push policies |
| Modern rules | `rules:` instead of `only/except` |
| Security | Masked vars, pinned images, OIDC auth |
| Performance | `interruptible:`, `parallel:`, `changes:` |
| Monitoring | CI/CD Analytics, Slack notifications |
| Organization | Split config into `.gitlab/ci/` files |

---

## Quick Revision Questions

1. **How does `needs:` improve pipeline speed?** *It creates a DAG, allowing jobs to start as soon as their specific dependencies finish rather than waiting for the entire previous stage.*
2. **How do you prevent duplicate MR and branch pipelines?** *Use `workflow:rules` to run MR pipelines when an MR exists and skip the branch pipeline, or only run branch pipelines on the default branch.*
3. **What is the recommended caching strategy?** *Use file-based cache keys (e.g., keyed on `package-lock.json`), separate cache paths from artifacts, and use `pull` policy on test jobs that only consume cached data.*
4. **How do you share pipeline configuration across projects?** *Use `include: project:` to reference templates from a dedicated CI templates repository, combined with `extends:` for job-level inheritance.*
5. **What security practices should CI/CD pipelines follow?** *Use masked/protected variables, pin image versions with SHA digests, use OIDC for cloud auth, route sensitive jobs to secure runners, and include security scanning templates.*
6. **How do you organize a complex `.gitlab-ci.yml`?** *Split jobs into separate files under `.gitlab/ci/` and include them from the main config. Protect CI files with CODEOWNERS.*

---

[← Previous: Auto DevOps](07-auto-devops.md) | [Back to README](../README.md) | [Next: Azure Pipelines →](../09-Other-CICD-Tools/01-azure-pipelines.md)

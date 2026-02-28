# Chapter 8.6: GitLab Environments & Review Apps

[← Previous: Artifacts & Caching](05-artifacts-caching.md) | [Back to README](../README.md) | [Next: GitLab Auto DevOps →](07-auto-devops.md)

---

## Overview

GitLab **environments** track deployments, and **review apps** automatically create temporary environments for merge requests — enabling live previews of every change.

---

## Environments

```
  ENVIRONMENT LIFECYCLE
  ═════════════════════

  Deployment History (visible in GitLab UI):
  
  production:
  ├── #12 Active    main@abc1234  Jan 15  @alice
  ├── #11 Inactive  main@def5678  Jan 14  @bob
  └── #10 Inactive  main@ghi9012  Jan 13  @alice

  staging:
  ├── #25 Active    main@abc1234  Jan 15  @ci
  └── #24 Inactive  main@def5678  Jan 14  @ci

  review/feature-auth:
  └── #5  Active    feature-auth  Jan 15  @charlie
      └── Auto-stops when MR is merged
```

### Defining Environments

```yaml
deploy-staging:
  stage: deploy
  script: ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop-staging           # Cleanup job
    auto_stop_in: 1 week            # Auto-stop after period

deploy-production:
  stage: deploy
  script: ./deploy.sh production
  environment:
    name: production
    url: https://app.example.com
    deployment_tier: production      # Tier for ordering in UI
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual

stop-staging:
  stage: deploy
  script: ./teardown.sh staging
  environment:
    name: staging
    action: stop                    # Marks environment as stopped
  when: manual
```

---

## Review Apps

```
  REVIEW APP FLOW
  ════════════════

  Developer creates MR: feature/add-auth
       │
       ▼
  Pipeline creates review environment
  URL: https://review-add-auth.example.com
       │
       ▼
  Team reviews live app at that URL
  ├── Designer checks UI
  ├── PM tests feature
  └── Developer iterates
       │
       ▼
  MR merged → Review app auto-destroyed
```

### Review App Configuration

```yaml
stages:
  - build
  - test
  - review
  - staging
  - production

# Build Docker image
build:
  stage: build
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# Deploy review app for MRs
review:
  stage: review
  script:
    - |
      kubectl create namespace review-$CI_MERGE_REQUEST_IID || true
      helm upgrade --install \
        review-$CI_MERGE_REQUEST_IID ./chart \
        --namespace review-$CI_MERGE_REQUEST_IID \
        --set image=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA \
        --set ingress.host=review-${CI_COMMIT_REF_SLUG}.example.com
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://review-${CI_COMMIT_REF_SLUG}.example.com
    on_stop: stop-review
    auto_stop_in: 3 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# Cleanup review app
stop-review:
  stage: review
  script:
    - kubectl delete namespace review-$CI_MERGE_REQUEST_IID
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
  allow_failure: true
```

---

## Deployment Tiers

```
  ┌──────────────┬────────────────────────────────────┐
  │ Tier         │ Examples                           │
  ├──────────────┼────────────────────────────────────┤
  │ production   │ Production, Live                   │
  │ staging      │ Staging, Pre-prod, Model           │
  │ testing      │ QA, Test, Integration              │
  │ development  │ Dev, Review apps                   │
  │ other        │ Anything else                      │
  └──────────────┴────────────────────────────────────┘

  Used for:
  • Ordering environments in the UI
  • Environment-specific variable scoping
  • Dashboard organization
```

---

## Protected Environments

```
  PROTECTED ENVIRONMENT: production
  ═════════════════════════════════

  Settings → CI/CD → Protected environments

  ┌────────────────────────────────────────┐
  │ Environment: production                │
  │                                        │
  │ Allowed to deploy:                     │
  │ ├── Maintainers                        │
  │ ├── @ops-team group                    │
  │ └── @alice (specific user)             │
  │                                        │
  │ Required approvals: 1                  │
  │ Approval rules:                        │
  │ └── @tech-lead must approve            │
  └────────────────────────────────────────┘
```

---

## Multi-Environment Pipeline

```yaml
stages:
  - build
  - test
  - review
  - staging
  - production

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

test:
  stage: test
  script: npm test

# Review app — MR only
review:
  stage: review
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://review-${CI_COMMIT_REF_SLUG}.example.com
    on_stop: stop-review
    auto_stop_in: 2 days
  script: helm upgrade --install review-$CI_MERGE_REQUEST_IID ./chart
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

stop-review:
  stage: review
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  script: helm uninstall review-$CI_MERGE_REQUEST_IID
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual

# Staging — auto-deploy on main
deploy-staging:
  stage: staging
  environment:
    name: staging
    url: https://staging.example.com
  script: helm upgrade --install staging ./chart --set image.tag=$CI_COMMIT_SHORT_SHA
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

# Production — manual deploy on main
deploy-production:
  stage: production
  environment:
    name: production
    url: https://app.example.com
    deployment_tier: production
  script: helm upgrade --install production ./chart --set image.tag=$CI_COMMIT_SHORT_SHA
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
```

---

## Environment Monitoring

```yaml
# Link monitoring to environment
deploy-production:
  script: ./deploy.sh
  environment:
    name: production
    url: https://app.example.com
    kubernetes:
      namespace: production        # Links K8s metrics to env

# GitLab auto-detects:
# • Pod count and status
# • CPU / memory usage
# • Error rates (with Prometheus integration)
```

---

## Real-World Scenario

> **Scenario**: A team wants every merge request to have a live preview that designers and PMs can review.
>
> **Solution**: Configure review apps using Kubernetes + Helm. Each MR gets a unique namespace and subdomain (e.g., `review-feature-auth.example.com`). Use `auto_stop_in: 3 days` to clean up stale environments. When the MR is merged, the `stop-review` job runs automatically. Link the environment URL in the MR widget so reviewers can click directly to the preview.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Environment URL not in MR | Missing `url:` in environment config | Add `url:` to environment block |
| Review app not created | Wrong rules/conditions | Ensure `merge_request_event` trigger |
| Old review apps not cleaned | Missing `on_stop` + `auto_stop_in` | Configure both stop job and auto-stop timer |
| "not allowed to deploy" | Protected environment | Add user/group to allowed deployers |
| Environment not tracked | Missing `environment:` keyword | Add `environment:` block to deploy job |
| Rollback not working | No deployment history | GitLab tracks deployments; use Re-deploy button in UI |

---

## Summary Table

| Feature | Purpose |
|---|---|
| `environment: name:` | Track deployments to named targets |
| `environment: url:` | Link to deployed application |
| `environment: on_stop:` | Cleanup job when environment stops |
| `auto_stop_in:` | Auto-stop environment after time period |
| `action: stop` | Mark environment as stopped |
| `deployment_tier:` | Categorize environment (prod/staging/dev) |
| Review Apps | Temporary per-MR environments |
| Protected Environments | Restrict who can deploy |

---

## Quick Revision Questions

1. **What are GitLab environments?** *Named deployment targets (staging, production) that track deployment history, provide URLs, and integrate with monitoring.*
2. **What are review apps?** *Temporary environments created automatically for merge requests, providing a live preview of changes that is destroyed when the MR is merged.*
3. **How does `auto_stop_in` work?** *Automatically stops the environment after the specified period (e.g., `3 days`), triggering the `on_stop` job for cleanup.*
4. **What are protected environments?** *Environments that restrict deployments to specific users, groups, or roles — configured in project settings.*
5. **How do you create a dynamic environment name?** *Use CI variables: `environment: name: review/$CI_COMMIT_REF_SLUG` — creates a unique environment per branch.*
6. **Where can you see deployment history?** *In the GitLab UI under Operate → Environments, showing all deployments with status, commit, timestamp, and who deployed.*

---

[← Previous: Artifacts & Caching](05-artifacts-caching.md) | [Back to README](../README.md) | [Next: GitLab Auto DevOps →](07-auto-devops.md)

# Chapter 8.4: GitLab CI/CD Variables & Secrets

[← Previous: GitLab Runners](03-gitlab-runners.md) | [Back to README](../README.md) | [Next: Artifacts & Caching →](05-artifacts-caching.md)

---

## Overview

GitLab CI/CD uses **variables** for both configuration and secrets. Variables can be defined at multiple levels with built-in protection and masking features.

---

## Variable Hierarchy

```
  VARIABLE PRECEDENCE (highest wins)
  ═══════════════════════════════════

  ┌──────────────────────────────┐
  │ Job-level variables          │  ← Highest priority
  ├──────────────────────────────┤
  │ .gitlab-ci.yml variables     │
  ├──────────────────────────────┤
  │ Project CI/CD Variables      │
  ├──────────────────────────────┤
  │ Group CI/CD Variables        │
  ├──────────────────────────────┤
  │ Instance CI/CD Variables     │  ← Lowest priority
  └──────────────────────────────┘
```

---

## Defining Variables

### In .gitlab-ci.yml

```yaml
# Global variables
variables:
  NODE_VERSION: "20"
  APP_NAME: "my-service"
  DATABASE_URL: "postgres://localhost:5432/test"

# Job-level variables (override global)
test:
  variables:
    NODE_ENV: "test"
    LOG_LEVEL: "debug"
  script:
    - echo "Running in $NODE_ENV with Node $NODE_VERSION"
```

### In GitLab UI (for Secrets)

```
  Settings → CI/CD → Variables

  ┌──────────────────────────────────────────┐
  │ Key:    DOCKER_PASSWORD                  │
  │ Value:  ●●●●●●●●●●●●                    │
  │                                          │
  │ Options:                                 │
  │ ☑ Protect variable                       │
  │   └── Available only on protected        │
  │       branches/tags (main, release/*)    │
  │ ☑ Mask variable                          │
  │   └── Hidden in job logs                 │
  │ ☐ Expand variable reference              │
  │                                          │
  │ Environment scope: All (default)         │
  │   └── Or specific: production, staging   │
  │                                          │
  │ Type: Variable | File                    │
  └──────────────────────────────────────────┘
```

---

## Predefined Variables

```yaml
# Commonly used predefined variables
job:
  script:
    - echo "Pipeline:  $CI_PIPELINE_ID"
    - echo "Commit:    $CI_COMMIT_SHA"
    - echo "Short SHA: $CI_COMMIT_SHORT_SHA"
    - echo "Branch:    $CI_COMMIT_REF_NAME"
    - echo "Tag:       $CI_COMMIT_TAG"
    - echo "Project:   $CI_PROJECT_NAME"
    - echo "Namespace: $CI_PROJECT_NAMESPACE"
    - echo "Registry:  $CI_REGISTRY_IMAGE"
    - echo "MR ID:     $CI_MERGE_REQUEST_IID"
    - echo "Source:    $CI_PIPELINE_SOURCE"
    - echo "Job Token: $CI_JOB_TOKEN"
    - echo "API URL:   $CI_API_V4_URL"
```

| Variable | Description |
|---|---|
| `CI_COMMIT_SHA` | Full commit hash |
| `CI_COMMIT_SHORT_SHA` | First 8 chars of commit hash |
| `CI_COMMIT_REF_NAME` | Branch or tag name |
| `CI_COMMIT_TAG` | Tag name (empty if not a tag) |
| `CI_PIPELINE_SOURCE` | How triggered: push, merge_request_event, schedule, web |
| `CI_REGISTRY_IMAGE` | Docker registry path for project |
| `CI_JOB_TOKEN` | Token for GitLab API/registry access |
| `CI_MERGE_REQUEST_IID` | Merge request number |
| `CI_PROJECT_DIR` | Full path of the cloned repo |

---

## Variable Types

### Regular Variables

```yaml
variables:
  API_URL: "https://api.example.com"

job:
  script:
    - curl $API_URL/health
```

### File Variables

```
  FILE TYPE VARIABLES
  ═══════════════════

  Set in UI: Type = "File"
  Key: KUBECONFIG
  Value: (paste kubeconfig content)

  → GitLab writes content to a temporary file
  → Variable contains the FILE PATH (not content)
  → Automatically cleaned up after job
```

```yaml
deploy:
  script:
    # $KUBECONFIG is a path to a temp file containing the config
    - kubectl --kubeconfig=$KUBECONFIG apply -f deployment.yaml

    # Also works with tools that read env var as filepath
    - export GOOGLE_APPLICATION_CREDENTIALS=$GCP_SA_KEY
    - gcloud auth activate-service-account --key-file=$GCP_SA_KEY
```

---

## Protected & Masked Variables

```
  PROTECTION LEVELS
  ═════════════════

  PROTECTED VARIABLE:
  ├── Only available on protected branches/tags
  ├── Feature branches CANNOT access it
  ├── Prevents accidental exposure in MR pipelines
  └── Use for: production credentials

  MASKED VARIABLE:
  ├── Value hidden in job logs (replaced with [MASKED])
  ├── Must meet masking requirements:
  │   ├── At least 8 characters
  │   ├── Single line
  │   └── Only uses characters in Base64 alphabet
  └── Use for: any sensitive value

  BEST PRACTICE: BOTH protected AND masked for secrets
```

---

## Environment-Scoped Variables

```yaml
# UI Variables with environment scope:
# DEPLOY_TOKEN (scope: staging)   → "staging-token-xxx"
# DEPLOY_TOKEN (scope: production) → "production-token-yyy"

deploy-staging:
  stage: deploy
  script:
    - echo "Token: $DEPLOY_TOKEN"   # Gets staging-token-xxx
  environment:
    name: staging

deploy-production:
  stage: deploy
  script:
    - echo "Token: $DEPLOY_TOKEN"   # Gets production-token-yyy
  environment:
    name: production
```

---

## Dynamic Variables

```yaml
build:
  script:
    # Set variable for subsequent jobs via dotenv artifact
    - echo "VERSION=$(cat VERSION)" >> build.env
    - echo "IMAGE=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA" >> build.env
  artifacts:
    reports:
      dotenv: build.env            # Exports as variables

deploy:
  needs: [build]
  script:
    - echo "Deploying $IMAGE version $VERSION"
    # $IMAGE and $VERSION are available from dotenv artifact
```

---

## Vault Integration

```yaml
# HashiCorp Vault integration (Premium feature)
variables:
  VAULT_SERVER_URL: https://vault.example.com

job:
  id_tokens:
    VAULT_ID_TOKEN:
      aud: https://vault.example.com
  secrets:
    DATABASE_PASSWORD:
      vault: production/db/password@secret
      file: false
    API_KEY:
      vault: production/api/key@secret
  script:
    - echo "DB: $DATABASE_PASSWORD"
    - echo "API: $API_KEY"
```

---

## Real-World Scenario

> **Scenario**: A team deploys to staging and production with different database credentials. Production credentials must never be accessible from feature branches.
>
> **Solution**: Create `DB_PASSWORD` as a project variable with two entries: scoped to `staging` environment and `production` environment. Mark both as protected and masked. Set staging as a non-protected environment. Set production as a protected environment available only on `main` branch. Feature branch pipelines get no DB credentials at all.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Variable is empty | Protected var on non-protected branch | Remove protection or protect the branch |
| Variable not masked in logs | Value doesn't meet masking requirements | Ensure ≥8 chars, single line, Base64 chars |
| "Masked variable value is too short" | Value under 8 characters | Pad or restructure the secret |
| Dotenv variables not available | Missing `reports: dotenv:` | Add dotenv report to artifacts |
| Variable override not working | Precedence order | Check hierarchy — job-level > project > group |
| File variable used as string | Expected content but got path | Use `cat $VAR` to read file content |

---

## Summary Table

| Feature | Purpose |
|---|---|
| `.gitlab-ci.yml` variables | Non-sensitive config in code |
| Project/Group variables | Secrets via UI (protected, masked) |
| Predefined variables | Built-in CI context (commit, branch, pipeline) |
| File type | Variable as temp file path (kubeconfig, certs) |
| Protected | Only on protected branches/tags |
| Masked | Hidden in job logs |
| Environment scope | Different values per environment |
| Dotenv artifacts | Pass variables between jobs |

---

## Quick Revision Questions

1. **Where should secrets be stored?** *In GitLab CI/CD Variables (Settings → CI/CD → Variables) — never in `.gitlab-ci.yml`.*
2. **What does "protected" mean for a variable?** *The variable is only available in pipelines running on protected branches or tags — feature branches cannot access it.*
3. **What does "masked" mean for a variable?** *The variable's value is replaced with `[MASKED]` in job logs to prevent accidental exposure.*
4. **How do you pass variables between jobs?** *Use dotenv artifacts: write key=value pairs to a file, declare it as `artifacts: reports: dotenv:`, and downstream jobs receive the variables.*
5. **What is a file type variable?** *GitLab writes the variable's content to a temporary file and sets the variable to the file's path — useful for kubeconfigs, certificates, and service account keys.*
6. **What is the variable precedence order?** *Job-level > `.gitlab-ci.yml` > Project > Group > Instance (highest to lowest).*

---

[← Previous: GitLab Runners](03-gitlab-runners.md) | [Back to README](../README.md) | [Next: Artifacts & Caching →](05-artifacts-caching.md)

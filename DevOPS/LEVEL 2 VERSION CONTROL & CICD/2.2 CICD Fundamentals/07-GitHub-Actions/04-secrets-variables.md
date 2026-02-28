# Chapter 7.4: Secrets & Variables Management

[← Previous: Runners](03-runners.md) | [Back to README](../README.md) | [Next: Matrix Builds →](05-matrix-builds.md)

---

## Overview

GitHub Actions provides **secrets** for sensitive data (tokens, passwords) and **variables** for non-sensitive configuration. Both can be scoped at repository, environment, or organization level.

---

## Secrets vs Variables

```
  SECRETS                           VARIABLES
  ═══════                           ═════════
  • Encrypted at rest               • Stored as plain text
  • Masked in logs                  • Visible in logs
  • Not visible after creation      • Can be viewed/edited
  • Access: ${{ secrets.NAME }}     • Access: ${{ vars.NAME }}
  • Use for: tokens, passwords,     • Use for: URLs, feature
    API keys, certificates            flags, version numbers

  SCOPE HIERARCHY (highest priority wins):
  ┌──────────────────────────┐
  │ Environment level        │  ← Highest priority
  ├──────────────────────────┤
  │ Repository level         │
  ├──────────────────────────┤
  │ Organization level       │  ← Lowest priority
  └──────────────────────────┘
```

---

## Setting Secrets

### Via GitHub UI

```
  Repository → Settings → Secrets and variables → Actions

  ┌─────────────────────────────────────────────┐
  │ Repository secrets                          │
  │                                             │
  │ Name: DOCKER_PASSWORD                       │
  │ Value: ●●●●●●●●●●●● (hidden after save)    │
  │                                             │
  │ [Add secret]                                │
  └─────────────────────────────────────────────┘
```

### Via GitHub CLI

```bash
# Set repository secret
gh secret set DOCKER_PASSWORD --body "my-secret-value"

# Set from file
gh secret set SSH_KEY < ~/.ssh/deploy_key

# Set organization secret
gh secret set API_KEY --org my-org --visibility selected \
  --repos repo-a,repo-b

# Set environment secret
gh secret set DB_PASSWORD --env production

# List secrets
gh secret list
gh secret list --env production
```

---

## Using Secrets in Workflows

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # As environment variable
      - name: Deploy to server
        env:
          SSH_KEY: ${{ secrets.SSH_KEY }}
          API_TOKEN: ${{ secrets.API_TOKEN }}
        run: |
          echo "$SSH_KEY" > deploy_key
          chmod 600 deploy_key
          ssh -i deploy_key user@server "./deploy.sh"

      # As action input
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # GITHUB_TOKEN (automatic)
      - uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: 'Build passed!'
            })
```

---

## GITHUB_TOKEN

```
  AUTOMATIC TOKEN
  ════════════════

  ${{ secrets.GITHUB_TOKEN }}
  ├── Automatically created per workflow run
  ├── Scoped to the repository
  ├── Expires when job completes
  └── Permissions configurable

  DEFAULT PERMISSIONS (restrictive):
  ├── contents: read
  ├── metadata: read
  └── packages: read

  CUSTOMIZE:
  permissions:
    contents: write       # Push commits, create releases
    pull-requests: write  # Comment on PRs
    issues: write         # Create/update issues
    packages: write       # Push container images
    deployments: write    # Create deployments
    actions: read         # Read workflow info
```

```yaml
# Set permissions at workflow or job level
permissions:
  contents: read
  pull-requests: write
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read    # Job-level overrides workflow-level
    steps:
      - uses: actions/checkout@v4
```

---

## Environment Secrets & Protection

```
  ENVIRONMENTS
  ════════════

  Repository → Settings → Environments

  ┌────────────────────────────────────────┐
  │ Environment: production                │
  │                                        │
  │ Protection Rules:                      │
  │ ├── Required reviewers: @ops-team      │
  │ ├── Wait timer: 10 minutes             │
  │ ├── Branch restrictions: main only     │
  │ └── Custom rules: deployment window    │
  │                                        │
  │ Secrets:                               │
  │ ├── DB_PASSWORD: ●●●●●●●●             │
  │ ├── AWS_ACCESS_KEY: ●●●●●●●●          │
  │ └── DEPLOY_TOKEN: ●●●●●●●●            │
  │                                        │
  │ Variables:                             │
  │ ├── API_URL: https://api.example.com   │
  │ └── REPLICAS: 3                        │
  └────────────────────────────────────────┘
```

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging       # Uses staging secrets/vars
    steps:
      - run: |
          echo "Deploying to ${{ vars.API_URL }}"
          ./deploy.sh --token ${{ secrets.DEPLOY_TOKEN }}

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production         # Uses production secrets/vars
      url: https://app.example.com
    steps:
      - run: |
          echo "Deploying to ${{ vars.API_URL }}"
          ./deploy.sh --token ${{ secrets.DEPLOY_TOKEN }}
```

---

## Variables

```yaml
# Using repository and organization variables
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: |
          echo "App: ${{ vars.APP_NAME }}"
          echo "Region: ${{ vars.AWS_REGION }}"
          echo "Replicas: ${{ vars.REPLICAS }}"
```

```bash
# Set variables via CLI
gh variable set APP_NAME --body "my-app"
gh variable set AWS_REGION --body "us-east-1"
gh variable set REPLICAS --body "3" --env production
gh variable list
```

---

## Security Best Practices

```
  SECRET SECURITY CHECKLIST
  ═════════════════════════

  ✓ Never hardcode secrets in workflow files
  ✓ Use environment-scoped secrets for deploy credentials
  ✓ Set minimum GITHUB_TOKEN permissions
  ✓ Rotate secrets regularly
  ✓ Use OIDC for cloud providers (no long-lived keys)
  ✓ Secrets are NOT available in PRs from forks
  ✕ Don't echo secrets (masked, but avoid)
  ✕ Don't write secrets to files without cleanup
  ✕ Don't pass secrets as command-line arguments (visible in process list)
```

### OIDC for Cloud Authentication (No Secrets Needed)

```yaml
# AWS OIDC — no access keys stored
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write    # Required for OIDC
      contents: read
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-actions
          aws-region: us-east-1
          # No access key / secret key needed!

      - run: aws s3 sync ./dist s3://my-bucket
```

---

## Real-World Scenario

> **Scenario**: A team deploys to AWS staging and production with different credentials and approval gates.
>
> **Solution**: Create two GitHub environments: `staging` and `production`. Each has its own AWS role ARN as a variable and deploy token as a secret. Production environment requires approval from `@ops-team` and restricts to `main` branch only. Use OIDC for AWS auth — no long-lived access keys stored. Staging deploys automatically; production waits for manual approval.

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Secret is empty in workflow | Name misspelled or wrong scope | Check exact name in Settings → Secrets |
| Secret visible in logs | Printed via multi-line echo | Avoid echoing secrets; GitHub masks single-line values |
| "Resource not accessible" | Insufficient GITHUB_TOKEN permissions | Add `permissions:` block with required scopes |
| Environment secrets not available | Job missing `environment:` key | Add `environment: env-name` to job config |
| Secrets unavailable in fork PRs | Security restriction | By design — fork PRs cannot access secrets |
| OIDC token error | Missing `id-token: write` permission | Add `permissions: id-token: write` to job |

---

## Summary Table

| Feature | Secrets | Variables |
|---|---|---|
| **Sensitivity** | Encrypted, masked | Plain text, visible |
| **Access** | `${{ secrets.NAME }}` | `${{ vars.NAME }}` |
| **Scopes** | Repo, Org, Environment | Repo, Org, Environment |
| **After creation** | Cannot view value | Can view and edit |
| **In logs** | Auto-masked | Shown as-is |
| **Use for** | Tokens, passwords, keys | URLs, flags, config |

---

## Quick Revision Questions

1. **What is the difference between secrets and variables?** *Secrets are encrypted and masked in logs (for sensitive data). Variables are plain text and visible (for configuration).*
2. **What is GITHUB_TOKEN?** *An automatically generated token scoped to the repository, available in every workflow run — no manual setup needed.*
3. **How do environment-level secrets override repository secrets?** *When a job specifies `environment: production`, environment secrets take precedence over repository secrets of the same name.*
4. **What is OIDC and why use it?** *OpenID Connect allows workflows to authenticate with cloud providers using short-lived tokens instead of storing long-lived access keys as secrets.*
5. **Are secrets available in pull requests from forks?** *No — for security, secrets are not passed to workflows triggered by fork pull requests.*
6. **How do you set minimum permissions for GITHUB_TOKEN?** *Add a `permissions:` block at workflow or job level specifying only the required scopes (e.g., `contents: read`).*

---

[← Previous: Runners](03-runners.md) | [Back to README](../README.md) | [Next: Matrix Builds →](05-matrix-builds.md)

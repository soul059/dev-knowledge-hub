# Chapter 4: CI/CD Pipelines

[← Previous: Version Control](03-version-control.md) | [Back to README](../README.md) | [Next: Security Practices →](05-security-practices.md)

---

## Overview

CI/CD pipelines automate Terraform workflows from code commit to infrastructure deployment. This chapter covers pipeline design, approval gates, plan/apply automation, and multi-environment promotion.

---

## 4.1 Pipeline Architecture

```
┌──────────────────────────────────────────────────────────┐
│           TERRAFORM CI/CD PIPELINE                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  PR Created/Updated                                       │
│       │                                                   │
│       ▼                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │  Format   │→│ Validate │→│  Lint    │→│ Security │ │
│  │  Check    │  │          │  │ (TFLint) │  │  Scan    │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
│       │                                                   │
│       ▼                                                   │
│  ┌──────────┐  ┌──────────┐                               │
│  │   Plan   │→│  Post    │  ← Comment plan on PR          │
│  │          │  │  to PR   │                               │
│  └──────────┘  └──────────┘                               │
│       │                                                   │
│  PR Merged to main                                        │
│       │                                                   │
│       ▼                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│  │  Plan    │→│ Approval │→│  Apply   │               │
│  │  (prod)  │  │  Gate    │  │  (prod)  │               │
│  └──────────┘  └──────────┘  └──────────┘               │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 GitHub Actions Pipeline

```yaml
# ─── .github/workflows/terraform.yml ─────────────
name: Terraform CI/CD
on:
  pull_request:
    branches: [main]
    paths:
      - '**.tf'
      - '**.tfvars'
      - '.github/workflows/terraform.yml'
  push:
    branches: [main]
    paths:
      - '**.tf'
      - '**.tfvars'

permissions:
  contents: read
  pull-requests: write
  id-token: write     # For OIDC auth

env:
  TF_VERSION: "1.7.0"
  WORKING_DIR: "environments/prod"

jobs:
  # ─── VALIDATION (runs on PRs) ──────────────────
  validate:
    name: Validate
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Format Check
        run: terraform fmt -check -recursive
        continue-on-error: false

      - name: Init
        working-directory: ${{ env.WORKING_DIR }}
        run: terraform init -backend=false

      - name: Validate
        working-directory: ${{ env.WORKING_DIR }}
        run: terraform validate

      - name: TFLint
        uses: terraform-linters/setup-tflint@v4
      - run: |
          tflint --init
          tflint --chdir=${{ env.WORKING_DIR }}

  # ─── SECURITY SCAN ─────────────────────────────
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4

      - name: Trivy Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: config
          scan-ref: ${{ env.WORKING_DIR }}
          severity: HIGH,CRITICAL
          exit-code: 1

  # ─── PLAN (runs on PRs) ────────────────────────
  plan:
    name: Plan
    runs-on: ubuntu-latest
    needs: [validate, security]
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS Credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Init
        working-directory: ${{ env.WORKING_DIR }}
        run: terraform init

      - name: Plan
        working-directory: ${{ env.WORKING_DIR }}
        id: plan
        run: terraform plan -no-color -out=tfplan
        continue-on-error: true

      - name: Comment Plan on PR
        uses: actions/github-script@v7
        with:
          script: |
            const plan = `${{ steps.plan.outputs.stdout }}`;
            const truncated = plan.length > 60000
              ? plan.substring(0, 60000) + '\n... (truncated)'
              : plan;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Terraform Plan\n\`\`\`\n${truncated}\n\`\`\``
            });

      - name: Fail if Plan Failed
        if: steps.plan.outcome == 'failure'
        run: exit 1

  # ─── APPLY (runs on merge to main) ────────────
  apply:
    name: Apply
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment: production    # Requires manual approval
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Init
        working-directory: ${{ env.WORKING_DIR }}
        run: terraform init

      - name: Apply
        working-directory: ${{ env.WORKING_DIR }}
        run: terraform apply -auto-approve
```

---

## 4.3 Multi-Environment Pipeline

```
┌──────────────────────────────────────────────────────────┐
│       STAGED DEPLOYMENT PIPELINE                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  PR Merged                                                │
│     │                                                     │
│     ▼                                                     │
│  ┌──────────────────────┐                                 │
│  │ Plan + Apply: DEV    │  ← Automatic                    │
│  └──────────┬───────────┘                                 │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                 │
│  │ Integration Tests    │  ← Automated tests              │
│  └──────────┬───────────┘                                 │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                 │
│  │ Plan: STAGING        │                                 │
│  │ [Manual Approval]    │  ← Team lead approves           │
│  │ Apply: STAGING       │                                 │
│  └──────────┬───────────┘                                 │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                 │
│  │ Smoke Tests          │  ← Verify staging               │
│  └──────────┬───────────┘                                 │
│             │                                             │
│             ▼                                             │
│  ┌──────────────────────┐                                 │
│  │ Plan: PROD           │                                 │
│  │ [Manual Approval]    │  ← Senior engineer approves     │
│  │ Apply: PROD          │                                 │
│  └──────────────────────┘                                 │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```yaml
# ─── Multi-environment job with matrix ────────────
jobs:
  deploy:
    strategy:
      max-parallel: 1
      matrix:
        environment: [dev, staging, prod]
    runs-on: ubuntu-latest
    environment: ${{ matrix.environment }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets[format('AWS_ROLE_{0}', matrix.environment)] }}
          aws-region: us-east-1

      - name: Deploy
        working-directory: environments/${{ matrix.environment }}
        run: |
          terraform init
          terraform plan -out=tfplan
          terraform apply tfplan
```

---

## 4.4 GitLab CI Pipeline

```yaml
# ─── .gitlab-ci.yml ──────────────────────────────
stages:
  - validate
  - plan
  - apply

variables:
  TF_DIR: "environments/prod"

.terraform_base:
  image: hashicorp/terraform:1.7
  before_script:
    - cd $TF_DIR
    - terraform init

validate:
  extends: .terraform_base
  stage: validate
  script:
    - terraform fmt -check -recursive
    - terraform validate
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

plan:
  extends: .terraform_base
  stage: plan
  script:
    - terraform plan -out=plan.tfplan
  artifacts:
    paths:
      - $TF_DIR/plan.tfplan
    expire_in: 1 hour
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'

apply:
  extends: .terraform_base
  stage: apply
  script:
    - terraform apply plan.tfplan
  dependencies:
    - plan
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual    # Manual approval gate
  environment:
    name: production
```

---

## 4.5 Terraform Cloud/Enterprise

```
┌──────────────────────────────────────────────────────────┐
│    TERRAFORM CLOUD PIPELINE (built-in)                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  VCS Integration (GitHub/GitLab/Bitbucket)                │
│       │                                                   │
│       ├── PR opened → Speculative Plan (read-only)        │
│       │                  └── Plan posted as PR comment     │
│       │                                                   │
│       └── PR merged → Run triggered                       │
│              │                                            │
│              ├── Plan phase                                │
│              ├── Sentinel/OPA policy check                 │
│              ├── Cost estimation                           │
│              ├── Manual approval (if configured)           │
│              └── Apply phase                               │
│                                                           │
│  Advantages:                                               │
│  • No CI/CD config needed                                 │
│  • Built-in state management                              │
│  • Policy-as-code (Sentinel/OPA)                          │
│  • Cost estimation                                        │
│  • Audit logging                                          │
│  • Run triggers between workspaces                        │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Terraform Cloud backend configuration ──────
terraform {
  cloud {
    organization = "my-company"

    workspaces {
      name = "webapp-prod"
    }
  }
}
```

---

## 4.6 Pipeline Safety Patterns

```yaml
# ─── Destroy protection ─────────────────────────
- name: Check for Destroys
  run: |
    DESTROYS=$(terraform plan -no-color | grep "destroy" | wc -l)
    if [ "$DESTROYS" -gt 0 ]; then
      echo "::warning::Plan includes resource destruction!"
      echo "NEEDS_APPROVAL=true" >> $GITHUB_ENV
    fi

# ─── Plan file caching ──────────────────────────
- name: Save Plan
  run: terraform plan -out=tfplan

- name: Upload Plan Artifact
  uses: actions/upload-artifact@v4
  with:
    name: terraform-plan
    path: tfplan
    retention-days: 1

# In apply job:
- name: Download Plan
  uses: actions/download-artifact@v4
  with:
    name: terraform-plan

- name: Apply Saved Plan
  run: terraform apply tfplan
  # Ensures apply matches exactly what was reviewed
```

```
┌──────────────────────────────────────────────────────────┐
│           PIPELINE SAFETY CHECKLIST                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ✓ Use saved plan files (plan → artifact → apply)         │
│  ✓ OIDC authentication (no long-lived credentials)        │
│  ✓ Manual approval for production                         │
│  ✓ Alert on destroy operations                            │
│  ✓ Lock state during pipeline runs                        │
│  ✓ Timeout on long-running operations                     │
│  ✓ Notifications on failure (Slack, email)                │
│  ✓ Audit log of all applies                               │
│  ✓ Restrict who can trigger production apply              │
│  ✓ Drift detection scheduled runs                         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Plan passes in CI, fails locally | Different provider versions | Commit `.terraform.lock.hcl`, pin Terraform version |
| CI apply differs from plan | Time between plan and apply | Save plan file as artifact, apply the saved plan |
| Pipeline hangs on apply | Resource creation timeout | Set `-parallelism` flag, add CI timeout |
| Auth fails in CI | Wrong credentials or expired tokens | Use OIDC for short-lived credentials |
| Concurrent pipeline runs conflict | State locking not configured | Use DynamoDB for S3 backend locking |

---

## Summary Table

| CI/CD Platform | Plan on PR | Approval Gate | Apply | Integration |
|---------------|-----------|--------------|-------|-------------|
| **GitHub Actions** | ✓ | Environment protection | On merge | Native |
| **GitLab CI** | ✓ | `when: manual` | Manual trigger | Native |
| **Terraform Cloud** | ✓ (speculative) | Built-in | Automatic/manual | VCS webhook |
| **Azure DevOps** | ✓ | Approval gates | Release pipeline | Extensions |
| **Jenkins** | ✓ | Input step | Pipeline stage | Plugins |

---

## Quick Revision Questions

1. **Why should you save the Terraform plan as an artifact?**
   > To ensure the apply step executes exactly what was reviewed during the plan — preventing changes that occurred between plan and apply.

2. **What authentication method is recommended for CI/CD?**
   > OIDC (OpenID Connect) for short-lived, auto-rotated credentials rather than long-lived access keys stored as secrets.

3. **When should manual approval be required?**
   > For production applies, operations that destroy resources, and any changes that affect critical infrastructure.

4. **How does Terraform Cloud differ from self-managed CI/CD?**
   > Terraform Cloud provides built-in state management, speculative plans on PRs, policy enforcement (Sentinel), cost estimation, and audit logging without needing to configure a CI/CD pipeline.

5. **What triggers a Terraform pipeline run?**
   > Typically: PRs trigger plan/validate, merges to main trigger apply. Path filters ensure only changes to `.tf`/`.tfvars` files trigger the pipeline.

---

[← Previous: Version Control](03-version-control.md) | [Back to README](../README.md) | [Next: Security Practices →](05-security-practices.md)

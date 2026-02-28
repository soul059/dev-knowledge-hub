# Chapter 6.3: PR Workflows

[← Previous: Branch Protection](02-branch-protection.md) | [Back to README](../README.md) | [Next → Testing GitOps](04-testing-gitops.md)

---

## Overview

In GitOps, **Pull Requests are deployment requests**. Every change to your cluster goes through a PR, providing a natural review gate, audit trail, and approval workflow. This chapter covers PR templates, automation, preview environments, and best practices.

---

## PR as a Deployment Gate

```
┌──────────────────────────────────────────────────────────┐
│           PR = DEPLOYMENT REQUEST                        │
│                                                          │
│  Traditional:                                            │
│  Developer → Pipeline → Deploy                           │
│  (who deployed what? check CI logs)                      │
│                                                          │
│  GitOps:                                                 │
│  Developer → PR → Review → Merge → Auto-deploy          │
│                                                          │
│  Benefits:                                               │
│  ├── Who: PR author + approvers visible                  │
│  ├── What: Exact diff in PR                              │
│  ├── When: Merge timestamp                               │
│  ├── Why: PR description and linked issues               │
│  └── Approval: Required reviewers signed off             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## PR Template for GitOps

```markdown
<!-- .github/pull_request_template.md -->
## Change Description
<!-- What is being changed and why? -->

## Environment(s) Affected
- [ ] Dev
- [ ] Staging
- [ ] Production

## Type of Change
- [ ] Application deployment (image update)
- [ ] Configuration change
- [ ] Infrastructure change
- [ ] New application onboarding
- [ ] Rollback

## Change Details
| Field | Value |
|-------|-------|
| Application | |
| Previous version | |
| New version | |
| Kubernetes resources modified | |

## Validation
- [ ] YAML lint passes
- [ ] Kustomize build succeeds
- [ ] Dry-run / diff reviewed
- [ ] Tested in lower environment first
- [ ] Rollback plan documented

## Rollback Plan
<!-- How to revert if something goes wrong -->
```

---

## Automated PR Checks

### CI Pipeline for Config Repo PRs

```yaml
# .github/workflows/pr-validate.yaml
name: PR Validation
on:
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Setup tools
      run: |
        curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
        curl -sL https://github.com/yannh/kubeconform/releases/latest/download/kubeconform-linux-amd64.tar.gz | tar xz

    - name: YAML Lint
      run: yamllint -c .yamllint.yaml .

    - name: Kustomize Build
      run: |
        for env in dev staging production; do
          echo "--- Building $env ---"
          kustomize build overlays/$env > /tmp/$env.yaml
        done

    - name: Schema Validation
      run: |
        for env in dev staging production; do
          echo "--- Validating $env ---"
          kubeconform -strict -summary /tmp/$env.yaml
        done

    - name: Policy Check
      run: |
        for env in dev staging production; do
          conftest test -p policies/ /tmp/$env.yaml
        done

  diff:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: ArgoCD Diff (preview what will change)
      run: |
        argocd app diff my-app \
          --revision ${{ github.event.pull_request.head.sha }} \
          --local overlays/production || true
      # Post diff as PR comment
```

---

## PR Diff Preview

### ArgoCD Diff Comment on PR

```yaml
# Show what will change when PR is merged
- name: Post diff comment
  uses: actions/github-script@v7
  with:
    script: |
      const { execSync } = require('child_process');
      const diff = execSync(
        'kustomize build overlays/production | kubectl diff -f - 2>&1 || true'
      ).toString();
      
      await github.rest.issues.createComment({
        owner: context.repo.owner,
        repo: context.repo.repo,
        issue_number: context.issue.number,
        body: `## Deployment Preview\n\`\`\`diff\n${diff}\n\`\`\``
      });
```

### Preview Environment on PR

```yaml
# Create a temporary preview environment for the PR
- name: Deploy preview
  run: |
    PREVIEW_NS="preview-pr-${{ github.event.pull_request.number }}"
    kubectl create ns $PREVIEW_NS --dry-run=client -o yaml | kubectl apply -f -
    kustomize build overlays/dev | \
      sed "s/namespace: .*/namespace: $PREVIEW_NS/" | \
      kubectl apply -f -

- name: Cleanup preview on PR close
  if: github.event.action == 'closed'
  run: |
    kubectl delete ns "preview-pr-${{ github.event.pull_request.number }}"
```

---

## PR Workflow Patterns

### Pattern 1: Direct PR to Main

```
┌──────────────────────────────────────────────────────────┐
│  Simple: feature branch → PR → main                     │
│                                                          │
│  feature/update-image ──PR──→ main                       │
│                          │                               │
│                     CI validates                         │
│                     2 approvals                          │
│                     merge → deploy                       │
│                                                          │
│  Best for: single-environment or dev environments        │
└──────────────────────────────────────────────────────────┘
```

### Pattern 2: Promotion PRs

```
┌──────────────────────────────────────────────────────────┐
│  Promotion: separate PRs per environment                 │
│                                                          │
│  PR 1: Update overlays/dev/          → auto-merge        │
│  PR 2: Update overlays/staging/      → 1 approval        │
│  PR 3: Update overlays/production/   → 2 approvals       │
│                                                          │
│  Each PR is a promotion gate                             │
│  Different approval requirements per environment         │
└──────────────────────────────────────────────────────────┘
```

### Pattern 3: Automated PR from CI

```
┌──────────────────────────────────────────────────────────┐
│  CI creates PR automatically                             │
│                                                          │
│  1. CI builds new image (app repo)                       │
│  2. CI opens PR in config repo:                          │
│     "Update my-app to v1.4.0"                            │
│  3. Dev overlay: auto-merge                              │
│  4. Staging: bot opens promotion PR                      │
│  5. Production: requires manual approval                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## PR Automation Tools

| Tool | Purpose |
|------|---------|
| Renovate/Dependabot | Auto-PR for dependency updates |
| Flux Image Automation | Auto-commit image tag updates |
| ArgoCD Image Updater | Auto-update image tags in Git |
| GitHub Actions | CI validation on PRs |
| atlantis | Plan/apply for Terraform via PR |
| kubecfg/kubeconform | Schema validation in CI |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| PR = deployment | Every cluster change goes through a PR |
| PR template | Structured description with checklist |
| CI validation | YAML lint, build, schema, policy on every PR |
| Diff preview | Show what will change in the cluster |
| CODEOWNERS | Auto-assigns reviewers per path |
| Promotion PRs | Separate PRs per environment with different approval levels |
| Automated PRs | CI creates PRs for image updates |
| Audit trail | Git history + PR discussions = full audit |

---

## Quick Revision Questions

1. **Why are PRs important in GitOps?**
   > PRs are deployment requests — they provide review, approval, validation, and an audit trail for every change to the cluster.

2. **What should CI check on a GitOps config repo PR?**
   > YAML lint, Kustomize/Helm build, Kubernetes schema validation (kubeconform), and policy checks (conftest/OPA).

3. **How do you preview what a PR will deploy?**
   > Run `kubectl diff` or `argocd app diff` against the PR branch and post the output as a PR comment.

4. **What is a promotion PR?**
   > A PR that updates a specific environment's overlay (e.g., staging → production), with environment-appropriate approval requirements.

5. **How do you automate image update PRs?**
   > Use Renovate, Dependabot, Flux Image Automation, or ArgoCD Image Updater to auto-create PRs when new images are available.

---

[← Previous: Branch Protection](02-branch-protection.md) | [Back to README](../README.md) | [Next → Testing GitOps](04-testing-gitops.md)

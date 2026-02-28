# Chapter 2.3: Change Management

[← Previous: Branch Strategies](02-branch-strategies.md) | [Back to README](../README.md) | [Next → Audit and Compliance](04-audit-and-compliance.md)

---

## Overview

In GitOps, **every change flows through Git**. This makes Git the control plane for all operational changes. Change management in GitOps is about defining processes for proposing, reviewing, approving, and deploying changes — all using familiar Git workflows.

---

## The GitOps Change Lifecycle

```
┌──────────────────────────────────────────────────────────────────┐
│                   CHANGE LIFECYCLE                               │
│                                                                  │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌──────────┐       │
│  │ PROPOSE │──>│ REVIEW  │──>│ APPROVE │──>│  MERGE   │       │
│  │         │   │         │   │         │   │          │       │
│  │ Feature │   │ PR gets │   │ Required│   │ Merged   │       │
│  │ branch  │   │ reviewed│   │ approvals  │ to main  │       │
│  │ + PR    │   │ by team │   │ met     │   │          │       │
│  └─────────┘   └─────────┘   └─────────┘   └────┬─────┘       │
│                                                   │             │
│                                            ┌──────▼──────┐     │
│                                            │   DEPLOY    │     │
│                                            │             │     │
│                                            │ GitOps agent│     │
│                                            │ auto-syncs  │     │
│                                            └──────┬──────┘     │
│                                                   │             │
│                                            ┌──────▼──────┐     │
│                                            │   VERIFY    │     │
│                                            │             │     │
│                                            │ Health check│     │
│                                            │ Monitoring  │     │
│                                            └─────────────┘     │
└──────────────────────────────────────────────────────────────────┘
```

---

## Change Types in GitOps

```
┌──────────────────────────────────────────────────────────────┐
│                    CHANGE TYPES                              │
│                                                              │
│  ┌─ Application Changes ─────────────────────────────────┐   │
│  │  • Image tag updates (new version)                    │   │
│  │  • Replica count changes                              │   │
│  │  • Environment variable updates                       │   │
│  │  • Resource limit adjustments                         │   │
│  │  • ConfigMap / Secret changes                         │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌─ Infrastructure Changes ──────────────────────────────┐   │
│  │  • New namespace creation                             │   │
│  │  • RBAC policy updates                                │   │
│  │  • Network policy changes                             │   │
│  │  • Ingress / certificate configuration                │   │
│  │  • Monitoring/alerting rule updates                   │   │
│  └───────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌─ Emergency Changes ───────────────────────────────────┐   │
│  │  • Hotfixes (expedited PR workflow)                   │   │
│  │  • Rollbacks (git revert)                             │   │
│  │  • Security patches                                   │   │
│  └───────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## PR-Based Change Process

### Standard Change

```bash
# 1. Create a feature branch
git checkout -b chore/update-frontend-v3

# 2. Make the change
cd overlays/production
# Edit kustomization.yaml to update image tag
cat > image-patch.yaml <<EOF
images:
- name: frontend
  newTag: v3.0.1
EOF

# 3. Validate locally
kustomize build overlays/production | kubeval --strict

# 4. Commit with descriptive message
git add .
git commit -m "chore(frontend): update to v3.0.1

- Fixes CVE-2025-9876 in nginx base image
- Changelog: https://github.com/myorg/frontend/releases/tag/v3.0.1
- Tested in staging (PR #89)

Ticket: JIRA-1234"

# 5. Push and create PR
git push origin chore/update-frontend-v3
# Open PR in GitHub/GitLab
```

### PR Template for GitOps Config Changes

```markdown
## Change Description
<!-- What is being changed and why? -->
Update frontend image from v3.0.0 to v3.0.1

## Change Type
- [ ] Application update (image, config)
- [x] Infrastructure change
- [ ] Emergency/hotfix
- [ ] Rollback

## Environments Affected
- [ ] Dev
- [ ] Staging
- [x] Production

## Validation
- [x] Tested in staging environment
- [x] `kustomize build` passes
- [x] `kubeval` schema validation passes
- [ ] Load test completed (if applicable)

## Rollback Plan
`git revert <this-commit>` → agent auto-syncs

## Risk Assessment
- **Risk Level**: Low
- **Blast Radius**: Frontend pods only
- **Expected Downtime**: Zero (rolling update)
```

---

## Approval Gates

```
┌──────────────────────────────────────────────────────────────┐
│                   APPROVAL MATRIX                            │
│                                                              │
│  Change Type          Required Approvals     Auto-Merge?     │
│  ─────────────────    ──────────────────     ───────────     │
│  Dev env change       1 team member          Yes             │
│  Staging change       1 team member          Yes             │
│  Prod app change      2 reviewers            No              │
│  Prod infra change    2 reviewers + lead     No              │
│  RBAC / security      Security team          No              │
│  Emergency hotfix     1 reviewer (fast-track) Yes            │
│                                                              │
│  Implemented via:                                            │
│  • GitHub CODEOWNERS                                         │
│  • Branch protection rules                                   │
│  • Required status checks                                    │
└──────────────────────────────────────────────────────────────┘
```

### CODEOWNERS Example

```
# .github/CODEOWNERS

# Platform team owns infrastructure
/infrastructure/         @platform-team
/clusters/              @platform-team

# Security team must approve RBAC changes
**/rbac/                @security-team
**/network-policies/    @security-team

# Team Alpha owns their apps
/apps/frontend/         @team-alpha
/apps/bff-gateway/      @team-alpha

# Team Beta owns their apps
/apps/payment-service/  @team-beta
/apps/invoice-service/  @team-beta

# Production changes need ops approval
**/overlays/production/ @ops-leads
```

---

## Emergency / Hotfix Process

```
┌──────────────────────────────────────────────────────────────┐
│              EMERGENCY CHANGE PROCESS                        │
│                                                              │
│  Normal Flow (hours):                                        │
│  Branch → PR → 2 Reviews → CI → Merge → Deploy              │
│                                                              │
│  Emergency Flow (minutes):                                   │
│  ┌────────┐  ┌──────────┐  ┌────────┐  ┌────────┐          │
│  │Hotfix  │─>│ 1 Review │─>│ Merge  │─>│Deploy  │          │
│  │Branch  │  │ (fast-   │  │ to main│  │ Agent  │          │
│  └────────┘  │  track)  │  └────────┘  │ syncs  │          │
│              └──────────┘              └────────┘          │
│                                                              │
│  Post-incident:                                              │
│  ├── Create incident ticket                                  │
│  ├── Add retroactive review comments                         │
│  └── Update runbook if needed                                │
│                                                              │
│  NEVER bypass Git entirely!                                  │
│  Even emergencies go through Git (just faster approval)      │
└──────────────────────────────────────────────────────────────┘
```

### Emergency Rollback

```bash
# Fastest rollback — revert the last commit
git revert HEAD --no-edit
git push origin main

# ArgoCD/Flux detects the revert and rolls back automatically
# Time: under 60 seconds

# If you need to rollback multiple commits
git revert HEAD~3..HEAD --no-edit
git push origin main
```

---

## Automated Change Workflows

### Image Update Automation (CI → Config Repo)

```yaml
# CI Pipeline updates the config repo automatically
# when a new image is built

name: Update Config Repo
on:
  workflow_run:
    workflows: ["Build and Push"]
    types: [completed]

jobs:
  update-config:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
    - name: Checkout config repo
      uses: actions/checkout@v4
      with:
        repository: myorg/myapp-config
        token: ${{ secrets.CONFIG_REPO_TOKEN }}

    - name: Update image tag
      run: |
        NEW_TAG="${{ github.event.workflow_run.head_sha }}"
        cd overlays/dev
        kustomize edit set image myapp=registry.example.com/myapp:${NEW_TAG}

    - name: Create PR
      uses: peter-evans/create-pull-request@v5
      with:
        title: "chore: update myapp to ${{ github.event.workflow_run.head_sha }}"
        branch: auto/update-myapp-${{ github.event.workflow_run.head_sha }}
        body: |
          Automated image tag update.
          Source: ${{ github.event.workflow_run.html_url }}
```

---

## Change Tracking and Notifications

```yaml
# ArgoCD Notification (Slack)
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  trigger.on-sync-succeeded: |
    - when: app.status.sync.status == 'Synced'
      send: [app-sync-succeeded]
  template.app-sync-succeeded: |
    message: |
      ✅ *{{.app.metadata.name}}* synced successfully
      Revision: {{.app.status.sync.revision}}
      Author: {{(call .repo.GetCommitMetadata .app.status.sync.revision).Author}}
      Message: {{(call .repo.GetCommitMetadata .app.status.sync.revision).Message}}
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Change lifecycle | Propose → Review → Approve → Merge → Deploy → Verify |
| All changes via Git | No `kubectl edit` — everything through PRs |
| CODEOWNERS | Enforce team ownership and required reviewers |
| Emergency process | Fast-track PR with single reviewer, never bypass Git |
| Rollback | `git revert` — fastest and most auditable rollback |
| Automated updates | CI creates PRs to update image tags in config repo |
| Notifications | ArgoCD/Flux can notify Slack/Teams on sync events |

---

## Quick Revision Questions

1. **What are the stages of the GitOps change lifecycle?**
   > Propose (feature branch + PR) → Review (team reviews diff) → Approve (required approvals met) → Merge (to main) → Deploy (agent syncs) → Verify (health checks).

2. **Why should emergency changes still go through Git?**
   > To maintain the audit trail, enable rollback via `git revert`, and ensure the cluster state matches Git. Use a fast-track review process instead of bypassing Git.

3. **What is the purpose of CODEOWNERS in a GitOps config repo?**
   > It enforces that specific teams must review and approve changes to files they own, ensuring proper authorization for sensitive changes.

4. **How can CI pipelines automate image tag updates in a GitOps workflow?**
   > After building and pushing an image, CI checks out the config repo, updates the image tag using `kustomize edit set image`, and creates a PR automatically.

5. **What information should a PR template for GitOps config changes include?**
   > Change description, type, affected environments, validation steps, rollback plan, and risk assessment.

---

[← Previous: Branch Strategies](02-branch-strategies.md) | [Back to README](../README.md) | [Next → Audit and Compliance](04-audit-and-compliance.md)

# Chapter 2.4: Audit and Compliance

[← Previous: Change Management](03-change-management.md) | [Back to README](../README.md) | [Next → Version Control Best Practices](05-version-control-best-practices.md)

---

## Overview

One of the strongest arguments for GitOps is **built-in auditability**. Every change is a Git commit — timestamped, attributed, immutable, and reviewable. This makes GitOps a natural fit for organizations with compliance requirements (SOC 2, PCI-DSS, HIPAA, ISO 27001).

---

## GitOps as an Audit System

```
┌──────────────────────────────────────────────────────────────────┐
│                  GITOPS AUDIT ARCHITECTURE                      │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    GIT REPOSITORY                        │    │
│  │                                                          │    │
│  │  Every commit records:                                   │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │    │
│  │  │   WHO    │ │   WHAT   │ │   WHEN   │ │   WHY    │   │    │
│  │  │ author   │ │ diff     │ │ timestamp│ │ commit   │   │    │
│  │  │ email    │ │ files    │ │ date     │ │ message  │   │    │
│  │  │ GPG sig  │ │ changed  │ │          │ │ PR link  │   │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │    │
│  │                                                          │    │
│  │  PR metadata adds:                                       │    │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │    │
│  │  │ APPROVED BY  │ │ CI CHECKS    │ │ DISCUSSION   │     │    │
│  │  │ reviewers    │ │ passed/failed│ │ comments     │     │    │
│  │  └──────────────┘ └──────────────┘ └──────────────┘     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  + ArgoCD/Flux adds:                                             │
│  ┌──────────────────────┐ ┌──────────────────────┐              │
│  │ SYNC EVENTS          │ │ HEALTH STATUS         │             │
│  │ when changes applied │ │ deployment outcome    │              │
│  └──────────────────────┘ └──────────────────────┘              │
└──────────────────────────────────────────────────────────────────┘
```

---

## Compliance Mapping

```
┌──────────────────────────────────────────────────────────────────┐
│           COMPLIANCE REQUIREMENTS vs GITOPS                     │
│                                                                  │
│  Requirement              How GitOps Satisfies It                │
│  ─────────────────────    ────────────────────────────────────   │
│                                                                  │
│  Change authorization     PRs require approved reviews           │
│  (SOC 2: CC8.1)          CODEOWNERS enforce required reviewers   │
│                                                                  │
│  Separation of duties    Author ≠ Approver (enforced via Git)    │
│  (PCI-DSS: 6.4.2)       Branch protection prevents self-merge   │
│                                                                  │
│  Change audit trail      Git log = immutable, complete history   │
│  (ISO 27001: A.12.1.2)  SHA hash = cryptographic integrity      │
│                                                                  │
│  Access control          Git RBAC + K8s RBAC                     │
│  (HIPAA: §164.312)      Developers never touch cluster directly │
│                                                                  │
│  Config integrity        Git detects tampering (hash-based)      │
│  (NIST: CM-3)           Agent reconciles drift = enforces state  │
│                                                                  │
│  Rollback capability     git revert provides instant rollback    │
│  (SOC 2: CC7.4)         Full history available for any point     │
└──────────────────────────────────────────────────────────────────┘
```

---

## Commit Signing

### Why Sign Commits?

Commit signing proves that a commit was made by the claimed author and hasn't been tampered with.

```
┌──────────────────────────────────────────────────────────────┐
│                   SIGNED vs UNSIGNED COMMITS                 │
│                                                              │
│  Unsigned commit:                                            │
│  ┌─────────────────────────────────────────────┐             │
│  │  commit abc1234                             │             │
│  │  Author: alice@company.com  ← anyone can    │             │
│  │  Date: 2026-02-28           set this!        │             │
│  │  Message: Update deployment                 │             │
│  │                                             │             │
│  │  Trust level: LOW (author not verified)     │             │
│  └─────────────────────────────────────────────┘             │
│                                                              │
│  Signed commit (GPG/SSH):                                    │
│  ┌─────────────────────────────────────────────┐             │
│  │  commit def5678  (GPG: VERIFIED ✓)          │             │
│  │  Author: alice@company.com                  │             │
│  │  Signature: 4096-bit RSA key fingerprint    │             │
│  │  Date: 2026-02-28                           │             │
│  │  Message: Update deployment                 │             │
│  │                                             │             │
│  │  Trust level: HIGH (cryptographically       │             │
│  │  verified author identity)                  │             │
│  └─────────────────────────────────────────────┘             │
└──────────────────────────────────────────────────────────────┘
```

### Setting Up GPG Commit Signing

```bash
# Generate a GPG key
gpg --full-generate-key
# Choose: RSA 4096 bit, your company email

# List your keys
gpg --list-secret-keys --keyid-format=long
# Output: sec   rsa4096/ABC123DEF456 2026-02-28

# Configure Git to use the key
git config --global user.signingkey ABC123DEF456
git config --global commit.gpgsign true

# Now all commits are automatically signed
git commit -m "chore: update frontend"
# Creates a GPG-signed commit

# Verify a signed commit
git log --show-signature -1
```

### Enforce Signed Commits (GitHub)

```
Branch Protection Rules:
├── ✓ Require signed commits
├── ✓ Require pull request reviews
├── ✓ Require status checks to pass
└── ✓ Include administrators
```

---

## Audit Queries

### Common Audit Questions Answered by Git

```bash
# Q: Who made changes to production in the last 30 days?
git log --since="30 days ago" --format="%h %an <%ae> %s" \
  -- overlays/production/

# Q: What changed in the payment-service config?
git log --all -p -- apps/payment-service/

# Q: When was a specific image version deployed?
git log --all -S "payments:v4.0" --format="%h %ad %an %s"

# Q: Show all PRs that modified RBAC policies
git log --grep="rbac" --format="%h %ad %an %s"

# Q: Compare current production to last month's state
git diff HEAD "HEAD@{30 days ago}" -- overlays/production/

# Q: Who approved the change that introduced the bug?
git log --format="%h %an %s %b" -1 abc1234
# Body includes PR number → check PR for approvers

# Q: Export audit log as CSV
git log --since="2026-01-01" --format='"%h","%an","%ae","%ad","%s"' \
  -- overlays/production/ > audit-log.csv
```

---

## ArgoCD Audit Events

```yaml
# ArgoCD stores audit events that complement Git history

# View sync history
$ argocd app history myapp

# Output:
# ID  DATE                  REVISION
# 1   2026-02-25 10:00:00  abc1234 (feat: initial deploy)
# 2   2026-02-26 14:30:00  def5678 (fix: increase memory)
# 3   2026-02-28 09:15:00  ghi9012 (chore: update to v3)

# ArgoCD also logs:
# - Who triggered a manual sync
# - Sync success/failure with details
# - Resource health transitions
# - RBAC access events
```

### Exporting ArgoCD Events to External Systems

```yaml
# ArgoCD → Audit Log Integration
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  # Send audit events to webhook (SIEM integration)
  service.webhook.audit: |
    url: https://siem.company.com/api/events
    headers:
    - name: Authorization
      value: Bearer $audit-token
  template.audit-event: |
    webhook:
      audit:
        method: POST
        body: |
          {
            "app": "{{.app.metadata.name}}",
            "sync_status": "{{.app.status.sync.status}}",
            "revision": "{{.app.status.sync.revision}}",
            "timestamp": "{{.app.status.operationState.finishedAt}}"
          }
  trigger.on-sync-status-unknown: |
    - send: [audit-event]
```

---

## Policy Enforcement

### OPA/Gatekeeper for Cluster-Side Policy

```yaml
# Prevent deployments without resource limits
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredResources
metadata:
  name: require-resource-limits
spec:
  match:
    kinds:
    - apiGroups: ["apps"]
      kinds: ["Deployment"]
  parameters:
    requiredResources:
    - limits
    - requests
```

### Kyverno Policy (Alternative)

```yaml
# Require specific labels on all resources
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: Enforce
  rules:
  - name: require-team-label
    match:
      resources:
        kinds:
        - Deployment
        - Service
    validate:
      message: "Label 'team' is required"
      pattern:
        metadata:
          labels:
            team: "?*"
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Git as audit log | Every commit = WHO, WHAT, WHEN, WHY |
| Commit signing | GPG/SSH signatures verify author identity |
| PR metadata | Adds approvers, CI results, discussion context |
| Compliance | GitOps satisfies SOC 2, PCI-DSS, HIPAA, ISO 27001 requirements |
| Audit queries | `git log` with filters answers any audit question |
| ArgoCD events | Sync history, health transitions, access logs |
| Policy enforcement | OPA/Gatekeeper/Kyverno enforce rules at cluster level |

---

## Quick Revision Questions

1. **What four pieces of information does every Git commit provide for auditing?**
   > WHO (author/email), WHAT (diff of changed files), WHEN (timestamp), and WHY (commit message/PR link).

2. **How does GitOps satisfy the SOC 2 separation of duties requirement?**
   > Branch protection rules ensure the PR author cannot be the same person who approves the merge. CODEOWNERS enforce required reviewers.

3. **Why is commit signing important for compliance?**
   > It cryptographically verifies the author's identity, preventing someone from impersonating another user in commit metadata.

4. **How can you export an audit trail from Git for a compliance review?**
   > Use `git log` with format strings and date filters to export changes as CSV or JSON for the relevant time period and paths.

5. **What role do OPA/Gatekeeper/Kyverno play in GitOps compliance?**
   > They enforce policies at the cluster level (e.g., requiring labels, resource limits), acting as a last line of defense even if a bad config passes PR review.

6. **How does ArgoCD complement Git's audit trail?**
   > ArgoCD adds sync events (when changes were applied), health status transitions, manual sync triggers, and RBAC access logs.

---

[← Previous: Change Management](03-change-management.md) | [Back to README](../README.md) | [Next → Version Control Best Practices](05-version-control-best-practices.md)

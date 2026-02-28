# Chapter 7: Drift Detection

[← Previous: State Security](06-state-security.md) | [Back to README](../README.md) | [Next: Unit 5 - Module Basics →](../05-Modules/01-module-basics.md)

---

## Overview

**Configuration drift** occurs when the actual state of infrastructure diverges from what Terraform expects. This happens when changes are made outside of Terraform — via the console, CLI, other tools, or automated processes.

---

## 7.1 What is Drift?

```
┌───────────────────────────────────────────────────────────┐
│                    CONFIGURATION DRIFT                     │
├───────────────────────────────────────────────────────────┤
│                                                            │
│  Terraform Config        State File          Real World    │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐ │
│  │ type=t3.micro│    │ type=t3.micro│    │ type=t3.LARGE│ │
│  │ port=80      │    │ port=80      │    │ port=80      │ │
│  │ sg=sg-abc    │    │ sg=sg-abc    │    │ sg=sg-abc    │ │
│  │              │    │              │    │ tag=MANUAL ← │ │
│  └──────────────┘    └──────────────┘    └──────────────┘ │
│       ↕ Match             ↕ Mismatch!                     │
│                                                            │
│  Drift: Real infrastructure differs from expected state    │
│  Cause: Someone changed instance type via AWS Console      │
│  Cause: Someone added a tag outside Terraform              │
│                                                            │
└───────────────────────────────────────────────────────────┘
```

---

## 7.2 How Terraform Detects Drift

```
┌──────────────────────────────────────────────────────────┐
│              DRIFT DETECTION FLOW                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  terraform plan / terraform apply                         │
│      │                                                    │
│      ├──1──► Read state file                              │
│      │                                                    │
│      ├──2──► REFRESH: Query real infrastructure           │
│      │       (API calls to cloud provider)                │
│      │                                                    │
│      ├──3──► Compare: Real World vs State                 │
│      │       └── If different → Update state             │
│      │                                                    │
│      ├──4──► Compare: Config (.tf) vs Updated State       │
│      │       └── If different → Plan changes             │
│      │                                                    │
│      └──5──► Show plan (includes drift corrections)       │
│                                                           │
│  The REFRESH step (2-3) is where drift is detected        │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

### Refresh Behavior

```bash
# Normal plan — includes refresh (default)
terraform plan
# Refreshing state... (queries real infrastructure)

# Explicit refresh only (update state, no plan)
terraform apply -refresh-only

# Skip refresh (dangerous — uses stale state)
terraform plan -refresh=false
```

---

## 7.3 Types of Drift

```
┌────────────────────────────────────────────────────────────┐
│                    TYPES OF DRIFT                           │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Attribute Drift                                         │
│     └── Resource attribute changed outside TF               │
│         Example: Instance type changed via console          │
│         Plan: ~ update in-place                             │
│                                                             │
│  2. Resource Deletion                                       │
│     └── Resource deleted outside TF but still in state      │
│         Example: EC2 instance terminated via console        │
│         Plan: + create (recreate the resource)              │
│                                                             │
│  3. Resource Addition                                       │
│     └── Resource created outside TF (not in state)          │
│         Terraform CANNOT detect this automatically          │
│         Must import or reconcile manually                   │
│                                                             │
│  4. Tag/Metadata Drift                                      │
│     └── Tags modified outside TF                            │
│         Plan: ~ update tags                                 │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 7.4 Handling Drift

### Option 1: Let Terraform Fix It

```bash
# Plan will show changes to bring real world back to config
$ terraform plan

  # aws_instance.web will be updated in-place
  ~ resource "aws_instance" "web" {
      ~ instance_type = "t3.large" -> "t3.micro"   # Drift!
        # (1 unchanged attribute hidden)
    }

$ terraform apply
# Reverts the manual change
```

### Option 2: Accept the Drift

```hcl
# Update your config to match reality
resource "aws_instance" "web" {
  instance_type = "t3.large"   # Accept the new value
}
```

```bash
$ terraform plan
# No changes.
```

### Option 3: Refresh Only

```bash
# Update state to match reality WITHOUT changing anything
$ terraform apply -refresh-only

# This updates the state file to reflect what's actually deployed
# Your config still says t3.micro, but state now says t3.large
# Next plan will show: t3.large → t3.micro
```

### Option 4: Ignore Changes

```hcl
# Tell Terraform to ignore specific attributes
resource "aws_instance" "web" {
  instance_type = "t3.micro"

  lifecycle {
    ignore_changes = [
      instance_type,   # Don't track instance_type changes
      tags["Modified"], # Don't track this specific tag
    ]
  }
}

# ignore_changes = all  ← Ignore ALL attribute changes
```

---

## 7.5 Continuous Drift Detection

### Automated Drift Detection with CI/CD

```yaml
# .github/workflows/drift-detection.yml
name: Terraform Drift Detection
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours

jobs:
  detect-drift:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: hashicorp/setup-terraform@v3

      - name: Terraform Init
        run: terraform init

      - name: Detect Drift
        id: plan
        run: terraform plan -detailed-exitcode -out=plan.tfplan
        continue-on-error: true
        # Exit codes:
        # 0 = No changes (no drift)
        # 1 = Error
        # 2 = Changes detected (drift!)

      - name: Alert on Drift
        if: steps.plan.outcome == 'failure'
        run: |
          echo "⚠️ Infrastructure drift detected!"
          # Send Slack notification, create issue, etc.
```

```
┌────────────────────────────────────────────────────────────┐
│           PLAN EXIT CODES                                   │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  terraform plan -detailed-exitcode                          │
│                                                             │
│  Exit 0 → No changes       (infrastructure matches config) │
│  Exit 1 → Error            (config error, auth failure)     │
│  Exit 2 → Changes detected (drift or pending changes)       │
│                                                             │
│  Use in CI/CD to automate drift detection                   │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 7.6 Preventing Drift

```
┌──────────────────────────────────────────────────────────┐
│              DRIFT PREVENTION STRATEGIES                  │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. Process Controls                                      │
│     ├── All changes through Terraform (never manual)      │
│     ├── CI/CD pipeline for all applies                    │
│     └── Code review for infrastructure changes            │
│                                                           │
│  2. Access Controls                                       │
│     ├── Limit console/CLI write access                    │
│     ├── Read-only roles for most team members             │
│     └── Break-glass procedures for emergencies            │
│                                                           │
│  3. Detection Controls                                    │
│     ├── Scheduled drift detection (plan -detailed-exitcode)│
│     ├── Alerting on detected drift                        │
│     └── AWS Config rules / Azure Policy / GCP Constraints │
│                                                           │
│  4. Technical Controls                                    │
│     ├── SCP/Service Control Policies to restrict console  │
│     ├── Tag enforcement (resources must have "ManagedBy") │
│     └── Immutable infrastructure patterns                 │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Plan shows unexpected changes | Drift from manual modifications | Decide: revert (apply) or accept (update config) |
| Resources "recreating" unexpectedly | Resource deleted outside Terraform | Investigate why; consider `lifecycle` rules |
| Constant drift on certain attributes | Auto-generated values (timestamps, etc.) | Use `ignore_changes` in lifecycle |
| Refresh is slow | Too many resources | Split into smaller state files; use `-target` |
| Drift not detected | Attribute not tracked by provider | Check provider docs; some attributes are ignore-by-default |
| False drift | Provider returns normalized values | Update config to match normalized format |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Drift** | Real infrastructure differs from expected state |
| **Refresh** | Query real infrastructure to update state |
| **-refresh-only** | Update state without changing infrastructure |
| **ignore_changes** | Skip tracking specific attributes |
| **-detailed-exitcode** | Exit code 2 = changes detected |
| **Prevention** | All changes via Terraform; restrict manual access |
| **Detection** | Scheduled `terraform plan` in CI/CD |

---

## Quick Revision Questions

1. **What is configuration drift?**
   > When the actual state of infrastructure diverges from what Terraform's config and state expect, typically due to manual changes.

2. **How does Terraform detect drift?**
   > During the refresh phase of `plan`/`apply`, Terraform queries the actual infrastructure via provider APIs and compares it to the stored state.

3. **What are the options for handling detected drift?**
   > 1) Apply to revert to config, 2) Update config to accept changes, 3) `apply -refresh-only` to update state, 4) `ignore_changes` to skip specific attributes.

4. **How do you automate drift detection?**
   > Schedule `terraform plan -detailed-exitcode` in CI/CD. Exit code 2 indicates drift/changes, which can trigger alerts.

5. **What does `ignore_changes` in a lifecycle block do?**
   > Tells Terraform to NOT track changes to specific attributes, preventing those differences from showing up in plans.

---

[← Previous: State Security](06-state-security.md) | [Back to README](../README.md) | [Next: Unit 5 - Module Basics →](../05-Modules/01-module-basics.md)

# Chapter 9.6: Secrets Management Best Practices

## Overview

Effective secrets management requires a combination of technology, processes, and culture. This chapter consolidates best practices across secret generation, storage, delivery, rotation, detection, and incident response — providing a comprehensive checklist for securing secrets throughout the DevSecOps lifecycle.

---

## Secrets Management Maturity Model

```
MATURITY LEVELS:

  Level 1: AD HOC (Most teams start here)
  ┌────────────────────────────────────────────┐
  │ • Secrets hardcoded or in config files      │
  │ • Shared credentials across team            │
  │ • No rotation policy                        │
  │ • No audit trail                            │
  └────────────────────────────────────────────┘
           │
           ▼
  Level 2: BASIC
  ┌────────────────────────────────────────────┐
  │ • Secrets in CI/CD variables (encrypted)    │
  │ • Environment-specific credentials          │
  │ • Pre-commit hooks for detection            │
  │ • Manual rotation on schedule               │
  └────────────────────────────────────────────┘
           │
           ▼
  Level 3: MANAGED
  ┌────────────────────────────────────────────┐
  │ • Centralized secrets manager (Vault/SM)   │
  │ • RBAC per secret                           │
  │ • Automated rotation (30-90 day)            │
  │ • Audit logging enabled                     │
  │ • Secret scanning in CI/CD                  │
  └────────────────────────────────────────────┘
           │
           ▼
  Level 4: ADVANCED
  ┌────────────────────────────────────────────┐
  │ • Dynamic secrets (per-request creds)       │
  │ • Zero standing privileges                  │
  │ • OIDC/Workload Identity everywhere         │
  │ • Real-time anomaly detection               │
  │ • Automated incident response               │
  └────────────────────────────────────────────┘
           │
           ▼
  Level 5: OPTIMIZED
  ┌────────────────────────────────────────────┐
  │ • No long-lived secrets anywhere            │
  │ • Full zero-trust identity                  │
  │ • Continuous compliance validation          │
  │ • ML-based access pattern monitoring        │
  │ • Self-healing rotation on anomaly          │
  └────────────────────────────────────────────┘
```

---

## Best Practices Checklist

### 1. Secret Generation

```
✅ GENERATION BEST PRACTICES:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  • Use cryptographically secure random generators  │
  │    (secrets module in Python, crypto/rand in Go)   │
  │                                                    │
  │  • Minimum password length: 32 characters          │
  │                                                    │
  │  • Include uppercase, lowercase, digits, symbols   │
  │                                                    │
  │  • Never generate secrets deterministically        │
  │    (no MD5/SHA of predictable values)              │
  │                                                    │
  │  • Use unique secrets per service/environment      │
  │                                                    │
  │  • Prefer dynamic secrets over static ones         │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

```python
# Correct: cryptographically secure
import secrets
import string

def generate_secure_password(length=32):
    alphabet = string.ascii_letters + string.digits + "!@#$%^&*"
    return ''.join(secrets.choice(alphabet) for _ in range(length))

# WRONG: predictable
import random
password = ''.join(random.choice('abcdef') for _ in range(8))  # ❌
```

### 2. Secret Storage

```
✅ STORAGE BEST PRACTICES:

  DO:                              DON'T:
  ┌───────────────────────┐       ┌───────────────────────┐
  │ Use Vault / Cloud SM  │       │ Hardcode in source    │
  │ Encrypt at rest       │       │ Store in .env files   │
  │ Enable versioning     │       │ Commit to Git         │
  │ Enable soft-delete    │       │ Store in plain text   │
  │ Enable purge protect  │       │ Use shared passwords  │
  │ Use namespaced paths  │       │ Store in wikis/docs   │
  │ Tag with metadata     │       │ Email credentials     │
  └───────────────────────┘       └───────────────────────┘
```

### 3. Secret Delivery

```
✅ DELIVERY BEST PRACTICES:

  PREFERRED (most to least secure):

  1. OIDC / Workload Identity Federation
     No secrets at all — identity-based auth
     ┌────────────────────────────────────────┐
     │ CI/CD ──OIDC──▶ Cloud Provider         │
     │ K8s   ──SA────▶ Vault                  │
     │ No stored credentials anywhere         │
     └────────────────────────────────────────┘

  2. Sidecar Agent / External Secrets Operator
     Transparent to application
     ┌────────────────────────────────────────┐
     │ Vault Agent writes to /vault/secrets/  │
     │ ESO syncs to K8s Secrets               │
     │ Auto-refresh on schedule               │
     └────────────────────────────────────────┘

  3. Direct API with short-lived tokens
     App fetches secrets via SDK
     ┌────────────────────────────────────────┐
     │ App ──API──▶ Vault/SM                  │
     │ Cache with TTL, refresh periodically   │
     └────────────────────────────────────────┘

  4. Platform environment injection
     Managed service integration
     ┌────────────────────────────────────────┐
     │ ECS/Lambda ── ARN ref ──▶ SM           │
     │ Platform handles injection             │
     └────────────────────────────────────────┘

  AVOID:
  ❌ Baking secrets into Docker images
  ❌ Passing secrets as CLI arguments (visible in ps)
  ❌ Using unencrypted K8s ConfigMaps for secrets
```

### 4. Secret Rotation

```
✅ ROTATION BEST PRACTICES:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  • Automate ALL rotation (no manual processes)    │
  │                                                    │
  │  • Use dual-user rotation for zero downtime       │
  │                                                    │
  │  • Prefer dynamic secrets (no rotation needed)    │
  │                                                    │
  │  • DB passwords: rotate every 30-90 days          │
  │                                                    │
  │  • API keys: rotate every 90 days                 │
  │                                                    │
  │  • TLS certs: 30-90 day auto-renewal              │
  │                                                    │
  │  • Cloud access keys: DON'T USE — use IAM roles   │
  │                                                    │
  │  • Test rotation in staging before production     │
  │                                                    │
  │  • Monitor rotation failures with alerts          │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

### 5. Secret Detection

```yaml
# .pre-commit-config.yaml — Multi-tool secret detection
repos:
  # Detect-secrets (baseline approach)
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  # Gitleaks (pattern matching)
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

# CI/CD pipeline scanning
# .github/workflows/secret-scan.yml
name: Secret Scan
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for git scanning

      - name: TruffleHog Scan
        uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --only-verified

      - name: Gitleaks Scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 6. Access Control

```hcl
# Vault Policy — Least privilege example
# Application can only read its own secrets
path "secret/data/myapp/*" {
  capabilities = ["read"]
}

# CI/CD can read and update deployment secrets
path "secret/data/myapp/deploy/*" {
  capabilities = ["read", "update"]
}

# Only admins can manage secret metadata
path "secret/metadata/*" {
  capabilities = ["list"]
}

# Deny access to other apps' secrets (implicit)
# All paths not listed = denied
```

---

## Incident Response for Compromised Secrets

```
SECRET BREACH RESPONSE PLAYBOOK:

  ┌──────────────────────────────────────────────────┐
  │ STEP 1: DETECT (minutes)                          │
  │ • Alert from secret scanning tool                 │
  │ • Anomalous access pattern in audit log           │
  │ • Report from security team                       │
  └──────────────────┬───────────────────────────────┘
                     │
                     ▼
  ┌──────────────────────────────────────────────────┐
  │ STEP 2: CONTAIN (< 1 hour)                        │
  │ • Immediately rotate/revoke compromised secret    │
  │ • Disable compromised access keys                 │
  │ • Block suspicious IP addresses                   │
  │ • Notify affected team members                    │
  └──────────────────┬───────────────────────────────┘
                     │
                     ▼
  ┌──────────────────────────────────────────────────┐
  │ STEP 3: ASSESS (hours)                            │
  │ • Review audit logs for unauthorized access       │
  │ • Determine scope of exposure                     │
  │ • Check for lateral movement                      │
  │ • Identify how secret was compromised             │
  └──────────────────┬───────────────────────────────┘
                     │
                     ▼
  ┌──────────────────────────────────────────────────┐
  │ STEP 4: REMEDIATE (days)                          │
  │ • Rotate ALL secrets that may be affected         │
  │ • Fix the root cause (remove from Git, etc.)      │
  │ • Update detection rules                          │
  │ • Clean Git history (BFG Repo-Cleaner)            │
  └──────────────────┬───────────────────────────────┘
                     │
                     ▼
  ┌──────────────────────────────────────────────────┐
  │ STEP 5: LEARN (weeks)                             │
  │ • Conduct blameless post-mortem                   │
  │ • Update policies and procedures                  │
  │ • Improve detection capabilities                  │
  │ • Training for development team                   │
  └──────────────────────────────────────────────────┘
```

```bash
# Emergency: Remove secret from Git history
# Install BFG Repo-Cleaner
java -jar bfg.jar --replace-text secrets-to-remove.txt repo.git

# Or use git-filter-repo
git filter-repo --invert-paths --path config-with-secrets.yaml

# Then force push (coordinate with team!)
git push --force

# IMPORTANT: Rotating the secret is still required!
# Removing from Git history doesn't invalidate the secret.
```

---

## Monitoring and Alerting

```yaml
# Vault audit log monitoring rules
# Alert on suspicious patterns:

rules:
  # Alert: Root token usage
  - name: vault_root_token_used
    condition: auth.token_type == "root"
    severity: critical
    action: page_on_call

  # Alert: High volume secret reads
  - name: vault_bulk_secret_access
    condition: count(operation == "read") > 100 per 5min
    severity: high
    action: create_ticket

  # Alert: Secret access from new IP
  - name: vault_new_ip_access
    condition: remote_address NOT IN known_ips
    severity: medium
    action: notify_slack

  # Alert: Failed authentication spike
  - name: vault_auth_failures
    condition: count(auth.result == "denied") > 10 per 1min
    severity: high
    action: page_on_call

  # Alert: Secret engine mount/unmount
  - name: vault_engine_changes
    condition: operation IN ["mount", "unmount"]
    severity: medium
    action: notify_security_team
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Secret found in Git history | Previously committed even if deleted from HEAD | Use BFG Repo-Cleaner to scrub history; rotate the secret immediately |
| Teams resist using secrets manager | Too complex, slows development | Provide wrapper scripts/SDKs; show local dev workflow with `.env.local`; automate onboarding |
| Rotation breaks applications | App doesn't handle credential refresh | Implement retry with credential refresh; use connection pool reconnect; test rotation in staging |
| Too many secrets to migrate | Large legacy codebase with embedded secrets | Prioritize by risk (production first); migrate incrementally; use detection tools to inventory |
| Audit log too noisy | Every secret read generates log entry | Filter expected access patterns; set up dashboards for anomalies; use sampling for high-volume reads |

---

## Summary Table

| Best Practice | Priority | Implementation |
|---------------|----------|---------------|
| **No hardcoded secrets** | Critical | Pre-commit hooks + CI scanning |
| **Centralized storage** | Critical | Vault / Cloud Secrets Manager |
| **Automated rotation** | High | Lambda/Cloud Functions + vault dynamic |
| **OIDC/Workload Identity** | High | Zero stored credentials in CI/CD |
| **Audit logging** | High | Vault audit + SIEM integration |
| **Least privilege access** | High | Per-app policies, no wildcard access |
| **Secret detection** | High | TruffleHog + gitleaks in pipeline |
| **Dynamic secrets** | Medium | Vault database/cloud engines |
| **Incident playbook** | Medium | Documented response + tabletop exercises |
| **Developer training** | Medium | Regular security awareness sessions |

---

## Quick Revision Questions

1. **What are the five levels of secrets management maturity?**
   > Ad Hoc (hardcoded, shared creds), Basic (CI/CD variables, manual rotation), Managed (centralized vault, automated rotation, audit logging), Advanced (dynamic secrets, zero standing privileges, OIDC everywhere), Optimized (no long-lived secrets, zero-trust identity, ML-based monitoring, self-healing rotation).

2. **What should happen when a secret is found in a Git repository?**
   > Immediately rotate/revoke the compromised secret (top priority). Remove the secret from Git history using BFG Repo-Cleaner or git-filter-repo. Review audit logs for unauthorized usage. Investigate how it was committed (missing pre-commit hooks?). Update detection rules and conduct a post-mortem.

3. **What is the preferred order for secrets delivery methods?**
   > OIDC/Workload Identity (no secrets at all), sidecar agent or External Secrets Operator (transparent auto-refresh), direct API calls with short-lived tokens (app-managed), platform environment injection (managed services). Avoid baking secrets into images, passing as CLI args, or using unencrypted ConfigMaps.

4. **How should Vault audit logs be monitored?**
   > Alert on root token usage (critical), bulk secret access (>100 reads/5min), access from unknown IP addresses, authentication failure spikes, and secret engine changes (mount/unmount). Ship audit logs to SIEM for correlation. Build dashboards tracking access patterns and anomalies.

5. **What should an incident response plan for compromised secrets include?**
   > Detect (alerting from scanning/audit logs), Contain (immediately rotate/revoke, block IPs, notify team), Assess (review audit logs, determine scope, check lateral movement), Remediate (rotate all affected secrets, fix root cause, clean Git history), Learn (blameless post-mortem, update procedures, improve detection, train team).

---

[← Previous: 9.5 Integration Patterns](05-integration-patterns.md) | [Next: 10.1 Security Monitoring →](../10-Security-Operations/01-security-monitoring.md)

[Back to Table of Contents](../README.md)

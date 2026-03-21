# Unit 5: Cloud IAM Security — Topic 4: API Key Management

## Overview

**API keys** are simple authentication tokens used to identify and authorize API requests. While easier to use than full IAM authentication, API keys present significant security risks if not properly managed. They are commonly found leaked in source code, logs, and public repositories. Proper API key management is essential for preventing unauthorized access to cloud services.

---

## 1. API Key Fundamentals

```
API KEY TYPES:

CLOUD PROVIDER API KEYS:
  AWS:
  → Access Key ID (AKIA...) + Secret Access Key
  → Used for CLI, SDK, programmatic access
  → Long-lived by default
  → Associated with IAM users
  
  Azure:
  → Storage account keys
  → Cognitive Services keys
  → Event Hub connection strings
  → Function keys
  → SAS tokens (shared access signatures)
  
  GCP:
  → API keys (restricted to specific APIs)
  → Service account keys (JSON)
  → OAuth client secrets
  
APPLICATION API KEYS:
  → Third-party service keys (Stripe, Twilio, SendGrid)
  → SaaS integration tokens
  → Webhook secrets
  → OAuth client credentials

API KEY vs OTHER AUTH:
  Method         | Complexity | Security | Use Case
  API Key        | Low        | Low      | Public APIs, rate limiting
  OAuth Token    | Medium     | High     | User-context access
  IAM Role       | Medium     | High     | Cloud resource access
  mTLS           | High       | Very High| Service-to-service
  
  API Keys:
  → Identify the calling project/app (not user)
  → No user context
  → Cannot be scoped to specific resources
  → Easy to leak
  → Hard to audit per-user
```

---

## 2. API Key Risks

```
COMMON EXPOSURE VECTORS:

SOURCE CODE:
  → Hardcoded in application code
  → Committed to Git repositories
  → In configuration files pushed to repos
  → In environment files (.env) committed
  
  Example:
  // DO NOT DO THIS
  const API_KEY = "AKIA1234567890ABCDEF";
  const stripe = require('stripe')('sk_live_xxx...');

CLIENT-SIDE CODE:
  → JavaScript in browser (visible in source)
  → Mobile app code (extractable)
  → Compiled binaries (strings extraction)

LOGS AND MONITORING:
  → Keys in URL query parameters (logged)
  → Keys in error messages
  → Keys in debug output
  → Keys in CI/CD pipeline logs

PUBLIC EXPOSURE:
  → GitHub public repositories
  → Stack Overflow questions
  → Pastebin/paste sites
  → Docker Hub images
  → Package registries (npm, PyPI)

CONSEQUENCES OF LEAKED KEYS:
  → Unauthorized API usage (financial impact)
  → Data breach (access to stored data)
  → Service abuse (spam, fraud)
  → Resource hijacking (crypto mining)
  → Account compromise
  → Reputation damage

REAL-WORLD INCIDENTS:
  → AWS keys leaked on GitHub → $50K crypto mining bill
  → Uber: API key in public repo → data breach
  → Twitch: Leaked keys → full source code exposure
  → Multiple companies: S3 keys → data exposure
```

---

## 3. API Key Security Best Practices

```
SECURE API KEY MANAGEMENT:

1. AVOID API KEYS WHEN POSSIBLE:
   → Use IAM roles for cloud resources
   → Use OAuth for user-context access
   → Use Workload Identity for cross-platform
   → Use managed identities
   → API keys only for public API rate limiting

2. RESTRICT API KEYS:
   AWS:
   → Use IAM policies to limit actions
   → Condition keys for source IP
   → Rotate every 90 days
   
   GCP:
   → Restrict to specific APIs
   → Restrict to specific IPs/referrers
   → Set application restrictions
   
   # Restrict GCP API key
   gcloud services api-keys update KEY_ID \
     --api-target=service=maps.googleapis.com \
     --allowed-ips=10.0.0.0/8

3. STORE SECURELY:
   → Use secrets managers (never env vars in code)
   → AWS Secrets Manager / Parameter Store
   → Azure Key Vault
   → GCP Secret Manager
   → HashiCorp Vault
   → Never in source code or config files

4. ROTATE REGULARLY:
   → Automated rotation schedule
   → Dual-key rotation (create new, deploy, revoke old)
   → Monitor for rotation failures
   → Test rotation process

5. MONITOR USAGE:
   → Log all API key usage
   → Alert on unusual patterns
   → Track geographic distribution
   → Monitor rate of requests
   → Detect usage from unexpected sources

ROTATION PATTERN:
  1. Create new key (Key B)
  2. Deploy Key B to application
  3. Verify Key B working
  4. Disable Key A
  5. Monitor for errors
  6. Delete Key A after grace period
```

---

## 4. Secret Scanning

```
SECRET SCANNING TOOLS:

PRE-COMMIT (PREVENT LEAKS):
  → git-secrets (AWS): Prevent committing AWS keys
  → pre-commit hooks with detect-secrets
  → Husky + custom secret patterns
  → GitHub pre-receive hooks

  # Install git-secrets
  git secrets --install
  git secrets --register-aws
  # Now git commit will block AWS keys

REPOSITORY SCANNING:
  → GitHub Secret Scanning (built-in)
  → GitLab Secret Detection
  → truffleHog: Scan entire Git history
  → gitleaks: Fast secret scanner
  → detect-secrets (Yelp): Baseline approach

  # truffleHog scan
  trufflehog git file://./my-repo --only-verified

  # gitleaks scan
  gitleaks detect --source . --verbose

CI/CD INTEGRATION:
  → Scan on every pull request
  → Block merge if secrets found
  → Alert security team
  → Automated secret revocation
  
  # GitHub Actions example
  - name: Secret Scanning
    uses: trufflesecurity/trufflehog@main
    with:
      path: ./
      extra_args: --only-verified

CLOUD-NATIVE SCANNING:
  AWS:
  → Macie (S3 data scanning)
  → CodeGuru (code analysis)
  → GuardDuty (credential compromise detection)
  
  Azure:
  → Credential Scanner (CredScan)
  → Azure DevOps secret scanning
  → Defender for Cloud recommendations
  
  GCP:
  → Secret Manager API
  → SCC findings
  → DLP API for secret patterns

COMMON PATTERNS TO DETECT:
  Pattern              | Regex Example
  AWS Access Key       | AKIA[0-9A-Z]{16}
  AWS Secret Key       | [0-9a-zA-Z/+]{40}
  Azure Storage Key    | [A-Za-z0-9+/]{86}==
  GCP Service Key      | "private_key_id"
  GitHub Token         | gh[ps]_[A-Za-z0-9_]{36}
  Stripe Key           | sk_live_[0-9a-zA-Z]{24}
  Private Key          | -----BEGIN .* PRIVATE KEY-----
```

---

## 5. Incident Response for Leaked Keys

```
WHEN API KEYS ARE LEAKED:

IMMEDIATE ACTIONS:
  1. REVOKE the leaked key immediately
  2. Create replacement key
  3. Update all applications using the key
  4. Assess exposure timeline
  5. Check for unauthorized usage

INVESTIGATION:
  → When was the key committed/exposed?
  → How long was it public?
  → What permissions did the key have?
  → Was the key used by unauthorized parties?
  → What resources were accessed?
  
  AWS:
  → Check CloudTrail for key usage
  → Filter by access key ID
  → Look for unfamiliar IPs, regions, actions
  
  Azure:
  → Check Azure AD sign-in logs
  → Review Activity Log for operations
  → Check Key Vault audit logs
  
  GCP:
  → Check Cloud Audit Logs
  → Filter by service account
  → Review Data Access logs

REMEDIATION:
  → Rotate all potentially exposed credentials
  → Review and restrict permissions
  → Scan for additional exposed secrets
  → Implement secret scanning in CI/CD
  → Add pre-commit hooks
  → Purge secrets from Git history
  
  # Remove from Git history
  git filter-branch --force --index-filter \
    'git rm --cached --ignore-unmatch path/to/secret.conf' \
    --prune-empty --tag-name-filter cat -- --all
  
  # Or use BFG Repo Cleaner (faster)
  bfg --delete-files secret.conf
  git reflog expire --expire=now --all
  git gc --prune=now --aggressive

POST-INCIDENT:
  → Document the incident
  → Update secret management procedures
  → Train developers on secure practices
  → Implement automated scanning
  → Review access control policies
```

---

## Summary Table

| Practice | Priority | Implementation |
|----------|----------|---------------|
| Avoid API keys | High | Use IAM roles/managed identities |
| Restrict keys | High | IP/API/referrer restrictions |
| Secure storage | High | Secrets managers |
| Pre-commit scanning | High | git-secrets, gitleaks |
| Rotation | Medium | 90-day automated rotation |
| CI/CD scanning | Medium | truffleHog in pipeline |
| Monitoring | Medium | Usage alerts and logging |

---

## Revision Questions

1. Why are API keys considered less secure than IAM roles?
2. What are the most common ways API keys get exposed?
3. How should API key rotation be implemented?
4. What tools can prevent API key commits to source code?
5. What steps should be taken when an API key is leaked?

---

*Previous: [03-service-accounts.md](03-service-accounts.md) | Next: [05-secrets-management.md](05-secrets-management.md)*

---

*[Back to README](../README.md)*

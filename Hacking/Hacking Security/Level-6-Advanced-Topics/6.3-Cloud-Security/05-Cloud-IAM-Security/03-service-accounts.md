# Unit 5: Cloud IAM Security — Topic 3: Service Accounts

## Overview

**Service accounts** are non-human identities used by applications, virtual machines, and automated processes to authenticate and interact with cloud services. They are among the most targeted identities in cloud environments because they often have broad permissions and their credentials can be stolen. Securing service accounts is critical for cloud security.

---

## 1. Service Account Types

```
SERVICE ACCOUNTS ACROSS CLOUDS:

AWS:
  IAM Roles (for EC2, Lambda, ECS):
  → Attached to compute resources
  → Temporary credentials via STS
  → No long-term keys needed
  → Instance metadata service (IMDS)
  → Assumed roles (cross-account)
  
  IAM Users (service users):
  → Access key ID + Secret access key
  → Long-term credentials
  → AVOID — use roles instead
  → If needed, rotate regularly

AZURE:
  Managed Identities:
  → System-assigned: Tied to resource lifecycle
  → User-assigned: Independent, reusable
  → No credential management
  → Azure AD handles tokens
  → PREFERRED method
  
  Service Principals:
  → App registrations in Azure AD
  → Client ID + Client Secret/Certificate
  → For non-Azure workloads
  → Secrets expire (configurable)
  
  Managed Identity vs Service Principal:
  Feature         | Managed Identity | Service Principal
  Credentials     | Azure-managed    | User-managed
  Rotation        | Automatic        | Manual
  Lifecycle       | Resource-tied    | Independent
  External use    | No               | Yes

GCP:
  Service Accounts:
  → SA_NAME@PROJECT.iam.gserviceaccount.com
  → Default SA (auto-created, avoid)
  → User-managed SA (create per workload)
  → Google-managed SA (internal use)
  
  Authentication Options:
  → Attached SA (compute resources)
  → Workload Identity (GKE)
  → SA keys (JSON) — AVOID
  → Impersonation (short-lived tokens)
```

---

## 2. Service Account Risks

```
COMMON RISKS:

OVERPERMISSIONING:
  → Default SA with admin/editor roles
  → Broad managed policies attached
  → Wildcard permissions (*)
  → Cross-account access without restrictions
  → Result: Compromise = full environment access

CREDENTIAL EXPOSURE:
  → SA keys committed to source code
  → Keys in environment variables
  → Keys in container images
  → Keys in CI/CD pipelines
  → Keys shared via email/chat
  → Keys never rotated

PRIVILEGE ESCALATION:
  → SA can create other SAs
  → SA can assign IAM roles
  → SA can access secrets/keys
  → Metadata service exploitation (SSRF → IMDS)
  → Confused deputy attacks

ATTACK SCENARIOS:

  SCENARIO 1: IMDS EXPLOITATION (AWS)
  1. Find SSRF vulnerability in web app
  2. Access metadata: http://169.254.169.254/latest/meta-data/iam/security-credentials/
  3. Retrieve temporary credentials
  4. Use credentials from attacker machine
  5. Access AWS resources as the EC2 role
  
  Mitigation: IMDSv2 (requires token), restrict role permissions

  SCENARIO 2: SA KEY THEFT (GCP)
  1. Find SA key JSON file in source code repo
  2. Authenticate as service account
  3. Access GCP resources
  4. Pivot to other resources via SA permissions
  
  Mitigation: Don't create keys, use Workload Identity

  SCENARIO 3: MANAGED IDENTITY ABUSE (Azure)
  1. Compromise VM with managed identity
  2. Query IMDS for access token
  3. Use token to access Azure resources
  4. Enumerate subscriptions, key vaults, storage
  
  Mitigation: Least privilege on managed identity roles
```

---

## 3. Service Account Security

```
SECURITY BEST PRACTICES:

1. MINIMIZE SERVICE ACCOUNTS:
   → One SA per workload/application
   → No shared service accounts
   → Disable unused SAs
   → Inventory all SAs regularly

2. NO LONG-TERM CREDENTIALS:
   → AWS: Use IAM roles, not access keys
   → Azure: Use managed identities, not secrets
   → GCP: Use attached SAs, not JSON keys
   → CI/CD: Use OIDC federation (GitHub → AWS/GCP)

3. LEAST PRIVILEGE:
   → Custom roles with exact permissions
   → Resource-level scoping
   → Conditions/constraints where possible
   → Regular permission usage analysis

4. CREDENTIAL ROTATION:
   If long-term credentials are unavoidable:
   → Rotate every 90 days or less
   → Automate rotation
   → Use secrets managers
   → Monitor for exposure

5. MONITORING:
   → Log all SA authentication events
   → Alert on unusual SA activity
   → Detect credential usage from unexpected IPs
   → Monitor for SA privilege escalation
   → Track SA key creation events

CLI COMMANDS:

  AWS:
  # List access keys for IAM user
  aws iam list-access-keys --user-name service-user
  
  # Get access key last used
  aws iam get-access-key-last-used --access-key-id AKIAIOSFODNN7EXAMPLE
  
  # List roles attached to instances
  aws ec2 describe-instances --query 'Reservations[].Instances[].IamInstanceProfile'

  Azure:
  # List managed identities
  az identity list
  
  # List service principals
  az ad sp list --all --query "[].{Name:displayName, AppId:appId}"
  
  # Check SP credentials
  az ad app credential list --id APP_ID

  GCP:
  # List service accounts
  gcloud iam service-accounts list
  
  # List SA keys
  gcloud iam service-accounts keys list --iam-account=SA_EMAIL
  
  # Check SA permissions
  gcloud projects get-iam-policy PROJECT --flatten="bindings[].members" \
    --filter="bindings.members:serviceAccount:SA_EMAIL"
```

---

## 4. Secrets Management for SAs

```
SECRETS MANAGEMENT:

AWS SECRETS MANAGER:
  → Store SA credentials securely
  → Automatic rotation
  → Fine-grained access control
  → Audit access via CloudTrail
  → SDK integration
  
  # Create secret
  aws secretsmanager create-secret \
    --name api-key --secret-string "my-secret-value"
  
  # Retrieve secret
  aws secretsmanager get-secret-value --secret-id api-key

AZURE KEY VAULT:
  → Store secrets, keys, certificates
  → Access policies or RBAC
  → Soft delete and purge protection
  → HSM-backed keys
  → Managed identity integration
  
  # Create secret
  az keyvault secret set --vault-name myVault \
    --name api-key --value "my-secret-value"

GCP SECRET MANAGER:
  → Version-managed secrets
  → IAM-based access control
  → Automatic replication
  → Audit logging
  → Rotation notifications
  
  # Create secret
  echo -n "my-secret-value" | \
    gcloud secrets create api-key --data-file=-

HASHICORP VAULT:
  → Multi-cloud secrets management
  → Dynamic secrets (generated on demand)
  → Lease-based with auto-revocation
  → Multiple auth backends
  → Encryption as a service

BEST PRACTICES:
  [ ] Never hardcode credentials
  [ ] Use cloud-native secrets managers
  [ ] Rotate secrets automatically
  [ ] Audit secret access
  [ ] Encrypt secrets at rest and in transit
  [ ] Limit who/what can read secrets
  [ ] Version secrets for rollback
  [ ] Alert on unexpected secret access
```

---

## 5. Assessment and Auditing

```
SERVICE ACCOUNT AUDIT:

DISCOVERY:
  → List all SAs across all projects/accounts
  → Identify owner/creator of each SA
  → Map SA to application/workload
  → Find orphaned SAs (no linked workload)
  → Identify SAs with multiple uses

PERMISSION ANALYSIS:
  → List all roles/policies per SA
  → Compare granted vs used permissions
  → Identify admin/owner SA roles
  → Check for cross-account access
  → Review resource scope

CREDENTIAL AUDIT:
  → Age of all SA keys/secrets
  → Last rotation date
  → Last usage date
  → Keys in source code (scanning)
  → Exposed keys in public repos

TOOLS:
  → truffleHog: Scan repos for secrets
  → git-secrets: Prevent secret commits
  → detect-secrets (Yelp): Secret scanning
  → ScoutSuite: Cloud security assessment
  → Prowler: AWS security assessment
  → CloudSploit: Multi-cloud scanning

AUDIT CHECKLIST:
  [ ] All SAs inventoried and documented
  [ ] Each SA has clear owner/purpose
  [ ] Permissions follow least privilege
  [ ] No long-term credentials where avoidable
  [ ] Credentials rotated within policy
  [ ] Unused SAs disabled/deleted
  [ ] SA activity monitored
  [ ] No SA keys in source code
  [ ] Secret scanning in CI/CD pipeline
  [ ] Break-glass procedures documented
```

---

## Summary Table

| Cloud | Preferred Method | Avoid | Key Management |
|-------|-----------------|-------|----------------|
| AWS | IAM Roles | Access Keys | STS temp creds |
| Azure | Managed Identity | Service Principal secrets | Auto-managed |
| GCP | Attached SA | SA JSON keys | Workload Identity |
| Multi-cloud | OIDC Federation | Static credentials | Short-lived tokens |

---

## Revision Questions

1. Why are service accounts high-value targets for attackers?
2. What is the difference between managed identities and service principals in Azure?
3. How can IMDS exploitation lead to credential theft?
4. What secrets management practices should be followed for service accounts?
5. How should service accounts be audited for security?

---

*Previous: [02-principle-of-least-privilege.md](02-principle-of-least-privilege.md) | Next: [04-api-key-management.md](04-api-key-management.md)*

---

*[Back to README](../README.md)*

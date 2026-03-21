# Unit 4: GCP Security — Topic 2: IAM Fundamentals

## Overview

**Google Cloud IAM (Identity and Access Management)** controls who (identity) has what access (role) to which resource. GCP IAM uses a policy-based model where IAM policies bind members to roles at specific resource hierarchy levels. Understanding GCP IAM is critical for implementing least privilege and securing cloud workloads.

---

## 1. IAM Components

```
GCP IAM MODEL:

  WHO (Member) + WHAT (Role) + WHERE (Resource) = IAM Policy

MEMBERS (WHO):
  → Google Account (user@gmail.com)
  → Service Account (sa@project.iam.gserviceaccount.com)
  → Google Group (group@googlegroups.com)
  → Google Workspace domain (domain.com)
  → Cloud Identity domain
  → allUsers (public — DANGEROUS)
  → allAuthenticatedUsers (any Google account — RISKY)

ROLES (WHAT):

  PRIMITIVE ROLES (legacy, broad):
  → Owner: Full access + manage IAM
  → Editor: Read/write access
  → Viewer: Read-only access
  ⚠ AVOID these — too broad!

  PREDEFINED ROLES (service-specific):
  → roles/compute.instanceAdmin
  → roles/storage.objectViewer
  → roles/bigquery.dataEditor
  → roles/cloudsql.client
  → roles/container.admin
  → ~1000+ predefined roles

  CUSTOM ROLES:
  → Define exact permissions needed
  → Can only include available permissions
  → Bound to organization or project
  → Best for least privilege

PERMISSIONS:
  Format: service.resource.action
  Examples:
  → compute.instances.start
  → compute.instances.stop
  → storage.objects.get
  → storage.objects.create
  → iam.serviceAccounts.actAs

IAM POLICY STRUCTURE:
  {
    "bindings": [
      {
        "role": "roles/storage.objectViewer",
        "members": [
          "user:alice@example.com",
          "group:devs@example.com"
        ],
        "condition": {
          "title": "Weekday only",
          "expression": "request.time.getDayOfWeek() > 0 && request.time.getDayOfWeek() < 6"
        }
      }
    ]
  }
```

---

## 2. Service Accounts

```
SERVICE ACCOUNTS:

TYPES:
  Default Service Account:
  → Auto-created for some services
  → PROJECT_NUMBER-compute@developer.gserviceaccount.com
  → Has Editor role by default (⚠ too broad!)
  → Should be restricted immediately

  User-Managed Service Account:
  → Created by users
  → SA_NAME@PROJECT_ID.iam.gserviceaccount.com
  → Assign specific roles
  → Best practice for workloads

  Google-Managed Service Account:
  → Created and managed by Google
  → Used by GCP services internally
  → Cannot modify

KEY MANAGEMENT:
  → Service account keys (JSON/P12)
  → AVOID creating keys when possible!
  → Use Workload Identity instead
  → Keys don't expire by default
  → Rotate keys regularly if used
  → Monitor key usage

BEST PRACTICES:
  [ ] Don't use default service accounts
  [ ] Create dedicated SA per workload
  [ ] Grant minimum required roles
  [ ] Use Workload Identity for GKE
  [ ] Avoid SA key creation (use attached SAs)
  [ ] Disable unused service accounts
  [ ] Monitor with Cloud Audit Logs
  [ ] Use IAM conditions for time/resource limits

IMPERSONATION:
  → Act as a service account temporarily
  → Better than using SA keys
  → Requires iam.serviceAccounts.getAccessToken
  → Short-lived credentials
  → Full audit trail
  
  gcloud auth print-access-token \
    --impersonate-service-account=SA@PROJECT.iam.gserviceaccount.com

CLI COMMANDS:
  # List service accounts
  gcloud iam service-accounts list
  
  # Create service account
  gcloud iam service-accounts create my-sa \
    --display-name "My Service Account"
  
  # Grant role to service account
  gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/storage.objectViewer"
  
  # List SA keys
  gcloud iam service-accounts keys list \
    --iam-account=my-sa@PROJECT_ID.iam.gserviceaccount.com
```

---

## 3. IAM Conditions and Organization Policies

```
IAM CONDITIONS:
  → Add conditions to role bindings
  → Attribute-based access control (ABAC)
  → Based on: resource, request, time
  
  Examples:
  → Allow access only during business hours
  → Allow access only to specific resource names
  → Allow access only from specific IPs (via VPC-SC)
  
  Condition Expression (CEL):
  resource.name.startsWith("projects/my-project/zones/us-central1-a")
  request.time.getHours("America/New_York") >= 9 &&
  request.time.getHours("America/New_York") <= 17

ORGANIZATION POLICIES:
  → Constraints applied at org/folder/project level
  → Override IAM (even Owner can't bypass)
  → Centralized governance
  
  Common Constraints:
  → Restrict VM external IPs
    constraints/compute.vmExternalIpAccess
  → Restrict service account key creation
    constraints/iam.disableServiceAccountKeyCreation
  → Restrict resource locations
    constraints/gcp.resourceLocations
  → Require OS Login
    constraints/compute.requireOsLogin
  → Restrict public buckets
    constraints/storage.uniformBucketLevelAccess
  
  # List org policies
  gcloud org-policies list --organization=ORG_ID
  
  # Set policy
  gcloud org-policies set-policy policy.yaml \
    --organization=ORG_ID
```

---

## 4. IAM Security Assessment

```
ASSESSMENT AND ENUMERATION:

# Get project IAM policy
gcloud projects get-iam-policy PROJECT_ID

# Find specific role bindings
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.role:roles/owner" \
  --format="table(bindings.members)"

# List custom roles
gcloud iam roles list --project=PROJECT_ID

# Describe a role (see permissions)
gcloud iam roles describe roles/compute.instanceAdmin

# Test IAM permissions
gcloud asset search-all-iam-policies \
  --scope=projects/PROJECT_ID

COMMON MISCONFIGURATIONS:
  → allUsers or allAuthenticatedUsers bindings
  → Primitive roles (Owner/Editor) at project level
  → Service accounts with Owner role
  → SA keys not rotated
  → Default service accounts with Editor role
  → Over-permissioned custom roles
  → No IAM conditions used
  → Missing audit logging

SECURITY TOOLS:
  → ScoutSuite: Multi-cloud security assessment
  → Forseti Security: GCP policy scanner (deprecated)
  → GCP SCC: Native security posture management
  → Cartography: Infrastructure graph analysis
  → gcpdiag: GCP diagnostics tool

FINDING OVER-PERMISSIONED ACCOUNTS:
  → IAM Recommender (built-in)
  → Analyzes permission usage patterns
  → Suggests role downgrades
  → Policy Analyzer for understanding access
  
  # Check IAM recommendations
  gcloud recommender recommendations list \
    --recommender=google.iam.policy.Recommender \
    --project=PROJECT_ID --location=global
```

---

## 5. IAM Deny Policies

```
IAM DENY POLICIES (newer feature):

PURPOSE:
  → Override ALLOW policies
  → Block specific permissions regardless of roles
  → Additional layer of defense
  → Applied at org, folder, or project level

HOW IT WORKS:
  Standard IAM: Only ALLOW (additive, most permissive wins)
  
  With Deny Policies:
  1. Check deny policies FIRST
  2. If denied → ACCESS DENIED (regardless of allow)
  3. If not denied → Check allow policies
  4. If allowed → ACCESS GRANTED
  5. If not allowed → ACCESS DENIED

USE CASES:
  → Prevent anyone from disabling audit logs
  → Block public access to storage (even Owners)
  → Prevent deletion of critical resources
  → Block service account key creation
  → Restrict sensitive API calls

EXAMPLE:
  # Deny policy: prevent disabling Cloud Audit Logs
  {
    "displayName": "Deny disable audit logs",
    "rules": [
      {
        "denyRule": {
          "deniedPrincipals": ["principalSet://goog/public:all"],
          "deniedPermissions": [
            "logging.sinks.delete",
            "logging.sinks.update"
          ]
        }
      }
    ]
  }
```

---

## Summary Table

| Component | Description | Best Practice |
|-----------|-------------|---------------|
| Primitive Roles | Owner/Editor/Viewer | Avoid — too broad |
| Predefined Roles | Service-specific | Use as baseline |
| Custom Roles | Exact permissions | Use for least privilege |
| Service Accounts | Workload identity | Dedicated per workload |
| SA Keys | Credential files | Avoid — use attached SAs |
| IAM Conditions | ABAC constraints | Add for sensitive access |
| Org Policies | Governance rules | Enforce at org level |
| Deny Policies | Override allows | Block critical actions |

---

## Revision Questions

1. What are the three types of roles in GCP IAM and when should each be used?
2. Why should default service accounts be avoided?
3. How do IAM conditions enable attribute-based access control?
4. What is the relationship between Organization Policies and IAM?
5. How do IAM deny policies provide additional security?

---

*Previous: [01-gcp-security-architecture.md](01-gcp-security-architecture.md) | Next: [03-vpc-and-firewall-rules.md](03-vpc-and-firewall-rules.md)*

---

*[Back to README](../README.md)*

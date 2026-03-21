# Unit 4: GCP Security — Topic 4: Cloud Audit Logs

## Overview

**Google Cloud Audit Logs** record administrative activities and access to data within GCP resources. Audit logs provide a comprehensive trail of who did what, where, and when across your GCP environment. They are essential for security monitoring, incident investigation, compliance, and forensic analysis.

---

## 1. Audit Log Types

```
GCP AUDIT LOG CATEGORIES:

ADMIN ACTIVITY LOGS:
  → API calls that modify resources
  → Always enabled (cannot be disabled)
  → Free (no charge)
  → Retained for 400 days
  → Examples:
    - Creating/deleting VMs
    - Changing IAM policies
    - Modifying firewall rules
    - Creating storage buckets
    - Updating configurations

DATA ACCESS LOGS:
  → API calls that read resource data
  → Disabled by default (must enable)
  → Charges apply (can be expensive)
  → Retained for 30 days (default)
  → Sub-categories:
    - ADMIN_READ: Reading resource metadata
    - DATA_READ: Reading user data
    - DATA_WRITE: Writing user data
  → Examples:
    - Reading storage objects
    - Querying BigQuery datasets
    - Listing VM instances
    - Reading database records

SYSTEM EVENT LOGS:
  → Google-generated administrative actions
  → Always enabled
  → Free
  → Retained for 400 days
  → Examples:
    - Live migration of VMs
    - Automatic scaling events
    - System maintenance actions

POLICY DENIED LOGS:
  → Logged when access is denied by policy
  → VPC Service Controls violations
  → Organization Policy violations
  → Security-relevant denied actions

LOG ENTRY STRUCTURE:
  {
    "logName": "projects/my-project/logs/cloudaudit.googleapis.com%2Factivity",
    "resource": {
      "type": "gce_instance",
      "labels": { "instance_id": "12345" }
    },
    "protoPayload": {
      "methodName": "v1.compute.instances.delete",
      "authenticationInfo": {
        "principalEmail": "user@example.com"
      },
      "requestMetadata": {
        "callerIp": "203.0.113.1",
        "callerSuppliedUserAgent": "gcloud/400.0.0"
      },
      "authorizationInfo": [{
        "permission": "compute.instances.delete",
        "granted": true
      }]
    },
    "timestamp": "2024-01-15T10:30:00Z"
  }
```

---

## 2. Configuring Audit Logs

```
ENABLING DATA ACCESS LOGS:

# Via gcloud (organization level)
gcloud organizations get-iam-policy ORG_ID > policy.yaml

# Edit policy.yaml to add:
auditConfigs:
- service: allServices
  auditLogConfigs:
  - logType: ADMIN_READ
  - logType: DATA_READ
  - logType: DATA_WRITE
- service: storage.googleapis.com
  auditLogConfigs:
  - logType: DATA_READ
  - logType: DATA_WRITE

gcloud organizations set-iam-policy ORG_ID policy.yaml

# At project level
gcloud projects get-iam-policy PROJECT_ID > policy.yaml
# Edit and set

EXEMPTING HIGH-VOLUME SERVICES:
  → Exclude specific users from data access logs
  → Reduce log volume and cost
  → Be careful — security blind spots

  auditConfigs:
  - service: bigquery.googleapis.com
    auditLogConfigs:
    - logType: DATA_READ
      exemptedMembers:
      - serviceAccount:analytics@project.iam.gserviceaccount.com

LOG ROUTING AND EXPORT:
  # Create log sink to Cloud Storage
  gcloud logging sinks create audit-sink \
    storage.googleapis.com/my-audit-bucket \
    --log-filter='logName:"cloudaudit.googleapis.com"'

  # Create log sink to BigQuery
  gcloud logging sinks create audit-bq-sink \
    bigquery.googleapis.com/projects/PROJECT/datasets/audit_logs \
    --log-filter='logName:"cloudaudit.googleapis.com"'

  # Create aggregated org-level sink
  gcloud logging sinks create org-audit-sink \
    storage.googleapis.com/org-audit-bucket \
    --organization=ORG_ID \
    --include-children \
    --log-filter='logName:"cloudaudit.googleapis.com"'
```

---

## 3. Querying Audit Logs

```
LOG EXPLORER QUERIES:

# All admin activity in last 24 hours
resource.type="project"
logName="projects/PROJECT/logs/cloudaudit.googleapis.com%2Factivity"

# IAM policy changes
protoPayload.methodName="SetIamPolicy"

# Firewall rule modifications
resource.type="gce_firewall_rule"
protoPayload.methodName=~"firewalls\.(insert|delete|patch)"

# Storage bucket access
resource.type="gcs_bucket"
protoPayload.methodName=~"storage\.buckets\."

# Failed access attempts
protoPayload.status.code!=0

# Actions by specific user
protoPayload.authenticationInfo.principalEmail="user@example.com"

# Service account key creation
protoPayload.methodName="google.iam.admin.v1.CreateServiceAccountKey"

# VM instance operations
resource.type="gce_instance"
protoPayload.methodName=~"compute\.instances\.(insert|delete|start|stop)"

gcloud CLI QUERIES:
  # List recent admin activity
  gcloud logging read \
    'logName:"cloudaudit.googleapis.com/activity"' \
    --project=PROJECT_ID \
    --limit=50 \
    --format="table(timestamp, protoPayload.methodName, protoPayload.authenticationInfo.principalEmail)"

  # Find IAM changes
  gcloud logging read \
    'protoPayload.methodName="SetIamPolicy"' \
    --project=PROJECT_ID \
    --freshness=7d

  # Find storage data access
  gcloud logging read \
    'logName:"cloudaudit.googleapis.com/data_access" AND resource.type="gcs_bucket"' \
    --project=PROJECT_ID
```

---

## 4. Security Monitoring with Audit Logs

```
KEY EVENTS TO MONITOR:

IDENTITY EVENTS:
  → IAM policy changes (SetIamPolicy)
  → Service account key creation/deletion
  → Service account creation
  → Role binding changes
  → User added to privileged group

NETWORK EVENTS:
  → Firewall rule changes
  → VPC creation/deletion
  → Route modifications
  → Load balancer configuration changes
  → VPN tunnel changes

DATA EVENTS:
  → Storage bucket policy changes
  → Bucket made public
  → Large data downloads
  → BigQuery exports
  → Cross-project data access

COMPUTE EVENTS:
  → VM creation in unusual region
  → Startup script modifications
  → Serial console access
  → Metadata modifications
  → Image/snapshot exports

ALERT EXAMPLES:
  Alert: "Service Account Key Created"
  Filter: protoPayload.methodName="google.iam.admin.v1.CreateServiceAccountKey"
  Severity: High
  
  Alert: "Firewall Rule Allows All Traffic"
  Filter: resource.type="gce_firewall_rule" AND 
          protoPayload.request.sourceRanges="0.0.0.0/0"
  Severity: Critical
  
  Alert: "Bucket Made Public"
  Filter: protoPayload.serviceData.policyDelta.bindingDeltas.member="allUsers"
  Severity: Critical

INTEGRATION WITH SCC:
  → Event Threat Detection (ETD)
  → Analyzes audit logs automatically
  → Detects suspicious patterns:
    - Cryptocurrency mining
    - Malware communication
    - Brute force attacks
    - Data exfiltration
    - Privilege escalation
```

---

## 5. Compliance and Retention

```
RETENTION AND COMPLIANCE:

DEFAULT RETENTION:
  Admin Activity:    400 days
  Data Access:       30 days
  System Events:     400 days
  Policy Denied:     30 days

EXTENDING RETENTION:
  → Export to Cloud Storage (long-term archive)
  → Export to BigQuery (queryable archive)
  → Set bucket retention policies
  → Compliance-grade archival
  
  # Create locked retention policy
  gsutil retention set 2555d gs://audit-archive-bucket
  gsutil retention lock gs://audit-archive-bucket

COMPLIANCE REQUIREMENTS:
  Standard    | Retention   | Requirement
  PCI DSS     | 1 year      | All access to cardholder data
  HIPAA       | 6 years     | Access to PHI
  SOX         | 7 years     | Financial system access
  GDPR        | Varies      | Processing activity records
  SOC 2       | 1 year      | Logical access monitoring

ACCESS TRANSPARENCY:
  → Separate from Cloud Audit Logs
  → Logs when Google staff access your data
  → Available with Premium support
  → Covers GCP and Google Workspace
  → Includes justification for access
  → Required for some compliance frameworks

BEST PRACTICES:
  [ ] Enable Data Access logs for sensitive services
  [ ] Export all audit logs to central location
  [ ] Retention period meets compliance needs
  [ ] Log sinks have appropriate permissions
  [ ] Alert on critical security events
  [ ] Regular audit log review process
  [ ] Protect log storage from deletion
  [ ] Enable Access Transparency if available
  [ ] Test log queries for incident response
  [ ] Document log retention policy
```

---

## Summary Table

| Log Type | Default State | Retention | Cost |
|----------|--------------|-----------|------|
| Admin Activity | Always on | 400 days | Free |
| Data Access | Off (enable) | 30 days | Paid |
| System Events | Always on | 400 days | Free |
| Policy Denied | Always on | 30 days | Free |

---

## Revision Questions

1. What are the four types of GCP audit logs?
2. Why are Data Access logs not enabled by default?
3. How can audit logs be exported for long-term retention?
4. What critical security events should be monitored via audit logs?
5. How does Event Threat Detection leverage audit logs?

---

*Previous: [03-vpc-and-firewall-rules.md](03-vpc-and-firewall-rules.md) | Next: [05-security-command-center.md](05-security-command-center.md)*

---

*[Back to README](../README.md)*

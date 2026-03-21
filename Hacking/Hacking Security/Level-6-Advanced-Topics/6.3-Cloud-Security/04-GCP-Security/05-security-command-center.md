# Unit 4: GCP Security — Topic 5: Security Command Center

## Overview

**Google Cloud Security Command Center (SCC)** is GCP's native security posture management platform. It provides comprehensive visibility into security risks, vulnerabilities, misconfigurations, and threats across your GCP organization. SCC aggregates findings from multiple security services and provides actionable recommendations to improve your security posture.

---

## 1. SCC Overview

```
SECURITY COMMAND CENTER:

  ┌──────────────────────────────────────────┐
  │     SECURITY COMMAND CENTER              │
  ├──────────────────┬───────────────────────┤
  │ Standard (Free)  │ Premium (Paid)        │
  │                  │                       │
  │ • Security Health│ • All Standard +      │
  │   Analytics      │ • Event Threat Detect  │
  │ • Web Security   │ • Container Threat Det │
  │   Scanner        │ • Virtual Machine TD   │
  │ • Anomaly        │ • Compliance reports   │
  │   Detection      │ • Continuous exports   │
  │ • Asset          │ • Attack path analysis │
  │   Discovery      │ • Toxic combinations   │
  │                  │ • Posture management   │
  └──────────────────┴───────────────────────┘

KEY CAPABILITIES:
  → Asset inventory and discovery
  → Security Health Analytics (misconfigurations)
  → Web Security Scanner (vulnerability scanning)
  → Event Threat Detection (log analysis)
  → Container Threat Detection (runtime)
  → Virtual Machine Threat Detection
  → Compliance monitoring
  → Attack path simulation

FINDINGS:
  → Security issues identified by SCC
  → Categorized by severity (Critical, High, Medium, Low)
  → Mapped to CIS benchmarks and compliance
  → Include remediation guidance
  → Can be muted, marked as resolved
```

---

## 2. Security Health Analytics

```
SECURITY HEALTH ANALYTICS:

DETECTORS (examples):

IDENTITY FINDINGS:
  → MFA_NOT_ENFORCED: MFA not enabled
  → ADMIN_SERVICE_ACCOUNT: SA has admin role
  → OVER_PRIVILEGED_ACCOUNT: Excessive permissions
  → SERVICE_ACCOUNT_KEY_NOT_ROTATED: Old SA keys
  → TOO_MANY_KMS_USERS: Excessive KMS access

NETWORK FINDINGS:
  → OPEN_FIREWALL: Overly permissive firewall
  → OPEN_SSH_PORT: SSH open to 0.0.0.0/0
  → OPEN_RDP_PORT: RDP open to 0.0.0.0/0
  → FIREWALL_NOT_MONITORED: No log alerts
  → FLOW_LOGS_DISABLED: VPC flow logs off

STORAGE FINDINGS:
  → PUBLIC_BUCKET_ACL: Bucket publicly accessible
  → BUCKET_POLICY_ONLY_DISABLED: Legacy ACLs
  → PUBLIC_LOG_BUCKET: Log bucket is public
  → BUCKET_LOGGING_DISABLED: No access logs

COMPUTE FINDINGS:
  → PUBLIC_IP_ADDRESS: VM has public IP
  → COMPUTE_SECURE_BOOT_DISABLED: No Secure Boot
  → DEFAULT_SERVICE_ACCOUNT: Using default SA
  → OS_LOGIN_DISABLED: OS Login not enforced

DATABASE FINDINGS:
  → SQL_PUBLIC_IP: Cloud SQL has public IP
  → SQL_NO_ROOT_PASSWORD: No root password
  → SQL_CROSS_DB_OWNERSHIP: Cross-DB chaining

ENCRYPTION:
  → KMS_KEY_NOT_ROTATED: Key not rotated
  → DEFAULT_ENCRYPTION: Not using CMEK
  → KMS_PUBLIC_KEY: KMS key publicly accessible
```

---

## 3. Threat Detection

```
EVENT THREAT DETECTION (ETD):

WHAT IT DETECTS:
  → Analyzes Cloud Audit Logs automatically
  → Uses threat intelligence and ML
  → Real-time detection
  → Premium tier only

DETECTION CATEGORIES:

MALWARE:
  → Cryptomining detection
  → Malicious script execution
  → Known malware signatures
  → C2 communication patterns

CREDENTIAL THREATS:
  → Brute force SSH/RDP
  → Leaked credentials used
  → Anomalous service account usage
  → Privilege escalation attempts

DATA EXFILTRATION:
  → Large data transfers
  → Cross-project data copying
  → External sharing of sensitive data
  → BigQuery data export anomalies

EVASION:
  → Audit log tampering
  → Firewall rule manipulation
  → VPC Service Control bypass attempts
  → Organization policy modifications

CONTAINER THREAT DETECTION:
  → Detects runtime threats in GKE
  → Suspicious binaries executed
  → Reverse shell detection
  → Container escape attempts
  → Privilege escalation in containers
  → Malicious scripts in containers

VIRTUAL MACHINE THREAT DETECTION:
  → Memory scanning for threats
  → Kernel-level rootkit detection
  → Cryptocurrency mining detection
  → No agent required (hypervisor-level)

EXAMPLE FINDING:
  Finding: CRYPTO_MINING_DETECTED
  Severity: Critical
  Category: Malware
  Resource: VM instance "web-server-1"
  Description: "Cryptocurrency mining software 
                detected on instance"
  Remediation: "Stop the instance, investigate,
                rebuild from clean image"
```

---

## 4. Compliance and Reporting

```
COMPLIANCE MONITORING:

SUPPORTED STANDARDS:
  → CIS Google Cloud Computing Platform Benchmark
  → PCI DSS 3.2.1 / 4.0
  → NIST 800-53
  → ISO 27001
  → SOC 2
  → HIPAA
  → Custom benchmarks

COMPLIANCE DASHBOARD:
  Standard              | Compliance % | Violations
  CIS GCP Benchmark     | 78%          | 45
  PCI DSS               | 72%          | 32
  NIST 800-53           | 68%          | 58
  ISO 27001             | 75%          | 38

ATTACK PATH ANALYSIS (Premium):
  → Simulates attacker paths through your environment
  → Identifies high-value targets
  → Shows exploitation chains
  → Prioritizes remediation
  
  Example Path:
  Public VM → Compromised SA → Access Storage → 
  Read Sensitive Data → Exfiltrate

EXPORT AND INTEGRATION:
  → Export findings to Pub/Sub
  → Forward to SIEM (Chronicle, Splunk)
  → Webhook notifications
  → Cloud Functions triggers
  → BigQuery for analytics
  
  # Export to Pub/Sub
  gcloud scc notifications create my-notification \
    --organization=ORG_ID \
    --pubsub-topic=projects/PROJECT/topics/scc-findings \
    --filter='severity="CRITICAL" OR severity="HIGH"'
```

---

## 5. Using SCC for Assessment

```
CLI COMMANDS:

# List all findings
gcloud scc findings list ORG_ID \
  --source="-" \
  --filter='state="ACTIVE" AND severity="HIGH"'

# List assets
gcloud scc assets list ORG_ID

# Get specific finding
gcloud scc findings list ORG_ID \
  --source="-" \
  --filter='category="OPEN_FIREWALL"'

# Update finding state
gcloud scc findings update ORG_ID \
  --source=SOURCE_ID \
  --finding=FINDING_ID \
  --state=INACTIVE

API ACCESS:
  from google.cloud import securitycenter
  
  client = securitycenter.SecurityCenterClient()
  org_name = "organizations/ORG_ID"
  
  # List high severity findings
  finding_iterator = client.list_findings(
      request={
          "parent": f"{org_name}/sources/-",
          "filter": 'severity="HIGH" AND state="ACTIVE"'
      }
  )
  
  for finding_result in finding_iterator:
      print(finding_result.finding.category)
      print(finding_result.finding.resource_name)

SECURITY ASSESSMENT WORKFLOW:
  1. Enable SCC Premium (trial available)
  2. Review Security Health Analytics findings
  3. Prioritize Critical and High severity
  4. Review attack paths
  5. Check compliance posture
  6. Enable Event Threat Detection
  7. Set up notifications for new findings
  8. Create remediation plan
  9. Track progress over time
  10. Regular posture reviews

THIRD-PARTY TOOLS:
  → ScoutSuite: Open-source cloud auditing
  → Prowler (limited GCP support)
  → gcpdiag: GCP diagnostics
  → Cartography: Infrastructure mapping
  → Cloud Custodian: Policy as code
```

---

## Summary Table

| Feature | Tier | Key Capability |
|---------|------|---------------|
| Security Health Analytics | Standard | Misconfiguration detection |
| Web Security Scanner | Standard | Vulnerability scanning |
| Event Threat Detection | Premium | Log-based threat detection |
| Container Threat Detection | Premium | Runtime container security |
| VM Threat Detection | Premium | Memory-level threat detection |
| Attack Path Analysis | Premium | Exploitation path simulation |

---

## Revision Questions

1. What is the difference between SCC Standard and Premium tiers?
2. What types of misconfigurations does Security Health Analytics detect?
3. How does Event Threat Detection identify threats?
4. What is attack path analysis and how does it help prioritize remediation?
5. How can SCC findings be exported for integration with external systems?

---

*Previous: [04-cloud-audit-logs.md](04-cloud-audit-logs.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

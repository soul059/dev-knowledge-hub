# Chapter 4: Security & Compliance

[← Previous: Cost Optimization](03-cost-optimization.md) | [Back to README](../README.md) | [Next: Observability as Code →](05-observability-as-code.md)

---

## Overview

Observability data often contains sensitive information — user IDs, IP addresses, request payloads, and internal system details. Securing this data, controlling access, and ensuring regulatory compliance (GDPR, HIPAA, SOC2, PCI-DSS) is essential for production systems.

---

## 4.1 Security Threat Model

```
┌──────────────────────────────────────────────────────────┐
│  OBSERVABILITY SECURITY RISKS                            │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │ DATA IN TRANSIT                                  │    │
│  │ • Unencrypted metrics/logs/traces between        │    │
│  │   services and collectors                        │    │
│  │ • Risk: eavesdropping, data tampering            │    │
│  │ • Fix: TLS everywhere                           │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │ DATA AT REST                                     │    │
│  │ • Sensitive data in logs/traces stored on disk   │    │
│  │ • Risk: unauthorized access to storage           │    │
│  │ • Fix: encryption, access controls               │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │ DATA EXPOSURE (PII LEAKAGE)                      │    │
│  │ • Passwords, tokens, PII in log messages         │    │
│  │ • Credit card numbers in trace attributes        │    │
│  │ • Risk: compliance violations, data breach       │    │
│  │ • Fix: scrub/redact at pipeline level            │    │
│  └─────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────┐    │
│  │ ACCESS CONTROL                                    │    │
│  │ • Over-permissive access to dashboards           │    │
│  │ • Risk: internal data leak, unauthorized changes │    │
│  │ • Fix: RBAC, SSO, audit logging                  │    │
│  └─────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 PII Detection and Redaction

### What to Redact

```
┌──────────────────────────────────────────────────────────┐
│  PII / SENSITIVE DATA CATEGORIES                         │
│                                                          │
│  Category          │ Examples                            │
│  ──────────────────┼──────────────────────────────────   │
│  Personal Identity │ Name, email, phone, SSN, DOB       │
│  Authentication    │ Passwords, API keys, tokens, JWTs  │
│  Financial         │ Credit card, bank account, CVV     │
│  Network           │ Internal IPs, hostnames, routes    │
│  Health (HIPAA)    │ Diagnoses, medications, records    │
│  Location          │ GPS coordinates, addresses          │
│  Custom            │ User IDs (if GDPR applies)         │
└──────────────────────────────────────────────────────────┘
```

### Redaction at Pipeline

```yaml
# OTel Collector: redact sensitive attributes
processors:
  # Remove specific attributes
  attributes/redact:
    actions:
      - key: http.request.header.authorization
        action: delete
      - key: http.request.header.cookie
        action: delete
      - key: db.statement
        action: hash        # Hash instead of delete (preserves uniqueness)
      - key: user.email
        action: delete
      - key: enduser.id
        action: hash

  # Regex-based redaction in log bodies
  transform/redact:
    log_statements:
      - context: log
        statements:
          # Redact email addresses
          - replace_pattern(body, 
              "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}", 
              "[REDACTED_EMAIL]")
          # Redact credit card numbers
          - replace_pattern(body,
              "\\b\\d{4}[- ]?\\d{4}[- ]?\\d{4}[- ]?\\d{4}\\b",
              "[REDACTED_CC]")
          # Redact API keys (common patterns)
          - replace_pattern(body,
              "(?i)(api[_-]?key|token|secret)[=: ]+[\"']?[a-zA-Z0-9_-]{20,}",
              "$1=[REDACTED]")

service:
  pipelines:
    logs:
      processors: [attributes/redact, transform/redact, batch]
    traces:
      processors: [attributes/redact, batch]
```

---

## 4.3 TLS Configuration

```yaml
# OTel Collector TLS - receiver side
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"
        tls:
          cert_file: /certs/collector.crt
          key_file: /certs/collector.key
          client_ca_file: /certs/ca.crt  # mTLS

# OTel Collector TLS - exporter side
exporters:
  otlp/backend:
    endpoint: "tempo.internal:4317"
    tls:
      cert_file: /certs/client.crt
      key_file: /certs/client.key
      ca_file: /certs/ca.crt

# Prometheus scrape with TLS
scrape_configs:
  - job_name: "secure-targets"
    scheme: https
    tls_config:
      ca_file: /certs/ca.crt
      cert_file: /certs/prometheus.crt
      key_file: /certs/prometheus.key
```

---

## 4.4 Access Control (RBAC)

```
┌──────────────────────────────────────────────────────────┐
│  RBAC MODEL FOR OBSERVABILITY                            │
│                                                          │
│  Role              │ Permissions                         │
│  ──────────────────┼──────────────────────────────────   │
│  Viewer            │ View dashboards, read metrics/logs  │
│  Editor            │ Create/edit dashboards, alerts      │
│  Admin             │ Manage data sources, users, orgs    │
│  Team Lead         │ View team data, manage team alerts  │
│  On-Call           │ Silence/acknowledge alerts          │
│  Security          │ View audit logs, manage access      │
│                                                          │
│  DATA ISOLATION:                                         │
│  • Grafana: Organization-based isolation                │
│  • Mimir/Loki/Tempo: X-Scope-OrgID header              │
│  • Elasticsearch: Index-level security (OpenSearch)     │
└──────────────────────────────────────────────────────────┘
```

### Grafana RBAC Configuration

```yaml
# grafana.ini
[auth]
disable_login_form = true

[auth.generic_oauth]
enabled = true
name = Okta
allow_sign_up = true
client_id = ${OAUTH_CLIENT_ID}
client_secret = ${OAUTH_CLIENT_SECRET}
scopes = openid profile email groups
auth_url = https://company.okta.com/oauth2/v1/authorize
token_url = https://company.okta.com/oauth2/v1/token
api_url = https://company.okta.com/oauth2/v1/userinfo
role_attribute_path = contains(groups[*], 'grafana-admins') && 'Admin' || contains(groups[*], 'grafana-editors') && 'Editor' || 'Viewer'

[auth.generic_oauth.group_mapping]
org_id = 1
teams_url = https://company.okta.com/api/v1/users/me/groups
```

---

## 4.5 Audit Logging

```yaml
# Grafana audit logging
[log]
level = info

[auditing]
enabled = true
loggers = file
log_dashboard_access = true

# Logs all actions: dashboard view, edit, delete, 
# data source changes, user management, etc.

# Elasticsearch audit logging (opensearch_security)
opendistro_security:
  audit:
    enable_rest: true
    enable_transport: true
    resolve_indices: true
    log_request_body: false    # Don't log request bodies (may contain PII)
    ignore_users:
      - "kibanaserver"
    ignore_requests:
      - "SearchRequest"        # Too noisy
```

---

## 4.6 Compliance Mapping

```
┌──────────────────────────────────────────────────────────────┐
│  COMPLIANCE REQUIREMENTS MAPPED TO OBSERVABILITY             │
│                                                              │
│  Regulation │ Requirement           │ Implementation         │
│  ───────────┼───────────────────────┼──────────────────────  │
│  GDPR       │ Right to erasure      │ Redact PII, retention │
│             │ Data minimization     │ Collect only needed    │
│             │ Lawful basis          │ Document purpose       │
│             │ Data processing log   │ Audit logging          │
│  ───────────┼───────────────────────┼──────────────────────  │
│  HIPAA      │ PHI protection        │ Encrypt, access ctrl  │
│             │ Audit trail           │ Audit logging          │
│             │ Minimum necessary     │ Redact health data     │
│  ───────────┼───────────────────────┼──────────────────────  │
│  SOC 2      │ Access controls       │ RBAC, SSO             │
│             │ Monitoring            │ Audit logs, alerts     │
│             │ Change management     │ GitOps for dashboards  │
│  ───────────┼───────────────────────┼──────────────────────  │
│  PCI-DSS    │ No card data in logs  │ Redact CC numbers     │
│             │ Network segmentation  │ TLS, network policies │
│             │ Log retention (1yr)   │ Archive to cold store  │
└──────────────────────────────────────────────────────────────┘
```

### Data Retention Policy

```yaml
# Retention policy configuration
retention_policies:
  metrics:
    hot:   14d    # SSD, fast queries
    warm:  90d    # S3 Standard
    cold:  365d   # S3 Glacier
    
  logs:
    security_logs:  365d   # Compliance requirement
    application:    30d
    debug:          0d     # Don't store
    
  traces:
    hot:    7d
    warm:   30d
    archive: 90d

# GDPR: data deletion process
# When user requests deletion:
# 1. Identify data by user_id (hashed in telemetry)
# 2. Since telemetry data is aggregated/anonymized at pipeline,
#    individual deletion may not be required
# 3. Document approach in Data Protection Impact Assessment
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| PII found in logs | No pipeline redaction | Add transform/redact processor |
| Unauthorized dashboard access | No RBAC configured | Enable SSO + Grafana org-based RBAC |
| Audit gap in compliance review | No audit logging | Enable Grafana/ES audit logging |
| Compliance violation on retention | Logs retained too long or too short | Configure per-stream retention with lifecycle policies |
| Internal IPs exposed | Trace attributes contain network info | Redact specific attributes at collector |

---

## Summary Table

| Area | Key Practice |
|------|--------------|
| PII Redaction | Scrub at pipeline level before storage |
| Encryption | TLS in transit, encryption at rest |
| Access Control | RBAC via SSO, org-level data isolation |
| Audit | Log all access and changes to dashboards/alerts |
| Retention | Per-signal/per-team retention policies |
| Compliance | Map regulations to specific technical controls |

---

## Quick Revision Questions

1. **What types of sensitive data commonly appear in observability telemetry?**
2. **How do you redact PII in an OTel Collector pipeline?**
3. **How does multi-tenant isolation work in Grafana + Mimir/Loki/Tempo?**
4. **What are the key GDPR requirements that apply to observability data?**
5. **Why is audit logging important for SOC 2 compliance?**

---

[← Previous: Cost Optimization](03-cost-optimization.md) | [Back to README](../README.md) | [Next: Observability as Code →](05-observability-as-code.md)

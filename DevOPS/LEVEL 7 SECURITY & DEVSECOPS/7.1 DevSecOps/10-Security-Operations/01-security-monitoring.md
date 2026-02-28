# Chapter 10.1: Security Monitoring

## Overview

Security monitoring is the continuous observation of systems, networks, and applications to detect threats, anomalies, and policy violations in real time. In DevSecOps, monitoring extends from infrastructure to application layers, integrating with CI/CD pipelines and cloud-native services. This chapter covers monitoring strategy, tools, and implementation.

---

## Security Monitoring Architecture

```
SECURITY MONITORING ARCHITECTURE:

  DATA SOURCES                    COLLECTION           ANALYSIS & ACTION
  ────────────                    ──────────           ─────────────────

  ┌──────────┐                  ┌──────────────┐     ┌──────────────┐
  │ App Logs │──────┐           │              │     │              │
  └──────────┘      │           │    LOG       │     │    SIEM      │
  ┌──────────┐      ├──────────▶│  AGGREGATOR  │────▶│   (Splunk/   │
  │ OS Logs  │──────┤           │  (Fluentd/   │     │   ELK/       │
  └──────────┘      │           │   Vector/    │     │   Wazuh)     │
  ┌──────────┐      │           │   Logstash)  │     │              │
  │ Cloud    │──────┤           │              │     │  • Correlate │
  │ Trails   │      │           └──────────────┘     │  • Alert     │
  └──────────┘      │                                │  • Dashboard │
  ┌──────────┐      │           ┌──────────────┐     │  • Respond   │
  │ Network  │──────┤           │              │     │              │
  │ Flows    │      ├──────────▶│   METRICS    │────▶│              │
  └──────────┘      │           │  (Prometheus │     └──────┬───────┘
  ┌──────────┐      │           │   Datadog)   │            │
  │ K8s Audit│──────┤           │              │            ▼
  │ Logs     │      │           └──────────────┘     ┌──────────────┐
  └──────────┘      │                                │   RESPONSE   │
  ┌──────────┐      │           ┌──────────────┐     │              │
  │ Runtime  │──────┘           │  RUNTIME     │────▶│  • PagerDuty │
  │ (Falco)  │─────────────────▶│  DETECTION   │     │  • Slack     │
  └──────────┘                  │  (Falco/     │     │  • SOAR      │
                                │   Sysdig)    │     │  • Tickets   │
                                └──────────────┘     └──────────────┘
```

---

## The ELK Stack (Elasticsearch, Logstash, Kibana)

```
ELK STACK ARCHITECTURE:

  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │  Beats   │───▶│ Logstash │───▶│ Elastic  │───▶│ Kibana   │
  │ (agents) │    │ (parse/  │    │ search   │    │ (visual- │
  │          │    │  enrich) │    │ (store/  │    │  ize)    │
  │ Filebeat │    │          │    │  index)  │    │          │
  │ Auditbeat│    │ Grok     │    │          │    │ Dashbords│
  │ Metricbt │    │ Filters  │    │ Queries  │    │ Alerts   │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### Logstash Security Pipeline

```ruby
# logstash-security.conf
input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate => "/etc/logstash/ssl/logstash.crt"
    ssl_key => "/etc/logstash/ssl/logstash.key"
  }
}

filter {
  # Parse SSH authentication logs
  if [fileset][name] == "auth" {
    grok {
      match => {
        "message" => "Failed password for %{USER:username} from %{IP:src_ip} port %{NUMBER:src_port}"
      }
      tag_on_failure => ["_ssh_parse_failure"]
    }
    
    # GeoIP enrichment
    geoip {
      source => "src_ip"
      target => "geoip"
    }
    
    # Tag brute force attempts
    throttle {
      before_count => 5
      after_count => 10
      period => 300
      key => "%{src_ip}"
      add_tag => "brute_force_suspect"
    }
  }

  # Parse application security events
  if [kubernetes][labels][app] == "webapp" {
    json {
      source => "message"
    }
    
    # Detect SQL injection attempts
    if [request_uri] =~ /('|--|union|select|drop|insert|update|delete)/i {
      mutate {
        add_tag => ["sql_injection_attempt"]
        add_field => { "security_event" => "sqli_detected" }
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["https://elasticsearch:9200"]
    index => "security-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "${ES_PASSWORD}"
  }
  
  # Alert on critical security events
  if "brute_force_suspect" in [tags] or "sql_injection_attempt" in [tags] {
    http {
      url => "https://hooks.slack.com/services/xxx/yyy/zzz"
      http_method => "post"
      format => "json"
      mapping => {
        "text" => "🚨 Security Alert: %{security_event} from %{src_ip}"
      }
    }
  }
}
```

---

## Wazuh — Open Source Security Monitoring

```
WAZUH ARCHITECTURE:

  ┌──────────┐    ┌──────────────┐    ┌──────────────┐
  │  Wazuh   │    │   Wazuh      │    │   Wazuh      │
  │  Agents  │───▶│   Manager    │───▶│   Dashboard   │
  │          │    │              │    │   (Kibana)    │
  │ • FIM    │    │ • Rules      │    │              │
  │ • Log    │    │ • Decoders   │    │ • Dashboards │
  │ • Vuln   │    │ • Active     │    │ • Compliance │
  │ • SCA    │    │   Response   │    │ • Alerts     │
  └──────────┘    └──────────────┘    └──────────────┘

  Capabilities:
  ├── File Integrity Monitoring (FIM)
  ├── Log analysis & SIEM
  ├── Vulnerability detection
  ├── Security Configuration Assessment (SCA)
  ├── Incident Response
  ├── Regulatory Compliance (PCI, HIPAA, GDPR)
  └── Cloud Security (AWS, Azure, GCP)
```

```xml
<!-- Wazuh custom rule — detect privilege escalation -->
<group name="privilege_escalation">
  <rule id="100100" level="12">
    <if_sid>5502</if_sid>
    <match>sudo</match>
    <user>^(?!admin|deploy)</user>
    <description>
      Unauthorized sudo usage by $(srcuser)
    </description>
    <group>authentication_failed,pci_dss_10.2.5</group>
  </rule>

  <rule id="100101" level="15">
    <if_sid>5303</if_sid>
    <match>passwd|shadow|sudoers</match>
    <description>
      Critical file modification detected: $(file)
    </description>
    <group>syscheck,pci_dss_11.5</group>
  </rule>
</group>
```

---

## Cloud-Native Monitoring

### AWS Security Monitoring

```bash
# Enable CloudTrail (all regions)
aws cloudtrail create-trail \
    --name security-trail \
    --s3-bucket-name security-logs-bucket \
    --is-multi-region-trail \
    --enable-log-file-validation \
    --include-global-service-events

# Enable GuardDuty (threat detection)
aws guardduty create-detector --enable

# CloudWatch Metric Filter — root account usage
aws logs put-metric-filter \
    --log-group-name CloudTrail/SecurityLogs \
    --filter-name RootAccountUsage \
    --filter-pattern '{ $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }' \
    --metric-transformations \
        metricName=RootAccountUsage,metricNamespace=SecurityMetrics,metricValue=1

# CloudWatch Alarm
aws cloudwatch put-metric-alarm \
    --alarm-name RootAccountUsageAlarm \
    --metric-name RootAccountUsage \
    --namespace SecurityMetrics \
    --statistic Sum \
    --period 300 \
    --threshold 1 \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --alarm-actions arn:aws:sns:us-east-1:123456:security-alerts
```

### Kubernetes Audit Logging

```yaml
# k8s-audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
  # Log all requests to secrets
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets"]

  # Log exec into pods (potential breakout)
  - level: RequestResponse
    resources:
      - group: ""
        resources: ["pods/exec", "pods/attach"]

  # Log RBAC changes
  - level: RequestResponse
    resources:
      - group: "rbac.authorization.k8s.io"
        resources: ["clusterroles", "clusterrolebindings", "roles", "rolebindings"]

  # Log namespace creation/deletion
  - level: Metadata
    resources:
      - group: ""
        resources: ["namespaces"]
    verbs: ["create", "delete"]

  # Don't log read-only requests to health endpoints
  - level: None
    resources:
      - group: ""
        resources: ["endpoints", "services"]
    verbs: ["get", "list", "watch"]
```

---

## Prometheus + Grafana Security Metrics

```yaml
# prometheus-security-rules.yml
groups:
  - name: security_alerts
    rules:
      # High rate of 4xx/5xx errors (potential attack)
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"4..|5.."}[5m])) 
          / sum(rate(http_requests_total[5m])) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High HTTP error rate detected"

      # Unusual login failures
      - alert: LoginBruteForce
        expr: |
          sum(rate(auth_login_failures_total[5m])) > 10
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Possible brute force attack: {{ $value }} failures/sec"

      # Container running as root
      - alert: ContainerRunningAsRoot
        expr: |
          container_processes_running_as_root > 0
        for: 0m
        labels:
          severity: warning
        annotations:
          summary: "Container {{ $labels.container }} running as root"

      # Certificate expiry
      - alert: CertificateExpiringSoon
        expr: |
          (x509_cert_not_after - time()) / 86400 < 30
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "Certificate expires in {{ $value }} days"
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Too many alerts (alert fatigue) | Thresholds too low; no prioritization | Tune thresholds; implement severity levels; suppress known false positives; deduplicate |
| Missing logs from some services | Not all services sending logs | Deploy log agents (Filebeat/Fluentd) to all nodes; verify log forwarding config |
| SIEM slow during searches | Too much unindexed data | Use index lifecycle management (ILM); archive old logs; optimize index patterns |
| Cloud audit logs not available | CloudTrail/Activity Log not enabled | Enable in all regions; check S3 bucket permissions; verify log delivery |
| K8s audit logs too verbose | Logging everything at RequestResponse level | Use audit policy tiers; log Metadata for reads, RequestResponse only for sensitive resources |

---

## Summary Table

| Tool | Type | Best For |
|------|------|----------|
| **ELK Stack** | Log aggregation + SIEM | Full-text search, log correlation, dashboards |
| **Wazuh** | Open-source SIEM + XDR | FIM, compliance, vulnerability detection, active response |
| **Prometheus + Grafana** | Metrics monitoring | Performance metrics, SLA tracking, alerting |
| **AWS GuardDuty** | Cloud threat detection | AWS-specific threat intelligence, anomaly detection |
| **Falco** | Runtime security | Container/K8s runtime syscall monitoring |
| **Datadog / Splunk** | Commercial SIEM | Enterprise monitoring, ML-based detection |

---

## Quick Revision Questions

1. **What is security monitoring and why is it critical for DevSecOps?**
   > Security monitoring is the continuous observation of systems, networks, and applications to detect threats, anomalies, and policy violations in real time. It's critical because it provides visibility into the security posture, enables early threat detection, supports incident response, and satisfies compliance requirements for logging and alerting.

2. **What are the components of the ELK Stack and their roles?**
   > Elasticsearch (search and analytics engine for storing and indexing logs), Logstash (data processing pipeline that ingests, parses, enriches, and transforms logs), Kibana (visualization layer for dashboards, queries, and alerts), and Beats (lightweight agents that ship data — Filebeat for logs, Auditbeat for audit events, Metricbeat for metrics).

3. **How does Wazuh differ from a basic SIEM?**
   > Wazuh combines SIEM capabilities with endpoint detection and response (XDR). Beyond log analysis, it provides file integrity monitoring, vulnerability detection, security configuration assessment, regulatory compliance mapping (PCI-DSS, HIPAA, GDPR), active response (automated remediation), and native cloud security monitoring for AWS/Azure/GCP.

4. **What Kubernetes events should be monitored for security?**
   > Secret access and modifications, pod exec/attach commands (potential breakout), RBAC changes (role/binding creation/deletion), namespace creation/deletion, privileged pod creation, failed authentication attempts, resource quota changes, and admission controller denials.

5. **How do you manage alert fatigue in security monitoring?**
   > Tune alert thresholds based on baseline behavior, implement severity-based prioritization (critical/high/medium/low), suppress known false positives with allowlists, deduplicate similar alerts, use time-based aggregation, route different severities to appropriate channels (pager for critical, ticket for medium), and regularly review alert effectiveness.

---

[← Previous: 9.6 Secrets Best Practices](../09-Secrets-Management/06-secrets-best-practices.md) | [Next: 10.2 Threat Detection →](02-threat-detection.md)

[Back to Table of Contents](../README.md)

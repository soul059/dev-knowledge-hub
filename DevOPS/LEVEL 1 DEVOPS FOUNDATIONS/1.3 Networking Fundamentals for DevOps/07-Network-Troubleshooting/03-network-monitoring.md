# Chapter 3: Network Monitoring

## Overview

Network monitoring is the practice of continuously observing network infrastructure to detect outages, performance degradation, and security anomalies. For DevOps teams, monitoring is not optional — it's the difference between proactively fixing issues and getting woken up by customer complaints at 3 AM.

---

## 3.1 What to Monitor

```
┌──────────────────────────────────────────────────────────────┐
│          NETWORK MONITORING LAYERS                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 1: INFRASTRUCTURE                                     │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ • Host up/down (ping, health checks)                   │  │
│  │ • Network interface errors/drops                       │  │
│  │ • Bandwidth utilization                                │  │
│  │ • Packet loss rate                                     │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Layer 2: CONNECTIVITY                                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ • DNS resolution time and failures                     │  │
│  │ • TCP connection success/failure rate                   │  │
│  │ • TLS handshake errors                                 │  │
│  │ • Latency between services                             │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Layer 3: APPLICATION                                        │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ • HTTP response codes (4xx, 5xx)                       │  │
│  │ • Request latency (p50, p95, p99)                      │  │
│  │ • Throughput (requests/second)                         │  │
│  │ • Error rate                                           │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Layer 4: SECURITY                                           │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ • Unusual traffic patterns (DDoS)                      │  │
│  │ • Failed connection attempts (port scans)              │  │
│  │ • VPC Flow Log analysis                                │  │
│  │ • Certificate expiry                                   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 The Four Golden Signals

```
┌──────────────────────────────────────────────────────────────┐
│        GOOGLE SRE — FOUR GOLDEN SIGNALS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. LATENCY                                                  │
│     Time to serve a request                                  │
│     ┌──────────────────────────────────────┐                 │
│     │ Metric: http_request_duration_seconds │                 │
│     │ Alert:  p99 latency > 500ms          │                 │
│     │ Track:  Successful vs failed latency │                 │
│     └──────────────────────────────────────┘                 │
│                                                              │
│  2. TRAFFIC                                                  │
│     How much demand on the system                            │
│     ┌──────────────────────────────────────┐                 │
│     │ Metric: http_requests_total           │                 │
│     │ Alert:  Sudden drop (outage?)        │                 │
│     │         Sudden spike (DDoS?)         │                 │
│     └──────────────────────────────────────┘                 │
│                                                              │
│  3. ERRORS                                                   │
│     Rate of failed requests                                  │
│     ┌──────────────────────────────────────┐                 │
│     │ Metric: http_requests_total{code=~"5.."} │             │
│     │ Alert:  Error rate > 1%              │                 │
│     │ Track:  5xx vs 4xx separately        │                 │
│     └──────────────────────────────────────┘                 │
│                                                              │
│  4. SATURATION                                               │
│     How "full" the system is                                 │
│     ┌──────────────────────────────────────┐                 │
│     │ Metric: CPU, memory, connections,     │                 │
│     │         disk I/O, bandwidth           │                 │
│     │ Alert:  >80% utilization             │                 │
│     └──────────────────────────────────────┘                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.3 Prometheus + Grafana Stack

```
┌──────────────────────────────────────────────────────────────┐
│        MONITORING ARCHITECTURE                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐     │
│  │  Application │   │ Node         │   │ Blackbox     │     │
│  │  /metrics    │   │ Exporter     │   │ Exporter     │     │
│  │  (app code)  │   │ (host stats) │   │ (probe URLs) │     │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘     │
│         │                  │                  │              │
│         └──────────┬───────┘──────────────────┘              │
│                    │  scrape /metrics every 15s              │
│                    ▼                                         │
│         ┌──────────────────┐                                 │
│         │   Prometheus     │ ← Time-series database          │
│         │   (scrape+store) │ ← PromQL query language         │
│         └────────┬─────────┘                                 │
│                  │                                           │
│         ┌────────▼─────────┐                                 │
│         │   Alertmanager   │ ← Route alerts to Slack/PD      │
│         └────────┬─────────┘                                 │
│                  │                                           │
│         ┌────────▼─────────┐                                 │
│         │   Grafana        │ ← Dashboards and visualization  │
│         └──────────────────┘                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Blackbox Exporter — Probe Endpoints

```yaml
# blackbox.yml — probe configuration
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      valid_status_codes: [200]
      method: GET
      follow_redirects: true
      preferred_ip_protocol: "ip4"
      tls_config:
        insecure_skip_verify: false

  tcp_connect:
    prober: tcp
    timeout: 5s

  dns_lookup:
    prober: dns
    timeout: 5s
    dns:
      query_name: "example.com"
      query_type: "A"

  icmp_ping:
    prober: icmp
    timeout: 5s
```

```yaml
# prometheus.yml — scrape blackbox exporter
scrape_configs:
  - job_name: 'blackbox-http'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://api.example.com/health
          - https://www.example.com
          - https://admin.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  - job_name: 'blackbox-tcp'
    metrics_path: /probe
    params:
      module: [tcp_connect]
    static_configs:
      - targets:
          - db.internal:5432
          - redis.internal:6379
          - kafka.internal:9092
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

### Network Alerting Rules

```yaml
# prometheus-rules.yml
groups:
  - name: network-alerts
    rules:
      # Endpoint down
      - alert: EndpointDown
        expr: probe_success == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Endpoint {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been unreachable for 2 minutes"

      # High latency
      - alert: HighLatency
        expr: probe_duration_seconds > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency on {{ $labels.instance }}"
          description: "Response time is {{ $value }}s (>1s threshold)"

      # SSL certificate expiring soon
      - alert: SSLCertExpiringSoon
        expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 14
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "SSL cert for {{ $labels.instance }} expires in {{ $value | humanizeDuration }}"

      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{code=~"5.."}[5m])) by (service)
          /
          sum(rate(http_requests_total[5m])) by (service)
          > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Error rate >1% for {{ $labels.service }}"

      # DNS resolution failure
      - alert: DNSResolutionFailed
        expr: probe_dns_lookup_time_seconds == 0 and probe_success == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "DNS resolution failed for {{ $labels.instance }}"
```

---

## 3.4 AWS CloudWatch for Network Monitoring

```
┌──────────────────────────────────────────────────────────────┐
│        AWS CLOUDWATCH NETWORK METRICS                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ALB Metrics:                                                │
│  ├─ ActiveConnectionCount                                    │
│  ├─ RequestCount                                             │
│  ├─ TargetResponseTime (latency)                             │
│  ├─ HTTPCode_Target_5XX_Count                                │
│  ├─ HTTPCode_ELB_5XX_Count                                   │
│  ├─ HealthyHostCount                                         │
│  └─ UnHealthyHostCount                                       │
│                                                              │
│  NAT Gateway Metrics:                                        │
│  ├─ BytesOutToDestination                                    │
│  ├─ PacketsDropCount  ← Important! Capacity limit            │
│  ├─ ErrorPortAllocation                                      │
│  └─ ActiveConnectionCount                                    │
│                                                              │
│  VPN Metrics:                                                │
│  ├─ TunnelState (0=down, 1=up)                               │
│  └─ TunnelDataIn / TunnelDataOut                             │
│                                                              │
│  VPC Flow Logs:                                              │
│  └─ Traffic accepted/rejected by SGs and NACLs               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### CloudWatch Alarms (Terraform)

```hcl
# Alert on unhealthy targets
resource "aws_cloudwatch_metric_alarm" "unhealthy_hosts" {
  alarm_name          = "alb-unhealthy-hosts"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "UnHealthyHostCount"
  namespace           = "AWS/ApplicationELB"
  period              = 60
  statistic           = "Maximum"
  threshold           = 0
  alarm_description   = "ALB has unhealthy targets"

  dimensions = {
    LoadBalancer = aws_lb.main.arn_suffix
    TargetGroup  = aws_lb_target_group.app.arn_suffix
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# Alert on high 5xx rate
resource "aws_cloudwatch_metric_alarm" "high_5xx" {
  alarm_name          = "alb-high-5xx-rate"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 3
  threshold           = 10

  metric_query {
    id          = "error_rate"
    expression  = "errors / requests * 100"
    label       = "Error Rate %"
    return_data = true
  }

  metric_query {
    id = "errors"
    metric {
      metric_name = "HTTPCode_Target_5XX_Count"
      namespace   = "AWS/ApplicationELB"
      period      = 60
      stat        = "Sum"
      dimensions = {
        LoadBalancer = aws_lb.main.arn_suffix
      }
    }
  }

  metric_query {
    id = "requests"
    metric {
      metric_name = "RequestCount"
      namespace   = "AWS/ApplicationELB"
      period      = 60
      stat        = "Sum"
      dimensions = {
        LoadBalancer = aws_lb.main.arn_suffix
      }
    }
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}

# Alert on NAT Gateway packet drops
resource "aws_cloudwatch_metric_alarm" "nat_drops" {
  alarm_name          = "nat-gateway-packet-drops"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 2
  metric_name         = "PacketsDropCount"
  namespace           = "AWS/NATGateway"
  period              = 300
  statistic           = "Sum"
  threshold           = 100
  alarm_description   = "NAT Gateway dropping packets - may need additional NATs"

  dimensions = {
    NatGatewayId = aws_nat_gateway.main.id
  }

  alarm_actions = [aws_sns_topic.alerts.arn]
}
```

---

## 3.5 VPC Flow Logs Analysis

```bash
# Enable VPC Flow Logs
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /vpc/flow-logs

# Flow log format:
# version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
# 2 123456789 eni-abc123 10.0.1.50 10.0.2.30 443 52341 6 25 12500 1625000000 1625000060 ACCEPT OK
# 2 123456789 eni-abc123 203.0.113.5 10.0.1.50 0 0 1 100 8400 1625000000 1625000060 REJECT OK

# Query flow logs with CloudWatch Insights
# Find rejected traffic (firewall blocks)
aws logs start-query \
  --log-group-name /vpc/flow-logs \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string '
    fields @timestamp, srcAddr, dstAddr, dstPort, action
    | filter action = "REJECT"
    | stats count(*) as rejections by srcAddr, dstAddr, dstPort
    | sort rejections desc
    | limit 20
  '

# Find top talkers
# fields srcAddr, dstAddr | stats sum(bytes) as totalBytes by srcAddr, dstAddr | sort totalBytes desc | limit 10
```

### Terraform VPC Flow Logs

```hcl
resource "aws_flow_log" "vpc" {
  vpc_id                   = aws_vpc.main.id
  traffic_type             = "ALL"
  log_destination_type     = "cloud-watch-logs"
  log_destination          = aws_cloudwatch_log_group.flow_logs.arn
  iam_role_arn             = aws_iam_role.flow_logs.arn
  max_aggregation_interval = 60  # 1 minute (vs default 10 min)

  tags = { Name = "vpc-flow-logs" }
}

resource "aws_cloudwatch_log_group" "flow_logs" {
  name              = "/vpc/flow-logs"
  retention_in_days = 30
}
```

---

## 3.6 Monitoring Tools Comparison

| Tool | Type | Best For | Cost |
|------|------|----------|------|
| **Prometheus + Grafana** | Self-hosted | Kubernetes, custom metrics | Free (OSS) |
| **CloudWatch** | AWS managed | AWS resources, flow logs | $$ |
| **Datadog** | SaaS | Full-stack, APM, network maps | $$$ |
| **New Relic** | SaaS | APM, browser, infra | $$ |
| **Zabbix** | Self-hosted | Traditional infra, SNMP | Free (OSS) |
| **Nagios** | Self-hosted | Legacy monitoring, plugins | Free (OSS) |
| **VictoriaMetrics** | Self-hosted | Long-term Prometheus storage | Free (OSS) |
| **Grafana Cloud** | SaaS | Managed Prometheus + Grafana | Freemium |

---

## 3.7 Troubleshooting Tips

| Problem | Monitor | Alert Threshold |
|---------|---------|----------------|
| Service down | Blackbox probe (probe_success) | == 0 for 2 min |
| High latency | HTTP duration p99 | > 500ms for 5 min |
| Cert expiry | probe_ssl_earliest_cert_expiry | < 14 days |
| NAT capacity | PacketsDropCount | > 0 |
| VPN tunnel down | TunnelState | == 0 for 1 min |
| 5xx spike | HTTPCode_Target_5XX | > 1% error rate |
| Flow log rejections | VPC Flow Logs | New REJECT patterns |
| Connection exhaustion | ActiveConnectionCount | > 80% of limit |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Four Golden Signals | Latency, Traffic, Errors, Saturation |
| Prometheus | Pull-based metrics, PromQL, time-series DB |
| Grafana | Visualization dashboards for any data source |
| Blackbox Exporter | Probe endpoints externally (HTTP, TCP, DNS, ICMP) |
| CloudWatch | AWS native monitoring, alarms, log insights |
| VPC Flow Logs | Record all network traffic accept/reject decisions |
| Alerting | Route critical alerts to PagerDuty/Slack/email |
| Certificate Monitoring | Alert before SSL certs expire |

---

## Quick Revision Questions

1. **What are the Four Golden Signals and why are they important?**
2. **How does Prometheus collect metrics from applications?**
3. **Write a Prometheus alert rule for when an endpoint is down for more than 2 minutes.**
4. **What information do VPC Flow Logs capture, and how would you find rejected traffic?**
5. **What CloudWatch metric would you monitor to detect NAT Gateway capacity issues?**
6. **Compare self-hosted (Prometheus) vs managed (Datadog) monitoring — when would you choose each?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Packet Capture](02-packet-capture.md) | [README](../README.md) | [Common Issues & Solutions →](04-common-issues-solutions.md) |

# Chapter 4: Filebeat & Agents

[← Previous: Kibana](03-kibana.md) | [Back to README](../README.md) | [Next: ELK Stack Deployment →](05-elk-stack-deployment.md)

---

## Overview

Filebeat is a lightweight log shipper from Elastic that reads log files and forwards them to Logstash or Elasticsearch. It is the recommended collection agent in the Elastic Stack.

---

## 4.1 Beats Family

```
┌──────────────────────────────────────────────────────────┐
│                   ELASTIC BEATS                          │
│                                                          │
│  ┌────────────┐  Lightweight, single-purpose agents     │
│  │ Filebeat   │  Log files → Elasticsearch/Logstash     │
│  ├────────────┤                                          │
│  │ Metricbeat │  System/service metrics → ES            │
│  ├────────────┤                                          │
│  │ Packetbeat │  Network traffic analysis → ES          │
│  ├────────────┤                                          │
│  │ Heartbeat  │  Uptime/availability monitoring → ES    │
│  ├────────────┤                                          │
│  │ Auditbeat  │  Audit log data → ES                    │
│  ├────────────┤                                          │
│  │ Winlogbeat │  Windows event logs → ES                │
│  └────────────┘                                          │
│                                                          │
│  All Beats share:                                        │
│  • Low memory (~30MB)                                    │
│  • Back-pressure handling                                │
│  • At-least-once delivery                                │
│  • Modules for common formats                            │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 Filebeat Architecture

```
┌──────────────────────────────────────────────────────────┐
│                 FILEBEAT INTERNALS                        │
│                                                          │
│  ┌──────────────┐    ┌───────────────┐                  │
│  │ Input (Harv.) │───▶│    Spooler    │                  │
│  │ /var/log/*.log│    │  (in-memory   │                  │
│  │               │    │   queue)      │                  │
│  │ Tracks offset │    └───────┬───────┘                  │
│  │ per file in   │            │                          │
│  │ registry      │            ▼                          │
│  └──────┬───────┘    ┌───────────────┐                  │
│         │            │   Output      │                  │
│         │            │ (ES/Logstash/ │                  │
│         │            │  Kafka/Redis) │                  │
│         │            └───────────────┘                  │
│         │                                                │
│  ┌──────▼───────┐    Registry File:                     │
│  │ data/registry│    Tracks position in every file       │
│  │              │    so Filebeat can resume after        │
│  │ file: offset │    restart (at-least-once delivery)   │
│  └──────────────┘                                        │
└──────────────────────────────────────────────────────────┘
```

---

## 4.3 Filebeat Configuration

```yaml
# filebeat.yml

# ── Inputs ──
filebeat.inputs:
  - type: filestream
    id: app-logs
    paths:
      - /var/log/app/*.log
    parsers:
      - ndjson:
          keys_under_root: true
          add_error_key: true
    fields:
      environment: production
      service: api-server
    fields_under_root: true

  - type: container
    paths:
      - /var/log/containers/*.log
    processors:
      - add_kubernetes_metadata:
          host: ${NODE_NAME}
          matchers:
            - logs_path:
                logs_path: "/var/log/containers/"

# ── Modules ──
filebeat.modules:
  - module: nginx
    access:
      enabled: true
      var.paths: ["/var/log/nginx/access.log"]
    error:
      enabled: true

  - module: system
    syslog:
      enabled: true

# ── Processors ──
processors:
  - drop_event:
      when:
        contains:
          message: "health_check"
  - add_host_metadata: ~
  - add_cloud_metadata: ~

# ── Output ──
output.elasticsearch:
  hosts: ["http://elasticsearch:9200"]
  index: "filebeat-%{+yyyy.MM.dd}"
  username: "elastic"
  password: "${ES_PASSWORD}"

# OR output to Logstash
# output.logstash:
#   hosts: ["logstash:5044"]
#   ssl.certificate_authorities: ["/etc/pki/root/ca.crt"]

# ── Logging ──
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
```

---

## 4.4 Filebeat on Kubernetes

```yaml
# filebeat-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
  namespace: elastic
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      serviceAccountName: filebeat
      containers:
        - name: filebeat
          image: docker.elastic.co/beats/filebeat:8.11.0
          args: ["-c", "/etc/filebeat.yml", "-e"]
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: config
              mountPath: /etc/filebeat.yml
              subPath: filebeat.yml
            - name: varlog
              mountPath: /var/log
              readOnly: true
            - name: data
              mountPath: /usr/share/filebeat/data
          resources:
            limits:
              memory: 200Mi
              cpu: 200m
      volumes:
        - name: config
          configMap:
            name: filebeat-config
        - name: varlog
          hostPath:
            path: /var/log
        - name: data
          hostPath:
            path: /var/lib/filebeat-data
```

---

## 4.5 Filebeat vs Fluent Bit vs Promtail

| Feature | Filebeat | Fluent Bit | Promtail |
|---------|----------|------------|----------|
| Ecosystem | Elastic Stack | CNCF | Grafana/Loki |
| Memory | ~30 MB | ~15 MB | ~25 MB |
| Best output | Elasticsearch | Any (Loki, ES, S3) | Loki only |
| Modules | 50+ pre-built | Custom configs | Basic |
| K8s metadata | Yes (processor) | Yes (filter) | Yes (built-in) |
| Parsing | Basic, modules | Full pipeline | LogQL-based |
| Backpressure | Registry + disk queue | Memory/file buffer | WAL |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Missing logs | Filebeat not tracking file | Check registry, verify file path wildcards |
| Duplicate logs | Pod restart resets registry | Use persistent volume for data directory |
| High CPU | Too many harvesters | Limit `harvester_limit` per input |
| Connection refused | Output unreachable | Check ES/Logstash connectivity |
| "Too many open files" | Monitoring many log files | Increase `ulimit -n`, set `close_inactive` |

---

## Quick Revision Questions

1. **What are the Elastic Beats and what does each one collect?**
2. **How does Filebeat ensure at-least-once delivery?**
3. **What is the registry file and why is it important?**
4. **How do you deploy Filebeat on Kubernetes?**
5. **Compare Filebeat, Fluent Bit, and Promtail — when would you use each?**
6. **What are Filebeat modules and name two common ones?**

---

[← Previous: Kibana](03-kibana.md) | [Back to README](../README.md) | [Next: ELK Stack Deployment →](05-elk-stack-deployment.md)

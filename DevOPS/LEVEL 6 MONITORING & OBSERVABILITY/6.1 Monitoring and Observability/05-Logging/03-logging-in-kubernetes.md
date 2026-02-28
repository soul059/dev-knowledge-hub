# Chapter 3: Logging in Kubernetes

[← Previous: Structured Logging](02-structured-logging.md) | [Back to README](../README.md) | [Next: Loki →](04-loki.md)

---

## Overview

Kubernetes logging follows a layered model: container logs, node-level collection, and cluster-level aggregation. Understanding this architecture is essential for operating production clusters.

---

## 3.1 Kubernetes Logging Architecture

```
┌──────────────────────────────────────────────────────────┐
│              KUBERNETES LOGGING LAYERS                    │
│                                                          │
│  Layer 3: Cluster-Level Aggregation                      │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Loki / Elasticsearch / CloudWatch Logs           │   │
│  │  (centralized log storage & query)                │   │
│  └──────────────────────────────────────────────────┘   │
│                         ▲                                │
│                         │ ships logs                     │
│  Layer 2: Node-Level Collection                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  DaemonSet: Fluent Bit / Fluentd / Filebeat       │   │
│  │  (runs on every node, reads log files)            │   │
│  └──────────────────────────────────────────────────┘   │
│                         ▲                                │
│                         │ reads from                     │
│  Layer 1: Container Logs                                 │
│  ┌──────────────────────────────────────────────────┐   │
│  │  /var/log/containers/*.log                        │   │
│  │  /var/log/pods/<namespace>_<pod>_<uid>/           │   │
│  │  (container runtime captures stdout/stderr)       │   │
│  └──────────────────────────────────────────────────┘   │
│                         ▲                                │
│                         │ stdout/stderr                  │
│  Layer 0: Application                                    │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Application writes logs to stdout/stderr         │   │
│  └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Container Log Format

```
# Raw container log line (CRI format):
2024-01-15T10:23:45.123456789Z stdout F {"level":"info","msg":"Request processed","status":200}

# Fields:
# ├── Timestamp (RFC 3339 nanoseconds)
# ├── Stream (stdout or stderr)
# ├── Flag (F=full line, P=partial)
# └── Log content (your application output)
```

### Log Locations on Node

```bash
# Container logs (symlinks)
/var/log/containers/<pod>_<namespace>_<container>-<id>.log

# Pod logs (actual files)
/var/log/pods/<namespace>_<pod>_<uid>/<container>/0.log

# Docker runtime logs
/var/lib/docker/containers/<id>/<id>-json.log

# View with kubectl
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container>    # specific container
kubectl logs <pod-name> --previous        # previous instance
kubectl logs <pod-name> -f                # follow/stream
kubectl logs <pod-name> --since=1h        # last hour
kubectl logs <pod-name> --tail=100        # last 100 lines
kubectl logs -l app=api-server            # by label selector
```

---

## 3.3 Log Collection Patterns

### Pattern 1: DaemonSet (Recommended)

```
  ┌──── Node 1 ────────────────┐
  │  ┌─────┐ ┌─────┐ ┌─────┐  │
  │  │Pod A│ │Pod B│ │Pod C│  │
  │  └──┬──┘ └──┬──┘ └──┬──┘  │
  │     │stdout  │       │      │
  │     ▼        ▼       ▼      │
  │  /var/log/containers/*.log  │
  │           ▲                  │
  │  ┌────────┴────────┐       │
  │  │ Fluent Bit       │       │ ← DaemonSet (one per node)
  │  │ (DaemonSet pod)  │       │
  │  └────────┬────────┘       │
  └───────────┼────────────────┘
              │
              ▼
        Loki / Elasticsearch
```

### Pattern 2: Sidecar

```
  ┌──── Pod ──────────────────┐
  │  ┌────────────┐           │
  │  │ App Container│          │
  │  │ writes to   │          │
  │  │ /var/log/   │          │
  │  └──────┬─────┘          │
  │         │ shared volume   │
  │  ┌──────▼─────┐          │
  │  │ Sidecar     │          │
  │  │ (Fluent Bit)│──▶ Loki  │
  │  └────────────┘          │
  └───────────────────────────┘

Use case: When app can't log to stdout
         or needs special parsing
```

---

## 3.4 Fluent Bit DaemonSet Configuration

```yaml
# fluent-bit-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      serviceAccountName: fluent-bit
      containers:
        - name: fluent-bit
          image: fluent/fluent-bit:2.2
          volumeMounts:
            - name: varlog
              mountPath: /var/log
              readOnly: true
            - name: config
              mountPath: /fluent-bit/etc/
          resources:
            limits:
              memory: 200Mi
              cpu: 200m
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
        - name: config
          configMap:
            name: fluent-bit-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     info
        Parsers_File  parsers.conf

    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        Parser            cri
        Tag               kube.*
        Mem_Buf_Limit     5MB
        Skip_Long_Lines   On
        Refresh_Interval  10

    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
        Merge_Log           On
        K8S-Logging.Parser  On

    [OUTPUT]
        Name          loki
        Match         *
        Host          loki.logging.svc
        Port          3100
        Labels        job=fluent-bit, namespace=$kubernetes['namespace_name']
        Auto_Kubernetes_Labels On
```

---

## 3.5 Log Rotation & Retention on Nodes

```yaml
# kubelet log rotation settings (kubelet flags)
--container-log-max-files=5      # Keep 5 rotated files
--container-log-max-size=10Mi    # Rotate at 10MB

# Without rotation, logs can fill up the node disk!
# Monitor with:
#   node_filesystem_avail_bytes{mountpoint="/var/log"}
```

---

## Summary Table

| Pattern | Pros | Cons | Use When |
|---------|------|------|----------|
| DaemonSet | Low overhead, simple | Shared config for all pods | Default choice |
| Sidecar | Per-pod customization | More resources per pod | Complex parsing needs |
| Direct push | No collector needed | App complexity, coupling | Serverless/FaaS |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `kubectl logs` empty | App logging to file not stdout | Change app to log to stdout |
| Logs truncated | Line too long or partial write | Check `Skip_Long_Lines` setting |
| Node disk full | No log rotation configured | Set kubelet log rotation flags |
| Missing pod metadata | Kubernetes filter not configured | Add kubernetes filter in Fluent Bit |
| Collector OOM | Too many log files or high volume | Increase memory limit, add filtering |

---

## Quick Revision Questions

1. **What are the three layers of Kubernetes logging architecture?**
2. **Why should containerized apps log to stdout instead of files?**
3. **What is the difference between DaemonSet and sidecar log collection patterns?**
4. **How does Fluent Bit enrich logs with Kubernetes metadata?**
5. **How do you configure log rotation in Kubernetes?**
6. **What kubectl commands would you use to debug a pod's logs?**

---

[← Previous: Structured Logging](02-structured-logging.md) | [Back to README](../README.md) | [Next: Loki →](04-loki.md)

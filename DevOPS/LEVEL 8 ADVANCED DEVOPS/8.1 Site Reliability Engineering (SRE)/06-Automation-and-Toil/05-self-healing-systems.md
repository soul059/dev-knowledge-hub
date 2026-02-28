# Chapter 6.5: Self-Healing Systems

[← Previous: Reducing Operational Burden](04-reducing-operational-burden.md) | [Back to README](../README.md) | [Next: Automation ROI →](06-automation-roi.md)

---

## Overview

Self-healing systems represent the highest level of automation maturity — Level 5 (autonomous). These systems automatically detect failures, diagnose root causes, and apply remediation without human intervention. Built on control loops, health checks, and predefined remediation playbooks, self-healing reduces incident response time from minutes to seconds.

---

## 1. Self-Healing Architecture

```
  ┌──────────────────────────────────────────────────────────┐
  │  SELF-HEALING CONTROL LOOP                               │
  │                                                          │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐            │
  │  │ OBSERVE  │──▶│ ANALYZE  │──▶│  ACT     │            │
  │  │          │   │          │   │          │             │
  │  │ • Health │   │ • Match  │   │ • Auto-  │            │
  │  │   checks │   │   against│   │   restart│            │
  │  │ • Metrics│   │   known  │   │ • Scale  │            │
  │  │ • Logs   │   │   failure│   │ • Failovr│            │
  │  │ • Traces │   │   patterns│  │ • Drain  │            │
  │  └──────────┘   └──────────┘   └────┬─────┘            │
  │       ▲                              │                   │
  │       │         ┌──────────┐         │                   │
  │       └─────────│ VERIFY   │◀────────┘                   │
  │                 │          │                              │
  │                 │ • Check  │  If fixed → log & close     │
  │                 │   health │  If not → escalate to human │
  │                 │ • Confirm│                              │
  │                 │   SLOs OK│                              │
  │                 └──────────┘                              │
  │                                                          │
  │  KEY: The loop runs continuously, typically every        │
  │  10-60 seconds, without human involvement                │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Kubernetes Self-Healing (Built-in)

```
  ┌──────────────────────────────────────────────────────────┐
  │  KUBERNETES NATIVE SELF-HEALING                          │
  │                                                          │
  │  ┌──────────────────┐  Pod crashes → kubelet restarts    │
  │  │ Restart Policy   │  it automatically (CrashLoopBackOff│
  │  │ (always/onfailure)│ with exponential backoff)         │
  │  └──────────────────┘                                    │
  │                                                          │
  │  ┌──────────────────┐  Pod fails liveness probe →        │
  │  │ Liveness Probes  │  kubelet kills and restarts it     │
  │  └──────────────────┘                                    │
  │                                                          │
  │  ┌──────────────────┐  Pod fails readiness probe →       │
  │  │ Readiness Probes │  removed from service endpoints   │
  │  └──────────────────┘  (no traffic until healthy)        │
  │                                                          │
  │  ┌──────────────────┐  Pod fails startup probe →         │
  │  │ Startup Probes   │  waits longer before applying      │
  │  └──────────────────┘  liveness checks (slow startup)    │
  │                                                          │
  │  ┌──────────────────┐  Node dies → pods rescheduled      │
  │  │ ReplicaSet       │  to healthy nodes automatically    │
  │  └──────────────────┘                                    │
  │                                                          │
  │  ┌──────────────────┐  CPU/memory pressure → auto-scale  │
  │  │ HPA / VPA        │  pods up or resize resource limits │
  │  └──────────────────┘                                    │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# Comprehensive health check configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: api
          image: api-service:v2.1
          ports:
            - containerPort: 8080
          
          # Slow-starting apps: give up to 5 min to start
          startupProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 60    # 60 × 5s = 300s max
          
          # Is the process hung? Kill and restart if so
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            periodSeconds: 10
            failureThreshold: 3     # 3 failures → restart
            timeoutSeconds: 5
          
          # Can it serve traffic? Remove from LB if not
          readinessProbe:
            httpGet:
              path: /ready
              port: 8080
            periodSeconds: 5
            failureThreshold: 2     # 2 failures → no traffic
            successThreshold: 2     # 2 successes → traffic OK
            timeoutSeconds: 3
          
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 512Mi
      
      # Evict pods from unhealthy nodes
      tolerations:
        - key: "node.kubernetes.io/not-ready"
          operator: "Exists"
          effect: "NoExecute"
          tolerationSeconds: 30     # Wait 30s then evict
```

---

## 3. Custom Self-Healing with Kubernetes Operators

```
  ┌──────────────────────────────────────────────────────────┐
  │  KUBERNETES OPERATOR PATTERN                             │
  │                                                          │
  │  Custom Resource (CR)        Operator Controller         │
  │  ┌──────────────────┐       ┌──────────────────┐        │
  │  │ kind: Database   │       │ Watch for CR     │        │
  │  │ spec:            │       │ changes          │        │
  │  │   replicas: 3    │ ────▶ │       │          │        │
  │  │   version: 14    │       │       ▼          │        │
  │  │   backup: daily  │       │ Compare desired  │        │
  │  └──────────────────┘       │ vs actual state  │        │
  │                             │       │          │        │
  │  Desired State              │       ▼          │        │
  │  (what you want)            │ Reconcile:       │        │
  │                             │ • Create replicas│        │
  │                             │ • Run migrations │        │
  │                             │ • Schedule backups│        │
  │                             │ • Handle failover│        │
  │                             │       │          │        │
  │                             │       ▼          │        │
  │                             │ Update status    │        │
  │                             └──────────────────┘        │
  │                                                          │
  │  Examples of self-healing operators:                     │
  │  ├─ PostgreSQL Operator (Zalando) → auto-failover       │
  │  ├─ MySQL Operator → replica promotion                  │
  │  ├─ Prometheus Operator → config reload                 │
  │  └─ Elasticsearch Operator → rolling restarts            │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. Auto-Remediation Pattern

```
  ┌──────────────────────────────────────────────────────────┐
  │  EVENT → DECISION → ACTION FRAMEWORK                     │
  │                                                          │
  │  Alert fires                                             │
  │      │                                                   │
  │      ▼                                                   │
  │  Is this a KNOWN failure pattern?                        │
  │  ├─ YES                                                  │
  │  │   Has remediation been validated?                     │
  │  │   ├─ YES                                              │
  │  │   │   Has it been attempted < 3 times in last hour?  │
  │  │   │   ├─ YES ──▶ EXECUTE auto-remediation            │
  │  │   │   │          └─ Log action + verify outcome       │
  │  │   │   └─ NO  ──▶ ESCALATE (remediation loop)         │
  │  │   └─ NO  ──▶ ESCALATE (untested remediation)         │
  │  └─ NO  ──▶ ESCALATE to on-call engineer                │
  │                                                          │
  │  Common auto-remediations:                               │
  │  ├─ High memory → Restart pod (with draining first)     │
  │  ├─ Disk full → Rotate/compress logs, alert if < 10%    │
  │  ├─ Connection pool exhausted → Kill idle connections    │
  │  ├─ Certificate expiring → Renew via cert-manager       │
  │  ├─ Node unhealthy → Cordon, drain, replace             │
  │  └─ Traffic spike → Scale up replicas                    │
  └──────────────────────────────────────────────────────────┘
```

```yaml
# Auto-remediation rules (using a tool like Rundeck/PagerDuty)
remediation_rules:
  - name: "disk-space-low"
    trigger:
      alert: "DiskSpaceLow"
      condition: "disk_usage > 85%"
    actions:
      - step: 1
        action: "cleanup_temp_files"
        command: "find /tmp -mtime +7 -delete"
      - step: 2
        action: "rotate_logs"
        command: "logrotate -f /etc/logrotate.conf"
      - step: 3
        action: "verify"
        command: "df -h / | awk 'NR==2{print $5}'"
        success_condition: "usage < 80%"
    escalate_if_failed: true
    max_attempts: 2
    cooldown: "30m"

  - name: "pod-crashloop"
    trigger:
      alert: "PodCrashLoopBackOff"
      condition: "restart_count > 5 in 10m"
    actions:
      - step: 1
        action: "collect_diagnostics"
        command: "kubectl logs $POD --previous > /tmp/crash.log"
      - step: 2
        action: "check_for_oom"
        command: "kubectl describe pod $POD | grep OOMKilled"
        on_match: "increase_memory_limit"
      - step: 3
        action: "rollback_if_recent_deploy"
        condition: "last_deploy < 1h ago"
        command: "kubectl rollout undo deployment/$DEPLOY"
    escalate_if_failed: true
    max_attempts: 1
```

---

## 5. Circuit Breaker Self-Healing

```
  ┌──────────────────────────────────────────────────────────┐
  │  CIRCUIT BREAKER STATE MACHINE                           │
  │                                                          │
  │  ┌────────┐    failure threshold    ┌────────┐           │
  │  │ CLOSED │ ────── exceeded ──────▶│  OPEN  │           │
  │  │(normal)│                         │(reject │           │
  │  │        │                         │ all    │           │
  │  └───┬────┘                         │requests│           │
  │      │                              └───┬────┘           │
  │      │                                  │                │
  │      │    success    ┌──────────┐    timeout              │
  │      └───────────────│HALF-OPEN │◀───────┘               │
  │     (reset counter) │(test with│                         │
  │                      │ limited  │                         │
  │                      │ traffic) │                         │
  │                      └──────────┘                         │
  │                                                          │
  │  Self-healing aspect:                                    │
  │  ├─ System automatically stops calling failing service   │
  │  ├─ Prevents cascade failures                            │
  │  ├─ Periodically probes to detect recovery               │
  │  └─ Automatically resumes when service recovers          │
  │                                                          │
  │  No human needed — the system heals itself!              │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Auto-Scaling as Self-Healing

```yaml
# KEDA — Event-driven auto-scaling for Kubernetes
# Scales based on external metrics (queue depth, etc.)
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: order-processor
spec:
  scaleTargetRef:
    name: order-processor
  minReplicaCount: 2
  maxReplicaCount: 50
  cooldownPeriod: 60
  triggers:
    # Scale based on Kafka consumer lag
    - type: kafka
      metadata:
        bootstrapServers: kafka:9092
        consumerGroup: order-processors
        topic: orders
        lagThreshold: "100"     # Scale up when lag > 100
    
    # Scale based on CPU
    - type: cpu
      metadata:
        type: Utilization
        value: "60"
    
    # Scale to zero when idle (cost saving)
  advanced:
    horizontalPodAutoscalerConfig:
      behavior:
        scaleUp:
          stabilizationWindowSeconds: 30
          policies:
            - type: Percent
              value: 100          # Can double pod count
              periodSeconds: 30
        scaleDown:
          stabilizationWindowSeconds: 300
          policies:
            - type: Percent
              value: 25           # Slow scale-down
              periodSeconds: 60
```

---

## 7. Real-World Scenario

### [SCENARIO] Building Self-Healing for a Payment Service

```
  BEFORE: Payment service required manual intervention
  ├─ 3-5 incidents per week needing human response
  ├─ MTTR for common issues: 25 minutes (paging + diagnosis)
  ├─ Most incidents: same 4 failure modes (80%)
  
  Self-healing implementation:
  ┌──────────────────────────────────────────────────────────┐
  │  FAILURE MODE        │ AUTO-REMEDIATION         │ MTTR  │
  ├──────────────────────┼────────────────────────── ┼───────┤
  │ Memory leak          │ Liveness probe detects    │ 30s   │
  │ (OOM after 72h)      │ → auto-restart (graceful) │       │
  │                      │ + alert if > 3 in 1h      │       │
  │                      │                           │       │
  │ DB connection pool   │ Detect via metric →        │ 10s   │
  │ exhaustion           │ kill idle connections      │       │
  │                      │ (PgBouncer auto-manages)   │       │
  │                      │                           │       │
  │ Downstream timeout   │ Circuit breaker opens →    │ 0s    │
  │ (bank API slow)      │ return cached/queued       │       │
  │                      │ response, retry later      │       │
  │                      │                           │       │
  │ Traffic spike        │ HPA scales 2→12 pods in   │ 45s   │
  │ (flash sale)         │ 45 seconds based on RPS   │       │
  └──────────────────────┴────────────────────────── ┴───────┘
  
  AFTER (measured over 3 months):
  ├─ Incidents requiring human: 3-5/week → 0.5/week
  ├─ Auto-healed incidents: 12-15/week (were manual before)
  ├─ MTTR for auto-healed: 30 seconds average
  ├─ On-call pages reduced by 85%
  └─ Availability improved: 99.9% → 99.97%
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "Auto-remediation keeps firing but doesn't fix the issue" | Set max retry limits (3 per hour). After limit, escalate to human. Investigate root cause. |
| "Liveness probe keeps restarting healthy pods" | Probe thresholds too aggressive. Increase `failureThreshold` and `timeoutSeconds`. Test under load. |
| "Self-healing masks underlying problems" | Always log and alert on auto-remediation events. Review weekly for patterns that need permanent fixes. |
| "Cascading restarts across the cluster" | Add PodDisruptionBudgets. Stagger health check timing. Rate-limit restarts per deployment. |

---

## Summary Table

| Mechanism | What It Heals | Speed | Complexity |
|-----------|--------------|-------|------------|
| **Liveness Probes** | Hung/crashed processes | 10-30s | Low |
| **Readiness Probes** | Temporarily unhealthy pods | 5-15s | Low |
| **ReplicaSet** | Pod/node failures | 30-60s | Low |
| **HPA/KEDA** | Traffic spikes, load changes | 30-120s | Medium |
| **Circuit Breaker** | Downstream failures | Instant | Medium |
| **K8s Operators** | Database failover, complex stateful apps | 30-120s | High |
| **Custom Remediation** | Application-specific failures | 10-60s | High |

---

## Quick Revision Questions

1. **What is a self-healing system?**
   > A system that automatically detects failures, diagnoses root causes, and applies remediation without human intervention. It uses a continuous observe → analyze → act → verify control loop.

2. **Name three Kubernetes built-in self-healing mechanisms.**
   > Liveness probes (restart unhealthy pods), readiness probes (remove from load balancer), and ReplicaSets (reschedule pods when nodes fail). Also HPA for auto-scaling.

3. **What's the difference between liveness and readiness probes?**
   > Liveness checks if the process is alive — failure triggers a restart. Readiness checks if the pod can serve traffic — failure removes it from the service endpoints but doesn't restart it.

4. **How do you prevent auto-remediation loops?**
   > Set max retry limits (e.g., 3 attempts per hour), implement cooldown periods between actions, escalate to humans after limits are exceeded, and log all auto-remediation events for review.

5. **What is the Kubernetes Operator pattern?**
   > A custom controller that watches Custom Resources and continuously reconciles the actual state of a complex application (like a database) to match the desired state defined in the CR.

6. **Why should you still alert on auto-healed incidents?**
   > Auto-remediation treats symptoms, not root causes. Without alerting, underlying problems go unnoticed and worsen. Weekly review of auto-healing patterns reveals what needs permanent fixes.

---

[← Previous: Reducing Operational Burden](04-reducing-operational-burden.md) | [Back to README](../README.md) | [Next: Automation ROI →](06-automation-roi.md)

# Chapter 7.7: Runtime Protection with Falco

## Overview

Falco is an open-source runtime security tool for Kubernetes that detects anomalous behavior using eBPF-based system call monitoring. It catches threats that static scanning misses — shell spawning, unexpected network connections, sensitive file access, and privilege escalation at runtime.

---

## How Falco Works

```
FALCO ARCHITECTURE:

  ┌──────────────────────────────────────────────────────┐
  │                   LINUX KERNEL                        │
  │                                                       │
  │  ┌───────────────────────────────────────────────┐   │
  │  │              eBPF PROBE                        │   │
  │  │  Intercepts system calls:                      │   │
  │  │  open, execve, connect, clone, write, ...      │   │
  │  └───────────────────┬───────────────────────────┘   │
  └──────────────────────┼────────────────────────────────┘
                         │ syscall events
                         ▼
  ┌──────────────────────────────────────────────────────┐
  │                   FALCO ENGINE                        │
  │                                                       │
  │  ┌──────────┐    ┌──────────┐    ┌──────────────┐   │
  │  │ Event    │───▶│ Rules    │───▶│ Alert        │   │
  │  │ Parser   │    │ Engine   │    │ Dispatcher   │   │
  │  └──────────┘    └──────────┘    └──────┬───────┘   │
  │                                          │           │
  └──────────────────────────────────────────┼───────────┘
                                             │
              ┌──────────────────────────────┼──────────┐
              │                              │          │
              ▼              ▼               ▼          ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐ ┌────────┐
        │ stdout   │  │ Slack    │  │ SIEM     │ │PagerDty│
        │ syslog   │  │ Teams    │  │ Splunk   │ │OpsGenie│
        │          │  │          │  │ ELK      │ │        │
        └──────────┘  └──────────┘  └──────────┘ └────────┘
```

---

## Installing Falco

```bash
# Helm installation (recommended for K8s)
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update

helm install falco falcosecurity/falco \
    -n falco --create-namespace \
    --set falcosidekick.enabled=true \
    --set falcosidekick.config.slack.webhookurl="https://hooks.slack.com/..." \
    --set driver.kind=ebpf \
    --set collectors.kubernetes.enabled=true

# Verify installation
kubectl get pods -n falco
kubectl logs -l app.kubernetes.io/name=falco -n falco --tail=20

# Check Falco is receiving events
kubectl logs -l app.kubernetes.io/name=falco -n falco | grep "Rule"
```

---

## Falco Rules

```yaml
# Default rule: Detect shell spawned in container
- rule: Terminal shell in container
  desc: >
    A shell was used as the entrypoint/exec point into a container
    with an attached terminal
  condition: >
    spawned_process and container and
    shell_procs and proc.tty != 0 and
    container_entrypoint
  output: >
    A shell was spawned in a container with an attached terminal
    (user=%user.name user_loginuid=%user.loginuid %container.info
    shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline
    terminal=%proc.tty container_id=%container.id image=%container.image.repository)
  priority: NOTICE
  tags: [container, shell, mitre_execution]
```

### Custom Security Rules

```yaml
# /etc/falco/custom-rules.yaml

# 1. Detect cryptocurrency mining
- rule: Detect Crypto Mining
  desc: Detect processes related to cryptocurrency mining
  condition: >
    spawned_process and container and
    (proc.name in (xmrig, minerd, minergate, cpuminer) or
     proc.cmdline contains "stratum+tcp" or
     proc.cmdline contains "mining" or
     proc.cmdline contains "cryptonight")
  output: >
    Cryptocurrency mining detected
    (user=%user.name container=%container.name
    process=%proc.name cmdline=%proc.cmdline
    image=%container.image.repository)
  priority: CRITICAL
  tags: [container, crypto, mitre_execution]

# 2. Detect sensitive file access
- rule: Read Sensitive Files in Container
  desc: Detect reads of sensitive files within containers
  condition: >
    open_read and container and
    fd.name in (/etc/shadow, /etc/passwd, /etc/sudoers,
                /root/.ssh/authorized_keys, /root/.bash_history)
  output: >
    Sensitive file read in container
    (user=%user.name file=%fd.name container=%container.name
    image=%container.image.repository)
  priority: WARNING
  tags: [container, filesystem, mitre_discovery]

# 3. Detect reverse shell
- rule: Reverse Shell in Container
  desc: Detect reverse shell connections
  condition: >
    spawned_process and container and
    ((proc.name = "bash" and proc.cmdline contains "/dev/tcp") or
     (proc.name = "python" and proc.cmdline contains "socket") or
     (proc.name = "nc" and proc.cmdline contains "-e") or
     (proc.name = "perl" and proc.cmdline contains "socket"))
  output: >
    Reverse shell detected
    (user=%user.name container=%container.name
    cmdline=%proc.cmdline image=%container.image.repository)
  priority: CRITICAL
  tags: [container, network, mitre_execution]

# 4. Detect kubectl exec
- rule: Kubectl Exec into Pod
  desc: Detect when someone exec's into a running pod
  condition: >
    spawned_process and container and
    proc.pname = "runc:[2:INIT]" and
    proc.name in (bash, sh, ash, dash, zsh)
  output: >
    Exec into pod detected
    (user=%user.name container=%container.name
    namespace=%k8s.ns.name pod=%k8s.pod.name
    command=%proc.cmdline)
  priority: WARNING
  tags: [container, k8s, mitre_execution]

# 5. Detect outbound connection to unusual port
- rule: Unexpected Outbound Connection
  desc: Container making connections to non-standard ports
  condition: >
    outbound and container and
    not (fd.sport in (53, 80, 443, 8080, 8443, 5432, 3306,
                      6379, 27017, 9200, 9300)) and
    not k8s.ns.name = "kube-system"
  output: >
    Unexpected outbound connection from container
    (container=%container.name ip=%fd.sip port=%fd.sport
    namespace=%k8s.ns.name image=%container.image.repository)
  priority: NOTICE
  tags: [container, network, mitre_exfiltration]

# 6. Detect binary written to disk
- rule: Write Binary to Container
  desc: Detect writing executable files in a container
  condition: >
    open_write and container and
    evt.arg.flags contains O_CREAT and
    fd.name startswith /tmp and
    (fd.name endswith ".sh" or fd.name endswith ".py" or
     fd.name endswith ".elf" or fd.name endswith ".bin")
  output: >
    Binary/script written to container
    (user=%user.name file=%fd.name container=%container.name
    image=%container.image.repository)
  priority: ERROR
  tags: [container, filesystem, mitre_persistence]
```

---

## Falco Sidekick (Alert Routing)

```yaml
# Falco Sidekick configuration
# Routes alerts to multiple destinations

# values.yaml for Helm
falcosidekick:
  enabled: true
  config:
    # Slack notifications
    slack:
      webhookurl: "https://hooks.slack.com/services/T00/B00/xxx"
      channel: "#security-alerts"
      outputformat: "all"
      minimumpriority: "warning"

    # PagerDuty for critical alerts
    pagerduty:
      routingkey: "your-routing-key"
      minimumpriority: "critical"

    # Elasticsearch / SIEM
    elasticsearch:
      hostport: "https://elasticsearch:9200"
      index: "falco-alerts"
      minimumpriority: "notice"

    # Prometheus metrics
    prometheus:
      extralabels: "source:falco"

    # Custom webhook
    webhook:
      address: "https://security-api.company.com/falco-alerts"
      minimumpriority: "warning"
```

```
ALERT ROUTING STRATEGY:

  Priority        │ Destinations
  ────────────────┼─────────────────────────────────
  EMERGENCY       │ PagerDuty + Slack + SIEM
  CRITICAL        │ PagerDuty + Slack + SIEM
  ERROR           │ Slack + SIEM
  WARNING         │ Slack + SIEM
  NOTICE          │ SIEM only
  INFORMATIONAL   │ SIEM only (if needed)
  DEBUG           │ Local log only
```

---

## Falco in Production

```yaml
# Production Falco deployment with Helm
# values.yaml

driver:
  kind: ebpf                    # eBPF driver (recommended)

falco:
  grpc:
    enabled: true
  grpcOutput:
    enabled: true
  jsonOutput: true
  logLevel: info

  # Performance tuning
  bufferedOutputs: true
  outputs:
    rate: 0                     # No rate limiting
    maxBurst: 1000

  # Rules
  rulesFile:
    - /etc/falco/falco_rules.yaml
    - /etc/falco/falco_rules.local.yaml
    - /etc/falco/custom-rules.yaml

# Resource limits
resources:
  requests:
    cpu: 100m
    memory: 512Mi
  limits:
    cpu: 1000m
    memory: 1024Mi

# Tolerations for running on all nodes
tolerations:
  - effect: NoSchedule
    operator: Exists

# Metrics for monitoring Falco itself
serviceMonitor:
  enabled: true
```

---

## Testing Falco Rules

```bash
# Trigger test events to verify Falco is working

# 1. Shell spawn (should trigger "Terminal shell in container")
kubectl exec -it test-pod -- /bin/bash

# 2. Read sensitive file
kubectl exec test-pod -- cat /etc/shadow

# 3. Write to /tmp
kubectl exec test-pod -- sh -c "echo 'test' > /tmp/malicious.sh"

# 4. Outbound connection to unusual port
kubectl exec test-pod -- nc -zv 8.8.8.8 12345

# 5. Run mining-like process name
kubectl exec test-pod -- sh -c "cp /bin/ls /tmp/xmrig && /tmp/xmrig"

# Check Falco detected the events
kubectl logs -l app.kubernetes.io/name=falco -n falco | grep -E "Warning|Error|Critical"

# Falco event generator (automated testing)
kubectl run falco-event-generator \
    --image=falcosecurity/event-generator \
    --restart=Never \
    -- run syscall --all
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Falco pod CrashLoopBackOff | Kernel headers missing for eBPF | Install kernel headers or use `driver.kind=modern_ebpf` |
| Too many alerts flooding Slack | Alert priority too low | Set `minimumpriority: warning` in Sidekick config |
| Falco not detecting events | Rules not loaded | Check `falco.rulesFile` paths; verify with test events |
| High CPU usage on nodes | Scanning too many syscalls | Tune rules to reduce scope; increase `rate` limiting |
| Sidekick not sending alerts | Wrong webhook URL or network issue | Test webhook manually; check Sidekick logs |

---

## Summary Table

| Component | Purpose |
|-----------|---------|
| **Falco Engine** | eBPF-based system call monitoring and rule evaluation |
| **Falco Rules** | YAML-based detection rules (default + custom) |
| **Falco Sidekick** | Alert routing to Slack, PagerDuty, SIEM, webhooks |
| **eBPF Driver** | Kernel-level event capture without kernel modules |
| **Event Generator** | Testing tool to trigger known events and verify detection |

---

## Quick Revision Questions

1. **How does Falco detect threats differently from static scanning tools?**
   > Static tools (kubescape, Trivy) check configurations and known vulnerabilities. Falco monitors actual system calls at runtime using eBPF — it detects live malicious activity: shell spawning, file access, network connections, privilege escalation. It catches zero-day attacks that no CVE database would flag.

2. **What is eBPF and why does Falco use it?**
   > eBPF (extended Berkeley Packet Filter) allows running sandboxed programs in the Linux kernel without modifying kernel source or loading kernel modules. Falco uses it to intercept system calls with minimal performance overhead, providing deep visibility into container and host activity.

3. **How do Falco rules work?**
   > Rules have a condition (filtering system call events using Falco's filter syntax), an output (message template with fields), and a priority. Conditions use macros and lists for reusability. When a system call matches a rule's condition, Falco generates an alert with the output fields.

4. **What is Falco Sidekick?**
   > Sidekick is an alert forwarder that takes Falco events and routes them to 50+ destinations: Slack, PagerDuty, Elasticsearch, Prometheus, S3, custom webhooks. It supports filtering by priority level, ensuring critical alerts go to on-call while informational ones go to SIEM.

5. **How should Falco be tested to ensure rules are working?**
   > Use the `falco-event-generator` container to trigger known events (shell spawn, file reads, network connections). Manually exec into test pods and perform suspicious actions. Check Falco logs for generated alerts. Integrate testing into regular security validation processes.

6. **What are common custom Falco rules for Kubernetes?**
   > Crypto mining detection (xmrig/minerd process names), reverse shell detection (bash /dev/tcp patterns), sensitive file reads (/etc/shadow, SSH keys), unexpected outbound connections (non-standard ports), binary writes to /tmp, and kubectl exec into production pods.

---

[← Previous: 7.6 Security Scanning Tools](06-security-scanning-tools.md) | [Next: 8.1 IaC Security Scanning →](../08-Infrastructure-Security/01-iac-security-scanning.md)

[Back to Table of Contents](../README.md)

# Unit 8: Container and Serverless Security — Topic 5: Runtime Protection

## Overview

**Runtime protection** for containers and serverless functions provides real-time threat detection and prevention during workload execution. While pre-deployment security (image scanning, code review) catches known vulnerabilities, runtime protection detects zero-day exploits, abnormal behaviors, and active attacks. Cloud-native and third-party runtime security solutions are essential for comprehensive workload protection.

---

## 1. Runtime Threats

```
CONTAINER RUNTIME THREATS:

PROCESS-LEVEL:
  → Unexpected process execution
  → Reverse shells (bash, python, nc)
  → Cryptocurrency miners
  → Malicious script execution
  → Privilege escalation attempts
  → Binary downloads and execution

FILE SYSTEM:
  → Modification of system binaries
  → Writing to sensitive paths (/etc/shadow)
  → Dropping malware
  → Reading sensitive files
  → Config file tampering

NETWORK:
  → Connections to C2 servers
  → Port scanning from container
  → DNS tunneling
  → Unexpected outbound connections
  → Lateral movement attempts
  → Data exfiltration

CONTAINER ESCAPE:
  → Kernel exploits (container → host)
  → Docker socket exploitation
  → Privileged container abuse
  → Host path volume mounts
  → CAP_SYS_ADMIN abuse

SERVERLESS RUNTIME THREATS:
  → Code injection via event data
  → Dependency hijacking
  → Function timeout abuse
  → Resource exhaustion
  → Temporary storage (/tmp) data leakage
  → Cold start information disclosure
```

---

## 2. Cloud-Native Runtime Protection

```
CLOUD PROVIDER SOLUTIONS:

AWS:
  GUARDDUTY EKS RUNTIME MONITORING:
  → Agent-based monitoring on EKS nodes
  → Detects: Crypto mining, reverse shells,
    privilege escalation, container escapes
  → Automated findings in Security Hub
  → No cluster modification needed (addon)
  
  GUARDDUTY LAMBDA PROTECTION:
  → Monitors Lambda network activity
  → Detects malicious network connections
  → DNS query monitoring
  → Integration with Security Hub
  
  # Enable GuardDuty EKS Runtime
  aws guardduty update-detector \
    --detector-id DETECTOR_ID \
    --features '[{
      "Name": "EKS_RUNTIME_MONITORING",
      "Status": "ENABLED"
    }]'

AZURE:
  DEFENDER FOR CONTAINERS:
  → Runtime threat detection
  → Vulnerability assessment
  → Admission control
  → Network-level protection
  → AKS, Arc-enabled K8s, standalone
  
  Detections:
  → Suspicious process execution
  → Web shell detection
  → Crypto mining
  → Binary exploitation
  → Reverse shell creation

GCP:
  CONTAINER THREAT DETECTION:
  → Monitors GKE at kernel level
  → No agent deployment needed
  → Findings in SCC
  
  Detections:
  → Added binary executed
  → Added library loaded
  → Malicious script executed
  → Reverse shell
  → Unexpected child shell

  VM THREAT DETECTION:
  → Hypervisor-level scanning
  → Memory scanning for threats
  → No agent required
  → Cryptocurrency mining detection
  → Rootkit detection
```

---

## 3. Open Source Runtime Security

```
FALCO:
  → CNCF project
  → Kernel-level monitoring (eBPF/kernel module)
  → Real-time threat detection
  → Rule-based alerting
  → Container and host monitoring
  → Kubernetes audit log analysis

  ARCHITECTURE:
  ┌────────────────────────────────┐
  │         FALCO ENGINE           │
  │                                │
  │  Kernel Module / eBPF Probe    │
  │  → Captures syscalls           │
  │                                │
  │  Rules Engine                  │
  │  → Evaluates against rules     │
  │                                │
  │  Alert Output                  │
  │  → stdout, syslog, HTTP, gRPC  │
  └────────────────────────────────┘

  EXAMPLE RULES:
  # Detect shell in container
  - rule: Terminal shell in container
    desc: Detect shell opened in container
    condition: >
      spawned_process and container and
      shell_procs and proc.tty != 0
    output: >
      Shell opened in container
      (user=%user.name container=%container.name
       shell=%proc.name parent=%proc.pname)
    priority: WARNING

  # Detect outbound connection
  - rule: Unexpected outbound connection
    desc: Detect network connection to suspicious port
    condition: >
      outbound and container and
      fd.sport in (4444, 5555, 6666, 8888, 9999)
    output: >
      Suspicious outbound connection from container
      (container=%container.name ip=%fd.rip port=%fd.sport)
    priority: CRITICAL

  # Detect file modification
  - rule: Write below /etc
    desc: Detect write to /etc directory
    condition: >
      write_etc_common and container
    output: >
      File written below /etc
      (user=%user.name file=%fd.name)
    priority: ERROR

TETRAGON (Cilium):
  → eBPF-based runtime enforcement
  → Process execution monitoring
  → Network monitoring
  → File access monitoring
  → Can BLOCK (not just detect)
  → Kubernetes-native
```

---

## 4. Security Policies and Response

```
RUNTIME SECURITY POLICIES:

DETECTION POLICIES:
  → Alert on unexpected process execution
  → Alert on network connections to known-bad IPs
  → Alert on file system modifications
  → Alert on privilege escalation
  → Alert on sensitive file access

PREVENTION POLICIES:
  → Block execution of unknown binaries
  → Block outbound connections to unauthorized IPs
  → Block file writes to system directories
  → Kill container on escape attempt
  → Enforce seccomp/AppArmor profiles

AUTOMATED RESPONSE:
  Detection → Alert → Response
  
  Response Actions:
  → Kill suspicious container
  → Isolate pod (network policy)
  → Cordon and drain node
  → Capture forensic data
  → Alert security team
  → Create incident ticket
  → Scale replacement containers

INCIDENT RESPONSE FOR CONTAINERS:
  1. DETECT: Runtime alert triggers
  2. CONTAIN: Isolate affected pod/container
  3. PRESERVE: Capture container state/logs
  4. INVESTIGATE: Analyze events timeline
  5. ERADICATE: Remove compromised workloads
  6. RECOVER: Deploy clean replacement
  7. LESSONS: Update policies and detection

FORENSIC DATA TO CAPTURE:
  → Container logs
  → Process listing (ps aux)
  → Network connections (netstat, ss)
  → File system diff (changed files)
  → Environment variables
  → Container image details
  → Kubernetes events
  → Node-level audit logs
```

---

## 5. Assessment

```
RUNTIME PROTECTION ASSESSMENT:

CHECKLIST:
  [ ] Runtime protection solution deployed
  [ ] Detection rules cover key threats
  [ ] Alerts routed to security team
  [ ] Automated response configured
  [ ] Container escape detection enabled
  [ ] Network monitoring active
  [ ] File integrity monitoring enabled
  [ ] Process monitoring enabled
  [ ] Forensic capture procedures documented
  [ ] Regular rule tuning and updates
  [ ] False positive management process

TESTING:
  # Test process detection
  kubectl exec -it test-pod -- /bin/bash
  
  # Test network detection
  kubectl exec test-pod -- curl http://suspicious-site.com
  
  # Test file modification detection
  kubectl exec test-pod -- touch /etc/test-file
  
  # Test crypto mining detection (simulation)
  kubectl exec test-pod -- stress --cpu 4
  
  ⚠ Only test in non-production environments

COMMON GAPS:
  Gap                              | Risk
  No runtime protection            | Critical
  Detection only (no prevention)   | High
  No container escape detection    | High
  No network monitoring            | Medium
  Alerts not routed to SOC         | High
  No forensic capture capability   | Medium
  Rules not tuned (noisy alerts)   | Medium

TOOLS COMPARISON:
  Tool         | Type    | Detection | Prevention | Agent
  GuardDuty    | Cloud   | Yes       | No         | Addon
  Defender     | Cloud   | Yes       | Partial    | Extension
  CTD (GCP)    | Cloud   | Yes       | No         | Agentless
  Falco        | OSS     | Yes       | No         | Agent/eBPF
  Tetragon     | OSS     | Yes       | Yes        | eBPF
  Aqua         | Commercial| Yes     | Yes        | Agent
  Sysdig       | Commercial| Yes     | Yes        | Agent
```

---

## Summary Table

| Solution | Provider | Detection | Prevention | Cost |
|----------|----------|-----------|------------|------|
| GuardDuty Runtime | AWS | Yes | No | Pay per event |
| Defender for Containers | Azure | Yes | Partial | Paid plan |
| Container Threat Detection | GCP | Yes | No | SCC Premium |
| Falco | Open Source | Yes | No | Free |
| Tetragon | Open Source | Yes | Yes | Free |

---

## Revision Questions

1. What runtime threats are specific to containers?
2. How do cloud-native runtime protection services detect threats?
3. How does Falco monitor container behavior at the kernel level?
4. What automated responses should be configured for runtime threats?
5. How should runtime protection be tested and assessed?

---

*Previous: [04-function-permissions.md](04-function-permissions.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

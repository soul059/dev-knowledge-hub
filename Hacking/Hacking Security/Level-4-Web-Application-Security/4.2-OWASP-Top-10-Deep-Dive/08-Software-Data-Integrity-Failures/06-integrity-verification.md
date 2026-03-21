# Unit 8: A08 - Software and Data Integrity Failures — Topic 6: Integrity Verification

## Overview

Integrity verification is the practice of **confirming that data, code, or configuration has not been altered** from its intended state. This encompasses file integrity monitoring, data validation, tamper detection, and continuous verification throughout the software lifecycle. Without integrity verification, unauthorized modifications go undetected.

---

## 1. Types of Integrity Verification

```
┌─────────────────────────────────────────────────────────────────┐
│           INTEGRITY VERIFICATION TYPES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. FILE INTEGRITY MONITORING (FIM)                            │
│     Monitor critical files for unauthorized changes            │
│     Tools: OSSEC, Tripwire, AIDE, Wazuh                       │
│                                                                 │
│  2. DATA INTEGRITY                                             │
│     Ensure database/API data hasn't been tampered              │
│     Methods: HMAC, digital signatures, checksums               │
│                                                                 │
│  3. CODE INTEGRITY                                             │
│     Verify deployed code matches signed artifacts              │
│     Methods: Code signing, hash comparison                     │
│                                                                 │
│  4. CONFIGURATION INTEGRITY                                    │
│     Detect unauthorized config changes                         │
│     Methods: IaC drift detection, config audit                 │
│                                                                 │
│  5. COMMUNICATION INTEGRITY                                    │
│     Verify data in transit hasn't been modified                │
│     Methods: TLS, HMAC, digital signatures                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. File Integrity Monitoring (FIM)

```bash
# AIDE — Advanced Intrusion Detection Environment
# Initialize baseline
aide --init
cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Check for changes
aide --check
# Output:
# Changed: /etc/passwd
# Changed: /usr/bin/ssh
# Added:   /tmp/suspicious_file

# OSSEC/Wazuh — Real-time FIM
# Configuration in ossec.conf:
<syscheck>
    <frequency>3600</frequency>
    <directories check_all="yes" realtime="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories check_all="yes">/boot</directories>
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
</syscheck>
```

---

## 3. Data Integrity with HMAC

```python
import hmac
import hashlib
import json

SECRET_KEY = b'your-secret-key-256-bits'

def sign_data(data):
    """Create HMAC signature for data integrity"""
    message = json.dumps(data, sort_keys=True).encode()
    signature = hmac.new(SECRET_KEY, message, hashlib.sha256).hexdigest()
    return signature

def verify_data(data, received_signature):
    """Verify data has not been tampered with"""
    expected_signature = sign_data(data)
    return hmac.compare_digest(expected_signature, received_signature)

# Usage
order = {"item": "laptop", "price": 999.99, "user_id": 42}
signature = sign_data(order)

# Send: order + signature
# Receive: order + signature → verify
if verify_data(received_order, received_signature):
    process_order(received_order)  # Data is authentic
else:
    reject_order()  # Data was tampered!
```

---

## 4. Configuration Drift Detection

```bash
# Terraform — Detect infrastructure drift
terraform plan
# Shows differences between desired state and actual state

# AWS Config — Continuous compliance monitoring
# Detects when resources deviate from rules

# Kubernetes — GitOps with ArgoCD
# ArgoCD continuously compares cluster state to Git repo
# Any manual changes are detected and can be auto-reverted
```

---

## 5. Continuous Integrity Verification Pipeline

```yaml
# Integrity verification in CI/CD
name: Integrity Check
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - name: Verify deployed artifacts
        run: |
          # Download deployed artifact hash
          DEPLOYED_HASH=$(ssh prod "sha256sum /app/binary" | cut -d' ' -f1)
          
          # Compare with signed build artifact hash
          EXPECTED_HASH=$(cat build-artifacts/binary.sha256)
          
          if [ "$DEPLOYED_HASH" != "$EXPECTED_HASH" ]; then
            echo "⚠️ INTEGRITY VIOLATION DETECTED"
            # Alert security team
            curl -X POST $SLACK_WEBHOOK -d '{"text":"Integrity violation on production!"}'
            exit 1
          fi
          echo "✅ Integrity verified"
```

---

## 6. Integrity Verification Checklist

```
□ FIM deployed on all production servers
□ Critical files baselined and monitored (config, binaries, scripts)
□ HMAC/signatures used for data exchanged between services
□ Deployed code verified against build artifacts
□ Infrastructure drift detection enabled
□ Database integrity checks (checksums on critical tables)
□ Log integrity (append-only, signed log entries)
□ Alert on any integrity violation
□ Regular integrity audits (weekly/monthly)
□ Immutable infrastructure reduces integrity concerns
```

---

## Summary Table

| Verification Type | Method | Tools |
|------------------|--------|-------|
| File integrity | Hash comparison | AIDE, OSSEC, Tripwire |
| Data integrity | HMAC/Digital signatures | Custom implementation |
| Code integrity | Artifact hash verification | cosign, SHA comparison |
| Config integrity | Drift detection | Terraform, AWS Config |
| Log integrity | Append-only + signing | Blockchain, signed logs |

---

## Revision Questions

1. What are the five types of integrity verification? Give an example tool or method for each.
2. Set up file integrity monitoring with AIDE or OSSEC for a Linux server.
3. Implement HMAC-based data integrity verification in Python.
4. What is configuration drift and how do you detect it in cloud infrastructure?
5. Design a continuous integrity verification pipeline that checks production against build artifacts.

---

*Previous: [05-code-signing.md](05-code-signing.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

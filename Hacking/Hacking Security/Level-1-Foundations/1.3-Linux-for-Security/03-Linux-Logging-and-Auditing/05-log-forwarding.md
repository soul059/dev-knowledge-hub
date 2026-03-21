# Log Forwarding

## Unit 3 - Topic 5 | Linux Logging and Auditing

---

## Overview

**Log forwarding** sends log data from individual systems to a centralized log server or SIEM. This is critical for security because attackers often **delete local logs** to cover their tracks. Centralized logging ensures logs are preserved, correlated across systems, and available for analysis even if the source system is compromised.

---

## 1. Why Forward Logs?

```
┌──────────────────────────────────────────────────────────────┐
│              CENTRALIZED LOGGING ARCHITECTURE                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌────────┐   ┌────────┐   ┌────────┐                      │
│  │ Web    │   │ DB     │   │ App    │                      │
│  │ Server │   │ Server │   │ Server │                      │
│  └───┬────┘   └───┬────┘   └───┬────┘                      │
│      │            │            │                             │
│      └────────────┼────────────┘                             │
│                   │ Forward logs (TLS encrypted)             │
│                   ▼                                          │
│         ┌─────────────────┐                                  │
│         │ CENTRAL LOG     │                                  │
│         │ SERVER / SIEM   │                                  │
│         │ (Elastic, Splunk│                                  │
│         │  Graylog)       │                                  │
│         └─────────────────┘                                  │
│                                                              │
│  BENEFITS:                                                   │
│  ✅ Logs preserved even if source compromised               │
│  ✅ Correlation across multiple systems                      │
│  ✅ Centralized search and alerting                          │
│  ✅ Compliance requirements (PCI DSS, HIPAA)                │
│  ✅ Incident response from single location                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. rsyslog Forwarding

```bash
# === CLIENT (Log Source) ===
# /etc/rsyslog.d/50-remote.conf

# Forward ALL logs via TCP (reliable)
*.* @@logserver.example.com:514

# Forward ALL logs via UDP (faster, less reliable)
*.* @logserver.example.com:514

# Forward only auth logs
auth,authpriv.* @@logserver.example.com:514

# Forward with TLS encryption
# Requires rsyslog-gnutls package
$DefaultNetstreamDriverCAFile /etc/ssl/certs/ca.pem
$DefaultNetstreamDriver gtls
$ActionSendStreamDriverMode 1
$ActionSendStreamDriverAuthMode anon
*.* @@logserver.example.com:6514

# Restart rsyslog
systemctl restart rsyslog


# === SERVER (Log Receiver) ===
# /etc/rsyslog.conf

# Enable TCP reception
module(load="imtcp")
input(type="imtcp" port="514")

# Enable UDP reception
module(load="imudp")
input(type="imudp" port="514")

# Store logs by hostname
$template RemoteLogs,"/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log"
*.* ?RemoteLogs
& ~

systemctl restart rsyslog
```

---

## 3. systemd-journal-remote

```bash
# Forward journald logs to remote server

# === CLIENT ===
apt install systemd-journal-remote

# /etc/systemd/journal-upload.conf
[Upload]
URL=http://logserver.example.com:19532
# ServerKeyFile=/etc/ssl/private/key.pem
# ServerCertificateFile=/etc/ssl/certs/cert.pem

systemctl enable --now systemd-journal-upload

# === SERVER ===
apt install systemd-journal-remote
systemctl enable --now systemd-journal-remote.socket
# Receives journals on port 19532
```

---

## 4. Forwarding Protocols

| Protocol | Port | Reliability | Encryption | Use Case |
|----------|:----:|:-----------:|:----------:|----------|
| **UDP Syslog** | 514 | Low (lossy) | ❌ | Legacy, high-volume |
| **TCP Syslog** | 514 | High | ❌ | Reliable delivery |
| **TLS Syslog** | 6514 | High | ✅ | Secure forwarding |
| **RELP** | 2514 | Very High | Optional | Guaranteed delivery |
| **HTTP/HTTPS** | 443 | High | ✅ | Cloud SIEM ingestion |

---

## 5. Queue and Buffer Configuration

```bash
# Ensure logs aren't lost during network outages
# rsyslog disk-assisted queue

$WorkDirectory /var/spool/rsyslog
$ActionQueueFileName remote_queue    # Queue file name
$ActionQueueMaxDiskSpace 1g          # Max 1GB disk queue
$ActionQueueSaveOnShutdown on        # Save queue on shutdown
$ActionQueueType LinkedList          # Use async queue
$ActionResumeRetryCount -1           # Infinite retries

*.* @@logserver.example.com:514
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Why forward** | Prevent log deletion by attackers |
| **rsyslog** | Primary forwarding mechanism |
| **TCP vs UDP** | TCP for reliability, UDP for speed |
| **TLS** | Encrypt log traffic in transit |
| **RELP** | Guaranteed delivery protocol |
| **Queuing** | Buffer logs during network outages |

---

## Quick Revision Questions

1. **Why is centralized logging important for security?**
2. **What is the difference between `@` and `@@` in rsyslog forwarding?**
3. **How do you encrypt log forwarding with rsyslog?**
4. **What happens to logs if the network is down?**
5. **What protocol provides guaranteed log delivery?**

---

[← Previous: Log Analysis](04-log-analysis.md) | [Next: SIEM Integration →](06-siem-integration.md)

---

*Unit 3 - Topic 5 of 6 | [Back to README](../README.md)*

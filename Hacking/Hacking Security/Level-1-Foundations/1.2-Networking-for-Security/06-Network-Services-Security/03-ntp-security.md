# NTP Security

## Unit 6 - Topic 3 | Network Services Security

---

## Overview

**NTP (Network Time Protocol)** synchronizes clocks across network devices. Accurate time is critical for **log correlation**, **certificate validation**, **Kerberos authentication**, and **forensics**. NTP is targeted for **amplification DDoS attacks** and **time manipulation** that can break security controls.

---

## 1. Why Time Matters for Security

| System | Time Dependency | Impact of Wrong Time |
|--------|----------------|---------------------|
| **Kerberos** | 5-minute max skew | Authentication fails completely |
| **TLS Certificates** | Valid from/to dates | Certificate rejected, HTTPS fails |
| **Log Correlation** | Timestamp matching | Can't correlate events across systems |
| **TOTP (MFA)** | 30-second windows | MFA codes won't match |
| **Forensics** | Timeline reconstruction | Invalid evidence timeline |
| **Scheduled Tasks** | Cron/Task Scheduler | Tasks run at wrong time |

---

## 2. NTP Attacks

### NTP Amplification DDoS

```
┌──────────────────────────────────────────────────────────────┐
│              NTP AMPLIFICATION                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "monlist" command returns list of last 600 clients          │
│  Request: ~234 bytes → Response: ~100 packets × 482 bytes   │
│  AMPLIFICATION FACTOR: ~556x!                                │
│                                                              │
│  Attack:                                                     │
│  1. Spoof source IP to victim's IP                           │
│  2. Send monlist to thousands of NTP servers                 │
│  3. Massive amplified responses flood victim                 │
│                                                              │
│  DEFENSE: Disable monlist command                            │
│  # /etc/ntp.conf                                             │
│  restrict default noquery nomodify notrap nopeer             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Time Manipulation

```
If attacker controls NTP responses:
• Kerberos: Roll time forward → old tickets still valid
• Certificates: Roll time back → expired certs become valid
• Logs: Timestamps become unreliable
• TOTP: MFA tokens won't validate

DEFENSE: NTP authentication, multiple time sources
```

---

## 3. NTP Security Configuration

```bash
# /etc/ntp.conf — Secure configuration

# Use multiple trusted time sources
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server time.google.com iburst
server time.cloudflare.com iburst

# Restrict access — deny by default
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1                    # Allow localhost
restrict ::1                          # Allow IPv6 localhost

# NTP Authentication (symmetric key)
keys /etc/ntp/keys
trustedkey 1
server time-server.corp.com key 1
```

```cisco
! Cisco NTP authentication
ntp authenticate
ntp authentication-key 1 md5 MyNTPSecret
ntp trusted-key 1
ntp server 10.0.0.50 key 1
```

---

## 4. NTP Best Practices

| Practice | Description |
|----------|-------------|
| **Multiple sources** | Use 3+ NTP servers for redundancy and validation |
| **NTP authentication** | Enable symmetric key or autokey authentication |
| **Disable monlist** | Prevent amplification abuse |
| **Restrict access** | Only allow queries from authorized networks |
| **Internal NTP hierarchy** | Dedicated internal NTP servers sync with external |
| **Monitor drift** | Alert if time skew exceeds threshold |
| **NTS (NTP over TLS)** | Modern encrypted NTP (chrony supports this) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **NTP** | Time synchronization — critical for security operations |
| **Amplification** | monlist enables 556x DDoS amplification |
| **Time Manipulation** | Can break Kerberos, certificates, MFA |
| **Defense** | Disable monlist, restrict access, use authentication |
| **Best Practice** | Multiple sources, NTP authentication, internal hierarchy |

---

## Quick Revision Questions

1. **Why is accurate time critical for network security?**
2. **How does NTP amplification work?**
3. **What security systems can be broken by time manipulation?**
4. **How do you secure NTP configuration?**
5. **What is the monlist command and why should it be disabled?**

---

[← Previous: DNS Security](02-dns-security-dnssec.md) | [Next: LDAP/Active Directory →](04-ldap-active-directory.md)

---

*Unit 6 - Topic 3 of 5 | [Back to README](../README.md)*

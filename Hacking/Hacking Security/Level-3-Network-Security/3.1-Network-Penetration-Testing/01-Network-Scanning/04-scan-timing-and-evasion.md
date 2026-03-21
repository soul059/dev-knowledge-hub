# Scan Timing and Evasion

## Unit 1 - Topic 4 | Network Scanning

---

## Overview

**Scan timing and evasion** techniques control how fast and how detectably port scans execute. Proper timing prevents overwhelming the target network, avoids IDS/IPS detection, and ensures accurate results on congested or unstable networks.

---

## 1. Nmap Timing Templates

```bash
# === TIMING TEMPLATES (-T0 to -T5) ===

# -T0 (Paranoid) — 5 minute wait between probes
nmap -T0 target.com
# Use: Extreme stealth, IDS evasion
# Speed: Extremely slow (hours for one host)
# Detection: Nearly undetectable

# -T1 (Sneaky) — 15 second wait between probes
nmap -T1 target.com
# Use: Stealth scanning, sensitive networks
# Speed: Very slow

# -T2 (Polite) — 0.4 second wait, serial scanning
nmap -T2 target.com
# Use: Production networks (don't disrupt)
# Speed: Slow but reliable

# -T3 (Normal) — DEFAULT
nmap -T3 target.com
# Use: Standard pen testing
# Speed: Moderate

# -T4 (Aggressive) — Faster timeouts, parallelism
nmap -T4 target.com
# Use: Reliable networks, time-limited engagements
# Speed: Fast
# ⚠️ May miss ports on congested networks

# -T5 (Insane) — Minimum timeouts, max parallelism
nmap -T5 target.com
# Use: Very reliable local networks only
# Speed: Fastest
# ⚠️ Sacrifices accuracy, triggers all IDS systems
```

---

## 2. Granular Timing Controls

```bash
# === FINE-GRAINED TIMING ===

# Minimum time between probes:
nmap --scan-delay 5s target.com        # Wait 5 sec between probes
nmap --max-scan-delay 10s target.com   # Maximum delay

# Parallelism (concurrent probes):
nmap --min-parallelism 10 target.com   # At least 10 simultaneous
nmap --max-parallelism 1 target.com    # One at a time (stealth)

# Timeout controls:
nmap --host-timeout 30m target.com     # Give up after 30 minutes
nmap --max-retries 2 target.com        # Max 2 retransmissions

# Rate limiting:
nmap --min-rate 100 target.com         # At least 100 packets/sec
nmap --max-rate 50 target.com          # Max 50 packets/sec

# === PRACTICAL STEALTH COMBINATION ===
sudo nmap -sS -T2 \
  --scan-delay 3s \
  --max-parallelism 1 \
  --max-retries 1 \
  -p 22,80,443,445,3389 \
  target.com

# === PRACTICAL SPEED COMBINATION ===
sudo nmap -sS -T4 \
  --min-rate 1000 \
  --max-retries 1 \
  -p- \
  target.com
```

---

## 3. IDS/IPS Evasion Techniques

```bash
# === PACKET FRAGMENTATION ===
sudo nmap -f target.com              # Fragment into 8-byte chunks
sudo nmap -f -f target.com           # Fragment into 16-byte chunks
sudo nmap --mtu 24 target.com        # Custom MTU (must be multiple of 8)
# Splits TCP header across fragments → some IDS can't reassemble

# === DECOY SCANNING ===
sudo nmap -D 10.0.0.1,10.0.0.2,ME,10.0.0.3 target.com
# Sends scans from decoy IPs AND your real IP
# Target sees scans from multiple sources
# Can't determine which is the real attacker

sudo nmap -D RND:10 target.com
# Use 10 random decoy IPs

# === SOURCE PORT MANIPULATION ===
sudo nmap --source-port 53 target.com
# Appear as DNS traffic (port 53)
# Some firewalls allow DNS source port through

sudo nmap --source-port 80 target.com
# Appear as web server response

# === DATA LENGTH PADDING ===
nmap --data-length 25 target.com
# Appends random bytes to packets
# Changes packet signature from standard Nmap

# === IDLE/ZOMBIE SCAN ===
sudo nmap -sI zombie.com target.com
# Uses a "zombie" host to scan the target
# Completely anonymous — your IP never touches target
# ⚠️ Requires finding a suitable zombie (predictable IP ID)
```

---

## 4. Timing vs Detection Trade-off

```
SPEED ←───────────────────────────→ STEALTH

T5 Insane ██████████████████████░░░░░ T0 Paranoid
│                                     │
├── Fastest results                   ├── Nearly invisible
├── Triggers ALL IDS                  ├── Takes hours/days
├── May miss ports                    ├── Most accurate
├── May crash services                ├── No network impact
└── Use: CTF, local lab               └── Use: Red team, real engagement

RECOMMENDED FOR PEN TESTING:
├── Internal network: -T4 (fast, reliable)
├── External target:  -T3 (default, balanced)
├── Stealth needed:   -T2 with --scan-delay
└── Red team:         -T1 or custom timing
```

---

## 5. Practical Evasion Scenarios

| Scenario | Technique | Command |
|----------|-----------|---------|
| **IDS monitoring** | Slow + fragmented | `-T1 -f --scan-delay 5s` |
| **Firewall filtering** | Source port spoof | `--source-port 53` |
| **Hiding identity** | Decoy scan | `-D RND:10` |
| **Full anonymity** | Idle scan | `-sI zombie target` |
| **Signature evasion** | Data padding | `--data-length 50` |
| **Rate limiting bypass** | Max rate control | `--max-rate 100` |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **-T0 to -T5** | Paranoid to Insane timing templates |
| **--scan-delay** | Control time between probes |
| **-f** | Fragment packets to evade IDS |
| **-D** | Decoy scan to hide among fake sources |
| **--source-port** | Disguise as trusted traffic (DNS/HTTP) |
| **-sI** | Idle scan for full anonymity |

---

## Quick Revision Questions

1. **What is the default Nmap timing template?**
2. **Which timing template is best for stealth scanning?**
3. **How does packet fragmentation help evade IDS?**
4. **What is a decoy scan and how does it work?**
5. **How does source port manipulation bypass firewalls?**
6. **What is an idle/zombie scan?**

---

[← Previous: Nmap Fundamentals](03-nmap-fundamentals.md) | [Next: Output Formats and Parsing →](05-output-formats-and-parsing.md)

---

*Unit 1 - Topic 4 of 6 | [Back to README](../README.md)*

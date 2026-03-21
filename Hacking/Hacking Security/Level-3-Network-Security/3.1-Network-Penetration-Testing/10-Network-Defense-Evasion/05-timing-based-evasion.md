# Timing-Based Evasion

## Unit 10 - Topic 5 | Network Defense Evasion

---

## Overview

**Timing-based evasion** exploits the way security systems handle time — detection thresholds, log retention, session timeouts, and analysis windows. By controlling the pace and timing of attacks, penetration testers can fly under the radar of time-sensitive security controls.

---

## 1. Slow Scanning

```bash
# === NMAP TIMING TEMPLATES ===
# -T0: Paranoid (5 min between probes)
# -T1: Sneaky (15 sec between probes)
# -T2: Polite (0.4 sec between probes)
# -T3: Normal (default)
# -T4: Aggressive (faster)
# -T5: Insane (fastest, noisy)

# Stealthy scan:
nmap -T1 -sS -p 1-1000 target.com
# Takes hours but avoids most rate-based detection

# Custom timing:
nmap --scan-delay 10s target.com           # 10 sec between probes
nmap --max-rate 1 target.com               # 1 packet per second
nmap --min-rate 0.1 --max-rate 0.5 target.com  # Very slow

# === WHY SLOW SCANNING WORKS ===
# IDS/IPS detection thresholds:
# ├── Alert: "50+ SYN packets in 10 seconds from same IP"
# ├── If we send 1 SYN every 30 seconds → below threshold
# ├── 1000 ports at 1/30sec = 8+ hours
# └── But completely undetected!

# === RANDOMIZE SCAN ORDER ===
nmap --randomize-hosts -iL targets.txt
# Don't scan sequentially — randomize targets
# Pattern: 10.0.0.1, 10.0.0.50, 10.0.0.23, ...
# Not: 10.0.0.1, 10.0.0.2, 10.0.0.3, ...
```

---

## 2. Time-Based Attack Scheduling

```
TIMING STRATEGIES:

BUSINESS HOURS ATTACK:
├── Attack during peak traffic hours
├── Malicious traffic blends with normal volume
├── SOC may be overwhelmed with alerts
├── Best time: Monday-Friday, 9 AM - 5 PM
└── Avoid: Late night (low traffic = easier to spot)

OFF-HOURS ATTACK:
├── SOC may have reduced staffing
├── Fewer people monitoring alerts
├── Best time: Weekends, holidays, 2-4 AM
├── Less traffic = faster scanning
└── Risk: Unusual traffic more visible

DISTRIBUTED TIMING:
├── Spread attack over days/weeks
├── Day 1: Scan subnet A
├── Day 2: Enumerate services
├── Day 3: Scan subnet B
├── Day 4: Attempt exploitation
└── Harder to correlate across days

EVENT-BASED TIMING:
├── During system maintenance windows
├── During software deployments
├── During backup operations
├── Network changes create noise
└── Attack blends with legitimate activity
```

---

## 3. Session and Threshold Evasion

```
THRESHOLD EVASION:

LOGIN BRUTE FORCE:
├── Lockout policy: 5 attempts in 30 minutes
├── Evasion: 3 attempts, wait 31 minutes, repeat
├── Stay 2 attempts below threshold
└── Takes longer but no lockouts

IDS RATE LIMITING:
├── Alert threshold: 100 events/minute from same IP
├── Evasion: Send 50 events/minute (below threshold)
├── Or: Distribute across multiple source IPs
└── Or: Use decoy addresses

SESSION TIMEOUT ABUSE:
├── Keep C2 connections alive with heartbeats
├── But space them to avoid "beacon" detection
├── Use jitter: sleep(60 ± 30 seconds)
├── Vary communication patterns
└── Don't be too regular (regular = suspicious)

LOG ROTATION EVASION:
├── If logs rotate every 7 days
├── Start attack on day 1
├── Evidence from day 1 deleted by day 8
├── Continue attack after rotation
└── Investigators lose early attack data
```

---

## 4. Beacon Jitter and C2 Timing

```bash
# === C2 SLEEP AND JITTER ===
# Regular beaconing (DETECTABLE):
# Check in every 60 seconds exactly
# ├── 60s, 60s, 60s, 60s → Pattern detected!

# Jittered beaconing (STEALTHY):
# Check in every 60 seconds ± 50% jitter
# ├── 45s, 82s, 31s, 73s → No clear pattern!

# === COBALT STRIKE ===
# sleep 60 50
# 60 second interval, 50% jitter
# Range: 30-90 seconds

# === SLIVER C2 ===
# Interactive mode → no sleep (real-time)
# Session mode → configurable jitter

# === CUSTOM C2 TIMING ===
import random, time

def beacon_home():
    base_sleep = 300     # 5 minutes base
    jitter = 0.5         # 50% jitter
    
    while True:
        variation = random.uniform(1-jitter, 1+jitter)
        sleep_time = base_sleep * variation
        time.sleep(sleep_time)
        # 150-450 seconds between check-ins
        phone_home()

# === LONG HAUL C2 ===
# For extended operations:
# ├── Check in every 4-8 hours
# ├── Only during business hours
# ├── Mimic user behavior patterns
# ├── Sleep overnight (no weekend traffic)
# └── Wake up Monday at 8 AM
```

---

## 5. Time-Based Evasion Checklist

| Technique | Detection It Evades |
|-----------|-------------------|
| **Slow scanning (-T1)** | Port scan detection thresholds |
| **Random host order** | Sequential scan pattern detection |
| **Business hours** | Unusual time-of-day alerts |
| **Below lockout threshold** | Account lockout policies |
| **Jittered beaconing** | Regular interval C2 detection |
| **Distributed over days** | Single-day anomaly detection |
| **Log rotation timing** | Forensic investigation |
| **Maintenance window** | Reduced monitoring during changes |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Slow scan** | Below rate-based detection thresholds |
| **Jitter** | Randomize timing to avoid patterns |
| **Business hours** | Blend with normal traffic volume |
| **Threshold evasion** | Stay below lockout/alert limits |
| **Distributed timing** | Spread attack across days/weeks |
| **Long haul C2** | Hours between check-ins for stealth |

---

## Quick Revision Questions

1. **Why does slow scanning evade IDS detection?**
2. **What is beacon jitter and why is it important?**
3. **When is the best time to perform stealthy scanning?**
4. **How do you stay below account lockout thresholds?**
5. **What is long haul C2 timing?**

---

[← Previous: Protocol Manipulation](04-protocol-manipulation.md)

---

*Unit 10 - Topic 5 of 5 | [Back to README](../README.md)*

---

**🎉 Congratulations! You've completed Subject 3.1: Network Penetration Testing!**

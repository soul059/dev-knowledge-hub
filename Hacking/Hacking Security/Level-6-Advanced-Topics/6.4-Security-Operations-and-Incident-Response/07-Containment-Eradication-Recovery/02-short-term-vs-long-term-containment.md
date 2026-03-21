# Unit 7: Containment, Eradication, Recovery — Topic 2: Short-Term vs Long-Term Containment

## Overview

Containment operates in two phases: **short-term** (immediate threat neutralization) and **long-term** (sustained containment while eradication is planned and executed). Understanding the distinction ensures rapid response without compromising thorough remediation.

---

## 1. Short-Term Containment

```
SHORT-TERM CONTAINMENT:

  PURPOSE:
  → Stop immediate damage/spread
  → Executed within minutes to hours
  → Temporary measures
  → Preserve evidence for investigation
  → Buy time for planning

  CHARACTERISTICS:
  → Quick to implement
  → May not be sustainable
  → Minimal planning needed
  → Pre-authorized actions
  → Can be reversed if needed

SHORT-TERM ACTIONS:
  ┌──────────────────────────────────────────┐
  │ IMMEDIATE (Minutes)                      │
  │                                          │
  │ → Isolate compromised host via EDR       │
  │ → Disable compromised user account       │
  │ → Block C2 IP at firewall               │
  │ → Kill malicious process                 │
  │ → Quarantine malware file               │
  │ → Revoke user sessions/tokens           │
  │ → Block malicious email sender           │
  │                                          │
  │ QUICK FOLLOW-UP (Hours)                  │
  │                                          │
  │ → DNS sinkhole for C2 domains           │
  │ → Block IOCs across security stack       │
  │ → Enhanced monitoring on scope           │
  │ → Notify key stakeholders               │
  │ → Preserve volatile evidence             │
  └──────────────────────────────────────────┘

EXAMPLE - RANSOMWARE SHORT-TERM:
  Minute 0:  Alert: Ransomware detected on HOST01
  Minute 2:  EDR isolate HOST01
  Minute 5:  Disable affected user account
  Minute 10: Block C2 IPs at firewall
  Minute 15: Block lateral movement ports (445, 135)
             for affected subnet
  Minute 20: Alert all IT to disconnect backup
             systems from network
  Minute 30: Enhanced monitoring active
  Status: Immediate spread stopped
```

---

## 2. Long-Term Containment

```
LONG-TERM CONTAINMENT:

  PURPOSE:
  → Sustain containment during investigation
  → Enable safe continued operations
  → Provide time for eradication planning
  → Can last days, weeks, or months

  CHARACTERISTICS:
  → More planned and sustainable
  → Enables business continuity
  → May involve temporary infrastructure
  → Requires ongoing monitoring
  → Bridges to eradication

LONG-TERM ACTIONS:
  ┌──────────────────────────────────────────┐
  │ SUSTAINED CONTAINMENT                    │
  │                                          │
  │ → Set up clean VLAN for rebuilds         │
  │ → Implement temporary firewall rules     │
  │ → Deploy additional monitoring           │
  │ → Enable enhanced logging                │
  │ → Create clean admin accounts            │
  │ → Establish trusted communication        │
  │   channel (out-of-band)                  │
  │ → Deploy temporary workarounds           │
  │ → Restrict high-risk activities          │
  │                                          │
  │ BUSINESS CONTINUITY                      │
  │                                          │
  │ → Stand up replacement systems           │
  │ → Redirect users to clean systems        │
  │ → Enable alternative workflows           │
  │ → Provide temporary access methods       │
  │ → Communicate status to users            │
  └──────────────────────────────────────────┘

EXAMPLE - APT LONG-TERM CONTAINMENT:
  Day 1:    Short-term contain known compromised
            systems
  Day 2-3:  Build clean network segment
  Day 4-5:  Deploy enhanced monitoring across
            entire environment
  Day 5-7:  Migrate critical services to clean
            segment
  Day 7-14: Continue investigation while business
            operates on clean infrastructure
  Day 14+:  Begin eradication when full scope
            is determined
```

---

## 3. Comparison

```
SHORT-TERM vs LONG-TERM:

  Aspect          | Short-Term      | Long-Term
  Timing          | Minutes-hours   | Days-weeks
  Planning        | Minimal/pre-auth| Detailed planning
  Sustainability  | Temporary       | Sustained
  Business impact | Higher          | Managed
  Evidence        | Must preserve   | Enables collection
  Reversibility   | Easy to reverse | More permanent
  Scope           | Specific targets| Environment-wide
  Monitoring      | Immediate checks| Sustained monitoring
  Authority       | SOC/IR lead     | Management approval

DECISION MATRIX:
  Scenario                   | Short-Term        | Long-Term
  Active ransomware          | Isolate + block    | Clean build
  APT presence               | Monitor + restrict | New infra
  Account compromise         | Disable account    | MFA + PAM
  Web shell                  | Block traffic      | App rebuild
  Phishing campaign          | Block sender/URLs  | Email policy
  Data exfiltration          | Block exfil path   | DLP + segment
```

---

## 4. Transitioning Between Phases

```
TRANSITION PLANNING:

  SHORT-TERM ──────▶ LONG-TERM ──────▶ ERADICATION

  TRANSITION CHECKLIST:
  [ ] Short-term containment verified effective
  [ ] Evidence preserved from volatile sources
  [ ] Investigation scope determined (initial)
  [ ] Business impact assessed
  [ ] Long-term plan developed
  [ ] Resources allocated
  [ ] Management approval obtained
  [ ] Communication plan established
  [ ] Clean infrastructure planned
  [ ] Monitoring enhanced across environment

MAINTAINING CONTAINMENT:
  → Regular verification that containment holds
  → Monitor for attacker attempting to break out
  → Watch for new C2 channels
  → Check for persistence mechanisms activating
  → Validate firewall rules remain in place
  → Ensure isolated hosts stay isolated
  → Monitor for attacker lateral movement attempts

CONTAINMENT MONITORING:
  # Alert on any traffic from contained hosts
  index=firewall src_ip IN (10.0.1.50, 10.0.1.51)
    action=allowed
  | stats count by dest_ip, dest_port
  
  # Check for new C2 beaconing patterns
  index=proxy
  | bin _time span=60s
  | stats count by src_ip, dest_host
  | where count > 10
  | eventstats dc(dest_host) as unique_dests by src_ip
  | where unique_dests < 3
```

---

## 5. Common Mistakes

```
CONTAINMENT PITFALLS:

  ✗ Acting too slowly
    → Every minute counts with active threats
    → Pre-authorize common containment actions

  ✗ Containment too aggressive
    → Shutting down everything unnecessarily
    → Consider business impact before acting

  ✗ Not verifying containment
    → Assuming actions worked without checking
    → Always verify C2 is stopped

  ✗ Tipping off the attacker
    → Attacker notices and deploys backup C2
    → Use coordinated containment for APT

  ✗ Skipping evidence preservation
    → Wiping systems before collecting evidence
    → Always image/collect before eradication

  ✗ Incomplete containment
    → Blocking one C2 but missing others
    → Search for all attacker infrastructure

  ✗ No monitoring after containment
    → Assuming containment = done
    → Monitor continuously during containment

  ✗ Jumping to eradication too early
    → Rebuilding before understanding full scope
    → Attacker re-compromises from unknown access
```

---

## Summary Table

| Phase | Timing | Actions | Goal |
|-------|--------|---------|------|
| Short-term | Minutes-hours | Isolate, block, disable | Stop immediate damage |
| Long-term | Days-weeks | Clean segment, enhanced monitoring | Sustain operations |
| Transition | Planned | Verify, plan, resource | Move to eradication |

---

## Revision Questions

1. What distinguishes short-term from long-term containment?
2. What are typical short-term containment actions for a ransomware incident?
3. Why is long-term containment important for APT incidents?
4. What must be verified when transitioning between containment phases?
5. What common mistakes should be avoided during containment?

---

*Previous: [01-containment-strategies.md](01-containment-strategies.md) | Next: [03-eradication-planning.md](03-eradication-planning.md)*

---

*[Back to README](../README.md)*

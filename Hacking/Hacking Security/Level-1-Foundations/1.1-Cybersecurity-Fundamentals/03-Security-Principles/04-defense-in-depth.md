# Defense in Depth (Principle)

## Unit 3 - Topic 4 | Security Principles

---

## Overview

**Defense in Depth** as a security principle means implementing multiple, overlapping layers of security controls so that if one control fails, others continue to protect. This topic focuses on the principle itself and how it guides security decision-making (the detailed technical implementation was covered in Unit 1, Topic 4).

---

## 1. The Principle

```
┌──────────────────────────────────────────────────────────────────┐
│                DEFENSE IN DEPTH AS A PRINCIPLE                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Never rely on a SINGLE security control."                      │
│                                                                  │
│  Single control:       Multiple controls:                        │
│                                                                  │
│  [Firewall] → ❌ Fail   [Firewall] → ❌ Fail                     │
│       ↓                      ↓                                   │
│  💀 Compromised         [IDS/IPS] → ❌ Fail                      │
│                              ↓                                   │
│                         [Segmentation] → ✅ Blocked!              │
│                              ↓                                   │
│                         [EDR] → ✅ Detected!                      │
│                              ↓                                   │
│                         🛡️ Protected                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Three Dimensions of Defense in Depth

| Dimension | Description | Examples |
|-----------|-------------|---------|
| **People** | Training, awareness, security culture | Phishing simulations, security champions |
| **Technology** | Technical controls at every layer | Firewalls, EDR, encryption, SIEM |
| **Operations** | Processes and procedures | Incident response, change management, auditing |

```
                    ┌──────────────┐
                    │   DEFENSE    │
                    │   IN DEPTH   │
                    └──────┬───────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
     ┌──────┴──────┐ ┌────┴────┐ ┌──────┴──────┐
     │   PEOPLE    │ │TECHNOLOGY│ │ OPERATIONS  │
     │             │ │          │ │             │
     │ • Training  │ │• Firewall│ │• IR plans   │
     │ • Awareness │ │• EDR     │ │• Auditing   │
     │ • Culture   │ │• Encrypt │ │• Patching   │
     │ • Policy    │ │• MFA     │ │• Monitoring │
     └─────────────┘ └──────────┘ └─────────────┘
```

---

## 3. Applying the Principle

### Decision Framework

When designing security for any system, ask:

1. **What am I protecting?** (Asset identification)
2. **What are the threats?** (Threat modeling)
3. **What controls exist?** (Current state)
4. **What if control X fails?** (Failure analysis)
5. **What additional layers can I add?** (Layered defense)
6. **Are my layers diverse?** (Different technologies/vendors)

### Diversity of Controls

```
┌──────────────────────────────────────────────────────────────┐
│               CONTROL DIVERSITY                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  BAD: Same type of control repeated                          │
│  Firewall A → Firewall B → Firewall C                        │
│  (Same vulnerability in all three = single point of failure) │
│                                                              │
│  GOOD: Different types of controls layered                   │
│  Firewall → IDS → Network Segmentation → EDR → Encryption   │
│  (Diverse controls = much harder to bypass ALL)              │
│                                                              │
│  ALSO GOOD: Different vendor products                        │
│  Palo Alto FW → Suricata IDS → CrowdStrike EDR              │
│  (Vulnerability in one vendor doesn't affect others)         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Anti-Patterns (What NOT to Do)

| Anti-Pattern | Problem |
|-------------|---------|
| **Perimeter-only security** | Once breached, everything is exposed |
| **Single vendor everything** | Vendor vulnerability = total compromise |
| **Same control repeated** | No diversity = same weakness repeated |
| **Security theater** | Controls that look good but don't actually protect |
| **Neglecting people layer** | Technology alone can't stop social engineering |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Core Principle** | Never rely on a single security control |
| **Three Dimensions** | People + Technology + Operations |
| **Diversity** | Use different types of controls and vendors |
| **Key Question** | "What happens if THIS control fails?" |
| **Goal** | Multiple independent chances to detect and stop attacks |

---

## Quick Revision Questions

1. **Why should you never rely on a single security control?**
2. **What are the three dimensions of Defense in Depth?**
3. **Why is control diversity important?**
4. **What is the danger of perimeter-only security?**
5. **Give an example of Defense in Depth for protecting a web application.**

---

[← Previous: Separation of Duties](03-separation-of-duties.md) | [Next: Zero Trust Model →](05-zero-trust-model.md)

---

*Unit 3 - Topic 4 of 6 | [Back to README](../README.md)*

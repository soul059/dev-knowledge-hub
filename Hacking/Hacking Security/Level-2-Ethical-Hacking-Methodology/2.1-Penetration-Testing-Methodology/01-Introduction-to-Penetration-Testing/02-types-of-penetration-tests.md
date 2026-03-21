# Types of Penetration Tests

## Unit 1 - Topic 2 | Introduction to Penetration Testing

---

## Overview

Penetration tests are categorized based on the **amount of information** provided to the tester, the **target** being tested, and the **approach** used. Understanding these classifications helps organizations choose the right type of test for their needs.

---

## 1. By Knowledge Level

```
┌──────────────────────────────────────────────────────────────┐
│              TESTING KNOWLEDGE LEVELS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  BLACK BOX (Zero Knowledge)                                  │
│  ├── Tester has NO prior information                         │
│  ├── Simulates external attacker perspective                 │
│  ├── Most realistic attack simulation                        │
│  ├── Time-consuming (must discover everything)               │
│  └── May miss internal vulnerabilities                       │
│                                                              │
│  WHITE BOX (Full Knowledge)                                  │
│  ├── Tester has FULL access (source code, docs, creds)      │
│  ├── Most thorough assessment                                │
│  ├── Finds deep, complex vulnerabilities                     │
│  ├── Efficient use of time                                   │
│  └── Less realistic (attacker rarely has full info)          │
│                                                              │
│  GRAY BOX (Partial Knowledge)                                │
│  ├── Tester has SOME information (user creds, docs)         │
│  ├── Simulates insider threat or compromised user            │
│  ├── Balance of realism and thoroughness                     │
│  ├── Most common type in practice                            │
│  └── Efficient while maintaining realism                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

| Aspect | Black Box | Gray Box | White Box |
|--------|:---------:|:--------:|:---------:|
| **Knowledge** | None | Partial | Full |
| **Realism** | ★★★★★ | ★★★★ | ★★ |
| **Thoroughness** | ★★ | ★★★★ | ★★★★★ |
| **Time needed** | Most | Medium | Least |
| **Cost** | Highest | Medium | Lowest |
| **Simulates** | External attacker | Insider/user | Developer/admin |
| **Best for** | External assessment | Most engagements | Code review |

---

## 2. By Target

| Target Type | Description | Examples |
|-------------|-------------|---------|
| **Network** | Test network infrastructure | Firewalls, routers, switches |
| **Web Application** | Test web apps | OWASP Top 10, API testing |
| **Mobile Application** | Test mobile apps | Android/iOS apps |
| **Wireless** | Test wireless networks | Wi-Fi, Bluetooth |
| **Social Engineering** | Test human factor | Phishing, vishing, pretexting |
| **Physical** | Test physical security | Badge cloning, tailgating |
| **Cloud** | Test cloud environments | AWS, Azure, GCP configs |
| **IoT** | Test connected devices | Firmware, protocols |

---

## 3. By Approach

### External Penetration Test

```
EXTERNAL PEN TEST:
┌─────────────┐          ┌─────────────────────┐
│  Pen Tester  │ ───────► │  Internet-facing     │
│  (Internet)  │          │  • Web servers       │
│              │          │  • Email servers     │
│              │          │  • VPN gateways      │
│              │          │  • DNS servers       │
│              │          │  • Firewalls         │
└─────────────┘          └─────────────────────┘
Goal: Breach the perimeter from outside
```

### Internal Penetration Test

```
INTERNAL PEN TEST:
┌─────────────┐          ┌─────────────────────┐
│  Pen Tester  │          │  Internal Network    │
│  (Inside     │ ───────► │  • AD/Domain        │
│   network)   │          │  • File shares      │
│              │          │  • Databases        │
│              │          │  • Internal apps    │
└─────────────┘          └─────────────────────┘
Goal: Assess internal security; simulate insider threat
```

---

## 4. Specialized Test Types

| Type | Description |
|------|-------------|
| **Red Team** | Full-scope adversary simulation (stealthy, long-term) |
| **Blue Team** | Defensive testing — detect and respond to attacks |
| **Purple Team** | Red + Blue collaboration for maximum learning |
| **Assumed Breach** | Start from inside — skip initial access |
| **Compliance** | Test against specific standard (PCI-DSS) |
| **Crystal Box** | Interactive testing with developers present |

---

## 5. Choosing the Right Test

```
DECISION TREE:
│
├── Need realistic external attack simulation?
│   └── BLACK BOX EXTERNAL PEN TEST
│
├── Testing web application security?
│   └── GRAY BOX WEB APPLICATION PEN TEST
│
├── Testing source code + logic flaws?
│   └── WHITE BOX / CODE REVIEW
│
├── Testing employee security awareness?
│   └── SOCIAL ENGINEERING ASSESSMENT
│
├── Testing full attack chain (APT simulation)?
│   └── RED TEAM ENGAGEMENT
│
├── Regulatory requirement (PCI-DSS)?
│   └── COMPLIANCE PEN TEST
│
└── Testing incident response capability?
    └── PURPLE TEAM EXERCISE
```

---

## Summary Table

| Type | Knowledge | Best For |
|------|-----------|----------|
| **Black Box** | None | Realistic external assessment |
| **Gray Box** | Partial | Most common, balanced approach |
| **White Box** | Full | Thorough code-level analysis |
| **External** | Varies | Perimeter security testing |
| **Internal** | Varies | Insider threat simulation |
| **Red Team** | None | Full adversary simulation |

---

## Quick Revision Questions

1. **What are the 3 knowledge levels for pen tests?**
2. **Which test type is most common in practice?**
3. **What does a black box tester simulate?**
4. **When would you choose a white box test?**
5. **What is the difference between a pen test and a red team engagement?**

---

[← Previous: What is Penetration Testing?](01-what-is-penetration-testing.md) | [Next: Pen Testing vs Vulnerability Assessment →](03-pentest-vs-vulnerability-assessment.md)

---

*Unit 1 - Topic 2 of 5 | [Back to README](../README.md)*

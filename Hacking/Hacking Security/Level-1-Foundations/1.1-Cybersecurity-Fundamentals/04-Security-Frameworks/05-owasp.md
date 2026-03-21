# OWASP

## Unit 4 - Topic 5 | Security Frameworks

---

## Overview

**OWASP (Open Worldwide Application Security Project)** is a nonprofit organization dedicated to improving software security. OWASP provides freely available tools, standards, and documentation — most famously the **OWASP Top 10**, which lists the most critical web application security risks.

---

## 1. What is OWASP?

```
┌──────────────────────────────────────────────────────────────────┐
│                         OWASP                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  • Open-source, community-driven                                 │
│  • Free resources for everyone                                   │
│  • Vendor-neutral                                                │
│  • Global chapters and conferences                               │
│                                                                  │
│  Key Projects:                                                   │
│  ├── OWASP Top 10 (Web)                                          │
│  ├── OWASP Mobile Top 10                                         │
│  ├── OWASP API Security Top 10                                   │
│  ├── OWASP Testing Guide                                         │
│  ├── OWASP ASVS (Application Security Verification Standard)     │
│  ├── OWASP SAMM (Software Assurance Maturity Model)              │
│  ├── OWASP ZAP (Zed Attack Proxy)                                │
│  └── OWASP Cheat Sheets                                         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. OWASP Top 10 (2021)

| Rank | Category | Description |
|------|----------|-------------|
| **A01** | Broken Access Control | Users acting outside their permissions |
| **A02** | Cryptographic Failures | Weak encryption, exposed sensitive data |
| **A03** | Injection | SQL, NoSQL, OS command injection |
| **A04** | Insecure Design | Missing security controls in design phase |
| **A05** | Security Misconfiguration | Default configs, unnecessary features, verbose errors |
| **A06** | Vulnerable Components | Using components with known vulnerabilities |
| **A07** | Authentication Failures | Broken authentication, weak session management |
| **A08** | Software & Data Integrity Failures | Unsigned updates, insecure CI/CD, deserialization |
| **A09** | Security Logging & Monitoring Failures | Insufficient logging, no alerting |
| **A10** | Server-Side Request Forgery (SSRF) | Server makes requests to unintended locations |

---

## 3. Key OWASP Projects for Security Professionals

| Project | Purpose | Audience |
|---------|---------|----------|
| **Top 10** | Awareness of critical web risks | Everyone |
| **Testing Guide** | How to test web applications for vulnerabilities | Pen testers |
| **ASVS** | Security requirements for development | Developers, architects |
| **SAMM** | Assess and improve software security practices | Management |
| **ZAP** | Free web application security scanner | Pen testers, developers |
| **Cheat Sheets** | Quick reference for secure coding | Developers |
| **Juice Shop** | Intentionally vulnerable app for practice | Learners |
| **ModSecurity CRS** | Web Application Firewall rules | Defenders |

---

## 4. OWASP in the Security Workflow

```
┌──────────────────────────────────────────────────────────────┐
│          OWASP IN THE DEVELOPMENT LIFECYCLE                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  REQUIREMENTS ──► Use ASVS for security requirements         │
│       │                                                      │
│       ▼                                                      │
│  DESIGN ─────────► Use Top 10 for threat awareness           │
│       │             Use Cheat Sheets for secure patterns      │
│       ▼                                                      │
│  DEVELOPMENT ───► Follow Cheat Sheets for secure coding      │
│       │                                                      │
│       ▼                                                      │
│  TESTING ────────► Use Testing Guide methodology             │
│       │             Use ZAP for automated scanning            │
│       ▼                                                      │
│  DEPLOYMENT ───► Use Top 10 as deployment checklist          │
│       │                                                      │
│       ▼                                                      │
│  MONITORING ──► Use Logging & Monitoring guidance (A09)      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **What** | Nonprofit focused on application security |
| **Top 10** | 10 most critical web application security risks |
| **Key Tool** | OWASP ZAP (free web app scanner) |
| **For Developers** | ASVS, Cheat Sheets, Secure Coding practices |
| **For Testers** | Testing Guide, ZAP, Juice Shop |
| **Cost** | All resources are free and open-source |

---

## Quick Revision Questions

1. **What does OWASP stand for?**
2. **List the OWASP Top 10 (2021) categories.**
3. **What is the OWASP ASVS and who is it for?**
4. **Name 3 OWASP tools and their purposes.**
5. **How would a developer use OWASP resources during coding?**
6. **What is OWASP Juice Shop and what is it used for?**

---

[← Previous: MITRE ATT&CK](04-mitre-attack.md) | [Next: Compliance Frameworks →](06-compliance-frameworks.md)

---

*Unit 4 - Topic 5 of 6 | [Back to README](../README.md)*

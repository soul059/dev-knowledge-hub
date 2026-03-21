# Security by Design

## Unit 3 - Topic 6 | Security Principles

---

## Overview

**Security by Design** (also called **Secure by Design**) is the principle that security should be built into systems from the very beginning — not bolted on as an afterthought. It means considering security at every stage of the development lifecycle, from requirements gathering to deployment and maintenance.

---

## 1. Core Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                   SECURITY BY DESIGN                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ❌ WRONG APPROACH (Bolt-on Security):                            │
│                                                                  │
│  Design → Build → Test → Deploy → "Oh wait, add security" → 💸  │
│                                           (Expensive, incomplete)│
│                                                                  │
│  ✅ RIGHT APPROACH (Built-in Security):                           │
│                                                                  │
│  [Security] embedded at EVERY stage:                             │
│  Requirements → Design → Build → Test → Deploy → Maintain       │
│       🔒           🔒        🔒      🔒       🔒        🔒       │
│                                                                  │
│  "Security is not a feature — it's a property of the system."   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Cost of Fixing Security

```
Cost to Fix
     ▲
     │                                          ┌───┐
     │                                          │   │ $30,000
     │                                    ┌───┐ │   │
     │                                    │   │ │   │
     │                              ┌───┐ │   │ │   │
     │                              │   │ │   │ │   │
     │                        ┌───┐ │   │ │   │ │   │
     │                  ┌───┐ │   │ │   │ │   │ │   │
     │            ┌───┐ │   │ │   │ │   │ │   │ │   │
     │      ┌───┐ │$50│ │   │ │   │ │   │ │   │ │   │
     │      │$10│ │   │ │   │ │   │ │   │ │   │ │   │
     └──────┴───┴─┴───┴─┴───┴─┴───┴─┴───┴─┴───┴─┴───┴───►
           Req   Design  Code   Test  Release  Post-
                                              Release

     The LATER you find a bug, the MORE EXPENSIVE it is to fix.
     Fix during requirements = $10. Fix in production = $30,000.
```

---

## 3. Security by Design Principles

| Principle | Description |
|-----------|-------------|
| **Minimize attack surface** | Expose only what's necessary; disable unused features |
| **Establish secure defaults** | Out-of-the-box configuration should be secure |
| **Least privilege** | Grant minimum permissions by default |
| **Defense in depth** | Multiple layers of security controls |
| **Fail securely** | When something fails, fail to a secure state |
| **Don't trust services** | Validate all external inputs and services |
| **Separation of duties** | Critical functions require multiple parties |
| **Avoid security by obscurity** | Security shouldn't depend on secrecy of design |
| **Keep security simple** | Complex security = more bugs = more vulnerabilities |
| **Fix issues correctly** | Address root cause, not just symptoms |

### Secure Defaults Example

```
┌──────────────────────────────────────────────────────────────┐
│                   SECURE DEFAULTS                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  INSECURE DEFAULT ❌          SECURE DEFAULT ✅               │
│  ────────────────────         ──────────────────             │
│  Debug mode: ON               Debug mode: OFF                │
│  Admin password: "admin"      Admin: force password set      │
│  All ports open               Only needed ports open         │
│  HTTP enabled                 HTTPS enforced                 │
│  Directory listing ON         Directory listing OFF          │
│  Error details shown          Generic error messages         │
│  All features enabled         Minimal features enabled       │
│                                                              │
│  "The default configuration should be the MOST SECURE."     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Secure Development Lifecycle (SDL)

```
┌──────────────────────────────────────────────────────────────────┐
│              SECURE DEVELOPMENT LIFECYCLE                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────┐                                                  │
│  │  TRAINING  │ Security awareness for all developers            │
│  └─────┬──────┘                                                  │
│        │                                                         │
│        ▼                                                         │
│  ┌────────────┐                                                  │
│  │REQUIREMENTS│ Security requirements, abuse cases               │
│  └─────┬──────┘                                                  │
│        ▼                                                         │
│  ┌────────────┐                                                  │
│  │   DESIGN   │ Threat modeling, security architecture           │
│  └─────┬──────┘                                                  │
│        ▼                                                         │
│  ┌────────────┐                                                  │
│  │   BUILD    │ Secure coding standards, SAST, code review       │
│  └─────┬──────┘                                                  │
│        ▼                                                         │
│  ┌────────────┐                                                  │
│  │   TEST     │ DAST, penetration testing, fuzzing               │
│  └─────┬──────┘                                                  │
│        ▼                                                         │
│  ┌────────────┐                                                  │
│  │  RELEASE   │ Final security review, incident response plan    │
│  └─────┬──────┘                                                  │
│        ▼                                                         │
│  ┌────────────┐                                                  │
│  │  RESPOND   │ Monitor, patch, vulnerability management         │
│  └────────────┘                                                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. Fail Securely

```
┌──────────────────────────────────────────────────────────────┐
│                   FAIL SECURELY                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FAIL OPEN ❌ (Insecure)        FAIL CLOSED ✅ (Secure)      │
│  ───────────────────────        ────────────────────────     │
│                                                              │
│  Firewall crashes →             Firewall crashes →           │
│  All traffic ALLOWED            All traffic BLOCKED          │
│                                                              │
│  Auth service down →            Auth service down →          │
│  Everyone gets access           No one gets access           │
│                                                              │
│  Error in validation →          Error in validation →        │
│  Input accepted                 Input rejected               │
│                                                              │
│  PRINCIPLE: When in doubt, DENY access.                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6. Security by Design in Practice

| Practice | Description | Tools |
|----------|-------------|-------|
| **Threat Modeling** | Identify threats during design | STRIDE, PASTA, DREAD |
| **Secure Code Review** | Review code for vulnerabilities | SonarQube, Checkmarx |
| **SAST** | Static Application Security Testing | Semgrep, Fortify |
| **DAST** | Dynamic Application Security Testing | OWASP ZAP, Burp Suite |
| **SCA** | Software Composition Analysis | Snyk, Dependabot |
| **IaC Scanning** | Scan infrastructure code | Checkov, tfsec |
| **Security Testing** | Penetration testing, fuzzing | Manual + automated |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Core Idea** | Build security in from the start, not bolt it on later |
| **Cost Principle** | Earlier you catch it, cheaper to fix |
| **Key Practices** | Secure defaults, fail securely, minimize attack surface |
| **SDL** | Security integrated at every SDLC phase |
| **Fail Securely** | Deny by default when failures occur |
| **Not Obscurity** | Security should work even if design is known |

---

## Quick Revision Questions

1. **Why is bolting security on after development a bad approach?**
2. **What does "secure by default" mean? Give 3 examples.**
3. **Explain the difference between "fail open" and "fail closed."**
4. **What are the phases of a Secure Development Lifecycle?**
5. **Why should security NOT rely on obscurity?**
6. **What is threat modeling and when should it be performed?**

---

[← Previous: Zero Trust Model](05-zero-trust-model.md)

---

*Unit 3 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Security Frameworks →](../04-Security-Frameworks/01-nist-framework.md)*

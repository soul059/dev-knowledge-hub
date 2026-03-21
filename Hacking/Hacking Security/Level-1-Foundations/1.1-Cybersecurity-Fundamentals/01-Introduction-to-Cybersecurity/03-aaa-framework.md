# AAA Framework

## Unit 1 - Topic 3 | Introduction to Cybersecurity

---

## Overview

The **AAA Framework** (Authentication, Authorization, and Accounting) is a security architecture model used to control who can access resources, what they can do, and track what they did. It is the backbone of access control in modern networks and systems.

---

## 1. The AAA Model

```
┌─────────────────────────────────────────────────────────────────┐
│                       AAA FRAMEWORK                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Step 1              Step 2                Step 3              │
│  ┌──────────┐      ┌──────────────┐      ┌──────────────┐      │
│  │  AUTHEN- │─────►│  AUTHORI-    │─────►│  ACCOUNTING  │      │
│  │  TICATION│      │  ZATION      │      │              │      │
│  │          │      │              │      │              │      │
│  │ "Who are │      │ "What can    │      │ "What did    │      │
│  │  you?"   │      │  you do?"    │      │  you do?"    │      │
│  └──────────┘      └──────────────┘      └──────────────┘      │
│                                                                 │
│   Verify Identity   Grant Permissions     Log Activities        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Authentication — "Who Are You?"

### Definition

> **Authentication** is the process of verifying the identity of a user, device, or system. It answers the question: "Are you really who you claim to be?"

### Authentication Factors

```
┌──────────────────────────────────────────────────────────────┐
│                   AUTHENTICATION FACTORS                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  SOMETHING YOU KNOW          (Knowledge Factor)      │    │
│  │  • Password / PIN / Passphrase                       │    │
│  │  • Security questions                                │    │
│  │  • Pattern lock                                      │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  SOMETHING YOU HAVE          (Possession Factor)     │    │
│  │  • Smart card / Token                                │    │
│  │  • Mobile phone (OTP)                                │    │
│  │  • USB security key (YubiKey)                        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  SOMETHING YOU ARE           (Inherence Factor)      │    │
│  │  • Fingerprint / Face scan                           │    │
│  │  • Retina / Iris scan                                │    │
│  │  • Voice recognition                                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  SOMEWHERE YOU ARE           (Location Factor)       │    │
│  │  • GPS location / IP geolocation                     │    │
│  │  • Network-based location                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  SOMETHING YOU DO            (Behavior Factor)       │    │
│  │  • Typing patterns (keystroke dynamics)              │    │
│  │  • Gait analysis / Mouse movement                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Multi-Factor Authentication (MFA)

MFA combines **two or more different factor types** for stronger security:

```
Single-Factor:    Password only              ──► WEAK
                                                  
Two-Factor (2FA): Password + OTP Code        ──► STRONG
                   (Know)     (Have)

Multi-Factor:     Password + OTP + Fingerprint ──► VERY STRONG
                   (Know)    (Have)  (Are)

NOTE: Two passwords = still single-factor (both are "something you know")
```

### Authentication Protocols

| Protocol | Description | Use Case |
|----------|-------------|----------|
| **RADIUS** | Remote Authentication Dial-In User Service | Network access (Wi-Fi, VPN) |
| **TACACS+** | Terminal Access Controller Access-Control System | Cisco device administration |
| **LDAP** | Lightweight Directory Access Protocol | Directory services (Active Directory) |
| **Kerberos** | Ticket-based authentication | Windows domain authentication |
| **OAuth 2.0** | Token-based delegated authorization | Web/API access |
| **SAML** | Security Assertion Markup Language | Enterprise SSO |
| **OpenID Connect** | Identity layer on top of OAuth 2.0 | Consumer SSO (Google/Facebook login) |

---

## 3. Authorization — "What Can You Do?"

### Definition

> **Authorization** determines what resources an authenticated user can access and what actions they can perform. It enforces permissions and access rights.

### Authorization Models

```
┌──────────────────────────────────────────────────────────────┐
│                   AUTHORIZATION MODELS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DAC (Discretionary Access Control)                          │
│  ┌─────────────────────────────────────────────────┐        │
│  │  Owner decides who gets access                   │        │
│  │  Example: File permissions in Windows/Linux      │        │
│  │  Flexible but less secure                        │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
│  MAC (Mandatory Access Control)                              │
│  ┌─────────────────────────────────────────────────┐        │
│  │  System enforces access based on labels          │        │
│  │  Example: SELinux, Military classifications      │        │
│  │  Very secure but less flexible                   │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
│  RBAC (Role-Based Access Control)                            │
│  ┌─────────────────────────────────────────────────┐        │
│  │  Access based on user's role in organization     │        │
│  │  Example: Admin, Manager, Employee roles         │        │
│  │  Most commonly used in enterprises               │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
│  ABAC (Attribute-Based Access Control)                       │
│  ┌─────────────────────────────────────────────────┐        │
│  │  Access based on attributes (user, resource, env)│        │
│  │  Example: "Allow if department=HR AND time=9-5"  │        │
│  │  Most granular and flexible                      │        │
│  └─────────────────────────────────────────────────┘        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### RBAC Example

```
┌─────────────────────────────────────────────────────────────┐
│                    RBAC IN ACTION                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  User: Alice ──► Role: Admin ──► Permissions:               │
│                                   • Read all files          │
│                                   • Write all files         │
│                                   • Delete files            │
│                                   • Manage users            │
│                                                             │
│  User: Bob ────► Role: Editor ──► Permissions:              │
│                                   • Read all files          │
│                                   • Write own files         │
│                                                             │
│  User: Carol ──► Role: Viewer ──► Permissions:              │
│                                   • Read public files       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Authorization Comparison Table

| Model | Who Decides | Flexibility | Security | Best For |
|-------|------------|-------------|----------|----------|
| **DAC** | Resource owner | High | Medium | Personal systems |
| **MAC** | System/Policy | Low | Very High | Military/Government |
| **RBAC** | Administrator | Medium | High | Enterprise |
| **ABAC** | Policy engine | Very High | High | Complex environments |

---

## 4. Accounting — "What Did You Do?"

### Definition

> **Accounting** (also called **Auditing**) is the process of tracking and logging user activities for review, compliance, and forensic purposes.

### What Gets Logged

```
┌──────────────────────────────────────────────────────────────┐
│                    ACCOUNTING RECORDS                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  WHO  ──► User identity (username, IP address)       │    │
│  │  WHAT ──► Action performed (login, read, modify)     │    │
│  │  WHEN ──► Timestamp (date, time, timezone)           │    │
│  │  WHERE──► System/resource accessed (server, file)    │    │
│  │  HOW  ──► Method of access (SSH, web, console)       │    │
│  │  RESULT─► Success or failure of the action           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Accounting Use Cases

| Use Case | Description |
|----------|-------------|
| **Security Monitoring** | Detect suspicious login attempts, privilege abuse |
| **Compliance Auditing** | Prove adherence to regulations (HIPAA, PCI-DSS) |
| **Forensic Investigation** | Reconstruct events after a security incident |
| **Billing/Usage** | Track resource consumption (ISP, cloud services) |
| **User Behavior Analytics** | Detect anomalous patterns indicating insider threats |
| **Troubleshooting** | Diagnose system issues using activity logs |

---

## 5. AAA in Action — Complete Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                        AAA COMPLETE FLOW                              │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  User           AAA Server          Network/Resource                 │
│   │                 │                      │                         │
│   │ 1. Login Request│                      │                         │
│   │────────────────►│                      │                         │
│   │                 │                      │                         │
│   │ 2. Challenge    │                      │  AUTHENTICATION        │
│   │◄────────────────│                      │                         │
│   │                 │                      │                         │
│   │ 3. Credentials  │                      │                         │
│   │────────────────►│                      │                         │
│   │                 │                      │                         │
│   │ 4. Verify ✅    │                      │                         │
│   │◄────────────────│                      │                         │
│   │                 │                      │                         │
│   │ 5. Request      │ 6. Check Policy      │  AUTHORIZATION         │
│   │ Access ─────────│─────────────────────►│                         │
│   │                 │                      │                         │
│   │                 │ 7. Grant/Deny        │                         │
│   │◄────────────────│◄─────────────────────│                         │
│   │                 │                      │                         │
│   │ 8. Activity     │                      │  ACCOUNTING            │
│   │ Logged ─────────│─────────────────────►│                         │
│   │                 │                      │                         │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 6. RADIUS vs TACACS+

| Feature | RADIUS | TACACS+ |
|---------|--------|---------|
| **Protocol** | UDP (1812, 1813) | TCP (49) |
| **Encryption** | Password only | Entire payload |
| **AAA Separation** | Combined auth + authz | Separate AAA processes |
| **Vendor** | Open standard (RFC) | Cisco proprietary |
| **Best For** | Network access (Wi-Fi, VPN) | Device administration |
| **Granularity** | Limited command control | Per-command authorization |

---

## Summary Table

| AAA Component | Question | Purpose | Examples |
|---------------|----------|---------|----------|
| **Authentication** | Who are you? | Verify identity | Password, MFA, biometrics |
| **Authorization** | What can you do? | Grant permissions | RBAC, ACLs, policies |
| **Accounting** | What did you do? | Track activities | Logs, audit trails, SIEM |

---

## Quick Revision Questions

1. **What are the three components of the AAA framework?**
2. **List the five authentication factors with an example of each.**
3. **Why is using two passwords NOT considered multi-factor authentication?**
4. **Compare DAC, MAC, RBAC, and ABAC authorization models.**
5. **What is the difference between RADIUS and TACACS+?**
6. **Why is the Accounting component critical for incident response?**

---

[← Previous: CIA Triad](02-cia-triad.md) | [Next: Defense in Depth →](04-defense-in-depth.md)

---

*Unit 1 - Topic 3 of 6 | [Back to README](../README.md)*

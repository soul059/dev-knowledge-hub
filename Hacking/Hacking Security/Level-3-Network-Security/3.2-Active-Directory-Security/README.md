# 3.2 Active Directory Security

## Level 3 — Network Security

---

## 📋 Course Overview

**Active Directory (AD)** is the backbone of most enterprise networks, managing authentication, authorization, and access control for millions of organizations worldwide. This subject provides comprehensive coverage of AD attack techniques, defense strategies, and hardening best practices — essential knowledge for penetration testers and security professionals.

---

## 📖 Table of Contents

### Unit 1: Active Directory Fundamentals
| # | Topic | File |
|---|-------|------|
| 1 | AD Architecture | [01-ad-architecture.md](01-Active-Directory-Fundamentals/01-ad-architecture.md) |
| 2 | Domains, Forests, Trusts | [02-domains-forests-trusts.md](01-Active-Directory-Fundamentals/02-domains-forests-trusts.md) |
| 3 | Objects (Users, Groups, Computers) | [03-objects-users-groups-computers.md](01-Active-Directory-Fundamentals/03-objects-users-groups-computers.md) |
| 4 | Group Policy Basics | [04-group-policy-basics.md](01-Active-Directory-Fundamentals/04-group-policy-basics.md) |
| 5 | Authentication Protocols (NTLM, Kerberos) | [05-authentication-protocols.md](01-Active-Directory-Fundamentals/05-authentication-protocols.md) |

### Unit 2: AD Enumeration
| # | Topic | File |
|---|-------|------|
| 1 | AD Enumeration Techniques | [01-ad-enumeration-techniques.md](02-AD-Enumeration/01-ad-enumeration-techniques.md) |
| 2 | LDAP Queries | [02-ldap-queries.md](02-AD-Enumeration/02-ldap-queries.md) |
| 3 | PowerShell Enumeration | [03-powershell-enumeration.md](02-AD-Enumeration/03-powershell-enumeration.md) |
| 4 | BloodHound Basics | [04-bloodhound-basics.md](02-AD-Enumeration/04-bloodhound-basics.md) |
| 5 | User and Group Enumeration | [05-user-and-group-enumeration.md](02-AD-Enumeration/05-user-and-group-enumeration.md) |
| 6 | Trust Enumeration | [06-trust-enumeration.md](02-AD-Enumeration/06-trust-enumeration.md) |

### Unit 3: Initial Access Scenarios
| # | Topic | File |
|---|-------|------|
| 1 | Credential Harvesting | [01-credential-harvesting.md](03-Initial-Access-Scenarios/01-credential-harvesting.md) |
| 2 | Phishing for Credentials | [02-phishing-for-credentials.md](03-Initial-Access-Scenarios/02-phishing-for-credentials.md) |
| 3 | Password Spraying | [03-password-spraying.md](03-Initial-Access-Scenarios/03-password-spraying.md) |
| 4 | LLMNR/NBNS Poisoning | [04-llmnr-nbns-poisoning.md](03-Initial-Access-Scenarios/04-llmnr-nbns-poisoning.md) |
| 5 | AS-REP Roasting | [05-asrep-roasting.md](03-Initial-Access-Scenarios/05-asrep-roasting.md) |
| 6 | Initial Foothold Strategies | [06-initial-foothold-strategies.md](03-Initial-Access-Scenarios/06-initial-foothold-strategies.md) |

### Unit 4: Credential Attacks
| # | Topic | File |
|---|-------|------|
| 1 | NTLM Hash Extraction | [01-ntlm-hash-extraction.md](04-Credential-Attacks/01-ntlm-hash-extraction.md) |
| 2 | Pass-the-Hash | [02-pass-the-hash.md](04-Credential-Attacks/02-pass-the-hash.md) |
| 3 | Pass-the-Ticket | [03-pass-the-ticket.md](04-Credential-Attacks/03-pass-the-ticket.md) |
| 4 | Kerberoasting | [04-kerberoasting.md](04-Credential-Attacks/04-kerberoasting.md) |
| 5 | Silver and Golden Tickets | [05-silver-and-golden-tickets.md](04-Credential-Attacks/05-silver-and-golden-tickets.md) |
| 6 | DCSync Attack | [06-dcsync-attack.md](04-Credential-Attacks/06-dcsync-attack.md) |

### Unit 5: Privilege Escalation in AD
| # | Topic | File |
|---|-------|------|
| 1 | Local Privilege Escalation | [01-local-privilege-escalation.md](05-Privilege-Escalation-in-AD/01-local-privilege-escalation.md) |
| 2 | Token Impersonation | [02-token-impersonation.md](05-Privilege-Escalation-in-AD/02-token-impersonation.md) |
| 3 | Misconfigured Permissions | [03-misconfigured-permissions.md](05-Privilege-Escalation-in-AD/03-misconfigured-permissions.md) |
| 4 | Group Policy Exploitation | [04-group-policy-exploitation.md](05-Privilege-Escalation-in-AD/04-group-policy-exploitation.md) |
| 5 | Certificate Abuse | [05-certificate-abuse.md](05-Privilege-Escalation-in-AD/05-certificate-abuse.md) |

### Unit 6: Lateral Movement
| # | Topic | File |
|---|-------|------|
| 1 | Remote Execution Techniques | [01-remote-execution-techniques.md](06-Lateral-Movement/01-remote-execution-techniques.md) |
| 2 | WMI Exploitation | [02-wmi-exploitation.md](06-Lateral-Movement/02-wmi-exploitation.md) |
| 3 | PowerShell Remoting | [03-powershell-remoting.md](06-Lateral-Movement/03-powershell-remoting.md) |
| 4 | PsExec and Alternatives | [04-psexec-and-alternatives.md](06-Lateral-Movement/04-psexec-and-alternatives.md) |
| 5 | RDP Hijacking | [05-rdp-hijacking.md](06-Lateral-Movement/05-rdp-hijacking.md) |
| 6 | Overpass-the-Hash | [06-overpass-the-hash.md](06-Lateral-Movement/06-overpass-the-hash.md) |

### Unit 7: Persistence
| # | Topic | File |
|---|-------|------|
| 1 | User Account Persistence | [01-user-account-persistence.md](07-Persistence/01-user-account-persistence.md) |
| 2 | Golden Ticket Persistence | [02-golden-ticket-persistence.md](07-Persistence/02-golden-ticket-persistence.md) |
| 3 | Skeleton Key | [03-skeleton-key.md](07-Persistence/03-skeleton-key.md) |
| 4 | AdminSDHolder Abuse | [04-adminsdholder-abuse.md](07-Persistence/04-adminsdholder-abuse.md) |
| 5 | Group Policy Persistence | [05-group-policy-persistence.md](07-Persistence/05-group-policy-persistence.md) |
| 6 | DSRM Abuse | [06-dsrm-abuse.md](07-Persistence/06-dsrm-abuse.md) |

### Unit 8: Domain Dominance
| # | Topic | File |
|---|-------|------|
| 1 | Domain Admin Escalation | [01-domain-admin-escalation.md](08-Domain-Dominance/01-domain-admin-escalation.md) |
| 2 | Domain Controller Compromise | [02-domain-controller-compromise.md](08-Domain-Dominance/02-domain-controller-compromise.md) |
| 3 | Forest Compromise | [03-forest-compromise.md](08-Domain-Dominance/03-forest-compromise.md) |
| 4 | Trust Exploitation | [04-trust-exploitation.md](08-Domain-Dominance/04-trust-exploitation.md) |
| 5 | Cross-Forest Attacks | [05-cross-forest-attacks.md](08-Domain-Dominance/05-cross-forest-attacks.md) |

### Unit 9: AD Hardening
| # | Topic | File |
|---|-------|------|
| 1 | Tiered Administration Model | [01-tiered-administration-model.md](09-AD-Hardening/01-tiered-administration-model.md) |
| 2 | Protected Users Group | [02-protected-users-group.md](09-AD-Hardening/02-protected-users-group.md) |
| 3 | Credential Guard | [03-credential-guard.md](09-AD-Hardening/03-credential-guard.md) |
| 4 | LAPS (Local Admin Password Solution) | [04-laps.md](09-AD-Hardening/04-laps.md) |
| 5 | Privileged Access Workstations | [05-privileged-access-workstations.md](09-AD-Hardening/05-privileged-access-workstations.md) |
| 6 | AD Security Best Practices | [06-ad-security-best-practices.md](09-AD-Hardening/06-ad-security-best-practices.md) |

---

## 🎯 Learning Objectives

After completing this subject, you will be able to:

- Understand Active Directory architecture and authentication
- Enumerate AD environments using various tools
- Perform credential attacks (PtH, PtT, Kerberoasting, DCSync)
- Escalate privileges through AD misconfigurations
- Move laterally using remote execution techniques
- Establish persistence in compromised domains
- Achieve domain dominance and cross-forest access
- Implement AD hardening and defense measures

---

## 🔧 Key Tools

| Tool | Purpose |
|------|---------|
| **BloodHound** | AD attack path visualization |
| **Mimikatz** | Credential extraction and manipulation |
| **Impacket** | Python tools for AD attacks |
| **CrackMapExec** | AD network swiss army knife |
| **Rubeus** | Kerberos attack toolkit |
| **PowerView** | PowerShell AD enumeration |
| **Evil-WinRM** | WinRM shell for AD exploitation |
| **Certipy** | AD Certificate Services abuse |
| **SharpHound** | BloodHound data collector |
| **ADFind** | Command-line AD query tool |

---

## 📊 Subject Statistics

| Metric | Count |
|--------|:-----:|
| **Units** | 9 |
| **Topic Files** | 51 |
| **Estimated Study Time** | 40-50 hours |
| **Difficulty Level** | Advanced |
| **Prerequisites** | Networking, Windows, Pen Testing Methodology |

---

*Part of [Level 3: Network Security](../README.md) | [Previous Subject: 3.1 Network Penetration Testing](../3.1-Network-Penetration-Testing/README.md)*

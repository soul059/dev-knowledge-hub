# 🐧 Linux for Security Professionals

## Subject 1.3 | Level 1 - Foundations

---

## Course Overview

This subject provides comprehensive coverage of **Linux from a security perspective** — from fundamental architecture and permissions to advanced hardening, forensics, and container security. Linux powers the majority of servers, security tools, and attack platforms, making it essential knowledge for every cybersecurity professional.

---

## Table of Contents

### Unit 1: Linux Security Fundamentals
| # | Topic | File |
|---|-------|------|
| 1 | Linux Architecture Overview | [01-linux-architecture-overview.md](01-Linux-Security-Fundamentals/01-linux-architecture-overview.md) |
| 2 | Security-Relevant Directories | [02-security-relevant-directories.md](01-Linux-Security-Fundamentals/02-security-relevant-directories.md) |
| 3 | File Permissions (Standard, Special) | [03-file-permissions.md](01-Linux-Security-Fundamentals/03-file-permissions.md) |
| 4 | User and Group Management | [04-user-and-group-management.md](01-Linux-Security-Fundamentals/04-user-and-group-management.md) |
| 5 | sudo and Privilege Escalation Prevention | [05-sudo-and-privilege-escalation.md](01-Linux-Security-Fundamentals/05-sudo-and-privilege-escalation.md) |

### Unit 2: Linux Hardening
| # | Topic | File |
|---|-------|------|
| 1 | Minimal Installation | [01-minimal-installation.md](02-Linux-Hardening/01-minimal-installation.md) |
| 2 | Service Management | [02-service-management.md](02-Linux-Hardening/02-service-management.md) |
| 3 | Unnecessary Service Removal | [03-unnecessary-service-removal.md](02-Linux-Hardening/03-unnecessary-service-removal.md) |
| 4 | Kernel Hardening (sysctl) | [04-kernel-hardening.md](02-Linux-Hardening/04-kernel-hardening.md) |
| 5 | PAM Configuration | [05-pam-configuration.md](02-Linux-Hardening/05-pam-configuration.md) |
| 6 | SSH Hardening | [06-ssh-hardening.md](02-Linux-Hardening/06-ssh-hardening.md) |
| 7 | Firewall Configuration (iptables, nftables) | [07-firewall-configuration.md](02-Linux-Hardening/07-firewall-configuration.md) |

### Unit 3: Linux Logging and Auditing
| # | Topic | File |
|---|-------|------|
| 1 | Syslog and journald | [01-syslog-and-journald.md](03-Linux-Logging-and-Auditing/01-syslog-and-journald.md) |
| 2 | Important Log Files | [02-important-log-files.md](03-Linux-Logging-and-Auditing/02-important-log-files.md) |
| 3 | Auditd Configuration | [03-auditd-configuration.md](03-Linux-Logging-and-Auditing/03-auditd-configuration.md) |
| 4 | Log Analysis | [04-log-analysis.md](03-Linux-Logging-and-Auditing/04-log-analysis.md) |
| 5 | Log Forwarding | [05-log-forwarding.md](03-Linux-Logging-and-Auditing/05-log-forwarding.md) |
| 6 | SIEM Integration | [06-siem-integration.md](03-Linux-Logging-and-Auditing/06-siem-integration.md) |

### Unit 4: Linux File System Security
| # | Topic | File |
|---|-------|------|
| 1 | File System Hierarchy | [01-file-system-hierarchy.md](04-Linux-File-System-Security/01-file-system-hierarchy.md) |
| 2 | Mount Options for Security | [02-mount-options-for-security.md](04-Linux-File-System-Security/02-mount-options-for-security.md) |
| 3 | File Integrity Monitoring | [03-file-integrity-monitoring.md](04-Linux-File-System-Security/03-file-integrity-monitoring.md) |
| 4 | AIDE and Tripwire | [04-aide-and-tripwire.md](04-Linux-File-System-Security/04-aide-and-tripwire.md) |
| 5 | Immutable Files | [05-immutable-files.md](04-Linux-File-System-Security/05-immutable-files.md) |
| 6 | Disk Encryption (LUKS) | [06-disk-encryption-luks.md](04-Linux-File-System-Security/06-disk-encryption-luks.md) |

### Unit 5: Linux Access Control
| # | Topic | File |
|---|-------|------|
| 1 | Discretionary Access Control (DAC) | [01-discretionary-access-control.md](05-Linux-Access-Control/01-discretionary-access-control.md) |
| 2 | Mandatory Access Control (MAC) | [02-mandatory-access-control.md](05-Linux-Access-Control/02-mandatory-access-control.md) |
| 3 | SELinux Basics | [03-selinux-basics.md](05-Linux-Access-Control/03-selinux-basics.md) |
| 4 | AppArmor Basics | [04-apparmor-basics.md](05-Linux-Access-Control/04-apparmor-basics.md) |
| 5 | Linux Capabilities | [05-linux-capabilities.md](05-Linux-Access-Control/05-linux-capabilities.md) |
| 6 | Namespaces and cgroups | [06-namespaces-and-cgroups.md](05-Linux-Access-Control/06-namespaces-and-cgroups.md) |

### Unit 6: Linux Network Security
| # | Topic | File |
|---|-------|------|
| 1 | Network Configuration Files | [01-network-configuration-files.md](06-Linux-Network-Security/01-network-configuration-files.md) |
| 2 | Netfilter and iptables | [02-netfilter-and-iptables.md](06-Linux-Network-Security/02-netfilter-and-iptables.md) |
| 3 | TCP Wrappers | [03-tcp-wrappers.md](06-Linux-Network-Security/03-tcp-wrappers.md) |
| 4 | Network Services Security | [04-network-services-security.md](06-Linux-Network-Security/04-network-services-security.md) |
| 5 | Network Namespaces | [05-network-namespaces.md](06-Linux-Network-Security/05-network-namespaces.md) |

### Unit 7: Linux Security Tools
| # | Topic | File |
|---|-------|------|
| 1 | Network Scanning (nmap) | [01-network-scanning-nmap.md](07-Linux-Security-Tools/01-network-scanning-nmap.md) |
| 2 | Vulnerability Scanning | [02-vulnerability-scanning.md](07-Linux-Security-Tools/02-vulnerability-scanning.md) |
| 3 | Password Crackers | [03-password-crackers.md](07-Linux-Security-Tools/03-password-crackers.md) |
| 4 | Traffic Analysis | [04-traffic-analysis.md](07-Linux-Security-Tools/04-traffic-analysis.md) |
| 5 | Forensic Tools | [05-forensic-tools.md](07-Linux-Security-Tools/05-forensic-tools.md) |
| 6 | Security Distributions (Kali, Parrot) | [06-security-distributions.md](07-Linux-Security-Tools/06-security-distributions.md) |

### Unit 8: Bash Scripting for Security
| # | Topic | File |
|---|-------|------|
| 1 | Bash Basics Review | [01-bash-basics-review.md](08-Bash-Scripting-for-Security/01-bash-basics-review.md) |
| 2 | Security Automation Scripts | [02-security-automation-scripts.md](08-Bash-Scripting-for-Security/02-security-automation-scripts.md) |
| 3 | Log Parsing Scripts | [03-log-parsing-scripts.md](08-Bash-Scripting-for-Security/03-log-parsing-scripts.md) |
| 4 | Network Monitoring Scripts | [04-network-monitoring-scripts.md](08-Bash-Scripting-for-Security/04-network-monitoring-scripts.md) |
| 5 | Incident Response Scripts | [05-incident-response-scripts.md](08-Bash-Scripting-for-Security/05-incident-response-scripts.md) |

### Unit 9: Container Security
| # | Topic | File |
|---|-------|------|
| 1 | Docker Security Basics | [01-docker-security-basics.md](09-Container-Security/01-docker-security-basics.md) |
| 2 | Container Isolation | [02-container-isolation.md](09-Container-Security/02-container-isolation.md) |
| 3 | Image Security | [03-image-security.md](09-Container-Security/03-image-security.md) |
| 4 | Runtime Security | [04-runtime-security.md](09-Container-Security/04-runtime-security.md) |
| 5 | Kubernetes Security Basics | [05-kubernetes-security-basics.md](09-Container-Security/05-kubernetes-security-basics.md) |

---

## Subject Statistics

| Metric | Count |
|--------|:-----:|
| Units | 9 |
| Total Topics | 51 |
| Estimated Study Hours | 45-60 |

---

## Prerequisites

- Basic computer literacy
- [Subject 1.1: Cybersecurity Fundamentals](../1.1-Cybersecurity-Fundamentals/README.md)
- [Subject 1.2: Networking for Security Professionals](../1.2-Networking-for-Security/README.md)

---

*Subject 1.3 of Level 1 - Foundations | [Back to Level 1](../)*

# Linux for DevOps — Comprehensive Study Notes

```
  ╔══════════════════════════════════════════════════════════════╗
  ║              LINUX FOR DEVOPS ENGINEERS                      ║
  ║          From Fundamentals to System Administration          ║
  ╠══════════════════════════════════════════════════════════════╣
  ║   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   ║
  ║   │  Linux   │  │ Command  │  │  Users & │  │ Process  │   ║
  ║   │ Fundamtls│→ │  Line    │→ │  Perms   │→ │  Mgmt    │   ║
  ║   └──────────┘  └──────────┘  └──────────┘  └──────────┘   ║
  ║   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   ║
  ║   │Networking│  │ Storage  │  │  Shell   │  │  System  │   ║
  ║   │          │→ │  Mgmt    │→ │Scripting │→ │  Admin   │   ║
  ║   └──────────┘  └──────────┘  └──────────┘  └──────────┘   ║
  ╚══════════════════════════════════════════════════════════════╝
```

## Course Overview

This course covers everything a DevOps engineer needs to know about Linux — from foundational concepts and command-line mastery to networking, storage, shell scripting, and system administration. Each unit builds upon the previous one, providing hands-on examples, real-world scenarios, and best practices used in production environments.

**Target Audience:** Aspiring DevOps Engineers, System Administrators, Cloud Engineers, SREs  
**Prerequisites:** Basic computer literacy  
**Estimated Study Time:** 40–60 hours

---

## Complete Table of Contents

### Unit 1: Linux Fundamentals
| # | Topic | Link |
|---|-------|------|
| 1 | Linux Architecture | [01-linux-architecture.md](01-Linux-Fundamentals/01-linux-architecture.md) |
| 2 | Linux Distributions | [02-linux-distributions.md](01-Linux-Fundamentals/02-linux-distributions.md) |
| 3 | File System Hierarchy | [03-file-system-hierarchy.md](01-Linux-Fundamentals/03-file-system-hierarchy.md) |
| 4 | Boot Process | [04-boot-process.md](01-Linux-Fundamentals/04-boot-process.md) |
| 5 | Package Managers | [05-package-managers.md](01-Linux-Fundamentals/05-package-managers.md) |

### Unit 2: Command Line Essentials
| # | Topic | Link |
|---|-------|------|
| 1 | Navigation Commands | [01-navigation-commands.md](02-Command-Line-Essentials/01-navigation-commands.md) |
| 2 | File Operations | [02-file-operations.md](02-Command-Line-Essentials/02-file-operations.md) |
| 3 | Text Viewing | [03-text-viewing.md](02-Command-Line-Essentials/03-text-viewing.md) |
| 4 | Text Processing | [04-text-processing.md](02-Command-Line-Essentials/04-text-processing.md) |
| 5 | Finding Files | [05-finding-files.md](02-Command-Line-Essentials/05-finding-files.md) |

### Unit 3: Users and Permissions
| # | Topic | Link |
|---|-------|------|
| 1 | User Management | [01-user-management.md](03-Users-And-Permissions/01-user-management.md) |
| 2 | Group Management | [02-group-management.md](03-Users-And-Permissions/02-group-management.md) |
| 3 | File Permissions | [03-file-permissions.md](03-Users-And-Permissions/03-file-permissions.md) |
| 4 | Special Permissions | [04-special-permissions.md](03-Users-And-Permissions/04-special-permissions.md) |
| 5 | Access Control Lists | [05-access-control-lists.md](03-Users-And-Permissions/05-access-control-lists.md) |
| 6 | Sudo and Sudoers | [06-sudo-and-sudoers.md](03-Users-And-Permissions/06-sudo-and-sudoers.md) |

### Unit 4: Process Management
| # | Topic | Link |
|---|-------|------|
| 1 | Process Lifecycle | [01-process-lifecycle.md](04-Process-Management/01-process-lifecycle.md) |
| 2 | Viewing Processes | [02-viewing-processes.md](04-Process-Management/02-viewing-processes.md) |
| 3 | Process Control | [03-process-control.md](04-Process-Management/03-process-control.md) |
| 4 | Background & Foreground Jobs | [04-background-foreground-jobs.md](04-Process-Management/04-background-foreground-jobs.md) |
| 5 | Systemd & Service Management | [05-systemd-service-management.md](04-Process-Management/05-systemd-service-management.md) |
| 6 | Cron Jobs & Scheduling | [06-cron-jobs-scheduling.md](04-Process-Management/06-cron-jobs-scheduling.md) |

### Unit 5: Networking
| # | Topic | Link |
|---|-------|------|
| 1 | Network Configuration | [01-network-configuration.md](05-Networking/01-network-configuration.md) |
| 2 | IP Addressing & Subnetting | [02-ip-addressing-subnetting.md](05-Networking/02-ip-addressing-subnetting.md) |
| 3 | Network Commands | [03-network-commands.md](05-Networking/03-network-commands.md) |
| 4 | DNS Configuration | [04-dns-configuration.md](05-Networking/04-dns-configuration.md) |
| 5 | Firewall Management | [05-firewall-management.md](05-Networking/05-firewall-management.md) |
| 6 | SSH Configuration & Keys | [06-ssh-configuration-keys.md](05-Networking/06-ssh-configuration-keys.md) |
| 7 | Network Troubleshooting | [07-network-troubleshooting.md](05-Networking/07-network-troubleshooting.md) |

### Unit 6: Storage Management
| # | Topic | Link |
|---|-------|------|
| 1 | Disk Partitioning | [01-disk-partitioning.md](06-Storage-Management/01-disk-partitioning.md) |
| 2 | File Systems | [02-file-systems.md](06-Storage-Management/02-file-systems.md) |
| 3 | Mounting & fstab | [03-mounting-and-fstab.md](06-Storage-Management/03-mounting-and-fstab.md) |
| 4 | Logical Volume Manager | [04-logical-volume-manager.md](06-Storage-Management/04-logical-volume-manager.md) |
| 5 | RAID Concepts | [05-raid-concepts.md](06-Storage-Management/05-raid-concepts.md) |
| 6 | Disk Monitoring | [06-disk-monitoring.md](06-Storage-Management/06-disk-monitoring.md) |

### Unit 7: Shell Scripting
| # | Topic | Link |
|---|-------|------|
| 1 | Bash Basics | [01-bash-basics.md](07-Shell-Scripting/01-bash-basics.md) |
| 2 | Variables & Data Types | [02-variables-data-types.md](07-Shell-Scripting/02-variables-data-types.md) |
| 3 | Conditionals | [03-conditionals.md](07-Shell-Scripting/03-conditionals.md) |
| 4 | Loops | [04-loops.md](07-Shell-Scripting/04-loops.md) |
| 5 | Functions | [05-functions.md](07-Shell-Scripting/05-functions.md) |
| 6 | I/O Redirection | [06-io-redirection.md](07-Shell-Scripting/06-io-redirection.md) |
| 7 | Pipes & Command Chaining | [07-pipes-command-chaining.md](07-Shell-Scripting/07-pipes-command-chaining.md) |
| 8 | Error Handling | [08-error-handling.md](07-Shell-Scripting/08-error-handling.md) |
| 9 | Best Practices | [09-best-practices.md](07-Shell-Scripting/09-best-practices.md) |

### Unit 8: System Administration
| # | Topic | Link |
|---|-------|------|
| 1 | Log Management | [01-log-management.md](08-System-Administration/01-log-management.md) |
| 2 | System Monitoring | [02-system-monitoring.md](08-System-Administration/02-system-monitoring.md) |
| 3 | Performance Tuning | [03-performance-tuning.md](08-System-Administration/03-performance-tuning.md) |
| 4 | Backup Strategies | [04-backup-strategies.md](08-System-Administration/04-backup-strategies.md) |
| 5 | Security Hardening | [05-security-hardening.md](08-System-Administration/05-security-hardening.md) |
| 6 | SELinux & AppArmor | [06-selinux-apparmor.md](08-System-Administration/06-selinux-apparmor.md) |

---

## How to Use These Notes

1. **Sequential Study:** Follow units in order — each builds on the previous.
2. **Hands-On Practice:** Spin up a Linux VM (Ubuntu/CentOS) and practice every command.
3. **Revision Questions:** Test yourself with the questions at the end of each chapter.
4. **Summary Tables:** Use them for quick revision before interviews or exams.
5. **Real-World Scenarios:** Pay attention to these — they mirror actual DevOps tasks.

## Recommended Lab Setup

```
┌─────────────────────────────────────────────┐
│              Your Host Machine              │
│  (Windows / macOS / Linux)                  │
│                                             │
│   ┌───────────────────────────────────┐     │
│   │  VirtualBox / VMware / WSL        │     │
│   │                                   │     │
│   │   ┌───────────┐  ┌───────────┐   │     │
│   │   │  Ubuntu   │  │  CentOS   │   │     │
│   │   │  22.04    │  │  Stream 9 │   │     │
│   │   │  (Primary)│  │(Secondary)│   │     │
│   │   └───────────┘  └───────────┘   │     │
│   │                                   │     │
│   └───────────────────────────────────┘     │
│                                             │
│   OR: AWS Free Tier EC2 Instance            │
│   OR: Docker containers                     │
└─────────────────────────────────────────────┘
```

---

*Happy Learning! 🐧*

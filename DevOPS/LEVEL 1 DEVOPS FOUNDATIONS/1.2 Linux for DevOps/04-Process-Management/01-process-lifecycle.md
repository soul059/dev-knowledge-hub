# Chapter 1: Process Lifecycle

[← Previous: Sudo and Sudoers](../03-Users-And-Permissions/06-sudo-and-sudoers.md) | [Back to README](../README.md) | [Next: Viewing Processes →](02-viewing-processes.md)

---

## Overview

Every program running on a Linux system exists as a process. Understanding how processes are created, scheduled, and terminated is essential for managing applications, debugging performance issues, and designing reliable services.

---

## 1.1 Process Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    PROCESS HIERARCHY                          │
│                                                               │
│  systemd (PID 1)                                             │
│  ├── sshd                                                     │
│  │   └── sshd (per connection)                               │
│  │       └── bash (user shell)                               │
│  │           ├── vim                                          │
│  │           └── python app.py                                │
│  ├── dockerd                                                  │
│  │   └── containerd                                           │
│  │       └── container processes                              │
│  ├── nginx (master)                                           │
│  │   ├── nginx (worker)                                       │
│  │   └── nginx (worker)                                       │
│  └── cron                                                     │
│      └── /bin/bash (cron job)                                │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 Process States

```
┌────────────────────────────────────────────────────────────────┐
│                    PROCESS STATE DIAGRAM                       │
│                                                                │
│                    ┌─────────┐                                 │
│     fork()         │ Created │                                 │
│  ┌─────────────────│  (New)  │                                 │
│  │                 └────┬────┘                                 │
│  │                      │                                      │
│  │                 ┌────▼────┐                                 │
│  │                 │  Ready  │◄──────────────┐                │
│  │                 │ (Runnable)              │                │
│  │                 └────┬────┘               │                │
│  │                      │ scheduled          │ I/O complete   │
│  │                 ┌────▼────┐          ┌────┴────┐           │
│  │                 │ Running │──────────│Sleeping │           │
│  │                 │ (on CPU)│ wait I/O │(Blocked)│           │
│  │                 └────┬────┘          └─────────┘           │
│  │                      │                                      │
│  │                 ┌────▼────┐                                 │
│  │                 │ Stopped │  (SIGSTOP / Ctrl+Z)            │
│  │                 └────┬────┘                                 │
│  │                      │                                      │
│  │                 ┌────▼────┐          ┌──────────┐          │
│  └────────────────→│Terminated├─────────→│  Zombie  │          │
│      exit()        │ (Exit)  │  parent  │(Defunct) │          │
│                    └─────────┘  hasn't  └──────────┘          │
│                                 waited                         │
└────────────────────────────────────────────────────────────────┘

  STAT codes in ps:
  R = Running/Runnable    S = Sleeping (interruptible)
  D = Sleeping (uninterruptible, waiting for I/O)
  T = Stopped             Z = Zombie
  I = Idle kernel thread
```

---

## 1.3 Process Creation: fork() and exec()

```
┌───────────────────────────────────────────────────────┐
│  Parent Process (bash)                                │
│  PID: 1000                                            │
│       │                                               │
│       │ fork()  ──→ Creates exact copy                │
│       │                                               │
│  ┌────▼────┐        ┌──────────┐                     │
│  │ Parent  │        │  Child   │                     │
│  │ PID:1000│        │ PID:1001 │                     │
│  │ (waits) │        │ (copy)   │                     │
│  └─────────┘        └────┬─────┘                     │
│                          │                            │
│                          │ exec()  ──→ Replace with   │
│                          │             new program    │
│                     ┌────▼─────┐                     │
│                     │  Child   │                     │
│                     │ PID:1001 │                     │
│                     │ (nginx)  │                     │
│                     └──────────┘                     │
└───────────────────────────────────────────────────────┘
```

---

## 1.4 Key Process Attributes

```bash
# View process details
ps -p $$ -o pid,ppid,uid,gid,stat,comm
# PID   PPID  UID  GID  STAT COMMAND
# 1234  1230  1000 1000 Ss   bash

# $$ = current shell PID
echo $$          # Current PID
echo $PPID       # Parent PID
echo $!          # Last background process PID
echo $?          # Last command exit code
```

| Attribute | Description |
|-----------|-------------|
| **PID** | Process ID (unique) |
| **PPID** | Parent Process ID |
| **UID/GID** | User/Group running the process |
| **STAT** | Current state (R, S, D, T, Z) |
| **Priority/Nice** | Scheduling priority (-20 to 19) |
| **TTY** | Terminal associated with process |

---

## 1.5 Zombie and Orphan Processes

```bash
# Zombie: terminated but parent hasn't read exit status
# Shows as <defunct> in ps
ps aux | grep defunct
ps aux | grep "Z"

# Fix zombies: kill the parent (or wait for parent to reap)
# Or send SIGCHLD to parent:
kill -SIGCHLD <parent_PID>

# Orphan: parent died before child
# Adopted by systemd (PID 1) — generally harmless
```

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **PID** | Unique process identifier |
| **PPID** | Parent process ID |
| **fork()** | Create child process (copy of parent) |
| **exec()** | Replace process image with new program |
| **Zombie** | Terminated but not reaped by parent |
| **Orphan** | Parent died; adopted by PID 1 |
| **States** | R (running), S (sleeping), D (disk sleep), T (stopped), Z (zombie) |

---

## Quick Revision Questions

1. **What is the difference between `fork()` and `exec()`?** How do they work together?
2. **List all process states and what each letter means in `ps` output.**
3. **What is a zombie process?** How do you identify and fix them?
4. **What is PID 1 and why is it special?**
5. **What is the difference between interruptible (S) and uninterruptible (D) sleep?**
6. **How does the process hierarchy work in Linux?** What happens when a parent dies?

---

[← Previous: Sudo and Sudoers](../03-Users-And-Permissions/06-sudo-and-sudoers.md) | [Back to README](../README.md) | [Next: Viewing Processes →](02-viewing-processes.md)

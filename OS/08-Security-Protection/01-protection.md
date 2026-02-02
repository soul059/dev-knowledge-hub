# Chapter 8.1: Protection

## Overview

Protection refers to mechanisms that control access of processes and users to the resources defined by a computer system. It ensures that only authorized processes can operate on memory segments, files, CPU, and other resources.

---

## Goals of Protection

```
┌──────────────────────────────────────────────────────────────────┐
│                    GOALS OF PROTECTION                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. PREVENT UNAUTHORIZED ACCESS                                  │
│     • Ensure only authorized users access resources             │
│     • Prevent malicious tampering                               │
│                                                                  │
│  2. ENSURE COMPLIANCE WITH POLICIES                              │
│     • Enforce defined access rules                              │
│     • Support organizational security policies                   │
│                                                                  │
│  3. IMPROVE RELIABILITY                                          │
│     • Detect errors at interface boundaries                     │
│     • Prevent accidental misuse of resources                    │
│                                                                  │
│  4. RESOURCE SHARING                                             │
│     • Allow safe sharing of resources                           │
│     • Enable collaboration while maintaining control            │
│                                                                  │
│  Why Protection ≠ Security:                                      │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Protection: Internal OS mechanisms for access control       ││
│  │ Security:   Defense against external attacks                ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Protection Domains

```
┌──────────────────────────────────────────────────────────────────┐
│                   PROTECTION DOMAINS                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  A protection domain defines a set of objects and operations    │
│  that a process can perform on those objects.                   │
│                                                                  │
│  Domain = Collection of <Object, Rights>                        │
│                                                                  │
│  Example:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Domain D1:                                                 ││
│  │    • <File1, {read, write}>                                 ││
│  │    • <File2, {read}>                                        ││
│  │    • <Printer, {print}>                                     ││
│  │                                                             ││
│  │  Domain D2:                                                 ││
│  │    • <File1, {read}>                                        ││
│  │    • <File3, {read, write, execute}>                        ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Domains may overlap (share access to same object)              │
│                                                                  │
│         ┌─────────────────────────────────────────────┐         │
│         │             Resources                        │         │
│         │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │         │
│         │  │File1 │ │File2 │ │File3 │ │Printer│       │         │
│         │  └──┬───┘ └──┬───┘ └──┬───┘ └───┬──┘       │         │
│         └─────┼────────┼────────┼─────────┼──────────┘         │
│               │        │        │         │                     │
│       ┌───────┴────┐   │   ┌────┴─────────┘                     │
│       ▼            ▼   ▼   ▼                                    │
│   ┌───────┐    ┌───────────────┐                                │
│   │  D1   │    │      D2       │                                │
│   │ RW,R  │◄──►│  R, RWX       │                                │
│   │ Print │    │               │                                │
│   └───────┘    └───────────────┘                                │
│     Process P1      Process P2                                  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Domain Implementation in Practice

```
┌──────────────────────────────────────────────────────────────────┐
│              DOMAIN IMPLEMENTATION EXAMPLES                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. UNIX-LIKE SYSTEMS                                            │
│     • Domain = User ID (UID) + Group ID (GID)                   │
│     • Processes inherit domain from parent                      │
│     • setuid/setgid for temporary domain switch                 │
│                                                                  │
│     ┌────────────────────────────────────────────────┐          │
│     │  User: alice (UID=1001, GID=100)               │          │
│     │  Domain: {files owned by alice or group 100}  │          │
│     │                                                │          │
│     │  $ passwd    ← runs with setuid (root domain) │          │
│     └────────────────────────────────────────────────┘          │
│                                                                  │
│  2. WINDOWS SYSTEMS                                              │
│     • Domain = Access Token                                     │
│     • Contains SID, privileges, group memberships               │
│                                                                  │
│  3. CPU RINGS (Hardware Protection)                              │
│                                                                  │
│     ┌─────────────────────────────────────────┐                 │
│     │  Ring 0 (Kernel)    - Full access      │                 │
│     │  ┌───────────────────────────────────┐  │                 │
│     │  │  Ring 1 (Drivers)                 │  │                 │
│     │  │  ┌─────────────────────────────┐  │  │                 │
│     │  │  │  Ring 2 (Device drivers)   │  │  │                 │
│     │  │  │  ┌───────────────────────┐  │  │  │                 │
│     │  │  │  │  Ring 3 (User apps)  │  │  │  │                 │
│     │  │  │  └───────────────────────┘  │  │  │                 │
│     │  │  └─────────────────────────────┘  │  │                 │
│     │  └───────────────────────────────────┘  │                 │
│     └─────────────────────────────────────────┘                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Access Matrix

```
┌──────────────────────────────────────────────────────────────────┐
│                      ACCESS MATRIX                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  An access matrix is a model for protection where:              │
│  • Rows = Domains (users/processes)                             │
│  • Columns = Objects (files, devices, etc.)                     │
│  • Entries = Access rights                                      │
│                                                                  │
│                        OBJECTS                                   │
│           ┌─────────┬─────────┬─────────┬─────────┐             │
│           │  File1  │  File2  │  File3  │ Printer │             │
│  ┌────────┼─────────┼─────────┼─────────┼─────────┤             │
│  │   D1   │  R, W   │    R    │    -    │  Print  │             │
│  ├────────┼─────────┼─────────┼─────────┼─────────┤             │
│  │   D2   │    R    │    -    │ R,W,X   │    -    │             │
│  ├────────┼─────────┼─────────┼─────────┼─────────┤             │
│  │   D3   │    -    │  R, W   │    R    │  Print  │             │
│  └────────┴─────────┴─────────┴─────────┴─────────┘             │
│  DOMAINS                                                         │
│                                                                  │
│  Rights: R=Read, W=Write, X=Execute, -=No access               │
│                                                                  │
│  Domain switching can also be in the matrix:                    │
│  ┌──────────┬────────┬────────┬────────┬──────────┐            │
│  │          │  D1    │  D2    │  D3    │  File1   │            │
│  ├──────────┼────────┼────────┼────────┼──────────┤            │
│  │    D1    │   -    │ switch │   -    │   R,W    │            │
│  │    D2    │   -    │   -    │ switch │   R      │            │
│  │    D3    │ switch │   -    │   -    │   -      │            │
│  └──────────┴────────┴────────┴────────┴──────────┘            │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Implementing Access Matrix

The access matrix is a conceptual model. Actual implementation uses one of these:

### 1. Access Control Lists (ACL)

```
┌──────────────────────────────────────────────────────────────────┐
│               ACCESS CONTROL LIST (ACL)                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Store access rights with each OBJECT (column-wise)             │
│  Each object has a list of <domain, rights> pairs              │
│                                                                  │
│  File1.acl:                                                      │
│  ┌────────────────────────────────────┐                         │
│  │  ┌──────────────────────────────┐  │                         │
│  │  │ Domain D1: read, write       │  │                         │
│  │  │ Domain D2: read              │  │                         │
│  │  │ Domain D3: (no entry=deny)   │  │                         │
│  │  └──────────────────────────────┘  │                         │
│  └────────────────────────────────────┘                         │
│                                                                  │
│  File2.acl:                                                      │
│  ┌────────────────────────────────────┐                         │
│  │  ┌──────────────────────────────┐  │                         │
│  │  │ Domain D1: read              │  │                         │
│  │  │ Domain D3: read, write       │  │                         │
│  │  └──────────────────────────────┘  │                         │
│  └────────────────────────────────────┘                         │
│                                                                  │
│  Example (UNIX permissions - simplified ACL):                   │
│  $ ls -l                                                        │
│  -rwxr-x--- alice staff file.txt                                │
│       │ │ │   │     │                                           │
│       │ │ │   │     └── Group                                   │
│       │ │ │   └──────── Owner                                   │
│       │ │ └──────────── Others: no access                       │
│       │ └────────────── Group: read, execute                    │
│       └──────────────── Owner: read, write, execute             │
│                                                                  │
│  Advantages:                                                     │
│  ✓ Easy to see who can access an object                        │
│  ✓ Easy to revoke all access to an object                      │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ Finding all objects a user can access is slow               │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 2. Capability Lists

```
┌──────────────────────────────────────────────────────────────────┐
│                  CAPABILITY LISTS                                 │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Store access rights with each DOMAIN (row-wise)                │
│  Each domain has a list of <object, rights> pairs              │
│  A capability = unforgeable token representing access rights    │
│                                                                  │
│  Domain D1's capabilities:                                       │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  ┌──────────────────────────────────────────────────────┐   ││
│  │  │ Capability 1: File1, read, write      [token: x7f3]  │   ││
│  │  │ Capability 2: File2, read             [token: a2b9]  │   ││
│  │  │ Capability 3: Printer, print          [token: c4d1]  │   ││
│  │  └──────────────────────────────────────────────────────┘   ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Domain D2's capabilities:                                       │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  ┌──────────────────────────────────────────────────────┐   ││
│  │  │ Capability 1: File1, read             [token: m5n2]  │   ││
│  │  │ Capability 2: File3, read, write, exe [token: p8q3]  │   ││
│  │  └──────────────────────────────────────────────────────┘   ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  How it works:                                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Process presents capability → Kernel verifies → Access OK  ││
│  │                                                             ││
│  │ Process: "I want to read File1, here's my capability"      ││
│  │ Kernel:  "Capability valid, granting read access"          ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Advantages:                                                     │
│  ✓ Easy to see what a user can access                          │
│  ✓ Easy to audit a user's permissions                          │
│  ✓ Capabilities can be transferred                              │
│                                                                  │
│  Disadvantages:                                                  │
│  ✗ Revoking access is difficult                                 │
│  ✗ Must ensure capabilities are unforgeable                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### ACL vs Capability Comparison

```
┌──────────────────────────────────────────────────────────────────┐
│               ACL vs CAPABILITY COMPARISON                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Aspect          │   ACL             │   Capability         ││
│  ├───────────────────┼───────────────────┼─────────────────────┤│
│  │  Stored with     │   Object          │   Domain/Process    ││
│  │  Question asked  │   "Who can access │   "What can this   ││
│  │                  │    this file?"    │    user access?"    ││
│  │  Add permission  │   Modify object   │   Create new       ││
│  │                  │   ACL             │   capability        ││
│  │  Revoke access   │   Easy            │   Difficult         ││
│  │  Audit user's    │   Slow (check all │   Fast (check user ││
│  │  permissions     │   objects)        │   capabilities)     ││
│  │  Real-world use  │   Unix, Windows   │   Some research OS  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Modern systems often combine both!                             │
│  • Use ACLs for persistent storage (files)                      │
│  • Use capability-like tokens for runtime (file descriptors)   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Role-Based Access Control (RBAC)

```
┌──────────────────────────────────────────────────────────────────┐
│              ROLE-BASED ACCESS CONTROL (RBAC)                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Instead of assigning permissions to users directly,            │
│  assign permissions to ROLES, then assign roles to users.       │
│                                                                  │
│  Traditional:  User ──────────────────────────► Permission      │
│                                                                  │
│  RBAC:         User ──────► Role ──────────────► Permission     │
│                                                                  │
│  Example:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                                                             ││
│  │   USERS              ROLES              PERMISSIONS         ││
│  │                                                             ││
│  │   Alice ───────► [Manager] ──────► Read financial reports  ││
│  │                     │              Approve expenses          ││
│  │   Bob ─────────► [Manager]                                  ││
│  │                                                             ││
│  │   Charlie ────► [Developer] ────► Read/Write source code   ││
│  │                     │              Deploy to test           ││
│  │   Diana ──────► [Developer]                                 ││
│  │                                                             ││
│  │   Eve ────────► [Admin] ────────► Full system access       ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Advantages:                                                     │
│  ✓ Easier to manage (manage roles, not individual permissions) │
│  ✓ Reflects organizational structure                            │
│  ✓ Easier auditing                                              │
│  ✓ Principle of least privilege                                 │
│                                                                  │
│  RBAC Hierarchy:                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    [Senior Manager]                         ││
│  │                          │                                  ││
│  │                ┌─────────┴─────────┐                        ││
│  │                ▼                   ▼                        ││
│  │           [Manager]           [Tech Lead]                   ││
│  │                │                   │                        ││
│  │                ▼                   ▼                        ││
│  │           [Employee]          [Developer]                   ││
│  │                                                             ││
│  │  Senior Manager inherits all permissions from lower roles  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Principle of Least Privilege

```
┌──────────────────────────────────────────────────────────────────┐
│              PRINCIPLE OF LEAST PRIVILEGE                         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Every program and every user should operate using the least  │
│   amount of privilege necessary to complete the job."           │
│                                                                  │
│  Examples:                                                       │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                                                             ││
│  │  BAD:  Web server runs as root                              ││
│  │        ↓                                                    ││
│  │        If compromised, attacker has full access            ││
│  │                                                             ││
│  │  GOOD: Web server runs as www-data (limited user)          ││
│  │        ↓                                                    ││
│  │        If compromised, attacker only has web permissions   ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  Implementation techniques:                                      │
│  • Drop privileges after initialization                         │
│  • Use separate accounts for different tasks                    │
│  • Temporary privilege elevation (sudo, setuid)                 │
│  • Sandboxing (containers, VMs)                                 │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  Modern OS Features:                                        ││
│  │                                                             ││
│  │  • Linux: capabilities (fine-grained root powers)          ││
│  │    CAP_NET_BIND_SERVICE - bind to ports < 1024             ││
│  │    CAP_SYS_ADMIN - various admin operations                ││
│  │                                                             ││
│  │  • Windows: User Account Control (UAC)                     ││
│  │    Standard user → Admin only when needed                  ││
│  │                                                             ││
│  │  • macOS: System Integrity Protection (SIP)                ││
│  │    Even root can't modify protected areas                  ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Description | Example |
|---------|-------------|---------|
| **Protection Domain** | Set of objects and access rights | User's UID + GID in Unix |
| **Access Matrix** | Conceptual model (domains × objects) | Rows=users, Columns=files |
| **ACL** | Permissions stored with object | File permissions in Unix |
| **Capabilities** | Tokens stored with domain | File descriptors |
| **RBAC** | Assign roles, not direct permissions | Admin, User, Guest roles |
| **Least Privilege** | Minimum necessary permissions | Web server as www-data |

---

## Quick Revision Questions

1. What is the difference between protection and security?
2. What are the three columns in an access matrix?
3. How do ACLs differ from capabilities?
4. Why is RBAC easier to manage than direct permission assignment?
5. Give an example of the principle of least privilege.

---

[← Previous: Disk Scheduling](../07-IO-Systems/03-disk-scheduling.md) | [Next: Security →](./02-security.md)

# Need to Know

## Unit 3 - Topic 2 | Security Principles

---

## Overview

The **Need to Know** principle restricts access to information based on whether a person genuinely requires that information to perform their specific duties. Even if someone has the security clearance or authorization level, they should not access data unless they have a legitimate business need.

---

## 1. Core Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                     NEED TO KNOW PRINCIPLE                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "Having permission does NOT automatically mean having access.   │
│   You must also have a LEGITIMATE REASON."                       │
│                                                                  │
│  Security Clearance (Authorization)  +  Need to Know (Justification)  │
│  ═══════════════════════════════════     ══════════════════════════    │
│  "Are you ALLOWED to see this?"      +  "Do you NEED to see this?"   │
│                                                                  │
│  BOTH must be true for access to be granted.                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Need to Know vs Least Privilege

| Aspect | Least Privilege | Need to Know |
|--------|----------------|-------------|
| **Focus** | Permissions and capabilities | Information access and data |
| **Question** | "What can you DO?" | "What can you SEE?" |
| **Scope** | System actions (read/write/execute) | Data and information content |
| **Example** | User can only edit, not delete files | User can only see files related to their project |
| **Origin** | IT security principle | Military/intelligence classification |
| **Relationship** | Broader concept | More specific, data-focused |

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   LEAST PRIVILEGE                                       │
│   ┌─────────────────────────────────────────────────┐  │
│   │                                                 │  │
│   │   NEED TO KNOW                                  │  │
│   │   ┌─────────────────────────────────────────┐  │  │
│   │   │                                         │  │  │
│   │   │  "Only the specific data you need       │  │  │
│   │   │   for your current task"                │  │  │
│   │   │                                         │  │  │
│   │   └─────────────────────────────────────────┘  │  │
│   │                                                 │  │
│   │  "Only the minimum permissions for your role"  │  │
│   │                                                 │  │
│   └─────────────────────────────────────────────────┘  │
│                                                         │
│  "Overall access control framework"                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Real-World Examples

| Scenario | Has Clearance? | Needs to Know? | Access Granted? |
|----------|:-:|:-:|:-:|
| HR manager accessing HR payroll data | ✅ | ✅ | ✅ Yes |
| HR manager accessing engineering source code | ✅ (same clearance) | ❌ | ❌ No |
| Developer accessing own project files | ✅ | ✅ | ✅ Yes |
| Developer accessing a different team's project | ✅ (same clearance) | ❌ | ❌ No |
| CEO accessing all company financials | ✅ | ✅ | ✅ Yes |
| CEO accessing individual medical records | ✅ (top clearance) | ❌ | ❌ No |

---

## 4. Implementation

| Method | Description |
|--------|-------------|
| **Data classification** | Label data by sensitivity (Public, Internal, Confidential, Restricted) |
| **Project-based access** | Only team members access project data |
| **Compartmentalization** | Divide information into isolated compartments |
| **Access request workflows** | Require justification and approval for data access |
| **Regular access reviews** | Remove access when role/project changes |
| **DLP policies** | Prevent unauthorized sharing of sensitive data |

---

## 5. Military Origin

```
┌──────────────────────────────────────────────────────────────┐
│            MILITARY CLASSIFICATION EXAMPLE                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Analyst A: TOP SECRET clearance                             │
│  Working on: Project Alpha                                   │
│  ├── Access to Project Alpha data ✅ (need to know)          │
│  └── Access to Project Beta data ❌ (no need to know)        │
│                                                              │
│  Analyst B: TOP SECRET clearance                             │
│  Working on: Project Beta                                    │
│  ├── Access to Project Beta data ✅ (need to know)           │
│  └── Access to Project Alpha data ❌ (no need to know)       │
│                                                              │
│  BOTH have same clearance level.                             │
│  NEITHER can see the other's project.                        │
│  This is Need to Know in action.                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Definition** | Access info only if you need it for your duties |
| **vs Least Privilege** | PoLP = permissions; Need to Know = data access |
| **Key Requirement** | Clearance + Justification = Access |
| **Implementation** | Data classification, compartmentalization, access reviews |
| **Origin** | Military/intelligence communities |

---

## Quick Revision Questions

1. **How does Need to Know differ from Least Privilege?**
2. **Can someone with top-level clearance access all data? Why or why not?**
3. **Give an example where Need to Know prevents data access despite authorization.**
4. **How does data classification support the Need to Know principle?**
5. **What is compartmentalization and how does it enforce Need to Know?**

---

[← Previous: Least Privilege](01-least-privilege.md) | [Next: Separation of Duties →](03-separation-of-duties.md)

---

*Unit 3 - Topic 2 of 6 | [Back to README](../README.md)*

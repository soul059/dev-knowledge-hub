# Chapter 9.2: Permission Systems

> **Unit 9 — Practical Applications**
> Implement Unix-style permissions, feature flags, and role-based access control using bit manipulation.

---

## Overview

Bit flags are one of the most common real-world uses of bitwise operations. A single integer can store dozens of boolean settings, with O(1) check, set, and clear operations.

---

## Unix File Permissions

```
  rwx rwx rwx
  ─┬─ ─┬─ ─┬─
   │   │   └── Others
   │   └────── Group
   └────────── Owner

  Each rwx triplet = 3 bits:
    r (read)    = 4 = 100
    w (write)   = 2 = 010
    x (execute) = 1 = 001
```

### Permission Bits Layout

```
  Bit:  8  7  6  5  4  3  2  1  0
        r  w  x  r  w  x  r  w  x
        ─────── ─────── ───────
         Owner   Group   Others

  chmod 755:
    Owner: 7 = 111 (rwx)
    Group: 5 = 101 (r-x)
    Others:5 = 101 (r-x)

  Binary: 111 101 101 = 0b111101101 = 0x1ED
```

### Operations

```
FUNCTION hasPermission(perms, flag):
    RETURN (perms & flag) != 0

FUNCTION grantPermission(perms, flag):
    RETURN perms | flag

FUNCTION revokePermission(perms, flag):
    RETURN perms & (~flag)

FUNCTION togglePermission(perms, flag):
    RETURN perms ^ flag
```

### Trace: Check if group has write on chmod 755

```
  perms = 0b111101101  (755)
  GROUP_WRITE = 0b000010000  (bit 4)

  perms & GROUP_WRITE:
    111101101
  & 000010000
  ──────────
    000000000  = 0

  Result: Group does NOT have write permission  ✓
  (755 gives group r-x, no w)
```

---

## Feature Flags in Software

```
  ┌────────────────────────────────────────────────┐
  │  Common in game engines, software config, etc. │
  │  Store many boolean features in one integer    │
  └────────────────────────────────────────────────┘

  FEATURE_DARK_MODE   = 1 << 0   // 0x01
  FEATURE_BETA_UI     = 1 << 1   // 0x02
  FEATURE_ANALYTICS   = 1 << 2   // 0x04
  FEATURE_EXPORT      = 1 << 3   // 0x08
  FEATURE_NOTIFICATIONS = 1 << 4 // 0x10
```

### Using Feature Flags

```
  // User's enabled features
  userFlags = FEATURE_DARK_MODE | FEATURE_EXPORT
  // = 0x01 | 0x08 = 0x09 = 0b01001

  // Check if analytics is enabled
  IF userFlags & FEATURE_ANALYTICS:   // 0x09 & 0x04 = 0 → NO
      trackUsage()

  // Enable beta UI
  userFlags |= FEATURE_BETA_UI       // 0x09 | 0x02 = 0x0B

  // Disable dark mode
  userFlags &= ~FEATURE_DARK_MODE    // 0x0B & 0xFE = 0x0A
```

---

## Role-Based Access Control (RBAC) with Bits

```
  Actions:
    READ    = 1 << 0   // 0x01
    WRITE   = 1 << 1   // 0x02
    DELETE  = 1 << 2   // 0x04
    ADMIN   = 1 << 3   // 0x08
    AUDIT   = 1 << 4   // 0x10

  Roles (combinations of actions):
    VIEWER  = READ                         // 0x01
    EDITOR  = READ | WRITE                 // 0x03
    MANAGER = READ | WRITE | DELETE        // 0x07
    ADMIN   = READ | WRITE | DELETE | ADMIN| AUDIT  // 0x1F
```

### Access Checks

```
FUNCTION canPerform(userRole, requiredPermissions):
    // Check ALL required permissions are present
    RETURN (userRole & requiredPermissions) == requiredPermissions

FUNCTION canPerformAny(userRole, permissions):
    // Check AT LEAST ONE permission is present
    RETURN (userRole & permissions) != 0
```

### Trace: Can EDITOR delete?

```
  EDITOR   = 0b00011  (READ | WRITE)
  DELETE   = 0b00100

  EDITOR & DELETE:
    00011
  & 00100
  ──────
    00000  ≠ DELETE

  Result: EDITOR cannot delete  ✓
```

---

## Combining Permissions from Multiple Sources

```
  User may have permissions from multiple roles:

  FUNCTION mergePermissions(roles[]):
      result = 0
      FOR each role in roles:
          result |= role      // union of all permissions
      RETURN result

  FUNCTION restrictPermissions(perms, mask):
      RETURN perms & mask     // intersection
```

```
  // User is both EDITOR and AUDIT role
  userPerms = EDITOR | AUDIT
            = 0b00011 | 0b10000
            = 0b10011  (READ, WRITE, AUDIT)

  // But restricted to department mask
  deptMask = READ | AUDIT   // 0b10001
  effective = userPerms & deptMask
            = 0b10011 & 0b10001
            = 0b10001  (READ, AUDIT only)
```

---

## Advantages Over Boolean Arrays

```
  ┌──────────────────────────┬────────────────┬────────────────┐
  │ Aspect                   │ Bool Array     │ Bit Flags      │
  ├──────────────────────────┼────────────────┼────────────────┤
  │ Storage (32 flags)       │ 32 bytes       │ 4 bytes        │
  │ Check one flag           │ O(1)           │ O(1)           │
  │ Check all flags at once  │ O(n)           │ O(1)           │
  │ Copy all flags           │ O(n)           │ O(1)           │
  │ Compare two sets         │ O(n)           │ O(1)           │
  │ Serialize (DB/network)   │ Complex        │ Single integer │
  │ Union / Intersection     │ O(n)           │ O(1)           │
  └──────────────────────────┴────────────────┴────────────────┘
```

---

## Summary Table

| Operation | Code | Description |
|-----------|------|-------------|
| Check flag | `flags & FLAG` | Non-zero if set |
| Set flag | `flags \| FLAG` | Turn on |
| Clear flag | `flags & ~FLAG` | Turn off |
| Toggle flag | `flags ^ FLAG` | Flip |
| Check all of set | `(flags & set) == set` | Has all required |
| Check any of set | `(flags & set) != 0` | Has at least one |
| Merge roles | `r1 \| r2` | Union of permissions |
| Restrict | `perms & mask` | Intersection |

---

## Quick Revision Questions

1. **Why use bitwise flags instead of separate booleans?**
   <details><summary>Answer</summary>Bitwise flags use 8× less memory, support O(1) bulk operations (union, intersection, comparison), and serialize to a single integer for easy storage and transmission.</details>

2. **How does `(flags & required) == required` differ from `(flags & required) != 0`?**
   <details><summary>Answer</summary>The first checks that ALL bits in `required` are set (AND check). The second checks that AT LEAST ONE bit is set (OR check). For permission systems, "all required" is the more common pattern.</details>

3. **What is chmod 644 in binary?**
   <details><summary>Answer</summary>6=110(rw-), 4=100(r--), 4=100(r--). Binary: 110 100 100. Owner can read/write, group and others can only read.</details>

4. **How do you combine permissions from multiple roles?**
   <details><summary>Answer</summary>OR them together: `result = role1 | role2 | role3`. This creates the union of all granted permissions. Each bit that is set in ANY role becomes set in the result.</details>

5. **What limits the number of flags in a single integer?**
   <details><summary>Answer</summary>The integer width: 32 flags for 32-bit int, 64 for 64-bit long. For more than 64 flags, use arrays of integers (bitsets) or dedicated bitset data structures.</details>

---

[← Previous: IP Address Manipulation](01-ip-address-manipulation.md) | [Next: Compression Basics →](03-compression-basics.md)

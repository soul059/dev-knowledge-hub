# Chapter 9.1: IP Address Manipulation

> **Unit 9 — Practical Applications**
> Use bitwise operations to efficiently store, compare, subnet, and manipulate IPv4 addresses.

---

## Overview

An IPv4 address is a 32-bit unsigned integer — a natural fit for bitwise operations. Networking code relies heavily on bit manipulation for subnetting, masking, and address range calculations.

---

## IP Address as a 32-bit Integer

```
  IPv4: 192.168.1.100

  192      .  168      .  1        .  100
  11000000    10101000    00000001    01100100

  Combined 32-bit integer:
  11000000 10101000 00000001 01100100
  = 0xC0A80164
  = 3,232,235,876 (decimal)
```

### Dotted Decimal → Integer

```
FUNCTION ipToInt(a, b, c, d):
    RETURN (a << 24) | (b << 16) | (c << 8) | d
```

### Integer → Dotted Decimal

```
FUNCTION intToIp(ip):
    a = (ip >> 24) & 0xFF
    b = (ip >> 16) & 0xFF
    c = (ip >> 8)  & 0xFF
    d = ip & 0xFF
    RETURN a.b.c.d
```

### Trace: 192.168.1.100

```
  a=192: 192 << 24 = 11000000 00000000 00000000 00000000
  b=168: 168 << 16 = 00000000 10101000 00000000 00000000
  c=1:   1   << 8  = 00000000 00000000 00000001 00000000
  d=100: 100       = 00000000 00000000 00000000 01100100

  OR all together:
  11000000 10101000 00000001 01100100 = 0xC0A80164  ✓
```

---

## Subnet Masks

```
  A subnet mask has N leading 1-bits followed by (32-N) 0-bits.

  /24 (255.255.255.0):
  11111111 11111111 11111111 00000000

  /16 (255.255.0.0):
  11111111 11111111 00000000 00000000

  /20 (255.255.240.0):
  11111111 11111111 11110000 00000000
```

### Generate Subnet Mask from CIDR Prefix

```
FUNCTION subnetMask(prefix):
    IF prefix == 0: RETURN 0
    RETURN 0xFFFFFFFF << (32 - prefix)
    // or equivalently: ~((1 << (32 - prefix)) - 1)
```

### Trace: /24

```
  32 - 24 = 8
  0xFFFFFFFF << 8:
  Before: 11111111 11111111 11111111 11111111
  After:  11111111 11111111 11111111 00000000
  = 255.255.255.0  ✓
```

---

## Key Network Calculations

### 1. Network Address (first address in subnet)

```
FUNCTION networkAddress(ip, prefix):
    mask = subnetMask(prefix)
    RETURN ip & mask
```

### 2. Broadcast Address (last address in subnet)

```
FUNCTION broadcastAddress(ip, prefix):
    mask = subnetMask(prefix)
    RETURN ip | (~mask & 0xFFFFFFFF)
```

### 3. Number of Hosts

```
FUNCTION hostCount(prefix):
    RETURN (1 << (32 - prefix)) - 2
    // subtract 2 for network and broadcast addresses
```

### 4. Check if IP is in Subnet

```
FUNCTION isInSubnet(ip, network, prefix):
    mask = subnetMask(prefix)
    RETURN (ip & mask) == (network & mask)
```

### Trace: Is 192.168.1.100 in 192.168.1.0/24?

```
  mask = 255.255.255.0 = 0xFFFFFF00

  ip & mask:      0xC0A80164 & 0xFFFFFF00 = 0xC0A80100
  network & mask: 0xC0A80100 & 0xFFFFFF00 = 0xC0A80100
  
  Equal? YES → 192.168.1.100 is in 192.168.1.0/24  ✓
```

---

## Wildcard Masks (Inverse Masks)

```
  Used by Cisco ACLs. Wildcard = ~subnetMask.

  Subnet mask:  255.255.255.0   = 11111111 11111111 11111111 00000000
  Wildcard:     0.0.0.255       = 00000000 00000000 00000000 11111111

  bit=0: must match
  bit=1: don't care
```

```
FUNCTION wildcardMask(prefix):
    RETURN ~subnetMask(prefix) & 0xFFFFFFFF
```

---

## Practical Operations Summary

```
  ┌──────────────────────────────────┬────────────────────────────┐
  │ Operation                        │ Bitwise Formula            │
  ├──────────────────────────────────┼────────────────────────────┤
  │ IP → integer                     │ (a<<24)|(b<<16)|(c<<8)|d  │
  │ Integer → octets                 │ (ip>>24)&0xFF, etc.        │
  │ Subnet mask from /N             │ 0xFFFFFFFF << (32-N)       │
  │ Network address                  │ ip & mask                  │
  │ Broadcast address                │ ip | ~mask                 │
  │ Host count                       │ 2^(32-N) - 2              │
  │ Is IP in subnet?                │ (ip & mask) == network     │
  │ Wildcard mask                    │ ~mask                      │
  │ Next subnet                      │ network + (1 << (32-N))   │
  └──────────────────────────────────┴────────────────────────────┘
```

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Any single IP operation | O(1) | O(1) |
| IP range scan | O(hosts) | O(1) |
| Subnet comparison | O(1) | O(1) |

---

## Quick Revision Questions

1. **Why are IP addresses naturally suited for bit manipulation?**
   <details><summary>Answer</summary>IPv4 addresses are 32-bit unsigned integers. Subnetting, masking, and network calculations all map directly to AND, OR, shift, and NOT operations.</details>

2. **How do you extract the second octet from a 32-bit IP?**
   <details><summary>Answer</summary>(ip >> 16) & 0xFF — shift right by 16 to bring the second octet to the lowest byte, then mask with 0xFF to isolate it.</details>

3. **What does `ip & subnetMask` compute?**
   <details><summary>Answer</summary>The network address — the first address in the subnet. It zeros out the host bits, keeping only the network prefix.</details>

4. **How many usable host addresses in a /28?**
   <details><summary>Answer</summary>2^(32-28) - 2 = 2^4 - 2 = 14 usable hosts. The -2 excludes the network address and broadcast address.</details>

5. **What is a wildcard mask and how is it computed?**
   <details><summary>Answer</summary>The bitwise complement of the subnet mask. 0-bits mean "must match" and 1-bits mean "don't care." Computed as ~subnetMask (with 32-bit truncation).</details>

---

[← Previous: Memory Savings](../08-Bit-Manipulation-In-DP/05-memory-savings.md) | [Next: Permission Systems →](02-permission-systems.md)

# 8.2 Rolling Hash Technique

## Unit 8: String Hashing Problems

---

## What is a Rolling Hash?

A **rolling hash** maintains a hash value over a sliding window, updating it in $O(1)$ when the window shifts by one position — instead of recomputing the full hash in $O(m)$.

```
╔══════════════════════════════════════════════════════════════╗
║  WITHOUT rolling hash:                                      ║
║  Window 1: hash("ABCD") = compute from scratch  → O(m)    ║
║  Window 2: hash("BCDE") = compute from scratch  → O(m)    ║
║  Window 3: hash("CDEF") = compute from scratch  → O(m)    ║
║  Total: O(nm)                                               ║
║                                                              ║
║  WITH rolling hash:                                         ║
║  Window 1: hash("ABCD") = compute from scratch  → O(m)    ║
║  Window 2: hash("BCDE") = update from window 1  → O(1)    ║
║  Window 3: hash("CDEF") = update from window 2  → O(1)    ║
║  Total: O(n)                                                ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Polynomial Rolling Hash

The most common rolling hash treats the string as a polynomial:

$$H(s[0..m-1]) = \sum_{i=0}^{m-1} s[i] \cdot d^{m-1-i} \mod q$$

Where:
- $d$ = base (typically 26, 31, or 256)
- $q$ = large prime modulus
- $s[i]$ = character value at position $i$

```
Example: hash("ABC") with d=31, q=10^9+7
  = A·31² + B·31¹ + C·31⁰
  = 65·961 + 66·31 + 67·1
  = 62465 + 2046 + 67
  = 64578
```

---

## Rolling Update Formula

When sliding the window from position $i$ to $i+1$:

$$H_{i+1} = d \cdot (H_i - s[i] \cdot d^{m-1}) + s[i+m] \pmod{q}$$

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  Old window: s[i]  s[i+1]  s[i+2] ... s[i+m-1]            ║
║  New window:        s[i+1]  s[i+2] ... s[i+m-1]  s[i+m]   ║
║                                                              ║
║  ┌────────────────────────────────────────────────┐         ║
║  │ 1. REMOVE: subtract s[i] * d^(m-1)            │         ║
║  │    removes the highest-order term              │         ║
║  │                                                │         ║
║  │ 2. SHIFT:  multiply by d                       │         ║
║  │    shifts all remaining terms up one power     │         ║
║  │                                                │         ║
║  │ 3. ADD:    add s[i+m]                          │         ║
║  │    adds the new lowest-order term              │         ║
║  └────────────────────────────────────────────────┘         ║
║                                                              ║
║  Concrete example:                                          ║
║  hash("ABCD") = A·d³ + B·d² + C·d + D                     ║
║                                                              ║
║  Remove A:     B·d² + C·d + D                               ║
║  Shift ×d:     B·d³ + C·d² + D·d                           ║
║  Add E:        B·d³ + C·d² + D·d + E                       ║
║  = hash("BCDE") ✓                                          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Implementation

```
CLASS RollingHash:
    CONSTRUCTOR(s, windowSize, base=31, mod=10^9+7):
        this.s = s
        this.m = windowSize
        this.d = base
        this.q = mod
        this.hash = 0
        this.highPower = modPow(d, m-1, q)  // d^(m-1) mod q

        // Compute initial hash
        FOR i = 0 TO m - 1:
            this.hash = (this.hash * d + s[i]) % q

    FUNCTION slide(oldChar, newChar):
        // Remove old, shift, add new
        this.hash = (d * (this.hash - oldChar * highPower) + newChar) % q
        IF this.hash < 0:
            this.hash += q
        RETURN this.hash

    FUNCTION currentHash():
        RETURN this.hash
```

### Trace

```
s = "ABCDE", window=3, d=10, q=97

Initial hash("ABC"):
  hash = (((0*10 + 1)*10 + 2)*10 + 3) = 123
  hash mod 97 = 26

  highPower = 10² mod 97 = 3

Slide to "BCD":
  hash = (10 * (26 - 1*3) + 4) mod 97
       = (10 * 23 + 4) mod 97
       = 234 mod 97 = 40

  Verify: hash("BCD") = 234 mod 97 = 40 ✓

Slide to "CDE":
  hash = (10 * (40 - 2*3) + 5) mod 97
       = (10 * 34 + 5) mod 97
       = 345 mod 97 = 54

  Verify: hash("CDE") = 345 mod 97 = 54 ✓
```

---

## Choosing Parameters

### Base ($d$)

| Base | Use Case | Notes |
|:---:|---------|-------|
| 26 | Lowercase letters only | Characters mapped to 0-25 |
| 31 | General strings | Good prime, few collisions |
| 256 | Full ASCII | No character mapping needed |
| 131 | Unicode-safe | Covers extended characters |

### Modulus ($q$)

| Modulus | Collision Rate | Overflow Risk |
|:---:|:---:|:---:|
| $10^9 + 7$ | ~$10^{-9}$ | Safe with 64-bit |
| $10^9 + 9$ | ~$10^{-9}$ | Safe with 64-bit |
| $2^{61} - 1$ | ~$5 \times 10^{-19}$ | Mersenne prime, fast mod |

```
╔══════════════════════════════════════════════════════════════╗
║  REDUCING COLLISION PROBABILITY                             ║
║                                                              ║
║  Single hash:  Pr[collision] ≈ 1/q                         ║
║  Double hash:  Pr[collision] ≈ 1/q₁ × 1/q₂                ║
║                                                              ║
║  For competitive programming:                               ║
║    Use TWO independent hashes with different (d, q) pairs   ║
║    Only declare match when BOTH hashes agree                ║
║    Collision probability drops to ~10^{-18}                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Double Hashing for Safety

```
CLASS DoubleRollingHash:
    hash1 = RollingHash(s, m, base=31, mod=10^9+7)
    hash2 = RollingHash(s, m, base=37, mod=10^9+9)

    FUNCTION matches(other):
        RETURN hash1.value == other.hash1.value
           AND hash2.value == other.hash2.value
```

---

## Common Applications

| Application | How Rolling Hash Helps |
|-------------|----------------------|
| Pattern matching | Rabin-Karp algorithm |
| Duplicate substring detection | Hash all substrings, find repeats |
| Longest common substring | Binary search + hash set |
| Plagiarism detection | Compare document fingerprints |
| File deduplication | Content-defined chunking |

---

## Pitfalls

```
╔══════════════════════════════════════════════════════════════╗
║  1. NEGATIVE MODULUS                                        ║
║     (a - b) % q can be negative in C++/Java                ║
║     Fix: ((a - b) % q + q) % q                             ║
║                                                              ║
║  2. INTEGER OVERFLOW                                        ║
║     d * hash can overflow before mod                        ║
║     Fix: Use long/int64, or reduce at each step            ║
║                                                              ║
║  3. HASH = 0 MATCHES                                       ║
║     Don't skip hash comparison when hash is 0              ║
║     Zero is a valid hash value                              ║
║                                                              ║
║  4. CHARACTER MAPPING                                       ║
║     'a'→0 is fine, but 'a'→0 with base 26 means           ║
║     "a" and "aa" can hash to 0. Use 'a'→1 instead.        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: If you recompute the hash from scratch for every window position, what is the total time complexity? How does the rolling hash improve this?**

<details>
<summary>Click to reveal answer</summary>

**Without rolling hash:**
- Each window hash computation: $O(m)$
- Number of windows: $n - m + 1$
- Total: $O(m \cdot (n - m + 1)) = O(nm)$

**With rolling hash:**
- First window hash: $O(m)$
- Each subsequent window: $O(1)$ (remove one char, add one char)
- Total: $O(m) + O(n - m) = O(n)$

**Improvement factor:** from $O(nm)$ to $O(n)$ — a factor of $m$.

The rolling hash achieves this by exploiting the **overlap** between consecutive windows. Since two adjacent windows share $m-1$ characters, the rolling hash avoids redundantly processing these shared characters. Only the one removed and one added character need to be handled.

This is the same principle behind sliding window algorithms in general — maintain a running aggregate and update it incrementally.

</details>

---

| [← Rabin-Karp Algorithm](01-rabin-karp-algorithm.md) | [Next: Polynomial Hashing →](03-polynomial-hashing.md) |
|:---|---:|

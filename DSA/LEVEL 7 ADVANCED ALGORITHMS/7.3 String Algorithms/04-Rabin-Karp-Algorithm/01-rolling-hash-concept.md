# Chapter 4.1 — Rolling Hash Concept

> **Unit 4: Rabin-Karp Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The Rabin-Karp algorithm replaces character-by-character comparison with
**hash comparison**. The magic is the **rolling hash** — a technique to update
the hash of a sliding window in O(1) time.

---

## 1. The Core Idea

```
  ┌──────────────────────────────────────────────────────┐
  │  Instead of comparing strings character by character │
  │  (O(m) per comparison), compare their HASH VALUES    │
  │  (O(1) per comparison).                              │
  │                                                      │
  │  If hash(window) ≠ hash(pattern) → definitely NO     │
  │  If hash(window) == hash(pattern) → MAYBE (verify!)  │
  └──────────────────────────────────────────────────────┘
```

```
  T = "ABCDABC"    P = "DAB"

  hash(P) = H_p

  Slide a window of size m=3 over T:
  ┌───┬───┬───┐───────────
  │ A │ B │ C │ D  A  B  C     hash("ABC") ≟ H_p?  No
  └───┴───┴───┘
    ┌───┬───┬───┐─────────
    │ B │ C │ D │ A  B  C      hash("BCD") ≟ H_p?  No
    └───┴───┴───┘
      ┌───┬───┬───┐───────
      │ C │ D │ A │ B  C       hash("CDA") ≟ H_p?  No
      └───┴───┴───┘
        ┌───┬───┬───┐─────
        │ D │ A │ B │ C        hash("DAB") ≟ H_p?  YES → verify!
        └───┴───┴───┘
```

---

## 2. Why "Rolling"?

Computing hash from scratch for each window would cost O(m) per window,
giving O(n × m) total — no better than brute force!

```
  The ROLLING hash reuses the previous hash:

  hash(new_window) = f( hash(old_window),
                        character leaving,
                        character entering )

  This update takes O(1)!

  ┌───────────────────────────────────────────────────┐
  │                                                   │
  │  Window slides by 1:                              │
  │                                                   │
  │  Old:  [A  B  C]  D  E                            │
  │  New:   A  [B  C  D]  E                            │
  │                                                   │
  │  hash(BCD) = ROLL(hash(ABC),                      │
  │                   remove = 'A',                   │
  │                   add = 'D')                      │
  │                                                   │
  │  Cost: O(1) instead of O(m)!                      │
  └───────────────────────────────────────────────────┘
```

---

## 3. Rolling Hash Formula

### Polynomial Hash

Treat each character as a digit in base `d` (usually `d = 26` or `d = 256`):

```
  hash("ABC") = A × d² + B × d¹ + C × d⁰   (mod q)

  where d = base, q = large prime modulus
```

### Rolling Update

```
  Old window: s[i..i+m-1]   →  hash_old
  New window: s[i+1..i+m]   →  hash_new

  hash_new = (hash_old - s[i] × d^(m-1)) × d + s[i+m]    (mod q)
              ────────────────────────────  ───   ────────
              remove leftmost char          shift  add new char

  Visual:
  ───────
  hash("ABC") = A×d² + B×d¹ + C×d⁰

  Remove A:    B×d¹ + C×d⁰
  Multiply by d: B×d² + C×d¹
  Add D:       B×d² + C×d¹ + D×d⁰ = hash("BCD")  ✓
```

### Step-by-Step

```
  1. Subtract:  hash -= s[i] × d^(m-1)     // remove outgoing char
  2. Multiply:  hash *= d                   // shift left
  3. Add:       hash += s[i + m]            // add incoming char
  4. Mod:       hash %= q                   // keep in range
```

---

## 4. Numerical Example

```
  d = 10,  q = 13  (small for illustration)

  T = "31415"    P = "14"    m = 2

  hash(P) = 1 × 10 + 4 = 14 mod 13 = 1

  Window [0,1] = "31":  hash = 3×10 + 1 = 31 mod 13 = 5    ≠ 1
  Window [1,2] = "14":  hash = (5 - 3×10)×10 + 4            
                       = (5 - 30)×10 + 4                     
                       = (-25)×10 + 4 = -246 mod 13          
                       Hmm, let's be more careful with mod.

  Let me redo properly:
  ─────────────────────
  d = 10,  q = 13

  hash(P) = (1×10 + 4) mod 13 = 14 mod 13 = 1

  h = d^(m-1) mod q = 10^1 mod 13 = 10

  Window "31": hash = (3×10 + 1) mod 13 = 31 mod 13 = 5     ≠ 1
  
  Roll to "14":
    hash = ((5 - 3×10) × 10 + 4) mod 13
         = ((5 - 30) × 10 + 4) mod 13
         = ((-25) × 10 + 4) mod 13
         = (-246) mod 13
         = (-246 + 19×13) mod 13 = (247 - 246) mod 13 = 1   == 1 ✓ → verify!

  Better approach (avoid negative):
    hash = ((hash - T[i] × h) mod q + q) mod q
    hash = (hash × d + T[i+m]) mod q

  Roll to "14":
    hash = ((5 - 3×10) % 13 + 13) % 13 = ((-25) % 13 + 13) % 13
         = (-12 + 13) % 13 = 1
    hash = (1 × 10 + 4) % 13 = 14 % 13 = 1     ✓ match with hash(P)!
```

---

## 5. Visualization of Rolling

```
  T = "a b c d e f g"     d = 26,  m = 3
       0 1 2 3 4 5 6

  Window 1: [a b c]
    hash = a×26² + b×26 + c

  Window 2:   [b c d]
    Remove a:  hash - a×26²
    Shift:     (hash - a×26²) × 26
    Add d:     result + d
    = b×26² + c×26 + d  ✓

  Window 3:     [c d e]
    Remove b:  hash - b×26²
    Shift:     × 26
    Add e:     + e
    = c×26² + d×26 + e  ✓

  Each update: O(1) — just 3 arithmetic operations + mod!
```

---

## 6. Key Properties

```
  ┌──────────────────────────────────────────────────┐
  │  ✅ Hash mismatch → guaranteed NO match          │
  │  ⚠️ Hash match → POSSIBLE match (verify O(m))    │
  │  ✅ Hash computation: O(m) for first, O(1) after │
  │  ✅ Total: O(n) hash computations                │
  │  ⚠️ Total: O(n + m) avg, O(nm) worst (collisions)│
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Core idea | Compare hashes instead of strings |
| Rolling hash | Update hash in O(1) as window slides |
| Formula | hash_new = (hash_old - leaving × d^(m-1)) × d + entering |
| Modulus | Use large prime q to keep hash values bounded |
| Hash match | Must verify with actual string comparison |
| Hash mismatch | Guaranteed no match — can skip |

---

## ❓ Quick Revision Questions

1. **Why is the hash called "rolling"?**
   <details><summary>Answer</summary>Because it's updated incrementally as the window "rolls" one position — removing one character and adding another — in O(1) time.</details>

2. **What are the three steps of a rolling hash update?**
   <details><summary>Answer</summary>1) Subtract the outgoing character × d^(m-1), 2) Multiply by d (shift), 3) Add the incoming character. All mod q.</details>

3. **Why do we need a modulus q?**
   <details><summary>Answer</summary>Without mod, the hash value grows exponentially large (d^m). The modulus keeps values in a manageable range and prevents overflow.</details>

4. **If two strings have different hashes, can they be equal?**
   <details><summary>Answer</summary>No — different strings can have different hashes, and if hashes differ, the strings definitely differ.</details>

5. **If two strings have the same hash, must they be equal?**
   <details><summary>Answer</summary>No — this is a "collision" or "spurious hit". The strings may differ despite having the same hash, so verification is needed.</details>

---

| [⬅️ Previous Unit: KMP Step-by-Step](../03-KMP-Algorithm/06-step-by-step-example.md) | [Next: Hash Function Design ➡️](02-hash-function-design.md) |
|:---|---:|

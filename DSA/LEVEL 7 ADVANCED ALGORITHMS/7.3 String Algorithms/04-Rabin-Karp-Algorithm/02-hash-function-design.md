# Chapter 4.2 — Hash Function Design

> **Unit 4: Rabin-Karp Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A good hash function minimizes collisions while being fast to compute. This
chapter covers the design choices for polynomial hashing used in Rabin-Karp.

---

## 1. Polynomial Hash Function

```
  🔑  hash(s[0..m-1]) = (s[0]×d^(m-1) + s[1]×d^(m-2) + ... + s[m-1]×d⁰) mod q

  Parameters:
  ─────────────
  d = base (alphabet size or chosen constant)
  q = modulus (large prime number)
```

### How to Choose d

| Choice | Value | When to Use |
|--------|-------|-------------|
| |Σ| (alphabet size) | 26 for lowercase | Common |
| 256 | Full ASCII | Byte strings |
| 31 | Java convention | General purpose |
| Any prime > |Σ| | e.g., 31, 37, 53 | Reduce collisions |

### How to Choose q

```
  Requirements:
  ─────────────
  1. q should be a LARGE PRIME — reduces collision probability
  2. d × q should fit in a machine word (avoid overflow)
  
  Common choices:
  ┌───────────────────────────────────┐
  │  q = 10⁹ + 7   (1000000007)     │ ← most popular in competitive programming
  │  q = 10⁹ + 9   (1000000009)     │
  │  q = 998244353                   │
  │  q = random large prime          │ ← best against adversarial inputs
  └───────────────────────────────────┘

  For 64-bit integers: d × q < 2⁶³ to avoid overflow
    d = 31,  q = 10⁹+7:  31 × 10⁹ ≈ 3.1 × 10¹⁰ < 9.2 × 10¹⁸ ✓
```

---

## 2. Computing the First Hash

```
  hash = 0
  for i = 0 to m-1:
      hash = (hash × d + s[i]) mod q

  Trace for s = "abc", d = 26, q = 101:
  ──────────────────────────────────────
  i=0: hash = (0 × 26 + 0) mod 101 = 0       (a = 0)
  i=1: hash = (0 × 26 + 1) mod 101 = 1       (b = 1)
  i=2: hash = (1 × 26 + 2) mod 101 = 28      (c = 2)

  hash("abc") = 0×26² + 1×26 + 2 = 28 mod 101 = 28  ✓
```

This is **Horner's method** — evaluating a polynomial efficiently:

```
  a×d² + b×d + c = ((a × d + b) × d + c)

  Inner to outer — avoids computing d^k powers separately.
  Time: O(m)
```

---

## 3. Precomputing d^(m-1) mod q

```
  h = d^(m-1) mod q

  We need this to remove the leftmost character during rolling.

  Compute with modular exponentiation:
  h = 1
  for i = 1 to m-1:
      h = (h × d) mod q

  Or use fast power: O(log m)
```

---

## 4. Properties of a Good Hash

```
  ┌──────────────────────────────────────────────────┐
  │  1. UNIFORMITY                                   │
  │     Outputs should be uniformly distributed       │
  │     over [0, q-1]. Each slot equally likely.      │
  │                                                   │
  │  2. DETERMINISM                                   │
  │     Same input → same hash. Always.               │
  │                                                   │
  │  3. EFFICIENCY                                    │
  │     O(1) per rolling update.                      │
  │                                                   │
  │  4. AVALANCHE                                     │
  │     Small input change → big hash change.         │
  │     "abc" and "abd" should have very different     │
  │     hashes.                                       │
  └──────────────────────────────────────────────────┘
```

---

## 5. Collision Probability

```
  With modulus q, two random strings of length m have:

  P(collision) ≈ 1/q

  For n windows checked:
  P(at least one collision) ≈ n/q

  ┌───────────────────────────────────────────────┐
  │  With q = 10⁹+7 and n = 10⁶:                │
  │  P(collision) ≈ 10⁶ / 10⁹ = 0.1% = 1/1000  │
  │                                               │
  │  With double hashing (two different q's):     │
  │  P(collision) ≈ 1/(q₁ × q₂) ≈ 10⁻¹⁸        │
  │  Essentially zero!                            │
  └───────────────────────────────────────────────┘
```

---

## 6. Hash Function Variants

### 6.1 Standard Polynomial

```
  hash = Σ s[i] × d^(m-1-i)  mod q
```

### 6.2 Reverse Polynomial

```
  hash = Σ s[i] × d^i  mod q

  Note: rolling update is different — removing from the right
  is easier with this form.
```

### 6.3 XOR-Based Hash (NOT recommended for Rabin-Karp)

```
  hash = s[0] XOR s[1] XOR ... XOR s[m-1]

  Problems:
  - "ab" and "ba" have the same hash (order-independent)
  - Very high collision rate for text
  - Not suitable for substring matching
```

### 6.4 Sum-Based Hash (NOT recommended)

```
  hash = s[0] + s[1] + ... + s[m-1]

  Same problem: "abc" and "bca" give the same hash.
  Doesn't encode character positions.
```

💡 **Polynomial hashing encodes both characters AND their positions.**

---

## 7. Handling Negative Values

```
  When computing (hash - s[i] × h) mod q:

  The subtraction can produce a negative number!

  Fix:
    hash = ((hash - s[i] × h) % q + q) % q

  In Python: % always returns non-negative (no issue).
  In C/C++/Java: % can return negative → must add q.
```

---

## 8. Code — Hash Function

```python
def compute_hash(s, d=31, q=10**9 + 7):
    """Compute polynomial hash of string s."""
    h = 0
    for c in s:
        h = (h * d + ord(c)) % q
    return h

def compute_power(d, m, q=10**9 + 7):
    """Compute d^(m-1) mod q."""
    h = 1
    for _ in range(m - 1):
        h = (h * d) % q
    return h
```

---

## 📝 Summary Table

| Design Choice | Recommendation |
|---------------|----------------|
| Base d | 31 or 26 (avoid 1 — no positional info) |
| Modulus q | Large prime: 10⁹+7 |
| Hash type | Polynomial (not XOR, not sum) |
| First hash | Horner's method — O(m) |
| Rolling update | O(1) per step |
| Negative handling | Add q before mod in C/Java |
| Collision reduction | Double hashing (two different q values) |

---

## ❓ Quick Revision Questions

1. **Why can't we use XOR as a hash for Rabin-Karp?**
   <details><summary>Answer</summary>XOR is order-independent — "abc" and "cba" would have the same hash, causing excessive collisions.</details>

2. **What is Horner's method?**
   <details><summary>Answer</summary>A way to evaluate a polynomial a×d²+b×d+c as ((a×d+b)×d+c), avoiding explicit power computation. Runs in O(m).</details>

3. **Why should q be a prime number?**
   <details><summary>Answer</summary>Prime modulus distributes hash values more uniformly, reducing collision probability.</details>

4. **What is d^(m-1) mod q used for?**
   <details><summary>Answer</summary>To remove the contribution of the leftmost character when rolling the hash window forward.</details>

5. **How does double hashing reduce collision probability?**
   <details><summary>Answer</summary>Two independent hash functions must both collide simultaneously. Probability ≈ 1/(q₁ × q₂), which is negligibly small.</details>

---

| [⬅️ Previous: Rolling Hash Concept](01-rolling-hash-concept.md) | [Next: Spurious Hits ➡️](03-spurious-hits.md) |
|:---|---:|

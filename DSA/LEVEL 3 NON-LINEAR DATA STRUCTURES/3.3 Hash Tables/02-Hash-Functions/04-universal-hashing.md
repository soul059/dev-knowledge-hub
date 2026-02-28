# 2.4 Universal Hashing

## Unit 2: Hash Functions

---

## The Problem with Fixed Hash Functions

Any single, fixed hash function can be defeated by an adversary who chooses keys that all hash to the same slot, guaranteeing O(n) worst case.

```
╔═══════════════════════════════════════════════════════════════╗
║             THE ADVERSARY PROBLEM                            ║
║                                                              ║
║   Fixed hash: h(k) = k mod 7                                ║
║                                                              ║
║   Adversary picks: 7, 14, 21, 28, 35, 42, 49               ║
║   All hash to 0!                                            ║
║                                                              ║
║   Index:  0          1    2    3    4    5    6              ║
║         ┌──────────┬────┬────┬────┬────┬────┬────┐          ║
║         │7→14→21→  │    │    │    │    │    │    │          ║
║         │28→35→42→ │    │    │    │    │    │    │          ║
║         │49        │    │    │    │    │    │    │          ║
║         └──────────┴────┴────┴────┴────┴────┴────┘          ║
║                                                              ║
║   All operations degrade to O(n)!                           ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Solution: Universal Hashing

**Universal Hashing** randomly selects a hash function from a family of functions at runtime. No adversary can consistently pick "bad" keys because they don't know which function was chosen.

```
╔═══════════════════════════════════════════════════════════════╗
║              UNIVERSAL HASHING CONCEPT                       ║
║                                                              ║
║   Family of hash functions: H = {h₁, h₂, h₃, ..., hₙ}    ║
║                                                              ║
║   At creation time: RANDOMLY pick one function hᵢ           ║
║                                                              ║
║   ┌─────────────────────────────────────┐                    ║
║   │  Hash Function Family H             │                    ║
║   │  ┌────┐ ┌────┐ ┌────┐ ┌────┐       │                    ║
║   │  │ h₁ │ │ h₂ │ │ h₃ │ │ h₄ │ ...   │                    ║
║   │  └────┘ └────┘ └─┬──┘ └────┘       │                    ║
║   └───────────────────┼─────────────────┘                    ║
║                       ▼                                      ║
║              "I'll use h₃ today!"                            ║
║                                                              ║
║   Adversary doesn't know which hᵢ was chosen               ║
║   → Cannot construct worst-case input!                      ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Formal Definition

A family $\mathcal{H}$ of hash functions is **universal** if for any two distinct keys $k_1 \neq k_2$:

$$\Pr_{h \in \mathcal{H}}[h(k_1) = h(k_2)] \leq \frac{1}{m}$$

This means: the probability of collision between any two keys is at most $\frac{1}{m}$ (the best we can hope for with $m$ slots).

---

## Carter-Wegman Universal Family

The most common universal hash family:

$$h_{a,b}(k) = ((a \cdot k + b) \mod p) \mod m$$

Where:
- $p$ = prime number larger than universe of keys
- $a$ = random integer, $1 \leq a \leq p-1$
- $b$ = random integer, $0 \leq b \leq p-1$
- $m$ = table size

```
╔═════════════════════════════════════════════════════════╗
║           CARTER-WEGMAN HASH FAMILY                    ║
║                                                        ║
║   Parameters: p = 17 (prime > max key), m = 5          ║
║                                                        ║
║   Randomly chosen: a = 3, b = 7                        ║
║                                                        ║
║   h(k) = ((3k + 7) mod 17) mod 5                      ║
║                                                        ║
║   h(4)  = ((12 + 7) mod 17) mod 5 = 19 mod 17 mod 5  ║
║         = 2 mod 5 = 2                                  ║
║   h(10) = ((30 + 7) mod 17) mod 5 = 37 mod 17 mod 5  ║
║         = 3 mod 5 = 3                                  ║
║   h(15) = ((45 + 7) mod 17) mod 5 = 52 mod 17 mod 5  ║
║         = 1 mod 5 = 1                                  ║
╚═════════════════════════════════════════════════════════╝
```

---

## Step-by-Step Trace

```
Universal hash family: h(k) = ((a*k + b) mod p) mod m
Parameters: p = 23, m = 7, a = 5, b = 11

Keys: 3, 8, 12, 19, 7

h(3)  = ((5×3 + 11) mod 23) mod 7  = (26 mod 23) mod 7  = 3 mod 7  = 3
h(8)  = ((5×8 + 11) mod 23) mod 7  = (51 mod 23) mod 7  = 5 mod 7  = 5
h(12) = ((5×12 + 11) mod 23) mod 7 = (71 mod 23) mod 7  = 2 mod 7  = 2
h(19) = ((5×19 + 11) mod 23) mod 7 = (106 mod 23) mod 7 = 14 mod 7 = 0
h(7)  = ((5×7 + 11) mod 23) mod 7  = (46 mod 23) mod 7  = 0 mod 7  = 0

Index:  0     1     2     3     4     5     6
      ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
      │19, 7│     │ 12  │  3  │     │  8  │     │
      └─────┴─────┴─────┴─────┴─────┴─────┴─────┘

One collision (19 and 7 at index 0) — expected with 5 keys in 7 slots.
Different a, b values would produce a completely different distribution!
```

---

## Pseudocode

```
CLASS UniversalHashTable:
    FUNCTION init(table_size):
        this.m = table_size
        this.p = next_prime_larger_than(MAX_KEY_VALUE)
        this.a = random(1, this.p - 1)    // Random, non-zero
        this.b = random(0, this.p - 1)    // Random
        this.table = new Array(this.m)
    
    FUNCTION hash(key):
        RETURN ((this.a * key + this.b) MOD this.p) MOD this.m
    
    // On rehash, pick NEW random a, b
    FUNCTION rehash():
        this.a = random(1, this.p - 1)
        this.b = random(0, this.p - 1)
        // Reinsert all elements with new hash
```

---

## Why Universal Hashing Works

```
╔═══════════════════════════════════════════════════════════════════╗
║                 EXPECTED PERFORMANCE                             ║
║                                                                  ║
║   With universal hashing and n keys in m slots:                 ║
║                                                                  ║
║   Expected chain length at any slot = n/m = α (load factor)    ║
║                                                                  ║
║   Expected search time = O(1 + α)                               ║
║                                                                  ║
║   This holds for ANY set of keys (not just "random" keys)!      ║
║   The randomness is in the choice of hash function,             ║
║   not in the keys themselves.                                    ║
║                                                                  ║
║   Key insight: The guarantee is probabilistic over the          ║
║   random choice of h, NOT over the input distribution.          ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: How does universal hashing protect against adversarial inputs? Why can't an attacker force worst-case behavior?**

<details>
<summary>Click to reveal answer</summary>

Universal hashing selects the hash function **randomly at runtime** from a family of functions. The attacker would need to know which specific function was chosen to craft keys that all collide. Since:

1. The function is chosen **after** the hash table is created (or even on each rehash)
2. The attacker doesn't know the random parameters $a$ and $b$
3. For any fixed set of keys, the **expected** number of collisions is bounded by $n/m$

Even if the attacker picks the worst possible keys for function $h_1$, those same keys are likely well-distributed under function $h_2$. The probability of collision between any two keys is at most $1/m$, matching the "ideal random" scenario.

</details>

---

| [← Previous: Multiplication Method](03-multiplication-method.md) | [Next: Cryptographic Hash Functions →](05-cryptographic-hash-functions.md) |
|:---|---:|

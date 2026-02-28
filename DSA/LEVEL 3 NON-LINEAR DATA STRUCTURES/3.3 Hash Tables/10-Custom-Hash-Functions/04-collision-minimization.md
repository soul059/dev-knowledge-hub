# 10.4 Collision Minimization Techniques

## Unit 10: Custom Hash Functions

---

## Goal

Design hash functions that distribute keys as **uniformly** as possible across buckets, minimizing collisions without expensive computation.

```
╔══════════════════════════════════════════════════════════════╗
║  IDEAL: Each bucket gets exactly n/m keys (load factor α)   ║
║                                                              ║
║  ┌───┬───┬───┬───┬───┬───┬───┬───┐                         ║
║  │ 2 │ 2 │ 2 │ 2 │ 2 │ 2 │ 2 │ 2 │  ← Perfect (16 keys,  ║
║  └───┴───┴───┴───┴───┴───┴───┴───┘    8 buckets, α=2)     ║
║                                                              ║
║  REALITY (bad hash):                                        ║
║  ┌───┬───┬───┬───┬───┬───┬───┬───┐                         ║
║  │ 8 │ 0 │ 1 │ 5 │ 0 │ 0 │ 2 │ 0 │  ← Terrible!          ║
║  └───┴───┴───┴───┴───┴───┴───┴───┘                         ║
║                                                              ║
║  REALITY (good hash):                                       ║
║  ┌───┬───┬───┬───┬───┬───┬───┬───┐                         ║
║  │ 2 │ 3 │ 1 │ 2 │ 2 │ 3 │ 1 │ 2 │  ← Close to uniform   ║
║  └───┴───┴───┴───┴───┴───┴───┴───┘                         ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Technique 1: Use ALL Significant Fields

```
BAD: Only use one field
─────────────────────
    hash(Student s) = s.age.hashCode()

    Students aged 20: → bucket X (ALL of them!)
    Students aged 21: → bucket Y
    → Massive clustering by age

GOOD: Combine all identity fields
───────────────────────────────────
    hash(Student s) = polynomial_combine(
        s.name,
        s.age,
        s.id,
        s.department
    )
    → Spread across many buckets
```

---

## Technique 2: Polynomial Accumulation (The Standard)

```
FUNCTION hash(fields[]):
    h = SEED              // Non-zero starting value (e.g., 17)
    FOR each field f:
        h = PRIME * h + hash(f)    // PRIME = 31, 37, or 53
    RETURN h
```

### Why This Works

```
╔══════════════════════════════════════════════════════════════╗
║  The multiplication by PRIME does two things:                ║
║                                                              ║
║  1. POSITION SENSITIVITY                                    ║
║     hash("AB") = 31 * hash('A') + hash('B')                ║
║     hash("BA") = 31 * hash('B') + hash('A')                ║
║     These are DIFFERENT because the multiplied term differs  ║
║                                                              ║
║  2. AVALANCHE EFFECT (partial)                              ║
║     Changing one field affects all subsequent additions     ║
║     because the change gets multiplied into future terms    ║
║                                                              ║
║  h₃ = 31 * (31 * (31 * seed + f₁) + f₂) + f₃              ║
║      = 31³·seed + 31²·f₁ + 31·f₂ + f₃                     ║
║                                                              ║
║  Each field has a DIFFERENT weight: 31², 31¹, 31⁰           ║
║  → order matters, spreading is maximized                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Technique 3: Bit Mixing (Spread the Entropy)

When hash values cluster in certain bit patterns, **bit mixing** redistributes them:

```
// Java HashMap's internal perturbation function:
static int hash(int h) {
    h ^= (h >>> 16);    // XOR high bits into low bits
    return h;
}
```

```
BEFORE mixing:               AFTER mixing:
Key A: 0001 0000 0000 0000   0001 0000 0001 0000
Key B: 0010 0000 0000 0000   0010 0000 0010 0000
Key C: 0011 0000 0000 0000   0011 0000 0011 0000

Low 4 bits (used for bucket index with table size 16):
BEFORE: 0000, 0000, 0000  ← ALL SAME → all collide!
AFTER:  0000, 0000, 0000  → Wait, let's use 20-bit example:

Key A: 0000 0000 0001 0000 0000 0000
Key B: 0000 0000 0010 0000 0000 0000
After XOR with >>> 10:
Key A: 0000 0000 0001 0000 0000 0100  ← low bits changed
Key B: 0000 0000 0010 0000 0000 1000  ← now different!
```

### MurmurHash3 Finalizer (Industry Standard)

```
FUNCTION fmix32(h):
    h ^= h >>> 16
    h *= 0x85ebca6b
    h ^= h >>> 13
    h *= 0xc2b2ae35
    h ^= h >>> 16
    RETURN h
```

```
╔══════════════════════════════════════════════════════════════╗
║  Each step serves a purpose:                                ║
║                                                              ║
║  XOR-shift:  Moves high-bit information into low bits       ║
║  Multiply:   Creates non-linear mixing (avalanche)          ║
║  XOR-shift:  Further diffuses the mixed bits                ║
║  Multiply:   Second round of non-linear mixing              ║
║  XOR-shift:  Final cleanup pass                             ║
║                                                              ║
║  Result: Changing ANY input bit changes ~50% of output      ║
║  bits → near-perfect avalanche property                     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Technique 4: Choose Table Size Wisely

```
╔══════════════════════════════════════════════════════════════╗
║  TABLE SIZE AND COLLISION RELATIONSHIP:                     ║
║                                                              ║
║  Power of 2 (e.g., 16, 32, 64):                            ║
║    bucket = hash & (size - 1)     // Fast bitwise AND       ║
║    ⚠ Only uses LOW bits of hash                             ║
║    ⚠ Vulnerable to patterns in low bits                     ║
║    ✓ Fix: Apply bit mixing before indexing                  ║
║                                                              ║
║  Prime number (e.g., 17, 31, 97):                           ║
║    bucket = hash % size           // Modulo by prime        ║
║    ✓ Uses ALL bits of hash (division mixes bits)            ║
║    ✓ Breaks up patterns in hash values                      ║
║    ⚠ Modulo is slower than bitwise AND                      ║
║                                                              ║
║  MODERN PRACTICE:                                           ║
║    Power-of-2 table + bit-mixing hash function              ║
║    (Java HashMap, Python dict, C++ unordered_map)           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Technique 5: Universal Hashing (Randomized)

Choose hash function **randomly** from a family, preventing adversarial collisions:

```
FAMILY: h_{a,b}(k) = ((a * k + b) mod p) mod m

    p = prime larger than key space
    a = random in [1, p-1]
    b = random in [0, p-1]
    m = table size

GUARANTEE: For any two distinct keys x ≠ y:
    Pr[h(x) = h(y)] ≤ 1/m

    This matches the collision probability of a truly
    random function — can't do better than this!
```

---

## Technique 6: Domain-Specific Hashing

Exploit structure in your data for better distribution:

```
╔══════════════════════════════════════════════════════════════╗
║  EXAMPLE: IP Addresses (e.g., "192.168.1.100")              ║
║                                                              ║
║  BAD: hash the string → slow, treats digits as chars        ║
║                                                              ║
║  GOOD: Pack into a 32-bit integer                           ║
║    hash = (192 << 24) | (168 << 16) | (1 << 8) | 100       ║
║    = unique integer → zero collisions for distinct IPs!     ║
║                                                              ║
║  EXAMPLE: GPS Coordinates                                   ║
║  BAD:  hash = lat + lng  (many points on same sum-line)     ║
║  GOOD: hash = Float.floatToIntBits(lat) * 31               ║
║              + Float.floatToIntBits(lng)                    ║
║                                                              ║
║  EXAMPLE: Short Strings (≤ 8 chars)                         ║
║  Pack chars into a long → perfect hash, no collisions!      ║
║  hash = s[0]<<56 | s[1]<<48 | ... | s[7]                   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Measuring Hash Quality

### Chi-Squared Test

```
FUNCTION chiSquaredTest(table, n, m):
    // n = number of keys, m = number of buckets
    expected = n / m   // ideal keys per bucket
    chiSq = 0
    FOR each bucket i:
        observed = table[i].size()
        chiSq += (observed - expected)² / expected
    RETURN chiSq

    // Good hash: chiSq ≈ m (within ± 2√m)
    // Bad hash:  chiSq >> m
```

### Quick Quality Check

```
Insert 10,000 random keys into 1,000 buckets.
Expected per bucket: 10 ± ~3

GOOD HASH:
  Min bucket size: 3    Max: 19    Std dev: 3.1
  
BAD HASH:
  Min bucket size: 0    Max: 150   Std dev: 25.4
```

---

## Comparison of Techniques

| Technique | Cost | Effectiveness | When to Use |
|-----------|:----:|:---:|-------------|
| Use all fields | Free | High | Always |
| Polynomial accumulation | O(k) | High | Multi-field objects |
| Bit mixing / finalizer | O(1) | High | Power-of-2 tables |
| Prime table size | O(1) | Medium | No bit mixing available |
| Universal hashing | O(1) | Optimal | Adversarial inputs possible |
| Domain-specific | Varies | Highest | Known key structure |
| Hash caching | O(1) amortized | N/A | Immutable keys, repeated lookups |

---

## Quick Revision Question

**Q: A hash table uses power-of-2 sizing (e.g., 16 buckets), and the bucket index is computed as `hash & 15`. Many keys have hash values where the low 4 bits are identical (e.g., all multiples of 16). What technique would fix the resulting collisions, and how does it work?**

<details>
<summary>Click to reveal answer</summary>

**Bit mixing (perturbation)** fixes this. The problem is that `hash & 15` only uses the lowest 4 bits, so keys whose hashes differ only in higher bits all collide.

**The fix — XOR high bits into low bits:**
```
h = key.hashCode();
h ^= (h >>> 16);    // Mix bits 16-31 into bits 0-15
bucket = h & 15;    // Now low bits contain info from ALL 32 bits
```

**Before mixing:**
```
hash(A) = 0x00100000  →  & 0xF = 0  (bucket 0)
hash(B) = 0x00200000  →  & 0xF = 0  (bucket 0) ← COLLISION
hash(C) = 0x00300000  →  & 0xF = 0  (bucket 0) ← COLLISION
```

**After mixing (h ^= h >>> 16):**
```
hash(A) = 0x00100010  →  & 0xF = 0   (bucket 0)
hash(B) = 0x00200020  →  & 0xF = 0   (bucket 0)
                                       ↑ still colliding with simple XOR
```

In practice, MurmurHash3's finalizer (multiply + XOR-shift) achieves near-perfect avalanche: changing any single input bit flips ~50% of output bits, making collisions extremely unlikely even with power-of-2 masking.

</details>

---

| [← Immutability Importance](03-immutability-importance.md) | [Next: Performance Tuning →](05-performance-tuning.md) |
|:---|---:|

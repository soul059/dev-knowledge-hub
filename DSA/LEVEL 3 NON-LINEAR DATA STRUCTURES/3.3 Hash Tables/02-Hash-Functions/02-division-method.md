# 2.2 Division Method

## Unit 2: Hash Functions

---

## Definition

The **Division Method** is the simplest and most widely used hash function. It computes the hash value by taking the **remainder** when the key is divided by the table size.

$$h(k) = k \mod m$$

Where:
- $k$ = key (or integer representation of key)
- $m$ = table size
- $h(k)$ = resulting index (0 to m-1)

---

## How It Works

```
╔═══════════════════════════════════════════════════════════╗
║                 DIVISION METHOD                          ║
║                                                          ║
║   Formula:  h(k) = k mod m                              ║
║                                                          ║
║   Example:  m = 7  (table size)                          ║
║                                                          ║
║   h(14) = 14 mod 7 = 0                                  ║
║   h(21) = 21 mod 7 = 0    ◄── collision with 14         ║
║   h(10) = 10 mod 7 = 3                                  ║
║   h(25) = 25 mod 7 = 4                                  ║
║   h(33) = 33 mod 7 = 5                                  ║
║   h(19) = 19 mod 7 = 5    ◄── collision with 33         ║
║                                                          ║
║   Index:  0     1     2     3     4     5     6          ║
║         ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐     ║
║         │14,21│     │     │ 10  │ 25  │33,19│     │     ║
║         └─────┴─────┴─────┴─────┴─────┴─────┴─────┘     ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Choosing Table Size (m)

The choice of `m` is **critical** for division method performance.

```
╔═════════════════════════════════════════════════════════════════╗
║              CHOOSING TABLE SIZE m                              ║
║                                                                ║
║   ✗ BAD CHOICES:                                               ║
║   ──────────────                                               ║
║   m = power of 2 (e.g., 16, 32, 64)                          ║
║     k mod 2^p = last p bits of k                              ║
║     Ignores higher-order bits → poor distribution             ║
║                                                                ║
║   m = 10, 100, 1000                                           ║
║     Only uses last few digits → ignores most of the key       ║
║                                                                ║
║   m = even number                                              ║
║     Even keys → even index, Odd keys → odd index              ║
║     If keys are all even, half the table is unused!           ║
║                                                                ║
║   ✓ GOOD CHOICES:                                              ║
║   ──────────────                                               ║
║   m = prime number not close to power of 2                    ║
║     Examples: 7, 13, 31, 61, 127, 251, 509, 1021             ║
║     Distributes keys more uniformly                           ║
╚═════════════════════════════════════════════════════════════════╝
```

### Why Primes Work Better

```
Keys: 12, 24, 36, 48, 60, 72

m = 12 (composite):        m = 13 (prime):
h(12) = 0                  h(12) = 12
h(24) = 0  ◄── collision   h(24) = 11
h(36) = 0  ◄── collision   h(36) = 10
h(48) = 0  ◄── collision   h(48) = 9
h(60) = 0  ◄── collision   h(60) = 8
h(72) = 0  ◄── collision   h(72) = 7

All multiples of 12 → ALL   Spread across 6 different
collide at index 0!          buckets! Much better!
```

---

## Step-by-Step Trace

```
Table size m = 11 (prime), Keys: 54, 26, 93, 17, 77, 31

Step 1: h(54) = 54 mod 11 = 10
Step 2: h(26) = 26 mod 11 = 4
Step 3: h(93) = 93 mod 11 = 5     (93 = 8*11 + 5)
Step 4: h(17) = 17 mod 11 = 6
Step 5: h(77) = 77 mod 11 = 0     (77 = 7*11 + 0)
Step 6: h(31) = 31 mod 11 = 9

Final Table:
Index:  0    1    2    3    4    5    6    7    8    9    10
      ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
      │ 77 │    │    │    │ 26 │ 93 │ 17 │    │    │ 31 │ 54 │
      └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘

Result: 6 keys in 11 slots, ZERO collisions! 
Good distribution thanks to prime table size.
```

---

## Pseudocode

```
FUNCTION hash_division(key, table_size):
    // Ensure key is a positive integer
    IF key is string:
        key = convert_to_integer(key)
    
    // Handle negative keys
    index = key MOD table_size
    IF index < 0:
        index = index + table_size
    
    RETURN index

// Finding a good table size
FUNCTION find_prime_table_size(min_size):
    candidate = min_size
    IF candidate is even:
        candidate = candidate + 1    // Start with odd number
    WHILE NOT is_prime(candidate):
        candidate = candidate + 2    // Check odd numbers only
    RETURN candidate
```

---

## Advantages and Disadvantages

| Aspect | Details |
|--------|---------|
| **Pros** | Simple to implement, fast (single modulo operation), works well with prime `m` |
| **Cons** | Sensitive to table size choice, poor with patterned data, doesn't use all bits of large keys |

---

## Quick Revision Question

**Q: Why should the table size `m` in the division method be a prime number? What happens if `m` is a power of 2?**

<details>
<summary>Click to reveal answer</summary>

**Prime table size**: A prime `m` ensures that the modulo operation distributes keys more uniformly because a prime has no common factors with typical key patterns. If keys share a common factor with `m`, they'll cluster into fewer buckets.

**Power of 2 problem**: When `m = 2^p`, the operation `k mod 2^p` only considers the **last p bits** of `k`. All information in the higher-order bits is completely ignored. For example, with `m = 16`, keys 1, 17, 33, 49, 65 all hash to 1 because they share the same last 4 bits. This can lead to severe clustering if keys differ mainly in higher bits.

</details>

---

| [← Previous: Properties of Good Hash Function](01-properties-of-good-hash-function.md) | [Next: Multiplication Method →](03-multiplication-method.md) |
|:---|---:|

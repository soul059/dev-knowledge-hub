# 1.2 Hash Function Concept

## Unit 1: Hashing Fundamentals

---

## What is a Hash Function?

A **hash function** is a mathematical function that maps data of **arbitrary size** to data of **fixed size**. In the context of hash tables, it converts a key into an array index.

```
╔════════════════════════════════════════════════════════════╗
║                   HASH FUNCTION                           ║
║                                                           ║
║         Universe of Keys (infinite)                       ║
║   ┌─────────────────────────────┐                         ║
║   │ "apple" "banana" "cherry"  │       Hash Table         ║
║   │ "date"  "elderberry" ...   │     ┌───┬───┬───┐       ║
║   │ 42  100  -5  3.14  ...     │ ──► │ 0 │ 1 │ 2 │ ...   ║
║   │ (infinite possible keys)   │     └───┴───┴───┘       ║
║   └─────────────────────────────┘     (finite slots)      ║
║                                                           ║
║   Many-to-Few mapping ──► Collisions are inevitable!      ║
╚════════════════════════════════════════════════════════════╝
```

---

## Formal Definition

$$h: U \rightarrow \{0, 1, 2, \ldots, m-1\}$$

Where:
- $U$ = Universe of all possible keys
- $m$ = Size of the hash table
- $h(k)$ = Index where key $k$ should be stored

---

## How Hash Functions Work

### Step-by-Step Process

```
╔═══════════════════════════════════════════════════════╗
║            HASH FUNCTION PIPELINE                    ║
║                                                      ║
║   Step 1: Key ──► Convert to Integer                 ║
║           "cat" ──► 99 + 97 + 116 = 312             ║
║                    (ASCII values summed)              ║
║                                                      ║
║   Step 2: Integer ──► Compress to Range              ║
║           312 ──► 312 % 10 = 2                       ║
║                   (mod table_size)                    ║
║                                                      ║
║   Step 3: Use Index                                  ║
║           table[2] = "cat"                           ║
╚═══════════════════════════════════════════════════════╝
```

---

## Simple Hash Function Examples

### Example 1: Integer Keys

```
Hash Function: h(k) = k % m      (where m = table size)

Table size m = 7

h(14) = 14 % 7 = 0
h(21) = 21 % 7 = 0    ◄── Collision with 14!
h(33) = 33 % 7 = 5
h(10) = 10 % 7 = 3
h(18) = 18 % 7 = 4

Index:  0     1     2     3     4     5     6
      ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
      │  14 │     │     │  10 │  18 │  33 │     │
      └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
```

### Example 2: String Keys (Sum of ASCII)

```
Hash Function: h(s) = (sum of ASCII values) % m

Table size m = 10

h("cat") = (99 + 97 + 116) % 10 = 312 % 10 = 2
h("dog") = (100 + 111 + 103) % 10 = 314 % 10 = 4
h("bat") = (98 + 97 + 116) % 10 = 311 % 10 = 1
h("tab") = (116 + 97 + 98) % 10 = 311 % 10 = 1   ◄── Collision!

Note: "bat" and "tab" collide because they have the same letters!
This is a weakness of simple sum-based hash functions.
```

---

## The Pigeonhole Principle

```
╔═══════════════════════════════════════════════════════════╗
║              PIGEONHOLE PRINCIPLE                        ║
║                                                          ║
║   If you have n pigeons and m holes, where n > m,       ║
║   at least one hole must contain more than one pigeon.   ║
║                                                          ║
║   Keys (pigeons):    🐦 🐦 🐦 🐦 🐦 🐦 🐦              ║
║                       7 keys                             ║
║                                                          ║
║   Slots (holes):     [ ][ ][ ][ ][ ]                    ║
║                       5 slots                            ║
║                                                          ║
║   ──► At least 2 keys MUST share a slot!                ║
║   ──► Collisions are MATHEMATICALLY UNAVOIDABLE         ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Desirable Properties

```
╔══════════════════════════════════════════════════════════════════╗
║                GOOD vs BAD HASH FUNCTIONS                      ║
║                                                                ║
║   GOOD Distribution:          BAD Distribution:                ║
║   ┌───┬───┬───┬───┬───┐      ┌───┬───┬───┬───┬───┐           ║
║   │ 2 │ 2 │ 2 │ 2 │ 2 │      │ 8 │ 1 │ 0 │ 1 │ 0 │          ║
║   └───┴───┴───┴───┴───┘      └───┴───┴───┴───┴───┘           ║
║   Even spread ──► fewer       Clustered ──► many               ║
║   collisions                  collisions                       ║
║                                                                ║
║   │██│██│██│██│██│            │████████│█ │  │█ │  │           ║
║   └──┴──┴──┴──┴──┘            └────────┴──┴──┴──┴──┘          ║
║    0  1  2  3  4               0  1  2  3  4                   ║
╚══════════════════════════════════════════════════════════════════╝
```

| Property | Good Hash Function | Bad Hash Function |
|----------|-------------------|-------------------|
| Distribution | Uniform across all slots | Clusters in few slots |
| Speed | O(1) computation | Slow computation |
| Determinism | Same key → same hash | Inconsistent results |
| Avalanche | Small input change → large output change | Similar inputs → similar outputs |

---

## Pseudocode: Simple Hash Functions

```
// Integer hash (modular)
FUNCTION hash_integer(key, table_size):
    RETURN key MOD table_size

// String hash (sum of characters)
FUNCTION hash_string_simple(key, table_size):
    sum = 0
    FOR each character c in key:
        sum = sum + ASCII(c)
    RETURN sum MOD table_size

// Better string hash (positional weighting)
FUNCTION hash_string_better(key, table_size):
    hash_val = 0
    base = 31
    FOR i = 0 TO length(key) - 1:
        hash_val = hash_val * base + ASCII(key[i])
    RETURN hash_val MOD table_size
```

---

## Trace: Positional String Hash

```
h("cat") with base = 31, table_size = 10

Step 1: hash_val = 0
Step 2: hash_val = 0 * 31 + 99  = 99       ('c' = 99)
Step 3: hash_val = 99 * 31 + 97 = 3166     ('a' = 97)
Step 4: hash_val = 3166 * 31 + 116 = 98262 ('t' = 116)
Step 5: 98262 % 10 = 2

h("cat") = 2

Now try "act" (same letters, different order):
Step 1: hash_val = 0
Step 2: hash_val = 0 * 31 + 97  = 97       ('a' = 97)
Step 3: hash_val = 97 * 31 + 99 = 3106     ('c' = 99)
Step 4: hash_val = 3106 * 31 + 116 = 96402 ('t' = 116)
Step 5: 96402 % 10 = 2

h("act") = 2   ◄── Still collides here, but in general
                    positional hashing differentiates anagrams
                    much better than simple sum
```

---

## Quick Revision Question

**Q: Why does summing ASCII values make a poor hash function for strings? What improvement addresses this?**

<details>
<summary>Click to reveal answer</summary>

Summing ASCII values is poor because **anagrams** (words with the same letters in different order like "cat" and "act") produce the **same hash value**. The position of characters is ignored, leading to unnecessary collisions. 

The improvement is **positional/polynomial hashing**, where each character's contribution is multiplied by a base raised to the power of its position: $h = \sum c_i \times b^i$. This ensures that character order affects the hash value.

</details>

---

| [← Previous: What is Hashing?](01-what-is-hashing.md) | [Next: Hash Table Structure →](03-hash-table-structure.md) |
|:---|---:|

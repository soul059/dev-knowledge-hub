# Chapter 9.5: Bloom Filter Bits

> **Unit 9 — Practical Applications**
> Build a space-efficient probabilistic data structure using bit arrays and multiple hash functions.

---

## Overview

A Bloom filter answers "Is element X in the set?" with:
- **Definite NO** (if any bit is 0) — guaranteed correct
- **Probably YES** (if all bits are 1) — may have false positives

It uses a bit array and k hash functions — pure bit manipulation at its core.

---

## Structure

```
  ┌───────────────────────────────────────────────┐
  │  Bit array of m bits, initially all zeros     │
  │  k independent hash functions                 │
  │  No deletions (basic version)                 │
  └───────────────────────────────────────────────┘

  m = 16, k = 3:
  Index:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
  Bits:  [0][0][0][0][0][0][0][0][0][0][0][0][0][0][0][0]
```

---

## Operations

### Insert

```
FUNCTION insert(bloomFilter, element):
    FOR i = 0 TO k-1:
        index = hash_i(element) % m
        bloomFilter |= (1 << index)    // Set bit at index
```

### Query

```
FUNCTION query(bloomFilter, element):
    FOR i = 0 TO k-1:
        index = hash_i(element) % m
        IF NOT (bloomFilter & (1 << index)):   // Bit not set?
            RETURN "DEFINITELY NOT IN SET"
    RETURN "PROBABLY IN SET"
```

---

## Trace: Insert "apple" and "banana", Query "cherry"

```
  m = 16, k = 3

  hash functions (example outputs):
    h1("apple") = 2,  h2("apple") = 7,  h3("apple") = 13
    h1("banana")= 5,  h2("banana")= 7,  h3("banana")= 11
    h1("cherry")= 2,  h2("cherry")= 5,  h3("cherry")= 11

  Insert "apple": set bits 2, 7, 13
  0000000000000000
         ↓    ↓        ↓
  0010000010000010
   bit: 2    7       13

  Insert "banana": set bits 5, 7, 11
  0010000010000010
            ↓ (already set)
  0010010010010010
        5    7   11  

  Final: 0010010010010010
          bits set: 2, 5, 7, 11, 13

  Query "cherry": check bits 2, 5, 11
    bit 2:  set ✓
    bit 5:  set ✓
    bit 11: set ✓
    Result: "PROBABLY IN SET"  ← FALSE POSITIVE!
    (cherry was never inserted, but its hash positions
     were all set by other elements)
```

---

## False Positive Rate

```
  After inserting n elements into m-bit filter with k hash functions:

  Probability a specific bit is still 0:
    (1 - 1/m)^(kn) ≈ e^(-kn/m)

  False positive rate:
    fp ≈ (1 - e^(-kn/m))^k

  Optimal k (minimizes fp):
    k_opt = (m/n) × ln(2) ≈ 0.693 × (m/n)
```

### Example Calculation

```
  m = 1000 bits, n = 100 elements
  k_opt = 0.693 × (1000/100) = 6.93 → k = 7

  fp ≈ (1 - e^(-7×100/1000))^7
     = (1 - e^(-0.7))^7
     = (1 - 0.497)^7
     = (0.503)^7
     ≈ 0.0082 ≈ 0.82%

  Less than 1% false positive rate with only 1000 bits!
```

---

## Bit Array Implementation

### Using Integer Array as Bit Array

```
FUNCTION setBit(array[], index):
    word = index / 32           // which int
    bit  = index % 32           // which bit within int
    array[word] |= (1 << bit)

FUNCTION getBit(array[], index):
    word = index / 32
    bit  = index % 32
    RETURN (array[word] >> bit) & 1

FUNCTION clearBit(array[], index):
    word = index / 32
    bit  = index % 32
    array[word] &= ~(1 << bit)
```

### Memory Layout

```
  m = 128 bits → 4 integers (32-bit each) → 16 bytes

  array[0]: bits  0-31
  array[1]: bits 32-63
  array[2]: bits 64-95
  array[3]: bits 96-127

  Bit 45 → array[1], bit 13
    (45 / 32 = 1, 45 % 32 = 13)
```

---

## Sizing Guide

```
  ┌──────────┬──────────┬──────────┬──────────────┐
  │ n        │ m (bits) │ k        │ fp rate      │
  ├──────────┼──────────┼──────────┼──────────────┤
  │ 1,000    │ 10,000   │ 7        │ 0.82%        │
  │ 10,000   │ 100,000  │ 7        │ 0.82%        │
  │ 1,000,000│ 10M      │ 7        │ 0.82%        │
  │ 1,000    │ 5,000    │ 3        │ 3.6%         │
  │ 1,000    │ 20,000   │ 14       │ 0.01%        │
  └──────────┴──────────┴──────────┴──────────────┘

  Rule of thumb: ~10 bits per element for ~1% fp rate
```

---

## Real-World Uses

```
  ┌──────────────────────────────────────────────────────┐
  │ • Web browsers:  check if URL is in malware database│
  │ • Databases:     skip disk reads for missing keys   │
  │ • CDN caches:    check if object might be cached    │
  │ • Spell check:   quick dictionary membership test   │
  │ • Network:       dedup packets, routing tables      │
  └──────────────────────────────────────────────────────┘
```

---

## Quick Revision Questions

1. **Can a Bloom filter produce false negatives?**
   <details><summary>Answer</summary>No. If an element was inserted, all its hash positions were set to 1 and bits are never cleared. A query will always find all bits set, so the answer is always "probably yes." False negatives are impossible.</details>

2. **Why can't you delete from a basic Bloom filter?**
   <details><summary>Answer</summary>Clearing a bit might affect other elements whose hash functions also map to that position. A deletion could cause false negatives for other elements. Counting Bloom filters solve this by using counters instead of single bits.</details>

3. **How do you choose the number of hash functions k?**
   <details><summary>Answer</summary>k_opt = (m/n) × ln(2) ≈ 0.693 × (m/n). Too few: bits aren't spread well, higher fp rate. Too many: too many bits set per element, also higher fp rate. The optimum balances these.</details>

4. **How is a bit array more efficient than a boolean array?**
   <details><summary>Answer</summary>A boolean array uses 1 byte per element (8 bits), while a bit array uses 1 bit. For m=1,000,000: boolean array = 1 MB, bit array = 125 KB — 8× less memory. Bit arrays use shifts and masks to access individual bits within integer words.</details>

5. **What is the space advantage of a Bloom filter vs. a hash set?**
   <details><summary>Answer</summary>A hash set stores actual elements plus overhead (pointers, load factor). For strings, this can be 50-100+ bytes per element. A Bloom filter uses ~10 bits (~1.25 bytes) per element for 1% fp rate — roughly 40-80× less memory.</details>

---

[← Previous: Error Detection](04-error-detection.md) | [Next: Graphics Basics →](06-graphics-basics.md)

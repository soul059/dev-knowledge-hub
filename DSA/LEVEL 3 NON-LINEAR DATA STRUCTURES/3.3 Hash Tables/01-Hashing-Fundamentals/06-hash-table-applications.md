# 1.6 Hash Table Applications

## Unit 1: Hashing Fundamentals

---

## Overview

Hash tables are one of the most versatile data structures, used in nearly every area of computer science. This topic covers real-world applications, language implementations, and practical use cases.

---

## Real-World Applications

```
╔════════════════════════════════════════════════════════════════╗
║              HASH TABLES IN THE REAL WORLD                    ║
║                                                               ║
║   1. DATABASE INDEXING                                        ║
║      ┌─────────┐     ┌──────────────────┐                    ║
║      │ Query:  │     │  Hash Index      │                    ║
║      │ id=1234 │──►  │  1234 → row ptr  │──► Data Row       ║
║      └─────────┘     └──────────────────┘                    ║
║                                                               ║
║   2. CACHING (Redis, Memcached)                              ║
║      ┌─────────┐     ┌──────────────────┐                    ║
║      │ Request │     │  Cache (HashMap) │                    ║
║      │ page/42 │──►  │  "/42" → HTML    │──► Fast Response   ║
║      └─────────┘     └──────────────────┘                    ║
║                                                               ║
║   3. SYMBOL TABLES (Compilers)                               ║
║      ┌─────────┐     ┌──────────────────┐                    ║
║      │ Code:   │     │  Symbol Table    │                    ║
║      │ int x;  │──►  │  "x" → {int,4B} │                    ║
║      └─────────┘     └──────────────────┘                    ║
║                                                               ║
║   4. DNS RESOLUTION                                          ║
║      ┌─────────────┐  ┌──────────────────────┐              ║
║      │ google.com  │──►│ "google.com"→142.x.x │             ║
║      └─────────────┘  └──────────────────────┘              ║
╚════════════════════════════════════════════════════════════════╝
```

---

## Language-Specific Implementations

| Language | Hash Map | Hash Set | Notes |
|----------|----------|----------|-------|
| **Java** | `HashMap<K,V>` | `HashSet<E>` | Also `LinkedHashMap`, `ConcurrentHashMap` |
| **Python** | `dict` | `set` | Built-in, highly optimized |
| **C++** | `unordered_map` | `unordered_set` | STL containers |
| **JavaScript** | `Map`, `Object` | `Set` | `Map` preserves insertion order |
| **C#** | `Dictionary<K,V>` | `HashSet<T>` | .NET collections |
| **Go** | `map[K]V` | — | Built-in map type |
| **Rust** | `HashMap<K,V>` | `HashSet<T>` | From `std::collections` |

---

## Application Categories

### 1. Frequency Counting

```
Problem: Count word frequencies in a document

Input:  "the cat and the dog and the fish"

Hash Map State:
┌──────────┬───────┐
│   Key    │ Count │
├──────────┼───────┤
│ "the"    │   3   │
│ "cat"    │   1   │
│ "and"    │   2   │
│ "dog"    │   1   │
│ "fish"   │   1   │
└──────────┴───────┘

FUNCTION count_words(text):
    freq = new HashMap()
    FOR each word in text:
        IF freq.contains(word):
            freq[word] = freq[word] + 1
        ELSE:
            freq[word] = 1
    RETURN freq
```

### 2. Deduplication

```
Problem: Remove duplicates from [4, 2, 7, 2, 4, 9, 7]

Using Hash Set:
  Step 1: Add 4 → {4}
  Step 2: Add 2 → {4, 2}
  Step 3: Add 7 → {4, 2, 7}
  Step 4: Add 2 → {4, 2, 7}       ◄── Already exists, skip
  Step 5: Add 4 → {4, 2, 7}       ◄── Already exists, skip
  Step 6: Add 9 → {4, 2, 7, 9}
  Step 7: Add 7 → {4, 2, 7, 9}    ◄── Already exists, skip

Result: {4, 2, 7, 9}    O(n) total
```

### 3. Two-Sum Pattern

```
Problem: Find two numbers that add up to target = 9
Input:  [2, 7, 11, 15]

Hash Map tracks: {value → index}

Step 1: num=2, need 9-2=7, map={} → not found → map={2:0}
Step 2: num=7, need 9-7=2, map={2:0} → FOUND at index 0!

Answer: indices [0, 1]     O(n) using hash map
```

### 4. Caching / Memoization

```
Problem: Fibonacci with memoization

Without cache:  fib(40) → ~1 billion recursive calls
With hash map:  fib(40) → 40 lookups + 40 computations

FUNCTION fib(n, cache):
    IF n in cache:
        RETURN cache[n]          // O(1) lookup
    IF n <= 1:
        RETURN n
    result = fib(n-1, cache) + fib(n-2, cache)
    cache[n] = result            // O(1) store
    RETURN result
```

---

## Application Summary Table

| Application | Key | Value | Why Hash Table? |
|-------------|-----|-------|-----------------|
| Phone book | Name | Number | O(1) name lookup |
| Spell checker | Word | Boolean | O(1) "is valid word?" |
| Cache | URL/Query | Response | O(1) cached result |
| Compiler symbols | Variable name | Type, scope | O(1) symbol lookup |
| Routing table | IP prefix | Next hop | O(1) route lookup |
| Counting | Element | Frequency | O(1) increment |
| Deduplication | Element | — (set) | O(1) membership check |
| Graph adjacency | Node | Neighbors list | O(1) neighbor lookup |

---

## Quick Revision Question

**Q: Name three different application categories where hash tables provide significant performance advantages, and explain why each benefits from O(1) operations.**

<details>
<summary>Click to reveal answer</summary>

1. **Caching/Memoization**: Web servers and compute-heavy functions need to check if a result is already cached before recomputing. O(1) lookup means cache checks add negligible overhead, making the cache worthwhile.

2. **Frequency Counting**: When processing large text or data streams, incrementing a counter for each item needs to be fast. With O(1) lookup and update, counting n items takes O(n) total — any slower structure would make this impractical for large datasets.

3. **Deduplication/Membership Testing**: Checking if an element has been seen before (duplicate detection, spell checking, blacklists) occurs millions of times in processing pipelines. O(1) membership testing keeps the total processing time linear.

</details>

---

| [← Previous: Why Hashing?](05-why-hashing.md) | [Unit 2: Properties of Good Hash Function →](../02-Hash-Functions/01-properties-of-good-hash-function.md) |
|:---|---:|

# 1.1 What is Hashing?

## Unit 1: Hashing Fundamentals

---

## Introduction

**Hashing** is the process of converting an input (called a **key**) into a fixed-size numerical value (called a **hash code** or **hash value**) using a mathematical function (called a **hash function**). This hash value is then used as an index to store and retrieve data in constant time.

```
╔═══════════════════════════════════════════════════════════════╗
║                    THE HASHING PROCESS                       ║
║                                                              ║
║   Input Key         Hash Function         Hash Value         ║
║   ─────────         ─────────────         ──────────         ║
║   "apple"    ──►    h("apple")     ──►    3                  ║
║   "banana"   ──►    h("banana")    ──►    7                  ║
║   "cherry"   ──►    h("cherry")    ──►    1                  ║
║   42         ──►    h(42)          ──►    2                  ║
║   3.14       ──►    h(3.14)        ──►    4                  ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Real-World Analogy: Library Catalog

Think of a **library** with thousands of books. Without any system, finding a specific book means checking every shelf — an **O(n)** operation. A **catalog system** assigns each book a shelf number based on its title, so you can go directly to the right shelf.

```
╔══════════════════════════════════════════════════════════╗
║                  LIBRARY ANALOGY                        ║
║                                                        ║
║   Book Title          Catalog System       Shelf #     ║
║   ──────────          ──────────────       ───────     ║
║   "Data Structures"   ──► catalog() ──►   Shelf 5     ║
║   "Algorithms"        ──► catalog() ──►   Shelf 2     ║
║   "Networks"          ──► catalog() ──►   Shelf 8     ║
║                                                        ║
║   Finding a book:  O(1) instead of O(n)!              ║
╚══════════════════════════════════════════════════════════╝
```

Hashing works exactly the same way — the hash function is your "catalog system," and the hash value tells you exactly where to store or find data.

---

## Key Properties of Hashing

| Property | Description |
|----------|-------------|
| **Deterministic** | Same input always produces the same output |
| **Fixed Output Size** | Output size is constant regardless of input size |
| **Efficient** | Computing the hash should be fast — O(1) |
| **Uniform Distribution** | Outputs should be spread evenly across the range |

```
╔═══════════════════════════════════════════════════════════╗
║              DETERMINISTIC PROPERTY                      ║
║                                                          ║
║   h("hello") = 5    ◄── Always 5, never changes         ║
║   h("hello") = 5    ◄── Call again, still 5              ║
║   h("hello") = 5    ◄── Always the same!                 ║
║                                                          ║
║   h("world") = 9    ◄── Different input, different hash  ║
║   h("world") = 9    ◄── But consistent for same input    ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Hashing vs Other Lookups

```
╔═══════════════════════════════════════════════════════════════════╗
║               HOW DATA LOOKUP APPROACHES COMPARE                 ║
║                                                                  ║
║   Linear Search:  [ ][ ][ ][ ][✓][ ][ ][ ]   Check one by one  ║
║                    1  2  3  4  5              O(n)               ║
║                                                                  ║
║   Binary Search:  [ ][ ][ ][✓][ ][ ][ ][ ]   Divide and conquer║
║                          ↗  ↑  ↘              O(log n)          ║
║                                                                  ║
║   Hashing:        Key ──► h(key) ──► [✓]      Direct access     ║
║                                               O(1) average      ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## Simple Example: Student ID Lookup

```
Given: Store student records for IDs: 1234, 5678, 9012

Hash Function: h(id) = id % 10   (last digit)

h(1234) = 1234 % 10 = 4
h(5678) = 5678 % 10 = 8
h(9012) = 9012 % 10 = 2

Table (size 10):
Index:  0    1    2    3    4    5    6    7    8    9
      ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
      │    │    │9012│    │1234│    │    │    │5678│    │
      └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
                 ▲         ▲                   ▲
                 │         │                   │
              h(9012)=2  h(1234)=4          h(5678)=8

To find student 5678:
  1. Compute h(5678) = 8
  2. Go directly to index 8
  3. Found! ──► O(1)
```

---

## The Collision Problem (Preview)

What happens when two different keys produce the **same hash value**?

```
h("cat") = 3
h("dog") = 3    ◄── COLLISION! Both map to index 3

     Index:  0    1    2    3    4    5
           ┌────┬────┬────┬────┬────┬────┐
           │    │    │    │ ?? │    │    │   Who goes in index 3?
           └────┴────┴────┴────┴────┴────┘
                           ▲
                      "cat" AND "dog"
                      both want this slot!
```

> **Note**: Collisions are **inevitable** (as explained by the Pigeonhole Principle). We study collision resolution in Units 3 and 4.

---

## Pseudocode: Basic Hashing Concept

```
FUNCTION hash(key, table_size):
    hash_value = compute_hash(key)
    index = hash_value MOD table_size
    RETURN index

// Example usage
table_size = 10
index = hash("apple", 10)    // Returns some index 0-9
table[index] = "apple"       // Store at computed index
```

---

## Quick Revision Question

**Q: Why is hashing preferred over binary search for lookups in many applications?**

<details>
<summary>Click to reveal answer</summary>

Hashing provides **O(1) average-case** lookup time, while binary search provides **O(log n)**. For large datasets, this difference is significant. Additionally, hashing does not require the data to be sorted, unlike binary search. However, binary search is preferred when order-based operations (like range queries) are needed.

</details>

---

| [← README](../README.md) | [Next: Hash Function Concept →](02-hash-function-concept.md) |
|:---|---:|

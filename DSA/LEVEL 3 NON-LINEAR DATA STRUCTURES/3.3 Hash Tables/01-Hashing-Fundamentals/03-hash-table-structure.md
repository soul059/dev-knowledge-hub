# 1.3 Hash Table Structure

## Unit 1: Hashing Fundamentals

---

## What is a Hash Table?

A **Hash Table** (also called a **Hash Map**) is a data structure that implements an associative array — a structure that maps **keys** to **values**. It uses a hash function to compute an index into an array of **buckets** or **slots**, from which the desired value can be found.

```
╔══════════════════════════════════════════════════════════════╗
║                   HASH TABLE ANATOMY                        ║
║                                                             ║
║   Key-Value Pair       Hash Function         Bucket Array   ║
║   ──────────────       ─────────────         ────────────   ║
║                                                             ║
║   ("name", "Alice")                          Index  Value   ║
║         │                                    ┌───┬────────┐ ║
║         ▼                                  0 │   │        │ ║
║     h("name") = 3  ─────────────────────►  1 │   │        │ ║
║                                            2 │   │        │ ║
║   ("age", 25)                              3 │ ● │"Alice" │ ║
║         │                                  4 │   │        │ ║
║         ▼                                  5 │ ● │  25    │ ║
║     h("age") = 5   ─────────────────────►  6 │   │        │ ║
║                                            7 │   │        │ ║
║                                              └───┴────────┘ ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Core Components

```
╔════════════════════════════════════════════════════════╗
║              HASH TABLE COMPONENTS                    ║
║                                                       ║
║   1. BUCKET ARRAY                                     ║
║      ┌────┬────┬────┬────┬────┬────┬────┐            ║
║      │ B0 │ B1 │ B2 │ B3 │ B4 │ B5 │ B6 │            ║
║      └────┴────┴────┴────┴────┴────┴────┘            ║
║      Fixed-size array of storage slots                ║
║                                                       ║
║   2. HASH FUNCTION                                    ║
║      h(key) → index                                  ║
║      Maps keys to valid array indices                 ║
║                                                       ║
║   3. KEY-VALUE PAIRS                                  ║
║      (key, value) stored at bucket[h(key)]           ║
║                                                       ║
║   4. COLLISION HANDLING MECHANISM                     ║
║      Strategy for when h(k1) = h(k2)                ║
╚════════════════════════════════════════════════════════╝
```

### Component Summary Table

| Component | Role | Example |
|-----------|------|---------|
| **Bucket Array** | Storage container | Array of size `m` |
| **Hash Function** | Key → Index mapping | `h(k) = k % m` |
| **Key** | Unique identifier | `"name"`, `42` |
| **Value** | Associated data | `"Alice"`, `[1,2,3]` |
| **Load Factor** | Fullness ratio | `α = n/m` |

---

## Memory Layout

```
╔══════════════════════════════════════════════════════════╗
║               MEMORY REPRESENTATION                     ║
║                                                         ║
║   Hash Table Object                                     ║
║   ┌─────────────────────┐                               ║
║   │ size: 3             │  ◄── number of entries        ║
║   │ capacity: 8         │  ◄── array length             ║
║   │ load_factor: 0.375  │  ◄── size / capacity          ║
║   │ array: ──────────┐  │                               ║
║   └──────────────────┼──┘                               ║
║                      ▼                                   ║
║   ┌────┬────┬────┬────┬────┬────┬────┬────┐             ║
║   │null│null│ ●  │null│ ●  │null│null│ ●  │             ║
║   └────┴────┴─┬──┴────┴─┬──┴────┴────┴─┬──┘            ║
║     0    1    2    3    4    5    6    7                  ║
║               │         │              │                 ║
║               ▼         ▼              ▼                 ║
║          ("b",20)  ("a",10)       ("c",30)              ║
╚══════════════════════════════════════════════════════════╝
```

---

## Basic Hash Table Pseudocode

```
CLASS HashTable:
    FUNCTION init(capacity):
        this.capacity = capacity
        this.size = 0
        this.table = new Array(capacity)   // Initialize all slots to null
        
    FUNCTION hash(key):
        RETURN key.hashCode() MOD this.capacity
    
    FUNCTION put(key, value):
        index = this.hash(key)
        this.table[index] = (key, value)   // Simple version — no collision handling
        this.size = this.size + 1
    
    FUNCTION get(key):
        index = this.hash(key)
        entry = this.table[index]
        IF entry != null AND entry.key == key:
            RETURN entry.value
        RETURN null
    
    FUNCTION remove(key):
        index = this.hash(key)
        IF this.table[index] != null AND this.table[index].key == key:
            this.table[index] = null
            this.size = this.size - 1
            RETURN true
        RETURN false
```

---

## Trace: Building a Hash Table

```
Operations: put("apple", 5), put("banana", 3), put("cherry", 7)
Hash: h(key) = len(key) % 8

Step 1: put("apple", 5)
  h("apple") = 5 % 8 = 5
  table[5] = ("apple", 5)
  
  ┌────┬────┬────┬────┬────┬──────────┬────┬────┐
  │    │    │    │    │    │("apple",5)│    │    │
  └────┴────┴────┴────┴────┴──────────┴────┴────┘
    0    1    2    3    4       5        6    7

Step 2: put("banana", 3)
  h("banana") = 6 % 8 = 6
  table[6] = ("banana", 3)
  
  ┌────┬────┬────┬────┬────┬──────────┬───────────┬────┐
  │    │    │    │    │    │("apple",5)│("banana",3)│    │
  └────┴────┴────┴────┴────┴──────────┴───────────┴────┘
    0    1    2    3    4       5           6         7

Step 3: put("cherry", 7)
  h("cherry") = 6 % 8 = 6    ◄── COLLISION with "banana"!
  
  In this simple version, "cherry" overwrites "banana"
  (real implementations use collision resolution)
```

---

## Hash Table vs Other Data Structures

| Feature | Array | Linked List | BST | Hash Table |
|---------|:-----:|:-----------:|:---:|:----------:|
| Search | O(n) | O(n) | O(log n) | **O(1)** avg |
| Insert | O(1)* | O(1) | O(log n) | **O(1)** avg |
| Delete | O(n) | O(n) | O(log n) | **O(1)** avg |
| Ordered | Yes | No | Yes | **No** |
| Space | O(n) | O(n) | O(n) | O(n) |

*\* Array insert at end only*

---

## Quick Revision Question

**Q: What are the four essential components of a hash table, and what role does each play?**

<details>
<summary>Click to reveal answer</summary>

1. **Bucket Array**: The underlying fixed-size array that stores the data. Each position is a "bucket" that can hold one or more entries.
2. **Hash Function**: A function that maps keys to valid array indices (0 to m-1). It determines where each key-value pair is stored.
3. **Key-Value Pairs**: The actual data stored in the table. The key is the unique identifier, and the value is the associated data.
4. **Collision Handling Mechanism**: A strategy (chaining or open addressing) to handle cases when two different keys map to the same index.

</details>

---

| [← Previous: Hash Function Concept](02-hash-function-concept.md) | [Next: Direct Addressing →](04-direct-addressing.md) |
|:---|---:|

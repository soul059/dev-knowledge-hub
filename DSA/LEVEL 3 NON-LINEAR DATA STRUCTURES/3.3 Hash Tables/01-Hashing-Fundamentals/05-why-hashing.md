# 1.5 Why Hashing?

## Unit 1: Hashing Fundamentals

---

## The Search Problem

Every program that manages data must solve one fundamental problem: **how to find a specific item quickly?**

```
╔══════════════════════════════════════════════════════════════════╗
║              DATA LOOKUP APPROACHES COMPARED                    ║
║                                                                 ║
║   Unsorted Array     Sorted Array       BST            Hashing  ║
║   ┌─┬─┬─┬─┬─┬─┐    ┌─┬─┬─┬─┬─┬─┐    ┌──┐           Key→h()  ║
║   │5│2│8│1│9│3│    │1│2│3│5│8│9│       │5 │            │       ║
║   └─┴─┴─┴─┴─┴─┘    └─┴─┴─┴─┴─┴─┘    / \            ▼       ║
║   Scan each one     Jump to middle   │2│ │8│      table[i]    ║
║    one by one        and halve       / \   \                   ║
║                                    │1│ │3│ │9│                  ║
║                                                                 ║
║   O(n)              O(log n)        O(log n)       O(1) avg    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Performance Comparison

| n (items) | Linear O(n) | Binary O(log n) | Hashing O(1) |
|:---------:|:-----------:|:---------------:|:------------:|
| 10 | 10 ops | 4 ops | **1 op** |
| 100 | 100 ops | 7 ops | **1 op** |
| 1,000 | 1,000 ops | 10 ops | **1 op** |
| 1,000,000 | 1,000,000 ops | 20 ops | **1 op** |
| 1,000,000,000 | 1,000,000,000 ops | 30 ops | **1 op** |

> At **1 billion** items, hashing is **1 billion times faster** than linear search and **30 times faster** than binary search!

---

## Comprehensive Comparison

```
╔═══════════════════════════════════════════════════════════════════╗
║                OPERATION COMPLEXITY COMPARISON                   ║
║                                                                  ║
║   Operation    │ Array │ Sorted │  BST     │ Hash Table          ║
║                │       │ Array  │(balanced)│                     ║
║   ─────────────┼───────┼────────┼──────────┼──────────           ║
║   Search       │ O(n)  │O(log n)│ O(log n) │ O(1) avg           ║
║   Insert       │ O(1)* │ O(n)  │ O(log n) │ O(1) avg           ║
║   Delete       │ O(n)  │ O(n)  │ O(log n) │ O(1) avg           ║
║   Min / Max    │ O(n)  │ O(1)  │ O(log n) │ O(n)               ║
║   Range Query  │ O(n)  │O(log n)│ O(log n) │ O(n)               ║
║   Ordered Iter │ O(n·lg)│ O(n)  │ O(n)     │ O(n·lg)            ║
║   ─────────────┴───────┴────────┴──────────┴──────────           ║
║   * Array insert at end only                                     ║
╚═══════════════════════════════════════════════════════════════════╝
```

---

## When TO Use Hash Tables

```
╔════════════════════════════════════════════════════════════╗
║              ✓  IDEAL USE CASES                          ║
║                                                           ║
║   1. FAST LOOKUPS                                         ║
║      "Is this username taken?"                            ║
║      "What's the price of item X?"                        ║
║      → O(1) key-based lookup                             ║
║                                                           ║
║   2. COUNTING / FREQUENCY                                 ║
║      "How many times does each word appear?"              ║
║      → O(1) increment per element                        ║
║                                                           ║
║   3. DEDUPLICATION                                        ║
║      "Remove all duplicates from this list"               ║
║      → O(1) membership check                             ║
║                                                           ║
║   4. CACHING                                              ║
║      "Store recently computed results"                    ║
║      → O(1) cache lookup and store                       ║
║                                                           ║
║   5. GROUPING / INDEXING                                  ║
║      "Group students by grade"                            ║
║      → O(1) bucket assignment                            ║
╚════════════════════════════════════════════════════════════╝
```

---

## When NOT to Use Hash Tables

```
╔════════════════════════════════════════════════════════════╗
║              ✗  AVOID HASH TABLES WHEN                   ║
║                                                           ║
║   1. ORDERED DATA NEEDED                                  ║
║      "Give me all items between 10 and 50"                ║
║      → Hash tables don't maintain order                  ║
║      → Use BST or sorted array instead                   ║
║                                                           ║
║   2. MIN / MAX QUERIES                                    ║
║      "What's the smallest element?"                       ║
║      → O(n) in hash table vs O(1) in heap               ║
║      → Use heap or BST instead                           ║
║                                                           ║
║   3. SMALL DATASETS                                       ║
║      Only 5-10 elements?                                  ║
║      → Linear search is fine; hash overhead not worth it ║
║                                                           ║
║   4. MEMORY-CRITICAL SYSTEMS                              ║
║      → Hash tables use extra memory (load factor < 1)    ║
║      → Sorted array is more memory-efficient             ║
║                                                           ║
║   5. WORST-CASE GUARANTEES REQUIRED                       ║
║      → Hash table worst case is O(n)                     ║
║      → Balanced BST guarantees O(log n)                  ║
╚════════════════════════════════════════════════════════════╝
```

---

## Decision Flowchart

```
                    Need to store key-value data?
                              │
                    ┌─────────┴─────────┐
                   Yes                   No
                    │                    │
            Need ordering?          Need fast lookup?
            ┌───┴───┐              ┌───┴───┐
           Yes      No            Yes      No
            │        │             │        │
        Use BST   Need O(1)?   Use Set    Use Array
                  ┌───┴───┐   (Hash Set)
                 Yes      No
                  │        │
             Hash Table   BST
```

---

## The Hash Table Trade-Off

```
╔═══════════════════════════════════════════════════════════╗
║                  THE TRADE-OFF                           ║
║                                                          ║
║   WHAT YOU GAIN              WHAT YOU LOSE               ║
║   ──────────────             ──────────────               ║
║   ✓ O(1) avg operations     ✗ No ordering               ║
║   ✓ Flexible key types      ✗ Extra memory overhead     ║
║   ✓ Simple implementation   ✗ Worst case O(n)           ║
║   ✓ Cache-friendly (OA)     ✗ Hash function cost        ║
║   ✓ Dynamic resizing        ✗ Non-deterministic timing  ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: You need to process a stream of 10 million transactions and quickly check if any transaction ID has been seen before. Which data structure would you choose and why?**

<details>
<summary>Click to reveal answer</summary>

A **Hash Set** (hash table storing only keys). Reasons:
1. **O(1) average lookup** — checking if a transaction ID exists is instant
2. **O(1) average insert** — adding new IDs is instant
3. With 10 million items, a BST would need ~23 comparisons per lookup (log₂ 10⁷ ≈ 23), while a hash set needs just 1
4. No ordering is needed — we only care about membership ("seen before?")
5. The trade-off (extra memory, no ordering) is acceptable for this use case

A **Bloom filter** could also work if false positives are acceptable and memory is constrained.

</details>

---

| [← Previous: Direct Addressing](04-direct-addressing.md) | [Next: Hash Table Applications →](06-hash-table-applications.md) |
|:---|---:|

# Chapter 5: When to Apply Hash Map

## 📋 Chapter Overview
Decision guide for when to use hash maps/sets and when alternatives are better.

---

## 🔍 Hash Map Signal Checklist

```
  ✓ Need O(1) lookup by key
  ✓ Counting frequencies / occurrences
  ✓ Finding complement (Two Sum pattern)
  ✓ Grouping elements by a shared property
  ✓ Memoization / caching
  ✓ Deduplication / membership testing (hash set)
  ✓ Mapping one value to another (bijection)
```

---

## 🧭 Decision Flowchart

```
  Need fast lookup by key?
  │
  ├─ YES ──▶ Need associated value?
  │           │
  │           ├─ YES ──▶ Hash Map ✅
  │           │   │
  │           │   ├─ Frequency → count map
  │           │   ├─ Complement → value:index map
  │           │   ├─ Grouping → key:list map
  │           │   └─ Cache → key:result map
  │           │
  │           └─ NO ───▶ Hash Set ✅ (existence only)
  │               │
  │               ├─ Duplicates → add, check
  │               ├─ Membership → convert to set
  │               └─ Cycle detection → store states
  │
  └─ NO ──▶ Need ordering?
              │
              ├─ YES ──▶ TreeMap/BST or sorted array
              └─ NO ───▶ Array (if index-based)
```

---

## ⚠️ When NOT to Use Hash Map

| Situation | Better Alternative | Why |
|-----------|-------------------|-----|
| Need sorted order | TreeMap / sorted array | Hash map is unordered |
| Keys are small integers (0 to n) | Array | Direct indexing is faster, less memory |
| Range queries | Segment tree / BIT | Hash map can't do range operations |
| Memory-critical | Sorting-based approach | Hash map has overhead per entry |
| Need min/max element | Heap | Hash map can't find min/max in O(1) |

---

## 🆚 Hash Map vs Alternatives

| Operation | Hash Map | Sorted Array | BST (TreeMap) | Array |
|-----------|----------|-------------|---------------|-------|
| Lookup | O(1) avg | O(log n) | O(log n) | O(1) by index |
| Insert | O(1) avg | O(n) | O(log n) | O(1) at end |
| Min/Max | O(n) | O(1) | O(log n) | O(n) |
| Range query | O(n) | O(log n) | O(log n + k) | O(range) |
| Ordered iteration | O(n log n) sort | O(n) | O(n) | O(n) |
| Space | High | Low | Medium | Lowest |

---

## 📊 Common Hash Map Patterns

| Pattern | Key → Value | Example Problems |
|---------|------------|-----------------|
| Counting | element → frequency | Majority element, top K frequent |
| Complement | value → index | Two Sum, 4Sum II |
| Grouping | property → list | Group anagrams, bucket sort |
| Prefix sum | cumulative sum → count | Subarray sum = K |
| Sliding window | char → count | Min window substring |
| Caching | inputs → result | Memoization in DP |
| Mapping | original → copy | Clone graph, copy random list |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Use hash map | O(1) lookup needed: counting, complement, grouping, caching |
| Use hash set | Only membership/existence check needed |
| Don't use | Need ordering, range queries, or integer keys (use array) |
| Trade-off | Speed (O(1)) vs memory overhead |
| Common pair | Hash map + prefix sum = O(n) subarray queries |

---

## ❓ Revision Questions

1. **Hash map vs TreeMap?** → HashMap: O(1) lookup, unordered. TreeMap: O(log n) lookup, sorted keys, range queries.
2. **When to use array instead of hash map?** → When keys are small contiguous integers (0 to n) — direct indexing is faster and uses less memory.
3. **Hash map + prefix sum: what problems?** → Subarray sum = K, count subarrays with given XOR, longest subarray with 0 sum.
4. **Hash map for memoization: when?** → When DP state space is sparse (most states never visited); array memo is better for dense states.
5. **Hash map space overhead?** → Each entry has key, value, hash, and pointer overhead. For n elements, typically 4-8x more memory than a plain array.
6. **Worst case O(n) lookup: when?** → When all keys hash to the same bucket (hash collision). Rare with good hash functions; Java 8+ uses red-black tree in buckets to cap at O(log n).

---

[← Previous: Classic Problems](04-classic-problems.md) | [Next: Union-Find Fundamentals →](../12-Union-Find-Pattern/01-union-find-fundamentals.md)

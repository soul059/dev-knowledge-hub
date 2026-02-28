# Chapter 5: When to Apply Heap

## 📋 Chapter Overview
Recognize when a heap (priority queue) is the right choice and avoid common pitfalls.

---

## 🔍 Heap Signal Checklist

```
  ✓ Need repeated access to min or max element
  ✓ "K largest" / "K smallest" / "K most frequent"
  ✓ Merge K sorted sequences
  ✓ Stream of data — running median, top-K
  ✓ Greedy selection of best candidate at each step
  ✓ Priority-based scheduling
```

---

## 🧭 Decision Flowchart

```
  Need min/max repeatedly?
  │
  ├─ YES ──▶ Fixed data or stream?
  │           │
  │           ├─ Stream ──▶ Heap ✅ (dynamic insert/extract)
  │           │
  │           └─ Fixed ──▶ Single min/max? → just scan O(n)
  │                        Kth element? → QuickSelect or Heap
  │                        Sorted order? → Sort
  │
  └─ NO ───▶ Not a heap problem
  
  Merging K sorted things?
  │
  ├─ YES ──▶ Min heap of size K ✅
  │
  └─ NO ───▶ Check other structures
  
  Running median?
  │
  ├─ YES ──▶ Two-heap pattern ✅
  │
  └─ NO ───▶ Other approach
```

---

## 🆚 Heap vs Other Structures

| Need | Heap | BST | Sorting | Hash Map |
|------|------|-----|---------|----------|
| Single min/max | O(1) peek | O(log n) | O(n log n) | O(n) scan |
| Insert + extract | O(log n) | O(log n) | N/A | N/A |
| Kth element | O(n log k) | O(n log n) | O(n log n) | QuickSelect O(n) |
| All sorted | O(n log n) | O(n log n) | O(n log n) | N/A |
| Frequency | Heap on freq | N/A | Bucket sort | Count only |

---

## ⚠️ Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using max heap for K largest | Use MIN heap of size K |
| Forgetting to handle ties | Include secondary sort key |
| Using heap when only one min/max needed | Simple linear scan is O(n) |
| Not considering QuickSelect | For single Kth element, average O(n) |
| Heap for sorted output | Heap sort is O(n log n) — same as regular sort |

---

## 📊 Complexity Reference

| Approach | Time | Space | Best For |
|----------|------|-------|----------|
| Min/Max heap | O(n log k) | O(k) | Top-K, streaming |
| Two heaps | O(n log n) | O(n) | Running median |
| Merge K sorted | O(N log k) | O(k) | K-way merge |
| QuickSelect | O(n) avg | O(1) | Single Kth element |
| Full sort | O(n log n) | O(n) | Need all elements sorted |
| Bucket sort | O(n) | O(n) | K most frequent (bounded) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Use heap | Repeated min/max access, streaming data, K-way merge |
| Don't use heap | Single min/max (scan), full sort (use sort), single Kth (QuickSelect) |
| Top-K trick | Use OPPOSITE heap — min heap for K largest |
| Two-heap | Running median = max heap (left) + min heap (right) |
| Merge K | Min heap of size K, extract global min |

---

## ❓ Revision Questions

1. **When NOT to use a heap?** → Single min/max (linear scan), fully sorted output (just sort), single Kth element (QuickSelect is O(n) average).
2. **Heap vs balanced BST?** → Heap: O(1) peek, O(log n) insert/extract, simpler. BST: O(log n) for search/rank queries, more flexible.
3. **Why min heap for K largest?** → Root is the Kth largest (smallest among top K); elements smaller than root are discarded.
4. **Heap for Dijkstra?** → Yes — min heap extracts the node with smallest distance, enabling greedy shortest-path computation.
5. **When to use bucket sort over heap for top-K?** → When the frequency range is bounded (e.g., array elements, character frequencies) — gives O(n) time.
6. **Two-heap vs sorted array for median?** → Sorted array: O(n) insertion. Two heaps: O(log n) insertion. Heap wins for streaming data.

---

[← Previous: Classic Problems](04-classic-problems.md) | [Next: Hash Map Fundamentals →](../11-Hash-Map-Pattern/01-hashmap-fundamentals.md)

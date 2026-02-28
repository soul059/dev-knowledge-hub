# Chapter 6: When to Apply Binary Search

## 📋 Chapter Overview
How to recognize binary search problems and choose the right template.

---

## ✅ Signal Checklist

| Signal | Template |
|--------|----------|
| Sorted array + find value | T1: Exact match |
| Sorted array + first/last occurrence | T2: Boundary |
| Sorted array + insertion point | T2: Boundary |
| Rotated sorted array | T1 variant |
| "Minimize the maximum" | T3: Answer space |
| "Maximum the minimum" | T3: Answer space |
| Monotonic condition over a range | T3: Answer space |
| O(log n) required by constraint | Binary search likely |

---

## 🗺️ Decision Flowchart

```
  Problem involves searching?
  │
  ├── Data is SORTED (or has monotonic property)?
  │   │
  │   ├── Find exact value? → Template 1
  │   │
  │   ├── Find boundary (first/last/insert)? → Template 2
  │   │
  │   ├── Rotated sorted array?
  │   │   ├── Find value → Rotated search variant
  │   │   └── Find minimum → Compare mid with hi
  │   │
  │   └── "Min of max" / "Max of min"? → Template 3
  │
  ├── Answer space is bounded + monotonic check?
  │   └── Template 3 (Binary Search on Answer)
  │
  └── None of above? → Not binary search
```

---

## ❌ When NOT Binary Search

| Scenario | Better Approach |
|----------|----------------|
| Unsorted + no monotonic property | Linear scan, hash map |
| Need all solutions, not one | Two pointers, backtracking |
| Graph/tree traversal | BFS/DFS |
| Optimization over subarray | Sliding window, DP |
| Small n (≤ 100) | Brute force is fine |

---

## 🔑 Constraint Hints

| Constraint | Suggests |
|-----------|----------|
| n ≤ 10⁵ and O(log n) mentioned | Standard binary search |
| n ≤ 10⁵ and O(n log n) ok | Sort + binary search |
| n ≤ 10⁵ and answer range ≤ 10⁹ | Binary search on answer |
| Two sorted arrays | Merge or binary partition |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Sorted data | First indicator for binary search |
| Monotonic property | Core requirement for correctness |
| 3 templates | Match the problem type to the right one |
| Answer space | Look for "minimize max" or "maximize min" |
| Constraints | O(log n) hints or large answer range |

---

## ❓ Revision Questions

1. **What is the #1 prerequisite for binary search?** → Sorted data or a monotonic property.
2. **"Minimize the maximum subarray sum" → which template?** → Template 3 (Answer Space).
3. **Sorted array, find insertion index → which template?** → Template 2 (Lower Bound).
4. **n = 10⁹ elements, can you binary search?** → Not directly (can't store), but binary search on answer range yes.
5. **How to decide between binary search and two pointers on sorted data?** → Two pointers for pair/sum problems; binary search for single-value lookup or boundary finding.

---

[← Previous: Classic Problems](05-classic-problems.md)

[← Back to README](../README.md) | [Next Unit: BFS/DFS Pattern →](../05-BFS-DFS-Pattern/01-bfs-fundamentals.md)

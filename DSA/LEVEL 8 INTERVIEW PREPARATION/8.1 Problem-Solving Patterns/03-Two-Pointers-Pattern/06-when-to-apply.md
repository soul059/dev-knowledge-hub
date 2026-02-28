# Chapter 6: When to Apply Two Pointers

## 📋 Chapter Overview
Definitive guide for recognizing two-pointer problems and choosing the correct variant.

---

## ✅ Signal Keywords

| Keyword/Constraint | Variant | Example |
|-------------------|---------|---------|
| "sorted array" + "pair/sum" | Opposite direction | Two Sum II |
| "in-place" + "remove/move" | Same direction | Remove Duplicates |
| "linked list" + "cycle/middle" | Fast/slow | Detect Cycle |
| "palindrome" | Opposite direction | Valid Palindrome |
| "container/area" | Opposite (squeeze) | Container With Most Water |
| "merge two sorted" | Same direction | Merge Sorted Arrays |
| "partition into groups" | Three pointers | Sort Colors |
| "k-sum" (3sum, 4sum) | Fix + opposite | Three Sum |

---

## ❌ When NOT Two Pointers

| Scenario | Use Instead |
|----------|-------------|
| Unsorted + need all pairs | Hash Map |
| Contiguous subarray optimization | Sliding Window |
| Non-contiguous subsequence | DP |
| Graph connectivity | BFS/DFS |
| Need all combinations | Backtracking |

---

## 🗺️ Quick Decision

```
  Is array SORTED (or can you sort it)?
    YES → Need pairs? → Opposite Direction
         Need in-place? → Same Direction
    NO  → Is it a linked list? → Fast/Slow
         Need pair lookup? → HashMap instead
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Core requirement | Linear data, elimination logic |
| Sorted data | Opposite direction |
| In-place modification | Same direction |
| Cycle/middle in list | Fast and slow |
| Time complexity | Always O(n) per pointer pair |
| Space complexity | Always O(1) |
| vs Sliding Window | Two Pointers for pairs; Sliding Window for contiguous ranges |

---

## ❓ Quick Revision Questions

1. **Sorted array + find pair → which variant?** → Opposite direction
2. **Remove elements in-place → which variant?** → Same direction
3. **Linked list cycle → which variant?** → Fast/slow
4. **Why is two pointers O(n)?** → Each pointer traverses at most n elements total.
5. **Two Pointers vs Sliding Window?** → Two Pointers for pair-based problems; Sliding Window for contiguous subarray/substring.
6. **Can Two Pointers handle unsorted data?** → Same-direction can; opposite-direction typically needs sorting first.

---

[← Previous: Classic Problems](05-classic-problems.md)

[← Back to README](../README.md) | [Next Unit: Binary Search Pattern →](../04-Binary-Search-Pattern/01-standard-binary-search.md)

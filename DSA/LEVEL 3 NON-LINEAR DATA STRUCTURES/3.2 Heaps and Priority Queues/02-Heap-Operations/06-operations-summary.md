# 2.6 Operations Summary

[← Previous: Delete & Replace](05-delete-and-replace.md) | [Next Unit: Heapify Process →](../03-Heapify-Process/01-heapify-up.md)

---

## All Operations at a Glance

```
┌─────────────────┬──────────────┬─────────────┬──────────────────────┐
│    Operation     │  Time (Avg)  │ Time (Worst)│  Method              │
├─────────────────┼──────────────┼─────────────┼──────────────────────┤
│ Insert           │ O(log n)     │ O(log n)    │ Append + Sift Up     │
│ Extract Min/Max  │ O(log n)     │ O(log n)    │ Replace root + Sift  │
│                  │              │             │ Down                 │
│ Peek             │ O(1)         │ O(1)        │ Return root          │
│ Build Heap       │ O(n)         │ O(n)        │ Bottom-up sift down  │
│ Delete Arbitrary │ O(n)         │ O(n)        │ Find + Replace + Fix │
│ Replace Top      │ O(log n)     │ O(log n)    │ Replace root + Sift  │
│                  │              │             │ Down                 │
│ Search           │ O(n)         │ O(n)        │ Linear scan          │
│ Size             │ O(1)         │ O(1)        │ Track array length   │
│ Is Empty         │ O(1)         │ O(1)        │ Check size           │
└─────────────────┴──────────────┴─────────────┴──────────────────────┘
```

---

## Space Complexity

| Aspect | Space |
|--------|-------|
| Heap storage | O(n) |
| Insert | O(1) auxiliary |
| Extract | O(1) auxiliary |
| Build Heap (in-place) | O(1) auxiliary |

---

## Visualizing the Core Pattern

```
INSERT:                             EXTRACT:

  Array: [... ... X]               Root removed
  Add X at end                     Last element → Root
       ↑                                ↓
     Bubble UP                       Bubble DOWN
       ↑                                ↓
  Fix upward path               Fix downward path
```

Both operations follow the same principle:
1. Make a structural change (add/remove)
2. Fix the heap property along **one path** (root-to-leaf or leaf-to-root)
3. Each path has at most **log n** nodes → O(log n)

---

## Common Mistakes

| Mistake | Why It's Wrong |
|---------|---------------|
| Sifting down on insert | New element is at the bottom — it might need to go UP |
| Sifting up on extract | Replacement element is at top — it might need to go DOWN |
| Swapping with any child | Must swap with the **larger** child (max-heap) or **smaller** child (min-heap) |
| Forgetting to check bounds | Left/right child indices may exceed array size |
| Off-by-one in parent formula | `(i-1)/2` for 0-indexed, `i/2` for 1-indexed |

---

## Unit 2 Summary Table

| Concept | Key Point |
|---------|-----------|
| Insert | Append at end, bubble up — O(log n) |
| Extract Min/Max | Replace root with last, sift down — O(log n) |
| Peek | Return root — O(1) |
| Build Heap | Bottom-up sift down (Floyd's) — O(n) |
| Delete Arbitrary | Replace with last, sift up or down — O(n) |
| Replace Top | Direct root replacement + sift down — O(log n) |
| Core Principle | Fix one root-to-leaf path after each change |
| Sift Up | Used after INSERT (child may be too large/small) |
| Sift Down | Used after EXTRACT (parent may be too small/large) |

---

[← Previous: Delete & Replace](05-delete-and-replace.md) | [Next Unit: Heapify Process →](../03-Heapify-Process/01-heapify-up.md)

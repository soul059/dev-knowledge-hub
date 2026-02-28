# 4.3 In-Place Sorting

[← Previous: Step-by-Step Trace](02-step-by-step-trace.md) | [Next: Time Complexity →](04-time-complexity.md)

---

## Overview

One of Heap Sort's greatest strengths — it sorts the array using only **O(1) auxiliary space**.

---

## How It Works

```
Original array is reused for both the heap AND the sorted output:

Before:  [───── unsorted array ──────]
After build: [──── max-heap ──────────]
During sort: [── heap ──|── sorted ──]
                shrinks     grows
Final:   [──── sorted ascending ─────]
```

No auxiliary array is needed. We only use a constant number of extra variables (for swapping and indexing).

---

## Space Complexity

| Aspect | Space |
|--------|-------|
| Input array | O(n) — shared between heap and sorted regions |
| Auxiliary variables | O(1) — just loop indices and swap temps |
| **Total auxiliary** | **O(1)** |

Compare with other O(n log n) sorts:

| Algorithm | Auxiliary Space |
|-----------|----------------|
| **Heap Sort** | **O(1)** |
| Merge Sort | O(n) |
| Quick Sort | O(log n) — call stack |
| Tim Sort | O(n) |

---

## Quick Check

1. **Why doesn't Heap Sort need an auxiliary array like Merge Sort?**
   <details><summary>Answer</summary>
   The swap-to-end trick lets Heap Sort reuse the same array: the heap shrinks from the left while sorted elements accumulate on the right. Merge Sort needs an extra buffer for the merge step.
   </details>

---

[← Previous: Step-by-Step Trace](02-step-by-step-trace.md) | [Next: Time Complexity →](04-time-complexity.md)

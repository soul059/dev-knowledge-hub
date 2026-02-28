# 7.4 Maximize Capital (IPO Problem)

[← Previous: Sliding Window Median](03-sliding-window-median.md) | [Next: Balancing Strategy →](05-balancing-strategy.md)

---

## Problem

You have initial capital `W`. You can do at most `k` projects. Each project `i` has:
- `capital[i]`: minimum capital needed to start
- `profits[i]`: profit earned after completing

Choose projects to **maximize total capital**.

---

## Key Insight: Two Heaps

```
MIN-HEAP (by capital):          MAX-HEAP (by profit):
All projects sorted by          Projects we CAN afford
minimum capital required.       (capital ≤ current W).
"Waiting room"                  "Ready to pick"

  As W increases, move          Always pick the most
  affordable projects →         profitable available
  from min-heap to max-heap     project.
```

---

## Algorithm

```
MAXIMIZE_CAPITAL(k, W, profits, capital):
    // Min-heap of (capital_needed, profit) — waiting room
    min_heap = [(capital[i], profits[i]) for all i]
    heapify(min_heap)
    
    // Max-heap of profits — ready projects
    max_heap = []
    
    FOR round = 1 TO k:
        // Move all affordable projects to max_heap
        WHILE min_heap not empty AND min_heap.peek().capital ≤ W:
            (cap, profit) = min_heap.extract()
            max_heap.insert(profit)
        
        // Pick most profitable available project
        IF max_heap is empty:
            BREAK   // No affordable projects
        
        W += max_heap.extract()   // Earn profit
    
    RETURN W
```

---

## Trace

```
k = 2, W = 0
Projects: capital = [0, 1, 1, 2], profits = [1, 2, 3, 5]

Min-heap (by capital): [(0,1), (1,2), (1,3), (2,5)]

── Round 1: W = 0 ──
Move affordable (capital ≤ 0): (0,1)
Max-heap (by profit): [1]
Min-heap: [(1,2), (1,3), (2,5)]

Pick best profit: 1
W = 0 + 1 = 1

── Round 2: W = 1 ──
Move affordable (capital ≤ 1): (1,2), (1,3)
Max-heap: [3, 2]
Min-heap: [(2,5)]

Pick best profit: 3
W = 1 + 3 = 4

Result: W = 4 ✅
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n log n + k log n) |
| Space | O(n) |

---

## Quick Check

1. **In the IPO problem, why do we need TWO heaps instead of just sorting by profit?**
   <details><summary>Answer</summary>
   Not all projects are immediately affordable. The min-heap (by capital) tracks which projects become affordable as our capital grows. The max-heap (by profit) lets us pick the most profitable among affordable projects. Simply sorting by profit ignores capital requirements.
   </details>

---

[← Previous: Sliding Window Median](03-sliding-window-median.md) | [Next: Balancing Strategy →](05-balancing-strategy.md)

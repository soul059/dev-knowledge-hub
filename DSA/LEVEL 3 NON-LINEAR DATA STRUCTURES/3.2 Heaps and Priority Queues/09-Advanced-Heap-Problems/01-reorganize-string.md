# 9.1 Reorganize String

[← Previous: Unit 8 — Variations & Applications](../08-Merge-K-Sorted/06-variations-and-applications.md) | [Next: Task Scheduler →](02-task-scheduler.md)

---

## Overview

This unit tackles advanced problems where heaps provide elegant solutions that might not be immediately obvious. Each problem teaches a different **way of thinking** about heaps — as schedulers, frequency managers, interval processors, and sequence generators.

---

## Problem

Given a string `s`, rearrange it so that **no two adjacent characters are the same**. Return any valid rearrangement, or empty string if impossible.

**Example**: `s = "aab"` → `"aba"` | `s = "aaab"` → `""` (impossible)

---

## Key Insight

> Place the **most frequent** character first, then the next most frequent, alternating. A **max-heap by frequency** always gives us the best candidate to place next.

### When Is It Impossible?

```
If any character has frequency > (n + 1) / 2, it's impossible.
Example: "aaab" → 'a' has freq 3, but (4+1)/2 = 2 → 3 > 2 → impossible
```

---

## Algorithm

```
REORGANIZE_STRING(s):
    freq = count_frequency(s)
    max_heap = []
    
    FOR (char, count) in freq:
        max_heap.insert((count, char))
    
    result = []
    prev = null   // Previously placed (char, count) — "cooling off"
    
    WHILE max_heap is not empty:
        (count, char) = max_heap.extract_max()
        result.append(char)
        
        // Put previous character back (it's cooled off)
        IF prev is not null AND prev.count > 0:
            max_heap.insert(prev)
        
        // Current character needs to cool off
        prev = (count - 1, char)
    
    IF len(result) != len(s):
        RETURN ""   // Impossible
    RETURN "".join(result)
```

---

## Trace

```
s = "aabbc"
freq: a=2, b=2, c=1

Max-heap: [(2,'a'), (2,'b'), (1,'c')]

Step 1: Extract (2,'a'), place 'a', prev=(1,'a')
  result: "a"

Step 2: Extract (2,'b'), place 'b', put prev (1,'a') back in heap
  heap: [(1,'a'), (1,'c')], prev=(1,'b')
  result: "ab"

Step 3: Extract (1,'a'), place 'a', put prev (1,'b') back
  heap: [(1,'b'), (1,'c')], prev=(0,'a')
  result: "aba"

Step 4: Extract (1,'b'), place 'b', prev (0,'a') → don't re-insert (count=0)
  heap: [(1,'c')], prev=(0,'b')
  result: "abab"

Step 5: Extract (1,'c'), place 'c', prev (0,'b') → don't re-insert
  heap: [], prev=(0,'c')
  result: "ababc" ✅
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n log A) where A = alphabet size (26 for lowercase) |
| Space | O(A) |

---

[← Previous: Unit 8 — Variations & Applications](../08-Merge-K-Sorted/06-variations-and-applications.md) | [Next: Task Scheduler →](02-task-scheduler.md)

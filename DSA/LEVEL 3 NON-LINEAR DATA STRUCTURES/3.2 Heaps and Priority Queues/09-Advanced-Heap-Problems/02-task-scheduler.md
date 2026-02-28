# 9.2 Task Scheduler

[← Previous: Reorganize String](01-reorganize-string.md) | [Next: Meeting Rooms →](03-meeting-rooms.md)

---

## Problem

Given tasks `['A','A','A','B','B','B']` and cooldown `n=2`, find the minimum number of intervals to execute all tasks. Same tasks must have at least `n` intervals between them.

**Example**: tasks = `[A,A,A,B,B,B]`, n=2 → `A B _ A B _ A B` → **8 intervals**

---

## Key Insight

> Schedule the **most frequent** task first. Fill gaps with other tasks. If no task available, use **idle** time.

---

## Algorithm

```
TASK_SCHEDULER(tasks, n):
    freq = count_frequency(tasks)
    max_heap = [count for count in freq.values()]
    heapify_max(max_heap)
    
    time = 0
    
    WHILE max_heap is not empty:
        // Process one cycle of length (n + 1)
        cycle = n + 1
        temp = []   // Tasks processed in this cycle
        
        FOR i = 0 TO cycle - 1:
            IF max_heap is not empty:
                count = max_heap.extract_max()
                IF count > 1:
                    temp.append(count - 1)
                time += 1
            ELSE IF temp is not empty:
                time += 1   // Idle
        
        // Re-insert remaining tasks
        FOR count in temp:
            max_heap.insert(count)
    
    RETURN time
```

---

## Visual Trace

```
tasks = [A,A,A,B,B,B,C,C], n = 2

freq: A=3, B=3, C=2
cycle length = n + 1 = 3

═══ Cycle 1 ═══
  Slot 1: Take A (3→2), Slot 2: Take B (3→2), Slot 3: Take C (2→1)
  Schedule: [A, B, C]
  Remaining: A=2, B=2, C=1
  time = 3

═══ Cycle 2 ═══
  Slot 1: Take A (2→1), Slot 2: Take B (2→1), Slot 3: Take C (1→0)
  Schedule: [A, B, C, A, B, C]  
  Remaining: A=1, B=1
  time = 6

═══ Cycle 3 ═══
  Slot 1: Take A (1→0), Slot 2: Take B (1→0), Slot 3: empty
  Schedule: [A, B, C, A, B, C, A, B]
  Remaining: nothing
  time = 8 ✅
```

---

## Formula Approach (Without Heap)

```
max_freq = highest frequency among all tasks
max_count = number of tasks with max_freq

result = max(len(tasks), (max_freq - 1) * (n + 1) + max_count)
```

**Explanation**:
```
A=3, B=3, C=2, n=2

A _ _ A _ _ A      ← (max_freq-1) gaps of size (n+1), plus 1
A B _ A B _ A B    ← Fill B (also max_freq)
A B C A B C A B    ← Fill C

Result = max(8, (3-1)*(2+1) + 2) = max(8, 8) = 8
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n × total_tasks) with heap; O(n) with formula |
| Space | O(26) = O(1) for alphabet |

---

## Quick Check

1. **What determines whether idle slots are needed?**
   <details><summary>Answer</summary>
   Idle slots appear when the number of distinct tasks available is less than the cooldown period (n+1). If there are enough different tasks to fill each cycle, no idle time is needed. The formula captures this: result = max(total_tasks, (max_freq-1)*(n+1) + max_count).
   </details>

---

[← Previous: Reorganize String](01-reorganize-string.md) | [Next: Meeting Rooms →](03-meeting-rooms.md)

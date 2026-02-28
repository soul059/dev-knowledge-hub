# 9.3 Meeting Rooms II

[← Previous: Task Scheduler](02-task-scheduler.md) | [Next: Employee Free Time →](04-employee-free-time.md)

---

## Problem

Given meeting intervals `[[s1,e1], [s2,e2], ...]`, find the **minimum number of conference rooms** required.

**Example**: `[[0,30],[5,10],[15,20]]` → **2 rooms** (meetings 1&2 overlap)

---

## Key Insight

> Track **end times** of ongoing meetings in a min-heap. When a new meeting starts, if the earliest ending meeting has already ended, reuse that room.

---

## Algorithm

```
MIN_MEETING_ROOMS(intervals):
    SORT intervals by start time
    
    min_heap = []   // Stores end times of ongoing meetings
    
    FOR each interval [start, end]:
        // If earliest meeting ended before this one starts, reuse room
        IF min_heap is not empty AND min_heap.peek() ≤ start:
            min_heap.extract_min()   // Room freed
        
        min_heap.insert(end)         // Occupy a room
    
    RETURN min_heap.size   // Number of rooms in use
```

---

## Trace

```
intervals = [[0,30], [5,10], [15,20]]
Sorted by start: [[0,30], [5,10], [15,20]]

Process [0, 30]:
  heap empty → insert 30
  heap: [30], rooms = 1

Process [5, 10]:
  peek = 30, 30 > 5 → can't reuse (meeting still running)
  insert 10
  heap: [10, 30], rooms = 2

Process [15, 20]:
  peek = 10, 10 ≤ 15 → meeting ended! Extract 10 (room freed)
  insert 20
  heap: [20, 30], rooms = 2

Answer: 2 rooms ✅
```

---

## Visual Timeline

```
Room 1: |=====meeting 1 (0-30)==============================|
Room 2: .....|-m2(5-10)-|.....|--m3(15-20)--|..............
         0    5    10   15    20            30
         
Meeting 2 ends at 10. Meeting 3 starts at 15. 
Room 2 is reused! Only 2 rooms needed.
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n log n) — sorting + heap ops |
| Space | O(n) — worst case all overlap |

---

## Quick Check

1. **Why does Meeting Rooms II use a min-heap of END times?**
   <details><summary>Answer</summary>
   We want to know which meeting ends earliest. If the earliest-ending meeting finishes before the new meeting starts, we can reuse that room. A min-heap of end times gives us the earliest end time in O(1) and allows room reuse decisions in O(log n).
   </details>

---

[← Previous: Task Scheduler](02-task-scheduler.md) | [Next: Employee Free Time →](04-employee-free-time.md)

# 9.4 Employee Free Time

[← Previous: Meeting Rooms](03-meeting-rooms.md) | [Next: Ugly Numbers →](05-ugly-numbers.md)

---

## Problem

Given schedules of K employees (each a list of non-overlapping intervals), find all **free time** common to all employees.

```
Employee 1: [[1,3], [6,7]]
Employee 2: [[2,4]]
Employee 3: [[2,5], [9,12]]

Working:   |--1--|     |1|
              |--2--|
              |---3---|    |---3---|
Timeline:  1  2  3  4  5  6  7  8  9  10 11 12

Free time: [[5,6], [7,9]]
```

---

## Approach: Merge K Sorted + Gap Finding

```
EMPLOYEE_FREE_TIME(schedules):
    // Step 1: Merge all intervals using K-way merge (by start time)
    min_heap = []
    FOR i = 0 TO K-1:
        IF schedules[i] not empty:
            min_heap.insert((schedules[i][0].start, i, 0))
    
    merged = []
    WHILE min_heap not empty:
        (start, emp_idx, interval_idx) = min_heap.extract_min()
        interval = schedules[emp_idx][interval_idx]
        
        // Merge overlapping intervals
        IF merged is empty OR interval.start > merged.last.end:
            merged.append(interval)
        ELSE:
            merged.last.end = max(merged.last.end, interval.end)
        
        // Advance to next interval of same employee
        IF interval_idx + 1 < schedules[emp_idx].length:
            next = schedules[emp_idx][interval_idx + 1]
            min_heap.insert((next.start, emp_idx, interval_idx + 1))
    
    // Step 2: Gaps between merged intervals = free time
    result = []
    FOR i = 1 TO merged.length - 1:
        result.append([merged[i-1].end, merged[i].start])
    
    RETURN result
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(N log K) where N = total intervals |
| Space | O(K + N) |

---

## Quick Check

1. **Why is this problem essentially a "Merge K Sorted" problem?**
   <details><summary>Answer</summary>
   Each employee's schedule is already sorted by time. We need to merge all K sorted schedules into one timeline. The K-way merge with a heap handles this in O(N log K). After merging, finding gaps between merged intervals is a simple linear scan.
   </details>

---

[← Previous: Meeting Rooms](03-meeting-rooms.md) | [Next: Ugly Numbers →](05-ugly-numbers.md)

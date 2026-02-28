# Chapter 2: Interval Problems

## 📋 Chapter Overview
Interval-based greedy problems: merging, scheduling, minimum platforms, and non-overlapping selections.

---

## 📝 Problem 1: Merge Intervals

**Problem:** Given a list of intervals, merge all overlapping intervals.

```
  Input:  [[1,3],[2,6],[8,10],[15,18]]
  Output: [[1,6],[8,10],[15,18]]
  
  Timeline:
  1---3
    2------6
              8--10
                       15--18
  Merged:
  1--------6  8--10    15--18
```

### Pseudocode

```
function mergeIntervals(intervals):
    sort intervals by start time
    merged = [intervals[0]]
    
    for i = 1 to n-1:
        if intervals[i].start <= merged.last.end:
            // Overlapping — extend end
            merged.last.end = max(merged.last.end, intervals[i].end)
        else:
            merged.add(intervals[i])
    
    return merged
```

### Trace

```
  Sorted: [1,3] [2,6] [8,10] [15,18]
  
  merged = [[1,3]]
  
  [2,6]: 2 ≤ 3 → overlap → end = max(3,6) = 6 → merged = [[1,6]]
  [8,10]: 8 > 6 → no overlap → merged = [[1,6],[8,10]]
  [15,18]: 15 > 10 → no overlap → merged = [[1,6],[8,10],[15,18]]
```

**Complexity:** O(n log n) time, O(n) space

---

## 📝 Problem 2: Non-Overlapping Intervals (Minimum Removals)

**Problem:** Find minimum number of intervals to remove so remaining are non-overlapping. (LeetCode 435)

```
  Equivalent to: find MAX non-overlapping intervals, then remove rest.
  
  This IS the activity selection problem!
```

### Pseudocode

```
function eraseOverlapIntervals(intervals):
    sort intervals by END time
    count = 0
    lastEnd = -INF
    
    for each interval:
        if interval.start >= lastEnd:
            // Keep this interval
            lastEnd = interval.end
        else:
            // Remove this interval (overlaps)
            count += 1
    
    return count
```

### Trace

```
  Input: [[1,2],[2,3],[3,4],[1,3]]
  Sorted by end: [1,2] [2,3] [1,3] [3,4]
  
  lastEnd = -INF
  [1,2]: 1 ≥ -INF → keep. lastEnd=2
  [2,3]: 2 ≥  2   → keep. lastEnd=3
  [1,3]: 1 <  3   → REMOVE. count=1
  [3,4]: 3 ≥  3   → keep. lastEnd=4
  
  Removed: 1
```

**Key insight:** Sort by END time (not start). Greedy: pick the interval that finishes earliest.

---

## 📝 Problem 3: Minimum Meeting Rooms (LeetCode 253)

**Problem:** Given meeting intervals, find minimum rooms needed so no two meetings in the same room overlap.

```
  Input: [[0,30],[5,10],[15,20]]
  
  Timeline:
  0==============================30
       5====10
                 15====20
  
  At time 5: meetings [0,30] and [5,10] overlap → need 2 rooms
  Answer: 2
```

### Pseudocode (Sweep Line)

```
function minMeetingRooms(intervals):
    events = []
    for each interval [s, e]:
        events.add( (s, +1) )   // meeting starts
        events.add( (e, -1) )   // meeting ends
    
    sort events by time (if tie, end before start)
    
    maxRooms = 0, current = 0
    for each (time, type) in events:
        current += type
        maxRooms = max(maxRooms, current)
    
    return maxRooms
```

### Trace

```
  Events: (0,+1) (5,+1) (10,-1) (15,+1) (20,-1) (30,-1)
  Sorted: same order
  
  time=0:  current = 0+1 = 1, max=1
  time=5:  current = 1+1 = 2, max=2
  time=10: current = 2-1 = 1, max=2
  time=15: current = 1+1 = 2, max=2
  time=20: current = 2-1 = 1, max=2
  time=30: current = 1-1 = 0, max=2
  
  Answer: 2
```

**Complexity:** O(n log n)

---

## 📝 Problem 4: Insert Interval (LeetCode 57)

**Problem:** Insert a new interval into sorted non-overlapping intervals and merge if needed.

```
function insert(intervals, newInterval):
    result = []
    i = 0, n = len(intervals)
    
    // Add all intervals ending before newInterval starts
    while i < n and intervals[i].end < newInterval.start:
        result.add(intervals[i])
        i++
    
    // Merge overlapping intervals
    while i < n and intervals[i].start <= newInterval.end:
        newInterval.start = min(newInterval.start, intervals[i].start)
        newInterval.end = max(newInterval.end, intervals[i].end)
        i++
    result.add(newInterval)
    
    // Add remaining intervals
    while i < n:
        result.add(intervals[i])
        i++
    
    return result
```

**Complexity:** O(n) time (already sorted)

---

## 📝 Problem 5: Minimum Arrows to Burst Balloons (LeetCode 452)

**Problem:** Balloons as intervals on x-axis. Arrow shot at x bursts all balloons covering x. Find minimum arrows.

```
  Same as: find minimum points that cover all intervals
  Equivalent to: max non-overlapping groups (one arrow per group)
  
function findMinArrowShots(points):
    sort points by END coordinate
    arrows = 1
    arrowPos = points[0].end
    
    for i = 1 to n-1:
        if points[i].start > arrowPos:
            arrows++
            arrowPos = points[i].end
    
    return arrows
```

### Trace

```
  Input: [[10,16],[2,8],[1,6],[7,12]]
  Sorted by end: [1,6] [2,8] [7,12] [10,16]
  
  arrows = 1, arrowPos = 6
  [2,8]:  start 2 ≤ 6 → same arrow
  [7,12]: start 7 > 6 → new arrow. arrows=2, arrowPos=12
  [10,16]: start 10 ≤ 12 → same arrow
  
  Answer: 2
```

---

## 📊 Interval Problem Summary

| Problem | Sort By | Greedy Rule | Complexity |
|---------|---------|-------------|-----------|
| Merge intervals | Start | Extend end if overlapping | O(n log n) |
| Non-overlapping (max) | End | Pick earliest-finishing | O(n log n) |
| Min removals | End | Count overlapping = n - max kept | O(n log n) |
| Meeting rooms | Events | Sweep line with +1/-1 | O(n log n) |
| Insert interval | Already sorted | Three-phase merge | O(n) |
| Min arrows / points | End | New arrow when no overlap | O(n log n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Sort by end | For selection/coverage problems |
| Sort by start | For merging problems |
| Sweep line | For counting concurrent events |
| Exchange argument | Earliest-finish greedy is provably optimal |

---

## ❓ Revision Questions

1. **Merge intervals: which sort?** → Sort by start time.
2. **Non-overlapping intervals: which sort?** → Sort by end time (activity selection).
3. **Meeting rooms: approach?** → Sweep line: events at start (+1) and end (-1), track max concurrent.
4. **Min arrows: why sort by end?** → Shooting at the end of the earliest-finishing balloon covers the most overlapping balloons.
5. **Insert interval: time complexity?** → O(n) since input is already sorted; no re-sort needed.

---

[← Previous: Greedy Fundamentals](01-greedy-fundamentals.md) | [Next: Array Greedy →](03-array-greedy.md)

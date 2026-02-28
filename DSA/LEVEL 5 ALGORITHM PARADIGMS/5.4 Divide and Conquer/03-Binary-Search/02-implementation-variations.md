# Chapter 2: Implementation Variations

## Overview

Binary Search has many variations depending on the problem requirements. This chapter covers the most important variants: finding the first/last occurrence, insertion point, and search in different data structures.

---

## Variation 1: Find First Occurrence (Lower Bound)

When duplicates exist, standard Binary Search may return any occurrence. This variant finds the **leftmost (first)** occurrence.

```
Array: [1, 2, 2, 2, 3, 4, 5]
Target: 2

Standard BS might return index 1, 2, or 3 — any of them!
We want: index 1 (the FIRST 2)

Strategy: When arr[mid] == target, DON'T stop!
          Continue searching LEFT for an earlier occurrence.
```

### Pseudocode

```
function findFirst(arr, target):
    low = 0, high = len(arr) - 1
    result = -1                        // track best answer
    
    while low <= high:
        mid = low + (high - low) / 2
        
        if arr[mid] == target:
            result = mid               // record this occurrence
            high = mid - 1             // keep searching LEFT
        else if arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    
    return result
```

### Trace

```
Array: [1, 2, 2, 2, 3, 4, 5],  Target: 2

Step 1: low=0, high=6, mid=3, arr[3]=2 → result=3, high=2
Step 2: low=0, high=2, mid=1, arr[1]=2 → result=1, high=0
Step 3: low=0, high=0, mid=0, arr[0]=1 < 2 → low=1
Step 4: low=1, high=0, STOP.

Return result = 1 ✓ (first occurrence of 2)
```

---

## Variation 2: Find Last Occurrence (Upper Bound)

Find the **rightmost (last)** occurrence of the target.

```
function findLast(arr, target):
    low = 0, high = len(arr) - 1
    result = -1
    
    while low <= high:
        mid = low + (high - low) / 2
        
        if arr[mid] == target:
            result = mid               // record this occurrence
            low = mid + 1              // keep searching RIGHT
        else if arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    
    return result
```

### Trace

```
Array: [1, 2, 2, 2, 3, 4, 5],  Target: 2

Step 1: low=0, high=6, mid=3, arr[3]=2 → result=3, low=4
Step 2: low=4, high=6, mid=5, arr[5]=4 > 2 → high=4
Step 3: low=4, high=4, mid=4, arr[4]=3 > 2 → high=3
Step 4: low=4, high=3, STOP.

Return result = 3 ✓ (last occurrence of 2)
```

---

## Variation 3: Count Occurrences

Combine first and last occurrence to count:

```
function countOccurrences(arr, target):
    first = findFirst(arr, target)
    if first == -1:
        return 0
    last = findLast(arr, target)
    return last - first + 1

Example: [1, 2, 2, 2, 3, 4, 5], target=2
  first = 1, last = 3
  count = 3 - 1 + 1 = 3 ✓
```

---

## Variation 4: Insertion Point (bisect_left)

Find the position where target **should be inserted** to maintain sorted order.

```
function insertionPoint(arr, target):
    low = 0, high = len(arr)    // NOTE: high is len(arr), not len-1!
    
    while low < high:           // NOTE: <, not <=
        mid = low + (high - low) / 2
        
        if arr[mid] < target:
            low = mid + 1
        else:
            high = mid          // NOTE: mid, not mid-1
    
    return low                  // insertion point
```

### Trace

```
Array: [1, 3, 5, 7, 9],  Target: 4

Step 1: low=0, high=5, mid=2, arr[2]=5 ≥ 4 → high=2
Step 2: low=0, high=2, mid=1, arr[1]=3 < 4 → low=2
Step 3: low=2, high=2, STOP.

Return 2 → Insert 4 at index 2: [1, 3, 4, 5, 7, 9] ✓
```

---

## Variation 5: Find Ceiling and Floor

### Floor: Largest element ≤ target

```
function floor(arr, target):
    low = 0, high = len(arr) - 1
    result = -1
    
    while low <= high:
        mid = low + (high - low) / 2
        if arr[mid] <= target:
            result = mid           // potential floor
            low = mid + 1
        else:
            high = mid - 1
    
    return result

Example: [1, 3, 5, 7, 9], target=6
  Floor = index 2 (value 5, since 5 ≤ 6) ✓
```

### Ceiling: Smallest element ≥ target

```
function ceiling(arr, target):
    low = 0, high = len(arr) - 1
    result = -1
    
    while low <= high:
        mid = low + (high - low) / 2
        if arr[mid] >= target:
            result = mid           // potential ceiling
            high = mid - 1
        else:
            low = mid + 1
    
    return result

Example: [1, 3, 5, 7, 9], target=6
  Ceiling = index 3 (value 7, since 7 ≥ 6) ✓
```

---

## Variation 6: Find Peak Element

Find any element that is greater than its neighbors (array not necessarily sorted).

```
function findPeak(arr, low, high):
    mid = low + (high - low) / 2
    
    // Check if mid is a peak
    if (mid == 0 or arr[mid-1] <= arr[mid]) and
       (mid == n-1 or arr[mid+1] <= arr[mid]):
        return mid
    
    // If left neighbor is greater, peak is on the left
    if mid > 0 and arr[mid-1] > arr[mid]:
        return findPeak(arr, low, mid - 1)
    
    // Otherwise, peak is on the right
    return findPeak(arr, mid + 1, high)
```

### Trace

```
Array: [1, 3, 20, 4, 1, 0]

Step 1: mid=2, arr[2]=20
        Left neighbors: arr[1]=3 ≤ 20 ✓
        Right neighbor: arr[3]=4 ≤ 20 ✓
        → 20 is a PEAK! Return index 2 ✓
```

---

## Comparison of Variations

```
Standard:        Find ANY occurrence of target
                 ┌─┬─┬─┬─┬─┬─┬─┐
                 │1│2│2│2│3│4│5│  → Returns index 1, 2, or 3
                 └─┴─┴─┴─┴─┴─┴─┘
                         ▲

First Occurrence: Find LEFTMOST target
                 ┌─┬─┬─┬─┬─┬─┬─┐
                 │1│2│2│2│3│4│5│  → Returns index 1
                 └─┴─┴─┴─┴─┴─┴─┘
                   ▲

Last Occurrence:  Find RIGHTMOST target
                 ┌─┬─┬─┬─┬─┬─┬─┐
                 │1│2│2│2│3│4│5│  → Returns index 3
                 └─┴─┴─┴─┴─┴─┴─┘
                       ▲

Insertion Point:  Where to INSERT target
                 ┌─┬─┬─┬─┬─┬─┬─┐
                 │1│3│5│7│9│ │ │  target=4 → Returns index 2
                 └─┴─┴─┴─┴─┴─┴─┘
                     ▲ insert here

Floor:           Largest element ≤ target
Ceiling:         Smallest element ≥ target
Peak:            Element > both neighbors
```

---

## Template Pattern: The "Left/Right Bias" Framework

Most Binary Search variants follow this meta-pattern:

```
function binarySearchVariant(arr, target):
    low = 0, high = len(arr) - 1  (or len(arr))
    result = <default>
    
    while low <= high:  (or low < high)
        mid = low + (high - low) / 2
        
        if <condition>(arr[mid], target):
            result = mid
            <update low or high to CONTINUE searching>
        else if arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1  (or high = mid)
    
    return result

The differences:
- Find first: continue searching LEFT after finding target
- Find last:  continue searching RIGHT after finding target
- Insertion:  use low < high loop with high = mid
```

---

## Summary Table

| Variation | Returns | Key Difference | Time |
|-----------|---------|----------------|------|
| Standard | Any index of target | Stop on first match | O(log n) |
| First Occurrence | Leftmost index | Continue left after match | O(log n) |
| Last Occurrence | Rightmost index | Continue right after match | O(log n) |
| Insertion Point | Index to insert | Use `low < high` loop | O(log n) |
| Floor | Largest ≤ target | Track potential answer | O(log n) |
| Ceiling | Smallest ≥ target | Track potential answer | O(log n) |
| Peak Element | Local maximum | Compare with neighbors | O(log n) |

---

## Quick Revision Questions

1. **How do you modify standard Binary Search to find the FIRST occurrence of a target?**
2. **What changes in the loop condition and update rules for the insertion point variant?**
3. **How can you count the number of occurrences of a target in O(log n)?**
4. **What is the difference between floor and ceiling in the context of Binary Search?**
5. **Why does the Peak Element search work even when the array isn't sorted?**
6. **Write the template pattern that all Binary Search variants follow.**

---

[← Previous: Binary Search Concept](01-binary-search-concept.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)

# Chapter 6: First and Last Occurrence

## Overview
Finding the first and last occurrence of a target in a sorted array extends binary search with a subtle twist — instead of stopping when found, we continue searching in one direction to find the boundary.

---

## Find First Occurrence

### Pseudocode
```
function findFirst(arr, target, low, high, result=-1):
    if low > high: return result
    mid = low + (high - low) / 2
    if arr[mid] == target:
        result = mid                          // Record this position
        return findFirst(arr, target, low, mid - 1, result)  // Keep searching LEFT
    else if arr[mid] < target:
        return findFirst(arr, target, mid + 1, high, result)
    else:
        return findFirst(arr, target, low, mid - 1, result)
```

### Trace: findFirst([1, 3, 3, 3, 5, 7], 3, 0, 5)

```
Step 1: low=0, high=5, mid=2, arr[2]=3 == target
        result=2, search LEFT (low=0, high=1)
        [1, 3, |3|, 3, 5, 7]
               ↑found, go left

Step 2: low=0, high=1, mid=0, arr[0]=1 < target
        search RIGHT (low=1, high=1)

Step 3: low=1, high=1, mid=1, arr[1]=3 == target
        result=1, search LEFT (low=1, high=0)
        [1, |3|, 3, 3, 5, 7]
            ↑found, go left

Step 4: low=1, high=0 → low > high → return result=1 ✓

First occurrence of 3 is at index 1
```

---

## Find Last Occurrence

### Pseudocode
```
function findLast(arr, target, low, high, result=-1):
    if low > high: return result
    mid = low + (high - low) / 2
    if arr[mid] == target:
        result = mid                           // Record this position
        return findLast(arr, target, mid + 1, high, result)  // Keep searching RIGHT
    else if arr[mid] < target:
        return findLast(arr, target, mid + 1, high, result)
    else:
        return findLast(arr, target, low, mid - 1, result)
```

### Trace: findLast([1, 3, 3, 3, 5, 7], 3, 0, 5)

```
Step 1: low=0, high=5, mid=2, arr[2]=3 == target
        result=2, search RIGHT (low=3, high=5)

Step 2: low=3, high=5, mid=4, arr[4]=5 > target
        search LEFT (low=3, high=3)

Step 3: low=3, high=3, mid=3, arr[3]=3 == target
        result=3, search RIGHT (low=4, high=3)

Step 4: low=4, high=3 → low > high → return result=3 ✓

Last occurrence of 3 is at index 3
```

---

## Count Occurrences

```
Using first and last occurrence:
  count = lastOccurrence - firstOccurrence + 1

For target=3 in [1, 3, 3, 3, 5, 7]:
  first = 1, last = 3
  count = 3 - 1 + 1 = 3 occurrences ✓

                 first     last
                   ↓         ↓
  [1,  3,  3,  3,  5,  7]
       ↑───────↑
       3 occurrences
```

---

## Complexity

```
Time:  O(log n) — binary search for each
Space: O(log n) — recursive stack depth
Both together: O(log n) — two binary searches = 2 × O(log n) = O(log n)
```

---

## Summary Table

| Aspect | First Occurrence | Last Occurrence |
|--------|-----------------|----------------|
| When found | Record + go LEFT | Record + go RIGHT |
| Base case | low > high → return result | low > high → return result |
| Direction | Continue searching left half | Continue searching right half |
| Time | O(log n) | O(log n) |
| Count formula | last - first + 1 | — |

---

## Quick Revision Questions

1. **What is the key difference** between regular binary search and finding first occurrence?
2. **Why do we continue searching** after finding the target?
3. **How do you count** total occurrences using first and last position?
4. **Trace findFirst([2,2,2,2,2], 2, 0, 4)** — what happens?
5. **What if the target isn't found?** What do both functions return?
6. **Can this technique work** on unsorted arrays? Why or why not?

---

[← Previous: Find Max/Min](05-find-max-min.md) | [Next: Unit 4 — Palindrome Check →](../04-Recursion-With-Strings/01-palindrome-check.md)

[← Back to README](../README.md)

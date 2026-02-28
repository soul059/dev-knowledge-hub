# Chapter 3: Finding Boundaries (Lower & Upper Bound)

## 📋 Chapter Overview
Finding the **first** or **last** occurrence of a value — or the **insertion point** — is the most interview-critical binary search variant. Master these and you solve 80% of binary search problems.

---

## 🧠 Core Concept: Boundary Search

### The Problem with Exact Match

Standard binary search finds **any** occurrence. But what if there are duplicates?

```
  arr = [1, 3, 3, 3, 3, 5, 7]
                ↑
  Standard BS might return index 3
  But we might need:
    First 3 → index 1 (lower bound)
    Last 3  → index 4 (upper bound)
    Insert point for 4 → index 5
```

---

## 📝 Lower Bound (First Occurrence / Left Boundary)

### Definition
Find the **leftmost** position where `arr[pos] >= target`.

### Template

```
function lowerBound(arr, target):
    lo = 0
    hi = len(arr)            // ← note: len, not len-1

    while lo < hi:           // ← strict <
        mid = lo + (hi - lo) / 2

        if arr[mid] < target:
            lo = mid + 1     // too small, go right
        else:
            hi = mid         // arr[mid] >= target, could be answer
    
    return lo                // first position where arr[pos] >= target
```

### Trace: arr = [1, 3, 3, 3, 5, 7], target = 3

```
  [1, 3, 3, 3, 5, 7]
   0  1  2  3  4  5    (indices)
```

| Step | lo | hi | mid | arr[mid] | Condition | Action |
|------|----|----|-----|----------|-----------|--------|
| 1 | 0 | 6 | 3 | 3 | 3 >= 3 | hi = 3 |
| 2 | 0 | 3 | 1 | 3 | 3 >= 3 | hi = 1 |
| 3 | 0 | 1 | 0 | 1 | 1 < 3 | lo = 1 |
| 4 | 1 | 1 | - | - | lo == hi | **Return 1** ✓ |

### Visualization

```
  Target = 3
  
  [1, 3, 3, 3, 5, 7]
   ✗  ✓  ✓  ✓  ✓  ✓   ← arr[i] >= 3 ?
      ↑
   lower bound = 1 (first ✓)
```

---

## 📝 Upper Bound (Past-the-Last / Right Boundary)

### Definition
Find the **leftmost** position where `arr[pos] > target` (strictly greater).

### Template

```
function upperBound(arr, target):
    lo = 0
    hi = len(arr)

    while lo < hi:
        mid = lo + (hi - lo) / 2

        if arr[mid] <= target:    // ← only difference: <= instead of <
            lo = mid + 1
        else:
            hi = mid
    
    return lo                     // first position where arr[pos] > target
```

### Trace: arr = [1, 3, 3, 3, 5, 7], target = 3

| Step | lo | hi | mid | arr[mid] | Condition | Action |
|------|----|----|-----|----------|-----------|--------|
| 1 | 0 | 6 | 3 | 3 | 3 <= 3 | lo = 4 |
| 2 | 4 | 6 | 5 | 7 | 7 > 3 | hi = 5 |
| 3 | 4 | 5 | 4 | 5 | 5 > 3 | hi = 4 |
| 4 | 4 | 4 | - | - | lo == hi | **Return 4** ✓ |

### Visualization

```
  Target = 3
  
  [1, 3, 3, 3, 5, 7]
   ✗  ✗  ✗  ✗  ✓  ✓   ← arr[i] > 3 ?
                ↑
   upper bound = 4 (first ✓)
   
   Last occurrence of 3 = upper_bound - 1 = 3
```

---

## 🔗 Relationship Between Bounds

```
  arr = [1, 3, 3, 3, 5, 7]
         0  1  2  3  4  5

  lower_bound(3) = 1     (first 3)
  upper_bound(3) = 4     (past last 3)
  
  Count of 3s = upper_bound - lower_bound = 4 - 1 = 3  ✓
  Last 3's index = upper_bound - 1 = 3                  ✓
  
  ┌────────────────────────┐
  │ [1, |3, 3, 3|, 5, 7]  │
  │      ↑        ↑        │
  │     LB=1    UB=4       │
  │    (first)  (past end) │
  └────────────────────────┘
```

---

## 🎯 Derived Operations

### First Occurrence

```
function firstOccurrence(arr, target):
    pos = lowerBound(arr, target)
    if pos < len(arr) AND arr[pos] == target:
        return pos
    return -1          // not found
```

### Last Occurrence

```
function lastOccurrence(arr, target):
    pos = upperBound(arr, target) - 1
    if pos >= 0 AND arr[pos] == target:
        return pos
    return -1
```

### Count Occurrences

```
function countOccurrences(arr, target):
    return upperBound(arr, target) - lowerBound(arr, target)
```

### Insertion Point

```
function insertionPoint(arr, target):
    return lowerBound(arr, target)
    // Inserting at this index keeps array sorted
```

---

## 🔍 Problem: Find First and Last Position (LeetCode 34)

**Given sorted array and target, find [first, last] position.**

```
  Input:  [5, 7, 7, 8, 8, 10], target = 8
  Output: [3, 4]
  
  Input:  [5, 7, 7, 8, 8, 10], target = 6
  Output: [-1, -1]
```

### Solution

```
function searchRange(nums, target):
    first = lowerBound(nums, target)
    
    if first == len(nums) OR nums[first] != target:
        return [-1, -1]
    
    last = upperBound(nums, target) - 1
    return [first, last]
```

### Trace: nums = [5, 7, 7, 8, 8, 10], target = 8

**Lower Bound:**

| Step | lo | hi | mid | nums[mid] | Action |
|------|----|----|-----|-----------|--------|
| 1 | 0 | 6 | 3 | 8 | 8 >= 8 → hi = 3 |
| 2 | 0 | 3 | 1 | 7 | 7 < 8 → lo = 2 |
| 3 | 2 | 3 | 2 | 7 | 7 < 8 → lo = 3 |
| → | 3 | 3 | - | - | Return 3 (first = 3) |

**Upper Bound:**

| Step | lo | hi | mid | nums[mid] | Action |
|------|----|----|-----|-----------|--------|
| 1 | 0 | 6 | 3 | 8 | 8 <= 8 → lo = 4 |
| 2 | 4 | 6 | 5 | 10 | 10 > 8 → hi = 5 |
| 3 | 4 | 5 | 4 | 8 | 8 <= 8 → lo = 5 |
| → | 5 | 5 | - | - | Return 5, last = 5-1 = 4 |

**Result: [3, 4]** ✓

---

## 🔍 Problem: Search Insert Position (LeetCode 35)

```
function searchInsert(nums, target):
    return lowerBound(nums, target)
```

This is literally just lower bound!

```
  [1, 3, 5, 6], target = 5 → index 2 (found)
  [1, 3, 5, 6], target = 2 → index 1 (insert here)
  [1, 3, 5, 6], target = 7 → index 4 (append)
```

---

## ⚠️ Critical: `hi = len(arr)` vs `hi = len(arr) - 1`

```
  ┌─────────────────────────────────────────┐
  │  Boundary Search: hi = len(arr)         │
  │  (answer might be PAST the array end)   │
  │                                         │
  │  Exact Match: hi = len(arr) - 1         │
  │  (answer must be within the array)      │
  └─────────────────────────────────────────┘
  
  Example: lowerBound([1, 2, 3], 5)
  Answer = 3 (past end) → need hi = 3 initially
```

---

## 📊 Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Lower bound | O(log n) | O(1) |
| Upper bound | O(log n) | O(1) |
| First + Last | O(log n) | O(1) |
| Count occurrences | O(log n) | O(1) |

---

## 📋 Summary Table

| Concept | Lower Bound | Upper Bound |
|---------|-------------|-------------|
| Finds | First pos ≥ target | First pos > target |
| Key condition | `arr[mid] < target` → right | `arr[mid] <= target` → right |
| hi init | `len(arr)` | `len(arr)` |
| Loop | `while lo < hi` | `while lo < hi` |
| Difference | Only the `<` vs `<=` in condition |

---

## ❓ Revision Questions

1. **Only difference between lower and upper bound code?**
   → `arr[mid] < target` (lower) vs `arr[mid] <= target` (upper).

2. **How to get count of target in sorted array?**
   → `upperBound(target) - lowerBound(target)`.

3. **Why `hi = len(arr)` instead of `len(arr) - 1`?**
   → The answer might be past the last element (e.g., all elements < target).

4. **Why `while lo < hi` instead of `while lo <= hi`?**
   → We're looking for a boundary where lo converges to hi; when `lo == hi`, that's the answer.

5. **What does lower_bound return if target is not in array?**
   → The index where target would be inserted to maintain sorted order.

6. **How to find the last occurrence?**
   → `upperBound(target) - 1`, then verify it equals target.

---

[← Previous: Binary Search on Answer](02-binary-search-on-answer.md) | [Next: Template and Variations →](04-template-and-variations.md)

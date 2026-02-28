# Chapter 1: Standard Binary Search

## 📋 Chapter Overview
Binary search is the fundamental **divide-and-conquer** search on sorted data. It eliminates half the search space every iteration, achieving **O(log n)** time.

---

## 🧠 Core Concept

### Why It Works
If an array is **sorted**, comparing the target with the **middle element** tells us which half to search next.

```
Array: [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]

Looking for 23:

  Step 1: lo=0, hi=9, mid=4
  [2, 5, 8, 12, |16|, 23, 38, 56, 72, 91]
                  ↑ mid
  arr[4]=16 < 23 → search right half

  Step 2: lo=5, hi=9, mid=7
  [_, _, _, _, _, 23, 38, |56|, 72, 91]
                          ↑ mid
  arr[7]=56 > 23 → search left half

  Step 3: lo=5, hi=6, mid=5
  [_, _, _, _, _, |23|, 38, _, _, _]
                   ↑ mid
  arr[5]=23 == 23 → FOUND at index 5
```

### Search Space Reduction

```
  n elements
  ┌──────────────────────────────┐
  │ □ □ □ □ □ □ □ □ □ □ □ □ □ □ │  Step 0: n
  └──────────────────────────────┘
  ┌──────────────┐
  │ □ □ □ □ □ □ □│                  Step 1: n/2
  └──────────────┘
  ┌──────┐
  │ □ □ □│                          Step 2: n/4
  └──────┘
  ┌──┐
  │ □│                              Step 3: n/8
  └──┘
        ...
  After log₂(n) steps → 1 element
```

---

## 📝 Template: Exact Match

```
function binarySearch(arr, target):
    lo = 0
    hi = len(arr) - 1

    while lo <= hi:                    // ← note: <=
        mid = lo + (hi - lo) / 2      // avoid overflow

        if arr[mid] == target:
            return mid                 // found
        else if arr[mid] < target:
            lo = mid + 1              // search right
        else:
            hi = mid - 1              // search left

    return -1                          // not found
```

### Why `lo + (hi - lo) / 2`?

```
  Naive: (lo + hi) / 2  → can overflow if lo + hi > INT_MAX
  Safe:  lo + (hi - lo) / 2  → always within range
  
  Example: lo = 2,000,000,000  hi = 2,000,000,001
  Naive:   (4,000,000,001) / 2  → INTEGER OVERFLOW!
  Safe:    2,000,000,000 + (1) / 2 = 2,000,000,000  ✓
```

---

## 🔍 Detailed Trace

### Example: Search for 72 in [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]

| Step | lo | hi | mid | arr[mid] | Action |
|------|----|----|-----|----------|--------|
| 1 | 0 | 9 | 4 | 16 | 16 < 72 → lo = 5 |
| 2 | 5 | 9 | 7 | 56 | 56 < 72 → lo = 8 |
| 3 | 8 | 9 | 8 | 72 | 72 == 72 → **FOUND** |

### Example: Search for 10 (not in array)

| Step | lo | hi | mid | arr[mid] | Action |
|------|----|----|-----|----------|--------|
| 1 | 0 | 9 | 4 | 16 | 16 > 10 → hi = 3 |
| 2 | 0 | 3 | 1 | 5 | 5 < 10 → lo = 2 |
| 3 | 2 | 3 | 2 | 8 | 8 < 10 → lo = 3 |
| 4 | 3 | 3 | 3 | 12 | 12 > 10 → hi = 2 |
| 5 | 3 | 2 | - | - | lo > hi → **NOT FOUND** |

---

## 🔄 Iterative vs Recursive

### Iterative (Preferred)
```
function binarySearch(arr, target):
    lo = 0, hi = len(arr) - 1
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if arr[mid] == target: return mid
        if arr[mid] < target: lo = mid + 1
        else: hi = mid - 1
    return -1
```

### Recursive
```
function binarySearch(arr, target, lo, hi):
    if lo > hi: return -1
    mid = lo + (hi - lo) / 2
    if arr[mid] == target: return mid
    if arr[mid] < target:
        return binarySearch(arr, target, mid + 1, hi)
    else:
        return binarySearch(arr, target, lo, mid - 1)
```

### Comparison

| Aspect | Iterative | Recursive |
|--------|-----------|-----------|
| Time | O(log n) | O(log n) |
| Space | O(1) | O(log n) stack |
| Speed | Faster (no call overhead) | Slightly slower |
| Interview | ✅ Preferred | Good for explanation |

---

## ⚠️ Common Pitfalls

### 1. Off-by-One Errors
```
  // WRONG: while lo < hi → misses case when target is at the only remaining element
  while lo < hi:     ❌ (for exact match)
  while lo <= hi:    ✅ (correct for exact match)
```

### 2. Infinite Loop
```
  // WRONG: not moving pointers past mid
  lo = mid       ❌ → infinite loop when lo == mid
  lo = mid + 1   ✅
  hi = mid        ← ok in some templates (boundary search)
  hi = mid - 1   ✅ (for exact match)
```

### 3. Integer Overflow
```
  mid = (lo + hi) / 2      ❌ overflow risk
  mid = lo + (hi - lo) / 2 ✅ always safe
```

---

## 📊 Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Binary Search | O(log n) | O(1) iterative |
| Linear Search | O(n) | O(1) |

### Why O(log n)?

```
  After k steps, remaining = n / 2^k
  Search ends when n / 2^k = 1
  → k = log₂(n)

  n = 1,000,000 → log₂(1,000,000) ≈ 20 steps
  vs linear: 1,000,000 steps!
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Prerequisite | Array must be SORTED |
| while condition | `lo <= hi` for exact match |
| Mid calculation | `lo + (hi - lo) / 2` to avoid overflow |
| Not found | Return -1 when `lo > hi` |
| Time | O(log n) — halving each step |
| Space | O(1) iterative, O(log n) recursive |
| Prefer | Iterative in interviews |

---

## ❓ Revision Questions

1. **Why must the array be sorted for binary search?**
   → Because we use `arr[mid]` comparison to determine which half has the target; without ordering, we can't eliminate half.

2. **What happens if you use `while lo < hi` for exact match?**
   → You miss the case when `lo == hi` and `arr[lo]` is the target.

3. **Why `lo + (hi - lo) / 2` instead of `(lo + hi) / 2`?**
   → Prevents integer overflow when `lo + hi` exceeds `INT_MAX`.

4. **How many comparisons for an array of 1 billion elements?**
   → log₂(10⁹) ≈ 30 comparisons.

5. **When would recursive binary search be preferred?**
   → Rarely in practice; iterative is preferred. Recursive can be useful for clarity in teaching or when the problem naturally recurses.

---

[← Back to README](../README.md) | [Next: Binary Search on Answer →](02-binary-search-on-answer.md)

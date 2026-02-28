# Chapter 2: Sentinel Linear Search

[← Previous: Basic Linear Search](01-basic-linear-search.md) | [Next: Linear Search Variations →](03-linear-search-variations.md)

---

## Overview

**Sentinel Search** is an optimized version of linear search that reduces the number of comparisons per iteration from **2 to 1** by eliminating the boundary check (`i < n`).

> **Key Idea:** Place the target value at the end of the array as a "sentinel" so the loop is guaranteed to find it — then check if the found position is valid.

---

## The Problem with Basic Linear Search

In basic linear search, **every iteration** performs **two comparisons**:

```
FOR i = 0 TO n - 1:          ← Comparison 1: Is i < n?
    IF arr[i] == target:      ← Comparison 2: Is this the target?
```

For an array of size `n`, this means up to **2n comparisons** in the worst case.

**Can we reduce this to ~n comparisons?** Yes! With a sentinel.

---

## How Sentinel Search Works

```
   Original Array:
   ┌────┬────┬────┬────┬────┬────┐
   │ 10 │ 23 │  5 │ 42 │ 17 │  8 │   n = 6, target = 42
   └────┴────┴────┴────┴────┴────┘

   Step 1: Save last element, place sentinel
   ┌────┬────┬────┬────┬────┬────┐
   │ 10 │ 23 │  5 │ 42 │ 17 │ 42 │   last = 8, arr[5] = 42 (sentinel)
   └────┴────┴────┴────┴────┴────┘
                                ^^
                            SENTINEL

   Step 2: Search WITHOUT boundary check
   i = 0: 10 ≠ 42, i++
   i = 1: 23 ≠ 42, i++
   i = 2:  5 ≠ 42, i++
   i = 3: 42 == 42 → STOP! i = 3

   Step 3: Restore last element
   arr[5] = 8 (restored)

   Step 4: Check — is i < n-1? Yes (3 < 5) → FOUND at index 3
```

---

## Algorithm (Pseudocode)

```
FUNCTION sentinelSearch(arr, n, target):
    // Step 1: Save last element and place sentinel
    last = arr[n - 1]
    arr[n - 1] = target
    
    // Step 2: Search without boundary check
    i = 0
    WHILE arr[i] != target:
        i = i + 1
    
    // Step 3: Restore the last element
    arr[n - 1] = last
    
    // Step 4: Check if found or it was the sentinel
    IF i < n - 1 OR arr[n - 1] == target:
        RETURN i
    ELSE:
        RETURN -1
```

---

## Step-by-Step Trace

### Example: `arr = [4, 9, 2, 7, 1, 5]`, target = `7`

**Step 1: Place sentinel**
```
   Before:  [4, 9, 2, 7, 1, 5]    last = 5
   After:   [4, 9, 2, 7, 1, 7]    sentinel placed at arr[5]
                              ^
```

**Step 2: Search (only ONE comparison per iteration)**

| Iteration | i | arr[i] | arr[i] == 7? | Action |
|-----------|---|--------|-------------|--------|
| 1 | 0 | 4 | No | i++ |
| 2 | 1 | 9 | No | i++ |
| 3 | 2 | 2 | No | i++ |
| 4 | 3 | 7 | **Yes** | **STOP** |

**Step 3:** Restore arr[5] = 5

**Step 4:** i = 3, n - 1 = 5 → 3 < 5 → **FOUND at index 3** ✓

---

### Example: Target NOT found — `target = 99`

**Step 1:** `last = 5`, place `arr[5] = 99`
```
   [4, 9, 2, 7, 1, 99]
```

**Step 2:** Search

| Iteration | i | arr[i] | arr[i] == 99? |
|-----------|---|--------|--------------|
| 1 | 0 | 4 | No |
| 2 | 1 | 9 | No |
| 3 | 2 | 2 | No |
| 4 | 3 | 7 | No |
| 5 | 4 | 1 | No |
| 6 | 5 | 99 | **Yes** (sentinel!) |

**Step 3:** Restore arr[5] = 5

**Step 4:** i = 5, n - 1 = 5 → i is NOT < n-1, and arr[5] = 5 ≠ 99 → **NOT FOUND, return -1**

---

## Comparison: Basic vs Sentinel

```
   Basic Linear Search (per iteration):
   ┌────────────────────────────────────┐
   │  Check 1: Is i < n?               │  ← Boundary check
   │  Check 2: Is arr[i] == target?    │  ← Equality check
   │  Total: 2 comparisons/iteration   │
   └────────────────────────────────────┘

   Sentinel Search (per iteration):
   ┌────────────────────────────────────┐
   │  Check 1: Is arr[i] == target?    │  ← Only equality check
   │  Total: 1 comparison/iteration    │
   └────────────────────────────────────┘
   (Boundary is guaranteed by sentinel)
```

| | Basic Linear | Sentinel Search |
|---|---|---|
| Comparisons per iteration | 2 | 1 |
| Worst case comparisons | 2n | n + 2 |
| Extra setup needed? | No | Yes (save + place + restore) |
| Array modified? | No | Temporarily yes |
| Thread-safe? | Yes | No (modifies array) |

---

## Implementation

### Java

```java
public static int sentinelSearch(int[] arr, int target) {
    int n = arr.length;
    if (n == 0) return -1;
    
    // Save last element and place sentinel
    int last = arr[n - 1];
    arr[n - 1] = target;
    
    // Search without boundary check
    int i = 0;
    while (arr[i] != target) {
        i++;
    }
    
    // Restore last element
    arr[n - 1] = last;
    
    // Check if truly found
    if (i < n - 1 || last == target) {
        return i;
    }
    return -1;
}
```

### Python

```python
def sentinel_search(arr, target):
    n = len(arr)
    if n == 0:
        return -1
    
    # Save and place sentinel
    last = arr[n - 1]
    arr[n - 1] = target
    
    # Search without boundary check
    i = 0
    while arr[i] != target:
        i += 1
    
    # Restore
    arr[n - 1] = last
    
    # Validate
    if i < n - 1 or last == target:
        return i
    return -1
```

---

## Visual Flow

```
    ┌──────────────────┐
    │   Save last elem │
    │   Place sentinel │
    └────────┬─────────┘
             │
    ┌────────▼─────────┐  No
    │ arr[i] == target? ├────────┐
    └────────┬─────────┘        │
             │ Yes         ┌────▼────┐
    ┌────────▼─────────┐   │  i++    │
    │  Restore last    │   └────┬────┘
    └────────┬─────────┘        │
             │           (Loop back ↑)
    ┌────────▼─────────┐
    │  i < n-1  OR     │
    │  last == target? │
    └───┬──────────┬───┘
        │Yes       │No
   ┌────▼───┐  ┌───▼────┐
   │Return i│  │Return -1│
   └────────┘  └────────┘
```

---

## Important Caveats

| Caveat | Explanation |
|--------|-------------|
| **Modifies the array** | Temporarily changes last element — not safe in concurrent environments |
| **Not thread-safe** | If another thread reads during search, it sees corrupted data |
| **Constant-size arrays** | If array is const/immutable, sentinel search cannot be used |
| **Empty arrays** | Must handle n = 0 separately |

---

## When to Use Sentinel Search

| Use When | Avoid When |
|----------|-----------|
| Performance matters on large unsorted data | Array is read-only or immutable |
| Single-threaded environment | Multi-threaded access |
| You need the slight speed edge | Simplicity is more important |
| Competitive programming (tight TLE) | Production code with concurrency |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Algorithm** | Place target at end, search without bounds check |
| **Best Case** | O(1) — target at index 0 |
| **Average Case** | O(n) — same as linear search |
| **Worst Case** | O(n) — but with ~half the comparisons |
| **Space Complexity** | O(1) |
| **Key Advantage** | ~50% fewer comparisons than basic linear search |
| **Key Limitation** | Temporarily modifies the array |

---

## Quick Revision Questions

1. **How does sentinel search reduce the number of comparisons compared to basic linear search?**
2. **What value is placed as the sentinel and where?**
3. **Why must we restore the last element after the search?**
4. **Is sentinel search thread-safe? Why or why not?**
5. **If the target happens to be the last element, how does the algorithm handle this correctly?**
6. **What is the total number of comparisons in the worst case for sentinel search on an array of size n?**

---

[← Previous: Basic Linear Search](01-basic-linear-search.md) | [Next: Linear Search Variations →](03-linear-search-variations.md)

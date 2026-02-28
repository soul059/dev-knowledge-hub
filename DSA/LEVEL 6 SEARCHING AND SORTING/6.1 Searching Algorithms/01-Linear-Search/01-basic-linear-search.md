# Chapter 1: Basic Linear Search

[← Back to README](../README.md) | [Next: Sentinel Search →](02-sentinel-search.md)

---

## Overview

**Linear Search** (also known as **Sequential Search**) is the simplest searching algorithm. It checks every element in a collection one by one from the beginning until it finds the target or exhausts the entire list.

> **Key Idea:** Walk through the array from index 0 to n-1; stop as soon as you find the target.

---

## Intuition

Imagine you have a shelf of unsorted books, and you need to find a specific title. You start from the left, pick up each book, read the title, and if it matches — you're done. If it doesn't, you move to the next book.

```
   Looking for: 42
   
   Index:   0    1    2    3    4    5    6    7
          ┌────┬────┬────┬────┬────┬────┬────┬────┐
   Array: │ 10 │ 23 │  5 │ 42 │ 17 │  8 │ 31 │ 56 │
          └────┴────┴────┴────┴────┴────┴────┴────┘
            ↑    ↑    ↑    ↑
           (1)  (2)  (3)  (4) ← FOUND at index 3!
```

---

## Algorithm (Pseudocode)

```
FUNCTION linearSearch(arr, n, target):
    FOR i = 0 TO n - 1:
        IF arr[i] == target:
            RETURN i          // Found! Return the index
    RETURN -1                 // Not found
```

---

## Step-by-Step Trace

### Example: `arr = [15, 8, 22, 7, 42, 3]`, target = `7`

| Step | Index (i) | arr[i] | arr[i] == 7? | Action |
|------|-----------|--------|-------------|--------|
| 1 | 0 | 15 | No | Continue |
| 2 | 1 | 8 | No | Continue |
| 3 | 2 | 22 | No | Continue |
| 4 | 3 | 7 | **Yes** | **Return 3** |

```
   Step 1:   [15]  8   22   7   42   3       ← 15 ≠ 7
              ^^
   Step 2:    15  [8]  22   7   42   3       ← 8 ≠ 7
                   ^^
   Step 3:    15   8  [22]  7   42   3       ← 22 ≠ 7
                        ^^
   Step 4:    15   8   22  [7]  42   3       ← 7 == 7 ✓ FOUND!
                             ^^
```

### Example: Target NOT found — `target = 99`

| Step | Index (i) | arr[i] | arr[i] == 99? | Action |
|------|-----------|--------|--------------|--------|
| 1 | 0 | 15 | No | Continue |
| 2 | 1 | 8 | No | Continue |
| 3 | 2 | 22 | No | Continue |
| 4 | 3 | 7 | No | Continue |
| 5 | 4 | 42 | No | Continue |
| 6 | 5 | 3 | No | Continue |
| — | — | — | — | **Return -1** |

All elements checked, none matched → Return `-1`.

---

## Implementation

### Java

```java
public static int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) {
            return i;  // Found at index i
        }
    }
    return -1;  // Not found
}
```

### Python

```python
def linear_search(arr, target):
    for i in range(len(arr)):
        if arr[i] == target:
            return i
    return -1
```

### C++

```cpp
int linearSearch(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target)
            return i;
    }
    return -1;
}
```

---

## How Linear Search Works — Visual Flow

```
                    ┌─────────────┐
                    │   START     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  i = 0      │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐     Yes
                    │  i < n ?    │────────────┐
                    └──────┬──────┘            │
                           │ No                │
                    ┌──────▼──────┐    ┌───────▼───────┐     Yes
                    │ Return -1   │    │ arr[i]==target?│──────────┐
                    │ (Not Found) │    └───────┬───────┘          │
                    └─────────────┘            │ No        ┌──────▼──────┐
                                        ┌──────▼──────┐   │  Return i   │
                                        │   i = i + 1 │   │  (Found!)   │
                                        └──────┬──────┘   └─────────────┘
                                               │
                                        (Go back to "i < n?")
```

---

## Key Properties

| Property | Value |
|----------|-------|
| **Data must be sorted?** | No |
| **Works on linked lists?** | Yes |
| **Works on any data type?** | Yes (with equality check) |
| **In-place?** | Yes (no extra space) |
| **Stable?** | Returns first occurrence |
| **Adaptive?** | No |

---

## When to Use Basic Linear Search

1. **Small arrays** (n < 20–30): Overhead of binary search isn't worth it
2. **Unsorted data**: Binary search requires sorting first
3. **Linked lists**: No random access → binary search is impractical
4. **Single query**: If you search only once, sorting + binary search is slower
5. **Searching by non-key attribute**: e.g., find employee by name in unsorted records

---

## Common Mistakes

| Mistake | Why It's Wrong |
|---------|---------------|
| Not returning -1 for not found | Caller can't distinguish "found at 0" from "not found" |
| Off-by-one: `i <= n` instead of `i < n` | ArrayIndexOutOfBounds! |
| Returning boolean only | Loses the *position* information |
| Not handling empty array | Should immediately return -1 |

---

## Real-World Applications

- **Contact search on old phones** — linear scan through contacts
- **`Array.indexOf()` in JavaScript** — internally does linear search
- **Finding in unsorted log files** — scanning line by line
- **Database full-table scan** — when no index exists

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Algorithm** | Check each element sequentially |
| **Best Case** | O(1) — target is at index 0 |
| **Average Case** | O(n) — target is somewhere in the middle |
| **Worst Case** | O(n) — target is at end or not present |
| **Space Complexity** | O(1) — no extra memory |
| **Prerequisite** | None — works on any array |

---

## Quick Revision Questions

1. **What is the worst-case time complexity of linear search and when does it occur?**
2. **Does linear search require the input array to be sorted? Why or why not?**
3. **If an array has 1000 elements, what is the maximum number of comparisons linear search will make?**
4. **What value should linear search return if the target is not found? Why not return 0?**
5. **Can linear search be applied to a linked list? What about binary search?**
6. **If the target appears multiple times, which occurrence does basic linear search return?**

---

[← Back to README](../README.md) | [Next: Sentinel Search →](02-sentinel-search.md)

# Chapter 5: Floor and Ceiling

[← Previous: Search Insert Position](04-search-insert-position.md) | [Next: Peak Element →](06-peak-element.md)

---

## Overview

**Floor** and **Ceiling** are two important binary search variations that find the closest elements to a target in a sorted array.

- **Floor:** Largest element **≤** target
- **Ceiling:** Smallest element **≥** target

---

## Definitions

```
   arr = [1, 2, 4, 6, 10, 12, 14]    target = 7

   ┌───┬───┬───┬───┬────┬────┬────┐
   │ 1 │ 2 │ 4 │ 6 │ 10 │ 12 │ 14 │
   └───┴───┴───┴───┴────┴────┴────┘
                  ↑    ↑
               FLOOR  CEILING
               (6)    (10)

   Floor(7)   = 6   (largest value ≤ 7)
   Ceiling(7) = 10  (smallest value ≥ 7)
```

```
   If target exists in array:
   arr = [1, 2, 4, 6, 10, 12, 14]    target = 6
   
   Floor(6)   = 6   (6 ≤ 6)
   Ceiling(6) = 6   (6 ≥ 6)  
   Both are the target itself!
```

---

## Floor: Largest Element ≤ Target

### Algorithm

```
FUNCTION floor(arr, n, target):
    lo = 0
    hi = n - 1
    result = -1                    // -1 means no floor exists

    WHILE lo <= hi:
        mid = lo + (hi - lo) / 2

        IF arr[mid] == target:
            RETURN mid             // Exact match is the floor
        ELSE IF arr[mid] < target:
            result = mid           // This could be the floor
            lo = mid + 1           // Check for a closer (larger) floor
        ELSE:
            hi = mid - 1           // arr[mid] too large

    RETURN result
```

### Trace: `arr = [1, 2, 4, 6, 10, 12, 14]`, Target: `7`

| Step | lo | hi | mid | arr[mid] | Action | result |
|------|----|----|-----|----------|--------|--------|
| 1 | 0 | 6 | 3 | 6 | 6 < 7: result=3, lo=4 | 3 |
| 2 | 4 | 6 | 5 | 12 | 12 > 7: hi=4 | 3 |
| 3 | 4 | 4 | 4 | 10 | 10 > 7: hi=3 | 3 |
| 4 | 4 | 3 | — | — | STOP | **3** |

```
   Floor(7) = arr[3] = 6 ✓
```

### Implementation (Java)

```java
public static int floor(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    int result = -1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            result = mid;          // Candidate for floor
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return result;
}
```

---

## Ceiling: Smallest Element ≥ Target

### Algorithm

```
FUNCTION ceiling(arr, n, target):
    lo = 0
    hi = n - 1
    result = -1                    // -1 means no ceiling exists

    WHILE lo <= hi:
        mid = lo + (hi - lo) / 2

        IF arr[mid] == target:
            RETURN mid             // Exact match is the ceiling
        ELSE IF arr[mid] > target:
            result = mid           // This could be the ceiling
            hi = mid - 1           // Check for a closer (smaller) ceiling
        ELSE:
            lo = mid + 1           // arr[mid] too small

    RETURN result
```

### Trace: `arr = [1, 2, 4, 6, 10, 12, 14]`, Target: `7`

| Step | lo | hi | mid | arr[mid] | Action | result |
|------|----|----|-----|----------|--------|--------|
| 1 | 0 | 6 | 3 | 6 | 6 < 7: lo=4 | -1 |
| 2 | 4 | 6 | 5 | 12 | 12 > 7: result=5, hi=4 | 5 |
| 3 | 4 | 4 | 4 | 10 | 10 > 7: result=4, hi=3 | **4** |
| 4 | 4 | 3 | — | — | STOP | **4** |

```
   Ceiling(7) = arr[4] = 10 ✓
```

### Implementation (Java)

```java
public static int ceiling(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    int result = -1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] > target) {
            result = mid;          // Candidate for ceiling
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return result;
}
```

---

## Python Implementations

```python
def floor_search(arr, target):
    lo, hi = 0, len(arr) - 1
    result = -1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            result = mid
            lo = mid + 1
        else:
            hi = mid - 1
    return result

def ceiling_search(arr, target):
    lo, hi = 0, len(arr) - 1
    result = -1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] > target:
            result = mid
            hi = mid - 1
        else:
            lo = mid + 1
    return result
```

---

## Edge Cases

```
   arr = [2, 4, 6, 8, 10]

   Floor(1)  = -1 (no element ≤ 1)     Ceiling(1)  = 2 (index 0)
   Floor(2)  = 2  (exact match)        Ceiling(2)  = 2 (exact match)
   Floor(5)  = 4  (index 1)            Ceiling(5)  = 6 (index 2)
   Floor(10) = 10 (exact match)        Ceiling(10) = 10 (exact match)
   Floor(11) = 10 (index 4)            Ceiling(11) = -1 (no element ≥ 11)
```

```
   ┌────────────────────────────────────────────────┐
   │  When target < all elements:                   │
   │    Floor = DOES NOT EXIST (-1)                 │
   │    Ceiling = First element                     │
   │                                                │
   │  When target > all elements:                   │
   │    Floor = Last element                        │
   │    Ceiling = DOES NOT EXIST (-1)               │
   └────────────────────────────────────────────────┘
```

---

## Side-by-Side Comparison

| Aspect | Floor | Ceiling |
|--------|-------|---------|
| **Definition** | Largest ≤ target | Smallest ≥ target |
| **On match: `arr[mid] == target`** | Return mid | Return mid |
| **When `arr[mid] < target`** | Candidate! `result=mid, lo=mid+1` | Too small: `lo=mid+1` |
| **When `arr[mid] > target`** | Too large: `hi=mid-1` | Candidate! `result=mid, hi=mid-1` |
| **No floor/ceiling exists** | Target < min(arr) | Target > max(arr) |

---

## Real-World Applications

| Application | Which One? |
|-------------|-----------|
| Round down a price to nearest discount tier | Floor |
| Round up shipping weight to nearest bracket | Ceiling |
| Find closest timestamp before an event | Floor |
| Find next available meeting slot after a time | Ceiling |
| Find last version before a feature was added | Floor |

---

## Summary Table

| Aspect | Floor | Ceiling |
|--------|-------|---------|
| **Meaning** | Largest ≤ target | Smallest ≥ target |
| **Time** | O(log n) | O(log n) |
| **Space** | O(1) | O(1) |
| **Candidate when** | `arr[mid] < target` | `arr[mid] > target` |
| **Search direction** | Go RIGHT for closer | Go LEFT for closer |

---

## Quick Revision Questions

1. **Define floor and ceiling of a target in a sorted array.**
2. **When does the floor of a target NOT exist?**
3. **If the target exists in the array, what are floor and ceiling equal to?**
4. **Trace `floor([1, 3, 5, 7, 9], 6)` and `ceiling([1, 3, 5, 7, 9], 6)`.**
5. **How are floor and ceiling related to lower_bound and upper_bound?**
6. **Give a real-world example where ceiling search is useful.**

---

[← Previous: Search Insert Position](04-search-insert-position.md) | [Next: Peak Element →](06-peak-element.md)

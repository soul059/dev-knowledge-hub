# Chapter 4: Template and Variations

## 📋 Chapter Overview
A unified view of all binary search templates and when to pick each one.

---

## 🗂️ The Three Templates

### Template 1: Exact Match

```
function exactMatch(arr, target):
    lo = 0
    hi = len(arr) - 1

    while lo <= hi:              // includes lo == hi
        mid = lo + (hi - lo) / 2
        if arr[mid] == target:
            return mid
        else if arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

**Use when:** Find a specific value. Return immediately on match.

---

### Template 2: Boundary (Lower/Upper Bound)

```
function boundary(arr, target):
    lo = 0
    hi = len(arr)                // past-end

    while lo < hi:               // converge
        mid = lo + (hi - lo) / 2
        if condition(arr[mid], target):
            lo = mid + 1
        else:
            hi = mid
    return lo                    // boundary point
```

**Use when:** Find first/last position, insertion point, or any boundary.

---

### Template 3: Answer Space

```
function answerSpace(lo, hi):
    while lo < hi:
        mid = lo + (hi - lo) / 2
        if isFeasible(mid):
            hi = mid             // minimize answer
        else:
            lo = mid + 1
    return lo
```

**Use when:** "Minimize the maximum" or "maximize the minimum."

---

## 🗺️ Template Decision Map

```
  What are you searching?
  │
  ├── A specific value in array?
  │   └── Template 1 (Exact Match)
  │       while lo <= hi, return on match
  │
  ├── A boundary/position in array?
  │   └── Template 2 (Boundary)
  │       while lo < hi, converge lo and hi
  │       ├── First >= target → condition: arr[mid] < target
  │       ├── First >  target → condition: arr[mid] <= target
  │       ├── Last <= target  → upperBound - 1
  │       └── Last <  target  → lowerBound - 1
  │
  └── A value in an answer range?
      └── Template 3 (Answer Space)
          while lo < hi, feasibility check
```

---

## 🔄 Variations on Sorted/Rotated Arrays

### Variation 1: Search in Rotated Sorted Array

```
  [4, 5, 6, 7, 0, 1, 2]
               ↑ pivot

  Key insight: one half is always sorted
```

```
function searchRotated(arr, target):
    lo = 0, hi = len(arr) - 1

    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if arr[mid] == target: return mid

        if arr[lo] <= arr[mid]:          // left half sorted
            if arr[lo] <= target < arr[mid]:
                hi = mid - 1            // target in left half
            else:
                lo = mid + 1
        else:                            // right half sorted
            if arr[mid] < target <= arr[hi]:
                lo = mid + 1            // target in right half
            else:
                hi = mid - 1
    return -1
```

### Trace: arr = [4,5,6,7,0,1,2], target = 0

| Step | lo | hi | mid | arr[mid] | Sorted half | Action |
|------|----|----|-----|----------|-------------|--------|
| 1 | 0 | 6 | 3 | 7 | Left [4..7] | 0 not in [4,7) → lo=4 |
| 2 | 4 | 6 | 5 | 1 | Right [1..2] | 0 not in (1,2] → hi=4 |
| 3 | 4 | 4 | 4 | 0 | Match! | **Return 4** ✓ |

---

### Variation 2: Find Minimum in Rotated Array

```
function findMin(arr):
    lo = 0, hi = len(arr) - 1

    while lo < hi:
        mid = lo + (hi - lo) / 2
        if arr[mid] > arr[hi]:
            lo = mid + 1       // min is in right half
        else:
            hi = mid           // min could be mid
    return arr[lo]
```

```
  [4, 5, 6, 7, 0, 1, 2]
  
  Step 1: mid=3, arr[3]=7 > arr[6]=2 → lo=4
  Step 2: mid=5, arr[5]=1 < arr[6]=2 → hi=5
  Step 3: mid=4, arr[4]=0 < arr[5]=1 → hi=4
  lo==hi==4 → return arr[4] = 0  ✓
```

---

### Variation 3: Peak Element

```
  Find any peak (element greater than neighbors)
  
  [1, 2, 3, 1]  → peak at index 2 (value 3)
```

```
function findPeak(arr):
    lo = 0, hi = len(arr) - 1

    while lo < hi:
        mid = lo + (hi - lo) / 2
        if arr[mid] < arr[mid + 1]:
            lo = mid + 1       // peak is to the right
        else:
            hi = mid           // peak is at mid or left
    return lo
```

---

### Variation 4: Search in 2D Matrix

```
  Matrix (m × n), sorted row by row:
  ┌──────────────┐
  │  1  3  5  7  │
  │ 10 11 16 20  │
  │ 23 30 34 60  │
  └──────────────┘
  
  Treat as 1D array of size m*n
  Index i → row = i / n, col = i % n
```

```
function searchMatrix(matrix, target):
    m = rows, n = cols
    lo = 0, hi = m * n - 1

    while lo <= hi:
        mid = lo + (hi - lo) / 2
        val = matrix[mid / n][mid % n]
        if val == target: return true
        if val < target: lo = mid + 1
        else: hi = mid - 1
    return false
```

---

## 📊 Template Comparison

| Feature | T1: Exact | T2: Boundary | T3: Answer |
|---------|-----------|-------------|------------|
| While | `lo <= hi` | `lo < hi` | `lo < hi` |
| hi init | `len-1` | `len` | `max_answer` |
| Returns | Index or -1 | Boundary pos | Optimal val |
| Terminates | On match or empty | lo == hi | lo == hi |
| Handles dupes | Any occurrence | First/last | N/A |

---

## ⚠️ Pitfall Summary

| Pitfall | Wrong | Right |
|---------|-------|-------|
| While condition | Mix up `<` and `<=` | T1: `<=`, T2/T3: `<` |
| hi initialization | Always `len-1` | T2/T3 may need `len` |
| Update rule | `lo = mid` | `lo = mid + 1` (avoid infinite loop) |
| Rotated array | Forget to check which half is sorted | Always check `arr[lo] <= arr[mid]` |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| 3 templates | Exact match, boundary, answer space |
| Template choice | Based on what you're searching for |
| Rotated array | One half always sorted → decide direction |
| 2D matrix | Flatten to 1D index: row = i/n, col = i%n |
| Peak element | Compare mid with mid+1, move toward increase |

---

## ❓ Revision Questions

1. **When do you use `while lo <= hi` vs `while lo < hi`?**
   → `<=` for exact match (T1); `<` for boundary/answer (T2, T3).

2. **In rotated array search, how do you know which half is sorted?**
   → If `arr[lo] <= arr[mid]`, the left half is sorted; otherwise, the right half is.

3. **How to search a sorted 2D matrix as 1D?**
   → Map index `i` to `matrix[i/cols][i%cols]`.

4. **Why does `lo = mid` cause infinite loops but `hi = mid` doesn't?**
   → When `lo == mid` (only 2 elements left), `lo = mid` doesn't advance. `hi = mid` does shrink the range.

5. **What template for "find minimum in rotated array"?**
   → Template 2 (boundary): compare `arr[mid]` with `arr[hi]`, converge to minimum.

---

[← Previous: Finding Boundaries](03-finding-boundaries.md) | [Next: Classic Problems →](05-classic-problems.md)

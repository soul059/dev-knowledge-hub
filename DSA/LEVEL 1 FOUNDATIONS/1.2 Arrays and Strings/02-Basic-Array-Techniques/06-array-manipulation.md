# Chapter 6: Array Manipulation

[← Previous: Rotation](05-rotation.md) | [Back to README](../README.md) | [Next Unit: Two Pointer Technique →](../03-Two-Pointer-Technique/01-what-is-two-pointer.md)

---

## Overview

Array manipulation covers common operations — rearranging, partitioning, merging, removing duplicates, and other transformations. These are building blocks for more complex algorithms.

---

## 1. Removing Duplicates (Sorted Array)

Since the array is sorted, duplicates are adjacent. Use two pointers:

```
FUNCTION removeDuplicates(arr, n):
    IF n == 0: RETURN 0
    writeIdx = 1
    FOR readIdx = 1 TO n-1:
        IF arr[readIdx] != arr[readIdx - 1]:
            arr[writeIdx] = arr[readIdx]
            writeIdx = writeIdx + 1
    RETURN writeIdx     // new length
```

### Trace: [1, 1, 2, 2, 3, 4, 4]

```
  writeIdx(W) = 1, readIdx(R) starts at 1

  R=1: arr[1]=1, arr[0]=1, same → skip
  R=2: arr[2]=2, arr[1]=1, diff → arr[W=1]=2, W=2
  R=3: arr[3]=2, arr[2]=2, same → skip
  R=4: arr[4]=3, arr[3]=2, diff → arr[W=2]=3, W=3
  R=5: arr[5]=4, arr[4]=3, diff → arr[W=3]=4, W=4
  R=6: arr[6]=4, arr[5]=4, same → skip

  Result: [1, 2, 3, 4, _, _, _]  new length = 4

  ┌───┬───┬───┬───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │ . │ . │ . │   (. = don't care)
  └───┴───┴───┴───┴───┴───┴───┘
   useful part ────┘
```

**Time: O(n)** | **Space: O(1)**

---

## 2. Move Zeros to End

Move all zeros to the end while maintaining order of non-zero elements.

```
FUNCTION moveZeros(arr, n):
    writeIdx = 0
    // Move all non-zero elements to front
    FOR i = 0 TO n-1:
        IF arr[i] != 0:
            arr[writeIdx] = arr[i]
            writeIdx = writeIdx + 1
    // Fill remaining with zeros
    WHILE writeIdx < n:
        arr[writeIdx] = 0
        writeIdx = writeIdx + 1
```

### Trace: [0, 1, 0, 3, 12]

```
  Pass 1 (collect non-zeros):
  i=0: arr[0]=0  → skip
  i=1: arr[1]=1  → arr[0]=1, W=1
  i=2: arr[2]=0  → skip
  i=3: arr[3]=3  → arr[1]=3, W=2
  i=4: arr[4]=12 → arr[2]=12, W=3

  Intermediate: [1, 3, 12, 3, 12]

  Pass 2 (fill zeros):
  arr[3]=0, arr[4]=0

  Result: [1, 3, 12, 0, 0] ✓
```

**Time: O(n)** | **Space: O(1)**

---

## 3. Dutch National Flag (3-Way Partition)

Sort an array containing only 0s, 1s, and 2s in a single pass.

```
FUNCTION dutchFlag(arr, n):
    low = 0          // boundary for 0s
    mid = 0          // current element
    high = n - 1     // boundary for 2s

    WHILE mid <= high:
        IF arr[mid] == 0:
            SWAP(arr[low], arr[mid])
            low = low + 1
            mid = mid + 1
        ELSE IF arr[mid] == 1:
            mid = mid + 1
        ELSE:  // arr[mid] == 2
            SWAP(arr[mid], arr[high])
            high = high - 1
            // don't increment mid! swapped element unknown
```

### Trace: [2, 0, 1, 2, 0, 1]

```
  L=0, M=0, H=5

  M=0: arr[0]=2 → swap(arr[0],arr[5]) → [1,0,1,2,0,2], H=4
  M=0: arr[0]=1 → M=1
  M=1: arr[1]=0 → swap(arr[0],arr[1]) → [0,1,1,2,0,2], L=1, M=2
  M=2: arr[2]=1 → M=3
  M=3: arr[3]=2 → swap(arr[3],arr[4]) → [0,1,1,0,2,2], H=3
  M=3: arr[3]=0 → swap(arr[1],arr[3]) → [0,0,1,1,2,2], L=2, M=4
  M=4: M > H → STOP

  Result: [0, 0, 1, 1, 2, 2] ✓

  ┌───┬───┬───┬───┬───┬───┐
  │ 0 │ 0 │ 1 │ 1 │ 2 │ 2 │
  └───┴───┴───┴───┴───┴───┘
   0s ──┘   1s ──┘   2s ──┘
```

**Time: O(n)** | **Space: O(1)** | **Single pass!**

---

## 4. Merge Two Sorted Arrays

```
FUNCTION mergeSorted(a, m, b, n):
    result = NEW ARRAY[m + n]
    i = 0, j = 0, k = 0

    WHILE i < m AND j < n:
        IF a[i] <= b[j]:
            result[k] = a[i]
            i = i + 1
        ELSE:
            result[k] = b[j]
            j = j + 1
        k = k + 1

    // Copy remaining from a
    WHILE i < m:
        result[k] = a[i]; i++; k++
    // Copy remaining from b
    WHILE j < n:
        result[k] = b[j]; j++; k++

    RETURN result
```

### Trace

```
  a = [1, 3, 5, 7]    b = [2, 4, 6]

  Compare 1,2 → take 1     result = [1]
  Compare 3,2 → take 2     result = [1,2]
  Compare 3,4 → take 3     result = [1,2,3]
  Compare 5,4 → take 4     result = [1,2,3,4]
  Compare 5,6 → take 5     result = [1,2,3,4,5]
  Compare 7,6 → take 6     result = [1,2,3,4,5,6]
  a remaining: 7            result = [1,2,3,4,5,6,7]
```

**Time: O(m + n)** | **Space: O(m + n)**

---

## 5. Partition Around a Pivot

Rearrange so elements < pivot are on left, ≥ pivot on right.

```
FUNCTION partition(arr, n, pivot):
    writeIdx = 0
    FOR i = 0 TO n-1:
        IF arr[i] < pivot:
            SWAP(arr[writeIdx], arr[i])
            writeIdx = writeIdx + 1
    RETURN writeIdx     // pivot position
```

```
  Partition [8, 3, 5, 1, 4, 2] around pivot = 4

  i=0: 8<4? No
  i=1: 3<4? Yes → swap(arr[0],arr[1]) → [3,8,5,1,4,2], W=1
  i=2: 5<4? No
  i=3: 1<4? Yes → swap(arr[1],arr[3]) → [3,1,5,8,4,2], W=2
  i=4: 4<4? No
  i=5: 2<4? Yes → swap(arr[2],arr[5]) → [3,1,2,8,4,5], W=3

  Result: [3, 1, 2 | 8, 4, 5]
           < pivot   ≥ pivot
```

**Time: O(n)** | **Space: O(1)**

---

## 6. Intersection and Union

### Intersection (Sorted Arrays)
```
FUNCTION intersection(a, m, b, n):
    i = 0, j = 0
    result = []
    WHILE i < m AND j < n:
        IF a[i] == b[j]:
            result.add(a[i])
            i++; j++
        ELSE IF a[i] < b[j]:
            i++
        ELSE:
            j++
    RETURN result
```

```
  a = [1, 2, 3, 4, 5]     b = [2, 4, 6, 8]

  1<2: i++   |  2==2: add 2, i++,j++  |  3<4: i++
  4==4: add 4, i++,j++  |  5<6: i++   |  i=5, done

  Intersection: [2, 4]
```

### Union (Sorted Arrays)
```
  a = [1, 2, 3]    b = [2, 4, 5]

  Take 1 (only in a), Take 2 (both), Take 3 (only in a),
  Take 4 (only in b), Take 5 (only in b)

  Union: [1, 2, 3, 4, 5]
```

---

## 7. Rearrange: Positive and Negative Alternating

```
  Input:  [1, -2, 3, -4, 5, -6]
  Output: [1, -2, 3, -4, 5, -6]  (or any valid alternating)

  Pattern:
  ┌────┬────┬────┬────┬────┬────┐
  │ +  │ -  │ +  │ -  │ +  │ -  │
  └────┴────┴────┴────┴────┴────┘
```

---

## Manipulation Patterns Cheat Sheet

```
  ┌────────────────────────────────────────────────┐
  │       COMMON MANIPULATION PATTERNS              │
  ├────────────────────────────────────────────────┤
  │                                                 │
  │  Read-Write Pointer (two-pointer, same dir)     │
  │  ───────────────────────────────────────────    │
  │  Used for: remove duplicates, move zeros,       │
  │            filter elements in-place              │
  │  Pattern: readIdx scans, writeIdx places         │
  │                                                 │
  │  Three-Way Partition                             │
  │  ───────────────────────────────────────────    │
  │  Used for: Dutch flag, sort 3 values,            │
  │            partition around pivot                │
  │  Pattern: low, mid, high pointers                │
  │                                                 │
  │  Merge Pattern                                   │
  │  ───────────────────────────────────────────    │
  │  Used for: merge sorted arrays, intersection,    │
  │            union, merge sort                     │
  │  Pattern: two pointers on two arrays             │
  └────────────────────────────────────────────────┘
```

---

## Summary Table

| Operation | Time | Space | Key Technique |
|-----------|------|-------|---------------|
| Remove duplicates (sorted) | O(n) | O(1) | Read-write pointers |
| Move zeros | O(n) | O(1) | Read-write pointers |
| Dutch National Flag | O(n) | O(1) | Three pointers |
| Merge sorted arrays | O(m+n) | O(m+n) | Two pointers |
| Partition around pivot | O(n) | O(1) | Swap to front |
| Intersection (sorted) | O(m+n) | O(min(m,n)) | Two pointers |
| Union (sorted) | O(m+n) | O(m+n) | Two pointers |

---

## Quick Revision Questions

1. **How does the read-write pointer technique work?** When is the write pointer behind the read pointer?

2. **In the Dutch National Flag problem, why don't we increment mid when we swap with high?**

3. **What is the time complexity of merging two sorted arrays of size m and n?** Why?

4. **How would you remove duplicates from an unsorted array?** What's the time/space trade-off?

5. **Explain the partition operation's role in QuickSort.**

6. **Can you find the intersection of two unsorted arrays in O(n) time?** What data structure would help?

---

[← Previous: Rotation](05-rotation.md) | [Back to README](../README.md) | [Next Unit: Two Pointer Technique →](../03-Two-Pointer-Technique/01-what-is-two-pointer.md)

# Chapter 4: Template and Variations

## 📋 Chapter Overview
All two-pointer templates consolidated in one place with a decision framework for choosing the right variant.

---

## 🏗️ Master Template Collection

### Template 1: Same Direction (Read-Write)

```
function readWrite(arr):
    write = 0
    for read = 0 to n - 1:
        if shouldKeep(arr[read]):
            arr[write] = arr[read]
            write += 1
    return write
```

### Template 2: Opposite Direction (Sorted Pair)

```
function pairSearch(arr, target):
    left = 0, right = n - 1
    while left < right:
        sum = arr[left] + arr[right]
        if sum == target: return [left, right]
        elif sum < target: left++
        else: right--
    return NOT_FOUND
```

### Template 3: Opposite Direction (Squeeze)

```
function squeeze(arr):
    left = 0, right = n - 1
    result = initial
    while left < right:
        result = best(result, compute(arr[left], arr[right]))
        if arr[left] < arr[right]: left++
        else: right--
    return result
```

### Template 4: Fast/Slow (Cycle)

```
function cycleDetect(head):
    slow = head, fast = head
    while fast AND fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast: return true
    return false
```

### Template 5: Three Pointers (Dutch National Flag)

```
function dutchFlag(arr):
    low = 0, mid = 0, high = n - 1
    while mid <= high:
        if arr[mid] == 0:
            swap(arr[low], arr[mid])
            low++, mid++
        elif arr[mid] == 1:
            mid++
        else:  // arr[mid] == 2
            swap(arr[mid], arr[high])
            high--
```

---

## 🗺️ Decision Map

```
  ┌──────────────────────────────────┐
  │  What type of two-pointer?      │
  └──────────┬───────────────────────┘
             │
    ┌────────┼────────┬──────────┐
    ▼        ▼        ▼          ▼
  SORTED   IN-PLACE  LINKED    3-WAY
  PAIR?    MODIFY?   LIST?     PARTITION?
    │        │        │          │
    ▼        ▼        ▼          ▼
  Opposite Same Dir  Fast/Slow  Dutch
  Direction (R-W)    Pointers   Nat. Flag
```

---

## 📊 Complexity Comparison

| Variant | Time | Space | Prerequisite |
|---------|------|-------|-------------|
| Same Direction | O(n) | O(1) | None |
| Opposite Direction | O(n) | O(1) | Often sorted |
| Fast/Slow | O(n) | O(1) | Linked list / sequence |
| K-Sum (fix + 2ptr) | O(n^(k-1)) | O(1) | Sorting required |
| Dutch National Flag | O(n) | O(1) | 3 distinct values |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| 5 Templates | Same-dir, Opposite, Fast/Slow, K-Sum, DNF |
| Choice Key | Data structure + operation type determines variant |
| Sorted → Opposite | Pair searching in sorted arrays |
| In-place → Same | Removing/filtering elements |
| Linked List → Fast/Slow | Cycle detection, finding middle |

---

## ❓ Quick Revision Questions

1. **Which template for "remove duplicates from sorted array"?** → Same Direction (Read-Write)
2. **Which template for "find pair with target sum in sorted array"?** → Opposite Direction
3. **Which template for "detect cycle in linked list"?** → Fast/Slow
4. **Which template for "sort colors (0,1,2)"?** → Dutch National Flag
5. **What is common to all two-pointer techniques?** → O(n) time, O(1) space, single pass or converging pass
6. **Can opposite-direction work on unsorted data?** → Rarely; container-with-water is a notable exception

---

[← Previous: Fast and Slow](03-fast-and-slow.md) | [Next: Classic Problems →](05-classic-problems.md)

[← Back to README](../README.md)

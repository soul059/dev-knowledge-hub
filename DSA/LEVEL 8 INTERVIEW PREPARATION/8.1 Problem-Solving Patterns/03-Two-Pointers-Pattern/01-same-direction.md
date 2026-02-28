# Chapter 1: Same Direction (Two Pointers)

## 📋 Chapter Overview
Two pointers moving in the **same direction** — typically a slow pointer and a fast pointer, or a read pointer and write pointer. Used for in-place array modifications, partitioning, and removing elements.

---

## 🎯 Core Concept

```
  SAME DIRECTION TWO POINTERS
  
  Both pointers start at the beginning and move RIGHT.
  One pointer reads, the other writes (or one leads, one follows).
  
  ┌────────────────────────────────────────────────┐
  │   Array: [ 1  1  2  2  3  4  4  5 ]           │
  │           ↑                                    │
  │          slow                                  │
  │          fast                                  │
  │                                                │
  │   fast scans every element                     │
  │   slow marks the "write" position              │
  │                                                │
  │   [ 1  1  2  2  3  4  4  5 ]                   │
  │     ↑  ↑                                       │
  │     s  f     f finds new value → write at s+1  │
  │                                                │
  │   Result: [ 1  2  3  4  5  ...  ]              │
  │                         ↑                      │
  │                        slow = 5 unique elements │
  └────────────────────────────────────────────────┘
```

---

## 📐 The Template

```
PSEUDOCODE — Same Direction Two Pointers:

function sameDirectionTwoPointers(arr):
    slow = 0              // write position or anchor
    
    for fast = 0 to n - 1:
        if condition(arr[fast], arr[slow]):
            slow += 1
            arr[slow] = arr[fast]    // write
    
    return slow + 1    // number of valid elements
```

---

## 🧪 Trace: Remove Duplicates from Sorted Array

**Problem:** Remove duplicates in-place from sorted array, return number of unique elements.

**Input:** `[1, 1, 2, 2, 3, 4, 4, 5]`

```
  ┌──────┬──────┬──────┬───────────────────────────────────┐
  │ fast │ slow │ cond │ Array State                        │
  ├──────┼──────┼──────┼───────────────────────────────────┤
  │  0   │  0   │  -   │ [1, 1, 2, 2, 3, 4, 4, 5]  init   │
  │  1   │  0   │ 1==1 │ same, skip                        │
  │  2   │  0   │ 2≠1  │ slow=1, write → [1,2,2,2,3,4,4,5]│
  │  3   │  1   │ 2==2 │ same, skip                        │
  │  4   │  1   │ 3≠2  │ slow=2, write → [1,2,3,2,3,4,4,5]│
  │  5   │  2   │ 4≠3  │ slow=3, write → [1,2,3,4,3,4,4,5]│
  │  6   │  3   │ 4==4 │ same, skip                        │
  │  7   │  3   │ 5≠4  │ slow=4, write → [1,2,3,4,5,4,4,5]│
  └──────┴──────┴──────┴───────────────────────────────────┘
  
  Result: First 5 elements [1, 2, 3, 4, 5], return 5
```

### Pseudocode:

```
function removeDuplicates(arr):
    if len(arr) == 0: return 0
    
    slow = 0
    
    for fast = 1 to len(arr) - 1:
        if arr[fast] != arr[slow]:
            slow += 1
            arr[slow] = arr[fast]
    
    return slow + 1
```

**Time:** O(n)  **Space:** O(1)

---

## 🧪 Trace: Remove Element

**Problem:** Remove all instances of value `val` in-place.

**Input:** `arr = [3, 2, 2, 3]`, `val = 3`

```
  ┌──────┬──────┬──────┬───────────────────────┐
  │ fast │ slow │ cond │ Array                  │
  ├──────┼──────┼──────┼───────────────────────┤
  │  0   │  0   │ 3==3 │ skip (is val)          │
  │  1   │  0   │ 2≠3  │ write → [2, 2, 2, 3]  │
  │  2   │  1   │ 2≠3  │ write → [2, 2, 2, 3]  │
  │  3   │  2   │ 3==3 │ skip (is val)          │
  └──────┴──────┴──────┴───────────────────────┘
  
  Result: [2, 2, ...], return 2
```

---

## 🧪 Trace: Move Zeroes

**Problem:** Move all zeros to the end, maintaining relative order of non-zero elements.

**Input:** `[0, 1, 0, 3, 12]`

```
  slow tracks next position for non-zero element
  
  ┌──────┬──────┬───────┬──────────────────────┐
  │ fast │ slow │ cond  │ Array                 │
  ├──────┼──────┼───────┼──────────────────────┤
  │  0   │  0   │ 0==0  │ skip                  │
  │  1   │  0   │ 1≠0   │ swap → [1, 0, 0, 3, 12] │
  │  2   │  1   │ 0==0  │ skip                  │
  │  3   │  1   │ 3≠0   │ swap → [1, 3, 0, 0, 12] │
  │  4   │  2   │ 12≠0  │ swap → [1, 3, 12, 0, 0] │
  └──────┴──────┴───────┴──────────────────────┘
  
  Result: [1, 3, 12, 0, 0] ✓
```

---

## 🔍 When Same-Direction Works

```
  ┌──────────────────────────────────────────────┐
  │  USE SAME-DIRECTION WHEN:                    │
  │                                              │
  │  ✓ In-place modification of array            │
  │  ✓ Removing/filtering elements               │
  │  ✓ Partitioning (DNF / quick-select)         │
  │  ✓ Merging overlapping elements              │
  │  ✓ One pointer reads, other writes           │
  └──────────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Structure | Both pointers move left to right |
| Slow pointer | Marks write position / anchor |
| Fast pointer | Scans every element |
| Key operation | Write at slow when condition met |
| Time | O(n) — single pass |
| Space | O(1) — in-place |

---

## ❓ Quick Revision Questions

1. **What role does the slow pointer play?**
   > It marks the write position — where the next valid element should be placed.

2. **Why is this O(1) space?**
   > Modifications happen in-place; no extra array is needed.

3. **How does "move zeroes" differ from "remove element"?**
   > Move zeroes uses swap instead of overwrite, preserving all elements (just reordered).

4. **What is the invariant maintained by the slow pointer?**
   > Everything at indices `0..slow` is the "processed" valid output.

5. **Can this work on unsorted arrays?**
   > Yes — remove element and move zeroes work on unsorted arrays.

6. **What's the return value convention?**
   > Return `slow + 1` (count of valid elements) or just `slow` depending on initialization.

---

[Next: Opposite Direction →](02-opposite-direction.md)

[← Back to README](../README.md)

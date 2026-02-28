# Chapter 4: Binary Search on Answer

## Overview

One of the most powerful Binary Search techniques is **Binary Search on the Answer**. Instead of searching for an element in an array, you search for the **optimal value** in a range of possible answers. This pattern applies to many optimization and decision problems.

---

## The Core Idea

```
┌──────────────────────────────────────────────────────────────┐
│  BINARY SEARCH ON ANSWER                                     │
│                                                              │
│  Instead of: "Find target in array"                          │
│  Think:      "Find the optimal answer in range [lo, hi]"     │
│                                                              │
│  For each candidate answer 'mid':                            │
│    → Check if 'mid' is FEASIBLE (satisfies the constraint)  │
│    → If feasible: try a better answer (move one direction)   │
│    → If not feasible: relax constraint (move other direction)│
│                                                              │
│  KEY REQUIREMENT: The feasibility function must be MONOTONIC │
│  (Once feasible, all values beyond are also feasible,        │
│   OR once infeasible, all values beyond are also infeasible) │
└──────────────────────────────────────────────────────────────┘
```

### Monotonicity Visualized

```
Answer value:  1  2  3  4  5  6  7  8  9  10
Feasible?:     ✗  ✗  ✗  ✗  ✓  ✓  ✓  ✓  ✓  ✓
                              ▲
                              │
                     First feasible = ANSWER
                     
Binary Search works because the ✗/✓ boundary is clear!
```

---

## Pattern: Minimization Problem

**"Find the minimum value of X such that condition C(X) is true"**

```
function minimizeAnswer(lo, hi):
    result = hi              // worst case answer
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if isFeasible(mid):
            result = mid     // this works, try smaller
            hi = mid - 1
        else:
            lo = mid + 1     // too small, try larger
    return result
```

---

## Pattern: Maximization Problem

**"Find the maximum value of X such that condition C(X) is true"**

```
function maximizeAnswer(lo, hi):
    result = lo              // worst case answer
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if isFeasible(mid):
            result = mid     // this works, try larger
            lo = mid + 1
        else:
            hi = mid - 1     // too large, try smaller
    return result
```

---

## Example 1: Square Root (Integer)

**Problem**: Find ⌊√n⌋ (largest integer whose square ≤ n).

```
function intSqrt(n):
    lo = 0, hi = n, result = 0
    
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if mid * mid <= n:
            result = mid         // mid works, try larger
            lo = mid + 1
        else:
            hi = mid - 1         // mid² > n, try smaller
    
    return result
```

### Trace: n = 27

```
lo=0, hi=27, mid=13: 13²=169 > 27    → hi=12
lo=0, hi=12, mid=6:  6²=36  > 27     → hi=5
lo=0, hi=5,  mid=2:  2²=4   ≤ 27     → result=2, lo=3
lo=3, hi=5,  mid=4:  4²=16  ≤ 27     → result=4, lo=5
lo=5, hi=5,  mid=5:  5²=25  ≤ 27     → result=5, lo=6
lo=6, hi=5:  STOP

Answer: 5 (since 5² = 25 ≤ 27 < 36 = 6²) ✓
```

---

## Example 2: Minimum Pages Allocation

**Problem**: Allocate n books (with given pages) to k students such that the **maximum pages assigned to any student is minimized**. Each student gets contiguous books.

```
Books: [12, 34, 67, 90]
Students: 2

Possible maximum allocations:
  [12 | 34, 67, 90] → max = 191
  [12, 34 | 67, 90] → max = 157
  [12, 34, 67 | 90] → max = 113  ← OPTIMAL!

Answer space: [90, 203]  (min = max book, max = sum of all)
```

### Solution

```
function minPages(books, k):
    lo = max(books)              // at least the largest book
    hi = sum(books)              // at most all books to 1 student
    result = hi
    
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if canAllocate(books, k, mid):
            result = mid         // feasible, try smaller max
            hi = mid - 1
        else:
            lo = mid + 1         // not feasible, increase max
    
    return result

function canAllocate(books, k, maxPages):
    students = 1
    currentPages = 0
    for each book in books:
        if currentPages + book > maxPages:
            students += 1         // new student
            currentPages = book
            if students > k:
                return false      // too many students needed
        else:
            currentPages += book
    return true
```

### Trace

```
Books: [12, 34, 67, 90], k = 2

lo=90, hi=203, mid=146:
  canAllocate(146)? [12,34,67]=113 | [90]=90 → 2 students ✓
  result=146, hi=145

lo=90, hi=145, mid=117:
  canAllocate(117)? [12,34,67]=113 | [90]=90 → 2 students ✓
  result=117, hi=116

lo=90, hi=116, mid=103:
  canAllocate(103)? [12,34]=46 | [67]=67 then +90>103 → 
  [12,34]=46 | [67,90]=157 > 103 → needs 3 students ✗
  lo=104

lo=104, hi=116, mid=110:
  canAllocate(110)? [12,34]=46 | [67]=67 then +90>110 →
  needs 3 students ✗ → lo=111

lo=111, hi=116, mid=113:
  canAllocate(113)? [12,34,67]=113 | [90]=90 → 2 students ✓
  result=113, hi=112

lo=111, hi=112, mid=111:
  canAllocate(111)? [12,34]=46 | [67]=67+? → [12,34]=46,[67,90]=157>111
  → needs 3 students ✗ → lo=112

lo=112, hi=112, mid=112:
  canAllocate(112)? [12,34,67]=113>112 → [12,34]=46,[67,90]=157>112
  → ✗ → lo=113

lo=113, hi=112: STOP

Answer: 113 ✓
```

---

## Example 3: Aggressive Cows (Maximize Minimum Distance)

**Problem**: Place k cows in n stalls. Maximize the **minimum distance** between any two cows.

```
Stalls: [1, 2, 4, 8, 9]
Cows: 3

Answer space: [1, 8]  (min = 1, max = last - first)

Binary search for the largest minimum distance.
```

```
function aggressiveCows(stalls, k):
    sort(stalls)
    lo = 1
    hi = stalls[n-1] - stalls[0]
    result = 0
    
    while lo <= hi:
        mid = lo + (hi - lo) / 2
        if canPlace(stalls, k, mid):
            result = mid        // feasible, try larger distance
            lo = mid + 1
        else:
            hi = mid - 1        // not feasible, try smaller
    
    return result

function canPlace(stalls, k, minDist):
    count = 1
    lastPos = stalls[0]
    for i = 1 to n-1:
        if stalls[i] - lastPos >= minDist:
            count += 1
            lastPos = stalls[i]
    return count >= k
```

---

## Example 4: Kth Smallest in Sorted Matrix

**Problem**: Find the kth smallest element in a row-wise and column-wise sorted matrix.

```
Matrix:     k = 8
[ 1,  5,  9]
[10, 11, 13]
[12, 13, 15]

Answer space: [1, 15]
Binary search on value; count elements ≤ mid.
```

---

## The Template

```
┌───────────────────────────────────────────────────────────┐
│  BINARY SEARCH ON ANSWER TEMPLATE                         │
│                                                           │
│  1. Identify the answer RANGE: [lo, hi]                   │
│                                                           │
│  2. Write the FEASIBILITY CHECK: isFeasible(mid)          │
│     This must be O(n) or O(n log n)                       │
│                                                           │
│  3. Verify MONOTONICITY:                                  │
│     If answer X is feasible, then X+1 also feasible       │
│     (or vice versa for maximization)                      │
│                                                           │
│  4. Apply binary search on the range                      │
│                                                           │
│  Total complexity: O(log(range) × cost_of_check)          │
└───────────────────────────────────────────────────────────┘
```

---

## Practice Problem Types

| Problem | What to Binary Search | Feasibility Check |
|---------|----------------------|-------------------|
| Square root | The answer value | mid² ≤ n? |
| Min pages allocation | Maximum pages per person | Can allocate with k people? |
| Aggressive cows | Minimum distance | Can place k cows? |
| Painter's partition | Maximum board length per painter | Can paint with k painters? |
| Capacity to ship | Ship capacity | Can ship in D days? |
| Split array largest sum | Largest subarray sum | Can split into k parts? |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| What | Binary search on the answer value, not array index |
| Requirement | Feasibility must be monotonic |
| Minimization | When feasible, search left (try smaller) |
| Maximization | When feasible, search right (try larger) |
| Complexity | O(log(range) × check cost) |
| Common pattern | isFeasible(mid) returns true/false |

---

## Quick Revision Questions

1. **What property must the feasibility function have for "binary search on answer" to work?**
2. **In the minimum pages problem, what defines the search range [lo, hi]?**
3. **How does the binary search template differ for minimization vs maximization problems?**
4. **What is the time complexity of binary search on answer if the range is R and the check costs O(n)?**
5. **For the aggressive cows problem, what are you binary searching on, and what is the feasibility check?**
6. **Give an example of a problem where binary search on answer applies but the array isn't sorted.**

---

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: Lower and Upper Bounds →](05-lower-and-upper-bounds.md)

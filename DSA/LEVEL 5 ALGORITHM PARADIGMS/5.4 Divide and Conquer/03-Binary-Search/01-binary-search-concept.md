# Chapter 1: Binary Search Concept

## Overview

**Binary Search** is the simplest and most elegant Divide and Conquer algorithm. It finds a target value in a **sorted array** by repeatedly cutting the search space in half. Despite its simplicity, it is the foundation for countless algorithmic techniques.

---

## The Core Idea

```
┌─────────────────────────────────────────────────────────────┐
│  Instead of checking every element (O(n)),                 │
│  compare with the MIDDLE element and                       │
│  ELIMINATE HALF the array each step → O(log n)             │
└─────────────────────────────────────────────────────────────┘
```

### Visual Analogy: Guessing Game

```
I'm thinking of a number between 1 and 100.

You: "50?"    Me: "Higher"    → Eliminate 1-50
You: "75?"    Me: "Lower"     → Eliminate 76-100
You: "62?"    Me: "Higher"    → Eliminate 51-62
You: "68?"    Me: "Lower"     → Eliminate 69-75
You: "65?"    Me: "Higher"    → Eliminate 63-65
You: "67?"    Me: "Correct!"  → Found in 6 guesses!

Maximum guesses for 100 numbers: ⌈log₂(100)⌉ = 7
Maximum guesses for 1,000,000: ⌈log₂(1,000,000)⌉ = 20
```

---

## How Binary Search Works

```
Sorted Array: [2, 5, 8, 12, 16, 23, 38, 56, 72, 91]
Target: 23

Step 1: low=0, high=9, mid=4
        [2, 5, 8, 12, |16|, 23, 38, 56, 72, 91]
                        ▲
                    arr[4]=16 < 23 → search RIGHT
                    low = mid + 1 = 5

Step 2: low=5, high=9, mid=7
        [23, 38, |56|, 72, 91]
                  ▲
              arr[7]=56 > 23 → search LEFT
              high = mid - 1 = 6

Step 3: low=5, high=6, mid=5
        [|23|, 38]
          ▲
      arr[5]=23 == 23 → FOUND at index 5! ✓
```

---

## D&C Structure of Binary Search

```
┌──────────────────────────────────────────┐
│  DIVIDE:   Compare target with arr[mid] │
│            Cost: O(1)                    │
│                                          │
│  CONQUER:  Recurse on LEFT or RIGHT half │
│            (only ONE subproblem!)         │
│            Cost: T(n/2)                  │
│                                          │
│  COMBINE:  Return the result             │
│            Cost: O(1)                    │
│                                          │
│  Recurrence: T(n) = T(n/2) + O(1)       │
│  Solution:   T(n) = O(log n)            │
└──────────────────────────────────────────┘
```

---

## Pseudocode: Recursive Binary Search

```
function binarySearch(arr, target, low, high):
    // BASE CASE: element not found
    if low > high:
        return -1
    
    // DIVIDE: find middle
    mid = low + (high - low) / 2    // avoids integer overflow!
    
    // CHECK: found?
    if arr[mid] == target:
        return mid
    
    // CONQUER: recurse on the relevant half
    if target < arr[mid]:
        return binarySearch(arr, target, low, mid - 1)
    else:
        return binarySearch(arr, target, mid + 1, high)
```

---

## Pseudocode: Iterative Binary Search

```
function binarySearch(arr, target):
    low = 0
    high = length(arr) - 1
    
    while low <= high:
        mid = low + (high - low) / 2
        
        if arr[mid] == target:
            return mid
        else if target < arr[mid]:
            high = mid - 1
        else:
            low = mid + 1
    
    return -1    // not found
```

---

## Step-by-Step Trace: Element NOT Found

```
Array: [1, 3, 5, 7, 9, 11, 13]
Target: 6

Step 1: low=0, high=6, mid=3
        [1, 3, 5, |7|, 9, 11, 13]
                   ▲
               7 > 6 → high = 2

Step 2: low=0, high=2, mid=1
        [1, |3|, 5]
             ▲
         3 < 6 → low = 2

Step 3: low=2, high=2, mid=2
        [5]
         ▲
     5 < 6 → low = 3

Step 4: low=3, high=2
        low > high → NOT FOUND! Return -1
```

---

## Visualization: Search Space Shrinking

```
n = 16 elements

Step 1: ████████████████  (16 elements)
Step 2: ████████          (8 elements)
Step 3: ████              (4 elements)  
Step 4: ██                (2 elements)
Step 5: █                 (1 element)

After log₂(16) = 4 comparisons, only 1 element remains!

General: n → n/2 → n/4 → n/8 → ... → 1
         Takes log₂(n) steps
```

---

## Why the Midpoint Formula Matters

```
DANGEROUS:  mid = (low + high) / 2
            If low + high > INT_MAX → INTEGER OVERFLOW!
            Example: low = 2,000,000,000, high = 2,000,000,000
                     sum = 4,000,000,000 > INT_MAX (2^31 - 1)

SAFE:       mid = low + (high - low) / 2
            Equivalent mathematically, but no overflow risk!
            
ALSO SAFE:  mid = (low + high) >>> 1    (unsigned right shift)
            Works in Java/C++ (language-specific)
```

---

## Key Properties

| Property | Value |
|----------|-------|
| **Prerequisite** | Array must be SORTED |
| **Best case** | O(1) — target is at the midpoint |
| **Average case** | O(log n) |
| **Worst case** | O(log n) — target not found |
| **Space (iterative)** | O(1) |
| **Space (recursive)** | O(log n) — call stack |
| **Number of comparisons** | At most ⌈log₂(n)⌉ + 1 |

---

## Binary Search as a Decision Tree

```
Array: [2, 5, 8, 12, 16, 23, 38]

                    arr[3]=12
                   /         \
              < 12            > 12
              /                  \
        arr[1]=5              arr[5]=23
        /      \              /       \
    < 5       > 5         < 23      > 23
    /            \         /            \
 arr[0]=2    arr[2]=8   arr[4]=16   arr[6]=38

Height = ⌈log₂(7)⌉ = 3
Any element found in at most 3 comparisons!

This is a BALANCED binary tree → optimal search.
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `mid = (low + high) / 2` | Integer overflow | `mid = low + (high - low) / 2` |
| `while (low < high)` | Misses single-element case | `while (low <= high)` |
| `high = mid` (with `low <= high`) | Infinite loop | `high = mid - 1` |
| Not checking if array is sorted | Wrong results | Verify or sort first |
| Off-by-one in return value | Returns wrong index | Careful boundary analysis |

---

## Real-World Applications

| Application | How Binary Search Is Used |
|-------------|--------------------------|
| Dictionary lookup | Find word alphabetically |
| Database indexing | B-tree search in databases |
| Git bisect | Find the commit that introduced a bug |
| Square root computation | Binary search on the answer |
| Load balancing | Find optimal threshold |
| IP routing | Look up routing table entries |
| Spell checking | Search sorted dictionaries |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| What | Search in sorted array by halving search space |
| D&C Pattern | Divide (compare), Conquer (ONE half), Combine (return) |
| Prerequisite | Array must be sorted |
| Time | O(log n) |
| Space | O(1) iterative, O(log n) recursive |
| Key Trick | Safe midpoint: `low + (high - low) / 2` |
| Decision Tree | Height = ⌈log₂(n)⌉ |

---

## Quick Revision Questions

1. **Why must the array be sorted for Binary Search to work?**
2. **What is the maximum number of comparisons for an array of size 1,000,000?**
3. **Why is `mid = low + (high - low) / 2` preferred over `mid = (low + high) / 2`?**
4. **Draw the decision tree for Binary Search on the array [1, 3, 5, 7, 9].**
5. **What is the recurrence relation for Binary Search and its solution?**
6. **Compare recursive and iterative Binary Search in terms of space complexity.**

---

[← Back to Unit 2](../02-Master-Theorem/06-when-it-doesnt-apply.md) | [Next: Implementation Variations →](02-implementation-variations.md)

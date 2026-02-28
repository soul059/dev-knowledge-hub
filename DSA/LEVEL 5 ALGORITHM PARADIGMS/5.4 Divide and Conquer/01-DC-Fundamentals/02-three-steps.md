# Chapter 2: Three Steps — Divide, Conquer, Combine

## Overview

Every Divide and Conquer algorithm follows a rigid three-step framework. Understanding each step deeply — what it does, how to design it, and what pitfalls to avoid — is the foundation for mastering D&C problem solving.

---

## The Three Steps in Detail

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   STEP 1: DIVIDE                                               │
│   ┌──────────────────────────────────────────────┐              │
│   │ Break the problem into smaller subproblems   │              │
│   │ • Same type as original problem              │              │
│   │ • Smaller in size                            │              │
│   │ • Ideally equal-sized parts                  │              │
│   └──────────────────────────────────────────────┘              │
│                         │                                       │
│                         ▼                                       │
│   STEP 2: CONQUER                                              │
│   ┌──────────────────────────────────────────────┐              │
│   │ Solve each subproblem recursively            │              │
│   │ • Apply the same algorithm to each part      │              │
│   │ • If subproblem is small enough → base case  │              │
│   │ • Trust the recursion!                       │              │
│   └──────────────────────────────────────────────┘              │
│                         │                                       │
│                         ▼                                       │
│   STEP 3: COMBINE                                              │
│   ┌──────────────────────────────────────────────┐              │
│   │ Merge subproblem solutions into final answer │              │
│   │ • This is often the HARDEST step             │              │
│   │ • Determines overall time complexity         │              │
│   │ • Must produce correct result for original   │              │
│   └──────────────────────────────────────────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Step 1: DIVIDE — Breaking the Problem

### What happens during DIVIDE?
The original problem of size `n` is split into **`a` subproblems**, each of size approximately `n/b`.

### Common Division Strategies

```
HALVING (Most Common):
┌──────────────────────────┐
│  [1, 5, 3, 8, 2, 7, 4]  │
└────────────┬─────────────┘
      ┌──────┴──────┐
      ▼              ▼
 [1, 5, 3, 8]    [2, 7, 4]      ← Split at midpoint
 (size n/2)       (size n/2)

THIRDS:
┌──────────────────────────┐
│  [1, 5, 3, 8, 2, 7, 4, 9, 6]  │
└──────────┬──────────────────┘
    ┌──────┼──────┐
    ▼      ▼      ▼
 [1,5,3] [8,2,7] [4,9,6]        ← Split into three parts

PIVOT-BASED (Quick Sort):
┌──────────────────────────┐
│  [3, 7, 1, 5, 2, 8, 4]  │   pivot = 4
└────────────┬─────────────┘
      ┌──────┴──────┐
      ▼              ▼
 [3, 1, 2]  [4]  [7, 5, 8]      ← Elements < pivot | pivot | > pivot
 (variable)       (variable)      Unequal sizes!
```

### Key Design Decisions for DIVIDE

| Decision | Impact |
|----------|--------|
| Split into how many parts? | Affects recurrence: `a` in T(n) = aT(n/b) + f(n) |
| Equal or unequal splits? | Equal → predictable; Unequal → may lead to worst cases |
| Cost of splitting? | Should be O(n) or less; expensive splits hurt performance |
| Where to split? | Midpoint (Merge Sort) vs Pivot (Quick Sort) |

---

## Step 2: CONQUER — Solving Subproblems

### The Recursive Faith

The key to CONQUER is **trusting the recursion**:

```
┌─────────────────────────────────────────────────┐
│  "Assume your recursive function works          │
│   correctly for all inputs smaller than n.      │
│   Then show it works for input of size n."      │
│                                                 │
│   This is the INDUCTIVE HYPOTHESIS of D&C.      │
└─────────────────────────────────────────────────┘
```

### Base Cases — When to Stop Recursing

```
                    Recursion
                       │
                       ▼
              Is size ≤ threshold?
              /                  \
           YES                    NO
            │                      │
            ▼                      ▼
     ┌─────────────┐      ┌──────────────┐
     │  SOLVE      │      │  DIVIDE      │
     │  DIRECTLY   │      │  further &   │
     │  (Base case)│      │  RECURSE     │
     └─────────────┘      └──────────────┘
```

**Common base cases:**

| Algorithm | Base Case |
|-----------|-----------|
| Merge Sort | Array of size 1 (already sorted) |
| Binary Search | Single element or empty range |
| Quick Sort | Array of size 0 or 1 |
| Strassen's | Small matrix (use naive multiplication) |
| Closest Pair | 2 or 3 points (compare directly) |

### Base Case Pitfalls

```
BAD: Missing base case
  → Infinite recursion → Stack overflow!

BAD: Wrong base case
  → Incorrect results for small inputs

BAD: Base case too large
  → Missed optimization opportunities

GOOD: Base case at smallest meaningful size
  → Clean, correct termination
```

---

## Step 3: COMBINE — Merging Solutions

### Why COMBINE is Often the Hardest Step

The combine step is where the "magic" happens. It's the step that makes D&C work, and it's typically the most algorithmically challenging part to design.

```
MERGE SORT — Combine Step:
┌───────────────────────────────────────────┐
│ Merge two sorted halves into one sorted   │
│                                           │
│ Left:  [1, 3, 5, 8]                      │
│ Right: [2, 4, 7]                          │
│                                           │
│ Compare front elements, pick smaller:     │
│ Step 1: 1 < 2  → pick 1  → [1]           │
│ Step 2: 3 > 2  → pick 2  → [1,2]         │
│ Step 3: 3 < 4  → pick 3  → [1,2,3]       │
│ Step 4: 5 > 4  → pick 4  → [1,2,3,4]     │
│ Step 5: 5 < 7  → pick 5  → [1,2,3,4,5]   │
│ Step 6: 8 > 7  → pick 7  → [1,2,3,4,5,7] │
│ Step 7: only 8 → pick 8  → [1,2,3,4,5,7,8]│
│                                           │
│ Cost: O(n)                                │
└───────────────────────────────────────────┘

BINARY SEARCH — Combine Step:
┌───────────────────────────────────────────┐
│ Simply return the result from the          │
│ subproblem that was explored.              │
│                                           │
│ Cost: O(1)  ← Trivial combine!            │
└───────────────────────────────────────────┘

CLOSEST PAIR — Combine Step:
┌───────────────────────────────────────────┐
│ Check if any pair spanning the dividing   │
│ line is closer than the minimum found     │
│ in each half.                             │
│                                           │
│ Requires careful "strip" analysis.        │
│ Cost: O(n log n) or O(n) with sorting     │
└───────────────────────────────────────────┘
```

---

## Putting It All Together: Complete Example — Sum of Array

### Problem: Find the sum of all elements in an array using D&C.

```
function dcSum(arr, low, high):
    // BASE CASE
    if low == high:
        return arr[low]
    
    // DIVIDE
    mid = (low + high) / 2
    
    // CONQUER
    leftSum  = dcSum(arr, low, mid)
    rightSum = dcSum(arr, mid + 1, high)
    
    // COMBINE
    return leftSum + rightSum
```

### Step-by-Step Trace

```
Array: [4, 2, 7, 1, 3]

dcSum([4, 2, 7, 1, 3], 0, 4)
│
├── DIVIDE: mid = 2
│
├── dcSum([4, 2, 7], 0, 2)          ← left subproblem
│   ├── DIVIDE: mid = 1
│   ├── dcSum([4, 2], 0, 1)
│   │   ├── DIVIDE: mid = 0
│   │   ├── dcSum([4], 0, 0) → 4    ← base case
│   │   ├── dcSum([2], 1, 1) → 2    ← base case
│   │   └── COMBINE: 4 + 2 = 6
│   ├── dcSum([7], 2, 2) → 7        ← base case
│   └── COMBINE: 6 + 7 = 13
│
├── dcSum([1, 3], 3, 4)              ← right subproblem
│   ├── DIVIDE: mid = 3
│   ├── dcSum([1], 3, 3) → 1        ← base case
│   ├── dcSum([3], 4, 4) → 3        ← base case
│   └── COMBINE: 1 + 3 = 4
│
└── COMBINE: 13 + 4 = 17  ✓ (4+2+7+1+3 = 17)
```

---

## The Cost of Each Step

The total running time of a D&C algorithm is:

```
T(n) = [Cost of DIVIDE] + [Cost of CONQUER] + [Cost of COMBINE]

     = D(n)  +  a · T(n/b)  +  C(n)

Where:
  a = number of subproblems
  b = factor by which size shrinks
  D(n) = time to divide
  C(n) = time to combine
```

### Examples

| Algorithm | D(n) | a | b | C(n) | Recurrence | Result |
|-----------|------|---|---|------|------------|--------|
| Binary Search | O(1) | 1 | 2 | O(1) | T(n) = T(n/2) + O(1) | O(log n) |
| Merge Sort | O(1) | 2 | 2 | O(n) | T(n) = 2T(n/2) + O(n) | O(n log n) |
| Quick Sort (avg) | O(n) | 2 | 2 | O(1) | T(n) = 2T(n/2) + O(n) | O(n log n) |
| Strassen's | O(n²) | 7 | 2 | O(n²) | T(n) = 7T(n/2) + O(n²) | O(n^2.807) |

---

## Common Patterns in the Three Steps

### Pattern 1: Heavy Divide, Light Combine (Quick Sort)
```
Work is front-loaded:
  DIVIDE:  O(n) partition ← Most of the work here
  CONQUER: Recursive calls
  COMBINE: O(1)           ← Nothing to do!
```

### Pattern 2: Light Divide, Heavy Combine (Merge Sort)
```
Work is back-loaded:
  DIVIDE:  O(1) find midpoint ← Trivial
  CONQUER: Recursive calls
  COMBINE: O(n) merge         ← Most of the work here
```

### Pattern 3: Light Divide, Light Combine (Binary Search)
```
Minimal overhead:
  DIVIDE:  O(1) compute mid ← Trivial
  CONQUER: ONE recursive call (not two!)
  COMBINE: O(1) return      ← Trivial
```

---

## Designing Your Own D&C Algorithm

```
Step-by-step methodology:

1. IDENTIFY:  Can this problem be broken into smaller
              versions of itself?

2. DEFINE:    What is the base case?
              (Smallest input you can solve directly)

3. DIVIDE:    How do you split the problem?
              (Halving? Pivot? Other?)

4. CONQUER:   Trust that recursion solves each subproblem.
              (Don't think about HOW — just trust it)

5. COMBINE:   How do you merge sub-solutions into the
              final solution? (This is the creative step!)

6. VERIFY:    Does the base case + combine step produce
              correct results for all inputs?

7. ANALYZE:   Write the recurrence relation.
              Apply Master Theorem.
```

---

## Summary Table

| Aspect | Divide | Conquer | Combine |
|--------|--------|---------|---------|
| **What** | Split problem | Solve subproblems | Merge results |
| **How** | Find midpoint, partition, etc. | Recursive calls | Algorithm-specific merge |
| **Typical Cost** | O(1) to O(n) | Recursive | O(1) to O(n) |
| **Key Challenge** | Balanced splits | Correct base case | Correctness & efficiency |
| **Creativity** | Moderate | Low (trust recursion) | HIGH |

---

## Quick Revision Questions

1. **Which step (Divide, Conquer, or Combine) is typically the most challenging to design? Why?**
2. **In Merge Sort, which step does O(n) work? What about in Quick Sort?**
3. **What happens if the base case is missing or incorrect?**
4. **Write the generic recurrence relation for a D&C algorithm that divides into `a` subproblems of size `n/b` with `f(n)` combine cost.**
5. **Compare the three-step pattern of Binary Search vs Merge Sort. How do they differ?**
6. **Why is it important that subproblems are of roughly equal size?**

---

[← Previous: What is D&C?](01-what-is-divide-and-conquer.md) | [Next: When to Use D&C →](03-when-to-use-dc.md)

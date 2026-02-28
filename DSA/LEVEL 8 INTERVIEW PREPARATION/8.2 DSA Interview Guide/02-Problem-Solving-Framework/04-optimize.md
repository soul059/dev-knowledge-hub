# Chapter 2.4: Optimize

```
╔══════════════════════════════════════════════════════════╗
║                   OPTIMIZE                              ║
║   The art of making your solution faster and smarter    ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Optimization is where interviews are won or lost. Going from brute force to optimal shows the interviewer your **depth of understanding**, pattern recognition, and algorithmic maturity. This chapter teaches systematic approaches to optimize any solution.

---

## The Optimization Toolkit

```
┌─────────────────────────────────────────────────────────────┐
│              10 OPTIMIZATION STRATEGIES                     │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  1. Hash Map         → O(n) lookups instead of O(n) │   │
│  │  2. Two Pointers     → Eliminate inner loop          │   │
│  │  3. Sliding Window   → Avoid recalculating subarrays │   │
│  │  4. Binary Search    → O(log n) instead of O(n)      │   │
│  │  5. Sorting First    → Enable two pointers / search  │   │
│  │  6. Prefix Sum       → O(1) range sum queries        │   │
│  │  7. Memoization/DP   → Avoid repeated subproblems    │   │
│  │  8. Priority Queue   → Efficient min/max extraction  │   │
│  │  9. Greedy           → Local optimal = global optimal │  │
│  │ 10. Space-Time Trade → Use more space to save time   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Strategy 1: Hash Map — Replace Search with Lookup

```
PATTERN: Inner loop is SEARCHING for something

BEFORE (O(n²)):                    AFTER (O(n)):
┌──────────────────────┐          ┌──────────────────────┐
│ for i in range(n):   │          │ seen = {}            │
│   for j in range(n): │  ──────► │ for i in range(n):   │
│     if match(i, j):  │          │   if complement in   │
│       return result  │          │            seen:     │
│                      │          │     return result    │
│                      │          │   seen[nums[i]] = i  │
└──────────────────────┘          └──────────────────────┘

EXAMPLE: Two Sum
nums = [2, 7, 11, 15], target = 9

Brute: Check all pairs
i=0: (2,7)✓  → O(n²)

Hash Map: For each num, look for (target - num)
seen = {}
i=0: need 7, not in seen → seen = {2:0}
i=1: need 2, found! → return [0,1]  → O(n)
```

---

## Strategy 2: Two Pointers — Collapse Nested Loops

```
PATTERN: Searching pairs in SORTED data

BEFORE (O(n²)):                    AFTER (O(n)):
┌──────────────────────┐          ┌────────────────────────┐
│ for i in range(n):   │          │ left = 0               │
│   for j in range(n): │  ──────► │ right = n - 1          │
│     check(i, j)      │          │ while left < right:    │
│                      │          │   sum = arr[l] + arr[r]│
│                      │          │   if sum == target: ✓  │
│                      │          │   elif sum < target:   │
│                      │          │     left += 1          │
│                      │          │   else: right -= 1     │
└──────────────────────┘          └────────────────────────┘

VISUAL TRACE: arr = [1, 3, 5, 7, 9], target = 8

Step 1: [1, 3, 5, 7, 9]    1+9=10 > 8 → move right
         L              R

Step 2: [1, 3, 5, 7, 9]    1+7=8 = 8 → FOUND!
         L           R
```

---

## Strategy 3: Sliding Window — Avoid Redundant Computation

```
PATTERN: Finding subarrays/substrings with certain properties

BEFORE (O(n²)):                    AFTER (O(n)):
┌───────────────────────┐         ┌─────────────────────────┐
│ for i in range(n):    │         │ left = 0                │
│   for j in range(i,n):│ ──────► │ for right in range(n):  │
│     check(i..j)       │         │   add arr[right] to win │
│                       │         │   while window invalid: │
│                       │         │     remove arr[left]    │
│                       │         │     left += 1           │
│                       │         │   update answer         │
└───────────────────────┘         └─────────────────────────┘

VISUAL: Max sum subarray of size 3
arr = [1, 4, 2, 10, 2, 3, 1, 0, 20]

Window slides:
[1, 4, 2] = 7
   [4, 2, 10] = 16    ← Instead of recalculating,
      [2, 10, 2] = 14    subtract left, add right:
         [10, 2, 3] = 15  7 - 1 + 10 = 16 ✓
```

---

## Strategy 4: Binary Search — Divide Search Space

```
PATTERN: Searching in SORTED data or finding a boundary

BEFORE (O(n)):                     AFTER (O(log n)):
┌──────────────────────┐          ┌──────────────────────┐
│ for i in range(n):   │          │ lo, hi = 0, n-1      │
│   if arr[i] == target│  ──────► │ while lo <= hi:      │
│     return i         │          │   mid = (lo+hi) // 2 │
│                      │          │   if arr[mid]==target:│
│                      │          │     return mid        │
│                      │          │   elif arr[mid]<target│
│                      │          │     lo = mid + 1      │
│                      │          │   else:               │
│                      │          │     hi = mid - 1      │
└──────────────────────┘          └──────────────────────┘

SEARCH SPACE REDUCTION:
n=1000000 elements

Linear: 1,000,000 comparisons (worst case)
Binary: log₂(1,000,000) ≈ 20 comparisons !!

Step by step:
1000000 → 500000 → 250000 → 125000 → ... → 1
         20 steps total
```

---

## Strategy 5: Prefix Sum — O(1) Range Queries

```
PATTERN: Repeated sum/count queries over ranges

BEFORE (O(n) per query):          AFTER (O(1) per query):
┌──────────────────────┐          ┌──────────────────────────┐
│ sum(arr[i..j])       │          │ Build prefix once O(n):  │
│ = loop from i to j   │  ──────► │ prefix[i] = sum(arr[0..i])│
│ = O(n) per query     │          │                          │
│                      │          │ Query: O(1)              │
│                      │          │ sum(i..j) =              │
│                      │          │   prefix[j] - prefix[i-1]│
└──────────────────────┘          └──────────────────────────┘

EXAMPLE:
arr =    [3, 1, 4, 1, 5, 9, 2, 6]
prefix = [3, 4, 8, 9, 14, 23, 25, 31]

Sum of arr[2..5] = prefix[5] - prefix[1] = 23 - 4 = 19
Verify: 4 + 1 + 5 + 9 = 19 ✓
```

---

## Strategy 6: Memoization/DP — Cache Repeated Work

```
PATTERN: Recursion with OVERLAPPING subproblems

BEFORE (Exponential):             AFTER (Polynomial):
┌──────────────────────┐          ┌──────────────────────┐
│ def fib(n):          │          │ memo = {}            │
│   if n <= 1:         │          │ def fib(n):          │
│     return n         │  ──────► │   if n in memo:      │
│   return fib(n-1)    │          │     return memo[n]   │
│        + fib(n-2)    │          │   if n <= 1:         │
│                      │          │     return n         │
│ Time: O(2ⁿ)         │          │   memo[n] = fib(n-1) │
│                      │          │           + fib(n-2) │
│                      │          │   return memo[n]     │
│                      │          │                      │
│                      │          │ Time: O(n)           │
└──────────────────────┘          └──────────────────────┘

CALL TREE WITHOUT MEMO (fib(5)):
                    fib(5)
                   /      \
              fib(4)      fib(3)     ← fib(3) computed TWICE
             /    \       /    \
         fib(3) fib(2) fib(2) fib(1) ← fib(2) computed 3 TIMES
        /   \
    fib(2) fib(1)

WITH MEMO: Each fib(k) computed ONCE → O(n) total
```

---

## The Optimization Decision Tree

```
┌─────────────────────────────────────────────────────────────┐
│  "MY BRUTE FORCE IS O(n²). HOW DO I OPTIMIZE?"            │
│                                                             │
│                    What's the bottleneck?                    │
│                    ╱              ╲                          │
│           Inner loop is       Inner loop is                 │
│           SEARCHING           CALCULATING                   │
│           ╱        ╲          ╱          ╲                   │
│     Data sorted?  No     Overlapping    Range                │
│     ╱         ╲          subprobs?      queries?             │
│   Yes         No            ╱  ╲        ╱                   │
│    │          │           Yes   No     │                    │
│    ▼          ▼            │    │      ▼                    │
│ Two Ptr/   Hash Map       │    │   Prefix Sum              │
│ Bin Srch               DP/Memo  │                           │
│                              ▼                              │
│                          Greedy?                            │
│                          Sliding                            │
│                          Window?                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Real Example: Full Optimization Flow

```
PROBLEM: "Find the length of the longest substring 
          without repeating characters"

STEP 1: BRUTE FORCE O(n³)
┌─────────────────────────────────────────────────────────┐
│  for i in range(n):                                     │
│      for j in range(i+1, n+1):                         │
│          substr = s[i:j]                                │
│          if all_unique(substr):   ← O(n) check         │
│              max_len = max(max_len, j - i)             │
│                                                         │
│  Three nested operations → O(n³)                       │
└─────────────────────────────────────────────────────────┘

STEP 2: OPTIMIZE with SET → O(n²)
┌─────────────────────────────────────────────────────────┐
│  for i in range(n):                                     │
│      seen = set()                                       │
│      for j in range(i, n):                             │
│          if s[j] in seen: break  ← O(1) check!        │
│          seen.add(s[j])                                 │
│          max_len = max(max_len, j - i + 1)             │
│                                                         │
│  Two loops → O(n²) — better!                           │
└─────────────────────────────────────────────────────────┘

STEP 3: OPTIMIZE with SLIDING WINDOW → O(n)
┌─────────────────────────────────────────────────────────┐
│  char_index = {}                                        │
│  left = 0                                               │
│  max_len = 0                                            │
│                                                         │
│  for right in range(n):                                 │
│      if s[right] in char_index:                        │
│          left = max(left, char_index[s[right]] + 1)    │
│      char_index[s[right]] = right                      │
│      max_len = max(max_len, right - left + 1)          │
│                                                         │
│  One pass → O(n) — optimal!                            │
└─────────────────────────────────────────────────────────┘

TRACE: s = "abcabcbb"
┌──────────────────────────────────────────────────────────┐
│ right=0: 'a' char_index={a:0}  left=0  len=1           │
│ right=1: 'b' char_index={a:0,b:1}  left=0  len=2      │
│ right=2: 'c' char_index={a:0,b:1,c:2}  left=0  len=3  │
│ right=3: 'a' dup! left=max(0,0+1)=1  len=3             │
│ right=4: 'b' dup! left=max(1,1+1)=2  len=3             │
│ right=5: 'c' dup! left=max(2,2+1)=3  len=3             │
│ right=6: 'b' dup! left=max(3,4+1)=5  len=2             │
│ right=7: 'b' dup! left=max(5,6+1)=7  len=1             │
│                                                          │
│ Answer: 3                                                │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Optimization | When to Use | Complexity Change | Key Insight |
|-------------|------------|:-:|---------|
| Hash Map | Search in inner loop | O(n²) → O(n) | O(1) lookup replaces O(n) search |
| Two Pointers | Pairs in sorted data | O(n²) → O(n) | One pointer moves per comparison |
| Sliding Window | Contiguous subarray/substring | O(n²) → O(n) | Window slides, avoids recomputation |
| Binary Search | Sorted data, find boundary | O(n) → O(log n) | Halve search space each step |
| Prefix Sum | Range sum queries | O(n) per query → O(1) | Precompute cumulative sums |
| Memoization/DP | Overlapping subproblems | Exponential → Polynomial | Cache results of subproblems |
| Sorting First | Enables other optimizations | O(n²) → O(n log n) | Sorted data unlocks two pointers, binary search |
| Greedy | Local optimal = global optimal | O(2ⁿ) → O(n log n) | Remove trying all combinations |
| Heap/PQ | Repeated min/max extraction | O(n) → O(log n) per op | Efficient priority management |

---

## Quick Revision Questions

1. **What's the first question to ask when optimizing a brute force?**
   - "What is the inner loop really doing?" — identify if it's searching, calculating, or checking something that can be done more efficiently

2. **When should you use a Hash Map vs Two Pointers?**
   - Hash Map: unsorted data, need O(1) lookups; Two Pointers: sorted data, searching for pairs

3. **How does Sliding Window optimize subarray problems?**
   - Instead of recalculating the entire subarray, it incrementally adds the new element and removes the old one as the window slides

4. **What's the key insight for applying Dynamic Programming?**
   - The problem has overlapping subproblems — the same computation happens multiple times in the recursive tree

5. **How does Prefix Sum achieve O(1) range queries?**
   - By precomputing cumulative sums, any range sum becomes a simple subtraction: sum(i..j) = prefix[j] - prefix[i-1]

6. **What optimization should you consider when data can be sorted?**
   - Sorting enables Two Pointers (O(n)), Binary Search (O(log n)), and removes the need for hash maps in many problems

---

[← Previous: Brute Force First](03-brute-force-first.md) | [Next: Code the Solution →](05-code-the-solution.md)

---
[← Back to README](../README.md)

# Chapter 6: Special Permutation — Bitmask DP on Permutations

## 📋 Overview
Count or find permutations satisfying adjacency constraints (e.g., consecutive elements must be divisible). Bitmask DP on permutations runs in $O(n^2 \cdot 2^n)$ and handles constraints that brute force cannot.

---

## 🧠 Core Idea

### Problem Template
```
Given an array nums of distinct positive integers, count
permutations where for every adjacent pair (nums[i], nums[i+1]):
  nums[i] % nums[i+1] == 0  OR  nums[i+1] % nums[i] == 0

This is a "special permutation" — adjacency constraint
makes standard permutation counting fail.
```

### State Definition
```
dp[mask][last] = number of special permutations of the
                 elements in 'mask', ending with element 'last'

Transition:
  For each element 'next' NOT in mask:
    if valid_pair(nums[last], nums[next]):
      dp[mask | (1<<next)][next] += dp[mask][last]

Base case:
  dp[1 << i][i] = 1  for each i
  (single element is a valid permutation)

Answer:
  Σ dp[FULL][i] for all i
```

---

## 🔍 Step-by-Step Trace

```
nums = [2, 3, 6]
Constraint: a % b == 0 or b % a == 0

Valid pairs:
  (2,3): 2%3≠0, 3%2≠0 → ✗
  (2,6): 6%2=0 → ✓
  (3,6): 6%3=0 → ✓

Base cases:
  dp[001][0] = 1  (just [2])
  dp[010][1] = 1  (just [3])
  dp[100][2] = 1  (just [6])

2-element:
  From dp[001][0]: try 1 → (2,3)✗, try 2 → (2,6)✓
    dp[101][2] += 1  → dp[101][2] = 1  perm: [2,6]
    
  From dp[010][1]: try 0 → (3,2)✗, try 2 → (3,6)✓
    dp[110][2] += 1  → dp[110][2] = 1  perm: [3,6]
    
  From dp[100][2]: try 0 → (6,2)✓, try 1 → (6,3)✓
    dp[101][0] += 1  → dp[101][0] = 1  perm: [6,2]
    dp[110][1] += 1  → dp[110][1] = 1  perm: [6,3]

3-element (full mask = 111):
  From dp[101][2]=1: try 1 → (6,3)✓
    dp[111][1] += 1  → dp[111][1] = 1  perm: [2,6,3]
    
  From dp[101][0]=1: try 1 → (2,3)✗
    no valid extension
    
  From dp[110][2]=1: try 0 → (6,2)✓
    dp[111][0] += 1  → dp[111][0] = 1  perm: [3,6,2]
    
  From dp[110][1]=1: try 0 → (3,2)✗
    no valid extension

Answer = dp[111][0] + dp[111][1] + dp[111][2]
       = 1 + 1 + 0 = 2

Permutations: [2,6,3] and [3,6,2]  ✓
```

---

## 💻 Pseudocode

```
function countSpecialPermutations(nums):
    n = len(nums)
    FULL = (1 << n) - 1
    
    // Precompute valid adjacency pairs
    valid = 2D array [n][n] = false
    for i = 0 to n-1:
        for j = 0 to n-1:
            if i == j: continue
            if nums[i] % nums[j] == 0 or nums[j] % nums[i] == 0:
                valid[i][j] = true
    
    // DP
    dp = 2D array [FULL+1][n] = 0
    
    // Base: single elements
    for i = 0 to n-1:
        dp[1 << i][i] = 1
    
    // Fill
    for mask = 1 to FULL:
        for last = 0 to n-1:
            if dp[mask][last] == 0: continue
            if not (mask & (1 << last)): continue
            
            for next = 0 to n-1:
                if mask & (1 << next): continue
                if valid[last][next]:
                    dp[mask | (1 << next)][next] += dp[mask][last]
    
    // Sum over all ending elements
    answer = 0
    for i = 0 to n-1:
        answer += dp[FULL][i]
    return answer
```

---

## 🌐 More Permutation DP Problems

### Beautiful Arrangement
```
Permutation of 1..n where for position i:
  perm[i] % i == 0  OR  i % perm[i] == 0

dp[mask] = number of valid arrangements for first
           popcount(mask) positions using elements in mask

Transition:
  pos = popcount(mask) + 1
  for each unused element val:
    if val % pos == 0 or pos % val == 0:
      dp[mask | (1 << val)] += dp[mask]
```

### Assignment Problem (Hungarian)
```
n workers, n jobs, cost[i][j]
Assign each worker to exactly one job, minimize total cost.

dp[mask] = min cost to assign first popcount(mask)
           workers to the jobs in mask

worker = popcount(mask)  // next worker to assign
for each job j in mask:
  dp[mask] = min(dp[mask], dp[mask^(1<<j)] + cost[worker-1][j])
```

### Minimum XOR Sum of Two Arrays
```
Rearrange one array to minimize Σ (a[i] XOR b[perm[i]])

dp[mask] = min XOR sum assigning first popcount(mask)
           elements of A to jobs in mask from B

i = popcount(mask)   // current element in A
for j in mask:
  dp[mask] = min(dp[mask], dp[mask^(1<<j)] + (A[i-1] XOR B[j]))
```

---

## 🔧 Optimization Techniques

```
1. Precompute popcount:
   popcount[mask] = __builtin_popcount(mask)
   Use current position = popcount(mask)

2. Compatible neighbor bitmask:
   compatible[i] = bitmask of elements that can follow i
   Skip inner loop when (compatible[last] & ~mask) == 0

3. Process masks in order of popcount:
   Ensures all sub-states computed before use
   (Natural when iterating mask = 0 to 2ⁿ-1)

4. Modular arithmetic:
   For counting problems, take mod at each addition
   dp[...] = (dp[...] + dp[...]) % MOD
```

---

## ⚡ Complexity Analysis

```
┌────────────────────────────────────────┐
│ Time:  O(n² · 2ⁿ)                     │
│   2ⁿ masks × n last × n next          │
│                                        │
│ Space: O(n · 2ⁿ)                      │
│   dp table                             │
│                                        │
│ For position-based (Beautiful Arr.):   │
│   O(n · 2ⁿ) — position = popcount     │
│   No 'last' dimension needed           │
│                                        │
│ Feasible: n ≤ 20 (n²·2²⁰ ≈ 4·10⁸)   │
│                                        │
│ vs Brute force:                        │
│   n=14: 14! ≈ 87 billion              │
│         14²·2¹⁴ ≈ 3.2 million         │
└────────────────────────────────────────┘
```

---

## 📊 Bitmask Permutation DP Decision Guide

```
┌──────────────────────────────────┐
│ Adjacency constraint?            │
│  YES → dp[mask][last]            │
│        need 'last' for next      │
│  NO  → dp[mask]                  │
│        position = popcount(mask) │
├──────────────────────────────────┤
│ Minimize/Maximize?               │
│  → Use min/max in dp             │
│ Count?                           │
│  → Use addition (with MOD)       │
├──────────────────────────────────┤
│ n ≤ 14: comfortable              │
│ n ≤ 20: possible with care       │
│ n > 20: need different approach   │
└──────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Count/optimize permutations with constraints |
| **State** | dp[mask][last] for adjacency constraints |
| **State (alt)** | dp[mask] when position = popcount(mask) |
| **Transition** | Add valid next element to permutation |
| **Complexity** | O(n² · 2ⁿ) adjacency, O(n · 2ⁿ) position |
| **Limit** | n ≤ 20 |

---

## ❓ Quick Revision Questions

1. **When do you need dp[mask][last] vs dp[mask]?**
2. **How do you determine which position you're filling from the mask?**
3. **How do you precompute valid adjacency pairs?**
4. **What is the answer extraction for counting permutations?**
5. **How does Beautiful Arrangement differ from Special Permutation in DP state?**
6. **What optimization can you do by precomputing compatible neighbor bitmasks?**

---

[← Previous: Parallel Courses](05-parallel-courses.md) | [Next Unit: DP Optimization →](../13-DP-Optimization/01-space-optimization.md)

[← Back to README](../README.md)

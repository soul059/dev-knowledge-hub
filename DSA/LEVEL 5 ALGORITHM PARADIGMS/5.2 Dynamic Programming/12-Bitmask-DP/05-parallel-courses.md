# Chapter 5: Parallel Courses — Bitmask DP with Prerequisites

## 📋 Overview
Given courses with prerequisite constraints and a limit on courses per semester, find the **minimum semesters** to complete all courses. Bitmask DP models which courses are completed and determines valid semester plans.

---

## 🧠 Core Idea

### Problem Statement
```
n courses (n ≤ 15), prerequisite edges, at most k courses per semester.
Each semester: choose a subset of at most k courses whose
prerequisites are ALL already completed.
Minimize number of semesters.

Example: n=4, k=2
  Prerequisites: 2→1, 3→1, 4→2, 4→3
  
  Semester 1: {1}
  Semester 2: {2, 3}    (prereq 1 done)
  Semester 3: {4}       (prereqs 2,3 done)
  Answer: 3
```

### Precompute Prerequisites as Bitmasks
```
For each course i, store its prerequisites as a bitmask:

prereq[i] = bitmask of all prerequisites of course i

Course 1: prereq = 0000 (none)
Course 2: prereq = 0001 (course 1)
Course 3: prereq = 0001 (course 1)
Course 4: prereq = 0110 (courses 2,3)

A course i is available when:
  (prereq[i] & completed_mask) == prereq[i]
  i.e., all prerequisites are in completed set
```

### State Definition
```
dp[mask] = minimum semesters to complete exactly
           the courses in 'mask'

Transition:
  From state mask, find all available courses → 'available'
  Choose a subset S of available with |S| ≤ k
  dp[mask | S] = min(dp[mask | S], dp[mask] + 1)
```

---

## 🔍 Step-by-Step Trace

```
n=4, k=2
prereq: [0000, 0001, 0001, 0110]  (courses 1-4, 0-indexed)

dp[0000] = 0

Available at mask=0000:
  Course 0: prereq=0000, (0000 & 0000)==0000 ✓
  Course 1: prereq=0001, (0000 & 0001)==0000 ≠ 0001 ✗
  Course 2: prereq=0001, ✗
  Course 3: prereq=0110, ✗
  Available = {0} → bitmask 0001

Subsets of {0} with size ≤ 2: {0}
  dp[0001] = min(INF, 0+1) = 1

Available at mask=0001:
  Course 0: already done
  Course 1: prereq=0001, (0001 & 0001)==0001 ✓
  Course 2: prereq=0001, (0001 & 0001)==0001 ✓
  Course 3: prereq=0110, (0001 & 0110)==0000 ✗
  Available = {1, 2} → bitmask 0110

Subsets of {1,2} with |S| ≤ 2:
  {1}: dp[0011] = min(INF, 1+1) = 2
  {2}: dp[0101] = min(INF, 1+1) = 2
  {1,2}: dp[0111] = min(INF, 1+1) = 2

At mask=0111:
  Course 3: prereq=0110, (0111 & 0110)==0110 ✓
  dp[1111] = min(INF, 2+1) = 3

Answer: dp[1111] = 3
```

---

## 💻 Pseudocode

```
function minSemesters(n, k, prerequisites):
    FULL = (1 << n) - 1
    
    // Precompute prerequisite masks
    prereq = array[n] = 0
    for each (u, v) in prerequisites:
        prereq[v] |= (1 << u)   // u is prereq of v
    
    // dp[mask] = min semesters
    dp = array [FULL+1] = INF
    dp[0] = 0
    
    for mask = 0 to FULL - 1:
        if dp[mask] == INF: continue
        
        // Find available courses
        available = 0
        for i = 0 to n-1:
            if mask & (1 << i): continue     // already done
            if (mask & prereq[i]) == prereq[i]:  // prereqs met
                available |= (1 << i)
        
        // Enumerate all subsets of 'available' with size ≤ k
        sub = available
        while sub > 0:
            if popcount(sub) <= k:
                dp[mask | sub] = min(dp[mask | sub], dp[mask] + 1)
            sub = (sub - 1) & available
    
    return dp[FULL]
```

---

## 🔧 Subset Enumeration Trick

```
Enumerate all subsets of a bitmask 'available':

sub = available
while sub > 0:
    process(sub)
    sub = (sub - 1) & available

This enumerates subsets in decreasing order.

Example: available = 1010
  sub = 1010 → 1000 → 0010 → stop

Why (sub - 1) & available works:
  sub - 1 flips the lowest set bit and sets all lower bits
  & available keeps only bits in the original set
  
Total iterations: 2^(popcount(available))
```

---

## ⚡ Complexity Analysis

```
┌─────────────────────────────────────────────────┐
│ For each mask: enumerate subsets of available    │
│                                                  │
│ Total subset enumerations across all masks:      │
│   Σ over all masks: 2^(popcount(available))      │
│   ≤ 3ⁿ (each element: done, available, blocked) │
│                                                  │
│ Time:  O(3ⁿ)                                    │
│ Space: O(2ⁿ)                                    │
│                                                  │
│ n ≤ 15: 3¹⁵ ≈ 14 million  ✓ feasible            │
│ n ≤ 20: 3²⁰ ≈ 3.5 billion ✗ too slow            │
│                                                  │
│ Optimization: only enumerate subsets of size ≤ k │
│ Skip subsets with popcount > k                   │
└─────────────────────────────────────────────────┘
```

---

## 🧩 Related Problems

### Course Schedule (Topological Sort)
```
No limit on courses per semester
→ BFS level = semester number
→ Answer = longest path in DAG
```

### Course Schedule with Groups
```
Courses in same group can't be in same semester
→ Additional constraint on subset selection
```

### Minimum Cost to Complete Tasks
```
Tasks with prerequisites + costs
dp[mask] = minimum cost to complete courses in mask
→ Weighted version of same approach
```

---

## 🔄 Optimization: Precompute Valid Semester Plans

```
Instead of enumerating subsets per mask:
1. Precompute all valid subsets of size ≤ k
2. For each mask, filter valid subsets

// Precompute valid subsets (no prerequisite conflicts)
valid_semesters = []
for sub = 1 to FULL:
    if popcount(sub) > k: continue
    if is_valid_semester(sub): valid_semesters.append(sub)

function is_valid_semester(sub):
    for each course i in sub:
        prereq[i] must be subset of some completed set
    // This check happens during DP, not precomputation
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Min semesters with k limit and prerequisites |
| **State** | dp[mask] = min semesters for completed = mask |
| **Prereq encoding** | Bitmask per course of required courses |
| **Available** | Courses whose prereqs ⊆ completed mask |
| **Subset enumeration** | (sub - 1) & available trick |
| **Complexity** | O(3ⁿ) time, O(2ⁿ) space |

---

## ❓ Quick Revision Questions

1. **How do you encode prerequisites as bitmasks?**
2. **How do you check if all prerequisites for course i are met?**
3. **Explain the `(sub-1) & available` subset enumeration trick.**
4. **Why is the total complexity O(3ⁿ) instead of O(4ⁿ)?**
5. **How does this differ from standard topological sort course scheduling?**
6. **What is the maximum n for which this approach is feasible?**

---

[← Previous: Partition K Subsets](04-partition-k-subsets.md) | [Next: Special Permutation →](06-special-permutation.md)

[← Back to README](../README.md)

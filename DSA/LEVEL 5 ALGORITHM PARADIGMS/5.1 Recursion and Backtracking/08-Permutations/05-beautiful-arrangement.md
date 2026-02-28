# Chapter 5: Beautiful Arrangement

## Overview
The Beautiful Arrangement problem asks: **how many permutations of [1..n] satisfy a divisibility constraint** at every position? This is a classic **constraint-based backtracking** problem with pruning.

---

## Problem Statement

```
A permutation perm of [1, 2, ..., n] is "beautiful" if for every i:
  perm[i] % i == 0  OR  i % perm[i] == 0

Input: n = 2
Output: 2

Permutation [1,2]: 1%1==0 ✓, 2%2==0 ✓ → Beautiful
Permutation [2,1]: 2%1==0 ✓, 1%2≠0 but 2%1==0 ✓ → Beautiful
Answer: 2
```

---

## Approach: Backtracking with Constraint Pruning

```
At each position i (1-indexed):
  Try every unused number num from 1 to n
  ONLY proceed if: num % i == 0 OR i % num == 0
  This PRUNES many invalid branches early!

FUNCTION countArrangement(n):
    used = [false] × (n + 1)
    RETURN backtrack(n, 1, used)

FUNCTION backtrack(n, pos, used):
    IF pos > n:
        RETURN 1                     ← Found a beautiful arrangement
    
    count = 0
    FOR num = 1 TO n:
        IF NOT used[num]:
            IF num % pos == 0 OR pos % num == 0:  ← CONSTRAINT
                used[num] = true
                count += backtrack(n, pos + 1, used)
                used[num] = false
    
    RETURN count
```

---

## Trace for n = 3

```
Positions: 1, 2, 3
Numbers:   1, 2, 3

backtrack(pos=1, used={})
│  Constraint: num%1==0 (always true!) or 1%num==0
│  → ALL numbers pass at position 1
├── num=1: used={1}, backtrack(pos=2)
│   │  Constraint at pos=2: num%2==0 or 2%num==0
│   ├── num=2: 2%2==0 ✓ → used={1,2}, backtrack(pos=3)
│   │   └── num=3: 3%3==0 ✓ → pos>3 → COUNT +1 → [1,2,3] ✓
│   └── num=3: 3%2≠0, 2%3≠0 → PRUNED ✗
├── num=2: used={2}, backtrack(pos=2)
│   │  Constraint at pos=2: num%2==0 or 2%num==0
│   ├── num=1: 1%2≠0, 2%1==0 ✓ → used={1,2}, backtrack(pos=3)
│   │   └── num=3: 3%3==0 ✓ → COUNT +1 → [2,1,3] ✓
│   └── num=3: 3%2≠0, 2%3≠0 → PRUNED ✗
└── num=3: used={3}, backtrack(pos=2)
    │  Constraint at pos=2: num%2==0 or 2%num==0
    ├── num=1: 2%1==0 ✓ → used={1,3}, backtrack(pos=3)
    │   └── num=2: 2%3≠0, 3%2≠0 → PRUNED ✗
    └── num=2: 2%2==0 ✓ → used={2,3}, backtrack(pos=3)
        └── num=1: 1%3≠0, 3%1==0 ✓ → COUNT +1 → [3,2,1] ✓

Answer: 3
```

---

## Pruning Effectiveness

```
n=8: Total permutations = 8! = 40,320
     Beautiful arrangements = 568
     
     Without pruning: check all 40,320
     With pruning: explore ~3,000 nodes
     
     PRUNING removes ~93% of the search space!

Results for various n:
┌─────┬─────────┬───────────────┬──────────────┐
│  n  │  n!     │  Beautiful    │  Pruned %    │
├─────┼─────────┼───────────────┼──────────────┤
│  1  │      1  │            1  │    0%        │
│  2  │      2  │            2  │    0%        │
│  3  │      6  │            3  │   50%        │
│  4  │     24  │            8  │   67%        │
│  5  │    120  │           10  │   92%        │
│  6  │    720  │           36  │   95%        │
│  8  │ 40,320  │          568  │   99%        │
│ 10  │3,628,800│         700   │   ~99.98%    │
└─────┴─────────┴───────────────┴──────────────┘
```

---

## Optimization: Process Most Constrained Position First

```
Position 1: every number is divisible by 1 → n choices
Position n: only divisors and multiples of n → fewer choices

OPTIMIZATION: Start from position n and go DOWN to 1.
This puts the most constrained positions first,
causing more pruning at higher levels of the tree.

FUNCTION backtrack(n, pos, used):
    IF pos == 0:
        RETURN 1
    
    count = 0
    FOR num = 1 TO n:
        IF NOT used[num] AND (num%pos==0 OR pos%num==0):
            used[num] = true
            count += backtrack(n, pos - 1, used)  ← pos-1 (go backwards)
            used[num] = false
    RETURN count
```

---

## Complexity

```
Time: O(k) where k ≪ n! due to heavy pruning
  Exact complexity is hard to express, but
  empirically much better than O(n!)

Space: O(n) for the used array + recursion stack

Can also be solved with bitmask DP: O(n × 2^n)
  State: (position, set of used numbers as bitmask)
  This is polynomial-ish for small n (n ≤ 20)
```

---

## Summary Table

| Technique | Purpose |
|-----------|---------|
| Divisibility check | Constraint pruning |
| Used array | Track assigned numbers |
| Reverse order (pos n→1) | Most constrained first |
| Bitmask DP (advanced) | O(n × 2^n) guaranteed |

---

## Quick Revision Questions

1. **What is the constraint** for a "beautiful" arrangement?
2. **Why is position 1** the least constrained?
3. **How much pruning** occurs for n=8?
4. **Why process position n first** instead of position 1?
5. **How many beautiful arrangements** exist for n=4?
6. **What is the bitmask DP** state for this problem?

---

[← Previous: Permutation Sequence](04-permutation-sequence.md) | [Next: String Permutations →](06-string-permutations.md)

[← Back to README](../README.md)

# Chapter 8.2: Assignment Problem (Bitmask DP)

> **Unit 8 — Bit Manipulation in DP**
> Assign n workers to n jobs optimally in O(n × 2^n) using bitmask DP — faster than the Hungarian algorithm for small n.

---

## Overview

**Assignment Problem:** Given n workers and n jobs with a cost matrix cost[i][j], assign each worker to exactly one job (and vice versa) to minimize total cost. The bitmask DP approach is simple and elegant.

---

## State Definition

```
  dp[mask] = minimum cost to assign the first popcount(mask)
             workers to the jobs indicated by set bits in mask.

  Worker assignment order: worker 0, then worker 1, then worker 2, ...
  mask tracks WHICH JOBS have been assigned so far.
```

---

## Recurrence

```
  Let k = popcount(mask)  (number of jobs assigned = number of workers assigned)
  Worker being assigned = k (0-indexed)
  
  dp[mask] = MIN over all set bits j in mask:
               dp[mask ^ (1<<j)] + cost[k-1][j]

  "Assign worker k-1 to job j. Previous state had job j unassigned."
```

---

## Visual Explanation

```
  n = 3 workers, 3 jobs
  
  mask = 110 (jobs 1,2 assigned)
  popcount = 2, so workers 0 and 1 are assigned
  
  How did we get 110?
    Option A: First assigned worker 0 to job 1 (010), then worker 1 to job 2 (110)
    Option B: First assigned worker 0 to job 2 (100), then worker 1 to job 1 (110)
    
  dp[110] = min(
      dp[010] + cost[1][2],   // worker 1 takes job 2
      dp[100] + cost[1][1]    // worker 1 takes job 1
  )
```

---

## Pseudocode

```
FUNCTION assignmentProblem(cost[][], n):
    dp = array of size (1 << n), initialized to INFINITY
    dp[0] = 0    // no workers assigned, no cost
    
    FOR mask = 0 TO (1 << n) - 1:
        IF dp[mask] == INFINITY: CONTINUE
        k = popcount(mask)    // next worker to assign
        IF k == n: CONTINUE   // all workers assigned
        
        FOR j = 0 TO n-1:
            IF mask & (1<<j): CONTINUE    // job j already taken
            dp[mask | (1<<j)] = MIN(dp[mask | (1<<j)],
                                    dp[mask] + cost[k][j])
    
    RETURN dp[(1<<n) - 1]
```

---

## Step-by-Step Trace

```
  cost = [[9, 2, 7],     // worker 0
          [6, 4, 3],     // worker 1  
          [5, 8, 1]]     // worker 2

  n = 3, masks 000 to 111

  dp[000] = 0  (base case)

  mask=000, k=0 (assign worker 0):
    j=0: dp[001] = min(∞, 0+9) = 9
    j=1: dp[010] = min(∞, 0+2) = 2
    j=2: dp[100] = min(∞, 0+7) = 7

  mask=001, k=1 (assign worker 1):
    j=1: dp[011] = min(∞, 9+4) = 13
    j=2: dp[101] = min(∞, 9+3) = 12

  mask=010, k=1 (assign worker 1):
    j=0: dp[011] = min(13, 2+6) = 8
    j=2: dp[110] = min(∞, 2+3) = 5

  mask=100, k=1 (assign worker 1):
    j=0: dp[101] = min(12, 7+6) = 12
    j=1: dp[110] = min(5, 7+4) = 5

  mask=011, k=2 (assign worker 2):
    j=2: dp[111] = min(∞, 8+1) = 9

  mask=101, k=2 (assign worker 2):
    j=1: dp[111] = min(9, 12+8) = 9

  mask=110, k=2 (assign worker 2):
    j=0: dp[111] = min(9, 5+5) = 9

  Answer: dp[111] = 9
  Optimal: worker 0→job 1 (2), worker 1→job 2 (3), worker 2→job 0 (5-wait no)
  Actually: Let's trace: dp[110]=5 came from worker 0→job 1(2), worker 1→job 2(3).
  dp[111]=9 from dp[110]+cost[2][0]=5+5=10? Let me recheck...
  dp[011]=8 from dp[010]+cost[1][0]=2+6=8. dp[111] from dp[011]+cost[2][2]=8+1=9.
  
  Optimal: worker 0→job 1 (2), worker 1→job 0 (6), worker 2→job 2 (1) = 9  ✓
```

---

## Assignment vs TSP

```
  ┌──────────────────┬──────────────────────────────────┐
  │  Assignment       │  TSP                             │
  ├──────────────────┼──────────────────────────────────┤
  │  dp[mask]         │  dp[mask][i]                     │
  │  O(n × 2^n)       │  O(n² × 2^n)                    │
  │  No "last" needed │  Need to track last city         │
  │  One-to-one       │  Sequential tour                  │
  │  No return trip   │  Must return to start            │
  └──────────────────┴──────────────────────────────────┘
```

> Assignment is simpler because the order of workers is fixed (0, 1, 2, ...), so we don't need the extra dimension.

---

## Complexity

| Aspect | Value |
|--------|-------|
| States | O(2^n) |
| Transitions per state | O(n) |
| Total time | **O(n × 2^n)** |
| Space | O(2^n) |
| Practical limit | n ≤ 23 |

---

## Comparison with Other Methods

| Method | Time | Best for |
|--------|------|----------|
| Bitmask DP | O(n × 2^n) | n ≤ 23 |
| Hungarian algorithm | O(n³) | Large n |
| Brute force | O(n × n!) | n ≤ 10 |

> For n > 23, use the Hungarian algorithm.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| State | dp[mask] = min cost assigning first popcount(mask) workers |
| Key insight | Worker order is implicit (by popcount) |
| No extra dimension | Unlike TSP, no "last" tracking needed |
| Time | O(n × 2^n) — better than TSP by factor of n |
| Space | O(2^n) |

---

## Quick Revision Questions

1. **Why don't we need dp[mask][i] like in TSP?**
   <details><summary>Answer</summary>In the assignment problem, workers are assigned in fixed order (0, 1, 2, ...). The "current worker" is determined by popcount(mask), so we don't need an extra dimension.</details>

2. **How do we know which worker is being assigned for a given mask?**
   <details><summary>Answer</summary>Worker index = popcount(mask). If 3 jobs are assigned (popcount=3), we're assigning worker 3 next (0-indexed).</details>

3. **Can this handle an unequal number of workers and jobs?**
   <details><summary>Answer</summary>If workers < jobs, add dummy workers with cost 0. If workers > jobs, add dummy jobs. Or modify the DP to allow some workers/jobs to be unassigned.</details>

4. **What's the advantage over the Hungarian algorithm?**
   <details><summary>Answer</summary>Simpler to implement. For n ≤ 20, it's fast enough in competitive programming. Hungarian is O(n³) but has a complex implementation.</details>

5. **How do you reconstruct the optimal assignment?**
   <details><summary>Answer</summary>Track which j achieved the minimum for each dp[mask]. Backtrack from dp[FULL]: worker n-1 was assigned the job that minimized dp[FULL], etc.</details>

---

[← Previous: Traveling Salesman](01-traveling-salesman.md) | [Next: Counting Paths with States →](03-counting-paths-with-states.md)

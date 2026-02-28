# Chapter 6: Palindrome Partitioning — Interval DP Perspective

## 📋 Overview
We revisited Palindrome Partitioning in Unit 7 from a palindrome DP angle. Here we see it through the **Interval DP lens**, highlighting the MCM-like structure: split an interval at every possible point, recursively handle both halves.

---

## 🧠 Core Problem

```
Given string s, find minimum cuts to partition into palindromes.

s = "aab"

Partitions:
  "a" | "a" | "b"   →  2 cuts
  "aa" | "b"         →  1 cut  ← minimum

Answer: 1
```

---

## 🔨 Interval DP Formulation

```
dp[i][j] = min cuts to make s[i..j] all palindromes

If s[i..j] is already a palindrome:
  dp[i][j] = 0  (no cuts needed)

Otherwise:
  dp[i][j] = min over k from i to j-1 of:
      dp[i][k] + dp[k+1][j] + 1
      \_______/   \________/   |
      left part    right part  the cut at k

This is the CLASSIC interval DP template:
  "try every split, combine with cost"
  
  Same as MCM but:
  - MCM: combining cost = matrix multiplication
  - Here: combining cost = 1 (the cut itself)
  - Special case: cost = 0 if entire interval is palindrome
```

---

## 💻 Pseudocode

```
function minCuts_intervalDP(s):
    n = len(s)
    
    // Precompute palindrome table
    isPalin = 2D boolean [n][n]
    for length = 1 to n:
        for i = 0 to n-length:
            j = i + length - 1
            if length == 1:
                isPalin[i][j] = true
            elif length == 2:
                isPalin[i][j] = (s[i] == s[j])
            else:
                isPalin[i][j] = (s[i] == s[j]) and isPalin[i+1][j-1]
    
    // Interval DP for min cuts
    dp = 2D array [n][n], all 0
    
    for length = 2 to n:
        for i = 0 to n-length:
            j = i + length - 1
            if isPalin[i][j]:
                dp[i][j] = 0
            else:
                dp[i][j] = ∞
                for k = i to j-1:
                    dp[i][j] = min(dp[i][j], dp[i][k] + dp[k+1][j] + 1)
    
    return dp[0][n-1]

Time: O(n³), Space: O(n²)
```

---

## 🔬 Trace: s = "aab"

```
isPalin:
     a  a  b
  a [T  T  F]
  a [   T  F]
  b [      T]

dp computation:
  Length 1: all dp[i][i] = 0

  Length 2:
    dp[0][1] "aa": isPalin[0][1]=T → dp[0][1] = 0
    dp[1][2] "ab": isPalin[1][2]=F →
      k=1: dp[1][1]+dp[2][2]+1 = 0+0+1 = 1
      dp[1][2] = 1

  Length 3:
    dp[0][2] "aab": isPalin[0][2]=F →
      k=0: dp[0][0]+dp[1][2]+1 = 0+1+1 = 2
      k=1: dp[0][1]+dp[2][2]+1 = 0+0+1 = 1
      dp[0][2] = 1

Answer: 1 ✓
```

---

## 🔑 Comparison: Interval DP vs 1D Approach

```
Interval DP (this chapter):
  dp[i][j] = min cuts for s[i..j]
  Time: O(n³), Space: O(n²)
  Template: MCM-like split at every k

1D Approach (from Unit 7):
  dp[i] = min cuts for s[0..i]
  For each j where s[j..i] is palindrome:
    dp[i] = min(dp[j-1] + 1)
  Time: O(n²), Space: O(n)
  
The 1D approach is FASTER because:
  - Only need left endpoint to be 0 (for the answer)
  - Avoids triple-nested loop
  - Uses expand-around-center for palindrome check

┌──────────────┬──────────┬────────┬──────────────┐
│ Approach     │ Time     │ Space  │ When to Use  │
├──────────────┼──────────┼────────┼──────────────┤
│ Interval DP  │ O(n³)    │ O(n²) │ Understanding │
│ 1D DP        │ O(n²)    │ O(n)  │ Practice     │
│ 1D + expand  │ O(n²)    │ O(n)  │ Optimal      │
└──────────────┴──────────┴────────┴──────────────┘
```

---

## 🌐 When Interval DP IS Necessary

```
Interval DP is necessary when the answer isn't just
for s[0..n-1] but for ALL intervals:

1. Multiple queries: "What is min cuts for s[i..j]?"
   → Need full dp table

2. Palindrome Removal (Unit 7, Chapter 5):
   → Must track arbitrary intervals

3. Problems where outer structure depends on inner:
   → Natural interval DP territory

For single-query "min cuts for entire string":
   → 1D DP is sufficient and preferred
```

---

## 🧩 Generalized Interval DP for Partitioning

```
Generic partition problem:
  Given sequence, partition into parts.
  Each part has a cost function f(i, j).
  Find min total cost of all parts + cuts.

dp[i][j] = min cost to handle s[i..j]

dp[i][j] = min over k of:
    dp[i][k] + dp[k+1][j] + splitCost

Special cases:
  f(i,j) = isPalindrome? 0 : ∞  → Palindrome partitioning
  f(i,j) = p[i-1]*p[k]*p[j]    → Matrix chain multiplication
  f(i,j) = sum(stones[i..j])    → Merge stones
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min cuts for s[i..j] |
| **Recurrence** | dp[i][j] = min(dp[i][k] + dp[k+1][j] + 1) |
| **Palindrome base** | dp[i][j] = 0 if s[i..j] is palindrome |
| **Complexity** | O(n³) time, O(n²) space |
| **Better approach** | 1D DP in O(n²) for single query |
| **Interval DP value** | Understanding the MCM connection |

---

## ❓ Quick Revision Questions

1. **How is palindrome partitioning structured as interval DP?**
2. **What is the combining cost (equivalent to MCM's multiplication cost)?**
3. **Why is the 1D approach more efficient for this specific problem?**
4. **When would you prefer interval DP over 1D DP?**
5. **How does precomputing the palindrome table help?**
6. **Name three other problems that use the same interval DP template.**

---

[← Previous: Optimal BST](05-optimal-bst.md) | [Next Unit: State Machine DP →](../11-State-Machine-DP/01-buy-sell-stock-intro.md)

[← Back to README](../README.md)

# Chapter 2: Define State

## 📋 Overview
**Defining the state** is the most critical and often most challenging step in DP. The state captures all the information needed to make decisions at each point and compute the answer for subproblems.

---

## 🧠 What is a State?

```
┌─────────────────────────────────────────────────────────┐
│                  WHAT IS A STATE?                       │
│                                                         │
│  A STATE is a set of variables that COMPLETELY          │
│  describes where you are in the problem at any          │
│  given point.                                           │
│                                                         │
│  ┌─────────────────────────────────┐                    │
│  │  dp[state] = answer for this   │                    │
│  │              particular state  │                    │
│  └─────────────────────────────────┘                    │
│                                                         │
│  The state must capture:                                │
│  • What subproblem are we solving?                      │
│  • What constraints have been used?                     │
│  • What information is needed to make the next choice?  │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 State Definition Process

```
Step 1: What CHANGES as we recurse?
         │
         ▼
Step 2: What INFORMATION is needed at each step?
         │
         ▼
Step 3: What is the MINIMUM info to uniquely identify a subproblem?
         │
         ▼
Step 4: Define dp[state] = what it REPRESENTS (in words)
         │
         ▼
Step 5: Verify: Can we write a recurrence using this state?
```

---

## 📐 Common State Definitions

### 1D State: `dp[i]`
```
Meaning: Answer considering elements 0..i (or first i elements)

Problems:
  Fibonacci:    dp[i] = i-th Fibonacci number
  Stairs:       dp[i] = ways to reach step i
  House Robber: dp[i] = max money robbing houses 0..i
  Coin Change:  dp[i] = min coins to make amount i

┌───────────────────────────────────┐
│  dp[0]  dp[1]  dp[2]  ...  dp[n] │
│  base   base   ←───────────►     │
│                computed from      │
│                previous states    │
└───────────────────────────────────┘
```

### 2D State: `dp[i][j]`
```
Meaning depends on problem type:

Grid:     dp[i][j] = answer at cell (i,j)
Strings:  dp[i][j] = answer for s1[0..i] and s2[0..j]
Knapsack: dp[i][j] = answer using items 0..i with capacity j
Interval: dp[i][j] = answer for range [i..j]

┌─────────────────────────┐
│     j=0  j=1  j=2  j=3 │
│  i=0  ·    ·    ·    ·  │
│  i=1  ·    ·    ·    ·  │
│  i=2  ·    ·    ·    ·  │
│  i=3  ·    ·    ·    ·  │
└─────────────────────────┘
```

### Multi-dimensional: `dp[i][j][k]`
```
Meaning: Multiple changing parameters

Stock:    dp[i][k][holding] = max profit at day i, 
          k transactions used, holding/not-holding
          
3D Grid:  dp[i][j][k] = answer at position (i,j,k)

Bitmask:  dp[mask][i] = answer with visited set = mask,
          currently at node i
```

---

## 🧪 State Definition Examples

### Example 1: Longest Common Subsequence
```
Problem: Find LCS of "ABCDE" and "ACE"

What changes? → Position in string 1 (i), position in string 2 (j)
What info needed? → How far we've processed each string
Minimum info? → Just i and j (previous chars already decided)

State: dp[i][j] = length of LCS of s1[0..i-1] and s2[0..j-1]

Verification:
  if s1[i-1] == s2[j-1]:
      dp[i][j] = dp[i-1][j-1] + 1     ✓ uses previous states
  else:
      dp[i][j] = max(dp[i-1][j], dp[i][j-1])  ✓
```

### Example 2: 0/1 Knapsack
```
Problem: Items with weight & value, capacity W

What changes? → Which item we're considering (i), remaining capacity (w)
What info needed? → Items available, current capacity
Minimum info? → Item index i, capacity w

State: dp[i][w] = max value using items 0..i with capacity w

Verification:
  dp[i][w] = max(
      dp[i-1][w],                    // skip item i
      dp[i-1][w-wt[i]] + val[i]     // take item i
  )  ✓
```

### Example 3: Stock Buy/Sell with Cooldown
```
Problem: Max profit with cooldown after selling

What changes? → Day (i), are we holding stock? (state)
What info needed? → Current day, whether we hold a stock
Minimum info? → Day i, holding status

WRONG State: dp[i] = max profit on day i
  (Can't determine if we're holding stock or not!)

RIGHT State: dp[i][0] = max profit on day i, NOT holding
             dp[i][1] = max profit on day i, HOLDING

Verification:
  dp[i][0] = max(dp[i-1][0], dp[i-1][1] + prices[i])  ✓
  dp[i][1] = max(dp[i-1][1], dp[i-2][0] - prices[i])  ✓
                                  ↑ cooldown: skip day i-1
```

---

## ⚠️ Common State Definition Mistakes

```
┌─────────────────────────────────────────────────────────┐
│  MISTAKE 1: State is too narrow (missing information)   │
│                                                         │
│  Knapsack with dp[i] only                               │
│  Missing capacity! Can't decide whether to take item.   │
│  Fix: dp[i][w] — add capacity dimension                 │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  MISTAKE 2: State is too broad (too many dimensions)    │
│                                                         │
│  dp[i][j][k][l] when dp[i][j] suffices                 │
│  Wastes memory and time.                                │
│  Fix: Remove redundant dimensions                       │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  MISTAKE 3: State doesn't match the question            │
│                                                         │
│  Problem asks for minimum but dp stores count.          │
│  dp[i] = "number of ways" but we need "minimum cost"   │
│  Fix: Redefine dp[i] = "minimum cost to reach i"       │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  MISTAKE 4: Not thinking about what "i" means           │
│                                                         │
│  Does dp[i] mean:                                       │
│    "answer using first i items"?                        │
│    "answer ending at index i"?                          │
│    "answer for value i"?                                │
│  These are DIFFERENT definitions!                        │
└─────────────────────────────────────────────────────────┘
```

---

## 💡 State Definition Checklist

```
□ Write dp[state] = _____ (in plain English)
□ Can I write a recurrence from this definition?
□ Can I identify base cases?
□ Does the state capture ALL necessary information?
□ Is the state space small enough (fits in memory)?
□ Does the answer to the original problem exist in my table?
   (Usually dp[n], dp[m][n], or max/min over all dp[i])
```

---

## 📊 Summary Table

| Problem Type | State Definition | Dimensions |
|-------------|-----------------|------------|
| **Fibonacci-like** | dp[i] = answer for i | 1D |
| **Grid Paths** | dp[i][j] = answer at cell (i,j) | 2D |
| **String Match** | dp[i][j] = answer for prefixes of length i,j | 2D |
| **Knapsack** | dp[i][w] = answer for items 0..i, capacity w | 2D |
| **Interval** | dp[i][j] = answer for range [i,j] | 2D |
| **Stock+State** | dp[i][state] = answer at position i in state | 2D |
| **Bitmask** | dp[mask] = answer for subset represented by mask | 1D (2ⁿ) |

---

## ❓ Quick Revision Questions

1. **What is a "state" in DP and why is it important?**
2. **For the problem "minimum coins to make amount n," what is the state and what does it represent?**
3. **Why is dp[i] insufficient for the stock buy/sell problem with cooldown?**
4. **How do you know if your state definition has too many dimensions?**
5. **What is the difference between dp[i] meaning "answer for first i items" vs "answer ending at item i"?**
6. **Write the state definition for: "Count ways to partition array into two subsets with equal sum."**

---

[← Previous: Identify Problem Type](01-identify-problem-type.md) | [Next: Define Recurrence Relation →](03-define-recurrence-relation.md)

[← Back to README](../README.md)

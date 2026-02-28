# Chapter 3: Define Recurrence Relation

## 📋 Overview
The **recurrence relation** is the mathematical formula that expresses how the solution to a problem relates to the solutions of its subproblems. It is the heart of every DP solution — once you have the correct recurrence, the implementation follows naturally.

---

## 🧠 What is a Recurrence Relation?

```
┌─────────────────────────────────────────────────────────┐
│              RECURRENCE RELATION                        │
│                                                         │
│  A formula that defines dp[current_state] in terms      │
│  of dp[smaller_states]                                  │
│                                                         │
│  General form:                                          │
│  dp[i] = f(dp[i-1], dp[i-2], ..., choices, costs)      │
│                                                         │
│  Components:                                            │
│  ┌──────────────┐  ┌───────────┐  ┌──────────────┐     │
│  │ Current State │  │  Choices  │  │ Combination  │     │
│  │  dp[i] = ?   │  │ What can  │  │  How to      │     │
│  │              │  │ we do at  │  │  combine     │     │
│  │              │  │ state i?  │  │  results?    │     │
│  └──────────────┘  └───────────┘  └──────────────┘     │
│                                                         │
│  Combination operators:                                 │
│  • min()  → optimization (minimize)                     │
│  • max()  → optimization (maximize)                     │
│  • +      → counting                                    │
│  • ||     → existence (can we?)                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔍 Building Recurrences Step by Step

### Framework: "At state i, what are my CHOICES?"

```
At state i, I can:
  Choice 1 → leads to state j₁ with cost c₁
  Choice 2 → leads to state j₂ with cost c₂
  ...
  Choice k → leads to state jₖ with cost cₖ

Recurrence:
  dp[i] = optimize over all choices:
           f(dp[next_state], cost_of_choice)
```

---

## 📐 Classic Recurrence Relations

### 1. Fibonacci Pattern
```
State: dp[i] = i-th value
Choices at i: come from i-1 or i-2
Recurrence: dp[i] = dp[i-1] + dp[i-2]

Diagram:
  dp[i-2] ──┐
             ├──► dp[i] = dp[i-1] + dp[i-2]
  dp[i-1] ──┘
```

### 2. Climbing Stairs (k steps)
```
State: dp[i] = ways to reach step i
Choices: take 1, 2, ..., or k steps
Recurrence: dp[i] = dp[i-1] + dp[i-2] + ... + dp[i-k]

Diagram:
  dp[i-k] ──┐
  ...        ├──► dp[i] = Σ dp[i-j] for j=1..k
  dp[i-2] ──┤
  dp[i-1] ──┘
```

### 3. House Robber
```
State: dp[i] = max money from houses 0..i
Choices at house i: rob it or skip it
Recurrence: dp[i] = max(
                dp[i-1],              // skip house i
                dp[i-2] + nums[i]     // rob house i
            )

Diagram:
  ┌─ Skip: dp[i-1] ──────────────────┐
  │                                    ├─► dp[i] = max(...)
  └─ Rob:  dp[i-2] + nums[i] ────────┘
```

### 4. Coin Change (Minimum Coins)
```
State: dp[i] = min coins for amount i
Choices: use any coin c where c ≤ i
Recurrence: dp[i] = min(dp[i-c] + 1) for all coins c

Diagram:
  For coins = [1, 5, 6]:
  dp[i-1] + 1 ──┐
  dp[i-5] + 1 ──┼──► dp[i] = min(all)
  dp[i-6] + 1 ──┘
```

### 5. Longest Common Subsequence
```
State: dp[i][j] = LCS length of s1[0..i-1], s2[0..j-1]
Choices: match characters or skip one

Recurrence:
  if s1[i-1] == s2[j-1]:
      dp[i][j] = dp[i-1][j-1] + 1       // characters match
  else:
      dp[i][j] = max(dp[i-1][j],         // skip from s1
                     dp[i][j-1])          // skip from s2

Diagram:
  dp[i-1][j-1] ──► (match) ──┐
                              ├──► dp[i][j]
  dp[i-1][j] ──► (skip s1) ──┤
  dp[i][j-1] ──► (skip s2) ──┘
```

### 6. 0/1 Knapsack
```
State: dp[i][w] = max value with items 0..i, capacity w
Choices: take item i or skip it

Recurrence:
  dp[i][w] = max(
      dp[i-1][w],                         // skip item
      dp[i-1][w-weight[i]] + value[i]     // take item (if fits)
  )

Diagram:
  ┌─ Skip: dp[i-1][w] ─────────────────────┐
  │                                          ├─► dp[i][w]
  └─ Take: dp[i-1][w-wt[i]] + val[i] ──────┘
           (only if w >= wt[i])
```

---

## 🧪 Building a Recurrence: Worked Example

### Problem: Minimum Path Sum in Grid

```
Given grid:
┌───┬───┬───┐
│ 1 │ 3 │ 1 │
├───┼───┼───┤
│ 1 │ 5 │ 1 │
├───┼───┼───┤
│ 4 │ 2 │ 1 │
└───┴───┴───┘

Step 1: Define state
  dp[i][j] = minimum path sum to reach cell (i,j)
  
Step 2: Identify choices at (i,j)
  We can arrive from (i-1,j) [above] or (i,j-1) [left]
  
Step 3: Write recurrence
  dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
  
Step 4: Handle boundaries
  First row:  dp[0][j] = dp[0][j-1] + grid[0][j]  (only from left)
  First col:  dp[i][0] = dp[i-1][0] + grid[i][0]  (only from above)
  Corner:     dp[0][0] = grid[0][0]                (start)

Step 5: Verify
  dp[0][0] = 1
  dp[0][1] = 1+3 = 4
  dp[0][2] = 4+1 = 5
  dp[1][0] = 1+1 = 2
  dp[1][1] = 5 + min(4, 2) = 5+2 = 7
  dp[1][2] = 1 + min(5, 7) = 1+5 = 6
  dp[2][0] = 2+4 = 6
  dp[2][1] = 2 + min(7, 6) = 2+6 = 8
  dp[2][2] = 1 + min(6, 8) = 1+6 = 7 ✓

Table:
┌───┬───┬───┐
│ 1 │ 4 │ 5 │
├───┼───┼───┤
│ 2 │ 7 │ 6 │
├───┼───┼───┤
│ 6 │ 8 │ 7 │ ◄── Answer
└───┴───┴───┘
```

---

## ⚡ Recurrence Patterns by Problem Type

```
┌──────────────┬──────────────────────────────────────────┐
│ Pattern      │ Typical Recurrence                       │
├──────────────┼──────────────────────────────────────────┤
│ Fibonacci    │ dp[i] = dp[i-1] + dp[i-2]               │
│              │                                          │
│ Take/Skip    │ dp[i] = max(dp[i-1], dp[i-2]+v[i])      │
│              │                                          │
│ Min over all │ dp[i] = min(dp[i-c]+1) ∀ valid c        │
│              │                                          │
│ Grid         │ dp[i][j] = f(dp[i-1][j], dp[i][j-1])    │
│              │                                          │
│ Two strings  │ dp[i][j] = f(dp[i-1][j-1],              │
│              │              dp[i-1][j], dp[i][j-1])     │
│              │                                          │
│ Interval     │ dp[i][j] = opt_k(dp[i][k] + dp[k+1][j]) │
│              │            + cost(i,j)                   │
│              │                                          │
│ Knapsack     │ dp[i][w] = max(dp[i-1][w],              │
│              │               dp[i-1][w-wt]+val)         │
└──────────────┴──────────────────────────────────────────┘
```

---

## 💡 Tips for Writing Recurrences

```
1. Start by asking: "What is the LAST decision made?"
   → This often reveals the choices and recurrence naturally

2. Consider ALL possible choices at each state
   → Don't miss any option

3. Make sure each choice leads to a SMALLER subproblem
   → Ensures the recursion terminates

4. Verify with a small example
   → Trace through by hand

5. Check: Is the combination operator correct?
   → min for minimization, max for maximization,
     + for counting, || for existence
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Recurrence Relation** | Formula expressing dp[i] in terms of smaller states |
| **Choices** | All possible decisions at current state |
| **Combination** | min/max/+/|| to combine sub-solutions |
| **Key Question** | "What is the last decision?" |
| **Verification** | Trace with small input by hand |
| **From State to Recurrence** | State + Choices + Combination = Recurrence |

---

## ❓ Quick Revision Questions

1. **What is a recurrence relation and what are its three components?**
2. **Write the recurrence for the House Robber problem and explain each term.**
3. **What combination operator do you use for counting problems? For optimization?**
4. **How does the question "What was the last decision?" help build recurrences?**
5. **Write the recurrence for "number of ways to make amount n using coins of denominations c₁, c₂, ..., cₖ."**
6. **For the LCS problem, why do we need three sub-states (dp[i-1][j-1], dp[i-1][j], dp[i][j-1])?**

---

[← Previous: Define State](02-define-state.md) | [Next: Identify Base Cases →](04-identify-base-cases.md)

[← Back to README](../README.md)

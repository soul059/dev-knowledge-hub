# Chapter 1: Identify the Problem Type

## 📋 Overview
The first step in solving any DP problem is **identifying what type of problem it is**. This determines your state definition, recurrence relation, and overall approach. Recognizing patterns quickly is a crucial skill.

---

## 🧠 Major DP Problem Categories

```
┌─────────────────────────────────────────────────────────────┐
│                   DP PROBLEM TAXONOMY                       │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  LINEAR DP   │  │   GRID DP    │  │   STRING DP      │  │
│  │  (1D array)  │  │   (2D grid)  │  │   (1 or 2 strs)  │  │
│  │              │  │              │  │                   │  │
│  │ • Fibonacci  │  │ • Unique     │  │ • LCS            │  │
│  │ • Stairs     │  │   Paths      │  │ • Edit Distance  │  │
│  │ • Robber     │  │ • Min Path   │  │ • Palindromes    │  │
│  │ • Kadane's   │  │   Sum        │  │ • Subsequences   │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  KNAPSACK    │  │  INTERVAL    │  │  STATE MACHINE   │  │
│  │  (items+cap) │  │  (ranges)    │  │  (transitions)   │  │
│  │              │  │              │  │                   │  │
│  │ • 0/1 Knap.  │  │ • MCM       │  │ • Stock Buy/Sell │  │
│  │ • Subset Sum │  │ • Burst      │  │ • With cooldown  │  │
│  │ • Coin Change│  │   Balloons   │  │ • With fee       │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │   LIS        │  │  BITMASK     │                        │
│  │  (sequences) │  │  (subsets)   │                        │
│  │              │  │              │                        │
│  │ • LIS        │  │ • TSP       │                        │
│  │ • Envelopes  │  │ • Hamilton  │                        │
│  │ • Bridges    │  │ • Partition │                        │
│  └──────────────┘  └──────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔍 Identification Flowchart

```
START: Read the problem
  │
  ├── Involves a SINGLE array/sequence?
  │     ├── Find max/min value? ──► 1D DP / Kadane's
  │     ├── Count ways? ──► 1D Counting DP
  │     ├── Longest increasing? ──► LIS Pattern
  │     └── Select items with constraint? ──► Knapsack
  │
  ├── Involves a 2D GRID?
  │     └── Path/sum/count in grid? ──► Grid DP
  │
  ├── Involves ONE or TWO strings?
  │     ├── Compare two strings? ──► LCS / Edit Distance
  │     ├── Single string palindrome? ──► Palindrome DP
  │     └── Substring/subsequence? ──► String DP
  │
  ├── Involves SPLITTING a range?
  │     └── Optimal way to split/merge? ──► Interval DP
  │
  ├── Involves STATES and TRANSITIONS?
  │     └── Multiple states per position? ──► State Machine DP
  │
  └── Involves SUBSETS of items?
        └── n ≤ 20? ──► Bitmask DP
```

---

## 📐 Pattern Recognition Table

| If the problem says... | Think... | Pattern |
|------------------------|----------|---------|
| "How many ways to reach..." | Counting paths | Grid/1D DP |
| "Minimum cost to..." | Optimization | Depends on structure |
| "Maximum value with weight limit" | Item selection | Knapsack |
| "Longest common..." | String comparison | String DP (LCS) |
| "Transform string A to B" | Edit operations | Edit Distance |
| "Is palindrome" or "make palindrome" | Palindrome check | Palindrome DP |
| "Split/merge optimally" | Range operations | Interval DP |
| "Buy and sell with rules" | State-based decisions | State Machine |
| "Visit all" with n ≤ 20 | All subsets | Bitmask DP |
| "Non-adjacent elements" | Gap constraint | Fibonacci-like |

---

## 🧪 Examples: Identifying the Type

### Example 1
```
"Given an array of house values, find maximum money you can 
rob without robbing adjacent houses."

Keywords: "maximum", "array", "adjacent"
Structure: Linear array, gap constraint
Pattern: 1D DP (Fibonacci-like / House Robber)
```

### Example 2
```
"Given two strings, find the length of their longest 
common subsequence."

Keywords: "two strings", "longest", "subsequence"
Structure: Two sequences to compare
Pattern: String DP (LCS)
```

### Example 3
```
"Given a set of items with weights and values, and a knapsack 
of capacity W, find maximum value."

Keywords: "items", "weights", "capacity", "maximum"
Structure: Items with constraint
Pattern: Knapsack DP
```

### Example 4
```
"Given stock prices over n days, find maximum profit with at 
most k transactions."

Keywords: "stock prices", "maximum profit", "k transactions"
Structure: Sequence + multiple states (holding/not, transactions left)
Pattern: State Machine DP
```

---

## 💡 Quick Classification Tips

```
┌────────────────────────────────────────────────────────┐
│  TIP 1: Count the changing parameters                  │
│    1 parameter  → 1D DP                                │
│    2 parameters → 2D DP (grid, strings, knapsack)      │
│    3+ params    → Multi-dim or bitmask                 │
│                                                        │
│  TIP 2: Look at data structure                         │
│    Array     → 1D DP, LIS, Kadane's                    │
│    Grid      → Grid DP                                 │
│    String(s) → String DP                               │
│    Graph     → Shortest path DP, Bitmask               │
│    Tree      → Tree DP                                 │
│                                                        │
│  TIP 3: Look at what's asked                           │
│    Min/Max   → Optimization DP                         │
│    Count     → Counting DP                             │
│    Yes/No    → Existence DP                            │
│    Construct → DP + backtracking reconstruction        │
└────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Category | Input Structure | State Dimensions | Example Problems |
|----------|----------------|-----------------|------------------|
| **1D DP** | Array/Sequence | dp[i] | Climbing stairs, House robber |
| **Grid DP** | 2D Matrix | dp[i][j] | Unique paths, Min path sum |
| **String DP** | 1-2 Strings | dp[i][j] | LCS, Edit distance |
| **Knapsack** | Items + Capacity | dp[i][w] | 0/1 Knapsack, Subset sum |
| **LIS** | Sequence | dp[i] | LIS, Envelopes |
| **Interval** | Range [i,j] | dp[i][j] | MCM, Burst balloons |
| **State Machine** | Sequence + States | dp[i][state] | Stock problems |
| **Bitmask** | Set (n ≤ 20) | dp[mask] | TSP, Hamiltonian |

---

## ❓ Quick Revision Questions

1. **What DP pattern would you use for "find minimum operations to convert string A to string B"?**
2. **If n ≤ 15 and the problem asks to visit all nodes, what pattern should you consider?**
3. **How do you distinguish between a Knapsack problem and a Coin Change problem?**
4. **What pattern fits: "given a row of balloons, find maximum coins by bursting them in optimal order"?**
5. **Name three signal words for Optimization DP problems.**
6. **How does the number of changing parameters help classify DP problems?**

---

[← Previous Unit: When to Use DP](../01-DP-Fundamentals/06-when-to-use-dp.md) | [Next: Define State →](02-define-state.md)

[← Back to README](../README.md)

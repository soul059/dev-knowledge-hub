# Chapter 1: Fibonacci Number

## 📋 Overview
The **Fibonacci sequence** is the foundation of many DP problems. Every number is the sum of the two preceding ones: F(0)=0, F(1)=1, F(n)=F(n-1)+F(n-2). Understanding Fibonacci deeply unlocks a whole family of DP patterns.

---

## 🧠 The Fibonacci Sequence

```
Index:  0  1  2  3  4  5  6  7   8   9  10
Value:  0  1  1  2  3  5  8  13  21  34  55

Growth pattern (Golden Ratio ≈ 1.618):
  F(n+1)/F(n) approaches φ = (1+√5)/2 ≈ 1.618

  │55
  │     ╱
  │    ╱
  │34 ╱
  │  ╱
  │21
  │13
  │8
  │5
  │3
  │1 2
  └────────────► n
```

---

## 🔍 All Four Approaches

### Approach 1: Naive Recursion — O(2ⁿ)
```
function fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)
```

### Approach 2: Memoization — O(n)
```
function fib(n, memo={}):
    if n in memo: return memo[n]
    if n <= 1: return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
```

### Approach 3: Tabulation — O(n) time, O(n) space
```
function fib(n):
    dp = [0, 1]
    for i = 2 to n:
        dp.append(dp[i-1] + dp[i-2])
    return dp[n]
```

### Approach 4: Space Optimized — O(n) time, O(1) space
```
function fib(n):
    if n <= 1: return n
    a, b = 0, 1
    for i = 2 to n:
        a, b = b, a + b
    return b
```

### Approach 5: Matrix Exponentiation — O(log n)
```
| F(n+1) |   | 1  1 |ⁿ   | 1 |
| F(n)   | = | 1  0 |  × | 0 |

Using fast matrix power: O(log n) time, O(1) space
```

---

## 🧪 Complexity Comparison

```
┌─────────────────────┬──────────┬────────┬──────────────────┐
│ Approach            │ Time     │ Space  │ Notes            │
├─────────────────────┼──────────┼────────┼──────────────────┤
│ Naive Recursion     │ O(2ⁿ)   │ O(n)   │ Exponential      │
│ Memoization         │ O(n)    │ O(n)   │ Top-down         │
│ Tabulation          │ O(n)    │ O(n)   │ Bottom-up        │
│ Space Optimized     │ O(n)    │ O(1)   │ Two variables    │
│ Matrix Expo         │ O(log n)│ O(1)   │ For very large n │
│ Binet's Formula     │ O(1)*   │ O(1)   │ Float precision  │
└─────────────────────┴──────────┴────────┴──────────────────┘
* Binet's has floating point precision issues for large n
```

---

## 💡 The Fibonacci Pattern in Other Problems

```
Many DP problems reduce to the Fibonacci pattern:
dp[i] = dp[i-1] + dp[i-2]  (or variants thereof)

┌──────────────────────┬──────────────────────────────────┐
│ Problem              │ How it maps to Fibonacci         │
├──────────────────────┼──────────────────────────────────┤
│ Climbing Stairs      │ dp[i] = dp[i-1] + dp[i-2]       │
│ House Robber         │ dp[i] = max(dp[i-1], dp[i-2]+v) │
│ Tribonacci           │ dp[i] = dp[i-1]+dp[i-2]+dp[i-3] │
│ Decode Ways          │ dp[i] = dp[i-1] + dp[i-2]*      │
│ Tiling 2×n board     │ dp[i] = dp[i-1] + dp[i-2]       │
│ Binary strings no    │ dp[i] = dp[i-1] + dp[i-2]       │
│ consecutive 1s       │                                  │
└──────────────────────┴──────────────────────────────────┘
* = with conditions on valid digits
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Definition** | F(0)=0, F(1)=1, F(n)=F(n-1)+F(n-2) |
| **Growth** | Exponential (~φⁿ where φ≈1.618) |
| **Best Simple Approach** | O(n) time, O(1) space |
| **Best Advanced** | O(log n) via matrix exponentiation |
| **Pattern** | dp[i] depends on fixed number of previous states |
| **Significance** | Foundation for understanding 1D DP |

---

## ❓ Quick Revision Questions

1. **What are the first 10 Fibonacci numbers?**
2. **Why is the naive recursive approach O(2ⁿ)?**
3. **How does memoization reduce this to O(n)?**
4. **Explain the space optimization from O(n) to O(1).**
5. **What is the matrix exponentiation approach and when would you use it?**
6. **Name three problems that follow the Fibonacci pattern.**

---

[← Previous Unit: Decode Ways](../03-1D-DP-Problems/06-decode-ways.md) | [Next: Tribonacci →](02-tribonacci.md)

[← Back to README](../README.md)

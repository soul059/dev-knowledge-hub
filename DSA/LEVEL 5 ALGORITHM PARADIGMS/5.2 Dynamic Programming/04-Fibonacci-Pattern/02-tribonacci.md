# Chapter 2: Tribonacci Number

## 📋 Overview
The **Tribonacci sequence** extends Fibonacci by summing the previous **three** numbers instead of two: T(n) = T(n-1) + T(n-2) + T(n-3). It demonstrates how the Fibonacci pattern generalizes to k-step recurrences and is a direct LeetCode problem (#1137).

---

## 🧠 Core Concept

```
Tribonacci: T(0)=0, T(1)=1, T(2)=1
            T(n) = T(n-1) + T(n-2) + T(n-3)  for n ≥ 3

Index:  0  1  2  3  4  5   6   7   8    9    10
Value:  0  1  1  2  4  7  13  24  44   81   149

Comparison with Fibonacci:
┌───────┬────────────┬──────────────┐
│   n   │ Fibonacci  │ Tribonacci   │
├───────┼────────────┼──────────────┤
│   0   │     0      │      0       │
│   1   │     1      │      1       │
│   2   │     1      │      1       │
│   3   │     2      │      2       │
│   4   │     3      │      4       │
│   5   │     5      │      7       │
│   6   │     8      │     13       │
│   7   │    13      │     24       │
│   8   │    21      │     44       │
│   9   │    34      │     81       │
│  10   │    55      │    149       │
└───────┴────────────┴──────────────┘
Tribonacci grows faster (ratio → 1.839... vs 1.618...)
```

---

## 🔨 Step-by-Step Development

### Step 1: Naive Recursion — O(3ⁿ)
```
function tribonacci(n):
    if n == 0: return 0
    if n <= 2: return 1
    return tribonacci(n-1) + tribonacci(n-2) + tribonacci(n-3)

Recursion tree for n=5:
                     T(5)
                   / |  \
              T(4)  T(3)  T(2)
             /|\    /|\     |
          T3 T2 T1 T2 T1 T0  1
         /|\  |  | |  |  |
       T2 T1 T0 1 1  1  1 0
       |   |  |
       1   1  0
```

### Step 2: Memoization — O(n) time, O(n) space
```
function tribonacci(n, memo={}):
    if n in memo: return memo[n]
    if n == 0: return 0
    if n <= 2: return 1
    memo[n] = tribonacci(n-1, memo) + tribonacci(n-2, memo) + tribonacci(n-3, memo)
    return memo[n]
```

### Step 3: Tabulation — O(n) time, O(n) space
```
function tribonacci(n):
    if n == 0: return 0
    if n <= 2: return 1
    dp = array of size n+1
    dp[0] = 0, dp[1] = 1, dp[2] = 1
    for i = 3 to n:
        dp[i] = dp[i-1] + dp[i-2] + dp[i-3]
    return dp[n]
```

### Step 4: Space Optimized — O(n) time, O(1) space
```
function tribonacci(n):
    if n == 0: return 0
    if n <= 2: return 1
    a, b, c = 0, 1, 1
    for i = 3 to n:
        next = a + b + c
        a = b
        b = c
        c = next
    return c
```

---

## 🔬 Trace: tribonacci(6)

```
Initial: a=0, b=1, c=1

i=3: next = 0+1+1 = 2   → a=1,  b=1,  c=2
i=4: next = 1+1+2 = 4   → a=1,  b=2,  c=4
i=5: next = 1+2+4 = 7   → a=2,  b=4,  c=7
i=6: next = 2+4+7 = 13  → a=4,  b=7,  c=13

Return c = 13 ✓

Verification: 0, 1, 1, 2, 4, 7, 13 ✓
```

---

## 🌐 Generalization: N-bonacci

```
K-bonacci: sum of previous k numbers

┌────────────┬──────────┬───────────────────────────────┐
│ Name       │ k value  │ Recurrence                    │
├────────────┼──────────┼───────────────────────────────┤
│ Fibonacci  │  k = 2   │ dp[i] = dp[i-1] + dp[i-2]    │
│ Tribonacci │  k = 3   │ dp[i] = Σ dp[i-j] for j=1..3 │
│ Tetranacci │  k = 4   │ dp[i] = Σ dp[i-j] for j=1..4 │
│ K-bonacci  │  k = k   │ dp[i] = Σ dp[i-j] for j=1..k │
└────────────┴──────────┴───────────────────────────────┘

General space-optimized approach:
- Use a sliding window of size k
- Or use a deque

function kbonacci(n, k):
    if n == 0: return 0
    if n <= k-1: return 1  (or problem-specific base)
    window = deque of size k
    windowSum = sum of window
    for i = k to n:
        next = windowSum
        windowSum -= window.popleft()
        window.append(next)
        windowSum += next
    return window[-1]

Time: O(n), Space: O(k)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Recurrence** | T(n) = T(n-1) + T(n-2) + T(n-3) |
| **Base Cases** | T(0)=0, T(1)=1, T(2)=1 |
| **Growth Rate** | ~1.839ⁿ (Tribonacci constant) |
| **Optimal** | O(n) time, O(1) space with 3 variables |
| **Generalization** | K-bonacci uses sliding window of size k |
| **LeetCode** | #1137 N-th Tribonacci Number |

---

## ❓ Quick Revision Questions

1. **What is the Tribonacci recurrence relation and its base cases?**
2. **Why is naive recursion O(3ⁿ) for Tribonacci?**
3. **How many variables do you need for space optimization?**
4. **How does the growth rate compare to Fibonacci?**
5. **How would you generalize to a K-bonacci sequence?**
6. **Write the space-optimized solution for Tribonacci.**

---

[← Previous: Fibonacci Number](01-fibonacci-number.md) | [Next: House Robber →](03-house-robber.md)

[← Back to README](../README.md)

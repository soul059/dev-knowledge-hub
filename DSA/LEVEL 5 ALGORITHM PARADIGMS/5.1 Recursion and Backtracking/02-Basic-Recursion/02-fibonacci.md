# Chapter 2: Fibonacci

## Overview
The Fibonacci sequence is a classic problem that demonstrates **binary recursion** (two recursive calls), exposes the **overlapping subproblems** issue, and naturally leads to dynamic programming optimization.

---

## Mathematical Definition

```
F(0) = 0
F(1) = 1
F(n) = F(n-1) + F(n-2)   for n ≥ 2

Sequence: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ...
```

---

## Naive Recursive Solution

### Pseudocode
```
function fib(n):
    if n <= 0: return 0      // Base case 1
    if n == 1: return 1      // Base case 2
    return fib(n-1) + fib(n-2)  // Two recursive calls
```

### Step-by-Step Trace: fib(5)

```
fib(5)
├── fib(4)
│   ├── fib(3)
│   │   ├── fib(2)
│   │   │   ├── fib(1) → 1
│   │   │   └── fib(0) → 0
│   │   │   returns 1
│   │   └── fib(1) → 1
│   │   returns 2
│   └── fib(2)
│       ├── fib(1) → 1
│       └── fib(0) → 0
│       returns 1
│   returns 3
└── fib(3)
    ├── fib(2)
    │   ├── fib(1) → 1
    │   └── fib(0) → 0
    │   returns 1
    └── fib(1) → 1
    returns 2
returns 5

Total function calls for fib(5) = 15 calls!
```

### Recursion Tree Visualization

```
                        fib(5)
                       /      \
                   fib(4)      fib(3)
                  /     \      /     \
              fib(3)  fib(2) fib(2) fib(1)
              /   \    / \    / \      |
          fib(2) fib(1) f(1) f(0) f(1) f(0)  1
          /   \    |     |    |    |    |
       fib(1) fib(0) 1   1    0   1    0
         |      |
         1      0

[!] Notice: fib(3) computed 2 times, fib(2) computed 3 times!
    → OVERLAPPING SUBPROBLEMS (wasteful!)
```

---

## The Overlapping Subproblems Problem

```
Call count for fib(n):

fib(5):  fib(0)=3  fib(1)=5  fib(2)=3  fib(3)=2  fib(4)=1  fib(5)=1
                                                    Total = 15

fib(10): Total calls ≈ 177
fib(20): Total calls ≈ 21,891
fib(30): Total calls ≈ 2,692,537
fib(40): Total calls ≈ 331,160,281
fib(50): Total calls ≈ 40,730,022,147

         EXPONENTIAL EXPLOSION!
         
         ┌──────────────────────────────────┐
  Calls  │                          ╱       │
  (log)  │                       ╱          │
         │                    ╱             │
         │                ╱                 │
         │            ╱                     │
         │        ╱                         │
         │    ╱                             │
         │╱                                 │
         └──────────────────────────────────┘
                      n →
```

---

## Optimized Solutions

### Solution 1: Memoization (Top-Down DP)

```
function fib_memo(n, memo={}):
    if n in memo: return memo[n]    // Already computed!
    if n <= 0: return 0
    if n == 1: return 1
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
    return memo[n]
```

```
Memoized call tree for fib(5):
                        fib(5)
                       /      \
                   fib(4)      fib(3) ← MEMO HIT! returns 2
                  /     \      
              fib(3)  fib(2) ← MEMO HIT! returns 1
              /   \    
          fib(2) fib(1)
          /   \    |
       fib(1) fib(0) 1
         |      |
         1      0

Total calls: 9 (vs 15 without memo)
For large n: O(n) calls instead of O(2^n)!
```

### Solution 2: Iterative (Bottom-Up)

```
function fib_iter(n):
    if n <= 0: return 0
    if n == 1: return 1
    a = 0, b = 1
    for i = 2 to n:
        temp = a + b
        a = b
        b = temp
    return b
```

```
Trace for n=6:
┌──────┬─────┬─────┬──────┐
│  i   │  a  │  b  │ temp │
├──────┼─────┼─────┼──────┤
│ init │  0  │  1  │  -   │
│  2   │  1  │  1  │  1   │
│  3   │  1  │  2  │  2   │
│  4   │  2  │  3  │  3   │
│  5   │  3  │  5  │  5   │
│  6   │  5  │  8  │  8   │
└──────┴─────┴─────┴──────┘
return b = 8 ✓  (F(6) = 8)
```

### Solution 3: Matrix Exponentiation (O(log n))

```
┌         ┐n     ┌           ┐
│  1   1  │    = │ F(n+1) F(n)│
│  1   0  │      │ F(n)  F(n-1)│
└         ┘      └           ┘

Using fast matrix power: O(log n) time!
(Advanced — used in competitive programming)
```

---

## Complexity Comparison

```
┌──────────────────────────────────────────────────────┐
│         FIBONACCI COMPLEXITY COMPARISON               │
│                                                      │
│  Method          │ Time      │ Space    │ Notes      │
│  ────────────────┼───────────┼──────────┼─────────── │
│  Naive recursive │ O(2^n)    │ O(n)     │ Terrible!  │
│  Memoized        │ O(n)      │ O(n)     │ Great      │
│  Iterative       │ O(n)      │ O(1)     │ Best       │
│  Matrix power    │ O(log n)  │ O(1)     │ Advanced   │
│  Closed form     │ O(1)*     │ O(1)     │ Imprecise  │
│                                                      │
│  * Golden ratio formula has floating-point errors    │
└──────────────────────────────────────────────────────┘
```

---

## Why Naive Fibonacci is O(2^n)

```
Recurrence: T(n) = T(n-1) + T(n-2) + O(1)

At each level, work roughly doubles:
Level 0:  1 call
Level 1:  2 calls
Level 2:  4 calls
Level 3:  8 calls
...
Level n:  ~2^n calls

More precisely: T(n) ≈ φ^n where φ = (1+√5)/2 ≈ 1.618
So T(n) = O(1.618^n) ≈ O(2^n)

        Level 0:          ●                    1
        Level 1:        ●   ●                  2
        Level 2:      ●  ● ●  ●               4
        Level 3:    ● ● ● ● ● ● ● ●          8
                              ...
        Level n:   ●●●●●●●●●●●●●●●●●         ~2^n
```

---

## Fibonacci Variants

| Variant | Description | Modification |
|---------|-------------|-------------|
| Tribonacci | T(n) = T(n-1)+T(n-2)+T(n-3) | 3 base cases |
| Climbing stairs | Ways to reach step n (1 or 2 steps) | Same as fib! |
| Fibonacci modulo | F(n) % M | Apply mod at each step |
| Negative Fibonacci | F(-n) | F(-n) = (-1)^(n+1) × F(n) |

---

## Real-World Applications

- **Nature**: Flower petals, pine cones, shell spirals
- **Algorithm analysis**: Worst case of Euclidean GCD
- **Data structures**: Fibonacci heaps
- **Competitive programming**: Pisano period (Fibonacci mod)
- **Dynamic programming**: Gateway to understanding DP

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Base cases | F(0)=0, F(1)=1 |
| Recursive case | F(n) = F(n-1) + F(n-2) |
| Naive time | O(2^n) — exponential |
| Optimized time | O(n) with memo/iteration |
| Key insight | Overlapping subproblems → memoize |
| Pattern type | Binary recursion |
| Connection | Leads directly to Dynamic Programming |

---

## Quick Revision Questions

1. **Why does naive recursive Fibonacci have** O(2^n) time complexity?
2. **What are overlapping subproblems?** Show with fib(5).
3. **How does memoization fix** the exponential blowup?
4. **What is the space complexity** of iterative Fibonacci? Why?
5. **How is the climbing stairs problem** related to Fibonacci?
6. **Trace the iterative solution** for fib(7) step by step.

---

[← Previous: Factorial](01-factorial.md) | [Next: Sum of Array →](03-sum-of-array.md)

[← Back to README](../README.md)

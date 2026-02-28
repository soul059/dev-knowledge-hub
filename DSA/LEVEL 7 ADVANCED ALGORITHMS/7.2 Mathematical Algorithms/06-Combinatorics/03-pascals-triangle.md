# Chapter 3: Pascal's Triangle

[← Previous: Combinations](02-combinations.md) | [Back to README](../README.md) | [Next: nCr Computation →](04-ncr-computation.md)

---

## Overview

**Pascal's Triangle** is a triangular array where each entry is the sum of the two entries directly above it. Row $n$, position $r$ gives $\binom{n}{r}$. It is one of the most elegant structures in combinatorics with deep connections to algebra, number theory, and probability.

---

## 3.1 Construction

```
  Row 0:               1
  Row 1:             1   1
  Row 2:           1   2   1
  Row 3:         1   3   3   1
  Row 4:       1   4   6   4   1
  Row 5:     1   5  10  10   5   1
  Row 6:   1   6  15  20  15   6   1
  Row 7: 1   7  21  35  35  21   7   1

  Rule: pascal[n][r] = pascal[n-1][r-1] + pascal[n-1][r]
  Base: pascal[n][0] = pascal[n][n] = 1
```

```
  How the rule works visually:

        C(n-1, r-1)   C(n-1, r)
              ↘         ↙
              C(n, r)

  Example:   C(4,1) + C(4,2) = 4 + 6 = 10 = C(5,2) ✓
```

---

## 3.2 Building Pascal's Triangle (Code)

```
FUNCTION buildPascal(n):
    triangle = 2D array of size (n+1) × (n+1)
    FOR i = 0 TO n:
        triangle[i][0] = 1
        triangle[i][i] = 1
        FOR j = 1 TO i - 1:
            triangle[i][j] = triangle[i-1][j-1] + triangle[i-1][j]
    RETURN triangle

Time:  O(n²)
Space: O(n²)
```

### Space-Optimized (Single Row at a Time)

```
FUNCTION buildRow(n):
    row = array of size n+1, all zeros
    row[0] = 1
    FOR i = 1 TO n:
        // Update RIGHT to LEFT to avoid overwriting
        FOR j = i DOWNTO 1:
            row[j] = row[j] + row[j-1]
    RETURN row

  Trace for n = 4:
  Start:   [1, 0, 0, 0, 0]
  i=1:     [1, 1, 0, 0, 0]
  i=2:     [1, 2, 1, 0, 0]
  i=3:     [1, 3, 3, 1, 0]
  i=4:     [1, 4, 6, 4, 1]  ← Row 4

Time:  O(n²)
Space: O(n)
```

---

## 3.3 Binomial Theorem

```
  (a + b)ⁿ = Σ C(n, r) × aⁿ⁻ʳ × bʳ    (r = 0 to n)
  
  ┌────────────────────────────────────────────────────────┐
  │  (a+b)⁰ = 1                                          │
  │  (a+b)¹ = a + b                                      │
  │  (a+b)² = a² + 2ab + b²                              │
  │  (a+b)³ = a³ + 3a²b + 3ab² + b³                     │
  │  (a+b)⁴ = a⁴ + 4a³b + 6a²b² + 4ab³ + b⁴            │
  └────────────────────────────────────────────────────────┘
       Coefficients match Pascal's rows!

  Special substitutions:
  a=1, b=1:  2ⁿ = Σ C(n,r)           ← sum of row n
  a=1, b=-1: 0  = Σ (-1)ʳ C(n,r)     ← alternating sum = 0
```

---

## 3.4 Hidden Patterns in Pascal's Triangle

```
  ┌─────────────────────────────────────────────────────────┐
  │  Pattern             │  Where in Triangle               │
  ├──────────────────────┼─────────────────────────────────-┤
  │  Natural numbers     │  Column 1: 1, 2, 3, 4, ...      │
  │  Triangular numbers  │  Column 2: 1, 3, 6, 10, ...     │
  │  Tetrahedral numbers │  Column 3: 1, 4, 10, 20, ...    │
  │  Powers of 2         │  Row sums: 1, 2, 4, 8, 16, ...  │
  │  Fibonacci numbers   │  Diagonal sums!                  │
  │  Powers of 11        │  Rows: 1, 11, 121, 1331, ...    │
  └──────────────────────┴──────────────────────────────────┘
  
  Fibonacci via diagonals:
  
          1
         1  1
        1  2  1
       1  3  3  1
      1  4  6  4  1
     1  5 10 10  5  1
     
  Sum along "/" diagonals:
  1, 1, 2, 3, 5, 8, 13 ... = Fibonacci!
```

---

## 3.5 Divisibility Pattern (Sierpiński Triangle)

```
  Replace entries by (entry mod 2):
  
  Row 0:               1
  Row 1:             1   1
  Row 2:           1   0   1
  Row 3:         1   1   1   1
  Row 4:       1   0   0   0   1
  Row 5:     1   1   0   0   1   1
  Row 6:   1   0   1   0   1   0   1
  Row 7: 1   1   1   1   1   1   1   1

  The pattern of 1s forms a fractal — the Sierpiński triangle!
  
  Kummer's Theorem: C(m+n, m) is odd iff
  there are no carries when adding m and n in binary.
```

---

## 3.6 Hockey Stick Identity

```
  C(r,r) + C(r+1,r) + C(r+2,r) + ... + C(n,r) = C(n+1, r+1)
  
  Visually in Pascal's triangle:
  
  1
  1  1
  1  2  1
  1  3  3   1
  1  4  6   4   1
  1  5 [10] 10   5   1
  1  6  15 [20]  15   6   1
  1  7  21  35  [35]  21   7  1
  1  8  28  56   70  [56]  28  8  1
  
  Summing along a diagonal: 1+3+6+10+15+21 = 56
  That equals C(n+1, r+1) at the "handle" of the hockey stick ✓
  
  Example: C(2,2)+C(3,2)+C(4,2)+C(5,2)+C(6,2)+C(7,2)
         = 1 + 3 + 6 + 10 + 15 + 21 = 56 = C(8,3) ✓
```

---

## 3.7 Vandermonde's Identity

```
  C(m+n, r) = Σ C(m, k) × C(n, r-k)   for k = 0 to r
  
  Interpretation: Choosing r items from (m+n) items split into
  two groups of m and n. Choose k from first, r-k from second.
  
  Example: C(4+3, 3)
  = C(4,0)×C(3,3) + C(4,1)×C(3,2) + C(4,2)×C(3,1) + C(4,3)×C(3,0)
  = 1×1 + 4×3 + 6×3 + 4×1
  = 1 + 12 + 18 + 4
  = 35 = C(7,3) ✓
```

---

## 3.8 Applications in Competitive Programming

### Application 1: Counting Subsets

```
  Count subsets of {1..n} of size exactly k: C(n, k)
  Count all subsets: 2ⁿ
  Count subsets of even size: 2ⁿ⁻¹
  Count subsets of odd size: 2ⁿ⁻¹
```

### Application 2: Multinomial Coefficients

```
  Ways to arrange n items with n₁ of type 1, n₂ of type 2, ...:
  
  n! / (n₁! × n₂! × ... × nₖ!)
  
  Example: "MISSISSIPPI"
  n=11, n_M=1, n_I=4, n_S=4, n_P=2
  = 11! / (1! × 4! × 4! × 2!) = 34,650
```

---

## 3.9 Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Build full triangle to row n | O(n²) | O(n²) |
| Build single row n | O(n) | O(n) |
| Query C(n,r) from built triangle | O(1) | — |
| Space-optimized row build | O(n²) | O(n) |

---

## Summary Table

| Property | Identity |
|----------|----------|
| Pascal's Rule | C(n,r) = C(n-1,r-1) + C(n-1,r) |
| Row Sum | Σ C(n,r) = 2ⁿ |
| Alternating Sum | Σ (-1)ʳ C(n,r) = 0 |
| Hockey Stick | Σ C(i,r) for i=r..n = C(n+1,r+1) |
| Vandermonde | C(m+n,r) = Σ C(m,k)C(n,r-k) |
| Binomial Theorem | (a+b)ⁿ = Σ C(n,r) aⁿ⁻ʳ bʳ |
| Fibonacci link | Diagonal sums = Fibonacci numbers |
| Sierpiński | C(n,r) mod 2 forms fractal pattern |

---

## Quick Revision Questions

1. **Build** Pascal's triangle up to row 6 by hand.
2. **What is** the sum of all entries in row 10?
3. **Verify** the Hockey Stick identity for r=2, n=5.
4. **Expand** (x+y)⁵ using the Binomial Theorem.
5. **Why does** the space-optimized approach iterate right to left?
6. **Prove** that diagonal sums give Fibonacci numbers.

---

[← Previous: Combinations](02-combinations.md) | [Back to README](../README.md) | [Next: nCr Computation →](04-ncr-computation.md)

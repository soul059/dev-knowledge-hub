# Chapter 6.4: Rank and Systems of Equations

[← Previous: Properties of Rank](03-properties-of-rank.md) | [Back to README](../README.md) | [Next: Unit 7 →](../07-Systems-of-Linear-Equations/README.md)

---

## 📚 Chapter Overview

The rank of a matrix is the key to understanding whether a system of linear equations has solutions, and if so, how many. This chapter establishes the fundamental connection between rank and the solvability of linear systems.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Use rank to determine if a system has solutions
- Distinguish between unique and infinite solutions
- Apply the rank condition for consistency
- Find the number of free variables in a solution

---

## 1. The Augmented Matrix

### Definition

For a system $Ax = b$, the **augmented matrix** is:

$$[A|b] = \left[\begin{array}{ccc|c} a_{11} & a_{12} & a_{13} & b_1 \\ a_{21} & a_{22} & a_{23} & b_2 \\ a_{31} & a_{32} & a_{33} & b_3 \end{array}\right]$$

```
┌─────────────────────────────────────────────────────────────────┐
│                  AUGMENTED MATRIX                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   System:          x + 2y + 3z = 9                              │
│                   2x + 3y + z = 8                               │
│                   3x + y + 2z = 7                               │
│                                                                  │
│   Coefficient Matrix A:    Augmented Matrix [A|b]:              │
│                                                                  │
│   ┌─────────────┐          ┌─────────────┬───┐                  │
│   │ 1   2   3   │          │ 1   2   3   │ 9 │                  │
│   │ 2   3   1   │          │ 2   3   1   │ 8 │                  │
│   │ 3   1   2   │          │ 3   1   2   │ 7 │                  │
│   └─────────────┘          └─────────────┴───┘                  │
│                                                                  │
│   rank(A) = ?              rank([A|b]) = ?                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. The Rank Condition for Consistency

### Fundamental Theorem (Rouché-Capelli Theorem)

> A system of linear equations Ax = b is **consistent** (has solutions) if and only if:
> $$\text{rank}(A) = \text{rank}([A|b])$$

### Interpretation

| Condition | Meaning |
|-----------|---------|
| rank(A) = rank([A\|b]) | System is consistent (has solutions) |
| rank(A) < rank([A\|b]) | System is inconsistent (no solutions) |

Note: It's always true that $\text{rank}(A) \leq \text{rank}([A|b])$.

---

## 3. Types of Solutions

### Complete Classification

For a system with n unknowns:

```
┌─────────────────────────────────────────────────────────────────┐
│              SOLUTION CLASSIFICATION                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    rank(A) vs rank([A|b])                        │
│                           │                                      │
│           ┌───────────────┼───────────────┐                      │
│           ▼               ▼               ▼                      │
│      rank(A) =       rank(A) =       rank(A) <                  │
│      rank([A|b])     rank([A|b])     rank([A|b])                │
│           │               │               │                      │
│           ▼               ▼               ▼                      │
│      CONSISTENT      CONSISTENT      INCONSISTENT               │
│           │               │          (No Solution)              │
│     ┌─────┴─────┐         │                                     │
│     ▼           ▼         │                                     │
│ rank = n    rank < n      │                                     │
│     │           │         │                                     │
│     ▼           ▼         ▼                                     │
│  UNIQUE    INFINITE                                             │
│ SOLUTION   SOLUTIONS                                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Summary Table

| rank(A) | rank([A\|b]) | n (unknowns) | Solution |
|---------|--------------|--------------|----------|
| r | r | r | Unique solution |
| r | r | n > r | Infinite solutions (n-r free variables) |
| r | r+1 | any | No solution |

---

## 4. Number of Free Variables

### Formula

When a consistent system has rank r with n unknowns:

$$\text{Number of free variables} = n - r = n - \text{rank}(A)$$

This is exactly the nullity of A!

### Example

System with 5 unknowns and rank(A) = 3:
- Free variables = 5 - 3 = 2
- Solution depends on 2 parameters

---

## 5. Worked Examples

### Example 1: Unique Solution

Determine the type of solution for:
$$\begin{cases} x + y + z = 6 \\ x - y + z = 2 \\ 2x + y - z = 1 \end{cases}$$

**Form augmented matrix and reduce:**

$$[A|b] = \left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 1 & -1 & 1 & 2 \\ 2 & 1 & -1 & 1 \end{array}\right]$$

$R_2 \to R_2 - R_1$, $R_3 \to R_3 - 2R_1$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & -2 & 0 & -4 \\ 0 & -1 & -3 & -11 \end{array}\right]$$

$R_3 \to R_3 - \frac{1}{2}R_2$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & -2 & 0 & -4 \\ 0 & 0 & -3 & -9 \end{array}\right]$$

**Analysis:**
- rank(A) = 3 (three pivots)
- rank([A|b]) = 3
- n = 3 unknowns

Since rank(A) = rank([A|b]) = n = 3 → **Unique solution**

---

### Example 2: Infinite Solutions

Determine the type of solution for:
$$\begin{cases} x + 2y - z = 1 \\ 2x + 4y - 2z = 2 \\ x + 3y + z = 4 \end{cases}$$

**Reduce:**

$$[A|b] = \left[\begin{array}{ccc|c} 1 & 2 & -1 & 1 \\ 2 & 4 & -2 & 2 \\ 1 & 3 & 1 & 4 \end{array}\right]$$

$R_2 \to R_2 - 2R_1$, $R_3 \to R_3 - R_1$:

$$\left[\begin{array}{ccc|c} 1 & 2 & -1 & 1 \\ 0 & 0 & 0 & 0 \\ 0 & 1 & 2 & 3 \end{array}\right]$$

Swap $R_2 \leftrightarrow R_3$:

$$\left[\begin{array}{ccc|c} 1 & 2 & -1 & 1 \\ 0 & 1 & 2 & 3 \\ 0 & 0 & 0 & 0 \end{array}\right]$$

**Analysis:**
- rank(A) = 2 (two non-zero rows)
- rank([A|b]) = 2
- n = 3 unknowns

Since rank(A) = rank([A|b]) = 2 < 3 → **Infinite solutions**

Free variables = 3 - 2 = 1

---

### Example 3: No Solution

Determine the type of solution for:
$$\begin{cases} x + 2y = 3 \\ 2x + 4y = 7 \end{cases}$$

**Reduce:**

$$[A|b] = \left[\begin{array}{cc|c} 1 & 2 & 3 \\ 2 & 4 & 7 \end{array}\right]$$

$R_2 \to R_2 - 2R_1$:

$$\left[\begin{array}{cc|c} 1 & 2 & 3 \\ 0 & 0 & 1 \end{array}\right]$$

**Analysis:**
- rank(A) = 1 (one pivot in A columns)
- rank([A|b]) = 2 (pivot in augmented column!)

Since rank(A) = 1 < rank([A|b]) = 2 → **No solution**

The second row says $0 = 1$, which is impossible.

---

## 6. Geometric Interpretation

### In Two Dimensions (2 variables)

| Rank Condition | Geometric Meaning |
|----------------|-------------------|
| rank = rank_aug = 2 | Two lines intersect at one point |
| rank = rank_aug = 1 | Two identical lines (infinite points) |
| rank = 1, rank_aug = 2 | Two parallel lines (no intersection) |

```
┌─────────────────────────────────────────────────────────────────┐
│           GEOMETRIC INTERPRETATION (2D)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Unique Solution         Infinite Solutions    No Solution     │
│   (Lines intersect)       (Same line)           (Parallel)      │
│                                                                  │
│        ╲   ╱                   ╱                    ╱            │
│         ╲ ╱                   ╱                    ╱             │
│          ╳                   ╱                    ╱              │
│         ╱ ╲                 ╱                    ╱               │
│        ╱   ╲               ╱                   ╱                 │
│                                                                  │
│   rank(A)=rank([A|b])=2   rank(A)=rank([A|b])=1  rank(A)=1     │
│                                                  rank([A|b])=2  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### In Three Dimensions (3 variables)

Each equation represents a plane:
- Unique solution: Three planes meet at a point
- Infinite solutions (line): Three planes meet along a line
- Infinite solutions (plane): All three are the same plane
- No solution: Planes don't share a common point

---

## 7. Homogeneous Systems

### Special Case: Ax = 0

For homogeneous systems (b = 0):
- Always consistent (x = 0 is always a solution)
- rank(A) = rank([A|0])

| Condition | Solution |
|-----------|----------|
| rank(A) = n | Only trivial solution (x = 0) |
| rank(A) < n | Infinite solutions (including non-trivial) |

### Key Insight

For homogeneous systems with n unknowns and m equations:

If m < n (more unknowns than equations):
- rank(A) ≤ m < n
- Therefore, **infinite solutions** are guaranteed

---

## 8. Finding Rank Efficiently

### Strategy

1. Form augmented matrix [A|b]
2. Row reduce to echelon form
3. Count non-zero rows in A portion → rank(A)
4. Check if any row has pattern [0 0 ... 0 | k] where k ≠ 0
   - If yes: rank([A|b]) > rank(A) → inconsistent
   - If no: ranks are equal → consistent

---

## 9. Parameter-Dependent Systems

### Example

For what value of k does the system have:
(a) No solution
(b) Unique solution
(c) Infinite solutions

$$\begin{cases} x + y + z = 6 \\ x + 2y + 3z = 10 \\ x + 2y + kz = 10 \end{cases}$$

**Reduce:**

$$[A|b] = \left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 1 & 2 & 3 & 10 \\ 1 & 2 & k & 10 \end{array}\right]$$

$R_2 \to R_2 - R_1$, $R_3 \to R_3 - R_1$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & 2 & 4 \\ 0 & 1 & k-1 & 4 \end{array}\right]$$

$R_3 \to R_3 - R_2$:

$$\left[\begin{array}{ccc|c} 1 & 1 & 1 & 6 \\ 0 & 1 & 2 & 4 \\ 0 & 0 & k-3 & 0 \end{array}\right]$$

**Analysis:**
- If $k \neq 3$: rank(A) = 3, rank([A|b]) = 3 → **Unique solution**
- If $k = 3$: Third row becomes [0 0 0 | 0]
  - rank(A) = 2, rank([A|b]) = 2 → **Infinite solutions**

No value of k gives "no solution" for this system.

---

## 📊 Summary: Decision Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│                  SOLUTION DECISION PROCESS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Step 1: Form augmented matrix [A|b]                           │
│                    ↓                                             │
│   Step 2: Row reduce to echelon form                            │
│                    ↓                                             │
│   Step 3: Find rank(A) and rank([A|b])                          │
│                    ↓                                             │
│   Step 4: Compare ranks                                          │
│           ┌───────┴───────┐                                      │
│           ↓               ↓                                      │
│    rank(A) < rank([A|b])  rank(A) = rank([A|b])                 │
│           ↓               ↓                                      │
│     NO SOLUTION     Compare with n                               │
│                     ┌─────┴─────┐                                │
│                     ↓           ↓                                │
│              rank = n      rank < n                              │
│                     ↓           ↓                                │
│              UNIQUE       INFINITE                               │
│             SOLUTION      SOLUTIONS                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## ❓ Quick Revision Questions

1. **What is the condition for a system to be consistent?**
   <details>
   <summary>Click for Answer</summary>
   rank(A) = rank([A|b])
   </details>

2. **If rank(A) = rank([A|b]) = n, how many solutions?**
   <details>
   <summary>Click for Answer</summary>
   Exactly one (unique solution)
   </details>

3. **System has 4 unknowns and rank 2. How many free variables?**
   <details>
   <summary>Click for Answer</summary>
   4 - 2 = 2 free variables
   </details>

4. **What does a row [0 0 0 | 5] indicate?**
   <details>
   <summary>Click for Answer</summary>
   Inconsistency (0 = 5 is impossible), so no solution exists.
   </details>

5. **Can a homogeneous system Ax = 0 be inconsistent?**
   <details>
   <summary>Click for Answer</summary>
   No, x = 0 is always a solution.
   </details>

6. **If a 3×3 system has rank 2, what are the possibilities?**
   <details>
   <summary>Click for Answer</summary>
   Either infinite solutions (if consistent) or no solution (if inconsistent).
   </details>

---

## 🔗 Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Properties of Rank](03-properties-of-rank.md) | [Unit 6: Rank](./README.md) | [Unit 7: Systems →](../07-Systems-of-Linear-Equations/README.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*

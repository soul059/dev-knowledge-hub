# Unit 6: Rank of a Matrix

[← Previous Unit](../05-Adjoint-and-Inverse/README.md) | [Back to Main](../README.md) | [Next Unit →](../07-Systems-of-Linear-Equations/README.md)

---

## 📚 Unit Overview

The rank of a matrix is a fundamental concept that captures the "essential dimension" of a matrix. It connects matrix structure to the solvability of linear systems and provides powerful tools for analyzing systems of equations.

---

## 🎯 Unit Objectives

By the end of this unit, you will be able to:
- Reduce matrices to echelon and reduced echelon form
- Calculate the rank of any matrix
- Apply properties of rank in problem solving
- Use rank to determine system consistency

---

## 📖 Chapters

### [6.1 Echelon Form](01-echelon-form.md)
- Row Echelon Form (REF)
- Reduced Row Echelon Form (RREF)
- Gaussian and Gauss-Jordan elimination
- Pivot positions

### [6.2 Definition and Calculation of Rank](02-definition-of-rank.md)
- Definition of rank
- Row reduction method
- Minor method
- Special cases

### [6.3 Properties of Rank](03-properties-of-rank.md)
- Rank bounds and transpose property
- Rank of matrix products
- Sylvester's inequality
- Rank-Nullity theorem

### [6.4 Rank and Systems of Equations](04-rank-and-systems.md)
- Consistency conditions
- Rouché-Capelli theorem
- Solution types based on rank
- Geometric interpretation

---

## 🗺️ Concept Map

```
┌─────────────────────────────────────────────────────────────────┐
│                    RANK OF A MATRIX                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌───────────────────────────────────────────────────────┐     │
│   │                Matrix A (m×n)                         │     │
│   └───────────────────────────────────────────────────────┘     │
│                         │                                        │
│                         ↓                                        │
│   ┌───────────────────────────────────────────────────────┐     │
│   │           Row Reduce to Echelon Form                  │     │
│   └───────────────────────────────────────────────────────┘     │
│                         │                                        │
│           ┌─────────────┼─────────────┐                         │
│           ↓             ↓             ↓                         │
│   Count Non-zero    Count Pivot    Find Largest                 │
│       Rows          Positions      Non-zero Minor               │
│           │             │             │                         │
│           └─────────────┴─────────────┘                         │
│                         │                                        │
│                         ↓                                        │
│             ┌─────────────────────┐                             │
│             │    RANK = r        │                              │
│             └─────────────────────┘                             │
│                         │                                        │
│           ┌─────────────┴─────────────┐                         │
│           ↓                           ↓                         │
│   ┌───────────────┐           ┌───────────────┐                 │
│   │ r = n         │           │ r < n         │                 │
│   │ Full column   │           │ Has null      │                 │
│   │ rank          │           │ space         │                 │
│   └───────────────┘           └───────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Key Formulas

| Concept | Formula |
|---------|---------|
| Rank bound | $0 \leq \text{rank}(A) \leq \min(m, n)$ |
| Transpose | $\text{rank}(A^T) = \text{rank}(A)$ |
| Product bound | $\text{rank}(AB) \leq \min(\text{rank}(A), \text{rank}(B))$ |
| Sylvester's lower | $\text{rank}(AB) \geq \text{rank}(A) + \text{rank}(B) - n$ |
| Rank-Nullity | $\text{rank}(A) + \text{nullity}(A) = n$ |
| Consistency | $\text{rank}(A) = \text{rank}([A|b])$ |

---

## 📊 Rank and Systems Summary

| rank(A) | rank([A\|b]) | n | Solution |
|---------|--------------|---|----------|
| r | r | r | Unique |
| r | r | n > r | Infinite (n-r parameters) |
| r | r + 1 | any | None |

---

## 🔗 Navigation

| Previous Unit | Main Index | Next Unit |
|---------------|------------|-----------|
| [← Unit 5: Adjoint & Inverse](../05-Adjoint-and-Inverse/README.md) | [Course Home](../README.md) | [Unit 7: Systems →](../07-Systems-of-Linear-Equations/README.md) |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*

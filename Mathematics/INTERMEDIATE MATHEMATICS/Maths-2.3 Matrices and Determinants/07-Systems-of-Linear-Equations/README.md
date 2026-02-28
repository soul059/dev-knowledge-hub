# Unit 7: Systems of Linear Equations

[← Previous Unit](../06-Rank-of-Matrix/README.md) | [Back to Main](../README.md)

---

## 📚 Unit Overview

This final unit applies all the concepts learned about matrices, determinants, and rank to solve systems of linear equations systematically. We cover both theoretical foundations and practical solution methods.

---

## 🎯 Unit Objectives

By the end of this unit, you will be able to:
- Classify systems as homogeneous or non-homogeneous
- Solve systems using matrix methods (Gaussian elimination)
- Apply Cramer's rule effectively
- Find general solutions with parameters
- Understand solution spaces geometrically

---

## 📖 Chapters

### [7.1 Homogeneous Systems](01-homogeneous-systems.md)
- Ax = 0 systems
- Trivial and non-trivial solutions
- Condition for non-trivial solutions
- Solution space structure

### [7.2 Non-Homogeneous Systems](02-non-homogeneous-systems.md)
- Ax = b systems
- Gaussian elimination method
- Gauss-Jordan elimination
- Back substitution

### [7.3 General Solutions and Parameters](03-general-solutions.md)
- Particular and general solutions
- Parameterized solutions
- Solution verification
- Summary and applications

---

## 🗺️ Concept Map

```
┌─────────────────────────────────────────────────────────────────┐
│              SYSTEMS OF LINEAR EQUATIONS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌───────────────────────────────────────────────────────┐     │
│   │              System: Ax = b                           │     │
│   └───────────────────────────────────────────────────────┘     │
│                         │                                        │
│           ┌─────────────┴─────────────┐                         │
│           ↓                           ↓                         │
│   ┌───────────────┐           ┌───────────────┐                 │
│   │ b = 0         │           │ b ≠ 0         │                 │
│   │ HOMOGENEOUS   │           │ NON-HOMOGENEOUS│                │
│   └───────────────┘           └───────────────┘                 │
│           │                           │                         │
│           ↓                           ↓                         │
│   Always consistent           May be consistent                  │
│   (x=0 is solution)          or inconsistent                    │
│           │                           │                         │
│           ↓                           ↓                         │
│   ┌───────────────┐           ┌───────────────┐                 │
│   │ rank(A) = n   │           │ rank(A) =     │                 │
│   │ Only x = 0    │           │ rank([A|b])   │                 │
│   │ (trivial)     │           │ Consistent    │                 │
│   └───────────────┘           └───────────────┘                 │
│           │                           │                         │
│   ┌───────────────┐           ┌───────────────┐                 │
│   │ rank(A) < n   │           │ rank(A) <     │                 │
│   │ Infinite      │           │ rank([A|b])   │                 │
│   │ solutions     │           │ Inconsistent  │                 │
│   └───────────────┘           └───────────────┘                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Key Formulas

| Concept | Formula/Condition |
|---------|-------------------|
| Homogeneous system | $Ax = 0$ |
| Non-homogeneous system | $Ax = b$ (b ≠ 0) |
| Consistency condition | rank(A) = rank([A\|b]) |
| Unique solution | rank(A) = rank([A\|b]) = n |
| Infinite solutions | rank(A) = rank([A\|b]) < n |
| No solution | rank(A) < rank([A\|b]) |
| Free variables | n - rank(A) |
| General solution | $x = x_p + x_h$ |

---

## 📊 Solution Methods Comparison

| Method | Applicable When | Advantages | Limitations |
|--------|-----------------|------------|-------------|
| Cramer's Rule | Square system, \|A\| ≠ 0 | Direct formula | Only unique solutions |
| Gaussian Elimination | Any system | Universal method | Requires many steps |
| Matrix Inverse | Square system, A invertible | x = A⁻¹b | Must find inverse |
| Rank Analysis | Any system | Shows solution type | Needs further solving |

---

## 🧪 Practice Problems Preview

1. Solve homogeneous systems and find solution space
2. Apply Gaussian elimination systematically
3. Determine consistency using rank
4. Express solutions in parametric form
5. Verify solutions by substitution

---

## 📋 Prerequisites Check

Before starting this unit, ensure you can:
- ☑️ Perform elementary row operations
- ☑️ Calculate determinants
- ☑️ Find matrix inverse
- ☑️ Determine rank of a matrix
- ☑️ Reduce to echelon form

---

## 🔗 Navigation

| Previous Unit | Main Index | Next |
|---------------|------------|------|
| [← Unit 6: Rank](../06-Rank-of-Matrix/README.md) | [Course Home](../README.md) | Course Complete! |

---

*© 2026 Matrices and Determinants Study Notes. All rights reserved.*

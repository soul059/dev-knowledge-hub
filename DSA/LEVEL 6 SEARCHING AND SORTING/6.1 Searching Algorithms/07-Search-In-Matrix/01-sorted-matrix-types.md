# Chapter 1: Types of Sorted Matrices

[← Previous Unit: BS on Answer Practice](../06-Binary-Search-On-Answer/06-practice-problems.md) | [Next: Row-Column Sorted Matrix →](02-row-col-sorted.md)

---

## Overview

Searching in a 2D matrix is a common interview topic. The approach depends on **how** the matrix is sorted. This chapter introduces the different types.

---

## Type 1: Fully Sorted (Row-Major Order)

```
   Each row is sorted, AND the first element of each row is
   greater than the last element of the previous row.
   
   ┌────┬────┬────┬────┐
   │  1 │  3 │  5 │  7 │
   ├────┼────┼────┼────┤
   │ 10 │ 11 │ 16 │ 20 │
   ├────┼────┼────┼────┤
   │ 23 │ 30 │ 34 │ 60 │
   └────┴────┴────┴────┘
   
   Reading left-to-right, top-to-bottom: 1,3,5,7,10,11,16,20,23,30,34,60
   → Fully sorted! Can treat as a 1D array.
   
   Search: O(log(m × n)) using standard binary search
```

---

## Type 2: Row-wise and Column-wise Sorted

```
   Each row is sorted left to right.
   Each column is sorted top to bottom.
   But first element of next row may NOT be > last of previous row.
   
   ┌────┬────┬────┬────┐
   │  1 │  4 │  7 │ 11 │
   ├────┼────┼────┼────┤
   │  2 │  5 │  8 │ 12 │
   ├────┼────┼────┼────┤
   │  3 │  6 │  9 │ 16 │
   ├────┼────┼────┼────┤
   │ 10 │ 13 │ 14 │ 17 │
   └────┴────┴────┴────┘
   
   Row 1: 1 < 4 < 7 < 11  ✓
   Col 1: 1 < 2 < 3 < 10  ✓
   But 11 > 2 (end of row 1 > start of row 2)
   
   NOT a flattened sorted array!
   Search: O(m + n) using staircase search
```

---

## Type 3: Sorted Rows Only

```
   Each row is sorted, but columns are not.
   
   ┌────┬────┬────┬────┐
   │  1 │  5 │  9 │ 11 │
   ├────┼────┼────┼────┤
   │  3 │  4 │  6 │ 12 │
   ├────┼────┼────┼────┤
   │  7 │  8 │ 10 │ 15 │
   └────┴────┴────┴────┘
   
   Col 1: 1, 3, 7 → sorted ✓ (coincidence here)
   But in general columns may not be sorted.
   
   Search: O(m × log n) — binary search each row
```

---

## Comparison Table

| Type | Row Sorted | Col Sorted | Flat Sorted | Best Search |
|------|-----------|-----------|------------|-------------|
| Fully Sorted | ✓ | ✓ | ✓ | O(log(mn)) |
| Row+Col Sorted | ✓ | ✓ | ✗ | O(m + n) |
| Rows Only | ✓ | ✗ | ✗ | O(m log n) |

---

## How to Identify the Type

```
   Check: Is matrix[row][last] < matrix[row+1][0]?
   
   YES for all rows → Type 1 (Fully Sorted)
     → Flatten and binary search
   
   NO, but each row AND column sorted → Type 2
     → Staircase search
   
   NO, only rows sorted → Type 3
     → Binary search per row
```

---

## Visual: How Each Type Looks

```
   Type 1 (Fully Sorted):         Type 2 (Row+Col Sorted):
   
   Values increase →              Values increase → and ↓
   ┌─────────────┐                ┌─────────────┐
   │  1  2  3  4 │                │  1  4  7 11 │
   │  5  6  7  8 │                │  2  5  8 12 │  ← overlap
   │  9 10 11 12 │                │  3  6  9 16 │     between rows
   └─────────────┘                └─────────────┘
   No overlap between rows        Rows overlap!
```

---

## Quick Revision Questions

1. **How do you distinguish Type 1 from Type 2?**
2. **Why can Type 1 be treated as a 1D sorted array?**
3. **What is the best time complexity for each type?**
4. **Can you use the staircase search on Type 1? What complexity?**
5. **Given a matrix, what property check tells you it's Type 1?**

---

[← Previous Unit: BS on Answer Practice](../06-Binary-Search-On-Answer/06-practice-problems.md) | [Next: Row-Column Sorted Matrix →](02-row-col-sorted.md)

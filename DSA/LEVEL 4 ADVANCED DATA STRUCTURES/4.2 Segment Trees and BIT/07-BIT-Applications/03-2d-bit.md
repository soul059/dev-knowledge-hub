# 2D BIT (Fenwick Tree)

## Chapter Overview

2D BIT extends the concept to two dimensions, supporting efficient point updates and rectangle sum queries on a 2D grid. It's simpler than a 2D segment tree and suitable for many matrix problems.

---

## Concept

```
2D BIT: a BIT of BITs

If 1D BIT supports:
  update(x, val)
  prefix_sum(x)

Then 2D BIT supports:
  update(x, y, val)           — add val to cell (x, y)
  prefix_sum(x, y)            — sum of rectangle (1,1) to (x,y)

Structure: B[x][y] where both dimensions use BIT indexing

B[x][y] covers:
  rows from (x - lowbit(x) + 1) to x
  columns from (y - lowbit(y) + 1) to y
```

---

## Operations

```
function update_2d(x, y, val):
    ix = x
    while ix <= n:
        iy = y
        while iy <= m:
            B[ix][iy] += val
            iy += iy & (-iy)
        ix += ix & (-ix)

function prefix_sum_2d(x, y):
    result = 0
    ix = x
    while ix > 0:
        iy = y
        while iy > 0:
            result += B[ix][iy]
            iy -= iy & (-iy)
        ix -= ix & (-ix)
    return result

Both operations: O(log n × log m)
  — outer loop: O(log n) for rows
  — inner loop: O(log m) for columns per row iteration
```

---

## Rectangle Sum Query

```
Sum of rectangle (r1, c1) to (r2, c2):

Using inclusion-exclusion on 2D prefix sums:

function rect_sum(r1, c1, r2, c2):
    return prefix_sum_2d(r2, c2)
         - prefix_sum_2d(r1-1, c2)
         - prefix_sum_2d(r2, c1-1)
         + prefix_sum_2d(r1-1, c1-1)

Visualizing inclusion-exclusion:

  ┌──────────────┬──────────┐
  │      A       │    B     │  r1-1
  ├──────────────┼──────────┤
  │      C       │    D     │  r2
  └──────────────┴──────────┘
       c1-1           c2

  rect_sum = prefix(r2,c2) - prefix(r1-1,c2) - prefix(r2,c1-1) + prefix(r1-1,c1-1)
           = (A+B+C+D) - (A+B) - (A+C) + A
           = D  ✓
```

---

## Worked Example

```
Grid (3×4):
    1  2  3  4
  ┌───────────────┐
1 │ 1  2  3  4   │
2 │ 5  6  7  8   │
3 │ 9  10 11 12  │
  └───────────────┘

Build 2D BIT by updating each cell:
  for r = 1 to 3:
    for c = 1 to 4:
      update_2d(r, c, grid[r][c])

Query: Sum of rectangle (2,2) to (3,3)
  = cells: 6 + 7 + 10 + 11 = 34

Using formula:
  prefix_sum_2d(3,3) = 1+2+3+5+6+7+9+10+11 = 54
  prefix_sum_2d(1,3) = 1+2+3 = 6
  prefix_sum_2d(3,1) = 1+5+9 = 15
  prefix_sum_2d(1,1) = 1

  rect_sum = 54 - 6 - 15 + 1 = 34  ✓
```

---

## Memory and Complexity

```
Memory: O(n × m) — same as the grid itself
  No 4× overhead like segment tree!

Time:
  Build:  O(n × m × log n × log m)
  Update: O(log n × log m)
  Query:  O(log n × log m)

For typical competitive programming:
  n, m ≤ 1000 → log n × log m ≈ 10 × 10 = 100
  Very fast in practice.

Comparison with 2D prefix sums:
  ┌───────────────┬──────────────┬──────────────┐
  │               │ 2D Prefix    │    2D BIT    │
  ├───────────────┼──────────────┼──────────────┤
  │ Build         │ O(n×m)       │ O(nm log n   │
  │               │              │   log m)     │
  │ Update        │ O(n×m) redo  │ O(log n      │
  │               │              │   log m)     │
  │ Query         │ O(1)         │ O(log n      │
  │               │              │   log m)     │
  └───────────────┴──────────────┴──────────────┘

  Use 2D prefix sums for static grids.
  Use 2D BIT when updates are needed.
```

---

## Extensions

```
1. 2D Range Update + Point Query
   Use 2D difference array in BIT:
   
   range_update(r1, c1, r2, c2, val):
     update_2d(r1, c1, +val)
     update_2d(r1, c2+1, -val)
     update_2d(r2+1, c1, -val)
     update_2d(r2+1, c2+1, +val)
   
   point_query(r, c):
     prefix_sum_2d(r, c)

2. 3D BIT
   Triple nested loops — same idea, O(log³ n)
   Practical for small n (e.g., n ≤ 100)

3. 2D BIT for counting
   Count points in rectangle — each point updates (x,y) by +1
   Rectangle count = rect_sum(r1, c1, r2, c2)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Structure | Nested BIT: outer for rows, inner for columns |
| Memory | O(n × m) |
| Point update | O(log n × log m) |
| Rectangle query | O(log n × log m) via inclusion-exclusion |
| Build | O(n × m × log n × log m) |
| When to use | Dynamic 2D grid with point updates and rectangle queries |
| vs 2D prefix | BIT for dynamic data; prefix sums for static |

---

## Quick Revision Questions

1. What is the update complexity of 2D BIT? *(O(log n × log m))*
2. How do you compute a rectangle sum? *(Inclusion-exclusion with 4 prefix_sum_2d calls)*
3. How much memory does 2D BIT use? *(O(n × m) — no overhead factor)*
4. How do you do range updates on a 2D BIT? *(Use 2D difference array: 4 point updates for each rectangle update)*
5. When should you use 2D BIT vs 2D prefix sums? *(2D BIT when updates are needed; 2D prefix sums for static data)*

---

[← Previous: Range Sum Applications](02-range-sum-applications.md) | [Next: BIT vs Segment Tree →](04-bit-vs-segment-tree.md)

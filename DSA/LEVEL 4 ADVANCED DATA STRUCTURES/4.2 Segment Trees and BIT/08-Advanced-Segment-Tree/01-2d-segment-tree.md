# 2D Segment Tree

## Chapter Overview

A 2D Segment Tree extends the concept to two dimensions — it supports range queries and updates on a 2D grid (matrix). This chapter covers the structure, operations, and trade-offs.

---

## Concept: Segment Tree of Segment Trees

```
For an n × m matrix, build a segment tree on ROWS,
where each node stores a SEGMENT TREE on COLUMNS.

Outer tree: segments over row ranges
Inner trees: segments over column ranges for each row range

                     rows [1..4]
                    /           \
            rows [1..2]       rows [3..4]
            /       \         /       \
        row [1]   row [2]  row [3]   row [4]

Each box above contains an inner segment tree over columns:

    row [1]:  cols [1..4] → inner segment tree
    rows [1..2]: cols [1..4] → merged inner segment tree
    rows [1..4]: cols [1..4] → merged inner segment tree
```

---

## Structure

```
For n = 4, m = 4 matrix:

tree[4*n][4*m]    // 2D array

tree[outer_node][inner_node] stores the aggregate for:
  - row range defined by outer_node
  - column range defined by inner_node

Total space: O(4n × 4m) = O(16nm) ≈ 16× the original matrix
```

---

## Build

```
build_y(outer_node, inner_node, ly, ry, lx, rx):
    // Build inner tree for the row range [lx..rx]
    if ly == ry:  // single column
        if lx == rx:  // single cell
            tree[outer_node][inner_node] = matrix[lx][ly]
        else:  // merge from children row-segments
            tree[outer_node][inner_node] = 
                merge(tree[2*outer_node][inner_node],
                      tree[2*outer_node+1][inner_node])
    else:
        mid = (ly + ry) / 2
        build_y(outer_node, 2*inner_node, ly, mid, lx, rx)
        build_y(outer_node, 2*inner_node+1, mid+1, ry, lx, rx)
        tree[outer_node][inner_node] = 
            merge(tree[outer_node][2*inner_node],
                  tree[outer_node][2*inner_node+1])

build_x(outer_node, lx, rx):
    if lx != rx:
        mid = (lx + rx) / 2
        build_x(2*outer_node, lx, mid)
        build_x(2*outer_node+1, mid+1, rx)
    build_y(outer_node, 1, 1, m, lx, rx)

// Call: build_x(1, 1, n)
```

---

## Point Update

```
update_y(outer_node, inner_node, ly, ry, col, lx, rx, val):
    if ly == ry:
        if lx == rx:
            tree[outer_node][inner_node] = val
        else:
            tree[outer_node][inner_node] = 
                merge(tree[2*outer_node][inner_node],
                      tree[2*outer_node+1][inner_node])
    else:
        mid = (ly + ry) / 2
        if col <= mid:
            update_y(outer_node, 2*inner_node, ly, mid, col, lx, rx, val)
        else:
            update_y(outer_node, 2*inner_node+1, mid+1, ry, col, lx, rx, val)
        tree[outer_node][inner_node] = 
            merge(tree[outer_node][2*inner_node],
                  tree[outer_node][2*inner_node+1])

update_x(outer_node, lx, rx, row, col, val):
    if lx != rx:
        mid = (lx + rx) / 2
        if row <= mid:
            update_x(2*outer_node, lx, mid, row, col, val)
        else:
            update_x(2*outer_node+1, mid+1, rx, row, col, val)
    update_y(outer_node, 1, 1, m, col, lx, rx, val)

// Call: update_x(1, 1, n, row, col, new_value)
// Time: O(log n × log m)
```

---

## Rectangle Query

```
Query: sum/min/max over rectangle [r1..r2, c1..c2]

query_y(outer_node, inner_node, ly, ry, c1, c2):
    if c1 > ry or c2 < ly:
        return identity
    if c1 <= ly and ry <= c2:
        return tree[outer_node][inner_node]
    mid = (ly + ry) / 2
    return merge(
        query_y(outer_node, 2*inner_node, ly, mid, c1, c2),
        query_y(outer_node, 2*inner_node+1, mid+1, ry, c1, c2))

query_x(outer_node, lx, rx, r1, r2, c1, c2):
    if r1 > rx or r2 < lx:
        return identity
    if r1 <= lx and rx <= r2:
        return query_y(outer_node, 1, 1, m, c1, c2)
    mid = (lx + rx) / 2
    return merge(
        query_x(2*outer_node, lx, mid, r1, r2, c1, c2),
        query_x(2*outer_node+1, mid+1, rx, r1, r2, c1, c2))

// Call: query_x(1, 1, n, r1, r2, c1, c2)
// Time: O(log n × log m)
```

---

## Worked Example

```
Matrix (3×3):                   Query: sum of [1..2, 2..3]
┌───┬───┬───┐                   
│ 1 │ 2 │ 3 │  row 1            Answer = 2 + 3 + 5 + 6 = 16
├───┼───┼───┤
│ 4 │ 5 │ 6 │  row 2            Outer tree visits row segments:
├───┼───┼───┤                     [1..2] → fully covered
│ 7 │ 8 │ 9 │  row 3            Inner tree for [1..2] queries cols [2..3]
└───┴───┴───┘                     → returns 2+3+5+6 = 16
```

---

## Complexity

```
            Build        Point Update    Rectangle Query
            ─────        ────────────    ───────────────
Time:       O(nm)        O(log n·log m)  O(log n·log m)
Space:      O(16nm)       —               —

Comparison with 2D BIT:
            Build        Point Update    Rectangle Query    Space
2D SegTree: O(nm)        O(log n·log m)  O(log n·log m)    16nm
2D BIT:     O(nm log²)   O(log n·log m)  O(log n·log m)    nm
```

---

## Limitations and Alternatives

```
Problem                          Solution
───────                          ────────
Memory (16nm too large)       → Use 2D BIT for sum/XOR
Need lazy propagation         → Very complex in 2D; consider other approaches
Sparse grid                   → Dynamic 2D Segment Tree (maps instead of arrays)
Only sum queries              → 2D BIT (simpler, faster)
Need min/max on rectangle     → 2D Segment Tree (BIT can't do this)
```

---

## Summary Table

| Aspect | 2D Segment Tree | 2D BIT |
|--------|----------------|--------|
| Operations | Any associative | Invertible only |
| Space | 16nm | nm |
| Code complexity | Very high | Moderate |
| Lazy support | Possible (complex) | Via tricks |
| Min/Max | Yes | No |
| Practical use | When BIT can't work | Default for sum |

---

## Quick Revision Questions

1. What is the core idea of a 2D segment tree? *(A segment tree of segment trees — outer tree on rows, inner tree on columns for each row range)*
2. What is the time complexity of a rectangle query? *(O(log n × log m))*
3. How much space does a 2D segment tree use? *(O(16nm) for an n×m matrix — 16× the original)*
4. When should you prefer 2D BIT over 2D segment tree? *(When the operation is invertible (sum, XOR) — it uses 16× less memory)*
5. Can you do lazy propagation in 2D? *(Yes, but it's very complex and rarely used)*

---

[← Previous: When to Use BIT](../07-BIT-Applications/06-when-to-use-bit.md) | [Next: Persistent Segment Tree →](02-persistent-segment-tree.md)

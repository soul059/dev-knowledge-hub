# Point Update

## Chapter Overview

A point update changes a single element in the array and propagates the change up through the tree. This chapter covers the algorithm, traces through examples, and analyzes the complexity.

---

## Update Algorithm

```
ALGORITHM: Point Update

Input:  Index i, new value val
Effect: Set A[i] = val, update all affected tree nodes

function update(node, start, end, index, value):
    // Base case: leaf node
    if start == end:
        A[index] = value
        tree[node] = value
        return

    mid = (start + end) / 2
    if index <= mid:
        update(2*node, start, mid, index, value)     // go left
    else:
        update(2*node+1, mid+1, end, index, value)   // go right
    
    // Re-merge after child update
    tree[node] = merge(tree[2*node], tree[2*node+1])

// Call: update(1, 0, n-1, index, value)
```

**Key difference from query**: We go to exactly ONE child (not both), because a point update affects only one leaf.

---

## Complete Trace: Sum Tree Update

```
Array A = [3, 1, 4, 1, 5, 9, 2, 6]    n = 8

Before update:
                  1:[0-7]=31
                 /          \
           2:[0-3]=9      3:[4-7]=22
           /    \          /      \
       4:[0-1]=4 5:[2-3]=5 6:[4-5]=14 7:[6-7]=8
       / \       / \       / \        / \
      8:3 9:1  10:4 11:1  12:5 13:9  14:2 15:6

Update: A[3] = 8  (was 1, diff = +7)
```

```
update(1, 0, 7, 3, 8)
  index 3 ≤ mid=3 → go LEFT
  
  ├── update(2, 0, 3, 3, 8)
  │   index 3 > mid=1 → go RIGHT
  │   
  │   ├── update(5, 2, 3, 3, 8)
  │   │   index 3 > mid=2 → go RIGHT
  │   │   
  │   │   ├── update(11, 3, 3, 3, 8)
  │   │   │   start == end → LEAF
  │   │   │   tree[11] = 8          ← UPDATE LEAF
  │   │   │   return
  │   │   │
  │   │   └── tree[5] = tree[10] + tree[11] = 4 + 8 = 12    ← re-merge
  │   │
  │   └── tree[2] = tree[4] + tree[5] = 4 + 12 = 16          ← re-merge
  │
  └── tree[1] = tree[2] + tree[3] = 16 + 22 = 38             ← re-merge
```

```
After update:
                  1:[0-7]=38    ← changed (31→38)
                 /          \
           2:[0-3]=16     3:[4-7]=22   ← unchanged
           /    \          /      \
       4:[0-1]=4 5:[2-3]=12 6:[4-5]=14 7:[6-7]=8
       / \       / \         ← 5 changed (5→12)
      8:3 9:1  10:4 11:8    ← 11 changed (1→8)

Changed nodes: 11 → 5 → 2 → 1  (path from leaf to root)
Unchanged: everything else
```

---

## Path from Leaf to Root

The update always follows a single path from the target leaf to the root:

```
n = 8, updating index 3:

Level 3 (leaf):   [3]         ← update value
Level 2:          [2-3]       ← re-merge children
Level 1:          [0-3]       ← re-merge children
Level 0 (root):   [0-7]       ← re-merge children

Path length = height = log₂(8) = 3
Nodes modified = height + 1 = 4
```

```
General pattern:

                    [root]         ← always updated
                   /      \
              [LEFT]      [RIGHT]  ← one updated, one unchanged
              ...
          [ancestor]               ← updated
          /        \
     [target]    [sibling]         ← only target path updated
      leaf
      
Nodes updated: O(log n)
Nodes unchanged: O(n) — the vast majority
```

---

## Increment Update Variant

Sometimes you want to ADD a value instead of replacing:

```
ALGORITHM: Increment Update

function incrementUpdate(node, start, end, index, delta):
    if start == end:
        A[index] += delta
        tree[node] += delta       // add, not replace
        return
    
    mid = (start + end) / 2
    if index <= mid:
        incrementUpdate(2*node, start, mid, index, delta)
    else:
        incrementUpdate(2*node+1, mid+1, end, index, delta)
    
    tree[node] = merge(tree[2*node], tree[2*node+1])

// For sum tree, you can optimize:
// tree[node] += delta  (instead of re-merging)
// This works because sum is linear.
```

**Optimization for sum tree**:
```
function incrementUpdate_optimized(node, start, end, index, delta):
    tree[node] += delta           // always add delta
    if start == end: return       // reached leaf
    
    mid = (start + end) / 2
    if index <= mid:
        incrementUpdate_optimized(2*node, start, mid, index, delta)
    else:
        incrementUpdate_optimized(2*node+1, mid+1, end, index, delta)

Note: This only works for SUM. For min/max, you must re-merge.
```

---

## Update for Min/Max Trees

For min/max trees, you MUST re-merge because changing a leaf might not change the parent's min/max:

```
Min tree before: 
         [0-3]=1
        /       \
    [0-1]=1    [2-3]=4
    / \        / \
   3   1      4   7

Update A[0] = 5 (was 3):

         [0-3]=1       ← stays 1 (min comes from A[1]=1)
        /       \
    [0-1]=1    [2-3]=4  ← stays 1
    / \
   5   1   ← A[0] changed, but min(5,1) still = 1

The parent didn't change even though a child did!
That's why we re-merge: tree[node] = min(tree[2*node], tree[2*node+1])
```

---

## Multiple Updates

```
Updates are independent — each takes O(log n):

Update A[0] = 10: path [0] → [0-1] → [0-3] → [0-7]    O(log n)
Update A[5] = 20: path [5] → [4-5] → [4-7] → [0-7]    O(log n)
Update A[3] = 15: path [3] → [2-3] → [0-3] → [0-7]    O(log n)

Total for k updates: O(k × log n)

Note: Updates cannot be batched (unlike lazy propagation for ranges).
Each update must complete before the next to maintain correctness.
```

---

## Iterative Point Update

For the iterative (bottom-up) segment tree:

```
ALGORITHM: Iterative Point Update (n = power of 2)

function update(index, value):
    pos = n + index                  // leaf position
    tree[pos] = value                // update leaf
    
    pos = pos / 2                    // move to parent
    while pos >= 1:
        tree[pos] = merge(tree[2*pos], tree[2*pos+1])
        pos = pos / 2

// Trace for n=8, update A[3]=8:
// pos = 8+3 = 11 → tree[11] = 8
// pos = 5 → tree[5] = merge(tree[10], tree[11]) = merge(4, 8) = 12
// pos = 2 → tree[2] = merge(tree[4], tree[5]) = merge(4, 12) = 16
// pos = 1 → tree[1] = merge(tree[2], tree[3]) = merge(16, 22) = 38
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Algorithm | Navigate to leaf, update it, re-merge up to root |
| Path traversed | Leaf → Root (single path) |
| Nodes visited | O(log n) |
| Nodes modified | O(log n) (same path) |
| Time complexity | O(log n) |
| Assignment vs Increment | Assignment replaces; Increment adds delta |
| Sum optimization | Can add delta on the way down (skip re-merge for sum) |
| Min/Max caveat | Must always re-merge — parent may not change |
| Iterative variant | Start at leaf position (n+i), walk up dividing by 2 |

---

## Quick Revision Questions

1. How many children does the update recurse into? *(Exactly one — the child containing the target index)*
2. What is the difference between assignment update and increment update? *(Assignment: tree[leaf] = value; Increment: tree[leaf] += delta)*
3. Why can't you optimize min/max updates the same way as sum? *(Changing a leaf doesn't always change the parent's min/max — must re-merge)*
4. What is the time complexity of a point update? *(O(log n))*
5. In iterative update, how do you find the leaf position for array index i? *(n + i, where n is the array size padded to power of 2)*

---

[← Previous: Range Query](05-range-query.md) | [Next Unit: Segment Tree Implementation →](../03-Segment-Tree-Implementation/01-recursive-implementation.md)

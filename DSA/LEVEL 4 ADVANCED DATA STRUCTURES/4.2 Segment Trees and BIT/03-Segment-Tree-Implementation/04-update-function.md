# Update Function

## Chapter Overview

The update function modifies a single element and refreshes all affected ancestors. This chapter covers both assignment and increment updates with complete traces.

---

## Assignment Update

```
function update(node, start, end, index, value):
    if start == end:                 // reached the leaf
        tree[node] = value
        return
    
    mid = start + (end - start) / 2
    if index <= mid:
        update(2*node, start, mid, index, value)
    else:
        update(2*node+1, mid+1, end, index, value)
    
    tree[node] = merge(tree[2*node], tree[2*node+1])
```

---

## Detailed Trace: Assignment Update

```
Array A = [5, 8, 6, 3, 2, 7, 1, 4],  Sum tree

Before:
                  1:[0-7]=36
                 /          \
           2:[0-3]=22     3:[4-7]=14
           /    \          /      \
       4:[0-1]=13 5:[2-3]=9  6:[4-5]=9  7:[6-7]=5
       / \       / \         / \        / \
      5   8     6   3       2   7      1   4

update(1, 0, 7, 4, 10)   — set A[4] = 10 (was 2)

Step 1: node=1, seg=[0,7], index=4
  4 > mid=3 → go RIGHT to node 3

Step 2: node=3, seg=[4,7], index=4
  4 ≤ mid=5 → go LEFT to node 6

Step 3: node=6, seg=[4,5], index=4
  4 ≤ mid=4 → go LEFT to node 12

Step 4: node=12, seg=[4,4], index=4
  start==end → LEAF → tree[12] = 10

Unwind:
  tree[6] = merge(tree[12], tree[13]) = merge(10, 7) = 17
  tree[3] = merge(tree[6], tree[7]) = merge(17, 5) = 22
  tree[1] = merge(tree[2], tree[3]) = merge(22, 22) = 44

After:
                  1:[0-7]=44         ← 36→44 (+8)
                 /          \
           2:[0-3]=22     3:[4-7]=22    ← 14→22 (+8)
           /    \          /      \
       4:[0-1]=13 5:[2-3]=9  6:[4-5]=17  7:[6-7]=5
       / \       / \         / \   ↑     / \
      5   8     6   3      10  7  +8    1   4
                            ↑
                         changed (2→10)

Nodes modified: 12 → 6 → 3 → 1  (4 nodes = log₂(8) + 1)
```

---

## Increment Update

```
function incrementUpdate(node, start, end, index, delta):
    if start == end:
        tree[node] += delta
        return
    
    mid = start + (end - start) / 2
    if index <= mid:
        incrementUpdate(2*node, start, mid, index, delta)
    else:
        incrementUpdate(2*node+1, mid+1, end, index, delta)
    
    tree[node] = merge(tree[2*node], tree[2*node+1])
```

**Shortcut for sum trees**:
```
// Since sum is linear, we can add delta while going down
function incrementUpdate_fast(node, start, end, index, delta):
    tree[node] += delta              // add delta to every ancestor
    if start == end: return
    
    mid = start + (end - start) / 2
    if index <= mid:
        incrementUpdate_fast(2*node, start, mid, index, delta)
    else:
        incrementUpdate_fast(2*node+1, mid+1, end, index, delta)

// No re-merge needed! Each node on the path gets +delta.
// This works ONLY for sum (not min/max/gcd).
```

---

## Update for Min Tree

```
Min tree before:
           1:[0-3]=2
          /        \
     2:[0-1]=3    3:[2-3]=2
     / \          / \
   4:5  5:3     6:6  7:2

update index 3 to value 8 (was 2):

Step: tree[7] = 8 (leaf)
Merge: tree[3] = min(tree[6], tree[7]) = min(6, 8) = 6   ← changed!
Merge: tree[1] = min(tree[2], tree[3]) = min(3, 6) = 3   ← changed!

After:
           1:[0-3]=3        ← was 2, now 3
          /        \
     2:[0-1]=3    3:[2-3]=6   ← was 2, now 6
     / \          / \
   4:5  5:3     6:6  7:8      ← was 2, now 8

Note: The minimum changed from 2 to 3 — a completely different element!
This is why you MUST re-merge for min/max trees.
```

---

## Multiple Sequential Updates

```
Starting tree (sum): [5, 8, 6, 3]

tree = [_, 22, 13, 9, 5, 8, 6, 3]

Update 1: A[0] = 1 (was 5)
  tree[4] = 1
  tree[2] = merge(1, 8) = 9
  tree[1] = merge(9, 9) = 18
  Current: [_, 18, 9, 9, 1, 8, 6, 3]

Update 2: A[3] = 10 (was 3)
  tree[7] = 10
  tree[3] = merge(6, 10) = 16
  tree[1] = merge(9, 16) = 25
  Current: [_, 25, 9, 16, 1, 8, 6, 10]

Update 3: A[1] = 0 (was 8)
  tree[5] = 0
  tree[2] = merge(1, 0) = 1
  tree[1] = merge(1, 16) = 17
  Current: [_, 17, 1, 16, 1, 0, 6, 10]

Each update: O(log n) = O(2) for n=4
Total: 3 × O(log n)
```

---

## Iterative Point Update

```
function iterativeUpdate(index, value):
    pos = n + index           // leaf position
    tree[pos] = value
    pos = pos >> 1            // parent
    while pos >= 1:
        tree[pos] = merge(tree[pos << 1], tree[pos << 1 | 1])
        pos = pos >> 1

// Trace: n=8, update A[4]=10
// pos = 8+4 = 12 → tree[12] = 10
// pos = 6 → tree[6] = merge(tree[12], tree[13])
// pos = 3 → tree[3] = merge(tree[6], tree[7])
// pos = 1 → tree[1] = merge(tree[2], tree[3])
// pos = 0 → exit
```

---

## Correctness Argument

```
After update(node, start, end, index, value):

  INVARIANT: tree[node] = merge(A[start], A[start+1], ..., A[end])
  
  where A[index] has been changed to value.

Proof:
  1. At the leaf, tree[node] = value = A[index] (after update).  ✓
  
  2. Going back up, each ancestor re-merges its children:
     tree[parent] = merge(tree[left], tree[right])
     
     One child has been correctly updated (recursive hypothesis).
     The other child is unchanged (still correct from build or previous update).
     
     So the merge produces the correct value.  ✓
  
  3. This continues up to the root.  ✓
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Path traversed | Root → Target leaf (one direction only) |
| Nodes modified | O(log n) — the path from leaf to root |
| Assignment | tree[leaf] = value |
| Increment | tree[leaf] += delta |
| Sum shortcut | Add delta to every ancestor (no re-merge needed) |
| Min/Max | Must always re-merge — parent value may change unpredictably |
| Iterative | Start at n + index, walk up dividing by 2 |
| Per-update cost | O(log n) |

---

## Quick Revision Questions

1. In the update function, do we recurse into one child or both? *(One — the child containing the target index)*
2. Why can't we use the "add delta on the way down" shortcut for min trees? *(Changing a leaf's value doesn't predictably change the min of ancestors)*
3. What is the last step in every update call (except at the leaf)? *(Re-merge: tree[node] = merge(tree[2*node], tree[2*node+1]))*
4. How does iterative update find the leaf position for index i? *(n + i, where n is padded to power of 2)*
5. How many merge operations does a single update perform? *(O(log n) — one per level on the path from leaf to root)*

---

[← Previous: Query Function](03-query-function.md) | [Next: Time Complexity →](05-time-complexity.md)

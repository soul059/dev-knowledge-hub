# Push Down Logic

## Chapter Overview

Push down (also called "propagate" or "push lazy") is the heart of lazy propagation. It transfers a node's pending update to its two children. This chapter details the exact mechanics for different operations.

---

## When Push Down Happens

```
Push down is called when we need to recurse into a node's children
BUT the node has a pending lazy update.

Two situations:

1. QUERY passes through a lazy node
   → Must push down before recursing to get correct child values

2. UPDATE partially overlaps a lazy node's range
   → Must push down before recursing to correctly split the update

       push_down(node)
       /            \
    child_L         child_R
    (now correct)   (now correct)

Push down is NOT called when:
  - Full overlap during update (just accumulate lazy)
  - No overlap (return immediately)
  - Node is a leaf (no children to push to)
```

---

## Push Down for Range Add (Sum Tree)

```
function push_down(node, start, end):
    if lazy[node] != 0:
        mid = (start + end) / 2
        left_len  = mid - start + 1
        right_len = end - mid
        
        // Update left child
        tree[2*node] += lazy[node] * left_len
        lazy[2*node] += lazy[node]
        
        // Update right child
        tree[2*node+1] += lazy[node] * right_len
        lazy[2*node+1] += lazy[node]
        
        // Clear my lazy
        lazy[node] = 0

Breakdown:
  tree[child] += lazy × child_length
    → child's sum increases by (added_value × number_of_elements)
  
  lazy[child] += lazy
    → child inherits the pending update for ITS children
  
  lazy[node] = 0
    → this node no longer has pending updates

Visual:
  BEFORE push_down:
    Node 2: tree=20, lazy=5, range=[0,3], length=4
      Child 4: tree=8,  lazy=0, range=[0,1], length=2
      Child 5: tree=7,  lazy=0, range=[2,3], length=2

  AFTER push_down(2):
    Node 2: tree=20, lazy=0    ← lazy cleared
      Child 4: tree=8+5×2=18,  lazy=0+5=5
      Child 5: tree=7+5×2=17,  lazy=0+5=5

  Verification: 18 + 17 = 35 ≠ 20?
  Wait — tree[2] was already updated when the range add happened!
  tree[2] was set to 20 = old_sum + 5×4 = 0 + 20.  
  Hmm, let me redo with clearer numbers.

Detailed example:
  Original tree (sum):
    Node 2: tree=10, range=[0,3]
      Child 4: tree=3, range=[0,1]
      Child 5: tree=7, range=[2,3]

  Range add +5 to [0,3] (full overlap with node 2):
    tree[2] += 5 × 4 = 10 + 20 = 30
    lazy[2] = 5

  Now Node 2: tree=30, lazy=5
    Children: STALE (still 3 and 7)

  push_down(2):
    tree[4] += 5 × 2 = 3 + 10 = 13    lazy[4] = 5
    tree[5] += 5 × 2 = 7 + 10 = 17    lazy[5] = 5
    lazy[2] = 0

  Verify: 13 + 17 = 30 = tree[2]  ✓
```

---

## Push Down for Range Set (Sum Tree)

```
function push_down(node, start, end):
    if has_lazy[node]:
        mid = (start + end) / 2
        val = lazy[node]
        
        // Left child: set all to val
        tree[2*node] = val * (mid - start + 1)
        lazy[2*node] = val
        has_lazy[2*node] = true
        
        // Right child: set all to val
        tree[2*node+1] = val * (end - mid)
        lazy[2*node+1] = val
        has_lazy[2*node+1] = true
        
        // Clear
        has_lazy[node] = false
        lazy[node] = 0

Key difference from range-add:
  - tree[child] = val × length  (ASSIGN, not increment)
  - lazy[child] = val           (REPLACE, not accumulate)
  - Need has_lazy flag (can't use 0 as sentinel)
```

---

## Push Down for Range Add (Min Tree)

```
function push_down(node, start, end):
    if lazy[node] != 0:
        // Min of (all elements + delta) = (min of all elements) + delta
        tree[2*node] += lazy[node]
        lazy[2*node] += lazy[node]
        
        tree[2*node+1] += lazy[node]
        lazy[2*node+1] += lazy[node]
        
        lazy[node] = 0

Simpler than sum! No length multiplication.
  
  Sum tree:  tree[child] += lazy × length
  Min tree:  tree[child] += lazy
  Max tree:  tree[child] += lazy

Why? Adding delta to every element shifts the min/max by delta,
regardless of how many elements there are.
```

---

## Step-by-Step Trace

```
Array A = [1, 3, 5, 7, 2, 4]   Sum segment tree

Build:
              1:[0-5]=22
              /          \
        2:[0-2]=9      3:[3-5]=13
        /     \         /      \
    4:[0-1]=4  5:[2]=5 6:[3-4]=9  7:[5]=4
    / \                 / \
   1   3               7   2

Operation: range_add(1, 4, +10)  → Add 10 to A[1..4]

Step 1: At node 1 [0-5], partial overlap (1,4 doesn't cover 0,5)
  push_down(1)  → lazy[1]=0, nothing to push
  Recurse both children

Step 2: At node 2 [0-2], partial overlap (1,4 partially covers 0,2)
  push_down(2)  → lazy[2]=0, nothing to push
  Recurse both children
  
Step 3: At node 4 [0-1], partial overlap
  push_down(4)  → nothing
  - node 8 [0-0]: no overlap → skip
  - node 9 [1-1]: FULL overlap
    tree[9] += 10 = 3+10 = 13
    (leaf, no lazy needed)
  tree[4] = tree[8]+tree[9] = 1+13 = 14

Step 4: At node 5 [2-2], FULL overlap
  tree[5] += 10×1 = 15
  lazy[5] = 10  (but it's a leaf, so effectively no children)
  Actually, leaf: just tree[5] = 5+10 = 15

Step 5: tree[2] = tree[4]+tree[5] = 14+15 = 29

Step 6: At node 3 [3-5], partial overlap
  push_down(3) → nothing

Step 7: At node 6 [3-4], FULL overlap!
  tree[6] += 10×2 = 9+20 = 29
  lazy[6] = 10   ← LAZY! Don't recurse further

Step 8: At node 7 [5-5], no overlap → skip

Step 9: tree[3] = tree[6]+tree[7] = 29+4 = 33
Step 10: tree[1] = tree[2]+tree[3] = 29+33 = 62

Final state:
              1:[0-5]=62
              /          \
        2:[0-2]=29     3:[3-5]=33
        /     \         /       \
    4:[0-1]=14 5:[2]=15 6:[3-4]=29  7:[5]=4
    / \                  lazy=10
   1   13

Node 6 has lazy=10: children still show old values (7, 2)
but node 6's sum (29) is correct (17+12=29 → (7+10)+(2+10)=29)
```

---

## Push Down: Flow Diagram

```
push_down(node):

  ┌─ lazy[node] == 0 (identity)?
  │    YES → return (nothing to do)
  │    NO  ↓
  │
  ├─ Apply lazy to LEFT child:
  │    tree[2*node] = f(tree[2*node], lazy[node], left_length)
  │    lazy[2*node] = compose(lazy[2*node], lazy[node])
  │
  ├─ Apply lazy to RIGHT child:
  │    tree[2*node+1] = f(tree[2*node+1], lazy[node], right_length)
  │    lazy[2*node+1] = compose(lazy[2*node+1], lazy[node])
  │
  └─ Clear: lazy[node] = identity

Where:
  f() depends on (tree type × update type)
  compose() combines two pending lazy values

Examples of f():
  Sum tree + Range add:  f(tree_val, lazy, len) = tree_val + lazy × len
  Sum tree + Range set:  f(tree_val, lazy, len) = lazy × len
  Min tree + Range add:  f(tree_val, lazy, len) = tree_val + lazy
  Min tree + Range set:  f(tree_val, lazy, len) = lazy
```

---

## Common Bugs in Push Down

```
Bug 1: Forgetting to push down before recursing
  → Children have stale values → wrong query results

Bug 2: Pushing down on leaf nodes
  → Accessing tree[2*node] for a leaf → array out of bounds!
  → Fix: only push_down when start < end

Bug 3: Not clearing lazy after push down
  → lazy accumulates → children get double-updated

Bug 4: Wrong length calculation
  → Using (end - start) instead of (end - start + 1) for sum
  → Off-by-one: sum tree gets wrong values

Bug 5: Not composing lazy correctly
  → Using = instead of += for add
  → Using += instead of = for set
```

---

## Summary Table

| Operation | tree[child] update | lazy[child] update | lazy identity |
|-----------|-------------------|-------------------|---------------|
| Range add (sum) | += lazy × len | += lazy | 0 |
| Range add (min) | += lazy | += lazy | 0 |
| Range set (sum) | = lazy × len | = lazy | NONE flag |
| Range set (min) | = lazy | = lazy | NONE flag |
| Range XOR (xor) | ^= lazy (if odd len) | ^= lazy | 0 |

---

## Quick Revision Questions

1. When is push_down called? *(Before recursing into children of a node that has a pending lazy update)*
2. What happens to tree[child] during push down for range-add on a sum tree? *(tree[child] += lazy[parent] × child_length)*
3. Why is push_down different for sum vs min trees? *(Sum needs length multiplication: adding delta to k elements adds delta×k to sum. Min just shifts by delta.)*
4. What happens if you forget to clear lazy[node] after pushing? *(Children will receive the same update again next time — double counting)*
5. Should push down be called on leaf nodes? *(No — leaves have no children. Stop when start == end.)*

---

[← Previous: Lazy Array Concept](02-lazy-array-concept.md) | [Next: Range Update with Lazy →](04-range-update-with-lazy.md)

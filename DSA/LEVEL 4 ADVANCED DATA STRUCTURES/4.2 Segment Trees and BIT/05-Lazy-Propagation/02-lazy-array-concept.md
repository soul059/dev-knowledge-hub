# Lazy Array Concept

## Chapter Overview

Lazy propagation requires a **parallel array** alongside the segment tree. This "lazy array" stores pending updates that haven't been pushed to children yet. Understanding its structure is essential before implementing push-down logic.

---

## Two Arrays, One Tree

```
Standard segment tree: 1 array
  tree[1..4n]    → stores aggregate values (sum, min, etc.)

Lazy segment tree: 2 arrays
  tree[1..4n]    → stores aggregate values (ALREADY applied)
  lazy[1..4n]    → stores PENDING updates for children

            tree[1] = 45    lazy[1] = 0
           /              \
    tree[2] = 20         tree[3] = 25
    lazy[2] = 5          lazy[3] = 0
    /        \            /        \
  tree[4]=8  tree[5]=7  tree[6]=15 tree[7]=10
  lazy[4]=0  lazy[5]=0  lazy[6]=0  lazy[7]=0

Interpretation:
  Node 2 has lazy[2]=5 meaning:
    "My value (20) is CORRECT"
    "But my children haven't received the +5 update yet"
    "When someone asks about my children, push +5 down first"
```

---

## What Lazy Stores

```
For different operations, lazy stores different things:

RANGE ADD:
  lazy[node] = accumulated delta to add to children
  Example: lazy[3] = 7 means
    "Add 7 to all elements in children's ranges"

RANGE SET (ASSIGNMENT):
  lazy[node] = value to assign to all children
  Need a flag (or sentinel) to distinguish "no pending" from "set to 0"
  Example: lazy[3] = 10 means
    "Set all elements in children's ranges to 10"

RANGE XOR:
  lazy[node] = accumulated XOR value
  Example: lazy[3] = 0b1010 means
    "XOR all elements in children's ranges by 1010"

RANGE MULTIPLY:
  lazy[node] = accumulated multiplier
  Example: lazy[3] = 3 means
    "Multiply all elements in children's ranges by 3"
```

---

## Initialization

```
At the start, no updates are pending:

For RANGE ADD:
  lazy[i] = 0     for all i      (adding 0 = no-op)

For RANGE SET:
  lazy[i] = NONE  or  has_lazy[i] = false
  (can't use 0 since "set to 0" is a valid update)

For RANGE XOR:
  lazy[i] = 0     for all i      (XOR with 0 = no-op)

For RANGE MULTIPLY:
  lazy[i] = 1     for all i      (multiply by 1 = no-op)
```

---

## How Lazy Values Accumulate

```
When multiple updates hit the same node WITHOUT being pushed down:

RANGE ADD:
  First update: +5   → lazy = 5
  Second update: +3  → lazy = 5 + 3 = 8
  Composition: ADD

RANGE SET:
  First update: =10  → lazy = 10
  Second update: =7  → lazy = 7   (overwrite! last one wins)
  Composition: REPLACE

RANGE XOR:
  First update: ⊕5   → lazy = 5
  Second update: ⊕3  → lazy = 5 ⊕ 3 = 6
  Composition: XOR

Visual timeline:
  
  Node 3, lazy = 0
       ↓ range_add(+5)
  Node 3, lazy = 5
       ↓ range_add(+3)    (no push-down happened in between)
  Node 3, lazy = 8
       ↓ query reaches node 3's child
  PUSH DOWN: children get +8, then lazy = 0
```

---

## Relationship Between tree[] and lazy[]

```
Critical invariant:

  tree[node] ALWAYS reflects the correct aggregate
  for its range, ASSUMING lazy[node] has been applied.

  lazy[node] is a PENDING update for node's CHILDREN,
  NOT for node itself.

Example (range sum, range add):

  Array: [1, 2, 3, 4]  → sum = 10
  
  After "add 5 to [0,3]":
    tree[1] = 10 + 5×4 = 30    ← CORRECT sum for [0,3]
    lazy[1] = 5                  ← children haven't been updated
    
    tree[2] = 3     ← STALE (should be 3 + 5×2 = 13)
    tree[3] = 7     ← STALE (should be 7 + 5×2 = 17)
    
  But tree[1] is correct! So queries using only node 1 work fine.
  We only fix tree[2], tree[3] when we NEED to go deeper.
```

---

## Memory Layout

```
tree[] and lazy[] have identical structures:

Index:     1    2    3    4    5    6    7    8  ...
tree[]:   [30] [ 3] [ 7] [ 1] [ 2] [ 3] [ 4] [--] ...
lazy[]:   [ 5] [ 0] [ 0] [ 0] [ 0] [ 0] [ 0] [--] ...

    tree[i] and lazy[i] refer to the SAME node
    tree[i] = aggregate value
    lazy[i] = pending update for children (2i, 2i+1)

Memory cost:
  Standard segment tree: 4n integers
  Lazy segment tree:     4n + 4n = 8n integers
  Overhead: 2× memory
```

---

## Leaf Nodes and Lazy

```
Leaf nodes (no children) have lazy values that are ALWAYS 0.

Why? Because when we push lazy to a leaf:
  - We apply the update directly to tree[leaf]
  - There are no children to propagate to
  - So lazy[leaf] is cleared immediately

In practice, push_down is never called on leaves,
because we stop recursing at start == end.

But if a range update covers a single element:
  tree[leaf] gets updated directly
  lazy[leaf] stays 0 (or becomes 0 immediately)
```

---

## Combined Lazy: Add + Set

```
Some problems need BOTH add and set operations.

WRONG approach: Two separate lazy arrays that don't interact.

RIGHT approach: Unified lazy with order:
  "First SET to val, then ADD delta"

  lazy_set[node] = value to set (or NONE)
  lazy_add[node] = value to add after set

  Applying "Set to X" on a node with pending "Add D":
    lazy_set = X
    lazy_add = 0     (set overrides prior add)

  Applying "Add D" on a node with pending "Set X":
    lazy_add += D    (add stacks on top of set)

  Push down:
    child.set = parent.set
    child.add = parent.add
    Then clear parent's lazy.

This is more advanced — we'll revisit in the implementation chapter.
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| What lazy[] stores | Pending update for children, NOT for the node itself |
| Initialization | Identity element of the update (0 for add, 1 for multiply, 0 for XOR) |
| Accumulation | Compose pending updates (add → sum, set → replace, xor → xor) |
| tree[node] correctness | Always correct for its range — lazy hasn't invalidated it |
| Memory overhead | 2× (need lazy array same size as tree) |
| Leaf lazy | Always 0 — no children to defer to |

---

## Quick Revision Questions

1. What does lazy[node] represent? *(A pending update that needs to be applied to node's children, not to node itself)*
2. Is tree[node] correct even when lazy[node] ≠ 0? *(Yes — tree[node] already includes the update; lazy only affects children)*
3. How do you initialize the lazy array for range-add? *(All zeros — adding 0 is a no-op)*
4. What happens when two range-add updates stack on the same node? *(They accumulate: lazy[node] += new_delta)*
5. Why can't you use 0 as the "no pending" sentinel for range-set? *(Because "set all to 0" is a valid operation — need a separate flag or sentinel value)*
6. What is the memory overhead of lazy propagation? *(2× — the lazy array is the same size as the tree array)*

---

[← Previous: Why Lazy Propagation](01-why-lazy-propagation.md) | [Next: Push Down Logic →](03-push-down-logic.md)

# Point Update in BIT

## Chapter Overview

Point update in BIT adds a value to a single element and propagates the change to all BIT nodes whose range includes that element. The algorithm is remarkably simple — just a few lines of code.

---

## The Algorithm

```
function update(i, delta):
    while i <= n:
        B[i] += delta
        i += i & (-i)      // i += lowbit(i)

That's it. The ENTIRE update algorithm.

What it does:
  1. Start at index i
  2. B[i] covers a range that includes index i → update it
  3. Move to the next BIT node that also includes i
  4. Continue until no more nodes (i > n)
```

---

## Walkthrough: update(3, +7) with n=8

```
Array: A = [0, 5, 3, 2, 8, 1, 4, 6]  (1-indexed)
BIT before: B = [0, 5, 8, 2, 18, 1, 5, 6, 30]

Step 1: i = 3 = 011₂
  B[3] += 7    →  B[3] = 2 + 7 = 9
  i += lowbit(3) = 3 + 1 = 4

Step 2: i = 4 = 100₂
  B[4] += 7    →  B[4] = 18 + 7 = 25
  i += lowbit(4) = 4 + 4 = 8

Step 3: i = 8 = 1000₂
  B[8] += 7    →  B[8] = 30 + 7 = 37
  i += lowbit(8) = 8 + 8 = 16

Step 4: i = 16 > 8 = n → STOP

Updated nodes: B[3], B[4], B[8]

Verify these are exactly the nodes whose range includes index 3:
  B[3] covers [3, 3]  ✓ includes 3
  B[4] covers [1, 4]  ✓ includes 3
  B[8] covers [1, 8]  ✓ includes 3
  B[2] covers [1, 2]  ✗ does NOT include 3
  B[6] covers [5, 6]  ✗ does NOT include 3

Every node containing index 3 was updated. ✓
```

---

## Visual: Update Propagation

```
update(5, +10) with n=16:

Index:  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16

        ─  ─  ─  ─  ●  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─
                     5

        ─  ─  ─  ─  ─  ●  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─
                        6 = 5 + lowbit(5) = 5 + 1

        ─  ─  ─  ─  ─  ─  ─  ●  ─  ─  ─  ─  ─  ─  ─  ─
                              8 = 6 + lowbit(6) = 6 + 2

        ─  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─  ─  ●
                                                         16 = 8 + lowbit(8) = 8 + 8

Path: 5 → 6 → 8 → 16 → (32 > 16, stop)

Each step jumps by an INCREASING amount (1, 2, 8)
because lowbit grows as we ascend.
```

---

## Assignment Update

```
BIT naturally supports DELTA updates (add/subtract).
For ASSIGNMENT (set A[i] = val), compute the delta first:

function assign(i, val):
    old_val = point_query(i)     // need current value
    delta = val - old_val
    update(i, delta)

How to get point_query(i)?
  Method 1: prefix_sum(i) - prefix_sum(i-1)   → O(log n)
  Method 2: Keep a separate copy of A[]         → O(1)
  Method 3: Special BIT point query (less common)
```

---

## Multiple Updates

```
Multiple point updates are independent and each takes O(log n):

update(2, +5)    → O(log n)
update(7, -3)    → O(log n)
update(4, +1)    → O(log n)

No interference between updates — they simply add to B[] values.

Combined effect: equivalent to:
  A[2] += 5
  A[7] -= 3
  A[4] += 1
applied to the original array.
```

---

## Building BIT from Array (via Updates)

```
Method 1: n individual updates → O(n log n)

function build_slow(A, n):
    B = array of size n+1, all zeros
    for i = 1 to n:
        update(i, A[i])

Method 2: Direct O(n) build

function build_fast(A, n):
    // Copy array values to BIT
    for i = 1 to n:
        B[i] = A[i]
    
    // Propagate each node to its parent
    for i = 1 to n:
        parent = i + (i & (-i))
        if parent <= n:
            B[parent] += B[i]

Why Method 2 is O(n):
  Each node is processed exactly once.
  Each node adds to at most one parent.
  Total operations: n additions.

Trace (A = [_, 3, 1, 4, ?, 2, 5, 3, ?]) (1-indexed):
  B = [0, 3, 1, 4, 0, 2, 5, 3, 0]  ← copy A

  i=1: parent=2,  B[2] += B[1] = 1+3  = 4
  i=2: parent=4,  B[4] += B[2] = 0+4  = 4
  i=3: parent=4,  B[4] += B[3] = 4+4  = 8
  i=4: parent=8,  B[8] += B[4] = 0+8  = 8
  i=5: parent=6,  B[6] += B[5] = 5+2  = 7
  i=6: parent=8,  B[8] += B[6] = 8+7  = 15
  i=7: parent=8,  B[8] += B[7] = 15+3 = 18

  B = [0, 3, 4, 4, 8, 2, 7, 3, 18]

  Verify: prefix_sum(8) = B[8] = 18  ← but we only have 7 elements?
  Let me recount: A = [_, 3, 1, 4, 0, 2, 5, 3, 0] (n=8, A[4]=0, A[8]=0)
  Sum = 3+1+4+0+2+5+3+0 = 18  ✓
```

---

## Code Comparison

```
BIT Point Update:              Segment Tree Point Update:

function update(i, delta):     function update(node, s, e, i, val):
    while i <= n:                  if s == e:
        B[i] += delta                  tree[node] = val
        i += i & (-i)                  return
                                   mid = (s+e)/2
                                   if i <= mid:
                                       update(2*node, s, mid, i, val)
                                   else:
                                       update(2*node+1, mid+1, e, i, val)
                                   tree[node] = tree[2*node] + tree[2*node+1]

  3 lines vs 10+ lines
  No recursion vs recursion
  No function call overhead
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Algorithm | While i ≤ n: B[i] += delta, i += i & (-i) |
| Time complexity | O(log n) — at most ⌊log₂ n⌋ + 1 iterations |
| Space complexity | O(1) additional (modifies B[] in-place) |
| Update type | Delta (additive) by default |
| Assignment | Compute delta = new - old, then update(i, delta) |
| Build from array | O(n) with parent propagation method |

---

## Quick Revision Questions

1. What is the BIT update algorithm? *(While i ≤ n: B[i] += delta, i += i & (-i))*
2. How does the index change during update? *(i increases by its lowest set bit each step: i += i & (-i))*
3. How many BIT nodes are modified during a point update? *(At most ⌊log₂ n⌋ + 1 nodes)*
4. How do you do an assignment update (set A[i] = val) with BIT? *(Compute delta = val - current_value, then call update(i, delta))*
5. What is the O(n) method for building a BIT? *(Copy A to B, then for each i, add B[i] to B[i + lowbit(i)])*

---

[← Previous: Binary Representation](02-binary-representation.md) | [Next: Prefix Query →](04-prefix-query.md)

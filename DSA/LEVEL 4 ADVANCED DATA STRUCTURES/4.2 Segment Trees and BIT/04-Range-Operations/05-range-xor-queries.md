# Range XOR Queries

## Chapter Overview

XOR (exclusive or) is a powerful bitwise operation for range queries. Unlike min/max/gcd, XOR **is invertible** (a ⊕ a = 0), so it works with prefix XOR and even BIT — not just segment trees.

---

## XOR Properties

```
1. ASSOCIATIVE:   (a ⊕ b) ⊕ c = a ⊕ (b ⊕ c)    ✓
2. COMMUTATIVE:   a ⊕ b = b ⊕ a                   ✓
3. IDENTITY:      a ⊕ 0 = a                        ✓  (0 is identity)
4. SELF-INVERSE:  a ⊕ a = 0                        ✓  (every element is its own inverse!)
5. INVERTIBLE:    YES — inverse of a is a itself    ✓
6. NOT IDEMPOTENT: a ⊕ a = 0 ≠ a (for a ≠ 0)      ✗

Key insight: XOR is like addition mod 2 on each bit
  It's invertible → prefix XOR works!
  It's NOT idempotent → Sparse Table doesn't work!
```

---

## Approach Comparison

```
Since XOR is invertible, multiple approaches work:

┌─────────────────────┬──────────┬──────────┬───────────┐
│     Approach        │  Build   │  Query   │  Update   │
├─────────────────────┼──────────┼──────────┼───────────┤
│  Prefix XOR         │  O(n)    │  O(1)    │  O(n)     │
│  BIT (Fenwick)      │  O(n)    │  O(log n)│  O(log n) │
│  Segment Tree       │  O(n)    │  O(log n)│  O(log n) │
└─────────────────────┴──────────┴──────────┴───────────┘

Prefix XOR wins for static data.
BIT or SegTree for dynamic data. BIT is simpler and faster.
```

---

## Prefix XOR Approach

```
P[0] = 0
P[i] = A[0] ⊕ A[1] ⊕ ... ⊕ A[i-1]

XOR(L, R) = P[R+1] ⊕ P[L]

Why it works:
  P[R+1] = A[0] ⊕ A[1] ⊕ ... ⊕ A[R]
  P[L]   = A[0] ⊕ A[1] ⊕ ... ⊕ A[L-1]
  
  P[R+1] ⊕ P[L] = A[L] ⊕ A[L+1] ⊕ ... ⊕ A[R]
  
  The common prefix cancels out because x ⊕ x = 0!

Example:
  A = [5, 3, 6, 2, 4]
  
  Binary:  101  011  110  010  100
  
  P[0] = 0     = 000
  P[1] = 5     = 101
  P[2] = 5⊕3   = 110
  P[3] = 6⊕6   = 000
  P[4] = 0⊕2   = 010
  P[5] = 2⊕4   = 110
  
  XOR(1, 3) = P[4] ⊕ P[1] = 010 ⊕ 101 = 111 = 7
  Check: 3 ⊕ 6 ⊕ 2 = 011 ⊕ 110 ⊕ 010 = 111 = 7  ✓
```

---

## Segment Tree for XOR

```
Merge function:     merge(a, b) = a ⊕ b
Identity element:   IDENTITY = 0

function build(node, start, end):
    if start == end:
        tree[node] = A[start]
        return
    mid = (start + end) / 2
    build(2*node, start, mid)
    build(2*node+1, mid+1, end)
    tree[node] = tree[2*node] ^ tree[2*node+1]

function query(node, start, end, L, R):
    if end < L or start > R:
        return 0                    // identity for XOR
    if L <= start and end <= R:
        return tree[node]
    mid = (start + end) / 2
    return query(2*node, start, mid, L, R) ^
           query(2*node+1, mid+1, end, L, R)

function update(node, start, end, idx, val):
    if start == end:
        tree[node] = val
        return
    mid = (start + end) / 2
    if idx <= mid:
        update(2*node, start, mid, idx, val)
    else:
        update(2*node+1, mid+1, end, idx, val)
    tree[node] = tree[2*node] ^ tree[2*node+1]
```

---

## Worked Example (Segment Tree)

```
A = [5, 3, 6, 2]   n = 4

XOR segment tree:

          1:[0-3] = 5⊕3⊕6⊕2 = 2
          /              \
    2:[0-1] = 5⊕3 = 6    3:[2-3] = 6⊕2 = 4
    /     \                /     \
  4:[0]=5  5:[1]=3       6:[2]=6  7:[3]=2

Binary view:
          1: 010
         /      \
     2: 110    3: 100
     / \        / \
   101  011  110   010

Query: XOR(0, 2) → should be 5 ⊕ 3 ⊕ 6 = 0 (= 101 ⊕ 011 ⊕ 110 = 000)

query(1, 0, 3, 0, 2):
  PARTIAL
  query(2, 0, 1, 0, 2): COMPLETE → 6 (110)
  query(3, 2, 3, 0, 2):
    PARTIAL
    query(6, 2, 2, 0, 2): COMPLETE → 6 (110)
    query(7, 3, 3, 0, 2): NO OVERLAP → 0 (000)
    6 ⊕ 0 = 6
  6 ⊕ 6 = 0

Answer: 0  ✓
```

---

## XOR Toggle Update

```
A special update for XOR: "toggle" (XOR with delta)

Instead of "set A[i] = val":
  "A[i] = A[i] ⊕ delta"

This is simpler because:
  - Updating A[i] by ⊕ delta changes the XOR of every ancestor by ⊕ delta
  - Can propagate in O(log n) without rebuild

function toggle_update(node, start, end, idx, delta):
    if start == end:
        tree[node] ^= delta
        return
    mid = (start + end) / 2
    if idx <= mid:
        toggle_update(2*node, start, mid, idx, delta)
    else:
        toggle_update(2*node+1, mid+1, end, idx, delta)
    tree[node] = tree[2*node] ^ tree[2*node+1]

Note: For assignment update (set A[i] = val):
  delta = old_value ⊕ new_value
  toggle_update(.., idx, delta)
  Because: old ⊕ (old ⊕ new) = new
```

---

## Range XOR with Range XOR Updates (Lazy)

```
"XOR all elements in [L,R] by val, then query XOR of [L',R']"

Lazy propagation insight:
  If range [L,R] has length len:
  - XOR all by val: result XOR changes by val if len is odd, 0 if even
  - Because: val ⊕ val = 0 (pairs cancel)

Lazy merge:
  lazy[node] ^= val   (accumulate with XOR)

Push down:
  tree[child] ^= lazy[node]  IF child_length is odd
  lazy[child] ^= lazy[node]
  lazy[node] = 0

This is a subtle operation — the parity of segment length matters!
```

---

## XOR Applications

| Problem | Technique |
|---------|-----------|
| Find missing number | XOR all elements, XOR cancels duplicates |
| Find element appearing odd times | Range XOR gives it directly |
| Maximum XOR subarray | XOR prefix + trie (advanced) |
| Swap without temp variable | a ^= b; b ^= a; a ^= b |
| Subset XOR queries | Segment tree / offline |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Merge function | a ⊕ b (XOR) |
| Identity element | 0 |
| Invertible? | Yes — inverse of a is a (a ⊕ a = 0) |
| Idempotent? | No — a ⊕ a = 0 ≠ a |
| Works with BIT? | Yes! (invertible) |
| Works with Prefix? | Yes! (invertible) |
| Sparse Table? | No (not idempotent) |
| Range XOR update | Parity-dependent: flips if odd length, no-op if even |

---

## Quick Revision Questions

1. What is the identity element for XOR? *(0, because a ⊕ 0 = a)*
2. Why does prefix XOR work but prefix min doesn't? *(XOR is invertible: a ⊕ a = 0, so prefix elements cancel. Min has no inverse.)*
3. Can BIT be used for range XOR queries? *(Yes — XOR is invertible, so BIT works perfectly)*
4. Why doesn't Sparse Table work for XOR? *(XOR is not idempotent: a ⊕ a = 0 ≠ a, so overlapping ranges give wrong results)*
5. What happens when you XOR all elements of a range by value v? *(The range XOR flips by v if the range has odd length, unchanged if even length)*

---

[← Previous: Range GCD Queries](04-range-gcd-queries.md) | [Next: Custom Operations →](06-custom-operations.md)

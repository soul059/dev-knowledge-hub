# Chapter 3: Middle of Linked List and Nth Node from End

[← Previous: Cycle Length and Start Point](02-cycle-length-and-start.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 6 — Iterative and Recursive Reversal →](../06-Linked-List-Reversal/01-iterative-and-recursive-reversal.md)

---

## Chapter Overview

Two classic problems solved elegantly with the two-pointer technique: finding the **middle node** and the **Nth node from the end**, both in a single pass.

---

## 3.1 Finding the Middle of a Linked List

### The Problem

Given a linked list, find the middle node without knowing the length.

### The Intuition

If fast moves at 2x speed and slow at 1x speed, when fast reaches the end, slow is at the **halfway point** — like two runners where one is twice as fast.

```
  When fast reaches the finish line:
  
  Start ─────────── Middle ─────────── End
  slow starts here   slow is here      fast is here
  │                  │                  │
  ▼                  ▼                  ▼
  ├────── n/2 ──────►├────── n/2 ──────►│
```

### Algorithm

```
function findMiddle(HEAD):
    if HEAD == NULL: return NULL
    
    slow = HEAD
    fast = HEAD
    
    while fast ≠ NULL and fast.next ≠ NULL:
        slow = slow.next
        fast = fast.next.next
    
    return slow                      ← This is the middle node
```

### Trace — Odd Length List: 1 → 2 → 3 → 4 → 5

```
  Step 0: slow=1, fast=1
  [1] → [2] → [3] → [4] → [5] → NULL
   S      
   F

  Step 1: slow=2, fast=3
  [1] → [2] → [3] → [4] → [5] → NULL
          S           F

  Step 2: slow=3, fast=5
  [1] → [2] → [3] → [4] → [5] → NULL
                 S                 F

  fast.next = NULL → STOP
  Middle = Node(3) ✓
```

### Trace — Even Length List: 1 → 2 → 3 → 4 → 5 → 6

```
  Step 0: slow=1, fast=1
  Step 1: slow=2, fast=3
  Step 2: slow=3, fast=5
  Step 3: slow=4, fast=NULL (5.next.next = NULL)
  
  Wait: fast=5, fast.next=6, fast.next.next=NULL
  So slow=4, fast goes to NULL? Let me re-check.
  
  Step 2: slow=3, fast=5
    fast(5).next = 6, fast(5).next.next = NULL
    Continue: fast ≠ NULL and fast.next ≠ NULL? 
    fast = 5 ≠ NULL ✓, fast.next = 6 ≠ NULL ✓ → continue
    
  Step 3: slow=4, fast=6.next=NULL → fast=NULL? No:
    fast = fast.next.next = 6.next = NULL... 
    Actually fast = 5.next.next = NULL
    
  Let me redo properly:
  Step 2 (entering loop): fast=5, fast.next=6 ≠ NULL → enter
    slow = 3.next = 4
    fast = 5.next.next = 6.next = NULL → fast = NULL? No:
    fast = (fast.next).next = 6.next = NULL
    
  So fast = NULL now. Loop condition: fast ≠ NULL fails → EXIT
  
  Middle = Node(4) ← second middle node
```

### Which Middle for Even-Length Lists?

```
  Even-length list: [1] → [2] → [3] → [4] → [5] → [6]
                                   ↑      ↑
                                  1st    2nd
                                middle  middle

  The algorithm above returns the 2ND middle (Node 4).

  To get the 1ST middle, modify the condition:
  
  while fast.next ≠ NULL and fast.next.next ≠ NULL:
      slow = slow.next
      fast = fast.next.next
  
  This stops one step earlier → returns Node(3).
```

### Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(n) | Fast traverses the whole list |
| Space | O(1) | Two pointers |

### Alternative: Two-Pass Approach

```
Pass 1: Count nodes → n
Pass 2: Traverse to n/2

Same O(n) time, but TWO traversals vs ONE.
Slow/fast does it in a SINGLE pass!
```

---

## 3.2 Nth Node from the End

### The Problem

Find the Nth node from the end of a linked list **in a single pass**.

```
  List: [1] → [2] → [3] → [4] → [5] → NULL
  
  N=1: Node(5) (1st from end)
  N=2: Node(4) (2nd from end)
  N=3: Node(3) (3rd from end)
```

### The Intuition

Use two pointers with an **N-gap** between them:

```
  Start fast N steps ahead of slow.
  Then move both at the same speed.
  When fast reaches the end, slow is at the Nth-from-end node.

  Gap = N = 3:
  
  slow                    fast
   │                       │
   ▼                       ▼
  [1] → [2] → [3] → [4] → [5] → NULL
  ├──── N=3 gap ────►│

  Move both by 1:
         slow                    fast
          │                       │
          ▼                       ▼
  [1] → [2] → [3] → [4] → [5] → NULL

  Move both by 1:
                slow                    fast
                 │                       │
                 ▼                       ▼
  [1] → [2] → [3] → [4] → [5] → NULL
                 ↑
           3rd from end!
```

### Algorithm

```
function nthFromEnd(HEAD, n):
    if HEAD == NULL: return NULL
    
    // Step 1: Move fast n steps ahead
    fast = HEAD
    for i = 1 to n:
        if fast == NULL:
            error("n is larger than list length")
        fast = fast.next
    
    // Step 2: Move both until fast reaches NULL
    slow = HEAD
    while fast ≠ NULL:
        slow = slow.next
        fast = fast.next
    
    return slow                      ← Nth from end
```

### Step-by-Step Trace: N=2 in list [10, 20, 30, 40, 50]

```
  Phase 1: Move fast 2 steps ahead
    fast = HEAD → Node(10)
    i=1: fast = Node(20)
    i=2: fast = Node(30)

  Phase 2: Move both
    slow = Node(10), fast = Node(30)

  | Step | slow | fast   |
  |------|------|--------|
  | 1    | 20   | 40     |
  | 2    | 30   | 50     |
  | 3    | 40   | NULL   | ← fast is NULL, STOP

  Return slow = Node(40) ← 2nd from end ✓
```

---

## 3.3 Delete Nth Node from End

A variation — delete instead of just finding.

### Algorithm

```
function deleteNthFromEnd(HEAD, n):
    dummy = createNode(0)            ← Dummy node handles edge cases
    dummy.next = HEAD
    
    fast = dummy
    slow = dummy
    
    // Move fast n+1 steps ahead (so slow ends up BEFORE the target)
    for i = 0 to n:
        fast = fast.next
    
    // Move both until fast reaches NULL
    while fast ≠ NULL:
        slow = slow.next
        fast = fast.next
    
    // Delete slow.next
    slow.next = slow.next.next
    
    return dummy.next                ← New HEAD
```

### Why n+1 Steps?

```
  We need slow to be ONE NODE BEFORE the target:
  
  N=2 in [1, 2, 3, 4, 5]:
  
  Target: Node(4) (2nd from end)
  Slow should stop at: Node(3) (one before target)
  
  Gap = N + 1 = 3:
  
  slow                          fast
   │                             │
   ▼                             ▼
  [D] → [1] → [2] → [3] → [4] → [5] → NULL
  
  Move both:
         slow                          fast
          │                             │
          ▼                             ▼
  [D] → [1] → [2] → [3] → [4] → [5] → NULL

  Move both:
                slow                          fast
                 │                             │
                 ▼                             ▼
  [D] → [1] → [2] → [3] → [4] → [5] → NULL
  
  Move both:
                      slow                          fast=NULL
                       │
                       ▼
  [D] → [1] → [2] → [3] → [4] → [5] → NULL
  
  slow = Node(3), slow.next = Node(4) ← DELETE THIS
  slow.next = slow.next.next = Node(5)
  
  Result: 1 → 2 → 3 → 5  ✓
```

---

## 3.4 Other Two-Pointer Applications

### Check if List Length is Even or Odd

```
function isEvenLength(HEAD):
    fast = HEAD
    while fast ≠ NULL and fast.next ≠ NULL:
        fast = fast.next.next
    
    return fast == NULL              ← NULL = even, not NULL = odd
```

```
  Even: [1] → [2] → [3] → [4] → NULL
  fast: 1 → 3 → NULL → return true (even)

  Odd:  [1] → [2] → [3] → NULL
  fast: 1 → 3 → 3 is not NULL → return false (odd)
```

### Find the 1/3 and 2/3 Points

Use three pointers with speeds 1, 2, and 3:

```
function findThirds(HEAD):
    p1 = HEAD  (speed 1)
    p2 = HEAD  (speed 2)  
    p3 = HEAD  (speed 3)
    
    while p3 ≠ NULL and p3.next ≠ NULL and p3.next.next ≠ NULL:
        p1 = p1.next
        p2 = p2.next.next
        p3 = p3.next.next.next
    
    return p1 (at 1/3), p2 (at 2/3)
```

---

## 3.5 Problem-Solving Strategy

### Pattern: "How far from the end?"

```
  If you need to find a node relative to the END:
  
  1. Use two pointers with a gap
  2. Move the "ahead" pointer by the desired gap
  3. Move both pointers together
  4. When the ahead pointer reaches the end, 
     the trailing pointer is at your target
```

### Pattern: "Find a proportional position"

```
  Need 1/2? → speeds 1 and 2 (slow/fast)
  Need 1/3? → speeds 1 and 3
  Need 1/4? → speeds 1 and 4
  Need k/n? → advance slow every k steps of fast
```

---

## Summary Table

| Problem | Technique | Time | Space |
|---------|-----------|------|-------|
| Find middle | Slow (×1) + Fast (×2) | O(n) | O(1) |
| Nth from end | Gap of N, then move together | O(n) | O(1) |
| Delete Nth from end | Gap of N+1, dummy node | O(n) | O(1) |
| Even/odd length | Fast (×2) — check if NULL | O(n) | O(1) |
| 1/3 point | Slow (×1) + Fast (×3) | O(n) | O(1) |
| All solved in single pass | No need to count length first! | — | — |

---

## Quick Revision Questions

1. **Which middle does the slow-fast algorithm return for even-length lists?**
   <details><summary>Answer</summary>The standard algorithm (while fast ≠ NULL and fast.next ≠ NULL) returns the 2nd middle node. To get the 1st middle, check fast.next and fast.next.next instead.</details>

2. **Why is the gap N+1 (not N) when deleting the Nth node from end?**
   <details><summary>Answer</summary>We need `slow` to stop at the node BEFORE the target (to update its `next` pointer). A gap of N+1 positions slow one node earlier than the target.</details>

3. **How does a dummy node help in the delete-Nth-from-end problem?**
   <details><summary>Answer</summary>If the node to delete is HEAD itself, there's no previous node. The dummy node sits before HEAD, so even HEAD has a "previous" node (dummy), eliminating the special case.</details>

4. **Can you find the middle of a linked list without slow-fast pointers?**
   <details><summary>Answer</summary>Yes: First pass to count nodes (n), second pass to traverse to n/2. But this requires two passes. Slow-fast does it in one pass.</details>

5. **How would you find the node at position n/4 in a linked list?**
   <details><summary>Answer</summary>Use two pointers where fast moves 4 steps for every 1 step of slow. When fast reaches the end, slow is at n/4.</details>

6. **What happens if N is larger than the list length when finding Nth from end?**
   <details><summary>Answer</summary>The fast pointer reaches NULL before completing N steps. This should be detected and an error returned (or NULL).</details>

---

[← Previous: Cycle Length and Start Point](02-cycle-length-and-start.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 6 — Iterative and Recursive Reversal →](../06-Linked-List-Reversal/01-iterative-and-recursive-reversal.md)

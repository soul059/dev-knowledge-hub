# Chapter 2: Cycle Length and Cycle Start Point

[← Previous: Concept and Cycle Detection](01-concept-and-cycle-detection.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Middle Node and Nth from End →](03-middle-node-and-nth-from-end.md)

---

## Chapter Overview

After detecting a cycle using Floyd's algorithm, we can determine the **cycle length** and the **exact node where the cycle begins**. These are essential follow-up problems that rely on the same slow/fast pointer foundation.

---

## 2.1 Finding the Cycle Length

Once slow and fast have met inside the cycle, finding the length is simple: **keep one pointer stationary, advance the other until they meet again**.

### Algorithm

```
function cycleLength(meetingNode):
    count = 1
    current = meetingNode.next
    
    while current ≠ meetingNode:
        count = count + 1
        current = current.next
    
    return count
```

### Visual Trace

```
  Cycle: 3 → 4 → 5 → 6 → 3  (length = 4)
  
  Meeting point (from Floyd's): Node 5

  Step 1: current = Node(6), count = 1
  Step 2: current = Node(3), count = 2
  Step 3: current = Node(4), count = 3
  Step 4: current = Node(5) == meetingNode → STOP

  Return 4  ✓
```

### Diagram

```
  meeting
     │
     ▼
  → [3] → [4] → [5] → [6] →
  ↑   ↑              │
  │   └──────────────┘
  │        cycle length = 4
  └─ keep going until back to meeting point
```

### Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(C) | C = cycle length, traverse cycle once |
| Space | O(1) | One pointer |

---

## 2.2 Finding the Cycle Start Point

This is the most **elegant** part of Floyd's algorithm.

### The Problem

```
  HEAD
   │
   ▼
  [1] → [2] → [3] → [4] → [5] → [6]
                ↑                   │
                └───────────────────┘

  Question: Which node is the START of the cycle? → Node 3
```

### The Algorithm

After Floyd's detection finds the meeting point:

1. Place one pointer at **HEAD**
2. Place another pointer at the **meeting point**
3. Move **both** pointers **one step at a time**
4. They will meet at the **cycle start**!

```
function findCycleStart(HEAD):
    // Phase 1: Detect cycle (Floyd's)
    slow = HEAD
    fast = HEAD
    
    while fast ≠ NULL and fast.next ≠ NULL:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            break                    ← Found meeting point
    
    if fast == NULL or fast.next == NULL:
        return NULL                  ← No cycle
    
    // Phase 2: Find start
    pointer1 = HEAD
    pointer2 = slow                  ← Meeting point
    
    while pointer1 ≠ pointer2:
        pointer1 = pointer1.next     ← Both move 1 step
        pointer2 = pointer2.next
    
    return pointer1                  ← Cycle start!
```

---

## 2.3 Why Does This Work? (Mathematical Proof)

```
  Let:
    D = distance from HEAD to cycle start
    K = distance from cycle start to meeting point
    C = cycle length

  HEAD ──[D nodes]──► cycle_start ──[K nodes]──► meeting_point
                           │                          │
                           ◄────── C - K nodes ───────┘

  When slow and fast meet:
    slow has traveled: D + K
    fast has traveled: D + K + nC  (n complete cycles, n ≥ 1)

  Since fast travels 2× slow's distance:
    2(D + K) = D + K + nC
    D + K = nC
    D = nC - K
    D = (n-1)C + (C - K)

  This means:
    Distance from HEAD to cycle start (D)
    = Distance from meeting point to cycle start going forward (C - K)
    + some complete cycles ((n-1)C)

  So if one pointer starts at HEAD and another at the meeting point,
  both moving 1 step at a time, they will meet at the cycle start!
```

### Visual Proof

```
  D = 3, K = 2, C = 4

  HEAD
   │
   ▼
  [1] → [2] → [3] → [4] → [5] → [6]
         D=3    ↑                   │
                └───────────────────┘
                     C = 4

  Meeting point: Node 5 (K=2 into cycle)
  
  D = nC - K = 1×4 - 2 = 2? No, let's recalculate.
  D = 2 (nodes 1, 2 before cycle start at 3)
  
  Actually: D = 2, K = 2, C = 4
  Check: D = nC - K → 2 = 4 - 2 = 2  ✓

  Phase 2:
  pointer1 starts at HEAD (Node 1)
  pointer2 starts at meeting point (Node 5)

  Step 1: p1 → Node 2,  p2 → Node 6
  Step 2: p1 → Node 3,  p2 → Node 3  ← MEET at cycle start! ✓
```

---

## 2.4 Complete Trace: Cycle Start Detection

**List:** `A → B → C → D → E → C` (cycle start at C)

```
  Phase 1: Floyd's Detection
  ──────────────────────────
  D = 2 (A, B before cycle), C_len = 3 (C, D, E)

  HEAD
   │
   ▼
  [A] → [B] → [C] → [D] → [E]
                ↑              │
                └──────────────┘

  | Step | slow | fast |
  |------|------|------|
  | Init | A    | A    |
  | 1    | B    | C    |
  | 2    | C    | E    |
  | 3    | D    | D    | ← MEET!

  Meeting point: D

  Phase 2: Find Start
  ────────────────────
  pointer1 = HEAD (A)
  pointer2 = Meeting point (D)

  | Step | p1 | p2 |
  |------|----|----|
  | 1    | B  | E  |
  | 2    | C  | C  | ← MEET at cycle start!

  Cycle start: C  ✓
```

---

## 2.5 Removing a Cycle

Once you know where the cycle starts, you can **break** it:

### Algorithm

```
function removeCycle(HEAD):
    // Step 1: Detect and find cycle start
    cycleStart = findCycleStart(HEAD)
    if cycleStart == NULL: return    ← No cycle
    
    // Step 2: Find the node whose next is cycleStart
    current = cycleStart
    while current.next ≠ cycleStart:
        current = current.next
    
    // Step 3: Break the cycle
    current.next = NULL
```

### Diagram

```
  Before:
  [1] → [2] → [3] → [4] → [5]
                ↑              │
                └──────────────┘

  cycleStart = Node(3)
  Traverse cycle: 3 → 4 → 5 → 5.next = 3 → STOP at Node(5)
  
  Set Node(5).next = NULL

  After:
  [1] → [2] → [3] → [4] → [5] → NULL
```

---

## 2.6 Alternative Method: Without Finding Start

You can detect the node *before* the cycle start during Phase 2:

```
function removeCycleAlt(HEAD):
    slow = HEAD, fast = HEAD
    
    // Phase 1: Find meeting point
    while fast ≠ NULL and fast.next ≠ NULL:
        slow = slow.next
        fast = fast.next.next
        if slow == fast: break
    
    if fast == NULL or fast.next == NULL: return   ← No cycle
    
    // Phase 2: One from HEAD, one from meeting point
    pointer1 = HEAD
    pointer2 = slow
    
    // Special case: cycle start is HEAD
    if pointer1 == pointer2:
        while pointer2.next ≠ pointer1:
            pointer2 = pointer2.next
        pointer2.next = NULL
        return
    
    // Move both until their NEXT nodes match
    while pointer1.next ≠ pointer2.next:
        pointer1 = pointer1.next
        pointer2 = pointer2.next
    
    // pointer2.next is the cycle start — break it
    pointer2.next = NULL
```

---

## 2.7 Summary of All Cycle Operations

```
  Full pipeline:

  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │  1. Detect   │────►│  2. Find     │────►│  3. Remove   │
  │  cycle       │     │  cycle start │     │  cycle       │
  │  (Floyd's)   │     │  (two-ptr)   │     │  (set NULL)  │
  └──────────────┘     └──────────────┘     └──────────────┘
   O(n), O(1)           O(n), O(1)           O(C), O(1)
```

---

## 2.8 Practice: Cycle Detection Variants

| Variant | Approach |
|---------|----------|
| Detect if cycle exists | Floyd's slow/fast |
| Find cycle length | Traverse from meeting point back to itself |
| Find cycle start | Reset one pointer to HEAD, both move at speed 1 |
| Remove the cycle | Find node before cycle start, set its next to NULL |
| Find cycle entry and length | Combine all above |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Cycle length | From meeting point, count steps back to meeting point |
| Cycle start | One pointer at HEAD, one at meeting point, both move 1 step |
| Why it works | D = nC - K → distances align at cycle start |
| Remove cycle | Find the node with `next = cycleStart`, set `next = NULL` |
| Full pipeline | Detect → Find start → Find length → Remove |
| All operations | O(n) time, O(1) space |

---

## Quick Revision Questions

1. **How do you find the length of a cycle after Floyd's detection?**
   <details><summary>Answer</summary>Keep one pointer at the meeting point. Advance another pointer from the meeting point, counting steps until it returns to the meeting point. The count equals the cycle length.</details>

2. **Explain the Phase 2 of finding the cycle start node.**
   <details><summary>Answer</summary>Place one pointer at HEAD and another at the meeting point. Move both one step at a time. They meet at the cycle start because the distance from HEAD to cycle start (D) equals the distance from meeting point to cycle start going forward.</details>

3. **Prove mathematically: why does D = nC - K?**
   <details><summary>Answer</summary>Slow travels D+K. Fast travels D+K+nC (n full cycles extra). Since fast = 2×slow: 2(D+K) = D+K+nC → D+K = nC → D = nC-K. This means D steps from HEAD equals traversing forward from the meeting point and arriving at the cycle start.</details>

4. **How do you remove a cycle from a linked list?**
   <details><summary>Answer</summary>Find the cycle start node. Then traverse the cycle to find the node whose `next` is the cycle start. Set that node's `next` to NULL.</details>

5. **What is the total time complexity for detecting a cycle and finding its start?**
   <details><summary>Answer</summary>O(n) total. Phase 1 (detection) is O(n), Phase 2 (finding start) is O(D + C - K) which is also O(n). Combined: O(n).</details>

6. **What happens in Phase 2 if the cycle start is the HEAD node itself?**
   <details><summary>Answer</summary>Both pointers start at HEAD (which is the cycle start), so they immediately match. This is a special case — pointer1 == pointer2 from the start. The cycle start is HEAD. To remove it, traverse the cycle to find the node pointing back to HEAD and set its next to NULL.</details>

---

[← Previous: Concept and Cycle Detection](01-concept-and-cycle-detection.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Middle Node and Nth from End →](03-middle-node-and-nth-from-end.md)

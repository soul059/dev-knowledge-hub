# Chapter 1: Concept and Cycle Detection (Floyd's Algorithm)

[← Previous: Use Cases and Problems](../04-Circular-Linked-List/03-use-cases-and-problems.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Cycle Length and Start Point →](02-cycle-length-and-start.md)

---

## Chapter Overview

The **fast and slow pointer** technique (also called the **tortoise and hare** algorithm) is one of the most powerful patterns in linked list problems. This chapter introduces the concept and covers **Floyd's Cycle Detection Algorithm** in depth.

---

## 1.1 The Two-Pointer Concept

Use two pointers that move at **different speeds** through the list:

- **Slow pointer**: moves **1 step** at a time
- **Fast pointer**: moves **2 steps** at a time

```
  Initial:
  slow  fast
   │     │
   ▼     ▼
  [1] → [2] → [3] → [4] → [5] → [6] → NULL

  After 1 step:
        slow        fast
         │           │
         ▼           ▼
  [1] → [2] → [3] → [4] → [5] → [6] → NULL

  After 2 steps:
               slow              fast
                │                 │
                ▼                 ▼
  [1] → [2] → [3] → [4] → [5] → [6] → NULL

  After 3 steps:
                      slow                    fast
                       │                       │
                       ▼                       ▼
  [1] → [2] → [3] → [4] → [5] → [6] → NULL
```

### Key Insight

In a list of length `n`:
- When fast reaches the end (or NULL), slow is at position `n/2` → **middle of the list**!
- If there's a cycle, fast and slow will **eventually meet** inside the cycle.

---

## 1.2 Why Do They Meet in a Cycle?

### The Intuition

Think of two runners on a circular track:
- Runner A runs at speed 1
- Runner B runs at speed 2

```
  Circular track:

       ╭───────╮
      ╱    →    ╲
     │  Runner B  │    Runner B is FASTER
     │     ↑      │    → B gains 1 position per step on A
      ╲   Runner A╱    → Eventually B laps A (they meet!)
       ╰───────╯
```

**Every step, the fast pointer gets 1 step closer to the slow pointer** (relative to the cycle). They MUST meet.

### Mathematical Proof

```
  Let cycle length = C
  
  When slow enters the cycle:
    - Let fast be F steps ahead of slow (within the cycle)
    - Relative speed of fast towards slow = 2 - 1 = 1 step/iteration
    - Distance to close = C - F
    - They meet after (C - F) iterations
  
  Since C - F < C, they meet within ONE loop of the cycle.
```

---

## 1.3 Cycle Detection: Floyd's Algorithm

### Problem

Given a linked list, determine if it contains a **cycle** (a node's next points back to a previous node).

### List with Cycle

```
  HEAD
   │
   ▼
  [1] → [2] → [3] → [4] → [5]
                ↑              │
                └──────────────┘
                
  Node 5's next points to Node 3 → CYCLE!
  Cycle: 3 → 4 → 5 → 3 → 4 → 5 → ...
```

### Algorithm

```
function hasCycle(HEAD):
    if HEAD == NULL or HEAD.next == NULL:
        return false
    
    slow = HEAD
    fast = HEAD
    
    while fast ≠ NULL and fast.next ≠ NULL:
        slow = slow.next             ← Move 1 step
        fast = fast.next.next        ← Move 2 steps
        
        if slow == fast:             ← They met!
            return true              ← Cycle exists
    
    return false                     ← Fast reached NULL → no cycle
```

### Step-by-Step Trace

**List:** `1 → 2 → 3 → 4 → 5 → 3` (cycle at node 3)

| Step | slow | fast | Meet? |
|------|------|------|-------|
| Init | 1 | 1 | — |
| 1 | 2 | 3 | No |
| 2 | 3 | 5 | No |
| 3 | 4 | 4 | **YES!** |

**Cycle detected in 3 steps!**

### Trace on a List WITHOUT Cycle

**List:** `1 → 2 → 3 → 4 → 5 → NULL`

| Step | slow | fast | fast.next |
|------|------|------|-----------|
| Init | 1 | 1 | — |
| 1 | 2 | 3 | 4 |
| 2 | 3 | 5 | NULL |
| 3 | — | — | fast.next = NULL → EXIT |

**Return false** — no cycle.

---

## 1.4 Detailed Visual Trace with Cycle

```
  List: 1 → 2 → 3 → 4 → 5 → 6 → 3 (cycle back to 3)
  Cycle: 3 → 4 → 5 → 6 → 3

  Step 0 (Init):
  S,F
   │
   ▼
  [1] → [2] → [3] → [4] → [5] → [6]
                ↑                   │
                └───────────────────┘

  Step 1: slow→2, fast→3
         S         F
         │         │
         ▼         ▼
  [1] → [2] → [3] → [4] → [5] → [6]
                ↑                   │
                └───────────────────┘

  Step 2: slow→3, fast→5
                S               F
                │               │
                ▼               ▼
  [1] → [2] → [3] → [4] → [5] → [6]
                ↑                   │
                └───────────────────┘

  Step 3: slow→4, fast→3 (6→3, then 3→4, fast is at 3 after 6)
  Wait, let me retrace: fast was at 5, moves 2: 5→6→3
                
                F    S
                │    │
                ▼    ▼
  [1] → [2] → [3] → [4] → [5] → [6]
                ↑                   │
                └───────────────────┘

  Step 4: slow→5, fast→5
  
                              S,F
                               │
                               ▼
  [1] → [2] → [3] → [4] → [5] → [6]
                ↑                   │
                └───────────────────┘

  MEET at node 5! → Cycle detected ✓
```

---

## 1.5 Why Check `fast ≠ NULL AND fast.next ≠ NULL`?

```
  Fast moves 2 steps. Before moving, we need:
  
  1. fast ≠ NULL        → fast exists (for even-length lists)
  2. fast.next ≠ NULL   → fast.next exists (so fast.next.next is valid)

  Without these checks:
  
  Odd length:  [1] → [2] → [3] → NULL
    fast at [3]: fast.next = NULL → fast.next.next crashes!
  
  Even length: [1] → [2] → [3] → [4] → NULL
    fast at [4]: fast = NULL after next.next → accessing NULL crashes!
```

---

## 1.6 Alternative: HashSet Approach

```
function hasCycleHash(HEAD):
    visited = empty HashSet
    current = HEAD
    
    while current ≠ NULL:
        if current in visited:
            return true          ← Already seen this node!
        visited.add(current)
        current = current.next
    
    return false
```

### Comparison

| Approach | Time | Space | Modifies List? |
|----------|------|-------|----------------|
| Floyd's (slow/fast) | O(n) | O(1) | No |
| HashSet | O(n) | O(n) | No |
| Mark visited (modify node) | O(n) | O(1) | Yes (destructive) |

> Floyd's is preferred — O(n) time with O(1) space.

---

## 1.7 Edge Cases for Cycle Detection

```
  Case 1: Empty list
  HEAD = NULL → return false immediately

  Case 2: Single node, no cycle
  [1] → NULL → fast.next = NULL → return false

  Case 3: Single node, self-loop
  [1] → [1] (next points to itself)
  Step 1: slow = 1, fast = 1 → MEET → return true

  Case 4: Cycle at the HEAD
  [1] → [2] → [3] → [1]
  Standard Floyd's works fine

  Case 5: Cycle at the TAIL (last two nodes loop)
  [1] → [2] → [3] → [4] → [3]
  Standard Floyd's works fine

  Case 6: Very long tail before cycle
  [1] → [2] → ... → [1000] → [500]
  Floyd's still works — just takes more steps
```

---

## 1.8 Pattern Recognition: When to Use Slow/Fast

| Problem | How Slow/Fast Helps |
|---------|-------------------|
| Cycle detection | Fast catches slow in cycle |
| Find middle | When fast reaches end, slow is at middle |
| Cycle start | Combine with second phase (next chapter) |
| Palindrome check | Find middle to split and reverse half |
| Nth from end | Start fast n steps ahead |
| Linked list intersection | Equalize lengths via speed difference |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Slow pointer | Moves 1 step per iteration |
| Fast pointer | Moves 2 steps per iteration |
| Cycle detection | If slow == fast, cycle exists |
| No cycle | Fast reaches NULL → no cycle |
| Floyd's algorithm | O(n) time, O(1) space |
| HashSet alternative | O(n) time, O(n) space |
| Why they meet | Fast gains 1 step/iteration in cycle → guaranteed meeting |
| Check before moving | Verify `fast ≠ NULL` and `fast.next ≠ NULL` |

---

## Quick Revision Questions

1. **Why does the fast pointer move exactly 2 steps (not 3 or more)?**
   <details><summary>Answer</summary>With speed 2, the relative speed difference is 1 — guaranteeing they meet at every position in the cycle. With higher speeds (e.g., 3), the relative speed is 2, and they might skip over each other in certain cycle lengths (though technically speed 2 is simplest and always works).</details>

2. **What is the time complexity of Floyd's cycle detection?**
   <details><summary>Answer</summary>O(n). The slow pointer traverses at most n nodes. The fast pointer traverses at most 2n nodes (it enters the cycle and catches up within one cycle length). Both are O(n).</details>

3. **Why is Floyd's preferred over the HashSet approach?**
   <details><summary>Answer</summary>Floyd's uses O(1) space (just two pointers) vs O(n) space for the HashSet. Both are O(n) time, but Floyd's is more memory efficient.</details>

4. **What happens if you don't check `fast.next ≠ NULL` before `fast = fast.next.next`?**
   <details><summary>Answer</summary>If `fast.next` is NULL, accessing `fast.next.next` dereferences a null pointer — causing a crash (NullPointerException or segfault).</details>

5. **Can Floyd's algorithm detect a cycle in a doubly linked list?**
   <details><summary>Answer</summary>Yes, the same algorithm works. You only use `next` pointers for forward movement. The `prev` pointers don't interfere.</details>

6. **If a list has n=100 nodes and the cycle starts at node 60 with length 40, approximately how many steps until detection?**
   <details><summary>Answer</summary>Slow enters the cycle at step 60. Fast is somewhere in the cycle. The relative distance closes at 1 step/iteration, so within 40 more steps (cycle length), they meet. Total: approximately 60 + 40 = 100 steps, which is O(n).</details>

---

[← Previous: Use Cases and Problems](../04-Circular-Linked-List/03-use-cases-and-problems.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Cycle Length and Start Point →](02-cycle-length-and-start.md)

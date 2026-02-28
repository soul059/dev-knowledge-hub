# Chapter 1: Iterative and Recursive Reversal

[← Previous: Middle Node and Nth from End](../05-Fast-And-Slow-Pointers/03-middle-node-and-nth-from-end.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Reverse in Groups →](02-reverse-in-groups.md)

---

## Chapter Overview

Reversing a linked list is one of the most fundamental operations. This chapter covers the iterative three-pointer technique and the recursive call-stack approach, with detailed visual traces for both.

---

## 1.1 Why Reversal Matters

Reversal appears in numerous problems:
- Checking if a list is a **palindrome**
- Adding two numbers represented as linked lists
- **Reverse in groups** (K-group reversal)
- Undoing operations / backtracking

---

## 1.2 Iterative Reversal — Three Pointer Technique

### The Core Idea

Walk through the list, flipping each `next` pointer to point **backward**.

Three pointers:
- **prev**: the node behind the current node (starts NULL)
- **curr**: the node we're processing
- **next**: saved reference to the next node (so we don't lose it)

### Algorithm

```
function reverseIterative(HEAD):
    prev = NULL
    curr = HEAD
    
    while curr ≠ NULL:
        next = curr.next         ← 1. Save next
        curr.next = prev         ← 2. Reverse link
        prev = curr              ← 3. Move prev forward
        curr = next              ← 4. Move curr forward
    
    return prev                  ← prev is the new HEAD
```

### Step-by-Step Trace: 1 → 2 → 3 → 4 → NULL

```
  Initial:
  prev = NULL    curr = 1
  NULL   [1] → [2] → [3] → [4] → NULL
          ↑
        curr

  ─── Step 1 ───
  next = 2
  curr.next = prev → 1.next = NULL
  prev = 1, curr = 2

  NULL ← [1]   [2] → [3] → [4] → NULL
          ↑     ↑
        prev  curr

  ─── Step 2 ───
  next = 3
  curr.next = prev → 2.next = 1
  prev = 2, curr = 3

  NULL ← [1] ← [2]   [3] → [4] → NULL
                 ↑     ↑
               prev  curr

  ─── Step 3 ───
  next = 4
  curr.next = prev → 3.next = 2
  prev = 3, curr = 4

  NULL ← [1] ← [2] ← [3]   [4] → NULL
                        ↑     ↑
                      prev  curr

  ─── Step 4 ───
  next = NULL
  curr.next = prev → 4.next = 3
  prev = 4, curr = NULL

  NULL ← [1] ← [2] ← [3] ← [4]
                              ↑
                            prev = NEW HEAD

  Result: 4 → 3 → 2 → 1 → NULL ✓
```

### Memory Diagram

```
  Before:
  HEAD → ┌───┬───┐   ┌───┬───┐   ┌───┬───┐   ┌───┬──────┐
         │ 1 │ ──┼──►│ 2 │ ──┼──►│ 3 │ ──┼──►│ 4 │ NULL │
         └───┴───┘   └───┴───┘   └───┴───┘   └───┴──────┘

  After:
  ┌──────┬───┐   ┌───┬───┐   ┌───┬───┐   ┌───┬───┐ ← HEAD
  │ NULL │ 1 │◄──┼── │ 2 │◄──┼── │ 3 │◄──┼── │ 4 │
  └──────┴───┘   └───┴───┘   └───┴───┘   └───┴───┘

  Same nodes, same memory — only the .next pointers changed!
```

---

## 1.3 Recursive Reversal

### The Intuition

Think recursively: **reverse the rest first**, then fix the current node.

```
  reverse(1 → 2 → 3 → 4)
  = reverse(2 → 3 → 4)  then attach 1
  = reverse(3 → 4)       then attach 2
  = reverse(4)            then attach 3
  = 4                     base case!
  
  Unwind:
  4 → 3 → 2 → 1 → NULL
```

### Algorithm

```
function reverseRecursive(HEAD):
    // Base case
    if HEAD == NULL or HEAD.next == NULL:
        return HEAD
    
    // Recurse: reverse the rest
    newHead = reverseRecursive(HEAD.next)
    
    // Fix: make next node point back to current
    HEAD.next.next = HEAD
    HEAD.next = NULL
    
    return newHead
```

### The Tricky Part: HEAD.next.next = HEAD

```
  Before fixing node 3:
  
  [3] ──► [4] ──► NULL        (4 is already the new head)
   ↑
  HEAD = 3

  HEAD.next = Node(4)
  HEAD.next.next = HEAD → Node(4).next = Node(3)
  HEAD.next = NULL → Node(3).next = NULL

  After:
  NULL ◄── [3] ◄── [4]
                    ↑
                 newHead
```

### Full Recursive Trace: 1 → 2 → 3

```
  Call Stack:
  
  reverse(1)
  │  reverse(2)
  │  │  reverse(3) → base case, return 3 (newHead)
  │  │
  │  │  Back in reverse(2):
  │  │    newHead = 3
  │  │    HEAD=2, HEAD.next=3
  │  │    HEAD.next.next = HEAD → 3.next = 2
  │  │    HEAD.next = NULL → 2.next = NULL
  │  │    
  │  │    State: 3 → 2 → NULL
  │  │    return newHead = 3
  │  │
  │  Back in reverse(1):
  │    newHead = 3
  │    HEAD=1, HEAD.next=2
  │    HEAD.next.next = HEAD → 2.next = 1
  │    HEAD.next = NULL → 1.next = NULL
  │    
  │    State: 3 → 2 → 1 → NULL
  │    return newHead = 3

  Final: HEAD = 3 → 2 → 1 → NULL ✓
```

### Call Stack Visualization

```
  ┌──────────────────────────┐
  │ reverse(3)  → return 3   │  ← Top of stack (resolves first)
  ├──────────────────────────┤
  │ reverse(2)  → 3→2→NULL  │
  ├──────────────────────────┤
  │ reverse(1)  → 3→2→1→NULL│  ← Bottom of stack (resolves last)
  └──────────────────────────┘
```

---

## 1.4 Iterative vs Recursive — Head-to-Head

| Aspect | Iterative | Recursive |
|--------|-----------|-----------|
| Time | O(n) | O(n) |
| Space | O(1) | O(n) — call stack |
| Stack overflow risk? | No | Yes, for long lists |
| Readability | Moderate | Elegant |
| Debug ease | Easy — print pointers | Hard — deep call stack |
| Preferred for production | ✅ Yes | Usually not |

### When to Use Which

```
  ┌─────────────────────────────────────────┐
  │ Use ITERATIVE when:                     │
  │   • List could be very long             │
  │   • Memory is constrained               │
  │   • Performance-critical code           │
  ├─────────────────────────────────────────┤
  │ Use RECURSIVE when:                     │
  │   • List is short / bounded             │
  │   • Part of a recursive algorithm       │
  │   • Coding interview prefers both       │
  └─────────────────────────────────────────┘
```

---

## 1.5 Tail-Recursive Reversal (Optimization)

Convert the recursive solution to tail-recursive to avoid stack buildup:

```
function reverseTailRecursive(curr, prev = NULL):
    if curr == NULL:
        return prev                  ← prev is the new HEAD
    
    next = curr.next
    curr.next = prev
    return reverseTailRecursive(next, curr)    ← Tail call
```

```
  This mirrors the iterative logic!
  Languages with tail-call optimization (TCO) treat this as O(1) space.
  Note: Java, Python do NOT have TCO. C, Scheme, Kotlin (tailrec) do.
```

---

## 1.6 Reverse a Sublist (Positions m to n)

Reverse only nodes from position m to n.

```
  Before: 1 → 2 → 3 → 4 → 5    (reverse positions 2 to 4)
  After:  1 → 4 → 3 → 2 → 5
```

### Algorithm

```
function reverseSublist(HEAD, m, n):
    if m == n: return HEAD
    
    dummy = createNode(0)
    dummy.next = HEAD
    
    // Step 1: Find node before position m
    pre = dummy
    for i = 1 to m-1:
        pre = pre.next
    
    // Step 2: Reverse from m to n
    curr = pre.next
    for i = 1 to (n - m):
        next = curr.next
        curr.next = next.next
        next.next = pre.next
        pre.next = next
    
    return dummy.next
```

### Trace: Reverse positions 2 to 4 in [1, 2, 3, 4, 5]

```
  After Step 1: pre = Node(1), curr = Node(2)
  
  ─── i=1 ───
  next = 3
  curr.next = 3.next = 4   → [2]→[4]
  next.next = pre.next = 2  → [3]→[2]
  pre.next = next = 3       → [1]→[3]
  
  State: 1 → 3 → 2 → 4 → 5
  
  ─── i=2 ───
  next = 4  (curr is still 2, curr.next is now 4)
  curr.next = 4.next = 5   → [2]→[5]
  next.next = pre.next = 3  → [4]→[3]
  pre.next = next = 4       → [1]→[4]
  
  State: 1 → 4 → 3 → 2 → 5 ✓
```

---

## 1.7 Common Bugs and Edge Cases

| Bug | Symptom | Fix |
|-----|---------|-----|
| Forgetting `curr.next = NULL` in recursion | Cycle! Original head still points forward | Always set `HEAD.next = NULL` |
| Not returning `prev` / `newHead` | Lose reference to reversed list | The last node processed is the new head |
| Off-by-one in sublist reversal | Wrong nodes reversed | Use dummy node, careful counting |
| Not handling single node | Works correctly — base case handles it | Verify: HEAD.next == NULL → return HEAD |
| Not handling empty list | NULL dereference | Check HEAD == NULL first |

---

## Summary Table

| Operation | Approach | Time | Space | Notes |
|-----------|----------|------|-------|-------|
| Full reversal (iterative) | 3 pointers (prev, curr, next) | O(n) | O(1) | Production-preferred |
| Full reversal (recursive) | Recurse rest, fix HEAD.next.next | O(n) | O(n) | Elegant but stack risk |
| Tail-recursive reversal | Carry prev as parameter | O(n) | O(1)* | *With TCO only |
| Reverse sublist (m to n) | Fix pointers with pre anchor | O(n) | O(1) | Use dummy node |

---

## Quick Revision Questions

1. **What are the three pointers needed for iterative reversal and what role does each play?**
   <details><summary>Answer</summary>`prev` — the node behind curr (what curr.next will point to). `curr` — the node being processed. `next` — saved reference to curr.next before we overwrite it. Without saving `next`, we'd lose the rest of the list.</details>

2. **What does `HEAD.next.next = HEAD` do in the recursive approach?**
   <details><summary>Answer</summary>It makes the node AFTER HEAD point back to HEAD. If HEAD=2 and HEAD.next=3, then `HEAD.next.next = HEAD` means `3.next = 2`, reversing the link between nodes 2 and 3.</details>

3. **Why does recursive reversal use O(n) space?**
   <details><summary>Answer</summary>Each recursive call adds a frame to the call stack. For n nodes, there are n recursive calls, so the stack grows to depth n, using O(n) space.</details>

4. **In sublist reversal, why do we use a dummy node?**
   <details><summary>Answer</summary>If m=1, the HEAD itself is part of the reversed sublist. The dummy node provides a stable anchor before position 1, so `pre` always has a valid node even when reversing from the start.</details>

5. **What goes wrong if you forget `HEAD.next = NULL` in recursive reversal?**
   <details><summary>Answer</summary>The original first node still has its `next` pointing to the original second node (which now points back to it). This creates a cycle: the last two nodes point to each other in an infinite loop.</details>

6. **How do you reverse a linked list in one line using iterative logic in Python?**
   <details><summary>Answer</summary>Not truly one line, but using tuple unpacking in the loop: `prev, curr = curr, next` after `next, curr.next = curr.next, prev`. The key is simultaneous assignment to avoid temp variables.</details>

---

[← Previous: Middle Node and Nth from End](../05-Fast-And-Slow-Pointers/03-middle-node-and-nth-from-end.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Reverse in Groups →](02-reverse-in-groups.md)

# Chapter 2: Head, Tail, and Traversal

[← Previous: What is a Linked List?](01-what-is-a-linked-list.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Array vs Linked List →](03-array-vs-linked-list.md)

---

## Chapter Overview

This chapter covers the **fundamental pointers** that manage a linked list (`HEAD` and `TAIL`) and the essential operation of **traversal** — visiting every node in the list. Traversal is the backbone of nearly every linked list algorithm.

---

## 2.1 The HEAD Pointer

The `HEAD` is a pointer variable that **always points to the first node** of the linked list. It is YOUR entry point into the list.

```
  HEAD
   │
   ▼
 ┌──────┬───┐    ┌──────┬───┐    ┌──────┬──────┐
 │  10  │ ──┼───►│  20  │ ──┼───►│  30  │ NULL │
 └──────┴───┘    └──────┴───┘    └──────┴──────┘
```

### Key Rules About HEAD

| Rule | Explanation |
|------|-------------|
| HEAD is NOT a node | It's a **pointer variable** that stores the address of the first node |
| HEAD = NULL → empty list | If HEAD is NULL, there are no nodes |
| Losing HEAD = losing the list | If you overwrite HEAD without saving it, the entire list is lost |
| Most operations start at HEAD | Traversal, search, insertion — all begin from HEAD |

### Pseudocode: Initialize a Linked List

```
function createLinkedList():
    HEAD = NULL        ← empty list
    return HEAD
```

---

## 2.2 The TAIL Pointer

The `TAIL` is an **optional** pointer that points to the **last node** of the list.

```
  HEAD                                       TAIL
   │                                          │
   ▼                                          ▼
 ┌──────┬───┐    ┌──────┬───┐    ┌──────┬──────┐
 │  10  │ ──┼───►│  20  │ ──┼───►│  30  │ NULL │
 └──────┴───┘    └──────┴───┘    └──────┴──────┘
```

### Why Maintain a TAIL Pointer?

Without TAIL: To insert at the end, you must **traverse the entire list** → O(n)  
With TAIL: You can insert at the end **directly** → O(1)

```
Without TAIL — Insert at end:
───────────────────────────────
  Start at HEAD → traverse → traverse → ... → reach last node → insert
  Time: O(n)

With TAIL — Insert at end:
───────────────────────────────
  Go directly to TAIL → insert after it → update TAIL
  Time: O(1)
```

### Trade-off of Maintaining TAIL

| Aspect | Without TAIL | With TAIL |
|--------|-------------|-----------|
| Insert at end | O(n) | O(1) |
| Delete from end | O(n)* | O(n)* |
| Extra memory | None | One pointer |
| Code complexity | Simpler | Slightly more bookkeeping |

\* Even with TAIL, deleting the last node in a **singly** linked list is O(n) because you need the **second-to-last** node to update its `next` to NULL. TAIL doesn't help here — you'd need a **doubly linked list**.

---

## 2.3 Traversal — The Fundamental Operation

**Traversal** means visiting every node in the list, one by one, from HEAD to the end.

### ASCII Diagram: Traversal

```
  Step 1:  current = HEAD
           │
           ▼
          [10] ──► [20] ──► [30] ──► [40] ──► NULL
           ^^
           Visit: 10

  Step 2:  current = current.next
                     │
                     ▼
          [10] ──► [20] ──► [30] ──► [40] ──► NULL
                    ^^
                    Visit: 20

  Step 3:  current = current.next
                               │
                               ▼
          [10] ──► [20] ──► [30] ──► [40] ──► NULL
                             ^^
                             Visit: 30

  Step 4:  current = current.next
                                        │
                                        ▼
          [10] ──► [20] ──► [30] ──► [40] ──► NULL
                                      ^^
                                      Visit: 40

  Step 5:  current = current.next = NULL
           STOP! We've reached the end.
```

### Pseudocode: Traversal (Print All Elements)

```
function traverse(HEAD):
    current = HEAD
    
    while current ≠ NULL:
        print(current.data)
        current = current.next
    
    // current is now NULL — we've visited all nodes
```

### Step-by-Step Trace

Given list: `10 → 20 → 30 → NULL`

| Step | current | current.data | current.next | Action |
|------|---------|-------------|--------------|--------|
| Init | 0x100 (Node 10) | 10 | 0x250 | — |
| 1 | 0x100 | 10 | 0x250 | Print 10, move to next |
| 2 | 0x250 | 20 | 0x180 | Print 20, move to next |
| 3 | 0x180 | 30 | NULL | Print 30, move to next |
| 4 | NULL | — | — | Loop ends |

**Output:** `10 20 30`

### Time & Space Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(n) | Visit each of `n` nodes exactly once |
| Space | O(1) | Only one pointer variable (`current`) used |

---

## 2.4 Counting Nodes (Length Calculation)

A variation of traversal — count the nodes instead of printing them.

### Pseudocode: Count Nodes

```
function length(HEAD):
    count = 0
    current = HEAD
    
    while current ≠ NULL:
        count = count + 1
        current = current.next
    
    return count
```

### Step-by-Step Trace

List: `5 → 15 → 25 → 35 → NULL`

| Step | current | count | Action |
|------|---------|-------|--------|
| Init | Node(5) | 0 | — |
| 1 | Node(5) | 1 | count++, move |
| 2 | Node(15) | 2 | count++, move |
| 3 | Node(25) | 3 | count++, move |
| 4 | Node(35) | 4 | count++, move |
| 5 | NULL | 4 | Exit loop |

**Return:** `4`

---

## 2.5 Searching for a Value

Another traversal variation — stop when you find the target value.

### Pseudocode: Search

```
function search(HEAD, target):
    current = HEAD
    position = 0
    
    while current ≠ NULL:
        if current.data == target:
            return position          ← Found! Return index
        current = current.next
        position = position + 1
    
    return -1                        ← Not found
```

### Step-by-Step Trace: Search for 25

List: `10 → 20 → 25 → 30 → NULL`

| Step | current.data | target | Match? | position |
|------|-------------|--------|--------|----------|
| 1 | 10 | 25 | No | 0 |
| 2 | 20 | 25 | No | 1 |
| 3 | 25 | 25 | **YES** | 2 |

**Return:** `2` (found at index 2)

### Search Complexity

| Case | Time | Explanation |
|------|------|-------------|
| Best case | O(1) | Target is the first node |
| Worst case | O(n) | Target is the last node or not present |
| Average case | O(n) | On average, scan half the list |

---

## 2.6 Recursive Traversal

Traversal can also be done **recursively** — useful for building intuition with recursion.

### Pseudocode: Recursive Print

```
function printList(node):
    if node == NULL:
        return                    ← Base case
    
    print(node.data)              ← Process current node
    printList(node.next)          ← Recurse on rest of list
```

### Call Stack Visualization

List: `10 → 20 → 30 → NULL`

```
printList(Node 10)
  │ print 10
  └─► printList(Node 20)
        │ print 20
        └─► printList(Node 30)
              │ print 30
              └─► printList(NULL)
                    │ return (base case)
                    │
              return ◄──┘
        return ◄──┘
  return ◄──┘
```

**Output:** `10 20 30`

### Reverse Print (Recursive Trick)

By moving `print` AFTER the recursive call, you print in **reverse order**:

```
function printReverse(node):
    if node == NULL:
        return
    
    printReverse(node.next)       ← Recurse FIRST
    print(node.data)              ← Print AFTER returning
```

### Call Stack for Reverse Print

```
printReverse(Node 10)
  └─► printReverse(Node 20)
        └─► printReverse(Node 30)
              └─► printReverse(NULL)
                    return
              print 30  ◄── prints while unwinding
        print 20  ◄──
  print 10  ◄──
```

**Output:** `30 20 10`

### Recursive Traversal Complexity

| Metric | Value | Reason |
|--------|-------|--------|
| Time | O(n) | Visit each node once |
| Space | O(n) | Recursion stack holds `n` frames |

> **Warning:** Recursive traversal uses O(n) stack space. For very long lists, this can cause **stack overflow**. Prefer iterative traversal in production code.

---

## 2.7 Common Traversal Patterns

### Pattern: "Runner" Approach

Use a secondary pointer that runs ahead:

```
function hasDuplicate(HEAD):
    current = HEAD
    while current ≠ NULL:
        runner = current.next
        while runner ≠ NULL:
            if runner.data == current.data:
                return true
            runner = runner.next
        current = current.next
    return false
```

### Pattern: "Previous" Pointer

Many operations need a pointer to the node BEFORE the target:

```
  prev    current
   │        │
   ▼        ▼
  [10] ──► [20] ──► [30] ──► NULL

  To delete current (20):
    prev.next = current.next
    
  Result:
  [10] ──► [30] ──► NULL
```

### Pattern: Two-Pointer (Slow and Fast)

One pointer moves 1 step, another moves 2 steps. (Covered in detail in Unit 5.)

```
  slow     fast
   │        │
   ▼        ▼
  [1] ──► [2] ──► [3] ──► [4] ──► [5] ──► NULL

  After 1 iteration: slow at [2], fast at [3]
  After 2 iterations: slow at [3], fast at [5]
  → slow is at the MIDDLE!
```

---

## 2.8 Common Mistakes in Traversal

| Mistake | What Happens | Fix |
|---------|-------------|-----|
| Forgetting NULL check | NullPointerException / segfault | Always check `current ≠ NULL` |
| Using HEAD directly | HEAD gets modified, list is "lost" | Use a `current` variable |
| Off-by-one in count | Wrong length returned | Trace with 0 and 1 element lists |
| Infinite loop | Never checking `current.next` | Ensure `current = current.next` happens |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| HEAD | Pointer to first node — entry point to the list |
| TAIL | Optional pointer to last node — enables O(1) end insertion |
| Traversal | Visit all nodes using `current = current.next` loop |
| Length | Count nodes during traversal — O(n) time, O(1) space |
| Search | Linear scan — O(n) worst case |
| Recursive traversal | Elegant but O(n) stack space — risk of overflow |
| Reverse print | Recursion trick — print after recursive call |
| Previous pointer | Essential pattern for deletion and insertion |
| Never modify HEAD | Always use a `current` variable for traversal |

---

## Quick Revision Questions

1. **Why should you never use HEAD directly for traversal?**
   <details><summary>Answer</summary>If you modify HEAD during traversal (e.g., HEAD = HEAD.next), you lose the reference to the beginning of the list. Always use a separate `current` variable.</details>

2. **What is the advantage of maintaining a TAIL pointer?**
   <details><summary>Answer</summary>It allows O(1) insertion at the end of the list instead of O(n) traversal to find the last node.</details>

3. **What is the time and space complexity of iterative traversal?**
   <details><summary>Answer</summary>Time: O(n), Space: O(1). You visit each node once and use only one extra pointer.</details>

4. **How can you print a linked list in reverse order using recursion?**
   <details><summary>Answer</summary>Recurse to the end of the list first, then print the node's data as the recursive calls unwind (place `print` after the recursive call).</details>

5. **Why is deletion from the end O(n) even with a TAIL pointer in a singly linked list?**
   <details><summary>Answer</summary>To delete the last node, you need to update the second-to-last node's `next` to NULL. In a singly linked list, you can't go backward from TAIL — you must traverse from HEAD to find the second-to-last node.</details>

6. **What happens if you traverse a linked list without checking for NULL?**
   <details><summary>Answer</summary>You'll attempt to access `next` on a NULL pointer, causing a NullPointerException (Java/Python) or segmentation fault (C/C++).</details>

---

[← Previous: What is a Linked List?](01-what-is-a-linked-list.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Array vs Linked List →](03-array-vs-linked-list.md)

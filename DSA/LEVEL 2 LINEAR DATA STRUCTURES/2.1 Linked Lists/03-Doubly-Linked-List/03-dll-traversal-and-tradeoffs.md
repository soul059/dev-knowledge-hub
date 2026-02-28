# Chapter 3: DLL Traversal and Trade-offs

[← Previous: DLL Insertion and Deletion](02-dll-insertion-and-deletion.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 4 — Circular List Types →](../04-Circular-Linked-List/01-circular-list-types.md)

---

## Chapter Overview

This chapter covers **forward and backward traversal** in a DLL, practical design trade-offs when choosing between SLL and DLL, and real-world applications where DLL shines.

---

## 3.1 Forward Traversal

Identical to SLL traversal — start from HEAD, follow `next` pointers.

```
function traverseForward(HEAD):
    current = HEAD
    while current ≠ NULL:
        print(current.data)
        current = current.next
```

### Diagram

```
  HEAD                                    TAIL
   │                                       │
   ▼         ──►       ──►       ──►       ▼
 [10]  ⇄  [20]  ⇄  [30]  ⇄  [40]  ──► NULL
  ↑
 start here, follow 'next'

 Output: 10 → 20 → 30 → 40
```

---

## 3.2 Backward Traversal

The unique capability of DLL — start from TAIL, follow `prev` pointers.

```
function traverseBackward(TAIL):
    current = TAIL
    while current ≠ NULL:
        print(current.data)
        current = current.prev
```

### Diagram

```
  HEAD                                    TAIL
   │                                       │
   ▼        ◄──       ◄──       ◄──        ▼
 [10]  ⇄  [20]  ⇄  [30]  ⇄  [40]
                                ↑
                        start here, follow 'prev'

 Output: 40 → 30 → 20 → 10
```

### Comparison with SLL Reverse Print

| Approach | Time | Space | Works? |
|----------|------|-------|--------|
| DLL backward traversal | O(n) | O(1) | Direct — follow `prev` |
| SLL recursive reverse print | O(n) | O(n) | Uses call stack |
| SLL reverse, then print | O(n) | O(1) | Modifies the list |
| SLL to array, reverse | O(n) | O(n) | Extra array needed |

> DLL wins — O(n) time, O(1) space, no modification.

---

## 3.3 Bidirectional Search

DLL can search from **both ends simultaneously**, reducing average search time.

### Algorithm

```
function bidirectionalSearch(HEAD, TAIL, target):
    front = HEAD
    back = TAIL
    
    while front ≠ NULL and back ≠ NULL:
        if front.data == target:
            return front
        if back.data == target:
            return back
        
        // Check if pointers have crossed
        if front == back or front.prev == back:
            break
        
        front = front.next
        back = back.prev
    
    return NULL   ← not found
```

### Trace: Search for 30 in [10, 20, 30, 40, 50]

```
  Step 1: front = [10], back = [50]
          10 ≠ 30, 50 ≠ 30 → continue

  Step 2: front = [20], back = [40]
          20 ≠ 30, 40 ≠ 30 → continue

  Step 3: front = [30], back = [30]
          30 == 30 → FOUND!

  Regular search: 3 steps to reach element
  Bidirectional: 3 steps, but worst case is n/2 instead of n
```

---

## 3.4 Real-World DLL Application: Browser History

```
  BACK ◄───────────── CURRENT ──────────────► FORWARD

  [Google] ⇄ [Wikipedia] ⇄ [YouTube] ⇄ [GitHub]
                              ↑
                           CURRENT

  ← Back button:   CURRENT = CURRENT.prev   → Wikipedia
  → Forward button: CURRENT = CURRENT.next   → GitHub
  
  Visit new page from Wikipedia:
    1. Delete everything after CURRENT (YouTube, GitHub)
    2. Insert new page after CURRENT
    
  [Google] ⇄ [Wikipedia] ⇄ [StackOverflow]
                              ↑
                           CURRENT
```

---

## 3.5 Real-World DLL Application: LRU Cache

**Least Recently Used (LRU) Cache** — the most classic DLL application.

```
  Structure: DLL + HashMap
  
  Most recently used ◄──────────────► Least recently used
  
  HEAD                                              TAIL
   │                                                 │
   ▼                                                 ▼
 [Page3] ⇄ [Page1] ⇄ [Page7] ⇄ [Page2] ⇄ [Page5]
   ↑          ↑          ↑          ↑          ↑
   └──────────┴──────────┴──────────┴──────────┘
              HashMap: key → node pointer

  Access Page7:
    1. Remove [Page7] from current position  → O(1) using node pointer
    2. Insert [Page7] at HEAD                → O(1)
    
  Cache full + new page:
    1. Remove TAIL (least recently used)     → O(1)
    2. Insert new page at HEAD               → O(1)
```

Both operations are O(1) — only possible with DLL!

---

## 3.6 Real-World DLL Application: Text Editor

```
  Cursor in a text editor can be modeled as a DLL:
  
  Each node = one character
  Cursor = pointer to current position
  
  H ⇄ e ⇄ l ⇄ l ⇄ o ⇄ _ ⇄ W ⇄ o ⇄ r ⇄ l ⇄ d
                    ↑
                  cursor

  ← Arrow: cursor = cursor.prev     (cursor at 'l')
  → Arrow: cursor = cursor.next     (cursor at '_')
  Delete:  remove cursor.next       O(1) in DLL!
  Insert:  add node after cursor    O(1) in DLL!
  
  With SLL, ← Arrow is O(n)!
```

---

## 3.7 Design Trade-off Analysis

### When Code Complexity vs Performance Matters

```
                    High ┌───────────────────────┐
                         │                       │
     Code Complexity     │   DLL                 │
                         │                       │
                    Low  │   SLL         Array    │
                         └───────────────────────┘
                         Low                 High
                              Performance (cache)
```

### Decision Matrix

| Factor | Weight | SLL Score | DLL Score | Winner |
|--------|--------|-----------|-----------|--------|
| Memory efficiency | High | 8/10 | 5/10 | SLL |
| Code simplicity | Medium | 8/10 | 5/10 | SLL |
| Insert/Delete at front | High | 9/10 | 9/10 | Tie |
| Delete from end | Medium | 3/10 | 9/10 | DLL |
| Delete given node | High | 3/10 | 10/10 | DLL |
| Backward traversal | Varies | 2/10 | 10/10 | DLL |
| Cache performance | High | 4/10 | 3/10 | SLL |

---

## 3.8 Common DLL Bugs and Prevention

### Bug 1: Forgetting to Update `prev`

```
❌ BAD (Insert after A):
    newNode.next = A.next
    A.next = newNode
    // Forgot: newNode.prev = A  and  newNode.next.prev = newNode!

✓ FIXED:
    newNode.next = A.next
    newNode.prev = A
    if A.next ≠ NULL:
        A.next.prev = newNode
    A.next = newNode
```

### Bug 2: Not Handling TAIL During Deletion

```
❌ BAD:
    node.prev.next = node.next
    node.next.prev = node.prev     ← CRASH if node is TAIL! (node.next is NULL)

✓ FIXED:
    if node.prev ≠ NULL: node.prev.next = node.next
    if node.next ≠ NULL: node.next.prev = node.prev
    if node == TAIL: TAIL = node.prev
    if node == HEAD: HEAD = node.next
```

### Bug 3: Dangling Pointers After Deletion

```
❌ BAD:
    free(node)
    print(node.data)               ← UNDEFINED BEHAVIOR! node is freed!

✓ FIXED:
    temp = node.data               ← Save data if needed
    free(node)
    node = NULL                    ← Set to NULL to avoid accidental use
```

---

## 3.9 Complete Complexity Summary: SLL vs DLL

| Operation | SLL | DLL | Notes |
|-----------|-----|-----|-------|
| Insert at beginning | O(1) | O(1) | Same |
| Insert at end | O(1)* | O(1)* | *With TAIL pointer |
| Insert before node | O(n) | O(1) | DLL has prev pointer |
| Insert after node | O(1) | O(1) | Same |
| Delete from beginning | O(1) | O(1) | Same |
| Delete from end | O(n) | O(1)* | *With TAIL pointer |
| Delete given node | O(n) | O(1) | DLL has prev pointer |
| Forward traversal | O(n) | O(n) | Same |
| Backward traversal | O(n)** | O(n) | **SLL needs reversal or stack |
| Search | O(n) | O(n) | DLL can search from both ends |
| Space per node | sizeof(data)+ptr | sizeof(data)+2×ptr | DLL uses more memory |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Forward traversal | Same as SLL — follow `next` from HEAD |
| Backward traversal | Follow `prev` from TAIL — O(n) time, O(1) space |
| Bidirectional search | Search from both ends — average n/2 comparisons |
| Browser history | Back/Forward = prev/next pointers on current page |
| LRU Cache | DLL + HashMap — O(1) access, eviction, and update |
| Text editor | Cursor movement, insert, delete — all O(1) with DLL |
| Key trade-off | DLL wins on flexibility, SLL wins on simplicity and memory |
| Bug prevention | Always update both `prev` and `next`, check for NULL |

---

## Quick Revision Questions

1. **What is the time and space complexity of backward traversal in a DLL?**
   <details><summary>Answer</summary>Time: O(n), Space: O(1). Start from TAIL, follow `prev` pointers until NULL.</details>

2. **How does a DLL implement browser back/forward navigation?**
   <details><summary>Answer</summary>Each page is a DLL node. `CURRENT.prev` is the back button, `CURRENT.next` is the forward button. When visiting a new page from a midpoint, delete all nodes after CURRENT and insert the new page.</details>

3. **Why is DLL essential for an O(1) LRU Cache?**
   <details><summary>Answer</summary>LRU Cache needs: 1) O(1) removal from any position (DLL with node pointer), 2) O(1) insertion at front (DLL insert at head), 3) O(1) eviction from back (DLL delete from tail). SLL can't do #1 in O(1).</details>

4. **What is the extra memory cost of DLL vs SLL for 10,000 integer nodes on a 64-bit system?**
   <details><summary>Answer</summary>Extra cost = 10,000 × 8 bytes (one `prev` pointer per node) = 80,000 bytes ≈ 78 KB extra.</details>

5. **Name the most common DLL bug and how to prevent it.**
   <details><summary>Answer</summary>Forgetting to update `prev` pointers during insertion or deletion, breaking the bidirectional invariant. Prevention: always check that if `A.next = B` then `B.prev = A` after every operation.</details>

6. **Can DLL achieve better than O(n) search time? Why or why not?**
   <details><summary>Answer</summary>No — even with bidirectional search (from both HEAD and TAIL), the worst case is still O(n/2) = O(n). Each element must be checked sequentially from one end or the other. Binary search isn't possible because you can't jump to the middle in O(1).</details>

---

[← Previous: DLL Insertion and Deletion](02-dll-insertion-and-deletion.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 4 — Circular List Types →](../04-Circular-Linked-List/01-circular-list-types.md)

# Chapter 3: Edge Cases and Complexity Analysis

[← Previous: Deletion and Search](02-deletion-and-search.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 3 — DLL Structure →](../03-Doubly-Linked-List/01-dll-structure-and-advantages.md)

---

## Chapter Overview

This chapter consolidates your understanding of edge cases, common bugs, and a thorough complexity analysis for all singly linked list operations. Mastering edge cases separates good code from bug-free code.

---

## 3.1 The Critical Edge Cases

Every linked list operation must handle these scenarios:

### Edge Case 1: Empty List (HEAD = NULL)

```
  HEAD ──► NULL

  Questions to ask:
  • What should insert do? → Create the first node
  • What should delete do? → Return error / do nothing
  • What should search do? → Return "not found"
  • What should traverse do? → Nothing (loop never executes)
```

### Edge Case 2: Single Node List

```
  HEAD
   │
   ▼
 [10] ──► NULL

  Questions to ask:
  • Delete from beginning? → HEAD becomes NULL
  • Delete from end? → Same as above (only one node!)
  • Insert at position 0? → New node becomes HEAD
  • Insert at position 1? → New node becomes the tail
```

### Edge Case 3: Two Node List

```
  HEAD
   │
   ▼
 [10] ──► [20] ──► NULL

  Why this matters:
  • Delete from end → HEAD is both first AND second-to-last
  • Some traversal patterns (like current.next.next) fail here
```

### Edge Case 4: Operation on Last Node

```
  HEAD
   │
   ▼
 [10] ──► [20] ──► [30] ──► NULL
                             ↑
                        Last node — no "next" to use

  • Delete without HEAD access → IMPOSSIBLE for last node
  • Insert after last → Must set next from NULL to new node
```

### Edge Case 5: Out of Bounds Position

```
  List length = 3 (indices 0, 1, 2)
  
  ❌ deleteAtPosition(5) → out of bounds
  ❌ insertAtPosition(-1) → invalid
  ❌ insertAtPosition(10) → past the end
```

---

## 3.2 Edge Case Testing Matrix

| Operation | Empty List | Single Node | Target at HEAD | Target at End | Target Not Found | Out of Bounds |
|-----------|-----------|-------------|----------------|---------------|-----------------|---------------|
| Insert at beginning | ✓ Create first | ✓ Prepend | — | — | — | — |
| Insert at end | ✓ Create first | ✓ Append | — | — | — | — |
| Insert at pos k | ✓ k=0 only | ✓ k=0 or k=1 | — | — | — | ✓ Handle |
| Delete beginning | ✗ Error/empty | ✓ HEAD=NULL | — | — | — | — |
| Delete end | ✗ Error/empty | ✓ HEAD=NULL | — | — | — | — |
| Delete at pos k | ✗ Error/empty | ✓ k=0 only | — | — | — | ✓ Handle |
| Delete by value | ✓ Return | ✓ May empty | ✓ Change HEAD | ✓ Standard | ✓ Not found | — |
| Search | ✓ Return -1 | ✓ Check 1 | ✓ Return 0 | ✓ Return n-1 | ✓ Return -1 | — |

---

## 3.3 Common Bugs and How to Avoid Them

### Bug 1: Not Handling Empty List

```
❌ BAD:
function insertAtEnd(HEAD, value):
    current = HEAD
    while current.next ≠ NULL:     ← CRASH! current is NULL!
        current = current.next

✓ FIXED:
function insertAtEnd(HEAD, value):
    newNode = createNode(value)
    if HEAD == NULL:               ← Check empty list first!
        return newNode
    current = HEAD
    while current.next ≠ NULL:
        current = current.next
    current.next = newNode
    return HEAD
```

### Bug 2: Losing the List (Modifying HEAD)

```
❌ BAD:
function printList(HEAD):
    while HEAD ≠ NULL:
        print(HEAD.data)
        HEAD = HEAD.next           ← HEAD is now NULL! List is lost!

✓ FIXED:
function printList(HEAD):
    current = HEAD                 ← Use a separate variable
    while current ≠ NULL:
        print(current.data)
        current = current.next     ← HEAD unchanged
```

### Bug 3: Memory Leak (Not Freeing Deleted Nodes)

```
❌ BAD (in C/C++):
function deleteFirst(HEAD):
    HEAD = HEAD.next               ← Old first node leaked!

✓ FIXED:
function deleteFirst(HEAD):
    temp = HEAD
    HEAD = HEAD.next
    free(temp)                     ← Properly deallocated
```

### Bug 4: NullPointerException on Last Node

```
❌ BAD:
while current.next.next ≠ NULL:   ← CRASH when current.next is NULL!
    current = current.next

✓ FIXED:
while current.next ≠ NULL and current.next.next ≠ NULL:
    current = current.next
    
// OR better: use a 'prev' pointer pattern
```

### Bug 5: Wrong Pointer Order During Insertion

```
❌ BAD:
current.next = newNode             ← Link to rest of list is LOST!
newNode.next = current.next        ← Points to itself!

✓ FIXED:
newNode.next = current.next        ← Save link first
current.next = newNode             ← Then redirect
```

### Bug 6: Infinite Loop

```
❌ BAD:
while current ≠ NULL:
    if current.data == target:
        print("Found!")
    // Forgot to advance!          ← INFINITE LOOP!

✓ FIXED:
while current ≠ NULL:
    if current.data == target:
        print("Found!")
        break
    current = current.next         ← Always advance
```

---

## 3.4 Defensive Coding Checklist

Before submitting any linked list solution, verify:

```
✅ Empty list (HEAD = NULL) handled?
✅ Single node list handled?
✅ Operation on HEAD handled? (Special case for first node)
✅ Operation on last node handled?
✅ Pointers set in correct order? (Link before break)
✅ Memory freed for deleted nodes? (In C/C++)
✅ HEAD returned/updated correctly?
✅ No infinite loops? (Every path advances or terminates)
✅ Out-of-bounds positions handled?
✅ Used separate variable instead of modifying HEAD?
```

---

## 3.5 Complete Time Complexity Reference

### Singly Linked List (Without TAIL)

| Operation | Best Case | Average Case | Worst Case | Why |
|-----------|-----------|-------------|------------|-----|
| Insert at beginning | O(1) | O(1) | O(1) | Direct pointer change |
| Insert at end | O(n) | O(n) | O(n) | Must traverse to last |
| Insert at position k | O(1) | O(k) | O(n) | Traverse to position k |
| Delete from beginning | O(1) | O(1) | O(1) | Direct pointer change |
| Delete from end | O(n) | O(n) | O(n) | Must find second-to-last |
| Delete at position k | O(1) | O(k) | O(n) | Traverse to position k-1 |
| Delete by value | O(1) | O(n) | O(n) | Linear search |
| Search by value | O(1) | O(n) | O(n) | Linear scan |
| Search by index | O(1) | O(k) | O(n) | Traverse k nodes |
| Traversal | O(n) | O(n) | O(n) | Visit all nodes |
| Length | O(n) | O(n) | O(n) | Count all nodes |
| Reverse | O(n) | O(n) | O(n) | Visit all nodes |

### Singly Linked List (With TAIL)

| Operation | Time | Improvement |
|-----------|------|-------------|
| Insert at end | O(1) | From O(n) — direct access via TAIL |
| Delete from end | O(n) | No improvement — still need prev node |
| Access last element | O(1) | From O(n) — direct access via TAIL |

### With Both HEAD and TAIL + Length Counter

| Operation | Time | How |
|-----------|------|-----|
| Get length | O(1) | Maintained as a counter, not computed |
| Insert at end | O(1) | Via TAIL pointer |
| Check empty | O(1) | Check if length == 0 or HEAD == NULL |

---

## 3.6 Space Complexity Analysis

### Per Operation

| Operation | Extra Space | What's Allocated |
|-----------|-------------|-----------------|
| Insert (any) | O(1) | One new node + temp pointers |
| Delete (any) | O(1) | Only temp pointers |
| Search | O(1) | Iterator pointer |
| Traverse | O(1) | Iterator pointer |
| Reverse (iterative) | O(1) | Three pointers |
| Reverse (recursive) | O(n) | Call stack |
| Copy list | O(n) | New list |

### Data Structure Overhead

```
Total memory for n nodes:

  Singly Linked List:
    n × (sizeof(data) + sizeof(pointer)) + sizeof(HEAD)
    = n × (4 + 8) + 8     (on 64-bit, int data)
    = 12n + 8 bytes

  With TAIL:
    12n + 16 bytes          (8 extra for TAIL pointer)

  With length counter:
    12n + 20 bytes          (4 extra for int counter)

  Compare to Array:
    n × sizeof(data) + overhead
    = 4n + ≈24 bytes        (array metadata)
```

---

## 3.7 Problem-Solving Strategy for Linked Lists

### Step-by-Step Framework

```
1. UNDERSTAND the problem
   └─ Draw the before and after diagrams
   └─ Identify which pointers need to change

2. IDENTIFY edge cases
   └─ Empty list? Single node? HEAD involved? TAIL involved?

3. CHOOSE the technique
   └─ Two pointers? Dummy node? Previous pointer? Recursion?

4. WRITE pseudocode
   └─ Handle edge cases first
   └─ Then the general case

5. TRACE through examples
   └─ Trace with empty, single-node, and normal lists
   └─ Verify pointer changes step by step

6. CHECK for common bugs
   └─ Pointer order correct?
   └─ Memory freed?
   └─ HEAD updated?
   └─ No infinite loops?
```

### Common Techniques Reference

| Technique | When to Use | Example |
|-----------|-------------|---------|
| **Two pointers (slow/fast)** | Cycle detection, middle node | Floyd's algorithm |
| **Previous pointer** | Deletion, insertion before a node | Delete by value |
| **Dummy/sentinel node** | Simplify HEAD-change operations | Merge sorted lists |
| **Runner technique** | Compare elements at different speeds | Find nth from end |
| **Recursion** | Reversal, print reverse, deep operations | Reverse linked list |
| **Hash set** | Detect duplicates, find intersection | Remove duplicates |

---

## 3.8 Practice Problem Types

### Beginner

| # | Problem | Key Operation | Difficulty |
|---|---------|--------------|------------|
| 1 | Insert at beginning | Prepend | Easy |
| 2 | Insert at end | Append + traversal | Easy |
| 3 | Delete first node | HEAD update | Easy |
| 4 | Print all elements | Traversal | Easy |
| 5 | Count nodes | Traversal + counter | Easy |
| 6 | Search for value | Linear search | Easy |

### Intermediate

| # | Problem | Key Technique | Difficulty |
|---|---------|--------------|------------|
| 7 | Insert at kth position | Position traversal | Medium |
| 8 | Delete by value | Previous pointer | Medium |
| 9 | Delete kth node from end | Two pointers | Medium |
| 10 | Find middle element | Slow/fast pointers | Medium |
| 11 | Remove duplicates (sorted) | Traverse + compare | Medium |
| 12 | Detect cycle | Floyd's algorithm | Medium |

### Advanced

| # | Problem | Key Technique | Difficulty |
|---|---------|--------------|------------|
| 13 | Reverse linked list | Three-pointer swap | Medium-Hard |
| 14 | Merge sorted lists | Dummy node + compare | Medium-Hard |
| 15 | Check palindrome | Reverse half + compare | Hard |
| 16 | Flatten multilevel list | Recursion + merge | Hard |
| 17 | Clone with random ptrs | Hash map + two pass | Hard |
| 18 | LRU Cache | DLL + Hash map | Hard |

---

## Summary Table

| Topic | Key Takeaway |
|-------|-------------|
| Empty list | Always check `HEAD == NULL` first |
| Single node | Test every operation with a 1-element list |
| HEAD changes | Return updated HEAD; use dummy nodes if complex |
| Pointer order | Link new connections BEFORE breaking old ones |
| Memory | Free deleted nodes in manual memory languages |
| SLL insert/delete | O(1) at beginning, O(n) at end or by value |
| Space overhead | ~3x memory per element vs arrays |
| Cache performance | Arrays win due to spatial locality |
| Problem solving | Draw diagrams → Handle edges → Write pseudocode → Trace |

---

## Quick Revision Questions

1. **List 5 edge cases every linked list operation should handle.**
   <details><summary>Answer</summary>1) Empty list (HEAD = NULL), 2) Single node list, 3) Operation on HEAD node, 4) Operation on last node, 5) Out-of-bounds position or value not found.</details>

2. **Why is `newNode.next = current.next` set BEFORE `current.next = newNode`?**
   <details><summary>Answer</summary>Setting the order correctly preserves the link to the rest of the list. If you set `current.next = newNode` first, you lose the reference to the remaining nodes.</details>

3. **What is the total memory usage of a singly linked list with 100 integer nodes on a 64-bit system?**
   <details><summary>Answer</summary>Each node: 4 bytes (int) + 8 bytes (pointer) = 12 bytes. Plus malloc overhead (~8-16 bytes per allocation). Total ≈ 100 × (12 + 16) = 2,800 bytes, plus 8 bytes for HEAD. Compare to array: 100 × 4 = 400 bytes.</details>

4. **Why does maintaining a TAIL pointer not help with deletion from the end?**
   <details><summary>Answer</summary>To delete the last node, you need to set the second-to-last node's `next` to NULL. TAIL points to the last node, but in a SLL you can't go backward to find the second-to-last node. You still need O(n) traversal from HEAD.</details>

5. **What is the "dummy node" technique and when should you use it?**
   <details><summary>Answer</summary>A dummy (sentinel) node is a temporary node placed before HEAD. It eliminates special-case code for operations that might change HEAD (like deletion of the first node or merging). Use it when the operation might modify HEAD in various cases.</details>

6. **What's the first thing you should do when solving a linked list problem?**
   <details><summary>Answer</summary>Draw the "before" and "after" diagrams showing the list state, identify which pointers need to change, and list all edge cases before writing any code.</details>

---

[← Previous: Deletion and Search](02-deletion-and-search.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 3 — DLL Structure →](../03-Doubly-Linked-List/01-dll-structure-and-advantages.md)

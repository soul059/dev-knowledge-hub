# Chapter 1: Flatten a Multilevel List and Clone with Random Pointers

[← Previous: Intersection and Union](../07-Merge-Operations/03-intersection-and-union.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Arithmetic and Reordering →](02-arithmetic-and-reordering.md)

---

## Chapter Overview

Two advanced problems that test deep understanding of pointer manipulation: **flattening** a multilevel linked list into a single level, and **deep cloning** a linked list that has random (arbitrary) pointers.

---

## 1.1 Flatten a Multilevel / Multi-Level Linked List

### Problem Variant 1: Sorted Sub-Lists with Down Pointers

Given a linked list where each node has `next` (right) and `down` (child) pointers. Each `down` list is sorted. Flatten everything into a single sorted list using the `down` pointer.

```
  5 → 10 → 19 → 28
  |    |    |    |
  7   20   22   35
  |         |    |
  8        50   40
  |              |
  30            45

  Flattened (using down pointer):
  5→7→8→10→19→20→22→28→30→35→40→45→50
```

### Algorithm: Merge from Right to Left

```
function flatten(HEAD):
    if HEAD == NULL or HEAD.next == NULL:
        return HEAD
    
    // Recursively flatten from the right
    HEAD.next = flatten(HEAD.next)
    
    // Merge current list (down) with the flattened rest
    HEAD = mergeSortedDown(HEAD, HEAD.next)
    
    return HEAD

function mergeSortedDown(A, B):
    if A == NULL: return B
    if B == NULL: return A
    
    if A.data ≤ B.data:
        result = A
        result.down = mergeSortedDown(A.down, B)
    else:
        result = B
        result.down = mergeSortedDown(A, B.down)
    
    result.next = NULL
    return result
```

### Visual Trace (Simplified)

```
  Step 1: Flatten from right
  
  Start: 5→10→19→28
  
  flatten(28): base case, return 28→35→40→45 (down list)
  
  flatten(19): merge 19→22→50 with 28→35→40→45
  Result: 19→22→28→35→40→45→50
  
  flatten(10): merge 10→20 with 19→22→28→35→40→45→50
  Result: 10→19→20→22→28→35→40→45→50
  
  flatten(5): merge 5→7→8→30 with 10→19→20→22→28→35→40→45→50
  Result: 5→7→8→10→19→20→22→28→30→35→40→45→50 ✓
```

---

### Problem Variant 2: Multilevel Doubly Linked List (LeetCode 430)

Each node has `next`, `prev`, and `child`. Flatten the child levels inline.

```
  1 ⇄ 2 ⇄ 3 ⇄ 4 ⇄ 5 ⇄ 6
           |
           7 ⇄ 8 ⇄ 9 ⇄ 10
               |
               11 ⇄ 12
  
  Flattened:
  1 ⇄ 2 ⇄ 3 ⇄ 7 ⇄ 8 ⇄ 11 ⇄ 12 ⇄ 9 ⇄ 10 ⇄ 4 ⇄ 5 ⇄ 6
```

### Algorithm

```
function flattenDLL(HEAD):
    curr = HEAD
    
    while curr ≠ NULL:
        if curr.child ≠ NULL:
            // Find tail of child list
            childTail = curr.child
            while childTail.next ≠ NULL:
                childTail = childTail.next
            
            // Connect child tail to curr.next
            childTail.next = curr.next
            if curr.next ≠ NULL:
                curr.next.prev = childTail
            
            // Connect curr to child
            curr.next = curr.child
            curr.child.prev = curr
            curr.child = NULL
        
        curr = curr.next
    
    return HEAD
```

### Trace

```
  At Node(3): child = 7
    Find childTail of 7→8→9→10 → childTail = 10
    
    But wait — Node(8) has child = 11→12!
    We'll handle it when curr reaches Node(8).
    
    Step at Node(3):
    10.next = 4 (was 3.next)
    4.prev = 10
    3.next = 7
    7.prev = 3
    3.child = NULL
    
    State: 1⇄2⇄3⇄7⇄8⇄9⇄10⇄4⇄5⇄6   (8 still has child)
    
  At Node(8): child = 11
    childTail of 11→12 = 12
    12.next = 9 (was 8.next)
    9.prev = 12
    8.next = 11
    11.prev = 8
    8.child = NULL
    
    Final: 1⇄2⇄3⇄7⇄8⇄11⇄12⇄9⇄10⇄4⇄5⇄6 ✓
```

---

## 1.2 Clone a Linked List with Random Pointers

### The Problem

Given a linked list where each node has:
- `next`: points to the next node
- `random`: points to any arbitrary node in the list (or NULL)

Create a **deep copy** — new nodes with the same structure.

```
  Original:
  [1] → [2] → [3] → [4] → NULL
   |     |     |     |
   ↓     ↓     ↓     ↓
  [3]   [1]  [NULL] [2]     (random pointers)
  
  Clone: Brand new nodes with same next/random relationships
```

### Why Is This Hard?

```
  When copying node 1, its random points to node 3.
  But cloned node 3 might not exist yet!
  
  We need a way to map: original node → cloned node
```

---

### Approach 1: HashMap — O(n) Time, O(n) Space

```
function cloneWithHash(HEAD):
    if HEAD == NULL: return NULL
    
    map = new HashMap()             ← original → clone
    
    // Pass 1: Create all clone nodes
    curr = HEAD
    while curr ≠ NULL:
        map[curr] = createNode(curr.data)
        curr = curr.next
    
    // Pass 2: Set next and random pointers
    curr = HEAD
    while curr ≠ NULL:
        map[curr].next = map[curr.next]
        map[curr].random = map[curr.random]
        curr = curr.next
    
    return map[HEAD]
```

### Trace

```
  Original: A(1) → B(2) → C(3)
  Randoms:  A→C,  B→A,  C→B
  
  Pass 1: Create clones
  map = {A→A', B→B', C→C'}
  
  Pass 2: Link clones
  A'.next = map[B] = B'     A'.random = map[C] = C'
  B'.next = map[C] = C'     B'.random = map[A] = A'
  C'.next = map[NULL] = NULL C'.random = map[B] = B'
  
  Clone: A'(1)→B'(2)→C'(3)  with A'→C', B'→A', C'→B' ✓
```

---

### Approach 2: Interleaving — O(n) Time, O(1) Space ⭐

The clever approach: weave clone nodes between original nodes.

```
function cloneInterleave(HEAD):
    if HEAD == NULL: return NULL
    
    // Step 1: Create clone nodes interleaved
    curr = HEAD
    while curr ≠ NULL:
        clone = createNode(curr.data)
        clone.next = curr.next
        curr.next = clone
        curr = clone.next
    
    // Step 2: Set random pointers for clones
    curr = HEAD
    while curr ≠ NULL:
        if curr.random ≠ NULL:
            curr.next.random = curr.random.next
        curr = curr.next.next
    
    // Step 3: Separate original and clone lists
    curr = HEAD
    cloneHead = HEAD.next
    while curr ≠ NULL:
        clone = curr.next
        curr.next = clone.next
        if clone.next ≠ NULL:
            clone.next = clone.next.next
        curr = curr.next
    
    return cloneHead
```

### Visual Trace

```
  Original: [1] → [2] → [3] → NULL
  Randoms:   1→3,  2→1,  3→NULL

  ═══ Step 1: Interleave ═══
  
  [1] → [1'] → [2] → [2'] → [3] → [3'] → NULL
  
  Each original node is followed by its clone.


  ═══ Step 2: Set random pointers ═══
  
  For Node(1): random = Node(3)
    1'.random = 1.random.next = 3.next = 3'  ← Clone of 3!
  
  For Node(2): random = Node(1)
    2'.random = 2.random.next = 1.next = 1'  ← Clone of 1!
  
  For Node(3): random = NULL
    3'.random = NULL
  
  The KEY INSIGHT: original.random.next is ALWAYS the clone 
  of original.random, because we interleaved them!


  ═══ Step 3: Separate ═══
  
  Original: [1] → [2] → [3] → NULL       (restored)
  Clone:    [1'] → [2'] → [3'] → NULL     (extracted)
  
  With randoms: 1'→3', 2'→1', 3'→NULL ✓
```

### Step 1 Detail

```
  Before interleaving:
  
  [1] ────────► [2] ────────► [3] ────────► NULL
  
  Processing Node(1):
  clone = Node(1')
  clone.next = 1.next = [2]      → [1'] → [2]
  1.next = clone                  → [1] → [1']
  curr = clone.next = [2]
  
  [1] → [1'] → [2] → [3] → NULL
  
  Processing Node(2):
  clone = Node(2')
  clone.next = 2.next = [3]
  2.next = clone
  curr = clone.next = [3]
  
  [1] → [1'] → [2] → [2'] → [3] → NULL
  
  Processing Node(3):
  clone = Node(3')
  clone.next = 3.next = NULL
  3.next = clone
  curr = clone.next = NULL → EXIT
  
  [1] → [1'] → [2] → [2'] → [3] → [3'] → NULL ✓
```

---

## 1.3 Complexity Comparison

| Approach | Time | Space | Modifies original? |
|----------|------|-------|-------------------|
| HashMap | O(n) | O(n) | No |
| Interleaving | O(n) | O(1)* | Temporarily yes |

*O(1) extra space besides the cloned nodes themselves.

---

## 1.4 Common Mistakes

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Shallow copy (copy pointers, not nodes) | Clone shares nodes with original | Create NEW nodes |
| Forgetting to restore original list | Original list is corrupted | Step 3 must fully separate |
| NULL check on random pointer | NullPointerException | Check `curr.random ≠ NULL` before accessing |
| Not handling NULL in hash map | Crash on `map[NULL]` | Return NULL for NULL keys |

---

## Summary Table

| Problem | Best Approach | Time | Space |
|---------|--------------|------|-------|
| Flatten sorted sub-lists | Merge from right | O(n) | O(1)* |
| Flatten multilevel DLL | Iterative with child-tail find | O(n) | O(1) |
| Clone with random (simple) | HashMap | O(n) | O(n) |
| Clone with random (optimal) | Interleaving | O(n) | O(1) |

*Plus recursion stack for merge approach.

---

## Quick Revision Questions

1. **Why do we flatten from right to left in the sorted sub-list problem?**
   <details><summary>Answer</summary>Flattening right to left means we always merge the current column with an already-flattened (sorted) list. This ensures each merge operation produces a sorted result, which is required for the next merge.</details>

2. **In the interleaving clone approach, how does `curr.random.next` give us the clone?**
   <details><summary>Answer</summary>After Step 1, every original node is immediately followed by its clone. So if `curr.random` is some original node X, then `X.next` is X's clone (X'). Therefore `curr.random.next` = clone of the random target.</details>

3. **Why is Step 3 (separation) necessary in the interleaving approach?**
   <details><summary>Answer</summary>After setting random pointers, the original and clone nodes are still woven together. We must separate them to restore the original list and extract the independent clone list. Without separation, both lists are corrupted.</details>

4. **In the multilevel DLL flatten, why do we find the child's tail?**
   <details><summary>Answer</summary>We need to connect the end of the child list to `curr.next` (the node that was after the parent). The child's tail is the last node in the child chain, whose `next` should bridge to the rest of the parent-level list.</details>

5. **Can you clone a list with random pointers using O(1) space and without modifying the original?**
   <details><summary>Answer</summary>Not easily. The interleaving approach temporarily modifies the original. Without modifying the original, you need a mapping (HashMap) which uses O(n) space. There is no known O(1) space, non-modifying approach.</details>

6. **What if two random pointers point to the same node? Does the clone handle this?**
   <details><summary>Answer</summary>Yes. Both approaches handle this correctly. The HashMap maps each original to one clone, and interleaving naturally handles it since `original.random.next` consistently yields the same clone regardless of how many nodes point there.</details>

---

[← Previous: Intersection and Union](../07-Merge-Operations/03-intersection-and-union.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Arithmetic and Reordering →](02-arithmetic-and-reordering.md)

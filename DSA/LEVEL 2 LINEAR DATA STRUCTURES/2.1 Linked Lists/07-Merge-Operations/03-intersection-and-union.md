# Chapter 3: Intersection and Union of Linked Lists

[← Previous: Merge K Lists and Merge Sort](02-merge-k-and-merge-sort.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 8 — Flatten and Clone →](../08-Advanced-Problems/01-flatten-and-clone.md)

---

## Chapter Overview

Finding **intersection points**, computing **set intersection / union**, and detecting **shared tails** between linked lists. These problems test pointer analysis, hashing, and length-difference techniques.

---

## 3.1 Intersection Point of Two Lists (Y-shaped)

### The Problem

Two singly linked lists converge at a node and share the remainder (Y-shape). Find that intersection node.

```
  List A:  1 → 2 ↘
                   6 → 7 → 8 → NULL    (shared tail)
  List B:  3 → 4 → 5 ↗
  
  Intersection = Node(6)
```

**Key insight:** Once two lists merge, they never diverge again (singly linked — each node has only ONE next pointer).

### Approach 1: Length Difference Method — O(n+m) Time, O(1) Space ⭐

```
function findIntersection(A, B):
    // Step 1: Find lengths
    lenA = length(A)
    lenB = length(B)
    
    // Step 2: Advance the longer list
    diff = |lenA - lenB|
    longer = (lenA ≥ lenB) ? A : B
    shorter = (lenA ≥ lenB) ? B : A
    
    for i = 1 to diff:
        longer = longer.next
    
    // Step 3: Move both until they meet
    while longer ≠ shorter:
        longer = longer.next
        shorter = shorter.next
    
    return longer                    ← Intersection node (or NULL)
```

### Visual Trace

```
  A: 1 → 2 → 6 → 7 → 8        lenA = 5
  B: 3 → 4 → 5 → 6 → 7 → 8   lenB = 6
  
  diff = 6 - 5 = 1
  Advance B by 1: B starts at Node(4)
  
  Now both march together:
  
  | Step | A    | B    | Same? |
  |------|------|------|-------|
  | 0    | 1    | 4    | No    |
  | 1    | 2    | 5    | No    |
  | 2    | 6    | 6    | YES!  |
  
  Intersection = Node(6) ✓
  
  Why it works:
  After aligning, both pointers have the SAME distance 
  to the intersection point (and to the end).
```

### Approach 2: Two-Pointer Cycle — O(n+m) Time, O(1) Space ⭐

An elegant approach without computing lengths:

```
function findIntersectionTwoPointer(A, B):
    if A == NULL or B == NULL: return NULL
    
    pA = A
    pB = B
    
    while pA ≠ pB:
        pA = (pA == NULL) ? B : pA.next
        pB = (pB == NULL) ? A : pB.next
    
    return pA                        ← Intersection or NULL
```

### Why Does This Work?

```
  List A has a unique prefix of length 'a' and shared tail of length 'c'.
  List B has a unique prefix of length 'b' and shared tail of length 'c'.
  
  Pointer pA traverses: a + c + b (A then switches to B)
  Pointer pB traverses: b + c + a (B then switches to A)
  
  Total for both: a + c + b = b + c + a ← EQUAL!
  
  They reach the intersection point at the same step.

  Example:
  A: [1] → [2] → [6] → [7]        a=2, c=2
  B: [3] → [4] → [5] → [6] → [7]  b=3, c=2
  
  pA: 1, 2, 6, 7, NULL→B, 3, 4, 5, [6] ← step 8
  pB: 3, 4, 5, 6, 7, NULL→A, 1, 2, [6] ← step 8
  
  Both arrive at Node(6) simultaneously! ✓
```

### What if No Intersection?

```
  A: [1] → [2] → NULL
  B: [3] → [4] → [5] → NULL
  
  pA: 1, 2, NULL→B, 3, 4, 5, NULL
  pB: 3, 4, 5, NULL→A, 1, 2, NULL
  
  Both reach NULL at the same time → return NULL ✓
```

### Approach 3: HashSet — O(n+m) Time, O(n) Space

```
function findIntersectionHash(A, B):
    visited = new HashSet()
    
    curr = A
    while curr ≠ NULL:
        visited.add(curr)            ← Store node references
        curr = curr.next
    
    curr = B
    while curr ≠ NULL:
        if visited.contains(curr):
            return curr              ← First shared node
        curr = curr.next
    
    return NULL
```

---

## 3.2 All Approaches Compared

| Approach | Time | Space | Needs lengths? |
|----------|------|-------|---------------|
| Length difference | O(n+m) | O(1) | Yes |
| Two-pointer switch | O(n+m) | O(1) | No |
| HashSet | O(n+m) | O(n) | No |
| Brute force (nested) | O(n×m) | O(1) | No |

---

## 3.3 Intersection of Sorted Lists (Set Intersection)

Different problem: create a **new list** containing elements common to both sorted lists.

```
  A: 1 → 2 → 3 → 4 → 6
  B: 2 → 4 → 6 → 8
  
  Intersection: 2 → 4 → 6
```

### Algorithm

```
function sortedIntersection(A, B):
    dummy = createNode(0)
    tail = dummy
    
    while A ≠ NULL and B ≠ NULL:
        if A.data == B.data:
            // Add to result
            tail.next = createNode(A.data)
            tail = tail.next
            A = A.next
            B = B.next
        else if A.data < B.data:
            A = A.next
        else:
            B = B.next
    
    return dummy.next
```

### Trace

```
  A: 1 → 3 → 4 → 6    B: 2 → 3 → 6 → 8

  | A | B | Action | Result |
  |---|---|--------|--------|
  | 1 | 2 | 1<2, advance A | - |
  | 3 | 2 | 3>2, advance B | - |
  | 3 | 3 | equal, add to result | 3 |
  | 4 | 6 | 4<6, advance A | 3 |
  | 6 | 6 | equal, add to result | 3→6 |
  | - | 8 | A=NULL, stop | 3→6 ✓ |
```

---

## 3.4 Union of Sorted Lists

Create a new list with **all unique elements** from both sorted lists.

```
  A: 1 → 2 → 4
  B: 2 → 3 → 4 → 5
  
  Union: 1 → 2 → 3 → 4 → 5
```

### Algorithm

```
function sortedUnion(A, B):
    dummy = createNode(0)
    tail = dummy
    
    while A ≠ NULL and B ≠ NULL:
        if A.data == B.data:
            tail.next = createNode(A.data)
            A = A.next
            B = B.next
        else if A.data < B.data:
            tail.next = createNode(A.data)
            A = A.next
        else:
            tail.next = createNode(B.data)
            B = B.next
        tail = tail.next
    
    // Append remaining
    while A ≠ NULL:
        tail.next = createNode(A.data)
        tail = tail.next
        A = A.next
    
    while B ≠ NULL:
        tail.next = createNode(B.data)
        tail = tail.next
        B = B.next
    
    return dummy.next
```

### Trace

```
  A: 1 → 3 → 5    B: 2 → 3 → 4

  | A | B | Action | Result |
  |---|---|--------|--------|
  | 1 | 2 | 1<2, add 1 | 1 |
  | 3 | 2 | 3>2, add 2 | 1→2 |
  | 3 | 3 | equal, add 3 | 1→2→3 |
  | 5 | 4 | 5>4, add 4 | 1→2→3→4 |
  | 5 | - | B=NULL, append rest | 1→2→3→4→5 ✓ |
```

---

## 3.5 Unsorted Lists — Intersection and Union

For unsorted lists, use **hashing**:

```
function unsortedIntersection(A, B):
    setA = new HashSet()
    curr = A
    while curr ≠ NULL:
        setA.add(curr.data)
        curr = curr.next
    
    result = NULL
    curr = B
    while curr ≠ NULL:
        if setA.contains(curr.data):
            result = insertAtBeginning(result, curr.data)
            setA.remove(curr.data)   ← Avoid duplicates
        curr = curr.next
    
    return result
```

| Operation | Sorted Lists | Unsorted Lists |
|-----------|-------------|----------------|
| Intersection | O(n+m) time, O(min(n,m)) space | O(n+m) time, O(n) space |
| Union | O(n+m) time, O(n+m) space | O(n+m) time, O(n+m) space |

---

## 3.6 Related Problem: Remove Duplicates

### From Sorted List

```
function removeDupsSorted(HEAD):
    curr = HEAD
    while curr ≠ NULL and curr.next ≠ NULL:
        if curr.data == curr.next.data:
            curr.next = curr.next.next    ← Skip duplicate
        else:
            curr = curr.next
    return HEAD
```

```
  1 → 1 → 2 → 3 → 3 → NULL
  
  curr=1: 1==1, skip → 1→2→3→3
  curr=1: 1≠2, advance → curr=2
  curr=2: 2≠3, advance → curr=3
  curr=3: 3==3, skip → 3→NULL
  
  Result: 1 → 2 → 3 → NULL ✓
```

### From Unsorted List

```
function removeDupsUnsorted(HEAD):
    if HEAD == NULL: return NULL
    
    seen = new HashSet()
    curr = HEAD
    seen.add(curr.data)
    
    while curr.next ≠ NULL:
        if seen.contains(curr.next.data):
            curr.next = curr.next.next
        else:
            seen.add(curr.next.data)
            curr = curr.next
    
    return HEAD
```

| List Type | Time | Space |
|-----------|------|-------|
| Sorted | O(n) | O(1) |
| Unsorted (hash) | O(n) | O(n) |
| Unsorted (no hash) | O(n²) | O(1) |

---

## Summary Table

| Problem | Technique | Time | Space |
|---------|-----------|------|-------|
| Y-shaped intersection | Length diff / two-pointer switch | O(n+m) | O(1) |
| Y-shaped intersection | HashSet | O(n+m) | O(n) |
| Set intersection (sorted) | Two-pointer merge-style | O(n+m) | O(min) |
| Set union (sorted) | Two-pointer merge-style | O(n+m) | O(n+m) |
| Set ops (unsorted) | HashSet | O(n+m) | O(n) |
| Remove duplicates (sorted) | Single pointer | O(n) | O(1) |
| Remove duplicates (unsorted) | HashSet | O(n) | O(n) |

---

## Quick Revision Questions

1. **In the two-pointer intersection method, why do both pointers meet at the intersection?**
   <details><summary>Answer</summary>Pointer A travels distance a + c + b, and pointer B travels b + c + a (where a, b are unique prefix lengths, c is shared tail). Since a+c+b = b+c+a, they traverse the same total distance and arrive at the intersection simultaneously.</details>

2. **How can you tell if two linked lists intersect without finding the actual node?**
   <details><summary>Answer</summary>Traverse both lists to their ends. If the last nodes are the same object (same memory address), they intersect. Once lists merge in a Y-shape, they share all subsequent nodes including the tail.</details>

3. **What's the difference between "intersection point" and "set intersection"?**
   <details><summary>Answer</summary>"Intersection point" (Y-shaped) finds the node where two lists physically merge — same memory. "Set intersection" creates a new list of common values — different nodes. They're fundamentally different problems.</details>

4. **For sorted intersection, why don't we need a HashSet?**
   <details><summary>Answer</summary>Because both lists are sorted, we can use a two-pointer approach: advance the pointer with the smaller value. When values are equal, add to the result and advance both. This runs in O(n+m) time with O(1) extra space (plus output).</details>

5. **How do you remove duplicates from an unsorted list in O(1) space?**
   <details><summary>Answer</summary>For each node, scan all subsequent nodes and remove matches. This is O(n²) time but O(1) space. For each node `curr`, use a runner to check and remove all duplicates of `curr.data` from the rest of the list.</details>

6. **Can two lists that intersect have different lengths?**
   <details><summary>Answer</summary>Yes! The unique prefixes can be different lengths. But from the intersection point onward, both lists share the exact same nodes and thus have the same length for the shared tail portion.</details>

---

[← Previous: Merge K Lists and Merge Sort](02-merge-k-and-merge-sort.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 8 — Flatten and Clone →](../08-Advanced-Problems/01-flatten-and-clone.md)

# Chapter 2: Reverse in Groups (K-Group Reversal)

[← Previous: Iterative and Recursive Reversal](01-iterative-and-recursive-reversal.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Palindrome and DLL Reversal →](03-palindrome-and-dll-reversal.md)

---

## Chapter Overview

Reversing an entire list is the building block. This chapter tackles the harder problem: **reverse every K nodes** as a group. This is a classic interview question that tests pointer manipulation mastery.

---

## 2.1 Problem Statement

Given a linked list and integer K, reverse every group of K nodes.

```
  K=3:
  Before:  [1] → [2] → [3] → [4] → [5] → [6] → [7] → [8] → NULL
  
  After:   [3] → [2] → [1] → [6] → [5] → [4] → [7] → [8] → NULL
                group 1         group 2       remaining (< K)
```

**Variations:**
1. Leave the last group as-is if it has fewer than K nodes ← most common
2. Reverse the last group too (even if fewer than K)

---

## 2.2 Iterative K-Group Reversal

### Algorithm

```
function reverseKGroup(HEAD, k):
    if HEAD == NULL or k ≤ 1: return HEAD
    
    dummy = createNode(0)
    dummy.next = HEAD
    prevGroupEnd = dummy
    
    while true:
        // Step 1: Check if K nodes exist
        kthNode = getKthNode(prevGroupEnd, k)
        if kthNode == NULL: break    ← Less than K nodes left
        
        nextGroupStart = kthNode.next
        
        // Step 2: Reverse K nodes
        prev = nextGroupStart        ← Connect end to next group
        curr = prevGroupEnd.next
        for i = 1 to k:
            next = curr.next
            curr.next = prev
            prev = curr
            curr = next
        
        // Step 3: Fix connections
        temp = prevGroupEnd.next     ← This was the 1st node, now last
        prevGroupEnd.next = kthNode  ← Connect to new first (was Kth)
        prevGroupEnd = temp          ← Move to end of reversed group
    
    return dummy.next

function getKthNode(start, k):
    curr = start
    for i = 1 to k:
        if curr == NULL: return NULL
        curr = curr.next
    return curr
```

### Step-by-Step Trace: K=3, List = [1, 2, 3, 4, 5, 6, 7, 8]

```
  Initial: dummy → 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → NULL
  prevGroupEnd = dummy

  ═══ Iteration 1 (Group: 1,2,3) ═══
  
  kthNode = getKthNode(dummy, 3) = Node(3)
  nextGroupStart = Node(4)
  
  Reverse 3 nodes starting from Node(1):
    prev=4, curr=1: 1→4, prev=1, curr=2
    prev=1, curr=2: 2→1, prev=2, curr=3
    prev=2, curr=3: 3→2, prev=3, curr=4
  
  Fix connections:
    temp = dummy.next = Node(1) (the old first, now last in group)
    dummy.next = Node(3) (kthNode, now first in group)
    prevGroupEnd = Node(1)
  
  State: dummy → 3 → 2 → 1 → 4 → 5 → 6 → 7 → 8 → NULL
                              ↑
                         prevGroupEnd


  ═══ Iteration 2 (Group: 4,5,6) ═══
  
  kthNode = getKthNode(Node(1), 3) = Node(6)
  nextGroupStart = Node(7)
  
  Reverse 3 nodes starting from Node(4):
    prev=7, curr=4: 4→7, prev=4, curr=5
    prev=4, curr=5: 5→4, prev=5, curr=6
    prev=5, curr=6: 6→5, prev=6, curr=7
  
  Fix connections:
    temp = Node(1).next = Node(4) (old first, now last)  
    Node(1).next = Node(6) (kthNode)
    prevGroupEnd = Node(4)
  
  State: dummy → 3 → 2 → 1 → 6 → 5 → 4 → 7 → 8 → NULL
                                          ↑
                                     prevGroupEnd


  ═══ Iteration 3 ═══
  
  kthNode = getKthNode(Node(4), 3) = NULL (only 2 nodes left)
  Break! → Leave 7 → 8 as-is
  
  Final: 3 → 2 → 1 → 6 → 5 → 4 → 7 → 8 → NULL ✓
```

---

## 2.3 Recursive K-Group Reversal

### Algorithm

```
function reverseKGroupRecursive(HEAD, k):
    // Step 1: Check if K nodes exist
    curr = HEAD
    count = 0
    while curr ≠ NULL and count < k:
        curr = curr.next
        count++
    
    if count < k:
        return HEAD                  ← Less than K nodes, don't reverse
    
    // Step 2: Reverse first K nodes
    prev = NULL
    curr = HEAD
    for i = 1 to k:
        next = curr.next
        curr.next = prev
        prev = curr
        curr = next
    
    // Step 3: Recursively reverse the remaining list
    // HEAD is now the LAST node of the reversed group
    HEAD.next = reverseKGroupRecursive(curr, k)
    
    return prev                      ← New head of this group
```

### Visual Recursion

```
  reverseKGroup([1→2→3→4→5→6→7→8], 3)
  │
  │  Reverse first 3: [3→2→1]  remaining: [4→5→6→7→8]
  │  1.next = reverseKGroup([4→5→6→7→8], 3)
  │           │
  │           │  Reverse first 3: [6→5→4]  remaining: [7→8]
  │           │  4.next = reverseKGroup([7→8], 3)
  │           │           │
  │           │           │  count=2 < 3 → return [7→8] as-is
  │           │           │
  │           │  4.next = [7→8]
  │           │  return [6→5→4→7→8]
  │           │
  │  1.next = [6→5→4→7→8]
  │  return [3→2→1→6→5→4→7→8] ✓
```

---

## 2.4 Variation: Reverse Alternate K Groups

Reverse every **other** group of K nodes.

```
  K=2:
  Before:  [1] → [2] → [3] → [4] → [5] → [6] → [7] → [8]
  After:   [2] → [1] → [3] → [4] → [6] → [5] → [7] → [8]
            reversed    kept as-is   reversed    kept as-is
```

### Algorithm

```
function reverseAlternateKGroup(HEAD, k):
    curr = HEAD
    count = 0
    
    // Check if K nodes exist
    temp = HEAD
    for i = 1 to k:
        if temp == NULL: return HEAD
        temp = temp.next
    
    // Reverse K nodes
    prev = NULL
    curr = HEAD
    for i = 1 to k:
        next = curr.next
        curr.next = prev
        prev = curr
        curr = next
    
    // Skip K nodes (don't reverse)
    HEAD.next = curr
    for i = 1 to k-1:
        if curr == NULL: break
        curr = curr.next
    
    // Recurse for the rest
    if curr ≠ NULL:
        curr.next = reverseAlternateKGroup(curr.next, k)
    
    return prev
```

---

## 2.5 Variation: Reverse in K Groups (Also Reverse Short Tail)

If you want to reverse the last group even when it has fewer than K nodes:

```
function reverseKGroupAll(HEAD, k):
    // DON'T check count — always reverse
    prev = NULL
    curr = HEAD
    count = 0
    
    while curr ≠ NULL and count < k:
        next = curr.next
        curr.next = prev
        prev = curr
        curr = next
        count++
    
    if curr ≠ NULL:
        HEAD.next = reverseKGroupAll(curr, k)
    
    return prev
```

```
  K=3, List: [1,2,3,4,5]
  
  Version 1 (leave short tail):  3→2→1→4→5
  Version 2 (reverse short tail): 3→2→1→5→4
```

---

## 2.6 Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Iterative K-group | O(n) | O(1) | Each node visited ≤ 2 times |
| Recursive K-group | O(n) | O(n/k) | Stack depth = n/k groups |
| Check K nodes exist | O(n) total | — | Each node checked once |

### Why O(n) Not O(n×k)?

```
  Each node is visited at most twice:
  1. Once during the "check if K nodes exist" 
  2. Once during the "reverse" step
  
  Total work = 2n → O(n)
```

---

## 2.7 Connection Visualization

The hardest part of K-group reversal is getting the **group connections** right.

```
  Before reversal of group 2:
  
  [group 1 end] ──→ [group 2 start] ──→ ... ──→ [group 2 end] ──→ [group 3 start]
        ↑                  ↑                          ↑                   ↑
   prevGroupEnd        1st node                    Kth node          nextGroupStart
  
  
  After reversal of group 2:
  
  [group 1 end] ──→ [group 2 end] ──→ ... ──→ [group 2 start] ──→ [group 3 start]
        ↑                  ↑                          ↑                   ↑
   prevGroupEnd         Kth node                   1st node          nextGroupStart
                    (now 1st in group)          (now last in group)
                                                = new prevGroupEnd
```

---

## Summary Table

| Variation | Description | Key Difference |
|-----------|-------------|----------------|
| Reverse K-group (skip short) | Reverse every K nodes, leave tail | Check count ≥ K before reversing |
| Reverse K-group (include short) | Reverse every K nodes, including tail | No count check |
| Reverse alternate K-group | Reverse group, skip group, repeat | Skip K nodes between reversals |
| Reverse first K nodes | Just reverse first K, leave rest | Single reversal only |

---

## Quick Revision Questions

1. **What is the time complexity of K-group reversal and why isn't it O(n × k)?**
   <details><summary>Answer</summary>O(n). Each node is visited at most twice — once to check if K nodes exist, and once during the actual reversal. The total work is 2n, which is O(n).</details>

2. **Why do we need a dummy node in the iterative approach?**
   <details><summary>Answer</summary>The first group's reversal changes the HEAD node. The dummy node provides a stable node before HEAD so that `prevGroupEnd.next` can be updated uniformly for all groups, including the first.</details>

3. **In the recursive approach, why does `HEAD.next = reverseKGroupRecursive(curr, k)` work?**
   <details><summary>Answer</summary>After reversing K nodes, HEAD becomes the LAST node of the reversed group. `curr` points to the start of the remaining list. So `HEAD.next` (the tail of this group) should connect to whatever the recursion returns as the head of the next group.</details>

4. **How would you handle "reverse last K nodes" instead of "reverse first K nodes"?**
   <details><summary>Answer</summary>Use the Nth-from-end technique: find the (K+1)th node from the end, then reverse the sublist from that point onwards. Or reverse the whole list, reverse first K, then reverse the rest.</details>

5. **What modification makes the algorithm reverse the short tail group too?**
   <details><summary>Answer</summary>Remove the count check at the beginning. Instead of returning HEAD when `count < k`, always proceed with reversal regardless of how many nodes remain.</details>

6. **How does alternate K-group reversal differ in logic from standard K-group?**
   <details><summary>Answer</summary>After reversing one group of K, skip the next K nodes without reversing them (just traverse forward K steps), then recurse on the rest. The skip step is an extra loop of K iterations.</details>

---

[← Previous: Iterative and Recursive Reversal](01-iterative-and-recursive-reversal.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Palindrome and DLL Reversal →](03-palindrome-and-dll-reversal.md)

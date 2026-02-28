# Chapter 3: Palindrome Check and Doubly Linked List Reversal

[← Previous: Reverse in Groups](02-reverse-in-groups.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 7 — Merge Two Sorted Lists →](../07-Merge-Operations/01-merge-two-sorted-lists.md)

---

## Chapter Overview

This chapter applies reversal to two important scenarios: checking if a linked list is a **palindrome** (a classic interview problem combining slow-fast + reversal), and reversing **doubly linked lists** (where two pointers must be swapped).

---

## 3.1 Palindrome Linked List

### The Problem

Determine if a singly linked list reads the same forward and backward.

```
  Palindrome:      1 → 2 → 3 → 2 → 1    ← YES
  Not palindrome:  1 → 2 → 3 → 4 → 5    ← NO
```

### Approach: Stack-Based (O(n) Space)

```
function isPalindromeStack(HEAD):
    stack = empty stack
    curr = HEAD
    
    // Push all values
    while curr ≠ NULL:
        stack.push(curr.data)
        curr = curr.next
    
    // Compare with list
    curr = HEAD
    while curr ≠ NULL:
        if curr.data ≠ stack.pop():
            return false
        curr = curr.next
    
    return true
```

Simple but uses **O(n) extra space**.

---

### Approach: Reverse Second Half (O(1) Space) ⭐

The optimal approach — combines **three techniques**:

1. **Find middle** (slow-fast pointers)
2. **Reverse second half**
3. **Compare** both halves

```
function isPalindrome(HEAD):
    if HEAD == NULL or HEAD.next == NULL: return true
    
    // Step 1: Find middle
    slow = HEAD, fast = HEAD
    while fast.next ≠ NULL and fast.next.next ≠ NULL:
        slow = slow.next
        fast = fast.next.next
    
    // Step 2: Reverse second half
    secondHalf = reverse(slow.next)
    
    // Step 3: Compare
    first = HEAD
    second = secondHalf
    result = true
    while second ≠ NULL:
        if first.data ≠ second.data:
            result = false
            break
        first = first.next
        second = second.next
    
    // Step 4 (optional): Restore the list
    slow.next = reverse(secondHalf)
    
    return result
```

### Visual Trace: Is [1, 2, 3, 2, 1] a palindrome?

```
  Step 1: Find middle
  
  s=1, f=1 → s=2, f=3 → s=3, f=5
  [1] → [2] → [3] → [2] → [1] → NULL
                ↑                  ↑
              slow               fast
  
  Middle = Node(3)


  Step 2: Reverse second half (after Node(3))
  
  Before: 3.next = [2] → [1] → NULL
  After:  3.next = [1] → [2] → NULL
  
  [1] → [2] → [3]    [1] → [2] → NULL
   ↑            ↑      ↑
  first       slow   secondHalf


  Step 3: Compare
  
  first=1, second=1  → match ✓
  first=2, second=2  → match ✓
  second=NULL         → STOP
  
  Result: TRUE — it's a palindrome! ✓


  Step 4: Restore
  
  Reverse secondHalf again: [1] → [2] becomes [2] → [1]
  slow.next = [2] → [1]
  
  Restored: [1] → [2] → [3] → [2] → [1] → NULL  ✓
```

### Trace: Is [1, 2, 3, 4] a palindrome?

```
  Step 1: Find middle (for even, stop at first middle)
  
  s=1, f=1 → s=2, f=3
  fast.next = 4, fast.next.next = NULL → STOP
  
  [1] → [2] → [3] → [4] → NULL
          ↑           ↑
        slow        fast
  
  Step 2: Reverse second half (after Node(2))
  [3] → [4] becomes [4] → [3]
  
  Step 3: Compare
  first=1, second=4  → NO MATCH ✗
  
  Result: FALSE ✓
```

---

## 3.2 Complexity Comparison — Palindrome Methods

| Method | Time | Space | Destroys list? |
|--------|------|-------|----------------|
| Stack | O(n) | O(n) | No |
| Reverse second half | O(n) | O(1) | Yes (can restore) |
| Recursion | O(n) | O(n) | No |

---

## 3.3 Recursive Palindrome Check

```
function isPalindromeRecursive(HEAD):
    left = HEAD
    
    function check(right):
        if right == NULL: return true
        
        // Recurse to the end first
        result = check(right.next)
        if result == false: return false
        
        // Compare left and right
        if left.data ≠ right.data: return false
        
        // Move left forward
        left = left.next
        return true
    
    return check(HEAD)
```

```
  How it works:
  
  check recurses to the END, then unwinds.
  During unwinding, 'right' moves backward (via stack),
  while 'left' moves forward (via closure).
  
  Call stack for [1,2,3,2,1]:
  
  check(1) → check(2) → check(3) → check(2) → check(1) → check(NULL)=true
  
  Unwinding:
  right=1, left=1 → match ✓, left→2
  right=2, left=2 → match ✓, left→3
  right=3, left=3 → match ✓, left→4 (doesn't matter)
  right=2, left=2... wait, left is now at position 4
  
  Actually left only compares for the first half!
  The second half comparisons are redundant but don't cause issues.
```

---

## 3.4 Doubly Linked List Reversal

### The Difference

In a DLL, each node has **two** pointers. Reversal swaps them.

```
  Before:
  NULL ← [1] ⇄ [2] ⇄ [3] ⇄ [4] → NULL
  HEAD ──►↑                    ↑◄── TAIL

  After:
  NULL ← [4] ⇄ [3] ⇄ [2] ⇄ [1] → NULL
  HEAD ──►↑                    ↑◄── TAIL
```

### Algorithm

```
function reverseDLL(HEAD):
    if HEAD == NULL: return NULL
    
    curr = HEAD
    temp = NULL
    
    while curr ≠ NULL:
        // Swap prev and next pointers
        temp = curr.prev
        curr.prev = curr.next
        curr.next = temp
        
        // Move to "next" node (which is now curr.prev after swap)
        curr = curr.prev
    
    // After the loop, temp.prev is the last node processed
    // But we need the new HEAD
    if temp ≠ NULL:
        HEAD = temp.prev
    
    return HEAD
```

### Why `curr = curr.prev`?

```
  After swapping prev and next:
  
  Before swap:         After swap:
  curr.prev = A        curr.prev = B (was next)
  curr.next = B        curr.next = A (was prev)
  
  To move FORWARD (to what was the next node), 
  we now go to curr.prev (because prev and next are swapped!)
```

### Step-by-Step Trace: 1 ⇄ 2 ⇄ 3

```
  Initial: NULL ←[1]⇄[2]⇄[3]→ NULL
  curr = Node(1), temp = NULL

  ─── Step 1: curr = Node(1) ───
  temp = curr.prev = NULL
  curr.prev = curr.next = Node(2)
  curr.next = temp = NULL
  curr = curr.prev = Node(2)    (move to what was "next")

  Node(1): prev→2, next→NULL
  
  
  ─── Step 2: curr = Node(2) ───
  temp = curr.prev = Node(1)
  curr.prev = curr.next = Node(3)
  curr.next = temp = Node(1)
  curr = curr.prev = Node(3)

  Node(2): prev→3, next→1
  

  ─── Step 3: curr = Node(3) ───
  temp = curr.prev = Node(2)
  curr.prev = curr.next = NULL
  curr.next = temp = Node(2)
  curr = curr.prev = NULL       → EXIT loop

  Node(3): prev→NULL, next→2


  New HEAD: temp.prev = Node(2).prev = Node(3)
  
  Wait, temp = Node(2) after step 3.
  temp.prev was set to Node(3) in step 2... 
  but in step 3, we set curr(Node(3)).prev to NULL and curr.next to Node(2)
  temp is still Node(2) from step 2.
  
  Hmm, let me re-read the final condition:
  After loop, curr = NULL. 
  temp = the last value assigned to temp in the loop = curr.prev from step 3.
  In step 3: temp = curr.prev = Node(2) (before swap)
  
  HEAD = temp.prev → But temp.prev was already modified in step 2!
  In step 2: Node(2).prev = Node(3)
  So temp.prev = Node(3) → HEAD = Node(3) ✓
  
  Final: NULL ← [3] ⇄ [2] ⇄ [1] → NULL
         HEAD → [3]  ✓
```

### Simpler (Clearer) DLL Reversal

```
function reverseDLLSimple(HEAD):
    if HEAD == NULL: return NULL
    
    curr = HEAD
    
    while curr ≠ NULL:
        // Swap prev and next
        swap(curr.prev, curr.next)
        
        // The new HEAD is the last node we visit
        if curr.prev == NULL:        ← After swap, prev=NULL means
            HEAD = curr              ← this was originally the TAIL
            break
        
        curr = curr.prev             ← Move forward (swapped!)
    
    return HEAD
```

---

## 3.5 DLL Reversal — Alternative (Using Two Pointers)

```
function reverseDLLTwoPointers(HEAD):
    if HEAD == NULL: return NULL
    
    // Find TAIL
    tail = HEAD
    while tail.next ≠ NULL:
        tail = tail.next
    
    // Swap data from both ends, moving inward
    left = HEAD
    right = tail
    while left ≠ right and left.prev ≠ right:
        swap(left.data, right.data)
        left = left.next
        right = right.prev
    
    return HEAD    ← HEAD doesn't change, only data swaps
```

```
  This swaps DATA, not pointers.
  Pro: Simpler logic
  Con: If data is large/complex, pointer swap is better
```

---

## 3.6 Applications of Linked List Reversal

| Application | How Reversal Helps |
|-------------|-------------------|
| Palindrome check | Reverse 2nd half, compare with 1st |
| Add two numbers | Numbers stored in reverse → process digit by digit |
| Binary to decimal | Reverse bits stored as nodes |
| Undo/Redo stack | Reverse operation sequence |
| Print in reverse (SLL) | Reverse → print → reverse back |
| Reorder list | L0→Ln→L1→Ln-1→... requires reversing second half |

---

## Summary Table

| Operation | Time | Space | Key Technique |
|-----------|------|-------|---------------|
| Palindrome (stack) | O(n) | O(n) | Push all, pop and compare |
| Palindrome (reverse half) | O(n) | O(1) | Find middle + reverse + compare |
| Palindrome (recursive) | O(n) | O(n) | Recurse to end, compare on unwind |
| DLL reversal (swap pointers) | O(n) | O(1) | Swap prev/next for each node |
| DLL reversal (swap data) | O(n) | O(1) | Two pointers from ends, swap data |

---

## Quick Revision Questions

1. **What three techniques are combined to check a palindrome in O(1) space?**
   <details><summary>Answer</summary>1) Find the middle using slow-fast pointers. 2) Reverse the second half of the list. 3) Compare the first half with the reversed second half node by node.</details>

2. **Why do we find the "first middle" (not second) for even-length palindrome checks?**
   <details><summary>Answer</summary>For even-length lists like [1,2,2,1], the first half is [1,2] and second half is [2,1]. If we take the second middle, the halves overlap. By stopping at the first middle, we split cleanly into equal halves.</details>

3. **Should you restore the list after palindrome check? Why?**
   <details><summary>Answer</summary>Good practice says yes — modifying the caller's list is a side effect. Restore by reversing the second half again. However, in many interview settings, the problem doesn't require restoration.</details>

4. **In DLL reversal, why do we move to `curr.prev` instead of `curr.next`?**
   <details><summary>Answer</summary>After swapping `prev` and `next` for the current node, what was originally `curr.next` is now stored in `curr.prev`. So to advance forward, we follow `curr.prev`.</details>

5. **What's the advantage of swapping pointers vs swapping data in DLL reversal?**
   <details><summary>Answer</summary>Swapping pointers works for any data type and size — you only change 2 pointers per node. Swapping data requires copying potentially large data structures and may not be possible for all types.</details>

6. **How would you check if a DLL is a palindrome efficiently?**
   <details><summary>Answer</summary>Use two pointers: one at HEAD moving forward, one at TAIL moving backward. Compare values until they meet in the middle. O(n) time, O(1) space — no reversal needed because DLL has backward pointers!</details>

---

[← Previous: Reverse in Groups](02-reverse-in-groups.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Unit 7 — Merge Two Sorted Lists →](../07-Merge-Operations/01-merge-two-sorted-lists.md)

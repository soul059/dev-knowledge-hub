# Chapter 2: Arithmetic Operations and List Reordering

[← Previous: Flatten and Clone](01-flatten-and-clone.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Rotation and Sorting →](03-rotation-and-sorting.md)

---

## Chapter Overview

Using linked lists to perform **arithmetic on large numbers** (adding, multiplying numbers digit-by-digit), and **reordering** problems like the classic "reorder list" (L0→Ln→L1→Ln-1→...).

---

## 2.1 Add Two Numbers (Digits in Reverse Order)

### Problem (LeetCode 2)

Two non-negative integers are stored as linked lists with digits in **reverse** order. Add them and return the sum as a linked list.

```
  342 + 465 = 807
  
  Input:   2 → 4 → 3     (342 reversed)
         + 5 → 6 → 4     (465 reversed)
  
  Output:  7 → 0 → 8     (807 reversed)
```

### Algorithm

```
function addTwoNumbers(A, B):
    dummy = createNode(0)
    tail = dummy
    carry = 0
    
    while A ≠ NULL or B ≠ NULL or carry ≠ 0:
        sum = carry
        if A ≠ NULL:
            sum += A.data
            A = A.next
        if B ≠ NULL:
            sum += B.data
            B = B.next
        
        carry = sum / 10             ← Integer division
        digit = sum % 10
        
        tail.next = createNode(digit)
        tail = tail.next
    
    return dummy.next
```

### Step-by-Step Trace

```
  A: 2 → 4 → 3     B: 5 → 6 → 4

  | Step | A | B | carry | sum | digit | carry_out | Result |
  |------|---|---|-------|-----|-------|-----------|--------|
  | 1    | 2 | 5 | 0     | 7   | 7     | 0         | 7      |
  | 2    | 4 | 6 | 0     | 10  | 0     | 1         | 7→0    |
  | 3    | 3 | 4 | 1     | 8   | 8     | 0         | 7→0→8  |

  All NULL and carry=0 → STOP
  Result: 7 → 0 → 8 ✓ (807 reversed)
```

### Edge Cases

```
  Different lengths:          Extra carry at end:
  99 + 1 (= 100)             999 + 1 (= 1000)
  
  9→9  +  1                  9→9→9  +  1
  
  | A | B | carry | digit |  | A | B | carry | digit |
  | 9 | 1 | 0→10  | 0    |  | 9 | 1 | 0→10  | 0    |
  | 9 | - | 1→10  | 0    |  | 9 | - | 1→10  | 0    |
  | - | - | 1→1   | 1    |  | 9 | - | 1→10  | 0    |
                             | - | - | 1→1   | 1    |
  
  Result: 0→0→1 (100) ✓    Result: 0→0→0→1 (1000) ✓
  
  The carry check in the while condition handles extra digits!
```

---

## 2.2 Add Two Numbers (Digits in Forward Order)

### Problem (LeetCode 445)

Same problem, but digits are in **forward** order (most significant first).

```
  7243 + 564 = 7807
  
  Input:   7 → 2 → 4 → 3
         +      5 → 6 → 4
  
  Output:  7 → 8 → 0 → 7
```

### Approach 1: Reverse, Add, Reverse

```
function addForward(A, B):
    A = reverse(A)
    B = reverse(B)
    result = addTwoNumbers(A, B)    ← Use reverse-order algorithm
    return reverse(result)
```

### Approach 2: Stack-Based (No Reversal)

```
function addForwardStack(A, B):
    stackA = empty stack
    stackB = empty stack
    
    curr = A
    while curr ≠ NULL:
        stackA.push(curr.data)
        curr = curr.next
    
    curr = B
    while curr ≠ NULL:
        stackB.push(curr.data)
        curr = curr.next
    
    carry = 0
    result = NULL
    
    while stackA not empty or stackB not empty or carry ≠ 0:
        sum = carry
        if stackA not empty: sum += stackA.pop()
        if stackB not empty: sum += stackB.pop()
        
        carry = sum / 10
        digit = sum % 10
        
        node = createNode(digit)
        node.next = result            ← Insert at beginning!
        result = node
    
    return result
```

```
  Why insert at beginning?
  We compute digits from LEAST significant (via stacks).
  Inserting at the beginning naturally reverses the order,
  giving us most-significant-first output.
```

---

## 2.3 Multiply Two Numbers as Linked Lists

```
  12 × 34 = 408
  
  A: 1 → 2
  B: 3 → 4
```

### Algorithm

```
function multiply(A, B):
    // Convert to numbers (for small numbers)
    numA = 0
    curr = A
    while curr ≠ NULL:
        numA = numA * 10 + curr.data
        curr = curr.next
    
    numB = 0
    curr = B
    while curr ≠ NULL:
        numB = numB * 10 + curr.data
        curr = curr.next
    
    product = numA × numB
    
    // Convert back to linked list
    return numberToList(product)
```

For very large numbers (overflow risk), use digit-by-digit multiplication:

```
  Algorithm: like long multiplication in school
  
       1  2
     × 3  4
    ──────
       4  8    (12 × 4)
    3  6  0    (12 × 3, shifted left by 1)
    ──────
    4  0  8
  
  Each partial product is a linked list.
  Sum all partial products using addTwoNumbers.
```

---

## 2.4 Reorder List

### Problem (LeetCode 143)

Reorder L0→L1→...→Ln to L0→Ln→L1→Ln-1→L2→Ln-2→...

```
  Before:  1 → 2 → 3 → 4 → 5
  After:   1 → 5 → 2 → 4 → 3
```

### The Three-Step Approach

```
  Step 1: Find the middle
  Step 2: Reverse the second half
  Step 3: Merge alternately (interleave)
```

### Algorithm

```
function reorderList(HEAD):
    if HEAD == NULL or HEAD.next == NULL: return
    
    // Step 1: Find middle
    slow = HEAD, fast = HEAD
    while fast.next ≠ NULL and fast.next.next ≠ NULL:
        slow = slow.next
        fast = fast.next.next
    
    // Step 2: Reverse second half
    secondHalf = reverse(slow.next)
    slow.next = NULL                  ← Cut the list
    
    // Step 3: Interleave
    first = HEAD
    second = secondHalf
    while second ≠ NULL:
        temp1 = first.next
        temp2 = second.next
        first.next = second
        second.next = temp1
        first = temp1
        second = temp2
```

### Visual Trace: [1, 2, 3, 4, 5]

```
  ═══ Step 1: Find middle ═══
  slow = 3, fast = 5
  Middle = Node(3)

  ═══ Step 2: Reverse second half ═══
  Second half: 4 → 5
  Reversed:    5 → 4
  Cut: 3.next = NULL
  
  First:  1 → 2 → 3 → NULL
  Second: 5 → 4 → NULL

  ═══ Step 3: Interleave ═══
  
  | Step | first | second | Action |
  |------|-------|--------|--------|
  | 1    | 1     | 5      | 1→5→2  |
  | 2    | 2     | 4      | 2→4→3  |
  | 3    | 3     | NULL   | STOP   |
  
  Result: 1 → 5 → 2 → 4 → 3 → NULL ✓
```

### Interleave Step Detail

```
  first=1, second=5:
    temp1 = 1.next = 2
    temp2 = 5.next = 4
    1.next = 5         → 1 → 5
    5.next = temp1 = 2 → 1 → 5 → 2
    first = 2, second = 4

  first=2, second=4:
    temp1 = 2.next = 3
    temp2 = 4.next = NULL
    2.next = 4         → 2 → 4
    4.next = temp1 = 3 → 2 → 4 → 3
    first = 3, second = NULL → STOP

  Final: 1 → 5 → 2 → 4 → 3 ✓
```

---

## 2.5 Odd-Even Linked List

### Problem (LeetCode 328)

Group all odd-indexed nodes together followed by even-indexed nodes.

```
  Before:  1 → 2 → 3 → 4 → 5
  After:   1 → 3 → 5 → 2 → 4
           (odd positions)  (even positions)
```

### Algorithm

```
function oddEvenList(HEAD):
    if HEAD == NULL: return NULL
    
    odd = HEAD
    even = HEAD.next
    evenHead = even                   ← Save start of even list
    
    while even ≠ NULL and even.next ≠ NULL:
        odd.next = even.next          ← Link odd to next odd
        odd = odd.next
        even.next = odd.next          ← Link even to next even
        even = even.next
    
    odd.next = evenHead               ← Attach even list after odd
    return HEAD
```

### Trace

```
  [1] → [2] → [3] → [4] → [5] → NULL
   odd   even

  Step 1: odd.next = 3, odd = 3
          even.next = 4, even = 4
  Odd:  1 → 3    Even: 2 → 4

  Step 2: odd.next = 5, odd = 5
          even.next = NULL, even = NULL
  Odd:  1 → 3 → 5    Even: 2 → 4

  even = NULL → STOP
  5.next = evenHead(2)
  
  Result: 1 → 3 → 5 → 2 → 4 → NULL ✓
```

---

## 2.6 Subtract Two Numbers as Linked Lists

Given two numbers as linked lists (most significant digit first), subtract the smaller from the larger.

```
  A: 1 → 0 → 0    (100)
  B:      5 → 0    (50)
  
  Result: 5 → 0    (50)
```

### Algorithm Outline

```
1. Determine which number is larger (compare lengths, then digit by digit)
2. Pad the shorter number with leading zeros
3. Subtract digit by digit with borrowing (recursion helps process right to left)
4. Remove leading zeros from the result
```

```
function subtract(A, B):
    // Pad shorter list
    lenA = length(A), lenB = length(B)
    while lenA < lenB: A = prependZero(A); lenA++
    while lenB < lenA: B = prependZero(B); lenB++
    
    // Ensure A ≥ B
    if isSmaller(A, B): swap(A, B)
    
    borrow = 0
    result = subtractHelper(A, B, borrow)
    
    // Remove leading zeros
    while result ≠ NULL and result.data == 0 and result.next ≠ NULL:
        result = result.next
    
    return result
```

---

## Summary Table

| Problem | Core Technique | Time | Space |
|---------|---------------|------|-------|
| Add two numbers (reversed) | Traverse + carry propagation | O(max(n,m)) | O(max(n,m)) |
| Add two numbers (forward) | Stack or reverse first | O(n+m) | O(n+m) |
| Multiply two numbers | Digit-by-digit + add | O(n×m) | O(n+m) |
| Reorder list | Find middle + reverse + interleave | O(n) | O(1) |
| Odd-even grouping | Two-pointer separation | O(n) | O(1) |
| Subtract two numbers | Padding + borrow propagation | O(max(n,m)) | O(max(n,m)) |

---

## Quick Revision Questions

1. **In "add two numbers" (reversed), why do we check `carry ≠ 0` in the while condition?**
   <details><summary>Answer</summary>When both lists are exhausted but carry is 1 (e.g., 99+1=100), we need an extra node for the carry. Without the carry check, we'd produce 00 instead of 001 (which represents 100).</details>

2. **What three techniques does "reorder list" combine?**
   <details><summary>Answer</summary>1) Find middle using slow-fast pointers. 2) Reverse the second half. 3) Interleave (merge alternately) the first half and reversed second half. Each technique is O(n), so the total is O(n).</details>

3. **In the stack-based forward addition, why do we insert at the beginning?**
   <details><summary>Answer</summary>Stacks give us digits from least significant to most significant. By inserting each new digit at the head of the result list, we naturally build the number in most-significant-first order without a final reversal.</details>

4. **Why is the odd-even list problem O(1) space?**
   <details><summary>Answer</summary>We only re-link existing nodes using a constant number of pointers (odd, even, evenHead). No new nodes are created. We split the list into two chains and then connect them.</details>

5. **How do you handle different-length lists in addition?**
   <details><summary>Answer</summary>The algorithm naturally handles it: when one list is exhausted (NULL), we treat its contribution as 0. The `if A ≠ NULL: sum += A.data` check skips exhausted lists while continuing with the other.</details>

6. **Could you solve "reorder list" without reversing the second half?**
   <details><summary>Answer</summary>Yes, using a deque or array of node pointers — but that uses O(n) space. The reverse + interleave approach achieves O(1) space, which is optimal.</details>

---

[← Previous: Flatten and Clone](01-flatten-and-clone.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Rotation and Sorting →](03-rotation-and-sorting.md)

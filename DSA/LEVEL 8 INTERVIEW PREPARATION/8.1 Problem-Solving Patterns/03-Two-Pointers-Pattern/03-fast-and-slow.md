# Chapter 3: Fast and Slow Pointers

## 📋 Chapter Overview
The **Floyd's Tortoise and Hare** technique — two pointers moving at different speeds. Essential for cycle detection in linked lists and finding middle elements.

---

## 🎯 Core Concept

```
  FAST AND SLOW POINTERS
  
  Linked List: 1 → 2 → 3 → 4 → 5 → 6 → NULL
               S                              
               F                              
  
  Slow moves 1 step, Fast moves 2 steps:
  
  Step 1:  1 → 2 → 3 → 4 → 5 → 6
              S         F              
  Step 2:  1 → 2 → 3 → 4 → 5 → 6
                  S              F     
  Step 3:  1 → 2 → 3 → 4 → 5 → 6 → NULL
                      S              F(NULL)
  
  When fast reaches end, slow is at MIDDLE!
```

---

## 🔄 Cycle Detection (Floyd's Algorithm)

```
  Linked List WITH Cycle:
  
  1 → 2 → 3 → 4 → 5
              ↑         ↓
              8 ← 7 ← 6
  
  Slow (1 step) and Fast (2 steps):
  
  Step 0: S=1, F=1
  Step 1: S=2, F=3
  Step 2: S=3, F=5
  Step 3: S=4, F=7
  Step 4: S=5, F=4  (fast wraps around cycle!)
  Step 5: S=6, F=6  ← MEET! Cycle detected! ✓
  
  ┌──────────────────────────────────────────────┐
  │  WHY THEY MUST MEET:                         │
  │                                              │
  │  Once both are in the cycle:                 │
  │  • Fast gains 1 step on slow each iteration  │
  │  • Distance between them decreases by 1      │
  │  • Eventually distance = 0 → they meet!      │
  │                                              │
  │  Like two runners on a circular track —      │
  │  the faster runner always laps the slower.   │
  └──────────────────────────────────────────────┘
```

### Pseudocode:

```
function hasCycle(head):
    if head == null: return false
    
    slow = head
    fast = head
    
    while fast != null AND fast.next != null:
        slow = slow.next           // 1 step
        fast = fast.next.next      // 2 steps
        
        if slow == fast:
            return true            // cycle detected!
    
    return false                   // no cycle
```

**Time:** O(n)  **Space:** O(1)

---

## 🔍 Finding the Cycle Start

```
  After detecting cycle (slow == fast at meeting point):
  
  1 → 2 → 3 → 4 → 5
              ↑         ↓
              8 ← 7 ← 6
                        ↑
                    Meeting point
  
  STEP 1: Reset one pointer to head
  STEP 2: Move both at SAME speed (1 step each)
  STEP 3: They meet at the CYCLE START!
  
  ptr1 = head = 1
  ptr2 = meeting = 6
  
  Step 1: ptr1=2, ptr2=7
  Step 2: ptr1=3, ptr2=8
  Step 3: ptr1=4, ptr2=4  ← MEET at cycle start!
  
  ┌──────────────────────────────────────────────┐
  │  WHY THIS WORKS (Mathematical Proof):        │
  │                                              │
  │  Let:                                        │
  │    a = distance from head to cycle start     │
  │    b = distance from cycle start to meeting  │
  │    c = cycle length                          │
  │                                              │
  │  At meeting point:                           │
  │    slow traveled: a + b                      │
  │    fast traveled: a + b + n·c (n loops)      │
  │    fast = 2 × slow                           │
  │    a + b + n·c = 2(a + b)                    │
  │    n·c = a + b                               │
  │    a = n·c - b                               │
  │                                              │
  │  So: distance from head to cycle start (a)   │
  │  = distance from meeting to cycle start      │
  │    going forward (c - b)                     │
  │    plus some full loops                      │
  └──────────────────────────────────────────────┘
```

### Pseudocode:

```
function detectCycleStart(head):
    slow = head
    fast = head
    
    // Phase 1: Detect cycle
    while fast != null AND fast.next != null:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            break
    
    if fast == null OR fast.next == null:
        return null    // no cycle
    
    // Phase 2: Find cycle start
    ptr1 = head
    ptr2 = slow    // meeting point
    
    while ptr1 != ptr2:
        ptr1 = ptr1.next
        ptr2 = ptr2.next
    
    return ptr1    // cycle start
```

---

## 🧪 Finding Middle of Linked List

```
  List: 1 → 2 → 3 → 4 → 5
  
  Slow (1 step), Fast (2 steps):
  
  Start: S=1, F=1
  Step 1: S=2, F=3
  Step 2: S=3, F=5
  F.next = null → STOP
  
  Middle = slow = 3 ✓
  
  Even length: 1 → 2 → 3 → 4 → 5 → 6
  
  Start: S=1, F=1
  Step 1: S=2, F=3
  Step 2: S=3, F=5
  Step 3: S=4, F=null → STOP
  
  Middle = slow = 4 (second middle) ✓
```

---

## 🧪 Happy Number Problem

**Problem:** Determine if n is a "happy number" (sum of squares of digits eventually reaches 1).

```
  n = 19:
  1² + 9² = 82
  8² + 2² = 68
  6² + 8² = 100
  1² + 0² + 0² = 1  ✓ Happy!
  
  n = 2:
  2² = 4
  4² = 16
  1² + 6² = 37
  3² + 7² = 58
  5² + 8² = 89
  8² + 9² = 145
  1² + 4² + 5² = 42
  4² + 2² = 20
  2² + 0² = 4  ← CYCLE! Not happy.
  
  ┌──────────────────────────────────────────────┐
  │  This is a LINKED LIST problem in disguise!  │
  │                                              │
  │  Each number points to the next (sum of sq.) │
  │  If it reaches 1 → terminates               │
  │  If it cycles → never reaches 1             │
  │                                              │
  │  Use fast/slow to detect the cycle!          │
  └──────────────────────────────────────────────┘
```

### Pseudocode:

```
function isHappy(n):
    slow = n
    fast = n
    
    do:
        slow = sumOfSquares(slow)           // 1 step
        fast = sumOfSquares(sumOfSquares(fast))  // 2 steps
    while slow != fast
    
    return slow == 1

function sumOfSquares(n):
    sum = 0
    while n > 0:
        digit = n % 10
        sum += digit * digit
        n = n / 10
    return sum
```

---

## 📊 Fast/Slow Applications

| Problem | Slow Speed | Fast Speed | What It Finds |
|---------|-----------|-----------|--------------|
| Cycle detection | 1 step | 2 steps | Whether cycle exists |
| Cycle start | 1 step | 1 step (phase 2) | Where cycle begins |
| Middle element | 1 step | 2 steps | Middle of list |
| Happy number | 1 transform | 2 transforms | Cycle in sequence |
| Palindrome list | 1 step (+ reverse) | 2 steps (find mid) | If list is palindrome |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Fast/Slow | Two pointers at different speeds |
| Cycle Detection | Fast catches slow if cycle exists |
| Cycle Start | Reset one to head, move both at speed 1 |
| Middle Finding | When fast reaches end, slow is at middle |
| Happy Number | Implicit linked list — detect cycle with fast/slow |
| Time | O(n) for all applications |
| Space | O(1) — no extra data structures |

---

## ❓ Quick Revision Questions

1. **Why must fast and slow pointers meet if there's a cycle?**
   > Fast gains 1 step on slow per iteration. The gap decreases by 1 each time until it's 0 (they meet). Like a faster runner lapping a slower one.

2. **How do you find the start of a cycle?**
   > After detection, reset one pointer to head. Move both at speed 1. They meet at the cycle start.

3. **Why does finding the middle work?**
   > Fast moves 2x speed. When fast traverses the full list, slow has traversed half — landing at the middle.

4. **How is "happy number" related to fast/slow pointers?**
   > The sequence of digit-square-sums forms an implicit linked list. If the sequence cycles (doesn't reach 1), fast/slow detects it.

5. **What's the space complexity of cycle detection?**
   > O(1) — only two pointers, no extra data structures.

6. **For an even-length list, which middle does slow point to?**
   > The second middle (e.g., in [1,2,3,4], slow points to 3, not 2).

---

[← Previous: Opposite Direction](02-opposite-direction.md) | [Next: Template and Variations →](04-template-and-variations.md)

[← Back to README](../README.md)

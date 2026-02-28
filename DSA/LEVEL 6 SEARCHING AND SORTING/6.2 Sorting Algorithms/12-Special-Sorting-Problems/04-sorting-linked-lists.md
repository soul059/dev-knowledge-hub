# Unit 12: Special Sorting Problems — Sorting Linked Lists

[← Previous: Custom Comparators](03-custom-comparators.md) | [Next: K-Way Merge →](05-k-way-merge.md)

---

## Overview

Sorting a linked list is fundamentally different from sorting an array: no random access means O(n) to reach any element. This changes which algorithms are efficient. **Merge Sort** is the champion for linked list sorting.

---

## Why Merge Sort Wins for Linked Lists

```
  ┌──────────────────────────────────────────────────────────────┐
  │  ARRAY vs LINKED LIST for sorting algorithms:               │
  │                                                              │
  │  Algorithm       │ Array    │ Linked List │ Why?             │
  │  ────────────────│──────────│─────────────│──────────────────│
  │  Quick Sort      │ Great ✓  │ Poor ✗      │ Pivot access O(n)│
  │  Heap Sort       │ Great ✓  │ Can't ✗     │ Needs array index│
  │  Insertion Sort  │ OK       │ OK          │ O(n²) both ways │
  │  Merge Sort      │ Good     │ BEST ✓      │ O(1) space LL!  │
  │  Bubble Sort     │ Bad      │ OK          │ Adjacent swaps  │
  │                                                              │
  │  Merge Sort on linked lists:                                │
  │  • O(n log n) time ✓                                       │
  │  • O(1) extra space ✓ (just rearrange pointers!)           │
  │  • Stable ✓                                                 │
  │  • No random access needed ✓                               │
  └──────────────────────────────────────────────────────────────┘
```

---

## Merge Sort on Linked List

### Algorithm

```
  1. Find the MIDDLE of the list (slow/fast pointer)
  2. Split into two halves
  3. Recursively sort each half
  4. MERGE the two sorted halves (rearrange pointers)
  
  ┌───────────────────────────────────────────┐
  │  4 → 2 → 1 → 3                          │
  │         ↓ split                           │
  │  4 → 2      1 → 3                        │
  │    ↓ split    ↓ split                     │
  │  4    2      1    3                       │
  │    ↓ merge    ↓ merge                     │
  │  2 → 4      1 → 3                        │
  │         ↓ merge                           │
  │  1 → 2 → 3 → 4                          │
  └───────────────────────────────────────────┘
```

### Java Implementation

```java
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}

public class LinkedListSort {
    
    public ListNode sortList(ListNode head) {
        // Base case
        if (head == null || head.next == null)
            return head;
        
        // Step 1: Find middle
        ListNode mid = getMiddle(head);
        ListNode rightHalf = mid.next;
        mid.next = null;  // Split the list
        
        // Step 2: Recursively sort
        ListNode left = sortList(head);
        ListNode right = sortList(rightHalf);
        
        // Step 3: Merge
        return merge(left, right);
    }
    
    private ListNode getMiddle(ListNode head) {
        ListNode slow = head, fast = head.next;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }
    
    private ListNode merge(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {  // <= for stability
                curr.next = l1;
                l1 = l1.next;
            } else {
                curr.next = l2;
                l2 = l2.next;
            }
            curr = curr.next;
        }
        
        curr.next = (l1 != null) ? l1 : l2;
        return dummy.next;
    }
}
```

### Python Implementation

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def sort_list(head):
    if not head or not head.next:
        return head
    
    # Find middle
    slow, fast = head, head.next
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    right = slow.next
    slow.next = None  # Split
    
    # Recursively sort
    left = sort_list(head)
    right = sort_list(right)
    
    # Merge
    return merge(left, right)

def merge(l1, l2):
    dummy = ListNode(0)
    curr = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            curr.next = l1
            l1 = l1.next
        else:
            curr.next = l2
            l2 = l2.next
        curr = curr.next
    
    curr.next = l1 or l2
    return dummy.next
```

---

## Bottom-Up Merge Sort (Iterative, True O(1) Space)

The recursive version uses O(log n) stack space. For true O(1) extra space:

```java
public ListNode sortListIterative(ListNode head) {
    if (head == null || head.next == null) return head;
    
    // Count length
    int len = 0;
    ListNode node = head;
    while (node != null) { len++; node = node.next; }
    
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    
    // Merge sublists of size 1, 2, 4, 8, ...
    for (int size = 1; size < len; size *= 2) {
        ListNode prev = dummy;
        ListNode curr = dummy.next;
        
        while (curr != null) {
            // Get left sublist of 'size' nodes
            ListNode left = curr;
            ListNode right = split(left, size);
            curr = split(right, size);  // remaining
            
            // Merge left and right
            prev = mergeAndReturnTail(prev, left, right);
        }
    }
    
    return dummy.next;
}

// Split list after 'size' nodes, return start of second part
private ListNode split(ListNode head, int size) {
    if (head == null) return null;
    for (int i = 1; i < size && head.next != null; i++) {
        head = head.next;
    }
    ListNode second = head.next;
    head.next = null;
    return second;
}
```

```
  Iterative Merge Sort Trace:
  
  List: 4 → 3 → 1 → 7 → 8 → 5 → 2 → 6
  
  size=1: Merge pairs of singles:
  (4,3)→3→4  (1,7)→1→7  (8,5)→5→8  (2,6)→2→6
  → 3→4 → 1→7 → 5→8 → 2→6
  
  size=2: Merge pairs of 2:
  (3→4, 1→7) → 1→3→4→7    (5→8, 2→6) → 2→5→6→8
  → 1→3→4→7 → 2→5→6→8
  
  size=4: Merge pairs of 4:
  (1→3→4→7, 2→5→6→8) → 1→2→3→4→5→6→7→8
  
  Done! ✓
```

---

## Insertion Sort on Linked List

For nearly sorted or small linked lists:

```java
public ListNode insertionSortList(ListNode head) {
    ListNode dummy = new ListNode(0);
    ListNode curr = head;
    
    while (curr != null) {
        ListNode next = curr.next;
        
        // Find insertion point in sorted part
        ListNode prev = dummy;
        while (prev.next != null && prev.next.val < curr.val) {
            prev = prev.next;
        }
        
        // Insert curr after prev
        curr.next = prev.next;
        prev.next = curr;
        
        curr = next;
    }
    
    return dummy.next;
}
```

```
  Trace: 4 → 2 → 1 → 3
  
  Sorted: (empty)
  Insert 4: 4
  Insert 2: 2 → 4          (2 goes before 4)
  Insert 1: 1 → 2 → 4      (1 goes before 2)
  Insert 3: 1 → 2 → 3 → 4  (3 goes between 2 and 4)
```

---

## Comparison for Linked Lists

```
  ┌──────────────────┬──────────┬──────────┬────────┐
  │  Algorithm       │ Time     │ Space    │ Stable │
  ├──────────────────┼──────────┼──────────┼────────┤
  │  Merge Sort (rec)│O(n lg n) │ O(lg n)  │ Yes    │
  │  Merge Sort (it) │O(n lg n) │ O(1) ✓  │ Yes    │
  │  Insertion Sort  │O(n²)     │ O(1)     │ Yes    │
  │  Quick Sort      │O(n lg n)*│ O(lg n)  │ No     │
  │  Bubble Sort     │O(n²)     │ O(1)     │ Yes    │
  └──────────────────┴──────────┴──────────┴────────┘
  
  * Quick Sort on LL: poor performance due to scanning for pivot
  
  WINNER: Iterative Merge Sort — O(n lg n) time, O(1) space, stable
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Best algorithm | Merge Sort |
| Why | No random access needed, O(1) extra space on LL |
| Finding middle | Slow/fast pointer technique |
| Merging | Rearrange pointers, no data copying |
| Space (recursive) | O(log n) stack |
| Space (iterative) | O(1) — true optimal |

---

## Quick Revision Questions

1. **Why is Merge Sort the best algorithm for linked lists?**
2. **How do you find the middle of a linked list?**
3. **Why does Merge Sort on a linked list use O(1) extra space (excluding recursion)?**
4. **How does bottom-up iterative merge sort work on a linked list?**
5. **Trace merge sort on the linked list: 5 → 1 → 3 → 2 → 4.**

---

[← Previous: Custom Comparators](03-custom-comparators.md) | [Next: K-Way Merge →](05-k-way-merge.md)

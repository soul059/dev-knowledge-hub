# Chapter 1: Memory Allocation and Memory Leaks

[← Previous: Rotation and Sorting](../08-Advanced-Problems/03-rotation-and-sorting.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Garbage Collection and Smart Pointers →](02-garbage-collection-and-smart-pointers.md)

---

## Chapter Overview

Understanding **how linked list nodes live in memory**, the differences between stack and heap allocation, how **memory leaks** occur, and how to prevent them. Essential knowledge for C/C++ and systems programming.

---

## 1.1 Stack vs Heap Memory

### Where Do Nodes Live?

```
  ┌─────────────────────────────────────────────────────────┐
  │                    MEMORY LAYOUT                        │
  ├─────────────────────────────────────────────────────────┤
  │                                                         │
  │  STACK (automatic)              HEAP (dynamic)          │
  │  ┌─────────────────┐          ┌─────────────────┐      │
  │  │ Local variables  │          │ malloc/new nodes │      │
  │  │ Function params  │          │ Persist beyond   │      │
  │  │ Return addresses │          │   function scope │      │
  │  │                  │          │                  │      │
  │  │ Auto-freed when  │          │ Must be freed    │      │
  │  │ function returns │          │ manually (C/C++) │      │
  │  └─────────────────┘          └─────────────────┘      │
  │                                                         │
  │  HEAD pointer                   Actual nodes             │
  │  (on stack)                     (on heap)                │
  └─────────────────────────────────────────────────────────┘
```

### Example in C

```c
// HEAD is on the stack
Node* head = NULL;

// Each node is on the HEAP
Node* newNode = (Node*)malloc(sizeof(Node));  // C
Node* newNode = new Node(data);                // C++

// The pointer 'newNode' is on the stack
// The Node it points to is on the heap
```

```
  Stack:                    Heap:
  ┌──────────┐             ┌────────────────┐
  │ head ────┼────────────►│ data: 10       │
  │          │             │ next: ─────────┼──►...
  └──────────┘             └────────────────┘
  
  'head' = a pointer variable (8 bytes on 64-bit)
  *head = the actual Node structure (on heap)
```

---

## 1.2 Node Size in Memory

### Typical Node Layout (64-bit System)

```
  Singly Linked List Node:
  ┌──────────────┬──────────────┐
  │    data      │    next      │
  │   (4 bytes)  │  (8 bytes)   │
  └──────────────┴──────────────┘
  Total: 12 bytes + padding = 16 bytes (aligned)

  Doubly Linked List Node:
  ┌──────────────┬──────────────┬──────────────┐
  │    data      │    prev      │    next       │
  │   (4 bytes)  │  (8 bytes)   │  (8 bytes)    │
  └──────────────┴──────────────┴──────────────┘
  Total: 20 bytes + padding = 24 bytes (aligned)
```

### Comparison: Array vs Linked List Memory

```
  Storing 1000 integers:
  
  Array:  1000 × 4 bytes = 4,000 bytes  (4 KB)
  
  SLL:    1000 × 16 bytes = 16,000 bytes (16 KB)
          ↑ 4x more memory!
  
  DLL:    1000 × 24 bytes = 24,000 bytes (24 KB)
          ↑ 6x more memory!
  
  The overhead comes from pointer storage and alignment padding.
```

---

## 1.3 Memory Allocation in Different Languages

| Language | Allocation | Deallocation | Risk |
|----------|-----------|-------------|------|
| C | `malloc()` | `free()` | Manual — leak if forgotten |
| C++ | `new` | `delete` | Manual — use smart pointers |
| Java | `new` | Garbage collected | No manual deallocation |
| Python | Object creation | Garbage collected | Reference cycles possible |
| JavaScript | Object creation | Garbage collected | No manual deallocation |
| Rust | `Box::new()` | Automatic (ownership) | Compile-time safety |

---

## 1.4 Memory Leaks

A memory leak occurs when allocated memory is no longer accessible but hasn't been freed.

### Leak Scenario 1: Lost Head Reference

```c
Node* head = createNode(1);
head->next = createNode(2);
head->next->next = createNode(3);

head = head->next;              // ← LEAK! Node(1) is now orphaned!
```

```
  Before head = head->next:
  head → [1] → [2] → [3] → NULL
  
  After head = head->next:
  head ──────→ [2] → [3] → NULL
         [1] → [2]   ← Node(1) still exists in memory
                       but nothing points to it!
                       LEAKED — 16 bytes wasted forever
```

### Leak Scenario 2: Forgetting to Free During Deletion

```c
// WRONG — memory leak
void deleteNode(Node** head_ref, int position) {
    Node* temp = *head_ref;
    for (int i = 0; i < position - 1; i++)
        temp = temp->next;
    
    temp->next = temp->next->next;  // ← Skipped node but didn't free it!
}

// CORRECT
void deleteNode(Node** head_ref, int position) {
    Node* temp = *head_ref;
    for (int i = 0; i < position - 1; i++)
        temp = temp->next;
    
    Node* toDelete = temp->next;
    temp->next = temp->next->next;
    free(toDelete);                  // ← Free the removed node!
}
```

### Leak Scenario 3: Losing Entire List

```c
Node* head = buildLargeList(1000000);

// Some operation...
head = NULL;                       // ← MASSIVE LEAK!
                                   // 1 million nodes lost!

// CORRECT
void freeList(Node* head) {
    while (head != NULL) {
        Node* temp = head;
        head = head->next;
        free(temp);
    }
}
freeList(head);
head = NULL;                       // Now safe
```

---

## 1.5 Dangling Pointers

A dangling pointer points to memory that has been freed.

```c
Node* a = createNode(10);
Node* b = a;                       // b also points to the same node

free(a);                           // Memory freed

printf("%d", b->data);             // ← UNDEFINED BEHAVIOR!
                                   // b is now a dangling pointer
```

```
  Before free:
  a ──→ ┌────────┐ ←── b
        │ data:10│
        └────────┘
  
  After free:
  a ──→ ┌────────┐ ←── b
        │ FREED  │        b is dangling!
        └────────┘
  
  The memory might be:
  1. Still contain 10 (seems to work — dangerous!)
  2. Contain garbage data
  3. Be allocated to something else
  4. Cause a segfault
```

### Prevention

```c
free(a);
a = NULL;                          // ← Always NULL after free
b = NULL;                          // ← Also NULL any aliases!
```

---

## 1.6 Double Free

Freeing the same memory twice causes undefined behavior.

```c
Node* a = createNode(10);
free(a);
free(a);                           // ← DOUBLE FREE! Heap corruption!
```

### Prevention

```c
free(a);
a = NULL;                          // NULL
free(a);                           // free(NULL) is safe — does nothing
```

---

## 1.7 Memory Leak Detection

### Tools

| Tool | Language | Description |
|------|----------|-------------|
| Valgrind | C/C++ | Detects leaks, invalid reads/writes |
| AddressSanitizer | C/C++ | Compile-time instrumentation |
| LeakSanitizer | C/C++ | Dedicated leak detector |
| Visual Studio | C/C++ | Built-in CRT debug heap |
| Java VisualVM | Java | Heap profiler |
| Python tracemalloc | Python | Memory allocation tracker |

### Valgrind Example

```bash
$ valgrind --leak-check=full ./my_program

==12345== LEAK SUMMARY:
==12345==    definitely lost: 48 bytes in 3 blocks
==12345==    indirectly lost: 0 bytes in 0 blocks
==12345==      possibly lost: 0 bytes in 0 blocks
==12345==    still reachable: 0 bytes in 0 blocks
==12345==         suppressed: 0 bytes in 0 blocks
```

---

## 1.8 Safe Deletion Patterns

### Delete Entire List

```c
void freeList(Node** head_ref) {
    Node* curr = *head_ref;
    Node* next;
    
    while (curr != NULL) {
        next = curr->next;         // Save next BEFORE freeing
        free(curr);
        curr = next;
    }
    
    *head_ref = NULL;              // Prevent dangling pointer
}
```

### Why Save Next Before Free?

```
  ┌─────────────┐     ┌─────────────┐
  │  data: 10   │     │  data: 20   │
  │  next: ─────┼────►│  next: NULL │
  └─────────────┘     └─────────────┘
       curr                curr->next
  
  If we free(curr) first:
  curr->next is GARBAGE — we can't reach the next node!
  
  So: save next = curr->next THEN free(curr)
```

---

## Summary Table

| Issue | Cause | Effect | Prevention |
|-------|-------|--------|------------|
| Memory leak | Not freeing nodes | Growing memory use | Always free deleted nodes |
| Dangling pointer | Using freed memory | Undefined behavior | Set pointers to NULL after free |
| Double free | Freeing same memory twice | Heap corruption | NULL after free, free(NULL) is safe |
| Buffer overflow | Writing past node bounds | Memory corruption | Bounds checking |
| Lost reference | Overwriting HEAD without free | Orphaned nodes | Free before re-assigning |

---

## Quick Revision Questions

1. **Why are linked list nodes allocated on the heap, not the stack?**
   <details><summary>Answer</summary>Stack memory is freed when a function returns. Nodes must persist beyond the function that creates them. Heap memory persists until explicitly freed (or garbage collected), making it suitable for dynamically growing data structures.</details>

2. **How much extra memory does a singly linked list use compared to an array for n integers?**
   <details><summary>Answer</summary>On a 64-bit system, each SLL node stores a 4-byte int + 8-byte pointer + 4-byte padding = 16 bytes vs. 4 bytes for an array element. That's 4× overhead. Plus, each node has a separate heap allocation with its own metadata overhead.</details>

3. **What is the difference between a memory leak and a dangling pointer?**
   <details><summary>Answer</summary>A memory leak means memory is allocated but never freed (inaccessible but still occupied). A dangling pointer means memory was freed but a pointer still references it (accessible but invalid). They're opposite problems.</details>

4. **Why must you save `curr->next` before calling `free(curr)`?**
   <details><summary>Answer</summary>After `free(curr)`, the memory at `curr` becomes invalid. Accessing `curr->next` would be undefined behavior — the next pointer value may be overwritten by the memory allocator. Saving it first ensures we can traverse.</details>

5. **Why is `free(NULL)` safe in C?**
   <details><summary>Answer</summary>The C standard guarantees that `free(NULL)` is a no-op (does nothing). This allows the defensive pattern: set pointer to NULL after freeing, then even an accidental second `free()` on the same variable is harmless.</details>

6. **In Java/Python, can linked lists still cause memory issues?**
   <details><summary>Answer</summary>Yes! While garbage collectors prevent traditional leaks, you can still have: (1) Logical leaks — holding references to unused nodes prevents GC. (2) Reference cycles (in some GC implementations). (3) Excessive memory use from node overhead. (4) OutOfMemoryError from too many allocations.</details>

---

[← Previous: Rotation and Sorting](../08-Advanced-Problems/03-rotation-and-sorting.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Garbage Collection and Smart Pointers →](02-garbage-collection-and-smart-pointers.md)

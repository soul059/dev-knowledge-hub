# Chapter 2: Garbage Collection and Smart Pointers

[← Previous: Memory Allocation and Leaks](01-memory-allocation-and-leaks.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Cache Performance and XOR List →](03-cache-performance-and-xor-list.md)

---

## Chapter Overview

How different languages **automatically manage memory** for linked list nodes: reference counting, tracing garbage collectors, and C++ smart pointers. Understanding these mechanisms helps you write leak-free code and avoid performance pitfalls.

---

## 2.1 The Memory Management Spectrum

```
  Manual ◄──────────────────────────────────────► Automatic
  
  C          C++           Rust        Java/Python/JS
  malloc     new/delete    Ownership   Garbage
  free       smart ptrs    + Borrow    Collection
             (RAII)        checking
  
  ← More control, more bugs    Less control, fewer bugs →
```

---

## 2.2 Reference Counting

### How It Works

Each object has a counter tracking how many references point to it. When the counter reaches 0, the object is freed.

```
  Node A created:      refcount = 1
  B = A;               refcount = 2
  B = NULL;            refcount = 1
  A = NULL;            refcount = 0 → FREE!
```

### Reference Counting and Linked Lists

```
  HEAD → [1] → [2] → [3] → NULL
  
  refcount:  1     1     1
  (each pointed to by exactly one previous node or HEAD)
  
  When we delete Node(2):
  HEAD → [1] → [3] → NULL
  
  Node(2) refcount drops to 0 → automatically freed ✓
```

### The Circular Reference Problem ⚠️

```
  [A] → [B] → [A]     (circular list)
  
  refcount(A) = 1  (B points to A)
  refcount(B) = 1  (A points to B)
  
  Now remove all external references:
  HEAD = NULL
  
  refcount(A) = 1  (B still points to A)
  refcount(B) = 1  (A still points to B)
  
  Neither reaches 0 → NEVER FREED! ← Memory leak!
```

```
  ┌───────────────────────────────┐
  │         Circular Leak         │
  │                               │
  │    ┌──[A]──┐                  │
  │    │   ↓   │                  │
  │    └──[B]──┘                  │
  │                               │
  │  No external references       │
  │  But refcounts are both 1     │
  │  → Objects never freed!       │
  └───────────────────────────────┘
```

### Languages Using Reference Counting

| Language | Primary | Cycle Handling |
|----------|---------|----------------|
| Python (CPython) | Reference counting | Cycle detector (generational GC) |
| Swift | ARC (Automatic RC) | `weak` / `unowned` references |
| Rust | Ownership (not RC) | `Rc<T>` for shared, `Weak<T>` for cycles |
| Objective-C | ARC | `weak` references |
| PHP | Reference counting | Cycle collector |

---

## 2.3 Tracing Garbage Collection

### How It Works

Periodically, the GC traverses all reachable objects from **root references** (stack variables, static fields). Anything not reachable is garbage.

```
  Root Set: [HEAD, temp, ...]
  
  Step 1: MARK — trace all reachable objects from roots
  Step 2: SWEEP — free all unmarked (unreachable) objects
  
  ROOT → [1] → [2] → [3]     ← All marked, kept
  
         [X] → [Y]            ← Not reachable from any root
                                  → SWEPT (freed)
```

### Mark-and-Sweep Diagram

```
  Before GC:
  ┌──────────┐
  │  ROOT    │
  │  HEAD ───┼──────► [1] ──► [2] ──► [3]
  │  temp ───┼──────► [5]
  └──────────┘
                      [4] ──► [6]          ← Unreachable!
  
  After MARK:
  [1]✓  [2]✓  [3]✓  [5]✓                 ← Marked
  [4]✗  [6]✗                              ← Unmarked
  
  After SWEEP:
  [4] and [6] are freed.
```

### Handles Circular References!

```
  [A] ⇄ [B]      (circular)
  
  No root points to A or B.
  
  Mark phase: A and B are NOT reachable from any root → unmarked.
  Sweep phase: Both freed. ✓
  
  Tracing GC solves the circular reference problem!
```

### GC Variants

| Type | How it Works | Pause Time | Used By |
|------|-------------|-----------|---------|
| Mark-Sweep | Mark reachable, sweep rest | Long pauses | — |
| Mark-Compact | Mark, then compact memory | Long pauses | — |
| Copying | Copy live objects to new space | Medium | — |
| Generational | Young/Old generations | Short pauses | Java, .NET, Python (cycle) |
| Concurrent | GC runs alongside program | Minimal pauses | Go, Java G1/ZGC |

---

## 2.4 C++ Smart Pointers (RAII)

RAII = **Resource Acquisition Is Initialization**. When an object goes out of scope, its destructor runs and frees resources.

### unique_ptr — Exclusive Ownership

```cpp
#include <memory>

struct Node {
    int data;
    std::unique_ptr<Node> next;
    
    Node(int val) : data(val), next(nullptr) {}
};

// Usage:
auto head = std::make_unique<Node>(1);
head->next = std::make_unique<Node>(2);
head->next->next = std::make_unique<Node>(3);

// When 'head' goes out of scope:
// 1. head's destructor runs → frees Node(1)
// 2. Node(1)'s unique_ptr next destructor → frees Node(2)
// 3. Node(2)'s unique_ptr next destructor → frees Node(3)
// ALL FREED AUTOMATICALLY! No leaks possible!
```

```
  unique_ptr ownership chain:
  
  head ──owns──► [1] ──owns──► [2] ──owns──► [3]
  
  Drop head → cascading destruction → all freed ✓
  
  Cannot copy a unique_ptr (compile error)
  Can move a unique_ptr (transfer ownership)
```

### shared_ptr — Shared Ownership (Reference Counting)

```cpp
struct Node {
    int data;
    std::shared_ptr<Node> next;
    
    Node(int val) : data(val), next(nullptr) {}
};

auto a = std::make_shared<Node>(1);  // refcount = 1
auto b = a;                           // refcount = 2
b = nullptr;                          // refcount = 1
a = nullptr;                          // refcount = 0 → freed
```

### weak_ptr — Breaking Circular References

```cpp
struct Node {
    int data;
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;          // ← Weak! Doesn't affect refcount
};

// In a DLL:
auto a = std::make_shared<Node>(1);
auto b = std::make_shared<Node>(2);
a->next = b;      // b's refcount: 2
b->prev = a;      // a's refcount: still 1! (weak doesn't count)

// When we drop 'a':
// a's refcount → 0 → freed
// b's refcount → 1 (only local variable 'b')
// When we drop 'b':
// b's refcount → 0 → freed
// NO CYCLE LEAK! ✓
```

### Smart Pointer Comparison

| Type | Ownership | Refcounted? | Can Copy? | Cycles? |
|------|-----------|-------------|-----------|---------|
| `unique_ptr` | Exclusive | No | No (move only) | N/A |
| `shared_ptr` | Shared | Yes | Yes | Can leak! |
| `weak_ptr` | Observer | No (doesn't own) | Yes | Breaks cycles |
| Raw pointer | None (manual) | No | Yes | N/A |

---

## 2.5 Rust's Ownership Model

```rust
struct Node {
    data: i32,
    next: Option<Box<Node>>,  // Box = unique ownership (like unique_ptr)
}

fn main() {
    let head = Box::new(Node {
        data: 1,
        next: Some(Box::new(Node {
            data: 2,
            next: None,
        })),
    });
    
    // 'head' goes out of scope → automatically dropped
    // No garbage collector needed!
    // The compiler GUARANTEES no leaks or dangling pointers.
}
```

```
  Rust prevents at COMPILE TIME:
  ✗ Use after free
  ✗ Double free
  ✗ Dangling pointers
  ✗ Data races
  
  For shared ownership: Rc<T> (single-thread) or Arc<T> (multi-thread)
  For cycle breaking: Weak<T>
```

---

## 2.6 GC Impact on Linked Lists

### Why GC and Linked Lists Don't Love Each Other

```
  Linked list: [1] → [2] → [3] → ... → [1000000]
  
  GC Mark Phase:
  Must traverse ALL 1,000,000 nodes just to mark them as alive.
  
  Problems:
  1. Long pause time (mark 1M nodes)
  2. Cache-unfriendly traversal (nodes scattered in heap)
  3. GC thread competes with application thread
  4. Old-generation promotion (long-lived list fills old gen)
```

### GC-Friendly Alternatives

```
  Instead of a linked list, consider:
  
  1. ArrayList / Vector — contiguous memory, GC-friendly
  2. ArrayDeque — circular buffer, much less GC pressure
  3. Pool allocator — allocate nodes from a pre-allocated block
  
  Use linked lists only when you truly need:
  - O(1) insert/delete at arbitrary points
  - No shifting of elements
  - Stable references (pointers don't invalidate)
```

---

## Summary Table

| Mechanism | Handles Cycles? | Pause Time | Languages |
|-----------|----------------|-----------|-----------|
| Manual (malloc/free) | N/A | None | C |
| Reference Counting | ❌ (needs weak refs) | None (immediate) | Python, Swift |
| Mark-Sweep GC | ✅ | Stop-the-world | Java, C# |
| Generational GC | ✅ | Short pauses | Java, .NET, Go |
| RAII / Smart Pointers | Depends on type | None | C++ |
| Ownership / Borrow | ✅ | None | Rust |

---

## Quick Revision Questions

1. **Why can't simple reference counting free circular linked lists?**
   <details><summary>Answer</summary>In a circular list, each node is referenced by its predecessor. Even when all external references are removed, each node's refcount remains ≥ 1 (from the cycle). Refcounts never reach 0, so nodes are never freed. This requires a separate cycle detector or tracing GC.</details>

2. **How does a tracing GC handle circular references that reference counting cannot?**
   <details><summary>Answer</summary>Tracing GC starts from root references and marks all reachable objects. Circular objects that aren't reachable from any root are simply never marked, so they get swept (freed) regardless of internal references.</details>

3. **What is RAII and how does it apply to linked lists in C++?**
   <details><summary>Answer</summary>RAII (Resource Acquisition Is Initialization) ties resource lifetime to object lifetime. Using `unique_ptr` for `next` pointers, when a node is destroyed, its destructor automatically destroys the next node, cascading through the entire list. No explicit `delete` needed.</details>

4. **Why use `weak_ptr` for `prev` in a C++ DLL instead of `shared_ptr`?**
   <details><summary>Answer</summary>`shared_ptr` for both `next` and `prev` creates reference cycles — each pair of adjacent nodes keeps the other alive. `weak_ptr` for `prev` doesn't increment the refcount, breaking the cycle and allowing proper deallocation when external references are dropped.</details>

5. **How does Rust prevent linked list memory bugs without a GC?**
   <details><summary>Answer</summary>Rust's ownership system ensures each value has exactly one owner. When the owner goes out of scope, the value is dropped. The borrow checker prevents dangling pointers at compile time. For shared ownership, `Rc<T>` with `Weak<T>` handles cycles.</details>

6. **Why are linked lists problematic for garbage collectors?**
   <details><summary>Answer</summary>During GC's mark phase, it must chase pointers through every node in the list. Nodes are scattered across the heap (not contiguous), causing cache misses. A million-node linked list means a million pointer dereferences during marking, causing long GC pauses.</details>

---

[← Previous: Memory Allocation and Leaks](01-memory-allocation-and-leaks.md) &nbsp;&nbsp;|&nbsp;&nbsp; [Next: Cache Performance and XOR List →](03-cache-performance-and-xor-list.md)

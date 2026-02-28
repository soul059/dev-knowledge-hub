# Chapter 2: Memory Representation

[← Previous: What is an Array?](01-what-is-an-array.md) | [Back to README](../README.md) | [Next: Array Operations →](03-array-operations.md)

---

## Overview

Understanding how arrays are laid out in memory is crucial for grasping why certain operations are fast (access) and others are slow (insertion/deletion). This chapter dives into the physical representation of arrays.

---

## Contiguous Memory Allocation

When you create an array, the operating system allocates a **single continuous block** of memory.

```
  Memory Addresses (example: 4 bytes per integer)

  Address:   1000    1004    1008    1012    1016
           ┌───────┬───────┬───────┬───────┬───────┐
  arr[5] = │  10   │  20   │  30   │  40   │  50   │
           └───────┴───────┴───────┴───────┴───────┘
             arr[0]  arr[1]  arr[2]  arr[3]  arr[4]

  Base Address = 1000
  Element Size = 4 bytes (int)
```

**Key insight:** There are no gaps between elements. This is what makes arrays different from linked lists.

---

## Address Calculation Formula

To find the memory address of any element:

$$\text{Address}(arr[i]) = \text{Base Address} + i \times \text{Size of Element}$$

### Step-by-Step Trace

```
Given: Base = 1000, Size = 4 bytes (int)

arr[0] → 1000 + 0 × 4 = 1000  ✓
arr[1] → 1000 + 1 × 4 = 1004  ✓
arr[2] → 1000 + 2 × 4 = 1008  ✓
arr[3] → 1000 + 3 × 4 = 1012  ✓
arr[4] → 1000 + 4 × 4 = 1016  ✓
```

This formula is why array access is **O(1)** — it's just arithmetic, no matter how large the array.

---

## Memory Layout for Different Data Types

```
  ┌─────────────────────────────────────────────────────────┐
  │          INT ARRAY (4 bytes each)                       │
  │  ┌──────┬──────┬──────┬──────┬──────┐                  │
  │  │ 4B   │ 4B   │ 4B   │ 4B   │ 4B   │  = 20 bytes     │
  │  └──────┴──────┴──────┴──────┴──────┘                  │
  ├─────────────────────────────────────────────────────────┤
  │          CHAR ARRAY (1 byte each)                       │
  │  ┌──┬──┬──┬──┬──┐                                      │
  │  │1B│1B│1B│1B│1B│                         = 5 bytes     │
  │  └──┴──┴──┴──┴──┘                                      │
  ├─────────────────────────────────────────────────────────┤
  │          DOUBLE ARRAY (8 bytes each)                    │
  │  ┌────────────┬────────────┬────────────┐               │
  │  │    8B      │    8B      │    8B      │ = 24 bytes    │
  │  └────────────┴────────────┴────────────┘               │
  └─────────────────────────────────────────────────────────┘
```

| Data Type | Typical Size | 100-element array |
|-----------|-------------|-------------------|
| char | 1 byte | 100 bytes |
| short | 2 bytes | 200 bytes |
| int | 4 bytes | 400 bytes |
| long | 8 bytes | 800 bytes |
| float | 4 bytes | 400 bytes |
| double | 8 bytes | 800 bytes |

---

## Stack vs Heap Allocation

Arrays can live in two places in memory:

```
  ┌─────────────────────────────────────────┐
  │              MEMORY LAYOUT              │
  ├─────────────────────────────────────────┤
  │                                         │
  │   ┌─────────────────────────────────┐   │
  │   │           STACK                  │   │
  │   │  (local variables, small arrays) │   │
  │   │                                  │   │
  │   │  int arr[5] = {1,2,3,4,5};      │   │
  │   │  ← grows downward               │   │
  │   └─────────────────────────────────┘   │
  │                                         │
  │              ... free space ...          │
  │                                         │
  │   ┌─────────────────────────────────┐   │
  │   │           HEAP                   │   │
  │   │  (dynamic arrays, large arrays)  │   │
  │   │                                  │   │
  │   │  int* arr = new int[1000];       │   │
  │   │  ← grows upward                 │   │
  │   └─────────────────────────────────┘   │
  │                                         │
  └─────────────────────────────────────────┘
```

| Aspect | Stack | Heap |
|--------|-------|------|
| Allocation speed | Very fast | Slower |
| Size limit | Small (1-8 MB typical) | Large (limited by RAM) |
| Management | Automatic (scope-based) | Manual (or GC) |
| Fragmentation | No | Possible |
| Use case | Small, temporary arrays | Large, persistent arrays |

---

## Cache and Memory Hierarchy

Arrays are **cache-friendly** because of their contiguous layout:

```
  ┌─────────────────────────────────────────────────┐
  │                CPU CACHE HIERARCHY               │
  │                                                  │
  │   CPU ──► L1 Cache ──► L2 Cache ──► L3 ──► RAM  │
  │           (fastest)                   (slowest)  │
  │           ~1 ns        ~4 ns    ~10ns   ~100 ns  │
  │                                                  │
  │   When you access arr[0], the cache loads a      │
  │   "cache line" (64 bytes typically), which        │
  │   includes arr[0]...arr[15] for int arrays.      │
  │                                                  │
  │   ┌───┬───┬───┬───┬───┬───┬───┬───┬─────────┐   │
  │   │a[0]│a[1]│a[2]│a[3]│a[4]│...│...│a[15]│ loaded│   │
  │   └───┴───┴───┴───┴───┴───┴───┴───┴─────────┘   │
  │   ◄──────── 64 bytes (one cache line) ────────►  │
  │                                                  │
  │   Next access to a[1], a[2], etc. → CACHE HIT!   │
  └─────────────────────────────────────────────────┘
```

**Array:** Sequential access → mostly cache hits → FAST

**Linked List:** Random memory → mostly cache misses → SLOW

```
  Array (contiguous):
  [a0][a1][a2][a3][a4][a5]...  → all together  ✓ cache friendly

  Linked List (scattered):
  [a0]──────►[a2]───►[a5]───►[a1]  → jumps around  ✗ cache hostile
       far         far        far
```

---

## Memory Representation of Strings

Strings are essentially **character arrays** with special handling:

### C-Style Strings (Null-Terminated)
```
  String: "Hello"

  ┌─────┬─────┬─────┬─────┬─────┬─────┐
  │ 'H' │ 'e' │ 'l' │ 'l' │ 'o' │ '\0'│
  └─────┴─────┴─────┴─────┴─────┴─────┘
    [0]   [1]   [2]   [3]   [4]   [5]

  Size in memory: 6 bytes (5 chars + null terminator)
```

### Java/Python Strings (Object with Length)
```
  ┌──────────────────────────────────┐
  │  String Object                    │
  │  ┌──────────────────────────────┐ │
  │  │ length: 5                    │ │
  │  │ data ──► ┌───┬───┬───┬───┬───┐│
  │  │          │ H │ e │ l │ l │ o ││
  │  │          └───┴───┴───┴───┴───┘│
  │  └──────────────────────────────┘ │
  └──────────────────────────────────┘
```

---

## Pointer Arithmetic and Arrays (C/C++)

In C, an array name is essentially a pointer to the first element:

```
  int arr[5] = {10, 20, 30, 40, 50};

  arr    → address of arr[0] = 1000
  arr+1  → address of arr[1] = 1004   (adds sizeof(int))
  arr+2  → address of arr[2] = 1008

  *arr     = arr[0] = 10
  *(arr+1) = arr[1] = 20
  *(arr+2) = arr[2] = 30
```

```
  Pointer arithmetic visualization:

  arr ──────────────────────────────────────►
  │                                          │
  ▼                                          │
  ┌──────┬──────┬──────┬──────┬──────┐       │
  │  10  │  20  │  30  │  40  │  50  │       │
  └──────┴──────┴──────┴──────┴──────┘
  ▲      ▲      ▲
  │      │      │
  arr   arr+1  arr+2
  *arr  *(arr+1) *(arr+2)
```

---

## Memory Wastage and Padding

Sometimes the compiler adds **padding bytes** for alignment:

```
  struct Record {
      char  grade;    // 1 byte
      // 3 bytes padding (for alignment)
      int   score;    // 4 bytes
      char  section;  // 1 byte
      // 3 bytes padding
  };

  Without padding: 1 + 4 + 1 = 6 bytes
  With padding:    4 + 4 + 4 = 12 bytes (aligned to 4-byte boundaries)

  ┌─────┬─────────────┬──────────────┬─────┬─────────────┐
  │grade│  padding(3) │    score     │ sec │  padding(3) │
  └─────┴─────────────┴──────────────┴─────┴─────────────┘
  │◄─── 4 bytes ─────►│◄── 4 bytes──►│◄─── 4 bytes ─────►│
```

This matters when you have an **array of structs** — padding is repeated per element.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Contiguous allocation | Elements stored side by side in memory |
| Address formula | `Base + i × ElementSize` gives O(1) access |
| Cache friendliness | Sequential data → cache line prefetch → fast |
| Stack allocation | Fast, small, automatic cleanup |
| Heap allocation | Slower, large, manual cleanup |
| Strings in memory | Char arrays (null-terminated in C, length-prefixed elsewhere) |
| Padding | Compilers may add bytes for alignment |

---

## Quick Revision Questions

1. **Calculate the address:** If `arr` starts at address 2000 and each element is 8 bytes, what is the address of `arr[7]`?

2. **Why is array access O(1)?** Explain using the address formula.

3. **What is a cache line?** How does it help with array traversal performance?

4. **Compare memory layout of arrays vs linked lists.** Why are arrays more cache-friendly?

5. **What is the null terminator in C strings?** Why is it needed?

6. **Stack vs Heap:** When would you allocate an array on the heap instead of the stack?

---

[← Previous: What is an Array?](01-what-is-an-array.md) | [Back to README](../README.md) | [Next: Array Operations →](03-array-operations.md)

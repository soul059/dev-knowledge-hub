# Chapter 4: Static vs Dynamic Arrays

[← Previous: Array Operations](03-array-operations.md) | [Back to README](../README.md) | [Next: Time Complexity →](05-time-complexity.md)

---

## Overview

Static arrays have a fixed size determined at compile time. Dynamic arrays can grow and shrink at runtime. Understanding the difference is key to writing efficient programs.

---

## Static Arrays

A static array has a **fixed capacity** that cannot change after creation.

```
// Declaration — size must be known at compile time
int arr[5];           // C/C++
int[] arr = new int[5];  // Java
```

```
  Static Array (capacity = 5):

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  __  │  __  │
  └──────┴──────┴──────┴──────┴──────┘
    [0]    [1]    [2]    [3]    [4]

  size = 3 (elements used)
  capacity = 5 (total slots)

  ✗ Cannot add a 6th element — no room!
```

### Properties of Static Arrays

| Property | Detail |
|----------|--------|
| Size | Fixed at creation |
| Memory | Allocated on stack (usually) |
| Speed | Fastest — no overhead |
| Flexibility | None — can't resize |
| Use case | When size is known in advance |

---

## Dynamic Arrays

A dynamic array **automatically resizes** when it runs out of space. This is what `ArrayList` (Java), `vector` (C++), and `list` (Python) use internally.

```
// Declaration
ArrayList<Integer> arr = new ArrayList<>();  // Java
vector<int> arr;                             // C++
arr = []                                     # Python
```

---

## How Dynamic Arrays Work — The Resizing Strategy

### The Problem
When a dynamic array is full and you want to add one more element, it must:
1. Allocate a **new, larger** block of memory
2. **Copy** all existing elements to the new block
3. **Free** the old block
4. Add the new element

### Doubling Strategy (Most Common)

```
  Step 1: Start with capacity 2
  ┌──────┬──────┐
  │  10  │  20  │   size=2, cap=2  → FULL!
  └──────┴──────┘

  Step 2: Insert 30 → Need to resize! Double to capacity 4.
  
  Old:  ┌──────┬──────┐
        │  10  │  20  │   (will be freed)
        └──────┴──────┘
              │ copy
              ▼
  New:  ┌──────┬──────┬──────┬──────┐
        │  10  │  20  │  30  │  __  │   size=3, cap=4
        └──────┴──────┴──────┴──────┘

  Step 3: Insert 40
  ┌──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │   size=4, cap=4 → FULL!
  └──────┴──────┴──────┴──────┘

  Step 4: Insert 50 → Resize! Double to capacity 8.
  
  Old:  ┌──────┬──────┬──────┬──────┐
        │  10  │  20  │  30  │  40  │   (freed)
        └──────┴──────┴──────┴──────┘
              │ copy
              ▼
  New:  ┌──────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┐
        │  10  │  20  │  30  │  40  │  50  │  __  │  __  │  __  │
        └──────┴──────┴──────┴──────┴──────┴──────┴──────┴──────┘
        size=5, cap=8
```

---

## Growth Factor Analysis

### Why Double? Why Not Add 1 Each Time?

**Strategy 1: Increase by 1 each time**
```
Insert 1st: copy 0 elements   → 0 work
Insert 2nd: copy 1 element    → 1 work
Insert 3rd: copy 2 elements   → 2 work
...
Insert nth: copy n-1 elements → n-1 work

Total: 0 + 1 + 2 + ... + (n-1) = n(n-1)/2 = O(n²)
Average per insert: O(n)  ← SLOW!
```

**Strategy 2: Double each time**
```
Resizes happen at: 1, 2, 4, 8, 16, ..., n

Copy work: 1 + 2 + 4 + 8 + ... + n = 2n - 1 ≈ 2n

For n insertions:
  n insertions × O(1) each + 2n copy work = 3n total

Average per insert: 3n / n = O(1) amortized  ← FAST!
```

### Amortized Analysis Visualization

```
  Insert #:  1  2  3  4  5  6  7  8  9  ...
  Cost:      1  2  3  1  5  1  1  1  9  ...
                ↑  ↑     ↑           ↑
                resize!  resize!    resize!
                (cap 2→4)(cap 4→8) (cap 8→16)

  Most inserts cost O(1), occasional resize costs O(n).
  Averaged out: O(1) per insert = "amortized O(1)"
```

---

## Common Growth Factors

| Language / Library | Growth Factor | Details |
|-------------------|---------------|---------|
| C++ `vector` | 2× | GCC implementation |
| Java `ArrayList` | 1.5× | `newCap = oldCap + oldCap >> 1` |
| Python `list` | ~1.125× | Over-allocates slightly |
| C# `List<T>` | 2× | .NET implementation |
| Go `slice` | 2× (small), ~1.25× (large) | Adaptive |

---

## Dynamic Array Pseudocode

```
CLASS DynamicArray:
    data[]          // internal static array
    size = 0        // number of elements
    capacity = 1    // current capacity

    FUNCTION append(value):
        IF size == capacity:
            resize(2 × capacity)    // double the capacity
        data[size] = value
        size = size + 1

    FUNCTION resize(newCapacity):
        newData = NEW ARRAY[newCapacity]
        FOR i = 0 TO size - 1:
            newData[i] = data[i]    // copy elements
        data = newData
        capacity = newCapacity

    FUNCTION get(index):
        IF index < 0 OR index >= size:
            ERROR "Index out of bounds"
        RETURN data[index]

    FUNCTION deleteAt(index):
        FOR i = index TO size - 2:
            data[i] = data[i + 1]
        size = size - 1
        // Optional: shrink if size < capacity/4
```

---

## Shrinking Strategy

Some implementations **shrink** the array when it becomes too empty:

```
  Rule: Shrink to half when size drops below 1/4 capacity

  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
  │10│20│__│__│__│__│__│__│__│__│__│__│__│__│__│__│  cap=16
  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
  size=2, which is < 16/4 = 4  → SHRINK!

  ┌──┬──┬──┬──┬──┬──┬──┬──┐
  │10│20│__│__│__│__│__│__│   cap=8
  └──┴──┴──┴──┴──┴──┴──┴──┘
```

**Why 1/4?** If we shrank at 1/2, alternating insert/delete could cause **thrashing** (resize, shrink, resize, shrink...).

---

## Static vs Dynamic Comparison

```
  STATIC ARRAY                    DYNAMIC ARRAY
  ┌──────────────────┐            ┌──────────────────┐
  │ Fixed size        │            │ Grows as needed    │
  │ No overhead       │            │ Resize overhead    │
  │ Stack allocated   │            │ Heap allocated     │
  │ Compile-time size │            │ Runtime size       │
  │ Fast, simple      │            │ Flexible, powerful │
  └──────────────────┘            └──────────────────┘
```

| Feature | Static Array | Dynamic Array |
|---------|-------------|---------------|
| Size | Fixed | Variable |
| Memory location | Stack (usually) | Heap |
| Resize | Not possible | Automatic |
| Insertion at end | O(1) if space | O(1) amortized |
| Memory overhead | None | Extra unused space |
| Cache performance | Excellent | Excellent |
| When to use | Size is known | Size is unknown |
| Examples | C arrays | ArrayList, vector, list |

---

## Memory Overhead of Dynamic Arrays

```
  capacity = 16, size = 9
  
  ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
  │##│##│##│##│##│##│##│##│##│  │  │  │  │  │  │  │
  └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
   ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲  ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲
         USED (9)                  WASTED (7)

  Worst case waste: just after doubling, ~50% is unused
  Best case waste: 0% (array is exactly full)
  Average waste: ~25-33%
```

---

## Summary Table

| Aspect | Static | Dynamic |
|--------|--------|---------|
| Declaration | `int arr[N]` | `vector<int>`, `ArrayList` |
| Resize | Impossible | Automatic (doubling) |
| Append | O(1) if room | O(1) amortized |
| Memory waste | None | Up to ~50% |
| Best for | Known-size data | Unknown-size data |
| Growth factor | N/A | Usually 1.5× – 2× |

---

## Quick Revision Questions

1. **What is amortized O(1)?** Explain why dynamic array append is amortized O(1) despite occasional O(n) resizes.

2. **Why double (2×) and not just add 10?** What happens to the total work if you grow by a constant?

3. **What is thrashing in the context of dynamic arrays?** Why do we shrink at 1/4 instead of 1/2?

4. **Calculate:** If you start with capacity 1 and insert 100 elements (doubling strategy), how many total copy operations occur?

5. **What's the maximum wasted memory** in a dynamic array with doubling right after a resize?

6. **Which languages use 1.5× vs 2× growth?** What are the trade-offs between these factors?

---

[← Previous: Array Operations](03-array-operations.md) | [Back to README](../README.md) | [Next: Time Complexity →](05-time-complexity.md)

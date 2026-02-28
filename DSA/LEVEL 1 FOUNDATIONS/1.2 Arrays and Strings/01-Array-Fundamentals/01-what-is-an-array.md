# Chapter 1: What is an Array?

[← Back to README](../README.md) | [Next: Memory Representation →](02-memory-representation.md)

---

## Overview

An array is the simplest and most widely used data structure. It stores a **fixed-size sequential collection** of elements of the **same type**. Understanding arrays deeply is the foundation for every other data structure and algorithm.

---

## Definition

> An **array** is a contiguous block of memory that stores elements of the same data type, accessible by an integer index.

Key properties:
- **Homogeneous** — all elements share the same type
- **Contiguous** — elements are stored in adjacent memory locations
- **Indexed** — each element has a numerical position (usually starting at 0)
- **Fixed size** — size is determined at creation (for static arrays)

---

## Conceptual Diagram

```
  Index:    0       1       2       3       4
         ┌───────┬───────┬───────┬───────┬───────┐
  arr =  │  10   │  20   │  30   │  40   │  50   │
         └───────┴───────┴───────┴───────┴───────┘
           ▲
           │
      Base Address (e.g., 0x1000)
```

Each box represents one **element**. The index tells us which position to look at.

---

## How Arrays Work — The Analogy

Think of an array like a **row of lockers** in a school hallway:

```
  Locker#:   0        1        2        3        4
         ┌────────┬────────┬────────┬────────┬────────┐
         │ Books  │ Shoes  │ Lunch  │ Phone  │ Keys   │
         └────────┴────────┴────────┴────────┴────────┘
```

- All lockers are the **same size** (same data type)
- They are **side by side** (contiguous memory)
- You access a locker by its **number** (index)
- The hallway has a **fixed number** of lockers (fixed size)

---

## Array Declaration (Pseudocode)

```
// Declare an array of size 5
DECLARE arr[5] OF INTEGER

// Initialize with values
arr = [10, 20, 30, 40, 50]

// Access element at index 2
value = arr[2]    // value = 30

// Modify element at index 3
arr[3] = 99       // arr = [10, 20, 30, 99, 50]
```

---

## Why Arrays are Important

### 1. O(1) Random Access
Given an index, you can jump directly to any element:
```
Address of arr[i] = Base_Address + i × Size_of_Element
```

### 2. Cache Friendly
Elements are stored next to each other in memory, so the CPU cache can load multiple elements at once — this makes sequential access very fast.

### 3. Foundation for Other Structures
Many data structures are built on top of arrays:

```
  ┌─────────────────────────────────────────────┐
  │                    ARRAY                     │
  ├──────────┬──────────┬───────────┬───────────┤
  │  Stack   │  Queue   │   Heap    │ Hash Table│
  │(arr-based)│(circular)│(arr-based)│ (buckets) │
  └──────────┴──────────┴───────────┴───────────┘
```

---

## Array Terminology

| Term | Meaning |
|------|---------|
| **Element** | A single value stored in the array |
| **Index** | The position of an element (starts at 0 in most languages) |
| **Size / Length** | Total number of elements the array can hold |
| **Base address** | Memory address of the first element |
| **Subscript** | Another word for index |
| **Bounds** | Valid index range: `[0, size - 1]` |
| **Out of bounds** | Accessing index < 0 or ≥ size (causes error) |

---

## 1D vs Multi-Dimensional Arrays

### 1D Array (Linear)
```
  ┌───┬───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │ 5 │
  └───┴───┴───┴───┴───┘
```

### 2D Array (Matrix / Table)
```
       Col 0  Col 1  Col 2
      ┌──────┬──────┬──────┐
Row 0 │  1   │  2   │  3   │
      ├──────┼──────┼──────┤
Row 1 │  4   │  5   │  6   │
      ├──────┼──────┼──────┤
Row 2 │  7   │  8   │  9   │
      └──────┴──────┴──────┘

Access: arr[1][2] = 6
```

### 3D Array (Cube — rare in practice)
```
  Depth 0:          Depth 1:
  ┌───┬───┐         ┌───┬───┐
  │ a │ b │         │ e │ f │
  ├───┼───┤         ├───┼───┤
  │ c │ d │         │ g │ h │
  └───┴───┘         └───┴───┘
```

---

## Zero-Based vs One-Based Indexing

| Language | Indexing |
|----------|----------|
| C / C++ | 0-based |
| Java | 0-based |
| Python | 0-based |
| JavaScript | 0-based |
| MATLAB | 1-based |
| Lua | 1-based |
| Fortran | 1-based (default) |

Why 0-based? Because the index represents the **offset** from the base address:
```
arr[0] → offset 0 → first element
arr[1] → offset 1 → second element
```

---

## Common Mistakes with Arrays

1. **Off-by-one errors** — Accessing `arr[n]` instead of `arr[n-1]` for the last element
2. **Uninitialized arrays** — Using an array before assigning values (garbage data)
3. **Fixed size overflow** — Trying to add more elements than the array can hold
4. **Negative indexing** — Not supported in most languages (Python is an exception)

---

## Real-World Applications

| Application | How Arrays are Used |
|-------------|-------------------|
| **Image Processing** | 2D array of pixel values (RGB) |
| **Spreadsheets** | Each row/column is an array |
| **Music/Audio** | Array of sound wave samples |
| **Databases** | Internal storage of records |
| **Games** | Game boards, inventories, maps |
| **Scientific Computing** | Vectors and matrices |

---

## Summary Table

| Property | Description |
|----------|-------------|
| Structure | Linear, contiguous memory |
| Element Type | Homogeneous (same type) |
| Access | O(1) by index |
| Size | Fixed at creation (static) |
| Indexing | Usually 0-based |
| Memory | Efficient, cache-friendly |
| Weakness | Fixed size, costly insertions |

---

## Quick Revision Questions

1. **What makes array access O(1)?** How does the formula `Base + i × Size` enable direct access?

2. **Why must all elements in an array be the same type?** What would happen if they weren't?

3. **What is an "out of bounds" error?** Give an example with an array of size 5.

4. **How is a 2D array different from a 1D array?** Draw a small example.

5. **Why do most languages use 0-based indexing?** What is the historical reason?

6. **Name three data structures that are built on top of arrays.**

---

[← Back to README](../README.md) | [Next: Memory Representation →](02-memory-representation.md)

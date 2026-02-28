# Chapter 6: Bubble Sort — Variations

[← Previous: When to Use](05-when-to-use.md) | [Next Unit: Selection Sort →](../03-Selection-Sort/01-algorithm-description.md)

---

## Overview

Several interesting variations of Bubble Sort exist, each addressing specific limitations or adapting the core idea for different scenarios.

---

## 1. Cocktail Shaker Sort (Bidirectional Bubble Sort)

Standard Bubble Sort only bubbles in **one direction** (left to right). Cocktail Shaker Sort alternates directions — left-to-right then right-to-left — reducing the "turtle problem."

### The Turtle Problem

```
  Array: [2, 3, 4, 5, 1]
  
  Standard Bubble Sort (left → right):
  Large values (rabbits) move RIGHT quickly: 5 reaches end in 1 pass
  Small values (turtles) move LEFT slowly:  1 moves left only 1 position per pass
  
  Pass 0: [2,3,4,1,5]    ← 1 moved 1 step left
  Pass 1: [2,3,1,4,5]    ← 1 moved 1 step left
  Pass 2: [2,1,3,4,5]    ← 1 moved 1 step left
  Pass 3: [1,2,3,4,5]    ← 1 finally home!  (4 passes!)
  
  The 🐢 "turtle" (small value near the end) is SLOW!
```

### How Cocktail Shaker Fixes It

```
  Array: [2, 3, 4, 5, 1]
  
  Pass 0 (left → right):  [2,3,4,1,5]    ← 5 bubbles right
  Pass 0 (right → left):  [1,2,3,4,5]    ← 1 bubbles left!   ✓ DONE!
  
  Only 1 pass (2 sweeps) instead of 4!
```

### Implementation

```python
def cocktail_shaker_sort(arr):
    n = len(arr)
    start = 0
    end = n - 1
    swapped = True
    
    while swapped:
        swapped = False
        
        # Left to right pass
        for i in range(start, end):
            if arr[i] > arr[i + 1]:
                arr[i], arr[i + 1] = arr[i + 1], arr[i]
                swapped = True
        
        if not swapped:
            break
        
        end -= 1      # largest is at the end
        swapped = False
        
        # Right to left pass
        for i in range(end - 1, start - 1, -1):
            if arr[i] > arr[i + 1]:
                arr[i], arr[i + 1] = arr[i + 1], arr[i]
                swapped = True
        
        start += 1    # smallest is at the start
```

### Trace

```
  Array: [5, 1, 4, 2, 8, 0, 3]
  
  ──► Left to Right:
  (5,1)swap (5,4)swap (5,2)swap (5,8)no (8,0)swap (8,3)swap
  → [1, 4, 2, 5, 0, 3, 8]                               ← 8 placed
  
  ◄── Right to Left:
  (0,3)no (5,0)swap (2,0)swap (4,0)swap (1,0)swap
  → [0, 1, 4, 2, 5, 3, 8]                               ← 0 placed
  
  ──► Left to Right:
  (1,4)no (4,2)swap (4,5)no (5,3)swap
  → [0, 1, 2, 4, 3, 5, 8]                               ← 5 placed
  
  ◄── Right to Left:
  (4,3)swap (2,3)no
  → [0, 1, 2, 3, 4, 5, 8]                               ← 3 placed ✓ DONE
```

---

## 2. Comb Sort

Comb Sort improves Bubble Sort by comparing elements with a **gap** larger than 1, gradually reducing the gap.

```
  Standard Bubble Sort: gap = 1 always
  Comb Sort: gap starts at n, shrinks by factor 1.3 each pass
  
  Array: [8, 4, 1, 3, 7, 5, 2, 6]    n=8
  
  Gap = 8/1.3 ≈ 6:
  Compare [8,2] [4,6] → [2,4,1,3,7,5,8,6]
  
  Gap = 6/1.3 ≈ 4:
  Compare positions 0&4, 1&5, 2&6, 3&7
  
  Gap = 4/1.3 ≈ 3:
  Compare positions 0&3, 1&4, 2&5, 3&6, 4&7
  
  ...continue until gap = 1 (then it's regular Bubble Sort)
```

### Implementation

```python
def comb_sort(arr):
    n = len(arr)
    gap = n
    shrink = 1.3
    sorted_flag = False
    
    while not sorted_flag:
        gap = int(gap / shrink)
        if gap <= 1:
            gap = 1
            sorted_flag = True  # will be set to False if swap occurs
        
        for i in range(n - gap):
            if arr[i] > arr[i + gap]:
                arr[i], arr[i + gap] = arr[i + gap], arr[i]
                sorted_flag = False
```

### Complexity
- **Average case**: O(n²/2ᵖ) where p = number of increments ≈ **O(n log n)**
- **Worst case**: O(n²) but extremely rare
- Much faster than Bubble Sort in practice

---

## 3. Odd-Even Sort (Brick Sort)

Alternates between comparing **odd-indexed** pairs and **even-indexed** pairs. Useful for **parallel processing**.

```
  Array: [5, 3, 4, 2, 1]
  
  Even phase (compare pairs at even indices):
  (5,3)swap  (4,2)swap  → [3,5,2,4,1]
   0,1        2,3
  
  Odd phase (compare pairs at odd indices):
  (5,2)swap  (4,1)swap  → [3,2,5,1,4]
   1,2        3,4
  
  Even phase:
  (3,2)swap  (5,1)swap  → [2,3,1,5,4]
  
  Odd phase:
  (3,1)swap  (5,4)swap  → [2,1,3,4,5]
  
  Even phase:
  (2,1)swap  (3,4)no    → [1,2,3,4,5]  ✓ DONE
```

### Implementation

```python
def odd_even_sort(arr):
    n = len(arr)
    sorted_flag = False
    
    while not sorted_flag:
        sorted_flag = True
        
        # Even phase: compare (0,1), (2,3), (4,5)...
        for i in range(0, n - 1, 2):
            if arr[i] > arr[i + 1]:
                arr[i], arr[i + 1] = arr[i + 1], arr[i]
                sorted_flag = False
        
        # Odd phase: compare (1,2), (3,4), (5,6)...
        for i in range(1, n - 1, 2):
            if arr[i] > arr[i + 1]:
                arr[i], arr[i + 1] = arr[i + 1], arr[i]
                sorted_flag = False
```

### Why It Matters

```
  ODD-EVEN SORT IS PARALLELIZABLE!
  
  Even phase:  (0,1) (2,3) (4,5) (6,7)  ← ALL can be done simultaneously!
  Odd phase:   (1,2) (3,4) (5,6) (7,8)  ← ALL can be done simultaneously!
  
  With n/2 processors: each phase takes O(1) time
  Total phases: O(n)
  Parallel time: O(n)
  
  Used in: GPU sorting, SIMD processing, hardware sorting networks
```

---

## 4. Recursive Bubble Sort

```python
def recursive_bubble_sort(arr, n=None):
    if n is None:
        n = len(arr)
    
    if n <= 1:
        return
    
    # One pass: bubble largest to end
    for i in range(n - 1):
        if arr[i] > arr[i + 1]:
            arr[i], arr[i + 1] = arr[i + 1], arr[i]
    
    # Recurse on smaller array (exclude last element)
    recursive_bubble_sort(arr, n - 1)
```

---

## Comparison of Variations

| Variation | Average | Key Advantage | Use Case |
|-----------|---------|---------------|----------|
| Standard | O(n²) | Simplicity | Learning |
| Cocktail Shaker | O(n²) | Handles turtles | Small/nearly sorted |
| Comb Sort | O(n log n)* | Gap-based improvement | Practical alternative |
| Odd-Even | O(n²) | Parallelizable | GPU/hardware sorting |
| Recursive | O(n²) | Demonstrates recursion | Educational |

*Comb Sort average case is debated; empirically near O(n log n)

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Cocktail Shaker | Bidirectional passes; fixes turtle problem |
| Comb Sort | Large gaps that shrink; near O(n log n) |
| Odd-Even Sort | Parallelizable; used in hardware/GPU |
| Turtle problem | Small values move slowly in standard Bubble Sort |
| Rabbit problem | Large values move quickly (already handled) |

---

## Quick Revision Questions

1. **What is the "turtle problem" in standard Bubble Sort?**
2. **How does Cocktail Shaker Sort solve the turtle problem?**
3. **Why is Comb Sort significantly faster than standard Bubble Sort?**
4. **Why is Odd-Even Sort useful for parallel processing?**
5. **What gap shrink factor does Comb Sort typically use?**
6. **Trace Cocktail Shaker Sort on [4, 1, 3, 2].**

---

[← Previous: When to Use](05-when-to-use.md) | [Next Unit: Selection Sort →](../03-Selection-Sort/01-algorithm-description.md)

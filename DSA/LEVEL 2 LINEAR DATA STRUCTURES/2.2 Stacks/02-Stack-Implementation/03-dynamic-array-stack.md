# Dynamic Array Stack Implementation

## Overview

A **dynamic array stack** combines the best features of array-based and linked list-based implementations. It uses a resizable array that automatically grows when full and optionally shrinks when mostly empty, providing both the performance of arrays and the flexibility of dynamic sizing.

---

## Core Concept

```
Dynamic Array Stack:

Initial State (capacity = 4):
      top = 2
          ↓
    ┌───┬───┬───┬───┐
    │10 │20 │30 │   │
    └───┴───┴───┴───┘
    size = 3, capacity = 4

After Full + Push (resize to 8):
      top = 4
              ↓
    ┌───┬───┬───┬───┬───┬───┬───┬───┐
    │10 │20 │30 │40 │50 │   │   │   │
    └───┴───┴───┴───┴───┴───┴───┴───┘
    size = 5, capacity = 8
    (automatically doubled!)

Key Features:
• Starts with small capacity
• Grows when full (typically doubles)
• Shrinks when mostly empty (typically halves at 25% full)
• Amortized O(1) operations
```

---

## Data Structure Design

### Complete Structure

```
Dynamic Stack Structure:
┌──────────────────────┐
│ array: pointer       │ ← Points to dynamic array
│ top: integer         │ ← Index of top element
│ size: integer        │ ← Current number of elements
│ capacity: integer    │ ← Current array capacity
└──────────────────────┘

Visual:
Stack Object:          Array in Memory:
┌──────────────┐      ┌───┬───┬───┬───┬───┬───┬───┬───┐
│ top = 3      │      │10 │20 │30 │40 │   │   │   │   │
│ size = 4     │  →   └───┴───┴───┴───┴───┴───┴───┴───┘
│ capacity = 8 │       0   1   2   3   4   5   6   7
│ array: ptr   │
└──────────────┘
```

---

## Resizing Strategy

### Growth Strategy

```
Common Strategies:

1. Doubling (Most Common):
   • New capacity = 2 × old capacity
   • Minimizes number of resizes
   • Wastes at most 50% space

2. Factor of 1.5:
   • New capacity = 1.5 × old capacity
   • More gradual growth
   • Better memory utilization

3. Fixed Increment:
   • New capacity = old capacity + constant
   • Predictable growth
   • More frequent resizes

Growth Comparison:
Start: 4

Doubling:    4 → 8 → 16 → 32 → 64 → 128
Factor 1.5:  4 → 6 → 9 → 13 → 19 → 28
Increment +4: 4 → 8 → 12 → 16 → 20 → 24
```

### Shrinking Strategy

```
Shrinkage Trigger (typical):
• When size ≤ capacity / 4
• Halve the capacity
• Prevents thrashing

Without Shrinking:
Push 1000 elements → capacity = 1024
Pop 999 elements → capacity still 1024
(wasteful!)

With Shrinking:
Push 1000 elements → capacity = 1024
Pop to 256 elements → capacity = 512
Pop to 128 elements → capacity = 256
```

---

## Operations Implementation

### 1. Initialize

```
INITIALIZE(initial_capacity = 4):
    capacity = initial_capacity
    array = new Array[capacity]
    top = -1
    size = 0

Visual:
┌──────────────┐
│ top = -1     │
│ size = 0     │
│ capacity = 4 │
└──────────────┘
       ↓
┌───┬───┬───┬───┐
│   │   │   │   │
└───┴───┴───┴───┘

Time: O(1)
Space: O(initial_capacity)
```

### 2. Resize Operation (Helper)

```
RESIZE(new_capacity):
    // Create new array
    new_array = new Array[new_capacity]
    
    // Copy elements
    FOR i = 0 TO top:
        new_array[i] = array[i]
    END FOR
    
    // Delete old array
    DELETE array
    
    // Update reference and capacity
    array = new_array
    capacity = new_capacity

Visual Trace: Resize from 4 to 8

Old Array:               New Array:
top = 3                  top = 3
  ↓                        ↓
┌───┬───┬───┬───┐       ┌───┬───┬───┬───┬───┬───┬───┬───┐
│10 │20 │30 │40 │  →    │10 │20 │30 │40 │   │   │   │   │
└───┴───┴───┴───┘       └───┴───┴───┴───┴───┴───┴───┴───┘
capacity = 4            capacity = 8

Time: O(n) where n = number of elements
Space: O(new_capacity)
```

### 3. Push Operation

```
PUSH(item):
    // Check if resize needed
    IF top = capacity - 1 THEN
        RESIZE(capacity × 2)  // Double capacity
    END IF
    
    // Standard push
    top = top + 1
    array[top] = item
    size = size + 1

Detailed Trace: Push when full

Before: (capacity = 4, all full)
      top = 3
          ↓
    ┌───┬───┬───┬───┐
    │10 │20 │30 │40 │ ← Full!
    └───┴───┴───┴───┘

Step 1: Detect full
IF top (3) = capacity - 1 (3): TRUE

Step 2: Resize to 8
         top = 3
             ↓
    ┌───┬───┬───┬───┬───┬───┬───┬───┐
    │10 │20 │30 │40 │   │   │   │   │
    └───┴───┴───┴───┴───┴───┴───┴───┘

Step 3: Push new element (50)
             top = 4
                  ↓
    ┌───┬───┬───┬───┬───┬───┬───┬───┐
    │10 │20 │30 │40 │50 │   │   │   │
    └───┴───┴───┴───┴───┴───┴───┴───┘

Amortized Time: O(1) - explained below
Worst Case Time: O(n) - when resizing
```

### 4. Pop Operation

```
POP():
    // Check underflow
    IF top = -1 THEN
        ERROR "Stack Underflow"
    END IF
    
    // Standard pop
    item = array[top]
    top = top - 1
    size = size - 1
    
    // Check if shrink needed
    IF size > 0 AND size ≤ capacity / 4 THEN
        RESIZE(capacity / 2)
    END IF
    
    RETURN item

Detailed Trace: Pop with shrinking

Before: (capacity = 8, size = 2)
      top = 1
          ↓
    ┌───┬───┬───┬───┬───┬───┬───┬───┐
    │10 │20 │   │   │   │   │   │   │
    └───┴───┴───┴───┴───┴───┴───┴───┘
    size = 2, capacity = 8

Step 1: Pop element
Returns: 20
top = 0, size = 1

Step 2: Check shrink condition
size (1) ≤ capacity / 4 (2): TRUE

Step 3: Shrink to capacity 4
      top = 0
          ↓
    ┌───┬───┬───┬───┐
    │10 │   │   │   │
    └───┴───┴───┴───┘
    size = 1, capacity = 4

Amortized Time: O(1)
Worst Case Time: O(n) - when resizing
```

---

## Amortized Analysis

### Why O(1) Amortized?

```
Amortized Analysis Explanation:

Consider n pushes starting from empty stack:

Push 1:      Cost = 1 (no resize)
Push 2:      Cost = 1 (no resize)
Push 3:      Cost = 3 (resize + copy 2 + insert)
Push 4:      Cost = 1
Push 5:      Cost = 5 (resize + copy 4 + insert)
Push 6:      Cost = 1
Push 7:      Cost = 1
Push 8:      Cost = 1
Push 9:      Cost = 9 (resize + copy 8 + insert)
...

Pattern:
Resize at positions: 3, 5, 9, 17, 33...
                     (when doubled from 2,4,8,16,32...)

Total Cost for n operations:
Copy costs: 2 + 4 + 8 + 16 + ... + n/2 ≈ n
Insert costs: n
Total: ~2n = O(n)

Average per operation: O(n) / n = O(1)

Amortized O(1) per operation!
```

### Visual Amortization

```
Operation Sequence (capacity starts at 2):

Op  | Action | Array State              | Actual Cost | Amortized
----|--------|--------------------------|-------------|----------
1   | Push   | [10, _]                  | 1           | 3
2   | Push   | [10, 20]                 | 1           | 3
3   | Push   | [10, 20, 30, _]          | 3 (resize)  | 3
4   | Push   | [10, 20, 30, 40]         | 1           | 3
5   | Push   | [10,20,30,40,50,_,_,_]   | 5 (resize)  | 3
6   | Push   | [10,20,30,40,50,60,_,_]  | 1           | 3
...

Each operation "pays" 3 credits:
• 1 for itself
• 2 saved for future resize

When resize needed, saved credits cover it!
```

---

## Complete Implementation

### Pseudocode

```pseudocode
CLASS DynamicArrayStack:
    PRIVATE:
        array: pointer to array
        top: integer
        size: integer
        capacity: integer
        
    PRIVATE RESIZE(new_capacity):
        new_array = new Array[new_capacity]
        FOR i = 0 TO top:
            new_array[i] = array[i]
        END FOR
        DELETE array
        array = new_array
        capacity = new_capacity

    PUBLIC DynamicArrayStack(initial_cap = 4):
        capacity = initial_cap
        array = new Array[capacity]
        top = -1
        size = 0

    PUBLIC PUSH(item):
        // Resize if needed (grow)
        IF top = capacity - 1 THEN
            RESIZE(capacity * 2)
        END IF
        
        top = top + 1
        array[top] = item
        size = size + 1

    PUBLIC POP():
        IF isEmpty() THEN
            ERROR "Stack Underflow"
        END IF
        
        item = array[top]
        top = top - 1
        size = size - 1
        
        // Resize if needed (shrink)
        IF size > 0 AND size <= capacity / 4 THEN
            RESIZE(capacity / 2)
        END IF
        
        RETURN item

    PUBLIC PEEK():
        IF isEmpty() THEN
            ERROR "Stack Empty"
        END IF
        RETURN array[top]

    PUBLIC isEmpty():
        RETURN (top = -1)

    PUBLIC SIZE():
        RETURN size

    PUBLIC GET_CAPACITY():
        RETURN capacity

    PUBLIC DESTRUCTOR():
        DELETE array
END CLASS
```

---

## Advantages of Dynamic Array Stack

```
✅ Advantages:

1. No Size Limit
   ✓ Grows as needed
   ✓ No overflow (until system memory full)
   ✓ Flexible like linked list

2. Cache Friendly
   ✓ Contiguous memory (like array)
   ✓ Good cache locality
   ✓ Fast access

3. Memory Efficient
   ✓ No pointer overhead per element
   ✓ Shrinks when underutilized
   ✓ Better than pure array

4. Amortized O(1)
   ✓ All operations O(1) amortized
   ✓ Predictable performance
   ✓ Rare resizing amortized away

5. Best of Both Worlds
   ✓ Array performance
   ✓ Dynamic sizing
   ✓ Balanced approach
```

---

## Disadvantages of Dynamic Array Stack

```
❌ Disadvantages:

1. Occasional O(n) Operation
   × Resize operations take O(n)
   × Unpredictable worst-case
   × Not suitable for real-time systems

2. Memory Waste (Temporary)
   × Can waste up to 50% space
   × Between resizes, excess capacity
   × Double buffering during resize

3. Implementation Complexity
   × More complex than fixed array
   × Resize logic needed
   × Shrinking strategy needed

4. Memory Allocation Overhead
   × Frequent allocations/deallocations
   × OS memory manager overhead
   × Possible fragmentation

Visual Memory Waste:
After many pushes then pops to 25%:
      top = 1
          ↓
┌───┬───┬───┬───┬───┬───┬───┬───┐
│10 │20 │   │   │   │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┘
← used →      ← 75% wasted →
```

---

## Resizing Policies Comparison

### Growth Factor Analysis

```
┌─────────────┬──────────┬────────────┬──────────────┐
│   Factor    │ Resizes  │ Max Waste  │ Performance  │
├─────────────┼──────────┼────────────┼──────────────┤
│ 2x (Double) │ log₂(n)  │ ~50%       │ Best         │
│ 1.5x        │ log₁.₅(n)│ ~33%       │ Good         │
│ +constant   │ n/c      │ Min        │ Poor         │
└─────────────┴──────────┴────────────┴──────────────┘

For 1000 elements:
Doubling:     10 resizes, max waste ~500 elements
Factor 1.5:   ~18 resizes, max waste ~333 elements
+100:         10 resizes, max waste ~100 elements

Trade-off: Fewer resizes vs. less waste
```

### Shrinking Threshold Analysis

```
Shrink at 50% full:
Problem: Thrashing!

Example:
capacity = 8, size = 4
Pop → size = 3 (< 4), shrink to 4
Push → size = 4, grow to 8
Pop → size = 3, shrink to 4
(Constant resizing!)

Shrink at 25% full:
Solution: Prevents thrashing

capacity = 8, size = 4 (Pop)
    → Don't shrink (4 > 2)
capacity = 8, size = 3 (Pop)
    → Don't shrink (3 > 2)
capacity = 8, size = 2 (Pop)
    → Shrink to 4 (2 ≤ 2)

No thrashing until size varies significantly!
```

---

## Performance Comparison

### Time Complexity

| Operation | Worst Case | Amortized | Best Case |
|-----------|-----------|-----------|-----------|
| **Push** | O(n) (resize) | O(1) | O(1) |
| **Pop** | O(n) (resize) | O(1) | O(1) |
| **Peek** | O(1) | O(1) | O(1) |
| **isEmpty** | O(1) | O(1) | O(1) |
| **Size** | O(1) | O(1) | O(1) |

### Space Complexity

```
Space Usage Over Time:

Perfect Usage:
size = capacity
No waste

Typical Usage After Growth:
size = capacity / 2 to capacity
25-50% potential waste

After Shrinking Trigger:
size ≈ capacity / 4
Shrinks to capacity / 2
New ratio: size ≈ capacity / 2
(Back to reasonable)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Storage** | Dynamically resized array |
| **Initial Size** | Small (e.g., 4 or 8) |
| **Growth** | Typically doubles (2x) |
| **Shrinkage** | Typically halves when 25% full |
| **Time (Amortized)** | O(1) for all operations |
| **Time (Worst)** | O(n) for push/pop (resize) |
| **Space** | O(n), with up to 50% overhead |
| **Cache Performance** | Excellent (contiguous) |
| **Best For** | General purpose, unknown size |
| **Major Advantage** | Best of array and dynamic sizing |
| **Major Disadvantage** | Occasional O(n) resize |

---

## Quick Revision Questions

1. **Q: Why use doubling strategy instead of adding a fixed amount?**
   - A: Doubling results in O(log n) resizes for n elements vs. O(n) resizes with fixed increment, making push amortized O(1).

2. **Q: Why shrink at 25% full instead of 50%?**
   - A: To prevent thrashing - rapidly alternating between shrinking and growing when size hovers around 50%.

3. **Q: What is the "cost" of a resize operation?**
   - A: O(n) where n is the number of elements to copy to the new array.

4. **Q: How does dynamic array achieve amortized O(1) push despite O(n) resizing?**
   - A: Resizes are infrequent (exponentially spaced). The total cost of all resizes for n operations is O(n), averaging O(1) per operation.

5. **Q: What happens during resize when memory allocation fails?**
   - A: Depends on implementation - could throw exception, return error, or terminate. Original array remains intact until new array is successfully allocated.

6. **Q: Why is dynamic array stack generally preferred over other implementations?**
   - A: It combines cache-friendly contiguous storage (array) with dynamic sizing (linked list) and amortized O(1) operations, making it versatile and efficient for most use cases.

---

## Navigation

- **[← Previous: Linked List-Based Stack](02-linked-list-stack.md)**
- **[Next: Comparison of Approaches →](04-comparison-approaches.md)**
- **[↑ Back to Unit 2](README.md)**
- **[⌂ Home](../README.md)**

---

*Dynamic arrays offer the best balance for most applications - understand the amortization to appreciate their efficiency!*

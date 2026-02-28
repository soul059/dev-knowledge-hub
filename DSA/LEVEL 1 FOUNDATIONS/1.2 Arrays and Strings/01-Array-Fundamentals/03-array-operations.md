# Chapter 3: Array Operations (Access, Insert, Delete)

[← Previous: Memory Representation](02-memory-representation.md) | [Back to README](../README.md) | [Next: Static vs Dynamic Arrays →](04-static-vs-dynamic-arrays.md)

---

## Overview

The three fundamental array operations are **accessing** (read/write), **inserting** (adding elements), and **deleting** (removing elements). Understanding the cost of each operation is essential for choosing the right data structure.

---

## 1. Access (Read / Write)

### How It Works
Access is the fastest operation — it uses the address formula to jump directly to any index.

```
FUNCTION access(arr, index):
    RETURN arr[index]     // O(1) — direct memory lookup
```

### Visualization

```
  Access arr[3]:

  Step 1: Calculate address = Base + 3 × 4 = 1000 + 12 = 1012
  Step 2: Read value at address 1012

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
    [0]    [1]    [2]    [3]    [4]
                          ▲
                          │
                    Direct jump!
                    Returns 40
```

| Operation | Time | Explanation |
|-----------|------|-------------|
| Read `arr[i]` | O(1) | Address arithmetic |
| Write `arr[i] = x` | O(1) | Address arithmetic + store |

---

## 2. Insertion

Insertion varies drastically depending on **where** you insert.

### Case A: Insert at the End

```
FUNCTION insertAtEnd(arr, size, value):
    arr[size] = value
    size = size + 1
```

```
  Before: [10, 20, 30, __, __]   size = 3, capacity = 5
                          ▲
                          │ insert 40 here
  After:  [10, 20, 30, 40, __]   size = 4

  Cost: O(1) — no shifting needed
```

### Case B: Insert at the Beginning

```
FUNCTION insertAtBeginning(arr, size, value):
    FOR i = size-1 DOWN TO 0:
        arr[i+1] = arr[i]       // shift right
    arr[0] = value
    size = size + 1
```

### Step-by-Step Trace: Insert 5 at index 0

```
  Original: [10, 20, 30, 40, __]   size = 4

  Step 1: Shift arr[3] → arr[4]
  [10, 20, 30, 40, 40]
                   └──►

  Step 2: Shift arr[2] → arr[3]
  [10, 20, 30, 30, 40]
              └──►

  Step 3: Shift arr[1] → arr[2]
  [10, 20, 20, 30, 40]
          └──►

  Step 4: Shift arr[0] → arr[1]
  [10, 10, 20, 30, 40]
      └──►

  Step 5: Place value at arr[0]
  [ 5, 10, 20, 30, 40]
    ▲
    └── inserted!

  Cost: O(n) — shifted all n elements
```

### Case C: Insert at Index k

```
FUNCTION insertAt(arr, size, index, value):
    FOR i = size-1 DOWN TO index:
        arr[i+1] = arr[i]       // shift elements right
    arr[index] = value
    size = size + 1
```

### Trace: Insert 25 at index 2

```
  Original: [10, 20, 30, 40, __]   size = 4

  Step 1: Shift arr[3] → arr[4]    [10, 20, 30, 40, 40]
  Step 2: Shift arr[2] → arr[3]    [10, 20, 30, 30, 40]
  Step 3: Place 25 at arr[2]       [10, 20, 25, 30, 40]

  Elements shifted: n - index = 4 - 2 = 2
  Cost: O(n - k), worst case O(n)
```

---

## 3. Deletion

### Case A: Delete from the End

```
FUNCTION deleteAtEnd(arr, size):
    size = size - 1     // just reduce the size
    // arr[size] still exists in memory but is "logically" removed
```

```
  Before: [10, 20, 30, 40, 50]   size = 5
  After:  [10, 20, 30, 40, __]   size = 4
                           ▲
                           └── logically removed

  Cost: O(1) — no shifting
```

### Case B: Delete from the Beginning

```
FUNCTION deleteAtBeginning(arr, size):
    FOR i = 0 TO size-2:
        arr[i] = arr[i+1]       // shift left
    size = size - 1
```

### Trace: Delete arr[0]

```
  Original: [10, 20, 30, 40, 50]   size = 5

  Step 1: arr[0] = arr[1]    [20, 20, 30, 40, 50]
                               ◄──┘

  Step 2: arr[1] = arr[2]    [20, 30, 30, 40, 50]
                                   ◄──┘

  Step 3: arr[2] = arr[3]    [20, 30, 40, 40, 50]
                                       ◄──┘

  Step 4: arr[3] = arr[4]    [20, 30, 40, 50, 50]
                                           ◄──┘

  Final:  [20, 30, 40, 50, __]   size = 4

  Cost: O(n) — shifted all remaining elements
```

### Case C: Delete at Index k

```
FUNCTION deleteAt(arr, size, index):
    FOR i = index TO size-2:
        arr[i] = arr[i+1]       // shift left
    size = size - 1
```

### Trace: Delete at index 1

```
  Original: [10, 20, 30, 40, 50]   size = 5

  Step 1: arr[1] = arr[2]    [10, 30, 30, 40, 50]
  Step 2: arr[2] = arr[3]    [10, 30, 40, 40, 50]
  Step 3: arr[3] = arr[4]    [10, 30, 40, 50, 50]

  Final:  [10, 30, 40, 50, __]   size = 4

  Elements shifted: n - k - 1 = 5 - 1 - 1 = 3
  Cost: O(n - k), worst case O(n)
```

---

## Delete Without Maintaining Order — O(1) Trick

If element **order doesn't matter**, you can delete in O(1):

```
FUNCTION deleteUnordered(arr, size, index):
    arr[index] = arr[size - 1]   // replace with last element
    size = size - 1
```

```
  Delete index 1 (unordered):

  Original: [10, 20, 30, 40, 50]   size = 5

  Step 1: arr[1] = arr[4]    [10, 50, 30, 40, 50]
  Step 2: size = 4            [10, 50, 30, 40, __]

  Cost: O(1) — but order is changed!
```

---

## Complete Operation Complexity Table

```
  ┌─────────────────────────┬────────────┬────────────┐
  │      OPERATION          │  BEST CASE │ WORST CASE │
  ├─────────────────────────┼────────────┼────────────┤
  │  Access (read/write)    │    O(1)    │    O(1)    │
  ├─────────────────────────┼────────────┼────────────┤
  │  Insert at end          │    O(1)    │    O(1)    │
  │  Insert at beginning    │    O(n)    │    O(n)    │
  │  Insert at index k      │    O(1)    │    O(n)    │
  ├─────────────────────────┼────────────┼────────────┤
  │  Delete at end          │    O(1)    │    O(1)    │
  │  Delete at beginning    │    O(n)    │    O(n)    │
  │  Delete at index k      │    O(1)    │    O(n)    │
  ├─────────────────────────┼────────────┼────────────┤
  │  Search (unsorted)      │    O(1)    │    O(n)    │
  │  Search (sorted)        │    O(1)    │   O(log n) │
  └─────────────────────────┴────────────┴────────────┘
```

---

## Decision Flowchart

```
  Need to perform operation on array?
              │
              ▼
  ┌─────────────────────────┐
  │  What operation?         │
  └────────┬────────────────┘
           │
     ┌─────┼──────────┐
     ▼     ▼          ▼
  Access  Insert    Delete
   O(1)    │          │
           ▼          ▼
       Where?      Where?
      ┌──┼──┐     ┌──┼──┐
      ▼  ▼  ▼     ▼  ▼  ▼
    End Mid Beg  End Mid Beg
    O(1)O(n)O(n) O(1)O(n)O(n)
```

---

## Summary Table

| Operation | Where | Time | Space |
|-----------|-------|------|-------|
| Access | Any index | O(1) | O(1) |
| Insert | End | O(1) | O(1) |
| Insert | Beginning / Middle | O(n) | O(1) |
| Delete | End | O(1) | O(1) |
| Delete | Beginning / Middle | O(n) | O(1) |
| Delete (unordered) | Any | O(1) | O(1) |

---

## Quick Revision Questions

1. **Why is insertion at the beginning O(n)?** What exactly happens in memory?

2. **Describe the "swap with last" deletion trick.** When can and can't you use it?

3. **Calculate:** In an array of 1000 elements, how many shifts are needed to insert at index 300?

4. **What's the average case for insertion at a random position?** Express in terms of n.

5. **Can you insert into a full static array?** What happens if you try?

6. **Why is deletion at the end O(1)?** What happens to the "deleted" element in memory?

---

[← Previous: Memory Representation](02-memory-representation.md) | [Back to README](../README.md) | [Next: Static vs Dynamic Arrays →](04-static-vs-dynamic-arrays.md)

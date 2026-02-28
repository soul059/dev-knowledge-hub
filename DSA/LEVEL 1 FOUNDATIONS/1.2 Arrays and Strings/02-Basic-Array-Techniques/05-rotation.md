# Chapter 5: Rotation (Left, Right)

[← Previous: Reversing an Array](04-reversing-an-array.md) | [Back to README](../README.md) | [Next: Array Manipulation →](06-array-manipulation.md)

---

## Overview

Array rotation shifts all elements by a given number of positions. This is a classic interview problem with multiple approaches, from brute force to elegant O(n) solutions.

---

## What is Rotation?

### Left Rotation by d positions
Every element moves d positions to the left. Elements that "fall off" the left end wrap around to the right.

```
  Left rotate [1, 2, 3, 4, 5] by d = 2:

  Before: [1, 2, 3, 4, 5]
           ──►──►
  After:  [3, 4, 5, 1, 2]

  Visualization:
  ┌───┬───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │ 5 │     original
  └───┴───┴───┴───┴───┘
    ↓   ↓   ↓   ↓   ↓
  ┌───┬───┬───┬───┬───┐
  │ 3 │ 4 │ 5 │ 1 │ 2 │     left rotated by 2
  └───┴───┴───┴───┴───┘
```

### Right Rotation by d positions
Every element moves d positions to the right. Right rotation by d = left rotation by (n - d).

```
  Right rotate [1, 2, 3, 4, 5] by d = 2:

  Before: [1, 2, 3, 4, 5]
                     ◄──◄──
  After:  [4, 5, 1, 2, 3]
```

---

## Approach 1: Brute Force — Rotate One at a Time

```
FUNCTION leftRotateByOne(arr, n):
    temp = arr[0]
    FOR i = 0 TO n-2:
        arr[i] = arr[i+1]
    arr[n-1] = temp

FUNCTION leftRotate(arr, n, d):
    d = d % n       // handle d > n
    FOR count = 1 TO d:
        leftRotateByOne(arr, n)
```

### Trace: Left rotate [1,2,3,4,5] by 2

```
  Rotation 1:
  temp = 1
  [2, 3, 4, 5, 5] → shift left
  [2, 3, 4, 5, 1] → place temp at end

  Rotation 2:
  temp = 2
  [3, 4, 5, 1, 1] → shift left
  [3, 4, 5, 1, 2] → place temp at end

  Result: [3, 4, 5, 1, 2] ✓
```

**Time: O(n × d)** | **Space: O(1)**

---

## Approach 2: Using Extra Array

```
FUNCTION leftRotate(arr, n, d):
    d = d % n
    temp = NEW ARRAY[n]
    FOR i = 0 TO n-1:
        temp[i] = arr[(i + d) % n]
    COPY temp TO arr
```

### Trace: Left rotate [1,2,3,4,5] by 2

```
  d = 2

  temp[0] = arr[(0+2) % 5] = arr[2] = 3
  temp[1] = arr[(1+2) % 5] = arr[3] = 4
  temp[2] = arr[(2+2) % 5] = arr[4] = 5
  temp[3] = arr[(3+2) % 5] = arr[0] = 1
  temp[4] = arr[(4+2) % 5] = arr[1] = 2

  Result: [3, 4, 5, 1, 2] ✓
```

**Time: O(n)** | **Space: O(n)**

---

## Approach 3: Reversal Algorithm — Optimal

The most elegant approach. Uses three reversals:

```
FUNCTION leftRotate(arr, n, d):
    d = d % n
    reverse(arr, 0, d-1)        // reverse first d elements
    reverse(arr, d, n-1)        // reverse remaining elements
    reverse(arr, 0, n-1)        // reverse the whole array
```

### Step-by-Step Trace: Left rotate [1,2,3,4,5] by 2

```
  Original:     [1, 2, 3, 4, 5]
                 ├──┤  ├──────┤
                 d=2    rest

  Step 1: Reverse first d=2 elements (indices 0 to 1)
                [2, 1, 3, 4, 5]
                 ▲  ▲
                reversed

  Step 2: Reverse remaining n-d=3 elements (indices 2 to 4)
                [2, 1, 5, 4, 3]
                       ▲  ▲  ▲
                      reversed

  Step 3: Reverse entire array (indices 0 to 4)
                [3, 4, 5, 1, 2]
                 ▲  ▲  ▲  ▲  ▲
                all reversed

  Result: [3, 4, 5, 1, 2] ✓
```

### Why does this work?

```
  Think of the array as two parts: [A | B]
  where A = first d elements, B = rest

  Original:    [A  |  B]      = [1,2 | 3,4,5]
  
  Step 1:      [A' |  B]      = [2,1 | 3,4,5]   (reverse A)
  Step 2:      [A' |  B']     = [2,1 | 5,4,3]   (reverse B)
  Step 3:      [B  |  A]      = [3,4,5 | 1,2]   (reverse all)

  Proof: reversing [A' | B'] gives [B  | A]
  because reverse(reverse(X)) = X
```

**Time: O(n)** | **Space: O(1)** | **Optimal!**

---

## Approach 4: Juggling Algorithm

Uses GCD to rotate elements in cycles:

```
FUNCTION leftRotate(arr, n, d):
    d = d % n
    g = GCD(n, d)
    FOR i = 0 TO g-1:
        temp = arr[i]
        j = i
        WHILE TRUE:
            k = (j + d) % n
            IF k == i: BREAK
            arr[j] = arr[k]
            j = k
        arr[j] = temp
```

### Trace: Left rotate [1,2,3,4,5,6] by 2

```
  n=6, d=2, GCD(6,2)=2 → 2 cycles

  Cycle 0 (start at i=0):
  ┌───┬───┬───┬───┬───┬───┐
  │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │   temp = arr[0] = 1
  └───┴───┴───┴───┴───┴───┘
    ↑       ↑       ↑
    0 ←──── 2 ←──── 4 ←──── 0(temp)

  arr[0] = arr[2] = 3
  arr[2] = arr[4] = 5
  arr[4] = temp = 1
  
  Result: [3, 2, 5, 4, 1, 6]

  Cycle 1 (start at i=1):
    ↑       ↑       ↑
    1 ←──── 3 ←──── 5 ←──── 1(temp)

  arr[1] = arr[3] = 4
  arr[3] = arr[5] = 6
  arr[5] = temp = 2

  Result: [3, 4, 5, 6, 1, 2] ✓
```

**Time: O(n)** | **Space: O(1)**

---

## Right Rotation

Right rotation by d is equivalent to left rotation by (n - d):

```
FUNCTION rightRotate(arr, n, d):
    d = d % n
    leftRotate(arr, n, n - d)

// Or using reversal: (reverse order of steps)
FUNCTION rightRotate(arr, n, d):
    d = d % n
    reverse(arr, 0, n-1)         // reverse all
    reverse(arr, 0, d-1)         // reverse first d
    reverse(arr, d, n-1)         // reverse rest
```

### Trace: Right rotate [1,2,3,4,5] by 2

```
  Step 1: Reverse all     → [5, 4, 3, 2, 1]
  Step 2: Reverse first 2 → [4, 5, 3, 2, 1]
  Step 3: Reverse rest    → [4, 5, 1, 2, 3]

  Result: [4, 5, 1, 2, 3] ✓
```

---

## Edge Cases

```
  ┌────────────────────────────────────────┐
  │  EDGE CASES TO HANDLE:                  │
  │                                         │
  │  1. d = 0 → no rotation needed          │
  │  2. d = n → same as no rotation         │
  │  3. d > n → use d = d % n              │
  │  4. d < 0 → left rotate by |d| = right │
  │  5. n = 1 → single element, no change  │
  │  6. n = 0 → empty array, no change     │
  └────────────────────────────────────────┘
```

---

## Comparison of All Approaches

| Approach | Time | Space | In-Place? | # Moves |
|----------|------|-------|-----------|---------|
| Brute force (one-by-one) | O(n × d) | O(1) | Yes | n × d |
| Extra array | O(n) | O(n) | No | 2n |
| **Reversal algorithm** | **O(n)** | **O(1)** | **Yes** | **2n** |
| Juggling (GCD) | O(n) | O(1) | Yes | n |

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Left rotate by d | First d elements go to end |
| Right rotate by d | Last d elements go to front |
| Right rotate by d | = Left rotate by (n - d) |
| Handle d > n | Use d = d % n |
| Best approach | Reversal algorithm: O(n) time, O(1) space |
| Key insight | Reverse parts, then reverse whole |

---

## Quick Revision Questions

1. **Explain the reversal algorithm for left rotation.** Why do three reversals produce the correct result?

2. **Left rotate by d is the same as right rotate by ___?** Fill in the blank in terms of n and d.

3. **What happens when d = n?** When d > n? How do you handle it?

4. **In the juggling algorithm, why does GCD(n, d) give the number of cycles?**

5. **Trace the reversal algorithm** for right-rotating [10, 20, 30, 40, 50, 60] by 3.

6. **Which approach would you use in an interview and why?**

---

[← Previous: Reversing an Array](04-reversing-an-array.md) | [Back to README](../README.md) | [Next: Array Manipulation →](06-array-manipulation.md)

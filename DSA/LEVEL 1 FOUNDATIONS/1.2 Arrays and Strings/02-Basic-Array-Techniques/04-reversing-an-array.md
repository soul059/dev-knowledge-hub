# Chapter 4: Reversing an Array

[← Previous: Finding Min/Max](03-finding-min-max.md) | [Back to README](../README.md) | [Next: Rotation →](05-rotation.md)

---

## Overview

Reversing an array is a fundamental operation that appears in many algorithms. It's also an excellent introduction to the two-pointer technique. This chapter covers multiple approaches and their trade-offs.

---

## Approach 1: Two-Pointer Swap (In-Place) — Optimal

```
FUNCTION reverse(arr, n):
    left = 0
    right = n - 1
    WHILE left < right:
        SWAP(arr[left], arr[right])
        left = left + 1
        right = right - 1
```

### Step-by-Step Trace

```
  Array: [1, 2, 3, 4, 5]

  Step 0:  L=0, R=4
  ┌─────┬─────┬─────┬─────┬─────┐
  │  1  │  2  │  3  │  4  │  5  │
  └─────┴─────┴─────┴─────┴─────┘
    L                         R
         swap(1, 5)

  Step 1:  L=1, R=3
  ┌─────┬─────┬─────┬─────┬─────┐
  │  5  │  2  │  3  │  4  │  1  │
  └─────┴─────┴─────┴─────┴─────┘
           L           R
              swap(2, 4)

  Step 2:  L=2, R=2  → L not < R → STOP
  ┌─────┬─────┬─────┬─────┬─────┐
  │  5  │  4  │  3  │  2  │  1  │
  └─────┴─────┴─────┴─────┴─────┘
                 LR
              (middle stays)

  Result: [5, 4, 3, 2, 1]  ✓
  Swaps: ⌊n/2⌋ = 2
```

**Time: O(n)** | **Space: O(1)** | **Swaps: n/2**

---

## Even vs Odd Length

```
  Even (n=4): [A, B, C, D]
  Step 1: swap A,D → [D, B, C, A]
  Step 2: swap B,C → [D, C, B, A]  ✓
  
  L and R meet — both pointers cross.

  ┌───┬───┬───┬───┐      ┌───┬───┬───┬───┐
  │ A │ B │ C │ D │  →   │ D │ C │ B │ A │
  └───┴───┴───┴───┘      └───┴───┴───┴───┘
   L           R             done!

  Odd (n=5): [A, B, C, D, E]
  Step 1: swap A,E → [E, B, C, D, A]
  Step 2: swap B,D → [E, D, C, B, A]  ✓
  
  Middle element (C) stays in place.

  ┌───┬───┬───┬───┬───┐      ┌───┬───┬───┬───┬───┐
  │ A │ B │ C │ D │ E │  →   │ E │ D │ C │ B │ A │
  └───┴───┴───┴───┴───┘      └───┴───┴───┴───┴───┘
   L       ↑       R                  stays
           middle
```

---

## Approach 2: Using Extra Array

```
FUNCTION reverseWithCopy(arr, n):
    result = NEW ARRAY[n]
    FOR i = 0 TO n-1:
        result[i] = arr[n - 1 - i]
    RETURN result
```

```
  Original: [1, 2, 3, 4, 5]

  result[0] = arr[4] = 5
  result[1] = arr[3] = 4
  result[2] = arr[2] = 3
  result[3] = arr[1] = 2
  result[4] = arr[0] = 1

  Result: [5, 4, 3, 2, 1]
```

**Time: O(n)** | **Space: O(n)**

---

## Approach 3: Recursive

```
FUNCTION reverseRecursive(arr, left, right):
    IF left >= right:
        RETURN
    SWAP(arr[left], arr[right])
    reverseRecursive(arr, left + 1, right - 1)
```

```
  Call stack for [1, 2, 3, 4, 5]:

  reverse(arr, 0, 4)  → swap(1,5) → [5,2,3,4,1]
    reverse(arr, 1, 3) → swap(2,4) → [5,4,3,2,1]
      reverse(arr, 2, 2) → left >= right → RETURN
```

**Time: O(n)** | **Space: O(n/2) = O(n)** due to recursion stack

---

## Approach 4: Using Stack

```
FUNCTION reverseWithStack(arr, n):
    stack = empty stack
    FOR i = 0 TO n-1:
        stack.push(arr[i])
    FOR i = 0 TO n-1:
        arr[i] = stack.pop()
```

```
  Push phase:          Pop phase:
  
  Stack:               Stack:
  ┌───┐               ┌───┐
  │ 5 │ ← top         │   │  → arr[0] = 5
  │ 4 │               │   │  → arr[1] = 4
  │ 3 │               │   │  → arr[2] = 3
  │ 2 │               │   │  → arr[3] = 2
  │ 1 │               │   │  → arr[4] = 1
  └───┘               └───┘

  Result: [5, 4, 3, 2, 1]
```

**Time: O(n)** | **Space: O(n)**

---

## Reverse a Subarray

Reverse only elements from index `start` to `end`:

```
FUNCTION reverseSubarray(arr, start, end):
    WHILE start < end:
        SWAP(arr[start], arr[end])
        start = start + 1
        end = end - 1
```

```
  Reverse indices 1 to 3 in [10, 20, 30, 40, 50]:

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
           [───── reverse ─────]

  Result: [10, 40, 30, 20, 50]
```

This is a building block for **rotation algorithms** (next chapter).

---

## Applications of Array Reversal

| Application | How Reversal Helps |
|-------------|-------------------|
| **Array rotation** | Reversal-based rotation algorithm |
| **Palindrome check** | Reverse and compare |
| **Stack implementation** | LIFO via reversed iteration |
| **String reversal** | Reverse character array |
| **Wave sort** | Partial reversals |
| **Next permutation** | Reverse suffix |

---

## Comparison of Approaches

| Approach | Time | Space | In-Place? | Notes |
|----------|------|-------|-----------|-------|
| Two-pointer swap | O(n) | O(1) | Yes | **Best approach** |
| Extra array | O(n) | O(n) | No | Simple but wasteful |
| Recursive | O(n) | O(n) | Yes* | Stack space overhead |
| Stack-based | O(n) | O(n) | No | Conceptual exercise |

*\* Modifies in-place but uses O(n) stack space for recursion*

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Best approach | Two-pointer swap |
| Time complexity | O(n) |
| Space complexity | O(1) for two-pointer |
| Number of swaps | ⌊n/2⌋ |
| Odd-length arrays | Middle element stays |
| Key technique | Converging pointers |
| Building block for | Rotation, palindrome, permutation |

---

## Quick Revision Questions

1. **How many swaps are needed to reverse an array of 100 elements?**

2. **Why is the recursive approach O(n) in space** even though it modifies the array in-place?

3. **Write pseudocode to reverse only the first k elements** of an array.

4. **If you reverse an already reversed array, what do you get?** What property does this reveal?

5. **How is reversing a subarray useful in array rotation?** (Preview of next chapter.)

6. **Can you reverse an array in fewer than n/2 swaps?** Why or why not?

---

[← Previous: Finding Min/Max](03-finding-min-max.md) | [Back to README](../README.md) | [Next: Rotation →](05-rotation.md)

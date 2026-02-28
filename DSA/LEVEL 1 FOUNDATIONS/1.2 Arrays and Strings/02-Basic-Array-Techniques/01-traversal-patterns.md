# Chapter 1: Traversal Patterns

[← Previous Unit: Array vs Other Structures](../01-Array-Fundamentals/06-array-vs-other-structures.md) | [Back to README](../README.md) | [Next: Linear Search →](02-linear-search.md)

---

## Overview

Array traversal means visiting every element in a specific order. Choosing the right traversal pattern is the first step in solving any array problem. This chapter covers all standard traversal patterns you'll encounter.

---

## 1. Forward Traversal (Left to Right)

The most common pattern — visit elements from index 0 to n-1.

```
FUNCTION forwardTraversal(arr, n):
    FOR i = 0 TO n-1:
        process(arr[i])
```

```
  Direction: ────────────────────►

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
    i=0    i=1    i=2    i=3    i=4

  Visit order: 10 → 20 → 30 → 40 → 50
  Time: O(n)    Space: O(1)
```

**Use cases:** Printing, summing, searching, counting.

---

## 2. Backward Traversal (Right to Left)

```
FUNCTION backwardTraversal(arr, n):
    FOR i = n-1 DOWN TO 0:
        process(arr[i])
```

```
  Direction: ◄────────────────────

  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
    i=4    i=3    i=2    i=1    i=0

  Visit order: 50 → 40 → 30 → 20 → 10
  Time: O(n)    Space: O(1)
```

**Use cases:** Reversing, deleting while traversing, some DP problems.

---

## 3. Two-End Traversal (Converging Pointers)

```
FUNCTION twoEndTraversal(arr, n):
    left = 0
    right = n - 1
    WHILE left < right:
        process(arr[left], arr[right])
        left = left + 1
        right = right - 1
```

```
  ┌──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │  50  │
  └──────┴──────┴──────┴──────┴──────┘
    L──────────────────────────────R      Step 0: L=0, R=4
       L──────────────────────R          Step 1: L=1, R=3
          L──────────R                   Step 2: L=2, R=2 → stop

  Pairs visited: (10,50), (20,40), then 30 is the middle
  Time: O(n/2) = O(n)    Space: O(1)
```

**Use cases:** Palindrome check, reversing in-place, two-sum (sorted).

---

## 4. Skip Traversal (Step Size > 1)

```
FUNCTION skipTraversal(arr, n, step):
    FOR i = 0 TO n-1 STEP step:
        process(arr[i])
```

```
  Step = 2 (visit every other element):

  ┌──────┬──────┬──────┬──────┬──────┬──────┐
  │  10  │  20  │  30  │  40  │  50  │  60  │
  └──────┴──────┴──────┴──────┴──────┴──────┘
    i=0           i=2           i=4
    ▲             ▲             ▲
  visited       visited       visited

  Visit order: 10 → 30 → 50
  Time: O(n/step)    Space: O(1)
```

**Use cases:** Even/odd indexed elements, sampling, jump search.

---

## 5. Nested Traversal (All Pairs)

```
FUNCTION allPairs(arr, n):
    FOR i = 0 TO n-1:
        FOR j = i+1 TO n-1:
            process(arr[i], arr[j])
```

```
  Array: [A, B, C, D]

  i=0: (A,B) (A,C) (A,D)
  i=1:       (B,C) (B,D)
  i=2:             (C,D)

  Total pairs: n(n-1)/2
  Time: O(n²)    Space: O(1)
```

**Use cases:** Finding pairs with a property, brute-force two-sum.

---

## 6. Zigzag Traversal

```
FUNCTION zigzagTraversal(arr, n):
    FOR i = 0 TO n-1:
        IF i is even:
            process arr[i] going right →
        ELSE:
            process arr[i] going left ←
```

```
  Most useful in 2D arrays:

  Row 0: → → → →
  Row 1: ← ← ← ←
  Row 2: → → → →

  ┌──┬──┬──┬──┐
  │ 1│ 2│ 3│ 4│  →
  ├──┼──┼──┼──┤
  │ 8│ 7│ 6│ 5│  ←
  ├──┼──┼──┼──┤
  │ 9│10│11│12│  →
  └──┴──┴──┴──┘

  Visit: 1,2,3,4,8,7,6,5,9,10,11,12
```

**Use cases:** Zigzag level-order traversal, snake-order problems.

---

## 7. Circular Traversal

```
FUNCTION circularTraversal(arr, n, start):
    FOR count = 0 TO n-1:
        index = (start + count) % n
        process(arr[index])
```

```
  Array: [A, B, C, D, E], start = 2

  ┌──────┬──────┬──────┬──────┬──────┐
  │  A   │  B   │  C   │  D   │  E   │
  └──────┴──────┴──────┴──────┴──────┘
    [0]    [1]    [2]    [3]    [4]
                   ▲─start
                   │
  Visit order: C → D → E → A → B
  Indices:     2 → 3 → 4 → 0 → 1

  Uses modulo: (2+0)%5=2, (2+1)%5=3, (2+2)%5=4, (2+3)%5=0, (2+4)%5=1
```

**Use cases:** Circular buffers, rotation problems, Josephus problem.

---

## 8. Window Traversal

```
FUNCTION windowTraversal(arr, n, windowSize):
    FOR i = 0 TO n - windowSize:
        process(arr[i .. i+windowSize-1])
```

```
  Array: [1, 3, 5, 2, 8, 7], window = 3

  ┌───┬───┬───┬───┬───┬───┐
  │ 1 │ 3 │ 5 │ 2 │ 8 │ 7 │
  └───┴───┴───┴───┴───┴───┘
  [─────────]                   Window 1: [1,3,5]
      [─────────]               Window 2: [3,5,2]
          [─────────]           Window 3: [5,2,8]
              [─────────]       Window 4: [2,8,7]

  Total windows: n - k + 1 = 6 - 3 + 1 = 4
```

**Use cases:** Sliding window problems (Unit 4), moving averages.

---

## Traversal Pattern Selection Guide

```
  ┌─────────────────────────────────────────────────┐
  │           WHICH TRAVERSAL TO USE?                │
  ├─────────────────────────────────────────────────┤
  │                                                  │
  │  Simple processing of all elements?              │
  │  → Forward traversal                             │
  │                                                  │
  │  Need to compare start/end of array?             │
  │  → Two-end traversal                             │
  │                                                  │
  │  Looking at subarrays of fixed size?             │
  │  → Window traversal                              │
  │                                                  │
  │  Need all pairs for comparison?                  │
  │  → Nested traversal (try to optimize!)           │
  │                                                  │
  │  Array wraps around?                             │
  │  → Circular traversal with modulo                │
  │                                                  │
  │  Need to process from end backward?              │
  │  → Backward traversal                            │
  └─────────────────────────────────────────────────┘
```

---

## Summary Table

| Pattern | Direction | Time | Space | Key Technique |
|---------|-----------|------|-------|---------------|
| Forward | → | O(n) | O(1) | `for i = 0 to n-1` |
| Backward | ← | O(n) | O(1) | `for i = n-1 downto 0` |
| Two-end | ←→ | O(n) | O(1) | Left and right pointers |
| Skip | → (step k) | O(n/k) | O(1) | `i += step` |
| Nested | All pairs | O(n²) | O(1) | Double loop |
| Zigzag | →←→← | O(n) | O(1) | Alternate direction |
| Circular | Wrap-around | O(n) | O(1) | Modulo operator |
| Window | Sliding | O(n) | O(k) | Fixed-size subarrays |

---

## Quick Revision Questions

1. **Which traversal pattern would you use for palindrome checking?** Why?

2. **What is the time complexity of visiting all pairs in an array of size n?** How many pairs are there?

3. **How does the modulo operator enable circular traversal?** Write the index formula.

4. **When would you use backward traversal instead of forward?** Give an example problem.

5. **How many windows of size k exist in an array of size n?**

6. **Can you traverse an array in O(n) and still solve an O(n²) brute-force problem?** (Hint: Think about two pointers.)

---

[← Previous Unit: Array vs Other Structures](../01-Array-Fundamentals/06-array-vs-other-structures.md) | [Back to README](../README.md) | [Next: Linear Search →](02-linear-search.md)

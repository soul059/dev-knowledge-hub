# Chapter 2: Opposite Direction Pointers

[← Previous: What is Two Pointer?](01-what-is-two-pointer.md) | [Back to README](../README.md) | [Next: Same Direction Pointers →](03-same-direction-pointers.md)

---

## Overview

Opposite direction (converging) pointers start at both ends of the array and move inward. This is the most classic two-pointer pattern, used primarily on **sorted arrays** to find pairs or compare extremes.

---

## How It Works

```
  ┌───┬───┬───┬───┬───┬───┬───┬───┐
  │ 1 │ 3 │ 5 │ 7 │ 9 │11 │13 │15 │   SORTED array
  └───┴───┴───┴───┴───┴───┴───┴───┘
    L                             R

  At each step:
  • Compute something from arr[L] and arr[R]
  • If result is too small → L++ (increase the smaller value)
  • If result is too big  → R-- (decrease the larger value)
  • If result is just right → found answer!

  KEY INSIGHT: In a sorted array, moving L right increases the sum,
               and moving R left decreases the sum.
```

---

## Problem 1: Pair with Target Sum (Sorted Array)

Find two numbers that add up to a target.

```
FUNCTION pairSum(arr, n, target):
    L = 0, R = n - 1
    WHILE L < R:
        sum = arr[L] + arr[R]
        IF sum == target:
            RETURN (L, R)
        ELSE IF sum < target:
            L = L + 1       // need bigger sum
        ELSE:
            R = R - 1       // need smaller sum
    RETURN "not found"
```

### Trace: arr = [1, 3, 5, 7, 9, 11], target = 12

```
  L=0, R=5: arr[0]+arr[5] = 1+11 = 12 == target → FOUND! (0, 5)

  Another example: target = 10
  L=0, R=5: 1+11 = 12 > 10 → R=4
  L=0, R=4: 1+9  = 10 == 10 → FOUND! (0, 4)

  Another example: target = 8
  L=0, R=5: 1+11 = 12 > 8  → R=4
  L=0, R=4: 1+9  = 10 > 8  → R=3
  L=0, R=3: 1+7  = 8  == 8 → FOUND! (0, 3)
```

### Why This Works (Proof Sketch)

```
  Sorted: a₀ ≤ a₁ ≤ ... ≤ aₙ₋₁

  If arr[L] + arr[R] < target:
    → arr[L] + arr[k] < target for ALL k < R  (since arr[k] ≤ arr[R])
    → L is too small for ANY partner ≤ R
    → Safe to discard L, move to L+1

  If arr[L] + arr[R] > target:
    → arr[k] + arr[R] > target for ALL k > L  (since arr[k] ≥ arr[L])
    → R is too large for ANY partner ≥ L
    → Safe to discard R, move to R-1

  We never skip a valid answer!
```

---

## Problem 2: Squaring a Sorted Array

Given a sorted array (may have negatives), return squares in sorted order.

```
  Input:  [-4, -2, 0, 1, 3, 5]
  Squares: [16, 4, 0, 1, 9, 25]
  Sorted:  [0, 1, 4, 9, 16, 25]
```

```
FUNCTION sortedSquares(arr, n):
    result = NEW ARRAY[n]
    L = 0, R = n - 1
    pos = n - 1     // fill result from back

    WHILE L <= R:
        leftSq = arr[L] * arr[L]
        rightSq = arr[R] * arr[R]
        IF leftSq > rightSq:
            result[pos] = leftSq
            L = L + 1
        ELSE:
            result[pos] = rightSq
            R = R - 1
        pos = pos - 1

    RETURN result
```

### Trace: [-4, -2, 0, 1, 3, 5]

```
  L=0, R=5, pos=5: (-4)²=16, (5)²=25 → 25>16, result[5]=25, R=4
  L=0, R=4, pos=4: (-4)²=16, (3)²=9  → 16>9,  result[4]=16, L=1
  L=1, R=4, pos=3: (-2)²=4,  (3)²=9  → 9>4,   result[3]=9,  R=3
  L=1, R=3, pos=2: (-2)²=4,  (1)²=1  → 4>1,   result[2]=4,  L=2
  L=2, R=3, pos=1: (0)²=0,   (1)²=1  → 1>0,   result[1]=1,  R=2
  L=2, R=2, pos=0: (0)²=0            →         result[0]=0

  Result: [0, 1, 4, 9, 16, 25] ✓
```

**Time: O(n)** | **Space: O(n)** for result

---

## Problem 3: Trapping Rain Water

Given heights, calculate how much water can be trapped.

```
  Heights: [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]

  Visualization:
                                  █
              █                   █ █     █
      █       █ █     █           █ █ █   █ █
  ────█───█───█─█─█───█───█───────█─█─█───█─█───
   0  1   0   2 1 0   1   3      2 1 2   1

  Water fills between bars:
              █ ~ ~ ~ ~ ~ ~ ~ ~ ~█
      █ ~ ~ ~ █ █ ~ ~ █ ~ ~ ~ ~ ~█ █ ~ ~█ █
  ────█───█───█─█─█───█───█───────█─█─█───█─█───
```

```
FUNCTION trapRainWater(height, n):
    L = 0, R = n - 1
    leftMax = 0, rightMax = 0
    water = 0

    WHILE L < R:
        IF height[L] < height[R]:
            IF height[L] >= leftMax:
                leftMax = height[L]
            ELSE:
                water = water + (leftMax - height[L])
            L = L + 1
        ELSE:
            IF height[R] >= rightMax:
                rightMax = height[R]
            ELSE:
                water = water + (rightMax - height[R])
            R = R - 1

    RETURN water
```

**Time: O(n)** | **Space: O(1)**

---

## Problem 4: Valid Palindrome (String)

Check if a string is a palindrome, ignoring non-alphanumeric characters.

```
FUNCTION isPalindrome(s):
    L = 0, R = length(s) - 1
    WHILE L < R:
        WHILE L < R AND NOT isAlphanumeric(s[L]):
            L = L + 1
        WHILE L < R AND NOT isAlphanumeric(s[R]):
            R = R - 1
        IF toLower(s[L]) != toLower(s[R]):
            RETURN false
        L = L + 1
        R = R - 1
    RETURN true
```

### Trace: "A man, a plan, a canal: Panama"

```
  Cleaned: "amanaplanacanalpanama"
  
  L=0('a'), R=19('a') → match ✓
  L=1('m'), R=18('m') → match ✓
  L=2('a'), R=17('a') → match ✓
  ...all match...
  
  Result: true (it's a palindrome!) ✓
```

---

## Problem 5: Remove Element In-Place

Remove all occurrences of a value, return new length.

```
FUNCTION removeElement(arr, n, val):
    L = 0, R = n - 1
    WHILE L <= R:
        IF arr[L] == val:
            arr[L] = arr[R]     // swap with end
            R = R - 1           // shrink valid range
        ELSE:
            L = L + 1
    RETURN L      // new length
```

### Trace: arr = [3, 2, 2, 3], val = 3

```
  L=0, R=3: arr[0]=3 == val → arr[0]=arr[3]=3, R=2
  L=0, R=2: arr[0]=3 == val → arr[0]=arr[2]=2, R=1
  L=0, R=1: arr[0]=2 ≠ val → L=1
  L=1, R=1: arr[1]=2 ≠ val → L=2
  L=2 > R=1 → STOP

  Result: [2, 2, ...], length = 2 ✓
```

---

## Pattern: When to Move Which Pointer

```
  ┌──────────────────────────────────────────────┐
  │    DECISION LOGIC FOR OPPOSITE POINTERS       │
  ├──────────────────────────────────────────────┤
  │                                               │
  │  If result too SMALL:                         │
  │    → Move LEFT pointer RIGHT (increase L)     │
  │    → This increases arr[L] (sorted array)     │
  │                                               │
  │  If result too LARGE:                         │
  │    → Move RIGHT pointer LEFT (decrease R)     │
  │    → This decreases arr[R] (sorted array)     │
  │                                               │
  │  If result is CORRECT:                        │
  │    → Record answer                            │
  │    → Move both (or one, depending on problem) │
  └──────────────────────────────────────────────┘
```

---

## Summary Table

| Problem | Time | Space | Key Insight |
|---------|------|-------|-------------|
| Pair sum (sorted) | O(n) | O(1) | Sum too big→R--, too small→L++ |
| Sorted squares | O(n) | O(n) | Largest squares at extremes |
| Trapping rain water | O(n) | O(1) | Process shorter side first |
| Palindrome check | O(n) | O(1) | Compare from both ends |
| Remove element | O(n) | O(1) | Swap unwanted to end |

---

## Quick Revision Questions

1. **Why must the array be sorted for the pair-sum problem?** What happens if it isn't sorted?

2. **In the sorted squares problem, why do we fill the result array from the back?**

3. **Explain why moving the left pointer right increases the sum** in a sorted array.

4. **For trapping rain water, why do we always process the shorter side first?**

5. **How many total iterations does the opposite-pointer loop take?** Prove it's O(n).

6. **Can opposite pointers find ALL pairs with a given sum?** How would you modify the algorithm?

---

[← Previous: What is Two Pointer?](01-what-is-two-pointer.md) | [Back to README](../README.md) | [Next: Same Direction Pointers →](03-same-direction-pointers.md)

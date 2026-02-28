# 6.1 Two Sum Problem

## Unit 6: Classic Hash Problems

---

## Problem Statement

> Given an array of integers `nums` and a target integer `target`, return the **indices** of two numbers that add up to `target`. Each input has exactly one solution, and you may not use the same element twice.

```
Example:
  nums = [2, 7, 11, 15], target = 9
  Output: [0, 1]   (because nums[0] + nums[1] = 2 + 7 = 9)
```

---

## Brute Force Approach

```
FUNCTION twoSumBrute(nums, target):
    FOR i = 0 TO n - 2:
        FOR j = i + 1 TO n - 1:
            IF nums[i] + nums[j] == target:
                RETURN [i, j]
    RETURN null

Time: O(n²)  |  Space: O(1)
```

Every pair is checked — for $n = 10{,}000$, that's ~50 million comparisons.

---

## Hash Table Approach: Core Insight

```
╔══════════════════════════════════════════════════════════════╗
║  KEY INSIGHT:                                               ║
║                                                              ║
║  Instead of asking "which two numbers add to target?"       ║
║  ask: "for each number x, does (target - x) exist?"        ║
║                                                              ║
║  a + b = target  →  b = target - a                         ║
║                                                              ║
║  • Walk through array once                                  ║
║  • For each number, compute its COMPLEMENT                  ║
║  • Check if complement is already in hash map               ║
║  • If yes → found pair!                                    ║
║  • If no  → store current number in map                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Algorithm

```
FUNCTION twoSum(nums, target):
    map = new HashMap()       // value → index

    FOR i = 0 TO n - 1:
        complement = target - nums[i]

        IF map.containsKey(complement):
            RETURN [map.get(complement), i]

        map.put(nums[i], i)

    RETURN null   // No solution found

Time: O(n)  |  Space: O(n)
```

---

## Step-by-Step Trace

```
nums = [2, 7, 11, 15], target = 9

Step 1: i=0, nums[0]=2
  complement = 9 - 2 = 7
  map = {} → 7 not found
  Store: map = {2: 0}

Step 2: i=1, nums[1]=7
  complement = 9 - 7 = 2
  map = {2: 0} → 2 FOUND at index 0! ✓
  RETURN [0, 1]

  ┌─────────────────────────────────────────────┐
  │ Only 2 hash lookups instead of 6 comparisons│
  └─────────────────────────────────────────────┘
```

### Larger Example

```
nums = [3, 5, -4, 8, 11, 1, -1, 6], target = 10

i=0: num=3,  comp=7,  map={}                     → not found → map={3:0}
i=1: num=5,  comp=5,  map={3:0}                  → not found → map={3:0, 5:1}
i=2: num=-4, comp=14, map={3:0, 5:1}             → not found → map={3:0, 5:1, -4:2}
i=3: num=8,  comp=2,  map={3:0, 5:1, -4:2}       → not found → map={..., 8:3}
i=4: num=11, comp=-1, map={..., 8:3}             → not found → map={..., 11:4}
i=5: num=1,  comp=9,  map={..., 11:4}            → not found → map={..., 1:5}
i=6: num=-1, comp=11, map={..., 11:4, 1:5}       → 11 FOUND at index 4! ✓

RETURN [4, 6]   (nums[4] + nums[6] = 11 + (-1) = 10) ✓
```

---

## Why One-Pass Works

```
╔══════════════════════════════════════════════════════════════╗
║  Consider pair (a, b) where a comes before b in array:     ║
║                                                              ║
║  When we process a:                                         ║
║    • We look for (target - a) = b                          ║
║    • b hasn't been seen yet → not found                    ║
║    • Store a in map                                         ║
║                                                              ║
║  When we later process b:                                   ║
║    • We look for (target - b) = a                          ║
║    • a IS in the map → FOUND!                              ║
║                                                              ║
║  Every valid pair is detected when the SECOND element       ║
║  of the pair is processed.                                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Variations

### Two Sum — Return All Pairs

```
FUNCTION twoSumAll(nums, target):
    map = new HashMap()    // value → list of indices
    result = []

    FOR i = 0 TO n - 1:
        complement = target - nums[i]
        IF map.containsKey(complement):
            FOR each j in map.get(complement):
                result.add([j, i])
        
        IF not map.containsKey(nums[i]):
            map.put(nums[i], [])
        map.get(nums[i]).add(i)

    RETURN result
```

### Two Sum — Sorted Array (Two Pointers)

```
FUNCTION twoSumSorted(nums, target):
    left = 0
    right = n - 1

    WHILE left < right:
        sum = nums[left] + nums[right]
        IF sum == target:
            RETURN [left, right]
        ELSE IF sum < target:
            left++
        ELSE:
            right--

Time: O(n)  |  Space: O(1)  — but requires sorted input
```

---

## Complexity Comparison

| Approach | Time | Space | Notes |
|----------|:---:|:---:|-------|
| Brute force | $O(n^2)$ | $O(1)$ | Check all pairs |
| Hash map (1-pass) | $O(n)$ | $O(n)$ | Best general approach |
| Sort + two pointers | $O(n \log n)$ | $O(1)$ | If indices not needed |
| Sort + binary search | $O(n \log n)$ | $O(1)$ | Alternative to two pointers |

---

## Quick Revision Question

**Q: In the one-pass hash map solution for Two Sum, why do we check for the complement BEFORE inserting the current element into the map?**

<details>
<summary>Click to reveal answer</summary>

We check before inserting to **avoid using the same element twice**. If we inserted first and then checked, an element could pair with itself.

**Example**: `nums = [3, 3], target = 6`
- If we insert first: put 3 → map has {3:0} → check complement 6-3=3 → found 3 → returns [0, 0] — **WRONG!** (same index used twice)
- If we check first: check complement 3 → map is empty → not found → put 3 → next iteration: check complement 3 → found at index 0 → returns [0, 1] — **CORRECT!**

By checking first, we guarantee the complement was stored during a previous iteration (different index).

</details>

---

| [← Unit 5: Space Complexity](../05-Hash-Table-Operations/06-space-complexity.md) | [Next: Frequency Counting →](02-frequency-counting.md) |
|:---|---:|

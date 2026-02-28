# 7.1 Duplicate Detection

## Unit 7: Hash Set Problems

---

## The Pattern

Use a **HashSet** to detect duplicates in $O(n)$ time — as you process each element, check if it already exists in the set.

```
╔══════════════════════════════════════════════════════════════╗
║              DUPLICATE DETECTION PATTERN                    ║
║                                                              ║
║  FOR each element:                                          ║
║    IF element IN set → DUPLICATE FOUND                     ║
║    ELSE → add element to set                               ║
║                                                              ║
║  Time: O(n)  |  Space: O(n)                                ║
║                                                              ║
║  vs Brute Force: O(n²) time, O(1) space                   ║
║  vs Sorting:     O(n log n) time, O(1) space               ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Problem 1: Contains Duplicate

> Given an integer array `nums`, return `true` if any value appears at least twice.

```
FUNCTION containsDuplicate(nums):
    seen = new HashSet()

    FOR each num in nums:
        IF seen.contains(num):
            RETURN true
        seen.add(num)

    RETURN false
```

### Trace

```
nums = [1, 2, 3, 1]

Step 1: num=1, seen={}, not found → add → seen={1}
Step 2: num=2, seen={1}, not found → add → seen={1,2}
Step 3: num=3, seen={1,2}, not found → add → seen={1,2,3}
Step 4: num=1, seen={1,2,3}, FOUND! → return true ✓
```

---

## Problem 2: Contains Duplicate II (Within K Distance)

> Return `true` if there are two distinct indices $i$ and $j$ such that `nums[i] == nums[j]` and $|i - j| \leq k$.

```
FUNCTION containsNearbyDuplicate(nums, k):
    window = new HashSet()

    FOR i = 0 TO n - 1:
        IF window.contains(nums[i]):
            RETURN true
        
        window.add(nums[i])

        // Maintain window of size k
        IF window.size() > k:
            window.remove(nums[i - k])

    RETURN false
```

### Trace

```
nums = [1, 2, 3, 1, 2, 3], k = 2

i=0: num=1, window={}, not found → add → window={1}
i=1: num=2, window={1}, not found → add → window={1,2}
i=2: num=3, window={1,2}, not found → add → window={1,2,3}
     window.size()=3 > k=2 → remove nums[0]=1 → window={2,3}
i=3: num=1, window={2,3}, not found → add → window={2,3,1}
     window.size()=3 > k=2 → remove nums[1]=2 → window={3,1}
i=4: num=2, window={3,1}, not found → add → window={3,1,2}
     window.size()=3 > k=2 → remove nums[2]=3 → window={1,2}
i=5: num=3, window={1,2}, not found → add → window={1,2,3}
     window.size()=3 > k=2 → remove nums[3]=1 → window={2,3}

No duplicate within k=2 → return false

Try with k=3: same array
i=3: num=1, window would be {1,2,3} → 1 FOUND! return true ✓
```

---

## Problem 3: Find All Duplicates

> Given an array where 1 ≤ a[i] ≤ n and each element appears once or twice, find all elements that appear twice.

```
FUNCTION findDuplicates(nums):
    seen = new HashSet()
    result = []

    FOR each num in nums:
        IF seen.contains(num):
            result.add(num)
        ELSE:
            seen.add(num)

    RETURN result

Trace: nums = [4, 3, 2, 7, 8, 2, 3, 1]
  Process: 4→add, 3→add, 2→add, 7→add, 8→add
           2→DUPLICATE!, 3→DUPLICATE!, 1→add
  Result: [2, 3]
```

---

## Problem 4: Single Number (HashSet XOR Alternative)

> Every element appears twice except one. Find it.

```
// HashSet approach
FUNCTION singleNumber(nums):
    seen = new HashSet()

    FOR each num in nums:
        IF seen.contains(num):
            seen.remove(num)    // Seen twice → remove
        ELSE:
            seen.add(num)

    RETURN seen.iterator().next()    // Only remaining element

Trace: nums = [2, 1, 4, 1, 2]
  2 → add → {2}
  1 → add → {2, 1}
  4 → add → {2, 1, 4}
  1 → remove → {2, 4}
  2 → remove → {4}
  Answer: 4 ✓
```

Note: The XOR approach ($O(1)$ space) is better for this specific problem, but the HashSet pattern generalizes to other "find unique" variations.

---

## Comparison of Duplicate Detection Methods

| Method | Time | Space | Modifies Input? |
|--------|:---:|:---:|:---:|
| Nested loops | $O(n^2)$ | $O(1)$ | No |
| Sort first | $O(n\log n)$ | $O(1)$ or $O(n)$ | Usually yes |
| HashSet | $O(n)$ | $O(n)$ | No |
| Bit manipulation | $O(n)$ | $O(1)$ | No (limited cases) |
| Index marking | $O(n)$ | $O(1)$ | Yes (if 1..n range) |

```
╔══════════════════════════════════════════════════════════════╗
║  WHEN TO USE WHICH:                                         ║
║                                                              ║
║  HashSet: General purpose, any data type, preserves input   ║
║  Sort:    When space is critical AND input can be modified  ║
║  Bit:     Only works for specific problems (XOR trick)      ║
║  Index:   Only when values are in range [1, n]              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Deduplication (Remove Duplicates)

```
FUNCTION removeDuplicates(arr):
    seen = new HashSet()
    result = []

    FOR each element in arr:
        IF not seen.contains(element):
            seen.add(element)
            result.add(element)

    RETURN result

// Preserves FIRST occurrence order

Trace: [3, 1, 4, 1, 5, 9, 2, 6, 5, 3]
  Result: [3, 1, 4, 5, 9, 2, 6]
```

---

## Quick Revision Question

**Q: You have a data stream of integers arriving one at a time. You need to detect the first repeated integer as soon as it arrives. Which data structure would you use and what is the time complexity per element?**

<details>
<summary>Click to reveal answer</summary>

Use a **HashSet**.

For each incoming integer:
1. Check if it exists in the set — $O(1)$ average
2. If yes → this is the first duplicate, report it
3. If no → add it to the set — $O(1)$ average

**Time per element: $O(1)$ average**, $O(n)$ total for $n$ elements.
**Space: $O(n)$** in the worst case (all unique until the last).

A HashSet is ideal because:
- It supports $O(1)$ lookup and insert
- It maintains exactly the "seen" set
- It works with any data type that is hashable
- No need for sorting (which requires all data upfront)

</details>

---

| [← Unit 6: Longest Substring](../06-Classic-Hash-Problems/06-longest-substring-no-repeat.md) | [Next: Intersection of Arrays →](02-intersection-of-arrays.md) |
|:---|---:|

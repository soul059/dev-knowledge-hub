# Unit 12: Special Sorting Problems — Dutch National Flag

[← Previous: Unit 11 — Language Built-in Sorts](../11-Sorting-Algorithm-Comparison/06-language-builtin-sorts.md) | [Next: Sort by Frequency →](02-sort-by-frequency.md)

---

## Overview

The **Dutch National Flag Problem** (proposed by Edsger Dijkstra) is a classic partitioning problem: given an array with three distinct values (or categories), sort it in a single pass using O(1) extra space.

---

## Problem Statement

```
  Given an array containing only 0s, 1s, and 2s,
  sort it in-place in O(n) time and O(1) space.
  
  Input:  [2, 0, 1, 2, 1, 0, 0, 2, 1]
  Output: [0, 0, 0, 1, 1, 1, 2, 2, 2]
  
  Named after the Dutch flag: 🔴 Red (0), ⚪ White (1), 🔵 Blue (2)
```

---

## The Three-Pointer Approach

```
  Maintain three pointers:
  
  lo  → boundary between 0s and 1s (next position for 0)
  mid → current element being examined
  hi  → boundary between 1s and 2s (next position for 2)
  
  Invariant at all times:
  
  ┌──────────────────────────────────────────────────────────────┐
  │  [0...lo-1]    all 0s                                       │
  │  [lo...mid-1]  all 1s                                       │
  │  [mid...hi]    unexamined                                   │
  │  [hi+1...n-1]  all 2s                                       │
  └──────────────────────────────────────────────────────────────┘
  
  ┌──────┬──────┬──────────────┬──────┐
  │ 0 0 0│ 1 1 1│ ? ? ? ? ? ? │ 2 2 2│
  └──────┴──────┴──────────────┴──────┘
         ↑      ↑              ↑
        lo     mid             hi
```

### Algorithm

```
  while (mid <= hi):
      if arr[mid] == 0:
          swap(arr[lo], arr[mid])
          lo++,  mid++          ← 0 goes to left, advance both
      
      else if arr[mid] == 1:
          mid++                 ← 1 is already in the middle
      
      else:  // arr[mid] == 2
          swap(arr[mid], arr[hi])
          hi--                  ← 2 goes to right, DON'T advance mid!
                                   (swapped element is unexamined)
```

---

## Step-by-Step Trace

```
  Array: [2, 0, 1, 2, 1, 0]
  lo = 0, mid = 0, hi = 5
  
  Step 1: arr[mid]=2 → swap(arr[0], arr[5])
  [0, 0, 1, 2, 1, 2]  lo=0, mid=0, hi=4
  
  Step 2: arr[mid]=0 → swap(arr[0], arr[0])
  [0, 0, 1, 2, 1, 2]  lo=1, mid=1, hi=4
  
  Step 3: arr[mid]=0 → swap(arr[1], arr[1])
  [0, 0, 1, 2, 1, 2]  lo=2, mid=2, hi=4
  
  Step 4: arr[mid]=1 → mid++
  [0, 0, 1, 2, 1, 2]  lo=2, mid=3, hi=4
  
  Step 5: arr[mid]=2 → swap(arr[3], arr[4])
  [0, 0, 1, 1, 2, 2]  lo=2, mid=3, hi=3
  
  Step 6: arr[mid]=1 → mid++
  [0, 0, 1, 1, 2, 2]  lo=2, mid=4, hi=3
  
  mid > hi → DONE ✓
  Result: [0, 0, 1, 1, 2, 2]
```

---

## Java Implementation

```java
public class DutchNationalFlag {
    
    public static void sort012(int[] arr) {
        int lo = 0, mid = 0, hi = arr.length - 1;
        
        while (mid <= hi) {
            switch (arr[mid]) {
                case 0:
                    swap(arr, lo, mid);
                    lo++;
                    mid++;
                    break;
                case 1:
                    mid++;
                    break;
                case 2:
                    swap(arr, mid, hi);
                    hi--;
                    // DON'T increment mid — swapped element is unexamined
                    break;
            }
        }
    }
    
    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
    
    public static void main(String[] args) {
        int[] arr = {2, 0, 1, 2, 1, 0, 0, 2, 1};
        sort012(arr);
        System.out.println(java.util.Arrays.toString(arr));
        // [0, 0, 0, 1, 1, 1, 2, 2, 2]
    }
}
```

---

## Python Implementation

```python
def dutch_national_flag(arr):
    lo, mid, hi = 0, 0, len(arr) - 1
    
    while mid <= hi:
        if arr[mid] == 0:
            arr[lo], arr[mid] = arr[mid], arr[lo]
            lo += 1
            mid += 1
        elif arr[mid] == 1:
            mid += 1
        else:  # arr[mid] == 2
            arr[mid], arr[hi] = arr[hi], arr[mid]
            hi -= 1
    
    return arr
```

---

## C++ Implementation

```cpp
void sortColors(vector<int>& nums) {
    int lo = 0, mid = 0, hi = nums.size() - 1;
    
    while (mid <= hi) {
        if (nums[mid] == 0) {
            swap(nums[lo], nums[mid]);
            lo++; mid++;
        } else if (nums[mid] == 1) {
            mid++;
        } else {
            swap(nums[mid], nums[hi]);
            hi--;
        }
    }
}
```

---

## Why NOT Increment `mid` After Case 2?

```
  When arr[mid] == 2:
  We swap arr[mid] with arr[hi].
  
  The element that was at arr[hi] is now at arr[mid].
  We DON'T KNOW what this element is — it could be 0, 1, or 2.
  So we must examine arr[mid] again in the next iteration.
  
  When arr[mid] == 0:
  We swap arr[lo] with arr[mid].
  The element that was at arr[lo] is now at arr[mid].
  This element was in the [lo...mid-1] region → it's a 1.
  So we can safely advance mid.
  
  Exception: if lo == mid (beginning), swapped element is itself.
  Still safe to advance mid.
```

---

## Generalized: K-Way Partition

For more than 3 values, the DNF approach extends but becomes more complex:

```
  4-way partition (values 0, 1, 2, 3):
  Use TWO passes of 3-way partition:
  
  Pass 1: Partition around 1.5 → [0,1 | 2,3]
  Pass 2a: Partition left half around 0.5 → [0 | 1]  
  Pass 2b: Partition right half around 2.5 → [2 | 3]
  
  Total: O(n) time, O(1) space, 2 passes
```

---

## Applications

```
  1. LeetCode 75: Sort Colors (exact same problem)
  
  2. Quick Sort: Three-way partition
     [< pivot | = pivot | > pivot]
     Avoids O(n²) on arrays with many duplicates
  
  3. Database queries: GROUP BY with 3 categories
  
  4. Image processing: Classify pixels into 3 brightness levels
  
  5. Task scheduling: Priority levels (High, Medium, Low)
```

---

## Complexity Analysis

```
  Time:  O(n) — each element examined at most twice
         (once as mid, once if swapped from hi)
  
  Space: O(1) — only three pointer variables
  
  Stable? No — swaps across long distances
  
  ┌─────────────────────────────────────────┐
  │  Number of swaps: at most n            │
  │  Number of comparisons: exactly n      │
  │  (one comparison per mid increment)    │
  └─────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Sort array of 0s, 1s, 2s |
| Algorithm | Three-pointer (lo, mid, hi) |
| Time | O(n) — single pass |
| Space | O(1) |
| Stable | No |
| Key insight | Don't advance mid after swapping with hi |

---

## Quick Revision Questions

1. **Why can't you advance `mid` after swapping with `hi` but you can after swapping with `lo`?**
2. **What invariant do lo, mid, and hi maintain throughout the algorithm?**
3. **Trace the DNF algorithm on [1, 0, 2, 1, 0, 2, 1, 0].**
4. **How would you extend this to 4 distinct values?**
5. **How is the DNF problem related to Quick Sort's three-way partition?**

---

[← Previous: Unit 11 — Language Built-in Sorts](../11-Sorting-Algorithm-Comparison/06-language-builtin-sorts.md) | [Next: Sort by Frequency →](02-sort-by-frequency.md)

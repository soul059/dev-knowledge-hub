# Chapter 1: Interpolation Search — Concept

[← Previous Unit: Exponential Practice](../09-Exponential-Search/03-practice-problems.md) | [Next: Uniform Distribution →](02-uniform-distribution.md)

---

## Overview

**Interpolation search** improves on binary search by making an **intelligent guess** about where the target lies, based on the values at the boundaries. Instead of always looking at the middle, it estimates the target's position using **linear interpolation** — like how we look up a word in the dictionary.

---

## The Dictionary Analogy

```
   How do you search for "Zebra" in a dictionary?
   
   Binary search approach:
   ┌──────────────────────────────────────────┐
   │  Always open to the MIDDLE page first.   │
   │  Then middle of the right half.          │
   │  Then middle again...                    │
   │  Doesn't use any knowledge of letters!   │
   └──────────────────────────────────────────┘
   
   Human approach (interpolation):
   ┌──────────────────────────────────────────┐
   │  "Z" is near the END of the alphabet.    │
   │  Open near the LAST pages directly!      │
   │  Adjust from there.                      │
   │  Uses VALUE to estimate POSITION.        │
   └──────────────────────────────────────────┘
```

---

## The Formula

```
   Binary search:  mid = lo + (hi - lo) / 2      ← always halves
   
   Interpolation:  mid = lo + (target - arr[lo])
                             ─────────────────── × (hi - lo)
                             (arr[hi] - arr[lo])
   
   ┌──────────────────────────────────────────────────────┐
   │                                                      │
   │  pos = lo + (target - arr[lo]) * (hi - lo)           │
   │             ─────────────────────────────             │
   │                 (arr[hi] - arr[lo])                   │
   │                                                      │
   │  This is LINEAR INTERPOLATION:                       │
   │  "How far is target between arr[lo] and arr[hi]?"    │
   │  "Place probe at same fraction of [lo, hi]."        │
   │                                                      │
   └──────────────────────────────────────────────────────┘
```

---

## Visual Comparison: Binary vs Interpolation

```
   arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
   target = 85
   
   Binary search:
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬────┐
   │10 │20 │30 │40 │50 │60 │70 │80 │90 │100 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴────┘
     0   1   2   3   4   5   6   7   8    9
                     ↑               ↑    ↑
                   mid=4           mid=7  mid=8
                   (1st)           (2nd)  (3rd)
                   → 3 steps
   
   Interpolation search:
   pos = 0 + (85-10)/(100-10) × (9-0) = 75/90 × 9 = 7.5 → 7
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬────┐
   │10 │20 │30 │40 │50 │60 │70 │80 │90 │100 │
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴────┘
     0   1   2   3   4   5   6   7   8    9
                                 ↑   ↑
                               pos=7 pos=8
                               (1st) (2nd)
                               → 2 steps!
```

---

## Pseudocode

```
   INTERPOLATION-SEARCH(arr, n, target):
       lo = 0
       hi = n - 1
       
       while lo <= hi AND target >= arr[lo] AND target <= arr[hi]:
           // Guard against division by zero
           if arr[lo] == arr[hi]:
               if arr[lo] == target: return lo
               else: break
           
           // Interpolation formula
           pos = lo + ((target - arr[lo]) * (hi - lo))
                      / (arr[hi] - arr[lo])
           
           if arr[pos] == target:
               return pos
           elif arr[pos] < target:
               lo = pos + 1
           else:
               hi = pos - 1
       
       return -1
```

---

## Step-by-Step Trace

```
   arr = [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
   target = 14
   
   Iteration 1:
   lo=0, hi=9, arr[lo]=2, arr[hi]=20
   pos = 0 + (14-2)/(20-2) × (9-0)
       = 0 + 12/18 × 9
       = 0 + 6.0
       = 6
   arr[6] = 14 = target → FOUND in 1 step! ✓
   
   Compare: Binary search would take 3 steps
   (mid=4→10<14, mid=7→16>14, mid=5→12<14, mid=6→14✓)
```

---

## Implementation (Java)

```java
public int interpolationSearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    
    while (lo <= hi && target >= arr[lo] && target <= arr[hi]) {
        // Division by zero guard
        if (arr[lo] == arr[hi]) {
            if (arr[lo] == target) return lo;
            else break;
        }
        
        // Interpolation formula
        int pos = lo + ((long)(target - arr[lo]) * (hi - lo))
                       / (arr[hi] - arr[lo]);
        
        if (arr[pos] == target) return pos;
        else if (arr[pos] < target) lo = pos + 1;
        else hi = pos - 1;
    }
    return -1;
}
```

---

## Implementation (Python)

```python
def interpolation_search(arr, target):
    lo, hi = 0, len(arr) - 1
    
    while lo <= hi and target >= arr[lo] and target <= arr[hi]:
        if arr[lo] == arr[hi]:
            if arr[lo] == target:
                return lo
            break
        
        # Interpolation formula
        pos = lo + ((target - arr[lo]) * (hi - lo)) // (arr[hi] - arr[lo])
        
        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            lo = pos + 1
        else:
            hi = pos - 1
    
    return -1
```

---

## Implementation (C++)

```cpp
int interpolationSearch(vector<int>& arr, int target) {
    int lo = 0, hi = (int)arr.size() - 1;
    
    while (lo <= hi && target >= arr[lo] && target <= arr[hi]) {
        if (arr[lo] == arr[hi]) {
            if (arr[lo] == target) return lo;
            else break;
        }
        
        // Use long long to prevent overflow
        int pos = lo + (long long)(target - arr[lo]) * (hi - lo)
                       / (arr[hi] - arr[lo]);
        
        if (arr[pos] == target) return pos;
        else if (arr[pos] < target) lo = pos + 1;
        else hi = pos - 1;
    }
    return -1;
}
```

---

## Key Conditions in the While Loop

```
   while lo <= hi                     ← Standard: valid range
     AND target >= arr[lo]            ← Target is not below range
     AND target <= arr[hi]            ← Target is not above range

   These extra conditions are ESSENTIAL:
   ┌─────────────────────────────────────────────────┐
   │ Without them, the interpolation formula can     │
   │ produce positions OUTSIDE [lo, hi], causing     │
   │ array index out-of-bounds errors!               │
   │                                                 │
   │ If target < arr[lo] or target > arr[hi],        │
   │ the formula gives pos < lo or pos > hi.         │
   └─────────────────────────────────────────────────┘
```

---

## Overflow Pitfall

```
   The formula: (target - arr[lo]) * (hi - lo)
   
   Both factors can be up to ~2×10⁹ for int arrays.
   Product can overflow int (max ~2.1×10⁹).
   
   Solution: Cast to long/long long BEFORE multiplication.
   
   Java:   (long)(target - arr[lo]) * (hi - lo)
   C++:    (long long)(target - arr[lo]) * (hi - lo)
   Python: No issue (arbitrary precision integers)
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Core idea | Estimate position using value proportions |
| Formula | lo + (target-arr[lo])/(arr[hi]-arr[lo]) × (hi-lo) |
| Best case | O(1) — direct hit |
| Average (uniform) | O(log log n) |
| Worst case | O(n) — skewed distribution |
| Space | O(1) |
| Prerequisite | Sorted array, ideally uniformly distributed |

---

## Quick Revision Questions

1. **What is the interpolation formula?**
2. **Why do we need `target >= arr[lo] && target <= arr[hi]` in the loop?**
3. **How is interpolation search like looking up a word in a dictionary?**
4. **What happens with integer overflow and how do we prevent it?**
5. **What does `arr[lo] == arr[hi]` mean and why must we handle it?**

---

[← Previous Unit: Exponential Practice](../09-Exponential-Search/03-practice-problems.md) | [Next: Uniform Distribution →](02-uniform-distribution.md)

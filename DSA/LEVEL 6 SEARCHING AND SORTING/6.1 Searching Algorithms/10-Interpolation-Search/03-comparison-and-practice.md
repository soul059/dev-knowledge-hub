# Chapter 3: Comparison & Practice Problems

[← Previous: Uniform Distribution](02-uniform-distribution.md) | [Back to README →](../README.md)

---

## The Grand Search Algorithm Comparison

```
   ┌─────────────────────┬──────────────┬──────────────┬──────────────┐
   │ Algorithm           │ Best Case    │ Average Case │ Worst Case   │
   ├─────────────────────┼──────────────┼──────────────┼──────────────┤
   │ Linear Search       │ O(1)         │ O(n)         │ O(n)         │
   │ Binary Search       │ O(1)         │ O(log n)     │ O(log n)     │
   │ Ternary Search      │ O(1)         │ O(log n)     │ O(log n)     │
   │ Exponential Search  │ O(1)         │ O(log i)     │ O(log n)     │
   │ Interpolation Search│ O(1)         │ O(log log n)*│ O(n)         │
   └─────────────────────┴──────────────┴──────────────┴──────────────┘
   * Only for uniformly distributed data
```

---

## Detailed Feature Comparison

```
   ┌─────────────────────┬──────────┬──────────┬──────────┬──────────┐
   │ Feature             │ Binary   │ Ternary  │ Exponent.│ Interp.  │
   ├─────────────────────┼──────────┼──────────┼──────────┼──────────┤
   │ Needs sorted data?  │ Yes      │ Yes      │ Yes      │ Yes      │
   │ Needs random access?│ Yes      │ Yes      │ Yes      │ Yes      │
   │ Needs array size?   │ Yes      │ Yes      │ No ✓     │ Yes      │
   │ Distribution agnost.│ Yes ✓    │ Yes ✓    │ Yes ✓    │ No ✗     │
   │ Handles duplicates? │ Yes      │ Yes      │ Yes      │ Tricky   │
   │ Deterministic?      │ Yes      │ Yes      │ Yes      │ Yes      │
   │ Comparison count    │ ~log n   │ ~2 log n │ ~2 log i │ ~log log│
   │ Cache behavior      │ Good     │ Good     │ Varies   │ Poor     │
   │ Parallelizable?     │ No       │ Maybe    │ No       │ No       │
   │ Implementation ease │ Easy     │ Medium   │ Easy     │ Medium   │
   └─────────────────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## Decision Flowchart

```
   Is the data sorted?
   │
   ├── No → LINEAR SEARCH (or sort first)
   │
   └── Yes → Do you know the array size?
              │
              ├── No → EXPONENTIAL SEARCH
              │
              └── Yes → Is the data uniformly distributed?
                        │
                        ├── Yes → INTERPOLATION SEARCH
                        │         (O(log log n) average)
                        │
                        └── No/Unknown → Is the function unimodal?
                                         │
                                         ├── Yes → TERNARY SEARCH
                                         │         (for max/min)
                                         │
                                         └── No → BINARY SEARCH ✓
                                                   (safest default)
```

---

## When to Use Which — Quick Reference

| Scenario | Best Algorithm | Why |
|----------|---------------|-----|
| General sorted search | Binary | Reliable O(log n), easy to implement |
| Unsorted data | Linear | No other option |
| Unknown array size | Exponential | Doesn't need n |
| Target near beginning | Exponential | O(log i) adapts |
| Uniformly distributed | Interpolation | O(log log n) |
| Find peak/valley | Ternary | Works on unimodal functions |
| Small array (<16) | Linear | Lower overhead |
| Database index lookup | Interpolation | Keys often uniform |
| Competitive programming | Binary | Reliable, familiar |

---

## Practice Problems

### Problem 1: Implement Interpolation Search ⭐

```
   Input:  arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100], target = 70
   Output: 6
   
   Verify: pos = 0 + (70-10)/(100-10) × 9 = 60/90 × 9 = 6.0 → pos=6
   arr[6] = 70 = target → found in 1 step!
```

---

### Problem 2: Compare Step Counts ⭐⭐

```
   For arr = [1, 2, 3, ..., 1000000] (1 million elements):
   Count the number of iterations for each algorithm
   when searching for target = 750000.
   
   Binary Search:
   log₂(1000000) ≈ 20 steps
   
   Interpolation Search (uniform data):
   Step 1: pos = (750000-1)/(1000000-1) × 999999 ≈ 749999
   arr[749999] = 750000 → FOUND in 1 step!
   
   For perfect uniform data, interpolation often finds
   the target in just 1-2 steps regardless of n!
```

---

### Problem 3: Worst Case Construction ⭐⭐

```
   Construct an array of 10 elements where interpolation
   search takes the maximum number of steps.
   
   Answer: Use exponential growth
   arr = [1, 2, 4, 8, 16, 32, 64, 128, 256, 512]
   target = 3
   
   Trace:
   lo=0, hi=9: pos = (3-1)/(512-1) × 9 = 0.035 → pos=0
   arr[0]=1 < 3 → lo=1
   
   lo=1, hi=9: pos = 1 + (3-2)/(512-2) × 8 = 1.016 → pos=1
   arr[1]=2 < 3 → lo=2
   
   lo=2, hi=9: pos = 2 + (3-4)/(512-4) × 7 → negative offset
   target < arr[lo]=4 → NOT FOUND (correctly)
   
   Even worse: arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10^9]
   target = 9: Formula always guesses pos ≈ 0 → linear scan
```

---

### Problem 4: Hybrid Search ⭐⭐

```
   Implement a search that:
   1. Detects if data is roughly uniform
   2. Uses interpolation search if uniform
   3. Uses binary search otherwise
```

```python
def smart_search(arr, target):
    n = len(arr)
    if n == 0:
        return -1
    
    # Quick uniformity check using 5 sample points
    uniform = True
    if n > 10:
        expected_step = (arr[-1] - arr[0]) / (n - 1)
        if expected_step > 0:
            for frac in [0.25, 0.5, 0.75]:
                idx = int(frac * (n - 1))
                expected_val = arr[0] + expected_step * idx
                actual_val = arr[idx]
                if abs(actual_val - expected_val) > expected_step * 5:
                    uniform = False
                    break
    
    if uniform:
        return interpolation_search(arr, target)
    else:
        return binary_search(arr, target)

def interpolation_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi and target >= arr[lo] and target <= arr[hi]:
        if arr[lo] == arr[hi]:
            return lo if arr[lo] == target else -1
        pos = lo + ((target - arr[lo]) * (hi - lo)) // (arr[hi] - arr[lo])
        if arr[pos] == target: return pos
        elif arr[pos] < target: lo = pos + 1
        else: hi = pos - 1
    return -1

def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if arr[mid] == target: return mid
        elif arr[mid] < target: lo = mid + 1
        else: hi = mid - 1
    return -1
```

---

### Problem 5: Interpolation on Strings ⭐⭐⭐

```
   Can we apply interpolation search to sorted strings?
   
   Idea: Convert first few characters to a numerical value
   and use that for interpolation.
   
   Example: Searching a dictionary for "python"
   "a" → 0, "z" → 25
   "p" → 15/25 = 0.60 → Probe at 60% of the way
   
   This is actually how humans search dictionaries!
```

```python
def string_to_numeric(s, depth=2):
    """Convert first 'depth' chars to a number for interpolation."""
    val = 0
    for i in range(min(depth, len(s))):
        val = val * 26 + (ord(s[i].lower()) - ord('a'))
    # Pad if string is shorter than depth
    for i in range(len(s), depth):
        val = val * 26
    return val

def interpolation_search_strings(arr, target):
    lo, hi = 0, len(arr) - 1
    
    while lo <= hi:
        if lo == hi:
            return lo if arr[lo] == target else -1
        
        t_val = string_to_numeric(target)
        lo_val = string_to_numeric(arr[lo])
        hi_val = string_to_numeric(arr[hi])
        
        if lo_val == hi_val:
            return lo if arr[lo] == target else -1
        
        frac = (t_val - lo_val) / (hi_val - lo_val)
        pos = lo + int(frac * (hi - lo))
        pos = max(lo, min(hi, pos))  # Clamp
        
        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            lo = pos + 1
        else:
            hi = pos - 1
    return -1
```

---

### Problem 6: Search in Log File by Timestamp ⭐⭐⭐

```
   Real-world scenario:
   A log file has millions of entries sorted by timestamp.
   Find all entries between two timestamps.
   
   Timestamps are roughly uniformly distributed (events
   occur at a steady rate), making interpolation ideal.
```

```python
from datetime import datetime

def find_range_in_logs(logs, start_time, end_time):
    """
    Each log entry is (timestamp, message).
    Find all entries between start_time and end_time.
    """
    def timestamp_to_num(t):
        return t.timestamp()  # Convert to Unix epoch
    
    n = len(logs)
    
    # Find start index using interpolation
    lo, hi = 0, n - 1
    start_idx = n  # Default: nothing found
    start_num = timestamp_to_num(start_time)
    
    while lo <= hi:
        lo_num = timestamp_to_num(logs[lo][0])
        hi_num = timestamp_to_num(logs[hi][0])
        
        if lo_num == hi_num:
            if lo_num >= start_num:
                start_idx = lo
            break
        
        frac = (start_num - lo_num) / (hi_num - lo_num)
        frac = max(0, min(1, frac))
        pos = lo + int(frac * (hi - lo))
        
        if timestamp_to_num(logs[pos][0]) >= start_num:
            start_idx = pos
            hi = pos - 1
        else:
            lo = pos + 1
    
    # Similarly find end index
    # ... (analogous interpolation for end_time)
    
    return logs[start_idx:end_idx + 1]
```

---

## All Search Algorithms — Final Summary

```
   ╔═══════════════════════════════════════════════════════════════╗
   ║              SEARCH ALGORITHMS MASTER REFERENCE              ║
   ╠═══════════════════════════════════════════════════════════════╣
   ║                                                              ║
   ║  LINEAR SEARCH          O(n) all cases                       ║
   ║  └─ Use for: unsorted, small arrays, linked lists            ║
   ║                                                              ║
   ║  BINARY SEARCH          O(log n) guaranteed                  ║
   ║  └─ Use for: sorted arrays (DEFAULT choice)                  ║
   ║     ├─ Variations: first/last occurrence, floor/ceiling      ║
   ║     ├─ Templates: standard, boundary-finding, left/right     ║
   ║     ├─ On answer: feasibility + monotonicity → BS answer     ║
   ║     └─ In matrix: 1D mapping or staircase approach           ║
   ║                                                              ║
   ║  TERNARY SEARCH         O(log n) with more comparisons       ║
   ║  └─ Use for: unimodal functions (find peak/valley)           ║
   ║                                                              ║
   ║  EXPONENTIAL SEARCH     O(log i) where i = target position   ║
   ║  └─ Use for: unbounded arrays, target near start             ║
   ║                                                              ║
   ║  INTERPOLATION SEARCH   O(log log n) avg, O(n) worst         ║
   ║  └─ Use for: uniformly distributed sorted data               ║
   ║                                                              ║
   ╠═══════════════════════════════════════════════════════════════╣
   ║  Key Takeaway: Binary search is the SAFE DEFAULT.            ║
   ║  Others excel in specific scenarios.                         ║
   ╚═══════════════════════════════════════════════════════════════╝
```

---

## Problem Summary Table

| # | Problem | Difficulty | Key Concept |
|---|---------|-----------|-------------|
| 1 | Basic interpolation search | ⭐ | Formula application |
| 2 | Compare step counts | ⭐⭐ | Performance analysis |
| 3 | Worst case construction | ⭐⭐ | Distribution awareness |
| 4 | Hybrid smart search | ⭐⭐ | Algorithm selection |
| 5 | Interpolation on strings | ⭐⭐⭐ | Domain adaptation |
| 6 | Log file timestamp search | ⭐⭐⭐ | Real-world application |

---

## Comprehensive Practice Checklist

```
   LINEAR SEARCH
   □ Implement basic linear search
   □ Implement sentinel search
   □ Search from both ends simultaneously
   
   BINARY SEARCH
   □ Iterative binary search
   □ Recursive binary search
   □ Find first/last occurrence
   □ Count occurrences
   □ Search insert position
   □ Floor and ceiling
   □ Peak element
   □ Search in rotated array (with & without duplicates)
   □ Find minimum in rotated array
   
   BINARY SEARCH ON ANSWER
   □ Square root (integer & floating point)
   □ Koko eating bananas / Ship packages
   □ Split array largest sum
   □ Aggressive cows / Magnetic balls
   
   MATRIX SEARCH
   □ Search sorted matrix (fully sorted)
   □ Search row-column sorted matrix (staircase)
   □ Kth smallest in sorted matrix
   
   ADVANCED SEARCH
   □ Ternary search on unimodal function
   □ Exponential search for unknown-size array
   □ Interpolation search on uniform data
   □ Hybrid search with distribution detection
```

---

## Quick Revision Questions

1. **Rank all 5 search algorithms by worst-case complexity (best to worst).**
2. **Which algorithm is the safest default choice and why?**
3. **When would you choose interpolation over binary search?**
4. **What is the key disadvantage of interpolation search?**
5. **If an interview asks "which search algorithm?", what should be your default answer and when would you suggest alternatives?**
6. **Summarize each algorithm in ONE sentence.**

---

## Exam / Interview Quick Reference

```
   Q: "What search algorithms do you know?"
   A:
   1. Linear Search     — O(n), works on anything
   2. Binary Search     — O(log n), sorted arrays
   3. Ternary Search    — O(log n), unimodal functions
   4. Exponential Search — O(log i), unknown-size arrays
   5. Interpolation     — O(log log n)*, uniform distribution
   
   Q: "Which one should I use?"
   A: Binary search. Always binary search. Unless you have
      a specific reason to use something else.
   
   Q: "What makes binary search so important?"
   A: It's the foundation for:
      - Finding boundaries (lower_bound, upper_bound)
      - Binary search on answer (optimization problems)
      - Decision problems (feasibility checking)
      - Divide and conquer paradigm
```

---

**Congratulations! You've completed the Searching Algorithms unit!** 🎉

Go back to the [Main README](../README.md) to review all units.

---

[← Previous: Uniform Distribution](02-uniform-distribution.md) | [Back to README →](../README.md)

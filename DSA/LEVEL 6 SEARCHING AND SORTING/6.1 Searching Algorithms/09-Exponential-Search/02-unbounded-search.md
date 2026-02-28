# Chapter 2: Searching in Unbounded / Infinite Arrays

[← Previous: Concept](01-concept.md) | [Next: Practice Problems →](03-practice-problems.md)

---

## The Problem

```
   You have an INFINITE sorted array (or an array of unknown size).
   Find a given target element.
   
   Constraints:
   - Array is sorted in ascending order
   - Array length is UNKNOWN (or effectively infinite)
   - You can access arr[i] for any valid index i
   - If you go out of bounds, you get ∞ (or an exception)
   
   Why can't we use binary search directly?
   → Binary search needs hi = n-1, but we DON'T KNOW n!
```

---

## The Approach: Exponential Bound Finding

```
   Step 1: Find a range [lo, hi] that contains the target
   ──────────────────────────────────────────────────────
   
   Start with hi = 1
   While arr[hi] < target:
       hi = hi * 2
   
   Set lo = hi / 2
   
     ...  arr[1]  arr[2]  arr[4]  arr[8]  arr[16]  arr[32] ...
            ↑       ↑       ↑       ↑       ↑
           1st     2nd     3rd     4th     5th check
   
   When arr[hi] ≥ target → range is [hi/2, hi]
   
   Step 2: Binary search within [lo, hi]
   ──────────────────────────────────────
   Standard binary search on a known range
```

---

## Visual Example

```
   Infinite sorted array:
   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬─────
   │ 1 │ 3 │ 5 │ 7 │ 9 │11 │13 │15 │17 │19 │21 │23 │ ...
   └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴─────
     0   1   2   3   4   5   6   7   8   9  10  11  
   
   target = 13
   
   Doubling phase:
     hi=1: arr[1]=3  < 13 → double
     hi=2: arr[2]=5  < 13 → double
     hi=4: arr[4]=9  < 13 → double
     hi=8: arr[8]=17 ≥ 13 → STOP!
   
   Binary search in [4, 8]:
   ┌───┬───┬───┬───┬───┐
   │ 9 │11 │13 │15 │17 │
   └───┴───┴───┴───┴───┘
     4   5   6   7   8
   
   mid=6: arr[6]=13 = target → FOUND at index 6 ✓
   
   Total: 4 doubling steps + 2 BS steps = 6 steps
   O(log 6) ≈ 2.6, so ~6 is reasonable
```

---

## Implementation (Java)

```java
/**
 * Search in an infinite sorted array.
 * Assume arr.get(i) returns Integer.MAX_VALUE for out-of-bounds i.
 */
public int searchInfiniteArray(int[] arr, int target) {
    // Edge case
    if (arr[0] == target) return 0;
    
    // Phase 1: Find bounds
    int lo = 0, hi = 1;
    while (arr[hi] < target) {
        lo = hi;
        hi *= 2;
    }
    
    // Phase 2: Binary search in [lo, hi]
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

---

## Implementation (Python)

```python
def search_infinite_array(arr, target):
    """
    Search in a sorted array of unknown size.
    arr supports arr[i], returns float('inf') for out-of-bounds.
    """
    if arr[0] == target:
        return 0
    
    # Phase 1: Find bounds
    lo, hi = 0, 1
    while arr[hi] < target:
        lo = hi
        hi *= 2
    
    # Phase 2: Binary search
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

---

## Implementation (C++)

```cpp
// Using ArrayReader interface (LeetCode style)
// reader.get(i) returns INT_MAX if out of bounds
int searchInfiniteArray(const ArrayReader& reader, int target) {
    if (reader.get(0) == target) return 0;
    
    int lo = 0, hi = 1;
    while (reader.get(hi) < target) {
        lo = hi;
        hi *= 2;
    }
    
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = reader.get(mid);
        if (val == target) return mid;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

---

## LeetCode 702: Search in a Sorted Array of Unknown Size

```
   Problem:
   Given an ArrayReader interface with get(index) method,
   find target in an ascending integer array.
   
   Constraints:
   - get(index) returns -1 if index is out of bounds *
   - 1 ≤ array length ≤ 10^4
   - -10^4 ≤ values ≤ 10^4
   - Array is sorted, all elements unique
   
   * Note: LeetCode uses 2^31-1 for out-of-bounds indicator
```

### Optimal Solution

```python
class Solution:
    def search(self, reader: 'ArrayReader', target: int) -> int:
        # Phase 1: Find upper bound
        hi = 1
        while reader.get(hi) < target:
            hi *= 2
        
        # Phase 2: Binary search
        lo = hi // 2
        while lo <= hi:
            mid = lo + (hi - lo) // 2
            val = reader.get(mid)
            if val == target:
                return mid
            elif val == 2147483647:  # Out of bounds
                hi = mid - 1
            elif val < target:
                lo = mid + 1
            else:
                hi = mid - 1
        return -1
```

---

## Handling Out-of-Bounds Access

```
   Strategy 1: Sentinel value
   ──────────────────────────
   arr[out_of_bounds] returns ∞ or MAX_VALUE
   → Naturally larger than any target, BS moves left
   
   Strategy 2: Exception handling
   ──────────────────────────────
   try:
       val = arr[index]
   except IndexError:
       val = float('inf')
   
   Strategy 3: Known upper limit
   ─────────────────────────────
   Clamp hi to a maximum possible value
   hi = min(hi, MAX_POSSIBLE_INDEX)
```

---

## Applications Beyond Arrays

### 1. Finding the First True in an Infinite Boolean Sequence

```
   Sequence: F F F F F F F F F F T T T T T T ...
   Find the first T.
   
   Use exponential search to find a T, then binary search
   to find the first T in the identified range.
```

### 2. Finding the Boundary of a Monotonic Function

```
   f(x) is monotonically increasing, find x where f(x) ≥ k
   
   Double x until f(x) ≥ k, then binary search.
```

### 3. Optimized List Operations (Skip Lists, Merge in TimSort)

```
   Java's TimSort uses "galloping" (a form of exponential search)
   to speed up merging when one run consistently has smaller elements.
   
   When consecutive wins from one side exceed a threshold:
     Switch from pairwise comparison to galloping/exponential search
     to quickly find how far the winning streak extends.
```

---

## Complexity Comparison

```
   ┌──────────────────┬──────────────┬───────────────────────┐
   │ Algorithm        │ Time         │ Works on Unknown     │
   │                  │              │ Size Array?          │
   ├──────────────────┼──────────────┼───────────────────────┤
   │ Linear Search    │ O(i)         │ ✓ (but slow)        │
   │ Binary Search    │ O(log n)     │ ✗ (needs n)         │
   │ Exponential      │ O(log i)     │ ✓ (ideal!)          │
   │ Interpolation    │ O(log log n) │ ✗ (needs n + dist)  │
   └──────────────────┴──────────────┴───────────────────────┘
   
   Where i = position of target, n = array size
   
   For unknown-size arrays, exponential search is the
   OPTIMAL choice: O(log i) without needing to know n.
```

---

## Proof of Optimality

```
   Claim: Exponential search is at most 2× the optimal
   number of comparisons for unknown-size search.
   
   Proof sketch:
   - Any algorithm must make Ω(log i) comparisons
     to distinguish position i from any other position.
   - Exponential search makes:
     - ⌈log₂ i⌉ doubling comparisons
     - ⌈log₂ i⌉ binary search comparisons
     - Total: 2⌈log₂ i⌉ = O(log i) comparisons
   
   Therefore, it's within a factor of 2 of optimal.
```

---

## Galloping Search Variant

```
   Galloping search is exponential search used in MERGE contexts.
   
   In TimSort (Java/Python's default sort):
   
   Merging two sorted runs A and B:
   
   If A[0] < B[0] < B[1] < ... < B[k] < A[1]:
     Normal merge does k+1 comparisons for B elements.
     Galloping: exponentially probe B to find where A[1] goes.
     
   gallop(key, array, base, len, hint):
       // Start at hint, gallop right
       lastOfs = 0
       ofs = 1
       while ofs < len and array[base + ofs] < key:
           lastOfs = ofs
           ofs = 2 * ofs + 1  // exponential growth
       // Binary search in [lastOfs, ofs]
```

---

## Summary

| Aspect | Detail |
|--------|--------|
| Core idea | Double bounds to find range, then binary search |
| Time | O(log i) where i = target position |
| Space | O(1) |
| Key advantage | Works without knowing array size |
| LeetCode | #702 (Search in Unknown Size Array) |
| Real-world use | TimSort galloping, skip lists |

---

## Quick Revision Questions

1. **Why can't we use binary search directly on an infinite array?**
2. **What is the doubling pattern used to find bounds?**
3. **If target is at index 100, how many doubling steps are needed?**
4. **What is galloping search and where is it used?**
5. **How does exponential search handle out-of-bounds accesses?**
6. **What is the competitive ratio of exponential vs optimal search?**

---

[← Previous: Concept](01-concept.md) | [Next: Practice Problems →](03-practice-problems.md)

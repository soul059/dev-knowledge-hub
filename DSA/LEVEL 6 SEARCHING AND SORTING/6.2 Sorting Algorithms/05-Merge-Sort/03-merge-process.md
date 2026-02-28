# Unit 5: Merge Sort — The Merge Process Deep Dive

[← Previous: Implementation](02-implementation.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)

---

## Overview

The **merge** operation is the heart of Merge Sort. Understanding it deeply is crucial because it's not just used in Merge Sort — it appears in many algorithmic problems (merge k sorted lists, external sorting, inversion counting, etc.).

---

## Two-Pointer Merge Technique

```
  The merge of two sorted arrays uses two pointers,
  one for each array, advancing the smaller element.
  
  Left:   [3, 12, 25, 40]    Right: [7, 15, 30]
           ↑ i=0                      ↑ j=0
  Result: [ ]
  
  ┌──────────────────────────────────────────────────────────┐
  │  INVARIANT: Everything already placed in Result is       │
  │  sorted, and all remaining elements in Left and Right    │
  │  are ≥ everything in Result.                             │
  └──────────────────────────────────────────────────────────┘
  
  Compare L[i] vs R[j]:
  
  Step 1: 3 < 7   → take 3     Result: [3]
  Step 2: 12 > 7  → take 7     Result: [3, 7]
  Step 3: 12 < 15 → take 12    Result: [3, 7, 12]
  Step 4: 25 > 15 → take 15    Result: [3, 7, 12, 15]
  Step 5: 25 < 30 → take 25    Result: [3, 7, 12, 15, 25]
  Step 6: 40 > 30 → take 30    Result: [3, 7, 12, 15, 25, 30]
  Step 7: Left has 40, Right exhausted → copy 40
                                Result: [3, 7, 12, 15, 25, 30, 40] ✓
```

---

## Visual Animation of Merge

```
  Merging: L = [1, 4, 7, 9]   R = [2, 5, 6, 8]
  
  ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐
  │ 1 │ 4 │ 7 │ 9 │   │ 2 │ 5 │ 6 │ 8 │
  └─↑─┴───┴───┴───┘   └─↑─┴───┴───┴───┘
    i                     j
  Result: ┌───┐
          │ 1 │  (1 < 2, take from L)
          └───┘
  
  ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐
  │ 1 │ 4 │ 7 │ 9 │   │ 2 │ 5 │ 6 │ 8 │
  └───┴─↑─┴───┴───┘   └─↑─┴───┴───┴───┘
        i                 j
  Result: ┌───┬───┐
          │ 1 │ 2 │  (4 > 2, take from R)
          └───┴───┘
  
  ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐
  │ 1 │ 4 │ 7 │ 9 │   │ 2 │ 5 │ 6 │ 8 │
  └───┴─↑─┴───┴───┘   └───┴─↑─┴───┴───┘
        i                     j
  Result: ┌───┬───┬───┐
          │ 1 │ 2 │ 4 │  (4 < 5, take from L)
          └───┴───┴───┘
  
  ┌───┬───┬───┬───┐   ┌───┬───┬───┬───┐
  │ 1 │ 4 │ 7 │ 9 │   │ 2 │ 5 │ 6 │ 8 │
  └───┴───┴─↑─┴───┘   └───┴─↑─┴───┴───┘
            i                 j
  Result: ┌───┬───┬───┬───┐
          │ 1 │ 2 │ 4 │ 5 │  (7 > 5, take from R)
          └───┴───┴───┴───┘
  
  Continuing: 6 < 7 → take 6, 7 < 8 → take 7, 8 < 9 → take 8, copy 9
  
  Result: ┌───┬───┬───┬───┬───┬───┬───┬───┐
          │ 1 │ 2 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │  ✓
          └───┴───┴───┴───┴───┴───┴───┴───┘
```

---

## Merge Comparison Count Analysis

```
  How many comparisons does merge do?
  
  Left (size m), Right (size n) → merged result (size m+n)
  
  ┌─────────────────────────────────────────────────────────┐
  │  BEST CASE: All of one array < all of other            │
  │                                                         │
  │  L = [1, 2, 3]    R = [4, 5, 6]                       │
  │  Compare: 1<4, 2<4, 3<4 → Left exhausted              │
  │  Copy remaining: 4, 5, 6                               │
  │  Comparisons = min(m, n) = 3                            │
  ├─────────────────────────────────────────────────────────┤
  │  WORST CASE: Elements interleave perfectly             │
  │                                                         │
  │  L = [1, 3, 5]    R = [2, 4, 6]                       │
  │  Compare: 1<2, 3>2, 3<4, 5>4, 5<6                     │
  │  Then copy: 6                                           │
  │  Comparisons = m + n - 1 = 5                            │
  ├─────────────────────────────────────────────────────────┤
  │  RANGE: min(m,n) ≤ comparisons ≤ m + n - 1            │
  └─────────────────────────────────────────────────────────┘
```

---

## Stability in Merge — Critical Detail

```
  WHY does <= matter for stability?
  
  Original array: [3a, 1, 3b, 2]
  
  After splitting and sorting:
    Left:  [1, 3a]     (3a came first in original)
    Right: [2, 3b]     (3b came second in original)
  
  MERGING with <=:
    1 ≤ 2?  Yes → take 1    [1]
    3a ≤ 2? No  → take 2    [1, 2]
    3a ≤ 3b? Yes (equal, take LEFT) → take 3a  [1, 2, 3a]
    Copy 3b                          [1, 2, 3a, 3b]  ✓ Stable!
  
  MERGING with <:
    1 < 2?  Yes → take 1    [1]
    3a < 2? No  → take 2    [1, 2]
    3a < 3b? No (equal, take RIGHT!) → take 3b  [1, 2, 3b]
    Copy 3a                           [1, 2, 3b, 3a]  ✗ Unstable!
  
  ┌──────────────────────────────────────────────────────┐
  │  RULE: When equal, take from LEFT array first.      │
  │  Code: use <= (not <) in the comparison.            │
  └──────────────────────────────────────────────────────┘
```

---

## The Four Cases in Merge

```java
// During merge, exactly one of four cases applies at each step:

for (int k = left; k <= right; k++) {
    if (i > mid) {
        // CASE 1: Left subarray exhausted
        // Copy remaining from right
        arr[k] = aux[j++];
    }
    else if (j > right) {
        // CASE 2: Right subarray exhausted
        // Copy remaining from left
        arr[k] = aux[i++];
    }
    else if (aux[i] <= aux[j]) {
        // CASE 3: Left element is smaller or equal
        // Take from left (ensures stability)
        arr[k] = aux[i++];
    }
    else {
        // CASE 4: Right element is smaller
        // Take from right
        arr[k] = aux[j++];
    }
}
```

```
  Case Flow:
  
  ┌───────────────────┐
  │ Is left exhausted?│── YES → Case 1: Take from right
  └────────┬──────────┘
           │ NO
  ┌────────▼──────────┐
  │ Is right exhausted?│── YES → Case 2: Take from left
  └────────┬──────────┘
           │ NO
  ┌────────▼──────────┐
  │ Left[i] ≤ Right[j]?│── YES → Case 3: Take from left
  └────────┬──────────┘
           │ NO
           └── Case 4: Take from right
```

---

## Merging K Sorted Arrays (Extension)

```
  Merge 2 arrays:  Use two pointers → O(n)
  Merge k arrays:  Two approaches
  
  Approach 1: Sequential merging
  ┌───────────────────────────────────────────────┐
  │  Merge arr1 + arr2 → temp1                   │
  │  Merge temp1 + arr3 → temp2                  │
  │  Merge temp2 + arr4 → temp3                  │
  │  ...                                          │
  │  Time: O(n·k) where n = total elements       │
  └───────────────────────────────────────────────┘
  
  Approach 2: Tournament-style (divide and conquer)
  ┌───────────────────────────────────────────────┐
  │  Merge pairs: (arr1,arr2), (arr3,arr4), ...  │
  │  Then merge resulting pairs                    │
  │  Time: O(n·log k)  ← BETTER!                 │
  │                                                │
  │  Level 0: [a1] [a2] [a3] [a4] [a5] [a6]     │
  │  Level 1: [a1+a2]   [a3+a4]   [a5+a6]       │
  │  Level 2: [a1+a2+a3+a4]     [a5+a6]         │
  │  Level 3: [fully merged]                      │
  └───────────────────────────────────────────────┘
  
  Approach 3: Min-Heap of k pointers
  ┌───────────────────────────────────────────────┐
  │  Put first element of each array in min-heap  │
  │  Extract min, put next element from that array│
  │  Time: O(n·log k)                             │
  │  Space: O(k) for the heap                     │
  └───────────────────────────────────────────────┘
```

---

## In-Place Merge (Advanced)

```
  Standard merge needs O(n) extra space.
  Can we merge in-place?
  
  YES, but it's complicated and rarely worth it:
  
  ┌──────────────────────────────────────────────────────────┐
  │  Simple in-place merge (O(n²)):                         │
  │  If left[i] > right[j], insert right[j] at position i  │
  │  by shifting everything in between.                     │
  │  → Destroys O(n log n) guarantee!                      │
  │                                                          │
  │  Block merge (O(n log n) in-place):                     │
  │  Used in "WikiSort" and "GrailSort"                     │
  │  Extremely complex — rarely implemented in practice     │
  │                                                          │
  │  Conclusion: The O(n) space of standard merge is        │
  │  considered an acceptable trade-off.                    │
  └──────────────────────────────────────────────────────────┘
```

---

## Counting Inversions Using Merge (Application)

```
  An INVERSION is a pair (i, j) where i < j but arr[i] > arr[j].
  
  Array: [2, 4, 1, 3, 5]
  Inversions: (2,1), (4,1), (4,3) → 3 inversions
  
  During MERGE, we can count inversions:
  When we take from RIGHT (meaning right element < left element),
  ALL remaining elements in left are inversions with that right element.
  
  Left: [2, 4]    Right: [1, 3, 5]
  
  Step 1: 2 > 1 → take 1, inversions += 2 (both 2 and 4 form inversions with 1)
  Step 2: 2 < 3 → take 2, no inversions
  Step 3: 4 > 3 → take 3, inversions += 1 (4 forms inversion with 3)
  Step 4: Take remaining 4, then 5
  
  Total: 3 inversions  ✓
```

```java
// Modified merge to count inversions
static long mergeSortCount(int[] arr, int[] aux, int lo, int hi) {
    if (lo >= hi) return 0;
    int mid = lo + (hi - lo) / 2;
    
    long count = 0;
    count += mergeSortCount(arr, aux, lo, mid);
    count += mergeSortCount(arr, aux, mid + 1, hi);
    count += mergeCount(arr, aux, lo, mid, hi);
    return count;
}

static long mergeCount(int[] arr, int[] aux, int lo, int mid, int hi) {
    for (int k = lo; k <= hi; k++) aux[k] = arr[k];
    
    long inversions = 0;
    int i = lo, j = mid + 1;
    
    for (int k = lo; k <= hi; k++) {
        if (i > mid)              arr[k] = aux[j++];
        else if (j > hi)          arr[k] = aux[i++];
        else if (aux[i] <= aux[j]) arr[k] = aux[i++];
        else {
            arr[k] = aux[j++];
            inversions += (mid - i + 1);  // KEY LINE
        }
    }
    return inversions;
}
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Core technique | Two-pointer merge of sorted arrays |
| Comparisons per merge | min(m,n) to m+n-1 |
| Stability condition | Use `<=` to prefer left element on ties |
| Four cases | Left exhausted / Right exhausted / Left smaller / Right smaller |
| Space | O(n) auxiliary for standard merge |
| In-place merge | Possible but complex (O(n²) simple, O(n log n) advanced) |
| Applications | Merge k sorted lists, inversion counting, external sort |

---

## Quick Revision Questions

1. **What is the two-pointer technique in merge?**
2. **What is the maximum number of comparisons when merging arrays of size m and n?**
3. **How does changing `<=` to `<` break stability?**
4. **Name the four cases in the merge loop.**
5. **How can you count inversions using merge sort?**
6. **Why is in-place merge rarely used in practice?**

---

[← Previous: Implementation](02-implementation.md) | [Next: Complexity Analysis →](04-complexity-analysis.md)

# 7.2 Intersection of Arrays

## Unit 7: Hash Set Problems

---

## Problem Statement

> Given two integer arrays, return their intersection — elements that appear in **both** arrays. Each element in the result must be unique.

```
╔══════════════════════════════════════════════════════════════╗
║  Input:  nums1 = [1, 2, 2, 1], nums2 = [2, 2]             ║
║  Output: [2]                                                ║
║                                                              ║
║  Input:  nums1 = [4, 9, 5], nums2 = [9, 4, 9, 8, 4]       ║
║  Output: [9, 4]    (order doesn't matter)                   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Approach 1: Two HashSets

```
FUNCTION intersection(nums1, nums2):
    set1 = new HashSet(nums1)
    result = new HashSet()

    FOR each num in nums2:
        IF set1.contains(num):
            result.add(num)

    RETURN result.toArray()
```

### Trace

```
nums1 = [4, 9, 5], nums2 = [9, 4, 9, 8, 4]

Step 1: Build set1 = {4, 9, 5}

Step 2: Scan nums2
  num=9: set1 has 9? YES → result={9}
  num=4: set1 has 4? YES → result={9,4}
  num=9: set1 has 9? YES → result={9,4}  (set ignores duplicate)
  num=8: set1 has 8? NO
  num=4: set1 has 4? YES → result={9,4}  (set ignores duplicate)

Result: [9, 4] ✓
```

**Complexity:** $O(n + m)$ time, $O(n)$ space  
(where $n$ = `|nums1|`, $m$ = `|nums2|`)

---

## Approach 2: Built-in Set Operations

Most languages provide set intersection directly:

```
// Python
set(nums1) & set(nums2)
set(nums1).intersection(set(nums2))

// Java
set1.retainAll(set2)

// JavaScript
new Set([...set1].filter(x => set2.has(x)))
```

---

## Approach 3: Sort + Two Pointers

When space is critical:

```
FUNCTION intersection(nums1, nums2):
    sort(nums1)
    sort(nums2)
    result = new HashSet()
    i = 0, j = 0

    WHILE i < len(nums1) AND j < len(nums2):
        IF nums1[i] < nums2[j]:
            i++
        ELSE IF nums1[i] > nums2[j]:
            j++
        ELSE:
            result.add(nums1[i])
            i++
            j++

    RETURN result.toArray()
```

### Trace

```
nums1 = [4, 9, 5] → sorted: [4, 5, 9]
nums2 = [9, 4, 9, 8, 4] → sorted: [4, 4, 8, 9, 9]

i=0, j=0: 4==4 → result={4}, i=1, j=1
i=1, j=1: 5>4  → j=2
i=1, j=2: 5<8  → i=2
i=2, j=2: 9>8  → j=3
i=2, j=3: 9==9 → result={4,9}, i=3 → DONE

Result: [4, 9] ✓
```

**Complexity:** $O(n\log n + m\log m)$ time, $O(1)$ extra space

---

## Variation: Intersection with Counts

> Return the intersection including **duplicates** (as many times as they appear in both).

```
Input:  nums1 = [1, 2, 2, 1], nums2 = [2, 2]
Output: [2, 2]   ← '2' appears twice in both

Input:  nums1 = [4, 9, 5], nums2 = [9, 4, 9, 8, 4]
Output: [4, 9]   ← '4' once, '9' once (min of each count)
```

```
FUNCTION intersect(nums1, nums2):
    // Use HashMap instead of HashSet
    countMap = new HashMap()

    FOR each num in nums1:
        countMap[num] = countMap.getOrDefault(num, 0) + 1

    result = []

    FOR each num in nums2:
        IF countMap.getOrDefault(num, 0) > 0:
            result.add(num)
            countMap[num]--

    RETURN result
```

### Trace

```
nums1 = [1, 2, 2, 1], nums2 = [2, 2]

Step 1: countMap = {1:2, 2:2}

Step 2: Scan nums2
  num=2: countMap[2]=2 > 0 → result=[2], countMap[2]=1
  num=2: countMap[2]=1 > 0 → result=[2,2], countMap[2]=0

Result: [2, 2] ✓
```

---

## Comparison of k-Array Intersection

For $k$ arrays:

```
╔══════════════════════════════════════════════════════════════╗
║  Sequential pairwise:                                       ║
║    result = intersection(arr[0], arr[1])                    ║
║    result = intersection(result, arr[2])                    ║
║    ...                                                      ║
║                                                              ║
║  Counter-based (all at once):                               ║
║    Count frequency of each element across all arrays        ║
║    Element is in intersection if count == k                 ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Method Comparison

| Approach | Time | Space | Preserves Order? | Handles Counts? |
|----------|:---:|:---:|:---:|:---:|
| Two HashSets | $O(n+m)$ | $O(n)$ | No | No |
| Built-in ops | $O(n+m)$ | $O(n)$ | No | No |
| Sort + 2 pointers | $O(n\log n+m\log m)$ | $O(1)$ | No | Possible |
| HashMap (counts) | $O(n+m)$ | $O(n)$ | No | Yes |

---

## Quick Revision Question

**Q: If the arrays are already sorted, which approach is most efficient and why?**

<details>
<summary>Click to reveal answer</summary>

**Two Pointers** on sorted arrays is most efficient:
- **Time: $O(n + m)$** — single pass through both
- **Space: $O(1)$** extra (excluding output)

This beats the HashSet approach because:
1. We skip the $O(n)$ space for building a set
2. No hash function computation overhead
3. Cache-friendly sequential memory access
4. We already paid zero sorting cost (pre-sorted)

The two-pointer technique works because sorted order guarantees:
- If `a[i] < b[j]`, no future element of `b` will equal `a[i]`
- If `a[i] > b[j]`, no future element of `a` will equal `b[j]`
- If equal, record it and advance both

</details>

---

| [← Duplicate Detection](01-duplicate-detection.md) | [Next: Union of Arrays →](03-union-of-arrays.md) |
|:---|---:|

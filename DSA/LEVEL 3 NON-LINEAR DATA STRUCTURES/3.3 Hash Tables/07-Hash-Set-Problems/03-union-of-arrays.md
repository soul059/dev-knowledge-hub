# 7.3 Union of Arrays

## Unit 7: Hash Set Problems

---

## Problem Statement

> Given two integer arrays, return their union — all **distinct** elements that appear in **either** array.

```
╔══════════════════════════════════════════════════════════════╗
║  Input:  nums1 = [1, 2, 2, 1], nums2 = [2, 3]             ║
║  Output: [1, 2, 3]                                          ║
║                                                              ║
║  Input:  nums1 = [4, 9, 5], nums2 = [9, 4, 9, 8, 4]       ║
║  Output: [4, 9, 5, 8]                                      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Approach 1: HashSet (Optimal)

```
FUNCTION union(nums1, nums2):
    unionSet = new HashSet()

    FOR each num in nums1:
        unionSet.add(num)

    FOR each num in nums2:
        unionSet.add(num)

    RETURN unionSet.toArray()
```

### Trace

```
nums1 = [4, 9, 5], nums2 = [9, 4, 9, 8, 4]

Add nums1:
  4 → {4}
  9 → {4, 9}
  5 → {4, 9, 5}

Add nums2:
  9 → {4, 9, 5}         (already exists)
  4 → {4, 9, 5}         (already exists)
  9 → {4, 9, 5}         (already exists)
  8 → {4, 9, 5, 8}      ← new
  4 → {4, 9, 5, 8}      (already exists)

Result: [4, 9, 5, 8] ✓
```

**Complexity:** $O(n + m)$ time, $O(n + m)$ space

---

## Approach 2: Built-in Set Operations

```
// Python
set(nums1) | set(nums2)
set(nums1).union(set(nums2))

// Java
Set<Integer> union = new HashSet<>(Arrays.asList(nums1));
union.addAll(Arrays.asList(nums2));

// JavaScript
new Set([...nums1, ...nums2])
```

The JavaScript approach is especially concise — spreading both arrays into one, then wrapping in a Set automatically deduplicates.

---

## Approach 3: Sort + Merge

```
FUNCTION union(nums1, nums2):
    sort(nums1)
    sort(nums2)
    result = []
    i = 0, j = 0

    WHILE i < len(nums1) AND j < len(nums2):
        // Skip duplicates within nums1
        IF i > 0 AND nums1[i] == nums1[i-1]:
            i++; CONTINUE

        // Skip duplicates within nums2
        IF j > 0 AND nums2[j] == nums2[j-1]:
            j++; CONTINUE

        IF nums1[i] < nums2[j]:
            result.add(nums1[i])
            i++
        ELSE IF nums1[i] > nums2[j]:
            result.add(nums2[j])
            j++
        ELSE:
            result.add(nums1[i])
            i++; j++

    // Add remaining elements (skip dups)
    WHILE i < len(nums1):
        IF i == 0 OR nums1[i] != nums1[i-1]:
            result.add(nums1[i])
        i++

    WHILE j < len(nums2):
        IF j == 0 OR nums2[j] != nums2[j-1]:
            result.add(nums2[j])
        j++

    RETURN result
```

### Trace

```
nums1 = [1, 2, 2, 1] → sorted: [1, 1, 2, 2]
nums2 = [2, 3]        → sorted: [2, 3]

i=0, j=0: 1<2  → result=[1], i=1
i=1, j=0: 1==prev → skip, i=2
i=2, j=0: 2==2 → result=[1,2], i=3, j=1
i=3, j=1: 2==prev → skip, i=4
Remaining nums2: j=1, nums2[1]=3 → result=[1,2,3]

Result: [1, 2, 3] ✓
```

**Complexity:** $O(n\log n + m\log m)$ time, $O(1)$ extra space

---

## Set Theory Relationships

```
╔══════════════════════════════════════════════════════════════╗
║  A = {1, 2, 3, 4}     B = {3, 4, 5, 6}                    ║
║                                                              ║
║  Union (A ∪ B):          {1, 2, 3, 4, 5, 6}               ║
║  Intersection (A ∩ B):   {3, 4}                            ║
║  Difference (A − B):     {1, 2}                            ║
║  Symmetric Diff (A △ B): {1, 2, 5, 6}                     ║
║                                                              ║
║  Identity: |A ∪ B| = |A| + |B| − |A ∩ B|                  ║
║  For above:    6    =  4  +  4  −  2    ✓                  ║
╚══════════════════════════════════════════════════════════════╝
```

### Implementing All Set Operations with HashSet

```
FUNCTION setDifference(nums1, nums2):
    set2 = new HashSet(nums2)
    result = new HashSet()

    FOR each num in nums1:
        IF NOT set2.contains(num):
            result.add(num)
    RETURN result

FUNCTION symmetricDifference(nums1, nums2):
    set1 = new HashSet(nums1)
    set2 = new HashSet(nums2)
    result = new HashSet()

    FOR each num in set1:
        IF NOT set2.contains(num):
            result.add(num)

    FOR each num in set2:
        IF NOT set1.contains(num):
            result.add(num)

    RETURN result
```

---

## Union of k Arrays

```
FUNCTION unionAll(arrays):
    unionSet = new HashSet()

    FOR each arr in arrays:
        FOR each num in arr:
            unionSet.add(num)

    RETURN unionSet.toArray()

// Time: O(N) where N = total elements across all arrays
// Space: O(U) where U = number of unique elements
```

### Trace

```
arrays = [[1,2], [2,3], [3,4]]

After [1,2]: {1, 2}
After [2,3]: {1, 2, 3}
After [3,4]: {1, 2, 3, 4}

Result: [1, 2, 3, 4]
```

---

## Method Comparison

| Approach | Time | Space | Sorted Output? |
|----------|:---:|:---:|:---:|
| HashSet | $O(n+m)$ | $O(n+m)$ | No |
| Built-in set ops | $O(n+m)$ | $O(n+m)$ | No |
| Sort + Merge | $O(n\log n+m\log m)$ | $O(1)$ extra | Yes |
| TreeSet | $O((n+m)\log(n+m))$ | $O(n+m)$ | Yes |

---

## Quick Revision Question

**Q: Given three arrays A, B, C, find elements that appear in at least two of the three arrays. How would you solve this efficiently with HashSets?**

<details>
<summary>Click to reveal answer</summary>

**Strategy:** Build a set for each array, then check pairwise intersections:

```
set_A = HashSet(A)
set_B = HashSet(B)
set_C = HashSet(C)

result = new HashSet()

// Add all pairwise intersections
FOR each x in set_A:
    IF set_B.contains(x) OR set_C.contains(x):
        result.add(x)

FOR each x in set_B:
    IF set_C.contains(x):
        result.add(x)
```

**Time:** $O(n_A + n_B + n_C)$  |  **Space:** $O(n_A + n_B + n_C)$

Alternative: Use a **HashMap** to count how many arrays each element appears in, then filter for `count ≥ 2`:

```
countMap = new HashMap()
FOR each arr in [A, B, C]:
    seen = new HashSet(arr)   // deduplicate within array
    FOR each x in seen:
        countMap[x]++

result = [x for x in countMap if countMap[x] >= 2]
```

This generalizes better to $k$ arrays.

</details>

---

| [← Intersection of Arrays](02-intersection-of-arrays.md) | [Next: First Unique Character →](04-first-unique-character.md) |
|:---|---:|

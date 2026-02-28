# 7.5 Contains Duplicate Variants

## Unit 7: Hash Set Problems

---

## Overview

The "Contains Duplicate" family tests mastery of HashSets and HashMaps with increasing constraints.

```
╔══════════════════════════════════════════════════════════════╗
║  VARIANT          │ CORE IDEA                               ║
║  ─────────────────┼─────────────────────────────────────────║
║  I   (basic)      │ Any duplicate exists?         HashSet   ║
║  II  (distance)   │ Dup within index distance k?  Sliding   ║
║  III (value range) │ Dup within value range t AND            ║
║                   │  index distance k?           Bucket Sort║
╚══════════════════════════════════════════════════════════════╝
```

---

## Variant I: Contains Duplicate (Basic)

> Return `true` if any element appears at least twice.

*(Covered in detail in [7.1 Duplicate Detection](01-duplicate-detection.md))*

```
FUNCTION containsDuplicate(nums):
    seen = new HashSet()
    FOR each num in nums:
        IF seen.contains(num): RETURN true
        seen.add(num)
    RETURN false

Time: O(n)  |  Space: O(n)
```

---

## Variant II: Contains Duplicate Within K Distance

> Return `true` if `nums[i] == nums[j]` and $|i - j| \leq k$.

### Key Insight: Sliding Window of Size k

Maintain a HashSet representing a **sliding window** of the last $k$ elements.

```
FUNCTION containsNearbyDuplicate(nums, k):
    window = new HashSet()

    FOR i = 0 TO n - 1:
        IF window.contains(nums[i]):
            RETURN true

        window.add(nums[i])

        IF window.size() > k:
            window.remove(nums[i - k])

    RETURN false
```

### Complete Trace

```
nums = [1, 0, 1, 1], k = 1

i=0: num=1
     window={}, not found → add → window={1}
     size=1, k=1 → not > k

i=1: num=0
     window={1}, not found → add → window={1,0}
     size=2 > k=1 → remove nums[1-1]=nums[0]=1 → window={0}

i=2: num=1
     window={0}, not found → add → window={0,1}
     size=2 > k=1 → remove nums[2-1]=nums[1]=0 → window={1}

i=3: num=1
     window={1}, FOUND! → return true ✓
     (indices 2 and 3, |3-2| = 1 ≤ k = 1)
```

### Alternative: HashMap Approach

Store the **last seen index** of each element:

```
FUNCTION containsNearbyDuplicate(nums, k):
    lastIndex = new HashMap()

    FOR i = 0 TO n - 1:
        IF lastIndex.containsKey(nums[i]):
            IF i - lastIndex[nums[i]] <= k:
                RETURN true

        lastIndex[nums[i]] = i

    RETURN false
```

```
Trace: nums = [1, 2, 3, 1, 2, 3], k = 2
  i=0: 1 not in map → map={1:0}
  i=1: 2 not in map → map={1:0, 2:1}
  i=2: 3 not in map → map={1:0, 2:1, 3:2}
  i=3: 1 in map, 3-0=3 > k=2 → update → map={1:3, 2:1, 3:2}
  i=4: 2 in map, 4-1=3 > k=2 → update → map={1:3, 2:4, 3:2}
  i=5: 3 in map, 5-2=3 > k=2 → update → map={1:3, 2:4, 3:5}
  No match → return false

  With k=3: at i=3, 3-0=3 ≤ 3 → return true ✓
```

---

## Variant III: Contains Duplicate Within K Distance AND Value Range t

> Return `true` if there exist indices $i \neq j$ such that:
> - $|nums[i] - nums[j]| \leq t$  (value difference)
> - $|i - j| \leq k$ (index difference)

### Key Insight: Bucket Sort

Divide numbers into **buckets of size $t+1$**. Two numbers within range $t$ of each other must be in the **same bucket** or **adjacent buckets**.

```
╔══════════════════════════════════════════════════════════════╗
║  If t = 3, bucket size = 4                                  ║
║                                                              ║
║  Bucket 0: [0, 3]     ← values 0-3                         ║
║  Bucket 1: [4, 7]     ← values 4-7                         ║
║  Bucket 2: [8, 11]    ← values 8-11                        ║
║                                                              ║
║  Value 5 → bucket 1                                         ║
║  Value 7 → bucket 1   ← same bucket! |7-5|=2 ≤ 3 ✓       ║
║  Value 8 → bucket 2   ← adjacent to bucket 1               ║
║                        check: |8-5|=3 ≤ 3 ✓               ║
║  Value 12 → bucket 3  ← NOT adjacent to bucket 1           ║
║                        |12-5|=7 > 3 ✗                      ║
╚══════════════════════════════════════════════════════════════╝
```

```
FUNCTION containsNearbyAlmostDuplicate(nums, k, t):
    IF t < 0: RETURN false
    buckets = new HashMap()     // bucket_id → value
    w = t + 1                   // bucket width

    FOR i = 0 TO n - 1:
        bucket_id = getBucketId(nums[i], w)

        // Same bucket → guaranteed within range t
        IF buckets.containsKey(bucket_id):
            RETURN true

        // Adjacent bucket → check value difference
        IF buckets.containsKey(bucket_id - 1):
            IF abs(nums[i] - buckets[bucket_id - 1]) <= t:
                RETURN true

        IF buckets.containsKey(bucket_id + 1):
            IF abs(nums[i] - buckets[bucket_id + 1]) <= t:
                RETURN true

        // Add to bucket
        buckets[bucket_id] = nums[i]

        // Maintain window of size k
        IF i >= k:
            old_bucket = getBucketId(nums[i - k], w)
            buckets.remove(old_bucket)

    RETURN false

FUNCTION getBucketId(value, w):
    IF value >= 0:
        RETURN value / w         // integer division
    ELSE:
        RETURN (value + 1) / w - 1   // handle negatives
```

### Trace

```
nums = [1, 5, 9, 1, 5, 9], k = 2, t = 3
w = t+1 = 4

i=0: num=1, bucket=0
     No same/adjacent buckets → buckets={0:1}

i=1: num=5, bucket=1
     Same bucket? No
     Adjacent bucket 0 exists: |5-1|=4 > t=3 → NO
     Adjacent bucket 2? No
     buckets={0:1, 1:5}

i=2: num=9, bucket=2
     Same bucket? No
     Adjacent bucket 1 exists: |9-5|=4 > t=3 → NO
     Adjacent bucket 3? No
     buckets={0:1, 1:5, 2:9}
     i=2 ≥ k=2 → remove bucket of nums[0]=1 (bucket 0)
     buckets={1:5, 2:9}

i=3: num=1, bucket=0
     Same bucket? No
     Adjacent bucket -1? No
     Adjacent bucket 1 exists: |1-5|=4 > t=3 → NO
     buckets={0:1, 1:5, 2:9}
     i=3 ≥ k=2 → remove bucket of nums[1]=5 (bucket 1)
     buckets={0:1, 2:9}

i=4: num=5, bucket=1
     Same bucket? No
     Adjacent bucket 0 exists: |5-1|=4 > t=3 → NO
     Adjacent bucket 2 exists: |5-9|=4 > t=3 → NO
     buckets={0:1, 1:5, 2:9}
     i=4 ≥ k=2 → remove bucket of nums[2]=9 (bucket 2)
     buckets={0:1, 1:5}

i=5: num=9, bucket=2
     Same bucket? No
     Adjacent bucket 1 exists: |9-5|=4 > t=3 → NO
     Adjacent bucket 3? No
     → return false ✓
```

**Why only same & adjacent buckets?**
- Same bucket: values differ by at most $w-1 = t$ → always valid
- Adjacent bucket: values *might* differ by $\leq t$ → must check
- Farther buckets: values differ by $\geq w = t+1$ → never valid

**Time:** $O(n)$  |  **Space:** $O(\min(n, k))$

---

## Variant Comparison

| Variant | Constraint | Data Structure | Time | Space |
|---------|-----------|:-:|:-:|:-:|
| I | Any duplicate | HashSet | $O(n)$ | $O(n)$ |
| II | Within $k$ indices | Sliding HashSet | $O(n)$ | $O(k)$ |
| III | Within $k$ indices + value $t$ | Bucket HashMap | $O(n)$ | $O(k)$ |

---

## Quick Revision Question

**Q: In Contains Duplicate III, why can't we just use a HashSet with a sliding window like Variant II? Why do we need the bucket approach?**

<details>
<summary>Click to reveal answer</summary>

In **Variant II**, we check for **exact equality** — `HashSet.contains(num)` is $O(1)$.

In **Variant III**, we check for **approximate equality** — is there any value within range $[num-t, num+t]$? With a plain HashSet, we'd need to check $2t+1$ values (from $num-t$ to $num+t$), making it $O(t)$ per element and $O(nt)$ total. If $t$ is large, this is too slow.

The **bucket approach** solves this by:
1. Mapping values to bucket IDs so that "within range $t$" corresponds to "same or adjacent bucket"
2. Each bucket holds at most ONE value (within the window), so lookups are $O(1)$
3. We only check 3 buckets (same, left neighbor, right neighbor) → always $O(1)$

**Alternative:** A **Balanced BST** (TreeSet/TreeMap) with `ceiling()` and `floor()` methods can find the nearest value in $O(\log k)$ time, giving $O(n\log k)$ total. The bucket approach is $O(n)$ because it trades the BST's ordering for hash-based bucket placement.

</details>

---

| [← First Unique Character](04-first-unique-character.md) | [Next: Cycle Detection →](06-cycle-detection.md) |
|:---|---:|

# Unit 10: Bucket Sort — Hybrid Approaches

[← Previous: Uniform Distribution](04-uniform-distribution.md) | [Next: Comparison with Other Sorts →](06-comparison-with-other-sorts.md)

---

## Overview

Pure bucket sort rarely appears alone in practice. This chapter explores how bucket sort combines with other algorithms to produce highly efficient hybrid sorts tailored to specific data patterns.

---

## Why Hybridize?

```
  ┌──────────────────────────────────────────────────────────┐
  │  Bucket sort is great at DISTRIBUTING, but needs help   │
  │  with the actual SORTING inside each bucket.            │
  │                                                          │
  │  Distribution phase:  O(n)   — always fast              │
  │  Inner sort phase:    varies — depends on bucket sizes  │
  │                                                          │
  │  Choice of inner sort determines:                       │
  │  • Worst-case behavior                                  │
  │  • Stability                                            │
  │  • Cache performance                                    │
  │  • Practical speed                                      │
  └──────────────────────────────────────────────────────────┘
```

---

## Hybrid 1: Bucket + Insertion Sort (Classic)

The default combination. Exploits the fact that with uniform data, buckets are tiny.

```
  Expected bucket size: n/k ≈ 1
  
  Insertion sort cost: O(m²) on bucket of size m
  But for m ≤ 16, insertion sort beats everything due to:
    • No recursion overhead
    • No extra memory
    • Great cache locality
    • Low constant factor

  Total: Σ O(nᵢ²) = O(n)  (uniform case)
```

```java
// Classic bucket + insertion sort
for (List<Integer> bucket : buckets) {
    // Insertion sort is perfect for small buckets
    for (int i = 1; i < bucket.size(); i++) {
        int key = bucket.get(i);
        int j = i - 1;
        while (j >= 0 && bucket.get(j) > key) {
            bucket.set(j + 1, bucket.get(j));
            j--;
        }
        bucket.set(j + 1, key);
    }
}
```

**Best when**: Data is uniformly distributed, buckets are small (< 16 elements).

---

## Hybrid 2: Bucket + Merge Sort (Guaranteed O(n log n))

When you can't guarantee uniform distribution, use merge sort inside buckets to cap worst-case.

```
  ┌─────────────────────────────────────────────────────┐
  │  Worst case: all n elements in one bucket          │
  │                                                     │
  │  With insertion sort: O(n²)   ← BAD               │
  │  With merge sort:    O(n log n) ← guaranteed      │
  │                                                     │
  │  Trade-off: merge sort has higher overhead for     │
  │  small buckets, but prevents catastrophic cases.   │
  └─────────────────────────────────────────────────────┘
```

```python
def bucket_sort_merge(arr):
    n = len(arr)
    buckets = [[] for _ in range(n)]
    
    min_val, max_val = min(arr), max(arr)
    r = (max_val - min_val) / n + 1e-9
    
    for val in arr:
        idx = min(int((val - min_val) / r), n - 1)
        buckets[idx].append(val)
    
    result = []
    for bucket in buckets:
        result.extend(merge_sort(bucket))  # O(m log m) per bucket
    arr[:] = result

def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)
```

**Best when**: Distribution is unknown or non-uniform, need worst-case guarantee.

---

## Hybrid 3: Bucket + Quick Sort (Practical Speed)

Quick Sort has best practical performance for medium-sized arrays.

```
  When bucket sizes are moderate (16-1000 elements):
  
  Bucket:  [47, 32, 41, 38, 35, 43, 50, 29, ...]
            └─── Quick Sort this (~50 elements) ───┘
  
  Quick Sort advantages here:
  • In-place → no extra memory per bucket
  • Cache-friendly → sequential access
  • O(n log n) average on each bucket
```

```java
for (List<Integer> bucket : buckets) {
    if (bucket.size() <= 16) {
        insertionSort(bucket);  // Tiny? Use insertion sort
    } else {
        Collections.sort(bucket);  // Java uses dual-pivot Quick Sort
    }
}
```

---

## Hybrid 4: Bucket + Counting/Radix Sort (For Integer Data)

When bucket values are integers in a known small range, use counting sort inside buckets.

```
  Example: Sort ages [0, 150] with bucket sort
  
  Bucket [0-30]:   ages within range → counting sort (range 0..30)
  Bucket [31-60]:  ages within range → counting sort (range 0..30)  
  Bucket [61-90]:  ages within range → counting sort (range 0..30)
  Bucket [91-150]: ages within range → counting sort (range 0..60)
  
  Each counting sort: O(nᵢ + range) = O(nᵢ + 30)
  Total: O(n + k × 30) = O(n)
```

```python
def bucket_counting_sort(arr, max_val):
    num_buckets = 10
    bucket_range = max_val // num_buckets + 1
    
    buckets = [[] for _ in range(num_buckets)]
    for val in arr:
        buckets[val // bucket_range].append(val)
    
    result = []
    for bucket in buckets:
        if bucket:
            # Use counting sort within each bucket
            local_max = max(bucket)
            local_min = min(bucket)
            count = [0] * (local_max - local_min + 1)
            for v in bucket:
                count[v - local_min] += 1
            for i, c in enumerate(count):
                result.extend([i + local_min] * c)
    
    arr[:] = result
```

---

## Hybrid 5: Multi-Level Bucket Sort (Recursive Bucketing)

Apply bucket sort recursively: if a bucket is too large, bucket-sort it again with finer granularity.

```
  Level 1: 10 buckets over [0, 1)
  
  [0.0-0.1] [0.1-0.2] ... [0.5-0.6] ... [0.9-1.0]
       2        1              15             3
  
  Bucket [0.5-0.6] has 15 elements → too many!
  
  Level 2: Sub-bucket [0.5-0.6] into 10 more buckets:
  [.50-.51] [.51-.52] ... [.55-.56] ... [.59-.60]
      2         1             3             1
  
  Now all are small → insertion sort
```

```java
public static void recursiveBucketSort(List<Double> arr, 
                                        double lo, double hi, 
                                        int threshold) {
    if (arr.size() <= threshold) {
        Collections.sort(arr);  // Base case
        return;
    }
    
    int k = 10;  // 10 sub-buckets
    double range = (hi - lo) / k;
    List<List<Double>> buckets = new ArrayList<>();
    for (int i = 0; i < k; i++) buckets.add(new ArrayList<>());
    
    for (double val : arr) {
        int idx = Math.min((int)((val - lo) / range), k - 1);
        buckets.get(idx).add(val);
    }
    
    arr.clear();
    for (int i = 0; i < k; i++) {
        recursiveBucketSort(buckets.get(i), 
                           lo + i * range, 
                           lo + (i + 1) * range, 
                           threshold);
        arr.addAll(buckets.get(i));
    }
}
```

---

## Choosing the Right Inner Sort

```
  ┌─────────────────────────────────────────────────────────────┐
  │              DECISION GUIDE                                │
  ├──────────────────────┬──────────────────────────────────────┤
  │  Bucket size ≤ 16   │ Insertion Sort                       │
  │  Bucket size 17-100 │ Quick Sort (or language default)     │
  │  Bucket size > 100  │ Merge Sort or recurse bucket sort   │
  │  Integer small range│ Counting Sort                        │
  │  Need stability     │ Merge Sort or Counting Sort         │
  │  Memory constrained │ Quick Sort (in-place)               │
  │  Unknown distribution│ Merge Sort (O(n log n) guarantee)  │
  └──────────────────────┴──────────────────────────────────────┘
```

---

## Adaptive Hybrid: Auto-Selecting Inner Sort

```java
public static void adaptiveBucketSort(int[] arr) {
    int n = arr.length;
    int numBuckets = (int) Math.sqrt(n);
    
    // ... distribute into buckets ...
    
    int index = 0;
    for (List<Integer> bucket : buckets) {
        int size = bucket.size();
        
        if (size <= 1) {
            // Nothing to sort
        } else if (size <= 16) {
            insertionSort(bucket);
        } else if (size <= 100) {
            Collections.sort(bucket);  // Quick Sort
        } else {
            // Too large — bucket sort recursively
            int[] sub = bucket.stream()
                              .mapToInt(Integer::intValue)
                              .toArray();
            adaptiveBucketSort(sub);
            bucket.clear();
            for (int v : sub) bucket.add(v);
        }
        
        for (int val : bucket)
            arr[index++] = val;
    }
}
```

---

## Performance Comparison of Hybrids

```
  n = 10,000 uniform [0, 1)  |  Time (relative)
  ────────────────────────────┼──────────────────
  Bucket + Insertion Sort     │  1.0x  (baseline)
  Bucket + Quick Sort         │  1.1x
  Bucket + Merge Sort         │  1.3x
  Bucket + Counting Sort      │  0.7x  (integer only)
  Pure Quick Sort (no bucket) │  2.5x
  
  n = 10,000 SKEWED data     │  Time (relative)
  ────────────────────────────┼──────────────────
  Bucket + Insertion Sort     │  8.0x  (buckets uneven)
  Bucket + Quick Sort         │  2.3x
  Bucket + Merge Sort         │  2.5x  (guaranteed)
  Adaptive hybrid             │  1.5x
  Pure Quick Sort             │  2.0x
```

---

## Summary Table

| Hybrid | Avg Time | Worst Time | Stable? | Extra Space |
|--------|----------|-----------|---------|-------------|
| Bucket + Insertion | O(n) | O(n²) | Yes | O(n) |
| Bucket + Merge | O(n) | O(n log n) | Yes | O(n) |
| Bucket + Quick | O(n) | O(n²)* | No | O(n) |
| Bucket + Counting | O(n) | O(n + k) | Yes | O(n + k) |
| Recursive Bucket | O(n) | O(n log n)** | Depends | O(n) |

\* Quick Sort's O(n²) is extremely rare with randomized pivot  
\** With recursion depth ~ log(n/threshold)

---

## Quick Revision Questions

1. **Why is insertion sort ideal for the classic bucket sort implementation?**
2. **When should you use merge sort instead of insertion sort inside buckets?**
3. **How does recursive bucket sort work? What's the base case?**
4. **Design an adaptive hybrid that picks the inner sort based on bucket size.**
5. **What's the advantage of using counting sort inside buckets for integer data?**
6. **Compare the worst-case guarantees of bucket+insertion vs bucket+merge.**

---

[← Previous: Uniform Distribution](04-uniform-distribution.md) | [Next: Comparison with Other Sorts →](06-comparison-with-other-sorts.md)

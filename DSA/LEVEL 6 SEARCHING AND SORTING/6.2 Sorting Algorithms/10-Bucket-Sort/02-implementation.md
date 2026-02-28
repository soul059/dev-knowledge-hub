# Unit 10: Bucket Sort — Implementation

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)

---

## Overview

This chapter provides complete Bucket Sort implementations for floating-point numbers and integers in Java, Python, and C++.

---

## Java — Floating-Point [0, 1)

```java
import java.util.*;

public class BucketSort {
    
    public static void bucketSort(float[] arr) {
        int n = arr.length;
        if (n <= 1) return;
        
        // Create n empty buckets
        @SuppressWarnings("unchecked")
        List<Float>[] buckets = new ArrayList[n];
        for (int i = 0; i < n; i++) {
            buckets[i] = new ArrayList<>();
        }
        
        // Distribute elements into buckets
        for (float val : arr) {
            int idx = (int)(val * n);
            if (idx == n) idx = n - 1;  // Handle edge case val = 1.0
            buckets[idx].add(val);
        }
        
        // Sort each bucket
        for (List<Float> bucket : buckets) {
            Collections.sort(bucket);  // or insertion sort
        }
        
        // Concatenate all buckets
        int index = 0;
        for (List<Float> bucket : buckets) {
            for (float val : bucket) {
                arr[index++] = val;
            }
        }
    }
    
    public static void main(String[] args) {
        float[] arr = {0.78f, 0.17f, 0.39f, 0.26f, 0.72f, 
                       0.94f, 0.21f, 0.12f, 0.23f, 0.68f};
        bucketSort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

---

## Java — General Integers

```java
public static void bucketSort(int[] arr, int numBuckets) {
    if (arr.length <= 1) return;
    
    int min = arr[0], max = arr[0];
    for (int num : arr) {
        min = Math.min(min, num);
        max = Math.max(max, num);
    }
    
    double range = (double)(max - min + 1) / numBuckets;
    
    @SuppressWarnings("unchecked")
    List<Integer>[] buckets = new ArrayList[numBuckets];
    for (int i = 0; i < numBuckets; i++)
        buckets[i] = new ArrayList<>();
    
    // Distribute
    for (int num : arr) {
        int idx = (int)((num - min) / range);
        if (idx == numBuckets) idx--;
        buckets[idx].add(num);
    }
    
    // Sort each bucket with insertion sort
    int index = 0;
    for (List<Integer> bucket : buckets) {
        insertionSort(bucket);
        for (int num : bucket)
            arr[index++] = num;
    }
}

private static void insertionSort(List<Integer> bucket) {
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

---

## Python Implementation

```python
def bucket_sort(arr):
    if len(arr) <= 1:
        return arr
    
    n = len(arr)
    min_val = min(arr)
    max_val = max(arr)
    
    # Create buckets
    bucket_range = (max_val - min_val) / n + 1e-9  # avoid division by zero
    buckets = [[] for _ in range(n)]
    
    # Distribute
    for val in arr:
        idx = int((val - min_val) / bucket_range)
        if idx >= n:
            idx = n - 1
        buckets[idx].append(val)
    
    # Sort each bucket and concatenate
    result = []
    for bucket in buckets:
        bucket.sort()  # Python's Timsort
        result.extend(bucket)
    
    arr[:] = result

# For floats in [0, 1)
def bucket_sort_float(arr):
    n = len(arr)
    buckets = [[] for _ in range(n)]
    
    for val in arr:
        idx = int(val * n)
        if idx == n:
            idx = n - 1
        buckets[idx].append(val)
    
    result = []
    for bucket in buckets:
        bucket.sort()
        result.extend(bucket)
    
    arr[:] = result
```

---

## C++ Implementation

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

void bucketSort(vector<float>& arr) {
    int n = arr.size();
    if (n <= 1) return;
    
    // Create n empty buckets
    vector<vector<float>> buckets(n);
    
    // Distribute
    for (float val : arr) {
        int idx = (int)(val * n);
        if (idx >= n) idx = n - 1;
        buckets[idx].push_back(val);
    }
    
    // Sort each bucket
    for (auto& bucket : buckets) {
        sort(bucket.begin(), bucket.end());
    }
    
    // Concatenate
    int index = 0;
    for (auto& bucket : buckets) {
        for (float val : bucket) {
            arr[index++] = val;
        }
    }
}

int main() {
    vector<float> arr = {0.78, 0.17, 0.39, 0.26, 0.72};
    bucketSort(arr);
    for (float x : arr) cout << x << " ";
    return 0;
}
```

---

## Insertion Sort Within Buckets: Why?

```
  ┌──────────────────────────────────────────────────────────┐
  │  With uniform distribution, each bucket has ~1 element. │
  │  Insertion sort on 1-2 elements = O(1).                 │
  │                                                          │
  │  Even if a bucket has a few elements:                   │
  │  Insertion sort's small overhead + good cache behavior  │
  │  makes it faster than merge sort for tiny arrays.       │
  │                                                          │
  │  Bucket size expectations (n elements, n buckets):      │
  │  • Expected size per bucket: 1                          │
  │  • Probability of bucket size > c×log(n): very low     │
  │  • Insertion sort on O(log n) elements: O(log²n)       │
  │                                                          │
  │  Average total: n × O(1) = O(n) ✓                      │
  └──────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Trace

```
  Array: [0.42, 0.32, 0.23, 0.52, 0.25, 0.47, 0.51]
  n = 7 buckets
  
  Distribution (index = ⌊val × 7⌋):
  0.42 → ⌊2.94⌋ = 2
  0.32 → ⌊2.24⌋ = 2
  0.23 → ⌊1.61⌋ = 1
  0.52 → ⌊3.64⌋ = 3
  0.25 → ⌊1.75⌋ = 1
  0.47 → ⌊3.29⌋ = 3
  0.51 → ⌊3.57⌋ = 3
  
  Buckets:
  [0]: []
  [1]: [0.23, 0.25]
  [2]: [0.42, 0.32]
  [3]: [0.52, 0.47, 0.51]
  [4-6]: []
  
  After sorting each bucket:
  [1]: [0.23, 0.25]
  [2]: [0.32, 0.42]
  [3]: [0.47, 0.51, 0.52]
  
  Concatenate: [0.23, 0.25, 0.32, 0.42, 0.47, 0.51, 0.52] ✓
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Bucket count | Typically n (same as input size) |
| Index formula (float [0,1)) | `⌊value × n⌋` |
| Index formula (int range) | `⌊(value - min) × k / range⌋` |
| Inner sort | Insertion sort (for small buckets) |
| Concatenation | Linear scan through all buckets |

---

## Quick Revision Questions

1. **How do you compute the bucket index for a value in range [min, max]?**
2. **Why is insertion sort preferred over merge sort inside buckets?**
3. **What happens if you use too few buckets?**
4. **What happens if you use too many buckets?**
5. **Trace bucket sort on [0.5, 0.1, 0.9, 0.3] with 4 buckets.**

---

[← Previous: Algorithm Description](01-algorithm-description.md) | [Next: Complexity Analysis →](03-complexity-analysis.md)

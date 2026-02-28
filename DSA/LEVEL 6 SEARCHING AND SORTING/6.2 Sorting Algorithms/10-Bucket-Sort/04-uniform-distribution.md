# Unit 10: Bucket Sort — Uniform Distribution

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: Hybrid Approaches →](05-hybrid-approaches.md)

---

## Overview

Bucket Sort's O(n) performance **depends entirely on how evenly elements spread across buckets**. This chapter explores why distribution matters, what goes wrong with skewed data, and how to handle non-uniform inputs.

---

## Why Uniformity Matters

```
  UNIFORM DISTRIBUTION               SKEWED DISTRIBUTION
  
  Bucket: [0]  [1]  [2]  [3]  [4]    [0]  [1]  [2]  [3]  [4]
          ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐   ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐
          │██│ │██│ │██│ │██│ │██│   │  │ │  │ │██│ │  │ │  │
          │██│ │  │ │██│ │  │ │██│   │  │ │  │ │██│ │  │ │  │
          │  │ │  │ │  │ │  │ │  │   │  │ │  │ │██│ │  │ │  │
          └──┘ └──┘ └──┘ └──┘ └──┘   │  │ │  │ │██│ │  │ │  │
                                      │  │ │  │ │██│ │  │ │  │
    ~2 per bucket → O(n)              └──┘ └──┘ │██│ └──┘ └──┘
                                                │██│
                                                │██│
                                                └──┘
                                      8 in one bucket → O(n²)
```

### Mathematical Connection

```
  Total sorting time = Σᵢ O(nᵢ²)
  
  By Cauchy-Schwarz inequality:
      Σ nᵢ² ≥ (Σ nᵢ)² / k = n² / k
  
  Equality when nᵢ = n/k for all i  (perfectly uniform!)
  
  If uniform:  Σ nᵢ² = k × (n/k)² = n²/k
  With k = n:  Σ nᵢ² = n²/n = n → O(n) total sorting
  
  If skewed:   Σ nᵢ² can be as large as n² → O(n²)
```

---

## Common Non-Uniform Distributions

### 1. Normal (Gaussian) Distribution

```
  Most data clusters around the mean:
                 ████
               ████████
             ████████████
           ██████████████████
  ────────██████████████████████────────
  Bucket: [0]  [1]  [2]  [3]  [4]  [5]
   Count:  1    3    8    7    2    1
  
  Middle buckets get overloaded.
```

### 2. Exponential Distribution

```
  Heavy at low end, sparse at high end:
  ████████████
  ██████████
  ████████
  ██████
  ████
  ██
  █
  ─────────────────────
  [0]  [1]  [2]  [3]  [4]
   10   5    3    1    1
```

### 3. Clustered Data

```
  Grades: mostly 70-90 range:
  
  [0-20] [20-40] [40-60] [60-80] [80-100]
    0       1       2      12       5
  
  Bucket [60-80] has 12 elements → O(12²) = O(144)
```

---

## Strategies for Non-Uniform Data

### Strategy 1: Adaptive Bucket Boundaries

Instead of equal-width buckets, use **equal-depth** (quantile-based) boundaries:

```java
// Compute percentile-based bucket boundaries
public static void adaptiveBucketSort(int[] arr) {
    int n = arr.length;
    int numBuckets = (int) Math.sqrt(n);
    
    // Sort a sample to estimate quantiles
    int[] sample = Arrays.copyOf(arr, Math.min(n, 1000));
    Arrays.sort(sample);
    
    // Create bucket boundaries at quantile positions
    double[] boundaries = new double[numBuckets + 1];
    boundaries[0] = Double.NEGATIVE_INFINITY;
    boundaries[numBuckets] = Double.POSITIVE_INFINITY;
    for (int i = 1; i < numBuckets; i++) {
        int pos = (int)((long) i * sample.length / numBuckets);
        boundaries[i] = sample[pos];
    }
    
    // Distribute using binary search on boundaries
    List<List<Integer>> buckets = new ArrayList<>();
    for (int i = 0; i < numBuckets; i++)
        buckets.add(new ArrayList<>());
    
    for (int val : arr) {
        int idx = Arrays.binarySearch(boundaries, val);
        if (idx < 0) idx = -(idx + 1) - 1;
        if (idx >= numBuckets) idx = numBuckets - 1;
        buckets.get(idx).add(val);
    }
    
    // Sort and concatenate
    int index = 0;
    for (List<Integer> bucket : buckets) {
        Collections.sort(bucket);
        for (int val : bucket)
            arr[index++] = val;
    }
}
```

```
  EQUAL-WIDTH vs EQUAL-DEPTH
  
  Equal-width (naive):
  [0──20] [20──40] [40──60] [60──80] [80──100]
     1       2        3       12        5
  
  Equal-depth (adaptive):
  [0──65] [65──72] [72──80] [80──88] [88──100]
     4       5        5        5        4
  
  Much more balanced! ✓
```

### Strategy 2: Histogram-Based Bucketing

```
  Step 1: Make a quick pass to build a histogram
  Step 2: Use histogram to set bucket boundaries
  
  Array: [3, 87, 45, 91, 88, 85, 90, 92, 44, 86]
  
  Quick histogram (pass 1):
  [0-25]:  1    [25-50]:  2    [50-75]:  0    [75-100]:  7
  
  Observation: 70% of data in [75-100]
  
  Refined buckets:
  [0-50]  [75-82]  [82-88]  [88-94]  [94-100]
    3        1        3        3         0
  
  Now each bucket has ≤ 3 elements ✓
```

### Strategy 3: Use a Better Hash Function

```python
import math

def bucket_sort_log(arr):
    """Use log-transform for exponential data."""
    n = len(arr)
    if n <= 1: return
    
    # Transform to make distribution more uniform
    log_arr = [(math.log(x + 1), i) for i, x in enumerate(arr)]
    log_min = min(v for v, _ in log_arr)
    log_max = max(v for v, _ in log_arr)
    
    buckets = [[] for _ in range(n)]
    r = log_max - log_min + 1e-9
    
    for log_val, orig_idx in log_arr:
        idx = int((log_val - log_min) / r * n)
        if idx >= n: idx = n - 1
        buckets[idx].append(arr[orig_idx])
    
    result = []
    for bucket in buckets:
        bucket.sort()
        result.extend(bucket)
    arr[:] = result
```

---

## Decision Framework

```
  ┌──────────────────────────────────────────┐
  │  What distribution does your data have? │
  └──────────┬───────────────────────────────┘
             │
       ┌─────┴─────┐
       ▼           ▼
  ┌─────────┐ ┌──────────┐
  │ Uniform │ │Non-uniform│
  └────┬────┘ └─────┬─────┘
       │            │
       ▼       ┌────┴─────────┐
  Use naive    ▼              ▼
  bucket   ┌────────┐  ┌──────────────┐
  sort     │Known   │  │Unknown       │
           │distrib.│  │distribution  │
           └───┬────┘  └──────┬───────┘
               │              │
               ▼              ▼
          Transform       Sample data,
          (log, sqrt)     build histogram,
          then bucket     adaptive boundaries
```

---

## Detecting Uniformity

```java
// Simple chi-squared test for uniformity
public static boolean isApproxUniform(int[] arr, int numBuckets) {
    int n = arr.length;
    int min = Arrays.stream(arr).min().getAsInt();
    int max = Arrays.stream(arr).max().getAsInt();
    double range = (double)(max - min + 1) / numBuckets;
    
    int[] counts = new int[numBuckets];
    for (int val : arr) {
        int idx = Math.min((int)((val - min) / range), numBuckets - 1);
        counts[idx]++;
    }
    
    double expected = (double) n / numBuckets;
    double chiSquared = 0;
    for (int count : counts) {
        chiSquared += (count - expected) * (count - expected) / expected;
    }
    
    // Rough threshold: if chi-squared < 2 * numBuckets, roughly uniform
    return chiSquared < 2 * numBuckets;
}
```

---

## Real-World Examples

| Data Type | Distribution | Strategy |
|-----------|-------------|----------|
| Sensor readings | Normal | Adaptive boundaries |
| File sizes | Log-normal | Log-transform |
| Ages of people | Roughly uniform [0,100] | Naive bucket sort |
| Exam scores | Clustered around 60-90 | Histogram-based |
| Random() output | Uniform [0,1) | Naive — ideal case |
| Income data | Power-law | Heavy transform needed |

---

## Summary Table

| Strategy | Overhead | Best For |
|----------|----------|----------|
| Naive (equal-width) | O(1) | Uniform data |
| Log-transform | O(n) | Exponential / log-normal |
| Histogram-based | O(n) extra pass | Unknown distribution |
| Quantile boundaries | O(n) sample + sort | Any distribution |
| Adaptive resize | O(n) amortized | Dynamic / streaming |

---

## Quick Revision Questions

1. **Why does Σnᵢ² = Θ(n) only when elements are uniformly distributed?**
2. **Give an example where naive bucket sort degrades to O(n²) on 10 elements.**
3. **How do quantile-based (equal-depth) boundaries help with skewed data?**
4. **What transformation makes exponentially distributed data more uniform?**
5. **Design a chi-squared test to decide whether to use bucket sort.**

---

[← Previous: Complexity Analysis](03-complexity-analysis.md) | [Next: Hybrid Approaches →](05-hybrid-approaches.md)

# 6.5 Top K Frequent Elements

[← Previous: K Closest Points](04-k-closest-points.md) | [Next: K Pairs Smallest Sums →](06-k-pairs-and-framework.md)

---

## Problem

Given an array of integers, return the K most frequent elements.

**Example**: nums = [1,1,1,2,2,3], K = 2 → [1, 2]

---

## Approach

1. **Count** frequencies using a hash map: O(n)
2. **Use a min-heap of size K** on frequency values: O(n log K)

```
TOP_K_FREQUENT(nums, K):
    // Step 1: Count frequencies
    freq_map = {}
    FOR num in nums:
        freq_map[num] = freq_map.get(num, 0) + 1
    
    // Step 2: Min-heap of size K (by frequency)
    min_heap = new MinHeap()    // min by frequency
    
    FOR (num, freq) in freq_map:
        IF min_heap.size < K:
            min_heap.insert((freq, num))
        ELSE IF freq > min_heap.peek().freq:
            min_heap.extract_min()
            min_heap.insert((freq, num))
    
    RETURN [num for (freq, num) in min_heap]
```

---

## Trace

```
nums = [1,1,1,2,2,3], K = 2

Step 1: freq_map = {1:3, 2:2, 3:1}

Step 2:
Process (1, freq=3): size < 2 → insert → heap = [(3,1)]
Process (2, freq=2): size < 2 → insert → heap = [(2,2), (3,1)]

  Min-heap (by freq):    (2,2)
                        /
                      (3,1)

Process (3, freq=1): 1 < peek_freq(2) → skip

Result: [2, 1] → elements with highest frequency ✅
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Time | O(n + m log K) where m = unique elements |
| Space | O(m + K) |

---

## Alternative: Bucket Sort Approach — O(n)

```
If n elements, max frequency = n
Create buckets[0..n] where buckets[f] = list of elements with frequency f
Scan from highest bucket downward, collect K elements.
```

This achieves O(n) time but O(n) space for the bucket array.

---

## Quick Check

1. **Why is the total time O(n + m log K) and not O(n log K)?**
   <details><summary>Answer</summary>
   O(n) is for building the frequency map (scanning all elements). The heap operations are O(m log K) where m is the number of unique elements. Since m ≤ n, the total is O(n + m log K).
   </details>

---

[← Previous: K Closest Points](04-k-closest-points.md) | [Next: K Pairs Smallest Sums →](06-k-pairs-and-framework.md)

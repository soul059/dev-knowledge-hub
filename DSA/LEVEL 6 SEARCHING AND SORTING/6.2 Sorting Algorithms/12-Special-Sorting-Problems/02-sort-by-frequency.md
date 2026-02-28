# Unit 12: Special Sorting Problems — Sort by Frequency

[← Previous: Dutch National Flag](01-dutch-national-flag.md) | [Next: Custom Comparators →](03-custom-comparators.md)

---

## Overview

**Sort by Frequency** means arranging elements so that elements with higher frequency appear first. If two elements have the same frequency, maintain their order of first appearance (or sort by value).

---

## Problem Statement

```
  Input:  [4, 5, 6, 5, 4, 3, 4]
  
  Frequencies: 4→3, 5→2, 6→1, 3→1
  
  Output (sort by decreasing frequency):
  [4, 4, 4, 5, 5, 6, 3]
  
  If same frequency, earlier first appearance comes first (6 before 3).
```

---

## Approach 1: HashMap + Custom Sort

```
  Step 1: Count frequencies using a HashMap
  Step 2: Sort using a custom comparator:
          - Primary: higher frequency first
          - Secondary: first appearance order (or smaller value)
```

### Java Implementation

```java
import java.util.*;

public class FrequencySort {
    
    public static int[] sortByFrequency(int[] arr) {
        // Step 1: Count frequencies
        Map<Integer, Integer> freq = new LinkedHashMap<>();
        for (int num : arr) {
            freq.put(num, freq.getOrDefault(num, 0) + 1);
        }
        
        // Step 2: Track first appearance
        Map<Integer, Integer> firstAppear = new HashMap<>();
        for (int i = 0; i < arr.length; i++) {
            firstAppear.putIfAbsent(arr[i], i);
        }
        
        // Step 3: Sort with custom comparator
        Integer[] boxed = Arrays.stream(arr).boxed().toArray(Integer[]::new);
        Arrays.sort(boxed, (a, b) -> {
            int freqCompare = freq.get(b) - freq.get(a); // higher freq first
            if (freqCompare != 0) return freqCompare;
            return firstAppear.get(a) - firstAppear.get(b); // earlier first
        });
        
        return Arrays.stream(boxed).mapToInt(Integer::intValue).toArray();
    }
    
    public static void main(String[] args) {
        int[] arr = {4, 5, 6, 5, 4, 3, 4};
        int[] result = sortByFrequency(arr);
        System.out.println(Arrays.toString(result));
        // [4, 4, 4, 5, 5, 6, 3]
    }
}
```

### Python Implementation

```python
from collections import Counter

def sort_by_frequency(arr):
    freq = Counter(arr)
    
    # Track first appearance
    first = {}
    for i, num in enumerate(arr):
        if num not in first:
            first[num] = i
    
    # Sort: higher frequency first, then by first appearance
    arr.sort(key=lambda x: (-freq[x], first[x]))
    return arr

# Example
print(sort_by_frequency([4, 5, 6, 5, 4, 3, 4]))
# [4, 4, 4, 5, 5, 6, 3]
```

---

## Approach 2: Counting Sort-Based (For Limited Range)

```
  When values are in a small range, use counting sort:
  
  Step 1: Count frequencies → freq[val] = count
  Step 2: Create (count, value) pairs
  Step 3: Sort pairs by count (descending)
  Step 4: Output each value count times
```

```java
public static int[] freqSortCounting(int[] arr, int maxVal) {
    int[] freq = new int[maxVal + 1];
    for (int num : arr) freq[num]++;
    
    // Create pairs (frequency, value)
    List<int[]> pairs = new ArrayList<>();
    for (int i = 0; i <= maxVal; i++) {
        if (freq[i] > 0) {
            pairs.add(new int[]{freq[i], i});
        }
    }
    
    // Sort by frequency descending
    pairs.sort((a, b) -> b[0] - a[0]);
    
    // Build result
    int[] result = new int[arr.length];
    int idx = 0;
    for (int[] pair : pairs) {
        for (int j = 0; j < pair[0]; j++) {
            result[idx++] = pair[1];
        }
    }
    return result;
}
```

---

## Approach 3: Bucket Sort by Frequency

```
  Key insight: frequency is bounded by n.
  Use an array of buckets where index = frequency.
  
  Step 1: Count frequencies
  Step 2: Create buckets[freq] = list of values with that frequency
  Step 3: Iterate buckets from n down to 1
  
  Time: O(n)    Space: O(n)
```

```python
def freq_sort_bucket(arr):
    from collections import Counter
    
    freq = Counter(arr)
    n = len(arr)
    
    # Buckets: buckets[i] = list of values with frequency i
    buckets = [[] for _ in range(n + 1)]
    for val, count in freq.items():
        buckets[count].append(val)
    
    result = []
    for f in range(n, 0, -1):       # highest frequency first
        for val in buckets[f]:
            result.extend([val] * f)
    
    return result

print(freq_sort_bucket([4, 5, 6, 5, 4, 3, 4]))
# [4, 4, 4, 5, 5, 6, 3]
```

---

## Step-by-Step Trace

```
  Input: [1, 1, 2, 2, 2, 3]
  
  Step 1 — Count frequencies:
  freq = {1: 2, 2: 3, 3: 1}
  
  Step 2 — Sort by frequency (descending), then value (ascending):
  Pairs: (2, 3), (1, 2), (3, 1)
  
  Step 3 — Build output:
  2 appears 3 times → [2, 2, 2]
  1 appears 2 times → [2, 2, 2, 1, 1]
  3 appears 1 time  → [2, 2, 2, 1, 1, 3]
  
  Output: [2, 2, 2, 1, 1, 3] ✓
```

---

## Variant: Sort Characters by Frequency (LeetCode 451)

```
  Input:  "tree"
  Output: "eert" or "eetr"   (e appears 2 times, t and r once each)
```

```java
public String frequencySort(String s) {
    Map<Character, Integer> freq = new HashMap<>();
    for (char c : s.toCharArray()) {
        freq.merge(c, 1, Integer::sum);
    }
    
    // Bucket sort by frequency
    List<Character>[] buckets = new List[s.length() + 1];
    for (var entry : freq.entrySet()) {
        int f = entry.getValue();
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(entry.getKey());
    }
    
    StringBuilder sb = new StringBuilder();
    for (int f = s.length(); f > 0; f--) {
        if (buckets[f] != null) {
            for (char c : buckets[f]) {
                sb.append(String.valueOf(c).repeat(f));
            }
        }
    }
    return sb.toString();
}
```

---

## Variant: Increasing Frequency Sort (LeetCode 1636)

```
  Sort by INCREASING frequency. If same frequency, sort by
  DECREASING value.
  
  Input:  [1, 1, 2, 2, 2, 3]
  Output: [3, 1, 1, 2, 2, 2]
  
  3 appears 1 time (least) → first
  1 appears 2 times → middle
  2 appears 3 times (most) → last
```

```python
def frequency_sort_increasing(nums):
    from collections import Counter
    freq = Counter(nums)
    # Lower frequency first; if tie, higher value first
    nums.sort(key=lambda x: (freq[x], -x))
    return nums
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| HashMap + Sort | O(n log n) | O(n) | General purpose |
| Counting Sort | O(n + k) | O(k) | Integer range k |
| Bucket by Freq | O(n) | O(n) | Fastest |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Problem | Sort elements by their frequency |
| Key technique | HashMap for counting + custom comparator |
| Optimal approach | Bucket sort by frequency: O(n) |
| Tie-breaking | First appearance or value order |
| Common variants | Characters, increasing freq, top-K frequent |

---

## Quick Revision Questions

1. **Sort [3, 3, 1, 1, 1, 2, 2, 2, 2] by decreasing frequency.**
2. **Why is bucket sort by frequency O(n)?**
3. **How do you handle tie-breaking when two elements have the same frequency?**
4. **Implement frequency sort for a string.**
5. **What's the difference between LeetCode 451 and 1636?**

---

[← Previous: Dutch National Flag](01-dutch-national-flag.md) | [Next: Custom Comparators →](03-custom-comparators.md)

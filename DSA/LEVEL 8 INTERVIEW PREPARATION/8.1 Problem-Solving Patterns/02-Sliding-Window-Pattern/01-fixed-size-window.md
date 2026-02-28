# Chapter 1: Fixed Size Window

## 📋 Chapter Overview
The **Fixed Size Sliding Window** is the simplest and most intuitive variant. The window size is given (or easily computed), and you slide it across the array to compute results.

---

## 🎯 Core Concept

```
  FIXED SIZE SLIDING WINDOW
  
  Array:  [ 2  1  5  1  3  2  4  7 ]
  
  k = 3 (window size)
  
  Step 1:  [2  1  5] 1  3  2  4  7    sum = 8
  Step 2:   2 [1  5  1] 3  2  4  7    sum = 7  (add 1, remove 2)
  Step 3:   2  1 [5  1  3] 2  4  7    sum = 9  (add 3, remove 1)
  Step 4:   2  1  5 [1  3  2] 4  7    sum = 6  (add 2, remove 5)
  Step 5:   2  1  5  1 [3  2  4] 7    sum = 9  (add 4, remove 1)
  Step 6:   2  1  5  1  3 [2  4  7]   sum = 13 (add 7, remove 3)
  
  Max sum = 13 ✓
```

### Key Insight: Don't Recompute — Update!

```
  BRUTE FORCE (recompute):        SLIDING WINDOW (update):
  
  Window [2,1,5]: 2+1+5 = 8      Window [2,1,5]: sum = 8
  Window [1,5,1]: 1+5+1 = 7      sum = 8 - 2 + 1 = 7   ← O(1) update!
  Window [5,1,3]: 5+1+3 = 9      sum = 7 - 1 + 3 = 9   ← O(1) update!
  
  O(n·k) per window               O(1) per slide
  Total: O(n·k)                   Total: O(n)
```

---

## 📐 The Template

```
PSEUDOCODE — Fixed Size Sliding Window:

function fixedSlidingWindow(arr, k):
    n = length(arr)
    
    // Step 1: Compute initial window (first k elements)
    windowResult = compute(arr[0..k-1])
    bestResult = windowResult
    
    // Step 2: Slide the window
    for i = k to n - 1:
        // ADD the new element entering the window
        windowResult = windowResult + arr[i]
        
        // REMOVE the element leaving the window
        windowResult = windowResult - arr[i - k]
        
        // UPDATE best result
        bestResult = best(bestResult, windowResult)
    
    return bestResult
```

### Data Flow Diagram:

```
  ┌─────────────────────────────────────────────────┐
  │  Index:   0    1    2    3    4    5    6    7   │
  │  Array: [ 2    1    5    1    3    2    4    7 ] │
  │                                                 │
  │  i = k:           ┌──────────┐                  │
  │                   │ WINDOW   │                  │
  │                   │ size = k │                  │
  │                   └──────────┘                  │
  │                        │                        │
  │            ┌───────────┼───────────┐            │
  │            ▼           │           ▼            │
  │     arr[i - k]         │       arr[i]           │
  │     LEAVES             │       ENTERS           │
  │     window             │       window           │
  │                        ▼                        │
  │                  windowResult                   │
  │                  = old - left + right            │
  └─────────────────────────────────────────────────┘
```

---

## 🧪 Step-by-Step Trace: Maximum Sum Subarray of Size K

**Problem:** Find the maximum sum of any contiguous subarray of size `k = 3`.

**Input:** `arr = [2, 1, 5, 1, 3, 2]`

```
  Initialization:
  window_sum = arr[0] + arr[1] + arr[2] = 2 + 1 + 5 = 8
  max_sum = 8

  ┌─────┬──────────┬────────────┬──────────┬────────────┬─────────┐
  │  i  │ Entering │  Leaving   │ Calculation            │ max_sum │
  ├─────┼──────────┼────────────┼────────────────────────┼─────────┤
  │  3  │ arr[3]=1 │ arr[0]=2   │ 8 - 2 + 1 = 7         │    8    │
  │  4  │ arr[4]=3 │ arr[1]=1   │ 7 - 1 + 3 = 9         │    9    │
  │  5  │ arr[5]=2 │ arr[2]=5   │ 9 - 5 + 2 = 6         │    9    │
  └─────┴──────────┴────────────┴────────────────────────┴─────────┘

  Answer: max_sum = 9 (subarray [5, 1, 3])
```

### Pseudocode:

```
function maxSumSubarrayK(arr, k):
    n = length(arr)
    if n < k: return -1
    
    // Compute first window
    window_sum = 0
    for i = 0 to k - 1:
        window_sum += arr[i]
    
    max_sum = window_sum
    
    // Slide the window
    for i = k to n - 1:
        window_sum += arr[i] - arr[i - k]
        max_sum = max(max_sum, window_sum)
    
    return max_sum
```

**Time Complexity:** O(n) — single pass  
**Space Complexity:** O(1) — only variables

---

## 🧪 Trace: Maximum Average Subarray of Size K

**Problem:** Find the contiguous subarray of size `k` with the maximum average.

**Input:** `arr = [1, 12, -5, -6, 50, 3]`, `k = 4`

```
  This is identical to max sum — just divide by k at the end!
  
  window_sum = 1 + 12 + (-5) + (-6) = 2     avg = 0.5
  
  i=4: sum = 2 - 1 + 50 = 51                avg = 12.75  ← MAX
  i=5: sum = 51 - 12 + 3 = 42               avg = 10.5
  
  Answer: 12.75 (subarray [12, -5, -6, 50])
```

---

## 🧪 Trace: Count Occurrences of Anagram

**Problem:** Given a string `s` and pattern `p`, count how many anagrams of `p` exist in `s`.

**Input:** `s = "cbaebabacd"`, `p = "abc"` (k = 3)

```
  Approach: Use frequency array as window state.
  
  Pattern frequency: {a:1, b:1, c:1}
  
  Step 1: Window "cba" → {c:1, b:1, a:1} == pattern? YES ✓ count=1
  Step 2: Window "bae" → remove 'c', add 'e' → {b:1, a:1, e:1} ≠ pattern
  Step 3: Window "aeb" → remove 'b', add 'b' → {a:1, e:1, b:1} ≠ pattern
  Step 4: Window "eba" → remove 'a', add 'a' → {e:1, b:1, a:1} ≠ pattern
  Step 5: Window "bab" → remove 'e', add 'b' → {b:2, a:1} ≠ pattern
  Step 6: Window "aba" → remove 'b', add 'a' → {a:2, b:1} ≠ pattern
  Step 7: Window "bac" → remove 'a', add 'c' → {b:1, a:1, c:1} = pattern? YES ✓ count=2
  Step 8: Window "acd" → remove 'b', add 'd' → {a:1, c:1, d:1} ≠ pattern
  
  Answer: 2
```

### Pseudocode:

```
function countAnagrams(s, p):
    k = length(p)
    n = length(s)
    count = 0
    
    patternFreq = frequency(p)
    windowFreq = frequency(s[0..k-1])
    
    if windowFreq == patternFreq: count += 1
    
    for i = k to n - 1:
        // Add new character
        windowFreq[s[i]] += 1
        
        // Remove old character
        windowFreq[s[i - k]] -= 1
        if windowFreq[s[i - k]] == 0:
            delete windowFreq[s[i - k]]
        
        // Compare
        if windowFreq == patternFreq: count += 1
    
    return count
```

**Time:** O(n · 26) = O(n) for lowercase English  
**Space:** O(26) = O(1) for frequency arrays

---

## 🔑 Common Fixed Window Problems

| Problem | Window Content | Update Operation |
|---------|---------------|-----------------|
| Max sum subarray of size K | sum | add/subtract |
| Max average subarray | sum (divide at end) | add/subtract |
| Count anagrams | frequency map | increment/decrement |
| Max of all windows of size K | deque (monotonic) | maintain max |
| Contains duplicate within K distance | hash set | add/remove |
| Max vowels in substring of length K | count | ±1 |

---

## ⚠️ Common Mistakes

```
  ╔══════════════════════════════════════════════════╗
  ║  MISTAKE 1: Off-by-one in window boundaries     ║
  ║  ──────────────────────────────────────────────  ║
  ║  WRONG: for i = k-1 to n    (starts too early)  ║
  ║  RIGHT: for i = k to n-1    (first slide at k)  ║
  ╠══════════════════════════════════════════════════╣
  ║  MISTAKE 2: Forgetting to initialize first window║
  ║  ──────────────────────────────────────────────  ║
  ║  Always compute arr[0..k-1] before the loop!    ║
  ╠══════════════════════════════════════════════════╣
  ║  MISTAKE 3: Not handling n < k                   ║
  ║  ──────────────────────────────────────────────  ║
  ║  If array is smaller than window, return early!  ║
  ╚══════════════════════════════════════════════════╝
```

---

## 📊 Complexity Analysis

| Aspect | Brute Force | Fixed Window |
|--------|-------------|-------------|
| Recomputation | Every window from scratch | Update in O(1) |
| Time Complexity | O(n · k) | O(n) |
| Space Complexity | O(1) | O(1) for sum, O(k) for frequency |
| Number of operations | n × k | n + k ≈ n |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Fixed Window | Window size is constant (given as k) |
| Core Idea | Slide by adding new element, removing old |
| Template | Init first window → loop from k → update |
| Time | O(n) — single pass |
| Space | O(1) for numeric, O(k) for hash-based |
| Key Signal | "subarray/substring of size k" |
| Common Mistake | Off-by-one errors, forgetting initial window |

---

## ❓ Quick Revision Questions

1. **What makes a "fixed" sliding window fixed?**
   > The window size k is constant throughout — it never grows or shrinks.

2. **Why is sliding window O(n) instead of O(n·k)?**
   > Because we update the window in O(1) by adding the new element and removing the old, instead of recomputing from scratch.

3. **What do you do before the sliding loop starts?**
   > Compute the initial window: process the first k elements.

4. **How do you handle the "count anagrams" variant?**
   > Use a frequency map as the window state, increment/decrement on slide, and compare with pattern frequency.

5. **What edge case must you always check?**
   > When `n < k` (array is smaller than the window size — return early).

6. **In the loop `for i = k to n-1`, what does `arr[i - k]` represent?**
   > The element that is leaving the window (the leftmost element of the previous window).

---

[Next: Variable Size Window →](02-variable-size-window.md)

[← Back to Unit 2](../README.md)

# Chapter 5: Optimization Tricks

## 📋 Chapter Overview
Advanced techniques to optimize sliding window solutions — from reducing comparisons to leveraging hash maps and deques for better performance.

---

## 🔧 Trick 1: Use a `matched` Counter Instead of Map Comparison

Instead of comparing two frequency maps (O(26) each time), track how many characters are fully matched:

```
  SLOW: Compare entire maps each step — O(26) per step
  ┌──────────────────────────────────────────┐
  │  if windowMap == targetMap:  // O(26)    │
  │      // valid window                     │
  └──────────────────────────────────────────┘

  FAST: Track matched count — O(1) per step
  ┌──────────────────────────────────────────┐
  │  // When adding char:                    │
  │  if windowMap[c] == targetMap[c]:        │
  │      matched += 1                        │
  │                                          │
  │  // When removing char:                  │
  │  if windowMap[c] < targetMap[c]:         │
  │      matched -= 1                        │
  │                                          │
  │  if matched == totalDistinctChars:       │
  │      // valid window — O(1) check!       │
  └──────────────────────────────────────────┘
```

```
  Performance comparison:
  
  Map comparison:   O(26) per window slide × O(n) slides = O(26n)
  Matched counter:  O(1) per window slide × O(n) slides = O(n)
  
  26x speedup for English lowercase problems!
```

---

## 🔧 Trick 2: HashMap with Last Index (Jump Optimization)

For "longest substring without repeating characters," instead of shrinking one step at a time, **jump** the left pointer:

```
  SLOW: Shrink one by one
  ┌──────────────────────────────────────────┐
  │  "abcba"                                 │
  │  right=4 ('a' duplicate)                 │
  │  left=0 → remove 'a' → left=1           │
  │  Still need to keep moving...            │
  │  Many steps to skip past the duplicate!  │
  └──────────────────────────────────────────┘

  FAST: Store last index, jump directly
  ┌──────────────────────────────────────────┐
  │  lastIndex = {a:0, b:1, c:2}             │
  │  right=3 ('b' at index 3, last seen at 1)│
  │  left = max(left, lastIndex['b'] + 1)    │
  │  left = max(0, 2) = 2   ← Jump!         │
  │  No intermediate steps needed!           │
  └──────────────────────────────────────────┘
```

### Pseudocode:

```
function longestNoRepeat_Optimized(s):
    left = 0
    maxLen = 0
    lastIndex = {}           // char → last seen index
    
    for right = 0 to len(s) - 1:
        if s[right] in lastIndex:
            // Jump left pointer past the last occurrence
            left = max(left, lastIndex[s[right]] + 1)
        
        lastIndex[s[right]] = right
        maxLen = max(maxLen, right - left + 1)
    
    return maxLen
```

**Why `max(left, ...)`?** The last occurrence might be *before* the current left pointer (already removed), so we don't move left backward.

---

## 🔧 Trick 3: Array Instead of HashMap

For problems with a known, small character set (like lowercase English), use a fixed-size array instead of a hash map:

```
  HashMap:                          Array:
  ┌──────────────────────┐          ┌──────────────────────┐
  │  freq = {}           │          │  freq = int[26]      │
  │  freq['a'] += 1      │          │  freq[0] += 1        │
  │  // Hash function    │          │  // Direct index     │
  │  // Collision handling│          │  // No overhead      │
  │  // Dynamic resize   │          │  // Fixed size       │
  │                      │          │                      │
  │  Overhead: ~50ns/op  │          │  Overhead: ~5ns/op   │
  └──────────────────────┘          └──────────────────────┘
  
  10x faster for character frequency problems!
```

```
  // Convert character to index:
  index = char - 'a'       // 'a'→0, 'b'→1, ..., 'z'→25
  
  // Convert index back to character:
  char = index + 'a'
```

---

## 🔧 Trick 4: Avoid Shrink Loop with "Don't Shrink" Technique

For some "longest" problems, instead of shrinking the window, you can **freeze** the window size and only grow:

```
  STANDARD: Shrink window when invalid
  ┌──────────────────────────────────────────┐
  │  while invalid:                          │
  │      remove left, left++                 │
  │  // Window may shrink significantly       │
  └──────────────────────────────────────────┘

  OPTIMIZED: Never shrink, only shift
  ┌──────────────────────────────────────────┐
  │  if invalid:                             │
  │      remove left, left++     // shift 1  │
  │  // Window never gets smaller!           │
  │  // Result = n - left (max size seen)    │
  └──────────────────────────────────────────┘
```

### Example: Longest Repeating Character Replacement

```
function charReplacement_Optimized(s, k):
    freq = int[26]
    maxFreq = 0
    left = 0
    
    for right = 0 to len(s) - 1:
        freq[s[right] - 'A'] += 1
        maxFreq = max(maxFreq, freq[s[right] - 'A'])
        
        // If window is invalid, shift (not shrink)
        if (right - left + 1) - maxFreq > k:
            freq[s[left] - 'A'] -= 1
            left += 1
            // NOTE: we don't update maxFreq downward!
        
    return len(s) - left
```

**Why maxFreq doesn't need to decrease:**
- We only care about the **maximum** valid window
- A valid window needs `len - maxFreq ≤ k`
- If maxFreq decreases, we can't get a larger window anyway
- So we only grow when maxFreq increases

---

## 🔧 Trick 5: Prefix Sum + HashMap for Subarray Sum

When the window condition involves a sum equal to a target (not ≥ or ≤), sliding window alone doesn't work with negative numbers. Use **prefix sum + hash map**:

```
  Problem: Count subarrays with sum = k
  
  ┌──────────────────────────────────────────────────┐
  │  KEY INSIGHT:                                    │
  │  If prefixSum[j] - prefixSum[i] == k             │
  │  Then subarray arr[i+1..j] has sum k             │
  │                                                  │
  │  So we need: prefixSum[j] - k = prefixSum[i]     │
  │  Store prefix sums in hash map!                  │
  │                                                  │
  │  prefix:    0  1  3  3  4  1                      │
  │  arr:      [1  2  0  1 -3]                        │
  │  target k = 3                                     │
  │                                                  │
  │  At prefix=3: need 3-3=0, found 1 time → count++ │
  │  At prefix=3: need 3-3=0, found 1 time → count++ │
  │  At prefix=4: need 4-3=1, found 1 time → count++ │
  └──────────────────────────────────────────────────┘
```

```
PSEUDOCODE:

function subarraySum(arr, k):
    prefixCount = {0: 1}    // base case: empty prefix
    currentSum = 0
    count = 0
    
    for num in arr:
        currentSum += num
        
        // Check if (currentSum - k) exists as a previous prefix
        if (currentSum - k) in prefixCount:
            count += prefixCount[currentSum - k]
        
        // Store current prefix sum
        prefixCount[currentSum] += 1
    
    return count
```

---

## 📊 Optimization Summary

| Trick | Problem Type | Improvement |
|-------|-------------|-------------|
| Matched counter | Anagram/permutation checking | O(26) → O(1) per step |
| Last index jump | No-repeat substring | Eliminate inner while loop |
| Array vs HashMap | Fixed alphabet problems | ~10x constant factor |
| Don't-shrink | Longest window problems | Simpler logic, fewer ops |
| Prefix sum + map | Subarray sum = K | Handles negative numbers |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Matched Counter | Track satisfied characters instead of full map comparison |
| Jump Optimization | Store last index, move left pointer directly |
| Array over Map | Use fixed array for known small alphabets |
| Don't Shrink | For "longest" problems, shift window instead of shrinking |
| Prefix Sum | Use when exact sum needed and negatives present |
| General Rule | Optimize the inner operations, keep O(n) overall |

---

## ❓ Quick Revision Questions

1. **Why track `matched` count instead of comparing full maps?**
   > Comparing maps is O(26) per step, but updating and checking `matched` is O(1) per step — more efficient.

2. **How does the "jump" optimization for no-repeat work?**
   > Store the last index of each character. When a duplicate is found, jump `left` directly to `lastIndex[char] + 1` instead of incrementing one by one.

3. **When should you use a fixed array instead of a hash map?**
   > When the character set is known and small (e.g., 26 lowercase letters, 128 ASCII chars).

4. **Why doesn't `maxFreq` need to decrease in the "don't shrink" technique?**
   > Because we only care about finding a larger valid window, which requires maxFreq to increase. Decreasing maxFreq can never lead to a better result.

5. **When does sliding window fail and you need prefix sum + hash map?**
   > When the array has negative numbers and you need subarrays with an exact sum (not ≥ or ≤).

6. **What is the base case in the prefix sum hash map?**
   > `{0: 1}` — representing the empty prefix with sum 0, which allows detecting subarrays starting from index 0.

---

[← Previous: Classic Problems](04-classic-problems.md) | [Next: When to Apply →](06-when-to-apply.md)

[← Back to README](../README.md)

# Chapter 2: Frequency Counting

## 📋 Chapter Overview
Using hash maps for frequency counting: majority element, first unique character, minimum window, and subarray sum equals K.

---

## 📝 Problem 1: Majority Element (LeetCode 169)

**Problem:** Element appearing more than n/2 times. Guaranteed to exist.

### Hash Map Approach

```
function majorityElement(nums):
    count = {}
    for each num:
        count[num] = count.getOrDefault(num, 0) + 1
        if count[num] > n / 2:
            return num
```

### Boyer-Moore Voting (O(1) space)

```
function majorityElement(nums):
    candidate = nums[0]
    votes = 1
    
    for i = 1 to n-1:
        if votes == 0:
            candidate = nums[i]
            votes = 1
        elif nums[i] == candidate:
            votes++
        else:
            votes--
    
    return candidate
```

### Trace

```
  nums = [2, 2, 1, 1, 1, 2, 2]
  
  candidate=2, votes=1
  i=1: 2==2 → votes=2
  i=2: 1≠2 → votes=1
  i=3: 1≠2 → votes=0
  i=4: votes==0 → candidate=1, votes=1
  i=5: 2≠1 → votes=0
  i=6: votes==0 → candidate=2, votes=1
  
  Answer: 2 ✓
```

---

## 📝 Problem 2: First Unique Character (LeetCode 387)

```
function firstUniqChar(s):
    count = {}
    for each char c in s:
        count[c] = count.getOrDefault(c, 0) + 1
    
    for i = 0 to len(s) - 1:
        if count[s[i]] == 1:
            return i
    
    return -1
```

### Trace

```
  s = "leetcode"
  count: l=1, e=3, t=1, c=1, o=1, d=1
  
  Scan: l(1) → index 0 ← first unique
  Answer: 0
```

---

## 📝 Problem 3: Subarray Sum Equals K (LeetCode 560)

**Problem:** Count substrings with sum equal to k.

```
  Key insight: if prefixSum[j] - prefixSum[i] == k,
  then subarray (i, j] sums to k.
  
  So for each j, count how many previous prefix sums equal
  prefixSum[j] - k.
```

```
function subarraySum(nums, k):
    prefixCount = {0: 1}   // prefix sum → count
    currentSum = 0
    count = 0
    
    for each num in nums:
        currentSum += num
        if (currentSum - k) in prefixCount:
            count += prefixCount[currentSum - k]
        prefixCount[currentSum] = prefixCount.getOrDefault(currentSum, 0) + 1
    
    return count
```

### Trace

```
  nums = [1, 1, 1], k = 2
  prefixCount = {0:1}, sum = 0, count = 0
  
  num=1: sum=1. sum-k=-1 not in map. prefixCount={0:1, 1:1}
  num=1: sum=2. sum-k=0 in map (count 1). count=1. prefixCount={0:1, 1:1, 2:1}
  num=1: sum=3. sum-k=1 in map (count 1). count=2. prefixCount={0:1, 1:1, 2:1, 3:1}
  
  Answer: 2 (subarrays [1,1] at index 0-1 and 1-2)
```

**Complexity:** O(n) time, O(n) space

---

## 📝 Problem 4: Minimum Window Substring (LeetCode 76)

**Problem:** Find smallest window in `s` containing all characters of `t`.

```
function minWindow(s, t):
    need = frequency map of t
    have = 0, required = number of unique chars in t
    windowCount = {}
    
    left = 0
    minLen = INF, minStart = 0
    
    for right = 0 to len(s) - 1:
        c = s[right]
        windowCount[c] = windowCount.getOrDefault(c, 0) + 1
        
        if c in need and windowCount[c] == need[c]:
            have++
        
        while have == required:
            // Update minimum
            if right - left + 1 < minLen:
                minLen = right - left + 1
                minStart = left
            
            // Shrink from left
            leftC = s[left]
            windowCount[leftC]--
            if leftC in need and windowCount[leftC] < need[leftC]:
                have--
            left++
    
    return minLen == INF ? "" : s[minStart : minStart + minLen]
```

### Trace

```
  s = "ADOBECODEBANC", t = "ABC"
  need = {A:1, B:1, C:1}, required = 3
  
  Expand right until have==3:
  ...right=5 ('C'): window="ADOBEC", have=3.
    Shrink: remove 'A' → have=2. Window found: "ADOBEC" (len 6)
  
  Continue expanding...
  right=10 ('A'): window="CODEBA", have=3 at some point
  ...right=12 ('C'): window="BANC", have=3.
    Shrink: "BANC" is len 4 < 6. Update.
    Remove 'B' → have=2.
  
  Answer: "BANC"
```

---

## 📝 Problem 5: Longest Substring Without Repeating Characters (LeetCode 3)

```
function lengthOfLongestSubstring(s):
    lastSeen = {}   // char → last index
    maxLen = 0
    left = 0
    
    for right = 0 to len(s) - 1:
        if s[right] in lastSeen and lastSeen[s[right]] >= left:
            left = lastSeen[s[right]] + 1
        lastSeen[s[right]] = right
        maxLen = max(maxLen, right - left + 1)
    
    return maxLen
```

---

## 📊 Frequency Counting Patterns

| Pattern | Hash Map Stores | Time |
|---------|----------------|------|
| Count occurrences | element → frequency | O(n) |
| Prefix sum + count | prefixSum → count of that sum | O(n) |
| Sliding window match | char → count in window vs needed | O(n) |
| Character last index | char → last seen position | O(n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Frequency map | Count occurrences for majority, uniqueness checks |
| Prefix sum + map | Count subarrays with given sum in O(n) |
| Window counting | Track have/required for substring matching |
| Boyer-Moore | O(1) space majority element via cancellation |

---

## ❓ Revision Questions

1. **Boyer-Moore voting: why does it work?** → The majority element survives all cancellations because it appears more than n/2 times.
2. **Subarray sum equals K: why prefix sum map?** → `prefixSum[j] - prefixSum[i] == k` means subarray (i,j] sums to k; map counts valid i's for each j.
3. **Min window: what does `have == required` mean?** → All unique characters from t are present in the window with sufficient frequency.
4. **Why initialize prefix count map with {0:1}?** → Handles subarrays starting from index 0 (entire prefix sums to k).
5. **Longest substring without repeating: why track last index?** → To jump left pointer forward past the last occurrence of the repeated character.

---

[← Previous: Hash Map Fundamentals](01-hashmap-fundamentals.md) | [Next: Hash Set Techniques →](03-hashset-techniques.md)

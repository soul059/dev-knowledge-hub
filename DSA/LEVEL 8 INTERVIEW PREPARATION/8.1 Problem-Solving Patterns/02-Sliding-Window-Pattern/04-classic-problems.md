# Chapter 4: Classic Problems

## 📋 Chapter Overview
Detailed walkthroughs of the most frequently asked sliding window problems in interviews. Each problem includes the approach, pseudocode, step-by-step trace, and complexity analysis.

---

## 🧪 Problem 1: Minimum Window Substring

**Problem:** Given strings `s` and `t`, find the minimum window in `s` that contains all characters of `t`.

**Input:** `s = "ADOBECODEBANC"`, `t = "ABC"`  
**Output:** `"BANC"`

### Approach:

```
  ┌─────────────────────────────────────────────────────┐
  │  1. Count frequency of each char in t               │
  │  2. Use sliding window on s                         │
  │  3. Expand right pointer — add chars to window      │
  │  4. When window contains all chars of t → VALID     │
  │  5. Shrink left to find minimum valid window        │
  │  6. Track the smallest valid window                 │
  └─────────────────────────────────────────────────────┘
```

### Step-by-Step Trace:

```
  s = "ADOBECODEBANC"    t = "ABC"
  need = {A:1, B:1, C:1}    required = 3 (unique chars needed)
  have = 0 (unique chars satisfied)

  right=0  'A': window{A:1}     have=1  (A satisfied)
  right=1  'D': window{A:1,D:1} have=1
  right=2  'O': have=1
  right=3  'B': window{..B:1}   have=2  (B satisfied)
  right=4  'E': have=2
  right=5  'C': window{..C:1}   have=3  ← ALL SATISFIED!
           Window: "ADOBEC" len=6      Record: [0,5] len=6
           Shrink: remove 'A' → have=2  (A no longer satisfied)
           left=1
  
  right=6  'O': have=2
  right=7  'D': have=2
  right=8  'E': have=2
  right=9  'B': have=2
  right=10 'A': window{..A:1}   have=3  ← ALL SATISFIED!
           Window: "DOBECODEBA" — too long, keep shrinking
           Shrink 'D','O','B','E','C','O','D','E'...
           Eventually: "BANC" len=4    Record: [9,12] len=4
  
  Answer: "BANC"
```

### Pseudocode:

```
function minWindow(s, t):
    if len(t) > len(s): return ""
    
    need = frequency(t)
    required = len(need)        // unique chars needed
    have = 0                    // unique chars satisfied
    windowFreq = {}
    
    left = 0
    minLen = INFINITY
    minStart = 0
    
    for right = 0 to len(s) - 1:
        char = s[right]
        windowFreq[char] += 1
        
        // Check if this char is now fully satisfied
        if char in need AND windowFreq[char] == need[char]:
            have += 1
        
        // Shrink while all chars are satisfied
        while have == required:
            // Update minimum
            if (right - left + 1) < minLen:
                minLen = right - left + 1
                minStart = left
            
            // Remove left char from window
            leftChar = s[left]
            windowFreq[leftChar] -= 1
            if leftChar in need AND windowFreq[leftChar] < need[leftChar]:
                have -= 1
            left += 1
    
    return s[minStart..minStart+minLen-1] if minLen != INFINITY else ""
```

**Time:** O(|s| + |t|)     **Space:** O(|s| + |t|)

---

## 🧪 Problem 2: Longest Repeating Character Replacement

**Problem:** Given string `s` and integer `k`, find the length of the longest substring containing the same letter after performing at most `k` replacements.

**Input:** `s = "AABABBA"`, `k = 1`  
**Output:** `4` (Replace one 'A' in "AABA" to get "AAAA" or "ABBB"→"BBBB")

### Key Insight:

```
  ┌──────────────────────────────────────────────────────┐
  │  In any window of length L:                          │
  │    - mostFrequent = count of the most common char    │
  │    - replacements needed = L - mostFrequent          │
  │    - Window is valid if: L - mostFrequent ≤ k        │
  │                                                      │
  │  Example: "AABA", k=1                                │
  │    L = 4, mostFrequent = 3 (A appears 3 times)       │
  │    Replacements = 4 - 3 = 1 ≤ k ✓                    │
  └──────────────────────────────────────────────────────┘
```

### Trace:

```
  s = "AABABBA", k = 1
  
  ┌──────┬──────┬──────────────┬──────┬────────┬────────┐
  │ right│ char │ freq         │maxF  │len-maxF│ maxLen │
  ├──────┼──────┼──────────────┼──────┼────────┼────────┤
  │  0   │  A   │ {A:1}        │  1   │ 0 ≤ 1 │   1    │
  │  1   │  A   │ {A:2}        │  2   │ 0 ≤ 1 │   2    │
  │  2   │  B   │ {A:2,B:1}    │  2   │ 1 ≤ 1 │   3    │
  │  3   │  A   │ {A:3,B:1}    │  3   │ 1 ≤ 1 │   4    │
  │  4   │  B   │ {A:3,B:2}    │  3   │ 2 > 1 │ SHRINK │
  │      │      │ remove A:    │      │        │        │
  │      │      │ {A:2,B:2} l=1│  2   │ 2 > 1 │ SHRINK │
  │      │      │ {A:1,B:2} l=2│  2   │ 1 ≤ 1 │   4    │
  │  5   │  B   │ {A:1,B:3}    │  3   │ 1 ≤ 1 │   4    │
  │  6   │  A   │ {A:2,B:3}    │  3   │ 2 > 1 │ SHRINK │
  │      │      │ {A:2,B:2} l=3│  2   │ 2 > 1 │ SHRINK │
  │      │      │ {A:2,B:1} l=4│  2   │ 1 ≤ 1 │   4    │
  └──────┴──────┴──────────────┴──────┴────────┴────────┘
  
  Answer: 4
```

**Time:** O(n)     **Space:** O(26) = O(1)

---

## 🧪 Problem 3: Max Consecutive Ones III

**Problem:** Given binary array and integer `k`, return the max number of consecutive 1's if you can flip at most `k` zeros.

**Input:** `nums = [1,1,1,0,0,0,1,1,1,1,0]`, `k = 2`  
**Output:** `6`

### Approach:

```
  ┌────────────────────────────────────────────────────┐
  │  Reframe: Find longest subarray with at most k 0s │
  │                                                    │
  │  [1,1,1,0,0,0,1,1,1,1,0]   k=2                    │
  │                                                    │
  │  Window: count zeros in window                     │
  │  If zeros > k → shrink from left                   │
  │  Track max window length                           │
  └────────────────────────────────────────────────────┘
```

### Trace:

```
  nums = [1,1,1,0,0,0,1,1,1,1,0]   k=2
  
   1  1  1  0  0  0  1  1  1  1  0
   0  1  2  3  4  5  6  7  8  9  10
  
  [1  1  1  0  0] 0  1  1  1  1  0    zeros=2, len=5 ✓
   1 [1  1  0  0  0] 1  1  1  1  0    zeros=3>2, shrink
   1  1 [1  0  0  0] 1  1  1  1  0    zeros=3>2, shrink
   1  1  1 [0  0  0] 1  1  1  1  0    zeros=3>2, shrink
   1  1  1  0 [0  0  1  1  1  1] 0    zeros=2, len=6 ✓ ← MAX
   1  1  1  0  0 [0  1  1  1  1  0]   zeros=2, len=6 ✓

  Answer: 6 (subarray from index 4 to 9, flipping zeros at 4,5)
```

**Time:** O(n)     **Space:** O(1)

---

## 🧪 Problem 4: Permutation in String

**Problem:** Given `s1` and `s2`, return true if `s2` contains a permutation of `s1`.

**Input:** `s1 = "ab"`, `s2 = "eidbaooo"`  
**Output:** `true` (s2 contains "ba" which is a permutation of "ab")

### Approach: Fixed window of size |s1|, compare frequency maps

```
  ┌──────────────────────────────────────────────────┐
  │  Fixed window of size k = len(s1)                │
  │  Slide over s2 and compare frequency with s1     │
  │                                                  │
  │  s2 = "e i d b a o o o"                          │
  │        ├───┤                 {e:1,i:1} ≠ {a:1,b:1}│
  │          ├───┤               {i:1,d:1}            │
  │            ├───┤             {d:1,b:1}            │
  │              ├───┤           {b:1,a:1} = {a:1,b:1}│
  │                              ← MATCH!             │
  └──────────────────────────────────────────────────┘
```

**Time:** O(|s2|)     **Space:** O(26) = O(1)

---

## 🧪 Problem 5: Subarrays with K Different Integers

**Problem:** Return number of subarrays with exactly K distinct integers.

**Input:** `nums = [1,2,1,2,3]`, `k = 2`  
**Output:** `7`

### Using the Exact K Trick:

```
  exactly(2) = atMost(2) - atMost(1)
  
  atMost(2):
  right=0: [1]       distinct=1 ≤ 2  → count += 1  (total: 1)
  right=1: [1,2]     distinct=2 ≤ 2  → count += 2  (total: 3)
  right=2: [1,2,1]   distinct=2 ≤ 2  → count += 3  (total: 6)
  right=3: [1,2,1,2] distinct=2 ≤ 2  → count += 4  (total: 10)
  right=4: [1,2,1,2,3] distinct=3>2, shrink to [2,3] → count += 2 (total: 12)
  atMost(2) = 12
  
  atMost(1):
  right=0: [1]       → count += 1 (1)
  right=1: [1,2] distinct=2>1, shrink → [2] → count += 1 (2)
  right=2: [2,1] distinct=2>1, shrink → [1] → count += 1 (3)
  right=3: [1,2] distinct=2>1, shrink → [2] → count += 1 (4)
  right=4: [2,3] distinct=2>1, shrink → [3] → count += 1 (5)
  atMost(1) = 5
  
  exactly(2) = 12 - 5 = 7 ✓
```

The 7 subarrays: [1,2], [2,1], [1,2,1], [2,1,2], [1,2,1,2], [2,3], [1,2,1,2,3]... wait let me recount: [1,2], [2,1], [1,2], [2,3], [1,2,1], [2,1,2], [1,2,1,2] = 7 ✓

---

## 📊 Classic Problems Summary

| Problem | Type | Key Insight | Complexity |
|---------|------|-------------|------------|
| Max Sum Subarray K | Fixed | Slide and update sum | O(n) / O(1) |
| Longest No Repeat | Variable (longest) | Set tracks duplicates | O(n) / O(min(n,26)) |
| Min Window Substring | Variable (shortest) | Freq map matching | O(n) / O(n) |
| Char Replacement | Variable (longest) | len - maxFreq ≤ k | O(n) / O(1) |
| Max Consecutive Ones | Variable (longest) | Count zeros ≤ k | O(n) / O(1) |
| Permutation in String | Fixed | Freq comparison | O(n) / O(1) |
| Exactly K Distinct | Count (exact) | atMost(k) - atMost(k-1) | O(n) / O(n) |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Min Window Substring | Track "have" vs "required" unique characters |
| Char Replacement | Window valid when `len - maxFreq ≤ k` |
| Max Ones III | Reframe as "longest subarray with ≤ k zeros" |
| Permutation Check | Fixed window with frequency comparison |
| Exactly K | `atMost(K) - atMost(K-1)` trick |

---

## ❓ Quick Revision Questions

1. **In minimum window substring, what determines when the window is valid?**
   > When `have == required` — i.e., all unique characters from `t` are satisfied in the window.

2. **How do you check if a window has a valid character replacement?**
   > The window is valid if `window_length - max_frequency_char ≤ k`.

3. **How do you reframe "max consecutive ones with k flips"?**
   > Find the longest subarray containing at most k zeros.

4. **What trick converts "exactly K" into sliding window?**
   > `exactly(K) = atMost(K) - atMost(K-1)`.

5. **What is the time complexity of minimum window substring?**
   > O(|s| + |t|) — one pass over s with at most two passes over each character.

6. **Why does the frequency comparison approach work for permutation in string?**
   > A permutation has the same character frequency as the original — so comparing frequency maps of a fixed-size window equals checking for a permutation.

---

[← Previous: Template and Variations](03-template-and-variations.md) | [Next: Optimization Tricks →](05-optimization-tricks.md)

[← Back to README](../README.md)

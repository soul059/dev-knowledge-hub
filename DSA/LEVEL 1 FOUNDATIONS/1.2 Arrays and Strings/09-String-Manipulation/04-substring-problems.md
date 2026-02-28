# Chapter 4: Substring Problems

[← Previous: Anagram Problems](03-anagram-problems.md) | [Back to README](../README.md) | [Next: String Compression →](05-string-compression.md)

---

## Overview

Substring problems — finding, counting, or optimizing over contiguous subsequences of a string — are among the most commonly asked interview questions. They combine sliding window, hash maps, two pointers, and dynamic programming techniques.

---

## Key Substring Patterns

```
  ┌──────────────────────────────────────────────────────┐
  │          SUBSTRING PROBLEM CATEGORIES                │
  │                                                      │
  │  1. Longest substring with property X               │
  │     → Sliding window (expand/contract)              │
  │                                                      │
  │  2. Shortest substring with property X              │
  │     → Sliding window (shrink when valid)            │
  │                                                      │
  │  3. Count substrings with property X                │
  │     → Sliding window + "at most k" trick            │
  │     → Or prefix sum + hash map                      │
  │                                                      │
  │  4. Check if substring exists                       │
  │     → Pattern matching (KMP, Rabin-Karp, Z)         │
  │                                                      │
  │  5. All distinct substrings                         │
  │     → Suffix array / trie                           │
  └──────────────────────────────────────────────────────┘
```

---

## 1. Longest Substring Without Repeating Characters (LeetCode 3)

```
  Input: "abcabcbb"
  Output: 3 ("abc")

  Sliding window with hash set:

  Window:  [a] b c a b c b b      set={a}, len=1
           [a b] c a b c b b      set={a,b}, len=2
           [a b c] a b c b b      set={a,b,c}, len=3
           a [b c a] b c b b      remove a, add a: set={b,c,a}, len=3
           a b [c a b] c b b      set={c,a,b}, len=3
           a b c [a b c] b b      remove c, set={a,b,c}, len=3
           a b c a [b c] b b      'b' duplicate...
           → keep shrinking until no duplicate
```

### Algorithm

```
FUNCTION longestSubstringNoRepeat(s):
    seen = HashMap<char, lastIndex>
    maxLen = 0
    left = 0

    FOR right = 0 TO length(s) - 1:
        IF s[right] IN seen AND seen[s[right]] ≥ left:
            left = seen[s[right]] + 1

        seen[s[right]] = right
        maxLen = MAX(maxLen, right - left + 1)

    RETURN maxLen
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(min(n, σ)) where σ = alphabet size |

---

## 2. Longest Substring with At Most K Distinct Characters (LeetCode 340)

```
  Input: s = "eceba", k = 2
  Output: 3 ("ece")

  Sliding window with frequency map:
  - Expand right
  - When distinct count > k, shrink left
  - Track maximum window size
```

### Algorithm

```
FUNCTION longestKDistinct(s, k):
    freq = HashMap<char, count>
    left = 0, maxLen = 0

    FOR right = 0 TO length(s) - 1:
        freq[s[right]]++

        WHILE size(freq) > k:
            freq[s[left]]--
            IF freq[s[left]] == 0:
                DELETE freq[s[left]]
            left++

        maxLen = MAX(maxLen, right - left + 1)

    RETURN maxLen
```

### Trace

```
  s = "eceba", k = 2

  r=0: freq={e:1}, distinct=1 ≤ 2, maxLen=1
  r=1: freq={e:1,c:1}, distinct=2 ≤ 2, maxLen=2
  r=2: freq={e:2,c:1}, distinct=2 ≤ 2, maxLen=3
  r=3: freq={e:2,c:1,b:1}, distinct=3 > 2!
       shrink: remove s[0]='e', freq={e:1,c:1,b:1}, still 3
       shrink: remove s[1]='c', freq={e:1,b:1}, distinct=2 ✓
       left=2, maxLen=max(3, 3-2+1)=3
  r=4: freq={e:1,b:1,a:1}, distinct=3 > 2!
       shrink: remove s[2]='e', freq={b:1,a:1}, distinct=2 ✓
       left=3, maxLen=max(3, 4-3+1)=3

  Answer: 3 ("ece")
```

---

## 3. Substring with Concatenation of All Words (LeetCode 30)

```
  Find starting indices where a concatenation of ALL words 
  (in any order) appears as a substring.

  s = "barfoothefoobarman"
  words = ["foo","bar"]  (each length 3)

  "barfoo" at index 0 → "bar"+"foo" ✓
  "foobar" at index 9 → "foo"+"bar" ✓

  Output: [0, 9]
```

### Algorithm

```
  Each word has length w. Total length = k × w.
  
  Slide a window of size k × w through s.
  Split window into k chunks of size w.
  Check if chunk frequencies match word frequencies.

  For each starting offset (0 to w-1):
    Use sliding window of w-sized words.

  Time: O(n × w) where n = text length, w = word length
```

---

## 4. Count Substrings with Exactly K Distinct Characters

```
  Trick: exactly(k) = atMost(k) - atMost(k-1)

  FUNCTION countExactlyK(s, k):
      RETURN countAtMostK(s, k) - countAtMostK(s, k-1)

  FUNCTION countAtMostK(s, k):
      freq = HashMap
      left = 0, count = 0

      FOR right = 0 TO length(s) - 1:
          freq[s[right]]++
          WHILE size(freq) > k:
              freq[s[left]]--
              IF freq[s[left]] == 0: DELETE freq[s[left]]
              left++
          count += right - left + 1  // all substrings ending at right

      RETURN count
```

### Why `right - left + 1`?

```
  Window [left..right] and all sub-windows ending at right:
  [left..right], [left+1..right], ..., [right..right]
  That's (right - left + 1) substrings.
  All have ≤ k distinct characters (since window does).
```

---

## 5. Repeated Substring Pattern (LeetCode 459)

```
  Check if string can be formed by repeating a substring.

  "abab" → "ab" × 2 ✓
  "abc"  → no repeating pattern ✗
  "abcabcabc" → "abc" × 3 ✓
```

### Elegant Solution

```
  s+s contains s as a substring at a position OTHER than 
  0 or len(s) iff s has a repeating pattern.

  s = "abab"
  s+s = "abababab"
  Remove first and last char: "bababa b"
  Does it contain "abab"?
  "bababab" → "abab" found at index 1 ✓

  FUNCTION hasRepeatingPattern(s):
      doubled = s + s
      RETURN s IN doubled[1 .. 2n-2]  // exclude first and last char positions
```

### Divisor Approach

```
  If s has length n and repeating unit of length k:
  - k must divide n
  - s[0..k-1] repeated n/k times = s

  FOR k = 1 TO n/2:
      IF n % k == 0:
          unit = s[0..k-1]
          IF unit repeated (n/k) times == s:
              RETURN true
  RETURN false
```

---

## 6. Longest Common Substring

```
  Find the longest string that is a substring of BOTH s1 and s2.

  s1 = "ABCXYZ"
  s2 = "XYZABC"
  Longest common substring: "ABC" or "XYZ" (length 3)

  DP approach:
  dp[i][j] = length of longest common substring ending 
             at s1[i-1] and s2[j-1]

  IF s1[i-1] == s2[j-1]:
      dp[i][j] = dp[i-1][j-1] + 1
  ELSE:
      dp[i][j] = 0

  Answer = max value in dp table
```

### DP Table

```
  s1 = "ABC", s2 = "AEC"

        ""  A  E  C
    ""   0  0  0  0
     A   0  1  0  0
     B   0  0  0  0
     C   0  0  0  1

  Max = 1 (only single chars match: "A" or "C")
```

| Metric | Value |
|--------|-------|
| Time | O(n × m) |
| Space | O(n × m), optimizable to O(min(n, m)) |

---

## Summary Table

| Problem | Technique | Time | Space |
|---------|-----------|------|-------|
| Longest no repeat | Sliding window + hashmap | O(n) | O(σ) |
| At most k distinct | Sliding window + freq map | O(n) | O(k) |
| Exactly k distinct | atMost(k) - atMost(k-1) | O(n) | O(k) |
| Repeated pattern | s+s trick or divisors | O(n) | O(n) |
| Common substring | DP table | O(nm) | O(nm) |
| Concatenation of words | Word-level sliding window | O(nw) | O(kw) |

---

## Quick Revision Questions

1. **How does sliding window solve "longest substring" problems?** What determines when to shrink?

2. **Explain the "at most k" trick** for counting substrings with exactly k distinct characters.

3. **Why does `right - left + 1` count all qualifying substrings** ending at `right`?

4. **How does the s+s trick detect repeating patterns?** Prove why it works.

5. **Trace the sliding window** for longest substring without repeating characters on "pwwkew".

6. **What is the DP recurrence** for longest common substring? How does it differ from longest common subsequence?

---

[← Previous: Anagram Problems](03-anagram-problems.md) | [Back to README](../README.md) | [Next: String Compression →](05-string-compression.md)

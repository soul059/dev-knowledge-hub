# Chapter 3: Anagram Problems

[← Previous: Palindrome Detection](02-palindrome-detection.md) | [Back to README](../README.md) | [Next: Substring Problems →](04-substring-problems.md)

---

## Overview

An **anagram** is a rearrangement of the characters of one string to form another. Anagram problems test your understanding of character frequency counting, hash maps, and sorting — fundamental techniques used across many string problems.

---

## Definition

```
  "listen" and "silent" are anagrams
  
  l-i-s-t-e-n  →  rearrange  →  s-i-l-e-n-t

  Same characters, same frequencies, different order.

  Formally: s1 is an anagram of s2 iff
  - length(s1) == length(s2)
  - frequency of each character is the same
```

---

## 1. Valid Anagram (LeetCode 242)

### Approach 1: Sort Both Strings

```
  sort("listen") = "eilnst"
  sort("silent") = "eilnst"
  Equal → anagram ✓

  Time: O(n log n)
  Space: O(n) or O(1) depending on sort
```

### Approach 2: Frequency Count (Optimal)

```
  Count character frequencies and compare.

  "listen":  {l:1, i:1, s:1, t:1, e:1, n:1}
  "silent":  {s:1, i:1, l:1, e:1, n:1, t:1}
  Same → anagram ✓
```

### Single Array Approach

```
FUNCTION isAnagram(s1, s2):
    IF length(s1) ≠ length(s2): RETURN false

    freq = array of size 26, all zeros

    FOR i = 0 TO length(s1) - 1:
        freq[s1[i] - 'a']++    // increment for s1
        freq[s2[i] - 'a']--    // decrement for s2

    FOR i = 0 TO 25:
        IF freq[i] ≠ 0: RETURN false

    RETURN true
```

### Trace

```
  s1 = "anagram", s2 = "nagaram"

  freq after processing all chars:
  s1: a→3, n→1, g→1, r→1, m→1
  s2: n→1, a→3, g→1, r→1, m→1

  Using single array (++ for s1, -- for s2):
  a: +3 -3 = 0 ✓
  n: +1 -1 = 0 ✓
  g: +1 -1 = 0 ✓
  r: +1 -1 = 0 ✓
  m: +1 -1 = 0 ✓
  All zero → anagram ✓
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(1) — fixed 26-element array |

---

## 2. Group Anagrams (LeetCode 49)

```
  Input:  ["eat","tea","tan","ate","nat","bat"]
  Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]

  Key: anagrams share the same SORTED form (canonical form).

  "eat" → sort → "aet"
  "tea" → sort → "aet"     → same group!
  "ate" → sort → "aet"     → same group!
  "tan" → sort → "ant"
  "nat" → sort → "ant"     → same group!
  "bat" → sort → "abt"
```

### Algorithm

```
FUNCTION groupAnagrams(strs):
    groups = HashMap<String, List<String>>

    FOR each word in strs:
        key = sort(word)                // canonical form
        groups[key].append(word)

    RETURN groups.values()
```

### Alternative Key: Frequency String

```
  Instead of sorting, use character count as key:
  "eat" → "1a0b0c0d1e...1t..." → "#1#0#0#0#1#0#0...#1..."
  
  This avoids O(k log k) sort per word.
  Time: O(n × k) where k = max word length
```

| Metric | Value |
|--------|-------|
| Time (sort key) | O(n × k log k) |
| Time (count key) | O(n × k) |
| Space | O(n × k) |

---

## 3. Find All Anagrams in a String (LeetCode 438)

```
  Find all starting indices where an anagram of pattern 
  appears in text.

  text = "cbaebabacd", pattern = "abc"

  Anagrams of "abc": "abc", "acb", "bac", "bca", "cab", "cba"

  Sliding window of size 3:
  "cba" at 0 → anagram ✓
  "bae" at 1 → no
  "aeb" at 2 → no
  "eba" at 3 → no
  "bab" at 4 → no
  "aba" at 5 → no
  "bac" at 6 → anagram ✓
  "acd" at 7 → no

  Output: [0, 6]
```

### Algorithm (Sliding Window + Frequency)

```
FUNCTION findAnagrams(text, pattern):
    m = length(pattern)
    n = length(text)
    IF m > n: RETURN []

    pFreq = frequency count of pattern
    wFreq = frequency count of text[0..m-1]
    result = []

    IF pFreq == wFreq: result.append(0)

    FOR i = m TO n-1:
        // Add new character (right side)
        wFreq[text[i]]++
        // Remove old character (left side)
        wFreq[text[i - m]]--

        IF pFreq == wFreq:
            result.append(i - m + 1)

    RETURN result
```

### Optimized: Match Counter

```
  Instead of comparing full frequency arrays each time,
  track HOW MANY characters have matching frequency.

  matches = 0 (out of 26 letters)
  When matches == 26: it's an anagram!

  Update matches incrementally:
  - When adding a char: if freq was matching before, matches--
                         update freq
                         if freq matches now, matches++
  - Same logic for removing a char.

  Each step: O(1) instead of O(26)!
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(1) — two 26-element arrays |

---

## 4. Minimum Window Anagram

```
  Find smallest window in text containing all chars of pattern 
  (with at least their frequency).

  This is essentially the "Minimum Window Substring" problem
  from the Sliding Window unit (Unit 4, Chapter 5).

  Use variable-size sliding window with frequency matching.
```

---

## 5. Check If Two Strings Are Close (LeetCode 1657)

```
  Two strings are "close" if you can make them equal using:
  1. Swap any two characters
  2. Transform all occurrences of one char to another and vice versa

  Key insight: strings are close iff:
  - Same set of characters
  - Same multiset of frequencies (sorted frequencies match)

  "cabbba" → freq: {a:2, b:3, c:1} → sorted: [1, 2, 3]
  "abbccc" → freq: {a:1, b:2, c:3} → sorted: [1, 2, 3]
  Same character set ✓, same sorted frequencies ✓ → close!
```

---

## Anagram Detection Patterns

```
  ┌──────────────────────────────────────────────────────┐
  │            ANAGRAM TECHNIQUE TOOLKIT                  │
  │                                                      │
  │  1. Frequency array (size 26 for lowercase)         │
  │     → O(n) check, O(1) space                        │
  │                                                      │
  │  2. Sort as canonical form                           │
  │     → O(n log n), simple to code                    │
  │                                                      │
  │  3. Sliding window + frequency                       │
  │     → O(n) for finding anagram occurrences          │
  │                                                      │
  │  4. Prime product (each char → prime number)        │
  │     → Unique product per anagram (overflow risk!)    │
  │                                                      │
  │  5. XOR (limited: same chars cancel, but collisions) │
  │     → Not reliable alone                            │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Problem | Technique | Time | Space |
|---------|-----------|------|-------|
| Valid anagram | Frequency count | O(n) | O(1) |
| Group anagrams | Sort/count as hash key | O(nk log k) | O(nk) |
| Find all anagrams | Sliding window + freq | O(n) | O(1) |
| Minimum anagram window | Variable sliding window | O(n) | O(1) |
| Close strings | Sorted frequencies | O(n) | O(1) |

---

## Quick Revision Questions

1. **Why is frequency counting better than sorting for anagram check?** Compare complexities.

2. **How do you efficiently group anagrams?** What serves as the hash map key?

3. **Trace the sliding window approach** for finding anagrams of "ab" in "abab".

4. **Can XOR alone determine if two strings are anagrams?** Give a counterexample.

5. **What is the "match counter" optimization** for finding anagrams in a string?

6. **How do you check if two strings are "close"?** Why do sorted frequencies matter?

---

[← Previous: Palindrome Detection](02-palindrome-detection.md) | [Back to README](../README.md) | [Next: Substring Problems →](04-substring-problems.md)

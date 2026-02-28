# Chapter 1.5 — Anagrams

> **Unit 1: String Basics Review** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Two strings are **anagrams** if one can be rearranged to form the other.
Anagram problems are among the most frequently asked in interviews and contests.
This chapter covers the core idea, multiple detection techniques, and the main
problem variations.

---

## 1. Definition

```
  🔑  Strings s and t are anagrams if and only if they contain the 
      same characters with the same frequencies.

  "listen"  ↔  "silent"     ✓  Anagram
  "hello"   ↔  "world"      ✗  Not anagram
  "aab"     ↔  "aba"        ✓  Anagram
  "aab"     ↔  "aabb"       ✗  Different lengths
```

### Necessary Conditions

```
  1.  |s| == |t|                      (same length)
  2.  freq(c, s) == freq(c, t)       for every character c
```

---

## 2. Detection Methods

### Method 1: Sorting

```
  sort(s) == sort(t)  →  anagram

  "listen" → "eilnst"
  "silent" → "eilnst"
  Same → Anagram ✓

  Time:  O(n log n)
  Space: O(n)  or  O(1) if in-place sort
```

### Method 2: Frequency Counting (Preferred)

```
  COUNT_COMPARE(s, t):
      if |s| ≠ |t|: return false
      freq[26] ← {0}
      for i = 0 to |s| - 1:
          freq[s[i] - 'a'] += 1
          freq[t[i] - 'a'] -= 1
      for i = 0 to 25:
          if freq[i] ≠ 0: return false
      return true

  Time:  O(n)
  Space: O(1)  (fixed-size array)
```

### Trace

```
  s = "listen",  t = "silent"

  Process each character pair:
  ─────────────────────────────
  i=0: freq['l'-'a']++ = 1,  freq['s'-'a']-- = -1
  i=1: freq['i'-'a']++ = 1,  freq['i'-'a']-- = 0
  i=2: freq['s'-'a']++ = 0,  freq['l'-'a']-- = 0
  i=3: freq['t'-'a']++ = 1,  freq['e'-'a']-- = -1
  i=4: freq['e'-'a']++ = 0,  freq['n'-'a']-- = -1
  i=5: freq['n'-'a']++ = 0,  freq['t'-'a']-- = 0

  All freq = 0 → Anagram ✓
```

### Method 3: Prime Number Product

```
  Assign each letter a unique prime:
    a=2, b=3, c=5, d=7, e=11, ...

  Product of primes for s == Product for t → anagram

  "abc" → 2 × 3 × 5 = 30
  "bca" → 3 × 5 × 2 = 30  ✓

  ⚠️ Risk of integer overflow for long strings!
```

### Method 4: Bitmask (for "unique chars" problems)

```
  XOR all characters — but this only works for detecting
  if each char appears an EVEN number of times overall.
  NOT a reliable anagram check, but useful for permutation parity.
```

---

## 3. Anagram Variations

### 3.1 Group Anagrams

```
  Input:  ["eat", "tea", "tan", "ate", "nat", "bat"]

  Group by sorted key:
    "aet" → ["eat", "tea", "ate"]
    "ant" → ["tan", "nat"]
    "abt" → ["bat"]

  Algorithm:
  ──────────
  MAP = HashMap<String, List<String>>
  for word in words:
      key = sort(word)
      MAP[key].append(word)
  return MAP.values()

  Time:  O(n × k log k)  where k = max word length
  Space: O(n × k)
```

### 3.2 Find All Anagrams in a String (Sliding Window)

```
  s = "cbaebabacd",  p = "abc"

  Find all starting indices where an anagram of p occurs in s.

  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
  │ c │ b │ a │ e │ b │ a │ b │ a │ c │ d │
  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
    0   1   2   3   4   5   6   7   8   9

  Window size = |p| = 3

  Sliding window with frequency matching:
  ─────────────────────────────────────────
  Window [0,2] = "cba" → freq matches "abc" → index 0 ✓
  Window [1,3] = "bae" → no match
  Window [2,4] = "aeb" → no match
  Window [3,5] = "eba" → no match
  Window [4,6] = "bab" → no match
  Window [5,7] = "aba" → no match
  Window [6,8] = "bac" → freq matches "abc" → index 6 ✓
  Window [7,9] = "acd" → no match

  Answer: [0, 6]
```

### Efficient Sliding Window Algorithm

```
  FIND_ANAGRAMS(s, p):
      pFreq[26] ← count frequencies of p
      wFreq[26] ← {0}
      result ← []
      matchCount ← 0       // number of chars with matching freq

      for right = 0 to |s| - 1:
          // Add s[right] to window
          c ← s[right] - 'a'
          wFreq[c] += 1
          if wFreq[c] == pFreq[c]: matchCount += 1

          // Remove leftmost if window too large
          if right >= |p|:
              left ← right - |p|
              d ← s[left] - 'a'
              if wFreq[d] == pFreq[d]: matchCount -= 1
              wFreq[d] -= 1

          // Check if all 26 chars match
          if matchCount == number_of_distinct_chars_in_p:
              result.append(right - |p| + 1)

      return result

  Time:  O(n)  where n = |s|
  Space: O(1)
```

### 3.3 Minimum Number of Steps to Make Two Strings Anagrams

```
  s = "bab",  t = "aba"

  freq_s: a=1, b=2
  freq_t: a=2, b=1

  Excess in s:  b has 1 extra
  Excess in t:  a has 1 extra

  Steps = 1 (change one 'b' in s to 'a', or vice versa)

  General formula:
      steps = Σ max(0, freq_s[c] - freq_t[c])   for all c
```

### 3.4 Minimum Window Substring (Anagram Superset)

```
  Find the smallest window in s that contains all chars of t
  (with at least the same frequency).

  s = "ADOBECODEBANC",  t = "ABC"
  Answer: "BANC"

  This is a generalization of the anagram window problem.
```

---

## 4. Anagram Properties

```
  ┌───────────────────────────────────────────────────────┐
  │  Property                     │  Note                 │
  ├───────────────────────────────┼───────────────────────┤
  │  Reflexive:  s ~ s            │  Always true          │
  │  Symmetric:  s ~ t → t ~ s   │  Order doesn't matter │
  │  Transitive: s ~ t, t ~ u    │  s ~ u                │
  │              → same freq      │                       │
  │  → Equivalence relation!      │                       │
  └───────────────────────────────┴───────────────────────┘

  The sorted form is the "canonical representative" of each
  equivalence class (anagram group).
```

---

## 5. Frequency Array Visualization

```
  s = "anagram"

  freq:
  Index: 0  1  2  3  4  5  6  7  8  9 10 11 12 13 ...
  Char:  a  b  c  d  e  f  g  h  i  j  k  l  m  n ...
  Count: 3  0  0  0  0  0  1  0  0  0  0  0  1  1 ...
         ▲                 ▲              ▲  ▲
         a×3               g×1            m  n

  Two strings are anagrams ⟺ their frequency arrays are identical.
```

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Definition | Same characters, same frequencies |
| Freq counting | O(n) time, O(1) space — best method |
| Sorting method | O(n log n) — simple but slower |
| Group anagrams | Sort each word → use as hash key |
| Sliding window anagrams | Fixed window + freq matching → O(n) |
| Canonical form | Sorted string is the class representative |
| Equivalence | Anagram relation is reflexive, symmetric, transitive |

---

## ❓ Quick Revision Questions

1. **What is the most efficient way to check if two strings are anagrams?**
   <details><summary>Answer</summary>Frequency counting: increment for one string, decrement for the other, then verify all counts are zero. O(n) time, O(1) space.</details>

2. **How do you find all anagram occurrences of pattern p in string s?**
   <details><summary>Answer</summary>Sliding window of size |p| over s, maintaining a frequency array and a match counter. O(n) total.</details>

3. **What is the canonical form used to group anagrams?**
   <details><summary>Answer</summary>The sorted string — all anagrams sort to the same result.</details>

4. **Can two strings of different lengths be anagrams?**
   <details><summary>Answer</summary>No — anagrams must have identical character frequencies, which requires equal length.</details>

5. **How do you compute the minimum swaps to make s an anagram of t?**
   <details><summary>Answer</summary>Count excess characters: steps = Σ max(0, freq_s[c] - freq_t[c]) for all c.</details>

---

| [⬅️ Previous: Subsequences](04-subsequences.md) | [Next: Palindromes ➡️](06-palindromes.md) |
|:---|---:|

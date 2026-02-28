# Chapter 2: Palindrome Detection

[← Previous: Reverse String Problems](01-reverse-string-problems.md) | [Back to README](../README.md) | [Next: Anagram Problems →](03-anagram-problems.md)

---

## Overview

A **palindrome** is a string that reads the same forwards and backwards. Palindrome problems are interview favorites because they combine string manipulation with two-pointer techniques and dynamic programming.

---

## Definition

```
  Palindrome: s == reverse(s)

  "racecar"  → r a c e c a r  ✓ palindrome
  "hello"    → o l l e h      ✗ not palindrome
  "a"        → a              ✓ palindrome (trivially)
  ""         → ""             ✓ palindrome (empty)
  "abba"     → a b b a        ✓ even-length palindrome
```

---

## 1. Check If Palindrome — Two Pointers

```
  Compare from outside in:

  "racecar"
   L           R
   r     =     r  ✓   L++, R--
    a   =   a     ✓
     c = c         ✓
      e            L ≥ R, done → palindrome!
```

### Algorithm

```
FUNCTION isPalindrome(s):
    left = 0
    right = length(s) - 1

    WHILE left < right:
        IF s[left] ≠ s[right]:
            RETURN false
        left = left + 1
        right = right - 1

    RETURN true
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(1) |

---

## 2. Valid Palindrome (LeetCode 125)

```
  Ignore non-alphanumeric characters and case.

  Input:  "A man, a plan, a canal: Panama"
  Clean:  "amanaplanacanalpanama"
  Check:  palindrome ✓

  Input:  "race a car"
  Clean:  "raceacar"
  Check:  not palindrome ✗
```

### Algorithm

```
FUNCTION isValidPalindrome(s):
    left = 0
    right = length(s) - 1

    WHILE left < right:
        WHILE left < right AND NOT isAlphanumeric(s[left]):
            left = left + 1
        WHILE left < right AND NOT isAlphanumeric(s[right]):
            right = right - 1

        IF toLower(s[left]) ≠ toLower(s[right]):
            RETURN false
        left = left + 1
        right = right - 1

    RETURN true
```

---

## 3. Valid Palindrome II (LeetCode 680)

```
  Can it become a palindrome by removing AT MOST ONE character?

  Input: "abca"
  Remove 'b' → "aca" ✓ palindrome
  OR remove 'c' → "aba" ✓ palindrome
  Answer: true

  Input: "abc"
  No single removal works. Answer: false.
```

### Algorithm

```
FUNCTION validPalindrome(s):
    left = 0, right = length(s) - 1

    WHILE left < right:
        IF s[left] ≠ s[right]:
            // Try skipping left OR skipping right
            RETURN isPalin(s, left+1, right) OR isPalin(s, left, right-1)
        left++, right--

    RETURN true

FUNCTION isPalin(s, l, r):
    WHILE l < r:
        IF s[l] ≠ s[r]: RETURN false
        l++, r--
    RETURN true
```

### Trace

```
  s = "abcba"  → true (already palindrome)

  s = "abcda"
  L=0, R=4: 'a'='a' ✓
  L=1, R=3: 'b'≠'d' → try:
    isPalin("abcda", 2, 3): "cd" → 'c'≠'d' → false
    isPalin("abcda", 1, 2): "bc" → 'b'≠'c' → false
  Answer: false

  s = "abca"
  L=0, R=3: 'a'='a' ✓
  L=1, R=2: 'b'≠'c' → try:
    isPalin("abca", 2, 2): "c" → true ✓
  Answer: true
```

---

## 4. Palindromic Substrings (LeetCode 647)

```
  Count all palindromic substrings.

  Input: "aaa"
  Palindromes: "a"(×3), "aa"(×2), "aaa"(×1)
  Count: 6

  Approach: Expand around center
```

### Expand Around Center

```
  Every palindrome has a CENTER:
  
  Odd length:   "aba"  → center at 'b'
  Even length:  "abba" → center between 'b' and 'b'

  For a string of length n:
  - n possible odd centers: indices 0, 1, ..., n-1
  - n-1 possible even centers: between indices i and i+1
  - Total: 2n - 1 centers

  For each center, expand outward while palindrome holds.

  "aba":
  Center at 0: "a" → count 1
  Center at 1: "b" → count 1, expand: "aba" ✓ → count 1
  Center at 2: "a" → count 1
  Even centers: none match
  Total: 4  wait, let me recount...
  Actually "aaa" example:
  Center 0: "a" (1), expand "aa" nope (out of bounds left)
  Center 1: "a" (1), expand "aaa" ✓ (1)  
  Center 2: "a" (1)
  Even(0,1): "aa" (1)
  Even(1,2): "aa" (1)
  Total: 6 ✓
```

### Algorithm

```
FUNCTION countPalindromicSubstrings(s):
    count = 0
    n = length(s)

    FOR i = 0 TO n-1:
        // Odd-length palindromes centered at i
        count += expandCount(s, i, i)
        // Even-length palindromes centered between i and i+1
        count += expandCount(s, i, i + 1)

    RETURN count

FUNCTION expandCount(s, left, right):
    count = 0
    WHILE left ≥ 0 AND right < length(s) AND s[left] == s[right]:
        count++
        left--, right++
    RETURN count
```

| Metric | Value |
|--------|-------|
| Time | O(n²) |
| Space | O(1) |

---

## 5. Longest Palindromic Substring (LeetCode 5)

```
  Find the longest palindromic substring.

  Input: "babad"
  Output: "bab" (or "aba" — both valid)

  Use expand around center, track the longest.
```

### Algorithm

```
FUNCTION longestPalindrome(s):
    start = 0, maxLen = 1

    FOR i = 0 TO length(s) - 1:
        // Odd
        len1 = expand(s, i, i)
        // Even
        len2 = expand(s, i, i + 1)

        len = MAX(len1, len2)
        IF len > maxLen:
            maxLen = len
            start = i - (len - 1) / 2

    RETURN s[start .. start + maxLen - 1]

FUNCTION expand(s, left, right):
    WHILE left ≥ 0 AND right < length(s) AND s[left] == s[right]:
        left--, right++
    RETURN right - left - 1
```

### Trace

```
  s = "babad"

  i=0: odd expand(0,0): "b" len=1
       even expand(0,1): 'b'≠'a' len=0

  i=1: odd expand(1,1): "a" → expand: 'b'='b'? 
       s[0]='b', s[2]='b' → "bab" len=3 ✓
       even expand(1,2): 'a'≠'b' len=0

  i=2: odd expand(2,2): "b" → expand: 'a'='a'?
       s[1]='a', s[3]='a' → "aba" len=3 (tie)
       even expand(2,3): 'b'≠'a' len=0

  i=3: odd expand(3,3): "a" len=1
       even expand(3,4): 'a'≠'d' len=0

  i=4: odd expand(4,4): "d" len=1

  Longest: "bab" (or "aba"), length 3
```

---

## 6. Manacher's Algorithm (O(n) — Bonus)

```
  Finds ALL palindromic substrings in O(n).
  
  Key idea: reuse previously computed palindrome info.
  Similar to Z-algorithm's "Z-box" concept.

  Input:  "abacaba"
  Insert separators: "#a#b#a#c#a#b#a#"
  Compute palindrome radius for each position.

  Time: O(n) — optimal!
  
  (Rarely asked in interviews but good to know exists)
```

---

## Summary Table

| Problem | Technique | Time | Space |
|---------|-----------|------|-------|
| Is palindrome | Two pointers | O(n) | O(1) |
| Valid palindrome (with cleanup) | Two pointers + filter | O(n) | O(1) |
| Palindrome with 1 deletion | Two pointers + branch | O(n) | O(1) |
| Count palindromic substrings | Expand around center | O(n²) | O(1) |
| Longest palindromic substring | Expand around center | O(n²) | O(1) |
| All palindromes (optimal) | Manacher's | O(n) | O(n) |

---

## Quick Revision Questions

1. **Check if "A man, a plan, a canal: Panama" is a palindrome.** What pre-processing do you need?

2. **How does expand around center work?** Why are there 2n-1 centers for a string of length n?

3. **Trace the "valid palindrome with one deletion"** on "abcbca".

4. **Count all palindromic substrings** of "abc". List each one.

5. **What is Manacher's Algorithm?** Why is it O(n) instead of O(n²)?

6. **If a string of length n has k palindromic substrings, what are the bounds on k?** (Minimum and maximum)

---

[← Previous: Reverse String Problems](01-reverse-string-problems.md) | [Back to README](../README.md) | [Next: Anagram Problems →](03-anagram-problems.md)

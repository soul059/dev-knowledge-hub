# Chapter 8.1 — Longest Palindromic Substring

> **Unit 8: Manacher's Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The **Longest Palindromic Substring (LPS)** problem asks for the longest
contiguous substring that reads the same forwards and backwards. This
chapter covers the problem, naive approaches, and sets the stage for
Manacher's linear-time solution.

---

## 1. Problem Definition

```
  Given a string S, find the longest substring that is a palindrome.

  Examples:
    "babad"    → "bab" or "aba"  (length 3)
    "cbbd"     → "bb"            (length 2)
    "racecar"  → "racecar"       (length 7)
    "abcde"    → "a" (any char)  (length 1)
    "aacabdkacaa" → "aca"        (length 3)
```

---

## 2. Approach 1: Brute Force O(n³)

```
  Check EVERY substring:

  for i = 0 to n-1:
      for j = i to n-1:
          if S[i..j] is palindrome:     ← O(n) check
              update max

  "babad":
    Substrings: b, ba, bab✓, baba, babad,
                a, ab, aba✓, abad,
                b, ba, bad,
                a, ad,
                d

  Time: O(n²) substrings × O(n) check = O(n³)
  Too slow for n > 1000.
```

---

## 3. Approach 2: Expand Around Center O(n²)

```
  Key insight: Every palindrome has a CENTER.
  Expand outward from each possible center.

  Centers for "babad" (length 5):
  ┌─────────────────────────────────────────┐
  │  Odd-length centers:  b  a  b  a  d     │
  │  (positions 0-4)       0  1  2  3  4    │
  │                                          │
  │  Even-length centers: ba ab ba ad        │
  │  (between positions)  0.5 1.5 2.5 3.5   │
  │                                          │
  │  Total centers: 2n - 1                   │
  └─────────────────────────────────────────┘

  Expansion from center at index 1 ('a'):
    Expand: S[0]='b' vs S[2]='b' → match! → "bab"
    Expand: S[-1] out of bounds → stop
    Palindrome: "bab" (length 3)

  Expansion from center at index 2 ('b'):
    Expand: S[1]='a' vs S[3]='a' → match! → "aba"  wait...
    Actually the center 'b':
      S[2] = 'b' → palindrome "b"
      S[1]='a' vs S[3]='a' → match → "aba"  
      S[0]='b' vs S[4]='d' → no match → stop
    Palindrome: "aba" (length 3)
```

---

## 4. Expand Around Center Implementation

```python
def longest_palindrome(s: str) -> str:
    if not s:
        return ""
    
    start, max_len = 0, 1
    
    def expand(left, right):
        nonlocal start, max_len
        while left >= 0 and right < len(s) and s[left] == s[right]:
            if right - left + 1 > max_len:
                start = left
                max_len = right - left + 1
            left -= 1
            right += 1
    
    for i in range(len(s)):
        expand(i, i)        # Odd-length palindromes
        expand(i, i + 1)    # Even-length palindromes
    
    return s[start:start + max_len]
```

---

## 5. Detailed Trace

```
  S = "cbbd"

  Center i=0 ('c'):
    Odd:  expand(0,0) → "c" (len 1)
    Even: expand(0,1) → 'c'≠'b' → stop

  Center i=1 ('b'):
    Odd:  expand(1,1) → "b"
          expand(0,2) → 'c'≠'b' → stop (len 1)
    Even: expand(1,2) → 'b'='b' → "bb" (len 2) ← max!
          expand(0,3) → 'c'≠'d' → stop

  Center i=2 ('b'):
    Odd:  expand(2,2) → "b" (len 1)
    Even: expand(2,3) → 'b'≠'d' → stop

  Center i=3 ('d'):
    Odd:  expand(3,3) → "d" (len 1)
    Even: expand(3,4) → out of bounds

  Result: "bb" starting at index 1, length 2
```

---

## 6. Why O(n²) Is Not Enough

```
  For competitive programming and large inputs (n = 10^6):
  O(n²) is too slow!

  Redundancy in expand-around-center:
  ┌─────────────────────────────────────────────────┐
  │  String: "abaaba"                               │
  │                                                  │
  │  Center 1: finds "aba" (positions 0-2)          │
  │  Center 4: finds "aba" (positions 3-5)          │
  │                                                  │
  │  These palindromes are MIRRORS of each other    │
  │  around the larger palindrome "abaaba".         │
  │  Manacher's algorithm exploits this symmetry!   │
  └─────────────────────────────────────────────────┘

  Complexity comparison:
  ┌─────────────────┬────────┐
  │ Method          │ Time   │
  ├─────────────────┼────────┤
  │ Brute force     │ O(n³)  │
  │ Expand center   │ O(n²)  │
  │ DP              │ O(n²)  │
  │ Manacher's      │ O(n)   │ ← goal!
  └─────────────────┴────────┘
```

---

## 7. DP Approach O(n²) — For Reference

```
  dp[i][j] = True if S[i..j] is a palindrome

  Base cases:
    dp[i][i] = True           (single char)
    dp[i][i+1] = (S[i]==S[i+1])  (two chars)

  Transition:
    dp[i][j] = dp[i+1][j-1] AND S[i]==S[j]

  Fill order: by increasing length (j - i)

  S = "babad"
  ┌───┬───┬───┬───┬───┬───┐
  │   │ 0 │ 1 │ 2 │ 3 │ 4 │
  ├───┼───┼───┼───┼───┼───┤
  │ 0 │ T │ F │ T │ F │ F │
  │ 1 │   │ T │ F │ T │ F │
  │ 2 │   │   │ T │ F │ F │
  │ 3 │   │   │   │ T │ F │
  │ 4 │   │   │   │   │ T │
  └───┴───┴───┴───┴───┴───┘

  dp[0][2]=T → "bab" is palindrome
  dp[1][3]=T → "aba" is palindrome
  Both length 3 → answer is either.
```

---

## 📝 Summary Table

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force | O(n³) | O(1) | Check all substrings |
| DP table | O(n²) | O(n²) | dp[i][j] = palindrome? |
| Expand center | O(n²) | O(1) | 2n-1 centers |
| Manacher's | O(n) | O(n) | Next chapter! |

---

## ❓ Quick Revision Questions

1. **How many possible centers exist in a string of length n?**
   <details><summary>Answer</summary>2n - 1 centers: n centers for odd-length palindromes (at each character) and n-1 centers for even-length palindromes (between adjacent characters).</details>

2. **What is the key limitation of expand-around-center?**
   <details><summary>Answer</summary>It doesn't reuse information from previously found palindromes. Each center expansion starts from scratch, leading to redundant comparisons. Manacher's resolves this.</details>

3. **In the DP approach, why do we fill by increasing length?**
   <details><summary>Answer</summary>dp[i][j] depends on dp[i+1][j-1] (a shorter substring). Filling by increasing length ensures dependencies are computed before they're needed.</details>

4. **What is the space complexity of expand-around-center?**
   <details><summary>Answer</summary>O(1) extra space — we only track the best start index and length. No table needed.</details>

5. **Why is "every palindrome has a center" useful?**
   <details><summary>Answer</summary>Instead of checking O(n²) substrings, we check 2n-1 centers. Each center expansion is at most O(n), giving O(n²) total — much better than brute force O(n³).</details>

---

| [⬅️ Previous: XOR with Trie](../07-Trie-Applications/06-xor-with-trie.md) | [Next: Transformed String ➡️](02-transformed-string.md) |
|:---|---:|

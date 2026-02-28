# Chapter 8.5 — Applications of Manacher's Algorithm

> **Unit 8: Manacher's Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Beyond finding the longest palindromic substring, Manacher's P-array
enables solving many palindrome-related problems efficiently.

---

## 1. All Palindromic Substrings

```
  The P-array directly tells us ALL palindromic substrings.

  S = "abaab"
  T = "^#a#b#a#a#b#$"
  P = [0,0,1,0,3,0,1,2,1,0,0,0,0]

  Palindromes found:
  P[2]=1 → "a"       (len 1, start 0)
  P[4]=3 → "aba"     (len 3, start 0)
  P[6]=1 → "a"       (len 1, start 2)
  P[7]=2 → "aa"      (len 2, start 2)
  P[8]=1 → "b"       (len 1, start 3)
  ... also single chars at other positions

  Total distinct palindromic substrings: enumerate from P.

  Count palindromic substrings:
  ┌─────────────────────────────────────────────┐
  │  Total count = Σ ⌈P[i]/2⌉ for all i        │
  │  (Each P[i] contains P[i]/2 palindromes    │
  │   of different lengths centered at i)       │
  └─────────────────────────────────────────────┘
```

---

## 2. Palindrome Partitioning

```
  Problem: Minimum cuts to partition S into palindromic substrings.

  S = "aab" → "aa" + "b" → 1 cut

  Manacher's helps precompute: is S[i..j] a palindrome?

  Using P-array:
  For each center c with radius r:
    Mark all substrings centered at c with lengths 1,3,5,...,2r+1
    as palindromes.

  Then use DP:
    dp[i] = min cuts for S[0..i]
    dp[i] = min(dp[j-1] + 1) for all j where S[j..i] is palindrome

  ┌─────────────────────────────────────────────────┐
  │  Manacher's: O(n) to build P-array              │
  │  DP:         O(n²) worst case for partitioning   │
  │  But O(n) palindrome check instead of O(n) each  │
  │  → Overall still O(n²), but with better constant │
  └─────────────────────────────────────────────────┘
```

---

## 3. Longest Palindromic Prefix / Suffix

```
  Longest palindromic prefix: longest palindrome starting at index 0.

  Using P-array from Manacher's:
  Find position i where the palindrome [i-P[i] .. i+P[i]]
  starts at position 0 in the original string.

  In transformed string:
  Palindrome starting at original index 0 ↔ starts at T index 2.
  Find i where i - P[i] = 2 (or equivalently = 1 for # boundary).

  Example: S = "abacaba"
  P[8]=7 → palindrome T[1..15] → S[0..6] = "abacaba"
  Starts at 0 → longest palindromic prefix = "abacaba"

  Example: S = "abacd"
  P[4]=3 → palindrome T[1..7] → S[0..2] = "aba"
  Longest palindromic prefix = "aba"

  Application: shortest palindrome by prepending characters.
  ┌────────────────────────────────────────────────┐
  │  Shortest palindrome = reverse(S[k+1:]) + S    │
  │  where S[0:k] is the longest palindromic prefix │
  └────────────────────────────────────────────────┘
```

---

## 4. Online Palindrome Detection

```
  Process characters one by one, maintaining palindrome info.

  Variation of Manacher's for streaming:
  - As each character arrives, extend the transformed string
  - Update P-array for new positions
  - Report if current suffix is palindromic

  Application: real-time text analysis, streaming data.
```

---

## 5. Palindromic Substring Queries

```
  Query: Is S[l..r] a palindrome?  (answer in O(1))

  Precomputation: Build P-array with Manacher's in O(n).

  Answer query (l, r):
    Transformed center = l + r + 2  (mapping to T)
    Required radius = r - l + 1
    Check if P[center] ≥ required radius

  Wait — simpler approach using P-array:

  Precompute for each position:
    odd[i] = max odd-length palindrome radius centered at i
    even[i] = max even-length palindrome radius centered at i

  S[l..r] is palindrome if:
    len = r - l + 1
    center = (l + r) / 2
    If len is odd:  odd[center] ≥ (len-1)/2
    If len is even: even[center] ≥ len/2

  Time: O(n) preprocess, O(1) per query.
```

---

## 6. Count Palindromes in Range

```
  Problem: How many palindromic substrings exist in S[l..r]?

  P-array gives all palindrome centers and radii.

  For each center c in [l..r]:
    Count palindromes fully within [l..r]:
      Effective radius = min(P_orig[c], c-l, r-c)
      Palindromes from this center = ⌈effective_radius / 1⌉

  Can be optimized with prefix sums for O(1) range queries
  after O(n) preprocessing.
```

---

## 7. Competitive Programming Template

```python
def manacher(s):
    """Returns array where P[i] = longest palindrome 
    length centered at position i in original string."""
    T = '^#' + '#'.join(s) + '#$'
    n = len(T)
    P = [0] * n
    C = R = 0
    
    for i in range(1, n - 1):
        mirror = 2 * C - i
        if i < R:
            P[i] = min(P[mirror], R - i)
        while T[i + P[i] + 1] == T[i - P[i] - 1]:
            P[i] += 1
        if i + P[i] > R:
            C, R = i, i + P[i]
    
    return P

def all_palindromes(s):
    """Return list of (start, length) of all maximal palindromes."""
    P = manacher(s)
    result = []
    for i in range(2, len(P) - 2):
        if P[i] > 0:
            length = P[i]
            start = (i - 1 - length) // 2
            result.append((start, length))
    return result

def is_palindrome_query(P, l, r):
    """O(1) check if s[l..r] is a palindrome."""
    length = r - l + 1
    center_in_T = l + r + 2  # transformed index
    return P[center_in_T] >= length

def count_palindromic_substrings(s):
    """Count total palindromic substrings."""
    P = manacher(s)
    count = 0
    for i in range(2, len(P) - 2):
        # Number of palindromes centered here:
        # P[i]//2 if at '#' (even-length), (P[i]+1)//2 if at char
        count += (P[i] + 1) // 2 if i % 2 == 0 else P[i] // 2
    return count
```

---

## 📝 Summary Table

| Application | How P-array Helps | Complexity |
|-------------|-------------------|------------|
| Longest palindromic substring | max(P[i]) | O(n) |
| All palindromic substrings | Enumerate from P | O(n + count) |
| Palindrome check query | P[center] ≥ required | O(n) / O(1) |
| Palindromic prefix/suffix | Find i with i-P[i]=boundary | O(n) |
| Palindrome partitioning | Precompute palindromes + DP | O(n²) |
| Count palindromes | Sum from P-array | O(n) |
| Shortest palindrome (prepend) | Longest palindromic prefix | O(n) |

---

## ❓ Quick Revision Questions

1. **How to count total palindromic substrings using Manacher's?**
   <details><summary>Answer</summary>For each position i in the P-array, the number of distinct palindromes centered at i is ⌈P[i]/2⌉. Sum across all positions gives the total count. This is O(n) after building P.</details>

2. **How does Manacher's help with palindrome partitioning DP?**
   <details><summary>Answer</summary>It precomputes in O(n) which substrings are palindromes. The DP then uses these precomputed values instead of checking each substring individually, improving the constant factor.</details>

3. **How to find the longest palindromic prefix?**
   <details><summary>Answer</summary>In the P-array, find the position i where the palindrome i-P[i]..i+P[i] starts at the beginning of the transformed string (position 2). The corresponding original palindrome starts at index 0.</details>

4. **Can Manacher's answer "is S[l..r] palindrome?" in O(1)?**
   <details><summary>Answer</summary>Yes. After O(n) preprocessing, compute the center position in the transformed string and check if P[center] is large enough to cover the range [l..r].</details>

5. **What is the connection between Manacher's and shortest palindrome by prepending?**
   <details><summary>Answer</summary>The shortest palindrome formed by prepending characters equals reverse(S[k+1:]) + S, where S[0:k] is the longest palindromic prefix. Manacher's finds this prefix in O(n).</details>

---

| [⬅️ Previous: Time Complexity](04-time-complexity.md) | [Next: Longest Common Subsequence ➡️](../09-String-DP/01-longest-common-subsequence.md) |
|:---|---:|

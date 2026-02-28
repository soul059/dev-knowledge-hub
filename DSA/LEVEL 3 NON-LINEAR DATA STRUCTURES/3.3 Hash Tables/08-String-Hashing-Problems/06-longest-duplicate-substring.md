# 8.6 Longest Duplicate Substring

## Unit 8: String Hashing Problems

---

## Problem Statement

> Given a string `s`, find the longest substring that occurs at least twice. Return any one such substring (or "" if none).

```
╔══════════════════════════════════════════════════════════════╗
║  Input:  "banana"                                           ║
║  Output: "ana"       (appears at index 1 and 3)             ║
║                                                              ║
║  Input:  "abcd"                                             ║
║  Output: ""          (no repeated substring)                ║
║                                                              ║
║  Naive: Check all O(n²) substring pairs → O(n³) or O(n⁴)  ║
║  Optimal: Binary search + rolling hash → O(n log n)         ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Key Insight: Binary Search on Length

If a duplicate substring of length $k$ exists, then duplicate substrings of all lengths $< k$ also exist (any substring of a repeated substring is also repeated).

```
╔══════════════════════════════════════════════════════════════╗
║  "banana"                                                    ║
║                                                              ║
║  Length 5: "banan","anana" → no duplicate                   ║
║  Length 4: "bana","anan","nana" → no duplicate              ║
║  Length 3: "ban","ana","nan","ana" → "ana" duplicated! ✓    ║
║  Length 2: "ba","an","na","an","na" → "an","na" dup ✓      ║
║  Length 1: "b","a","n","a","n","a" → "a","n" dup ✓         ║
║                                                              ║
║  Monotonic! → Binary search for max length                  ║
║                                                              ║
║  ──────────────────────────────────────────────             ║
║  Length:   1  2  3  4  5                                    ║
║  Has dup:  ✓  ✓  ✓  ✗  ✗                                  ║
║            ↑           ↑                                    ║
║            answer=3    binary search boundary               ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Algorithm

```
FUNCTION longestDupSubstring(s):
    n = len(s)
    lo = 1, hi = n - 1
    result = ""

    WHILE lo <= hi:
        mid = (lo + hi) / 2
        dup = findDuplicate(s, mid)    // Check if dup of length mid exists

        IF dup != "":
            result = dup
            lo = mid + 1               // Try longer
        ELSE:
            hi = mid - 1               // Try shorter

    RETURN result

FUNCTION findDuplicate(s, length):
    // Use rolling hash to find any repeated substring of given length
    seen = new HashSet()
    d = 31
    q = 2^61 - 1     // Large Mersenne prime

    // Compute hash of first window
    hash = 0
    highPow = 1
    FOR i = 0 TO length - 1:
        hash = (hash * d + s[i]) % q
        IF i > 0: highPow = (highPow * d) % q

    seen.add(hash)

    // Roll through remaining windows
    FOR i = 1 TO len(s) - length:
        hash = (hash * d - s[i-1] * highPow * d + s[i+length-1]) % q
        // Simplified: hash = (d * (hash - s[i-1] * highPow) + s[i+length-1]) % q
        IF hash < 0: hash += q

        IF seen.contains(hash):
            // Verify to avoid false positive
            RETURN s[i..i+length-1]
        seen.add(hash)

    RETURN ""     // No duplicate of this length
```

---

## Complete Trace

```
s = "banana" → [2,1,14,1,14,1] (a=1,b=2,...,n=14)
n = 6

Binary search: lo=1, hi=5

Iteration 1: mid=3
  findDuplicate(s, 3):
    Window "ban" → hash₁, add to seen
    Window "ana" → hash₂, add to seen
    Window "nan" → hash₃, add to seen
    Window "ana" → hash₂, FOUND IN SEEN!
    Return "ana"
  
  result = "ana", lo = 4

Iteration 2: mid=4
  findDuplicate(s, 4):
    Window "bana" → hash₁
    Window "anan" → hash₂
    Window "nana" → hash₃
    All unique → return ""
  
  hi = 3

Iteration 3: lo=4 > hi=3 → STOP

Answer: "ana" ✓
```

---

## Complexity Analysis

```
╔══════════════════════════════════════════════════════════════╗
║  Binary search: O(log n) iterations                         ║
║                                                              ║
║  Each iteration (findDuplicate):                            ║
║    Rolling hash: O(n) time                                  ║
║    HashSet operations: O(n) average                         ║
║                                                              ║
║  Total: O(n log n) average                                  ║
║  Space: O(n) for the hash set                               ║
║                                                              ║
║  Comparison with alternatives:                              ║
║  ┌────────────────────┬─────────────┬───────────┐          ║
║  │ Approach           │ Time        │ Space     │          ║
║  ├────────────────────┼─────────────┼───────────┤          ║
║  │ Brute force        │ O(n³)       │ O(1)      │          ║
║  │ All substrings+Set │ O(n²)       │ O(n²)     │          ║
║  │ Binary+RollingHash │ O(n log n)  │ O(n)      │          ║
║  │ Suffix array+LCP   │ O(n)        │ O(n)      │          ║
║  └────────────────────┴─────────────┴───────────┘          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Handling Hash Collisions

For this problem, a false positive in `findDuplicate` means reporting a duplicate that doesn't exist. This could cause the binary search to go in the wrong direction.

```
╔══════════════════════════════════════════════════════════════╗
║  STRATEGIES:                                                ║
║                                                              ║
║  1. VERIFY on match (safest):                               ║
║     When hash matches, compare actual characters O(m)       ║
║     Doesn't change average complexity                       ║
║                                                              ║
║  2. Double hashing (practical):                             ║
║     Use two independent hash functions                      ║
║     Match only when BOTH agree                              ║
║     Collision probability ≈ 1/q₁q₂ ≈ 10⁻¹⁸               ║
║                                                              ║
║  3. Store (hash, startIndex) pairs:                         ║
║     When hashes match, compare from stored index            ║
║     Allows precise verification                             ║
╚══════════════════════════════════════════════════════════════╝
```

### Using (hash, index) for Verification

```
FUNCTION findDuplicate(s, length):
    seen = new HashMap()    // hash → start index

    FOR i = 0 TO len(s) - length:
        h = getHash(i, i + length - 1)

        IF seen.containsKey(h):
            j = seen[h]
            IF s[j..j+length-1] == s[i..i+length-1]:
                RETURN s[i..i+length-1]

        seen[h] = i

    RETURN ""
```

---

## Related Problems

| Problem | Reduction |
|---------|-----------|
| Longest repeated substring | Exactly this problem |
| Shortest unique substring | Binary search + hash (complement) |
| Count distinct substrings | Hash all substrings, count set size |
| Longest common substring of 2 strings | Binary search + hash intersection |

### Longest Common Substring of Two Strings

```
FUNCTION longestCommonSubstring(s1, s2):
    lo = 1, hi = min(len(s1), len(s2))
    result = ""

    WHILE lo <= hi:
        mid = (lo + hi) / 2
        common = findCommon(s1, s2, mid)

        IF common != "":
            result = common
            lo = mid + 1
        ELSE:
            hi = mid - 1

    RETURN result

FUNCTION findCommon(s1, s2, length):
    // Hash all substrings of s1 with given length
    hashes1 = new HashSet()
    FOR i = 0 TO len(s1) - length:
        hashes1.add(getHash(s1, i, i+length-1))

    // Check if any substring of s2 matches
    FOR i = 0 TO len(s2) - length:
        h = getHash(s2, i, i+length-1)
        IF hashes1.contains(h):
            RETURN s2[i..i+length-1]    // verify in practice

    RETURN ""
```

---

## Quick Revision Question

**Q: Why does binary search work for finding the longest duplicate substring? What property of the problem makes it monotonic?**

<details>
<summary>Click to reveal answer</summary>

Binary search works because of the **monotonicity property**: if a repeated substring of length $k$ exists, then repeated substrings of all lengths $1, 2, \ldots, k-1$ also exist.

**Proof:** If substring $s[i..i+k-1] = s[j..j+k-1]$ (length $k$ duplicate), then for any $l < k$:
$$s[i..i+l-1] = s[j..j+l-1]$$
This is a length-$l$ duplicate — just take the first $l$ characters of each matching occurrence.

**This creates a monotonic structure:**
```
Length:     1  2  3  ...  L  L+1  ...  n-1
Has dup:   ✓  ✓  ✓  ...  ✓   ✗   ...   ✗
```

There's a clear **threshold** $L$ such that:
- All lengths $\leq L$ have duplicates
- All lengths $> L$ don't

Binary search finds this $L$ in $O(\log n)$ iterations.

**Note:** The converse isn't true — having a length-$l$ duplicate doesn't guarantee a length-$(l+1)$ duplicate. But we search from the "YES" side, which is sufficient.

</details>

---

| [← String Matching Applications](05-string-matching-applications.md) | [Unit 9: Advanced Hashing →](../09-Advanced-Hashing/01-consistent-hashing.md) |
|:---|---:|

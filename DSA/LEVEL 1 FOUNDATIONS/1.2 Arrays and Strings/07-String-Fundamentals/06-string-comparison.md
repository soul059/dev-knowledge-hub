# Chapter 6: String Comparison

[← Previous: Character Encoding](05-character-encoding.md) | [Back to README](../README.md) | [Next: Pattern Matching Basics →](../08-String-Algorithms/01-pattern-matching-basics.md)

---

## Overview

Comparing strings is one of the most common operations in programming — sorting, searching, deduplication, and many algorithms depend on it. This chapter covers lexicographic ordering, equality checking, case-insensitive comparison, and their complexities.

---

## Lexicographic Order

```
  Dictionary order: compare character by character

  "apple" vs "banana"
   a < b  → "apple" < "banana"  (decided at first char!)

  "apple" vs "application"
   a = a
   p = p
   p = p
   l = l
   e ≠ i  → 'e' (101) < 'i' (105) → "apple" < "application"

  "app" vs "apple"
   a = a
   p = p
   p = p
   "app" ends → shorter string is smaller
   → "app" < "apple"
```

### Formal Definition

$$s_1 < s_2 \iff \exists\, k : s_1[0..k{-}1] = s_2[0..k{-}1] \text{ and } (s_1[k] < s_2[k] \text{ or } |s_1| = k < |s_2|)$$

---

## Algorithm: String Comparison

```
FUNCTION compare(s1, s2):
    n1 = length(s1)
    n2 = length(s2)
    i = 0

    WHILE i < n1 AND i < n2:
        IF s1[i] < s2[i]:
            RETURN -1      // s1 comes first
        IF s1[i] > s2[i]:
            RETURN +1      // s2 comes first
        i = i + 1

    // All compared characters were equal
    IF n1 < n2: RETURN -1  // s1 is prefix of s2
    IF n1 > n2: RETURN +1  // s2 is prefix of s1
    RETURN 0                // identical
```

### Trace

```
  compare("code", "coder"):
  
  i=0: 'c' == 'c' ✓
  i=1: 'o' == 'o' ✓
  i=2: 'd' == 'd' ✓
  i=3: 'e' == 'e' ✓
  i=4: end of s1 (length 4), s2 has 'r' left
  
  n1=4 < n2=5 → return -1
  "code" < "coder"
```

### Complexity

| Case | Time |
|------|------|
| Best | O(1) — differ at first character |
| Worst | O(min(n, m)) — differ at end or identical |
| Average | Depends on data distribution |

---

## Equality Check

```
  Two strings are equal if and only if:
  1. Same length
  2. Same character at every position

  FUNCTION equals(s1, s2):
      IF length(s1) ≠ length(s2):
          RETURN false               // O(1) quick reject!
      FOR i = 0 TO length(s1) - 1:
          IF s1[i] ≠ s2[i]:
              RETURN false
      RETURN true

  Time: O(n) worst case, but often O(1) due to length check
```

### Reference vs Value Equality

```
  Java:
  String a = "hello";
  String b = "hello";
  
  a == b       // TRUE (same interned reference)
  a.equals(b)  // TRUE (same content)

  String c = new String("hello");
  a == c       // FALSE (different objects!)
  a.equals(c)  // TRUE (same content)

  RULE: Always use .equals() for content comparison in Java
        Use == only for reference/identity check
```

---

## Case-Insensitive Comparison

```
  "Hello" vs "hello" → equal (case-insensitive)

  Approach 1: Convert both to same case
  lower("Hello") = "hello"
  lower("hello") = "hello"
  Compare: equal ✓

  Approach 2: Compare with case folding inline
  FUNCTION equalsIgnoreCase(s1, s2):
      IF length(s1) ≠ length(s2): RETURN false
      FOR i = 0 TO length(s1) - 1:
          c1 = toLower(s1[i])
          c2 = toLower(s2[i])
          IF c1 ≠ c2: RETURN false
      RETURN true

  toLower(c):
      IF 'A' ≤ c ≤ 'Z': RETURN c + 32
      RETURN c
```

### Bit Trick for Case

```
  ASCII:
  'A' = 0100 0001 = 65
  'a' = 0110 0001 = 97

  Difference: bit 5 (value 32)!

  toLower: c | 0x20   (set bit 5)
  toUpper: c & ~0x20  (clear bit 5)
  
  Case-insensitive equal: (c1 | 0x20) == (c2 | 0x20)
  
  WARNING: Only works for A-Z / a-z, not for special
  characters like 'ß' → 'SS' in German.
```

---

## Comparing with Hashing

```
  For repeated comparisons, pre-compute hash:

  hash("hello") = 99162322
  hash("world") = 113318802

  If hashes differ → strings definitely differ: O(1)
  If hashes match → strings MIGHT be equal: verify O(n)

  Useful for:
  • Hash maps / hash sets with string keys
  • Rabin-Karp string matching
  • Deduplication

  ┌─────────────────────────────────────────────┐
  │  Hash collision: different strings, same hash│
  │  "hello" → 42                                │
  │  "xyz"   → 42  (collision!)                  │
  │                                              │
  │  hash(s1) ≠ hash(s2) → s1 ≠ s2  (certain)  │
  │  hash(s1) = hash(s2) → maybe equal (verify) │
  └─────────────────────────────────────────────┘
```

---

## Sorting Strings

```
  Sorting an array of strings uses comparison as a subroutine.

  Strings: ["banana", "apple", "cherry"]
  Sorted:  ["apple", "banana", "cherry"]

  Comparison sort: O(n log n) comparisons
  Each comparison: O(L) where L = average string length
  
  Total: O(n × L × log n)

  This is MORE than O(n log n) for integers!
```

---

## Collation: Beyond ASCII Order

```
  ASCII order isn't always "correct" for humans:

  ASCII sort: "Banana" < "apple" (B=66 < a=97)
  Human sort: "apple" < "Banana" (case-insensitive)

  Locale-specific sorting (collation):
  • German: ä sorts with a
  • Swedish: ä sorts after z
  • Numbers: "file2" < "file10" (natural sort)

  For DSA: we usually use simple ASCII/Unicode ordering
  For production: use locale-aware comparators
```

---

## Common Patterns in DSA

```
  1. Check if two strings are anagrams:
     Sort both and compare → O(n log n)
     Or use frequency count → O(n)

  2. Check if one string is a rotation of another:
     s2 is a rotation of s1 iff s2 is in s1 + s1
     "elloH" is in "HelloH" + "ello" → "HelloHello"
     Check: (s1+s1).contains(s2) → O(n)

  3. Check if one is a subsequence of another:
     Two pointers: O(n + m)
     
  4. Longest common prefix:
     Compare character by character → O(min lengths)
```

---

## Summary Table

| Comparison Type | Time Complexity | Notes |
|----------------|----------------|-------|
| Lexicographic | O(min(n, m)) | Character by character |
| Equality | O(n) | O(1) if lengths differ |
| Case-insensitive | O(n) | Convert or fold inline |
| Hash-based | O(1) avg* | Need O(n) verification on match |
| Anagram check | O(n) | Frequency counting |
| String sorting | O(nL log n) | L = avg string length |

*After hash precomputation

---

## Quick Revision Questions

1. **Define lexicographic order.** How is "app" compared to "apple"?

2. **Why should you use `.equals()` instead of `==` for strings in Java?**

3. **How can you compare characters case-insensitively using bit manipulation?** What is the limitation?

4. **Why is sorting strings O(nL log n)** instead of O(n log n)?

5. **How can hashing speed up string comparison?** When does it fail?

6. **How do you check if "waterbottle" is a rotation of "erbottlewat" in O(n)?**

---

[← Previous: Character Encoding](05-character-encoding.md) | [Back to README](../README.md) | [Next: Pattern Matching Basics →](../08-String-Algorithms/01-pattern-matching-basics.md)

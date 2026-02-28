# Chapter 1: Reverse String Problems

[← Previous: Algorithm Comparison](../08-String-Algorithms/06-algorithm-comparison.md) | [Back to README](../README.md) | [Next: Palindrome Detection →](02-palindrome-detection.md)

---

## Overview

Reversing strings is one of the most common interview problems. While the basic reversal is straightforward, variations like reversing words, reversing vowels, or reversing within constraints test deeper understanding of array/string manipulation.

---

## 1. Reverse a String (LeetCode 344)

```
  Input:  ['h','e','l','l','o']
  Output: ['o','l','l','e','h']

  Two-pointer swap:
  
  [h] [e] [l] [l] [o]
   L               R     swap h↔o
  [o] [e] [l] [l] [h]
       L       R          swap e↔l
  [o] [l] [l] [e] [h]
           LR             L ≥ R → done!
```

### Algorithm

```
FUNCTION reverse(s):
    left = 0
    right = length(s) - 1

    WHILE left < right:
        SWAP(s[left], s[right])
        left = left + 1
        right = right - 1
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(1) |

---

## 2. Reverse Words in a String (LeetCode 151)

```
  Input:  "the sky is blue"
  Output: "blue is sky the"

  Strategy: Reverse entire string, then reverse each word.

  Step 1: Reverse everything
  "eulb si yks eht"

  Step 2: Reverse each word
  "blue is sky the"

  ┌───────────────────────────────┐
  │ "the sky is blue"             │  original
  │ "eulb si yks eht"            │  reverse all
  │ "blue si yks eht"            │  reverse word 1
  │ "blue is yks eht"            │  reverse word 2
  │ "blue is sky eht"            │  reverse word 3
  │ "blue is sky the"            │  reverse word 4 ✓
  └───────────────────────────────┘
```

### Algorithm

```
FUNCTION reverseWords(s):
    // Step 1: Reverse entire string
    reverse(s, 0, length(s) - 1)

    // Step 2: Reverse each word
    start = 0
    FOR i = 0 TO length(s):
        IF i == length(s) OR s[i] == ' ':
            reverse(s, start, i - 1)
            start = i + 1

    // (Handle extra spaces if needed)
```

### Trace

```
  s = "hello world"

  Reverse all: "dlrow olleh"
  
  Word 1 (0..4): reverse "dlrow" → "world"
  → "world olleh"
  
  Word 2 (6..10): reverse "olleh" → "hello"
  → "world hello"  ✓
```

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(1) in-place |

---

## 3. Reverse Words in a String III (LeetCode 557)

```
  Reverse each word individually (don't reverse word order).

  Input:  "Let's take LeetCode contest"
  Output: "s'teL ekat edoCteeL tsetnoc"

  Algorithm: find each word, reverse it in place.

  FOR each word (between spaces):
      reverse(word)
```

---

## 4. Reverse Vowels of a String (LeetCode 345)

```
  Reverse only the vowels, keep consonants in place.

  Input:  "hello"
  Output: "holle"

  Vowels: e (pos 1), o (pos 4) → swap
  Result: h-o-l-l-e = "holle"

  Algorithm: Two pointers, but only stop at vowels.
```

### Algorithm

```
FUNCTION reverseVowels(s):
    vowels = {'a','e','i','o','u','A','E','I','O','U'}
    left = 0
    right = length(s) - 1

    WHILE left < right:
        WHILE left < right AND s[left] NOT IN vowels:
            left = left + 1
        WHILE left < right AND s[right] NOT IN vowels:
            right = right - 1
        
        SWAP(s[left], s[right])
        left = left + 1
        right = right - 1
```

### Trace

```
  s = "leetcode"
       l e e t c o d e
       0 1 2 3 4 5 6 7

  Vowels at: 1(e), 2(e), 5(o), 7(e)

  L=0→1(e), R=7(e): swap → "leetcode" (same: e↔e)
  L=2(e), R=6→5(o): swap → "leotcode" wait...

  Let me redo:
  L=0: 'l' not vowel, L=1
  L=1: 'e' is vowel. R=7: 'e' is vowel.
  Swap s[1]↔s[7]: both 'e', no visible change. L=2, R=6.
  
  L=2: 'e' is vowel. R=6: 'd' not vowel, R=5: 'o' is vowel.
  Swap s[2]↔s[5]: e↔o → "leotcede"? 
  Actually: l-e-o-t-c-e-d-e → "leotcede"
  L=3, R=4.
  
  L=3: 't' not vowel, L=4: 'c' not vowel, L=5: L > R? No, L=5>R=4, stop.

  Result: "leotcede"  
```

---

## 5. Reverse String II (LeetCode 541)

```
  Reverse first k characters of every 2k block.

  Input: s = "abcdefg", k = 2
  
  Block 1 (0..3): reverse first 2 of "abcd" → "bacd"
  Block 2 (4..7): reverse first 2 of "efg"  → "feg" (only 3 chars, reverse first 2)
  
  Result: "bacdfeg"
```

### Algorithm

```
FUNCTION reverseStr(s, k):
    FOR i = 0, step 2k:
        left = i
        right = MIN(i + k - 1, length(s) - 1)
        reverse(s, left, right)
```

---

## 6. Reverse Only Letters (LeetCode 917)

```
  Reverse only alphabetic characters, keep others in place.

  Input:  "a-bC-dEf-ghIj"
  Output: "j-Ih-gfE-dCba"

  Same idea as reverse vowels but with isLetter check.

  Two pointers: skip non-letters on both sides,
  swap when both point to letters.
```

---

## Pattern Summary

```
  ┌──────────────────────────────────────────────────────┐
  │          REVERSE PATTERN FAMILY                      │
  │                                                      │
  │  Basic reverse:       Two pointers, swap all         │
  │  Reverse words:       Reverse all, then each word    │
  │  Reverse each word:   Find word boundaries, reverse  │
  │  Reverse vowels:      Two pointers, skip non-vowels  │
  │  Reverse letters:     Two pointers, skip non-letters │
  │  Partial reverse:     Reverse blocks of size k       │
  │                                                      │
  │  Common technique: TWO POINTERS + SELECTIVE SWAP     │
  └──────────────────────────────────────────────────────┘
```

---

## Summary Table

| Problem | Technique | Time | Space |
|---------|-----------|------|-------|
| Reverse string | Two pointers | O(n) | O(1) |
| Reverse words | Reverse all + each word | O(n) | O(1) |
| Reverse each word | Find + reverse | O(n) | O(1) |
| Reverse vowels | Two pointers, filter | O(n) | O(1) |
| Reverse first k in 2k | Block iteration | O(n) | O(1) |
| Reverse only letters | Two pointers, filter | O(n) | O(1) |

---

## Quick Revision Questions

1. **How do you reverse words in O(1) space?** What is the "reverse all then reverse each" trick?

2. **Trace the reverse vowels algorithm** on "programming".

3. **Why does the two-pointer approach work for selective reversal** (vowels, letters)?

4. **What edge cases** should you handle in reverse words? (leading/trailing/multiple spaces)

5. **How is "reverse first k of every 2k" different** from "reverse every k"?

6. **Can you reverse a string recursively?** What is the time and space complexity?

---

[← Previous: Algorithm Comparison](../08-String-Algorithms/06-algorithm-comparison.md) | [Back to README](../README.md) | [Next: Palindrome Detection →](02-palindrome-detection.md)

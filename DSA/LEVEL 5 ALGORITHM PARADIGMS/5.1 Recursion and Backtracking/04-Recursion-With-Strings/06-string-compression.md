# Chapter 6: String Compression

## Overview
String compression (Run-Length Encoding) recursively demonstrates how to process sequential patterns in a string — counting consecutive identical characters and building a compressed representation.

---

## Problem Statement
Compress "aaabbbccdd" → "a3b3c2d2". If compressed isn't shorter, return original.

---

## Recursive Solution

### Pseudocode
```
function compress(s, index=0):
    if index >= length(s): return ""                 // Base case
    
    // Count consecutive characters
    count = 1
    while index + count < length(s) AND s[index + count] == s[index]:
        count += 1
    
    // Build: character + count + compress rest
    return s[index] + str(count) + compress(s, index + count)
```

### Trace: compress("aaabbbccdd", 0)

```
Step 1: index=0, s[0]='a', count=3 (aaa)
        "a3" + compress(s, 3)

Step 2: index=3, s[3]='b', count=3 (bbb)
        "b3" + compress(s, 6)

Step 3: index=6, s[6]='c', count=2 (cc)
        "c2" + compress(s, 8)

Step 4: index=8, s[8]='d', count=2 (dd)
        "d2" + compress(s, 10)

Step 5: index=10 >= length=10 → return ""  ← BASE

Unwinding: "" → "d2" → "c2d2" → "b3c2d2" → "a3b3c2d2"

Input:  "aaabbbccdd" (length 10)
Output: "a3b3c2d2"  (length 8) → Shorter, use compressed ✓
```

---

## Pure Recursive (No While Loop)

```
function compress(s, index, current_char, count):
    if index >= length(s):
        return current_char + str(count)      // Flush last group
    
    if s[index] == current_char:
        return compress(s, index+1, current_char, count+1)  // Same char
    else:
        // Different char: output current group + start new
        return current_char + str(count) + compress(s, index+1, s[index], 1)

// Initial call: compress(s, 1, s[0], 1)
```

---

## Complexity

```
Time:  O(n) — each character processed once
Space: O(n) — stack depth + result string

Note: In worst case (all different chars like "abcd"),
      compressed = "a1b1c1d1" is LONGER than original!
      → Return original in that case
```

---

## Variations

| Variation | Example |
|-----------|---------|
| Skip count=1 | "aabbc" → "a2b2c" |
| Decompression | "a3b2c1" → "aaabbc" |
| Adjacent pairs | "aabbcc" → "a2b2c2" |
| With separators | "a:3,b:2,c:1" |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Pattern | Group consecutive chars + recurse on rest |
| Base case | Past end of string |
| Time | O(n) |
| Space | O(n) |
| Key insight | Jump by count, not by 1 |
| Application | Run-Length Encoding (RLE), data compression |

---

## Quick Revision Questions

1. **What is Run-Length Encoding** and where is it used?
2. **Why might compressed output** be longer than the original?
3. **Trace compress("aabccc", 0)** step by step.
4. **How would you write** a recursive decompression function?
5. **What is the time complexity** and why is it O(n)?
6. **How does the index jump** differ from typical character-by-character recursion?

---

[← Previous: Permutations Basics](05-permutations-basics.md) | [Next: Unit 5 — Visualizing Recursion →](../05-Understanding-Recursion-Tree/01-visualizing-recursion.md)

[← Back to README](../README.md)

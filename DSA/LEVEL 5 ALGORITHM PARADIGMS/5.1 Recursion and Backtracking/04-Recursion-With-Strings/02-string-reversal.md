# Chapter 2: String Reversal (Deep Dive)

## Overview
Building upon the basic reversal from Unit 2, this chapter explores **multiple recursive strategies** for string reversal, including in-place swap, accumulator pattern, and divide-and-conquer.

---

## Approach 1: First-to-End (Head Recursion)

```
function reverse(s):
    if length(s) <= 1: return s
    return reverse(s[1:]) + s[0]

// Trace: "abcd" → reverse("bcd")+"a" → reverse("cd")+"b"+"a" 
//        → reverse("d")+"c"+"b"+"a" → "d"+"c"+"b"+"a" = "dcba"
```

## Approach 2: Two-Pointer Swap (In-Place)

```
function reverse(chars[], left, right):
    if left >= right: return
    swap(chars[left], chars[right])
    reverse(chars, left+1, right-1)

// Best for mutable char arrays — O(1) extra space (beyond stack)
```

```
Trace for "abcde":
  a b c d e       swap(a,e) → e b c d a
  e b c d a       swap(b,d) → e d c b a
  e d c b a       left>=right, DONE!
    ↑       ↑         ↑   ↑       ↑
    L       R         L   R      L=R
```

## Approach 3: Divide and Conquer

```
function reverse(s):
    if length(s) <= 1: return s
    mid = length(s) / 2
    left = s[0:mid]
    right = s[mid:]
    return reverse(right) + reverse(left)

// Trace: "abcd" → reverse("cd") + reverse("ab")
//                   = "dc" + "ba" = "dcba"
```

---

## Complexity Comparison

| Approach | Time | Space |
|----------|------|-------|
| First-to-end | O(n²) | O(n) |
| Two-pointer | O(n) | O(n) stack |
| Divide & conquer | O(n log n) | O(n) |
| Accumulator | O(n) | O(n) |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Best approach | Two-pointer swap for char arrays |
| Most intuitive | First-to-end (but O(n²)) |
| Key insight | Swap outermost, recurse inward |
| Connection | Same pattern as palindrome check |

---

## Quick Revision Questions

1. **Why is the concatenation approach** O(n²)?
2. **How does two-pointer swap** achieve O(n) time?
3. **Trace the divide-and-conquer** approach for "hello".
4. **Which approach works best** for immutable strings?
5. **How is string reversal related** to palindrome checking?
6. **Can you reverse a string** using only one pointer and recursion?

---

[← Previous: Palindrome Check](01-palindrome-check.md) | [Next: Remove Character →](03-remove-character.md)

[← Back to README](../README.md)

# Chapter 6: Reverse a String

## Overview
Reversing a string recursively is a fundamental problem that demonstrates multiple recursive approaches — shrinking from the front, shrinking from the back, and using two pointers. It also illustrates how recursion naturally handles string problems.

---

## Approach 1: Remove First, Append at End

### Pseudocode
```
function reverse(s):
    if length(s) <= 1: return s        // Base case
    return reverse(s[1:]) + s[0]       // Reverse rest + first char at end
```

### Trace: reverse("hello")

```
reverse("hello")
│  return reverse("ello") + "h"
│         │
│         reverse("ello")
│         │  return reverse("llo") + "e"
│         │         │
│         │         reverse("llo")
│         │         │  return reverse("lo") + "l"
│         │         │         │
│         │         │         reverse("lo")
│         │         │         │  return reverse("o") + "l"
│         │         │         │         │
│         │         │         │         reverse("o") → "o"  ← BASE
│         │         │         │
│         │         │         │  return "o" + "l" = "ol"
│         │         │         
│         │         │  return "ol" + "l" = "oll"
│         │         
│         │  return "oll" + "e" = "olle"
│         
│  return "olle" + "h" = "olleh"  ✓
```

```
Building the result:

  Step 1: "o"                     (base case)
  Step 2: "o" + "l"   = "ol"     (unwinding)
  Step 3: "ol" + "l"  = "oll"
  Step 4: "oll" + "e" = "olle"
  Step 5: "olle" + "h"= "olleh"  (final answer)
  
  ┌─────┐    ┌─────┐
  │hello│ → │olleh│
  └─────┘    └─────┘
```

---

## Approach 2: Two-Pointer Swap

### Pseudocode
```
function reverse(s, left, right):
    if left >= right: return        // Base case: pointers met/crossed
    swap(s[left], s[right])         // Swap outer characters
    reverse(s, left + 1, right - 1) // Recurse on inner portion
```

### Trace: reverse("hello", 0, 4)

```
Original: h e l l o
          0 1 2 3 4
          L       R

Step 1: swap(s[0], s[4])  →  o e l l h
        L=0, R=4               L+1  R-1

Step 2: swap(s[1], s[3])  →  o l l e h
        L=1, R=3               L+1  R-1

Step 3: L=2, R=2  →  L >= R, BASE CASE!

Result: o l l e h  ✓

Pointer movement:
  ┌───┬───┬───┬───┬───┐
  │ h │ e │ l │ l │ o │  Before
  └───┴───┴───┴───┴───┘
    L               R

  ┌───┬───┬───┬───┬───┐
  │ o │ e │ l │ l │ h │  After swap 1
  └───┴───┴───┴───┴───┘
        L       R

  ┌───┬───┬───┬───┬───┐
  │ o │ l │ l │ e │ h │  After swap 2
  └───┴───┴───┴───┴───┘
            LR          (L >= R, stop!)
```

---

## Approach 3: Using Helper with Accumulator

### Pseudocode
```
function reverse(s):
    return reverseHelper(s, "")

function reverseHelper(s, acc):
    if s is empty: return acc
    return reverseHelper(s[1:], s[0] + acc)  // Prepend first char to acc
```

### Trace: reverseHelper("hello", "")

```
reverseHelper("hello", "")
  → reverseHelper("ello", "h")       // prepend 'h'
  → reverseHelper("llo",  "eh")      // prepend 'e'
  → reverseHelper("lo",   "leh")     // prepend 'l'
  → reverseHelper("o",    "lleh")    // prepend 'l'
  → reverseHelper("",     "olleh")   // prepend 'o'
  → return "olleh"                    // BASE: s is empty
  
[*] This is TAIL RECURSIVE — accumulator builds result going down!
```

---

## Comparison of Approaches

```
┌────────────────────────────────────────────────────────┐
│  Approach 1: reverse(rest) + first                     │
│  ✓ Simple and intuitive                                │
│  ✗ String concatenation can be O(n) → total O(n²)     │
│                                                        │
│  Approach 2: Two-pointer swap                          │
│  ✓ In-place, O(n) total                               │
│  ✓ Most efficient for mutable strings                  │
│  ✗ Needs mutable string (char array)                   │
│                                                        │
│  Approach 3: Accumulator                               │
│  ✓ Tail recursive                                      │
│  ✓ Natural for immutable strings                       │
│  ✗ Slightly harder to understand                       │
└────────────────────────────────────────────────────────┘
```

---

## Complexity Analysis

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Append at end | O(n²) | O(n) | String concat is O(n) each |
| Two-pointer | O(n) | O(n) stack | Best for char arrays |
| Accumulator | O(n²) or O(n) | O(n) | Depends on string impl |

```
For Approach 1 (O(n²) breakdown):
  Call 1: concat strings of length 4 + 1 = O(5)
  Call 2: concat strings of length 3 + 1 = O(4)
  Call 3: concat strings of length 2 + 1 = O(3)
  Call 4: concat strings of length 1 + 1 = O(2)
  Total: 5 + 4 + 3 + 2 = O(n²)
```

---

## Related Problems

| Problem | Variation |
|---------|-----------|
| Reverse words in string | Split by spaces, reverse each |
| Reverse integer | Extract digits recursively |
| Check palindrome | Compare first and last chars |
| Reverse linked list | Change pointers recursively |
| Reverse array | Same two-pointer technique |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Problem | Reverse string recursively |
| Base case | Empty or single character string |
| Best approach | Two-pointer swap (in-place) |
| Time | O(n) for swap, O(n²) for concatenation |
| Space | O(n) stack depth |
| Key pattern | Shrink from both ends or strip first char |
| Connection | Builds intuition for palindrome checking |

---

## Quick Revision Questions

1. **Why is the concatenation approach O(n²)** in time?
2. **What advantage does** the two-pointer approach have?
3. **Trace reverse("abcde")** using the two-pointer method.
4. **How is the accumulator approach** tail recursive?
5. **What changes** if the string has even vs odd length in two-pointer?
6. **How would you reverse** only a portion of a string (index i to j)?

---

[← Previous: Print N to 1 and 1 to N](05-print-n-to-1-and-1-to-n.md) | [Next: Unit 3 — Linear Search Recursive →](../03-Recursion-With-Arrays/01-linear-search-recursive.md)

[← Back to README](../README.md)

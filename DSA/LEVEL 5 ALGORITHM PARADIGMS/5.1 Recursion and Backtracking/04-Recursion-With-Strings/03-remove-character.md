# Chapter 3: Remove Character

## Overview
Removing all occurrences of a specific character from a string recursively demonstrates the **filter pattern** — decide whether to include each character, process the rest, and build the result.

---

## Problem Statement
Remove all occurrences of character `ch` from string `s`.

## Recursive Solution

### Pseudocode
```
function removeChar(s, ch, index=0):
    if index >= length(s): return ""             // Base: empty
    rest = removeChar(s, ch, index + 1)          // Process rest first
    if s[index] == ch:
        return rest                               // Skip this character
    else:
        return s[index] + rest                    // Keep this character
```

### Trace: removeChar("hello world", 'l', 0)

```
Processing each character:
  h → keep  → "h" + removeChar(rest)
  e → keep  → "e" + removeChar(rest)
  l → SKIP  → removeChar(rest)
  l → SKIP  → removeChar(rest)
  o → keep  → "o" + removeChar(rest)
  ' '→ keep → " " + removeChar(rest)
  w → keep  → "w" + removeChar(rest)
  o → keep  → "o" + removeChar(rest)
  r → keep  → "r" + removeChar(rest)
  l → SKIP  → removeChar(rest)
  d → keep  → "d" + removeChar(rest)
  ""→ BASE  → ""

Building result (unwinding):
  "" → "d" → "rd" → "ord" → "word" → " word" → "o word" → "eo word" → "heo word"
  
Result: "heo word" ✓
```

---

## Alternative: Build from Left (Tail-Like)

```
function removeChar(s, ch, index=0, result=""):
    if index >= length(s): return result
    if s[index] != ch:
        result = result + s[index]
    return removeChar(s, ch, index + 1, result)
```

---

## Generalization: Remove by Condition

```
function removeIf(s, condition, index=0):
    if index >= length(s): return ""
    rest = removeIf(s, condition, index + 1)
    if condition(s[index]):
        return rest              // Remove
    return s[index] + rest       // Keep

// Remove all vowels:
removeIf("hello", isVowel)  →  "hll"

// Remove digits:
removeIf("a1b2c3", isDigit)  →  "abc"
```

---

## The Filter Pattern

```
┌──────────────────────────────────────────┐
│           FILTER PATTERN                 │
│                                          │
│  For each element:                       │
│    if condition(element):                │
│        SKIP (don't include in result)    │
│    else:                                 │
│        INCLUDE in result                 │
│    recurse on remaining elements         │
│                                          │
│  This pattern applies to:                │
│  • Remove character from string          │
│  • Filter array elements                 │
│  • Remove duplicates                     │
│  • Extract specific items                │
└──────────────────────────────────────────┘
```

---

## Complexity

```
Time:  O(n) — visit each character once
Space: O(n) — stack + result string
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Pattern | Filter — include/skip each element |
| Base case | Empty string |
| Decision | Keep if not target, skip if target |
| Time | O(n) |
| Generalization | Works with any condition function |

---

## Quick Revision Questions

1. **What is the filter pattern** in recursion?
2. **Trace removeChar("abcabc", 'b')** showing each step.
3. **How would you modify** this to remove all vowels?
4. **What is the difference** between building result top-down vs bottom-up?
5. **Can this approach remove** the first occurrence only? How?
6. **What is the space complexity** and can it be improved?

---

[← Previous: String Reversal](02-string-reversal.md) | [Next: Subsequences of String →](04-subsequences-of-string.md)

[← Back to README](../README.md)

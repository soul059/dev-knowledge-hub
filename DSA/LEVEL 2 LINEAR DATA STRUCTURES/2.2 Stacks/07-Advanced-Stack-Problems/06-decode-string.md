# Chapter 6: Decode String

[← Previous: Asteroid Collision](05-asteroid-collision.md) | [Next: Function Call Stack →](../08-Stack-Applications/01-function-call-stack.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

---

## Overview

The **Decode String** problem (LeetCode #394) involves decoding an encoded string where patterns like `k[encoded]` mean the `encoded` string is repeated `k` times. Nested patterns require a **stack** to correctly handle the innermost brackets first.

---

## Problem Statement

```
Input: "3[a2[c]]"
Output: "accaccacc"

Decoding:
  Inner:  2[c] → "cc"
  Middle: a + cc → "acc"  
  Outer:  3[acc] → "accaccacc"

More examples:
  "2[abc]3[cd]ef" → "abcabccdcdcdef"
  "10[a]"         → "aaaaaaaaaa"
```

---

## Key Insight

```
┌──────────────────────────────────────────────────────────┐
│  When we see '[': Save current state and start fresh    │
│  When we see ']': Complete current level, merge with    │
│                   saved state                           │
│                                                          │
│  Stack stores: (previous_string, repeat_count) pairs    │
│                                                          │
│  This mirrors function calls:                           │
│    '[' = entering a nested call (save context)          │
│    ']' = returning from call (restore + combine)        │
└──────────────────────────────────────────────────────────┘
```

---

## Algorithm

```
FUNCTION decodeString(s):
    stack ← empty stack          // Stores (prevString, count) pairs
    currentStr ← ""
    currentNum ← 0
    
    FOR each char c in s:
        IF c is digit:
            currentNum ← currentNum × 10 + (c - '0')   // Handle multi-digit
        
        ELSE IF c == '[':
            stack.push((currentStr, currentNum))
            currentStr ← ""     // Reset for new level
            currentNum ← 0
        
        ELSE IF c == ']':
            (prevStr, count) ← stack.pop()
            currentStr ← prevStr + currentStr × count
        
        ELSE:  // Letter
            currentStr ← currentStr + c
    
    RETURN currentStr
```

---

## Detailed Trace

### Input: "3[a2[c]]"

```
Processing each character:

char '3': currentNum = 3
          currentStr = ""

char '[': Push ("", 3) onto stack
          Reset: currentStr = "", currentNum = 0
          Stack: [("", 3)]

char 'a': currentStr = "a"

char '2': currentNum = 2

char '[': Push ("a", 2) onto stack
          Reset: currentStr = "", currentNum = 0
          Stack: [("", 3), ("a", 2)]

char 'c': currentStr = "c"

char ']': Pop ("a", 2)
          currentStr = "a" + "c" × 2 = "a" + "cc" = "acc"
          Stack: [("", 3)]

char ']': Pop ("", 3)
          currentStr = "" + "acc" × 3 = "accaccacc"
          Stack: []

Result: "accaccacc" ✓
```

### Visual Stack State

```
After "3[a2[":
  
  Stack:
  ┌──────────┐
  │ "a", 2   │ ← Level 2 context
  │ "", 3    │ ← Level 1 context
  └──────────┘
  
  currentStr = ""  (building inner content)
  
After first ']':
  
  Stack:
  ┌──────────┐
  │ "", 3    │ ← Level 1 context
  └──────────┘
  
  currentStr = "acc"  (inner decoded)
  
After second ']':
  
  Stack: empty
  
  currentStr = "accaccacc"  (fully decoded)
```

---

## Trace: "2[abc]3[cd]ef"

```
'2':  num=2
'[':  push("", 2), reset       Stack: [("",2)]
'a':  str="a"
'b':  str="ab"
'c':  str="abc"
']':  pop("",2)
      str = "" + "abc"×2 = "abcabc"    Stack: []

'3':  num=3
'[':  push("abcabc", 3), reset  Stack: [("abcabc",3)]
'c':  str="c"
'd':  str="cd"
']':  pop("abcabc",3)
      str = "abcabc" + "cd"×3 = "abcabccdcdcd"  Stack: []

'e':  str = "abcabccdcdcde"
'f':  str = "abcabccdcdcdef"

Result: "abcabccdcdcdef" ✓
```

---

## Handling Multi-Digit Numbers

```
Input: "10[a]"

'1': currentNum = 1
'0': currentNum = 1 × 10 + 0 = 10
'[': push("", 10)
'a': currentStr = "a"
']': pop → "" + "a" × 10 = "aaaaaaaaaa"

The formula currentNum = currentNum × 10 + digit
handles any number of digits: 1, 10, 100, etc.
```

---

## Recursive Alternative

```
FUNCTION decodeString_recursive(s, index):
    result ← ""
    
    WHILE index < length(s) AND s[index] != ']':
        IF s[index] is letter:
            result ← result + s[index]
            index ← index + 1
        ELSE:
            // Read number
            num ← 0
            WHILE s[index] is digit:
                num ← num × 10 + (s[index] - '0')
                index ← index + 1
            
            index ← index + 1    // Skip '['
            (decoded, index) ← decodeString_recursive(s, index)
            index ← index + 1    // Skip ']'
            
            result ← result + decoded × num
    
    RETURN (result, index)
```

---

## Complexity Analysis

| Aspect | Complexity |
|--------|-----------|
| **Time** | O(maxK × n) where maxK is the max repeat count, n is output length |
| **Space** | O(n) for stack depth + output string |

More precisely: O(sum of all k_i × len_i) where k_i and len_i are the count and length at nesting level i.

---

## Edge Cases

```
1. No encoding:      "abc"        → "abc"
2. No nesting:       "3[a]"       → "aaa"
3. Deep nesting:     "2[3[a]]"    → "aaaaaa"
4. Adjacent:         "2[a]2[b]"   → "aabb"
5. Multi-digit:      "100[a]"     → "aaa...a" (100 a's)
6. Empty brackets:   "3[]"        → "" (edge case)
```

---

## Related Problems

```
┌──────────────────────────────────────────────┐
│ Related stack-based string problems:         │
│                                              │
│ • Number of Atoms (LeetCode #726)            │
│ • Basic Calculator (LeetCode #224)           │
│ • Brace Expansion (LeetCode #1087)           │
│ • Remove Outermost Parentheses (#1021)       │
│ • Simplify Path (LeetCode #71)               │
│                                              │
│ Pattern: Save context on '[' or '('          │
│          Restore and combine on ']' or ')'   │
└──────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Decode `k[encoded]` patterns |
| **Stack Stores** | (previous_string, repeat_count) pairs |
| **On `[`** | Save current context, reset |
| **On `]`** | Pop context, combine: `prev + current × count` |
| **Multi-digit** | `num = num × 10 + digit` |
| **Time** | O(output length) |
| **LeetCode** | #394 |

---

## Quick Revision Questions

1. **What does the stack store in Decode String?**
   > Pairs of (previous_string, repeat_count) — the context before entering each bracket level.

2. **How are multi-digit numbers handled?**
   > By accumulating: `currentNum = currentNum × 10 + digit` for each consecutive digit character.

3. **What happens at `[`?**
   > Push current (string, number) onto stack, then reset both to start building the inner content.

4. **What happens at `]`?**
   > Pop (prevString, count), then set `currentStr = prevString + currentStr × count`.

5. **For "2[a3[b]]", what is the output?**
   > Inner: 3[b] = "bbb". Middle: a + bbb = "abbb". Outer: 2[abbb] = "abbbabbb".

---

[← Previous: Asteroid Collision](05-asteroid-collision.md) | [Next: Function Call Stack →](../08-Stack-Applications/01-function-call-stack.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

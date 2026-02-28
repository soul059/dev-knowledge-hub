# Chapter 2: Valid Parentheses (Extended)

[← Previous: Balanced Parentheses](01-balanced-parentheses.md) | [Next: Reverse a String →](03-reverse-string.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)

---

## Overview

The **Valid Parentheses** problem extends the balanced parentheses concept to handle **multiple types of brackets**: `()`, `{}`, and `[]`. Each opening bracket must be closed by the **same type** and in the **correct order**. This is a classic LeetCode problem (Problem #20) and a staple of coding interviews.

---

## Problem Statement

Given a string `s` containing only the characters `(`, `)`, `{`, `}`, `[`, and `]`, determine if the input string is valid.

### Validity Rules

```
Rule 1: Every opening bracket must have a corresponding closing bracket
Rule 2: Every closing bracket must match the most recent unmatched opening bracket
Rule 3: Brackets must close in the correct order
```

### Examples

```
Input: "()"        → Output: true
Input: "()[]{}"    → Output: true
Input: "(]"        → Output: false
Input: "([)]"      → Output: false   ← interleaved, not nested
Input: "{[]}"      → Output: true    ← properly nested
Input: ""          → Output: true    ← empty string is valid
Input: "("         → Output: false   ← unclosed
Input: "((()))"    → Output: true    ← deeply nested
Input: "{[()()]}"  → Output: true    ← complex nesting
```

---

## Key Insight

```
┌──────────────────────────────────────────────────┐
│              WHY A STACK WORKS                   │
│                                                  │
│  When we encounter a CLOSING bracket, it MUST    │
│  match the MOST RECENTLY seen unmatched OPENING  │
│  bracket.                                        │
│                                                  │
│  "Most recently" = LIFO = Stack!                 │
│                                                  │
│  Example: { [ ( ) ] }                            │
│              ↑ ↑                                  │
│              ( matches ) — most recent opening    │
│            ↑       ↑                              │
│            [ matches ] — next most recent         │
│          ↑           ↑                            │
│          { matches } — first opening              │
└──────────────────────────────────────────────────┘
```

### Why Interleaved Brackets Fail

```
    "( [ ) ]"
        
Stack trace:

  Push (     Stack: [(]
  Push [     Stack: [(, []
  See )      Top is [ ← MISMATCH! ) ≠ [
  
  The ) tries to close [, but they don't match.
  Brackets must NEST, not INTERLEAVE.
  
  Valid nesting:    Invalid interleaving:
  
  ( [ ] )           ( [ ) ]
  ├─┤ ├─┤           ├─┤ ├─┤
  └───┘             └──X──┘  ← crossing!
```

---

## Algorithm

### Pseudocode

```
FUNCTION isValid(s):
    CREATE empty stack
    
    CREATE matchMap:
        ')' → '('
        '}' → '{'
        ']' → '['
    
    FOR each character c in s:
        IF c is '(' or '{' or '[':
            stack.push(c)
        ELSE:
            // c is a closing bracket
            IF stack.isEmpty():
                RETURN false    // nothing to match
            
            top ← stack.pop()
            IF top ≠ matchMap[c]:
                RETURN false    // wrong type
    
    RETURN stack.isEmpty()      // all brackets matched?
```

---

## Step-by-Step Trace

### Trace 1: `"{[()]}"`  → Valid

```
Step 1: char = '{'
  Action: Opening bracket → Push
  Stack: [{]
  
Step 2: char = '['
  Action: Opening bracket → Push
  Stack: [{, []
  
Step 3: char = '('
  Action: Opening bracket → Push
  Stack: [{, [, (]
  
Step 4: char = ')'
  Action: Closing bracket → Pop and check
  Pop: (    Match: ( == matchMap[')'] → ( ✓
  Stack: [{, []
  
Step 5: char = ']'
  Action: Closing bracket → Pop and check
  Pop: [    Match: [ == matchMap[']'] → [ ✓
  Stack: [{]
  
Step 6: char = '}'
  Action: Closing bracket → Pop and check
  Pop: {    Match: { == matchMap['}'] → { ✓
  Stack: []
  
Result: Stack is empty → VALID ✓
```

```
Visual:

  String: { [ ( ) ] }
          ↓ ↓ ↓ ↑ ↑ ↑
          │ │ │ │ │ │
  Stack:  │ │ └─┘ │ │
          │ └─────┘ │
          └─────────┘
          
  Perfect nesting!
```

### Trace 2: `"([)]"` → Invalid

```
Step 1: char = '('
  Action: Push
  Stack: [(]
  
Step 2: char = '['
  Action: Push
  Stack: [(, []
  
Step 3: char = ')'
  Action: Pop and check
  Pop: [    Match: [ ≠ matchMap[')'] → ( ✗ MISMATCH!
  
Result: INVALID ✗
```

```
Visual:

  String: ( [ ) ]
          ↓ ↓ ↑ ↑
          │ │ │ │
          │ └─┤ │    ← ] tries to close [, 
          └───┤      ← but ) gets there first
              X      ← INTERLEAVING ERROR
```

### Trace 3: `"((("` → Invalid (Unclosed)

```
Step 1: char = '('  → Push    Stack: [(]
Step 2: char = '('  → Push    Stack: [(, (]
Step 3: char = '('  → Push    Stack: [(, (, (]

End of string reached.
Stack is NOT empty → INVALID ✗

Three unclosed opening brackets remain.
```

---

## Edge Cases

```
┌────────────────────────┬────────┬──────────────────────────┐
│ Input                  │ Result │ Why                      │
├────────────────────────┼────────┼──────────────────────────┤
│ ""                     │ true   │ Empty string is valid    │
│ "("                    │ false  │ Unclosed bracket         │
│ ")"                    │ false  │ No matching opening      │
│ "()"                   │ true   │ Simple pair              │
│ ")("                   │ false  │ Wrong order              │
│ "(((())))"             │ true   │ Deep nesting             │
│ "()[]{}"               │ true   │ Sequential pairs         │
│ "([{}])"               │ true   │ Complex nesting          │
│ Single character       │ false  │ Always unmatched         │
│ Odd-length string      │ false  │ Cannot fully pair        │
└────────────────────────┴────────┴──────────────────────────┘
```

### Early Termination Optimization

```
FUNCTION isValid_Optimized(s):
    // Quick check: odd-length strings can never be valid
    IF length(s) is odd:
        RETURN false
    
    CREATE empty stack
    
    FOR each character c in s:
        IF c == '(':
            stack.push(')')     // Push expected closing
        ELSE IF c == '{':
            stack.push('}')     // Push expected closing
        ELSE IF c == '[':
            stack.push(']')     // Push expected closing
        ELSE:
            IF stack.isEmpty() OR stack.pop() ≠ c:
                RETURN false
    
    RETURN stack.isEmpty()
```

This optimized version pushes the **expected closing bracket** instead of the opening one, simplifying the comparison to a direct equality check.

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Single pass through string |
| **Space** | O(n) | Worst case: all opening brackets e.g. `"((("` |
| **Best Case Time** | O(1) | Odd-length string → immediate false |
| **Best Case Space** | O(1) | Odd-length string → no stack used |

### Space Analysis Detail

```
Best case:  s = ")"  → Stack never grows beyond 0
            Stack usage: O(1)

Average:    s = "([]){}"  → Stack grows to ~n/2
            Stack usage: O(n/2) = O(n)

Worst case: s = "((((((("  → All pushed, none popped
            Stack usage: O(n)
```

---

## Variations

### Variation 1: Count Minimum Bracket Insertions

Given a string of brackets, find the minimum number of insertions to make it valid.

```
FUNCTION minInsertions(s):
    stack ← empty
    insertions ← 0
    
    FOR each char c in s:
        IF c is opening bracket:
            stack.push(c)
        ELSE:
            IF stack is not empty AND matches(stack.top(), c):
                stack.pop()
            ELSE:
                insertions ← insertions + 1    // Need an opening
    
    insertions ← insertions + stack.size()      // Need closings for unmatched
    RETURN insertions

Example: "((]"
  ( → push     Stack: [(]
  ( → push     Stack: [(, (]
  ] → top is (, no match → insertions = 1
  End: stack size = 2, so insertions += 2
  Total: 3 insertions needed
```

### Variation 2: Remove Invalid Parentheses

Remove the minimum number of invalid brackets to make the string valid.

```
FUNCTION removeInvalid(s):
    indices_to_remove ← empty set
    stack ← empty  // stores (index, char) pairs
    
    FOR i = 0 TO length(s) - 1:
        IF s[i] is opening bracket:
            stack.push(i)
        ELSE IF s[i] is closing bracket:
            IF stack is not empty AND matches(s[stack.top()], s[i]):
                stack.pop()
            ELSE:
                indices_to_remove.add(i)
    
    // Remaining in stack are unmatched openings
    WHILE stack is not empty:
        indices_to_remove.add(stack.pop())
    
    result ← ""
    FOR i = 0 TO length(s) - 1:
        IF i NOT IN indices_to_remove:
            result ← result + s[i]
    
    RETURN result

Example: "(a(b)c)d)" 
  Index:  0 1 2 3 4 5 6 7 8
  Char:   ( a ( b ) c ) d )
  
  i=0: ( → push 0
  i=2: ( → push 2
  i=4: ) → match s[2]='(' → pop 2
  i=6: ) → match s[0]='(' → pop 0
  i=8: ) → stack empty → remove index 8
  
  Result: "(a(b)c)d"
```

### Variation 3: Longest Valid Parentheses Substring

Find the length of the longest valid substring of parentheses.

```
FUNCTION longestValid(s):
    stack ← empty
    stack.push(-1)    // Sentinel for boundary
    maxLen ← 0
    
    FOR i = 0 TO length(s) - 1:
        IF s[i] == '(':
            stack.push(i)
        ELSE:
            stack.pop()
            IF stack.isEmpty():
                stack.push(i)    // New boundary
            ELSE:
                maxLen ← max(maxLen, i - stack.top())
    
    RETURN maxLen

Example: ")()())"
  i=0: ) → pop -1, empty → push 0    Stack: [0]
  i=1: ( → push 1                     Stack: [0, 1]
  i=2: ) → pop 1, len=2-0=2, max=2   Stack: [0]
  i=3: ( → push 3                     Stack: [0, 3]
  i=4: ) → pop 3, len=4-0=4, max=4   Stack: [0]
  i=5: ) → pop 0, empty → push 5     Stack: [5]
  
  Result: 4 (the substring "()()")
```

---

## Common Mistakes

```
┌────────────────────────────────────────────────────────┐
│                   COMMON PITFALLS                      │
├────────────────────────────────────────────────────────┤
│                                                        │
│ 1. Forgetting to check if stack is empty before pop    │
│    → Causes underflow/crash on strings like ")"        │
│                                                        │
│ 2. Not checking stack emptiness at the end              │
│    → Misses unclosed brackets like "((("               │
│                                                        │
│ 3. Using counter instead of stack for multiple types   │
│    → Counter works for single type only                │
│    → "([)]" needs stack to detect interleaving         │
│                                                        │
│ 4. Wrong mapping direction                             │
│    → Map closing→opening (not opening→closing)         │
│    → We encounter closing bracket and need to verify   │
│                                                        │
│ 5. Not handling mixed content                          │
│    → Real strings have letters: "(a+b)*[c-d]"         │
│    → Skip non-bracket characters                       │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Check if string with `(){}[]` is valid |
| **Data Structure** | Stack |
| **Strategy** | Push openings, match closings against top |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(n) |
| **Key Optimization** | Odd-length → immediate false |
| **Trick** | Push expected closing instead of opening |
| **Common Variations** | Min insertions, remove invalid, longest valid |

---

## Quick Revision Questions

1. **Why can't a simple counter approach work for multiple bracket types?**
   > Because a counter only tracks quantity, not type. `"([)]"` has equal opening and closing counts but is invalid due to interleaving.

2. **What is the maximum stack size for a valid string of length n?**
   > n/2, because at most half the characters can be opening brackets in a valid string.

3. **Why do we check `stack.isEmpty()` at the end?**
   > To catch unmatched opening brackets that were pushed but never popped (e.g., `"(("`).

4. **What's the advantage of pushing the expected closing bracket?**
   > It simplifies the matching step to a direct equality check (`stack.pop() ≠ c`) instead of looking up a mapping.

5. **Can an odd-length string ever be valid?**
   > No. Every bracket needs a pair, so valid strings always have even length.

6. **How would you handle strings with non-bracket characters like `"(a+b)"`?**
   > Simply skip (ignore) any character that isn't a bracket. Only process `(`, `)`, `{`, `}`, `[`, `]`.

---

[← Previous: Balanced Parentheses](01-balanced-parentheses.md) | [Next: Reverse a String →](03-reverse-string.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)

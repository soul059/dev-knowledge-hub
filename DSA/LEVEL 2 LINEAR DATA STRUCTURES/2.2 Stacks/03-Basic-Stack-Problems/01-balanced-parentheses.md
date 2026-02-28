# Balanced Parentheses Problem

## Overview

The **balanced parentheses** problem is one of the most fundamental stack applications. It asks: given a string of parentheses `()`, brackets `[]`, and braces `{}`, determine if they are properly balanced and nested.

This problem teaches pattern matching, stack usage for tracking state, and forms the foundation for more complex parsing problems.

---

## Problem Statement

```
Input: A string containing '(', ')', '{', '}', '[', ']'
Output: true if balanced, false otherwise

Balanced means:
1. Every opening bracket has a matching closing bracket
2. Brackets are closed in correct order
3. Closing bracket matches the most recent opening bracket

Examples:
✓ "()"           → true
✓ "()[]{}"       → true
✓ "{[()]}"       → true
✗ "([)]"         → false (wrong order)
✗ "((("          → false (unclosed)
✗ ")"            → false (no opening)
```

---

## Key Insight

```
Why Stack?

Most recent opening bracket must match first!

Example: {[()]}

Read:  {  [  (  )  ]  }
       ↓  ↓  ↓
Push opens, pop on close matching most recent

Stack evolution:
{ → {[ → {[( → {[ → { → empty ✓

LIFO perfectly matches nested structure!
```

---

## Algorithm

### Approach

```
Algorithm:
1. Create empty stack
2. For each character:
   a. If opening bracket → push
   b. If closing bracket:
      - If stack empty → return false
      - Pop and check if it matches
      - If mismatch → return false
3. After processing:
   - If stack empty → return true
   - If stack not empty → return false
```

### Pseudocode

```pseudocode
FUNCTION isBalanced(s):
    stack = new Stack()
    
    FOR each character c in s:
        // Opening brackets - push
        IF c in ['(', '{', '['] THEN
            stack.push(c)
        
        // Closing brackets - pop and match
        ELSE IF c in [')', '}', ']'] THEN
            IF stack.isEmpty() THEN
                RETURN false  // No matching opening
            END IF
            
            opening = stack.pop()
            
            IF NOT matches(opening, c) THEN
                RETURN false  // Mismatch
            END IF
        END IF
    END FOR
    
    // Check if all brackets closed
    RETURN stack.isEmpty()
END FUNCTION

FUNCTION matches(open, close):
    RETURN (open = '(' AND close = ')') OR
           (open = '{' AND close = '}') OR
           (open = '[' AND close = ']')
END FUNCTION
```

---

## Step-by-Step Trace

### Example 1: "{[()]}" - Valid

```
Input: "{[()]}"

Step 1: Read '{'
Stack: ['{']
Action: Push opening

Step 2: Read '['
Stack: ['{', '[']
Action: Push opening

Step 3: Read '('
Stack: ['{', '[', '(']
Action: Push opening

Step 4: Read ')'
Stack: ['{', '[']
Action: Pop '(', matches ')' ✓

Step 5: Read ']'
Stack: ['{']
Action: Pop '[', matches ']' ✓

Step 6: Read '}'
Stack: []
Action: Pop '{', matches '}' ✓

Final: Stack empty → TRUE ✓

Visual:
         Input: { [ ( ) ] }
                ↓ ↓ ↓ ↑ ↑ ↑
Stack Growth:  { ← ← matches
              {[ ← matches
             {[( matches
```

### Example 2: "([)]" - Invalid

```
Input: "([)]"

Step 1: Read '('
Stack: ['(']

Step 2: Read '['
Stack: ['(', '[']

Step 3: Read ')'
Stack: ['(']
Pop '[', try match with ')'
'[' does NOT match ')' ✗

RETURN FALSE

Problem: Expected ']' to close '[', got ')' instead
```

### Example 3: "(((" - Invalid

```
Input: "((("

Step 1-3: Read three '('
Stack: ['(', '(', '(']

End of string
Stack NOT empty ✗

RETURN FALSE

Problem: Unclosed opening brackets
```

---

## Complexity Analysis

```
Time Complexity: O(n)
- One pass through string
- Each character processed once
- Push/Pop are O(1)
- Total: O(n)

Space Complexity: O(n)
- Worst case: all opening brackets
- Example: "(((((" requires stack of size n
- Best case: O(1) for balanced pairs "()()()

Where n = length of string
```

---

## Implementation Variations

### Using Character Map

```pseudocode
FUNCTION isBalanced(s):
    stack = new Stack()
    
    // Map closing to opening brackets
    pairs = {')': '(', '}': '{', ']': '['}
    
    FOR each character c in s:
        IF c in ['(', '{', '['] THEN
            stack.push(c)
        
        ELSE IF c in pairs.keys() THEN
            IF stack.isEmpty() OR stack.pop() ≠ pairs[c] THEN
                RETURN false
            END IF
        END IF
    END FOR
    
    RETURN stack.isEmpty()
END FUNCTION
```

### Without Stack (Using Counter) - Only for Single Type

```
// Works only for single bracket type ()
FUNCTION isBalancedSimple(s):
    count = 0
    
    FOR each character c in s:
        IF c = '(' THEN
            count++
        ELSE IF c = ')' THEN
            count--
            IF count < 0 THEN
                RETURN false
            END IF
        END IF
    END FOR
    
    RETURN count = 0
END FUNCTION

Note: This doesn't work for mixed types!
Example: "([)]" would incorrectly return true
```

---

## Common Mistakes

```
❌ Mistake 1: Not checking stack empty before pop
Solution: Always check isEmpty() first

❌ Mistake 2: Not checking if stack empty at end
Solution: Return stack.isEmpty(), not just true

❌ Mistake 3: Using counter for multiple bracket types
Solution: Must use stack for proper matching

❌ Mistake 4: Matching any open with any close
Solution: Must match specific types: ( with ), [ with ], etc.
```

---

## Variations of the Problem

### 1. Only Parentheses

```
Input: Only '(' and ')'
Simpler - can use counter approach

FUNCTION isValid(s):
    count = 0
    FOR c in s:
        count += (c = '(' ? 1 : -1)
        IF count < 0: RETURN false
    RETURN count = 0
```

### 2. Remove Invalid Parentheses

```
Input: "(()"
Output: "()" - remove one '('

Requires: BFS or DFS with stack
More complex variation
```

### 3. Minimum Additions to Balance

```
Input: "((("
Output: 3 - need three closing brackets

FUNCTION minAdditions(s):
    stack = new Stack()
    FOR c in s:
        IF c = '(' THEN push
        ELSE IF NOT empty THEN pop
    RETURN stack.size() + unmatched_closes
```

---

## Practice Problems

```
1. Basic:
   - Check if string has balanced parentheses ()
   
2. Intermediate:
   - Check multiple bracket types ()[]{} ✓ (this chapter)
   - Return position of first unbalanced bracket
   
3. Advanced:
   - Longest valid parentheses substring
   - Score of parentheses
   - Remove invalid parentheses (minimum removals)
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem Type** | String validation, pattern matching |
| **Data Structure** | Stack |
| **Key Insight** | Most recent opening matches first closing |
| **Time Complexity** | O(n) - single pass |
| **Space Complexity** | O(n) worst case |
| **Difficulty** | Easy to Medium |
| **Applications** | Compilers, syntax checking, code editors |
| **Pattern** | Matching pairs with nesting |

---

## Quick Revision Questions

1. **Q: Why can't we use a simple counter for multiple bracket types?**
   - A: Counter can't distinguish between types. Example: "([)]" has balanced counts but wrong order - we need stack to match specific types.

2. **Q: What does an empty stack at the end indicate?**
   - A: All opening brackets have been properly closed and matched.

3. **Q: What happens if we read a closing bracket and the stack is empty?**
   - A: Invalid - the closing bracket has no matching opening bracket. Return false immediately.

4. **Q: In "{[()]}", when we read ')', what do we expect to pop?**
   - A: '(' - the most recent opening bracket, which the stack provides via LIFO.

5. **Q: What is the space complexity for a balanced string like "()()()"?**
   - A: O(1) in practice - stack never grows beyond size 1, but worst case is still O(n) for strings like "(((((".

6. **Q: Can this algorithm handle strings with non-bracket characters?**
   - A: Yes! Just skip non-bracket characters, or modify to only process brackets.

---

## Navigation

- **[← Previous: Implementation Trade-offs](../02-Stack-Implementation/06-implementation-tradeoffs.md)**
- **[Next: Valid Parentheses →](02-valid-parentheses.md)**
- **[↑ Back to Unit 3](README.md)**
- **[⌂ Home](../README.md)**

---

*This is a fundamental problem - master it to build intuition for all stack-based matching problems!*

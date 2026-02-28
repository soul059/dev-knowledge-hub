# Chapter 3: Reverse a String Using Stack

[← Previous: Valid Parentheses](02-valid-parentheses.md) | [Next: Reverse a Stack →](04-reverse-stack.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)

---

## Overview

Reversing a string is one of the simplest and most elegant applications of a stack. It directly exploits the **LIFO (Last In, First Out)** property: push all characters in order, then pop them all out to get the reverse.

---

## Problem Statement

Given a string, reverse it using a stack.

```
Input:  "HELLO"
Output: "OLLEH"

Input:  "stacks"
Output: "skcats"

Input:  "abcde"
Output: "edcba"
```

---

## Key Insight

```
┌───────────────────────────────────────────────┐
│          WHY STACK REVERSES ORDER              │
│                                               │
│  Push order:  H → E → L → L → O              │
│  Pop  order:  O → L → L → E → H              │
│                                               │
│  First pushed = Last popped  → REVERSAL!      │
│                                               │
│      PUSH                    POP              │
│  ┌─────────┐           ┌─────────┐           │
│  │ O │ ← last in       │ O │ ← first out     │
│  │ L │                  │ L │                  │
│  │ L │                  │ L │                  │
│  │ E │                  │ E │                  │
│  │ H │ ← first in      │ H │ ← last out      │
│  └─────────┘           └─────────┘           │
└───────────────────────────────────────────────┘
```

---

## Algorithm

### Pseudocode

```
FUNCTION reverseString(str):
    CREATE empty stack
    
    // Phase 1: Push all characters
    FOR each character c in str:
        stack.push(c)
    
    // Phase 2: Pop all characters
    reversed ← ""
    WHILE stack is NOT empty:
        reversed ← reversed + stack.pop()
    
    RETURN reversed
```

---

## Step-by-Step Trace

### Input: `"HELLO"`

```
═══════════════════════════════════════════════
  PHASE 1: PUSHING CHARACTERS
═══════════════════════════════════════════════

Step 1: Push 'H'          Step 2: Push 'E'
  ┌───┐                     ┌───┐
  │   │                     │   │
  │   │                     │   │
  │   │                     │   │
  │   │                     │ E │ ← top
  │ H │ ← top               │ H │
  └───┘                     └───┘

Step 3: Push 'L'          Step 4: Push 'L'
  ┌───┐                     ┌───┐
  │   │                     │   │
  │   │                     │ L │ ← top
  │ L │ ← top               │ L │
  │ E │                     │ E │
  │ H │                     │ H │
  └───┘                     └───┘

Step 5: Push 'O'
  ┌───┐
  │ O │ ← top
  │ L │
  │ L │
  │ E │
  │ H │
  └───┘

═══════════════════════════════════════════════
  PHASE 2: POPPING CHARACTERS
═══════════════════════════════════════════════

Pop 1: 'O'    reversed = "O"
  ┌───┐
  │ L │ ← top
  │ L │
  │ E │
  │ H │
  └───┘

Pop 2: 'L'    reversed = "OL"
  ┌───┐
  │ L │ ← top
  │ E │
  │ H │
  └───┘

Pop 3: 'L'    reversed = "OLL"
  ┌───┐
  │ E │ ← top
  │ H │
  └───┘

Pop 4: 'E'    reversed = "OLLE"
  ┌───┐
  │ H │ ← top
  └───┘

Pop 5: 'H'    reversed = "OLLEH"
  ┌───┐
  │   │ ← empty
  └───┘

Result: "OLLEH" ✓
```

---

## Complexity Analysis

| Aspect | Complexity | Explanation |
|--------|-----------|-------------|
| **Time** | O(n) | Push all n chars + Pop all n chars = 2n = O(n) |
| **Space** | O(n) | Stack holds all n characters |

### Comparison with Other Reversal Methods

```
┌────────────────────────┬──────────┬───────────┐
│ Method                 │ Time     │ Space     │
├────────────────────────┼──────────┼───────────┤
│ Stack-based            │ O(n)     │ O(n)      │
│ Two-pointer swap       │ O(n)     │ O(1)      │
│ Recursion              │ O(n)     │ O(n)*     │
│ New string (iterate)   │ O(n)     │ O(n)      │
│ Built-in reverse       │ O(n)     │ varies    │
└────────────────────────┴──────────┴───────────┘
* Recursion uses O(n) call stack space
```

> **Note**: The two-pointer swap method is more space-efficient (O(1)), but the stack-based approach is a great exercise in understanding stack behavior.

---

## Variations

### Variation 1: Reverse Words in a Sentence

Reverse the order of words, not individual characters.

```
Input:  "Hello World Stacks"
Output: "Stacks World Hello"

FUNCTION reverseWords(sentence):
    words ← split(sentence, " ")
    stack ← empty
    
    FOR each word in words:
        stack.push(word)
    
    result ← ""
    WHILE stack is NOT empty:
        result ← result + stack.pop()
        IF stack is NOT empty:
            result ← result + " "
    
    RETURN result

Trace:
  Push: "Hello" → "World" → "Stacks"
  
  Stack:  ┌─────────┐
          │ Stacks  │ ← top
          │ World   │
          │ Hello   │
          └─────────┘
  
  Pop: "Stacks" → "World" → "Hello"
  Result: "Stacks World Hello"
```

### Variation 2: Reverse Only Certain Characters

Reverse only letters, keeping non-letter characters in place.

```
Input:  "a-b-c"
Output: "c-b-a"

FUNCTION reverseLetters(str):
    stack ← empty
    
    // Push only letters
    FOR each char c in str:
        IF isLetter(c):
            stack.push(c)
    
    // Rebuild string
    result ← ""
    FOR each char c in str:
        IF isLetter(c):
            result ← result + stack.pop()
        ELSE:
            result ← result + c    // Keep non-letter in place
    
    RETURN result

Trace for "a-b-c":
  Push letters: a, b, c
  Stack: [a, b, c]  (top = c)
  
  Rebuild:
    'a' → letter → pop 'c' → "c"
    '-' → not letter → keep → "c-"
    'b' → letter → pop 'b' → "c-b"
    '-' → not letter → keep → "c-b-"
    'c' → letter → pop 'a' → "c-b-a"
```

### Variation 3: Check if String is Palindrome

Use a stack to check if a string reads the same forwards and backwards.

```
FUNCTION isPalindrome(str):
    stack ← empty
    n ← length(str)
    
    // Push first half
    FOR i = 0 TO (n/2 - 1):
        stack.push(str[i])
    
    // Skip middle character if odd length
    start ← n/2
    IF n is odd:
        start ← start + 1
    
    // Compare second half with stack
    FOR i = start TO n - 1:
        IF stack.pop() ≠ str[i]:
            RETURN false
    
    RETURN true

Trace for "RACECAR":
  Push: R, A, C
  Skip middle: E
  Compare: C=C ✓, A=A ✓, R=R ✓
  Result: true (palindrome!)
```

---

## Practical Applications

```
┌──────────────────────────────────────────────┐
│        WHERE STRING REVERSAL IS USED         │
├──────────────────────────────────────────────┤
│                                              │
│  1. Palindrome checking                      │
│  2. DNA sequence complement                  │
│  3. Number base conversion                   │
│  4. Binary to decimal (reversed remainders)  │
│  5. Undo operations (reverse last action)    │
│  6. Text processing and formatting           │
│  7. Compiler: reversing token streams        │
│  8. Cryptography: simple substitution        │
│                                              │
└──────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Reverse a string using a stack |
| **Core Idea** | LIFO naturally reverses insertion order |
| **Algorithm** | Push all → Pop all |
| **Time Complexity** | O(n) |
| **Space Complexity** | O(n) |
| **Better Alternatives** | Two-pointer swap for O(1) space |
| **Variations** | Reverse words, reverse letters only, palindrome check |

---

## Quick Revision Questions

1. **Why does a stack naturally reverse order?**
   > Because of LIFO: the first element pushed is the last one popped, and the last element pushed is the first one popped.

2. **What is the space complexity of stack-based string reversal?**
   > O(n), since all n characters are stored in the stack simultaneously.

3. **Is stack the most efficient way to reverse a string?**
   > No. The two-pointer swap method does it in O(1) space. The stack approach is O(n) space but demonstrates LIFO behavior clearly.

4. **How would you reverse words but not characters within words?**
   > Push entire words (not characters) onto the stack, then pop them all.

5. **How can you use a stack for palindrome checking?**
   > Push the first half of the string, then compare pops with the second half. If all match, it's a palindrome.

---

[← Previous: Valid Parentheses](02-valid-parentheses.md) | [Next: Reverse a Stack →](04-reverse-stack.md) | [↑ Back to Unit 3](../README.md#unit-3-basic-stack-problems) | [🏠 Home](../README.md)

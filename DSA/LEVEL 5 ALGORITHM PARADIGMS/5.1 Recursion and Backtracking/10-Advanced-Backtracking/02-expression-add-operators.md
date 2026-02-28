# Chapter 2: Expression Add Operators

## Overview
Given a string of digits and a target value, insert `+`, `-`, or `*` between digits to reach the target. This is one of the **hardest backtracking problems** because it combines string partitioning with arithmetic evaluation, and multiplication introduces operator precedence challenges.

---

## Problem Statement

```
Input:  num = "123", target = 6
Output: ["1+2+3", "1*2*3"]

Input:  num = "232", target = 8
Output: ["2+3*2", "2*3+2"]

Input:  num = "105", target = 5
Output: ["1*0+5", "10-5"]

Key challenges:
  1. Multi-digit numbers (e.g., "10" from "105")
  2. Leading zeros NOT allowed (e.g., "05" invalid)
  3. Multiplication precedence (2+3*2 = 8, not 10)
```

---

## The Precedence Problem

```
Evaluating left-to-right gives WRONG results for ×:
  "2+3*2" left-to-right = (2+3)*2 = 10  ✗
  "2+3*2" correct       = 2+(3*2) = 8   ✓

Solution: Track the LAST operand for multiplication undo.

When we see ×, we need to:
  1. Subtract the last operand from running total
  2. Add (last operand × current number)

Example: evaluating "2+3*2"
  After "2":     value=2,   last=2
  After "2+3":   value=5,   last=3
  After "2+3*2": 
    value = 5 - 3 + (3*2) = 8  ✓
    last  = 3*2 = 6
```

---

## Pseudocode

```
FUNCTION addOperators(num, target):
    result = []
    backtrack(num, target, 0, "", 0, 0, result)
    RETURN result

FUNCTION backtrack(num, target, index, expr, value, last, result):
    IF index == length(num):
        IF value == target:
            result.add(expr)             ← Valid expression!
        RETURN
    
    FOR i = index TO length(num)-1:
        // Prevent leading zeros: "05" invalid
        IF i > index AND num[index] == '0':
            BREAK
        
        curStr = num[index..i]
        curNum = parseInt(curStr)
        
        IF index == 0:
            // First number (no operator before it)
            backtrack(num, target, i+1, curStr, curNum, curNum, result)
        ELSE:
            // Try +
            backtrack(num, target, i+1, 
                      expr + "+" + curStr,
                      value + curNum,
                      curNum, result)
            
            // Try -
            backtrack(num, target, i+1,
                      expr + "-" + curStr,
                      value - curNum,
                      -curNum, result)
            
            // Try *  (undo last, apply multiplication)
            backtrack(num, target, i+1,
                      expr + "*" + curStr,
                      value - last + last * curNum,
                      last * curNum, result)
```

---

## Trace: num = "123", target = 6

```
backtrack("123", 6, 0, "", 0, 0)
│
├─ i=0: cur="1", num=1
│  backtrack("123", 6, 1, "1", 1, 1)
│  │
│  ├─ i=1: cur="2", num=2
│  │  ├─ +: backtrack(2, "1+2", 3, 2)
│  │  │  ├─ i=2: cur="3", num=3
│  │  │  │  ├─ +: backtrack(3, "1+2+3", 6, 3)
│  │  │  │  │  index==3, value==6==target → ADD "1+2+3" ✓
│  │  │  │  ├─ -: backtrack(3, "1+2-3", 0, -3) → 0≠6
│  │  │  │  ├─ *: backtrack(3, "1+2*3", 3-2+6=7, 6) → 7≠6
│  │  │
│  │  ├─ -: backtrack(2, "1-2", -1, -2)
│  │  │  ├─ +: "1-2+3" → 0  ✗
│  │  │  ├─ -: "1-2-3" → -4 ✗
│  │  │  ├─ *: "1-2*3" → -1-(-2)+(-2)*3 = -5 ✗
│  │  │
│  │  ├─ *: backtrack(2, "1*2", 1-1+1*2=2, 2)
│  │  │  ├─ +: "1*2+3" → 5  ✗
│  │  │  ├─ -: "1*2-3" → -1 ✗
│  │  │  ├─ *: "1*2*3" → 2-2+2*3=6, last=6
│  │  │  │  index==3, value==6==target → ADD "1*2*3" ✓
│  │
│  ├─ i=2: cur="23", num=23
│  │  ├─ +: "1+23" → 24 ✗
│  │  ├─ -: "1-23" → -22 ✗
│  │  ├─ *: "1*23" → 23 ✗
│
├─ i=1: cur="12", num=12
│  backtrack("123", 6, 2, "12", 12, 12)
│  ├─ +: "12+3" → 15 ✗
│  ├─ -: "12-3" → 9 ✗
│  ├─ *: "12*3" → 36 ✗
│
├─ i=2: cur="123", num=123
│  backtrack("123", 6, 3, "123", 123, 123)
│  → 123≠6 ✗

Result: ["1+2+3", "1*2*3"]
```

---

## Handling Leading Zeros

```
Input: num = "105", target = 5

Valid splits:
  "1", "0", "5"    ← 0 alone is valid
  "1", "05"        ← "05" has leading zero → INVALID!
  "10", "5"        ← "10" is fine (no leading zero)
  "105"            ← fine

Rule: IF i > index AND num[index] == '0': BREAK
  → Once we see a leading zero, no longer splits are valid
  → "0" alone is okay (single digit)
  → "05", "052" etc. are all invalid
```

---

## Handling Overflow

```
For very long digit strings, numbers can overflow int/long.
In competitive programming:
  - Use long/long long (64-bit)
  - Some problems limit to 10 digits (fits in long)
  - Python has arbitrary precision (no overflow)

Also watch for:
  - Target might be negative
  - curNum can be very large for multi-digit numbers
```

---

## Complexity

```
┌───────────────────────────────────────────────────────┐
│ At each gap between digits: 4 choices                 │
│   (join digits, +, -, *)                              │
│                                                       │
│ n-1 gaps between n digits                             │
│                                                       │
│ Time:  O(4^n × n)                                     │
│   4^n paths × O(n) for string operations per path     │
│                                                       │
│ Space: O(n) recursion depth                           │
│   + O(4^n × n) to store all valid expressions         │
└───────────────────────────────────────────────────────┘
```

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Input | Digit string + target integer |
| Choices | Split point + operator (+, -, *) |
| Key Trick | Track `last` operand for × precedence |
| × formula | value - last + last*curNum |
| Leading zero | Break if i > index and first char is '0' |
| First number | No operator, just set value and last |
| Time | O(4^n × n) |
| Space | O(n) recursion stack |

---

## Quick Revision Questions

1. **Why do we track** the `last` operand?
2. **How is multiplication** handled differently from + and -?
3. **Why do we break** on leading zeros instead of just skipping?
4. **What are the 4 choices** at each position between digits?
5. **Why is the first number** handled as a special case?
6. **What's the multiplication** value update formula?

---

[← Previous: Palindrome Partitioning](01-palindrome-partitioning.md) | [Next: Word Break II →](03-word-break-ii.md)

[← Back to README](../README.md)

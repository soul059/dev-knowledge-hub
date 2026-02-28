# Chapter 1: Stack Fundamentals

## 📋 Chapter Overview
Core stack concepts, operations, and foundational problems: valid parentheses, evaluate expressions, and basic calculator.

---

## 🧠 Stack — LIFO Principle

```
  PUSH 1    PUSH 2    PUSH 3    POP → 3   POP → 2
  
  ┌───┐     ┌───┐     ┌───┐     ┌───┐     ┌───┐
  │   │     │   │     │ 3 │     │   │     │   │
  │   │     │ 2 │     │ 2 │     │ 2 │     │   │
  │ 1 │     │ 1 │     │ 1 │     │ 1 │     │ 1 │
  └───┘     └───┘     └───┘     └───┘     └───┘
  
  Last In, First Out
  All operations: O(1)
```

---

## 📝 Problem 1: Valid Parentheses (LeetCode 20)

**Problem:** Given string of `()[]{}`, determine if valid.

```
function isValid(s):
    stack = []
    map = { ')':'(', ']':'[', '}':'{' }
    
    for each char c in s:
        if c is opening bracket:
            stack.push(c)
        else:
            if stack is empty or stack.pop() ≠ map[c]:
                return false
    
    return stack is empty
```

### Trace

```
  s = "({[]})"
  
  '(' → push. stack = ['(']
  '{' → push. stack = ['(', '{']
  '[' → push. stack = ['(', '{', '[']
  ']' → pop '[' == map[']'] ✓. stack = ['(', '{']
  '}' → pop '{' == map['}'] ✓. stack = ['(']
  ')' → pop '(' == map[')'] ✓. stack = []
  
  Stack empty → true ✓
```

---

## 📝 Problem 2: Evaluate Reverse Polish Notation (LeetCode 150)

**Problem:** Evaluate expression in postfix notation.

```
function evalRPN(tokens):
    stack = []
    
    for each token:
        if token is number:
            stack.push(number)
        else:
            b = stack.pop()
            a = stack.pop()
            stack.push(apply(a, token, b))
    
    return stack.pop()
```

### Trace

```
  tokens = ["2","1","+","3","*"]
  
  "2" → push 2.         stack = [2]
  "1" → push 1.         stack = [2, 1]
  "+" → pop 1,2 → 2+1=3. stack = [3]
  "3" → push 3.         stack = [3, 3]
  "*" → pop 3,3 → 3*3=9. stack = [9]
  
  Result: 9
  Equivalent infix: (2 + 1) * 3
```

---

## 📝 Problem 3: Min Stack (LeetCode 155)

**Problem:** Design stack that supports push, pop, top, and getMin in O(1).

```
  Approach: two stacks — one for values, one for current minimums
  
class MinStack:
    stack = []
    minStack = []
    
    push(val):
        stack.push(val)
        if minStack is empty or val <= minStack.top():
            minStack.push(val)
    
    pop():
        if stack.pop() == minStack.top():
            minStack.pop()
    
    top():
        return stack.top()
    
    getMin():
        return minStack.top()
```

### Trace

```
  push(5):  stack=[5]      minStack=[5]
  push(3):  stack=[5,3]    minStack=[5,3]
  push(7):  stack=[5,3,7]  minStack=[5,3]    (7 > 3, don't push to min)
  getMin(): → 3
  pop():    pop 7 ≠ 3 → minStack unchanged
            stack=[5,3]    minStack=[5,3]
  pop():    pop 3 == 3 → pop minStack too
            stack=[5]      minStack=[5]
  getMin(): → 5
```

---

## 📝 Problem 4: Basic Calculator II (LeetCode 227)

**Problem:** Evaluate string expression with `+`, `-`, `*`, `/` (no parentheses).

```
function calculate(s):
    stack = []
    num = 0
    sign = '+'
    
    for i = 0 to len(s):
        if s[i] is digit:
            num = num * 10 + (s[i] - '0')
        
        if (s[i] is operator or i == len(s)-1):
            if sign == '+': stack.push(num)
            if sign == '-': stack.push(-num)
            if sign == '*': stack.push(stack.pop() * num)
            if sign == '/': stack.push(trunc(stack.pop() / num))
            sign = s[i]
            num = 0
    
    return sum(stack)
```

### Trace

```
  s = "3+2*2"
  
  '3': num=3
  '+': sign=='+' → push 3. sign='+'. num=0. stack=[3]
  '2': num=2
  '*': sign=='+' → push 2. sign='*'. num=0. stack=[3,2]
  '2': num=2
  end: sign=='*' → push pop()*2 = 2*2=4. stack=[3,4]
  
  Sum: 3+4 = 7
```

---

## 📐 Stack Applications Overview

```
  ┌────────────────────────┬───────────────────────────────┐
  │ Application            │ Why Stack?                    │
  ├────────────────────────┼───────────────────────────────┤
  │ Parentheses matching   │ Match inner-to-outer (LIFO)   │
  │ Expression evaluation  │ Hold operands until operator  │
  │ Undo operations        │ Reverse chronological order   │
  │ DFS (iterative)        │ Simulate recursion stack      │
  │ Monotonic problems     │ Maintain order invariant      │
  │ History tracking       │ Browser back/forward          │
  └────────────────────────┴───────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---------|-------------|
| Stack | LIFO, O(1) push/pop/peek |
| Valid parentheses | Push open, pop and match on close |
| RPN evaluation | Push numbers, pop two on operator |
| Min stack | Auxiliary stack tracking minimums |
| Calculator | Delayed evaluation with sign tracking |

---

## ❓ Revision Questions

1. **Stack vs Queue?** → Stack is LIFO (last in, first out); Queue is FIFO (first in, first out).
2. **Valid parentheses: why stack?** → Inner brackets must close before outer — matches LIFO order.
3. **Min stack: why auxiliary stack?** → Tracks the minimum at each depth level; O(1) getMin without scanning.
4. **RPN: why no parentheses needed?** → Postfix notation inherently encodes precedence by position.
5. **Calculator: why push on `+/-` and compute on `*,/`?** → Multiplication/division have higher precedence; must evaluate immediately. Addition/subtraction deferred to final sum.

---

[← Previous: Greedy — When to Apply](../08-Greedy-Pattern/06-when-to-apply.md) | [Next: Monotonic Stack →](02-monotonic-stack.md)

# Chapter 4: Syntax Parsing

[← Previous: Browser History](03-browser-history.md) | [Next: Backtracking with Stack →](05-backtracking.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

---

## Overview

**Syntax parsing** is one of the oldest and most important stack applications. Compilers, interpreters, and text processors use stacks to parse nested structures — from parentheses matching to full expression evaluation. This chapter covers practical parsing applications beyond basic bracket matching.

---

## Application 1: Simplify Path (Unix Path)

```
Given a Unix file path, simplify it to its canonical form.

Input:  "/home/../usr/./local/../bin/./test/"
Output: "/usr/bin/test"

Rules:
  "."  → current directory (ignore)
  ".." → parent directory (go up one level)
  Multiple slashes → treat as one
```

### Algorithm

```
FUNCTION simplifyPath(path):
    stack ← empty stack
    parts ← split path by "/"
    
    FOR each part in parts:
        IF part == "" OR part == ".":
            CONTINUE                  // Skip empty and current dir
        ELSE IF part == "..":
            IF stack NOT empty:
                stack.pop()           // Go up one level
        ELSE:
            stack.push(part)          // Valid directory name
    
    RETURN "/" + join(stack, "/")
```

### Trace: "/home/../usr/./local/../bin/./test/"

```
parts: ["", "home", "..", "usr", ".", "local", "..", "bin", ".", "test", ""]

"":      skip
"home":  push → stack: ["home"]
"..":    pop  → stack: []
"usr":   push → stack: ["usr"]
".":     skip
"local": push → stack: ["usr", "local"]
"..":    pop  → stack: ["usr"]
"bin":   push → stack: ["usr", "bin"]
".":     skip
"test":  push → stack: ["usr", "bin", "test"]
"":      skip

Result: "/usr/bin/test" ✓
```

---

## Application 2: Basic Calculator

```
Evaluate: "1 + (2 - (3 + 4))"

Uses stack for nested parentheses with sign tracking.

Step through:
  1 + (2 - (3 + 4))
  = 1 + (2 - 7)
  = 1 + (-5)
  = -4
```

### Algorithm

```
FUNCTION basicCalculator(s):
    stack ← empty stack
    result ← 0
    num ← 0
    sign ← +1
    
    FOR each char c in s:
        IF c is digit:
            num ← num × 10 + (c - '0')
        
        ELSE IF c == '+':
            result ← result + sign × num
            num ← 0
            sign ← +1
        
        ELSE IF c == '-':
            result ← result + sign × num
            num ← 0
            sign ← -1
        
        ELSE IF c == '(':
            stack.push(result)    // Save current result
            stack.push(sign)      // Save current sign
            result ← 0           // Reset for sub-expression
            sign ← +1
        
        ELSE IF c == ')':
            result ← result + sign × num
            num ← 0
            prevSign ← stack.pop()
            prevResult ← stack.pop()
            result ← prevResult + prevSign × result
    
    // Don't forget the last number
    result ← result + sign × num
    RETURN result
```

### Trace: "1 + (2 - (3 + 4))"

```
c='1': num=1
c='+': result=0+1×1=1, num=0, sign=+1
c='(': push(1), push(+1), reset result=0, sign=+1
       stack: [1, +1]
c='2': num=2
c='-': result=0+1×2=2, num=0, sign=-1
c='(': push(2), push(-1), reset result=0, sign=+1
       stack: [1, +1, 2, -1]
c='3': num=3
c='+': result=0+1×3=3, num=0, sign=+1
c='4': num=4
c=')': result=3+1×4=7. Pop sign=-1, pop prevResult=2
       result=2+(-1)×7=2-7=-5
       stack: [1, +1]
c=')': result=-5+1×0=-5. Pop sign=+1, pop prevResult=1
       result=1+1×(-5)=1-5=-4
       stack: []

Result: -4 ✓
```

---

## Application 3: HTML/XML Tag Matching

```
Verify that HTML tags are properly nested:

<html>
  <body>
    <p>Hello</p>
    <div>
      <span>World</span>
    </div>
  </body>
</html>

Algorithm: Push opening tags, pop and match with closing tags.
```

```
FUNCTION validateHTML(tokens):
    stack ← empty stack
    
    FOR each token:
        IF token is opening tag (e.g., <div>):
            stack.push(tag_name)
        
        ELSE IF token is closing tag (e.g., </div>):
            IF stack is empty:
                RETURN false        // No matching open tag
            IF stack.top() != tag_name:
                RETURN false        // Mismatched tags
            stack.pop()
        
        // Self-closing tags (<br/>) are ignored
    
    RETURN stack is empty    // All tags matched
```

### Trace: "<html><body><p></p></body></html>"

```
<html>:  push "html"    stack: ["html"]
<body>:  push "body"    stack: ["html", "body"]
<p>:     push "p"       stack: ["html", "body", "p"]
</p>:    top="p" ✓ pop  stack: ["html", "body"]
</body>: top="body" ✓   stack: ["html"]
</html>: top="html" ✓   stack: []

Empty stack → Valid ✓
```

---

## Application 4: Nested List Parsing

```
Parse: "[1,[2,3],[4,[5,6]]]"

Stack-based parsing of nested structures:

c='[':  Push new list onto stack
digit:  Add number to current list (stack top)
c=',':  Separator (skip)
c=']':  Pop completed list, append to parent list
```

```
Trace of "[1,[2,3],[4,[5,6]]]":

'[':    push new list        stack: [[]]
'1':    add to top           stack: [[1]]
',':    skip
'[':    push new list        stack: [[1], []]
'2':    add to top           stack: [[1], [2]]
',':    skip
'3':    add to top           stack: [[1], [2,3]]
']':    pop [2,3], add to parent  stack: [[1, [2,3]]]
',':    skip
'[':    push                 stack: [[1,[2,3]], []]
'4':    add                  stack: [[1,[2,3]], [4]]
',':    skip
'[':    push                 stack: [[1,[2,3]], [4], []]
'5':    add                  stack: [[1,[2,3]], [4], [5]]
',':    skip
'6':    add                  stack: [[1,[2,3]], [4], [5,6]]
']':    pop, add to parent   stack: [[1,[2,3]], [4,[5,6]]]
']':    pop, add to parent   stack: [[1,[2,3],[4,[5,6]]]]

Result: [1, [2,3], [4, [5,6]]] ✓
```

---

## The Common Stack-Parsing Pattern

```
┌──────────────────────────────────────────────────────────┐
│  All syntax parsing problems follow the same pattern:    │
│                                                          │
│  Opening delimiter ('(', '[', '{', '<tag>'):             │
│    → PUSH context (save current state)                  │
│    → Start fresh for nested content                     │
│                                                          │
│  Closing delimiter (')', ']', '}', '</tag>'):            │
│    → POP context (restore previous state)               │
│    → Combine inner result with outer context            │
│                                                          │
│  Content (numbers, operators, text):                    │
│    → Process in current context (stack top)             │
│                                                          │
│  This is why stacks are fundamental to parsing!         │
└──────────────────────────────────────────────────────────┘
```

---

## Complexity Analysis

| Application | Time | Space |
|-------------|------|-------|
| Simplify Path | O(n) | O(n) |
| Basic Calculator | O(n) | O(n) |
| HTML Validation | O(n) | O(d) where d = nesting depth |
| Nested List Parsing | O(n) | O(d) |

---

## Summary Table

| Application | Stack Stores | Push On | Pop On |
|-------------|-------------|---------|--------|
| Simplify Path | Directory names | Valid dir name | ".." |
| Basic Calculator | (result, sign) pairs | "(" | ")" |
| HTML Validation | Tag names | Opening tag | Closing tag |
| Nested List | Lists being built | "[" | "]" |

---

## Quick Revision Questions

1. **What does the stack store in path simplification?**
   > Directory names in the current path. ".." pops the most recent directory.

2. **How does the Basic Calculator handle nested parentheses?**
   > Pushes current result and sign onto stack at "(", resets for sub-expression, then combines at ")".

3. **What makes HTML/XML tag matching different from bracket matching?**
   > Tag names must match (not just bracket types), and tags can have attributes. Self-closing tags don't need matching.

4. **What is the common pattern across all syntax parsing problems?**
   > Open delimiter → push context and reset. Close delimiter → pop and combine. Content → process in current context.

5. **Why is the stack ideal for parsing nested structures?**
   > Nesting is inherently LIFO — the innermost structure must be completed first before processing outer levels, matching stack behavior exactly.

---

[← Previous: Browser History](03-browser-history.md) | [Next: Backtracking with Stack →](05-backtracking.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

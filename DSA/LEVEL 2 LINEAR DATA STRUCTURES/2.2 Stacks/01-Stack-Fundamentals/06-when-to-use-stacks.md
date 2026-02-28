# When to Use Stacks

## Overview

Knowing **when** to use a stack is as important as knowing **how** to use one. This chapter provides decision-making frameworks, pattern recognition strategies, and practical guidelines to help you identify stack-appropriate problems.

---

## The Stack Decision Framework

```
Problem Analysis Framework

                Start
                  ↓
        ┌─────────────────┐
        │ Analyze Problem │
        └─────────────────┘
                  ↓
        ┌─────────────────────────────┐
        │ Does it involve...          │
        │ • Reversing?                │
        │ • Nested structures?        │
        │ • Backtracking?             │
        │ • Most recent first?        │
        │ • Matching pairs?           │
        └─────────────────────────────┘
                  ↓
        ┌──────Yes──────┐   No
        ↓               ↓
    Use Stack     Consider:
                  • Queue (FIFO)
                  • Array (Random Access)
                  • Tree (Hierarchy)
                  • Graph (Relations)
```

---

## Key Indicators for Stack Usage

### 1. LIFO Pattern Recognition

```
✅ LIFO Required:

Pattern: "Process most recent item first"

Examples:
┌────────────────────────────────────────┐
│ Undo Last Action                       │
│ ┌─────┐                                │
│ │ A5  │ ← Most recent, undo first      │
│ ├─────┤                                │
│ │ A4  │                                │
│ ├─────┤                                │
│ │ A3  │                                │
│ └─────┘                                │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│ Return from Nested Function Call       │
│ ┌──────┐                               │
│ │ F3() │ ← Most recent call, return first│
│ ├──────┤                               │
│ │ F2() │                               │
│ ├──────┤                               │
│ │ F1() │                               │
│ └──────┘                               │
└────────────────────────────────────────┘

Stack is Natural Solution!
```

### 2. Nested Structure Handling

```
✅ Nested Structures:

Pattern: "Inner-most processed before outer-most"

Example 1: Parentheses
{[()]}
 └─┘   Inner processed first
└────┘ Outer processed last

Example 2: HTML Tags
<div>
  <p>
    <span>text</span>  ← Closes first
  </p>                 ← Closes second
</div>                 ← Closes last

Example 3: Function Calls
main() {
  foo() {
    bar() {
      baz()  ← Returns first
    }        ← Returns second
  }          ← Returns third
}            ← Returns last

Stack Maintains Nesting Level!
```

### 3. Reversal Requirements

```
✅ Reversal Needed:

Pattern: "Output in reverse order of input"

Input:  A → B → C → D → E
Output: E → D → C → B → A

Stack Process:
Push: A, B, C, D, E
┌───┐
│ E │ ← Top
├───┤
│ D │
├───┤
│ C │
├───┤
│ B │
├───┤
│ A │
└───┘
Pop: E, D, C, B, A (Reversed!)

Applications:
• String reversal
• Array reversal
• Reverse Polish notation
• Postorder traversal
```

### 4. Backtracking Problems

```
✅ Backtracking:

Pattern: "Try path, if fail, go back and try alternative"

Maze Solving:
┌───┬───┬───┬───┐
│ S │ → │ ↓ │   │
├───┼───┼───┼───┤
│   │ X │ → │ ↓ │
├───┼───┼───┼───┤
│   │ X │   │ ↓ │
├───┼───┼───┼───┤
│   │   │ ← │ E │
└───┴───┴───┴───┘

Decision Stack:
┌────────┐
│ Right  │ ← Current choice
├────────┤
│ Down   │ ← Can backtrack here
├────────┤
│ Right  │
└────────┘

If blocked:
1. Pop last decision
2. Try alternative
3. If no alternative, pop again

Stack Enables Systematic Exploration!
```

### 5. Matching Pairs

```
✅ Matching Pairs:

Pattern: "Match items in reverse order of occurrence"

Parentheses: ( { [ ] } )
Opening: ( { [
Closing: ] } )
Each closing must match most recent opening!

DNA Pairing: ATCG
A matches T
C matches G
Stack helps validate complementary sequences

Stack Strategy:
• Push opening element
• On closing element, pop and verify match
• Empty stack at end = all matched
```

---

## Problem Patterns That Suggest Stacks

### Pattern 1: "Next Greater/Smaller Element"

```
Problem: For each element, find next greater element

Array: [4, 5, 2, 10, 8]

Brute Force: O(n²)
For each element, scan right until greater found

Stack Solution: O(n)
┌────────────────────────────────────┐
│ Use Monotonic Decreasing Stack    │
│                                    │
│ Process: 4                         │
│ Stack: [4]                         │
│                                    │
│ Process: 5 (greater than 4)        │
│ Pop 4, answer[4] = 5               │
│ Stack: [5]                         │
│                                    │
│ Process: 2                         │
│ Stack: [5, 2]                      │
│                                    │
│ Process: 10 (greater than 2, 5)    │
│ Pop 2, answer[2] = 10              │
│ Pop 5, answer[5] = 10              │
│ Stack: [10]                        │
│                                    │
│ Process: 8                         │
│ Stack: [10, 8]                     │
└────────────────────────────────────┘

Result: [5, 10, 10, -1, -1]

Signal: "Next Greater/Smaller" → Use Stack!
```

### Pattern 2: "Balanced/Valid Sequences"

```
Problem: Check if parentheses are balanced

Examples:
"(())"     → Valid
"(()("     → Invalid
"({[]})"   → Valid
"({[}])"   → Invalid

Stack Approach:
┌────────────────────────────┐
│ For each character:        │
│   If opening → Push        │
│   If closing → Pop & Match │
│   If mismatch → Invalid    │
│ End: Stack empty → Valid   │
└────────────────────────────┘

Signal: "Balanced/Matching" → Use Stack!
```

### Pattern 3: "Recent History Matters"

```
Problem: Implement undo functionality

Operations: Type, Delete, Format
Need to reverse most recent operation first

Stack Naturally Fits:
┌──────────────┐
│ Format Bold  │ ← Undo this first
├──────────────┤
│ Delete 3     │
├──────────────┤
│ Type "Hi"    │
└──────────────┘

Signal: "Recent History" → Use Stack!
```

### Pattern 4: "Depth-First Exploration"

```
Problem: Traverse graph depth-first

Graph:
    A
   / \
  B   C
 / \
D   E

Want: A → B → D → E → C (deep first)

Stack Implementation:
┌───────────────────┐
│ Start: Push A     │
│ Visit A: Push C,B │
│ Visit B: Push E,D │
│ Visit D: Leaf     │
│ Visit E: Leaf     │
│ Visit C: Leaf     │
└───────────────────┘

Signal: "Depth-First" → Use Stack!
```

### Pattern 5: "Expression Evaluation"

```
Problem: Evaluate arithmetic expression

Expression: 3 + 4 * 2

Issues:
• Operator precedence
• Parentheses
• Left-to-right processing

Stack Solution:
Two stacks: Operands + Operators
Handle precedence naturally

Signal: "Expression/Formula" → Use Stack!
```

---

## Decision Tree

```
                 Problem Type?
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Ordering      Structure    Processing
        ↓             ↓             ↓
    ┌───────┐   ┌─────────┐   ┌─────────┐
    │ LIFO? │   │Nested?  │   │Recent   │
    │  ↓    │   │Pairs?   │   │First?   │
    │ YES   │   │  ↓      │   │  ↓      │
    │  ↓    │   │ YES     │   │ YES     │
    │STACK! │   │  ↓      │   │  ↓      │
    └───────┘   │STACK!   │   │STACK!   │
                └─────────┘   └─────────┘

    ┌───────┐   ┌─────────┐   ┌─────────┐
    │ FIFO? │   │Random   │   │Priority?│
    │  ↓    │   │Access?  │   │  ↓      │
    │ YES   │   │  ↓      │   │ YES     │
    │  ↓    │   │ YES     │   │  ↓      │
    │QUEUE! │   │  ↓      │   │PRI.QUEUE│
    └───────┘   │ARRAY!   │   └─────────┘
                └─────────┘
```

---

## Common Problem Keywords

### Strong Stack Indicators

```
🎯 High Probability Stack Problems:

Keywords/Phrases:
✓ "Balanced parentheses"
✓ "Valid brackets"
✓ "Next greater/smaller"
✓ "Reverse"
✓ "Undo/Redo"
✓ "Recently"
✓ "Most recent"
✓ "Nested"
✓ "Matching pairs"
✓ "Backtrack"
✓ "Function calls"
✓ "Depth-first"
✓ "Evaluate expression"
✓ "Infix/Postfix/Prefix"
✓ "Stock span"
✓ "Histogram"
✓ "Trapping water"
```

### Weak/Uncertain Stack Indicators

```
⚠️ May or May Not Be Stack:

Keywords/Phrases:
? "Sort" (usually not, but "sort a stack" exists)
? "Search" (depends on search type)
? "First occurrence" (usually not)
? "Shortest path" (usually BFS/queue)
? "Level order" (BFS/queue)
? "FIFO" (definitely not - use queue)
? "Priority" (use priority queue)
```

---

## Comparison: Stack vs. Alternatives

### Stack vs. Queue

```
Use Stack When:                Use Queue When:
✓ LIFO order needed           ✓ FIFO order needed
✓ Recent first                ✓ Fair processing
✓ Depth-first search          ✓ Breadth-first search
✓ Undo operations             ✓ Task scheduling
✓ Recursion simulation        ✓ Buffer management
✓ Backtracking                ✓ Order preservation

Example:
Function calls → Stack
Print jobs     → Queue
```

### Stack vs. Array

```
Use Stack When:                Use Array When:
✓ Only top access needed      ✓ Random access needed
✓ LIFO pattern                ✓ Index-based access
✓ Dynamic size                ✓ Fixed size known
✓ Frequent push/pop           ✓ Frequent indexed reads
✓ Temporary storage           ✓ Permanent storage

Example:
Expression evaluation → Stack
Student grades       → Array
```

### Stack vs. Recursion

```
Stack Explicitly:             Recursion (Implicit Stack):
✓ Iterative approach          ✓ Cleaner code
✓ Control over stack          ✓ Less code
✓ No stack overflow risk      ✓ Natural for some problems
✓ Can optimize                ✗ Stack overflow risk

Example:
Tree traversal → Either works
DFS            → Either works
Factorial      → Recursion cleaner
```

---

## Real-World Scenario Analysis

### Scenario 1: Calculator App

```
Requirement: Build a calculator

Analysis:
• Need to evaluate expressions
• Handle operator precedence
• Support parentheses

Decision: Use Stack!
Reason:
✓ Expression evaluation is classic stack problem
✓ Precedence handling natural with stack
✓ Parentheses matching requires stack

Implementation:
• Operator stack
• Operand stack
• Shunting-yard algorithm
```

### Scenario 2: Text Editor Undo

```
Requirement: Implement undo/redo

Analysis:
• Need to reverse recent actions
• LIFO order for undo
• Need to track action history

Decision: Use Two Stacks!
Reason:
✓ Undo = pop from action stack
✓ Redo = pop from undo stack, push to action
✓ Most recent action undone first

Implementation:
• Action stack (all operations)
• Redo stack (undone operations)
```

### Scenario 3: Task Scheduler

```
Requirement: Schedule tasks fairly

Analysis:
• First submitted should run first
• FIFO order
• Fair resource allocation

Decision: Use Queue, NOT Stack!
Reason:
✗ Stack would process newest first (unfair)
✓ Queue processes oldest first (fair)

Implementation:
• Task queue (FIFO)
• Not related to stacks
```

### Scenario 4: Web Crawler

```
Requirement: Crawl website depth-first

Analysis:
• Explore one branch fully before others
• Need to remember where to return
• Depth-first exploration

Decision: Use Stack!
Reason:
✓ Depth-first naturally uses stack
✓ Recently discovered links explored first
✓ Backtrack when no more links

Implementation:
• URL stack
• Process top URL
• Add discovered URLs to stack
```

---

## Quick Decision Checklist

```
┌─────────────────────────────────────┐
│ Stack Usage Checklist               │
├─────────────────────────────────────┤
│ □ Need LIFO order?                  │
│ □ Processing nested structures?     │
│ □ Matching pairs/brackets?          │
│ □ Reversing something?              │
│ □ Implementing undo/redo?           │
│ □ Backtracking needed?              │
│ □ Depth-first traversal?            │
│ □ Expression evaluation?            │
│ □ Most recent data prioritized?     │
│ □ Function call simulation?         │
├─────────────────────────────────────┤
│ If 2+ checked → Strong Stack Signal │
│ If 1 checked  → Consider Stack      │
│ If 0 checked  → Probably Not Stack  │
└─────────────────────────────────────┘
```

---

## Anti-Patterns (When NOT to Use Stack)

```
❌ Don't Use Stack For:

1. FIFO Processing
   Problem: Process customers in arrival order
   Solution: Use Queue

2. Random Access
   Problem: Access 5th element efficiently
   Solution: Use Array/List

3. Priority-Based
   Problem: Process highest priority first
   Solution: Use Priority Queue

4. Sorted Data
   Problem: Maintain sorted order
   Solution: Use BST or Heap

5. Key-Value Lookup
   Problem: Fast lookup by key
   Solution: Use Hash Map

6. Bidirectional Access
   Problem: Add/remove from both ends
   Solution: Use Deque

7. Minimum/Maximum Tracking
   Problem: Always get min/max efficiently
   Solution: Use Heap (unless Min Stack problem)
```

---

## Summary Table

| Indicator | Use Stack? | Alternative |
|-----------|------------|-------------|
| **LIFO order** | ✅ Yes | - |
| **FIFO order** | ❌ No | Queue |
| **Nested structures** | ✅ Yes | - |
| **Random access** | ❌ No | Array |
| **Reversal** | ✅ Yes | - |
| **Priority processing** | ❌ No | Priority Queue |
| **Matching pairs** | ✅ Yes | - |
| **Fair ordering** | ❌ No | Queue |
| **Backtracking** | ✅ Yes | Recursion |
| **Sorted maintenance** | ❌ No | BST/Heap |
| **Recent first** | ✅ Yes | - |
| **Key lookup** | ❌ No | Hash Map |
| **Depth-first** | ✅ Yes | Recursion |
| **Breadth-first** | ❌ No | Queue |

---

## Quick Revision Questions

1. **Q: What is the primary indicator that a problem requires a stack?**
   - A: LIFO (Last In, First Out) ordering is required, or processing must prioritize the most recent item.

2. **Q: How can you identify if a problem involves nested structures?**
   - A: Look for matching pairs (parentheses, brackets), hierarchical relationships, or inner-outer processing requirements.

3. **Q: Should you use a stack for a task scheduling system where fairness matters?**
   - A: No! Use a Queue (FIFO) to ensure first come, first served processing.

4. **Q: What keywords in a problem statement strongly suggest using a stack?**
   - A: "Balanced," "valid parentheses," "next greater," "reverse," "undo," "nested," "backtrack," "depth-first."

5. **Q: When is recursion preferable to an explicit stack?**
   - A: When the problem naturally fits recursive structure and stack depth won't be too large (to avoid stack overflow).

6. **Q: If a problem asks to "find the first element that...", should you use a stack?**
   - A: Not necessarily - "first" often suggests FIFO (queue) or linear search, not LIFO (stack). Context matters.

---

## Navigation

- **[← Previous: Stack Applications Overview](05-applications-overview.md)**
- **[Next: Unit 2 - Array-Based Stack →](../02-Stack-Implementation/01-array-based-stack.md)**
- **[↑ Back to Unit 1](README.md)**
- **[⌂ Home](../README.md)**

---

*Master pattern recognition and you'll know instantly when a stack is the right tool for the job!*

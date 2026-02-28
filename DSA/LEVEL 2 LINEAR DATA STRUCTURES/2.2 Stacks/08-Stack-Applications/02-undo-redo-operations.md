# Chapter 2: Undo-Redo Operations

[← Previous: Function Call Stack](01-function-call-stack.md) | [Next: Browser History →](03-browser-history.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

---

## Overview

**Undo/Redo** is one of the most ubiquitous stack applications. Every text editor, graphics program, and spreadsheet uses two stacks to implement reversible operations. This is a perfect demonstration of how LIFO ordering naturally supports "going back in time."

---

## The Two-Stack Model

```
┌──────────────────────────────────────────────────────────┐
│              UNDO / REDO ARCHITECTURE                    │
│                                                          │
│   UNDO Stack            REDO Stack                       │
│   (past actions)        (undone actions)                 │
│                                                          │
│   ┌──────────┐          ┌──────────┐                    │
│   │ Action 3 │ ← last   │          │ (empty initially)  │
│   │ Action 2 │          │          │                    │
│   │ Action 1 │          │          │                    │
│   └──────────┘          └──────────┘                    │
│                                                          │
│   DO action:   Push to UNDO, clear REDO                 │
│   UNDO:        Pop from UNDO, push to REDO              │
│   REDO:        Pop from REDO, push to UNDO              │
└──────────────────────────────────────────────────────────┘
```

---

## Algorithm

```
CLASS UndoRedoManager:
    undoStack ← empty stack    // Past actions
    redoStack ← empty stack    // Undone actions
    
    FUNCTION doAction(action):
        // Perform action and record it
        execute(action)
        undoStack.push(action)
        redoStack.clear()    // New action invalidates redo history!
    
    FUNCTION undo():
        IF undoStack is empty: RETURN    // Nothing to undo
        action ← undoStack.pop()
        reverseExecute(action)           // Reverse the action
        redoStack.push(action)
    
    FUNCTION redo():
        IF redoStack is empty: RETURN    // Nothing to redo
        action ← redoStack.pop()
        execute(action)                  // Re-apply the action
        undoStack.push(action)
```

---

## Trace: Text Editor

```
Document: ""

═══ Type "Hello" ═══
  execute: insert "Hello"
  undoStack: ["insert Hello"]    redoStack: []
  Document: "Hello"

═══ Type " World" ═══
  execute: insert " World"
  undoStack: ["insert Hello", "insert World"]    redo: []
  Document: "Hello World"

═══ Bold "World" ═══
  execute: bold(6,11)
  undoStack: ["insert Hello", "insert World", "bold(6,11)"]
  Document: "Hello **World**"

═══ UNDO ═══
  Pop "bold(6,11)" from undo → reverse: unbold
  undoStack: ["insert Hello", "insert World"]
  redoStack: ["bold(6,11)"]
  Document: "Hello World"

═══ UNDO ═══
  Pop "insert World" → reverse: delete " World"
  undoStack: ["insert Hello"]
  redoStack: ["bold(6,11)", "insert World"]
  Document: "Hello"

═══ REDO ═══
  Pop "insert World" from redo → re-execute
  undoStack: ["insert Hello", "insert World"]
  redoStack: ["bold(6,11)"]
  Document: "Hello World"

═══ Type "!" ═══   (NEW action after undo/redo!)
  execute: insert "!"
  undoStack: ["insert Hello", "insert World", "insert !"]
  redoStack: []    ← CLEARED! "bold(6,11)" is gone forever!
  Document: "Hello World!"
```

---

## Why Clear Redo on New Action?

```
Timeline:
  A → B → C          (original actions)
           ↑ UNDO to here
  A → B               (after undo C)
  
  Now if user does NEW action D:
  A → B → D           (new timeline)
  
  C is no longer relevant! History branched.
  Clearing redo prevents confusion about which
  timeline we're in.

  Some advanced editors keep ALL branches (tree undo),
  but the standard model discards the old branch.
```

---

## Implementation: Simple Integer Editor

```
CLASS NumberEditor:
    value ← 0
    undoStack ← empty stack
    redoStack ← empty stack
    
    FUNCTION add(n):
        undoStack.push(("add", n))
        redoStack.clear()
        value ← value + n
    
    FUNCTION multiply(n):
        undoStack.push(("multiply", n))
        redoStack.clear()
        value ← value × n
    
    FUNCTION undo():
        IF undoStack is empty: RETURN
        (op, n) ← undoStack.pop()
        redoStack.push((op, n))
        IF op == "add":
            value ← value - n
        ELSE IF op == "multiply":
            value ← value / n
    
    FUNCTION redo():
        IF redoStack is empty: RETURN
        (op, n) ← redoStack.pop()
        undoStack.push((op, n))
        IF op == "add":
            value ← value + n
        ELSE IF op == "multiply":
            value ← value × n
```

### Trace:

```
add(5):      value=5   undo:[add,5]      redo:[]
multiply(3): value=15  undo:[add5,mul3]  redo:[]
add(2):      value=17  undo:[a5,m3,a2]   redo:[]
undo():      value=15  undo:[a5,m3]       redo:[a2]
undo():      value=5   undo:[a5]          redo:[a2,m3]
redo():      value=15  undo:[a5,m3]       redo:[a2]
add(10):     value=25  undo:[a5,m3,a10]   redo:[]  ← cleared!
```

---

## Command Pattern (Design Pattern)

```
The undo/redo system often uses the COMMAND PATTERN:

INTERFACE Command:
    execute()
    unexecute()

CLASS InsertCommand implements Command:
    position, text
    execute():   document.insert(position, text)
    unexecute(): document.delete(position, length(text))

CLASS DeleteCommand implements Command:
    position, text  (save deleted text for undo!)
    execute():   document.delete(position, length(text))
    unexecute(): document.insert(position, text)

Each command knows how to DO and UNDO itself.
The undo/redo manager just pushes/pops these objects.
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| `doAction` | O(1) | O(1) per action |
| `undo` | O(1) | O(1) |
| `redo` | O(1) | O(1) |
| `clear redo` | O(k) | — (k = redo stack size) |
| **Total space** | — | O(n) for n actions |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Structure** | Two stacks: undo (history) + redo (future) |
| **Do** | Push to undo, clear redo |
| **Undo** | Pop from undo, push to redo |
| **Redo** | Pop from redo, push to undo |
| **Why clear redo** | New action creates new timeline branch |
| **Pattern** | Command Pattern (DO / UNDO interface) |

---

## Quick Revision Questions

1. **Why are two stacks needed for undo/redo?**
   > One stack tracks past actions (undo stack), the other tracks undone actions that can be redone (redo stack).

2. **Why is the redo stack cleared on a new action?**
   > A new action creates a new timeline; the previously undone actions are no longer valid to redo.

3. **What information must each action store?**
   > Enough information to both execute AND reverse the action (e.g., inserted text and position, deleted text for restoration).

4. **What design pattern is commonly used for undo/redo?**
   > The Command Pattern — each action is an object with `execute()` and `unexecute()` methods.

5. **What is the time complexity of undo and redo?**
   > O(1) to pop/push the action. The actual reversal depends on the action itself.

---

[← Previous: Function Call Stack](01-function-call-stack.md) | [Next: Browser History →](03-browser-history.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

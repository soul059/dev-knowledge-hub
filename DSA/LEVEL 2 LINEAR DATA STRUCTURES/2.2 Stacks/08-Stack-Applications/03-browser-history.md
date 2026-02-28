# Chapter 3: Browser History

[← Previous: Undo-Redo Operations](02-undo-redo-operations.md) | [Next: Syntax Parsing →](04-syntax-parsing.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

---

## Overview

**Browser back/forward** navigation (LeetCode #1472) is a classic two-stack application. Visiting a new page is like "do," clicking Back is "undo," and Forward is "redo." This mirrors the undo/redo pattern applied to URL navigation.

---

## Problem Statement

```
Design a browser history system:
  visit(url)    — Visit a new URL
  back(steps)   — Go back at most `steps` pages
  forward(steps)— Go forward at most `steps` pages

Example:
  visit("google.com")
  visit("facebook.com")
  visit("youtube.com")
  back(1)        → "facebook.com"
  back(1)        → "google.com"
  forward(1)     → "facebook.com"
  visit("linkedin.com")   ← clears forward history!
  forward(2)     → "linkedin.com" (can't go forward)
  back(2)        → "google.com"
  back(7)        → "google.com" (can't go back further)
```

---

## Two-Stack Solution

```
CLASS BrowserHistory:
    backStack    ← empty stack    // Past pages
    forwardStack ← empty stack    // Pages we went back from
    currentPage  ← homepage
    
    FUNCTION __init__(homepage):
        currentPage ← homepage
    
    FUNCTION visit(url):
        backStack.push(currentPage)    // Save current page
        currentPage ← url
        forwardStack.clear()           // New visit clears forward!
    
    FUNCTION back(steps):
        WHILE steps > 0 AND backStack NOT empty:
            forwardStack.push(currentPage)
            currentPage ← backStack.pop()
            steps ← steps - 1
        RETURN currentPage
    
    FUNCTION forward(steps):
        WHILE steps > 0 AND forwardStack NOT empty:
            backStack.push(currentPage)
            currentPage ← forwardStack.pop()
            steps ← steps - 1
        RETURN currentPage
```

---

## Detailed Trace

```
BrowserHistory("homepage.com")
  current: "homepage.com"
  back: []    forward: []

═══ visit("google.com") ═══
  back: ["homepage.com"]    forward: [] (cleared)
  current: "google.com"

═══ visit("facebook.com") ═══
  back: ["homepage.com", "google.com"]    forward: []
  current: "facebook.com"

═══ visit("youtube.com") ═══
  back: ["homepage.com", "google.com", "facebook.com"]
  forward: []
  current: "youtube.com"

  Visual:
  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ homepage │→│  google  │→│ facebook │→│ youtube  │
  └──────────┘  └──────────┘  └──────────┘  └──────────┘
                                              ↑ current

═══ back(1) ═══
  Move youtube → forward    Move facebook → current
  back: ["homepage.com", "google.com"]
  forward: ["youtube.com"]
  current: "facebook.com"
  Returns: "facebook.com" ✓

═══ back(1) ═══
  Move facebook → forward    Move google → current
  back: ["homepage.com"]
  forward: ["youtube.com", "facebook.com"]
  current: "google.com"
  Returns: "google.com" ✓

═══ forward(1) ═══
  Move google → back    Move facebook → current
  back: ["homepage.com", "google.com"]
  forward: ["youtube.com"]
  current: "facebook.com"
  Returns: "facebook.com" ✓

═══ visit("linkedin.com") ═══
  back: ["homepage.com", "google.com", "facebook.com"]
  forward: [] ← CLEARED! youtube.com is gone!
  current: "linkedin.com"

═══ forward(2) ═══
  forward stack empty → can't go forward
  Returns: "linkedin.com" ✓

═══ back(2) ═══
  Step 1: linkedin→forward, facebook→current
  Step 2: facebook→forward, google→current
  back: ["homepage.com"]
  forward: ["linkedin.com", "facebook.com"]
  current: "google.com"
  Returns: "google.com" ✓

═══ back(7) ═══
  Only 1 page in back stack
  Step 1: google→forward, homepage→current
  Steps 2-7: back empty, stop
  Returns: "homepage.com" ✓
```

---

## Alternative: Array/List Approach

```
CLASS BrowserHistory_Array:
    history ← [homepage]
    current ← 0        // Index pointing to current page
    last    ← 0        // Last valid index
    
    FUNCTION visit(url):
        current ← current + 1
        IF current == length(history):
            history.append(url)
        ELSE:
            history[current] ← url
        last ← current    // Invalidate forward entries
    
    FUNCTION back(steps):
        current ← MAX(0, current - steps)
        RETURN history[current]
    
    FUNCTION forward(steps):
        current ← MIN(last, current + steps)
        RETURN history[current]
```

---

## Comparison

```
┌──────────────────┬────────────────┬────────────────┐
│ Operation        │ Two Stacks     │ Array          │
├──────────────────┼────────────────┼────────────────┤
│ visit            │ O(1) + clear   │ O(1)           │
│ back(k)          │ O(k)           │ O(1)           │
│ forward(k)       │ O(k)           │ O(1)           │
│ Space            │ O(n)           │ O(n)           │
│ Simplicity       │ Simple         │ Slightly more  │
└──────────────────┴────────────────┴────────────────┘

Array approach is O(1) for back/forward but uses index math.
Stack approach is more intuitive but O(k) per operation.
```

---

## Real Browser Behavior

```
┌──────────────────────────────────────────────────────────┐
│  Real browsers are more complex:                         │
│                                                          │
│  • Session history per tab (separate stacks per tab)     │
│  • Hash navigation (#section) within same page           │
│  • pushState/replaceState (History API)                   │
│  • Maximum history limit (browsers cap at ~50 entries)   │
│  • Forward history preserved on page refresh             │
│  • Anchor scrolling vs page navigation                   │
│                                                          │
│  But the fundamental model is still TWO STACKS!          │
└──────────────────────────────────────────────────────────┘
```

---

## Complexity Analysis

| Operation | Two-Stack | Array |
|-----------|-----------|-------|
| `visit` | O(1) amortized | O(1) |
| `back(k)` | O(min(k, size)) | O(1) |
| `forward(k)` | O(min(k, size)) | O(1) |
| **Space** | O(n) | O(n) |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Model** | Two stacks (back + forward) + current page |
| **Visit** | Push current to back, set new current, clear forward |
| **Back** | Push current to forward, pop from back |
| **Forward** | Push current to back, pop from forward |
| **Clear forward** | New visit invalidates forward history |
| **LeetCode** | #1472 |

---

## Quick Revision Questions

1. **Why does visiting a new page clear the forward stack?**
   > It creates a new browsing path; the old forward pages are no longer reachable in the new timeline.

2. **What happens when back(steps) exceeds available history?**
   > We go back as far as possible (to the first page) and stop.

3. **How does browser history relate to undo/redo?**
   > Visit=do, Back=undo, Forward=redo. Same two-stack architecture.

4. **What advantage does the array approach have over two stacks?**
   > O(1) back/forward operations using index arithmetic instead of O(k) stack pops.

5. **In a real browser, what is the scope of the back/forward stacks?**
   > Per-tab. Each tab maintains its own independent navigation history.

---

[← Previous: Undo-Redo Operations](02-undo-redo-operations.md) | [Next: Syntax Parsing →](04-syntax-parsing.md) | [↑ Back to Unit 8](../README.md#unit-8-stack-applications) | [🏠 Home](../README.md)

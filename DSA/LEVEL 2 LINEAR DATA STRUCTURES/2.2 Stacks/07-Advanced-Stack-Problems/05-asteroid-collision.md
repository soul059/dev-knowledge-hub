# Chapter 5: Asteroid Collision

[← Previous: Celebrity Problem](04-celebrity-problem.md) | [Next: Decode String →](06-decode-string.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

---

## Overview

The **Asteroid Collision** problem (LeetCode #735) simulates asteroids moving in a line. Positive values move right, negative move left. When two collide, the smaller one explodes. If equal, both explode. A stack naturally models this because collisions happen between the most recent right-moving and the incoming left-moving asteroid.

---

## Problem Statement

```
Given: asteroids = [5, 10, -5]

Rules:
  - Positive (+) = moving RIGHT →
  - Negative (-) = moving LEFT  ←
  - Collision happens when: right-moving meets left-moving
  - Larger asteroid survives, smaller explodes
  - Equal size: both explode
  - Same direction: no collision

Output: [5, 10]

5 → and 10 → both move right, no collision.
10 → meets -5 ←: |10| > |-5|, so -5 explodes.
Result: [5, 10]
```

---

## When Do Collisions Happen?

```
┌──────────────────────────────────────────────────────────┐
│  Collision: stack.top() > 0 AND incoming < 0             │
│  (Right-moving on stack meets left-moving incoming)      │
│                                                          │
│  No collision cases:                                     │
│  → →  (both positive): no collision                      │
│  ← ←  (both negative): no collision                      │
│  ← →  (moving apart): no collision                       │
│  → ←  (moving toward each other): COLLISION!             │
│                                                          │
│  Stack stores: survivors so far                          │
│  Incoming left-moving asteroid collides with stack top   │
└──────────────────────────────────────────────────────────┘
```

---

## Algorithm

```
FUNCTION asteroidCollision(asteroids):
    stack ← empty stack
    
    FOR each asteroid in asteroids:
        alive ← true
        
        // Collision: right-moving on stack vs left-moving incoming
        WHILE alive AND stack NOT empty AND asteroid < 0 AND stack.top() > 0:
            IF stack.top() < ABS(asteroid):
                stack.pop()        // Stack top explodes, check next
            ELSE IF stack.top() == ABS(asteroid):
                stack.pop()        // Both explode
                alive ← false
            ELSE:
                alive ← false      // Incoming explodes
        
        IF alive:
            stack.push(asteroid)
    
    RETURN stack contents (bottom to top)
```

---

## Detailed Trace

### Input: [5, 10, -5]

```
asteroid=5:  stack empty, push 5.         Stack: [5]
asteroid=10: 10>0, no collision. Push 10.  Stack: [5, 10]
asteroid=-5: -5<0, top=10>0 → COLLISION!
             |10| > |-5| → -5 explodes.   alive=false
             Stack: [5, 10]

Result: [5, 10] ✓
```

### Input: [8, -8]

```
asteroid=8:  push.                    Stack: [8]
asteroid=-8: -8<0, top=8>0 → COLLISION!
             |8| == |-8| → BOTH explode!
             alive=false
             Stack: []

Result: [] ✓
```

### Input: [10, 2, -5]

```
asteroid=10: push.                    Stack: [10]
asteroid=2:  push (both positive).   Stack: [10, 2]
asteroid=-5: -5<0, top=2>0 → COLLISION!
             |2| < |-5| → 2 explodes.
             Stack: [10]
             top=10>0 → COLLISION!
             |10| > |-5| → -5 explodes.
             alive=false
             Stack: [10]

Result: [10] ✓
```

### Input: [-2, -1, 1, 2]

```
asteroid=-2: push (no right-mover on stack).  Stack: [-2]
asteroid=-1: push (-1<0, top=-2<0 → no collision). Stack: [-2, -1]
asteroid=1:  push (1>0, no collision with negatives). Stack: [-2, -1, 1]
asteroid=2:  push (2>0). Stack: [-2, -1, 1, 2]

Result: [-2, -1, 1, 2] ✓ (← ← → →, no collisions)
```

### Input: [-2, 2, -1, -2]

```
asteroid=-2: push.                Stack: [-2]
asteroid=2:  push (← →, no collision). Stack: [-2, 2]
asteroid=-1: -1<0, top=2>0 → COLLISION!
             |2| > |-1| → -1 explodes.
             Stack: [-2, 2]
asteroid=-2: -2<0, top=2>0 → COLLISION!
             |2| == |-2| → BOTH explode!
             Stack: [-2]
             top=-2<0 → no more collisions.

Result: [-2] ✓
```

---

## Visual Simulation

```
Input: [5, 10, -15, 3, -4]

Time 0:  5→  10→     ←15   3→  ←4
         ▓▓  ▓▓▓▓    ▓▓▓▓  ▓▓  ▓▓

Time 1:  5→  10→←15   3←4
         ▓▓  COLLISION  COLLISION

         |10| < |-15| → 10 explodes
         |3| < |-4| → 3 explodes

Time 2:  5→  ←15     ←4
         COLLISION

         |5| < |-15| → 5 explodes

Result:  ←15  ←4 → [-15, -4]
```

### Stack Trace for [5, 10, -15, 3, -4]:

```
Push 5:    Stack: [5]
Push 10:   Stack: [5, 10]
Process -15:
  |-15|>|10| → pop 10      Stack: [5]
  |-15|>|5|  → pop 5       Stack: []
  Push -15                  Stack: [-15]
Push 3:    Stack: [-15, 3]   (top=-15<0 → no collision with left-moving 3? No, 3>0 and -15<0 → ← → → no collision ✓)
Process -4:
  -4<0, top=3>0 → COLLISION
  |-4|>|3| → pop 3         Stack: [-15]
  top=-15<0 → no collision
  Push -4                   Stack: [-15, -4]

Result: [-15, -4] ✓
```

---

## Edge Cases

```
1. All positive: [1, 2, 3] → [1, 2, 3] (no collisions)
2. All negative: [-3, -2, -1] → [-3, -2, -1] (no collisions)
3. All equal opposite: [1, -1] → [] (mutual destruction)
4. Large vs many small: [100, -1, -1, -1] → [100]
5. Small vs large: [1, -100] → [-100]
```

---

## Complexity Analysis

| Aspect | Complexity |
|--------|-----------|
| **Time** | O(n) — each asteroid pushed/popped at most once |
| **Space** | O(n) — stack may hold all asteroids |

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **Problem** | Simulate asteroid collisions |
| **Collision Condition** | `stack.top() > 0 AND incoming < 0` |
| **Larger wins** | Smaller explodes, larger stays |
| **Equal** | Both explode |
| **Time** | O(n) |
| **LeetCode** | #735 |

---

## Quick Revision Questions

1. **When do two asteroids collide?**
   > When a right-moving asteroid (positive, on stack) meets a left-moving asteroid (negative, incoming).

2. **Why don't same-direction asteroids collide?**
   > They're moving in the same direction and never meet (both → or both ←).

3. **Why don't ← → asteroids collide?**
   > They're moving apart from each other.

4. **What happens in `[1, -1, 1, -1]`?**
   > First 1 and -1 collide (both explode). Then second 1 and -1 collide (both explode). Result: [].

5. **What is the key stack invariant in this problem?**
   > After processing each asteroid, no collision-possible pair exists on the stack. Left-movers are always below right-movers in their respective sections.

---

[← Previous: Celebrity Problem](04-celebrity-problem.md) | [Next: Decode String →](06-decode-string.md) | [↑ Back to Unit 7](../README.md#unit-7-advanced-stack-problems) | [🏠 Home](../README.md)

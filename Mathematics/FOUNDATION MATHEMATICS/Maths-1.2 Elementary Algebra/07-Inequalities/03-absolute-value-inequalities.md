# Chapter 7.3: Absolute Value Inequalities

[← Previous: Compound Inequalities](02-compound-inequalities.md) | [Back to Contents](../README.md) | [Next: Graphing Inequalities →](04-graphing-inequalities.md)

---

## 📚 Chapter Overview

Absolute value inequalities extend our understanding of both absolute values and inequalities. They arise naturally when dealing with tolerances, error margins, and distances. This chapter teaches you to solve and interpret these important inequalities.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Recall the meaning of absolute value as distance
- Solve "less than" absolute value inequalities (|x| < a)
- Solve "greater than" absolute value inequalities (|x| > a)
- Handle multi-step absolute value inequalities
- Identify special cases (no solution, all real numbers)
- Apply absolute value inequalities to real problems

---

## 1. Review of Absolute Value

### Definition

```
┌─────────────────────────────────────────────────────────────────────┐
│              ABSOLUTE VALUE                                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   |x| = distance of x from 0 on the number line                   │
│                                                                     │
│   Formal definition:                                               │
│           ┌  x    if x ≥ 0                                        │
│   |x| =   │                                                        │
│           └ -x    if x < 0                                        │
│                                                                     │
│   Examples:                                                         │
│   |5| = 5       (5 is 5 units from 0)                             │
│   |-5| = 5      (-5 is also 5 units from 0)                       │
│   |0| = 0                                                          │
│                                                                     │
│   Key property: |x| is always non-negative (|x| ≥ 0)             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Absolute Value as Distance

```
|x - a| = distance from x to a

Examples:
|x - 3| = distance from x to 3
|x + 2| = |x - (-2)| = distance from x to -2

On the number line:
       |x - 3| = 5 means x is 5 units from 3
       
       ←── 5 ──→|←── 5 ──→
───┼────┼────┼────┼────┼────┼────┼───→
  -2    0    2    3    4    6    8
   ↑                        ↑
  x=-2                     x=8
```

---

## 2. Solving |x| < a (Less Than)

### The Rule

```
┌─────────────────────────────────────────────────────────────────────┐
│              "LESS THAN" ABSOLUTE VALUE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For a > 0:                                                       │
│                                                                     │
│   |x| < a   is equivalent to   -a < x < a                         │
│                                                                     │
│   |x| ≤ a   is equivalent to   -a ≤ x ≤ a                         │
│                                                                     │
│   Think: "x is LESS THAN a units from 0"                          │
│   → x is BETWEEN -a and a                                         │
│                                                                     │
│                 ○═══════════════○                                  │
│   ─────┼───────┼───────────────┼───────┼─────→                    │
│       -a       0               a                                   │
│                                                                     │
│   This is an "AND" compound inequality!                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples

**Example 1: Basic**
```
Solve: |x| < 4

Rewrite as compound inequality:
-4 < x < 4

Solution: (-4, 4)

Graph:
        ○═══════════════○
───┼────┼────┼────┼────┼────┼───→
  -5   -4   -2    0    2    4
```

**Example 2: With linear expression**
```
Solve: |x - 3| ≤ 5

Rewrite:
-5 ≤ x - 3 ≤ 5

Add 3 to all parts:
-5 + 3 ≤ x ≤ 5 + 3
-2 ≤ x ≤ 8

Solution: [-2, 8]

Interpretation: x is within 5 units of 3
```

---

## 3. Solving |x| > a (Greater Than)

### The Rule

```
┌─────────────────────────────────────────────────────────────────────┐
│              "GREATER THAN" ABSOLUTE VALUE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For a > 0:                                                       │
│                                                                     │
│   |x| > a   is equivalent to   x < -a  OR  x > a                  │
│                                                                     │
│   |x| ≥ a   is equivalent to   x ≤ -a  OR  x ≥ a                  │
│                                                                     │
│   Think: "x is MORE THAN a units from 0"                          │
│   → x is OUTSIDE the interval [-a, a]                             │
│                                                                     │
│   ════════○               ○════════                                │
│   ─────┼───────┼───────────────┼───────┼─────→                    │
│       -a       0               a                                   │
│                                                                     │
│   This is an "OR" compound inequality!                            │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Examples

**Example 1: Basic**
```
Solve: |x| > 3

Rewrite as compound inequality:
x < -3  OR  x > 3

Solution: (-∞, -3) ∪ (3, ∞)

Graph:
═══════○               ○═══════════
───┼────┼────┼────┼────┼────┼───→
  -5   -3   -1    0    1    3
```

**Example 2: With linear expression**
```
Solve: |2x + 1| ≥ 7

Rewrite:
2x + 1 ≤ -7  OR  2x + 1 ≥ 7

Solve first:          Solve second:
2x + 1 ≤ -7           2x + 1 ≥ 7
2x ≤ -8               2x ≥ 6
x ≤ -4                x ≥ 3

Solution: x ≤ -4 OR x ≥ 3
         (-∞, -4] ∪ [3, ∞)
```

---

## 4. Memory Aid: The V-Rule

```
┌─────────────────────────────────────────────────────────────────────┐
│              THE V-RULE MEMORY AID                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Graph of y = |x|:                                                │
│                                                                     │
│            y                                                        │
│            │      ╱                                                │
│            │    ╱                                                  │
│       a ───│──●───────●                                           │
│            │╱    ╲                                                 │
│            ●──────────────→ x                                     │
│           -a  0  a                                                 │
│                                                                     │
│   |x| < a: Where is the V below the line y = a?                   │
│            Answer: Between -a and a (the "valley")                │
│                                                                     │
│   |x| > a: Where is the V above the line y = a?                   │
│            Answer: Left of -a OR right of a (the "wings")         │
│                                                                     │
│   "Less than" → INSIDE (AND)                                      │
│   "Greater than" → OUTSIDE (OR)                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Multi-Step Absolute Value Inequalities

### Isolate the Absolute Value First

```
Solve: 3|2x - 4| + 5 < 20

Step 1: Isolate the absolute value
3|2x - 4| < 15
|2x - 4| < 5

Step 2: Apply the "less than" rule
-5 < 2x - 4 < 5

Step 3: Solve the compound inequality
Add 4:
-1 < 2x < 9

Divide by 2:
-1/2 < x < 9/2

Solution: (-1/2, 9/2) or (-0.5, 4.5)
```

### Another Example

```
Solve: 2|x + 3| - 1 ≥ 9

Step 1: Isolate the absolute value
2|x + 3| ≥ 10
|x + 3| ≥ 5

Step 2: Apply the "greater than" rule
x + 3 ≤ -5  OR  x + 3 ≥ 5

Step 3: Solve each inequality
x ≤ -8  OR  x ≥ 2

Solution: (-∞, -8] ∪ [2, ∞)
```

---

## 6. Special Cases

### Case 1: |x| < 0 (Negative on right side)

```
Solve: |x - 2| < -3

Since |x - 2| is always ≥ 0, it can NEVER be less than -3.

Solution: No solution (∅)

The absolute value cannot be negative!
```

### Case 2: |x| > 0 

```
Solve: |x + 5| > 0

|x + 5| = 0 only when x + 5 = 0, i.e., x = -5
For all other x, |x + 5| > 0

Solution: All real numbers except -5
          x ≠ -5
          (-∞, -5) ∪ (-5, ∞)
```

### Case 3: |x| ≥ 0 (Always True)

```
Solve: |3x - 1| ≥ 0

Absolute value is ALWAYS ≥ 0!

Solution: All real numbers, (-∞, ∞)
```

### Case 4: |x| < a where a ≤ 0

```
Solve: |x| ≤ -2

No number's absolute value can be negative or zero (except |0|=0).
|x| is never ≤ -2 (a negative number).

Solution: No solution (∅)
```

### Case 5: |x| > a where a < 0

```
Solve: |x| > -4

Since |x| ≥ 0 for all x, and 0 > -4, every x works!

Solution: All real numbers, (-∞, ∞)
```

---

## 7. Applications

### Tolerance/Error
```
A machine produces bolts that should be 10 mm in diameter.
The tolerance is ±0.05 mm (acceptable error).
What diameters are acceptable?

|d - 10| ≤ 0.05

-0.05 ≤ d - 10 ≤ 0.05
9.95 ≤ d ≤ 10.05

Acceptable diameter: 9.95 mm to 10.05 mm
```

### Temperature Range
```
A medication must be stored at a temperature that differs 
from 40°F by less than 5 degrees. What temperatures are safe?

|T - 40| < 5

-5 < T - 40 < 5
35 < T < 45

Safe temperature range: 35°F to 45°F
```

---

## ✏️ Solved Examples

### Example 1: Easy - Less than

**Problem:** Solve: |x| ≤ 6

**Solution:**
```
-6 ≤ x ≤ 6

Solution: [-6, 6]
```

### Example 2: Easy - Greater than

**Problem:** Solve: |x| > 2

**Solution:**
```
x < -2 OR x > 2

Solution: (-∞, -2) ∪ (2, ∞)
```

### Example 3: Medium - Linear expression

**Problem:** Solve: |3x - 6| < 12

**Solution:**
```
-12 < 3x - 6 < 12

Add 6:
-6 < 3x < 18

Divide by 3:
-2 < x < 6

Solution: (-2, 6)
```

### Example 4: Medium - Greater than with expression

**Problem:** Solve: |4 - x| ≥ 3

**Solution:**
```
4 - x ≤ -3  OR  4 - x ≥ 3

First:          Second:
-x ≤ -7         -x ≥ -1
x ≥ 7           x ≤ 1

Solution: x ≤ 1 OR x ≥ 7
         (-∞, 1] ∪ [7, ∞)
```

### Example 5: Hard - Multi-step

**Problem:** Solve: 5 - 2|3x + 1| > -7

**Solution:**
```
Step 1: Isolate absolute value
-2|3x + 1| > -12
|3x + 1| < 6      (divide by -2, flip sign!)

Step 2: Apply rule
-6 < 3x + 1 < 6

Step 3: Solve
Subtract 1:
-7 < 3x < 5

Divide by 3:
-7/3 < x < 5/3

Solution: (-7/3, 5/3) or approximately (-2.33, 1.67)
```

### Example 6: Hard - Special case

**Problem:** Solve: |2x + 5| + 3 < 1

**Solution:**
```
Isolate:
|2x + 5| < -2

Absolute value cannot be negative!

Solution: No solution (∅)
```

---

## ❓ Practice Problems

### Easy Level

1. Solve: |x| < 5

2. Solve: |x| ≥ 4

3. Solve: |x - 2| ≤ 3

### Medium Level

4. Solve: |2x + 3| < 7

5. Solve: |5 - x| > 2

6. Solve: |3x - 9| ≥ 6

### Hard Level

7. Solve: 4|2x - 1| - 3 ≤ 13

8. Solve: |x - 4| + |x - 4| < 6

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. -5 < x < 5 → **(-5, 5)**

2. x ≤ -4 OR x ≥ 4 → **(-∞, -4] ∪ [4, ∞)**

3. -3 ≤ x - 2 ≤ 3 → -1 ≤ x ≤ 5 → **[-1, 5]**

4. -7 < 2x + 3 < 7 → -10 < 2x < 4 → **-5 < x < 2** or **(-5, 2)**

5. 5 - x < -2 OR 5 - x > 2
   x > 7 OR x < 3 → **(-∞, 3) ∪ (7, ∞)**

6. 3x - 9 ≤ -6 OR 3x - 9 ≥ 6
   3x ≤ 3 OR 3x ≥ 15
   x ≤ 1 OR x ≥ 5 → **(-∞, 1] ∪ [5, ∞)**

7. 4|2x - 1| ≤ 16 → |2x - 1| ≤ 4
   -4 ≤ 2x - 1 ≤ 4
   -3 ≤ 2x ≤ 5
   **-3/2 ≤ x ≤ 5/2** or **[-1.5, 2.5]**

8. 2|x - 4| < 6 → |x - 4| < 3
   -3 < x - 4 < 3
   **1 < x < 7** or **(1, 7)**

</details>

---

## 📋 Summary Table

| Inequality Type | Equivalent Form | Solution Type |
|-----------------|-----------------|---------------|
| \|x\| < a (a > 0) | -a < x < a | AND (interval) |
| \|x\| ≤ a (a > 0) | -a ≤ x ≤ a | AND (closed interval) |
| \|x\| > a (a ≥ 0) | x < -a OR x > a | OR (two rays) |
| \|x\| ≥ a (a ≥ 0) | x ≤ -a OR x ≥ a | OR (two rays) |
| \|x\| < a (a ≤ 0) | (impossible) | No solution |
| \|x\| > a (a < 0) | (always true) | All real numbers |

---

## 🔄 Quick Revision Questions

1. **|x| < 3 means x is between ___ and ___.**

2. **|x| > 5 means x < ___ OR x > ___.**

3. **What is the first step when solving 2|x+1| - 4 > 6?**

4. **Can |x - 3| = -2 have a solution? Why?**

5. **|x - 5| < 2 means x is within ___ units of ___.**

6. **Write |x - 7| ≤ 4 as a compound inequality.**

<details>
<summary>Quick Answers</summary>

1. -3 and 3
2. -5, 5
3. Isolate the absolute value: |x+1| > 5
4. No, absolute value is always ≥ 0
5. 2 units of 5
6. -4 ≤ x - 7 ≤ 4, which gives 3 ≤ x ≤ 11

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ |x| < a (less than):                                          │
│     → x is CLOSE to 0 (within a units)                            │
│     → Compound AND: -a < x < a                                    │
│     → INSIDE the interval                                         │
│                                                                     │
│   ★ |x| > a (greater than):                                       │
│     → x is FAR from 0 (more than a units)                         │
│     → Compound OR: x < -a OR x > a                                │
│     → OUTSIDE the interval                                        │
│                                                                     │
│   ★ Always isolate the absolute value first!                      │
│                                                                     │
│   ★ Special cases to watch:                                       │
│     • |x| < (negative) → No solution                              │
│     • |x| > (negative) → All real numbers                         │
│                                                                     │
│   ★ |x - c| < d means x is within d units of c                    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Compound Inequalities](02-compound-inequalities.md) | [Back to Contents](../README.md) | [Next: Graphing Inequalities →](04-graphing-inequalities.md)

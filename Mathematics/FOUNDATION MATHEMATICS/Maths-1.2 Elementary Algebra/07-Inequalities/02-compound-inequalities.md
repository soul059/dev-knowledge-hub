# Chapter 7.2: Compound Inequalities

[← Previous: Linear Inequalities](01-linear-inequalities.md) | [Back to Contents](../README.md) | [Next: Absolute Value Inequalities →](03-absolute-value-inequalities.md)

---

## 📚 Chapter Overview

Compound inequalities combine two or more inequalities using "and" or "or." These are essential for describing ranges of values and modeling real-world constraints that have both upper and lower bounds.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Distinguish between "and" and "or" compound inequalities
- Solve and graph "and" (conjunction) inequalities
- Solve and graph "or" (disjunction) inequalities
- Express solutions in interval notation
- Solve three-part (double) inequalities
- Apply compound inequalities to real-world problems

---

## 1. Types of Compound Inequalities

### "And" vs "Or"

```
┌─────────────────────────────────────────────────────────────────────┐
│              COMPOUND INEQUALITY TYPES                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   "AND" (Conjunction)                                              │
│   ─────────────────────                                            │
│   • Both conditions must be TRUE                                   │
│   • Written: x > 2 AND x < 7  or  2 < x < 7                       │
│   • Solution: INTERSECTION of the two sets                        │
│   • Graph: Values in BOTH regions (overlap)                       │
│                                                                     │
│   "OR" (Disjunction)                                               │
│   ─────────────────────                                            │
│   • At least ONE condition must be TRUE                           │
│   • Written: x < 2 OR x > 7                                       │
│   • Solution: UNION of the two sets                               │
│   • Graph: Values in EITHER region                                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│              "AND" vs "OR" ON NUMBER LINE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   x > 2 AND x < 7  (same as 2 < x < 7)                            │
│                                                                     │
│        ○═══════════════════○                                       │
│   ─────┼─────┼─────┼─────┼─────┼─────┼─────→                      │
│        0     2     3     5     7     9                             │
│             ↑                 ↑                                    │
│             must be here     (both conditions)                     │
│                                                                     │
│   Interval: (2, 7)                                                 │
│                                                                     │
│   ───────────────────────────────────────────                      │
│                                                                     │
│   x < 2 OR x > 7                                                   │
│                                                                     │
│   ═════════○           ○═════════════                              │
│   ─────┼─────┼─────┼─────┼─────┼─────┼─────→                      │
│        0     2     3     5     7     9                             │
│        ↑                       ↑                                   │
│   here OR                      here                                │
│                                                                     │
│   Interval: (-∞, 2) ∪ (7, ∞)                                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Solving "And" Compound Inequalities

### Method 1: Solve Separately

```
Solve: x + 3 > 1 AND x - 2 < 5

Solve first inequality:        Solve second inequality:
x + 3 > 1                      x - 2 < 5
x > -2                         x < 7

Combine with AND:
x > -2 AND x < 7
-2 < x < 7

Solution: (-2, 7)

Graph:
        ○═══════════════════════○
───┼────┼────┼────┼────┼────┼────┼────┼───→
  -3   -2   -1    0    2    4    6    7
```

### Method 2: Three-Part Inequality

When the variable appears in the middle of a "double inequality":

```
Solve: -3 ≤ 2x + 1 < 9

Step 1: Work on all three parts simultaneously
        (subtract 1 from all parts)
        -3 - 1 ≤ 2x + 1 - 1 < 9 - 1
        -4 ≤ 2x < 8

Step 2: Divide all parts by 2
        -4/2 ≤ 2x/2 < 8/2
        -2 ≤ x < 4

Solution: [-2, 4)

Graph:
        ●═══════════════════════○
───┼────┼────┼────┼────┼────┼────┼────┼───→
  -3   -2   -1    0    1    2    3    4
```

### Key Rule for Three-Part Inequalities

```
┌─────────────────────────────────────────────────────────────────────┐
│              THREE-PART INEQUALITY RULE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Whatever operation you perform, do it to ALL THREE parts!       │
│                                                                     │
│   If you multiply or divide by a negative, flip BOTH signs:       │
│                                                                     │
│   Example: -6 < -2x ≤ 4                                           │
│   Divide by -2:                                                    │
│   -6/(-2) > x ≥ 4/(-2)                                            │
│   3 > x ≥ -2                                                       │
│   Rewrite: -2 ≤ x < 3                                             │
│                                                                     │
│   (It's often cleaner to rewrite with smaller number on left)     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Solving "Or" Compound Inequalities

### Method: Solve Each Separately, Then Unite

```
Solve: 3x + 2 < -4 OR 2x - 1 > 5

Solve first inequality:        Solve second inequality:
3x + 2 < -4                    2x - 1 > 5
3x < -6                        2x > 6
x < -2                         x > 3

Combine with OR:
x < -2 OR x > 3

Solution: (-∞, -2) ∪ (3, ∞)

Graph:
═══════════○                    ○═══════════
───┼────┼────┼────┼────┼────┼────┼────┼───→
  -4   -3   -2   -1    0    1    3    4
```

---

## 4. Special Cases

### "And" with No Overlap (No Solution)

```
Solve: x < 2 AND x > 5

Graph attempt:
x < 2:    ═══════○
x > 5:                    ○═══════
          ─────┼─────┼─────┼─────┼─────→
               2     3     5

No overlap! A number cannot be less than 2 AND greater than 5.

Solution: No solution (∅ or empty set)
```

### "Or" with Complete Overlap (All Real Numbers)

```
Solve: x ≤ 5 OR x ≥ 2

Graph:
x ≤ 5:    ═══════════════●
x ≥ 2:            ●═══════════════
          ─────┼─────┼─────┼─────┼─────→
               1     2     5     7

Every number satisfies at least one condition!

Solution: All real numbers, (-∞, ∞), or ℝ
```

---

## 5. Interval Notation for Compound Inequalities

### Summary Table

```
┌─────────────────────────────────────────────────────────────────────┐
│              INTERVAL NOTATION                                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Compound Inequality          Interval Notation                   │
│   ─────────────────────────────────────────────────────────────    │
│   a < x < b (AND)              (a, b)                              │
│   a ≤ x ≤ b (AND)              [a, b]                              │
│   a ≤ x < b (AND)              [a, b)                              │
│   a < x ≤ b (AND)              (a, b]                              │
│                                                                     │
│   x < a OR x > b               (-∞, a) ∪ (b, ∞)                   │
│   x ≤ a OR x ≥ b               (-∞, a] ∪ [b, ∞)                   │
│                                                                     │
│   Note: ∪ means "union" (combines both sets)                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Applications

### Temperature Range
```
A reaction requires temperature between 50°C and 80°C, inclusive.
Write this as a compound inequality.

Let T = temperature in °C

50 ≤ T ≤ 80

Interval: [50, 80]
```

### Speed Limits
```
The speed limit on a road is "at least 45 mph but at most 65 mph."
What speeds are legal?

Let s = speed in mph

45 ≤ s ≤ 65

Interval: [45, 65]
```

### Manufacturing Tolerance
```
A machine part must be between 4.95 cm and 5.05 cm 
(but not exactly 5 cm). What lengths are acceptable?

Let L = length in cm

(4.95 < L < 5) OR (5 < L < 5.05)
That is: L ≠ 5 AND 4.95 < L < 5.05

Interval: (4.95, 5) ∪ (5, 5.05)
```

---

## ✏️ Solved Examples

### Example 1: Easy - AND inequality

**Problem:** Solve: x + 5 > 2 AND x - 3 ≤ 4

**Solution:**
```
First inequality:           Second inequality:
x + 5 > 2                   x - 3 ≤ 4
x > -3                      x ≤ 7

Combined: x > -3 AND x ≤ 7
         -3 < x ≤ 7

Solution: (-3, 7]

Graph:
        ○═══════════════════════●
───┼────┼────┼────┼────┼────┼────┼───→
  -4   -3    0    2    4    6    7
```

### Example 2: Easy - OR inequality

**Problem:** Solve: x - 2 ≤ -5 OR x + 1 > 4

**Solution:**
```
First inequality:           Second inequality:
x - 2 ≤ -5                  x + 1 > 4
x ≤ -3                      x > 3

Combined: x ≤ -3 OR x > 3

Solution: (-∞, -3] ∪ (3, ∞)

Graph:
═══════●                         ○═══════
───┼────┼────┼────┼────┼────┼────┼───→
  -5   -3   -1    0    1    3    5
```

### Example 3: Medium - Three-part inequality

**Problem:** Solve: -7 ≤ 3x - 1 < 8

**Solution:**
```
Add 1 to all parts:
-7 + 1 ≤ 3x - 1 + 1 < 8 + 1
-6 ≤ 3x < 9

Divide all parts by 3:
-6/3 ≤ x < 9/3
-2 ≤ x < 3

Solution: [-2, 3)

Graph:
        ●═══════════════════○
───┼────┼────┼────┼────┼────┼───→
  -3   -2   -1    0    2    3
```

### Example 4: Medium - Negative coefficient in three-part

**Problem:** Solve: 4 < -2x + 6 ≤ 12

**Solution:**
```
Subtract 6 from all parts:
4 - 6 < -2x + 6 - 6 ≤ 12 - 6
-2 < -2x ≤ 6

Divide by -2 (FLIP BOTH inequality signs):
-2/(-2) > x ≥ 6/(-2)
1 > x ≥ -3

Rewrite with smaller number on left:
-3 ≤ x < 1

Solution: [-3, 1)

Graph:
        ●═══════════════════○
───┼────┼────┼────┼────┼────┼───→
  -4   -3   -2    0    1    2
```

### Example 5: Hard - Complex OR

**Problem:** Solve: 2(x - 1) < -4 OR 3(x + 2) ≥ 15

**Solution:**
```
First inequality:           Second inequality:
2(x - 1) < -4               3(x + 2) ≥ 15
2x - 2 < -4                 3x + 6 ≥ 15
2x < -2                     3x ≥ 9
x < -1                      x ≥ 3

Combined: x < -1 OR x ≥ 3

Solution: (-∞, -1) ∪ [3, ∞)

Graph:
════════○                   ●════════════
───┼────┼────┼────┼────┼────┼────┼───→
  -3   -1    0    1    2    3    5
```

### Example 6: Hard - Application

**Problem:** The sum of three consecutive integers is between 24 and 39 (exclusive). Find all possible values for the first integer.

**Solution:**
```
Let n = first integer
Consecutive integers: n, n+1, n+2

Sum = n + (n+1) + (n+2) = 3n + 3

Condition: 24 < sum < 39
24 < 3n + 3 < 39

Subtract 3:
21 < 3n < 36

Divide by 3:
7 < n < 12

Since n must be an integer: n ∈ {8, 9, 10, 11}

Check n = 8: 8 + 9 + 10 = 27 ✓ (24 < 27 < 39)
Check n = 11: 11 + 12 + 13 = 36 ✓ (24 < 36 < 39)

Solution: The first integer is 8, 9, 10, or 11
```

---

## ❓ Practice Problems

### Easy Level

1. Solve and graph: x - 2 > 1 AND x + 3 < 10

2. Solve and graph: 2x < -6 OR x - 1 ≥ 3

3. Solve: -5 < x + 2 ≤ 6

### Medium Level

4. Solve: -8 ≤ 2x - 4 < 10

5. Solve: 5 - 3x > -1 AND 2x - 7 < 3

6. Solve: 3x - 2 ≤ -8 OR 4 - x < -1

### Hard Level

7. Solve: 1 < (5 - 2x)/3 ≤ 4

8. A student must score at least 70 but less than 90 to get a B. If the student has scores of 65, 72, 88, and 80, what range of scores on the 5th test gives a B average?

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. x > 3 AND x < 7 → **3 < x < 7** or **(3, 7)**

2. x < -3 OR x ≥ 4 → **(-∞, -3) ∪ [4, ∞)**

3. Subtract 2: -7 < x ≤ 4 → **(-7, 4]**

4. Add 4: -4 ≤ 2x < 14; divide by 2: **-2 ≤ x < 7** or **[-2, 7)**

5. First: 5 - 3x > -1 → -3x > -6 → x < 2
   Second: 2x < 10 → x < 5
   Combined: x < 2 AND x < 5 → **x < 2** or **(-∞, 2)**

6. First: 3x ≤ -6 → x ≤ -2
   Second: -x < -5 → x > 5
   Combined: **x ≤ -2 OR x > 5** or **(-∞, -2] ∪ (5, ∞)**

7. Multiply by 3: 3 < 5 - 2x ≤ 12
   Subtract 5: -2 < -2x ≤ 7
   Divide by -2 (flip): 1 > x ≥ -7/2
   Rewrite: **-7/2 ≤ x < 1** or **[-3.5, 1)**

8. Current sum: 65 + 72 + 88 + 80 = 305
   Let x = 5th score. Average = (305 + x)/5
   For B: 70 ≤ (305 + x)/5 < 90
   350 ≤ 305 + x < 450
   **45 ≤ x < 145**
   (Realistically, x ≤ 100, so **45 ≤ x ≤ 100**)

</details>

---

## 📋 Summary Table

| Type | Keyword | Solution Type | Interval Symbol |
|------|---------|---------------|-----------------|
| Conjunction | AND | Intersection (overlap) | Single interval |
| Disjunction | OR | Union (either) | Uses ∪ |
| Three-part | (none) | "AND" implied | Single interval |

---

## 🔄 Quick Revision Questions

1. **For x > 3 AND x < 7, what numbers are in the solution?**

2. **Is x = 5 in the solution of x < 2 OR x > 4?**

3. **What does the symbol ∪ mean?**

4. **If -4 < 2x < 10, what is x?**

5. **Can x < 5 AND x > 8 have a solution?**

6. **What is the interval notation for x ≤ -1 OR x ≥ 3?**

<details>
<summary>Quick Answers</summary>

1. Numbers between 3 and 7 (not including 3 and 7)
2. Yes! (5 > 4, so the "OR" condition is satisfied)
3. Union (combining both sets)
4. Divide by 2: -2 < x < 5
5. No solution (no number is both less than 5 and greater than 8)
6. (-∞, -1] ∪ [3, ∞)

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ "AND" = Intersection (both must be true)                      │
│     • Graph shows OVERLAP of both conditions                      │
│     • Can have no solution if regions don't overlap               │
│                                                                     │
│   ★ "OR" = Union (at least one must be true)                      │
│     • Graph shows BOTH regions combined                           │
│     • Can be all real numbers if regions cover everything         │
│                                                                     │
│   ★ Three-part inequalities (a < bx + c < d):                     │
│     • Perform operations on all three parts                       │
│     • If dividing by negative, flip BOTH inequality signs         │
│                                                                     │
│   ★ Interval notation:                                             │
│     • AND → single interval                                       │
│     • OR → use ∪ to join intervals                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Linear Inequalities](01-linear-inequalities.md) | [Back to Contents](../README.md) | [Next: Absolute Value Inequalities →](03-absolute-value-inequalities.md)

# Chapter 5.2: Elimination Method

[← Previous: Substitution Method](01-substitution-method.md) | [Back to Contents](../README.md) | [Next: Cross-Multiplication Method →](03-cross-multiplication-method.md)

---

## 📚 Chapter Overview

The **elimination method** (also called the addition method) solves systems of linear equations by adding or subtracting equations to eliminate one variable. This method is especially useful when the coefficients are easily made equal or opposite.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Understand the principle behind the elimination method
- Add or subtract equations to eliminate a variable
- Multiply equations to create opposite coefficients
- Recognize when elimination is more efficient than substitution
- Handle special cases (no solution, infinitely many)
- Apply elimination to real-world problems

---

## 1. The Elimination Principle

### How It Works

If we add or subtract two equations, we can eliminate one variable when its coefficients are equal (for subtraction) or opposite (for addition).

```
┌─────────────────────────────────────────────────────────────────────┐
│              THE ELIMINATION PRINCIPLE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If:   a₁x + b₁y = c₁                                            │
│   and   a₂x + b₂y = c₂                                            │
│                                                                     │
│   Then adding the equations:                                       │
│   (a₁ + a₂)x + (b₁ + b₂)y = c₁ + c₂                              │
│                                                                     │
│   If b₁ = -b₂, then y is eliminated!                              │
│                                                                     │
│   Example:                                                         │
│      x + y = 5                                                     │
│    + x - y = 1                                                     │
│    ─────────────                                                   │
│      2x    = 6                                                     │
│          x = 3                                                     │
│                                                                     │
│   The y terms cancel: (+y) + (-y) = 0                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Basic Elimination: Opposite Coefficients

### When Coefficients Are Already Opposite

```
┌─────────────────────────────────────────────────────────────────────┐
│              OPPOSITE COEFFICIENTS - ADD                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Solve: { 2x + 3y = 13                                            │
│          { 2x - 3y = -1                                            │
│                                                                     │
│   Notice: 3y and -3y are opposite coefficients                    │
│                                                                     │
│   ADD the equations:                                               │
│      2x + 3y = 13                                                  │
│    + 2x - 3y = -1                                                  │
│    ───────────────                                                 │
│      4x      = 12                                                  │
│           x  = 3                                                   │
│                                                                     │
│   Substitute x = 3 into equation 1:                               │
│   2(3) + 3y = 13                                                   │
│   6 + 3y = 13                                                      │
│   3y = 7                                                           │
│   y = 7/3                                                          │
│                                                                     │
│   Solution: (3, 7/3)                                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### When Coefficients Are Equal

```
┌─────────────────────────────────────────────────────────────────────┐
│              EQUAL COEFFICIENTS - SUBTRACT                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Solve: { 3x + 2y = 11                                            │
│          { x + 2y = 5                                              │
│                                                                     │
│   Notice: Both have 2y (equal coefficients)                       │
│                                                                     │
│   SUBTRACT the equations:                                          │
│      3x + 2y = 11                                                  │
│    - (x + 2y = 5)                                                  │
│    ───────────────                                                 │
│      2x      = 6                                                   │
│           x  = 3                                                   │
│                                                                     │
│   Substitute x = 3 into equation 2:                               │
│   3 + 2y = 5                                                       │
│   2y = 2                                                           │
│   y = 1                                                            │
│                                                                     │
│   Solution: (3, 1)                                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Creating Opposite Coefficients

### Multiplying to Create Opposites

When coefficients aren't naturally opposite or equal, multiply one or both equations by constants.

```
┌─────────────────────────────────────────────────────────────────────┐
│              MULTIPLYING EQUATIONS                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Solve: { 2x + 3y = 7    ... (1)                                 │
│          { 5x + 2y = 12   ... (2)                                 │
│                                                                     │
│   To eliminate y, we need equal or opposite y-coefficients:       │
│   • Multiply (1) by 2: 4x + 6y = 14                              │
│   • Multiply (2) by -3: -15x - 6y = -36                          │
│                                                                     │
│   Add:                                                             │
│      4x + 6y = 14                                                  │
│   -15x - 6y = -36                                                  │
│   ─────────────────                                                │
│     -11x     = -22                                                 │
│           x  = 2                                                   │
│                                                                     │
│   Substitute x = 2 into (1):                                      │
│   2(2) + 3y = 7                                                    │
│   4 + 3y = 7                                                       │
│   y = 1                                                            │
│                                                                     │
│   Solution: (2, 1)                                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Choosing Multipliers

```
┌─────────────────────────────────────────────────────────────────────┐
│              CHOOSING MULTIPLIERS                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   For: { 2x + 3y = 7                                               │
│        { 5x + 2y = 12                                              │
│                                                                     │
│   To eliminate x:                                                  │
│   • LCM of 2 and 5 is 10                                          │
│   • Multiply first by 5: 10x + 15y = 35                          │
│   • Multiply second by -2: -10x - 4y = -24                       │
│                                                                     │
│   To eliminate y:                                                  │
│   • LCM of 3 and 2 is 6                                           │
│   • Multiply first by 2: 4x + 6y = 14                            │
│   • Multiply second by -3: -15x - 6y = -36                       │
│                                                                     │
│   Either approach works! Choose the one with smaller numbers.    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Step-by-Step Process

### Complete Algorithm

```
┌─────────────────────────────────────────────────────────────────────┐
│              ELIMINATION METHOD STEPS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: Write both equations in standard form (ax + by = c)     │
│                                                                     │
│   Step 2: Choose which variable to eliminate                       │
│           (Pick the one with simpler coefficients)                │
│                                                                     │
│   Step 3: Multiply one or both equations to create opposite       │
│           coefficients for the chosen variable                    │
│                                                                     │
│   Step 4: Add the equations to eliminate that variable            │
│                                                                     │
│   Step 5: Solve the resulting equation for the remaining variable │
│                                                                     │
│   Step 6: Substitute back to find the eliminated variable         │
│                                                                     │
│   Step 7: Check the solution in both original equations           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Decision Flowchart

```
          START
            │
            ▼
   Are coefficients of one
   variable already opposite?
            │
     ┌──────┴──────┐
    YES           NO
     │             │
     ▼             ▼
   ADD the     Are coefficients of one
  equations    variable already equal?
     │             │
     │      ┌──────┴──────┐
     │     YES           NO
     │      │             │
     │      ▼             ▼
     │   SUBTRACT    MULTIPLY to create
     │   equations   opposite coefficients
     │      │             │
     │      │             ▼
     │      │        ADD equations
     │      │             │
     └──────┴──────┬──────┘
                   │
                   ▼
           Solve for remaining
               variable
                   │
                   ▼
             Back-substitute
                   │
                   ▼
                 CHECK
```

---

## 5. Special Cases

### No Solution

```
Solve: { 2x + 4y = 8     ... (1)
       { x + 2y = 5      ... (2)

Multiply (2) by -2:
-2x - 4y = -10

Add to (1):
  2x + 4y =  8
 -2x - 4y = -10
─────────────────
      0   = -2

This is FALSE! No value of x or y works.

The lines are parallel. No solution.
```

### Infinitely Many Solutions

```
Solve: { 3x - 6y = 12    ... (1)
       { x - 2y = 4      ... (2)

Multiply (2) by -3:
-3x + 6y = -12

Add to (1):
  3x - 6y =  12
 -3x + 6y = -12
─────────────────
      0   =  0

This is ALWAYS TRUE!

The equations are the same line. Infinitely many solutions.
```

---

## 6. Elimination vs. Substitution

### When to Use Each Method

| Situation | Best Method |
|-----------|-------------|
| One variable already isolated | Substitution |
| Coefficient of 1 or -1 | Substitution |
| Opposite coefficients exist | Elimination (add) |
| Equal coefficients exist | Elimination (subtract) |
| Neither coefficient is 1 | Elimination |
| Large numbers, would create fractions | Elimination |

---

## ✏️ Solved Examples

### Example 1: Easy - Opposite Coefficients

**Problem:** Solve:
$\begin{cases} x + y = 8 \\ x - y = 2 \end{cases}$

**Solution:**
```
The y coefficients are opposite (+1 and -1)

ADD the equations:
  x + y = 8
+ x - y = 2
──────────
  2x    = 10
      x = 5

Substitute into equation 1:
5 + y = 8
y = 3

Check:
Eq 1: 5 + 3 = 8 ✓
Eq 2: 5 - 3 = 2 ✓

Answer: (5, 3)
```

### Example 2: Easy - Equal Coefficients

**Problem:** Solve:
$\begin{cases} 3x + 2y = 14 \\ x + 2y = 6 \end{cases}$

**Solution:**
```
Both equations have 2y

SUBTRACT equation 2 from equation 1:
  3x + 2y = 14
- (x + 2y = 6)
──────────────
  2x      = 8
       x  = 4

Substitute into equation 2:
4 + 2y = 6
2y = 2
y = 1

Answer: (4, 1)
```

### Example 3: Medium - Multiply One Equation

**Problem:** Solve:
$\begin{cases} 2x + y = 8 \\ 3x + 2y = 14 \end{cases}$

**Solution:**
```
Multiply equation 1 by -2 to eliminate y:
-2(2x + y = 8) → -4x - 2y = -16

Add to equation 2:
  -4x - 2y = -16
+  3x + 2y =  14
───────────────
   -x      = -2
        x  = 2

Substitute into equation 1:
2(2) + y = 8
4 + y = 8
y = 4

Answer: (2, 4)
```

### Example 4: Medium - Multiply Both Equations

**Problem:** Solve:
$\begin{cases} 3x + 4y = 10 \\ 2x + 3y = 7 \end{cases}$

**Solution:**
```
To eliminate x: LCM(3, 2) = 6
• Multiply equation 1 by 2: 6x + 8y = 20
• Multiply equation 2 by -3: -6x - 9y = -21

Add:
  6x + 8y =  20
 -6x - 9y = -21
─────────────────
      -y  = -1
       y  = 1

Substitute into equation 1:
3x + 4(1) = 10
3x = 6
x = 2

Answer: (2, 1)
```

### Example 5: Hard - Fractional Coefficients

**Problem:** Solve:
$\begin{cases} \frac{x}{2} + \frac{y}{3} = 2 \\ \frac{x}{4} + \frac{y}{2} = 1 \end{cases}$

**Solution:**
```
First, clear fractions:
Eq 1 × 6: 3x + 2y = 12
Eq 2 × 4: x + 2y = 4

Now use elimination:
SUBTRACT:
  3x + 2y = 12
- (x + 2y = 4)
──────────────
  2x      = 8
      x   = 4

Substitute:
4 + 2y = 4
2y = 0
y = 0

Check in original equations:
Eq 1: 4/2 + 0/3 = 2 + 0 = 2 ✓
Eq 2: 4/4 + 0/2 = 1 + 0 = 1 ✓

Answer: (4, 0)
```

### Example 6: Hard - Word Problem

**Problem:** Tickets to a concert cost $10 for adults and $6 for children. If 200 tickets were sold for $1,560, how many of each type were sold?

**Solution:**
```
Let a = number of adult tickets
Let c = number of child tickets

Equation 1 (total tickets):
a + c = 200

Equation 2 (total revenue):
10a + 6c = 1560

Multiply equation 1 by -6:
-6a - 6c = -1200

Add to equation 2:
  10a + 6c =  1560
  -6a - 6c = -1200
──────────────────
   4a      = 360
        a  = 90

Substitute:
90 + c = 200
c = 110

Check:
90 + 110 = 200 ✓
10(90) + 6(110) = 900 + 660 = 1560 ✓

Answer: 90 adult tickets and 110 child tickets
```

---

## ❓ Practice Problems

### Easy Level

1. Solve: $\begin{cases} x + y = 10 \\ x - y = 4 \end{cases}$

2. Solve: $\begin{cases} 2x + y = 9 \\ 2x + 3y = 13 \end{cases}$

3. Solve: $\begin{cases} 3x - 2y = 8 \\ 3x + 2y = 16 \end{cases}$

### Medium Level

4. Solve: $\begin{cases} 2x + 3y = 13 \\ 4x - y = 5 \end{cases}$

5. Solve: $\begin{cases} 5x + 3y = 21 \\ 2x + 5y = 16 \end{cases}$

6. Solve: $\begin{cases} 3x + 4y = 25 \\ 2x - 3y = 6 \end{cases}$

### Hard Level

7. Solve: $\begin{cases} 4x - 6y = 10 \\ 2x - 3y = 5 \end{cases}$

8. A store sold 75 items, some at $12 each and some at $8 each, for a total of $780. How many of each were sold?

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. Add: 2x = 14 → x = 7, y = 3
   **$(7, 3)$**

2. Subtract: -2y = -4 → y = 2, x = 7/2
   **$(7/2, 2)$**

3. Add: 6x = 24 → x = 4; Substitute: y = 2
   **$(4, 2)$**

4. Multiply eq 2 by 3: 12x - 3y = 15
   Add to eq 1 × 1: 2x + 3y = 13; gives 14x = 28 → x = 2
   Substitute: 4 + 3y = 13 → y = 3
   **$(2, 3)$**

5. Multiply eq 1 by 5: 25x + 15y = 105
   Multiply eq 2 by -3: -6x - 15y = -48
   Add: 19x = 57 → x = 3
   Substitute: 6 + 5y = 16 → y = 2
   **$(3, 2)$**

6. Multiply eq 1 by 3: 9x + 12y = 75
   Multiply eq 2 by 4: 8x - 12y = 24
   Add: 17x = 99 → x = 99/17
   Substitute to find y = 61/17
   **$(99/17, 61/17)$** or approximately **(5.82, 3.59)**

7. Multiply eq 2 by -2: -4x + 6y = -10
   Add to eq 1: 0 = 0
   **Infinitely many solutions**

8. Let x = $12 items, y = $8 items
   x + y = 75 and 12x + 8y = 780
   Multiply first by -8: -8x - 8y = -600
   Add: 4x = 180 → x = 45, y = 30
   **45 items at $12 and 30 items at $8**

</details>

---

## 📋 Summary Table

| Coefficient Relationship | Action |
|-------------------------|--------|
| Already opposite (like 3y and -3y) | Add equations |
| Already equal (like 2x and 2x) | Subtract equations |
| Neither | Multiply to create opposites, then add |

---

## 🔄 Quick Revision Questions

1. **What do you do when y-coefficients are 4 and -4?**

2. **How do you eliminate x if coefficients are 3x and 5x?**

3. **What does "0 = 0" mean after elimination?**

4. **What does "0 = 5" mean after elimination?**

5. **To eliminate y in 2y and 5y, what multipliers could you use?**

6. **Which method works when coefficients are already equal?**

<details>
<summary>Quick Answers</summary>

1. Add the equations
2. Multiply first by 5 and second by -3 (or first by -5 and second by 3)
3. Infinitely many solutions
4. No solution
5. Multiply first equation by 5 and second by -2
6. Subtraction

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Elimination adds/subtracts equations to eliminate a variable │
│                                                                     │
│   ★ Opposite coefficients → ADD equations                         │
│     Equal coefficients → SUBTRACT equations                       │
│                                                                     │
│   ★ Multiply equations to create opposite coefficients           │
│                                                                     │
│   ★ Special results:                                               │
│     • 0 = 0 → Infinitely many solutions                           │
│     • 0 = k (k ≠ 0) → No solution                                 │
│                                                                     │
│   ★ Elimination is often easier than substitution when:          │
│     • Coefficients are easily made equal/opposite                 │
│     • No coefficient is 1 or -1                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Substitution Method](01-substitution-method.md) | [Back to Contents](../README.md) | [Next: Cross-Multiplication Method →](03-cross-multiplication-method.md)

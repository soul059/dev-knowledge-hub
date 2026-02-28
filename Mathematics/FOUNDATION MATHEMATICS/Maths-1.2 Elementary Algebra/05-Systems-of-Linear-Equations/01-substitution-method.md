# Chapter 5.1: Substitution Method

[← Previous: Graphical Representation](../04-Linear-Equations/04-graphical-representation.md) | [Back to Contents](../README.md) | [Next: Elimination Method →](02-elimination-method.md)

---

## 📚 Chapter Overview

A **system of linear equations** consists of two or more linear equations with the same variables. The **substitution method** solves these systems by expressing one variable in terms of another and substituting into the other equation.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Understand what a system of linear equations is
- Recognize when substitution is the best method
- Solve systems using substitution
- Interpret the three possible outcomes
- Apply substitution to word problems

---

## 1. Systems of Linear Equations

### Definition

A **system of linear equations** is a set of two or more equations with the same variables.

```
┌─────────────────────────────────────────────────────────────────────┐
│              SYSTEM OF LINEAR EQUATIONS                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   A system of two equations in two variables:                     │
│                                                                     │
│   { a₁x + b₁y = c₁                                                │
│   { a₂x + b₂y = c₂                                                │
│                                                                     │
│   Example:                                                         │
│   { x + y = 5                                                      │
│   { 2x - y = 4                                                     │
│                                                                     │
│   A SOLUTION is an ordered pair (x, y) that satisfies             │
│   BOTH equations simultaneously.                                   │
│                                                                     │
│   For the example above: (3, 2)                                   │
│   Check: 3 + 2 = 5 ✓                                              │
│          2(3) - 2 = 4 ✓                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Types of Solutions

```
┌─────────────────────────────────────────────────────────────────────┐
│              THREE TYPES OF SYSTEMS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1. CONSISTENT & INDEPENDENT                                      │
│      • Exactly ONE solution                                        │
│      • Lines intersect at one point                               │
│      • Different slopes                                            │
│                                                                     │
│   2. INCONSISTENT                                                  │
│      • NO solution                                                 │
│      • Lines are parallel                                         │
│      • Same slope, different y-intercepts                         │
│                                                                     │
│   3. CONSISTENT & DEPENDENT                                        │
│      • INFINITELY MANY solutions                                   │
│      • Lines are identical (coincident)                           │
│      • Same slope, same y-intercept                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Substitution Method

### When to Use Substitution

The substitution method works best when:
- One variable is already isolated (e.g., $y = 3x + 2$)
- One variable has a coefficient of 1 or -1
- The equation is easily solved for one variable

### Step-by-Step Process

```
┌─────────────────────────────────────────────────────────────────────┐
│              SUBSTITUTION METHOD STEPS                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Step 1: SOLVE one equation for one variable                     │
│           Choose the equation/variable that's easiest             │
│                                                                     │
│   Step 2: SUBSTITUTE that expression into the other equation     │
│           Replace the variable with the expression               │
│                                                                     │
│   Step 3: SOLVE the resulting equation                            │
│           You now have one equation with one variable            │
│                                                                     │
│   Step 4: BACK-SUBSTITUTE to find the other variable             │
│           Use the value found to get the second variable         │
│                                                                     │
│   Step 5: CHECK the solution in BOTH original equations          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Representation

```
     Equation 1                  Equation 2
         │                           │
         ▼                           │
   Solve for x or y                 │
   (get x = ... or y = ...)        │
         │                           │
         └──────┬────────────────────┘
                │
                ▼
        Substitute into Eq 2
                │
                ▼
        Solve for remaining variable
                │
                ▼
        Back-substitute
                │
                ▼
           Solution (x, y)
```

---

## 3. Solving Systems by Substitution

### Basic Example

```
┌─────────────────────────────────────────────────────────────────────┐
│              EXAMPLE: BASIC SUBSTITUTION                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Solve: { y = 2x + 1      ... (1)                                │
│          { 3x + y = 11     ... (2)                                │
│                                                                     │
│   Step 1: Equation (1) already gives y in terms of x              │
│           y = 2x + 1                                               │
│                                                                     │
│   Step 2: Substitute into equation (2)                            │
│           3x + (2x + 1) = 11                                       │
│                                                                     │
│   Step 3: Solve for x                                              │
│           5x + 1 = 11                                              │
│           5x = 10                                                  │
│           x = 2                                                    │
│                                                                     │
│   Step 4: Back-substitute to find y                               │
│           y = 2(2) + 1 = 5                                        │
│                                                                     │
│   Step 5: Check in both equations                                 │
│           (1): 5 = 2(2) + 1 = 5 ✓                                 │
│           (2): 3(2) + 5 = 11 ✓                                    │
│                                                                     │
│   Solution: (2, 5)                                                 │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### When You Need to Solve First

```
Solve: { x + 2y = 7    ... (1)
       { 3x - y = 11   ... (2)

Step 1: Solve equation (1) for x (coefficient is 1)
        x = 7 - 2y

Step 2: Substitute into equation (2)
        3(7 - 2y) - y = 11

Step 3: Solve for y
        21 - 6y - y = 11
        21 - 7y = 11
        -7y = -10
        y = 10/7

Step 4: Back-substitute
        x = 7 - 2(10/7)
        x = 7 - 20/7
        x = 49/7 - 20/7
        x = 29/7

Solution: (29/7, 10/7)
```

---

## 4. Special Cases

### No Solution (Inconsistent System)

```
┌─────────────────────────────────────────────────────────────────────┐
│              NO SOLUTION                                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Solve: { y = 2x + 3                                              │
│          { y = 2x - 1                                              │
│                                                                     │
│   Substitute:                                                      │
│   2x + 3 = 2x - 1                                                  │
│   3 = -1                                                           │
│                                                                     │
│   This is FALSE! There's no value of x that makes this true.     │
│                                                                     │
│   Interpretation:                                                  │
│   Both lines have slope 2 but different y-intercepts.            │
│   They are PARALLEL and never intersect.                          │
│                                                                     │
│   Answer: No solution (∅) - Inconsistent system                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Infinitely Many Solutions (Dependent System)

```
┌─────────────────────────────────────────────────────────────────────┐
│           INFINITELY MANY SOLUTIONS                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Solve: { y = 3x + 2                                              │
│          { 2y = 6x + 4                                             │
│                                                                     │
│   Substitute y = 3x + 2 into second equation:                     │
│   2(3x + 2) = 6x + 4                                               │
│   6x + 4 = 6x + 4                                                  │
│   4 = 4                                                            │
│                                                                     │
│   This is ALWAYS TRUE! Any x satisfies this.                      │
│                                                                     │
│   Interpretation:                                                  │
│   The second equation simplifies to y = 3x + 2 (same as first).  │
│   Both equations represent the SAME LINE.                         │
│                                                                     │
│   Answer: Infinitely many solutions                               │
│           Solution set: {(x, 3x + 2) | x ∈ ℝ}                     │
│           or: {(x, y) | y = 3x + 2}                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Strategies for Choosing Which Variable to Isolate

### Decision Guide

| Situation | Best Choice |
|-----------|-------------|
| One variable already isolated | Use that equation |
| Coefficient of 1 on a variable | Isolate that variable |
| Coefficient of -1 on a variable | Isolate that variable |
| Fractions would result | Consider elimination method |
| All coefficients are equal | Either variable works |

### Example: Choosing Wisely

```
System: { 2x + 3y = 12
        { x - 4y = 5

The second equation has x with coefficient 1.
Solve for x: x = 5 + 4y

This avoids fractions!

If you solved 2x + 3y = 12 for x:
x = (12 - 3y)/2 = 6 - 3y/2  ← creates fractions
```

---

## 6. Applications

### Word Problem Setup

```
┌─────────────────────────────────────────────────────────────────────┐
│           SETTING UP SYSTEMS                                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Problem: The sum of two numbers is 25. One number is 3 more     │
│   than twice the other. Find the numbers.                         │
│                                                                     │
│   Let x = first number, y = second number                         │
│                                                                     │
│   Equation 1 (from "sum is 25"):                                   │
│   x + y = 25                                                       │
│                                                                     │
│   Equation 2 (from "one is 3 more than twice the other"):         │
│   x = 2y + 3                                                       │
│                                                                     │
│   System: { x + y = 25                                             │
│           { x = 2y + 3                                             │
│                                                                     │
│   Substitute x = 2y + 3 into first equation:                      │
│   (2y + 3) + y = 25                                                │
│   3y + 3 = 25                                                      │
│   3y = 22                                                          │
│   y = 22/3                                                         │
│                                                                     │
│   x = 2(22/3) + 3 = 44/3 + 9/3 = 53/3                             │
│                                                                     │
│   Answer: The numbers are 53/3 and 22/3                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## ✏️ Solved Examples

### Example 1: Easy - Variable Already Isolated

**Problem:** Solve the system:
$\begin{cases} y = x + 4 \\ 2x + y = 10 \end{cases}$

**Solution:**
```
Step 1: y is already isolated in equation 1
        y = x + 4

Step 2: Substitute into equation 2
        2x + (x + 4) = 10

Step 3: Solve for x
        3x + 4 = 10
        3x = 6
        x = 2

Step 4: Back-substitute
        y = 2 + 4 = 6

Step 5: Check
        Eq 1: 6 = 2 + 4 ✓
        Eq 2: 2(2) + 6 = 10 ✓

Answer: (2, 6)
```

### Example 2: Easy - Coefficient of 1

**Problem:** Solve:
$\begin{cases} x + 3y = 11 \\ 2x - y = 1 \end{cases}$

**Solution:**
```
Step 1: Solve equation 1 for x
        x = 11 - 3y

Step 2: Substitute into equation 2
        2(11 - 3y) - y = 1
        22 - 6y - y = 1
        22 - 7y = 1

Step 3: Solve for y
        -7y = -21
        y = 3

Step 4: Back-substitute
        x = 11 - 3(3) = 11 - 9 = 2

Answer: (2, 3)
```

### Example 3: Medium - Need to Solve First

**Problem:** Solve:
$\begin{cases} 3x + 2y = 8 \\ x - y = 1 \end{cases}$

**Solution:**
```
Step 1: Solve equation 2 for x (coefficient is 1)
        x = y + 1

Step 2: Substitute into equation 1
        3(y + 1) + 2y = 8
        3y + 3 + 2y = 8
        5y = 5
        y = 1

Step 4: Back-substitute
        x = 1 + 1 = 2

Check:
Eq 1: 3(2) + 2(1) = 6 + 2 = 8 ✓
Eq 2: 2 - 1 = 1 ✓

Answer: (2, 1)
```

### Example 4: Medium - Negative Coefficient

**Problem:** Solve:
$\begin{cases} 4x - y = 9 \\ 2x + 3y = 8 \end{cases}$

**Solution:**
```
Step 1: Solve equation 1 for y (coefficient is -1)
        -y = 9 - 4x
        y = 4x - 9

Step 2: Substitute into equation 2
        2x + 3(4x - 9) = 8
        2x + 12x - 27 = 8
        14x = 35
        x = 35/14 = 5/2

Step 4: Back-substitute
        y = 4(5/2) - 9 = 10 - 9 = 1

Answer: (5/2, 1)
```

### Example 5: Hard - No Solution

**Problem:** Solve:
$\begin{cases} 2x - y = 5 \\ 6x - 3y = 10 \end{cases}$

**Solution:**
```
Step 1: Solve equation 1 for y
        y = 2x - 5

Step 2: Substitute into equation 2
        6x - 3(2x - 5) = 10
        6x - 6x + 15 = 10
        15 = 10

This is FALSE! The system has no solution.

Verification: Equation 2 divided by 3 gives 2x - y = 10/3
             Equation 1 says 2x - y = 5
             Since 10/3 ≠ 5, the lines are parallel.

Answer: No solution (system is inconsistent)
```

### Example 6: Hard - Word Problem

**Problem:** A rectangle's perimeter is 36 cm. Its length is 4 cm more than its width. Find the dimensions.

**Solution:**
```
Let l = length, w = width

From "perimeter is 36":
2l + 2w = 36

From "length is 4 more than width":
l = w + 4

Substitute l = w + 4 into first equation:
2(w + 4) + 2w = 36
2w + 8 + 2w = 36
4w = 28
w = 7 cm

l = 7 + 4 = 11 cm

Check: Perimeter = 2(11) + 2(7) = 22 + 14 = 36 ✓

Answer: Length = 11 cm, Width = 7 cm
```

---

## ❓ Practice Problems

### Easy Level

1. Solve: $\begin{cases} y = 2x \\ x + y = 9 \end{cases}$

2. Solve: $\begin{cases} x = y + 5 \\ 3x - 2y = 17 \end{cases}$

3. Solve: $\begin{cases} x + y = 10 \\ x - y = 4 \end{cases}$

### Medium Level

4. Solve: $\begin{cases} 2x + y = 7 \\ x - 3y = -6 \end{cases}$

5. Solve: $\begin{cases} 3x + 4y = 10 \\ x + 2y = 4 \end{cases}$

6. Solve: $\begin{cases} 5x + 2y = 16 \\ 3x - y = 4 \end{cases}$

### Hard Level

7. Solve: $\begin{cases} 4x - 2y = 10 \\ -2x + y = -5 \end{cases}$

8. The sum of two numbers is 50. The larger is 8 less than twice the smaller. Find the numbers.

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. Substitute y = 2x: x + 2x = 9 → 3x = 9 → x = 3, y = 6
   **$(3, 6)$**

2. Substitute x = y + 5: 3(y+5) - 2y = 17 → y + 15 = 17 → y = 2, x = 7
   **$(7, 2)$**

3. From x - y = 4: x = y + 4
   Substitute: (y + 4) + y = 10 → 2y = 6 → y = 3, x = 7
   **$(7, 3)$**

4. From equation 2: x = 3y - 6
   Substitute: 2(3y - 6) + y = 7 → 6y - 12 + y = 7 → 7y = 19 → y = 19/7
   x = 3(19/7) - 6 = 57/7 - 42/7 = 15/7
   **$(15/7, 19/7)$**

5. From equation 2: x = 4 - 2y
   Substitute: 3(4 - 2y) + 4y = 10 → 12 - 6y + 4y = 10 → -2y = -2 → y = 1
   x = 4 - 2(1) = 2
   **$(2, 1)$**

6. From equation 2: y = 3x - 4
   Substitute: 5x + 2(3x - 4) = 16 → 5x + 6x - 8 = 16 → 11x = 24 → x = 24/11
   y = 3(24/11) - 4 = 72/11 - 44/11 = 28/11
   **$(24/11, 28/11)$**

7. From equation 2: y = 2x - 5
   Substitute: 4x - 2(2x - 5) = 10 → 4x - 4x + 10 = 10 → 10 = 10
   **Infinitely many solutions** (equations are equivalent)

8. Let s = smaller, L = larger
   s + L = 50 and L = 2s - 8
   Substitute: s + (2s - 8) = 50 → 3s = 58 → s = 58/3
   L = 2(58/3) - 8 = 116/3 - 24/3 = 92/3
   **Smaller = 58/3 ≈ 19.33, Larger = 92/3 ≈ 30.67**

</details>

---

## 📋 Summary Table

| Situation | Result |
|-----------|--------|
| Unique solution found | One ordered pair (x, y) |
| False statement (3 = 5) | No solution (parallel lines) |
| True statement (0 = 0) | Infinitely many solutions (same line) |

---

## 🔄 Quick Revision Questions

1. **When is substitution most convenient?**

2. **What does it mean if you get "5 = 5" after substituting?**

3. **Solve for x in terms of y: $x - 3y = 7$**

4. **What is the first step in the substitution method?**

5. **If two lines are parallel, how many solutions does the system have?**

6. **After finding one variable, how do you find the other?**

<details>
<summary>Quick Answers</summary>

1. When one variable is already isolated or has coefficient 1 or -1
2. Infinitely many solutions (same line)
3. x = 7 + 3y
4. Solve one equation for one variable
5. No solutions
6. Back-substitute into one of the original equations

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ A system solution satisfies ALL equations simultaneously     │
│                                                                     │
│   ★ Substitution steps:                                            │
│     1. Solve one equation for one variable                        │
│     2. Substitute into the other equation                         │
│     3. Solve the resulting equation                               │
│     4. Back-substitute to find the other variable                │
│     5. Check in both original equations                          │
│                                                                     │
│   ★ Three possible outcomes:                                       │
│     • One solution (lines intersect)                              │
│     • No solution (parallel lines)                                │
│     • Infinite solutions (same line)                              │
│                                                                     │
│   ★ Choose the easiest variable to isolate (coefficient 1 or -1) │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Graphical Representation](../04-Linear-Equations/04-graphical-representation.md) | [Back to Contents](../README.md) | [Next: Elimination Method →](02-elimination-method.md)

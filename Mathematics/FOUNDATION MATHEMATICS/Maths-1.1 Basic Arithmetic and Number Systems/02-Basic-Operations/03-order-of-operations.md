# Chapter 2.3: Order of Operations (BODMAS/PEMDAS)

[← Previous: Multiplication and Division](02-multiplication-division.md) | [Back to Contents](../README.md) | [Next: Properties of Operations →](04-properties-of-operations.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Understand and apply the order of operations correctly
- Use BODMAS/PEMDAS rules to evaluate expressions
- Handle nested brackets and complex expressions
- Avoid common mistakes in order of operations
- Solve real-world problems using correct order

---

## 1. Why Order of Operations Matters

### The Problem
Without a standard order, the same expression can give different answers!

```
Evaluate: 2 + 3 × 4

Method 1 (left to right):
2 + 3 × 4 = 5 × 4 = 20

Method 2 (multiply first):
2 + 3 × 4 = 2 + 12 = 14

Which is correct? 🤔
```

### The Solution
Mathematicians agreed on a **universal standard**: BODMAS/PEMDAS

**The correct answer is 14** because multiplication comes before addition.

---

## 2. BODMAS Rule

### What BODMAS Stands For
```
┌─────────────────────────────────────────────────────┐
│                    B O D M A S                       │
├──────┬──────┬──────────┬────────────┬──────┬────────┤
│  B   │  O   │    D     │     M      │  A   │   S    │
├──────┼──────┼──────────┼────────────┼──────┼────────┤
│Brack-│Orders│Division  │Multipli-   │Addi- │Subtrac-│
│ets   │      │          │cation      │tion  │tion    │
├──────┼──────┼──────────┴────────────┼──────┴────────┤
│ ( )  │Powers│ ←  Same level  →     │ ← Same level →│
│ [ ]  │Roots │   Left to Right      │  Left to Right│
│ { }  │      │                       │               │
└──────┴──────┴───────────────────────┴───────────────┘
```

### Memory Trick
```
"BODMAS: Big Old Dogs Make Awful Sounds"
```

---

## 3. PEMDAS Rule (American Version)

### What PEMDAS Stands For
```
┌─────────────────────────────────────────────────────┐
│                    P E M D A S                       │
├──────────────┬──────────┬──────────┬──────────┬─────┤
│      P       │    E     │   M/D    │   A/S    │     │
├──────────────┼──────────┼──────────┼──────────┼─────┤
│ Parentheses  │Exponents │Multiply  │ Add      │     │
│    ( )       │ Powers   │ Divide   │Subtract  │     │
│              │          │(L to R)  │(L to R)  │     │
└──────────────┴──────────┴──────────┴──────────┴─────┘
```

### Memory Trick
```
"Please Excuse My Dear Aunt Sally"
```

### BODMAS vs PEMDAS - Same Rules!
```
Both systems give the SAME results:
• Brackets/Parentheses = Same thing
• Orders/Exponents = Same thing
• Division and Multiplication = Same level (left to right)
• Addition and Subtraction = Same level (left to right)
```

---

## 4. Priority Levels Explained

### Level 1: Brackets/Parentheses
**Highest priority - Do these FIRST**

Types of brackets (innermost first):
```
Parentheses: ( )
Square brackets: [ ]
Curly braces: { }

Order of evaluation:
{ [ ( innermost ) ] }
    ↑ First
      ↑ Second
        ↑ Third
```

### Level 2: Orders/Exponents
**Powers and roots come second**
```
Examples:
• 2³ = 8
• √16 = 4
• 5² = 25
```

### Level 3: Division and Multiplication
**Same priority - work LEFT to RIGHT**
```
Important: D and M are EQUAL!
6 ÷ 2 × 3 = 3 × 3 = 9 (left to right)
NOT: 6 ÷ 6 = 1 ✗
```

### Level 4: Addition and Subtraction
**Same priority - work LEFT to RIGHT**
```
Important: A and S are EQUAL!
10 - 4 + 3 = 6 + 3 = 9 (left to right)
NOT: 10 - 7 = 3 ✗
```

---

## 5. Step-by-Step Examples

### Example 1: Basic BODMAS
```
Evaluate: 18 + 6 ÷ 2 × 3 - 4

Step 1: No brackets
Step 2: No exponents
Step 3: Division and Multiplication (left to right)
        18 + 6 ÷ 2 × 3 - 4
        = 18 + 3 × 3 - 4
        = 18 + 9 - 4
Step 4: Addition and Subtraction (left to right)
        = 27 - 4
        = 23

Answer: 23
```

### Example 2: With Brackets
```
Evaluate: (8 + 4) × 3 - 6

Step 1: Brackets first
        (8 + 4) × 3 - 6
        = 12 × 3 - 6
Step 2: No exponents
Step 3: Multiplication
        = 36 - 6
Step 4: Subtraction
        = 30

Answer: 30
```

### Example 3: With Exponents
```
Evaluate: 5 + 2³ × 4 - 10

Step 1: No brackets
Step 2: Exponents
        5 + 2³ × 4 - 10
        = 5 + 8 × 4 - 10
Step 3: Multiplication
        = 5 + 32 - 10
Step 4: Addition and Subtraction
        = 37 - 10
        = 27

Answer: 27
```

### Example 4: Nested Brackets
```
Evaluate: 2 × [3 + (7 - 4)²]

Step 1: Innermost brackets first
        2 × [3 + (7 - 4)²]
        = 2 × [3 + (3)²]
Step 2: Exponents inside remaining bracket
        = 2 × [3 + 9]
Step 3: Remaining bracket
        = 2 × 12
Step 4: Multiplication
        = 24

Answer: 24
```

### Example 5: Complex Expression
```
Evaluate: 24 ÷ (8 - 2) × 2 + 3² - 1

Step 1: Brackets
        24 ÷ (8 - 2) × 2 + 3² - 1
        = 24 ÷ 6 × 2 + 3² - 1

Step 2: Exponents
        = 24 ÷ 6 × 2 + 9 - 1

Step 3: Division and Multiplication (left to right)
        = 4 × 2 + 9 - 1
        = 8 + 9 - 1

Step 4: Addition and Subtraction (left to right)
        = 17 - 1
        = 16

Answer: 16
```

---

## 6. The Division and Multiplication Trap

### Common Misconception
Many students think Division always comes before Multiplication. **This is WRONG!**

```
D and M have EQUAL priority!
Work LEFT to RIGHT.

Example: 12 ÷ 4 × 3

Wrong thinking: "Division before Multiplication"
12 ÷ 4 × 3 = 12 ÷ 12 = 1 ✗

Correct method: Left to Right
12 ÷ 4 × 3 = 3 × 3 = 9 ✓
```

### More Examples
```
8 × 4 ÷ 2 = 32 ÷ 2 = 16 ✓ (left to right)
6 ÷ 3 × 2 = 2 × 2 = 4 ✓ (left to right)
10 ÷ 2 ÷ 5 = 5 ÷ 5 = 1 ✓ (left to right)
```

---

## 7. The Addition and Subtraction Trap

### Same Rule Applies
A and S have EQUAL priority - work LEFT to RIGHT.

```
Example: 15 - 8 + 3

Wrong: 15 - 11 = 4 ✗ (doing addition first)
Correct: 7 + 3 = 10 ✓ (left to right)
```

### More Examples
```
20 - 5 + 3 - 2 = 15 + 3 - 2 = 18 - 2 = 16 ✓
10 + 4 - 6 + 2 = 14 - 6 + 2 = 8 + 2 = 10 ✓
```

---

## 8. Special Cases

### Negative Numbers in Expressions
```
Evaluate: -3² vs (-3)²

-3² = -(3²) = -(9) = -9
The negative is applied AFTER squaring

(-3)² = (-3) × (-3) = 9
The entire -3 is squared

These are DIFFERENT!
```

### Implied Multiplication
```
In some contexts:
2(3) = 2 × 3 = 6
a(b) = a × b

Some calculators treat implied multiplication with higher priority.
To be safe, always use explicit × or brackets.
```

### Fraction Bars Act as Brackets
```
    8 + 4
    ───── = (8 + 4) ÷ (2 + 1) = 12 ÷ 3 = 4
    2 + 1

The fraction bar groups numerator and denominator.
```

---

## 9. Visual BODMAS Chart

```
┌────────────────────────────────────────────────────────┐
│                  ORDER OF OPERATIONS                    │
│                                                        │
│    HIGHEST PRIORITY                                    │
│          ↓                                             │
│    ┌─────────────┐                                     │
│    │  BRACKETS   │  ( ) [ ] { }                        │
│    │             │  Work innermost to outermost        │
│    └──────┬──────┘                                     │
│           ↓                                            │
│    ┌─────────────┐                                     │
│    │   ORDERS    │  Powers: 2³, 5²                     │
│    │             │  Roots: √, ∛                        │
│    └──────┬──────┘                                     │
│           ↓                                            │
│    ┌─────────────┐                                     │
│    │  DIVISION   │                                     │
│    │     and     │  ← Same level →                     │
│    │MULTIPLICATION│  Work LEFT to RIGHT                │
│    └──────┬──────┘                                     │
│           ↓                                            │
│    ┌─────────────┐                                     │
│    │  ADDITION   │                                     │
│    │     and     │  ← Same level →                     │
│    │ SUBTRACTION │  Work LEFT to RIGHT                 │
│    └─────────────┘                                     │
│          ↓                                             │
│    LOWEST PRIORITY                                     │
└────────────────────────────────────────────────────────┘
```

---

## 10. Worked Examples with Detailed Steps

### Example 1: All Operations
```
Evaluate: 3 + 4 × 2² - 8 ÷ 4

Step 1: Brackets? None
Step 2: Orders (Exponents)
        2² = 4
        3 + 4 × 4 - 8 ÷ 4

Step 3: Division and Multiplication (L→R)
        4 × 4 = 16
        8 ÷ 4 = 2
        3 + 16 - 2

Step 4: Addition and Subtraction (L→R)
        3 + 16 = 19
        19 - 2 = 17

Answer: 17
```

### Example 2: Multiple Bracket Types
```
Evaluate: 50 - {20 + [18 - (4 + 6)]}

Step 1: Innermost brackets
        (4 + 6) = 10
        50 - {20 + [18 - 10]}

Step 2: Square brackets
        [18 - 10] = 8
        50 - {20 + 8}

Step 3: Curly braces
        {20 + 8} = 28
        50 - 28

Step 4: Subtraction
        50 - 28 = 22

Answer: 22
```

### Example 3: Fractions with BODMAS
```
Evaluate: (1/2 + 1/3) × 6 - 4/5

Step 1: Brackets
        1/2 + 1/3 = 3/6 + 2/6 = 5/6
        (5/6) × 6 - 4/5

Step 2: Multiplication
        5/6 × 6 = 5
        5 - 4/5

Step 3: Subtraction
        5 - 4/5 = 25/5 - 4/5 = 21/5

Answer: 21/5 or 4.2 or 4⅕
```

### Example 4: With Square Roots
```
Evaluate: √(16 + 9) + 2³ × 3

Step 1: Inside the square root (like brackets)
        √(16 + 9) = √25 = 5
        5 + 2³ × 3

Step 2: Exponents
        2³ = 8
        5 + 8 × 3

Step 3: Multiplication
        8 × 3 = 24
        5 + 24

Step 4: Addition
        5 + 24 = 29

Answer: 29
```

### Example 5: Expression with Variables (Substitution)
```
If a = 2 and b = 3, evaluate: 2a² + 3b - ab

Substitute:
2(2)² + 3(3) - (2)(3)

Step 1: Exponents
        2(4) + 3(3) - (2)(3)

Step 2: All multiplications
        8 + 9 - 6

Step 3: Addition and subtraction
        17 - 6 = 11

Answer: 11
```

---

## 11. Real-World Applications

### Discount Calculations
```
Original price: ₹500
Discount: 20%
Tax: 10% on discounted price

Price = 500 - (500 × 0.20) + (500 - 500 × 0.20) × 0.10
      = 500 - 100 + (400) × 0.10
      = 500 - 100 + 40
      = 440

Or with proper BODMAS:
Price = 500 × (1 - 0.20) × (1 + 0.10)
      = 500 × 0.80 × 1.10
      = 440
```

### Speed Calculations
```
Average speed for a round trip:
Distance each way = 60 km
Speed going = 40 km/h
Speed returning = 60 km/h

Average speed = 2 × d ÷ (d/v₁ + d/v₂)
              = 2 × 60 ÷ (60/40 + 60/60)
              = 120 ÷ (1.5 + 1)
              = 120 ÷ 2.5
              = 48 km/h
```

---

## 12. Common Mistakes to Avoid

### Mistake 1: Left-to-Right Confusion
```
Wrong: 20 ÷ 4 × 5 = 20 ÷ 20 = 1 ✗
Correct: 20 ÷ 4 × 5 = 5 × 5 = 25 ✓
```

### Mistake 2: Forgetting Brackets Change Priority
```
Expression: 6 + 2 × 3 = 6 + 6 = 12
With brackets: (6 + 2) × 3 = 8 × 3 = 24
They are DIFFERENT!
```

### Mistake 3: Negative Signs
```
Wrong: -2² = 4 ✗
Correct: -2² = -(2²) = -4 ✓

But: (-2)² = 4 ✓
```

### Mistake 4: Ignoring Division Bar as Grouping
```
     6 + 2
     ───── = ?
       4

Wrong: 6 + 2 ÷ 4 = 6 + 0.5 = 6.5 ✗
Correct: (6 + 2) ÷ 4 = 8 ÷ 4 = 2 ✓
```

---

## 📊 Summary Table

### BODMAS Priority
| Priority | Operation | Symbol | Notes |
|----------|-----------|--------|-------|
| 1 (Highest) | Brackets | ( ) [ ] { } | Innermost first |
| 2 | Orders | ² ³ √ | Powers and roots |
| 3 | Division/Multiplication | ÷ × | Left to right |
| 4 (Lowest) | Addition/Subtraction | + - | Left to right |

### Quick Reference
| Expression | Answer | Explanation |
|------------|--------|-------------|
| 2 + 3 × 4 | 14 | Multiply first |
| (2 + 3) × 4 | 20 | Brackets first |
| 2³ + 1 | 9 | Exponent first |
| 12 ÷ 4 × 3 | 9 | Left to right |
| 10 - 4 + 2 | 8 | Left to right |
| -3² | -9 | Square then negate |
| (-3)² | 9 | Negate then square |

---

## ❓ Quick Revision Questions

1. **Evaluate**: 8 + 2 × 5 - 4 ÷ 2

2. **Evaluate**: (15 - 5) ÷ 2 + 3²

3. **Evaluate**: 24 ÷ 6 ÷ 2 × 3

4. **Evaluate**: 100 - {50 - [30 - (10 + 5)]}

5. **Which is greater**: 4 × 3² or (4 × 3)²?

6. **True or False**: In BODMAS, Division always comes before Multiplication.

<details>
<summary>Click to see answers</summary>

1. 8 + 2 × 5 - 4 ÷ 2
   = 8 + 10 - 2
   = 18 - 2
   = **16**

2. (15 - 5) ÷ 2 + 3²
   = 10 ÷ 2 + 9
   = 5 + 9
   = **14**

3. 24 ÷ 6 ÷ 2 × 3
   = 4 ÷ 2 × 3
   = 2 × 3
   = **6**

4. 100 - {50 - [30 - (10 + 5)]}
   = 100 - {50 - [30 - 15]}
   = 100 - {50 - 15}
   = 100 - 35
   = **65**

5. 4 × 3² = 4 × 9 = 36
   (4 × 3)² = 12² = 144
   **(4 × 3)² is greater**

6. **False** - Division and Multiplication have equal priority; work left to right.

</details>

---

## 🔗 Navigation

[← Previous: Multiplication and Division](02-multiplication-division.md) | [Back to Contents](../README.md) | [Next: Properties of Operations →](04-properties-of-operations.md)

# Chapter 3.4: Divisibility Rules

[← Previous: HCF and LCM](03-hcf-lcm.md) | [Back to Contents](../README.md) | [Next: Types of Fractions →](../04-Fractions/01-types-of-fractions.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Apply divisibility rules for 2, 3, 4, 5, 6, 7, 8, 9, 10, and 11
- Quickly determine divisibility without calculation
- Combine rules for composite number tests
- Use divisibility to simplify problems

---

## 1. What is Divisibility?

### Definition
A number **a** is divisible by **b** if dividing a by b leaves no remainder.

```
a is divisible by b  ⟺  a = b × k for some integer k
                     ⟺  a mod b = 0 (remainder is 0)
```

### Notation
```
b | a  means "b divides a" or "a is divisible by b"

Examples:
3 | 12  ✓ (12 = 3 × 4)
4 | 20  ✓ (20 = 4 × 5)
5 | 17  ✗ (17 = 5 × 3 + 2, remainder 2)
```

---

## 2. Divisibility by 2

### Rule
A number is divisible by 2 if its **last digit is even** (0, 2, 4, 6, or 8).

```
╔══════════════════════════════════════╗
║  Divisible by 2 if last digit is:    ║
║         0, 2, 4, 6, or 8             ║
╚══════════════════════════════════════╝
```

### Examples
```
124    → Last digit 4 (even)   → Divisible by 2 ✓
537    → Last digit 7 (odd)    → Not divisible by 2 ✗
1,280  → Last digit 0 (even)   → Divisible by 2 ✓
45,673 → Last digit 3 (odd)    → Not divisible by 2 ✗
```

### Why It Works
```
Any number can be written as: 10a + b (where b is last digit)
10a is always even (divisible by 2).
So the whole number is even only if b (last digit) is even.

Example: 346 = 340 + 6 = 34×10 + 6
         340 is even, and 6 is even
         So 346 is even ✓
```

---

## 3. Divisibility by 3

### Rule
A number is divisible by 3 if the **sum of its digits** is divisible by 3.

```
╔══════════════════════════════════════╗
║  Divisible by 3 if:                  ║
║  Sum of all digits is ÷ by 3        ║
╚══════════════════════════════════════╝
```

### Examples
```
123  → 1 + 2 + 3 = 6      → 6 ÷ 3 = 2    → Divisible ✓
456  → 4 + 5 + 6 = 15     → 15 ÷ 3 = 5   → Divisible ✓
742  → 7 + 4 + 2 = 13     → 13 ÷ 3 = 4.3 → Not divisible ✗
8571 → 8 + 5 + 7 + 1 = 21 → 21 ÷ 3 = 7   → Divisible ✓
```

### Repeated Application
```
Is 987,654 divisible by 3?

First sum: 9 + 8 + 7 + 6 + 5 + 4 = 39
Second sum: 3 + 9 = 12
Third sum: 1 + 2 = 3 ✓

Since we reached 3, it's divisible by 3!
```

### Why It Works
```
Powers of 10 leave remainder 1 when divided by 3:
10 = 9 + 1 = 3(3) + 1
100 = 99 + 1 = 3(33) + 1
1000 = 999 + 1 = 3(333) + 1

So any number:
abc = 100a + 10b + c
    = (99+1)a + (9+1)b + c
    = 99a + 9b + (a + b + c)
    = 3(33a + 3b) + (a + b + c)

The number is ÷ by 3 if (a + b + c) is ÷ by 3.
```

---

## 4. Divisibility by 4

### Rule
A number is divisible by 4 if the **last two digits** form a number divisible by 4.

```
╔══════════════════════════════════════╗
║  Divisible by 4 if:                  ║
║  Last 2 digits form a number ÷ by 4 ║
╚══════════════════════════════════════╝
```

### Examples
```
524   → Last 2 digits: 24 → 24 ÷ 4 = 6    → Divisible ✓
1,356 → Last 2 digits: 56 → 56 ÷ 4 = 14   → Divisible ✓
2,318 → Last 2 digits: 18 → 18 ÷ 4 = 4.5  → Not divisible ✗
5,000 → Last 2 digits: 00 → 0 ÷ 4 = 0     → Divisible ✓
```

### Quick Reference: Two-Digit Numbers Divisible by 4
```
04, 08, 12, 16, 20, 24, 28, 32, 36, 40,
44, 48, 52, 56, 60, 64, 68, 72, 76, 80,
84, 88, 92, 96, 00
```

### Why It Works
```
100 = 4 × 25, so 100 is divisible by 4.
Any multiple of 100 is divisible by 4.

For number ...xy (last two digits xy):
The number = 100k + xy (for some k)
100k is ÷ by 4, so we only check xy.
```

---

## 5. Divisibility by 5

### Rule
A number is divisible by 5 if its **last digit is 0 or 5**.

```
╔══════════════════════════════════════╗
║  Divisible by 5 if last digit is:    ║
║              0 or 5                  ║
╚══════════════════════════════════════╝
```

### Examples
```
125  → Last digit 5 → Divisible ✓
340  → Last digit 0 → Divisible ✓
732  → Last digit 2 → Not divisible ✗
1,005 → Last digit 5 → Divisible ✓
```

### Why It Works
```
10 = 5 × 2, so any multiple of 10 ends in 0.
And 5 divides any number ending in 0 or 5.
```

---

## 6. Divisibility by 6

### Rule
A number is divisible by 6 if it's divisible by **both 2 AND 3**.

```
╔══════════════════════════════════════╗
║  Divisible by 6 if:                  ║
║  • Last digit is even (test for 2)  ║
║  AND                                 ║
║  • Digit sum is ÷ by 3 (test for 3) ║
╚══════════════════════════════════════╝
```

### Examples
```
126 → Even? 6 ✓ | Digit sum: 1+2+6=9 ÷3 ✓ → Divisible ✓
324 → Even? 4 ✓ | Digit sum: 3+2+4=9 ÷3 ✓ → Divisible ✓
135 → Even? 5 ✗ | (Don't need to check 3) → Not divisible ✗
148 → Even? 8 ✓ | Digit sum: 1+4+8=13 ✗ → Not divisible ✗
```

### Why It Works
```
6 = 2 × 3 (2 and 3 are co-prime)

For divisibility by any composite number n:
Check divisibility by its co-prime factor pairs.
```

---

## 7. Divisibility by 7

### Rule (Doubling and Subtracting)
```
Step 1: Take the last digit
Step 2: Double it
Step 3: Subtract from the remaining number
Step 4: Repeat until result is small
Step 5: Check if final result is ÷ by 7
```

### Examples
```
Is 203 divisible by 7?

Last digit: 3, Double: 6
Remaining: 20
20 - 6 = 14

14 ÷ 7 = 2 ✓ → 203 is divisible by 7
```

```
Is 364 divisible by 7?

Last digit: 4, Double: 8
36 - 8 = 28

28 ÷ 7 = 4 ✓ → 364 is divisible by 7
```

```
Is 905 divisible by 7?

Last digit: 5, Double: 10
90 - 10 = 80

80 ÷ 7 = 11.4... ✗ → 905 is NOT divisible by 7
```

### For Larger Numbers
```
Is 2,401 divisible by 7?

2401 → 240 - 2(1) = 240 - 2 = 238
 238 → 23 - 2(8) = 23 - 16 = 7

7 ÷ 7 = 1 ✓ → 2,401 is divisible by 7
```

---

## 8. Divisibility by 8

### Rule
A number is divisible by 8 if the **last three digits** form a number divisible by 8.

```
╔══════════════════════════════════════╗
║  Divisible by 8 if:                  ║
║  Last 3 digits form a number ÷ by 8 ║
╚══════════════════════════════════════╝
```

### Examples
```
1,256 → Last 3 digits: 256 → 256 ÷ 8 = 32  → Divisible ✓
5,000 → Last 3 digits: 000 → 0 ÷ 8 = 0     → Divisible ✓
2,718 → Last 3 digits: 718 → 718 ÷ 8 = 89.75 → Not divisible ✗
3,168 → Last 3 digits: 168 → 168 ÷ 8 = 21  → Divisible ✓
```

### Alternative Method for 8
```
For a 3-digit number abc:
Test: 4a + 2b + c

If this result is ÷ by 8, the number is ÷ by 8.

Example: Is 168 divisible by 8?
4(1) + 2(6) + 8 = 4 + 12 + 8 = 24
24 ÷ 8 = 3 ✓ → 168 is divisible by 8
```

### Why It Works
```
1000 = 8 × 125, so 1000 is divisible by 8.
Any multiple of 1000 is divisible by 8.
We only need to check the last 3 digits.
```

---

## 9. Divisibility by 9

### Rule
A number is divisible by 9 if the **sum of its digits** is divisible by 9.

```
╔══════════════════════════════════════╗
║  Divisible by 9 if:                  ║
║  Sum of all digits is ÷ by 9        ║
╚══════════════════════════════════════╝
```

### Examples
```
729  → 7 + 2 + 9 = 18     → 18 ÷ 9 = 2    → Divisible ✓
5,463 → 5 + 4 + 6 + 3 = 18 → 18 ÷ 9 = 2   → Divisible ✓
284  → 2 + 8 + 4 = 14     → 14 ÷ 9 = 1.5  → Not divisible ✗
```

### Relationship with Divisibility by 3
```
If a number is divisible by 9:
→ It's also divisible by 3 (since 3 | 9)

If digit sum is 9, 18, 27, 36, ... → Divisible by 9
If digit sum is 3, 6, 12, 15, ... → Divisible by 3 only
```

### Example Comparison
```
Number: 123
Digit sum: 1 + 2 + 3 = 6
6 ÷ 3 = 2 ✓ but 6 ÷ 9 ✗
→ Divisible by 3, NOT by 9

Number: 126
Digit sum: 1 + 2 + 6 = 9
9 ÷ 3 = 3 ✓ and 9 ÷ 9 = 1 ✓
→ Divisible by BOTH 3 and 9
```

---

## 10. Divisibility by 10

### Rule
A number is divisible by 10 if its **last digit is 0**.

```
╔══════════════════════════════════════╗
║  Divisible by 10 if:                 ║
║  Last digit is 0                     ║
╚══════════════════════════════════════╝
```

### Examples
```
150  → Last digit 0 → Divisible ✓
2,340 → Last digit 0 → Divisible ✓
105  → Last digit 5 → Not divisible ✗
```

### Extensions
```
Divisible by 100: Last 2 digits are 00
Divisible by 1000: Last 3 digits are 000

Examples:
1,200 → Ends in 00 → Divisible by 100 ✓
5,000 → Ends in 000 → Divisible by 1000 ✓
```

---

## 11. Divisibility by 11

### Rule
A number is divisible by 11 if the **alternating sum of digits** (from right) is divisible by 11.

```
╔══════════════════════════════════════════════════╗
║  Divisible by 11 if:                             ║
║  (Sum of odd-position digits from right) -       ║
║  (Sum of even-position digits from right)        ║
║  is divisible by 11                              ║
╚══════════════════════════════════════════════════╝
```

### Method
```
For number with digits: d₁ d₂ d₃ d₄ d₅ ... (left to right)

Alternating sum = d₁ - d₂ + d₃ - d₄ + d₅ - ...

Or equivalently:
(Sum of digits in odd positions) - (Sum of digits in even positions)
```

### Examples
```
Is 121 divisible by 11?

Positions:  1   2   1
           (1) (2) (3)

Alternating sum: 1 - 2 + 1 = 0
0 is divisible by 11 ✓ → 121 is divisible by 11
```

```
Is 8,294 divisible by 11?

Positions:  8   2   9   4
           (1) (2) (3) (4)

Alternating sum: 8 - 2 + 9 - 4 = 11
11 is divisible by 11 ✓ → 8,294 is divisible by 11
```

```
Is 9,537 divisible by 11?

Alternating sum: 9 - 5 + 3 - 7 = 0
0 is divisible by 11 ✓ → 9,537 is divisible by 11
```

```
Is 123,456 divisible by 11?

1 - 2 + 3 - 4 + 5 - 6 = -3
-3 is not divisible by 11 ✗ → Not divisible
```

### Negative Results Are Fine
```
The alternating sum can be 0, positive, or negative.
Just check if it's 0, ±11, ±22, ±33, etc.

Example: -22 is divisible by 11 ✓
```

---

## 12. Divisibility Rules Summary Chart

```
┌──────────┬──────────────────────────────────────────────┐
│ Divisor  │                    Rule                      │
├──────────┼──────────────────────────────────────────────┤
│    2     │ Last digit is 0, 2, 4, 6, or 8              │
├──────────┼──────────────────────────────────────────────┤
│    3     │ Sum of digits divisible by 3                │
├──────────┼──────────────────────────────────────────────┤
│    4     │ Last 2 digits form number divisible by 4    │
├──────────┼──────────────────────────────────────────────┤
│    5     │ Last digit is 0 or 5                        │
├──────────┼──────────────────────────────────────────────┤
│    6     │ Divisible by both 2 AND 3                   │
├──────────┼──────────────────────────────────────────────┤
│    7     │ Double last digit, subtract from rest       │
├──────────┼──────────────────────────────────────────────┤
│    8     │ Last 3 digits form number divisible by 8    │
├──────────┼──────────────────────────────────────────────┤
│    9     │ Sum of digits divisible by 9                │
├──────────┼──────────────────────────────────────────────┤
│   10     │ Last digit is 0                             │
├──────────┼──────────────────────────────────────────────┤
│   11     │ Alternating sum of digits divisible by 11   │
├──────────┼──────────────────────────────────────────────┤
│   12     │ Divisible by both 3 AND 4                   │
├──────────┼──────────────────────────────────────────────┤
│   15     │ Divisible by both 3 AND 5                   │
├──────────┼──────────────────────────────────────────────┤
│   18     │ Divisible by both 2 AND 9                   │
└──────────┴──────────────────────────────────────────────┘
```

---

## 13. Combining Rules for Composite Divisors

### Rule for Composite Numbers
```
To test divisibility by a composite number n:
1. Factor n into co-prime factors
2. Test divisibility by each factor

Examples:
12 = 3 × 4 (3 and 4 are co-prime)
    Test: divisible by 3? AND divisible by 4?

15 = 3 × 5 (3 and 5 are co-prime)
    Test: divisible by 3? AND divisible by 5?

18 = 2 × 9 (2 and 9 are co-prime)
    Test: divisible by 2? AND divisible by 9?
```

### Example: Is 1,620 Divisible by 12?
```
12 = 3 × 4

Test for 3:
1 + 6 + 2 + 0 = 9 → 9 ÷ 3 = 3 ✓

Test for 4:
Last 2 digits: 20 → 20 ÷ 4 = 5 ✓

Both pass → 1,620 is divisible by 12!
```

### Example: Is 2,835 Divisible by 45?
```
45 = 5 × 9

Test for 5:
Last digit: 5 ✓

Test for 9:
2 + 8 + 3 + 5 = 18 → 18 ÷ 9 = 2 ✓

Both pass → 2,835 is divisible by 45!
```

---

## 14. Solved Examples

### Example 1: Complete Divisibility Test
```
Test 3,456 for divisibility by 2, 3, 4, 5, 6, 8, 9

By 2: Last digit 6 (even) → ✓
By 3: 3+4+5+6 = 18 → 18÷3 = 6 → ✓
By 4: Last 2 digits 56 → 56÷4 = 14 → ✓
By 5: Last digit 6 (not 0 or 5) → ✗
By 6: ÷ by 2 ✓ and ÷ by 3 ✓ → ✓
By 8: Last 3 digits 456 → 456÷8 = 57 → ✓
By 9: Digit sum 18 → 18÷9 = 2 → ✓

3,456 is divisible by: 2, 3, 4, 6, 8, 9
Not divisible by: 5
```

### Example 2: Finding Missing Digit
```
Find the value of * if 52*4 is divisible by 9.

For ÷ by 9: digit sum must be ÷ by 9
5 + 2 + * + 4 = 11 + *

11 + * must be 18 (next multiple of 9 after 11)
* = 7

Answer: 5274
Check: 5+2+7+4 = 18 → 18÷9 = 2 ✓
```

### Example 3: Finding Missing Digit for Two Rules
```
Find * if 73*8 is divisible by 4 and 9.

For ÷ by 4: Last 2 digits (*8) must be ÷ by 4
*8 could be: 08, 28, 48, 68, 88
So * could be: 0, 2, 4, 6, 8

For ÷ by 9: digit sum must be ÷ by 9
7 + 3 + * + 8 = 18 + *
Need: 18 + * = 18 or 27
So * = 0 or 9

Common value: * = 0

Answer: 7308
Verify: 08 ÷ 4 = 2 ✓
        7+3+0+8 = 18 → 18÷9 = 2 ✓
```

### Example 4: Divisibility by 11
```
Is 31,415 divisible by 11?

Alternating sum (from left):
3 - 1 + 4 - 1 + 5 = 10

10 is not divisible by 11.
31,415 is NOT divisible by 11.

Closest: 31,416 - 6 = 31,410 + 9 = 31,416
Actually: For 31,416: 3-1+4-1+6 = 11 ✓
```

### Example 5: Divisibility by 7
```
Is 1,372 divisible by 7?

1372 → 137 - 2(2) = 137 - 4 = 133
133 → 13 - 2(3) = 13 - 6 = 7

7 ÷ 7 = 1 ✓

1,372 is divisible by 7.
1,372 ÷ 7 = 196
```

---

## 15. Mental Math Tips 🧠

### Quick Tests in Order
```
Always test easier rules first:
1. Check for 2 (look at last digit)
2. Check for 5 (last digit 0 or 5)
3. Check for 3 (sum digits)
4. Check for 4 (last two digits)
5. Then test others as needed
```

### Patterns for 4 and 8
```
For 4: Numbers ending in 00, 04, 08, 12, 16, 20, 24, ...
       Pattern: 00, 04, 08, 12, ..., 96

For 8: Quick multiples: 8, 16, 24, 32, 40, 48, 56, 64, ...
       Last 3 digits divisible by 8
```

### The "Casting Out Nines" Trick
```
For checking ÷ by 9:
Keep adding digits until single digit.
If result is 9, divisible by 9.

Example: 873,234
8+7+3+2+3+4 = 27
2+7 = 9 ✓

Quick: Cross out 9s and pairs summing to 9
8,7,3,2,3,4 → (8+1 not there), 7+2=9, 3+3+3=9, 4 remains
Wait, let me redo: 8,7,3,2,3,4
Pairs to 9: none directly... just sum: 27 → 9 ✓
```

---

## 📊 Summary Table

### Quick Reference Chart

| Test | What to Check | Example |
|------|---------------|---------|
| ÷ by 2 | Last digit even | 348 → 8 even ✓ |
| ÷ by 3 | Sum of digits | 123 → 1+2+3=6 ÷3 ✓ |
| ÷ by 4 | Last 2 digits | 524 → 24÷4=6 ✓ |
| ÷ by 5 | Last digit 0 or 5 | 175 → ends in 5 ✓ |
| ÷ by 6 | ÷2 AND ÷3 | 324 → even, sum=9 ✓ |
| ÷ by 8 | Last 3 digits | 1032 → 032÷8=4 ✓ |
| ÷ by 9 | Sum ÷ by 9 | 729 → 7+2+9=18÷9 ✓ |
| ÷ by 10 | Last digit 0 | 540 → ends in 0 ✓ |
| ÷ by 11 | Alt. sum | 847 → 8-4+7=11 ✓ |

### Memory Aids
```
2 → Even Even Even!
3 → Sum it up, divide by 3
4 → Last TWO (2²=4)
5 → High FIVE (0 or 5)
6 → 2 × 3 (both tests)
8 → Last THREE (2³=8)
9 → Sum it up, divide by 9
10 → Zero at the END
11 → Alternating dance
```

---

## ❓ Quick Revision Questions

1. **Test**: Is 4,536 divisible by 4, 6, 8, and 9?

2. **Find**: The smallest value of n that makes 52n4 divisible by 9.

3. **Determine**: Is 7,293 divisible by 11?

4. **Test**: Is 1,295 divisible by 7?

5. **Find**: All values of a (0-9) for which 31a2 is divisible by 4.

6. **Problem**: A number is divisible by both 8 and 9. What is the smallest such 3-digit number?

<details>
<summary>Click to see answers</summary>

1. 4,536:
   - ÷ by 4: 36 ÷ 4 = 9 ✓
   - ÷ by 6: Even? Yes. Sum = 18, 18÷3=6 ✓ So yes ✓
   - ÷ by 8: 536 ÷ 8 = 67 ✓
   - ÷ by 9: Sum = 4+5+3+6 = 18, 18÷9=2 ✓
   
   **4,536 is divisible by all four!**

2. For 52n4 ÷ by 9: 5+2+n+4 = 11+n must be ÷ by 9
   11+n = 18 (next multiple of 9)
   n = **7** (giving 5274)

3. 7,293: Alt sum = 7-2+9-3 = 11
   11 is divisible by 11 ✓
   **Yes, 7,293 is divisible by 11**

4. 1,295: 129 - 2(5) = 129 - 10 = 119
   119: 11 - 2(9) = 11 - 18 = -7
   -7 is divisible by 7 ✓
   **Yes, 1,295 is divisible by 7** (1295÷7=185)

5. For 31a2 ÷ by 4: Last 2 digits "a2" must be ÷ by 4
   02÷4 ✗, 12÷4=3 ✓, 22÷4 ✗, 32÷4=8 ✓, 42÷4 ✗, 
   52÷4=13 ✓, 62÷4 ✗, 72÷4=18 ✓, 82÷4 ✗, 92÷4=23 ✓
   
   **a = 1, 3, 5, 7, or 9**

6. Divisible by 8 AND 9 means divisible by LCM(8,9) = 72
   Smallest 3-digit multiple of 72:
   72 × 1 = 72 (2-digit)
   72 × 2 = 144 (3-digit) ✓
   
   **Answer: 144**

</details>

---

## 🔗 Navigation

[← Previous: HCF and LCM](03-hcf-lcm.md) | [Back to Contents](../README.md) | [Next: Types of Fractions →](../04-Fractions/01-types-of-fractions.md)

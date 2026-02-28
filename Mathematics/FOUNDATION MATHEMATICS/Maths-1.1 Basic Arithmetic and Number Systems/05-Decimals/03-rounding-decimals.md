# Chapter 5.3: Rounding Decimals

[← Previous: Operations on Decimals](02-operations-on-decimals.md) | [Back to Contents](../README.md) | [Next: Scientific Notation →](04-scientific-notation.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Round decimals to specified decimal places
- Round to significant figures
- Apply different rounding rules
- Estimate calculations using rounding
- Understand when rounding is appropriate

---

## 1. Why Round Decimals?

### Practical Reasons
```
• Simplify numbers for communication
• Match precision of measurements
• Make mental calculations easier
• Avoid false precision
• Meet formatting requirements
```

### Example: Real World Rounding
```
Calculator shows: 3.141592653589793...
For everyday use:
• π ≈ 3.14 (2 decimal places)
• π ≈ 3.1416 (4 decimal places)
• π ≈ 3 (whole number)

Money: $45.6789 → $45.68 (to nearest cent)
```

---

## 2. Basic Rounding Rules

### The Decision Digit
```
To round to n decimal places:
Look at the digit in position (n + 1)

This is the "decision digit" or "key digit"
```

### The Rule
```
╔═══════════════════════════════════════════════════════╗
║  If decision digit is 0, 1, 2, 3, or 4 → Round DOWN   ║
║  If decision digit is 5, 6, 7, 8, or 9 → Round UP     ║
╚═══════════════════════════════════════════════════════╝

Round DOWN: Keep the last digit the same
Round UP: Increase the last digit by 1
```

### Visual Guide
```
Decision digit:
0 1 2 3 4 │ 5 6 7 8 9
    ↓     │     ↓
Round DOWN│ Round UP
```

---

## 3. Rounding to Decimal Places

### Rounding to 1 Decimal Place
```
3.47 → 3.5  (7 ≥ 5, round up)
3.42 → 3.4  (2 < 5, round down)
3.45 → 3.5  (5 = 5, round up)
3.95 → 4.0  (5 = 5, round up; 9+1=10, carry)

Step by step for 3.47:
• Target: 1 decimal place (tenths)
• Decision digit: 7 (in hundredths place)
• 7 ≥ 5, so round UP
• 3.47 → 3.5
```

### Rounding to 2 Decimal Places
```
4.567 → 4.57  (7 ≥ 5, round up)
4.562 → 4.56  (2 < 5, round down)
4.5649 → 4.56 (4 < 5, round down)
4.995 → 5.00  (5 = 5, round up; cascades)

Step by step for 4.567:
• Target: 2 decimal places (hundredths)
• Decision digit: 7 (in thousandths place)
• 7 ≥ 5, so round UP
• 4.567 → 4.57
```

### Rounding to 3 Decimal Places
```
0.3456 → 0.346  (6 ≥ 5, round up)
0.3454 → 0.345  (4 < 5, round down)
2.71828 → 2.718 (2 < 5, round down)
```

### Practice Table
```
Number     1 dp    2 dp    3 dp
───────────────────────────────
3.14159   3.1     3.14    3.142
2.71828   2.7     2.72    2.718
0.6789    0.7     0.68    0.679
9.9996    10.0    10.00   10.000
4.5555    4.6     4.56    4.556
```

---

## 4. Rounding to Nearest Whole Number

### The Process
```
Look at the tenths place (first decimal digit)

12.3 → 12  (3 < 5, round down)
12.7 → 13  (7 ≥ 5, round up)
12.5 → 13  (5 = 5, round up)
```

### Examples
```
4.2 → 4
4.8 → 5
4.5 → 5
99.9 → 100
0.4 → 0
0.6 → 1
```

### Visual: Number Line
```
Round 7.3 and 7.8:

       7.3        7.8
        ↓          ↓
7 |----●----|----|----●----|  8
        ↓               ↓
       ↙               ↘
      7                  8

7.3 is closer to 7, so rounds to 7
7.8 is closer to 8, so rounds to 8
```

---

## 5. Rounding to 10s, 100s, 1000s

### To Nearest 10
```
47 → 50   (7 ≥ 5)
43 → 40   (3 < 5)
45 → 50   (5 = 5)
234 → 230 (4 < 5)
```

### To Nearest 100
```
347 → 300   (47 < 50)
372 → 400   (72 ≥ 50)
350 → 400   (50 = 50)
1,456 → 1,500
```

### To Nearest 1000
```
3,456 → 3,000   (456 < 500)
3,678 → 4,000   (678 ≥ 500)
15,500 → 16,000
```

### Examples Table
```
Number    Nearest 10   Nearest 100   Nearest 1000
─────────────────────────────────────────────────
4,567     4,570        4,600         5,000
3,234     3,230        3,200         3,000
8,950     8,950        9,000         9,000
12,456    12,460       12,500        12,000
```

---

## 6. Significant Figures

### What Are Significant Figures?
```
Significant figures (sig figs) are the meaningful digits in a number.

Rules for counting significant figures:
1. Non-zero digits are always significant
2. Zeros between non-zeros are significant
3. Leading zeros are NOT significant
4. Trailing zeros after decimal ARE significant
5. Trailing zeros without decimal may or may not be significant
```

### Examples of Counting
```
Number        Sig Figs    Explanation
─────────────────────────────────────────────
345           3           All non-zero
3,045         4           Zero between 3 and 4 counts
0.0045        2           Leading zeros don't count
4.500         4           Trailing zeros after decimal count
300           1, 2, or 3  Ambiguous without context
3.00 × 10²    3           Scientific notation clarifies
```

### Rounding to Significant Figures
```
Round 0.004567 to:

2 sig figs: 0.0046
           First 2 sig figs are 4 and 5
           Decision digit: 6 ≥ 5, round up

3 sig figs: 0.00457
           First 3 sig figs are 4, 5, 6
           Decision digit: 7 ≥ 5, round up
```

### More Examples
```
Round 34,567 to:
2 sig figs: 35,000
3 sig figs: 34,600
4 sig figs: 34,570

Round 0.0023456 to:
2 sig figs: 0.0023
3 sig figs: 0.00235
4 sig figs: 0.002346
```

---

## 7. Special Rounding Cases

### The "Rounding 9" Problem
```
When rounding up causes 9 to become 10:

4.97 → 5.0  (round to 1 dp: 7 ≥ 5)
           9 + 1 = 10, carry to ones

9.98 → 10.0 (round to 1 dp)
           9 + 1 = 10, carry continues

0.999 → 1.0 (round to 1 dp)
            Cascading carry
```

### Rounding to 5 (Banker's Rounding)
```
Some systems use "round half to even":
When digit is exactly 5, round to nearest even number.

2.5 → 2  (nearest even)
3.5 → 4  (nearest even)
4.5 → 4  (nearest even)
5.5 → 6  (nearest even)

This reduces bias over many operations.
(Not commonly used in basic math education)
```

### Consecutive Rounding Error
```
DON'T round in steps!

Wrong: 2.449 → 2.45 → 2.5 (two rounds)
Right: 2.449 → 2.4 (direct to 1 dp, since 4 < 5)

Always round from the original number!
```

---

## 8. Estimation by Rounding

### Estimating Calculations
```
Estimate before calculating to check your answer!

4.87 × 3.12
≈ 5 × 3 = 15
Actual: 15.1944 ✓ (close to estimate)

23.45 + 16.78 + 9.34
≈ 23 + 17 + 9 = 49
Actual: 49.57 ✓
```

### Estimating in Real Life
```
Shopping estimate:
$4.99 + $7.49 + $12.99 + $3.49
≈ $5 + $7 + $13 + $3 = $28
Actual: $28.96 (estimate was close!)

Distance estimate:
12.3 km + 8.7 km + 15.2 km
≈ 12 + 9 + 15 = 36 km
Actual: 36.2 km
```

### Front-End Estimation
```
Add the front (leading) digits first:
$4.29 + $3.87 + $2.45 + $1.99

Front-end: 4 + 3 + 2 + 1 = $10
Adjustment: 0.29 + 0.87 + 0.45 + 0.99 ≈ $2.60
Estimate: $10 + $2.60 = $12.60
Actual: $12.60 ✓
```

---

## 9. Solved Examples

### Example 1: Round to Specified Places
```
Round 56.7894 to:

Nearest whole: 57 (7 ≥ 5)
1 dp: 56.8 (8 ≥ 5)
2 dp: 56.79 (9 ≥ 5)
3 dp: 56.789 (4 < 5)
```

### Example 2: Round to Significant Figures
```
Round 0.0034567 to:

2 sf: 0.0035 (5 ≥ 5)
3 sf: 0.00346 (6 ≥ 5)
4 sf: 0.003457 (7 ≥ 5)
```

### Example 3: Cascading Round
```
Round 99.95 to 1 dp:

Decision digit: 5 ≥ 5, round up
9 + 1 = 10, write 0 carry 1
9 + 1 = 10, write 0 carry 1

99.95 → 100.0
```

### Example 4: Estimation Check
```
Is 34.5 × 2.8 = 966 correct?

Estimate: 35 × 3 = 105
Answer 966 is WAY too big!

Actual: 34.5 × 2.8 = 96.6 ← Decimal error in original!
```

### Example 5: Money Rounding
```
Round $127.456 to nearest cent (2 dp):

$127.456
Decision digit: 6 ≥ 5
Round up: $127.46
```

### Example 6: Measurement Context
```
A measurement is 4.567 cm with precision to 0.1 cm.
How should you report it?

Round to 1 dp (matching precision): 4.6 cm
```

---

## 10. Mental Math Tips 🧠

### Quick Rounding Checks
```
Is it over or under the halfway point?

For rounding 3.67 to 1 dp:
Halfway would be 3.65
3.67 > 3.65, so rounds UP to 3.7
```

### Remember the Rule
```
"5 or more, raise the score"
"4 or less, let it rest"
```

### Estimation Shortcuts
```
For × by number close to 10:
4.8 × 9.7 ≈ 5 × 10 = 50 (slight overestimate)

For money:
Round up all prices for a safe budget estimate
```

---

## 📊 Summary Table

### Rounding Rules

| Round To | Look At | Example |
|----------|---------|---------|
| Whole number | Tenths | 3.7 → 4 |
| 1 dp | Hundredths | 3.74 → 3.7 |
| 2 dp | Thousandths | 3.745 → 3.75 |
| Nearest 10 | Ones | 347 → 350 |
| 2 sig figs | 3rd sig fig | 345 → 350 |

### Decision Digit Actions

| Digit | Action | Example |
|-------|--------|---------|
| 0, 1, 2, 3, 4 | Round DOWN | 3.44 → 3.4 |
| 5, 6, 7, 8, 9 | Round UP | 3.45 → 3.5 |

### Common Decimal Roundings

| Original | 1 dp | 2 dp | Whole |
|----------|------|------|-------|
| 3.14159 | 3.1 | 3.14 | 3 |
| 2.71828 | 2.7 | 2.72 | 3 |
| 9.9999 | 10.0 | 10.00 | 10 |

---

## ❓ Quick Revision Questions

1. **Round** 4.6789 to 2 decimal places.

2. **Round** 0.00567 to 2 significant figures.

3. **Round** 3,456 to the nearest hundred.

4. **Round** 9.95 to 1 decimal place.

5. **Estimate** 4.9 × 6.1 by rounding to whole numbers.

6. **Round** $79.995 to the nearest cent.

<details>
<summary>Click to see answers</summary>

1. 4.6789 to 2 dp:
   Decision digit: 8 ≥ 5
   **4.68**

2. 0.00567 to 2 sf:
   First 2 sig figs: 5, 6
   Decision digit: 7 ≥ 5
   **0.0057**

3. 3,456 to nearest 100:
   Look at tens digit: 5 ≥ 5
   **3,500**

4. 9.95 to 1 dp:
   Decision digit: 5 ≥ 5, round up
   9 + 1 = 10, cascade
   **10.0**

5. 4.9 × 6.1
   ≈ 5 × 6 = **30**
   (Actual: 29.89)

6. $79.995 to nearest cent:
   Decision digit: 5 ≥ 5
   **$80.00**

</details>

---

## 🔗 Navigation

[← Previous: Operations on Decimals](02-operations-on-decimals.md) | [Back to Contents](../README.md) | [Next: Scientific Notation →](04-scientific-notation.md)

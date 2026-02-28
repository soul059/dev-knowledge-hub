# Chapter 3.1: Prime and Composite Numbers

[← Previous: Properties of Operations](../02-Basic-Operations/04-properties-of-operations.md) | [Back to Contents](../README.md) | [Next: Prime Factorization →](02-prime-factorization.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Define and identify prime numbers
- Define and identify composite numbers
- Understand the unique status of 0 and 1
- Test whether a number is prime
- Memorize prime numbers up to 100

---

## 1. Factors: The Building Blocks

### What are Factors?
**Factors** are numbers that divide exactly into another number (with no remainder).

```
Factors of 12:
12 ÷ 1 = 12 ✓    12 ÷ 2 = 6  ✓
12 ÷ 3 = 4  ✓    12 ÷ 4 = 3  ✓
12 ÷ 5 = 2.4 ✗   12 ÷ 6 = 2  ✓
12 ÷ 12 = 1 ✓

Factors of 12: {1, 2, 3, 4, 6, 12}
```

### Finding Factors Systematically
```
To find all factors of n:
1. Start with 1 and n (they're always factors)
2. Check 2, 3, 4, ... up to √n
3. For each factor f, n/f is also a factor

Example: Factors of 36
√36 = 6, so check 1 to 6:

1 × 36 = 36 ✓ → 1 and 36 are factors
2 × 18 = 36 ✓ → 2 and 18 are factors
3 × 12 = 36 ✓ → 3 and 12 are factors
4 × 9 = 36  ✓ → 4 and 9 are factors
5 × ? = 36  ✗ → 36/5 = 7.2, not integer
6 × 6 = 36  ✓ → 6 is a factor

Factors of 36: {1, 2, 3, 4, 6, 9, 12, 18, 36}
```

---

## 2. Prime Numbers

### Definition
A **prime number** is a natural number greater than 1 that has exactly **two factors**: 1 and itself.

```
A prime number p > 1:
• Can only be divided evenly by 1 and p
• Cannot be broken down into smaller factors
• Is a "building block" for other numbers
```

### Examples of Prime Numbers
```
2  → Factors: {1, 2}         ✓ Only 2 factors
3  → Factors: {1, 3}         ✓ Only 2 factors
5  → Factors: {1, 5}         ✓ Only 2 factors
7  → Factors: {1, 7}         ✓ Only 2 factors
11 → Factors: {1, 11}        ✓ Only 2 factors
13 → Factors: {1, 13}        ✓ Only 2 factors
```

### Visual: Prime Numbers as Indivisible
```
Can you arrange 7 objects into equal groups (other than 1 group or 7 groups)?

●●●●●●●

• 2 groups? 7 ÷ 2 = 3.5 ✗ (not whole)
• 3 groups? 7 ÷ 3 = 2.33 ✗ (not whole)

Only possible arrangements:
• 1 group of 7: ●●●●●●●
• 7 groups of 1: ● ● ● ● ● ● ●

Therefore, 7 is PRIME!
```

```
Can you arrange 6 objects into equal groups?

●●●●●●

• 2 groups: ●●● ●●●  ✓
• 3 groups: ●● ●● ●● ✓

6 can be divided into 2 × 3, so 6 is NOT prime!
```

---

## 3. Composite Numbers

### Definition
A **composite number** is a natural number greater than 1 that has **more than two factors**.

```
A composite number n > 1:
• Can be divided by numbers other than 1 and n
• Can be expressed as a product of smaller numbers
• Is "composed" of prime factors
```

### Examples of Composite Numbers
```
4  → Factors: {1, 2, 4}           ✓ More than 2 factors
6  → Factors: {1, 2, 3, 6}        ✓ More than 2 factors
8  → Factors: {1, 2, 4, 8}        ✓ More than 2 factors
9  → Factors: {1, 3, 9}           ✓ More than 2 factors
10 → Factors: {1, 2, 5, 10}       ✓ More than 2 factors
12 → Factors: {1, 2, 3, 4, 6, 12} ✓ More than 2 factors
```

### Composite Numbers Can Be "Broken Down"
```
12 = 2 × 6 = 2 × 2 × 3
20 = 4 × 5 = 2 × 2 × 5
15 = 3 × 5

Composite numbers are products of smaller factors!
```

---

## 4. Special Cases: 0 and 1

### Why is 1 NOT Prime?
```
1 → Factors: {1}    ← Only ONE factor!

By definition, primes have EXACTLY two factors.
1 has only one factor (itself).
Therefore, 1 is NEITHER prime NOR composite.

Also, if 1 were prime, prime factorization 
wouldn't be unique:
12 = 2 × 2 × 3 = 1 × 2 × 2 × 3 = 1 × 1 × 2 × 2 × 3 ...
```

### Why is 0 NOT Prime or Composite?
```
0 is divisible by EVERY non-zero number!
0 ÷ 2 = 0
0 ÷ 5 = 0
0 ÷ 100 = 0

0 has infinitely many factors.
0 is NEITHER prime NOR composite.
Also, 0 is not greater than 1.
```

### Classification Summary
```
┌─────────────────────────────────────────────────────────┐
│                CLASSIFICATION OF NUMBERS                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  0  →  Neither prime nor composite (special case)       │
│                                                         │
│  1  →  Neither prime nor composite (special case)       │
│                                                         │
│  2, 3, 5, 7, 11, 13, ...  →  PRIME NUMBERS             │
│                                                         │
│  4, 6, 8, 9, 10, 12, ...  →  COMPOSITE NUMBERS         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 5. The Number 2: Special Prime

### 2 is the Only Even Prime!
```
2 is the SMALLEST and ONLY even prime number.

Why?
• 2 = 1 × 2 (only two factors)
• Every other even number is divisible by 2:
  4 = 2 × 2
  6 = 2 × 3
  8 = 2 × 4
  ...

So all even numbers > 2 have at least three factors:
{1, 2, the number itself}

Therefore, all even numbers > 2 are COMPOSITE.
```

### Odd Primes
```
All primes except 2 are odd:
3, 5, 7, 11, 13, 17, 19, 23, 29, 31, ...

But NOT all odd numbers are prime:
9 = 3 × 3 (composite)
15 = 3 × 5 (composite)
21 = 3 × 7 (composite)
```

---

## 6. Prime Numbers 1 to 100

### The Sieve of Eratosthenes
A method to find all primes up to a given number:

```
Step 1: Write numbers 2 to 100
Step 2: Circle 2, cross out all multiples of 2
Step 3: Circle 3, cross out all multiples of 3
Step 4: Circle 5, cross out all multiples of 5
Step 5: Circle 7, cross out all multiples of 7
Step 6: All remaining numbers are prime!

Why stop at 7? Because √100 = 10, and 7 is the largest prime ≤ 10.
```

### Visual: Sieve of Eratosthenes
```
    1   2   3   4   5   6   7   8   9  10
   11  12  13  14  15  16  17  18  19  20
   21  22  23  24  25  26  27  28  29  30
   31  32  33  34  35  36  37  38  39  40
   41  42  43  44  45  46  47  48  49  50
   51  52  53  54  55  56  57  58  59  60
   61  62  63  64  65  66  67  68  69  70
   71  72  73  74  75  76  77  78  79  80
   81  82  83  84  85  86  87  88  89  90
   91  92  93  94  95  96  97  98  99 100

After sieving:
    ×   ②   ③   ×   ⑤   ×   ⑦   ×   ×   ×
   ⑪   ×  ⑬   ×   ×   ×  ⑰   ×  ⑲   ×
    ×   ×  ㉓   ×   ×   ×   ×   ×  ㉙   ×
   ㉛   ×   ×   ×   ×   ×  ㊲   ×   ×   ×
   ㊶   ×  ㊸   ×   ×   ×  ㊼   ×   ×   ×
    ×   ×  ㊾   ×   ×   ×   ×   ×  ㊿   ×
   61   ×   ×   ×   ×   ×  67   ×   ×   ×
   71   ×  73   ×   ×   ×   ×   ×  79   ×
    ×   ×  83   ×   ×   ×   ×   ×  89   ×
    ×   ×   ×   ×   ×   ×  97   ×   ×   ×
```

### Prime Numbers from 1 to 100
```
2, 3, 5, 7, 11, 13, 17, 19, 23, 29,
31, 37, 41, 43, 47, 53, 59, 61, 67, 71,
73, 79, 83, 89, 97

Total: 25 prime numbers between 1 and 100
```

### Memory Trick: Primes by Tens
```
1-10:   2, 3, 5, 7           (4 primes)
11-20:  11, 13, 17, 19       (4 primes)
21-30:  23, 29               (2 primes)
31-40:  31, 37               (2 primes)
41-50:  41, 43, 47           (3 primes)
51-60:  53, 59               (2 primes)
61-70:  61, 67               (2 primes)
71-80:  71, 73, 79           (3 primes)
81-90:  83, 89               (2 primes)
91-100: 97                   (1 prime)
```

---

## 7. Testing for Primality

### Method 1: Trial Division
Test divisibility by all primes up to √n.

```
Is 97 prime?

√97 ≈ 9.85, so test primes up to 9: {2, 3, 5, 7}

97 ÷ 2 = 48.5 ✗ (not even)
97 ÷ 3 = 32.33 ✗ (digit sum = 16, not divisible by 3)
97 ÷ 5 = 19.4 ✗ (doesn't end in 0 or 5)
97 ÷ 7 = 13.86 ✗

No prime ≤ 9 divides 97, so 97 is PRIME!
```

### Method 2: Quick Elimination
```
Before testing, eliminate obvious composites:

1. Even numbers > 2 → Composite (divisible by 2)
2. Numbers ending in 5 (except 5) → Composite (divisible by 5)
3. Numbers ending in 0 → Composite (divisible by 2 and 5)

So any prime > 5 must end in: 1, 3, 7, or 9
```

### Example: Is 91 Prime?
```
91 ends in 1 → Could be prime

√91 ≈ 9.5, test primes up to 9: {2, 3, 5, 7}

91 ÷ 2 = 45.5 ✗
91 ÷ 3 = 30.33 ✗ (digit sum = 10, not ÷ by 3)
91 ÷ 5 = 18.2 ✗
91 ÷ 7 = 13 ✓

91 = 7 × 13

Therefore, 91 is COMPOSITE, not prime!
```

### Why √n is Sufficient
```
If n = a × b, then at least one of a or b must be ≤ √n.

Example: 36 = 2 × 18 = 3 × 12 = 4 × 9 = 6 × 6

√36 = 6

Pairs: (2,18) (3,12) (4,9) (6,6)
       ↓      ↓      ↓     ↓
       2≤6   3≤6   4≤6   6≤6

In each pair, at least one number is ≤ 6!
```

---

## 8. Properties of Prime Numbers

### Fundamental Theorem of Arithmetic
```
Every integer greater than 1 is either:
• A prime number, OR
• A unique product of prime numbers

Examples:
12 = 2 × 2 × 3 = 2² × 3
30 = 2 × 3 × 5
100 = 2 × 2 × 5 × 5 = 2² × 5²

This factorization is UNIQUE (except for order)!
```

### Infinitely Many Primes
```
There are INFINITELY many prime numbers!

Euclid's proof (over 2000 years old):
Assume there are finitely many primes: p₁, p₂, ..., pₙ
Consider N = (p₁ × p₂ × ... × pₙ) + 1
N is not divisible by any of p₁, p₂, ..., pₙ
So N is either prime or has a prime factor not in our list
Contradiction! Therefore, primes are infinite.
```

### Prime Gaps
```
The gap between consecutive primes varies:

2 and 3:    gap = 1 (only case)
3 and 5:    gap = 2
5 and 7:    gap = 2
7 and 11:   gap = 4
11 and 13:  gap = 2
23 and 29:  gap = 6
89 and 97:  gap = 8

Gaps get larger on average as numbers increase,
but there are always more primes ahead!
```

### Twin Primes
```
Twin primes are pairs that differ by 2:
(3, 5), (5, 7), (11, 13), (17, 19), (29, 31), 
(41, 43), (59, 61), (71, 73), ...

It's conjectured (but not proven) that there are 
infinitely many twin primes.
```

---

## 9. Solved Examples

### Example 1: Classify 1 to 20
```
Number  Factors              Classification
1       {1}                  Neither
2       {1, 2}               Prime
3       {1, 3}               Prime
4       {1, 2, 4}            Composite
5       {1, 5}               Prime
6       {1, 2, 3, 6}         Composite
7       {1, 7}               Prime
8       {1, 2, 4, 8}         Composite
9       {1, 3, 9}            Composite
10      {1, 2, 5, 10}        Composite
11      {1, 11}              Prime
12      {1, 2, 3, 4, 6, 12}  Composite
13      {1, 13}              Prime
14      {1, 2, 7, 14}        Composite
15      {1, 3, 5, 15}        Composite
16      {1, 2, 4, 8, 16}     Composite
17      {1, 17}              Prime
18      {1, 2, 3, 6, 9, 18}  Composite
19      {1, 19}              Prime
20      {1, 2, 4, 5, 10, 20} Composite
```

### Example 2: Is 127 Prime?
```
√127 ≈ 11.27
Test primes up to 11: {2, 3, 5, 7, 11}

127 is odd → not divisible by 2
1 + 2 + 7 = 10 → not divisible by 3
Ends in 7 → not divisible by 5
127 ÷ 7 = 18.14 → not divisible by 7
127 ÷ 11 = 11.55 → not divisible by 11

127 is PRIME!
```

### Example 3: Find All Primes Between 50 and 70
```
Candidates (odd, not ending in 5): 51, 53, 57, 59, 61, 63, 67, 69

51 = 3 × 17     ✗ Composite
53              ✓ Prime (check: not ÷ by 2,3,5,7)
57 = 3 × 19     ✗ Composite
59              ✓ Prime (check: not ÷ by 2,3,5,7)
61              ✓ Prime (check: not ÷ by 2,3,5,7)
63 = 7 × 9      ✗ Composite
67              ✓ Prime (check: not ÷ by 2,3,5,7)
69 = 3 × 23     ✗ Composite

Primes between 50 and 70: 53, 59, 61, 67
```

### Example 4: Express 84 as Product of Primes
```
84 = 2 × 42
   = 2 × 2 × 21
   = 2 × 2 × 3 × 7
   = 2² × 3 × 7

Prime factorization: 2² × 3 × 7
```

---

## 10. Mental Math Tips 🧠

### Quick Prime Check for 2-Digit Numbers
```
1. Is it even? → Composite (unless it's 2)
2. Does it end in 5? → Composite (unless it's 5)
3. Is digit sum ÷ by 3? → Composite
4. Otherwise, test division by 7, 11, etc.
```

### Memorizing Primes
```
First 10 primes: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29
                 Sum = 129

Pattern: After 2, all primes are 6k ± 1
         5 = 6(1) - 1
         7 = 6(1) + 1
         11 = 6(2) - 1
         13 = 6(2) + 1
         17 = 6(3) - 1
         19 = 6(3) + 1
         
(But not all 6k ± 1 are prime: 25 = 6(4) + 1 is not prime)
```

---

## 📊 Summary Table

### Classification Summary

| Number Type | Number of Factors | Examples |
|-------------|-------------------|----------|
| 1 | Exactly 1 | 1 (special) |
| Prime | Exactly 2 | 2, 3, 5, 7, 11, 13 |
| Composite | More than 2 | 4, 6, 8, 9, 10, 12 |

### First 25 Primes

| Position | Prime | Position | Prime |
|----------|-------|----------|-------|
| 1st | 2 | 14th | 43 |
| 2nd | 3 | 15th | 47 |
| 3rd | 5 | 16th | 53 |
| 4th | 7 | 17th | 59 |
| 5th | 11 | 18th | 61 |
| 6th | 13 | 19th | 67 |
| 7th | 17 | 20th | 71 |
| 8th | 19 | 21st | 73 |
| 9th | 23 | 22nd | 79 |
| 10th | 29 | 23rd | 83 |
| 11th | 31 | 24th | 89 |
| 12th | 37 | 25th | 97 |
| 13th | 41 | | |

---

## ❓ Quick Revision Questions

1. **True or False**: 1 is a prime number.

2. **List**: All prime numbers between 20 and 40.

3. **Test**: Is 87 prime or composite?

4. **Why**: Is 2 the only even prime number?

5. **Find**: The smallest composite number.

6. **Determine**: How many prime numbers are there between 1 and 50?

<details>
<summary>Click to see answers</summary>

1. **False** - 1 has only one factor (itself), but primes must have exactly 2 factors.

2. **23, 29, 31, 37** (4 primes between 20 and 40)

3. 8 + 7 = 15, which is divisible by 3
   87 ÷ 3 = 29
   87 = 3 × 29
   **87 is COMPOSITE**

4. Every even number > 2 is divisible by 2. So they all have at least three factors: 1, 2, and the number itself. Only 2 is even and has exactly two factors (1 and 2), making it the **only even prime**.

5. The smallest composite number is **4**
   (4 = 2 × 2, factors: {1, 2, 4})

6. Primes 1-50: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47
   **15 prime numbers**

</details>

---

## 🔗 Navigation

[← Previous: Properties of Operations](../02-Basic-Operations/04-properties-of-operations.md) | [Back to Contents](../README.md) | [Next: Prime Factorization →](02-prime-factorization.md)

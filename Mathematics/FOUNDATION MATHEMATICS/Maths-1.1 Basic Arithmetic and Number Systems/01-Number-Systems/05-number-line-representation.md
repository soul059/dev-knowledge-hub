# Chapter 1.5: Number Line Representation

[← Previous: Real Number System](04-real-number-system.md) | [Back to Contents](../README.md) | [Next: Unit 2 - Addition and Subtraction →](../02-Basic-Operations/01-addition-subtraction.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Understand the structure of the number line
- Represent all types of numbers on a number line
- Perform geometric constructions for irrational numbers
- Use number lines for comparing and ordering numbers
- Visualize operations on the number line

---

## 1. What is a Number Line?

### Definition
A **number line** is a straight line where every point corresponds to exactly one real number, and every real number corresponds to exactly one point.

### Basic Structure
```
                    Negative          Zero          Positive
                    ◄─────────────────────────────────────────►
                    
    ━━━━━╋━━━━━╋━━━━━╋━━━━━╋━━━━━╋━━━━━╋━━━━━╋━━━━━╋━━━━━╋━━━━━➤
        -4    -3    -2    -1     0     1     2     3     4
                                 ↑
                             Origin
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Origin** | The point representing zero (0) |
| **Positive direction** | Points to the right of origin |
| **Negative direction** | Points to the left of origin |
| **Unit length** | The distance between consecutive integers |
| **Scale** | The chosen unit for measurement |

---

## 2. Representing Different Types of Numbers

### Natural Numbers on the Number Line
```
Natural Numbers: {1, 2, 3, 4, 5, ...}

    0     1     2     3     4     5     6     7     8
    ○─────●─────●─────●─────●─────●─────●─────●─────●─────➤
          ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑
        Start of Natural Numbers
    
    ○ = Not included (0 is not natural)
    ● = Included
```

### Whole Numbers on the Number Line
```
Whole Numbers: {0, 1, 2, 3, 4, 5, ...}

    0     1     2     3     4     5     6     7     8
    ●─────●─────●─────●─────●─────●─────●─────●─────●─────➤
    ↑
    0 is included
```

### Integers on the Number Line
```
Integers: {..., -3, -2, -1, 0, 1, 2, 3, ...}

◄─────●─────●─────●─────●─────●─────●─────●─────●─────●─────➤
     -4    -3    -2    -1     0     1     2     3     4
      ↑                       ↑                       ↑
   Negative              Origin                   Positive
   Integers                                      Integers

Integers extend infinitely in BOTH directions
```

### Rational Numbers on the Number Line

Rational numbers are represented by dividing unit segments into equal parts.

**Example**: Represent 3/4
```
Step 1: 3/4 lies between 0 and 1
Step 2: Divide [0,1] into 4 equal parts
Step 3: Count 3 parts from 0

    0              1/4            2/4            3/4             1
    |───────────────|───────────────|───────────────|───────────────|
    0             0.25            0.5            0.75             1
                                                  ●
                                                 3/4
```

**Example**: Represent -5/3
```
Step 1: -5/3 = -1 2/3, lies between -2 and -1
Step 2: Divide each unit into 3 parts
Step 3: Count 5 parts from 0 in negative direction

   -2        -5/3       -4/3        -1        -2/3       -1/3         0
    |──────────●──────────|──────────|──────────|──────────|──────────|
   -6/3      -5/3       -4/3       -3/3       -2/3       -1/3        0/3
              ●
            -5/3 ≈ -1.67
```

### Irrational Numbers on the Number Line

Irrational numbers require geometric constructions.

---

## 3. Constructing √2 on the Number Line

### Method: Pythagorean Theorem

**Principle**: In a right triangle, if two legs are 1 unit each, the hypotenuse is √2.

```
1² + 1² = (√2)²
1 + 1 = 2 ✓
```

### Step-by-Step Construction
```
Step 1: Draw a number line and mark 0 and 1
        
    0                              1
    ●──────────────────────────────●

Step 2: At point 1, draw a perpendicular line of length 1 unit
        
                                   B
                                   │
                                   │ 1 unit
                                   │
    0                              1(A)
    ●──────────────────────────────●

Step 3: Connect point 0 to B (this is the hypotenuse = √2)
        
                                   B
                                  ╱│
                           √2   ╱  │ 1
                              ╱    │
    0                       ╱     1(A)
    ●─────────────────────◢───────●
    
Step 4: Using compass, with center at 0 and radius OB, 
        draw an arc to cut the number line
        
                                   B
                                  ╱│
                           √2   ╱  │ 1
                              ╱    │
    0                       ╱     1(A)      √2
    ●─────────────────────◢───────●─────────●
                         ╲                  ↑
                          ╲________________╱
                              Arc with radius √2
```

### Final Result
```
    0                    1                   √2                  2
    ●────────────────────●────────────────────●────────────────────●
                                            ≈1.414
```

---

## 4. Constructing √3 on the Number Line

### Method: Build on √2

**Principle**: Create a right triangle with legs 1 and √2.

```
1² + (√2)² = (√3)²
1 + 2 = 3 ✓
```

### Step-by-Step Construction
```
Step 1: Start with √2 already on the number line

    0                    1                   √2
    ●────────────────────●────────────────────●

Step 2: At √2, draw a perpendicular of length 1
        
                                              C
                                              │
                                              │ 1
                                              │
    0                    1                   √2
    ●────────────────────●────────────────────●

Step 3: Connect 0 to C (hypotenuse = √3)
        
                                              C
                                             ╱│
                                      √3   ╱  │ 1
                                         ╱    │
    0                    1              ╱    √2
    ●────────────────────●────────────◢──────●

Step 4: Arc from 0 with radius = √3
        
    0         1         √2        √3          2
    ●─────────●──────────●─────────●───────────●
                                 ≈1.732
```

---

## 5. Constructing √n on the Number Line

### General Method: Square Root Spiral

For any positive integer n, we can construct √n using the **Spiral of Theodorus**.

```
Each successive hypotenuse has length √2, √3, √4, √5, ...

                    √5  √6
                   ╱   ╱
              √4 ╱   ╱  
             ╱  ╱   ╱    1
        √3 ╱  ╱   ╱     ╱
       ╱  ╱  ╱   ╱     ╱
  √2 ╱  ╱  ╱   ╱     ╱
   ╱  ╱  ╱   ╱ 1   ╱ 1
  ╱  ╱  ╱   ╱     ╱
 ╱  ╱  ╱   ╱     ╱
◢──◢──◢───◢─────◢
 1  1   1    1
```

### Construction for √5
```
Method: Use right triangle with legs 1 and 2
        1² + 2² = 1 + 4 = 5
        Hypotenuse = √5

    0         1         2        √5          3
    ●─────────●─────────●─────────●───────────●
                                ≈2.236
```

### Alternative Method for √5
```
Right triangle with legs √4 = 2 and 1:
2² + 1² = 4 + 1 = 5
Hypotenuse = √5
```

---

## 6. Locating Decimal Numbers

### Terminating Decimals
```
Represent 2.7 on the number line:

Step 1: 2.7 lies between 2 and 3
Step 2: Divide [2,3] into 10 equal parts
Step 3: Count 7 parts from 2

    2    2.1  2.2  2.3  2.4  2.5  2.6  2.7  2.8  2.9   3
    |─────|────|────|────|────|────|────|────|────|─────|
                                         ●
                                        2.7
```

### More Precision: 3.14159 (approximation of π)
```
Step 1: Between 3 and 4
Step 2: Between 3.1 and 3.2 (more precisely)
Step 3: Between 3.14 and 3.15

    3.0        3.1       3.14      3.15       3.2        3.3
    |──────────|──────────|─────●────|──────────|──────────|
                               π ≈ 3.14159
```

---

## 7. Comparing Numbers on the Number Line

### Key Principle
```
On a number line going left to right:
• Numbers to the RIGHT are GREATER
• Numbers to the LEFT are SMALLER
```

### Visual Comparison
```
Compare: -2, √3, 0.5, -1.5, 2

   -2      -1.5     -1       0      0.5      1      √3       2
    ●────────●───────|───────●───────●───────|───────●───────●
    ↑        ↑               ↑       ↑               ↑       ↑
  Smallest                                                 Largest

Order: -2 < -1.5 < 0 < 0.5 < √3 < 2
       (√3 ≈ 1.732)
```

### Comparing Fractions
```
Compare: 2/3, 3/4, 5/8

Convert to decimals or common denominator:
2/3 ≈ 0.667
3/4 = 0.75
5/8 = 0.625

    0       5/8     2/3     3/4      1
    |────────●───────●───────●───────|
           0.625   0.667   0.75

Order: 5/8 < 2/3 < 3/4
```

---

## 8. Operations on the Number Line

### Addition Using Number Line
```
Calculate: 3 + (-5)

Start at 3, move 5 units LEFT (negative direction)

   -3    -2    -1     0     1     2     3     4
    |─────|─────|─────|─────|─────|─────|─────|
    ●←─────────────────────────────────●
   -2                                  Start at 3
         ◄───────────────────────────
               Move 5 units left

3 + (-5) = -2
```

### Subtraction Using Number Line
```
Calculate: 2 - 5

Start at 2, move 5 units LEFT

   -4    -3    -2    -1     0     1     2     3
    |─────|─────|─────|─────|─────|─────|─────|
              ●←─────────────────────────●
             -3                        Start at 2
              ◄─────────────────────────
                   Move 5 units left

2 - 5 = -3
```

### Multiplication Visualization
```
Calculate: 3 × 2 (3 groups of 2)

    0     1     2     3     4     5     6     7
    |─────|─────|─────|─────|─────|─────|─────|
    ○───────────●     ○───────────●     ●
         2                 2           Final
    ├─────────────────────────────────→
              3 jumps of 2 = 6

3 × 2 = 6
```

### Absolute Value on Number Line
```
|x| = Distance from 0

    -5    -4    -3    -2    -1     0     1     2     3     4     5
     |─────|─────|─────|─────|─────|─────|─────|─────|─────|─────|
     ◄─────────────────────────────●─────────────────────────────►
                    5 units        │        5 units
                                   
    |-5| = 5                      |5| = 5
    Both are 5 units from 0
```

---

## 9. Number Line for Inequalities

### Simple Inequalities
```
x > 2 (x is greater than 2)

   -1     0     1     2     3     4     5
    |─────|─────|─────○═════●═════●═════●═════➤
                      ↑
                Open circle (2 not included)
                Arrow shows "extends to infinity"
```

```
x ≤ 1 (x is less than or equal to 1)

◄═════●═════●═════●═════●─────|─────|─────|
     -2    -1     0     1     2     3     4
                        ↑
                Closed circle (1 IS included)
```

### Compound Inequalities
```
-2 < x ≤ 3

   -4    -3    -2    -1     0     1     2     3     4
    |─────|─────○═════●═════●═════●═════●═════●─────|
                ↑                               ↑
          Open (not included)           Closed (included)
```

```
x < -1 OR x ≥ 2

   -4    -3    -2    -1     0     1     2     3     4
══════●═════●═════●═════○─────|─────○═════●═════●═════➤
                        ↑           ↑
              Not included    Included (≥)
```

---

## 10. Distance on the Number Line

### Formula
The **distance** between two points a and b on the number line is:
```
Distance = |a - b| = |b - a|
```

### Examples
```
Distance between 3 and 7:
|3 - 7| = |-4| = 4

    0     1     2     3     4     5     6     7     8
    |─────|─────|─────●═══════════════════════●─────|
                      ├───────────────────────┤
                            4 units
```

```
Distance between -2 and 5:
|-2 - 5| = |-7| = 7

   -3    -2    -1     0     1     2     3     4     5     6
    |─────●═══════════════════════════════════════════●─────|
          ├───────────────────────────────────────────┤
                          7 units
```

```
Distance between -4 and -1:
|-4 - (-1)| = |-4 + 1| = |-3| = 3

   -5    -4    -3    -2    -1     0     1
    |─────●═══════════════════●─────|─────|
          ├───────────────────┤
                3 units
```

### Midpoint Formula
The **midpoint** between two points a and b is:
```
Midpoint = (a + b) / 2
```

**Example**: Midpoint between -2 and 6
```
Midpoint = (-2 + 6) / 2 = 4 / 2 = 2

   -3    -2    -1     0     1     2     3     4     5     6     7
    |─────●═══════════════════════════●═══════════════════════●─────|
          ├───────────────────────────┤
                    4 units           Midpoint = 2
          ├─────────────────────────────────────────────────┤
                              8 units total
```

---

## 11. Solved Examples

### Example 1: Represent -5/4 and 7/4 on a number line

**Solution**:
```
-5/4 = -1.25 (between -2 and -1)
7/4 = 1.75 (between 1 and 2)

   -2     -5/4    -1    -1/2     0    1/2     1     7/4     2
    |───────●──────|──────|──────|─────|──────|───────●──────|
         -1.25                                       1.75
```

### Example 2: Show √10 on a number line

**Solution**:
```
Method: Use right triangle with legs 3 and 1
        3² + 1² = 9 + 1 = 10
        Hypotenuse = √10

Step 1: Mark 3 on the number line
Step 2: Draw perpendicular of length 1 at point 3
Step 3: Connect to 0 to get length √10
Step 4: Use compass to mark √10

    0         1         2         3        √10        4
    ●─────────●─────────●─────────●──────────●─────────●
                                           ≈3.162

Verification: 3² < 10 < 4² (9 < 10 < 16)
So √10 is between 3 and 4 ✓
```

### Example 3: Find three rational numbers between 1/4 and 1/3

**Solution**:
```
1/4 = 0.25 = 3/12
1/3 = 0.333... = 4/12

Need more divisions. Use LCD = 60:
1/4 = 15/60
1/3 = 20/60

Three rationals between them:
16/60 = 4/15 ≈ 0.267
17/60 ≈ 0.283
18/60 = 3/10 = 0.3
19/60 ≈ 0.317

Number Line:
    1/4                      1/3
    0.25                     0.333
    |─────●─────●─────●─────●─────|
        4/15  17/60  3/10  19/60
```

### Example 4: Represent the solution set of |x - 1| ≤ 2

**Solution**:
```
|x - 1| ≤ 2
-2 ≤ x - 1 ≤ 2
-2 + 1 ≤ x ≤ 2 + 1
-1 ≤ x ≤ 3

   -2    -1     0     1     2     3     4
    |─────●═════●═════●═════●═════●─────|
          ↑                       ↑
    Closed circle          Closed circle
    (included)             (included)

Solution: [-1, 3] or {x ∈ ℝ : -1 ≤ x ≤ 3}
```

### Example 5: Find the distance and midpoint between -3.5 and 2.5

**Solution**:
```
Distance = |-3.5 - 2.5| = |-6| = 6 units

Midpoint = (-3.5 + 2.5) / 2 = -1 / 2 = -0.5

   -4   -3.5   -3   -2.5   -2   -1.5   -1   -0.5    0    0.5    1    1.5    2    2.5    3
    |─────●─────|─────|─────|─────|─────|─────●─────|─────|─────|─────|─────|─────●─────|
          ├──────────────────────────────────────────────────────────────────────┤
                                    6 units
                                         ●
                                     Midpoint
                                     = -0.5
```

---

## 12. Mental Math Tricks 🧠

### Trick 1: Quick Location of √n
```
To estimate √n, find perfect squares around it:
√50 → 49 < 50 < 64 → 7 < √50 < 8 → closer to 7

√n is between √(a²) and √(b²) where a² < n < b²
```

### Trick 2: Converting Fractions to Decimal Position
```
1/2 = 0.5 → halfway between 0 and 1
1/4 = 0.25 → quarter of the way
3/4 = 0.75 → three-quarters
1/3 ≈ 0.33 → one-third
2/3 ≈ 0.67 → two-thirds
```

### Trick 3: Quick Distance Calculation
```
Distance between negatives: Add absolute values if opposite sides of 0
Distance from -3 to 4: |-3| + |4| = 3 + 4 = 7

Distance between same-sign numbers: Subtract
Distance from 2 to 7: |7 - 2| = 5
Distance from -5 to -2: |-2 - (-5)| = |-2 + 5| = 3
```

### Trick 4: Visualizing Negative Numbers
```
Think of debt (negative) vs savings (positive)

-5 is 5 units of debt
+3 is 3 units of savings
Distance = Total change needed = 8 units
```

---

## 📊 Summary Table

### Number Types on the Number Line

| Number Type | Starting Point | Direction | Example Points |
|-------------|----------------|-----------|----------------|
| Natural (ℕ) | 1 | Right only | 1, 2, 3, 4, ... |
| Whole (W) | 0 | Right only | 0, 1, 2, 3, ... |
| Integer (ℤ) | 0 | Both | ..., -2, -1, 0, 1, 2, ... |
| Rational (ℚ) | Between integers | Both | 1/2, -3/4, 5/3 |
| Irrational | Special construction | Both | √2, π, e |
| Real (ℝ) | Every point | Both | All of the above |

### Key Formulas

| Concept | Formula | Example |
|---------|---------|---------|
| Distance | \|a - b\| | Distance from -3 to 5 = 8 |
| Midpoint | (a + b)/2 | Midpoint of -2 and 6 = 2 |
| Absolute Value | Distance from 0 | \|-7\| = 7 |

### Inequality Symbols on Number Line

| Symbol | Meaning | Circle Type |
|--------|---------|-------------|
| < | Less than | Open ○ |
| > | Greater than | Open ○ |
| ≤ | Less than or equal | Closed ● |
| ≥ | Greater than or equal | Closed ● |

---

## ❓ Quick Revision Questions

1. **Locate**: Mark √7 on a number line (approximate position).

2. **Calculate**: What is the distance between -4.5 and 3.5?

3. **Find**: The midpoint between -6 and 10.

4. **Draw**: The number line representation of -3 ≤ x < 2.

5. **Construct**: Explain how to locate √13 on a number line.

6. **Compare**: Arrange on a number line: -√2, -1, 0, √3, 2, -2.5

<details>
<summary>Click to see answers</summary>

1. √7 ≈ 2.646 (since 2² = 4 < 7 < 9 = 3², √7 is between 2 and 3, closer to 3)
   ```
   0     1     2     √7    3
   |─────|─────|──────●────|
                    ≈2.65
   ```

2. Distance = |-4.5 - 3.5| = |-8| = **8 units**

3. Midpoint = (-6 + 10)/2 = 4/2 = **2**

4. **Number line for -3 ≤ x < 2**:
   ```
   -4   -3   -2   -1    0    1    2    3
    |────●════●════●════●════●════○────|
         ↑                        ↑
      Closed                    Open
   ```

5. **To construct √13**:
   - Use right triangle with legs 2 and 3 (since 2² + 3² = 4 + 9 = 13)
   - Draw perpendicular of length 3 at point 2
   - Connect to origin to get √13
   - Use compass to transfer this length to number line
   - √13 ≈ 3.606

6. **Order from left to right**:
   -2.5 < -√2 < -1 < 0 < √3 < 2
   (-2.5) (-1.41) (-1) (0) (1.73) (2)
   ```
   -2.5  -√2   -1    0    √3    2
    ●─────●─────●────●─────●─────●
   ```

</details>

---

## 🎓 Unit 1 Complete!

Congratulations! You've completed Unit 1: Number Systems. You now understand:
- Natural, Whole, and Integer numbers
- Rational and Irrational numbers
- The complete Real Number System
- How to represent all numbers on a number line

---

## 🔗 Navigation

[← Previous: Real Number System](04-real-number-system.md) | [Back to Contents](../README.md) | [Next: Unit 2 - Addition and Subtraction →](../02-Basic-Operations/01-addition-subtraction.md)

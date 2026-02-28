# Chapter 1.4: Trigonometric Tables

## Overview

Before calculators became widespread, trigonometric tables were essential tools for computing values of trigonometric functions. Understanding how to read and use these tables provides insight into the nature of trigonometric functions and is still useful for quick estimations and exam situations where calculators may not be permitted.

---

## 📚 What Are Trigonometric Tables?

Trigonometric tables are pre-computed lists of trigonometric function values for various angles. They typically provide values for:
- Sine, Cosine, Tangent (primary functions)
- Angles from 0° to 90° (other quadrants derived from these)
- Values to 4 decimal places

### Historical Context

```
    ┌─────────────────────────────────────────────────────────┐
    │                  HISTORICAL TIMELINE                     │
    ├─────────────────────────────────────────────────────────┤
    │  ~150 CE    Ptolemy creates first known trig tables      │
    │  ~500 CE    Indian mathematicians develop sine tables    │
    │  ~1400 CE   Al-Kashi computes tables to 8 decimal places │
    │  1624       Henry Briggs publishes 14-decimal tables     │
    │  1970s      Scientific calculators replace tables        │
    └─────────────────────────────────────────────────────────┘
```

---

## 📊 Structure of Trigonometric Tables

### Standard Table Format

A typical trigonometric table shows:
- Main angle in degrees (rows)
- Minutes (0' to 60') or decimal degrees (columns)
- Mean differences for interpolation

```
    ┌──────┬─────────────────────────────────────────────────┐
    │  θ°  │    0'      6'     12'     18'     24'    ...   │
    ├──────┼─────────────────────────────────────────────────┤
    │  30  │ 0.5000  0.5015  0.5030  0.5045  0.5060   ...   │
    │  31  │ 0.5150  0.5165  0.5180  0.5195  0.5210   ...   │
    │  32  │ 0.5299  0.5314  0.5329  0.5344  0.5358   ...   │
    │  33  │ 0.5446  0.5461  0.5476  0.5490  0.5505   ...   │
    │  ...  │   ...     ...     ...     ...     ...    ...   │
    └──────┴─────────────────────────────────────────────────┘
```

### Natural Sine Table (Partial)

| θ° | 0' | 6' | 12' | 18' | 24' | 30' | 36' | 42' | 48' | 54' |
|----|------|------|------|------|------|------|------|------|------|------|
| 0° | .0000 | .0017 | .0035 | .0052 | .0070 | .0087 | .0105 | .0122 | .0140 | .0157 |
| 10° | .1736 | .1754 | .1771 | .1788 | .1805 | .1822 | .1840 | .1857 | .1874 | .1891 |
| 20° | .3420 | .3437 | .3453 | .3469 | .3486 | .3502 | .3518 | .3535 | .3551 | .3567 |
| 30° | .5000 | .5015 | .5030 | .5045 | .5060 | .5075 | .5090 | .5105 | .5120 | .5135 |
| 40° | .6428 | .6441 | .6455 | .6468 | .6481 | .6494 | .6508 | .6521 | .6534 | .6547 |
| 45° | .7071 | .7083 | .7096 | .7108 | .7120 | .7133 | .7145 | .7157 | .7169 | .7181 |
| 50° | .7660 | .7672 | .7683 | .7694 | .7705 | .7716 | .7727 | .7738 | .7749 | .7760 |
| 60° | .8660 | .8669 | .8678 | .8686 | .8695 | .8704 | .8712 | .8721 | .8729 | .8738 |
| 70° | .9397 | .9403 | .9409 | .9415 | .9421 | .9426 | .9432 | .9438 | .9444 | .9449 |
| 80° | .9848 | .9851 | .9854 | .9857 | .9860 | .9863 | .9866 | .9869 | .9871 | .9874 |
| 90° | 1.000 | - | - | - | - | - | - | - | - | - |

---

## 📖 How to Read Trigonometric Tables

### Step 1: Identify the Angle

Convert the angle to degrees and minutes if necessary.

**Example:** 32.5° = 32°30'

### Step 2: Locate the Row

Find the row corresponding to the whole degree part.

### Step 3: Locate the Column

Find the column corresponding to the minutes.

### Step 4: Read the Value

The intersection gives the trigonometric value.

```
    Reading sin 32°30':
    
    1. Find row for 32°
    2. Find column for 30'
    3. Read the value
    
    ┌──────┬───────┬───────┬───────┬───────┬───────┐
    │  θ°  │  24'  │  30'  │  36'  │  42'  │  48'  │
    ├──────┼───────┼───────┼───────┼───────┼───────┤
    │  31  │ .5210 │ .5225 │ .5240 │ .5255 │ .5270 │
    │  32  │ .5358 │→.5373←│ .5388 │ .5402 │ .5417 │
    │  33  │ .5505 │ .5519 │ .5534 │ .5548 │ .5563 │
    └──────┴───────┴───────┴───────┴───────┴───────┘
    
    sin 32°30' = 0.5373
```

---

## 🔢 Interpolation Using Mean Differences

When the exact angle isn't in the table, use **interpolation** with mean differences.

### Mean Difference Table

```
    ┌────────────────────────────────────────────────────┐
    │  Mean Differences (per minute) for sin θ           │
    ├──────┬─────────────────────────────────────────────┤
    │  θ°  │   1'    2'    3'    4'    5'               │
    ├──────┼─────────────────────────────────────────────┤
    │  30  │   3     5     8    10    13                │
    │  31  │   3     5     8    10    13                │
    │  32  │   2     5     7    10    12                │
    │  33  │   2     5     7    10    12                │
    └──────┴─────────────────────────────────────────────┘
    
    Values shown are in units of 0.0001
```

### Interpolation Method

For angle θ° m' where m is not a multiple of 6:

1. Find the nearest tabulated value (round down to nearest 6')
2. Calculate the extra minutes
3. Multiply extra minutes by mean difference
4. Add to the tabulated value

**Example: Find sin 32°34'**

```
    Step 1: sin 32°30' = 0.5373 (from table)
    
    Step 2: Extra minutes = 34' - 30' = 4'
    
    Step 3: Mean difference for 4' at 32° = 10 (× 0.0001)
    
    Step 4: sin 32°34' = 0.5373 + 0.0010 = 0.5383
```

---

## 📐 Reading Different Function Tables

### Sine Table (Direct Reading)
- Read directly from the natural sine table
- Values increase from 0 to 1 as angle increases from 0° to 90°

### Cosine Table (Complementary Reading)
Most tables don't have separate cosine tables. Use the relationship:
$$\cos \theta = \sin(90° - \theta)$$

```
    To find cos 32°:
    cos 32° = sin(90° - 32°) = sin 58°
    
    Read sin 58° from the sine table
```

Alternatively, cosine tables read **from bottom to top** and **right to left**.

### Tangent Table
- Separate tables for tangent
- Values increase from 0 to ∞ as angle increases from 0° to 90°
- Near 45°, tan θ ≈ 1
- For angles > 45°, use: tan θ = 1/tan(90° - θ)

### Natural Tangent Table (Partial)

| θ° | 0' | 6' | 12' | 18' | 24' | 30' |
|----|------|------|------|------|------|------|
| 0° | .0000 | .0017 | .0035 | .0052 | .0070 | .0087 |
| 15° | .2679 | .2698 | .2717 | .2736 | .2754 | .2773 |
| 30° | .5774 | .5797 | .5820 | .5844 | .5867 | .5890 |
| 45° | 1.0000 | 1.0035 | 1.0070 | 1.0105 | 1.0141 | 1.0176 |
| 60° | 1.7321 | 1.7391 | 1.7461 | 1.7532 | 1.7603 | 1.7675 |
| 75° | 3.7321 | 3.7760 | 3.8208 | 3.8667 | 3.9136 | 3.9617 |

---

## ↩️ Finding Angles from Ratios (Inverse Lookup)

To find an angle when given a trigonometric value:

### Method

1. Search the table for the given value
2. Find the closest match
3. Determine the row (degrees) and column (minutes)
4. Use interpolation for more precision

### Example: Find θ if sin θ = 0.5446

```
    Search in sine table:
    
    ┌──────┬───────┬───────┬───────┐
    │  θ°  │   0'  │   6'  │  12'  │
    ├──────┼───────┼───────┼───────┤
    │  32  │ .5299 │ .5314 │ .5329 │
    │  33  │→.5446←│ .5461 │ .5476 │
    │  34  │ .5592 │ .5606 │ .5621 │
    └──────┴───────┴───────┴───────┘
    
    sin 33°0' = 0.5446
    
    Therefore, θ = 33°
```

---

## 🧮 Worked Examples

### Example 1: Direct Table Reading

Find sin 25°18' using the table.

**Solution:**
From a standard sine table:
- Row: 25°
- Column: 18'
- Value: sin 25°18' = **0.4274**

### Example 2: Using Interpolation

Find sin 47°23' given:
- sin 47°18' = 0.7353
- Mean difference for 1' = 2 (× 0.0001)

**Solution:**
Extra minutes = 23' - 18' = 5'
Correction = 5 × 2 × 0.0001 = 0.0010
sin 47°23' = 0.7353 + 0.0010 = **0.7363**

### Example 3: Finding Cosine

Find cos 37° using sine tables.

**Solution:**
cos 37° = sin(90° - 37°) = sin 53°

From sine table: sin 53° = 0.7986

Therefore, cos 37° = **0.7986**

### Example 4: Inverse Lookup

If cos θ = 0.8290, find θ.

**Solution:**
Since cos θ = sin(90° - θ):
We need sin(90° - θ) = 0.8290

Search sine table for 0.8290:
sin 56° = 0.8290 (approximately)

So 90° - θ = 56°
θ = 90° - 56° = **34°**

### Example 5: Using Tangent Tables

Find tan 52°24'.

**Solution:**
From tangent table (or using tan θ = sin θ/cos θ):
- sin 52°24' ≈ 0.7916
- cos 52°24' ≈ 0.6111
- tan 52°24' = 0.7916/0.6111 = **1.2954**

Or read directly from tangent table: tan 52°24' ≈ **1.2954**

---

## ⚠️ Common Sources of Error

### 1. Reading Wrong Row or Column
Always double-check the degree and minute values.

### 2. Forgetting Mean Difference Addition
For non-tabulated values, interpolation is necessary.

### 3. Incorrect Complementary Reading
When using sine tables for cosine, remember: cos θ = sin(90° - θ)

### 4. Sign Errors for Large Angles
Tables only cover 0° to 90°. For other quadrants, adjust signs appropriately.

```
    Error Prevention Checklist:
    
    □ Verified the correct function (sin/cos/tan)
    □ Checked angle is in correct format
    □ Located correct row (degrees)
    □ Located correct column (minutes)
    □ Applied interpolation if needed
    □ Double-checked the final answer
```

---

## 🔄 Converting Between Functions

### Using Tables for All Six Functions

| Function | How to Find Using Sine/Tangent Tables |
|----------|--------------------------------------|
| sin θ | Direct from sine table |
| cos θ | sin(90° - θ) |
| tan θ | Direct from tangent table, or sin θ / cos θ |
| cot θ | tan(90° - θ), or 1/tan θ |
| sec θ | 1/cos θ = 1/sin(90° - θ) |
| csc θ | 1/sin θ |

---

## 🌍 Real-World Applications

### 1. Surveying
Before electronic equipment, surveyors used trigonometric tables with theodolites to calculate distances and elevations.

### 2. Navigation
Maritime and aerial navigation historically relied on trig tables for celestial navigation calculations.

### 3. Engineering
Structural engineers used tables for stress and force calculations before computer-aided design.

### 4. Astronomy
Astronomers calculated planetary positions using extensive trigonometric tables.

---

## 📋 Summary Table

| Concept | Key Point |
|---------|-----------|
| Table structure | Rows = degrees, Columns = minutes |
| Sine reading | Direct from table, values 0 to 1 |
| Cosine reading | Use cos θ = sin(90° - θ) |
| Tangent reading | Direct from tan table, or sin θ/cos θ |
| Interpolation | Value = Table value + (extra min × mean diff) |
| Inverse lookup | Search table, find row and column |
| Accuracy | Typically 4 decimal places |
| Range | Tables cover 0° to 90°; use identities for other quadrants |

### Quick Reference for Table Reading

| Task | Method |
|------|--------|
| sin θ | Direct reading |
| cos θ | Read sin(90° - θ) |
| tan θ | Direct or sin θ ÷ cos θ |
| Find angle from sin | Inverse lookup in table |
| Find angle from cos | Inverse lookup using sin(90° - θ) |

---

## ❓ Quick Revision Questions

1. **Using the sine table, find sin 42°36' if sin 42°30' = 0.6756 and the mean difference for 6' is 12 (× 0.0001).**

2. **Explain how to find cos 28° using only a sine table.**

3. **If sin θ = 0.6691, use inverse lookup to find θ (given sin 42° = 0.6691).**

4. **Why do most standard tables only cover angles from 0° to 90°?**

5. **Find tan 63° if sin 63° = 0.8910 and cos 63° = 0.4540.**

6. **Using interpolation, find sin 35°43' if sin 35°42' = 0.5835 and the mean difference for 1' is 2 (× 0.0001).**

<details>
<summary>Click to see answers</summary>

1. sin 42°36' = 0.6756 + (6 × 12 × 0.0001)/6 = 0.6756 + 0.0012 = **0.6768**  
   (Note: If difference is already for 6', just add it directly)

2. cos 28° = sin(90° - 28°) = sin 62°. Look up sin 62° in the sine table to get **cos 28° ≈ 0.8829**

3. Since sin 42° = 0.6691, **θ = 42°**

4. Because all other angles can be found using:
   - Complementary relationships (cos θ = sin(90° - θ))
   - Quadrant rules (ASTC for signs)
   - Reference angles

5. tan 63° = sin 63° / cos 63° = 0.8910 / 0.4540 = **1.9626**

6. sin 35°43' = 0.5835 + (1 × 2 × 0.0001) = 0.5835 + 0.0002 = **0.5837**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 1.3 Standard Angles](03-standard-angles.md) | [Unit 1 Index](README.md) | [Unit 2: Unit Circle →](../02-Unit-Circle/README.md) |

---

[← Back to Main Index](../README.md)

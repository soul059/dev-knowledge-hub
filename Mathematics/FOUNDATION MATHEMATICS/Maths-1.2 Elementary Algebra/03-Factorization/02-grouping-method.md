# Chapter 3.2: Grouping Method

[← Previous: Common Factor Method](01-common-factor-method.md) | [Back to Contents](../README.md) | [Next: Factoring Quadratic Trinomials →](03-factoring-quadratic-trinomials.md)

---

## 📚 Chapter Overview

The **Grouping Method** (also called factoring by grouping) is a technique used primarily for polynomials with four or more terms. By strategically grouping terms and factoring out common factors from each group, we can reveal a common binomial factor.

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Recognize when to apply the grouping method
- Group terms strategically to reveal common factors
- Factor polynomials with four terms
- Handle regrouping when initial grouping fails
- Apply grouping to factor special expressions

---

## 1. Introduction to Grouping

### When to Use the Grouping Method

```
┌─────────────────────────────────────────────────────────────────────┐
│            WHEN TO USE GROUPING METHOD                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Use the grouping method when:                                    │
│                                                                     │
│   1. The polynomial has 4 (or more) terms                          │
│   2. There is no common factor among ALL terms                     │
│   3. Pairs of terms share common factors                           │
│                                                                     │
│   Examples:                                                         │
│   • ax + ay + bx + by  (4 terms, pairs share factors)             │
│   • x³ + x² + x + 1    (4 terms, can be grouped)                  │
│   • 2x³ - 4x² + 3x - 6 (4 terms)                                  │
│                                                                     │
│   NOT for:                                                          │
│   • x² + 5x + 6  (3 terms → use trinomial factoring)              │
│   • x² - 9       (2 terms → difference of squares)                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Grouping Process

### Step-by-Step Method

```
┌─────────────────────────────────────────────────────────────────────┐
│              FACTORING BY GROUPING                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Example: Factor ax + ay + bx + by                                │
│                                                                     │
│   Step 1: Group terms in pairs                                     │
│           (ax + ay) + (bx + by)                                    │
│                                                                     │
│   Step 2: Factor out GCF from each group                          │
│           a(x + y) + b(x + y)                                      │
│           ▲          ▲                                              │
│           └────┬─────┘                                              │
│                │                                                    │
│           Same binomial factor!                                    │
│                                                                     │
│   Step 3: Factor out the common binomial                          │
│           (x + y)(a + b)                                           │
│                                                                     │
│   Result: ax + ay + bx + by = (x + y)(a + b)                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Visual Representation

```
        ax + ay + bx + by
            ↙      ↘
    (ax + ay)  +  (bx + by)      ← Group into pairs
         ↓            ↓
     a(x + y)  +  b(x + y)       ← Factor each group
              ↘    ↙
            (x + y)              ← Common binomial
              ↓
        (x + y)(a + b)           ← Final answer
```

---

## 3. The Key Principle

### Why Grouping Works

The grouping method is based on the **distributive property** applied in reverse:

$$a \cdot c + b \cdot c = c(a + b)$$

When two groups share a common binomial factor, we can factor it out just like a common monomial.

```
┌─────────────────────────────────────────────────────────────────────┐
│              THE GROUPING PRINCIPLE                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   If:    Group 1 = A · (common factor)                            │
│   And:   Group 2 = B · (common factor)                            │
│                                                                     │
│   Then:  Group 1 + Group 2                                         │
│        = A·(common) + B·(common)                                   │
│        = (common factor)(A + B)                                    │
│                                                                     │
│   The common factor can be:                                        │
│   • A monomial: 2x, 3y, x², etc.                                  │
│   • A binomial: (x + 1), (a - b), etc.                            │
│   • Any polynomial expression                                      │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 4. Strategies for Grouping

### Strategy 1: Adjacent Pairing

Group the first two terms and the last two terms.

```
Expression: 2x³ + 6x² + 5x + 15

Group: (2x³ + 6x²) + (5x + 15)
Factor: 2x²(x + 3) + 5(x + 3)
Result: (x + 3)(2x² + 5)
```

### Strategy 2: Rearranging Terms

Sometimes you need to rearrange terms before grouping.

```
Expression: ax + by + bx + ay

Rearrange: ax + bx + ay + by
Group: (ax + bx) + (ay + by)
Factor: x(a + b) + y(a + b)
Result: (a + b)(x + y)
```

### Strategy 3: Handling Negative Signs

When the third term is negative, be careful with signs.

```
Expression: x³ - 2x² - 3x + 6

Group: (x³ - 2x²) + (-3x + 6)
     = (x³ - 2x²) - (3x - 6)    ← Factor out -1 from second group
Factor: x²(x - 2) - 3(x - 2)
Result: (x - 2)(x² - 3)
```

---

## 5. Common Pitfalls and Solutions

### Pitfall 1: Groups Don't Match

```
┌─────────────────────────────────────────────────────────────────────┐
│           WHEN GROUPS DON'T MATCH                                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Problem: x² + 2x + 3x + 6                                        │
│                                                                     │
│   WRONG grouping:                                                  │
│   (x² + 2x) + (3x + 6)                                             │
│   x(x + 2) + 3(x + 2)  ← These match! (x + 2)                     │
│   = (x + 2)(x + 3) ✓                                               │
│                                                                     │
│   WRONG grouping (different pairs):                                │
│   (x² + 3x) + (2x + 6)                                             │
│   x(x + 3) + 2(x + 3)  ← Also match! (x + 3)                      │
│   = (x + 3)(x + 2) ✓                                               │
│                                                                     │
│   Both groupings work! Same final answer.                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Pitfall 2: Factoring Out -1

```
┌─────────────────────────────────────────────────────────────────────┐
│           CREATING A COMMON FACTOR WITH -1                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   Problem: 2x - 2y - ax + ay                                       │
│                                                                     │
│   Group: (2x - 2y) + (-ax + ay)                                    │
│   Factor: 2(x - y) + a(-x + y)                                     │
│                                                                     │
│   Hmm... (x - y) and (-x + y) don't match!                        │
│                                                                     │
│   But: -x + y = -(x - y)                                          │
│                                                                     │
│   So: 2(x - y) + a(-(x - y))                                       │
│     = 2(x - y) - a(x - y)                                          │
│     = (x - y)(2 - a) ✓                                             │
│                                                                     │
│   Key insight: (a - b) = -(b - a)                                  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Important Identity

$$-(a - b) = -a + b = b - a$$

or equivalently:

$$(a - b) = -(b - a)$$

---

## 6. Grouping with More Than Four Terms

### Six Terms Example

```
Expression: ax + ay + az + bx + by + bz

Group into two groups of three:
= (ax + ay + az) + (bx + by + bz)
= a(x + y + z) + b(x + y + z)
= (x + y + z)(a + b)
```

### Alternative: Group in Threes

```
Expression: x² + xy + xz + xy + y² + yz

Rearrange: x² + xy + xy + y² + xz + yz
         = x² + 2xy + y² + z(x + y)
         = (x + y)² + z(x + y)
         = (x + y)(x + y + z)
```

---

## ✏️ Solved Examples

### Example 1: Easy - Basic Grouping

**Problem:** Factor $xy + 2x + 3y + 6$

**Solution:**
```
Step 1: Group in pairs
        (xy + 2x) + (3y + 6)

Step 2: Factor each group
        x(y + 2) + 3(y + 2)
        
Step 3: Factor out common binomial (y + 2)
        (y + 2)(x + 3)

Verification:
(y + 2)(x + 3) = xy + 3y + 2x + 6 = xy + 2x + 3y + 6 ✓

Answer: (y + 2)(x + 3)
```

### Example 2: Easy - With Coefficients

**Problem:** Factor $6x² + 9x + 2x + 3$

**Solution:**
```
Step 1: Group in pairs
        (6x² + 9x) + (2x + 3)

Step 2: Factor each group
        3x(2x + 3) + 1(2x + 3)
        
Step 3: Factor out common binomial
        (2x + 3)(3x + 1)

Answer: (2x + 3)(3x + 1)
```

### Example 3: Medium - Negative Terms

**Problem:** Factor $x³ - 4x² - 3x + 12$

**Solution:**
```
Step 1: Group in pairs
        (x³ - 4x²) + (-3x + 12)

Step 2: Factor each group
        x²(x - 4) + (-3)(x - 4)
        = x²(x - 4) - 3(x - 4)
        
Step 3: Factor out common binomial
        (x - 4)(x² - 3)

Verification:
(x - 4)(x² - 3) = x³ - 3x - 4x² + 12 = x³ - 4x² - 3x + 12 ✓

Answer: (x - 4)(x² - 3)
```

### Example 4: Medium - Needs Rearranging

**Problem:** Factor $ac - bd + ad - bc$

**Solution:**
```
Try grouping as written:
(ac - bd) + (ad - bc)
c(a - ?) + d(a - ?)  ← Doesn't work easily!

Rearrange terms:
ac + ad - bc - bd

Step 1: Group in pairs
        (ac + ad) + (-bc - bd)
        = (ac + ad) - (bc + bd)

Step 2: Factor each group
        a(c + d) - b(c + d)
        
Step 3: Factor out common binomial
        (c + d)(a - b)

Answer: (c + d)(a - b)
```

### Example 5: Hard - Creating Common Factor

**Problem:** Factor $2x - 2y - ax + ay$

**Solution:**
```
Step 1: Group in pairs
        (2x - 2y) + (-ax + ay)

Step 2: Factor each group
        2(x - y) + a(-x + y)
        
Note: (-x + y) = -(x - y), so:
        2(x - y) - a(x - y)
        
Step 3: Factor out common binomial
        (x - y)(2 - a)

Answer: (x - y)(2 - a)
```

### Example 6: Hard - Complete Factoring

**Problem:** Factor completely: $2x³ + 4x² - 2x - 4$

**Solution:**
```
Step 1: Factor out GCF first
        2(x³ + 2x² - x - 2)

Step 2: Apply grouping to the remaining polynomial
        x³ + 2x² - x - 2
        = (x³ + 2x²) + (-x - 2)
        = x²(x + 2) - 1(x + 2)
        = (x + 2)(x² - 1)

Step 3: Factor x² - 1 (difference of squares)
        x² - 1 = (x + 1)(x - 1)

Step 4: Write complete factorization
        2(x + 2)(x + 1)(x - 1)

Answer: 2(x + 2)(x + 1)(x - 1)
```

---

## ❓ Practice Problems

### Easy Level

1. Factor: $xy + 3x + 2y + 6$

2. Factor: $ab + 5a + 3b + 15$

3. Factor: $2x² + 4x + 3x + 6$

### Medium Level

4. Factor: $x³ + 2x² + 5x + 10$

5. Factor: $3xy - 6x - 5y + 10$

6. Factor: $a² - ab - ac + bc$

### Hard Level

7. Factor completely: $3x³ - 9x² - 3x + 9$

8. Factor: $x³ + x² - x - 1$

---

## ✅ Answers to Practice Problems

<details>
<summary>Click to reveal answers</summary>

1. $(y + 3)(x + 2)$

2. $(b + 5)(a + 3)$

3. $(x + 2)(2x + 3)$

4. $(x + 2)(x² + 5)$

5. Group: $(3xy - 6x) - (5y - 10) = 3x(y - 2) - 5(y - 2)$
   **$(y - 2)(3x - 5)$**

6. Rearrange: $a² - ab - ac + bc = a(a-b) - c(a-b)$
   **$(a - b)(a - c)$**

7. Factor out 3: $3(x³ - 3x² - x + 3)$
   Group: $3[x²(x - 3) - 1(x - 3)] = 3(x - 3)(x² - 1)$
   Complete: **$3(x - 3)(x + 1)(x - 1)$**

8. Group: $x²(x + 1) - 1(x + 1) = (x + 1)(x² - 1)$
   Complete: **$(x + 1)(x + 1)(x - 1) = (x + 1)²(x - 1)$**

</details>

---

## 📋 Summary Table

| Scenario | Strategy |
|----------|----------|
| 4 terms, no common GCF | Try grouping first two and last two |
| Groups don't match | Rearrange terms and try again |
| One group has opposite sign | Factor out -1 to create common factor |
| $(a - b)$ vs $(b - a)$ | Remember: $(a - b) = -(b - a)$ |
| More than 4 terms | Group into manageable subgroups |
| GCF exists | Always factor out GCF first |

---

## 🔄 Quick Revision Questions

1. **When should you use the grouping method?**

2. **What must be true about the groups for factoring to work?**

3. **How is $(x - 3)$ related to $(3 - x)$?**

4. **Factor: $ax + bx + ay + by$**

5. **What should you always do BEFORE trying to group?**

6. **If grouping the first two and last two doesn't work, what should you try?**

<details>
<summary>Quick Answers</summary>

1. When the polynomial has 4 or more terms
2. Both groups must share a common binomial factor
3. $(x - 3) = -(3 - x)$
4. $(a + b)(x + y)$
5. Factor out the GCF of all terms
6. Rearrange the terms and try different groupings

</details>

---

## 🔗 Key Takeaways

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CHAPTER SUMMARY                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ★ Grouping method works for 4+ terms                            │
│                                                                     │
│   ★ Goal: Create groups with a common binomial factor             │
│                                                                     │
│   ★ Steps:                                                         │
│     1. Factor out any overall GCF first                           │
│     2. Group terms in pairs                                        │
│     3. Factor out GCF from each group                             │
│     4. Factor out the common binomial                              │
│                                                                     │
│   ★ If groups don't match, rearrange terms                        │
│                                                                     │
│   ★ Use the identity: (a - b) = -(b - a)                          │
│                                                                     │
│   ★ Always check if factors can be factored further               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

[← Previous: Common Factor Method](01-common-factor-method.md) | [Back to Contents](../README.md) | [Next: Factoring Quadratic Trinomials →](03-factoring-quadratic-trinomials.md)

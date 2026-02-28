# Chapter 7.2: Graphs of Inverse Trigonometric Functions

## Overview

Understanding the graphs of inverse trigonometric functions helps visualize their behavior, domains, ranges, and relationships with their parent functions. The graphs are obtained by reflecting the restricted portions of the original functions across the line y = x.

---

## 📐 The Reflection Principle

### How to Get Inverse Graphs

```
    To obtain the graph of f⁻¹(x) from f(x):
    
    1. Start with the restricted portion of f(x)
    2. Reflect across the line y = x
    3. The result is f⁻¹(x)
    
    ┌─────────────────────────────────────────┐
    │                                         │
    │   Points (a, b) on f(x)                │
    │        ↓                                │
    │   Become (b, a) on f⁻¹(x)              │
    │                                         │
    └─────────────────────────────────────────┘
```

---

## 📈 Graph of y = sin⁻¹(x)

### Construction

```
    Start with y = sin x restricted to [-π/2, π/2]
    Reflect across y = x
    
    y = sin x (restricted)          y = sin⁻¹(x)
    
         y                               y
         │                               │
     π/2 ┤        ___                π/2 ┤           *
         │      /                        │         *
         │    /                          │       *
       0 ┼──*────────              0 ┼───*────────
         │    \                          │     *
         │      \                        │   *
    -π/2 ┤        ‾‾‾               -π/2 ┤ *
         │                               │
         └───────────────x               └───────────x
           -1   0   1                     -1   0   1
```

### Key Features

| Property | Value |
|----------|-------|
| Domain | [-1, 1] |
| Range | [-π/2, π/2] |
| x-intercept | (0, 0) |
| Odd function | sin⁻¹(-x) = -sin⁻¹(x) |
| Increasing | Throughout domain |
| Concavity | Concave up for x < 0, concave down for x > 0 |

---

## 📈 Graph of y = cos⁻¹(x)

### Construction

```
    Start with y = cos x restricted to [0, π]
    Reflect across y = x
    
    y = cos x (restricted)          y = cos⁻¹(x)
    
         y                               y
         │                               │
       1 ┤*                            π ┤*
         │ \                             │  \
         │  \                            │   \
       0 ┼───\────────            π/2 ┼────\────────
         │    \                          │     \
         │     \                         │      \
      -1 ┤      *                      0 ┤        *
         │                               │
         └───────────x                   └───────────x
           0   π/2  π                    -1   0   1
```

### Key Features

| Property | Value |
|----------|-------|
| Domain | [-1, 1] |
| Range | [0, π] |
| y-intercept | (0, π/2) |
| Neither odd nor even | cos⁻¹(-x) = π - cos⁻¹(x) |
| Decreasing | Throughout domain |
| Concavity | Concave down for x < 0, concave up for x > 0 |

---

## 📈 Graph of y = tan⁻¹(x)

### Construction

```
    Start with y = tan x restricted to (-π/2, π/2)
    Reflect across y = x
    
    y = tan x (restricted)          y = tan⁻¹(x)
    
         y                               y
         │        :                      │
         │       /                   π/2 ┼ - - - - - - - -
         │      /                        │            ___─
         │     /                         │         __─
       0 ┼────/────                  0 ┼────────*─────────
         │   /                           │     __─
         │  /                            │  __─
         │ /                        -π/2 ┼─‾ - - - - - - -
         │:                              │
         └───────────x                   └────────────────x
          -π/2  0  π/2                   
```

### Key Features

| Property | Value |
|----------|-------|
| Domain | (-∞, ∞) |
| Range | (-π/2, π/2) |
| x-intercept | (0, 0) |
| Odd function | tan⁻¹(-x) = -tan⁻¹(x) |
| Increasing | Throughout domain |
| Horizontal asymptotes | y = π/2 (as x→∞), y = -π/2 (as x→-∞) |
| Inflection point | At origin |

---

## 📈 Graph of y = cot⁻¹(x)

### Key Features

```
         y
         │
       π ┤‾ - - - - - - -
         │\
         │ \
    π/2  ┤  ────────────────
         │           \
         │            \
       0 ┤- - - - - - - \‾
         │
         └────────────────x
```

| Property | Value |
|----------|-------|
| Domain | (-∞, ∞) |
| Range | (0, π) |
| y-intercept | (0, π/2) |
| Decreasing | Throughout domain |
| Horizontal asymptotes | y = 0 (as x→∞), y = π (as x→-∞) |

---

## 📈 Graphs of y = sec⁻¹(x) and y = csc⁻¹(x)

### y = sec⁻¹(x)

```
         y
         │
       π ┤    *           *    
         │   *           *
    π/2  ┤─ ─ ─ ─ ─ ─ ─ ─ ─ ─ (asymptote)
         │  *           *
       0 ┤ *           *
         │
         └─────────────────────x
              -1       1
```

| Property | Value |
|----------|-------|
| Domain | (-∞, -1] ∪ [1, ∞) |
| Range | [0, π], y ≠ π/2 |

### y = csc⁻¹(x)

```
         y
         │
    π/2  ┤    *           *
         │   *           *
       0 ┤─ ─ ─ ─ ─ ─ ─ ─ ─ ─ (asymptote)
         │  *           *
   -π/2  ┤ *           *
         │
         └─────────────────────x
              -1       1
```

| Property | Value |
|----------|-------|
| Domain | (-∞, -1] ∪ [1, ∞) |
| Range | [-π/2, π/2], y ≠ 0 |

---

## 📊 Comparison of Main Graphs

```
    ┌────────────────────────────────────────────────────────────────┐
    │              ALL THREE MAIN INVERSE FUNCTIONS                  │
    │                                                                │
    │         y                                                      │
    │         │                                                      │
    │     π   ┤     C                    C = cos⁻¹(x)               │
    │         │    C C                                               │
    │    π/2  ┤   C   C    T T T         T = tan⁻¹(x)               │
    │         │  C     C T               S = sin⁻¹(x)               │
    │      0  ┼────C───S─T────────────                              │
    │         │        S T                                           │
    │   -π/2  ┤       S   T T T                                      │
    │         │      S                                               │
    │         │                                                      │
    │         └──────────────────── x                                │
    │             -1  0  1                                           │
    │                                                                │
    │  Note: sin⁻¹ and tan⁻¹ pass through origin                   │
    │        cos⁻¹ passes through (0, π/2) and (1, 0)               │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📐 Symmetry Properties

### Odd Functions (Symmetric about Origin)

$$\sin^{-1}(-x) = -\sin^{-1}(x)$$

$$\tan^{-1}(-x) = -\tan^{-1}(x)$$

$$\csc^{-1}(-x) = -\csc^{-1}(x)$$

These graphs pass through the origin and are symmetric about it.

### Special Symmetry for Cosine

$$\cos^{-1}(-x) = \pi - \cos^{-1}(x)$$

This means:
- cos⁻¹(x) + cos⁻¹(-x) = π
- The graph is symmetric about the point (0, π/2)

### Special Symmetry for Secant

$$\sec^{-1}(-x) = \pi - \sec^{-1}(x)$$

---

## 📊 Key Points on Each Graph

### Points on y = sin⁻¹(x)

| x | y = sin⁻¹(x) |
|---|--------------|
| -1 | -π/2 |
| -√3/2 | -π/3 |
| -√2/2 | -π/4 |
| -1/2 | -π/6 |
| 0 | 0 |
| 1/2 | π/6 |
| √2/2 | π/4 |
| √3/2 | π/3 |
| 1 | π/2 |

### Points on y = cos⁻¹(x)

| x | y = cos⁻¹(x) |
|---|--------------|
| -1 | π |
| -√3/2 | 5π/6 |
| -√2/2 | 3π/4 |
| -1/2 | 2π/3 |
| 0 | π/2 |
| 1/2 | π/3 |
| √2/2 | π/4 |
| √3/2 | π/6 |
| 1 | 0 |

### Points on y = tan⁻¹(x)

| x | y = tan⁻¹(x) |
|---|--------------|
| -√3 | -π/3 |
| -1 | -π/4 |
| -1/√3 | -π/6 |
| 0 | 0 |
| 1/√3 | π/6 |
| 1 | π/4 |
| √3 | π/3 |

---

## 📐 Relationship Between sin⁻¹ and cos⁻¹

### The Complementary Relationship

$$\sin^{-1}(x) + \cos^{-1}(x) = \frac{\pi}{2}$$

for all x ∈ [-1, 1].

```
    Graphical interpretation:
    
    The vertical distance from sin⁻¹(x) to the line y = π/2
    equals the value of cos⁻¹(x).
    
         y
         │
       π ┤         C
         │       C
    π/2  ┼─ ─ ─●─ ─ ─ ─ ─     (This constant sum = π/2)
         │   S  ↕
         │  S   = cos⁻¹(x)
       0 ┼──────────
         │
         └───────────── x
             x
    
    S = sin⁻¹(x), C = cos⁻¹(x)
    S + C = π/2
```

---

## 📋 Summary Table

### Graph Properties

| Function | Domain | Range | Increasing/Decreasing | Odd/Even |
|----------|--------|-------|----------------------|----------|
| sin⁻¹(x) | [-1, 1] | [-π/2, π/2] | Increasing | Odd |
| cos⁻¹(x) | [-1, 1] | [0, π] | Decreasing | Neither |
| tan⁻¹(x) | ℝ | (-π/2, π/2) | Increasing | Odd |
| cot⁻¹(x) | ℝ | (0, π) | Decreasing | Neither |
| sec⁻¹(x) | \|x\|≥1 | [0,π], y≠π/2 | Inc. on [1,∞), Inc. on (-∞,-1] | Neither |
| csc⁻¹(x) | \|x\|≥1 | [-π/2,π/2], y≠0 | Dec. on [1,∞), Dec. on (-∞,-1] | Odd |

### Important Intercepts

| Function | x-intercept | y-intercept |
|----------|-------------|-------------|
| sin⁻¹(x) | (0, 0) | (0, 0) |
| cos⁻¹(x) | (1, 0) | (0, π/2) |
| tan⁻¹(x) | (0, 0) | (0, 0) |
| cot⁻¹(x) | None | (0, π/2) |

---

## ❓ Quick Revision Questions

1. **What is the horizontal asymptote of y = tan⁻¹(x) as x → ∞?**

2. **Is y = sin⁻¹(x) an increasing or decreasing function?**

3. **What is the y-intercept of y = cos⁻¹(x)?**

4. **Describe the symmetry of the graph of y = tan⁻¹(x).**

5. **What is the relationship between sin⁻¹(x) and cos⁻¹(x) for any x in [-1, 1]?**

6. **Sketch the graph of y = sin⁻¹(x) + π/2 and describe its relationship to y = cos⁻¹(x).**

<details>
<summary>Click to see answers</summary>

1. As x → ∞, y = tan⁻¹(x) → **π/2**  
   (The horizontal asymptote is y = π/2)

2. y = sin⁻¹(x) is **increasing** throughout its domain [-1, 1].

3. When x = 0, y = cos⁻¹(0) = **π/2**  
   So the y-intercept is **(0, π/2)**

4. y = tan⁻¹(x) is an **odd function**, so its graph is **symmetric about the origin**.  
   If (a, b) is on the graph, then (-a, -b) is also on the graph.

5. For any x ∈ [-1, 1]:  
   **sin⁻¹(x) + cos⁻¹(x) = π/2**  
   They are complementary.

6. y = sin⁻¹(x) + π/2:
   - This shifts sin⁻¹(x) up by π/2
   - Domain: [-1, 1]
   - Range: [0, π] (same as cos⁻¹(x)!)
   
   Using the identity sin⁻¹(x) + cos⁻¹(x) = π/2:
   sin⁻¹(x) = π/2 - cos⁻¹(x)
   sin⁻¹(x) + π/2 = π - cos⁻¹(x)
   
   This is NOT the same as cos⁻¹(x), but rather its reflection about y = π/2.
   
   Actually, we can write:
   y = sin⁻¹(x) + π/2 gives the same range [0, π] but different shape.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 7.1 Definition and Principal Values](01-definition-principal-values.md) | [Unit 7 Index](README.md) | [7.3 Properties and Identities →](03-properties-identities.md) |

---

[← Back to Main Index](../README.md)

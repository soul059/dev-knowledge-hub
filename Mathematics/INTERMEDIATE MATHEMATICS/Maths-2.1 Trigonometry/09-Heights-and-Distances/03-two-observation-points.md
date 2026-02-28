# Chapter 9.3: Problems from Two Observation Points

## Overview

When observations are made from two different points, we can solve more complex problems that would be impossible from a single location. This technique is fundamental to surveying, navigation, and locating objects in three-dimensional space.

---

## 📐 Type 1: Two Points on the Same Side (Collinear)

### Configuration

```
                    T (top)
                    |
                    |
                    | h
                    |
                    |
        α     β     |
    A─────────B─────C
         d       x
```

Where:
- α = angle of elevation from A
- β = angle of elevation from B (β > α)
- d = distance between observation points
- x = distance from B to object base

### Formulas

From point A: $\tan\alpha = \frac{h}{d + x}$

From point B: $\tan\beta = \frac{h}{x}$

Eliminating x:

$$\boxed{h = \frac{d \tan\alpha \tan\beta}{\tan\beta - \tan\alpha}}$$

$$\boxed{h = \frac{d}{\cot\alpha - \cot\beta}}$$

### Example

**From two points A and B, 100 m apart on the same side of a tower, the angles of elevation of the top are 30° and 60° respectively. Find the tower's height and its distance from B.**

**Solution:**

```
    Using the formula:
    h = d × tan α × tan β/(tan β - tan α)
    h = 100 × tan 30° × tan 60°/(tan 60° - tan 30°)
    h = 100 × (1/√3) × √3/(√3 - 1/√3)
    h = 100 × 1/((3-1)/√3)
    h = 100 × 1/(2/√3)
    h = 100 × √3/2
    h = 50√3 ≈ 86.6 m
    
    Distance from B:
    x = h/tan 60° = 50√3/√3 = 50 m
```

**Answer: Height = 50√3 m, Distance from B = 50 m**

---

## 📐 Type 2: Two Points on Opposite Sides

### Configuration

```
                    T
                    |
                    |
                    | h
                    |
                    |
        α           |           β
    A───────────────C───────────────B
           x                 y
            ←───── d ─────→
```

Where d = x + y = AB

### Formulas

From A: $\tan\alpha = \frac{h}{x}$ → $x = h\cot\alpha$

From B: $\tan\beta = \frac{h}{y}$ → $y = h\cot\beta$

Since x + y = d:
$$h\cot\alpha + h\cot\beta = d$$

$$\boxed{h = \frac{d}{\cot\alpha + \cot\beta} = \frac{d \tan\alpha \tan\beta}{\tan\alpha + \tan\beta}}$$

### Example

**Two people standing on opposite sides of a tower observe its top at angles of elevation of 30° and 45°. If the distance between them is 100 m, find the height.**

**Solution:**

```
    h = d/(cot α + cot β)
    h = 100/(cot 30° + cot 45°)
    h = 100/(√3 + 1)
    
    Rationalizing:
    h = 100(√3 - 1)/((√3 + 1)(√3 - 1))
    h = 100(√3 - 1)/(3 - 1)
    h = 100(√3 - 1)/2
    h = 50(√3 - 1)
    h ≈ 36.6 m
```

**Answer: Height = 50(√3 - 1) ≈ 36.6 m**

---

## 📐 Type 3: Observations Not in Line with Object

### Configuration

```
                    T (top)
                    |
                    | h
                    |
                    C
                   /|\
                  / | \
                 /  |  \
                /   |   \
               /    |    \
              A─────+─────B
                    d
    
    View from above:
    
              C
             /|\
            / | \
           /  |  \
          /   |   \
         A────M────B
              d
```

When the observation points A and B are not in line with the object C, we need additional information:
- The angle ACB (angle subtended at the object)
- Or the bearing from each point

### Using Sine Rule

In triangle ABC:
$$\frac{AC}{\sin B} = \frac{BC}{\sin A} = \frac{AB}{\sin C}$$

Then use the elevation angles to find height.

---

## 📐 Type 4: Moving Observer

### Configuration

```
                    T
                    |
                    |
                    | h
                    |
                    |
        α     β     |
    A─────────B─────C
         d
```

Where the observer moves from A to B (distance d) towards the tower.

This is similar to Type 1, but often the problem asks for height when the observer moves a known distance.

### Example

**Walking 50 m towards a tower, the angle of elevation changes from 30° to 45°. Find the height.**

**Solution:**

```
    Let remaining distance from B = x
    
    From B: tan 45° = h/x → h = x
    From A: tan 30° = h/(x + 50) → h = (x + 50)/√3
    
    Equating:
    x = (x + 50)/√3
    x√3 = x + 50
    x(√3 - 1) = 50
    x = 50/(√3 - 1)
    x = 50(√3 + 1)/2 = 25(√3 + 1)
    
    h = x = 25(√3 + 1) ≈ 68.3 m
```

---

## 📐 Type 5: Object Between Two Observers

### Configuration

```
    A─────────────T─────────────B
    ↑            /|\            ↑
    |α          / | \          β|
    |          /  |  \          |
               / h|   \
              /   |    \
             /    |     \
            /     |      \
           P──────+───────Q
                  C
```

Where T is on top of a pole at C, and observers at A and B see T at angles of depression α and β.

---

## 📐 Type 6: Bearings and Directions

### Configuration with Bearings

```
                N
                ↑
                |
            T   |
           /|\  |
          / | \ |
         /  |  \|
        A───+───●───→ E
                B
```

When bearing information is given:
- Convert bearings to angles
- Use sine and cosine rules
- Draw clear diagrams

### Example

**From point A, a tower bears N60°E. From point B, 100 m due east of A, the tower bears N30°W. Find the distance of the tower from A.**

**Solution:**

```
                    N
                    ↑
                    |
                T   |
               /    |
              / 30° |
             /      |
            /       |
           /    60° |
          A─────────B──→ E
              100m
    
    In triangle ABT:
    Angle at A = 90° - 60° = 30° (from east)
    Angle at B = 90° + 30° = 120° (from east)
    
    Actually, let's reconsider:
    At A: bearing N60°E means 60° from North towards East
    At B: bearing N30°W means 30° from North towards West
    
    Angle TAB = 90° - 60° = 30°
    Angle TBA = 90° + 30° = 120°
    Angle ATB = 180° - 30° - 120° = 30°
    
    Using Sine Rule:
    AT/sin(∠ABT) = AB/sin(∠ATB)
    AT/sin 120° = 100/sin 30°
    AT = 100 × sin 120°/sin 30°
    AT = 100 × (√3/2)/(1/2)
    AT = 100√3 ≈ 173.2 m
```

---

## 📐 Type 7: Three-Dimensional Problems

### Aircraft Observation Problem

```
    Aircraft at P, altitude h
    
                P (aircraft)
               /|\
              / | \
             /  |  \
            /   |h  \
           /    |    \
          /     |     \
         A──────O──────B
              ground
```

Two observers at A and B see the aircraft at different angles.

### Example

**Two observers 1000 m apart observe an aircraft. From A, the angle of elevation is 60°. From B, the angle of elevation is 45°. If A, B, and the point directly below the aircraft are collinear, find the altitude.**

**Solution:**

```
    Let the aircraft be directly above point C.
    Let AC = x, then BC = 1000 - x (if C is between A and B)
    
    From A: tan 60° = h/x → h = x√3
    From B: tan 45° = h/(1000-x) → h = 1000 - x
    
    Equating:
    x√3 = 1000 - x
    x√3 + x = 1000
    x(√3 + 1) = 1000
    x = 1000/(√3 + 1) = 500(√3 - 1)
    
    h = x√3 = 500(√3 - 1)√3 = 500(3 - √3)
    h = 1500 - 500√3 ≈ 634 m
```

---

## 📝 Worked Examples

### Example 1: Complete Problem

**Two buildings are 60 m apart. From the top of the shorter building, the angles of elevation and depression to the top and bottom of the taller building are 30° and 45° respectively. Find the heights of both buildings.**

**Solution:**

```
                    T (tall top)
                    |
                    | h₂
         Horizontal |
    ─────────S──────P────
         /30°|\45°  |
        /    |      |
       /     | h₁   | h₁
      /      |      |
     /       |      |
    A────────+──────B
              60m
    
    From angle of depression 45° (to taller building base):
    tan 45° = h₁/60
    1 = h₁/60
    h₁ = 60 m (height of shorter building)
    
    From angle of elevation 30° (to taller building top):
    tan 30° = h₂/60
    h₂ = 60/√3 = 20√3 m
    
    Height of taller building = h₁ + h₂ = 60 + 20√3 ≈ 94.64 m
```

---

### Example 2: River Width Problem

**A person on one bank of a river observes that the angle of elevation of the top of a tree on the opposite bank is 60°. When he moves 40 m away from the river, the angle becomes 30°. Find the width of the river and the height of the tree.**

**Solution:**

```
                    T
                    |
                    | h
                    |
        30°   60°   |
    A─────────B─────C
        40m     w
    
    From B (at river): tan 60° = h/w → h = w√3 ... (1)
    
    From A (40m back): tan 30° = h/(w + 40)
                       h = (w + 40)/√3 ... (2)
    
    From (1) and (2):
    w√3 = (w + 40)/√3
    3w = w + 40
    2w = 40
    w = 20 m (river width)
    
    h = 20√3 ≈ 34.64 m (tree height)
```

---

### Example 3: Non-Collinear Points

**From two points A and B on the ground, the angles of elevation of the top of a tower are 30° and 45° respectively. If AB = 100 m and the tower is not on line AB, but angle ACB = 90° (where C is the base of the tower), find the height.**

**Solution:**

```
    View from above:
    
              C (tower base)
             /|
            / |
        x  /  | y
          /   |
         /90° |
        A─────B
          100
    
    By Pythagorean theorem: x² + y² = 100² ... (1)
    
    From elevation angles:
    tan 30° = h/x → x = h√3 ... (2)
    tan 45° = h/y → y = h ... (3)
    
    Substituting in (1):
    (h√3)² + h² = 10000
    3h² + h² = 10000
    4h² = 10000
    h² = 2500
    h = 50 m
```

**Answer: Tower height = 50 m**

---

## 📋 Summary Table

### Two-Point Problems

| Configuration | Formula for Height h | Notes |
|--------------|---------------------|-------|
| Same side, angles α, β | h = d tan α tan β/(tan β - tan α) | β > α, d = distance between points |
| Opposite sides, angles α, β | h = d tan α tan β/(tan α + tan β) | d = total distance |
| Moving towards, angles change | h = d/(cot α - cot β) | d = distance moved |

### Key Strategies

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   1. Draw clear diagrams with all given information        │
    │                                                             │
    │   2. Set up equations from each observation point          │
    │                                                             │
    │   3. Use common height or distance to link equations       │
    │                                                             │
    │   4. For non-collinear points, consider:                   │
    │      - Sine Rule and Cosine Rule                           │
    │      - 3D geometry                                         │
    │      - Bearings and directions                             │
    │                                                             │
    │   5. Check: answers should be positive and reasonable      │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## ❓ Quick Revision Questions

1. **From two points on the same side of a tower, 80 m apart, the angles of elevation are 30° and 45°. Find the tower height.**

2. **Two people on opposite sides of a pole observe its top at 60° and 30°. If they are 40 m apart, find the pole height.**

3. **Write the formula for height when observations are made from two points on the same side.**

4. **A person walks 60 m towards a building and the angle of elevation changes from 30° to 60°. Find the building height.**

5. **From point A, a lighthouse bears N45°E. From B, 100√2 m due north of A, it bears S45°E. Find the distance from A to the lighthouse.**

6. **Two observers 200 m apart on opposite sides of a tower measure angles of elevation as 45° each. Find the tower height.**

<details>
<summary>Click to see answers</summary>

1. **Tower height (same side, 80 m apart, angles 30° and 45°):**
   
   h = d tan α tan β/(tan β - tan α)
   h = 80 × tan 30° × tan 45°/(tan 45° - tan 30°)
   h = 80 × (1/√3) × 1/(1 - 1/√3)
   h = 80/(√3 - 1)
   h = 80(√3 + 1)/2 = **40(√3 + 1) ≈ 109.3 m**

2. **Pole height (opposite sides, 40 m, angles 60° and 30°):**
   
   h = d/(cot α + cot β)
   h = 40/(cot 60° + cot 30°)
   h = 40/(1/√3 + √3)
   h = 40/((1 + 3)/√3)
   h = 40√3/4 = **10√3 ≈ 17.32 m**

3. **Formula:**
   $$h = \frac{d \tan\alpha \tan\beta}{\tan\beta - \tan\alpha} = \frac{d}{\cot\alpha - \cot\beta}$$

4. **Building height (60 m movement, angles 30° to 60°):**
   
   h = d/(cot α - cot β)
   h = 60/(cot 30° - cot 60°)
   h = 60/(√3 - 1/√3)
   h = 60/((3 - 1)/√3)
   h = 60√3/2 = **30√3 ≈ 51.96 m**

5. **Distance to lighthouse:**
   
   In triangle formed:
   - Angle at A = 45° (bearing N45°E means 45° from north)
   - Angle at B = 180° - 45° = 135° (bearing S45°E from B)
   - Angle at lighthouse = 180° - 45° - 135° = 0°... 
   
   Wait, let me reconsider. From B, bearing S45°E means 45° from south towards east.
   
   Angle at A (from AB towards lighthouse) = 45°
   Angle at B (from BA towards lighthouse) = 45° (on the other side)
   So angle at lighthouse = 180° - 45° - 45° = 90°
   
   Using Sine Rule:
   AL/sin 45° = 100√2/sin 90°
   AL = 100√2 × (√2/2)/1 = **100 m**

6. **Tower height (opposite sides, 200 m, both angles 45°):**
   
   h = d/(cot α + cot β)
   h = 200/(cot 45° + cot 45°)
   h = 200/(1 + 1)
   h = **100 m**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 9.2 Problems from Single Point](02-single-observation-point.md) | [Unit 9 Index](README.md) | [9.4 Practical Applications →](04-practical-applications.md) |

---

[← Back to Main Index](../README.md)

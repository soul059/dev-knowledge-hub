# Chapter 9.1: Angles of Elevation and Depression

## Overview

Understanding angles of elevation and depression is fundamental to solving practical problems in surveying, navigation, astronomy, and everyday applications. This chapter introduces these concepts and simple applications.

---

## 📐 Line of Sight and Horizontal Line

### Definitions

**Line of Sight**: The straight line drawn from the eye of an observer to the point being observed.

**Horizontal Line**: A line parallel to the ground, passing through the observer's eye level.

```
    Observer's Eye ──────────────────────── Horizontal Line
                   \
                    \  Line of Sight
                     \
                      \
                       * Object
```

---

## 📐 Angle of Elevation

### Definition

The **angle of elevation** is the angle formed between the horizontal line of sight and the line of sight to an object that is **above** the horizontal level.

```
                                   * P (Object above)
                                  /|
                                 / |
                                /  |
           Line of Sight      /   |
                             /    | vertical
                            /     | height
                           /      |
                          /θ      |
         ────────────────*────────+──────── Horizontal
                         O        A
                     (Observer)   
                         
         θ = Angle of Elevation of P from O
```

### Key Points

- The observer looks **UP** to see the object
- The angle is measured from the horizontal to the line of sight
- Always measured in the plane containing the horizontal and the line of sight

---

## 📐 Angle of Depression

### Definition

The **angle of depression** is the angle formed between the horizontal line of sight and the line of sight to an object that is **below** the horizontal level.

```
         ────────────────*──────────────── Horizontal
                         O\α
                     (Observer)\
                               \
                                \  Line of Sight
                                 \
                                  \
                                   * P (Object below)
         
         α = Angle of Depression of P from O
```

### Key Points

- The observer looks **DOWN** to see the object
- The angle is measured from the horizontal downward
- The angle is always taken as positive

---

## 📐 Relationship Between Elevation and Depression

### Alternate Angles Property

```
                    Horizontal 2
         ──────────────*──────────────
                       B\α
                         \
                          \
                           \
                            \
                    ────────*α────────── Horizontal 1
                            A
    
    The angle of depression from B to A equals
    the angle of elevation from A to B.
    
    (These are alternate angles with parallel horizontals)
```

$$\boxed{\text{Angle of depression from B} = \text{Angle of elevation from A}}$$

This property is extremely useful in problem-solving!

---

## 📐 Basic Formulas for Right Triangles

### Standard Configuration

```
                    P
                    |\
                    | \
                    |  \
               h    |   \ (line of sight)
              (opp) |    \
                    |     \
                    |    θ \
                    A───────O
                       d
                    (adj)
```

### Formulas

$$\tan\theta = \frac{h}{d} = \frac{\text{opposite}}{\text{adjacent}}$$

$$h = d \tan\theta \quad \text{(finding height)}$$

$$d = \frac{h}{\tan\theta} = h \cot\theta \quad \text{(finding distance)}$$

---

## 📝 Worked Examples

### Example 1: Finding Height (Simple)

**A person standing 50 m away from a tower observes its top at an angle of elevation of 60°. Find the height of the tower.**

```
                    P (top)
                    |
                    |
                    | h = ?
                    |
                    |
        60°         |
    O───────────────A
         50 m
```

**Solution:**

$$\tan 60° = \frac{h}{50}$$

$$\sqrt{3} = \frac{h}{50}$$

$$h = 50\sqrt{3} \approx 86.6 \text{ m}$$

**Answer: Height = 50√3 m ≈ 86.6 m**

---

### Example 2: Finding Distance

**From the top of a 100 m high lighthouse, the angle of depression of a boat is 30°. How far is the boat from the base of the lighthouse?**

```
         Horizontal
    ─────────*──────────
             L\30°
         100m  \
               \
                \
                 \
                  * B (boat)
    
    (Using alternate angles, angle of elevation from boat = 30°)
```

**Solution:**

```
    The angle of elevation from B to L = 30° (alternate angles)
    
    tan 30° = 100/d
    
    1/√3 = 100/d
    
    d = 100√3 ≈ 173.2 m
```

**Answer: Distance = 100√3 m ≈ 173.2 m**

---

### Example 3: Observer Not at Ground Level

**From the top of a building 20 m high, the angle of elevation of a tower's top is 30° and the angle of depression of its base is 45°. Find the height of the tower.**

```
                         T (tower top)
                         |
                         |
                         | h₂
         Horizontal      |
    ─────────B───────────+──────
         /30°|45°        |
        /    |           |
       /     | 20 m      | 20 m (h₁)
      /      |           |
     /       |           |
    A────────+───────────F
              d
```

**Solution:**

```
    Let the distance BF = d
    
    From angle of depression 45°:
    tan 45° = 20/d
    1 = 20/d
    d = 20 m
    
    From angle of elevation 30°:
    tan 30° = h₂/d
    1/√3 = h₂/20
    h₂ = 20/√3 = 20√3/3 m
    
    Total height of tower = h₁ + h₂
                         = 20 + 20√3/3
                         = 20(1 + 1/√3)
                         = 20(√3 + 1)/√3
                         = 20(√3 + 1)√3/3
                         = (60 + 20√3)/3 m
                         ≈ 31.55 m
```

**Answer: Height of tower = 20(1 + 1/√3) ≈ 31.55 m**

---

### Example 4: Moving Observer

**A person walks 100 m towards a tower. The angle of elevation of the top changes from 30° to 60°. Find the height of the tower.**

```
                    T
                    |
                    |
                    | h
                    |
                    |
    30°      60°    |
    A────────B──────C
       100m     d
```

**Solution:**

```
    Let BC = d (remaining distance)
    
    From position B (angle 60°):
    tan 60° = h/d
    √3 = h/d
    h = d√3  ... (1)
    
    From position A (angle 30°):
    tan 30° = h/(d + 100)
    1/√3 = h/(d + 100)
    h = (d + 100)/√3  ... (2)
    
    From (1) and (2):
    d√3 = (d + 100)/√3
    3d = d + 100
    2d = 100
    d = 50 m
    
    h = d√3 = 50√3 ≈ 86.6 m
```

**Answer: Height = 50√3 m ≈ 86.6 m**

---

### Example 5: Two Objects Observed

**From a point on the ground, the angles of elevation of the top and bottom of a transmission tower fixed on top of a 20 m high building are 60° and 45° respectively. Find the height of the tower.**

```
                    T (tower top)
                    |
                    | h (tower)
                    |
    ────────────────P (tower base/building top)
                    |
                    | 20 m (building)
                    |
         45° 60°    |
    O───────────────B
            d
```

**Solution:**

```
    From angle 45° (to building top):
    tan 45° = 20/d
    1 = 20/d
    d = 20 m
    
    From angle 60° (to tower top):
    tan 60° = (20 + h)/d
    √3 = (20 + h)/20
    20√3 = 20 + h
    h = 20√3 - 20
    h = 20(√3 - 1)
    h ≈ 14.64 m
```

**Answer: Tower height = 20(√3 - 1) ≈ 14.64 m**

---

## 📐 Common Angle Values

### Quick Reference

| Angle θ | sin θ | cos θ | tan θ |
|---------|-------|-------|-------|
| 30° | 1/2 | √3/2 | 1/√3 |
| 45° | √2/2 | √2/2 | 1 |
| 60° | √3/2 | 1/2 | √3 |

### Height Formulas for Common Angles

| Angle | Height h (given distance d) |
|-------|----------------------------|
| 30° | h = d/√3 = d√3/3 |
| 45° | h = d |
| 60° | h = d√3 |

---

## 📋 Summary Table

### Key Concepts

| Concept | Definition | Diagram Feature |
|---------|------------|-----------------|
| Angle of Elevation | Angle looking UP from horizontal | Observer below object |
| Angle of Depression | Angle looking DOWN from horizontal | Observer above object |
| Line of Sight | Direct line from observer to object | Hypotenuse of right triangle |
| Horizontal | Level line at observer's eye | Base reference |

### Problem-Solving Checklist

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   □ Draw a clear diagram                                   │
    │   □ Mark all angles correctly                              │
    │   □ Use alternate angles where applicable                  │
    │   □ Identify right triangles                               │
    │   □ Choose appropriate trig ratio (usually tan)            │
    │   □ Set up equation(s)                                     │
    │   □ Solve for unknowns                                     │
    │   □ Check answer for reasonableness                        │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## ❓ Quick Revision Questions

1. **Define angle of elevation.**

2. **What is the relationship between the angle of elevation from A to B and the angle of depression from B to A?**

3. **A 1.6 m tall person stands 50 m away from a building. If the angle of elevation of the top is 45°, find the building's height.**

4. **From the top of a 60 m cliff, the angle of depression of a boat is 30°. Find the distance of the boat from the cliff base.**

5. **An observer's angle of elevation to a bird is 60°. If the bird is 50 m directly above the ground, find the horizontal distance from the observer to the point directly below the bird.**

6. **As you walk towards a tower, what happens to the angle of elevation of its top?**

<details>
<summary>Click to see answers</summary>

1. **Angle of elevation** is the angle formed between the horizontal line of sight and the line of sight to an object that is above the horizontal level.

2. They are **equal** (alternate angles with parallel horizontal lines).

3. **Building height:**
   
   Let building height be h.
   Height above observer's eyes = h - 1.6
   
   tan 45° = (h - 1.6)/50
   1 = (h - 1.6)/50
   h - 1.6 = 50
   **h = 51.6 m**

4. **Distance from cliff:**
   
   tan 30° = 60/d
   1/√3 = 60/d
   d = 60√3 ≈ **103.92 m**

5. **Horizontal distance:**
   
   tan 60° = 50/d
   √3 = 50/d
   d = 50/√3 = 50√3/3 ≈ **28.87 m**

6. The angle of elevation **increases** as you walk towards the tower (the tower appears steeper as you get closer).

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Unit 8: Properties of Triangles](../08-Properties-of-Triangles/README.md) | [Unit 9 Index](README.md) | [9.2 Problems from Single Point →](02-single-observation-point.md) |

---

[← Back to Main Index](../README.md)

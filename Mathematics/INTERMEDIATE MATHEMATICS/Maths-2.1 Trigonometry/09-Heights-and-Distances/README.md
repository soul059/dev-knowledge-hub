# Unit 9: Heights and Distances

## Overview

This unit applies all the trigonometric concepts learned so far to solve practical problems involving heights of objects and distances that cannot be measured directly. These techniques are fundamental to surveying, navigation, astronomy, and engineering.

---

## 📚 Chapter Contents

| Chapter | Topic | Description |
|---------|-------|-------------|
| 9.1 | [Angles of Elevation and Depression](01-angles-elevation-depression.md) | Basic concepts and simple problems |
| 9.2 | [Problems from Single Point](02-single-observation-point.md) | Observations from one location |
| 9.3 | [Problems from Two Points](03-two-observation-points.md) | Using multiple observation points |
| 9.4 | [Practical Applications](04-practical-applications.md) | Real-world problem solving |

---

## 📐 Fundamental Concepts

### Angle of Elevation

The angle formed between the horizontal line of sight and the line of sight to an object **above** the horizontal.

```
                          * Object (P)
                         /|
                        / |
                       /  |
                      /   |
                     /    | height
                    /θ    |
    Observer ──────*──────+──── Horizontal
                   O
    
    θ = Angle of Elevation
```

### Angle of Depression

The angle formed between the horizontal line of sight and the line of sight to an object **below** the horizontal.

```
    Horizontal ────*──────────
                   O\θ
                     \
                      \  θ = Angle of Depression
                       \
                        \
                         * Object (P)
```

### Key Relationship

$$\text{Angle of Elevation from A to B} = \text{Angle of Depression from B to A}$$

(Alternate angles with the horizontal)

---

## 🔑 Key Formulas Used

### Basic Right Triangle Relationships

$$\tan\theta = \frac{\text{opposite}}{\text{adjacent}} = \frac{\text{height}}{\text{distance}}$$

$$\sin\theta = \frac{\text{opposite}}{\text{hypotenuse}}$$

$$\cos\theta = \frac{\text{adjacent}}{\text{hypotenuse}}$$

### Triangle Laws

$$\frac{a}{\sin A} = \frac{b}{\sin B} = \frac{c}{\sin C}$$

$$a^2 = b^2 + c^2 - 2bc\cos A$$

---

## 📊 Types of Problems

### Classification by Difficulty

| Type | Description | Example |
|------|-------------|---------|
| Simple | Direct application of one trig ratio | Finding height from angle and distance |
| Intermediate | Two triangles sharing a common element | Object between two observers |
| Advanced | Multiple angles, moving objects | Aircraft tracking, surveying |

### Classification by Geometry

| Configuration | Key Feature |
|---------------|-------------|
| Same horizontal plane | Observer and base of object at same level |
| Different levels | Observer above or below base |
| Moving observer | Multiple observations from different points |
| Moving object | Object changes position between observations |

---

## 📐 Standard Problem-Solving Strategy

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   Step 1: Draw a clear diagram                             │
    │           - Mark all given angles and distances            │
    │           - Identify unknown quantities                    │
    │                                                             │
    │   Step 2: Identify right triangles                         │
    │           - Look for vertical heights and horizontal       │
    │             distances                                       │
    │                                                             │
    │   Step 3: Set up equations                                 │
    │           - Use tan for height-distance problems           │
    │           - Use sine/cosine rules for oblique triangles    │
    │                                                             │
    │   Step 4: Solve the equations                              │
    │           - Eliminate unknown intermediates                │
    │           - Express answer in required form                │
    │                                                             │
    │   Step 5: Verify                                           │
    │           - Check units                                    │
    │           - Confirm answer is reasonable                   │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## 📊 Common Configurations

### Configuration 1: Simple Height Problem

```
                    P (top)
                    |
                    | h
                    |
    Observer ───────A─────── Base
        θ     d
```

$$h = d \tan\theta$$

### Configuration 2: Two Observations (Same Side)

```
                    P
                    |
                    | h
                    |
    B─────α─────A───────────Q
      d         x
```

### Configuration 3: Two Observations (Opposite Sides)

```
                    P
                    |
                    | h
                    |
    A─────α─────────Q─────β─────B
          x              y
```

---

## 🎯 Learning Objectives

By the end of this unit, you will be able to:

1. Identify angles of elevation and depression in real-world scenarios
2. Draw accurate diagrams for height and distance problems
3. Solve problems involving single observation points
4. Handle complex problems with multiple observation points
5. Apply trigonometry to surveying, navigation, and engineering problems

---

## Navigation

| Previous Unit | Main Index |
|---------------|------------|
| [← Unit 8: Properties of Triangles](../08-Properties-of-Triangles/README.md) | [Main Index](../README.md) |

---

**Begin with**: [Chapter 9.1: Angles of Elevation and Depression →](01-angles-elevation-depression.md)

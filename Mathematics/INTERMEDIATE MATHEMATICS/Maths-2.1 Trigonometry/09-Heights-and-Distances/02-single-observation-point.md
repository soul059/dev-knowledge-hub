# Chapter 9.2: Problems from a Single Observation Point

## Overview

Many practical problems involve observations made from a single location. This chapter covers various scenarios where an observer measures angles to one or more objects and uses trigonometry to find heights, distances, or positions.

---

## 📐 Type 1: Finding Height of an Object

### Standard Problem Setup

```
                    P (top of object)
                    |
                    |
                    | h (height)
                    |
                    |
        θ           |
    O───────────────A (base)
         d (distance)
```

### Formula

$$h = d \tan\theta$$

### Example: Finding the Height of a Tree

**A surveyor stands 40 m from the base of a tree. The angle of elevation to the top is 32°. Find the tree's height.**

**Solution:**

```
    h = d × tan θ
    h = 40 × tan 32°
    h = 40 × 0.6249
    h ≈ 25 m
```

---

## 📐 Type 2: Finding Distance to an Object

### When Height is Known

$$d = \frac{h}{\tan\theta} = h \cot\theta$$

### Example: Distance to a Tower

**The angle of elevation of a 75 m tall tower from a point is 56°. Find the distance from the point to the tower base.**

**Solution:**

```
    d = h/tan θ
    d = 75/tan 56°
    d = 75/1.483
    d ≈ 50.6 m
```

---

## 📐 Type 3: Observer at a Height

### Configuration

```
                         T (top of tower)
                         |
         Horizontal      | h₂
    ─────────O───────────P──────
         /α  |β          |
        /    |           |
       /     | h₁        | h₁
      /      |           |
     /       |           |
    A────────+───────────B
                 d
```

Where:
- O is the observer at height h₁
- α is the angle of elevation to the tower top
- β is the angle of depression to the tower base
- h₂ is the height of tower above O's level

### Formulas

From angle of depression β:
$$d = h_1 \cot\beta = \frac{h_1}{\tan\beta}$$

From angle of elevation α:
$$h_2 = d \tan\alpha$$

Total tower height:
$$H = h_1 + h_2$$

### Example: Finding Tower Height from Building Top

**From the top of a 30 m building, the angle of elevation of a tower's top is 45° and the angle of depression of its base is 60°. Find the tower's height.**

**Solution:**

```
    From angle of depression (to tower base):
    d = 30/tan 60° = 30/√3 = 10√3 m
    
    From angle of elevation (to tower top):
    h₂ = d × tan 45° = 10√3 × 1 = 10√3 m
    
    Tower height = 30 + 10√3
                = 30 + 17.32
                ≈ 47.32 m
```

**Answer: Tower height = (30 + 10√3) m ≈ 47.32 m**

---

## 📐 Type 4: Object on Top of Another Object

### Configuration

```
                    F (flag top)
                    |
                    | h₂ (flag)
                    |
    ────────────────P (pole top/flag bottom)
                    |
                    | h₁ (pole)
                    |
        α    β      |
    O───────────────A
           d
```

### Given angles α (to flag top) and β (to pole top):

$$\tan\alpha = \frac{h_1 + h_2}{d}, \quad \tan\beta = \frac{h_1}{d}$$

### Example: Flag on a Tower

**From a point on the ground, the angles of elevation of the bottom and top of a flag mounted on a 50 m tower are 30° and 45° respectively. Find the flag's height.**

**Solution:**

```
    Let distance to tower = d
    
    From angle to tower top (30°):
    tan 30° = 50/d
    d = 50/tan 30° = 50√3 m
    
    From angle to flag top (45°):
    tan 45° = (50 + h)/d
    1 = (50 + h)/(50√3)
    50√3 = 50 + h
    h = 50√3 - 50 = 50(√3 - 1)
    h ≈ 36.6 m
```

**Answer: Flag height = 50(√3 - 1) ≈ 36.6 m**

---

## 📐 Type 5: Shadow Problems

### Configuration

```
                    P (top)
                    |\
                    | \
                    |  \ (sun's rays)
               h    |   \
                    |    \
                    |     \
                    |    θ \
                    A───────S (shadow tip)
                       s
```

### Relationship

$$\tan\theta = \frac{h}{s}$$

where θ is the sun's altitude (angle of elevation).

### Example: Finding Height from Shadow

**A vertical pole casts a 15 m shadow when the sun's altitude is 60°. Find the pole's height.**

**Solution:**

```
    h = s × tan θ
    h = 15 × tan 60°
    h = 15√3
    h ≈ 25.98 m
```

### Example: Change in Shadow Length

**A tree's shadow is 30 m when the sun's altitude is 30°. What will be the shadow length when the altitude becomes 60°?**

**Solution:**

```
    First, find tree height:
    h = 30 × tan 30° = 30/√3 = 10√3 m
    
    New shadow length when altitude = 60°:
    s' = h/tan 60° = 10√3/√3 = 10 m
```

---

## 📐 Type 6: Problems with Movement Towards/Away

### Configuration

```
                    T
                    |
                    |
                    | h
                    |
                    |
    α         β     |
    A─────────B─────C
        d₁      d₂
```

### Example: Person Approaching a Tower

**A person observes a tower at an angle of elevation of 30°. After walking 100 m towards the tower, the angle becomes 60°. Find the height.**

**Solution:**

```
    Let remaining distance = d
    
    From position B: tan 60° = h/d → h = d√3 ... (1)
    
    From position A: tan 30° = h/(d + 100)
                     1/√3 = h/(d + 100)
                     h = (d + 100)/√3 ... (2)
    
    Equating (1) and (2):
    d√3 = (d + 100)/√3
    3d = d + 100
    2d = 100
    d = 50 m
    
    h = 50√3 ≈ 86.6 m
```

---

## 📐 Type 7: Multiple Objects from One Point

### Configuration

```
        P                   Q
        |                   |
        | h₁                | h₂
        |                   |
    α   |         β         |
    O───A───────────────────B
       d₁         D        d₂
```

### Example: Two Buildings Observed

**From a point between two buildings, the angles of elevation of their tops are 30° and 60°. If the buildings are 100 m apart, find the heights if the point is equidistant from both.**

**Solution:**

```
    Since the point is equidistant, it's 50 m from each building.
    
    Height of first building (angle 30°):
    h₁ = 50 × tan 30° = 50/√3 = 50√3/3 ≈ 28.87 m
    
    Height of second building (angle 60°):
    h₂ = 50 × tan 60° = 50√3 ≈ 86.6 m
```

---

## 📝 Worked Examples

### Example 1: Lighthouse Problem

**From the deck of a ship 10 m above water level, the angle of elevation of the top of a lighthouse is 30° and the angle of depression of its base is 45°. Find the height of the lighthouse and its distance from the ship.**

**Solution:**

```
                    L (lighthouse top)
                    |
                    | h
         Horizontal |
    ─────────S──────P────
         /30°|45°   |
        /    |      |
       /     | 10   | 10
      /      |      |
     /       |      |
    W────────+──────B (lighthouse base)
                d
    
    From angle of depression 45° (to base):
    tan 45° = 10/d
    1 = 10/d
    d = 10 m
    
    From angle of elevation 30° (to top):
    tan 30° = h/d
    1/√3 = h/10
    h = 10/√3 = 10√3/3 ≈ 5.77 m
    
    Total lighthouse height = 10 + h = 10 + 10/√3
                           = 10(1 + 1/√3)
                           = 10(√3 + 1)/√3
                           ≈ 15.77 m
```

**Answer: Distance = 10 m, Lighthouse height ≈ 15.77 m**

---

### Example 2: Aircraft Problem

**An aircraft is flying horizontally at an altitude of 3000 m. From a point on the ground, the angle of elevation of the aircraft changes from 45° to 60° in 15 seconds. Find the speed of the aircraft.**

**Solution:**

```
                A₁───────────────A₂
                 \               |
                  \              |
                   \             | 3000 m
                    \     3000 m |
                     \           |
                      \60°   45° |
    ─────────────────P──────────────────
                        d₁    d₂
    
    When angle = 60° (position A₂):
    d₂ = 3000/tan 60° = 3000/√3 = 1000√3 m
    
    When angle = 45° (position A₁):
    d₁ = 3000/tan 45° = 3000/1 = 3000 m
    
    Distance traveled = d₁ - d₂ = 3000 - 1000√3
                      = 1000(3 - √3)
                      ≈ 1268 m
    
    Speed = Distance/Time = 1268/15 ≈ 84.5 m/s
                         = 84.5 × 3.6 km/h
                         ≈ 304 km/h
```

**Answer: Speed ≈ 304 km/h**

---

### Example 3: Angle Changes at Same Point

**The angle of elevation of a balloon from a point is 60°. After some time, the balloon moves vertically upward, and the angle becomes 75°. If the horizontal distance remains 100 m, find how much the balloon rose.**

**Solution:**

```
                    B₂ (new position)
                    |
                    | Δh
                    |
                    B₁ (old position)
                    |
                    | h₁
                    |
        60° 75°     |
    O───────────────A
          100 m
    
    Initial height: h₁ = 100 × tan 60° = 100√3 m
    
    Final height: h₂ = 100 × tan 75°
    
    tan 75° = tan(45° + 30°) = (1 + 1/√3)/(1 - 1/√3)
            = (√3 + 1)/(√3 - 1)
            = (√3 + 1)²/(3 - 1)
            = (3 + 2√3 + 1)/2
            = 2 + √3
    
    h₂ = 100(2 + √3) ≈ 373.2 m
    
    Rise = h₂ - h₁ = 100(2 + √3) - 100√3
         = 100(2 + √3 - √3)
         = 200 m
```

**Answer: The balloon rose 200 m**

---

## 📋 Summary Table

### Problem Types and Approaches

| Type | Given | Find | Key Formula |
|------|-------|------|-------------|
| Height | Distance, elevation angle | Height | h = d tan θ |
| Distance | Height, elevation angle | Distance | d = h cot θ |
| Observer at height | Two angles | Object height | Use both angles |
| Object on object | Two elevation angles | Top portion height | Difference of heights |
| Shadow | Shadow length, sun's angle | Height | h = s tan θ |
| Moving observer | Two angles, distance moved | Height | Simultaneous equations |

### Common Pitfalls

```
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   ⚠️ Common Mistakes:                                       │
    │                                                             │
    │   1. Forgetting to add observer's height to calculations   │
    │                                                             │
    │   2. Confusing angle of elevation with angle of depression │
    │                                                             │
    │   3. Not using alternate angles property correctly         │
    │                                                             │
    │   4. Incorrect triangle identification                     │
    │                                                             │
    │   5. Wrong trigonometric ratio (tan vs sin/cos)            │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

---

## ❓ Quick Revision Questions

1. **From a point 80 m from a tower's base, the angle of elevation of the top is 30°. Find the height.**

2. **A tower casts a 40 m shadow when the sun's altitude is 45°. Find the tower's height.**

3. **From the top of a 50 m cliff, the angle of depression of a boat is 60°. Find the distance of the boat from the cliff base.**

4. **The angles of elevation of the bottom and top of a flag on a 20 m pole are 30° and 45° from a point. Find the flag length.**

5. **A person walks 50 m towards a lamp post. The angle of elevation changes from 30° to 45°. Find the lamp post's height.**

6. **From a window 12 m above ground, the angles of elevation and depression of a tower's top and bottom are 45° and 30° respectively. Find the tower's height.**

<details>
<summary>Click to see answers</summary>

1. **Tower height:**
   h = 80 × tan 30° = 80/√3 = **80√3/3 ≈ 46.2 m**

2. **Tower height from shadow:**
   h = 40 × tan 45° = 40 × 1 = **40 m**

3. **Distance of boat:**
   d = 50/tan 60° = 50/√3 = **50√3/3 ≈ 28.87 m**

4. **Flag length:**
   Let distance = d
   
   tan 30° = 20/d → d = 20√3 m
   
   tan 45° = (20 + flag)/d
   1 = (20 + flag)/(20√3)
   20√3 = 20 + flag
   flag = 20√3 - 20 = **20(√3 - 1) ≈ 14.64 m**

5. **Lamp post height:**
   Let remaining distance = d
   
   tan 45° = h/d → h = d
   tan 30° = h/(d + 50) → h = (d + 50)/√3
   
   d = (d + 50)/√3
   d√3 = d + 50
   d(√3 - 1) = 50
   d = 50/(√3 - 1) = 25(√3 + 1)
   
   h = d = **25(√3 + 1) ≈ 68.3 m**

6. **Tower height:**
   From depression 30°: d = 12/tan 30° = 12√3 m
   
   From elevation 45°: h₂ = d × tan 45° = 12√3 m
   
   Tower height = 12 + 12√3 ≈ **32.78 m**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 9.1 Angles of Elevation and Depression](01-angles-elevation-depression.md) | [Unit 9 Index](README.md) | [9.3 Problems from Two Points →](03-two-observation-points.md) |

---

[← Back to Main Index](../README.md)

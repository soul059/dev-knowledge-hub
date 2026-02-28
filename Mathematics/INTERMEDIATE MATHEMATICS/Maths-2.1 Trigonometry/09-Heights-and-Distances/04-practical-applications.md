# Chapter 9.4: Practical Applications

## Overview

This chapter brings together all the concepts learned to solve real-world problems in surveying, navigation, engineering, astronomy, and everyday situations. These applications demonstrate the power and utility of trigonometry in measuring inaccessible heights and distances.

---

## 📐 Application 1: Surveying

### Measuring Land and Terrain

Surveyors use theodolites and total stations to measure angles precisely. Combined with known baseline distances, they can map entire regions.

### Example: Triangulation

**A surveyor needs to find the width of a river. She sets up at point A and sights a tree T on the opposite bank. She then walks 100 m along the bank to point B and measures ∠ABT = 60°. At A, ∠TAB = 90°. Find the river width.**

```
              T (tree)
              |
              | river width = w
              |
    A─────────+────
    |\90°
    | \
    |  \
    |60°\
    B─────────
      100 m
    
    Since ∠TAB = 90°, the river bank is perpendicular to AT.
```

**Solution:**

```
    In right triangle ABT:
    tan 60° = AT/AB
    √3 = w/100
    w = 100√3 ≈ 173.2 m
    
    Wait, let me reconsider the geometry.
    
    Actually, if ∠ABT = 60° and ∠TAB = 90°:
    Then ∠ATB = 30°
    
    tan 60° = AT/AB... but AT is not the river width directly.
    
    In triangle ABT:
    tan 60° = AT/AB where AB = 100
    AT = 100 tan 60° = 100√3 m
    
    But the river width is the perpendicular distance from A to T.
    Since ∠TAB = 90°, AT IS the river width.
    
    River width = 100√3 ≈ 173.2 m
```

---

### Inaccessible Height Measurement

**A surveyor wants to find the height of a cliff. From point A, the angle of elevation of the cliff top is 45°. Moving 200 m towards the cliff to point B, the angle becomes 60°. Find the cliff height.**

```
                    C (cliff top)
                    |
                    |
                    | h
                    |
                    |
        45°   60°   |
    A─────────B─────D
        200m    d
```

**Solution:**

```
    From B: tan 60° = h/d → h = d√3 ... (1)
    From A: tan 45° = h/(d + 200) → h = d + 200 ... (2)
    
    From (1) and (2):
    d√3 = d + 200
    d(√3 - 1) = 200
    d = 200/(√3 - 1) = 100(√3 + 1) ≈ 273.2 m
    
    h = d√3 = 100(√3 + 1)√3 = 100(3 + √3)
    h = 300 + 100√3 ≈ 473.2 m
```

---

## 📐 Application 2: Navigation

### Bearing Problems

Bearings are measured clockwise from North, expressed as three-digit angles (e.g., 045°, 270°).

```
              N (000°)
              |
              |
    W ────────+──────── E (090°)
    (270°)    |
              |
              S (180°)
```

### Example: Ship Navigation

**A ship sails from port P on a bearing of 060° for 50 km to point A. It then changes course to bearing 150° and sails for 30 km to point B. Find the distance and bearing of B from P.**

```
                  N
                  |
                  | 60°
              A   |/
             /\   *
            /  \ /
         30/    *P
          / 150°
         /
        B
```

**Solution:**

```
    At A, the ship turns. The angle at A in triangle PAB:
    Exterior angle at A = 150° (new bearing)
    Interior angle = 180° - 150° + 60° = 90°? 
    
    Actually, the change of direction:
    Initial direction: 060° (from N)
    New direction: 150° (from N)
    
    Angle turned = 150° - 60° = 90° (turned right)
    
    So angle PAB = 180° - 90° = 90° (interior angle)
    
    Wait, let me recalculate:
    At A, coming from bearing 060°, the reverse bearing is 240°.
    New bearing is 150°.
    Turn angle = 240° - 150° = 90° (turned right)
    
    Interior angle at A = 180° - 90° = 90°... 
    
    Hmm, let me use coordinates instead.
    
    P at origin.
    A at: (50 sin 60°, 50 cos 60°) = (25√3, 25)
    
    From A, bearing 150° means direction (sin 150°, cos 150°) = (0.5, -√3/2)
    
    B at: A + 30(sin 150°, cos 150°)
        = (25√3 + 15, 25 - 15√3)
        = (25√3 + 15, 25 - 15√3)
        ≈ (58.3, -0.98)
    
    Distance PB = √[(25√3 + 15)² + (25 - 15√3)²]
    
    Let me compute:
    (25√3 + 15)² = 625(3) + 750√3 + 225 = 2100 + 750√3
    (25 - 15√3)² = 625 - 750√3 + 675 = 1300 - 750√3
    
    Sum = 2100 + 750√3 + 1300 - 750√3 = 3400
    
    PB = √3400 = 10√34 ≈ 58.3 km
    
    Bearing of B from P:
    tan θ = (25√3 + 15)/(25 - 15√3)
    
    Since x > 0 and y ≈ -1 (small negative), B is almost due east,
    slightly south.
    
    Bearing ≈ 090° + small angle ≈ 091°
```

---

### Aircraft Tracking

**An aircraft is flying at a constant altitude. At time t = 0, the angle of elevation from a ground station is 30°. After 10 seconds, it is 45°. After another 10 seconds, it is 60°. Find the aircraft's altitude and speed.**

```
        A₁────────────A₂────────────A₃
         \            |            /
          \           |           /
           \          |h         /
            \         |         /
             \30°  45°|60°     /
    ──────────────────O──────────────────
                      d₁    d₂    d₃
```

**Solution:**

```
    Let altitude = h, speed = v
    
    At t = 0: angle = 30°, horizontal distance from O = d₁
    tan 30° = h/d₁ → d₁ = h√3
    
    At t = 10s: angle = 45°, horizontal distance = d₂
    tan 45° = h/d₂ → d₂ = h
    
    At t = 20s: angle = 60°, horizontal distance = d₃
    tan 60° = h/d₃ → d₃ = h/√3
    
    Distance traveled in first 10s: d₁ - d₂ = h√3 - h = h(√3 - 1)
    Distance traveled in second 10s: d₂ - d₃ = h - h/√3 = h(1 - 1/√3)
    
    Since speed is constant:
    h(√3 - 1) = h(1 - 1/√3)  ... This should give a consistent h
    
    Actually, this gives:
    √3 - 1 = 1 - 1/√3
    √3 - 1 = (√3 - 1)/√3
    
    This would only work if √3 = 1, which is false.
    
    So the aircraft is not flying directly towards/away from O.
    The problem needs additional information or is flying at an angle.
    
    Let's assume it's flying directly towards O (decreasing angle scenario):
    
    Actually with increasing angles, the aircraft IS approaching.
    The issue is my equation. Let me reconsider.
    
    For constant speed, in 10 seconds the aircraft covers distance v × 10.
    
    d₁ - d₂ = 10v (first 10 seconds)
    d₂ - d₃ = 10v (second 10 seconds)
    
    h√3 - h = 10v → h(√3 - 1) = 10v ... (1)
    h - h/√3 = 10v → h(√3 - 1)/√3 = 10v ... (2)
    
    Dividing (1) by (2):
    √3 = √3 ✓
    
    So any h satisfies this... we need absolute values.
    
    Actually, in problems like this, we're usually given the final 
    answer or one more piece of information.
    
    Let's say the aircraft passes directly overhead eventually.
    Or we're told the first distance traveled.
```

---

## 📐 Application 3: Architecture and Construction

### Finding Building Height Without Direct Access

**An architect needs to find the height of an existing building. From point A, 100 m from the building, the angle of elevation of the top is 60°. The architect's eye level is 1.5 m above the ground. Find the building height.**

```
                    T (top)
                    |
                    |
                    | h - 1.5
                    |
    Eye level ──────+──────
                    |
                    | 1.5 m
                    |
        60°         |
    A───────────────B
          100 m
```

**Solution:**

```
    Height above eye level:
    tan 60° = (h - 1.5)/100
    √3 = (h - 1.5)/100
    h - 1.5 = 100√3
    h = 100√3 + 1.5
    h ≈ 173.2 + 1.5 = 174.7 m
```

---

### Ramp Design

**A wheelchair ramp must not exceed a 1:12 slope (rise:run). What is the maximum angle of elevation?**

```
    tan θ = 1/12
    θ = tan⁻¹(1/12)
    θ ≈ 4.76°
```

**If a building entrance is 0.6 m above ground, what minimum ramp length is needed?**

```
    Rise = 0.6 m, Slope = 1:12
    Run = 12 × 0.6 = 7.2 m
    
    Ramp length = √(0.6² + 7.2²) = √(0.36 + 51.84) = √52.2 ≈ 7.22 m
```

---

## 📐 Application 4: Astronomy

### Finding the Distance to a Star (Parallax)

```
    As Earth orbits the Sun, nearby stars appear to shift 
    against distant background stars.
    
                      Star
                       *
                      /|\
                     / | \
                    /  |  \
                   /   |   \
                  /    |    \
               E₁──────S──────E₂
                   Earth's orbit
    
    The parallax angle p is half the total shift.
    Distance = 1/p (in parsecs, if p is in arcseconds)
```

### Solar Eclipse Calculation

**During a solar eclipse, the Moon (diameter 3,474 km, distance 384,400 km) exactly covers the Sun (diameter 1,392,000 km). Find the Sun's distance.**

```
    Using similar triangles:
    
    Moon diameter / Moon distance = Sun diameter / Sun distance
    
    3474/384400 = 1392000/d
    d = 1392000 × 384400/3474
    d ≈ 154,000,000 km ≈ 1.54 × 10⁸ km
    
    (Actual average distance: 1.496 × 10⁸ km)
```

---

## 📐 Application 5: Everyday Problems

### Finding Height of a Tree (Shadow Method)

**A 2 m pole casts a 1.5 m shadow at the same time a tree casts a 15 m shadow. Find the tree's height.**

```
    Since the sun's angle is the same:
    
    Tree height / Tree shadow = Pole height / Pole shadow
    
    h/15 = 2/1.5
    h = 15 × 2/1.5
    h = 20 m
```

### Ladder Safety

**A ladder should be placed at an angle between 70° and 80° for safety. If a ladder is 6 m long, how far from the wall should the base be?**

```
    For 75° (middle of safe range):
    
              Wall
                |
                |\
                | \
                |  \ 6 m (ladder)
                |   \
                |75° \
    ────────────+─────────
                  d
    
    cos 75° = d/6
    d = 6 × cos 75°
    d = 6 × 0.259
    d ≈ 1.55 m
    
    Safe range: approximately 1.0 m to 2.1 m from wall
```

### Flagpole Problem

**A flagpole is mounted on top of a building. From a point 100 m from the building, the angles of elevation of the top and bottom of the flagpole are 60° and 45°. Find the height of the flagpole.**

```
    Building height: h₁ = 100 × tan 45° = 100 m
    Total height: H = 100 × tan 60° = 100√3 m
    
    Flagpole height: H - h₁ = 100√3 - 100 = 100(√3 - 1) ≈ 73.2 m
```

---

## 📝 Mixed Problem Set

### Problem 1: Lighthouse and Boats

**From the top of a 100 m lighthouse, the angles of depression of two boats in a line with the lighthouse are 30° and 45°. Find the distance between the boats.**

**Solution:**

```
                    L (lighthouse top)
         Horizontal ────────────────
                    |\30°\45°
                    | \    \
                100m|  \    \
                    |   \    \
                    +────\────\────
                    B    B₁   B₂
    
    Distance to B₁ (angle 45°): d₁ = 100/tan 45° = 100 m
    Distance to B₂ (angle 30°): d₂ = 100/tan 30° = 100√3 m
    
    Distance between boats = d₂ - d₁ = 100√3 - 100 = 100(√3 - 1) ≈ 73.2 m
```

---

### Problem 2: Mountain Peak

**From a point on the ground, the angle of elevation of a mountain peak is 30°. After walking 1 km towards the mountain on a 10° inclined slope, the angle of elevation becomes 60°. Find the height of the peak above the starting point.**

**Solution:**

```
    This is more complex due to the inclined path.
    
    Let's set up coordinates with the starting point at origin.
    
    After walking 1 km at 10° incline:
    - Horizontal distance covered: 1 × cos 10° ≈ 0.985 km
    - Vertical rise: 1 × sin 10° ≈ 0.174 km
    
    Let the peak be at horizontal distance x from start, height h.
    
    From start: tan 30° = h/x → h = x/√3 ... (1)
    
    From new position: tan 60° = (h - 0.174)/(x - 0.985)
    √3 = (h - 0.174)/(x - 0.985)
    h - 0.174 = √3(x - 0.985) ... (2)
    
    Substituting (1) into (2):
    x/√3 - 0.174 = √3x - 0.985√3
    x/√3 - √3x = 0.174 - 0.985√3
    x(1/√3 - √3) = 0.174 - 1.706
    x(-2/√3) = -1.532
    x = 1.532 × √3/2 = 0.766√3 ≈ 1.327 km
    
    h = x/√3 = 0.766 km = 766 m
```

---

## 📋 Summary Table

### Application Areas

| Field | Typical Problems | Key Techniques |
|-------|-----------------|----------------|
| Surveying | Land measurement, mapping | Triangulation, baseline |
| Navigation | Bearings, distances | Sine/Cosine Rules |
| Construction | Heights, slopes, angles | Elevation/depression |
| Astronomy | Distances, sizes | Similar triangles, parallax |
| Aviation | Altitude, tracking | Multiple observations |
| Everyday | Shadows, ladders, ramps | Basic trig ratios |

### Quick Reference Formulas

| Situation | Formula |
|-----------|---------|
| Height from distance and angle | h = d tan θ |
| Distance from height and angle | d = h cot θ |
| Shadow problems | h₁/s₁ = h₂/s₂ |
| Two-point same side | h = d tan α tan β/(tan β - tan α) |
| Two-point opposite sides | h = d tan α tan β/(tan α + tan β) |

---

## ❓ Quick Revision Questions

1. **A 2 m stick casts a 3 m shadow. What is the sun's angle of elevation?**

2. **From the top of a 60 m cliff, two ships are seen at angles of depression 30° and 60°. If they are in line with the cliff base, find the distance between them.**

3. **A surveyor needs to find the width of a canyon. Standing at point A on one edge, she measures the angle of depression to the opposite edge as 35°. Her theodolite is 1.5 m above the ground, and the opposite edge is 50 m below her ground level. Find the canyon width.**

4. **A plane is flying at 3000 m altitude. From the ground, its angle of elevation is 60°. One minute later, the angle is 30° and the plane has passed overhead. Find the plane's speed in km/h.**

5. **Design problem: A zip line needs to go from a 25 m high platform to a point 100 m away horizontally at ground level. What angle does the zip line make with the horizontal?**

<details>
<summary>Click to see answers</summary>

1. **Sun's angle of elevation:**
   tan θ = 2/3
   θ = tan⁻¹(2/3) ≈ **33.69°**

2. **Distance between ships:**
   Distance to closer ship (60° depression): d₁ = 60/tan 60° = 60/√3 = 20√3 m
   Distance to farther ship (30° depression): d₂ = 60/tan 30° = 60√3 m
   
   Distance between = d₂ - d₁ = 60√3 - 20√3 = **40√3 ≈ 69.3 m**

3. **Canyon width:**
   Total vertical drop = 50 + 1.5 = 51.5 m
   
   tan 35° = 51.5/width
   width = 51.5/tan 35°
   width = 51.5/0.700 ≈ **73.6 m**

4. **Plane speed:**
   At 60°: horizontal distance = 3000/tan 60° = 3000/√3 = 1000√3 m
   At 30°: horizontal distance = 3000/tan 30° = 3000√3 m (other side)
   
   Total distance = 1000√3 + 3000√3 = 4000√3 m
   Time = 1 minute = 1/60 hour
   
   Speed = 4000√3 m/min = 4000√3 × 60 m/h = 4√3 km × 60/h
   Speed = 240√3 km/h ≈ **415.7 km/h**

5. **Zip line angle:**
   tan θ = 25/100 = 0.25
   θ = tan⁻¹(0.25) ≈ **14.04°**
   
   The zip line makes about 14° with the horizontal.

</details>

---

## 🎓 Final Summary: Complete Trigonometry Course

Congratulations on completing this comprehensive trigonometry study guide! You have learned:

1. **Unit 1-2**: Foundation concepts, unit circle, and trigonometric functions
2. **Unit 3-5**: Identities, compound angles, and multiple angle formulas
3. **Unit 6**: Solving trigonometric equations
4. **Unit 7**: Inverse trigonometric functions
5. **Unit 8**: Properties of triangles (Sine Rule, Cosine Rule, Area, Radii)
6. **Unit 9**: Practical applications in heights and distances

These skills form the foundation for advanced mathematics, physics, engineering, and many other fields.

---

## Navigation

| Previous | Up | Course Complete |
|----------|-------|-----------------|
| [← 9.3 Problems from Two Points](03-two-observation-points.md) | [Unit 9 Index](README.md) | [🎓 Return to Main Index](../README.md) |

---

[← Back to Main Index](../README.md)

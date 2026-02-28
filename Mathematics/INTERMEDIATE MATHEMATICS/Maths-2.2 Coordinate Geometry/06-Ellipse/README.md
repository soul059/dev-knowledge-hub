# Unit 6: Ellipse

## 📚 Unit Overview

The **ellipse** is a conic section formed when a plane cuts a cone at an angle. It's the locus of points where the sum of distances from two fixed points (foci) is constant. Ellipses describe planetary orbits, whispering galleries, and many engineering applications.

---

## 🎯 Learning Objectives

By the end of this unit, you will be able to:

- Understand the definition and derive standard equations of ellipse
- Calculate eccentricity and relate it to ellipse shape
- Identify major axis, minor axis, foci, directrices, and vertices
- Apply parametric representation for points on ellipse
- Derive and apply tangent and normal equations
- Understand auxiliary circle and eccentric angle

---

## 📖 Chapter List

| Chapter | Topic | Key Concepts |
|:-------:|-------|--------------|
| 6.1 | [Standard Equation of Ellipse](01-standard-equation.md) | Definition, two forms, comparison |
| 6.2 | [Eccentricity and Elements](02-eccentricity-elements.md) | e, foci, vertices, directrices, latus rectum |
| 6.3 | [Parametric Form](03-parametric-form.md) | Auxiliary circle, eccentric angle |
| 6.4 | [Position of Point and Line](04-position-point-line.md) | Inside/outside, chord, secant, tangent |
| 6.5 | [Equation of Tangent](05-equation-of-tangent.md) | Point form, parametric, slope form |
| 6.6 | [Equation of Normal](06-equation-of-normal.md) | Normal equations, properties |

---

## 📐 Ellipse at a Glance

```
                         Minor Axis
                            │
            B●──────────────│──────────────●B'
            ╱               │               ╲
           ╱                │                ╲
          ╱                 │                 ╲
    ─────●F'────────────────●C─────────────────●F─────
        ╱                   │                   ╲
       ╱                    │                    ╲
      ╱                     │                     ╲
     ●──────────────────────│──────────────────────●
    A'                      │                       A
                            │
                       Major Axis
                       
    C = Center (origin)
    A, A' = Vertices on major axis  
    B, B' = Vertices on minor axis
    F, F' = Foci
    For any point P: PF + PF' = 2a (constant)
```

---

## ⚡ Quick Reference: Standard Forms

| Form | Equation | Major Axis | $a^2$ | $b^2$ |
|------|----------|------------|-------|-------|
| Horizontal | $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ (a > b) | Along x-axis | Under $x^2$ | Under $y^2$ |
| Vertical | $\frac{x^2}{b^2} + \frac{y^2}{a^2} = 1$ (a > b) | Along y-axis | Under $y^2$ | Under $x^2$ |

---

## 📝 Key Formulas Summary

| Property | Formula |
|----------|---------|
| Eccentricity | $e = \frac{c}{a}$ where $c^2 = a^2 - b^2$ |
| Relation | $b^2 = a^2(1 - e^2)$ |
| Foci (horizontal) | $(\pm ae, 0) = (\pm c, 0)$ |
| Directrices (horizontal) | $x = \pm \frac{a}{e}$ |
| Latus rectum | $\frac{2b^2}{a}$ |
| Parametric point | $(a\cos\theta, b\sin\theta)$ |
| Sum of focal distances | $2a$ |
| Tangent at $(x_1, y_1)$ | $\frac{xx_1}{a^2} + \frac{yy_1}{b^2} = 1$ |
| Tangent with slope $m$ | $y = mx \pm \sqrt{a^2m^2 + b^2}$ |

---

## 🔍 Comparison: Circle vs Ellipse vs Parabola

| Property | Circle | Ellipse | Parabola |
|----------|--------|---------|----------|
| Eccentricity | $e = 0$ | $0 < e < 1$ | $e = 1$ |
| Definition | Equal distance from center | Sum of distances from foci = constant | Equal distance from focus & directrix |
| Symmetry | Infinite axes | Two axes | One axis |
| Equation | $x^2 + y^2 = r^2$ | $\frac{x^2}{a^2} + \frac{y^2}{b^2} = 1$ | $y^2 = 4ax$ |
| Center/Vertex | Center | Center | Vertex |
| Special case | Circle is ellipse with $a = b$ | - | - |

---

## 💡 Study Tips

1. **a is always larger**: In standard form, $a > b$ always, determining major axis
2. **Eccentricity significance**: $e$ close to 0 → nearly circular; $e$ close to 1 → very elongated
3. **Relation $b^2 = a^2 - c^2$**: Derived from definition; crucial for calculations
4. **Auxiliary circle**: Helps visualize parametric form
5. **Compare with circle**: Many formulas are extensions of circle formulas

---

## 🌍 Real-World Applications

- **Planetary orbits**: Planets follow elliptical paths (Kepler's laws)
- **Satellite orbits**: Elliptical trajectories around Earth
- **Whispering galleries**: Sound focuses at foci of elliptical rooms
- **Medical imaging**: Lithotripters use elliptical reflectors
- **Architecture**: Elliptical arches and domes
- **Optics**: Elliptical mirrors in telescopes

---

## 📖 Prerequisites

Before studying this unit, ensure you're comfortable with:

- [x] Parabola equations and properties
- [x] Distance formula and midpoint
- [x] Basic trigonometry
- [x] Differentiation for tangent slopes

---

## ⏭️ Navigation

| [← Previous: Parabola](../05-Parabola/README.md) | [Course Home](../README.md) | [Start Chapter 6.1 →](01-standard-equation.md) |
|:-------------------------------------------------|:---------------------------:|-----------------------------------------------:|

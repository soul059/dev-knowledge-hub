# Unit 8: Three-Dimensional Coordinate Geometry

## 📚 Unit Overview

**Three-dimensional (3D) coordinate geometry** extends the concepts of 2D coordinate geometry to three-dimensional space. This unit introduces the coordinate system, direction cosines and ratios, and equations of lines and planes in 3D.

---

## 🎯 Learning Objectives

After completing this unit, you will be able to:

- Work with 3D coordinate systems and locate points in space
- Calculate distances and section formulas in 3D
- Understand and apply direction cosines and direction ratios
- Derive and use equations of straight lines in 3D
- Find equations of planes in various forms
- Calculate angles between lines and planes
- Determine distances of points from lines and planes

---

## 📋 Unit Contents

| Chapter | Topic | Description |
|:-------:|-------|-------------|
| 1 | [3D Coordinate System](01-coordinate-system.md) | Axes, octants, coordinates, distance formula |
| 2 | [Section Formula](02-section-formula.md) | Internal/external division, centroid, collinearity |
| 3 | [Direction Cosines & Ratios](03-direction-cosines.md) | Direction cosines, ratios, angle between lines |
| 4 | [Straight Line in 3D](04-straight-line.md) | Equations of line, symmetric form, parametric form |
| 5 | [Equation of Plane](05-equation-of-plane.md) | Various forms, angle between planes, distance |
| 6 | [Line and Plane](06-line-and-plane.md) | Intersection, coplanarity, skew lines |

---

## 📐 Visual Overview: 3D Coordinate System

```
                z
                |
                |
                |
                |_________ y
               /
              /
             /
            x
            
   The three mutually perpendicular axes
   meeting at origin O form right-handed system
```

### The Eight Octants

```
        z
        |
   II   |   I     (upper octants)
        |
   -----+------- y
        |
   III  |   IV    (lower octants)
       /
      x
      
   Each quadrant in xy-plane splits into 2 octants
   (above and below xy-plane)
```

---

## 🔑 Key Formulas

### Distance and Section

| Formula | Expression |
|---------|------------|
| Distance | $d = \sqrt{(x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2}$ |
| Section (Internal) | $\left(\frac{mx_2+nx_1}{m+n}, \frac{my_2+ny_1}{m+n}, \frac{mz_2+nz_1}{m+n}\right)$ |
| Midpoint | $\left(\frac{x_1+x_2}{2}, \frac{y_1+y_2}{2}, \frac{z_1+z_2}{2}\right)$ |
| Centroid | $\left(\frac{x_1+x_2+x_3}{3}, \frac{y_1+y_2+y_3}{3}, \frac{z_1+z_2+z_3}{3}\right)$ |

### Direction Cosines

| Property | Formula |
|----------|---------|
| Direction cosines | $\cos\alpha = l$, $\cos\beta = m$, $\cos\gamma = n$ |
| Fundamental relation | $l^2 + m^2 + n^2 = 1$ |
| From DRs $(a, b, c)$ | $l = \frac{a}{\sqrt{a^2+b^2+c^2}}$, etc. |
| Angle between lines | $\cos\theta = l_1l_2 + m_1m_2 + n_1n_2$ |

### Equations of Line

| Form | Equation |
|------|----------|
| Symmetric | $\frac{x-x_1}{a} = \frac{y-y_1}{b} = \frac{z-z_1}{c}$ |
| Parametric | $x = x_1 + at$, $y = y_1 + bt$, $z = z_1 + ct$ |
| Two-point | $\frac{x-x_1}{x_2-x_1} = \frac{y-y_1}{y_2-y_1} = \frac{z-z_1}{z_2-z_1}$ |

### Equations of Plane

| Form | Equation |
|------|----------|
| General | $ax + by + cz + d = 0$ |
| Normal | $lx + my + nz = p$ |
| Intercept | $\frac{x}{a} + \frac{y}{b} + \frac{z}{c} = 1$ |
| Point-normal | $a(x-x_1) + b(y-y_1) + c(z-z_1) = 0$ |

---

## 📊 Key Concepts at a Glance

### Coordinate Planes

| Plane | Equation | Coordinate = 0 |
|-------|----------|----------------|
| xy-plane | $z = 0$ | z |
| yz-plane | $x = 0$ | x |
| xz-plane | $y = 0$ | y |

### Distance Formulas

| Distance | Formula |
|----------|---------|
| Point to point | $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2}$ |
| Point to plane | $\frac{|ax_1 + by_1 + cz_1 + d|}{\sqrt{a^2 + b^2 + c^2}}$ |
| Between planes | $\frac{|d_1 - d_2|}{\sqrt{a^2 + b^2 + c^2}}$ |

---

## 🔄 2D vs 3D Comparison

| Property | 2D | 3D |
|----------|----|----|
| Coordinates | $(x, y)$ | $(x, y, z)$ |
| Axes | 2 | 3 |
| Quadrants/Octants | 4 quadrants | 8 octants |
| Distance | $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$ | $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2 + (z_2-z_1)^2}$ |
| Line equation | $ax + by + c = 0$ | Symmetric/parametric form |
| Plane equation | N/A | $ax + by + cz + d = 0$ |

---

## 🌍 Applications

1. **Aviation**: Flight paths and navigation in 3D space
2. **Computer Graphics**: 3D modeling and rendering
3. **Physics**: Particle motion and field analysis
4. **Engineering**: Structural analysis and CAD
5. **Robotics**: Arm movement and positioning
6. **Astronomy**: Celestial object locations

---

## ⏭️ Navigation

| [← Previous Unit: Hyperbola](../07-Hyperbola/README.md) | [Home](../README.md) | [Next Chapter: Coordinate System →](01-coordinate-system.md) |
|:-------------------------------------------------------|:--------------------:|------------------------------------------------------------:|

---

## 📖 Study Tips

1. **Visualize**: Always try to sketch the 3D scenario
2. **Right-hand Rule**: Use for determining axis directions
3. **Build on 2D**: Many formulas are natural extensions of 2D
4. **Practice Projections**: Understanding projections helps visualization
5. **Check Units**: Each coordinate should have same units

---

*Welcome to the fascinating world of Three-Dimensional Geometry!*

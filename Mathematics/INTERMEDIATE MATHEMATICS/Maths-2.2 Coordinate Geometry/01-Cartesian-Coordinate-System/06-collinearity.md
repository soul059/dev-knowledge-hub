# Chapter 1.6: Collinearity

## 📚 Chapter Overview

**Collinearity** refers to the property of three or more points lying on the same straight line. Testing for collinearity is a fundamental skill in coordinate geometry with applications in proving geometric theorems and solving practical problems.

---

## 📝 Definition

> **Collinear Points**: Three or more points are said to be **collinear** if they all lie on the same straight line.

```
    Collinear Points               Non-Collinear Points
    
          ●────────●────────●              ●
          A        B        C             A \
                                             \    ●C
    A, B, C lie on the same line             ●B
```

---

## 📝 Methods to Test Collinearity

### Method 1: Using Area of Triangle

Three points $A(x_1, y_1)$, $B(x_2, y_2)$, and $C(x_3, y_3)$ are collinear if and only if the area of triangle ABC is **zero**.

> **Condition for Collinearity**:
>
> $$\boxed{x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2) = 0}$$

**Or equivalently:**

$$\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix} = 0$$

---

### Method 2: Using Slope

Three points A, B, C are collinear if:

$$\text{Slope of AB} = \text{Slope of BC} = \text{Slope of AC}$$

> **Slope Condition**:
>
> $$\boxed{\frac{y_2 - y_1}{x_2 - x_1} = \frac{y_3 - y_2}{x_3 - x_2}}$$

```
    Collinear: Same slope throughout
    
         C ●
          /
         /   slope = m (same)
        /
       ● B
      /
     /   slope = m (same)
    /
   ● A
```

---

### Method 3: Using Distance Formula

Three points A, B, C are collinear if one of them divides the segment joining the other two. This means:

$$AB + BC = AC$$

(where C is between A and B, or one of the similar relationships holds)

> **Distance Condition**: The sum of two shorter distances equals the longest distance.

---

### Method 4: Using Section Formula

Point B$(x_2, y_2)$ lies on line joining A$(x_1, y_1)$ and C$(x_3, y_3)$ if B divides AC in some ratio $k:1$.

$$x_2 = \frac{kx_3 + x_1}{k + 1} \quad \text{and} \quad y_2 = \frac{ky_3 + y_1}{k + 1}$$

If a consistent value of $k$ exists, the points are collinear.

---

## ✏️ Worked Examples

### Example 1: Using Area Method

**Problem**: Check if points A(1, 2), B(3, 4), and C(5, 6) are collinear.

**Solution**:

Using the collinearity condition:
$$x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2)$$

$$= 1(4 - 6) + 3(6 - 2) + 5(2 - 4)$$

$$= 1(-2) + 3(4) + 5(-2)$$

$$= -2 + 12 - 10 = 0$$

Since the result is **0**, the points are **collinear**.

```
    Y
    ↑
  6 │         ●C(5, 6)
    │        /
  4 │     ●B(3, 4)
    │    /
  2 │  ●A(1, 2)
    │ /
    O────────────→ X
        1 3 5
```

---

### Example 2: Using Slope Method

**Problem**: Show that points P(−2, 3), Q(4, 6), and R(10, 9) are collinear.

**Solution**:

**Slope of PQ:**
$$m_{PQ} = \frac{6 - 3}{4 - (-2)} = \frac{3}{6} = \frac{1}{2}$$

**Slope of QR:**
$$m_{QR} = \frac{9 - 6}{10 - 4} = \frac{3}{6} = \frac{1}{2}$$

Since $m_{PQ} = m_{QR} = \frac{1}{2}$, the points are **collinear**.

---

### Example 3: Finding Unknown for Collinearity

**Problem**: Find the value of k if points (1, 2), (k, 0), and (4, −3) are collinear.

**Solution**:

Using the collinearity condition:
$$1(0 - (-3)) + k((-3) - 2) + 4(2 - 0) = 0$$

$$1(3) + k(-5) + 4(2) = 0$$

$$3 - 5k + 8 = 0$$

$$11 - 5k = 0$$

$$k = \frac{11}{5} = 2.2$$

**Answer**: $k = \frac{11}{5}$

---

### Example 4: Using Distance Method

**Problem**: Verify that points A(1, 1), B(4, 4), and C(7, 7) are collinear using the distance formula.

**Solution**:

$$AB = \sqrt{(4-1)^2 + (4-1)^2} = \sqrt{9 + 9} = \sqrt{18} = 3\sqrt{2}$$

$$BC = \sqrt{(7-4)^2 + (7-4)^2} = \sqrt{9 + 9} = \sqrt{18} = 3\sqrt{2}$$

$$AC = \sqrt{(7-1)^2 + (7-1)^2} = \sqrt{36 + 36} = \sqrt{72} = 6\sqrt{2}$$

Check: $AB + BC = 3\sqrt{2} + 3\sqrt{2} = 6\sqrt{2} = AC$ ✓

Since $AB + BC = AC$, point B lies between A and C, confirming **collinearity**.

---

### Example 5: Collinearity in Context

**Problem**: The vertices of a triangle are A(3, 4), B(6, 2), and C(−3, 16). Prove that the centroid, G, lies on the line joining A and the midpoint of BC.

**Solution**:

**Step 1: Find centroid G**
$$G = \left(\frac{3 + 6 + (-3)}{3}, \frac{4 + 2 + 16}{3}\right) = \left(\frac{6}{3}, \frac{22}{3}\right) = \left(2, \frac{22}{3}\right)$$

**Step 2: Find midpoint M of BC**
$$M = \left(\frac{6 + (-3)}{2}, \frac{2 + 16}{2}\right) = \left(\frac{3}{2}, 9\right)$$

**Step 3: Check if A, G, M are collinear**

Using the collinearity condition with $A(3, 4)$, $G(2, \frac{22}{3})$, $M(\frac{3}{2}, 9)$:

$$= 3\left(\frac{22}{3} - 9\right) + 2\left(9 - 4\right) + \frac{3}{2}\left(4 - \frac{22}{3}\right)$$

$$= 3\left(\frac{22 - 27}{3}\right) + 2(5) + \frac{3}{2}\left(\frac{12 - 22}{3}\right)$$

$$= 3\left(\frac{-5}{3}\right) + 10 + \frac{3}{2}\left(\frac{-10}{3}\right)$$

$$= -5 + 10 - 5 = 0$$

Since the result is 0, points A, G, and M are **collinear** (which confirms that the centroid lies on the median from A).

---

### Example 6: Three Points with Parameter

**Problem**: For what value of m are the points (3, 5), (m, 1), and (1, −1) collinear?

**Solution**:

**Using Slope Method:**

Slope of line joining (3, 5) and (1, −1):
$$m_1 = \frac{-1 - 5}{1 - 3} = \frac{-6}{-2} = 3$$

For collinearity, slope from (3, 5) to (m, 1) must also be 3:
$$\frac{1 - 5}{m - 3} = 3$$

$$\frac{-4}{m - 3} = 3$$

$$-4 = 3(m - 3)$$

$$-4 = 3m - 9$$

$$3m = 5$$

$$m = \frac{5}{3}$$

**Answer**: $m = \frac{5}{3}$

---

## 📝 Extension: Collinearity of More Than Three Points

To check if four or more points are collinear:
1. Take any three points and check collinearity
2. If collinear, check if the fourth point also satisfies the line equation
3. Repeat for all points

> **All points on a line** can be represented by: $y = mx + c$ for some constants $m$ and $c$.

---

## 📝 Applications of Collinearity

### 1. Proving Points on a Line

Many geometry problems require proving that certain special points (like centroid, orthocenter, etc.) lie on a line.

### 2. Menelaus' Theorem

If a transversal cuts the sides BC, CA, AB of a triangle at D, E, F respectively, then D, E, F are collinear if:

$$\frac{BD}{DC} \cdot \frac{CE}{EA} \cdot \frac{AF}{FB} = -1$$

### 3. Euler Line

In any triangle, the centroid, circumcenter, and orthocenter are collinear (they lie on the **Euler Line**).

```
        A
        ●
       /|\
      / | \
     /  |  \
    /   ●H  \     H = Orthocenter
   /    |    \    G = Centroid  
  /     ●G    \   O = Circumcenter
 /      |      \
●───────●───────●
B       O       C

H, G, O are collinear (Euler Line)
```

---

## 📝 Common Mistakes to Avoid

| Mistake | Correct Approach |
|---------|------------------|
| Forgetting that the condition is "= 0" | Always check if expression equals zero |
| Calculation errors with negative numbers | Be careful with signs |
| Not considering all possible orderings | The formula works regardless of order |
| Assuming collinearity without proof | Always verify mathematically |

---

## 💡 Tips and Insights

> 💡 **Quick Check**: If all points have the same x-coordinate, they lie on a vertical line. If all have the same y-coordinate, they lie on a horizontal line.

> 💡 **Slope Tip**: If any two points have the same x-coordinate (vertical segment), use the area method instead of slope.

> ⚠️ **Division by Zero**: Be careful when using slope method — if x-coordinates are equal, slope is undefined.

> 💡 **Determinant Memory**: The determinant form equals zero for collinear points.

---

## 🌍 Real-World Applications

1. **Quality Control**: Checking if manufactured parts are aligned
2. **Surveying**: Verifying that boundary markers are in line
3. **Computer Graphics**: Determining if points lie on a line for simplification
4. **Navigation**: Checking if waypoints are aligned on a straight path
5. **Architecture**: Ensuring structural elements are properly aligned
6. **Astronomy**: Determining if celestial objects appear in a line from Earth

---

## 📋 Summary Table

| Method | Condition for Collinearity |
|--------|---------------------------|
| Area Method | $x_1(y_2 - y_3) + x_2(y_3 - y_1) + x_3(y_1 - y_2) = 0$ |
| Determinant | $\begin{vmatrix} x_1 & y_1 & 1 \\ x_2 & y_2 & 1 \\ x_3 & y_3 & 1 \end{vmatrix} = 0$ |
| Slope Method | $\frac{y_2 - y_1}{x_2 - x_1} = \frac{y_3 - y_2}{x_3 - x_2}$ |
| Distance Method | $AB + BC = AC$ (or similar) |
| Section Formula | Point divides line in some ratio $k:1$ |

---

## ❓ Quick Revision Questions

1. **Check if points (2, 1), (4, 3), and (6, 5) are collinear.**

2. **Find the value of k for which (k, 2), (3, 4), and (5, 6) are collinear.**

3. **Prove that the points (a, 0), (0, b), and (1, 1) are collinear if $\frac{1}{a} + \frac{1}{b} = 1$.**

4. **Are the points (0, 0), (5, 3), and (10, 6) collinear?**

5. **The midpoint of line segment joining (2, 3) and (4, 7) is M. Show that (0, −1), M, and (6, 11) are collinear.**

6. **Find the value of p if the points (−1, 3), (2, p), and (5, −1) are collinear.**

<details>
<summary><b>Click to see answers</b></summary>

1. Using slope: $\frac{3-1}{4-2} = \frac{2}{2} = 1$ and $\frac{5-3}{6-4} = \frac{2}{2} = 1$
   Same slope, so **collinear**.

2. Using collinearity condition:
   $k(4-6) + 3(6-2) + 5(2-4) = 0$
   $-2k + 12 - 10 = 0$
   $k = 1$

3. Area condition: $a(b-1) + 0(1-0) + 1(0-b) = 0$
   $ab - a - b = 0$
   $ab = a + b$
   Dividing by $ab$: $1 = \frac{1}{b} + \frac{1}{a}$
   Hence proved.

4. Slope from (0,0) to (5,3) = $\frac{3}{5}$
   Slope from (5,3) to (10,6) = $\frac{3}{5}$
   Same slope, so **collinear**.

5. M = $\left(\frac{2+4}{2}, \frac{3+7}{2}\right) = (3, 5)$
   Check collinearity of (0, −1), (3, 5), (6, 11):
   $0(5-11) + 3(11-(-1)) + 6((-1)-5) = 0 + 36 - 36 = 0$
   **Collinear**.

6. $(-1)(p-(-1)) + 2((-1)-3) + 5(3-p) = 0$
   $(-1)(p+1) + 2(-4) + 5(3-p) = 0$
   $-p - 1 - 8 + 15 - 5p = 0$
   $-6p + 6 = 0$
   $p = 1$

</details>

---

## ⏭️ Navigation

| [← Previous: Area of Triangle](05-area-of-triangle.md) | [Back to Unit 1 →](README.md) |
|:------------------------------------------------------|-----------------------------:|

| [Next Unit: Straight Lines →](../02-Straight-Lines/README.md) |
|:-------------------------------------------------------------|

# 3.5 Complex Power Functions

[← Previous: Logarithmic Function](04-logarithmic-function.md) | [Next: Multi-valued Functions →](06-multi-valued-functions.md)

---

## Chapter Overview

Complex powers $z^c$ (where both $z$ and $c$ may be complex) are defined via the exponential and logarithm: $z^c = e^{c\log z}$. Since $\log z$ is multi-valued, complex powers are generally multi-valued too. The number of distinct values depends on whether $c$ is an integer, rational, or irrational/complex.

---

## 1. Definition

For $z \ne 0$ and $c \in \mathbb{C}$:

$$\boxed{z^c = e^{c\,\log z} = e^{c[\ln|z| + i(\text{Arg}(z) + 2k\pi)]}}, \quad k \in \mathbb{Z}$$

### 1.1 Principal Value

$$\text{p.v.}\; z^c = e^{c\,\text{Log}(z)}$$

---

## 2. Cases by Type of Exponent

| Exponent $c$ | Number of Values | Example |
|-------------|-----------------|---------|
| $c = n \in \mathbb{Z}$ | 1 (single-valued) | $z^3$ |
| $c = p/q \in \mathbb{Q}$ ($\gcd(p,q)=1$) | $q$ distinct values | $z^{2/3}$ has 3 values |
| $c \in \mathbb{R} \setminus \mathbb{Q}$ | $\infty$ values | $z^{\sqrt{2}}$ |
| $c \in \mathbb{C} \setminus \mathbb{R}$ | $\infty$ values | $z^i$ |

### 2.1 Why?

The different values of $z^c$ are $e^{c(\text{Log}(z) + 2k\pi i)}$. Two values for $k_1, k_2$ coincide iff $e^{2\pi i c(k_1-k_2)} = 1$, i.e., $c(k_1-k_2) \in \mathbb{Z}$.

- If $c = n$ (integer): always an integer → single-valued
- If $c = p/q$: $k_1 - k_2$ must be a multiple of $q$ → exactly $q$ values
- If $c$ is irrational or complex: never an integer → infinitely many

---

## 3. Important Special Cases

### 3.1 $i^i$ — A Surprising Real Number

$$i^i = e^{i\log i} = e^{i[\ln 1 + i(\pi/2 + 2k\pi)]} = e^{i \cdot i(\pi/2 + 2k\pi)} = e^{-(\pi/2 + 2k\pi)}$$

Principal value: $i^i = e^{-\pi/2} \approx 0.2079$

All values are **real**!

### 3.2 $(-1)^{1/3}$

$$(-1)^{1/3} = e^{\frac{1}{3}\log(-1)} = e^{\frac{1}{3}[i(\pi + 2k\pi)]} = e^{i(2k+1)\pi/3}$$

For $k = 0, 1, 2$: $e^{i\pi/3}$, $e^{i\pi}$, $e^{i5\pi/3}$

The three cube roots of $-1$: $\frac{1+i\sqrt{3}}{2}$, $-1$, $\frac{1-i\sqrt{3}}{2}$

---

## 4. Design Example

### Example: Find all values of $(1+i)^{1-i}$.

**Step 1.** $\log(1+i) = \ln\sqrt{2} + i(\pi/4 + 2k\pi) = \frac{1}{2}\ln 2 + i\frac{(8k+1)\pi}{4}$

**Step 2.**
$$(1+i)^{1-i} = e^{(1-i)\log(1+i)} = e^{(1-i)[\frac{1}{2}\ln 2 + i\frac{(8k+1)\pi}{4}]}$$

**Step 3.** Expand:
$$= e^{\frac{1}{2}\ln 2 + \frac{(8k+1)\pi}{4} + i[-\frac{1}{2}\ln 2 + \frac{(8k+1)\pi}{4}]}$$

**Principal value** ($k=0$):
$$= e^{\frac{1}{2}\ln 2 + \pi/4} \cdot e^{i(\pi/4 - \frac{1}{2}\ln 2)}$$

$$= \sqrt{2}\,e^{\pi/4} \cdot [\cos(\pi/4 - \tfrac{1}{2}\ln 2) + i\sin(\pi/4 - \tfrac{1}{2}\ln 2)]$$

---

## 5. Summary Table

| Concept | Key Formula |
|---------|-------------|
| General power | $z^c = e^{c\log z}$ |
| Principal value | $e^{c\,\text{Log}(z)}$ |
| Integer power | Single-valued |
| Rational power $p/q$ | $q$ values |
| Irrational/complex power | Infinitely many values |
| $i^i$ | $e^{-\pi/2 - 2k\pi}$ (all real!) |
| Derivative | $\frac{d}{dz}z^c = cz^{c-1}$ (on a branch) |

---

## Quick Revision Questions

1. **Compute** all values of $i^{1/2}$ and verify they match $\sqrt{i}$.

2. **Find** the principal value of $(1+i\sqrt{3})^i$.

3. **Show** that $i^i$ is real and compute its principal value.

4. **How many** distinct values does $z^{3/5}$ have? Justify.

5. **Prove** that $z^c$ is single-valued if and only if $c$ is an integer.

6. **Find** $(e)^{2\pi i}$ — is it 1? Explain carefully.

---

[← Previous: Logarithmic Function](04-logarithmic-function.md) | [Next: Multi-valued Functions →](06-multi-valued-functions.md)

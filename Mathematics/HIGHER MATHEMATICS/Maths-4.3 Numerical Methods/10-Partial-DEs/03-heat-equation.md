# Chapter 10.3: Numerical Solution of the Heat Equation

[← Previous: Finite Difference Methods](02-finite-difference-pde.md) | [Next: Wave Equation →](04-wave-equation.md)

---

## Overview

The **heat (diffusion) equation** $u_t = \alpha u_{xx}$ is the prototype parabolic PDE. We study three finite difference schemes: the explicit FTCS, the implicit backward Euler, and the Crank–Nicolson method.

---

## 1. Problem Statement

$$\frac{\partial u}{\partial t} = \alpha \frac{\partial^2 u}{\partial x^2}, \quad 0 < x < L, \quad t > 0$$

**Initial condition:** $u(x,0) = f(x)$

**Boundary conditions:** $u(0,t) = g_0(t)$, $u(L,t) = g_L(t)$

Grid: $x_i = ih$, $t_n = nk$, $r = \frac{\alpha k}{h^2}$ (mesh ratio).

---

## 2. Explicit Scheme (FTCS)

Forward-Time Central-Space:

$$\boxed{u_i^{n+1} = ru_{i-1}^n + (1-2r)u_i^n + ru_{i+1}^n}$$

```
  Time level n+1:         ● u_i^{n+1}
                         ╱ | ╲
  Time level n:     ●───── ● ─────●
                  u_{i-1}  u_i  u_{i+1}
```

| Property | Value |
|----------|-------|
| Accuracy | $O(k) + O(h^2)$ |
| Stability | $r \leq 1/2$ |
| Cost | Very cheap (no system to solve) |

---

## 3. Implicit Scheme (Backward Euler)

$$u_i^{n+1} - r\left(u_{i-1}^{n+1} - 2u_i^{n+1} + u_{i+1}^{n+1}\right) = u_i^n$$

$$\boxed{-ru_{i-1}^{n+1} + (1+2r)u_i^{n+1} - ru_{i+1}^{n+1} = u_i^n}$$

This gives a **tridiagonal system** at each time step:

$$\begin{bmatrix} 1+2r & -r & & \\ -r & 1+2r & -r & \\ & \ddots & \ddots & \ddots \\ & & -r & 1+2r \end{bmatrix} \mathbf{u}^{n+1} = \mathbf{u}^n + \text{BC terms}$$

| Property | Value |
|----------|-------|
| Accuracy | $O(k) + O(h^2)$ |
| Stability | **Unconditionally stable** |
| Cost | Solve tridiagonal system per step |

---

## 4. Crank–Nicolson Scheme

Average of forward and backward Euler (trapezoidal rule in time):

$$\boxed{\frac{u_i^{n+1}-u_i^n}{k} = \frac{\alpha}{2}\left[\frac{\delta^2 u_i^{n+1}}{h^2} + \frac{\delta^2 u_i^n}{h^2}\right]}$$

Rearranging:

$$-\frac{r}{2}u_{i-1}^{n+1} + (1+r)u_i^{n+1} - \frac{r}{2}u_{i+1}^{n+1} = \frac{r}{2}u_{i-1}^n + (1-r)u_i^n + \frac{r}{2}u_{i+1}^n$$

| Property | Value |
|----------|-------|
| Accuracy | $O(k^2) + O(h^2)$ |
| Stability | **Unconditionally stable** |
| Cost | Solve tridiagonal system per step |

This is the **preferred method** — second-order in both space and time.

---

## 5. Worked Example

Solve $u_t = u_{xx}$ on $[0,1]$, $u(x,0) = \sin(\pi x)$, $u(0,t) = u(1,t) = 0$.

Take $h = 0.25$, $k = 0.025$, so $r = 0.025/0.0625 = 0.4$.

**Exact solution:** $u(x,t) = e^{-\pi^2 t}\sin(\pi x)$

### Initial Values

| $i$ | $x_i$ | $u_i^0 = \sin(\pi x_i)$ |
|-----|--------|------------------------|
| 0 | 0 | 0 |
| 1 | 0.25 | 0.7071 |
| 2 | 0.50 | 1.0000 |
| 3 | 0.75 | 0.7071 |
| 4 | 1.00 | 0 |

### FTCS: Step $n=0 \to n=1$ ($r = 0.4$)

$$u_1^1 = 0.4(0) + 0.2(0.7071) + 0.4(1.0000) = 0.1414 + 0.4000 = 0.5414$$

$$u_2^1 = 0.4(0.7071) + 0.2(1.0000) + 0.4(0.7071) = 0.2828 + 0.2000 + 0.2828 = 0.7657$$

$$u_3^1 = 0.4(1.0000) + 0.2(0.7071) + 0.4(0) = 0.4000 + 0.1414 = 0.5414$$

### Comparison at $t = 0.025$

| $x$ | FTCS | Exact | Error |
|------|------|-------|-------|
| 0.25 | 0.5414 | 0.5532 | 0.0118 |
| 0.50 | 0.7657 | 0.7824 | 0.0167 |
| 0.75 | 0.5414 | 0.5532 | 0.0118 |

---

## 6. Stability Diagram

```
  k
  │
  │  FTCS Unstable
  │  ┌─────────────
  │  │
  ──│── k = h²/(2α)  ← stability limit
  │  │
  │  │  FTCS Stable
  │  │
  └──┼──────────────── h
     │
     
  Crank-Nicolson and Backward Euler:
  Stable for ALL r (unconditionally stable)
```

---

## 7. Comparison of Schemes

| Scheme | Order | Stability | System to solve? | Best for |
|--------|-------|-----------|-----------------|----------|
| FTCS (Explicit) | $O(k)+O(h^2)$ | $r \leq 1/2$ | No | Quick tests |
| Backward Euler | $O(k)+O(h^2)$ | Unconditional | Tridiagonal | Stiff problems |
| Crank–Nicolson | $O(k^2)+O(h^2)$ | Unconditional | Tridiagonal | **General use** |

---

## 8. Thomas Algorithm for Tridiagonal Systems

Both implicit and Crank–Nicolson require solving a tridiagonal system $A\mathbf{u}=\mathbf{b}$ at each time step. The **Thomas algorithm** does this in $O(n)$:

1. **Forward sweep**: eliminate lower diagonal
2. **Back substitution**: solve from last to first

This makes implicit methods very efficient for 1D problems.

---

## Summary Table

| Property | FTCS | Backward Euler | Crank–Nicolson |
|----------|------|---------------|----------------|
| Time accuracy | $O(k)$ | $O(k)$ | $O(k^2)$ |
| Space accuracy | $O(h^2)$ | $O(h^2)$ | $O(h^2)$ |
| Stability | $r \leq 1/2$ | Unconditional | Unconditional |
| Implicit? | No | Yes | Yes |
| Oscillations? | No (if stable) | No | Possible if $r$ large |

---

## Quick Revision Questions

1. **Write the FTCS scheme for $u_t = \alpha u_{xx}$. What is the stability condition?**

2. **Derive the Crank–Nicolson scheme and show it is $O(k^2) + O(h^2)$.**

3. **Solve one time step of the FTCS scheme for $u_t = u_{xx}$ with $h=0.2$, $k=0.02$, $u(x,0)=x(1-x)$, $u(0,t)=u(1,t)=0$.**

4. **Why is the Crank–Nicolson method preferred over FTCS and backward Euler?**

5. **Explain the Thomas algorithm and why it is efficient for 1D implicit schemes.**

6. **What happens to the FTCS solution when $r > 1/2$? Demonstrate with an example.**

---

[← Previous: Finite Difference Methods](02-finite-difference-pde.md) | [Next: Wave Equation →](04-wave-equation.md)

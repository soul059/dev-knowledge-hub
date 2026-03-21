# Chapter 7: Bellman Equations

> **Unit 2: Markov Decision Processes** — *The recursive equations at the heart of all reinforcement learning algorithms.*

---

## Overview

The Bellman equations express a fundamental recursive relationship: the value of a state
is the immediate reward plus the discounted value of the successor state. These equations
are the theoretical backbone of dynamic programming, temporal-difference learning, and
virtually every RL algorithm. This chapter derives both the Bellman expectation equations
(for a fixed policy) and the Bellman optimality equations (for the optimal policy), presents
them in matrix form, and solves them analytically and numerically for small MDPs.

---

## 1. The Key Insight: Recursive Decomposition

The return $G_t$ can be decomposed recursively:

$$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots$$
$$= R_{t+1} + \gamma (R_{t+2} + \gamma R_{t+3} + \cdots)$$
$$= R_{t+1} + \gamma G_{t+1}$$

Taking expectations:

$$V^\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s] = \mathbb{E}_\pi[R_{t+1} + \gamma G_{t+1} \mid S_t = s]$$

This is the starting point for all Bellman equations.

---

## 2. Bellman Expectation Equation for $V^\pi$

### Derivation

Starting from $V^\pi(s) = \mathbb{E}_\pi[R_{t+1} + \gamma G_{t+1} \mid S_t = s]$:

**Step 1**: Expand the expectation over actions (using policy $\pi$):

$$V^\pi(s) = \sum_{a} \pi(a \mid s) \; \mathbb{E}[R_{t+1} + \gamma G_{t+1} \mid S_t = s, A_t = a]$$

**Step 2**: Expand over next states (using transition probabilities):

$$V^\pi(s) = \sum_{a} \pi(a \mid s) \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma \; \mathbb{E}_\pi[G_{t+1} \mid S_{t+1} = s']\right]$$

**Step 3**: Recognize $\mathbb{E}_\pi[G_{t+1} \mid S_{t+1} = s'] = V^\pi(s')$:

$$\boxed{V^\pi(s) = \sum_{a} \pi(a \mid s) \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma \, V^\pi(s')\right]}$$

### Interpretation

The value of state $s$ under policy $\pi$ is:
1. **Sum over actions** the policy might take (weighted by $\pi(a|s)$)
2. For each action, **sum over possible next states** (weighted by $P(s'|s,a)$)
3. Each next state contributes: **immediate reward** $R(s,a,s')$ + **discounted future value** $\gamma V^\pi(s')$

### Backup Diagram for $V^\pi$

```
                     V^π(s)
                       │
              ┌────────┼────────┐
              │        │        │
           π(a₁|s) π(a₂|s) π(a₃|s)      ← Policy probabilities
              │        │        │
             (a₁)     (a₂)    (a₃)       ← Action nodes
              │        │        │
         ┌────┼───┐    │    ┌──┼───┐
         │    │   │    │    │  │   │
     P(s₁|.) │  P(s₃|.)  P(s₁|.) │
         │   │    │    │    │  │   │      ← Transition probs
        (s₁)(s₂)(s₃) (s₂) (s₁)(s₂)(s₃)  ← Next states
         │   │    │    │    │  │   │
        r+γV r+γV r+γV r+γV r+γV r+γV r+γV  ← Rewards + discounted values
```

---

## 3. Bellman Expectation Equation for $Q^\pi$

### Formula

$$\boxed{Q^\pi(s, a) = \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma \sum_{a'} \pi(a' \mid s') \, Q^\pi(s', a')\right]}$$

### Derivation

$$Q^\pi(s, a) = \mathbb{E}[R_{t+1} + \gamma G_{t+1} \mid S_t=s, A_t=a]$$
$$= \sum_{s'} P(s'|s,a)\left[R(s,a,s') + \gamma \, \mathbb{E}_\pi[G_{t+1} \mid S_{t+1}=s']\right]$$
$$= \sum_{s'} P(s'|s,a)\left[R(s,a,s') + \gamma \, V^\pi(s')\right]$$
$$= \sum_{s'} P(s'|s,a)\left[R(s,a,s') + \gamma \sum_{a'} \pi(a'|s') Q^\pi(s',a')\right]$$

### Backup Diagram for $Q^\pi$

```
              Q^π(s,a)
                 │
                (a)           ← Given action
                 │
          ┌──────┼──────┐
          │      │      │
      P(s₁|s,a) │  P(s₃|s,a)    ← Transition probabilities
          │      │      │
         (s₁)   (s₂)  (s₃)      ← Next states
         /|\    /|\    /|\
        / | \  / | \  / | \
      a₁ a₂ a₃               ← Next actions (weighted by π)
       │  │  │
     Q^π Q^π Q^π               ← Recurse
```

---

## 4. Bellman Optimality Equations

### For $V^*$

The optimal value function satisfies:

$$\boxed{V^*(s) = \max_a \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma \, V^*(s')\right]}$$

**Key difference from expectation equation**: $\max_a$ replaces $\sum_a \pi(a|s)$.

### For $Q^*$

$$\boxed{Q^*(s, a) = \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma \max_{a'} Q^*(s', a')\right]}$$

### Backup Diagrams: Expectation vs Optimality

```
   BELLMAN EXPECTATION (V^π)          BELLMAN OPTIMALITY (V*)
   
         V^π(s)                            V*(s)
           │                                 │
    ┌──────┼──────┐                   ┌──────┼──────┐
   π(a₁)  π(a₂)  π(a₃)    ← SUM     max   max   max    ← MAX
    │      │      │                    │     │     │
   (a₁)   (a₂)   (a₃)              (a₁)   (a₂)  (a₃)
    │      │      │                   │      │     │
   ┌┼┐   ┌┼┐   ┌┼┐               ┌──┼─┐  ┌─┼┐  ┌┼──┐
   ···    ···   ···               ···     ···    ···
   s'     s'    s'                s'      s'     s'
   
   Weight by π,                   Take the BEST action,
   then average over s'           then average over s'
```

### Derivation of Bellman Optimality for $V^*$

$$V^*(s) = \max_\pi V^\pi(s)$$

The optimal policy must select the action that maximizes expected value:

$$V^*(s) = \max_a Q^*(s, a)$$
$$= \max_a \mathbb{E}[R_{t+1} + \gamma V^*(S_{t+1}) \mid S_t=s, A_t=a]$$
$$= \max_a \sum_{s'} P(s'|s,a)\left[R(s,a,s') + \gamma V^*(s')\right]$$

---

## 5. Matrix Form of the Bellman Equation

### Bellman Expectation in Matrix Form

For a fixed policy $\pi$ and $n$ states, the Bellman equation becomes:

$$\mathbf{V}^\pi = \mathbf{R}^\pi + \gamma \mathbf{P}^\pi \mathbf{V}^\pi$$

where:
- $\mathbf{V}^\pi \in \mathbb{R}^n$ is the value function vector
- $\mathbf{R}^\pi \in \mathbb{R}^n$ is the expected immediate reward vector: $R^\pi_i = \sum_a \pi(a|s_i) \sum_{s'} P(s'|s_i,a) R(s_i,a,s')$
- $\mathbf{P}^\pi \in \mathbb{R}^{n \times n}$ is the policy transition matrix: $P^\pi_{ij} = \sum_a \pi(a|s_i) P(s_j|s_i,a)$

### Closed-Form Solution

Rearranging:

$$\mathbf{V}^\pi - \gamma \mathbf{P}^\pi \mathbf{V}^\pi = \mathbf{R}^\pi$$
$$(I - \gamma \mathbf{P}^\pi) \mathbf{V}^\pi = \mathbf{R}^\pi$$

$$\boxed{\mathbf{V}^\pi = (I - \gamma \mathbf{P}^\pi)^{-1} \mathbf{R}^\pi}$$

**Complexity**: $O(n^3)$ for matrix inversion — feasible only for small MDPs.

### Why the Inverse Exists

The matrix $(I - \gamma P^\pi)$ is always invertible when $0 \leq \gamma < 1$:
- $P^\pi$ is a stochastic matrix, so its eigenvalues satisfy $|\lambda_i| \leq 1$
- The eigenvalues of $\gamma P^\pi$ satisfy $|\gamma \lambda_i| < 1$
- Thus $(I - \gamma P^\pi)$ has all eigenvalues with positive real part

The inverse can be expressed as a **Neumann series**:

$$(I - \gamma P^\pi)^{-1} = \sum_{k=0}^{\infty} (\gamma P^\pi)^k = I + \gamma P^\pi + \gamma^2 (P^\pi)^2 + \cdots$$

This sum converges because $\gamma < 1$, and its $k$-th term represents the discounted
$k$-step transition probabilities.

---

## 6. Solving Bellman Equations Analytically

### Example MDP

```
States: {s₀, s₁}     Actions: {a}  (single action = MRP)     γ = 0.9

  ┌──── 0.6 ────┐
  │              │
  ▼              │
 (s₀)── 0.4 ──▶(s₁)
  │    R=+2      │
  │              │
  │    0.3       │── 0.7 ──┐
  └──────────────┘         │
        R=+5               │
                           ▼
                          (s₁) self-loop
                          R=+1
```

Wait, let me give a cleaner example:

```
Transitions:                          Rewards:
  s₀ → s₀ with P=0.6, R=+2            R(s₀→s₀) = +2
  s₀ → s₁ with P=0.4, R=+5            R(s₀→s₁) = +5
  s₁ → s₀ with P=0.3, R=+1            R(s₁→s₀) = +1
  s₁ → s₁ with P=0.7, R=+3            R(s₁→s₁) = +3
```

### Step 1: Build Matrices

$$P = \begin{bmatrix} 0.6 & 0.4 \\ 0.3 & 0.7 \end{bmatrix}, \quad R = \begin{bmatrix} 0.6(2) + 0.4(5) \\ 0.3(1) + 0.7(3) \end{bmatrix} = \begin{bmatrix} 3.2 \\ 2.4 \end{bmatrix}$$

### Step 2: Compute $(I - \gamma P)$

$$I - 0.9P = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix} - \begin{bmatrix} 0.54 & 0.36 \\ 0.27 & 0.63 \end{bmatrix} = \begin{bmatrix} 0.46 & -0.36 \\ -0.27 & 0.37 \end{bmatrix}$$

### Step 3: Invert

$$\det = 0.46 \times 0.37 - (-0.36)(-0.27) = 0.1702 - 0.0972 = 0.073$$

$$(I - \gamma P)^{-1} = \frac{1}{0.073} \begin{bmatrix} 0.37 & 0.36 \\ 0.27 & 0.46 \end{bmatrix} = \begin{bmatrix} 5.068 & 4.932 \\ 3.699 & 6.301 \end{bmatrix}$$

### Step 4: Solve

$$V = (I - \gamma P)^{-1} R = \begin{bmatrix} 5.068 & 4.932 \\ 3.699 & 6.301 \end{bmatrix} \begin{bmatrix} 3.2 \\ 2.4 \end{bmatrix} = \begin{bmatrix} 28.06 \\ 26.96 \end{bmatrix}$$

$$V(s_0) \approx 28.06, \quad V(s_1) \approx 26.96$$

### Verification

$$V(s_0) = 3.2 + 0.9(0.6 \times 28.06 + 0.4 \times 26.96)$$
$$= 3.2 + 0.9(16.836 + 10.784) = 3.2 + 0.9 \times 27.62 = 3.2 + 24.858 = 28.058 \approx 28.06 \quad \checkmark$$

---

## 7. Python Code: Solving Bellman Equations

```python
import numpy as np

class BellmanSolver:
    """Solve Bellman equations for finite MDPs."""

    def __init__(self, n_states, gamma=0.99):
        self.n = n_states
        self.gamma = gamma

    def solve_analytical(self, P_pi, R_pi):
        """
        Solve V = (I - γP)^{-1} R analytically.
        P_pi: n×n policy transition matrix
        R_pi: n-vector of expected immediate rewards
        """
        I = np.eye(self.n)
        A = I - self.gamma * P_pi
        V = np.linalg.solve(A, R_pi)
        return V

    def solve_iterative(self, P_pi, R_pi, tol=1e-10, max_iter=10000):
        """
        Solve V^π iteratively: V_{k+1} = R + γPV_k.
        """
        V = np.zeros(self.n)
        for k in range(max_iter):
            V_new = R_pi + self.gamma * P_pi @ V
            delta = np.max(np.abs(V_new - V))
            V = V_new
            if delta < tol:
                return V, k + 1
        return V, max_iter

    def solve_optimality(self, P, R, actions, tol=1e-10, max_iter=10000):
        """
        Solve Bellman optimality equation (value iteration).
        P: dict {action: n×n transition matrix}
        R: dict {action: n-vector of expected rewards}
        """
        V = np.zeros(self.n)
        for k in range(max_iter):
            V_new = np.full(self.n, -np.inf)
            for a in actions:
                Q_a = R[a] + self.gamma * P[a] @ V
                V_new = np.maximum(V_new, Q_a)
            delta = np.max(np.abs(V_new - V))
            V = V_new
            if delta < tol:
                return V, k + 1
        return V, max_iter

    def neumann_series(self, P_pi, R_pi, k_terms=100):
        """
        Compute V using Neumann series: V = Σ_{k=0}^∞ (γP)^k R.
        """
        V = np.zeros(self.n)
        gamma_P_k = np.eye(self.n)  # (γP)^0 = I
        for k in range(k_terms):
            V += gamma_P_k @ R_pi
            gamma_P_k = self.gamma * gamma_P_k @ P_pi
        return V


# --- Example: 2-state MRP ---
print("=" * 60)
print("BELLMAN EQUATION SOLUTIONS")
print("=" * 60)

P = np.array([
    [0.6, 0.4],
    [0.3, 0.7]
])
R = np.array([
    0.6 * 2 + 0.4 * 5,  # = 3.2
    0.3 * 1 + 0.7 * 3   # = 2.4
])
gamma = 0.9

solver = BellmanSolver(n_states=2, gamma=gamma)

# Method 1: Analytical (matrix inversion)
V_analytical = solver.solve_analytical(P, R)
print(f"\n1. Analytical Solution (matrix inversion):")
print(f"   V(s₀) = {V_analytical[0]:.4f}")
print(f"   V(s₁) = {V_analytical[1]:.4f}")

# Method 2: Iterative
V_iterative, iters = solver.solve_iterative(P, R)
print(f"\n2. Iterative Solution ({iters} iterations):")
print(f"   V(s₀) = {V_iterative[0]:.4f}")
print(f"   V(s₁) = {V_iterative[1]:.4f}")

# Method 3: Neumann series
V_neumann = solver.neumann_series(P, R, k_terms=200)
print(f"\n3. Neumann Series (200 terms):")
print(f"   V(s₀) = {V_neumann[0]:.4f}")
print(f"   V(s₁) = {V_neumann[1]:.4f}")

# Verify all methods agree
print(f"\n   Max difference between methods: "
      f"{max(np.max(np.abs(V_analytical - V_iterative)), np.max(np.abs(V_analytical - V_neumann))):.2e}")


# --- Bellman Optimality Equation ---
print(f"\n{'=' * 60}")
print("BELLMAN OPTIMALITY EQUATIONS")
print("=" * 60)

# 3-state MDP with 2 actions
P_left = np.array([
    [0.9, 0.1, 0.0],
    [0.5, 0.3, 0.2],
    [0.0, 0.0, 1.0],
])
P_right = np.array([
    [0.1, 0.8, 0.1],
    [0.0, 0.2, 0.8],
    [0.0, 0.0, 1.0],
])
R_left = np.array([
    0.9 * 1 + 0.1 * 0,     # s0: mostly stay, small reward
    0.5 * 0 + 0.3 * 0 + 0.2 * 0,  # s1: no reward for left
    0.0                      # s2: terminal
])
R_right = np.array([
    0.1 * 0 + 0.8 * 2 + 0.1 * 0,  # s0: moving right gives reward
    0.0 * 0 + 0.2 * 0 + 0.8 * 10, # s1: reaching s2 gives big reward
    0.0                             # s2: terminal
])

solver3 = BellmanSolver(n_states=3, gamma=0.95)

actions = ["left", "right"]
P_dict = {"left": P_left, "right": P_right}
R_dict = {"left": R_left, "right": R_right}

V_star, iters = solver3.solve_optimality(P_dict, R_dict, actions)
print(f"\nV* (converged in {iters} iterations):")
for i, name in enumerate(["s0", "s1", "s2"]):
    print(f"   V*({name}) = {V_star[i]:.4f}")

# Compute Q* and extract optimal policy
print(f"\nQ* and optimal policy:")
for i, name in enumerate(["s0", "s1", "s2"]):
    q_values = {}
    for a in actions:
        q = R_dict[a][i] + 0.95 * P_dict[a][i] @ V_star
        q_values[a] = q
    best_a = max(q_values, key=q_values.get)
    q_str = ", ".join(f"{a}={q:.4f}" for a, q in q_values.items())
    print(f"   Q*({name}, ·) = {{{q_str}}}  →  π*({name}) = {best_a}")


# --- Verify Bellman equations hold ---
print(f"\n{'=' * 60}")
print("VERIFICATION: Bellman Equations Hold")
print("=" * 60)

for i, name in enumerate(["s0", "s1", "s2"]):
    # V*(s) should equal max_a Q*(s,a)
    max_q = max(
        R_dict[a][i] + 0.95 * P_dict[a][i] @ V_star
        for a in actions
    )
    residual = abs(V_star[i] - max_q)
    status = "✓" if residual < 1e-8 else "✗"
    print(f"   {status} V*({name}) = {V_star[i]:.6f}, max_a Q* = {max_q:.6f}, "
          f"residual = {residual:.2e}")
```

---

## 8. Summary of All Bellman Equations

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        BELLMAN EQUATIONS                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  EXPECTATION (for fixed policy π):                                    │
│                                                                       │
│  V^π(s) = Σ_a π(a|s) Σ_{s'} P(s'|s,a) [R(s,a,s') + γ V^π(s')]      │
│                                                                       │
│  Q^π(s,a) = Σ_{s'} P(s'|s,a) [R(s,a,s') + γ Σ_{a'} π(a'|s') Q^π(s',a')]  │
│                                                                       │
│  Matrix form: V^π = (I - γP^π)^{-1} R^π                              │
│                                                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  OPTIMALITY (for optimal policy π*):                                  │
│                                                                       │
│  V*(s) = max_a Σ_{s'} P(s'|s,a) [R(s,a,s') + γ V*(s')]              │
│                                                                       │
│  Q*(s,a) = Σ_{s'} P(s'|s,a) [R(s,a,s') + γ max_{a'} Q*(s',a')]      │
│                                                                       │
│  No closed-form solution — solved iteratively (value iteration)       │
│                                                                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  KEY RELATIONSHIPS:                                                   │
│                                                                       │
│  V^π(s) = Σ_a π(a|s) Q^π(s,a)                                        │
│  V*(s)  = max_a Q*(s,a)                                               │
│  π*(s)  = argmax_a Q*(s,a)                                            │
│                                                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Solution Methods Comparison

| Method | Equation | Complexity | When to Use |
|---|---|---|---|
| **Matrix inversion** | $(I - \gamma P)^{-1} R$ | $O(n^3)$ | Small MDPs, exact solution |
| **Iterative evaluation** | $V_{k+1} = R + \gamma P V_k$ | $O(n^2)$ per iter | Medium MDPs |
| **Value iteration** | $V_{k+1}(s) = \max_a [R + \gamma P V_k]$ | $O(n^2 \|\mathcal{A}\|)$ per iter | Optimality equations |
| **Neumann series** | $\sum_k (\gamma P)^k R$ | $O(n^2 K)$ | Theoretical insight |

---

## 10. Real-World Applications

| Domain | Bellman Equation Use | Practical Note |
|---|---|---|
| **Dynamic pricing** | Optimal price = max expected revenue | State = inventory + demand signal |
| **Inventory control** | Optimal order quantity | Classic OR application |
| **Network routing** | Shortest path with stochastic delays | Bellman-Ford algorithm is a special case |
| **Option pricing** | American option = Bellman optimality | Finance: exercise vs hold decision |
| **Robot planning** | Optimal trajectory via value iteration | Discretize state space |

---

## Summary

| Concept | Key Point |
|---|---|
| Recursive decomposition | $G_t = R_{t+1} + \gamma G_{t+1}$ |
| Bellman expectation (V) | $V^\pi(s) = \sum_a \pi(a \mid s) \sum_{s'} P(s' \mid s,a)[R + \gamma V^\pi(s')]$ |
| Bellman expectation (Q) | $Q^\pi(s,a) = \sum_{s'} P(s' \mid s,a)[R + \gamma \sum_{a'} \pi(a' \mid s') Q^\pi(s',a')]$ |
| Bellman optimality (V) | $V^*(s) = \max_a \sum_{s'} P(s' \mid s,a)[R + \gamma V^*(s')]$ |
| Bellman optimality (Q) | $Q^*(s,a) = \sum_{s'} P(s' \mid s,a)[R + \gamma \max_{a'} Q^*(s',a')]$ |
| Matrix form | $V^\pi = (I - \gamma P^\pi)^{-1} R^\pi$ |
| Expectation vs optimality | Sum-over-π vs max-over-actions |
| Value iteration | Repeatedly apply Bellman optimality operator until convergence |

---

## Revision Questions

1. **Write the Bellman expectation equation for $V^\pi(s)$.** Explain each term and its role.

2. **What is the key difference** between the Bellman expectation equation and the Bellman optimality equation? Show both and highlight the difference.

3. **Derive the matrix form** $V^\pi = (I - \gamma P^\pi)^{-1} R^\pi$ from the Bellman expectation equation. Why does the inverse always exist for $\gamma < 1$?

4. **For the 2-state MRP** with $P = \begin{bmatrix} 0.6 & 0.4 \\ 0.3 & 0.7 \end{bmatrix}$ and $R = \begin{bmatrix} 3.2 \\ 2.4 \end{bmatrix}$ with $\gamma = 0.9$: solve for $V$ analytically. Show all steps.

5. **Draw backup diagrams** for both the Bellman expectation equation ($V^\pi$) and the Bellman optimality equation ($V^*$). Explain how they differ at the action level.

6. **Why can't the Bellman optimality equation be solved by matrix inversion** like the expectation equation? What method is used instead?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Value Functions](./06-value-functions.md) | [Unit 2: MDPs](./README.md) | [Policy Evaluation](../03-Dynamic-Programming/01-policy-evaluation.md) |

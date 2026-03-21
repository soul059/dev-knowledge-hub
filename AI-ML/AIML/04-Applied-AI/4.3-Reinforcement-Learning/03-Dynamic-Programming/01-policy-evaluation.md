# Chapter 1: Policy Evaluation (Prediction)

> **Unit 3: Dynamic Programming** — *Computing the value function for a given policy.*

---

## Overview

Policy Evaluation answers the **prediction problem** in reinforcement learning: given a
policy $\pi$, compute its state-value function $V^\pi$. This is the first and most
fundamental dynamic programming algorithm. It applies the Bellman expectation equation as
an iterative update rule, sweeping through all states until the value function converges.
Policy evaluation is not just useful on its own — it serves as the inner loop of
**policy iteration**, one of the two classic DP control algorithms.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║                POLICY EVALUATION AT A GLANCE                    ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • Input : An MDP and a policy π                                ║
 ║  • Output: The value function V^π                               ║
 ║  • Method: Iteratively apply the Bellman expectation equation    ║
 ║  • Type  : Prediction (not control)                             ║
 ║  • Key   : Guaranteed to converge for any policy                ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. The Prediction Problem

Given:
- A finite MDP $(\mathcal{S}, \mathcal{A}, P, R, \gamma)$
- A fixed policy $\pi(a \mid s)$

Find:

$$V^\pi(s) = \mathbb{E}_\pi\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1} \;\middle|\; S_0 = s\right] \quad \text{for all } s \in \mathcal{S}$$

From the Bellman expectation equation (Chapter 7), we know:

$$V^\pi(s) = \sum_{a} \pi(a \mid s) \sum_{s'} P(s' \mid s, a)\left[R(s, a, s') + \gamma \, V^\pi(s')\right]$$

For small MDPs, we could solve this system of $|\mathcal{S}|$ linear equations directly
(matrix inversion: $V^\pi = (I - \gamma P^\pi)^{-1} R^\pi$). But matrix inversion costs
$O(|\mathcal{S}|^3)$, which is impractical for large state spaces. **Iterative policy
evaluation** provides a scalable alternative.

---

## 2. Iterative Policy Evaluation Algorithm

### The Update Rule

Start with an arbitrary initial value function $V_0$ (often all zeros). At each iteration
$k$, update every state:

$$\boxed{V_{k+1}(s) = \sum_{a} \pi(a \mid s) \sum_{s', r} P(s' \mid s, a)\left[R(s, a, s') + \gamma \, V_k(s')\right]}$$

This is a **full backup**: for each state, we look ahead at all possible actions and all
possible successor states, weighted by their probabilities.

```
                    BACKUP DIAGRAM FOR POLICY EVALUATION

                              V(s)
                            /  |  \
                         π(a₁) π(a₂) π(a₃)    ← Policy weights
                        /      |      \
                      a₁      a₂      a₃       ← All actions
                     /|\      /|\      /|\
                    / | \    / | \    / | \
                  s₁  s₂ s₃ s₁ s₂ s₃ s₁ s₂ s₃  ← Successor states
                                                    (weighted by P)
          Each successor contributes: P(s'|s,a) × [R(s,a,s') + γ V_k(s')]
```

### Convergence Guarantee

The sequence $\{V_k\}$ converges to $V^\pi$ as $k \to \infty$. This is guaranteed because
the Bellman expectation operator is a **contraction mapping** with factor $\gamma < 1$:

$$\|V_{k+1} - V^\pi\|_\infty \leq \gamma \|V_k - V^\pi\|_\infty$$

After $k$ iterations, the maximum error is at most $\gamma^k \|V_0 - V^\pi\|_\infty$.

### Stopping Criterion

In practice, we stop when the largest change across all states falls below a threshold:

$$\Delta = \max_s |V_{k+1}(s) - V_k(s)| < \theta$$

where $\theta$ is a small positive number (e.g., $10^{-6}$).

---

## 3. Full Pseudocode

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║  ITERATIVE POLICY EVALUATION (Two-Array Version)                    ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║  Input: policy π, MDP (S, A, P, R, γ), threshold θ                 ║
 ║  Output: V ≈ V^π                                                   ║
 ║                                                                     ║
 ║  1.  Initialize V(s) = 0  for all s ∈ S                            ║
 ║  2.  Repeat:                                                        ║
 ║  3.      Δ ← 0                                                     ║
 ║  4.      For each s ∈ S:                                            ║
 ║  5.          v ← V(s)                                               ║
 ║  6.          V(s) ← Σ_a π(a|s) Σ_{s'} P(s'|s,a)[R(s,a,s') + γV(s')]  ║
 ║  7.          Δ ← max(Δ, |v − V(s)|)                                ║
 ║  8.      Until Δ < θ                                                ║
 ║  9.  Return V                                                       ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

> **Note:** The pseudocode above uses the **in-place** variant (updating $V(s)$ immediately).
> The two-array variant stores updates in a separate array $V_\text{new}$ and swaps after
> each full sweep. Both converge to $V^\pi$; in-place typically converges faster.

---

## 4. In-Place vs Two-Array Variants

| Variant | Description | Memory | Convergence |
|---|---|---|---|
| **Two-array** | Keep $V_k$ and $V_{k+1}$ separate; swap after full sweep | $2 \times |\mathcal{S}|$ | Exact Bellman iterations |
| **In-place** | Update $V(s)$ immediately; later states see updated values | $1 \times |\mathcal{S}|$ | Usually faster; still converges |

```
  TWO-ARRAY:                           IN-PLACE:

  V_old: [0.0, 0.0, 0.0, 0.0]         V: [0.0, 0.0, 0.0, 0.0]
         ↓    ↓    ↓    ↓                  ↓
  V_new: [2.1, 1.5, 3.0, 0.8]         V: [2.1, ?, ?, ?]
         (all use V_old values)             ↓
                                        V: [2.1, 1.7, ?, ?]  ← uses updated V(s₁)=2.1
                                             ↓
                                        V: [2.1, 1.7, 3.2, ?]  ← uses updated V(s₁,s₂)
                                             ↓
                                        V: [2.1, 1.7, 3.2, 0.9]
```

In the in-place variant, the order of state updates matters. States updated later in the
sweep benefit from already-updated values, which propagates information faster.

---

## 5. Grid World Example — Step-by-Step

Consider a 4×4 grid world (Sutton & Barto Example 4.1):

```
  ┌─────────────────────────────────────────────────┐
  │              4×4 GRID WORLD                      │
  │                                                  │
  │    ┌─────┬─────┬─────┬─────┐                    │
  │    │  T  │  1  │  2  │  3  │   T = Terminal      │
  │    ├─────┼─────┼─────┼─────┤   (state 0)         │
  │    │  4  │  5  │  6  │  7  │                      │
  │    ├─────┼─────┼─────┼─────┤   Actions: N,S,E,W  │
  │    │  8  │  9  │ 10  │ 11  │   (equiprobable)    │
  │    ├─────┼─────┼─────┼─────┤                      │
  │    │ 12  │ 13  │ 14  │  T  │   Reward = -1 for   │
  │    └─────┴─────┴─────┴─────┘   every transition   │
  │                                                  │
  │    Policy: π(N|s) = π(S|s) = π(E|s) = π(W|s) = 0.25  │
  │    Discount: γ = 1.0 (undiscounted)              │
  └─────────────────────────────────────────────────┘
```

**Rules:**
- Moving off the grid leaves the agent in the same cell (still gets $R = -1$).
- Terminal states (top-left and bottom-right) have $V(T) = 0$ always.
- Policy is **uniform random**: each of the 4 actions has probability 0.25.

### Iteration 0 (Initialization)

$$V_0(s) = 0 \quad \text{for all } s$$

```
    ┌──────┬──────┬──────┬──────┐
    │  0.0 │  0.0 │  0.0 │  0.0 │
    ├──────┼──────┼──────┼──────┤
    │  0.0 │  0.0 │  0.0 │  0.0 │
    ├──────┼──────┼──────┼──────┤
    │  0.0 │  0.0 │  0.0 │  0.0 │
    ├──────┼──────┼──────┼──────┤
    │  0.0 │  0.0 │  0.0 │  0.0 │
    └──────┴──────┴──────┴──────┘
```

### Iteration 1

For state 1 (row 0, col 1):
- North: off grid → stays in state 1, $R = -1$, $V_0(1) = 0$
- South: goes to state 5, $R = -1$, $V_0(5) = 0$
- East: goes to state 2, $R = -1$, $V_0(2) = 0$
- West: goes to terminal state 0, $R = -1$, $V_0(0) = 0$

$$V_1(1) = 0.25 \times [(-1 + 0) + (-1 + 0) + (-1 + 0) + (-1 + 0)] = 0.25 \times (-4) = -1.0$$

By symmetry, every non-terminal state gets $V_1(s) = -1.0$.

```
    ┌──────┬──────┬──────┬──────┐
    │  0.0 │ -1.0 │ -1.0 │ -1.0 │
    ├──────┼──────┼──────┼──────┤
    │ -1.0 │ -1.0 │ -1.0 │ -1.0 │
    ├──────┼──────┼──────┼──────┤
    │ -1.0 │ -1.0 │ -1.0 │ -1.0 │
    ├──────┼──────┼──────┼──────┤
    │ -1.0 │ -1.0 │ -1.0 │  0.0 │
    └──────┴──────┴──────┴──────┘
```

### Iteration 2

For state 1 (neighbors: T=0, 1, 5, 2):

$$V_2(1) = 0.25 \times [(-1+0) + (-1+(-1)) + (-1+(-1)) + (-1+(-1))]$$
$$= 0.25 \times [(-1) + (-2) + (-2) + (-2)] = 0.25 \times (-7) = -1.75$$

For state 5 (center, neighbors: 1, 9, 6, 4):

$$V_2(5) = 0.25 \times [(-1+(-1)) + (-1+(-1)) + (-1+(-1)) + (-1+(-1))]$$
$$= 0.25 \times (-8) = -2.0$$

```
    ┌──────┬──────┬──────┬──────┐
    │  0.0 │ -1.7 │ -2.0 │ -2.0 │
    ├──────┼──────┼──────┼──────┤
    │ -1.7 │ -2.0 │ -2.0 │ -2.0 │
    ├──────┼──────┼──────┼──────┤
    │ -2.0 │ -2.0 │ -2.0 │ -1.7 │
    ├──────┼──────┼──────┼──────┤
    │ -2.0 │ -2.0 │ -1.7 │  0.0 │
    └──────┴──────┴──────┴──────┘
```

### Iteration 3

For state 1 (neighbors: T=0, 1=-1.75, 5=-2.0, 2=-2.0):

$$V_3(1) = 0.25 \times [(-1+0) + (-1 + (-1.75)) + (-1 + (-2.0)) + (-1 + (-2.0))]$$
$$= 0.25 \times [-1 + (-2.75) + (-3.0) + (-3.0)] = 0.25 \times (-9.75) = -2.4375$$

### Converged Values (after many iterations)

```
    ┌──────┬──────┬──────┬──────┐
    │  0.0 │  -14 │  -20 │  -22 │
    ├──────┼──────┼──────┼──────┤
    │  -14 │  -18 │  -20 │  -20 │
    ├──────┼──────┼──────┼──────┤
    │  -20 │  -20 │  -18 │  -14 │
    ├──────┼──────┼──────┼──────┤
    │  -22 │  -20 │  -14 │  0.0 │
    └──────┴──────┴──────┴──────┘

    V^π for the uniform random policy (rounded to integers)
```

The symmetry across the diagonal reflects the symmetry of the grid and the two terminal
states. The center states have the most negative values — the random walk takes longest to
reach a terminal state from the middle.

---

## 6. Mathematical Foundations

### Contraction Mapping

Define the Bellman operator $T^\pi$ as:

$$(T^\pi V)(s) = \sum_a \pi(a \mid s) \sum_{s'} P(s' \mid s, a)[R(s,a,s') + \gamma V(s')]$$

$T^\pi$ is a **$\gamma$-contraction** in the $\|\cdot\|_\infty$ norm:

$$\|T^\pi V_1 - T^\pi V_2\|_\infty \leq \gamma \|V_1 - V_2\|_\infty$$

By the **Banach fixed-point theorem**, $T^\pi$ has a unique fixed point ($V^\pi$), and
repeated application of $T^\pi$ converges to it from any starting point.

### Rate of Convergence

After $k$ iterations:

$$\|V_k - V^\pi\|_\infty \leq \gamma^k \|V_0 - V^\pi\|_\infty$$

To achieve error $\leq \varepsilon$, we need approximately:

$$k \geq \frac{\log(\varepsilon / \|V_0 - V^\pi\|_\infty)}{\log \gamma}$$

For $\gamma = 0.99$ and initial error bound of 100, achieving $\varepsilon = 0.01$ requires about $k \geq \frac{\log(10^{-4})}{\log(0.99)} \approx 917$ iterations.

> **Key Insight:** Smaller $\gamma$ means faster convergence but a more short-sighted agent.
> There is an inherent trade-off between solution quality and computational cost.

---

## 7. Python Implementation

```python
import numpy as np

def iterative_policy_evaluation(
    P,          # P[s, a, s'] = transition probability
    R,          # R[s, a, s'] = reward
    pi,         # pi[s, a] = policy probability
    gamma=1.0,  # discount factor
    theta=1e-8, # convergence threshold
    in_place=True
):
    """
    Iterative Policy Evaluation.

    Parameters
    ----------
    P : np.ndarray, shape (|S|, |A|, |S|)
    R : np.ndarray, shape (|S|, |A|, |S|)
    pi : np.ndarray, shape (|S|, |A|)
    gamma : float
    theta : float
    in_place : bool

    Returns
    -------
    V : np.ndarray, shape (|S|,)
    history : list of np.ndarray (value function at each iteration)
    """
    n_states = P.shape[0]
    V = np.zeros(n_states)
    history = [V.copy()]
    iteration = 0

    while True:
        delta = 0.0
        V_new = V if in_place else V.copy()

        for s in range(n_states):
            v_old = V_new[s]
            # Bellman expectation backup
            v_new = 0.0
            for a in range(P.shape[1]):
                for s_prime in range(n_states):
                    v_new += pi[s, a] * P[s, a, s_prime] * (
                        R[s, a, s_prime] + gamma * V[s_prime]
                    )
            V_new[s] = v_new
            delta = max(delta, abs(v_old - v_new))

        if not in_place:
            V = V_new
        iteration += 1
        history.append(V.copy())

        if delta < theta:
            break

    print(f"Converged in {iteration} iterations (Δ = {delta:.2e})")
    return V, history


# ========================================
# Example: 4×4 Grid World
# ========================================
SIZE = 4
N_STATES = SIZE * SIZE
N_ACTIONS = 4  # 0=N, 1=S, 2=E, 3=W
TERMINALS = {0, 15}

# Action effects: (row_delta, col_delta)
ACTION_EFFECTS = {0: (-1, 0), 1: (1, 0), 2: (0, 1), 3: (0, -1)}

def state_to_rc(s):
    return s // SIZE, s % SIZE

def rc_to_state(r, c):
    return r * SIZE + c

# Build transition model
P = np.zeros((N_STATES, N_ACTIONS, N_STATES))
R = np.zeros((N_STATES, N_ACTIONS, N_STATES))

for s in range(N_STATES):
    if s in TERMINALS:
        # Terminal states: self-loop with 0 reward
        for a in range(N_ACTIONS):
            P[s, a, s] = 1.0
        continue
    r, c = state_to_rc(s)
    for a in range(N_ACTIONS):
        dr, dc = ACTION_EFFECTS[a]
        nr, nc = r + dr, c + dc
        # Boundary check: if off grid, stay in place
        if 0 <= nr < SIZE and 0 <= nc < SIZE:
            s_prime = rc_to_state(nr, nc)
        else:
            s_prime = s
        P[s, a, s_prime] = 1.0
        R[s, a, s_prime] = -1.0

# Uniform random policy
pi = np.ones((N_STATES, N_ACTIONS)) / N_ACTIONS

# Run evaluation
V, history = iterative_policy_evaluation(P, R, pi, gamma=1.0, theta=1e-4)

# Display result
print("\nFinal V^π (4×4 grid, uniform random policy):")
for row in range(SIZE):
    vals = [f"{V[rc_to_state(row, c)]:7.1f}" for c in range(SIZE)]
    print("  " + " ".join(vals))
```

**Expected output:**

```
Converged in 174 iterations (Δ = 9.89e-05)

Final V^π (4×4 grid, uniform random policy):
      0.0   -14.0   -20.0   -22.0
    -14.0   -18.0   -20.0   -20.0
    -20.0   -20.0   -18.0   -14.0
    -22.0   -20.0   -14.0     0.0
```

---

## 8. Vectorized (NumPy) Implementation

For larger problems, we can vectorize the Bellman backup:

```python
def policy_evaluation_vectorized(P, R, pi, gamma=1.0, theta=1e-8, max_iter=10000):
    """
    Vectorized iterative policy evaluation.

    P: (|S|, |A|, |S|)   R: (|S|, |A|, |S|)   pi: (|S|, |A|)
    """
    n_states = P.shape[0]
    V = np.zeros(n_states)

    # Precompute: P^π[s, s'] = Σ_a π(s,a) P(s'|s,a)
    # R^π[s]    = Σ_a π(s,a) Σ_{s'} P(s'|s,a) R(s,a,s')
    P_pi = np.einsum('sa,sap->sp', pi, P)                # (|S|, |S|)
    R_pi = np.einsum('sa,sap,sap->s', pi, P, R)          # (|S|,)

    for k in range(max_iter):
        V_new = R_pi + gamma * P_pi @ V
        delta = np.max(np.abs(V_new - V))
        V = V_new
        if delta < theta:
            print(f"Converged in {k+1} iterations")
            return V

    print(f"Warning: did not converge in {max_iter} iterations")
    return V

V_vec = policy_evaluation_vectorized(P, R, pi, gamma=1.0, theta=1e-8)
```

This approach computes the full backup as a single matrix-vector multiplication $V_{k+1} = R^\pi + \gamma P^\pi V_k$, which is vastly faster for large state spaces.

---

## 9. Real-World Applications

| Domain | Prediction Problem | Why Policy Evaluation? |
|---|---|---|
| **Call center staffing** | Given current scheduling policy, what is the expected wait time per state? | Evaluate before deciding to change |
| **Network routing** | Under current routing protocol, what is the expected latency per node? | Benchmark existing protocol |
| **Healthcare** | Given treatment protocol, what is patient's expected outcome? | Must evaluate current practice before proposing changes |
| **Inventory management** | Under current reorder policy, what is the expected cost per state? | Cost analysis of status quo |
| **Game AI** | Under a fixed strategy, what is the expected score from each position? | Evaluate strength of a strategy |

---

## 10. Summary

| Concept | Key Point |
|---|---|
| Prediction problem | Find $V^\pi$ for a given policy $\pi$ |
| Update rule | $V_{k+1}(s) = \sum_a \pi(a \mid s) \sum_{s'} P(s' \mid s,a)[R + \gamma V_k(s')]$ |
| Convergence | Guaranteed by contraction mapping; error $\leq \gamma^k \|V_0 - V^\pi\|_\infty$ |
| Stopping criterion | $\Delta = \max_s \|V_{k+1}(s) - V_k(s)\| < \theta$ |
| In-place variant | Updates $V(s)$ immediately; converges faster in practice |
| Two-array variant | Keeps $V_k$ and $V_{k+1}$ separate; exact Bellman iteration |
| Complexity per sweep | $O(|\mathcal{S}|^2 \times |\mathcal{A}|)$ |
| Use case | Inner loop of policy iteration; evaluating existing policies |

---

## Revision Questions

1. **Write the iterative policy evaluation update rule.** Explain each component ($\pi$, $P$, $R$, $\gamma$) and describe how they combine to compute the new value.

2. **Why does iterative policy evaluation converge?** State the contraction mapping property and explain how the Banach fixed-point theorem guarantees convergence.

3. **In the 4×4 grid world example, compute $V_2(5)$ by hand** (state 5, center of the grid). Show all four action contributions and verify the result matches the table.

4. **Compare in-place and two-array variants.** Why does the in-place variant typically converge faster? Give an intuitive explanation.

5. **How many iterations are needed** to achieve $\varepsilon = 0.001$ accuracy when $\gamma = 0.95$ and the initial error bound is 50? Show the calculation.

6. **When would you prefer the vectorized implementation** (matrix form $V_{k+1} = R^\pi + \gamma P^\pi V_k$) over the loop-based version? What are the trade-offs in terms of memory and computation?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Bellman Equations](../02-Markov-Decision-Processes/07-bellman-equations.md) | [Unit 3: Dynamic Programming](./README.md) | [Policy Iteration](./02-policy-iteration.md) |

# Chapter 3: Value Iteration

> **Unit 3: Dynamic Programming** — *Finding the optimal value function and policy in a single iterative process.*

---

## Overview

Value Iteration combines policy evaluation and policy improvement into a **single update
step**. Instead of fully evaluating a policy before improving it, value iteration applies
the Bellman **optimality** operator at every sweep: it takes the maximum over actions rather
than averaging with policy weights. This produces the optimal value function $V^*$ directly,
from which the optimal policy $\pi^*$ is extracted in one final pass. Value iteration can be
viewed as truncated policy iteration where the evaluation step is limited to exactly one sweep.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║                  VALUE ITERATION AT A GLANCE                    ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • Input : An MDP (S, A, P, R, γ)                              ║
 ║  • Output: Optimal value function V* and policy π*              ║
 ║  • Method: Iteratively apply Bellman optimality operator        ║
 ║  • Key   : No separate evaluation/improvement phases            ║
 ║  • View  : Truncated policy iteration with 1-sweep evaluation   ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. The Key Idea: Merge Evaluation and Improvement

In policy iteration, the inner evaluation loop runs many sweeps before improvement.
Value iteration asks: **what if we improve after just one sweep?**

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │        POLICY ITERATION vs VALUE ITERATION                          │
  │                                                                      │
  │  Policy Iteration:                                                   │
  │    π₀ ──[full eval]──▶ V^π₀ ──[improve]──▶ π₁ ──[full eval]──▶ ...│
  │         (many sweeps)                          (many sweeps)         │
  │                                                                      │
  │  Value Iteration:                                                    │
  │    V₀ ──[1 sweep + max]──▶ V₁ ──[1 sweep + max]──▶ V₂ ──▶ ...    │
  │         (eval + improve      (eval + improve                        │
  │          fused together)      fused together)                        │
  │                                                                      │
  │  The Bellman optimality operator does both at once:                  │
  │    V_{k+1}(s) = max_a Σ_{s'} P(s'|s,a) [R(s,a,s') + γ V_k(s')]    │
  │                 ─────                                                │
  │                 ↑ this "max" is the improvement step                 │
  └──────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Update Rule

The value iteration update applies the **Bellman optimality operator** $T^*$:

$$\boxed{V_{k+1}(s) = \max_a \sum_{s'} P(s' \mid s, a)\left[R(s, a, s') + \gamma \, V_k(s')\right]}$$

Compare with policy evaluation (Bellman expectation operator):

$$V_{k+1}(s) = \sum_a \pi(a \mid s) \sum_{s'} P(s' \mid s, a)\left[R(s, a, s') + \gamma \, V_k(s')\right]$$

The only difference: **$\max_a$ replaces $\sum_a \pi(a \mid s)$**.

```
                BACKUP DIAGRAM: VALUE ITERATION

                           V(s)
                          / | \
                        /   |   \
                      a₁    a₂    a₃       ← Take MAX over actions
                     /|\   /|\   /|\
                    / | \ / | \ / | \
                  s' s' s' s' s' s' s' s' s'  ← Weighted by P(s'|s,a)

        V_{k+1}(s) = MAX_a [ Σ_{s'} P(s'|s,a) [R + γ V_k(s')] ]
                      ───
                      ↑ key difference from policy evaluation
```

---

## 3. Value Iteration as Truncated Policy Iteration

Value iteration is equivalent to policy iteration where the evaluation phase is
**truncated to a single sweep**:

| Algorithm | Evaluation Sweeps per Improvement | Improvement Method |
|---|---|---|
| Policy Iteration | Until convergence ($\Delta < \theta$) | Greedy w.r.t. $V^\pi$ |
| Value Iteration | Exactly 1 | Built into the update (max) |
| Truncated PI ($k$ sweeps) | Exactly $k$ | Greedy w.r.t. $V_k$ |

> **Key Insight:** We don't need $V^\pi$ to be exact before improving. Even a rough
> approximation still yields a better policy. Value iteration takes this to the extreme
> with just one sweep, yet still converges to $V^*$.

---

## 4. Convergence Guarantee

The Bellman optimality operator $T^*$ is a **$\gamma$-contraction**:

$$\|T^* V_1 - T^* V_2\|_\infty \leq \gamma \|V_1 - V_2\|_\infty$$

By the Banach fixed-point theorem:
1. $T^*$ has a **unique fixed point** $V^*$ (the optimal value function).
2. Starting from any $V_0$, the sequence $V_k = (T^*)^k V_0$ converges to $V^*$.
3. The rate of convergence: $\|V_k - V^*\|_\infty \leq \gamma^k \|V_0 - V^*\|_\infty$.

### Stopping Criterion

Same as policy evaluation: stop when $\Delta = \max_s |V_{k+1}(s) - V_k(s)| < \theta$.

After stopping, the **error bound** on $V_k$ relative to $V^*$ is:

$$\|V_k - V^*\|_\infty \leq \frac{\gamma}{1 - \gamma} \cdot \Delta$$

For $\gamma = 0.99$ and $\Delta = 10^{-4}$, the bound is $\frac{0.99}{0.01} \times 10^{-4} = 0.0099$. This means even a loose stopping criterion gives a reasonable value function.

---

## 5. Algorithm Pseudocode

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║  VALUE ITERATION                                                    ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║  Input: MDP (S, A, P, R, γ), threshold θ                           ║
 ║  Output: Optimal policy π* and value function V*                    ║
 ║                                                                     ║
 ║  1. Initialize V(s) = 0 for all s ∈ S                              ║
 ║                                                                     ║
 ║  2. Repeat:                                                         ║
 ║  3.     Δ ← 0                                                      ║
 ║  4.     For each s ∈ S:                                             ║
 ║  5.         v ← V(s)                                                ║
 ║  6.         V(s) ← max_a Σ_{s'} P(s'|s,a)[R(s,a,s') + γV(s')]     ║
 ║  7.         Δ ← max(Δ, |v − V(s)|)                                 ║
 ║  8.     Until Δ < θ                                                 ║
 ║                                                                     ║
 ║  ── Extract optimal policy: ──                                      ║
 ║  9. For each s ∈ S:                                                 ║
 ║ 10.     π*(s) ← argmax_a Σ_{s'} P(s'|s,a)[R(s,a,s') + γV(s')]     ║
 ║                                                                     ║
 ║  Return π*, V                                                       ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

> **Note:** The policy is extracted **only once** after value convergence. During the
> iteration, no explicit policy is maintained — only the value function is updated.

---

## 6. Grid World Example — Step-by-Step

Using the same 4×4 grid world ($R = -1$ per step, $\gamma = 1.0$, two terminal corners):

```
    ┌─────┬─────┬─────┬─────┐
    │  T  │  1  │  2  │  3  │     T = Terminal (states 0, 15)
    ├─────┼─────┼─────┼─────┤     R = -1 for every transition
    │  4  │  5  │  6  │  7  │     γ = 1.0
    ├─────┼─────┼─────┼─────┤
    │  8  │  9  │ 10  │ 11  │
    ├─────┼─────┼─────┼─────┤
    │ 12  │ 13  │ 14  │  T  │
    └─────┴─────┴─────┴─────┘
```

### Iteration 0 (Initialization)

$V_0(s) = 0$ for all $s$.

### Iteration 1

For each state, take $\max_a Q(s, a)$:

**State 1** (row 0, col 1):
- $Q(1, N) = -1 + V_0(1) = -1 + 0 = -1$ (hits wall)
- $Q(1, S) = -1 + V_0(5) = -1 + 0 = -1$ (goes to 5)
- $Q(1, E) = -1 + V_0(2) = -1 + 0 = -1$ (goes to 2)
- $Q(1, W) = -1 + V_0(0) = -1 + 0 = -1$ (goes to terminal)

$V_1(1) = \max(-1, -1, -1, -1) = -1$

All non-terminal states get $V_1(s) = -1$ (same as iteration 1 of policy evaluation
with uniform policy — the max doesn't help yet because all neighbor values are 0).

```
  V₁:
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

**State 1** (neighbors via N=wall→1, S→5, E→2, W→T):
- $Q(1, N) = -1 + (-1) = -2$
- $Q(1, S) = -1 + (-1) = -2$
- $Q(1, E) = -1 + (-1) = -2$
- $Q(1, W) = -1 + 0 = -1$

$V_2(1) = \max(-2, -2, -2, -1) = -1$ ← the max picks West (toward terminal)!

**State 5** (neighbors: N→1, S→9, E→6, W→4):
- All neighbors have $V_1 = -1$

$V_2(5) = \max(-1+(-1), -1+(-1), -1+(-1), -1+(-1)) = -2$

**State 2** (neighbors: N=wall→2, S→6, E→3, W→1):
- $Q(2, W) = -1 + (-1) = -2$, all others also $-2$

$V_2(2) = -2$

```
  V₂:
    ┌──────┬──────┬──────┬──────┐
    │  0.0 │ -1.0 │ -2.0 │ -2.0 │
    ├──────┼──────┼──────┼──────┤
    │ -1.0 │ -2.0 │ -2.0 │ -2.0 │
    ├──────┼──────┼──────┼──────┤
    │ -2.0 │ -2.0 │ -2.0 │ -1.0 │
    ├──────┼──────┼──────┼──────┤
    │ -2.0 │ -2.0 │ -1.0 │  0.0 │
    └──────┴──────┴──────┴──────┘
```

### Iteration 3

**State 1**: $Q(1, W) = -1 + 0 = -1$, rest $\leq -2$. $V_3(1) = -1$ (no change).

**State 5**: best neighbor is state 1 or state 4 with value $-1$.
$V_3(5) = -1 + (-1) = -2$ (no change).

**State 6**: best neighbor is state 7 ($V_2 = -2$) or state 5 ($V_2 = -2$).
$V_3(6) = -1 + (-2) = -3$... wait, let's check: best is $-1 + (-2) = -3$ for all.

Actually, state 10 has value $-2$, so $Q(6, S) = -1 + (-2) = -3$.
State 2 has value $-2$, so $Q(6, N) = -1 + (-2) = -3$.

$V_3(6) = -3$. And state 10 similarly gets $V_3(10) = -3$.

```
  V₃:
    ┌──────┬──────┬──────┬──────┐
    │  0.0 │ -1.0 │ -2.0 │ -3.0 │
    ├──────┼──────┼──────┼──────┤
    │ -1.0 │ -2.0 │ -3.0 │ -2.0 │
    ├──────┼──────┼──────┼──────┤
    │ -2.0 │ -3.0 │ -2.0 │ -1.0 │
    ├──────┼──────┼──────┼──────┤
    │ -3.0 │ -2.0 │ -1.0 │  0.0 │
    └──────┴──────┴──────┴──────┘
```

This is already $V^*$! The values equal the negative Manhattan distance to the nearest
terminal. Value iteration converges after 3 sweeps for this grid (compared to 2 outer
iterations of policy iteration, but each PI iteration requires many evaluation sweeps).

### Extracting the Optimal Policy

From $V^*$, extract $\pi^*(s) = \arg\max_a \sum_{s'} P(s' \mid s, a)[R + \gamma V^*(s')]$:

```
  π* (optimal policy):

    ┌──────┬──────┬──────┬──────┐
    │  T   │  ←   │  ←   │  ↓  │
    ├──────┼──────┼──────┼──────┤
    │  ↑   │ ↑/←  │  ↓/← │  ↓  │
    ├──────┼──────┼──────┼──────┤
    │  ↑   │ ↓/→  │ ↓/→  │  ↓  │
    ├──────┼──────┼──────┼──────┤
    │  ↑   │  →   │  →   │  T  │
    └──────┴──────┴──────┴──────┘

    Each arrow points toward the nearest terminal state.
```

---

## 7. Comparison: Policy Iteration vs Value Iteration

| Criterion | Policy Iteration | Value Iteration |
|---|---|---|
| **Update rule** | Bellman expectation (average over $\pi$) | Bellman optimality ($\max$ over actions) |
| **Structure** | Separate eval + improve phases | Single combined step |
| **Evaluations per iter** | Many sweeps (until convergence) | Exactly 1 sweep |
| **Outer iterations** | Few (2–10 typically) | Many more |
| **Total sweeps** | Fewer if eval converges quickly | Fewer if eval is slow |
| **Policy stored?** | Yes, explicitly | No (extracted at end) |
| **Convergence** | Finite iterations (exact) | Asymptotic ($\gamma^k$ bound) |
| **Memory** | $O(\|\mathcal{S}\| + \|\mathcal{S}\| \times \|\mathcal{A}\|)$ for policy + values | $O(\|\mathcal{S}\|)$ for values only |
| **Best for** | Small action spaces; when exact eval is cheap | Large action spaces; simple implementation |

> **Rule of thumb:** For small, well-structured MDPs, policy iteration is often faster
> overall. For large MDPs or when you want simplicity, value iteration is preferred.

---

## 8. Python Implementation

```python
import numpy as np

def value_iteration(P, R, gamma=1.0, theta=1e-8, max_iter=10000):
    """
    Value Iteration algorithm.

    Parameters
    ----------
    P : np.ndarray, shape (|S|, |A|, |S|) — transition probabilities
    R : np.ndarray, shape (|S|, |A|, |S|) — rewards
    gamma : float — discount factor
    theta : float — convergence threshold

    Returns
    -------
    pi : np.ndarray, shape (|S|,) — optimal policy
    V  : np.ndarray, shape (|S|,) — optimal value function
    """
    n_states, n_actions, _ = P.shape
    V = np.zeros(n_states)
    history = [V.copy()]

    for k in range(max_iter):
        delta = 0.0
        for s in range(n_states):
            v_old = V[s]
            # Bellman optimality backup
            q_values = np.zeros(n_actions)
            for a in range(n_actions):
                q_values[a] = sum(
                    P[s, a, sp] * (R[s, a, sp] + gamma * V[sp])
                    for sp in range(n_states)
                )
            V[s] = np.max(q_values)
            delta = max(delta, abs(v_old - V[s]))
        history.append(V.copy())
        if delta < theta:
            print(f"Value iteration converged in {k+1} iterations (Δ={delta:.2e})")
            break

    # Extract optimal policy
    pi = np.zeros(n_states, dtype=int)
    for s in range(n_states):
        q_values = np.zeros(n_actions)
        for a in range(n_actions):
            q_values[a] = sum(
                P[s, a, sp] * (R[s, a, sp] + gamma * V[sp])
                for sp in range(n_states)
            )
        pi[s] = np.argmax(q_values)

    return pi, V, history


def value_iteration_vectorized(P, R, gamma=1.0, theta=1e-8, max_iter=10000):
    """
    Vectorized value iteration using NumPy.

    P : (|S|, |A|, |S|)   R : (|S|, |A|, |S|)
    """
    n_states, n_actions, _ = P.shape
    V = np.zeros(n_states)

    # Precompute expected rewards: R_sa[s, a] = Σ_{s'} P(s'|s,a) R(s,a,s')
    R_sa = np.einsum('sap,sap->sa', P, R)  # shape: (|S|, |A|)

    for k in range(max_iter):
        # Q[s, a] = R_sa[s,a] + γ Σ_{s'} P(s,a,s') V(s')
        Q = R_sa + gamma * np.einsum('sap,p->sa', P, V)  # shape: (|S|, |A|)
        V_new = np.max(Q, axis=1)
        delta = np.max(np.abs(V_new - V))
        V = V_new
        if delta < theta:
            print(f"Vectorized VI converged in {k+1} iterations")
            break

    pi = np.argmax(R_sa + gamma * np.einsum('sap,p->sa', P, V), axis=1)
    return pi, V


# ═══════════════════════════════════════════════════
# Example: 4×4 Grid World
# ═══════════════════════════════════════════════════
SIZE = 4
N_STATES = SIZE * SIZE
N_ACTIONS = 4  # 0=N, 1=S, 2=E, 3=W
TERMINALS = {0, 15}
ACTION_NAMES = {0: '↑', 1: '↓', 2: '→', 3: '←'}
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
        for a in range(N_ACTIONS):
            P[s, a, s] = 1.0
        continue
    r, c = state_to_rc(s)
    for a in range(N_ACTIONS):
        dr, dc = ACTION_EFFECTS[a]
        nr, nc = r + dr, c + dc
        if 0 <= nr < SIZE and 0 <= nc < SIZE:
            sp = rc_to_state(nr, nc)
        else:
            sp = s
        P[s, a, sp] = 1.0
        R[s, a, sp] = -1.0

# Run Value Iteration
print("=" * 50)
print("VALUE ITERATION — 4×4 Grid World")
print("=" * 50)

pi_star, V_star, hist = value_iteration(P, R, gamma=1.0, theta=1e-4)

print(f"\nOptimal Value Function V*:")
for row in range(SIZE):
    vals = [f"{V_star[rc_to_state(row, c)]:6.1f}" for c in range(SIZE)]
    print("  " + " ".join(vals))

print(f"\nOptimal Policy π*:")
for row in range(SIZE):
    acts = []
    for c in range(SIZE):
        s = rc_to_state(row, c)
        if s in TERMINALS:
            acts.append("  T  ")
        else:
            acts.append(f"  {ACTION_NAMES[pi_star[s]]}  ")
    print("  " + " ".join(acts))

# Show convergence
print(f"\nConvergence history (max |V_k - V*| per iteration):")
for i in range(min(6, len(hist))):
    err = np.max(np.abs(hist[i] - V_star))
    print(f"  Iter {i}: max error = {err:.4f}")
```

**Expected output:**

```
==================================================
VALUE ITERATION — 4×4 Grid World
==================================================
Value iteration converged in 4 iterations (Δ=0.00e+00)

Optimal Value Function V*:
     0.0   -1.0   -2.0   -3.0
    -1.0   -2.0   -3.0   -2.0
    -2.0   -3.0   -2.0   -1.0
    -3.0   -2.0   -1.0    0.0

Optimal Policy π*:
    T    ←    ←    ↓
    ↑    ↑    ←    ↓
    ↑    ↓    →    ↓
    ↑    →    →    T
```

---

## 9. When to Prefer Value Iteration vs Policy Iteration

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║            DECISION GUIDE: VI vs PI                             ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║                                                                 ║
 ║  Use VALUE ITERATION when:                                      ║
 ║  • Large action spaces (|A| is big)                             ║
 ║  • You want a simple, single-loop algorithm                     ║
 ║  • Memory is limited (no need to store policy)                  ║
 ║  • Approximate solutions are acceptable                         ║
 ║                                                                 ║
 ║  Use POLICY ITERATION when:                                     ║
 ║  • Small action spaces                                          ║
 ║  • You need exact convergence                                   ║
 ║  • Policy evaluation is fast (sparse transitions)               ║
 ║  • You want fewer total sweeps (PI is often faster overall)     ║
 ║                                                                 ║
 ║  Use TRUNCATED PI (k sweeps) when:                              ║
 ║  • You want a middle ground                                     ║
 ║  • k = 1 gives VI; k = ∞ gives PI                              ║
 ║  • Tuning k can outperform both pure approaches                 ║
 ║                                                                 ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 10. Real-World Applications

| Domain | Application | Why Value Iteration? |
|---|---|---|
| **Robotics** | Grid-based path planning | Simple to implement; discretized state space |
| **Game AI** | Solving small games (tic-tac-toe, simple board games) | Known transition model; want optimal strategy |
| **Operations research** | Stochastic shortest path problems | Classic application of Bellman optimality |
| **Autonomous vehicles** | Route planning with stochastic travel times | Maximize expected on-time arrival |
| **Resource management** | Optimal harvesting / scheduling | Maximize long-term yield with known dynamics |

---

## 11. Summary

| Concept | Key Point |
|---|---|
| Update rule | $V_{k+1}(s) = \max_a \sum_{s'} P(s' \mid s,a)[R(s,a,s') + \gamma V_k(s')]$ |
| Bellman optimality operator | $T^* V(s) = \max_a \sum_{s'} P(s' \mid s,a)[R + \gamma V(s')]$ |
| Convergence | $T^*$ is a $\gamma$-contraction; converges to unique fixed point $V^*$ |
| Error bound | $\|V_k - V^*\|_\infty \leq \frac{\gamma}{1-\gamma} \Delta$ after stopping |
| Relation to PI | Truncated policy iteration with 1 evaluation sweep |
| Policy extraction | $\pi^*(s) = \arg\max_a \sum_{s'} P[R + \gamma V^*(s')]$ — done once at end |
| Advantage over PI | Simpler; no separate evaluation loop; less memory |
| Disadvantage | More total sweeps; convergence is asymptotic, not exact in finite steps |

---

## Revision Questions

1. **Write the value iteration update rule** and explain how it differs from the policy evaluation update rule. What is the significance of $\max$ vs $\sum_a \pi(a \mid s)$?

2. **Prove that the Bellman optimality operator is a contraction.** State the contraction factor and explain why this guarantees convergence.

3. **In the 4×4 grid world, compute $V_2(5)$ using value iteration.** Show the Q-values for all four actions and verify $V_2(5) = \max_a Q(5, a)$.

4. **Explain the error bound** $\|V_k - V^*\|_\infty \leq \frac{\gamma}{1-\gamma}\Delta$. Why does a smaller $\gamma$ give tighter bounds but potentially a less useful value function?

5. **Why is value iteration sometimes called "truncated policy iteration with $k=1$"?** Describe the spectrum from value iteration ($k=1$) to policy iteration ($k=\infty$).

6. **Compare total computation** of policy iteration vs value iteration on a grid world with $|\mathcal{S}| = 1000$ and $|\mathcal{A}| = 4$. Under what conditions does each algorithm win?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Policy Iteration](./02-policy-iteration.md) | [Unit 3: Dynamic Programming](./README.md) | [Limitations of DP](./04-limitations-of-dp.md) |

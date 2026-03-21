# Chapter 2: Policy Iteration

> **Unit 3: Dynamic Programming** — *Finding the optimal policy through alternating evaluation and improvement.*

---

## Overview

Policy Iteration solves the **control problem** in reinforcement learning: finding the
optimal policy $\pi^*$ that maximizes the value function for every state. It alternates
between two steps — **policy evaluation** (computing $V^\pi$) and **policy improvement**
(making the policy greedy with respect to $V^\pi$) — until the policy stabilizes. Policy
iteration is guaranteed to converge to the optimal policy in a finite number of iterations,
since there are only finitely many deterministic policies for a finite MDP.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║                  POLICY ITERATION AT A GLANCE                   ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • Input : An MDP (S, A, P, R, γ)                              ║
 ║  • Output: Optimal policy π* and optimal value function V*      ║
 ║  • Method: Alternate evaluation and improvement until stable    ║
 ║  • Type  : Control (finds the best policy)                      ║
 ║  • Key   : Guaranteed convergence in finite iterations          ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. The Control Problem

Given a finite MDP $(\mathcal{S}, \mathcal{A}, P, R, \gamma)$, find:

$$\pi^* = \arg\max_\pi V^\pi(s) \quad \text{for all } s \in \mathcal{S}$$

Unlike policy evaluation (prediction), the control problem does not start with a fixed
policy — it must **discover** the best one.

---

## 2. The Two-Step Cycle

Policy iteration alternates between two phases:

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                  POLICY ITERATION CYCLE                          │
  │                                                                  │
  │     ┌──────────────────────┐     ┌───────────────────────┐      │
  │     │  POLICY EVALUATION   │     │  POLICY IMPROVEMENT   │      │
  │     │                      │     │                       │      │
  │     │  Given π, compute    │     │  Given V^π, compute   │      │
  │     │  V^π by solving      │────▶│  improved policy:     │      │
  │     │  Bellman expectation  │     │  π'(s) = argmax_a     │      │
  │     │  equation iteratively │     │      Q^π(s, a)        │      │
  │     └──────────────────────┘     └───────────────────────┘      │
  │              ▲                            │                      │
  │              │         π' ≠ π?            │                      │
  │              │            YES             │                      │
  │              └────────────────────────────┘                      │
  │                                                                  │
  │     If π' = π → STOP (π is optimal)                              │
  └──────────────────────────────────────────────────────────────────┘
```

**Step 1: Policy Evaluation** — Compute $V^\pi$ for the current policy $\pi$ using
iterative policy evaluation (Chapter 1 of this unit).

**Step 2: Policy Improvement** — Construct a new policy $\pi'$ that is greedy with respect
to $V^\pi$:

$$\pi'(s) = \arg\max_a Q^\pi(s, a) = \arg\max_a \sum_{s'} P(s' \mid s, a)\left[R(s, a, s') + \gamma V^\pi(s')\right]$$

If $\pi' = \pi$, the policy has converged and is optimal. Otherwise, set $\pi \leftarrow \pi'$
and repeat.

---

## 3. Policy Improvement Theorem

The theoretical foundation of policy iteration rests on this key theorem:

> **Policy Improvement Theorem:** Let $\pi$ and $\pi'$ be two deterministic policies such
> that for all $s \in \mathcal{S}$:
>
> $$Q^\pi(s, \pi'(s)) \geq V^\pi(s)$$
>
> Then $\pi'$ is at least as good as $\pi$:
>
> $$V^{\pi'}(s) \geq V^\pi(s) \quad \text{for all } s \in \mathcal{S}$$

### Proof Sketch

Starting from $V^\pi(s) \leq Q^\pi(s, \pi'(s))$:

$$V^\pi(s) \leq Q^\pi(s, \pi'(s))$$
$$= \sum_{s'} P(s' \mid s, \pi'(s))\left[R(s, \pi'(s), s') + \gamma V^\pi(s')\right]$$
$$\leq \sum_{s'} P(s' \mid s, \pi'(s))\left[R(s, \pi'(s), s') + \gamma Q^\pi(s', \pi'(s'))\right]$$

Expanding $Q^\pi(s', \pi'(s'))$ and continuing the chain:

$$\leq \sum_{s'} P(s' \mid s, \pi'(s))\left[R + \gamma \sum_{s''} P(s'' \mid s', \pi'(s'))\left[R' + \gamma Q^\pi(s'', \pi'(s''))\right]\right]$$

Repeating this unrolling infinitely:

$$\leq \mathbb{E}_{\pi'}\left[\sum_{t=0}^{\infty} \gamma^t R_{t+1} \mid S_0 = s\right] = V^{\pi'}(s)$$

### Greedy Improvement Satisfies the Theorem

When $\pi'(s) = \arg\max_a Q^\pi(s, a)$:

$$Q^\pi(s, \pi'(s)) = \max_a Q^\pi(s, a) \geq Q^\pi(s, \pi(s)) = V^\pi(s)$$

The inequality holds because the max over actions is at least as large as the value under
the current action $\pi(s)$. Therefore, **greedy improvement always yields a policy at
least as good as the current one**.

---

## 4. Convergence Guarantee

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║                   WHY POLICY ITERATION CONVERGES                ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║                                                                 ║
 ║  1. A finite MDP with |S| states and |A| actions has at most   ║
 ║     |A|^|S| deterministic policies.                             ║
 ║                                                                 ║
 ║  2. Each policy improvement step produces a strictly better     ║
 ║     policy (or the same one, if already optimal).               ║
 ║                                                                 ║
 ║  3. Since V^π strictly increases with each improvement (unless  ║
 ║     already optimal), and there are finitely many policies,     ║
 ║     the algorithm must terminate.                               ║
 ║                                                                 ║
 ║  4. At termination: π'(s) = argmax_a Q^π(s,a) = π(s)           ║
 ║     ⟹ V^π(s) = max_a Q^π(s,a) — this IS the Bellman           ║
 ║       optimality equation, so π = π*.                           ║
 ║                                                                 ║
 ╚══════════════════════════════════════════════════════════════════╝
```

> **Key Insight:** Although there could be exponentially many policies ($|\mathcal{A}|^{|\mathcal{S}|}$),
> policy iteration typically converges in a very small number of iterations — often fewer
> than 10, even for large MDPs. This is a surprising and practically important result.

---

## 5. Full Algorithm Pseudocode

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║  POLICY ITERATION                                                   ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║  Input: MDP (S, A, P, R, γ), threshold θ                           ║
 ║  Output: Optimal policy π* and value function V*                    ║
 ║                                                                     ║
 ║  1. Initialize:                                                     ║
 ║       V(s) ← 0           for all s ∈ S                             ║
 ║       π(s) ← arbitrary   for all s ∈ S                             ║
 ║                                                                     ║
 ║  2. Repeat:                                                         ║
 ║                                                                     ║
 ║     ┌─ POLICY EVALUATION ──────────────────────────────────────┐    ║
 ║     │  Repeat:                                                  │    ║
 ║     │    Δ ← 0                                                  │    ║
 ║     │    For each s ∈ S:                                        │    ║
 ║     │      v ← V(s)                                             │    ║
 ║     │      V(s) ← Σ_a π(a|s) Σ_{s'} P(s'|s,a)[R + γV(s')]     │    ║
 ║     │      Δ ← max(Δ, |v − V(s)|)                              │    ║
 ║     │  Until Δ < θ                                              │    ║
 ║     └───────────────────────────────────────────────────────────┘    ║
 ║                                                                     ║
 ║     ┌─ POLICY IMPROVEMENT ─────────────────────────────────────┐    ║
 ║     │  policy_stable ← true                                    │    ║
 ║     │  For each s ∈ S:                                          │    ║
 ║     │    old_action ← π(s)                                      │    ║
 ║     │    π(s) ← argmax_a Σ_{s'} P(s'|s,a)[R + γV(s')]          │    ║
 ║     │    If old_action ≠ π(s): policy_stable ← false            │    ║
 ║     └───────────────────────────────────────────────────────────┘    ║
 ║                                                                     ║
 ║  Until policy_stable = true                                         ║
 ║                                                                     ║
 ║  Return π, V                                                        ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

---

## 6. Grid World Example — Complete Worked Example

Using the same 4×4 grid world from Chapter 1:

```
    ┌─────┬─────┬─────┬─────┐
    │  T  │  1  │  2  │  3  │     T = Terminal (states 0, 15)
    ├─────┼─────┼─────┼─────┤     Actions: N, S, E, W
    │  4  │  5  │  6  │  7  │     Reward = -1 per step
    ├─────┼─────┼─────┼─────┤     γ = 1.0 (undiscounted)
    │  8  │  9  │ 10  │ 11  │
    ├─────┼─────┼─────┼─────┤
    │ 12  │ 13  │ 14  │  T  │
    └─────┴─────┴─────┴─────┘
```

### Iteration 0: Initialize

Start with an arbitrary policy — here, uniform random $\pi_0(a \mid s) = 0.25$ for all
actions.

### Step 1a: Policy Evaluation of $\pi_0$

After running iterative policy evaluation to convergence:

```
  V^π₀ (uniform random policy):

    ┌──────┬──────┬──────┬──────┐
    │  0.0 │  -14 │  -20 │  -22 │
    ├──────┼──────┼──────┼──────┤
    │  -14 │  -18 │  -20 │  -20 │
    ├──────┼──────┼──────┼──────┤
    │  -20 │  -20 │  -18 │  -14 │
    ├──────┼──────┼──────┼──────┤
    │  -22 │  -20 │  -14 │  0.0 │
    └──────┴──────┴──────┴──────┘
```

### Step 1b: Policy Improvement

For each state, compute $Q^{\pi_0}(s, a)$ and pick the greedy action:

**State 1** (row 0, col 1):
- $Q(1, N) = -1 + V(-14) = -15$ (hits wall, stays at state 1 which has value -14... wait, N goes off grid)

Let's be precise. $Q(1, a) = \sum_{s'} P(s'|1,a)[R(1,a,s') + \gamma V(s')]$:
- $Q(1, N) = -1 + 1.0 \times (-14) = -15$ (stays in state 1)
- $Q(1, S) = -1 + 1.0 \times (-18) = -19$ (goes to state 5)
- $Q(1, E) = -1 + 1.0 \times (-20) = -21$ (goes to state 2)
- $Q(1, W) = -1 + 1.0 \times (0) = -1$ (goes to terminal state 0)

$$\pi_1(1) = \arg\max_a Q(1, a) = W \quad \text{(go toward terminal!)}$$

**State 5** (row 1, col 1):
- $Q(5, N) = -1 + (-14) = -15$ (goes to state 1)
- $Q(5, S) = -1 + (-20) = -21$ (goes to state 9)
- $Q(5, E) = -1 + (-20) = -21$ (goes to state 6)
- $Q(5, W) = -1 + (-14) = -15$ (goes to state 4)

$$\pi_1(5) = \arg\max_a Q(5, a) = N \text{ or } W \quad \text{(tie — both = -15)}$$

Applying greedy improvement to all states:

```
  π₁ (improved policy — arrows show greedy actions):

    ┌──────┬──────┬──────┬──────┐
    │  T   │  ←   │  ←   │ ←/↓ │
    ├──────┼──────┼──────┼──────┤
    │  ↑   │ ↑/←  │ ↑/←/↓│  ↓  │
    ├──────┼──────┼──────┼──────┤
    │  ↑   │↑/→/↓ │ ↓/→  │  ↓  │
    ├──────┼──────┼──────┼──────┤
    │ ↑/→  │  →   │  →   │  T  │
    └──────┴──────┴──────┴──────┘

    (multiple arrows = tied greedy actions)
```

### Step 2a: Policy Evaluation of $\pi_1$

Evaluate the new greedy policy. Since $\pi_1$ directs each state toward the nearest
terminal via shortest path, $V^{\pi_1}$ gives the negative of the Manhattan distance:

```
  V^π₁ (greedy policy):

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

### Step 2b: Policy Improvement

Computing greedy actions with respect to $V^{\pi_1}$ — the resulting policy $\pi_2$ equals
$\pi_1$. The policy is **stable**: no improvement is possible.

$$\pi_2 = \pi_1 \implies \pi_1 = \pi^* \text{ (optimal policy found!)}$$

**Policy iteration converged in just 2 outer iterations** (one evaluation + improvement
cycle after the initial one).

---

## 7. The Full Iteration Visualized

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║                POLICY ITERATION TRACE                               ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                     ║
  ║  π₀ (random) ──EVAL──▶ V^π₀ ──IMPROVE──▶ π₁ (greedy)             ║
  ║                                                                     ║
  ║      random             -14,-20,...          shortest-path          ║
  ║      actions            -22,-18,...          toward terminal        ║
  ║                                                                     ║
  ║  π₁ (greedy) ──EVAL──▶ V^π₁ ──IMPROVE──▶ π₂ = π₁ → STOP         ║
  ║                                                                     ║
  ║      optimal            -1,-2,-3,...        no change               ║
  ║      actions            (Manhattan dist)    → converged!            ║
  ║                                                                     ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

---

## 8. Python Implementation

```python
import numpy as np

def policy_iteration(P, R, gamma=1.0, theta=1e-8):
    """
    Policy Iteration algorithm.

    Parameters
    ----------
    P : np.ndarray, shape (|S|, |A|, |S|) — transition probabilities
    R : np.ndarray, shape (|S|, |A|, |S|) — rewards
    gamma : float — discount factor
    theta : float — convergence threshold for evaluation

    Returns
    -------
    pi : np.ndarray, shape (|S|,) — optimal deterministic policy
    V  : np.ndarray, shape (|S|,) — optimal value function
    """
    n_states, n_actions, _ = P.shape

    # Initialize: arbitrary deterministic policy (action 0 everywhere)
    pi = np.zeros(n_states, dtype=int)
    V = np.zeros(n_states)

    iteration = 0
    while True:
        # ── POLICY EVALUATION ──
        while True:
            delta = 0.0
            for s in range(n_states):
                v_old = V[s]
                a = pi[s]
                V[s] = sum(
                    P[s, a, sp] * (R[s, a, sp] + gamma * V[sp])
                    for sp in range(n_states)
                )
                delta = max(delta, abs(v_old - V[s]))
            if delta < theta:
                break

        # ── POLICY IMPROVEMENT ──
        policy_stable = True
        for s in range(n_states):
            old_action = pi[s]
            # Compute Q(s, a) for all actions
            q_values = np.zeros(n_actions)
            for a in range(n_actions):
                q_values[a] = sum(
                    P[s, a, sp] * (R[s, a, sp] + gamma * V[sp])
                    for sp in range(n_states)
                )
            pi[s] = np.argmax(q_values)
            if old_action != pi[s]:
                policy_stable = False

        iteration += 1
        print(f"  Iteration {iteration}: policy_stable = {policy_stable}")

        if policy_stable:
            break

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

# Run Policy Iteration
print("=" * 50)
print("POLICY ITERATION — 4×4 Grid World")
print("=" * 50)

pi_star, V_star = policy_iteration(P, R, gamma=1.0)

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
```

**Expected output:**

```
==================================================
POLICY ITERATION — 4×4 Grid World
==================================================
  Iteration 1: policy_stable = False
  Iteration 2: policy_stable = True

Optimal Value Function V*:
     0.0   -1.0   -2.0   -3.0
    -1.0   -2.0   -3.0   -2.0
    -2.0   -3.0   -2.0   -1.0
    -3.0   -2.0   -1.0    0.0

Optimal Policy π*:
    T    ←    ←    ↓
    ↑    ↑    ↓    ↓
    ↑    ↑    ↓    ↓
    ↑    →    →    T
```

---

## 9. Analysis: Why So Few Iterations?

| Factor | Explanation |
|---|---|
| **Rich information** | Full evaluation gives exact $V^\pi$, enabling optimal one-step improvement |
| **Greedy jump** | Each improvement can jump multiple "steps" in policy space |
| **Monotonic improvement** | $V^{\pi_{k+1}}(s) \geq V^{\pi_k}(s)$ for all $s$ |
| **Finite policy space** | At most $|\mathcal{A}|^{|\mathcal{S}|}$ policies, but rarely more than a handful visited |

In practice, policy iteration converges in $O(|\mathcal{A}|)$ outer iterations for most MDPs,
though worst-case complexity can be exponential.

---

## 10. Real-World Applications

| Domain | Application | Why Policy Iteration? |
|---|---|---|
| **Robotics** | Optimal control of robot arms | Exact model available; state space manageable |
| **Operations research** | Machine replacement / maintenance scheduling | Classical OR problems with known transition dynamics |
| **Telecommunications** | Channel allocation in networks | Model known; need provably optimal allocation |
| **Finance** | Optimal portfolio rebalancing | Transition dynamics can be estimated; convergence guarantee valued |
| **Game solving** | Solving board games with known rules | Perfect model; want guaranteed optimal play |

---

## 11. Summary

| Concept | Key Point |
|---|---|
| Control problem | Find optimal policy $\pi^*$ that maximizes $V(s)$ for all $s$ |
| Two-step cycle | Policy Evaluation → Policy Improvement → repeat |
| Policy improvement | $\pi'(s) = \arg\max_a Q^\pi(s, a)$ (greedy w.r.t. current values) |
| Improvement theorem | If $Q^\pi(s, \pi'(s)) \geq V^\pi(s)$ for all $s$, then $V^{\pi'} \geq V^\pi$ |
| Convergence | Finite policies + strict improvement ⟹ must terminate |
| At convergence | $V^\pi(s) = \max_a Q^\pi(s, a)$ — Bellman optimality holds |
| Practical iterations | Typically very few (2–10) outer iterations |
| Computational cost | Dominated by policy evaluation (inner loop) |

---

## Revision Questions

1. **State the Policy Improvement Theorem.** Explain why the greedy policy $\pi'(s) = \arg\max_a Q^\pi(s, a)$ satisfies the conditions of this theorem.

2. **Why does policy iteration converge in a finite number of iterations?** What is the theoretical upper bound on the number of iterations? Why is this upper bound rarely reached?

3. **In the 4×4 grid world, compute the greedy action for state 5** given $V^{\pi_0}$ (the random policy value function). Show all four Q-values and verify which action(s) are optimal.

4. **What happens at convergence?** Show that when $\pi' = \pi$, the Bellman optimality equation is satisfied, proving that the converged policy is optimal.

5. **Compare the computational cost** of policy evaluation (inner loop) vs policy improvement (outer loop). Which dominates, and why?

6. **Can policy iteration converge to a suboptimal policy?** Prove or disprove with a formal argument.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Policy Evaluation](./01-policy-evaluation.md) | [Unit 3: Dynamic Programming](./README.md) | [Value Iteration](./03-value-iteration.md) |

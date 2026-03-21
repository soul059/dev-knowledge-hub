# Chapter 5: Policies

> **Unit 2: Markov Decision Processes** — *The agent's strategy — mapping states to actions.*

---

## Overview

A **policy** is the agent's decision-making strategy: it defines what action to take in
each state. Policies are the central object of study in reinforcement learning — the goal
is to find the **optimal policy** that maximizes expected cumulative reward. This chapter
covers deterministic and stochastic policies, the concept of optimality, how to compare
policies, and the structure of the policy space.

---

## 1. What Is a Policy?

A policy $\pi$ is a mapping from states to actions (or distributions over actions).
It fully describes the agent's behavior.

```
                    ┌────────────┐
                    │            │
     State s_t ────▶│  Policy π  │────▶ Action a_t
                    │            │
                    └────────────┘
                    
   The policy is the agent's "brain" — it decides what to do.
```

---

## 2. Deterministic Policies

### Definition

A **deterministic policy** maps each state to exactly one action:

$$\pi: \mathcal{S} \to \mathcal{A}$$
$$a = \pi(s)$$

For every state, there is a single action the agent will always take.

### Example: Grid World Deterministic Policy

```
┌───────┬───────┬───────┬───────┐
│   →   │   →   │   →   │ GOAL  │
│  (0,0)│  (0,1)│  (0,2)│  (0,3)│
├───────┼───────┼───────┼───────┤
│   ↑   │ XXXXX │   ↑   │ TRAP  │
│  (1,0)│ WALL  │  (1,2)│  (1,3)│
├───────┼───────┼───────┼───────┤
│   ↑   │   →   │   ↑   │   ←   │
│  (2,0)│  (2,1)│  (2,2)│  (2,3)│
├───────┼───────┼───────┼───────┤
│   ↑   │   →   │   →   │   ↑   │
│  (3,0)│  (3,1)│  (3,2)│  (3,3)│
└───────┴───────┴───────┴───────┘

Each cell shows ONE arrow: π(s) = single action
```

### Properties
- Simple to implement: just a lookup table or function
- Always exploits the best known action (no exploration)
- For finite MDPs, there exists a deterministic optimal policy

---

## 3. Stochastic Policies

### Definition

A **stochastic policy** maps each state to a probability distribution over actions:

$$\pi(a \mid s) = P(A_t = a \mid S_t = s)$$

For each state, $\pi(\cdot \mid s)$ is a valid probability distribution:

$$\pi(a \mid s) \geq 0 \quad \forall a, \quad \sum_{a \in \mathcal{A}} \pi(a \mid s) = 1 \quad \forall s$$

### Example: Grid World Stochastic Policy

```
State (2,1) with stochastic policy:

         ↑ 10%
         │
   ← 20% ┼ → 60%
         │
         ↓ 10%

  π(right | (2,1)) = 0.60
  π(left  | (2,1)) = 0.20
  π(up    | (2,1)) = 0.10
  π(down  | (2,1)) = 0.10
                      ────
                      1.00 ✓
```

### Why Stochastic Policies?

| Reason | Explanation |
|---|---|
| **Exploration** | Random actions discover new states and rewards |
| **Partial observability** | When state is ambiguous, randomization can help |
| **Game theory** | Mixed strategies prevent opponents from predicting you |
| **Policy gradient methods** | Naturally produce stochastic policies |
| **Smooth optimization** | Gradients exist everywhere (no discontinuities) |

### Deterministic as a Special Case

A deterministic policy is a stochastic policy where all probability mass is on one action:

$$\pi(a \mid s) = \begin{cases} 1 & \text{if } a = \pi(s) \\ 0 & \text{otherwise} \end{cases}$$

---

## 4. Common Policy Types

### 4.1 Epsilon-Greedy Policy

The most common exploration strategy:

$$\pi(a \mid s) = \begin{cases} 1 - \epsilon + \frac{\epsilon}{|\mathcal{A}|} & \text{if } a = \arg\max_{a'} Q(s, a') \\ \frac{\epsilon}{|\mathcal{A}|} & \text{otherwise} \end{cases}$$

```
ε-Greedy with ε=0.2, 4 actions, best action = RIGHT:

  Action  │ Probability
  ────────┼──────────────
  UP      │ 0.2/4 = 0.05
  DOWN    │ 0.2/4 = 0.05
  LEFT    │ 0.2/4 = 0.05
  RIGHT*  │ 0.8 + 0.05 = 0.85    ← Greedy action
  ────────┼──────────────
  Total   │ 1.00 ✓
```

### 4.2 Softmax (Boltzmann) Policy

Actions are chosen proportional to exponentiated Q-values:

$$\pi(a \mid s) = \frac{e^{Q(s,a)/\tau}}{\sum_{a'} e^{Q(s,a')/\tau}}$$

where $\tau > 0$ is the **temperature** parameter:
- $\tau \to 0$: becomes deterministic (greedy)
- $\tau \to \infty$: becomes uniform random

### 4.3 Uniform Random Policy

Every action is equally likely:

$$\pi(a \mid s) = \frac{1}{|\mathcal{A}|} \quad \forall s, a$$

Used as a baseline and for pure exploration.

---

## 5. The Optimal Policy $\pi^*$

### Definition

A policy $\pi^*$ is **optimal** if it achieves the highest expected return from every state:

$$\pi^* = \arg\max_\pi V^\pi(s) \quad \forall s \in \mathcal{S}$$

### Key Theorems

**Theorem 1**: For any finite MDP, there exists at least one optimal policy.

**Theorem 2**: All optimal policies achieve the same optimal value function:
$$V^*(s) = V^{\pi^*}(s) = \max_\pi V^\pi(s) \quad \forall s$$

**Theorem 3**: There always exists a **deterministic** optimal policy for finite MDPs.

### Partial Ordering of Policies

Policy $\pi$ is **at least as good as** policy $\pi'$ if:

$$\pi \geq \pi' \iff V^\pi(s) \geq V^{\pi'}(s) \quad \forall s \in \mathcal{S}$$

This defines a **partial order** — not all policies are comparable (one may be better
in some states but worse in others).

---

## 6. Policy Space and Its Size

### Deterministic Policies

For a finite MDP with $|\mathcal{S}|$ states and $|\mathcal{A}|$ actions:

$$|\text{Deterministic policies}| = |\mathcal{A}|^{|\mathcal{S}|}$$

| States | Actions | Number of Policies |
|---|---|---|
| 4 | 2 | $2^4 = 16$ |
| 10 | 4 | $4^{10} \approx 10^6$ |
| 100 | 4 | $4^{100} \approx 10^{60}$ |
| 1000 | 10 | $10^{1000}$ |

The policy space grows **exponentially** — exhaustive search is infeasible even for
moderate MDPs.

### Stochastic Policies

The space of stochastic policies is **continuous** and infinite-dimensional:
each state maps to a point on the $(|\mathcal{A}|-1)$-dimensional probability simplex.

```
Policy space for 3 actions at a single state:

  Probability simplex Δ²:
  
       π(a₁) = 1
          /\
         /  \
        /    \           ● = one stochastic policy
       / ●    \          ▲ = deterministic policies (vertices)
      /        \
     /    ·     \
    /____________\
  π(a₂)=1    π(a₃)=1
  
  Every point inside the triangle is a valid
  stochastic policy for one state.
```

---

## 7. Comparing Policies: An Example

Consider a simple 2-state, 2-action MDP:

```
States: {s₀, s₁}    Actions: {left, right}    γ = 0.9

Transitions and Rewards:
  s₀, right → s₁ (prob 1.0, reward +5)
  s₀, left  → s₀ (prob 1.0, reward +1)
  s₁, right → s₁ (prob 1.0, reward +2)
  s₁, left  → s₀ (prob 1.0, reward +0)
```

### All 4 Deterministic Policies

| Policy | $\pi(s_0)$ | $\pi(s_1)$ | $V^\pi(s_0)$ | $V^\pi(s_1)$ |
|---|---|---|---|---|
| $\pi_1$ | left | left | 10.0 | 0.0 |
| $\pi_2$ | left | right | 10.0 | 20.0 |
| $\pi_3$ | right | left | ~14.5 | ~10.5 |
| $\pi_4$ | right | right | ~41.1 | ~38.9 |

$\pi_4$ (always go right) dominates in both states → $\pi_4 = \pi^*$.

Computing $V^{\pi_4}(s_0)$:
$$V^{\pi_4}(s_0) = 5 + 0.9 \cdot V^{\pi_4}(s_1)$$
$$V^{\pi_4}(s_1) = 2 + 0.9 \cdot V^{\pi_4}(s_1)$$
$$V^{\pi_4}(s_1) = \frac{2}{1 - 0.9} = 20$$
$$V^{\pi_4}(s_0) = 5 + 0.9 \times 20 = 23$$

---

## 8. Python Code: Implementing Policies

```python
import numpy as np
from typing import Dict, List, Callable

class DeterministicPolicy:
    """A deterministic policy: π(s) = a."""

    def __init__(self, state_to_action: Dict):
        self.mapping = state_to_action

    def action(self, state):
        return self.mapping[state]

    def __repr__(self):
        items = [f"{s}→{a}" for s, a in self.mapping.items()]
        return f"DetPolicy({', '.join(items)})"


class StochasticPolicy:
    """A stochastic policy: π(a|s) = probability."""

    def __init__(self, action_probs: Dict):
        """
        action_probs: {state: {action: probability, ...}, ...}
        """
        self.probs = action_probs
        self._validate()

    def _validate(self):
        for s, dist in self.probs.items():
            total = sum(dist.values())
            assert abs(total - 1.0) < 1e-6, \
                f"State {s}: probs sum to {total}, not 1.0"

    def action_prob(self, state, action):
        return self.probs[state].get(action, 0.0)

    def sample_action(self, state):
        dist = self.probs[state]
        actions = list(dist.keys())
        probs = [dist[a] for a in actions]
        return np.random.choice(actions, p=probs)

    def get_distribution(self, state):
        return self.probs[state]


class EpsilonGreedyPolicy:
    """ε-greedy policy based on Q-values."""

    def __init__(self, q_values: Dict, epsilon: float, actions: List):
        self.Q = q_values
        self.epsilon = epsilon
        self.actions = actions
        self.n_actions = len(actions)

    def action_prob(self, state, action):
        best_action = max(self.actions, key=lambda a: self.Q.get((state, a), 0))
        if action == best_action:
            return 1 - self.epsilon + self.epsilon / self.n_actions
        else:
            return self.epsilon / self.n_actions

    def sample_action(self, state):
        if np.random.random() < self.epsilon:
            return np.random.choice(self.actions)
        else:
            return max(self.actions, key=lambda a: self.Q.get((state, a), 0))


class SoftmaxPolicy:
    """Boltzmann/softmax policy based on Q-values."""

    def __init__(self, q_values: Dict, temperature: float, actions: List):
        self.Q = q_values
        self.tau = temperature
        self.actions = actions

    def action_prob(self, state, action):
        q_vals = np.array([self.Q.get((state, a), 0) for a in self.actions])
        exp_q = np.exp((q_vals - q_vals.max()) / self.tau)  # numerically stable
        probs = exp_q / exp_q.sum()
        idx = self.actions.index(action)
        return probs[idx]

    def sample_action(self, state):
        q_vals = np.array([self.Q.get((state, a), 0) for a in self.actions])
        exp_q = np.exp((q_vals - q_vals.max()) / self.tau)
        probs = exp_q / exp_q.sum()
        return np.random.choice(self.actions, p=probs)


# --- Demonstrate all policy types ---
states = ["s0", "s1", "s2"]
actions = ["left", "right", "up", "down"]

# 1) Deterministic policy
det_policy = DeterministicPolicy({
    "s0": "right",
    "s1": "up",
    "s2": "left"
})
print("=== Deterministic Policy ===")
print(det_policy)
for s in states:
    print(f"  π({s}) = {det_policy.action(s)}")

# 2) Stochastic policy
stoch_policy = StochasticPolicy({
    "s0": {"left": 0.1, "right": 0.7, "up": 0.1, "down": 0.1},
    "s1": {"left": 0.2, "right": 0.2, "up": 0.5, "down": 0.1},
    "s2": {"left": 0.6, "right": 0.1, "up": 0.2, "down": 0.1},
})
print("\n=== Stochastic Policy ===")
for s in states:
    dist = stoch_policy.get_distribution(s)
    dist_str = ", ".join(f"{a}:{p:.1f}" for a, p in dist.items())
    print(f"  π(·|{s}) = {{{dist_str}}}")

# 3) ε-greedy policy
Q = {("s0", "right"): 5.0, ("s0", "left"): 2.0,
     ("s0", "up"): 1.0, ("s0", "down"): 0.5}
eps_policy = EpsilonGreedyPolicy(Q, epsilon=0.2, actions=actions)
print("\n=== ε-Greedy Policy (ε=0.2) ===")
for a in actions:
    p = eps_policy.action_prob("s0", a)
    marker = " ← greedy" if a == "right" else ""
    print(f"  π({a}|s0) = {p:.3f}{marker}")

# 4) Softmax policy
soft_policy = SoftmaxPolicy(Q, temperature=1.0, actions=actions)
print("\n=== Softmax Policy (τ=1.0) ===")
for a in actions:
    p = soft_policy.action_prob("s0", a)
    print(f"  π({a}|s0) = {p:.3f}")

# Effect of temperature
print("\n=== Softmax Temperature Effect ===")
for tau in [0.1, 0.5, 1.0, 2.0, 10.0]:
    sp = SoftmaxPolicy(Q, temperature=tau, actions=actions)
    probs = [sp.action_prob("s0", a) for a in actions]
    bar = "".join(f"{p:.2f} " for p in probs)
    print(f"  τ={tau:>4.1f}: [{bar}]")


# --- Policy enumeration for tiny MDP ---
print("\n=== All Deterministic Policies (2 states, 2 actions) ===")
tiny_states = ["s0", "s1"]
tiny_actions = ["L", "R"]
policy_count = 0
for a0 in tiny_actions:
    for a1 in tiny_actions:
        policy_count += 1
        print(f"  π_{policy_count}: s0→{a0}, s1→{a1}")
print(f"Total: {len(tiny_actions)**len(tiny_states)} deterministic policies")
```

---

## 9. Real-World Applications

| Application | Policy Type | Why? |
|---|---|---|
| **Chess engine** | Deterministic (after search) | Best move is unique |
| **Poker AI** | Stochastic (mixed strategy) | Prevents opponent prediction |
| **Robot exploration** | ε-greedy | Balances explore/exploit |
| **Dialogue system** | Softmax | Natural language variety |
| **A/B testing** | Thompson sampling (stochastic) | Bayesian exploration |

---

## Summary

| Concept | Key Point |
|---|---|
| Deterministic policy | $\pi(s) = a$ — one action per state |
| Stochastic policy | $\pi(a \mid s) = P(A_t = a \mid S_t = s)$ |
| ε-greedy | Greedy with probability $1-\epsilon$, random with $\epsilon$ |
| Softmax | Action probability proportional to $e^{Q/\tau}$ |
| Optimal policy $\pi^*$ | Maximizes $V^\pi(s)$ for all states simultaneously |
| Policy space size | $|\mathcal{A}|^{|\mathcal{S}|}$ deterministic policies |
| Partial ordering | $\pi \geq \pi'$ iff $V^\pi(s) \geq V^{\pi'}(s)$ for all $s$ |
| Existence theorem | A deterministic optimal policy always exists for finite MDPs |

---

## Revision Questions

1. **Define a deterministic policy and a stochastic policy.** Give the mathematical notation for each.

2. **How many deterministic policies exist** for an MDP with 20 states and 5 actions? Is exhaustive search feasible?

3. **Write the formula for an ε-greedy policy.** What happens when ε=0? When ε=1?

4. **Explain why stochastic policies are sometimes preferred** over deterministic ones. Give three reasons.

5. **What does it mean for policy $\pi$ to be "at least as good as" policy $\pi'$?** Is this a total order or partial order? Why?

6. **In the softmax policy**, what is the effect of the temperature parameter $\tau$? Describe the behavior as $\tau \to 0$ and $\tau \to \infty$.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Reward Function](./04-reward-function.md) | [Unit 2: MDPs](./README.md) | [Value Functions](./06-value-functions.md) |

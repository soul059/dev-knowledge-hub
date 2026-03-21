# Chapter 6: Value Functions

> **Unit 2: Markov Decision Processes** — *Quantifying how good it is to be in a state or take an action.*

---

## Overview

Value functions are the central tool for evaluating and improving policies in reinforcement
learning. They assign a numeric value to each state (or state-action pair) representing the
expected return from that point onward, under a given policy. This chapter covers the
state-value function $V^\pi$, the action-value function $Q^\pi$, their relationship,
optimal value functions, and how to compute them for small MDPs.

---

## 1. State-Value Function $V^\pi(s)$

### Definition

The **state-value function** of a policy $\pi$ gives the expected return starting from
state $s$ and following policy $\pi$ thereafter:

$$V^\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]$$

$$V^\pi(s) = \mathbb{E}_\pi\left[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \;\middle|\; S_t = s\right]$$

### Interpretation

$V^\pi(s)$ answers the question: **"How good is it to be in state $s$ if I follow policy $\pi$?"**

```
                    ┌─────────────────────────────────────┐
                    │  V^π(s) = Expected total reward     │
                    │  from state s onward, under π       │
                    └─────────────────────────────────────┘
                    
    State s ──π──▶ s' ──π──▶ s'' ──π──▶ s''' ──π──▶ ...
         │         │          │           │
         ▼         ▼          ▼           ▼
        r_1       r_2        r_3         r_4
         
    V^π(s) = E[r_1 + γr_2 + γ²r_3 + γ³r_4 + ...]
```

### Properties

- $V^\pi(s_T) = 0$ for terminal states (no future rewards)
- $V^\pi(s) \in \left[-\frac{R_{\max}}{1-\gamma}, \frac{R_{\max}}{1-\gamma}\right]$ for bounded rewards
- $V^\pi$ depends on the policy — different policies yield different value functions

---

## 2. Action-Value Function $Q^\pi(s, a)$

### Definition

The **action-value function** gives the expected return starting from state $s$, taking
action $a$, and then following policy $\pi$:

$$Q^\pi(s, a) = \mathbb{E}_\pi[G_t \mid S_t = s, A_t = a]$$

$$Q^\pi(s, a) = \mathbb{E}_\pi\left[\sum_{k=0}^{\infty} \gamma^k R_{t+k+1} \;\middle|\; S_t = s, A_t = a\right]$$

### Interpretation

$Q^\pi(s, a)$ answers: **"How good is it to take action $a$ in state $s$, then follow $\pi$?"**

```
                    ┌─────────────────────────────────────┐
                    │  Q^π(s,a) = Expected total reward   │
                    │  taking a in s, then following π    │
                    └─────────────────────────────────────┘
                    
    State s ──a──▶ s' ──π──▶ s'' ──π──▶ s''' ──π──▶ ...
         │         │          │           │
         ▼         ▼          ▼           ▼
        r_1       r_2        r_3         r_4
         ▲
         │
    First action is 'a' (might differ from π(s))
```

### Why Q-Values Matter

- Directly enable **action selection**: pick $a = \arg\max_a Q^\pi(s, a)$
- Q-learning learns $Q^*$ directly without needing a model
- Policy improvement: if $Q^\pi(s, a) > V^\pi(s)$, action $a$ is better than what $\pi$ currently does

---

## 3. Relationship Between V and Q

### From Q to V

$$V^\pi(s) = \sum_{a \in \mathcal{A}} \pi(a \mid s) \cdot Q^\pi(s, a)$$

The state value is the **weighted average** of action values under the policy.

### From V to Q

$$Q^\pi(s, a) = \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma V^\pi(s')\right]$$

The action value equals the expected immediate reward plus discounted value of the next state.

### Combined

$$V^\pi(s) = \sum_a \pi(a \mid s) \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma V^\pi(s')\right]$$

This is the **Bellman expectation equation** (covered in detail in Chapter 7).

### ASCII Backup Diagrams

```
   V^π(s) backup:                    Q^π(s,a) backup:
   
        (s)                              (s)
       / | \                              |
     π/ π|  \π                           (a)    ← action node
     /   |   \                          / | \
   (a₁) (a₂) (a₃)     ← actions      P/ P|  \P
    |     |    |                       /   |   \
    |     |    |                     (s₁) (s₂) (s₃)  ← next states
  Q^π   Q^π  Q^π                      |    |    |
                                    r+γV  r+γV r+γV
  V(s) = Σ_a π(a|s) Q(s,a)
                                    Q(s,a) = Σ_s' P(s'|s,a)[R + γV(s')]
```

### Full Two-Level Backup Diagram

```
         V^π(s)
            │
     ┌──────┼──────┐
     │      │      │        Level 1: Sum over actions (weighted by π)
    (a₁)   (a₂)   (a₃)
     │      │      │
   ┌─┼─┐ ┌─┼─┐ ┌─┼─┐      Level 2: Sum over next states (weighted by P)
   │ │ │ │ │ │ │ │ │
  s' s' s' s' s' s' s' s' s'  ← Each contributes R + γV^π(s')
  
  V^π(s) = Σ_a π(a|s) Σ_s' P(s'|s,a) [R(s,a,s') + γ V^π(s')]
```

---

## 4. Optimal Value Functions

### Optimal State-Value Function

$$V^*(s) = \max_\pi V^\pi(s) \quad \forall s$$

The best possible expected return achievable from state $s$ under any policy.

### Optimal Action-Value Function

$$Q^*(s, a) = \max_\pi Q^\pi(s, a) \quad \forall s, a$$

The best possible expected return from taking action $a$ in state $s$ and then behaving
optimally.

### Relationship

$$V^*(s) = \max_a Q^*(s, a)$$

$$Q^*(s, a) = \sum_{s'} P(s' \mid s, a) \left[R(s, a, s') + \gamma V^*(s')\right]$$

### Extracting the Optimal Policy from $Q^*$

Once we know $Q^*$, the optimal policy is simply:

$$\pi^*(s) = \arg\max_a Q^*(s, a)$$

This is why Q-learning is so powerful: learn $Q^*$, and the optimal policy falls out immediately.

```
   V*(s) backup:                    Q*(s,a) backup:
   
        (s)                              (s)
       / | \                              |
   max/ max\ max                         (a)
     /   |   \                          / | \
   (a₁) (a₂) (a₃)                    P/ P|  \P
    |     |    |                       /   |   \
    ●     ●    ●                     (s₁) (s₂) (s₃)
                                      |    |    |
  V*(s) = max_a Q*(s,a)            r+γV* r+γV* r+γV*
  
                                    Q*(s,a) = Σ_s' P(s'|s,a)[R + γV*(s')]
  
  Note: MAX replaces the weighted sum over π
```

---

## 5. Computing Value Functions: A Small MDP Example

### The MDP

```
States: {s₀, s₁, s₂ (terminal)}
Actions: {left, right}
γ = 0.9

Transitions:
  s₀, right → s₁  (P=1.0, R=+2)
  s₀, left  → s₀  (P=1.0, R=+1)
  s₁, right → s₂  (P=1.0, R=+10)
  s₁, left  → s₀  (P=1.0, R=+0)
  s₂ = terminal (absorbing, R=0)
```

### Policy: $\pi$ = always go right

$$\pi(s_0) = \text{right}, \quad \pi(s_1) = \text{right}$$

### Computing $V^\pi$ by Hand

**For $s_2$ (terminal):** $V^\pi(s_2) = 0$

**For $s_1$:**
$$V^\pi(s_1) = R(s_1, \text{right}, s_2) + \gamma \cdot V^\pi(s_2)$$
$$= 10 + 0.9 \times 0 = 10$$

**For $s_0$:**
$$V^\pi(s_0) = R(s_0, \text{right}, s_1) + \gamma \cdot V^\pi(s_1)$$
$$= 2 + 0.9 \times 10 = 11$$

### Computing $Q^\pi$ by Hand

For $s_0$:
$$Q^\pi(s_0, \text{right}) = 2 + 0.9 \times V^\pi(s_1) = 2 + 9 = 11$$
$$Q^\pi(s_0, \text{left}) = 1 + 0.9 \times V^\pi(s_0) = 1 + 9.9 = 10.9$$

For $s_1$:
$$Q^\pi(s_1, \text{right}) = 10 + 0.9 \times V^\pi(s_2) = 10$$
$$Q^\pi(s_1, \text{left}) = 0 + 0.9 \times V^\pi(s_0) = 9.9$$

### Verification: $V^\pi(s) = \sum_a \pi(a|s) Q^\pi(s,a)$

$$V^\pi(s_0) = 1.0 \times Q^\pi(s_0, \text{right}) = 11 \quad \checkmark$$
$$V^\pi(s_1) = 1.0 \times Q^\pi(s_1, \text{right}) = 10 \quad \checkmark$$

### Is This Policy Optimal?

Check: is there a better action at any state?

At $s_0$: $Q^\pi(s_0, \text{right}) = 11 > Q^\pi(s_0, \text{left}) = 10.9$ → right is better ✓
At $s_1$: $Q^\pi(s_1, \text{right}) = 10 > Q^\pi(s_1, \text{left}) = 9.9$ → right is better ✓

Policy $\pi$ is already optimal! $V^\pi = V^*$.

---

## 6. Python Code: Computing Value Functions

```python
import numpy as np

class ValueFunctionComputer:
    """Compute value functions for a small finite MDP."""

    def __init__(self, states, actions, transitions, rewards, gamma=0.99):
        """
        transitions: dict {(s, a): [(s', prob), ...]}
        rewards: dict {(s, a, s'): reward}
        """
        self.states = states
        self.actions = actions
        self.transitions = transitions
        self.rewards = rewards
        self.gamma = gamma
        self.n_states = len(states)
        self.state_idx = {s: i for i, s in enumerate(states)}

    def compute_V_iterative(self, policy, tol=1e-10, max_iter=10000):
        """Compute V^π using iterative policy evaluation."""
        V = np.zeros(self.n_states)

        for iteration in range(max_iter):
            V_new = np.zeros(self.n_states)
            for s in self.states:
                i = self.state_idx[s]
                value = 0.0
                for a in self.actions:
                    pi_a = policy.get((s, a), 0.0)
                    if pi_a == 0:
                        continue
                    for s_prime, prob in self.transitions.get((s, a), []):
                        j = self.state_idx[s_prime]
                        r = self.rewards.get((s, a, s_prime), 0.0)
                        value += pi_a * prob * (r + self.gamma * V[j])
                V_new[i] = value

            if np.max(np.abs(V_new - V)) < tol:
                print(f"  Converged in {iteration+1} iterations")
                return V_new
            V = V_new

        print("  Warning: did not converge")
        return V

    def compute_Q_from_V(self, V):
        """Compute Q^π(s,a) from V^π."""
        Q = {}
        for s in self.states:
            for a in self.actions:
                q = 0.0
                for s_prime, prob in self.transitions.get((s, a), []):
                    j = self.state_idx[s_prime]
                    r = self.rewards.get((s, a, s_prime), 0.0)
                    q += prob * (r + self.gamma * V[j])
                Q[(s, a)] = q
        return Q

    def compute_V_star(self, tol=1e-10, max_iter=10000):
        """Compute V* using value iteration."""
        V = np.zeros(self.n_states)

        for iteration in range(max_iter):
            V_new = np.zeros(self.n_states)
            for s in self.states:
                i = self.state_idx[s]
                max_val = float('-inf')
                for a in self.actions:
                    q = 0.0
                    transitions = self.transitions.get((s, a), [])
                    if not transitions:
                        continue
                    for s_prime, prob in transitions:
                        j = self.state_idx[s_prime]
                        r = self.rewards.get((s, a, s_prime), 0.0)
                        q += prob * (r + self.gamma * V[j])
                    max_val = max(max_val, q)
                V_new[i] = max_val if max_val > float('-inf') else 0.0

            if np.max(np.abs(V_new - V)) < tol:
                print(f"  V* converged in {iteration+1} iterations")
                return V_new
            V = V_new

        return V

    def extract_optimal_policy(self, V):
        """Extract greedy policy from value function."""
        policy = {}
        for s in self.states:
            best_a, best_q = None, float('-inf')
            for a in self.actions:
                q = 0.0
                for s_prime, prob in self.transitions.get((s, a), []):
                    j = self.state_idx[s_prime]
                    r = self.rewards.get((s, a, s_prime), 0.0)
                    q += prob * (r + self.gamma * V[j])
                if q > best_q:
                    best_q = q
                    best_a = a
            policy[s] = best_a
        return policy


# --- Build the small MDP ---
states = ["s0", "s1", "s2"]
actions = ["left", "right"]

transitions = {
    ("s0", "right"): [("s1", 1.0)],
    ("s0", "left"):  [("s0", 1.0)],
    ("s1", "right"): [("s2", 1.0)],
    ("s1", "left"):  [("s0", 1.0)],
    # s2 is terminal — no transitions
}

rewards = {
    ("s0", "right", "s1"): 2.0,
    ("s0", "left", "s0"):  1.0,
    ("s1", "right", "s2"): 10.0,
    ("s1", "left", "s0"):  0.0,
}

computer = ValueFunctionComputer(states, actions, transitions, rewards, gamma=0.9)

# --- Compute V^π for "always right" policy ---
print("=== V^π (policy: always right) ===")
policy_right = {("s0", "right"): 1.0, ("s1", "right"): 1.0}
V_pi = computer.compute_V_iterative(policy_right)
for s in states:
    i = computer.state_idx[s]
    print(f"  V^π({s}) = {V_pi[i]:.4f}")

# --- Compute Q^π ---
print("\n=== Q^π ===")
Q_pi = computer.compute_Q_from_V(V_pi)
for s in ["s0", "s1"]:
    for a in actions:
        print(f"  Q^π({s}, {a}) = {Q_pi[(s, a)]:.4f}")

# --- Verify V = Σ π(a|s) Q(s,a) ---
print("\n=== Verification: V = Σ π(a|s) Q(s,a) ===")
for s in ["s0", "s1"]:
    i = computer.state_idx[s]
    v_from_q = sum(policy_right.get((s, a), 0) * Q_pi[(s, a)] for a in actions)
    print(f"  V^π({s}) = {V_pi[i]:.4f}, Σ π Q = {v_from_q:.4f} ✓")

# --- Compute V* and optimal policy ---
print("\n=== V* (optimal value function) ===")
V_star = computer.compute_V_star()
for s in states:
    i = computer.state_idx[s]
    print(f"  V*({s}) = {V_star[i]:.4f}")

print("\n=== Optimal Policy ===")
opt_policy = computer.extract_optimal_policy(V_star)
for s in ["s0", "s1"]:
    print(f"  π*({s}) = {opt_policy[s]}")

# --- Q* ---
print("\n=== Q* ===")
Q_star = computer.compute_Q_from_V(V_star)
for s in ["s0", "s1"]:
    for a in actions:
        marker = " ← optimal" if a == opt_policy[s] else ""
        print(f"  Q*({s}, {a}) = {Q_star[(s, a)]:.4f}{marker}")
```

---

## 7. Value Function Visualization

```
Value Function Landscape for a 4×4 Grid World:

  V^π values:           Q^π arrows (best action):
  
  ┌──────┬──────┬──────┬──────┐    ┌──────┬──────┬──────┬──────┐
  │  5.2 │  7.1 │  8.9 │ 10.0 │    │  →   │  →   │  →   │ GOAL │
  ├──────┼──────┼──────┼──────┤    ├──────┼──────┼──────┼──────┤
  │  3.8 │  0.0 │  6.5 │ -5.0 │    │  ↑   │WALL  │  ↑   │TRAP  │
  ├──────┼──────┼──────┼──────┤    ├──────┼──────┼──────┼──────┤
  │  2.9 │  3.5 │  4.8 │  3.2 │    │  ↑   │  →   │  ↑   │  ←   │
  ├──────┼──────┼──────┼──────┤    ├──────┼──────┼──────┼──────┤
  │  1.5 │  2.2 │  3.0 │  2.1 │    │  ↑   │  →   │  →   │  ↑   │
  └──────┴──────┴──────┴──────┘    └──────┴──────┴──────┴──────┘
  
  Values increase toward the goal (0,3).
  The optimal policy arrows follow the value gradient.
```

---

## 8. Real-World Applications

| Domain | Value Function Use | Key Insight |
|---|---|---|
| **Game AI** | Evaluate board positions | Higher V(s) = stronger position |
| **Robotics** | Value of robot configurations | Guides motion planning |
| **Finance** | Value of portfolio states | Expected future returns |
| **Healthcare** | Value of patient health states | Guides treatment decisions |
| **Recommendation** | Value of user engagement state | Predicts long-term satisfaction |

---

## Summary

| Concept | Key Point |
|---|---|
| $V^\pi(s)$ | Expected return from state $s$ under policy $\pi$ |
| $Q^\pi(s,a)$ | Expected return from $(s,a)$ then following $\pi$ |
| V–Q relationship | $V^\pi(s) = \sum_a \pi(a \mid s) Q^\pi(s,a)$ |
| Q–V relationship | $Q^\pi(s,a) = \sum_{s'} P(s' \mid s,a)[R + \gamma V^\pi(s')]$ |
| $V^*(s)$ | $\max_\pi V^\pi(s)$ — best achievable value |
| $Q^*(s,a)$ | $\max_\pi Q^\pi(s,a)$ — best value for action $a$ in state $s$ |
| $V^*$ to $\pi^*$ | $\pi^*(s) = \arg\max_a Q^*(s,a)$ |
| Backup diagram | Tree showing how value at one level depends on values at the next |

---

## Revision Questions

1. **Define $V^\pi(s)$ and $Q^\pi(s,a)$ in words and mathematically.** What question does each answer?

2. **Derive $V^\pi(s)$ from $Q^\pi(s,a)$** and vice versa. Show the formulas and explain each term.

3. **For the 3-state MDP in this chapter**, compute $V^\pi$ for the "always left" policy. Compare it with the "always right" policy.

4. **Draw a backup diagram** for $V^\pi(s)$ showing both the action level and the next-state level. Label all edges.

5. **Explain why $Q^*$ is sufficient to derive $\pi^*$** without knowing the transition model. Why is this important?

6. **If $V^*(s_0) = 15$ and $Q^*(s_0, a_1) = 15$, $Q^*(s_0, a_2) = 12$**, what is $\pi^*(s_0)$? What if $Q^*(s_0, a_3) = 15$ as well?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Policies](./05-policies.md) | [Unit 2: MDPs](./README.md) | [Bellman Equations](./07-bellman-equations.md) |

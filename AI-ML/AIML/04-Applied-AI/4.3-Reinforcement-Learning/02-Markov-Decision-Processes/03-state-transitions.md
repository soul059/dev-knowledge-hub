# Chapter 3: State Transitions

> **Unit 2: Markov Decision Processes** — *How the environment evolves in response to the agent's actions.*

---

## Overview

The transition function $P(s' \mid s, a)$ is the engine of an MDP — it encodes the
environment's dynamics. This chapter explores transition probabilities in depth: how they
are represented, the difference between deterministic and stochastic transitions, how to
compute multi-step distributions, and how to work with transition matrices numerically.

---

## 1. The Transition Probability Function

### Formal Definition

The **state transition probability function** maps a current state and action to a
probability distribution over next states:

$$P(s' \mid s, a) = P(S_{t+1} = s' \mid S_t = s, A_t = a)$$

### Properties

For all states $s \in \mathcal{S}$ and actions $a \in \mathcal{A}$:

1. **Non-negativity**: $P(s' \mid s, a) \geq 0 \quad \forall s'$
2. **Normalization**: $\sum_{s' \in \mathcal{S}} P(s' \mid s, a) = 1$

### State Transition Function Variants

The transition function can also return the **expected next state** along with
the reward, combining both dynamics aspects:

$$p(s', r \mid s, a) = P(S_{t+1} = s', R_{t+1} = r \mid S_t = s, A_t = a)$$

From this joint distribution, we can derive:
- $P(s' \mid s, a) = \sum_r p(s', r \mid s, a)$
- $R(s, a) = \sum_{s'} \sum_r r \cdot p(s', r \mid s, a)$

---

## 2. Transition Matrix Representation

For a finite MDP with $|\mathcal{S}| = n$ states, each action $a$ defines an
$n \times n$ **transition matrix** $P^a$:

$$(P^a)_{ij} = P(S_{t+1} = s_j \mid S_t = s_i, A_t = a)$$

### Example: 3-State MDP with 2 Actions

```
Action = "left"                    Action = "right"
                                   
   P^left:                           P^right:
   ┌                    ┐            ┌                    ┐
   │ 0.9   0.1   0.0    │            │ 0.1   0.8   0.1    │
   │ 0.5   0.3   0.2    │            │ 0.0   0.2   0.8    │
   │ 0.0   0.6   0.4    │            │ 0.0   0.0   1.0    │
   └                    ┘            └                    ┘
     s0    s1    s2                     s0    s1    s2
```

Each row sums to 1.0. Reading row $i$, column $j$ gives $P(s_j \mid s_i, a)$.

### Full Transition Tensor

For an MDP, the transitions form a **3D tensor** of shape $|\mathcal{S}| \times |\mathcal{A}| \times |\mathcal{S}|$:

$$\mathcal{P}[s, a, s'] = P(s' \mid s, a)$$

```
          s'=s0   s'=s1   s'=s2
         ┌───────────────────────┐
a=left   │  0.9    0.1    0.0    │  ← from s0
         │  0.5    0.3    0.2    │  ← from s1
         │  0.0    0.6    0.4    │  ← from s2
         ├───────────────────────┤
a=right  │  0.1    0.8    0.1    │  ← from s0
         │  0.0    0.2    0.8    │  ← from s1
         │  0.0    0.0    1.0    │  ← from s2
         └───────────────────────┘
```

---

## 3. Deterministic vs Stochastic Transitions

### Deterministic Transitions

In a **deterministic** MDP, each state-action pair maps to exactly one next state:

$$P(s' \mid s, a) = \begin{cases} 1 & \text{if } s' = f(s, a) \\ 0 & \text{otherwise} \end{cases}$$

where $f: \mathcal{S} \times \mathcal{A} \to \mathcal{S}$ is a deterministic function.

```
Deterministic Grid World:

  Action = RIGHT from (1,0):
  
  ┌─────┬─────┬─────┐
  │     │     │     │
  │(0,0)│(0,1)│(0,2)│
  ├─────┼─────┼─────┤     P((1,1) | (1,0), RIGHT) = 1.0
  │  ●══╪══▶  │     │     P(other | (1,0), RIGHT) = 0.0
  │(1,0)│(1,1)│(1,2)│
  ├─────┼─────┼─────┤
  │     │     │     │
  │(2,0)│(2,1)│(2,2)│
  └─────┴─────┴─────┘
```

### Stochastic Transitions

In a **stochastic** MDP, actions lead to probability distributions over next states:

```
Stochastic Grid World ("Slippery Floor"):

  Action = RIGHT from (1,0):
  
  ┌─────┬─────┬─────┐
  │  ▲  │     │     │
  │10%  │     │     │
  ├──│──┼─────┼─────┤     P((1,1) | (1,0), RIGHT) = 0.8
  │  ●══╪══▶  │     │     P((0,0) | (1,0), RIGHT) = 0.1
  │  │  │ 80% │     │     P((2,0) | (1,0), RIGHT) = 0.1
  ├──│──┼─────┼─────┤
  │  ▼  │     │     │
  │ 10% │     │     │
  └─────┴─────┴─────┘
  
  Intended direction: 80%
  Perpendicular slip:  10% each
```

### Comparison

| Property | Deterministic | Stochastic |
|---|---|---|
| Predictability | Fully predictable | Uncertain outcomes |
| Transition matrix | Binary (0s and 1s) | Continuous probabilities |
| Planning | Simpler (no uncertainty) | Must consider all outcomes |
| Real-world fidelity | Idealized | More realistic |
| Examples | Chess, Go | Robot navigation, weather |

---

## 4. Computing Next-State Distributions

### Single Step

Given a state $s$ and action $a$, the next-state distribution is simply the row
of $P^a$ corresponding to $s$:

$$\text{dist}(S_{t+1}) = P^a[s, :] = [P(s_0 \mid s, a), P(s_1 \mid s, a), \ldots, P(s_n \mid s, a)]$$

### From a State Distribution

If the agent is in a **distribution** over states $\mu_t$ (a row vector), then after
taking action $a$:

$$\mu_{t+1} = \mu_t \cdot P^a$$

### Multi-Step Transitions Under a Fixed Policy

Under a deterministic policy $\pi$, the effective transition matrix is:

$$P^\pi[s, s'] = P(s' \mid s, \pi(s))$$

The $k$-step distribution starting from $\mu_0$:

$$\mu_k = \mu_0 \cdot (P^\pi)^k$$

### Numerical Example

Given:
$$P^{\text{right}} = \begin{bmatrix} 0.1 & 0.8 & 0.1 \\ 0.0 & 0.2 & 0.8 \\ 0.0 & 0.0 & 1.0 \end{bmatrix}$$

Starting in $s_0$ (i.e., $\mu_0 = [1, 0, 0]$), always taking action "right":

**Step 1**: $\mu_1 = [1, 0, 0] \cdot P^{\text{right}} = [0.1, 0.8, 0.1]$

**Step 2**: $\mu_2 = [0.1, 0.8, 0.1] \cdot P^{\text{right}}$
$$= [0.1 \times 0.1, \; 0.1 \times 0.8 + 0.8 \times 0.2, \; 0.1 \times 0.1 + 0.8 \times 0.8 + 0.1 \times 1.0]$$
$$= [0.01, \; 0.24, \; 0.75]$$

```
Distribution evolution:

Step 0:  s0 [████████████████████] 100%    s1 [] 0%     s2 [] 0%
Step 1:  s0 [██] 10%    s1 [████████████████] 80%       s2 [██] 10%
Step 2:  s0 [▏] 1%      s1 [█████] 24%      s2 [███████████████] 75%
Step 3:  s0 [▏] ~0%     s1 [█] ~7%          s2 [██████████████████] ~93%
```

---

## 5. Absorbing States and Terminal Transitions

An **absorbing state** has $P(s \mid s, a) = 1$ for all actions — once entered,
the agent stays there forever.

Terminal states in episodic MDPs are typically modeled as absorbing states with zero reward:

$$P(s_T \mid s_T, a) = 1, \quad R(s_T, a, s_T) = 0 \quad \forall a$$

```
Absorbing state example:

  s0 ──0.5──▶ s1 ──0.8──▶ s2 (absorbing)
  │            │           │
  └──0.5──┐   └──0.2──┐   └──1.0──┐
           ▼            ▼          ▼
          s0           s1         s2
          
  Once in s2, the agent never leaves.
```

---

## 6. Python Code: Transition Matrices

```python
import numpy as np

class TransitionModel:
    """Represents the transition dynamics of a finite MDP."""

    def __init__(self, n_states: int, n_actions: int, state_names=None):
        self.n_states = n_states
        self.n_actions = n_actions
        self.state_names = state_names or [f"s{i}" for i in range(n_states)]
        # 3D tensor: P[action, state, next_state]
        self.P = np.zeros((n_actions, n_states, n_states))

    def set_transition(self, s, a, next_state_probs):
        """Set transition probabilities for (state, action) pair."""
        probs = np.array(next_state_probs)
        assert len(probs) == self.n_states
        assert np.isclose(probs.sum(), 1.0), f"Probabilities must sum to 1, got {probs.sum()}"
        self.P[a, s, :] = probs

    def get_distribution(self, s, a):
        """Get next-state distribution for (state, action)."""
        return self.P[a, s, :]

    def sample_next_state(self, s, a):
        """Sample a next state according to P(s'|s,a)."""
        probs = self.P[a, s, :]
        return np.random.choice(self.n_states, p=probs)

    def multi_step_distribution(self, s, a, k):
        """Compute k-step state distribution starting from s, always taking action a."""
        P_a = self.P[a]  # Transition matrix for action a
        mu = np.zeros(self.n_states)
        mu[s] = 1.0
        for _ in range(k):
            mu = mu @ P_a
        return mu

    def policy_transition_matrix(self, policy):
        """
        Compute effective transition matrix under a deterministic policy.
        policy: array of shape (n_states,) mapping state -> action
        """
        P_pi = np.zeros((self.n_states, self.n_states))
        for s in range(self.n_states):
            a = policy[s]
            P_pi[s, :] = self.P[a, s, :]
        return P_pi

    def is_absorbing(self, s):
        """Check if state s is absorbing (self-loop for all actions)."""
        for a in range(self.n_actions):
            if not np.isclose(self.P[a, s, s], 1.0):
                return False
        return True

    def validate(self):
        """Verify all rows sum to 1."""
        for a in range(self.n_actions):
            row_sums = self.P[a].sum(axis=1)
            assert np.allclose(row_sums, 1.0), \
                f"Action {a}: row sums = {row_sums}"
        print("✓ All transition probabilities valid")


# --- Build the 3-state example ---
model = TransitionModel(n_states=3, n_actions=2, state_names=["s0", "s1", "s2"])

# Action 0 = left
model.set_transition(s=0, a=0, next_state_probs=[0.9, 0.1, 0.0])
model.set_transition(s=1, a=0, next_state_probs=[0.5, 0.3, 0.2])
model.set_transition(s=2, a=0, next_state_probs=[0.0, 0.6, 0.4])

# Action 1 = right
model.set_transition(s=0, a=1, next_state_probs=[0.1, 0.8, 0.1])
model.set_transition(s=1, a=1, next_state_probs=[0.0, 0.2, 0.8])
model.set_transition(s=2, a=1, next_state_probs=[0.0, 0.0, 1.0])

model.validate()

# --- Demonstrate transitions ---
print("\n=== Next-State Distributions ===")
for s in range(3):
    for a, a_name in enumerate(["left", "right"]):
        dist = model.get_distribution(s, a)
        dist_str = ", ".join(f"{model.state_names[i]}={dist[i]:.1f}" for i in range(3))
        print(f"  P(·|{model.state_names[s]}, {a_name}) = [{dist_str}]")

# --- Multi-step distribution ---
print("\n=== Multi-Step Distribution (start=s0, action=right) ===")
for k in [1, 2, 3, 5, 10]:
    dist = model.multi_step_distribution(s=0, a=1, k=k)
    dist_str = ", ".join(f"{dist[i]:.4f}" for i in range(3))
    print(f"  After {k:2d} steps: [{dist_str}]")

# --- Policy transition matrix ---
print("\n=== Policy Transition Matrix ===")
policy = np.array([1, 1, 0])  # right, right, left
P_pi = model.policy_transition_matrix(policy)
print(f"  Policy: s0→right, s1→right, s2→left")
print(f"  P^π =\n{P_pi}")

# --- Simulate trajectories ---
print("\n=== Simulated Trajectories (10 episodes, start=s0, action=right) ===")
np.random.seed(42)
for ep in range(5):
    trajectory = [0]
    s = 0
    for _ in range(8):
        s = model.sample_next_state(s, a=1)
        trajectory.append(s)
        if model.is_absorbing(s):
            break
    traj_str = " → ".join(model.state_names[s] for s in trajectory)
    print(f"  Episode {ep+1}: {traj_str}")

# --- Check absorbing states ---
print("\n=== Absorbing States ===")
for s in range(3):
    if model.is_absorbing(s):
        print(f"  {model.state_names[s]} is absorbing")
```

---

## 7. Transition Diagrams

### State-Transition Diagram

A common way to visualize transitions is a **directed graph** where edges are labeled
with action and probability:

```
            left, 0.9               right, 0.8
           ┌───────┐              ┌───────────────┐
           │       │              │               │
           ▼       │              ▼               │
        ┌──────┐   │           ┌──────┐        ┌──────┐
   ┌───▶│  s0  │───┘    ┌────▶│  s1  │───────▶│  s2  │◀──┐
   │    └──────┘        │     └──────┘ r:0.8   └──────┘   │
   │       │            │        │                 │      │
   │       │ r:0.1      │        │ l:0.2           │      │
   │       └────────────┘        └─────────────────┘      │
   │     right, 0.1          left, 0.6 / right, 0.8       │
   │                                                      │
   └──────────── left, 0.5 ──────────────────────────────┘
                 (from s1)              right, 1.0 (self)
                                        left, 0.4 (self)
```

---

## 8. Real-World Applications

| Domain | Transition Type | Key Challenge |
|---|---|---|
| **Robotics** | Stochastic (motor noise, friction) | Accurate physics modeling |
| **Board games** | Deterministic (or with dice) | Large state space |
| **Network routing** | Stochastic (packet loss, congestion) | Time-varying dynamics |
| **Inventory mgmt** | Stochastic (random demand) | Demand distribution estimation |
| **Clinical trials** | Stochastic (patient response) | Ethical constraints on exploration |

---

## Summary

| Concept | Key Point |
|---|---|
| Transition function | $P(s' \mid s, a)$ — probability of next state given current state and action |
| Transition matrix | $n \times n$ stochastic matrix for each action |
| Transition tensor | $|\mathcal{S}| \times |\mathcal{A}| \times |\mathcal{S}|$ tensor |
| Deterministic | $P(s' \mid s,a) \in \{0, 1\}$, one successor per (s,a) |
| Stochastic | Multiple possible next states with associated probabilities |
| Multi-step | $\mu_{t+k} = \mu_t (P^a)^k$ for repeated action $a$ |
| Policy matrix | $P^\pi[s,s'] = P(s' \mid s, \pi(s))$ |
| Absorbing state | Self-transition with probability 1 for all actions |

---

## Revision Questions

1. **State the two properties** that any valid transition probability function must satisfy.

2. **Given $P^{\text{right}} = \begin{bmatrix} 0.1 & 0.8 & 0.1 \\ 0.0 & 0.2 & 0.8 \\ 0.0 & 0.0 & 1.0 \end{bmatrix}$**, compute the 2-step transition probability from $s_0$ to $s_2$ when always taking action "right".

3. **What is the difference between a transition matrix and a transition tensor?** When do you need each?

4. **Explain why stochastic transitions are more common** in real-world applications than deterministic ones. Give two examples.

5. **What is an absorbing state?** How are terminal states in episodic MDPs typically modeled using absorbing states?

6. **Given the policy** $\pi(s_0) = \text{right}, \pi(s_1) = \text{left}, \pi(s_2) = \text{right}$, construct the policy transition matrix $P^\pi$ from the transition matrices in this chapter.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [MDP Components](./02-mdp-components.md) | [Unit 2: MDPs](./README.md) | [Reward Function](./04-reward-function.md) |

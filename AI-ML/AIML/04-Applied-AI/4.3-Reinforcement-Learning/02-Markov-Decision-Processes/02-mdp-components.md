# Chapter 2: MDP Components

> **Unit 2: Markov Decision Processes** — *The formal framework that defines every reinforcement learning problem.*

---

## Overview

A Markov Decision Process (MDP) extends a Markov chain by adding **actions** and **rewards**,
creating the mathematical framework for sequential decision-making under uncertainty. Every RL
problem can be formalized as an MDP. This chapter defines each component of the MDP tuple,
traces the evolution from Markov Chains to Markov Reward Processes to full MDPs, and builds
a complete example using a grid world.

---

## 1. Formal Definition: The MDP Tuple

An MDP is defined by the 5-tuple:

$$\mathcal{M} = (\mathcal{S}, \mathcal{A}, P, R, \gamma)$$

| Symbol | Name | Description |
|---|---|---|
| $\mathcal{S}$ | State space | Set of all possible states |
| $\mathcal{A}$ | Action space | Set of all possible actions |
| $P$ | Transition function | $P(s' \mid s, a)$ — probability of next state given current state and action |
| $R$ | Reward function | $R(s, a, s')$ — immediate reward for a transition |
| $\gamma$ | Discount factor | $\gamma \in [0, 1]$ — how much to value future rewards |

### The MDP Interaction Loop

```
        ┌─────────────────────────────────────────┐
        │              ENVIRONMENT                │
        │                                         │
        │   State s_t ──────┐                     │
        │   Reward r_t ─────┤                     │
        │                   ▼                     │
        └──────────────── Agent ──────────────────┘
                            │
                       Action a_t
                            │
                            ▼
        ┌─────────────────────────────────────────┐
        │              ENVIRONMENT                │
        │                                         │
        │   P(s_{t+1} | s_t, a_t) → s_{t+1}      │
        │   R(s_t, a_t, s_{t+1})  → r_{t+1}      │
        │                                         │
        └─────────────────────────────────────────┘
```

---

## 2. Each Component in Detail

### 2.1 State Space $\mathcal{S}$

The state space is the set of all possible configurations the environment can be in.

**Finite state space**: $\mathcal{S} = \{s_1, s_2, \ldots, s_n\}$
- Grid world positions, board game configurations
- $|\mathcal{S}|$ is finite and countable

**Infinite/continuous state space**: $\mathcal{S} \subseteq \mathbb{R}^d$
- Robot joint angles, car velocity
- Requires function approximation for practical use

**Terminal states**: Special states $s_T \in \mathcal{S}^+$ where episodes end.
The augmented state space $\mathcal{S}^+ = \mathcal{S} \cup \{s_T\}$ includes terminal states.

### 2.2 Action Space $\mathcal{A}$

The set of all actions available to the agent.

**Discrete actions**: $\mathcal{A} = \{a_1, a_2, \ldots, a_m\}$
- Move up/down/left/right, buy/sell/hold

**Continuous actions**: $\mathcal{A} \subseteq \mathbb{R}^k$
- Torque applied to joints, steering angle

**State-dependent actions**: $\mathcal{A}(s) \subseteq \mathcal{A}$
- Not all actions may be available in every state
- Example: Can't move left at the left wall

### 2.3 Transition Function $P(s' \mid s, a)$

The **dynamics model** of the environment:

$$P(s' \mid s, a) = P(S_{t+1} = s' \mid S_t = s, A_t = a)$$

Properties:
- $P(s' \mid s, a) \geq 0$ for all $s', s, a$
- $\sum_{s' \in \mathcal{S}} P(s' \mid s, a) = 1$ for all $s, a$

This encodes the Markov property — the next state depends only on the current state and action.

### 2.4 Reward Function $R(s, a, s')$

The immediate scalar feedback signal:

$$R(s, a, s') = \mathbb{E}[R_{t+1} \mid S_t = s, A_t = a, S_{t+1} = s']$$

Common simplifications:
- $R(s, a)$: reward depends on state and action only
- $R(s)$: reward depends on state only

The reward is always a **scalar** — this is the **reward hypothesis**:
> All goals can be described by the maximization of the expected cumulative scalar reward.

### 2.5 Discount Factor $\gamma$

The discount factor $\gamma \in [0, 1]$ controls the trade-off between immediate and
future rewards in the **return**:

$$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$$

| Value | Meaning |
|---|---|
| $\gamma = 0$ | Completely myopic — only cares about immediate reward |
| $\gamma = 1$ | No discounting — values all future rewards equally |
| $\gamma = 0.99$ | Far-sighted — strongly values future |
| $\gamma = 0.9$ | Moderate discounting |

**Why discount?**
1. Mathematical convenience — ensures $G_t$ is finite for bounded rewards
2. Models uncertainty about the future
3. Reflects preference for sooner rewards (time value)
4. For $|R| \leq R_{max}$: $|G_t| \leq \frac{R_{max}}{1 - \gamma}$

---

## 3. Finite vs Infinite MDPs

| Property | Finite MDP | Infinite/Continuous MDP |
|---|---|---|
| State space | $|\mathcal{S}| < \infty$ | $\mathcal{S} \subseteq \mathbb{R}^d$ |
| Action space | $|\mathcal{A}| < \infty$ | $\mathcal{A} \subseteq \mathbb{R}^k$ |
| Transition function | Matrix/table | Probability density function |
| Solution methods | Exact (DP, linear algebra) | Approximate (function approximation) |
| Examples | Grid world, board games | Robotics, continuous control |

---

## 4. Evolution: Markov Chain → MRP → MDP

### 4.1 Markov Chain (MC)

$$\text{MC} = (\mathcal{S}, P)$$

- States and transitions only
- No actions, no rewards
- Models: weather patterns, random walks

### 4.2 Markov Reward Process (MRP)

$$\text{MRP} = (\mathcal{S}, P, R, \gamma)$$

- Adds rewards and discounting
- Still no actions (agent has no control)
- Models: stock price with dividends, natural processes

### 4.3 Markov Decision Process (MDP)

$$\text{MDP} = (\mathcal{S}, \mathcal{A}, P, R, \gamma)$$

- Adds actions — the agent can influence transitions
- Full sequential decision-making framework

```
Markov Chain          Markov Reward Process       MDP
(S, P)                (S, P, R, γ)                (S, A, P, R, γ)
                                                 
  s₁ ──▶ s₂           s₁ ──▶ s₂                   s₁ ─a₁─▶ s₂
  │       │            │  +5   │  -1               │  a₂     │
  ▼       ▼            ▼       ▼                   ▼  ─────▶ ▼
  s₃ ──▶ s₄           s₃ ──▶ s₄                   s₃ ─a₁─▶ s₄
                       +2      +10                   └─a₂─▶ s₁
  No rewards          Rewards, no control          Rewards + control
```

---

## 5. Grid World: A Complete MDP Example

### The 4×4 Grid World

```
┌─────┬─────┬─────┬─────┐
│  S  │     │     │  G  │
│(0,0)│(0,1)│(0,2)│(0,3)│
├─────┼─────┼─────┼─────┤
│     │ XXX │     │  T  │
│(1,0)│WALL │(1,2)│TRAP │
├─────┼─────┼─────┼─────┤
│     │     │     │     │
│(2,0)│(2,1)│(2,2)│(2,3)│
├─────┼─────┼─────┼─────┤
│     │     │     │     │
│(3,0)│(3,1)│(3,2)│(3,3)│
└─────┴─────┴─────┴─────┘

S = Start,  G = Goal (+10),  T = Trap (-10),  XXX = Wall
```

### MDP Definition for This Grid World

| Component | Specification |
|---|---|
| $\mathcal{S}$ | $\{(r,c) : r \in \{0..3\}, c \in \{0..3\}\} \setminus \{(1,1)\}$ — 15 states (wall excluded) |
| $\mathcal{A}$ | $\{\text{UP}, \text{DOWN}, \text{LEFT}, \text{RIGHT}\}$ — 4 actions |
| $P(s' \mid s, a)$ | 0.8 intended direction, 0.1 each perpendicular; wall = stay |
| $R(s, a, s')$ | $+10$ if $s' = G$, $-10$ if $s' = T$, $-0.1$ otherwise (step penalty) |
| $\gamma$ | $0.95$ |
| Terminal | $G = (0,3)$, $T = (1,3)$ |

---

## 6. Python Code: Defining an MDP Class

```python
import numpy as np
from typing import Dict, List, Tuple, Optional

class MDP:
    """A finite Markov Decision Process."""

    def __init__(
        self,
        states: List[str],
        actions: List[str],
        transition_probs: Dict[Tuple[str, str, str], float],
        rewards: Dict[Tuple[str, str, str], float],
        gamma: float = 0.99,
        terminal_states: Optional[List[str]] = None
    ):
        """
        Args:
            states: List of state names
            actions: List of action names
            transition_probs: {(s, a, s'): probability}
            rewards: {(s, a, s'): reward}
            gamma: Discount factor
            terminal_states: States where episodes end
        """
        self.states = states
        self.actions = actions
        self.transition_probs = transition_probs
        self.rewards = rewards
        self.gamma = gamma
        self.terminal_states = set(terminal_states or [])
        self.n_states = len(states)
        self.n_actions = len(actions)

    def get_transitions(self, state: str, action: str) -> List[Tuple[str, float, float]]:
        """Return list of (next_state, probability, reward) for (state, action)."""
        transitions = []
        for s_prime in self.states:
            key = (state, action, s_prime)
            prob = self.transition_probs.get(key, 0.0)
            if prob > 0:
                reward = self.rewards.get(key, 0.0)
                transitions.append((s_prime, prob, reward))
        return transitions

    def is_terminal(self, state: str) -> bool:
        return state in self.terminal_states

    def expected_reward(self, state: str, action: str) -> float:
        """Compute E[R | s, a] = sum over s' of P(s'|s,a) * R(s,a,s')."""
        transitions = self.get_transitions(state, action)
        return sum(prob * reward for _, prob, reward in transitions)

    def __repr__(self):
        return (f"MDP(states={self.n_states}, actions={self.n_actions}, "
                f"gamma={self.gamma})")


# --- Build a simple 3-state MDP ---
states = ["s0", "s1", "s2"]
actions = ["left", "right"]

# Transition probabilities: (state, action, next_state) -> probability
T = {
    ("s0", "right", "s0"): 0.1,  ("s0", "right", "s1"): 0.9,
    ("s0", "left",  "s0"): 1.0,
    ("s1", "right", "s1"): 0.2,  ("s1", "right", "s2"): 0.8,
    ("s1", "left",  "s1"): 0.3,  ("s1", "left",  "s0"): 0.7,
    ("s2", "right", "s2"): 1.0,
    ("s2", "left",  "s2"): 0.4,  ("s2", "left",  "s1"): 0.6,
}

# Rewards: (state, action, next_state) -> reward
R = {
    ("s0", "right", "s1"): 0.0,
    ("s1", "right", "s2"): 10.0,   # Reaching goal
    ("s1", "left",  "s0"): -1.0,
    ("s2", "left",  "s1"): -1.0,
}

mdp = MDP(states, actions, T, R, gamma=0.95, terminal_states=["s2"])
print(mdp)

# Examine transitions
for s in states:
    for a in actions:
        transitions = mdp.get_transitions(s, a)
        if transitions:
            print(f"\n  {s}, {a}:")
            for s_prime, prob, reward in transitions:
                print(f"    → {s_prime}: P={prob:.1f}, R={reward:.1f}")

# Expected rewards
print("\n=== Expected Rewards ===")
for s in states:
    for a in actions:
        er = mdp.expected_reward(s, a)
        if er != 0:
            print(f"  E[R | {s}, {a}] = {er:.2f}")


# --- Grid World MDP using Gymnasium ---
print("\n\n=== Grid World with Gymnasium ===")
try:
    import gymnasium as gym

    # FrozenLake is a classic grid world MDP
    env = gym.make("FrozenLake-v1", is_slippery=True)

    print(f"State space:  {env.observation_space}")   # Discrete(16)
    print(f"Action space: {env.action_space}")         # Discrete(4)
    print(f"Actions: 0=Left, 1=Down, 2=Right, 3=Up")

    # Examine transition dynamics for state 0, action Right(2)
    state, action = 0, 2
    print(f"\nTransitions from state {state}, action Right:")
    for prob, next_state, reward, done in env.unwrapped.P[state][action]:
        print(f"  → state {next_state}: P={prob:.4f}, R={reward}, done={done}")

    env.close()
except ImportError:
    print("Gymnasium not installed. Install with: pip install gymnasium")
```

---

## 7. Real-World Applications

| Application | States | Actions | Rewards | $\gamma$ |
|---|---|---|---|---|
| **Robot navigation** | $(x, y, \theta)$ | velocity commands | +1 goal, -1 collision | 0.99 |
| **Game playing** | board/screen | game moves | score change | 0.99 |
| **Portfolio mgmt** | asset weights | buy/sell/hold | portfolio return | 0.95 |
| **HVAC control** | temp, humidity | heat/cool/off | -energy + comfort | 0.99 |
| **Treatment planning** | patient state | drug dosages | health outcome | 0.98 |

---

## Summary

| Concept | Key Point |
|---|---|
| MDP tuple | $\mathcal{M} = (\mathcal{S}, \mathcal{A}, P, R, \gamma)$ |
| State space $\mathcal{S}$ | All possible environment configurations |
| Action space $\mathcal{A}$ | All possible agent decisions |
| Transition $P(s' \mid s,a)$ | Stochastic dynamics of the environment |
| Reward $R(s,a,s')$ | Scalar feedback signal |
| Discount $\gamma$ | Trade-off: immediate vs future rewards |
| Return $G_t$ | $\sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$ |
| MC → MRP → MDP | Progressive addition of rewards, then actions |

---

## Revision Questions

1. **Write out the full MDP tuple** and explain each component in one sentence.

2. **What is the difference between $R(s,a,s')$, $R(s,a)$, and $R(s)$?** When might you use each form?

3. **If $\gamma = 0.9$ and $R_{max} = 1$**, what is the maximum possible return $G_t$?

4. **Explain the progression from Markov Chain → MRP → MDP.** What does each addition (rewards, actions) enable?

5. **In the grid world example**, if the agent is at $(2,2)$ and takes action UP with 0.8 probability of success and 0.1 probability of slipping to each side, what are all possible next states and their probabilities?

6. **Why is the discount factor important** even in episodic tasks that always terminate? Give two reasons.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Markov Property](./01-markov-property.md) | [Unit 2: MDPs](./README.md) | [State Transitions](./03-state-transitions.md) |

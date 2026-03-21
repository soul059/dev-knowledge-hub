# Chapter 3: Agent, Environment, State, Action, Reward

> **Unit 1 · Introduction to RL** — File 3 of 5

---

## 1. Overview

Every RL problem is defined by the interaction between two entities: an **agent** and an
**environment**. At each time step the agent observes a **state**, selects an **action**,
and receives a **reward**. This chapter formalises each component and shows how they
connect through the **interaction loop**.

```
 ╔════════════════════════════════════════════════════════════════════╗
 ║               THE FIVE CORE RL COMPONENTS                        ║
 ╠════════════════════════════════════════════════════════════════════╣
 ║                                                                   ║
 ║   Agent  ──  the learner / decision-maker                        ║
 ║   Environment  ──  everything the agent interacts with           ║
 ║   State (sₜ)  ──  a snapshot of the world at time t              ║
 ║   Action (aₜ)  ──  what the agent does at time t                 ║
 ║   Reward (rₜ₊₁)  ──  scalar feedback after taking aₜ in sₜ      ║
 ║                                                                   ║
 ╚════════════════════════════════════════════════════════════════════╝
```

---

## 2. The Agent-Environment Interaction Loop

### 2.1 Diagram

```
          ┌─────────────────────────────────────────────────┐
          │            AGENT-ENVIRONMENT LOOP                │
          │                                                  │
          │         ┌──────────────┐                         │
          │         │              │                         │
          │         │    AGENT     │                         │
          │         │   π(s) → a   │                         │
          │         │              │                         │
          │         └──────┬───────┘                         │
          │         action │  ▲  state sₜ                    │
          │           aₜ   │  │  reward rₜ                   │
          │                ▼  │                               │
          │         ┌──────┴───────┐                         │
          │         │              │                         │
          │         │ ENVIRONMENT  │                         │
          │         │  P(s'|s,a)   │                         │
          │         │  R(s,a)      │                         │
          │         │              │                         │
          │         └──────────────┘                         │
          │                                                  │
          └─────────────────────────────────────────────────┘
```

### 2.2 Step-by-Step Protocol

At each discrete time step $t = 0, 1, 2, \dots$:

| Step | What Happens                                          | Notation              |
|------|-------------------------------------------------------|-----------------------|
| 1    | Agent observes the current state                      | $s_t \in S$           |
| 2    | Agent selects an action according to policy $\pi$     | $a_t \sim \pi(\cdot | s_t)$ |
| 3    | Environment transitions to a new state                | $s_{t+1} \sim P(\cdot | s_t, a_t)$ |
| 4    | Environment emits a reward                            | $r_{t+1} = R(s_t, a_t)$ |
| 5    | Agent stores the experience $(s_t, a_t, r_{t+1}, s_{t+1})$ | experience tuple |

---

## 3. The Agent

### 3.1 Definition

The **agent** is the entity that:
- **Perceives** the state of the environment.
- **Decides** which action to take.
- **Learns** from the consequences of its actions.

### 3.2 Agent Components

```
  ┌──────────────────────────────────────────┐
  │              AGENT INTERNALS             │
  │                                          │
  │   ┌────────────────────────────┐         │
  │   │  Policy   π(a | s)        │ ← core  │
  │   ├────────────────────────────┤         │
  │   │  Value fn V(s) or Q(s,a)  │ ← eval  │
  │   ├────────────────────────────┤         │
  │   │  Model    P̂(s'|s,a)       │ ← opt.  │
  │   └────────────────────────────┘         │
  │                                          │
  │   Policy: maps states to actions         │
  │   Value function: estimates return       │
  │   Model: predicts env dynamics (if any)  │
  └──────────────────────────────────────────┘
```

| Component          | Formula                             | Required? |
|-------------------|--------------------------------------|-----------|
| **Policy**         | $\pi: S \rightarrow \Delta(A)$       | Always    |
| **Value function** | $V^\pi(s) = \mathbb{E}_\pi[G_t \mid s_t=s]$ | Usually |
| **Model**          | $\hat{P}(s' \mid s, a)$              | Optional  |

---

## 4. The Environment

### 4.1 Definition

The **environment** is everything external to the agent. It:
- Receives the agent's action.
- Transitions to a new state.
- Produces a reward signal.

### 4.2 Environment Dynamics

The environment is characterised by:

$$P(s', r \mid s, a) = \Pr\{S_{t+1} = s', R_{t+1} = r \mid S_t = s, A_t = a\}$$

This is the **four-argument dynamics function** from which we derive:

| Derived Quantity           | Formula                                                            |
|---------------------------|--------------------------------------------------------------------|
| State-transition probs    | $P(s' \mid s,a) = \sum_r P(s',r \mid s,a)$                       |
| Expected reward           | $r(s,a) = \mathbb{E}[R_{t+1} \mid S_t=s, A_t=a] = \sum_r r \sum_{s'} P(s',r \mid s,a)$ |
| Expected reward (s,a,s')  | $r(s,a,s') = \mathbb{E}[R_{t+1} \mid S_t=s, A_t=a, S_{t+1}=s']$ |

---

## 5. State Space $S$

### 5.1 Formal Definition

The **state** $s_t$ contains all information available to the agent at time $t$.

$$s_t \in S$$

### 5.2 Discrete vs Continuous States

```
  ┌───────────────────────────────────────────────────────────┐
  │           STATE SPACE EXAMPLES                            │
  │                                                           │
  │  DISCRETE                      CONTINUOUS                 │
  │  ┌───┬───┬───┬───┐            ┌─────────────────┐        │
  │  │ 0 │ 1 │ 2 │ 3 │            │  s = [x, ẋ,     │        │
  │  ├───┼───┼───┼───┤            │       θ, θ̇ ]    │        │
  │  │ 4 │ 5 │ 6 │ 7 │            │                  │        │
  │  ├───┼───┼───┼───┤            │  s ∈ ℝ⁴          │        │
  │  │ 8 │ 9 │10 │11 │            │                  │        │
  │  └───┴───┴───┴───┘            └─────────────────┘        │
  │  Grid world: |S| = 12         CartPole: s ∈ ℝ⁴           │
  └───────────────────────────────────────────────────────────┘
```

| Property    | Discrete States              | Continuous States            |
|-------------|------------------------------|------------------------------|
| Type        | Finite or countable set      | Subset of $\mathbb{R}^n$    |
| Examples    | Chess board, grid position   | Joint angles, velocities     |
| Storage     | Tabular (look-up table)      | Function approximation (NN)  |
| Algorithms  | Q-Learning, SARSA            | DQN, PPO, SAC                |

### 5.3 Observation vs State

- **State** $s_t$: Complete description (Markov property satisfied).
- **Observation** $o_t$: Partial view of the state (Partially Observable MDP).

$$\text{If } o_t = s_t: \text{ fully observable (MDP)}$$
$$\text{If } o_t \neq s_t: \text{ partially observable (POMDP)}$$

---

## 6. Action Space $A$

### 6.1 Formal Definition

$$a_t \in A \quad \text{or} \quad a_t \in A(s_t) \text{ (state-dependent)}$$

### 6.2 Discrete vs Continuous Actions

```
  ┌──────────────────────────────────────────────────────────┐
  │              ACTION SPACE EXAMPLES                       │
  │                                                          │
  │  DISCRETE                     CONTINUOUS                 │
  │  ┌─────┐                      ┌────────────────┐        │
  │  │  ↑  │  Up                  │ torque ∈ [-2,2]│        │
  │  │← →│  Left / Right        │ throttle ∈ [0,1]│        │
  │  │  ↓  │  Down                │ steering angle  │        │
  │  └─────┘                      └────────────────┘        │
  │  |A| = 4                      A ⊂ ℝ^d                   │
  └──────────────────────────────────────────────────────────┘
```

| Property    | Discrete Actions             | Continuous Actions           |
|-------------|------------------------------|------------------------------|
| Type        | Finite set                   | Subset of $\mathbb{R}^d$    |
| Examples    | {left, right, up, down}      | Steering angle, force        |
| Selection   | argmax over Q-values         | Sampled from policy dist.    |
| Algorithms  | DQN, SARSA                   | DDPG, TD3, SAC, PPO          |

---

## 7. Reward Signal $R_t$

### 7.1 Formal Definition

$$R_{t+1} = R(s_t, a_t) \in \mathbb{R}$$

The reward is the agent's **sole signal** for evaluating the quality of its actions.

### 7.2 Reward Design Examples

| Environment       | Positive Reward          | Negative Reward          | Terminal Reward  |
|-------------------|--------------------------|--------------------------|------------------|
| **Grid World**    | +1 reach goal            | -0.01 per step           | 0 at boundary    |
| **CartPole**      | +1 every balanced step   | 0 (episode ends)         | N/A              |
| **Chess**         | 0 per move               | 0 per move               | +1 win, -1 loss  |
| **Robot Walk**    | +1 per metre forward     | -1 for falling           | Episode end      |

### 7.3 Sparse vs Dense Rewards

```
  Dense Rewards                     Sparse Rewards
  ┌──┬──┬──┬──┬──┬──┬──┬──┐       ┌──┬──┬──┬──┬──┬──┬──┬──┐
  │+1│+1│+2│-1│+3│+1│+2│+5│       │ 0│ 0│ 0│ 0│ 0│ 0│ 0│+1│
  └──┴──┴──┴──┴──┴──┴──┴──┘       └──┴──┴──┴──┴──┴──┴──┴──┘
   t=0              t=7              t=0              t=7

  Easier to learn from              Harder — requires exploration
  But may cause reward hacking      More natural for many tasks
```

---

## 8. Putting It All Together: A Grid World Example

```
  ┌───┬───┬───┬───┐
  │ S │   │   │   │    S = Start (agent begins here)
  ├───┼───┼───┼───┤    G = Goal  (+1 reward)
  │   │ ▓ │   │   │    ▓ = Wall  (cannot enter)
  ├───┼───┼───┼───┤    T = Trap  (-1 reward)
  │   │   │ T │ G │
  └───┴───┴───┴───┘

  State space  S = {(0,0), (0,1), ..., (2,3)} \ {(1,1)}   |S| = 11
  Action space A = {UP, DOWN, LEFT, RIGHT}                  |A| = 4
  Rewards:     R(s, a) = +1 if s' = G
                         -1 if s' = T
                       -0.01 otherwise (step penalty)
```

---

## 9. Gymnasium (OpenAI Gym) Code Examples

### 9.1 CartPole — Discrete Actions, Continuous States

```python
import gymnasium as gym

# Create CartPole environment
env = gym.make("CartPole-v1")

# Inspect spaces
print(f"State space : {env.observation_space}")       # Box(4,)
print(f"Action space: {env.action_space}")             # Discrete(2)
print(f"State shape : {env.observation_space.shape}")  # (4,)
print(f"Num actions : {env.action_space.n}")           # 2

# Run one episode
state, info = env.reset(seed=42)
print(f"\nInitial state: {state}")

total_reward = 0
for t in range(500):
    action = env.action_space.sample()  # random action
    next_state, reward, terminated, truncated, info = env.step(action)

    print(f"  t={t:3d}  a={action}  r={reward:.1f}  "
          f"s'={next_state[:2]}...  done={terminated or truncated}")

    total_reward += reward
    state = next_state

    if terminated or truncated:
        break

print(f"\nEpisode finished after {t+1} steps | Total reward: {total_reward}")
env.close()
```

### 9.2 MountainCar — Understanding State & Action

```python
import gymnasium as gym

env = gym.make("MountainCar-v0")

print("=== MountainCar Environment ===")
print(f"State space  : {env.observation_space}")
print(f"  Low bounds : {env.observation_space.low}")
print(f"  High bounds: {env.observation_space.high}")
print(f"Action space : {env.action_space}  (0=left, 1=none, 2=right)")

state, _ = env.reset(seed=0)
print(f"\nStart position: {state[0]:.4f}, velocity: {state[1]:.4f}")

# Take 5 "push right" actions
for i in range(5):
    state, reward, term, trunc, _ = env.step(2)  # action=2 (right)
    print(f"  Step {i+1}: pos={state[0]:.4f}  vel={state[1]:.4f}  r={reward}")

env.close()
```

### 9.3 Creating a Custom Grid-World Environment

```python
import numpy as np

class SimpleGridWorld:
    """Minimal grid world to demonstrate RL components."""

    def __init__(self, rows=3, cols=4):
        self.rows, self.cols = rows, cols
        self.start = (0, 0)
        self.goal  = (2, 3)
        self.trap  = (2, 2)
        self.wall  = (1, 1)
        self.state = self.start

        # Action mapping: 0=UP, 1=DOWN, 2=LEFT, 3=RIGHT
        self.actions = {0: (-1, 0), 1: (1, 0), 2: (0, -1), 3: (0, 1)}
        self.action_names = {0: "UP", 1: "DOWN", 2: "LEFT", 3: "RIGHT"}
        self.n_states  = rows * cols
        self.n_actions = 4

    def reset(self):
        self.state = self.start
        return self.state

    def step(self, action):
        dr, dc = self.actions[action]
        new_r = max(0, min(self.rows - 1, self.state[0] + dr))
        new_c = max(0, min(self.cols - 1, self.state[1] + dc))
        new_state = (new_r, new_c)

        if new_state == self.wall:
            new_state = self.state  # can't move into wall

        self.state = new_state

        if self.state == self.goal:
            return self.state, +1.0, True
        elif self.state == self.trap:
            return self.state, -1.0, True
        else:
            return self.state, -0.01, False

    def render(self):
        for r in range(self.rows):
            row_str = "|"
            for c in range(self.cols):
                pos = (r, c)
                if pos == self.state:
                    row_str += " A |"
                elif pos == self.goal:
                    row_str += " G |"
                elif pos == self.trap:
                    row_str += " T |"
                elif pos == self.wall:
                    row_str += " ▓ |"
                else:
                    row_str += "   |"
            print(row_str)
        print()

# --- Run the custom environment ---
env = SimpleGridWorld()
state = env.reset()
env.render()

total_reward = 0
for step in range(20):
    action = np.random.randint(4)
    next_state, reward, done = env.step(action)
    total_reward += reward
    print(f"Step {step+1}: {env.action_names[action]:>5s}  "
          f"→ {next_state}  r={reward:+.2f}")
    if done:
        print(f"\nEpisode done! Total reward: {total_reward:.2f}")
        env.render()
        break
```

---

## 10. Algorithm: Generic RL Interaction Loop

```
Algorithm: Agent-Environment Interaction Loop
──────────────────────────────────────────────────
Input : environment env, policy π, number of episodes K
Output: collected experience

1.  FOR episode = 1 TO K DO
2.      s ← env.reset()
3.      done ← False
4.      WHILE NOT done DO
5.          a ← π(s)                        ▷ select action
6.          s', r, done ← env.step(a)       ▷ execute in env
7.          Store (s, a, r, s') in memory
8.          Update π using (s, a, r, s')    ▷ learning step
9.          s ← s'
10.     END WHILE
11. END FOR
```

---

## 11. Summary

```
╔════════════════════════════════════════════════════════════════════╗
║  CHAPTER 3 — KEY TAKEAWAYS                                       ║
╠════════════════════════════════════════════════════════════════════╣
║  1. The agent-environment loop is the foundation of all RL       ║
║  2. State sₜ captures all information needed for decision        ║
║  3. Actions can be discrete or continuous                         ║
║  4. Reward is scalar, immediate, and task-specific               ║
║  5. The dynamics function P(s',r|s,a) fully specifies the env    ║
║  6. Gymnasium provides standard interfaces for RL environments    ║
║  7. Observation ≠ State when the environment is partially        ║
║     observable (POMDP)                                            ║
╚════════════════════════════════════════════════════════════════════╝
```

| Component    | Symbol         | Description                             |
|-------------|----------------|-----------------------------------------|
| Agent        | —              | The learner and decision-maker          |
| Environment  | —              | The external world                      |
| State        | $s_t \in S$    | Current situation                       |
| Action       | $a_t \in A$    | Agent's choice                          |
| Reward       | $r_{t+1}$      | Scalar feedback                         |
| Policy       | $\pi(a \mid s)$| Action selection strategy               |
| Dynamics     | $P(s' \mid s,a)$| State transition probabilities         |

---

## 12. Revision Questions

1. **Draw the agent-environment interaction loop** and label every arrow with its
   mathematical symbol.

2. **What is the difference between a state and an observation?** When does this
   distinction matter?

3. **Give one example each** of a discrete state space and a continuous state space.
   What algorithmic implications does each have?

4. **Write the four-argument dynamics function** $P(s', r \mid s, a)$ and derive the
   state-transition probability $P(s' \mid s, a)$ from it.

5. **What is reward shaping?** Why might a dense reward be easier to learn from than a
   sparse reward?

6. **Using Gymnasium**, write pseudocode to run 100 episodes of CartPole with a random
   policy and compute the average episode length.

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [RL vs SL vs UL](02-rl-vs-sl-vs-ul.md) | [Unit 1 Index](../README.md) | [Episodes and Trajectories](04-episodes-trajectories.md) |

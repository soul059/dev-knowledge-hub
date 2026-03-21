# Chapter 1: What is Reinforcement Learning?

> **Unit 1 · Introduction to RL** — File 1 of 5

---

## 1. Overview

Reinforcement Learning (RL) is a branch of machine learning where an **agent** learns to
make decisions by interacting with an **environment**. Unlike supervised learning, the agent
is never told the correct action — it must discover which actions yield the highest
cumulative **reward** through trial and error.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║                REINFORCEMENT LEARNING AT A GLANCE               ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • An agent takes actions in an environment                     ║
 ║  • The environment returns a new state and a reward             ║
 ║  • The agent's goal: maximise the total reward over time        ║
 ║  • Learning happens through trial-and-error, not from labels    ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 2. Learning from Interaction

At its core, RL mirrors how humans and animals learn. A child touches a hot stove
(action), feels pain (negative reward), and learns to avoid the stove in the future.
No one handed the child a labelled dataset of "things not to touch."

### The Trial-and-Error Paradigm

```
  ┌──────────────────────────────────────────────────────────────┐
  │                   THE RL LEARNING LOOP                       │
  │                                                              │
  │        ┌───────────┐    action aₜ     ┌─────────────┐       │
  │        │           │ ───────────────▶  │             │       │
  │        │   AGENT   │                   │ ENVIRONMENT │       │
  │        │           │ ◀───────────────  │             │       │
  │        └───────────┘  state sₜ₊₁      └─────────────┘       │
  │              ▲        reward rₜ₊₁            │               │
  │              │                                │               │
  │              └────────────────────────────────┘               │
  │                     feedback loop                             │
  └──────────────────────────────────────────────────────────────┘
```

The loop repeats at every **time step** $t$:

1. The agent observes state $s_t$.
2. The agent selects action $a_t$ according to its **policy** $\pi$.
3. The environment transitions to state $s_{t+1}$ and emits reward $r_{t+1}$.
4. The agent updates its policy to improve future decisions.

---

## 3. Reward Signals as Feedback

The **reward signal** $R_t \in \mathbb{R}$ is the only feedback the agent receives.
It is a scalar value that tells the agent how good or bad its last action was.

$$R_t : S \times A \rightarrow \mathbb{R}$$

### Properties of Reward Signals

| Property              | Description                                                |
|-----------------------|------------------------------------------------------------|
| **Scalar**            | A single number, not a vector                              |
| **Immediate**         | Given right after the action (but may reflect delayed impact)|
| **Task-specific**     | Designed by the engineer to encode the objective           |
| **Sparse or dense**   | May be given every step or only at episode end             |

> **Key Insight:** The agent's entire objective is to maximise the **expected cumulative
> reward** (called the *return*), not just the immediate reward.

$$G_t = \sum_{k=0}^{\infty} \gamma^k \, r_{t+k+1}$$

where $\gamma \in [0, 1]$ is the **discount factor**.

---

## 4. Why Not Just Program the Rules?

In many problems, writing explicit rules is impractical or impossible:

```
  ┌─────────────────────────────────────────────────────┐
  │     RULE-BASED vs REINFORCEMENT LEARNING            │
  │                                                     │
  │  Rule-Based:                                        │
  │    IF obstacle_ahead THEN turn_left                 │
  │    IF goal_visible   THEN move_forward              │
  │    IF enemy_near     THEN ???  ← breaks down        │
  │                                                     │
  │  Reinforcement Learning:                            │
  │    Agent discovers strategies by itself              │
  │    Adapts to unseen situations                       │
  │    Can find super-human strategies                   │
  │    Handles enormous state spaces                     │
  └─────────────────────────────────────────────────────┘
```

| Criterion               | Rule-Based               | RL                        |
|--------------------------|--------------------------|---------------------------|
| Scalability              | Poor for large spaces    | Handles huge state spaces |
| Adaptability             | Static rules             | Learns and adapts         |
| Design effort            | Must anticipate all cases| Define reward only        |
| Optimality               | As good as the designer  | Can exceed human skill    |

---

## 5. Key Characteristics of RL

### 5.1 Sequential Decision Making

Unlike a single prediction, RL involves a **sequence** of decisions where each action
affects future states and rewards.

```
  Time ──▶  t=0       t=1       t=2       t=3       t=4
            ┌──┐      ┌──┐      ┌──┐      ┌──┐      ┌──┐
  States:   │s₀│─────▶│s₁│─────▶│s₂│─────▶│s₃│─────▶│s₄│
            └──┘      └──┘      └──┘      └──┘      └──┘
              │  a₀     │  a₁     │  a₂     │  a₃
              ▼         ▼         ▼         ▼
            r₁=+1     r₂=-1     r₃=+5     r₄=+2
```

### 5.2 Delayed Rewards

A chess player sacrifices a pawn now for a checkmate ten moves later. The reward
for the sacrifice is **delayed** — it arrives many steps after the decision.

$$\text{Immediate reward is not always the best signal for long-term success.}$$

### 5.3 Exploration vs Exploitation

The agent faces a fundamental dilemma:

```
  ┌──────────────────────────────────────────────────┐
  │         EXPLORATION vs EXPLOITATION              │
  │                                                  │
  │   EXPLOITATION                EXPLORATION        │
  │   ┌──────────────┐           ┌──────────────┐   │
  │   │ Use the best │           │ Try new,     │   │
  │   │ known action │           │ unknown      │   │
  │   │ to get high  │    vs     │ actions to   │   │
  │   │ reward now   │           │ discover     │   │
  │   │              │           │ better ones  │   │
  │   └──────────────┘           └──────────────┘   │
  │                                                  │
  │   Example: Always go to your favourite           │
  │   restaurant vs trying a new one.                │
  └──────────────────────────────────────────────────┘
```

A good RL agent must **balance** both:
- **Exploit** current knowledge to gain reward.
- **Explore** new actions to potentially find higher rewards.

---

## 6. Mathematical Foundations

### Core Definitions

| Symbol       | Name              | Definition                                                         |
|--------------|-------------------|--------------------------------------------------------------------|
| $S$          | State space       | Set of all possible states                                         |
| $A$          | Action space      | Set of all possible actions                                        |
| $R$          | Reward function   | $R: S \times A \rightarrow \mathbb{R}$                             |
| $\pi$        | Policy            | $\pi: S \rightarrow A$ (or distribution over $A$)                  |
| $\gamma$     | Discount factor   | $\gamma \in [0, 1]$, balances immediate vs future reward           |
| $G_t$        | Return            | $G_t = \sum_{k=0}^{\infty} \gamma^k r_{t+k+1}$                    |
| $V^\pi(s)$   | Value function    | $V^\pi(s) = \mathbb{E}_\pi[G_t \mid s_t = s]$                     |
| $Q^\pi(s,a)$ | Action-value fn   | $Q^\pi(s,a) = \mathbb{E}_\pi[G_t \mid s_t = s, a_t = a]$         |

### The RL Objective

The agent seeks a policy $\pi^*$ that maximises the expected return:

$$\pi^* = \arg\max_\pi \; \mathbb{E}_\pi \left[ \sum_{k=0}^{\infty} \gamma^k \, r_{t+k+1} \right]$$

---

## 7. A Simple RL Loop in Python

Below is a minimal RL loop using the **ε-greedy** strategy with a multi-armed bandit.

```python
import numpy as np

# --- Simple Multi-Armed Bandit RL Loop ---

np.random.seed(42)

n_arms = 5
n_steps = 1000
epsilon = 0.1  # exploration rate

# True (unknown) reward distribution for each arm
true_rewards = np.random.randn(n_arms)

# Agent's estimates and counts
Q = np.zeros(n_arms)       # estimated action values
N = np.zeros(n_arms)       # action counts
total_reward = 0.0
rewards_over_time = []

for t in range(1, n_steps + 1):
    # --- Action Selection: epsilon-greedy ---
    if np.random.rand() < epsilon:
        action = np.random.randint(n_arms)       # explore
    else:
        action = np.argmax(Q)                     # exploit

    # --- Environment returns a reward ---
    reward = true_rewards[action] + np.random.randn() * 0.5

    # --- Update estimates (incremental mean) ---
    N[action] += 1
    Q[action] += (reward - Q[action]) / N[action]

    total_reward += reward
    rewards_over_time.append(total_reward / t)

print(f"True best arm : {np.argmax(true_rewards)}")
print(f"Agent's best  : {np.argmax(Q)}")
print(f"Average reward: {total_reward / n_steps:.3f}")
```

### Pseudocode: ε-Greedy Action Selection

```
Algorithm: ε-Greedy Action Selection
─────────────────────────────────────────
Input : Q[a] for all a ∈ A, exploration rate ε
Output: selected action a

1.  Generate random number p ~ Uniform(0, 1)
2.  IF p < ε THEN
3.      a ← random action from A          ▷ Explore
4.  ELSE
5.      a ← argmax_a Q[a]                  ▷ Exploit
6.  END IF
7.  RETURN a
```

---

## 8. Real-World Applications Overview

```
  ┌──────────────────────────────────────────────────────┐
  │              RL IN THE REAL WORLD                     │
  │                                                      │
  │  🎮  Games           AlphaGo, Atari, StarCraft II   │
  │  🤖  Robotics        Grasping, walking, flying      │
  │  🚗  Self-driving    Lane keeping, navigation       │
  │  💊  Healthcare      Treatment optimisation          │
  │  📈  Finance         Portfolio management            │
  │  🗣️  NLP / LLMs      RLHF for ChatGPT              │
  │  🏭  Industrial      HVAC control, supply chain     │
  └──────────────────────────────────────────────────────┘
```

| Domain              | Example Problem           | RL Technique          |
|---------------------|---------------------------|-----------------------|
| Games               | Atari game playing        | Deep Q-Network (DQN)  |
| Robotics            | Robotic arm control       | PPO, SAC              |
| Autonomous Driving  | Lane following            | Actor-Critic          |
| Finance             | Portfolio optimisation    | Policy Gradient       |
| Healthcare          | Drug dosing               | Contextual Bandits    |
| LLM Alignment       | ChatGPT fine-tuning       | RLHF (PPO)           |

---

## 9. Summary

```
╔════════════════════════════════════════════════════════════════════╗
║  CHAPTER 1 — KEY TAKEAWAYS                                       ║
╠════════════════════════════════════════════════════════════════════╣
║  1. RL = learning from interaction via trial-and-error            ║
║  2. The agent receives scalar rewards, not correct labels         ║
║  3. Sequential decisions — each action affects future states      ║
║  4. Delayed rewards — good actions may pay off much later         ║
║  5. Exploration vs exploitation — a fundamental trade-off         ║
║  6. Goal: find policy π* that maximises expected cumulative       ║
║     reward (return)                                               ║
║  7. RL is applicable wherever sequential decisions must be made   ║
╚════════════════════════════════════════════════════════════════════╝
```

| Concept                | One-Line Summary                                         |
|------------------------|----------------------------------------------------------|
| Reinforcement Learning | Learning optimal behaviour through reward feedback       |
| Agent                  | The decision-maker                                       |
| Environment            | Everything outside the agent                             |
| Reward                 | Scalar feedback signal                                   |
| Policy                 | Mapping from states to actions                           |
| Exploration            | Trying new actions to gather information                 |
| Exploitation           | Using known best actions for maximum reward              |

---

## 10. Revision Questions

1. **Define reinforcement learning in one sentence.** How does it differ from being
   given a labelled dataset?

2. **What is the exploration vs exploitation dilemma?** Give an everyday example that
   is *not* related to restaurants.

3. **Why can't we always use hand-coded rules instead of RL?** Name two domains where
   rule-based approaches fail.

4. **Write the formula for the discounted return $G_t$.** Explain each symbol.

5. **What role does the reward signal play?** Why must it be a scalar rather than a
   vector?

6. **In the ε-greedy strategy, what happens when ε = 0? When ε = 1?** Which setting
   leads to pure exploitation?

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [README](../README.md) | [Unit 1 Index](../README.md) | [RL vs SL vs UL](02-rl-vs-sl-vs-ul.md) |

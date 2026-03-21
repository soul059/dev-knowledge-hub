# Chapter 4: Episodes and Trajectories

> **Unit 1 · Introduction to RL** — File 4 of 5

---

## 1. Overview

When an RL agent interacts with an environment, the sequence of states, actions, and
rewards it generates is called a **trajectory**. If the interaction has a natural ending
point (e.g., the game is won or lost), it constitutes an **episode**. This chapter
formalises trajectories, returns, and the discount factor $\gamma$.

```
 ╔════════════════════════════════════════════════════════════════════╗
 ║                 EPISODES & TRAJECTORIES                           ║
 ╠════════════════════════════════════════════════════════════════════╣
 ║  Trajectory τ  =  (s₀, a₀, r₁, s₁, a₁, r₂, s₂, ... )          ║
 ║  Episode       =  a trajectory that terminates                    ║
 ║  Return Gₜ     =  Σ γ^k · r_{t+k+1}  (discounted sum of rewards)║
 ║  Discount γ    =  how much we value future vs. present rewards    ║
 ╚════════════════════════════════════════════════════════════════════╝
```

---

## 2. Episodic vs Continuing Tasks

### 2.1 Episodic Tasks

An **episodic task** naturally breaks into sequences (episodes) that end in a
**terminal state** $s_T$.

```
  Episode 1:   s₀ ──a₀──▶ s₁ ──a₁──▶ s₂ ──a₂──▶ s₃ ──a₃──▶ [sT]  ← terminal
  Episode 2:   s₀ ──a₀──▶ s₁ ──a₁──▶ s₂ ──a₂──▶ [sT]              ← terminal
  Episode 3:   s₀ ──a₀──▶ s₁ ──a₁──▶ s₂ ──a₂──▶ s₃ ──a₃──▶ s₄ ──▶ [sT]
```

**Examples:**
- A game of chess (ends with win/loss/draw).
- CartPole (ends when the pole falls or 500 steps reached).
- A maze (ends when the exit is found).

### 2.2 Continuing Tasks

A **continuing task** has no natural endpoint — it runs indefinitely.

```
  ... ──▶ sₜ ──aₜ──▶ sₜ₊₁ ──aₜ₊₁──▶ sₜ₊₂ ──aₜ₊₂──▶ sₜ₊₃ ──▶ ...
                        (no terminal state)
```

**Examples:**
- A robot continuously monitoring a factory floor.
- An HVAC controller maintaining building temperature.
- A stock trading agent operating every trading day.

### 2.3 Comparison

| Property            | Episodic Tasks              | Continuing Tasks              |
|---------------------|-----------------------------|-------------------------------|
| Terminal state      | Yes                         | No                            |
| Horizon             | Finite ($T < \infty$)       | Infinite ($T = \infty$)       |
| Reset               | Environment resets each ep. | No reset                      |
| Return              | Finite sum possible         | Must discount ($\gamma < 1$)  |
| Examples            | Games, mazes, exams         | Temperature control, trading  |

---

## 3. Trajectory Notation

A **trajectory** $\tau$ is the complete record of one sequence of interactions:

$$\tau = (s_0, a_0, r_1, s_1, a_1, r_2, s_2, a_2, r_3, \dots)$$

```
  ┌─────────────────────────────────────────────────────────────────┐
  │                    ANATOMY OF A TRAJECTORY                      │
  │                                                                 │
  │  Time:     t=0        t=1        t=2        t=3                │
  │            ┌──┐       ┌──┐       ┌──┐       ┌──┐              │
  │  State:    │s₀│──────▶│s₁│──────▶│s₂│──────▶│s₃│ ...          │
  │            └──┘       └──┘       └──┘       └──┘              │
  │              │          │          │                            │
  │  Action:    a₀         a₁         a₂                           │
  │              │          │          │                            │
  │  Reward:   r₁         r₂         r₃                            │
  │            (received   (received   (received                   │
  │             at t=1)     at t=2)     at t=3)                    │
  │                                                                 │
  │  τ = (s₀, a₀, r₁, s₁, a₁, r₂, s₂, a₂, r₃, s₃, ...)         │
  └─────────────────────────────────────────────────────────────────┘
```

For an **episodic** task of length $T$:

$$\tau = (s_0, a_0, r_1, s_1, a_1, r_2, \dots, s_{T-1}, a_{T-1}, r_T, s_T)$$

where $s_T$ is the terminal state.

---

## 4. Return $G_t$

The **return** $G_t$ is the total reward the agent accumulates starting from time $t$.

### 4.1 Undiscounted Return (Episodic, Finite Horizon)

$$G_t = r_{t+1} + r_{t+2} + r_{t+3} + \dots + r_T = \sum_{k=0}^{T-t-1} r_{t+k+1}$$

### 4.2 Discounted Return

For continuing tasks (or when we want to value near-term rewards more), we apply a
**discount factor** $\gamma \in [0, 1]$:

$$\boxed{G_t = \sum_{k=0}^{\infty} \gamma^k \, r_{t+k+1} = r_{t+1} + \gamma \, r_{t+2} + \gamma^2 \, r_{t+3} + \dots}$$

### 4.3 Recursive Definition

The return satisfies a convenient **recursive** relationship:

$$G_t = r_{t+1} + \gamma \, G_{t+1}$$

**Proof:**

$$
\begin{aligned}
G_t &= r_{t+1} + \gamma \, r_{t+2} + \gamma^2 \, r_{t+3} + \dots \\
    &= r_{t+1} + \gamma \big( r_{t+2} + \gamma \, r_{t+3} + \gamma^2 \, r_{t+4} + \dots \big) \\
    &= r_{t+1} + \gamma \, G_{t+1}
\end{aligned}
$$

This recursion is the basis of **temporal-difference (TD) learning**.

---

## 5. The Discount Factor $\gamma$

### 5.1 Why Discount?

1. **Mathematical necessity**: For continuing tasks, an undiscounted infinite sum may
   diverge. With $\gamma < 1$ and bounded rewards $|r_t| \le R_{max}$:

$$G_t \le \sum_{k=0}^{\infty} \gamma^k R_{max} = \frac{R_{max}}{1 - \gamma}$$

2. **Modelling uncertainty**: Future rewards are less certain — discounting encodes this
   preference for the present.

3. **Practical effectiveness**: Discounting helps algorithms converge faster.

### 5.2 Geometric Series Bound

If all rewards equal $r$:

$$G_t = r \sum_{k=0}^{\infty} \gamma^k = \frac{r}{1 - \gamma}$$

### 5.3 Effect of Different $\gamma$ Values

```
  ┌────────────────────────────────────────────────────────────┐
  │           EFFECT OF DISCOUNT FACTOR γ                      │
  │                                                            │
  │  γ = 0.0    │▓▓▓▓▓▓▓▓▓▓│          │          │            │
  │  (myopic)    r₁ only                                       │
  │                                                            │
  │  γ = 0.5    │▓▓▓▓▓▓▓▓▓▓│▓▓▓▓▓    │▓▓       │▓           │
  │              r₁         0.5·r₂    0.25·r₃   0.125·r₄     │
  │                                                            │
  │  γ = 0.9    │▓▓▓▓▓▓▓▓▓▓│▓▓▓▓▓▓▓▓▓│▓▓▓▓▓▓▓▓│▓▓▓▓▓▓▓     │
  │              r₁         0.9·r₂    0.81·r₃   0.729·r₄     │
  │                                                            │
  │  γ = 1.0    │▓▓▓▓▓▓▓▓▓▓│▓▓▓▓▓▓▓▓▓▓│▓▓▓▓▓▓▓▓▓▓│▓▓▓▓▓▓▓▓▓▓│
  │  (far-sighted) all rewards weighted equally                │
  └────────────────────────────────────────────────────────────┘
```

| $\gamma$ | Behaviour           | Effective Horizon $\approx \frac{1}{1-\gamma}$ | Use Case              |
|----------|---------------------|------------------------------------------------|------------------------|
| 0.0      | Completely myopic   | 1 step                                         | Bandits               |
| 0.5      | Short-sighted       | 2 steps                                        | Quick tasks           |
| 0.9      | Balanced            | 10 steps                                       | Most RL tasks         |
| 0.99     | Far-sighted         | 100 steps                                      | Long-horizon tasks    |
| 0.999    | Very far-sighted    | 1000 steps                                     | Complex planning      |
| 1.0      | No discounting      | ∞ (episodic only)                              | Finite episodes only  |

### 5.4 Numerical Example

Suppose the agent receives rewards $r_1=1, r_2=2, r_3=3, r_4=4$.

| $\gamma$ | $G_0$ Calculation                                         | $G_0$     |
|----------|-----------------------------------------------------------|-----------|
| 0.0      | $1$                                                        | **1.000** |
| 0.5      | $1 + 0.5(2) + 0.25(3) + 0.125(4)$                        | **3.250** |
| 0.9      | $1 + 0.9(2) + 0.81(3) + 0.729(4)$                        | **7.146** |
| 1.0      | $1 + 2 + 3 + 4$                                           | **10.00** |

---

## 6. Mathematical Derivation: Discounted Return

### Finite Horizon

For an episode of length $T$:

$$G_t = \sum_{k=0}^{T-t-1} \gamma^k \, r_{t+k+1}$$

Expanding:

$$G_0 = r_1 + \gamma r_2 + \gamma^2 r_3 + \dots + \gamma^{T-1} r_T$$

### Infinite Horizon (Continuing)

$$G_t = \lim_{T \to \infty} \sum_{k=0}^{T-t-1} \gamma^k \, r_{t+k+1}$$

**Convergence condition**: $\gamma < 1$ and $|r_t| \le R_{max} < \infty$

$$|G_t| \le R_{max} \sum_{k=0}^{\infty} \gamma^k = \frac{R_{max}}{1-\gamma} < \infty \quad \checkmark$$

### Connection to Value Functions

The return is central to defining value functions:

$$V^\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]$$

$$Q^\pi(s, a) = \mathbb{E}_\pi[G_t \mid S_t = s, A_t = a]$$

The **Bellman equation** directly uses the recursive return:

$$V^\pi(s) = \mathbb{E}_\pi\big[r_{t+1} + \gamma \, V^\pi(S_{t+1}) \mid S_t = s\big]$$

---

## 7. Python Code: Computing Returns

### 7.1 Compute Discounted Return from a Reward Sequence

```python
import numpy as np

def compute_return(rewards, gamma=0.99):
    """Compute discounted return G_t for each time step t."""
    T = len(rewards)
    returns = np.zeros(T)

    # Compute backwards (using recursive formula G_t = r_{t+1} + γ G_{t+1})
    G = 0.0
    for t in reversed(range(T)):
        G = rewards[t] + gamma * G
        returns[t] = G

    return returns


# Example trajectory rewards
rewards = [1, 2, 3, 4, 5]

print("Rewards:", rewards)
print()

for gamma in [0.0, 0.5, 0.9, 0.99, 1.0]:
    G = compute_return(rewards, gamma)
    print(f"γ = {gamma:.2f}  →  G₀ = {G[0]:.4f}  |  Returns: {np.round(G, 3)}")
```

**Expected Output:**
```
Rewards: [1, 2, 3, 4, 5]

γ = 0.00  →  G₀ = 1.0000  |  Returns: [1. 2. 3. 4. 5.]
γ = 0.50  →  G₀ = 3.0625  |  Returns: [3.062 4.125 5.25  6.5   5.   ]
γ = 0.90  →  G₀ = 11.2051 |  Returns: [11.205 11.339 10.377  8.5   5.   ]
γ = 0.99  →  G₀ = 14.5614 |  Returns: [14.561 13.698 11.817  8.95  5.   ]
γ = 1.00  →  G₀ = 15.0000 |  Returns: [15. 14. 12.  9.  5.]
```

### 7.2 Collecting Trajectories with Gymnasium

```python
import gymnasium as gym
import numpy as np

def collect_trajectory(env_name="CartPole-v1", seed=42):
    """Collect a complete trajectory (episode) using a random policy."""
    env = gym.make(env_name)
    state, info = env.reset(seed=seed)

    trajectory = {
        "states": [state],
        "actions": [],
        "rewards": [],
    }

    done = False
    while not done:
        action = env.action_space.sample()
        next_state, reward, terminated, truncated, info = env.step(action)
        done = terminated or truncated

        trajectory["actions"].append(action)
        trajectory["rewards"].append(reward)
        trajectory["states"].append(next_state)

    env.close()
    return trajectory


def trajectory_stats(trajectory, gamma=0.99):
    """Print trajectory statistics."""
    rewards = trajectory["rewards"]
    T = len(rewards)
    returns = np.zeros(T)

    G = 0.0
    for t in reversed(range(T)):
        G = rewards[t] + gamma * G
        returns[t] = G

    print(f"Episode length   : {T}")
    print(f"Total reward     : {sum(rewards):.2f}")
    print(f"Discounted return: {returns[0]:.4f}  (γ={gamma})")
    print(f"Min/Max reward   : {min(rewards):.2f} / {max(rewards):.2f}")
    return returns


# Collect and analyse a trajectory
tau = collect_trajectory("CartPole-v1")
G = trajectory_stats(tau, gamma=0.99)
```

### 7.3 Visualising Discount Effect

```python
import numpy as np

def visualise_discount(n_steps=20, gamma_values=[0.0, 0.5, 0.9, 0.99, 1.0]):
    """Show how discount weights decay over time."""
    print(f"{'Step':>4} ", end="")
    for g in gamma_values:
        print(f"| γ={g:<5}", end="")
    print("|")
    print("-" * (6 + 9 * len(gamma_values)))

    for k in range(n_steps):
        print(f"{k:>4} ", end="")
        for g in gamma_values:
            weight = g ** k
            bar = "▓" * int(weight * 8)
            print(f"| {weight:.3f} ", end="")
        print("|")


visualise_discount(n_steps=10)
```

---

## 8. Algorithm: Compute Monte Carlo Return

```
Algorithm: Monte Carlo Return Computation
──────────────────────────────────────────
Input : reward sequence [r₁, r₂, ..., r_T], discount factor γ
Output: returns [G₀, G₁, ..., G_{T-1}]

1.  G ← 0
2.  FOR t = T-1 DOWNTO 0 DO
3.      G ← r_{t+1} + γ · G          ▷ recursive formula
4.      returns[t] ← G
5.  END FOR
6.  RETURN returns
```

**Time complexity:** $O(T)$ — single backward pass.
**Space complexity:** $O(T)$ — storing all returns.

---

## 9. Real-World Applications

| Concept            | Application                                     |
|--------------------|-------------------------------------------------|
| Episodic tasks     | Board games, dialogue sessions, clinical trials |
| Continuing tasks   | Portfolio management, server load balancing      |
| High $\gamma$      | Long-term investment, chess (delayed checkmate)  |
| Low $\gamma$       | Day trading, immediate safety responses          |
| Return $G_t$       | Evaluating any RL policy's performance           |

---

## 10. Common Pitfalls

```
  ┌────────────────────────────────────────────────────────────────┐
  │                    COMMON MISTAKES                             │
  │                                                                │
  │  ✗  Using γ = 1.0 for continuing tasks                        │
  │     → Return may diverge to infinity!                          │
  │                                                                │
  │  ✗  Confusing reward rₜ₊₁ with return Gₜ                      │
  │     → Reward is one step; return is the cumulative sum         │
  │                                                                │
  │  ✗  Computing returns forward instead of backward              │
  │     → Backward computation is O(T); forward is O(T²)          │
  │                                                                │
  │  ✗  Forgetting the +1 offset in rₜ₊₁                          │
  │     → rₜ₊₁ is the reward received AFTER taking aₜ in sₜ       │
  │                                                                │
  │  ✓  Always use the recursive formula G_t = r_{t+1} + γ G_{t+1}│
  └────────────────────────────────────────────────────────────────┘
```

---

## 11. Summary

```
╔════════════════════════════════════════════════════════════════════╗
║  CHAPTER 4 — KEY TAKEAWAYS                                       ║
╠════════════════════════════════════════════════════════════════════╣
║  1. A trajectory τ records the full (s, a, r) sequence           ║
║  2. Episodic tasks end; continuing tasks run forever             ║
║  3. Return Gₜ = Σ γᵏ rₜ₊ₖ₊₁ is the agent's objective           ║
║  4. Recursive: Gₜ = rₜ₊₁ + γ Gₜ₊₁ (basis of TD learning)      ║
║  5. γ < 1 ensures convergence for infinite horizons              ║
║  6. γ controls the trade-off between short- and long-term reward ║
║  7. Compute returns backward in O(T) time                        ║
╚════════════════════════════════════════════════════════════════════╝
```

| Concept             | Formula / Definition                                             |
|---------------------|------------------------------------------------------------------|
| Trajectory          | $\tau = (s_0, a_0, r_1, s_1, a_1, r_2, \dots)$                 |
| Return              | $G_t = \sum_{k=0}^{\infty} \gamma^k r_{t+k+1}$                 |
| Recursive return    | $G_t = r_{t+1} + \gamma G_{t+1}$                                |
| Discount factor     | $\gamma \in [0, 1]$                                              |
| Effective horizon   | $\approx \frac{1}{1 - \gamma}$                                  |
| Bounded return      | $|G_t| \le \frac{R_{max}}{1 - \gamma}$                          |

---

## 12. Revision Questions

1. **Define a trajectory $\tau$ and an episode.** How do they differ?

2. **Write the discounted return formula** and explain why we need discounting for
   continuing tasks.

3. **Prove the recursive relationship** $G_t = r_{t+1} + \gamma G_{t+1}$ starting from
   the definition of $G_t$.

4. **A trajectory has rewards** $[2, 3, 5, 1]$. Compute $G_0$ for $\gamma = 0.9$.
   *(Answer: $2 + 0.9(3) + 0.81(5) + 0.729(1) = 9.479$)*

5. **Why is $\gamma = 1.0$ dangerous for continuing tasks?** What mathematical property
   breaks down?

6. **Explain the connection** between the recursive return and the Bellman equation.
   Why is this recursion important for practical RL algorithms?

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [Agent, Environment, State, Action, Reward](03-agent-environment.md) | [Unit 1 Index](../README.md) | [RL Applications](05-rl-applications.md) |

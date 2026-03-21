# Chapter 1: The Markov Property

> **Unit 2: Markov Decision Processes** — *The foundational assumption that makes reinforcement learning tractable.*

---

## Overview

The Markov property is the cornerstone assumption of reinforcement learning. It states that
the future is conditionally independent of the past, given the present. Without this property,
an agent would need to remember its entire history to make optimal decisions — making learning
computationally infeasible. This chapter explores what the Markov property means, why it
matters, and how to design state representations that satisfy it.

---

## 1. The Memoryless Property

### Formal Definition

A state $S_t$ has the **Markov property** if and only if:

$$P(S_{t+1} | S_t) = P(S_{t+1} | S_1, S_2, \ldots, S_t)$$

In words: *the probability of transitioning to the next state depends only on the current
state, not on the sequence of states that preceded it.*

### Conditional Independence

Equivalently, given $S_t$, the future $(S_{t+1}, S_{t+2}, \ldots)$ is conditionally
independent of the past $(S_1, S_2, \ldots, S_{t-1})$:

$$S_{t+1} \perp (S_1, \ldots, S_{t-1}) \mid S_t$$

The current state $S_t$ is a **sufficient statistic** for the history. It captures all the
information from the past that is relevant for predicting the future.

### Intuition

Think of it this way:

```
Past               Present         Future
┌──────────────┐   ┌────────┐   ┌──────────────┐
│ s_1, s_2 ... │──▶│  s_t   │──▶│ s_{t+1}, ... │
│  s_{t-1}     │   │        │   │              │
└──────────────┘   └────────┘   └──────────────┘
        │                ▲              │
        │                │              │
        └── NOT needed ──┘   depends ONLY on s_t
```

If you know the current state, knowing how you got there adds no extra predictive power.

---

## 2. Why the Markov Property Matters for RL

| Without Markov Property | With Markov Property |
|---|---|
| Agent must store entire history | Agent only needs current state |
| State space grows exponentially with time | State space is fixed |
| Value functions depend on trajectory | Value functions depend on state alone |
| Learning is intractable | Bellman equations apply |

The Markov property enables:

1. **Bellman equations** — recursive decomposition of value functions
2. **Dynamic programming** — solving MDPs via backward induction
3. **Temporal-difference learning** — bootstrapping from successor states
4. **Compact policies** — mapping states to actions without history

Without the Markov property, the entire theoretical foundation of RL collapses.

---

## 3. Markov Chains

A **Markov chain** is a sequence of random variables $(S_0, S_1, S_2, \ldots)$ where each
state transition satisfies the Markov property.

### Definition

A Markov chain is defined by:
- A finite set of states $S = \{s_1, s_2, \ldots, s_n\}$
- A **transition probability matrix** $P$ where $P_{ij} = P(S_{t+1} = s_j \mid S_t = s_i)$
- An initial distribution $\mu_0$ over states

### Properties of the Transition Matrix

Each row of $P$ sums to 1 (it is a **stochastic matrix**):

$$\sum_{j=1}^{n} P_{ij} = 1 \quad \forall i$$

### ASCII Diagram: A Simple Markov Chain (Weather Model)

```
                0.7
          ┌───────────┐
          ▼           │
       ┌──────┐  0.3  │   ┌──────┐
       │ Sunny│───────────▶│Rainy │
       └──────┘            └──────┘
          ▲      0.4          │
          └───────────────────┘
                              │
                         0.6  │
                    ┌─────────┘
                    ▼
               (self-loop)

    Transition Matrix P:
    ┌              ┐
    │  0.7    0.3  │   ← From Sunny
    │  0.4    0.6  │   ← From Rainy
    └              ┘
      Sunny  Rainy
```

### Multi-Step Transitions

The probability of being in state $s_j$ after $k$ steps, starting from $s_i$:

$$P(S_{t+k} = s_j \mid S_t = s_i) = (P^k)_{ij}$$

where $P^k$ is the $k$-th matrix power of $P$.

---

## 4. Examples: Markov vs Non-Markov Systems

### Markov Systems

| System | State | Why Markov? |
|---|---|---|
| Chess | Board position + whose turn | All info needed is visible on the board |
| Thermostat | Current temperature | Next temp depends only on current temp + action |
| Inventory | Current stock level | Future demand is independent of past stock path |

### Non-Markov Systems

| System | Naive State | Why NOT Markov? | Fix |
|---|---|---|---|
| Blackjack (card counting) | Current hand | Future depends on cards already dealt | Include dealt-card counts in state |
| Stock market | Current price | Past price trends affect future movement | Add price history / technical indicators |
| Bouncing ball | Position only | Need velocity to predict trajectory | Add velocity to state: $(x, \dot{x})$ |

### The Bouncing Ball Example

```
Non-Markov state (position only):         Markov state (position + velocity):

  Same position, different futures:         State fully determines future:

  ●───▶  (moving right)                    (x=3, v=+2)  ───▶  predictable
  ●◀───  (moving left)                     (x=3, v=-2)  ◀───  predictable

  s_t = x = 3 in both cases!               Different states, different futures ✓
  Cannot predict s_{t+1} from s_t alone!
```

---

## 5. State Representation to Achieve the Markov Property

### The Key Insight

The Markov property is not a property of the underlying system — it is a property of the
**state representation**. Almost any system can be made Markov with the right state design.

### Strategies for Markov State Design

1. **Include velocity / derivatives**: If position alone isn't Markov, add velocity
2. **Stack observations**: Use the last $k$ observations as the state
   - Atari games: stack last 4 frames → captures motion
3. **Add memory variables**: Include counters, flags, or summaries of history
4. **Use recurrent models**: Let an RNN/LSTM learn a Markov state from history

### Frame Stacking (Atari Example)

```
 Frame t-3    Frame t-2    Frame t-1    Frame t
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│         │ │    ●    │ │      ●  │ │        ●│
│         │ │         │ │         │ │         │
│  ▬▬▬    │ │  ▬▬▬    │ │  ▬▬▬    │ │   ▬▬▬  │
└─────────┘ └─────────┘ └─────────┘ └─────────┘
     │           │           │           │
     └───────────┴───────────┴───────────┘
                     │
              ┌──────┴──────┐
              │ Stacked     │
              │ State s_t   │  ← Now Markov!
              │ (4 frames)  │  Ball direction is
              └─────────────┘  captured by the stack
```

---

## 6. Mathematical Foundations

### First-Order Markov Property

$$P(S_{t+1} = s' \mid S_t = s_t, S_{t-1} = s_{t-1}, \ldots, S_0 = s_0) = P(S_{t+1} = s' \mid S_t = s_t)$$

### k-th Order Markov Property

A process is **k-th order Markov** if:

$$P(S_{t+1} \mid S_t, S_{t-1}, \ldots, S_0) = P(S_{t+1} \mid S_t, S_{t-1}, \ldots, S_{t-k+1})$$

Any k-th order Markov process can be converted to a first-order Markov process by
redefining the state as a tuple of the last $k$ observations:

$$\tilde{S}_t = (S_t, S_{t-1}, \ldots, S_{t-k+1})$$

### Stationary Distribution

A Markov chain has a **stationary distribution** $\pi$ if:

$$\pi = \pi P$$

where $\pi$ is a row vector. Under mild conditions (irreducibility and aperiodicity),
$\pi$ is unique and the chain converges to it regardless of the starting state.

---

## 7. Python Example: Demonstrating the Markov Property

```python
import numpy as np

class MarkovChain:
    """Simple Markov chain demonstrating the memoryless property."""

    def __init__(self, transition_matrix, state_names=None):
        self.P = np.array(transition_matrix)
        n = self.P.shape[0]
        self.state_names = state_names or [f"s{i}" for i in range(n)]

        # Validate: rows must sum to 1
        assert np.allclose(self.P.sum(axis=1), 1.0), "Rows must sum to 1"

    def step(self, current_state):
        """Transition to next state (Markov: depends ONLY on current_state)."""
        probs = self.P[current_state]
        next_state = np.random.choice(len(self.P), p=probs)
        return next_state

    def simulate(self, start_state, n_steps):
        """Generate a trajectory from the Markov chain."""
        trajectory = [start_state]
        state = start_state
        for _ in range(n_steps):
            state = self.step(state)
            trajectory.append(state)
        return trajectory

    def multi_step_transition(self, k):
        """Compute k-step transition matrix P^k."""
        return np.linalg.matrix_power(self.P, k)

    def stationary_distribution(self):
        """Compute the stationary distribution via eigendecomposition."""
        eigenvalues, eigenvectors = np.linalg.eig(self.P.T)
        # Find eigenvector for eigenvalue = 1
        idx = np.argmin(np.abs(eigenvalues - 1.0))
        stationary = np.real(eigenvectors[:, idx])
        stationary = stationary / stationary.sum()  # normalize
        return stationary


# --- Weather Model Example ---
# States: 0 = Sunny, 1 = Rainy
weather_chain = MarkovChain(
    transition_matrix=[
        [0.7, 0.3],  # Sunny → Sunny (0.7), Sunny → Rainy (0.3)
        [0.4, 0.6],  # Rainy → Sunny (0.4), Rainy → Rainy (0.6)
    ],
    state_names=["Sunny", "Rainy"]
)

# Demonstrate: future depends only on current state
np.random.seed(42)
print("=== Markov Property Demonstration ===\n")

# Run many trajectories starting from Sunny
n_trials = 10000
next_states_from_sunny = []
for _ in range(n_trials):
    next_state = weather_chain.step(0)  # 0 = Sunny
    next_states_from_sunny.append(next_state)

sunny_count = next_states_from_sunny.count(0)
rainy_count = next_states_from_sunny.count(1)
print(f"From Sunny → Sunny: {sunny_count/n_trials:.3f} (expected 0.700)")
print(f"From Sunny → Rainy: {rainy_count/n_trials:.3f} (expected 0.300)")

# Multi-step transitions
print("\n=== Multi-Step Transition Probabilities ===")
for k in [1, 2, 5, 10, 50]:
    P_k = weather_chain.multi_step_transition(k)
    print(f"\nP^{k}:")
    print(f"  {P_k}")

# Stationary distribution
pi = weather_chain.stationary_distribution()
print(f"\nStationary distribution: Sunny={pi[0]:.4f}, Rainy={pi[1]:.4f}")
print(f"Analytical solution:     Sunny={4/7:.4f}, Rainy={3/7:.4f}")


# --- Demonstrating Non-Markov → Markov via state augmentation ---
print("\n=== Non-Markov to Markov State Augmentation ===")

class BouncingBall:
    """Ball that bounces between 0 and 10."""

    def __init__(self, position=5.0, velocity=1.0):
        self.x = position
        self.v = velocity

    def step(self, dt=1.0):
        """Non-Markov if state = position only; Markov if state = (pos, vel)."""
        self.x += self.v * dt
        # Bounce off walls
        if self.x >= 10:
            self.x = 10 - (self.x - 10)
            self.v = -self.v
        elif self.x <= 0:
            self.x = -self.x
            self.v = -self.v
        return (self.x, self.v)  # Full Markov state

ball = BouncingBall(position=5.0, velocity=2.0)
print("Time | Position | Velocity | Full State (Markov)")
print("─" * 50)
for t in range(8):
    x, v = ball.step()
    print(f"  {t+1}  |  {x:6.2f}  |  {v:+5.2f}   | ({x:.2f}, {v:+.2f})")
```

---

## 8. Real-World Applications

| Domain | Markov State Design | Key Variables |
|---|---|---|
| **Robotics** | Joint angles + velocities | $(q, \dot{q})$ |
| **Game AI** | Board/screen state (possibly stacked) | Pixel frames, game memory |
| **Finance** | Price + volume + indicators | OHLCV data window |
| **Healthcare** | Patient vitals + lab results + history | Current clinical state |
| **Autonomous Driving** | Sensor fusion: LIDAR + camera + velocity | Ego state + environment map |

---

## Summary

| Concept | Key Point |
|---|---|
| Markov property | Future depends only on present, not past |
| Formal definition | $P(S_{t+1} \mid S_t) = P(S_{t+1} \mid S_1, \ldots, S_t)$ |
| Markov chain | Sequence of states with Markov transitions |
| Transition matrix | Row-stochastic matrix encoding $P(s' \mid s)$ |
| Sufficient statistic | Current state captures all relevant history |
| State augmentation | Add velocity, stack frames, or use memory to make states Markov |
| Stationary distribution | Long-run probability of being in each state |

---

## Revision Questions

1. **Define the Markov property in your own words.** Why is it also called the "memoryless" property?

2. **Give an example of a system that is NOT Markov with position-only state.** How would you augment the state to make it Markov?

3. **Given the weather transition matrix** $P = \begin{bmatrix} 0.7 & 0.3 \\ 0.4 & 0.6 \end{bmatrix}$, compute $P^2$ and interpret the entries.

4. **What is the stationary distribution** of the weather Markov chain above? Verify by solving $\pi P = \pi$.

5. **In Atari game playing**, why do we stack the last 4 frames instead of using a single frame? What property does frame stacking help achieve?

6. **Explain the difference between a first-order and k-th order Markov process.** How can a k-th order process be converted to first-order?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [RL Applications](../01-Introduction-to-RL/05-rl-applications.md) | [Unit 2: MDPs](./README.md) | [MDP Components](./02-mdp-components.md) |

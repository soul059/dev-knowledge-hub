# Chapter 1: TD(0) Prediction

> **Unit 5: Temporal Difference Learning** — *Learning value functions one step at a time, without waiting for episode end.*

---

## Overview

Temporal Difference (TD) learning is one of the most central and novel ideas in reinforcement learning. TD methods combine the strengths of **Monte Carlo (MC)** methods — learning directly from experience without a model — with the strengths of **Dynamic Programming (DP)** — updating estimates based on other learned estimates (bootstrapping). TD(0), the simplest form, updates value estimates after every single transition rather than waiting for an episode to complete.

```
╔══════════════════════════════════════════════════════════════════════╗
║                     TD(0) PREDICTION AT A GLANCE                   ║
╠══════════════════════════════════════════════════════════════════════╣
║                                                                    ║
║   Given: policy π to evaluate                                      ║
║                                                                    ║
║   Goal:  learn V(s) ≈ v_π(s) from experience under π               ║
║                                                                    ║
║   Idea:  after each transition S_t → S_{t+1}, update:              ║
║                                                                    ║
║          V(S_t) ← V(S_t) + α [ R_{t+1} + γV(S_{t+1}) − V(S_t) ]  ║
║                                  ───────────────────               ║
║                                      TD target                     ║
║                                  ─────────────────────────────     ║
║                                          TD error (δ_t)            ║
║                                                                    ║
║   Key:   bootstraps — uses V(S_{t+1}) instead of waiting for G_t   ║
║                                                                    ║
╚══════════════════════════════════════════════════════════════════════╝
```

Unlike MC methods that must wait until the end of an episode to compute the actual return $G_t$, TD(0) updates the value function after **every single time step**. This one-step lookahead, combined with the current estimate of the next state's value, is what gives TD methods their remarkable efficiency and flexibility.

---

## 1. Motivation — Why Not Just Use Monte Carlo?

Monte Carlo prediction estimates $V(s)$ by averaging the actual returns $G_t$ observed after visiting state $s$. While simple and unbiased, MC has significant limitations:

1. **Must wait for episode termination** — cannot learn online or from incomplete episodes
2. **High variance** — actual returns $G_t$ can vary wildly across episodes
3. **Inapplicable to continuing tasks** — tasks that never terminate have no episode end

> **Key Insight:** TD methods fix all three problems by replacing the actual return $G_t$ with a one-step estimated return called the **TD target**.

| Property | Monte Carlo | TD(0) |
|---|---|---|
| Update timing | End of episode | Every time step |
| Target | Actual return $G_t$ | Estimated return $R_{t+1} + \gamma V(S_{t+1})$ |
| Bias | Unbiased | Biased (bootstraps) |
| Variance | High | Lower |
| Continuing tasks | ✗ Cannot handle | ✓ Works naturally |
| Requires episodes | Yes | No |

---

## 2. The TD(0) Update Rule

The core of TD(0) prediction is a single, elegant update equation. After each transition $(S_t, A_t, R_{t+1}, S_{t+1})$ under policy $\pi$:

$$V(S_t) \leftarrow V(S_t) + \alpha \Big[ R_{t+1} + \gamma V(S_{t+1}) - V(S_t) \Big]$$

where:
- $V(S_t)$ is the current estimate of the value of state $S_t$
- $\alpha \in (0, 1]$ is the **step size** (learning rate)
- $R_{t+1}$ is the **immediate reward** received after transitioning
- $\gamma \in [0, 1]$ is the **discount factor**
- $V(S_{t+1})$ is the current estimate of the next state's value

### 2.1 Decomposing the Update

The update has three critical components:

**TD Target** — the estimated return:

$$\text{TD Target} = R_{t+1} + \gamma V(S_{t+1})$$

This replaces the full return $G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots$ used by MC. Instead of waiting for all future rewards, we use the immediate reward plus the discounted estimated value of the next state.

**TD Error** — the temporal difference:

$$\delta_t = R_{t+1} + \gamma V(S_{t+1}) - V(S_t)$$

The TD error $\delta_t$ measures the difference between the TD target (a better estimate) and the current estimate $V(S_t)$. If $\delta_t > 0$, the actual transition was **better** than expected; if $\delta_t < 0$, it was **worse**.

**Update Direction** — the step size controls learning speed:

$$V(S_t) \leftarrow V(S_t) + \alpha \cdot \delta_t$$

> **Key Insight:** The TD error acts as a **surprise signal**. When the agent is surprised (large $|\delta_t|$), it makes a large update. When the transition matches expectations ($\delta_t \approx 0$), the value estimate barely changes.

---

## 3. Bootstrapping — The Key Idea

**Bootstrapping** means updating an estimate based on another estimate, rather than waiting for the true value. In TD(0), we use our current guess $V(S_{t+1})$ as a stand-in for the true value $v_\pi(S_{t+1})$.

```
┌───────────────────────────────────────────────────────────────────┐
│                  BOOTSTRAPPING IN TD(0)                          │
│                                                                  │
│   True return:    G_t = R_{t+1} + γR_{t+2} + γ²R_{t+3} + ...   │
│                         ├──────── unknown future ────────┤       │
│                                                                  │
│   MC estimate:    Uses actual G_t  (waits until episode ends)    │
│                                                                  │
│   TD estimate:    R_{t+1} + γV(S_{t+1})                         │
│                   ├─known─┤  ├─ bootstrap ─┤                     │
│                                                                  │
│   The "bootstrap" replaces the entire unknown future with        │
│   our current estimate V(S_{t+1}).                               │
└───────────────────────────────────────────────────────────────────┘
```

This creates a **bias-variance tradeoff**:
- **Bias**: $V(S_{t+1})$ may be wrong, so the TD target is a biased estimate of $G_t$
- **Lower variance**: the TD target depends on only **one** random transition $(R_{t+1}, S_{t+1})$, whereas $G_t$ depends on the entire sequence of future actions and transitions

---

## 4. Backup Diagrams — TD(0) vs Monte Carlo

A **backup diagram** shows which future states and rewards are used to update the current state's value estimate. This is one of the most instructive ways to compare algorithms.

```
        TD(0) Backup                    MC Backup
        ────────────                    ─────────

          S_t                             S_t
           │                               │
           │ R_{t+1}                       │ R_{t+1}
           │                               │
           ▼                               ▼
         (S_{t+1})                        S_{t+1}
           ●                               │
       [bootstrap]                         │ R_{t+2}
                                           │
     Only ONE step deep!                   ▼
                                          S_{t+2}
     Target:                               │
     R_{t+1} + γV(S_{t+1})                │ R_{t+3}
                                           │
                                           ▼
                                          S_{t+3}
                                           │
                                           ⋮
                                           │
                                           ▼
                                        Terminal
                                           ●

                                     ALL steps to end!

                                     Target: G_t
```

The TD(0) backup diagram has a distinctive feature: it terminates at the **filled circle** (●), which denotes a bootstrapped value estimate rather than a true terminal value. This single-step backup is what makes TD(0) the simplest member of the TD family.

| Feature | TD(0) Backup | MC Backup |
|---|---|---|
| Depth | 1 step | Full episode |
| Width | 1 successor | 1 trajectory |
| Uses estimate? | Yes (bootstraps $V(S_{t+1})$) | No (uses actual $G_t$) |
| Requires termination? | No | Yes |

---

## 5. TD(0) Prediction Algorithm

```
Algorithm: Tabular TD(0) Prediction (for estimating v_π)
─────────────────────────────────────────────────────────────────
Input : policy π to evaluate
         step size α ∈ (0, 1]
         discount factor γ ∈ [0, 1]
Output: value function V ≈ v_π

1.  Initialize V(s) arbitrarily for all s ∈ S
    Initialize V(terminal) = 0

2.  REPEAT (for each episode):
3.      Initialize S (starting state of episode)

4.      REPEAT (for each step of episode):
5.          A ← action given by π for S
6.          Take action A, observe R, S'

7.          // Compute TD error
8.          δ ← R + γ · V(S') − V(S)

9.          // Update value estimate
10.         V(S) ← V(S) + α · δ

11.         S ← S'

12.     UNTIL S is terminal

13. UNTIL convergence or max episodes reached
```

> **Note:** When $S'$ is a terminal state, $V(S') = 0$ by definition, so the TD target reduces to just $R$.

---

## 6. Worked Numerical Example

Consider a simple 5-state random walk (states A through E) with terminal states on each end:

```
┌────────────────────────────────────────────────────────┐
│                   5-State Random Walk                  │
│                                                        │
│   [Left]  ◄──  A  ──  B  ──  C  ──  D  ──  E  ──► [Right]  │
│   Terminal     ←─── equal probability ───→     Terminal│
│   r = 0                                       r = +1  │
│                                                        │
│   All non-terminal transitions give r = 0              │
│   except the rightmost transition which gives r = +1   │
│   γ = 1 (undiscounted)                                 │
└────────────────────────────────────────────────────────┘
```

**True values** under the uniform random policy (equal probability left/right):

$$v_\pi(A) = \frac{1}{6}, \quad v_\pi(B) = \frac{2}{6}, \quad v_\pi(C) = \frac{3}{6}, \quad v_\pi(D) = \frac{4}{6}, \quad v_\pi(E) = \frac{5}{6}$$

### Step-by-Step TD(0) Updates

**Setup:** $\alpha = 0.1$, $\gamma = 1.0$, initialize all $V(s) = 0.5$

**Episode 1:** Suppose the agent starts at C and the trajectory is: C → D → E → [Right Terminal]

| Step | $S_t$ | $R_{t+1}$ | $S_{t+1}$ | $V(S_{t+1})$ | TD Target | $\delta_t$ | Update | New $V(S_t)$ |
|------|--------|-----------|-----------|--------------|-----------|-----------|--------|--------------|
| 1 | C | 0 | D | 0.5 | 0 + 1.0 × 0.5 = 0.5 | 0.5 − 0.5 = 0.0 | 0.5 + 0.1 × 0.0 = 0.5 | 0.5 |
| 2 | D | 0 | E | 0.5 | 0 + 1.0 × 0.5 = 0.5 | 0.5 − 0.5 = 0.0 | 0.5 + 0.1 × 0.0 = 0.5 | 0.5 |
| 3 | E | 1 | Terminal | 0.0 | 1 + 1.0 × 0.0 = 1.0 | 1.0 − 0.5 = 0.5 | 0.5 + 0.1 × 0.5 = 0.55 | **0.55** |

After episode 1: $V = [0.5, 0.5, 0.5, 0.5, 0.55]$ — only $V(E)$ changed!

**Episode 2:** Start at C, trajectory: C → B → A → [Left Terminal]

| Step | $S_t$ | $R_{t+1}$ | $S_{t+1}$ | $V(S_{t+1})$ | TD Target | $\delta_t$ | Update | New $V(S_t)$ |
|------|--------|-----------|-----------|--------------|-----------|-----------|--------|--------------|
| 1 | C | 0 | B | 0.5 | 0 + 1.0 × 0.5 = 0.5 | 0.5 − 0.5 = 0.0 | 0.5 + 0.1 × 0.0 = 0.5 | 0.5 |
| 2 | B | 0 | A | 0.5 | 0 + 1.0 × 0.5 = 0.5 | 0.5 − 0.5 = 0.0 | 0.5 + 0.1 × 0.0 = 0.5 | 0.5 |
| 3 | A | 0 | Terminal | 0.0 | 0 + 1.0 × 0.0 = 0.0 | 0.0 − 0.5 = −0.5 | 0.5 + 0.1 × (−0.5) = 0.45 | **0.45** |

After episode 2: $V = [0.45, 0.5, 0.5, 0.5, 0.55]$

**Episode 3:** Start at C, trajectory: C → D → E → [Right Terminal]

| Step | $S_t$ | $R_{t+1}$ | $S_{t+1}$ | $V(S_{t+1})$ | TD Target | $\delta_t$ | Update | New $V(S_t)$ |
|------|--------|-----------|-----------|--------------|-----------|-----------|--------|--------------|
| 1 | C | 0 | D | 0.5 | 0.5 | 0.0 | 0.5 + 0.0 = 0.5 | 0.5 |
| 2 | D | 0 | E | 0.55 | 0.55 | 0.05 | 0.5 + 0.1 × 0.05 = 0.505 | **0.505** |
| 3 | E | 1 | Terminal | 0.0 | 1.0 | 0.45 | 0.55 + 0.1 × 0.45 = 0.595 | **0.595** |

After episode 3: $V = [0.45, 0.5, 0.5, 0.505, 0.595]$

> **Key Insight:** Notice how the updated value of $V(E) = 0.55$ from episode 1 now **propagates backward** to update $V(D)$ in episode 3. This is bootstrapping in action — information flows backward through the chain of states over successive episodes.

---

## 7. Convergence Properties

TD(0) converges to the true value function $v_\pi$ under appropriate conditions.

### 7.1 Robbins-Monro Conditions

For convergence with probability 1, the step-size sequence $\{\alpha_n\}$ must satisfy:

$$\sum_{n=1}^{\infty} \alpha_n = \infty \qquad \text{and} \qquad \sum_{n=1}^{\infty} \alpha_n^2 < \infty$$

The first condition ensures the steps are large enough to overcome any initial conditions or random fluctuations. The second condition ensures the steps become small enough for convergence. A common schedule satisfying both is $\alpha_n = 1/n$.

In practice, a **constant step size** $\alpha$ is often used. This does not satisfy the second condition, so the estimates will never fully converge — instead, they continue to fluctuate. However, this is often desirable in non-stationary environments because it allows the agent to track changes.

### 7.2 Certainty Equivalence

An important theoretical result: TD(0) with a constant step size converges (in the mean) to the **certainty-equivalence estimate** — the maximum likelihood estimate of $v_\pi$ given the observed data, assuming the MDP model estimated from experience is correct.

$$V_{TD} \rightarrow V_{CE} = \arg\min_V \sum_{t} \Big( V(S_t) - R_{t+1} - \gamma V(S_{t+1}) \Big)^2$$

This is a stronger result than MC convergence. MC converges to estimates that minimize mean squared error from observed returns, while TD converges to estimates consistent with the **Markov structure** of the problem.

### 7.3 Batch TD(0) vs Batch MC

When given a finite batch of experience and repeatedly trained to convergence:

| Method | Converges To | Leverages Markov Property? |
|---|---|---|
| Batch MC | Minimum MSE w.r.t. observed returns | No |
| Batch TD(0) | Certainty-equivalence (ML) estimate | **Yes** |

> **Key Insight:** TD(0) exploits the Markov property to produce better value estimates from limited data. This is why TD often learns faster than MC in Markov environments.

---

## 8. Advantages and Disadvantages of TD(0)

### Advantages

1. **Online learning** — updates after every step, no need to wait for episode end
2. **Works with continuing tasks** — does not require episodes to terminate
3. **Lower variance** — TD target depends on one transition, not an entire trajectory
4. **Faster propagation** — information about value flows backward through bootstrapping
5. **Exploits Markov property** — converges to certainty-equivalence estimate
6. **Naturally incremental** — constant per-step computation and memory

### Disadvantages

1. **Biased estimates** — bootstrapping introduces bias since $V(S_{t+1})$ is an estimate
2. **Sensitive to initialization** — poor initial values can slow convergence
3. **Dependent on step size** — too large causes oscillation, too small causes slow learning
4. **Only one step of lookahead** — limited credit assignment per update
5. **Harder to analyze** — convergence proofs are more complex than for MC

---

## 9. The Bias-Variance Tradeoff

The fundamental difference between MC and TD targets creates a bias-variance tradeoff:

$$\underbrace{G_t}_{\text{MC target}} = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots$$

$$\underbrace{R_{t+1} + \gamma V(S_{t+1})}_{\text{TD target}}$$

**MC target ($G_t$)** is an **unbiased** estimate of $v_\pi(S_t)$ because:

$$\mathbb{E}_\pi[G_t \mid S_t = s] = v_\pi(s)$$

**TD target** is a **biased** estimate because $V(S_{t+1}) \neq v_\pi(S_{t+1})$ in general. However:

$$\text{Var}[\text{TD target}] \ll \text{Var}[G_t]$$

The MC return $G_t$ depends on many random variables — every action, transition, and reward for the rest of the episode. The TD target depends on only **one** random action, transition, and reward. This dramatically lower variance often means TD learns faster despite the bias.

```
┌──────────────────────────────────────────────────────────────┐
│              BIAS-VARIANCE TRADEOFF                          │
│                                                              │
│   High Variance, No Bias          Low Variance, Some Bias   │
│   ┌─────────────────────┐         ┌─────────────────────┐   │
│   │                     │         │                     │   │
│   │    ×   ×            │         │        ×  ×         │   │
│   │         ×           │         │       ×  ×          │   │
│   │  ×          ×       │         │        ●×           │   │
│   │       ●             │         │       ×  ×          │   │
│   │            ×        │         │        ×            │   │
│   │   ×       ×         │         │                     │   │
│   │                     │         │                     │   │
│   └─────────────────────┘         └─────────────────────┘   │
│    MC: scattered around            TD: clustered but         │
│    true value ●                    slightly off-center ●     │
└──────────────────────────────────────────────────────────────┘
```

---

## 10. Relationship to Dynamic Programming

TD(0) can be viewed as a **sample-based version of DP**. Compare the DP update with the TD update:

**DP (full Bellman backup):**

$$V(s) \leftarrow \sum_{a} \pi(a|s) \sum_{s', r} p(s', r \mid s, a) \Big[ r + \gamma V(s') \Big]$$

**TD(0) (sample-based backup):**

$$V(S_t) \leftarrow V(S_t) + \alpha \Big[ R_{t+1} + \gamma V(S_{t+1}) - V(S_t) \Big]$$

DP requires the complete model $p(s', r \mid s, a)$ and sweeps over **all** states. TD(0) replaces the expectation over all transitions with a **single sample** from actual experience. This makes TD model-free while retaining the bootstrapping advantage of DP.

| Feature | DP | TD(0) | MC |
|---|---|---|---|
| Requires model? | Yes | No | No |
| Bootstraps? | Yes | Yes | No |
| Sample-based? | No | Yes | Yes |
| Full/sample backup | Full | Sample | Sample |

---

## 11. Python Implementation — Random Walk & FrozenLake

Below is a complete implementation of TD(0) on the classic 5-state random walk, followed by a Gymnasium FrozenLake example.

```python
import numpy as np
import matplotlib.pyplot as plt

class RandomWalkEnv:
    """5-state random walk (A..E). Left terminal r=0, right terminal r=+1."""
    def __init__(self, n_states=5):
        self.n_states = n_states
        self.state = None

    def reset(self):
        self.state = self.n_states // 2
        return self.state

    def step(self):
        self.state += np.random.choice([-1, 1])
        if self.state < 0:
            return self.state, 0.0, True
        elif self.state >= self.n_states:
            return self.state, 1.0, True
        return self.state, 0.0, False


def td0_prediction(env, n_episodes=100, alpha=0.1, gamma=1.0):
    """Tabular TD(0) prediction. Returns learned V and snapshot history."""
    V = np.full(env.n_states, 0.5)
    history = [V.copy()]

    for _ in range(n_episodes):
        state = env.reset()
        while True:
            next_state, reward, done = env.step()
            td_target = reward if done else reward + gamma * V[next_state]
            V[state] += alpha * (td_target - V[state])
            if done:
                break
            state = next_state
        history.append(V.copy())
    return V, history


# --- Run and display results ---
env = RandomWalkEnv(n_states=5)
true_values = np.array([1/6, 2/6, 3/6, 4/6, 5/6])
V_learned, history = td0_prediction(env, n_episodes=100, alpha=0.1)

state_labels = ['A', 'B', 'C', 'D', 'E']
print("State | True Value | Learned Value | Error")
print("------+------------+---------------+------")
for i, label in enumerate(state_labels):
    print(f"  {label}   |   {true_values[i]:.4f}   |    {V_learned[i]:.4f}    | "
          f"{abs(true_values[i] - V_learned[i]):.4f}")
```

### FrozenLake with Gymnasium

```python
import gymnasium as gym
import numpy as np

def td0_frozenlake(n_episodes=10000, alpha=0.01, gamma=0.99):
    """TD(0) prediction on FrozenLake-v1 with a random policy."""
    env = gym.make('FrozenLake-v1', is_slippery=True)
    V = np.zeros(env.observation_space.n)

    for _ in range(n_episodes):
        state, _ = env.reset()
        while True:
            action = env.action_space.sample()
            next_state, reward, terminated, truncated, _ = env.step(action)
            td_target = reward if terminated else reward + gamma * V[next_state]
            V[state] += alpha * (td_target - V[state])
            if terminated or truncated:
                break
            state = next_state
    env.close()
    return V

V = td0_frozenlake()
print("Learned Value Function (4×4 grid):")
print(np.round(V.reshape(4, 4), 4))
```

---

## 12. Why TD(0) Works — Bellman Connection

The true value function satisfies the **Bellman equation**:

$$v_\pi(s) = \mathbb{E}_\pi \Big[ R_{t+1} + \gamma \, v_\pi(S_{t+1}) \mid S_t = s \Big]$$

The TD target $R_{t+1} + \gamma V(S_{t+1})$ is a **sample** from the right-hand side (with $v_\pi$ replaced by our estimate $V$). TD(0) performs **stochastic approximation** to find the fixed point where the expected TD error is zero:

$$\mathbb{E}_\pi[\delta_t \mid S_t = s] = \mathbb{E}_\pi\Big[R_{t+1} + \gamma V(S_{t+1}) - V(S_t) \mid S_t = s\Big] = 0$$

This is precisely the Bellman equation — confirming that the fixed point of TD(0) is $v_\pi$.

---

## 13. Where TD(0) Sits in the RL Landscape

```
┌─────────────────────────────────────────────────────────────┐
│              REINFORCEMENT LEARNING METHODS                 │
│                                                             │
│                   Bootstrapping?                            │
│                   Yes            No                         │
│              ┌────────────┬────────────┐                    │
│   Requires   │            │            │                    │
│   Model?     │     DP     │  Exhaustive│                    │
│   Yes        │            │   Search   │                    │
│              ├────────────┼────────────┤                    │
│              │            │            │                    │
│   No         │  ★ TD(0)   │    MC      │                    │
│              │   SARSA    │            │                    │
│              │   Q-learn  │            │                    │
│              └────────────┴────────────┘                    │
│                                                             │
│   TD methods = sample-based + bootstrapping (best of both)  │
└─────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Key Point |
|---|---|
| **TD(0) Update** | $V(S_t) \leftarrow V(S_t) + \alpha[\,R_{t+1} + \gamma V(S_{t+1}) - V(S_t)\,]$ |
| **TD Error** | $\delta_t = R_{t+1} + \gamma V(S_{t+1}) - V(S_t)$ — the surprise signal |
| **TD Target** | $R_{t+1} + \gamma V(S_{t+1})$ — one-step estimated return |
| **Bootstrapping** | Uses current estimate $V(S')$ instead of waiting for true return $G_t$ |
| **MC vs TD Bias** | MC is unbiased but high variance; TD is biased but lower variance |
| **Online Learning** | TD updates every step; MC must wait for episode end |
| **Continuing Tasks** | TD works naturally; MC cannot handle non-episodic settings |
| **Convergence** | Converges to $v_\pi$ under Robbins-Monro step-size conditions |
| **Certainty Equiv.** | Batch TD converges to the maximum likelihood (Markov-consistent) estimate |
| **Backup Diagram** | One step deep — single transition with bootstrapped successor value |
| **Relation to DP** | TD(0) is a sample-based, model-free version of the DP Bellman backup |

---

## Revision Questions

1. **Write out the TD(0) update rule and identify the TD target and TD error within it.** Explain what each component represents and why the TD error can be interpreted as a surprise signal.

2. **What is bootstrapping, and why does it introduce bias?** Compare the MC target $G_t$ and the TD target $R_{t+1} + \gamma V(S_{t+1})$ in terms of bias and variance.

3. **Draw the backup diagrams for TD(0) and Monte Carlo side by side.** Explain how the depth and width of each diagram reflect the computational and data requirements of each method.

4. **State the Robbins-Monro conditions for step-size convergence.** Why is a constant step size $\alpha$ commonly used in practice despite violating these conditions?

5. **In the random walk example, explain how the updated value of $V(E)$ propagates backward to influence $V(D)$ in subsequent episodes.** Why does this demonstrate an advantage of TD over MC?

6. **What is the certainty-equivalence estimate, and why does TD(0) converge to it while MC does not?** What property of the environment does TD exploit that MC ignores?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Every-Visit vs First-Visit](../04-Monte-Carlo-Methods/05-every-visit-vs-first-visit.md) | [Unit 5: TD Learning](./README.md) | [SARSA](./02-sarsa.md) |

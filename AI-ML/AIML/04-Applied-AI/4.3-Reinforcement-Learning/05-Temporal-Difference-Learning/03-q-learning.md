# Chapter 3: Q-Learning — Off-Policy TD Control

> **Unit 5: Temporal Difference Learning** — *The most important breakthrough in RL: learning the optimal policy directly.*

---

## Overview

Q-Learning is arguably the most celebrated algorithm in reinforcement learning. Introduced by Chris Watkins in his 1989 PhD thesis, it was the first algorithm proven to converge directly to the **optimal** action-value function $q_*$ regardless of the policy being followed — making it an **off-policy** method. While SARSA updates toward the value of the action the agent *actually takes* next, Q-Learning updates toward the value of the *best possible* action, effectively separating exploration from the learned policy.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                       Q-LEARNING AT A GLANCE                           ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║   Given: nothing — learns while interacting                            ║
║                                                                        ║
║   Goal:  learn Q(s, a) ≈ q_*(s, a) directly (the optimal values)      ║
║                                                                        ║
║   Each transition provides:                                            ║
║                                                                        ║
║          S_t ──(A_t)──► R_{t+1}, S_{t+1}                               ║
║                                                                        ║
║   Update:                                                              ║
║          Q(S_t, A_t) ← Q(S_t, A_t)                                    ║
║                        + α [ R_{t+1} + γ max_a Q(S_{t+1}, a)          ║
║                              − Q(S_t, A_t) ]                          ║
║                                       ▲                                ║
║                                  max over ALL actions                  ║
║                                                                        ║
║   Behavior: ε-greedy (explores)    Target: greedy (optimal)            ║
║                                                                        ║
╚══════════════════════════════════════════════════════════════════════════╝
```

The key difference from SARSA is a single word: **max**. Instead of using the Q-value of the action actually taken in the next state ($Q(S_{t+1}, A_{t+1})$), Q-Learning uses the maximum Q-value over all actions ($\max_a Q(S_{t+1}, a)$). This seemingly small change has profound consequences — it decouples the exploration strategy from the values being learned, enabling the agent to explore freely while still converging to the optimal policy.

---

## 1. From SARSA to Q-Learning — One Critical Change

Recall the SARSA update:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \, Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) \Big]$$

Here $A_{t+1}$ is the action **actually chosen** by the $\varepsilon$-greedy policy. Now consider Q-Learning:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a) - Q(S_t, A_t) \Big]$$

The difference is:
- **SARSA:** uses $Q(S_{t+1}, A_{t+1})$ where $A_{t+1} \sim \pi(\cdot \mid S_{t+1})$ — the action the agent **will** take
- **Q-Learning:** uses $\max_a Q(S_{t+1}, a)$ — the value of the **best** action, regardless of what the agent actually does

> **Key Insight:** The $\max$ operator makes Q-Learning update toward the value of the **greedy** (optimal) policy, even though the agent is following an exploratory $\varepsilon$-greedy behavior policy. This is what makes Q-Learning off-policy.

```
┌─────────────────────────────────────────────────────────────────────┐
│               THE CRITICAL DIFFERENCE                               │
│                                                                     │
│   SARSA:                                                            │
│     S_t ─(A_t)─► R_{t+1}, S_{t+1} ─(A_{t+1})─►                    │
│                                      ▲                              │
│                                      │                              │
│                              A_{t+1} ~ ε-greedy                    │
│                              (the action we WILL take)              │
│                                                                     │
│   Q-Learning:                                                       │
│     S_t ─(A_t)─► R_{t+1}, S_{t+1}                                  │
│                               ▲                                     │
│                               │                                     │
│                       max_a Q(S_{t+1}, a)                           │
│                       (the BEST action's value)                     │
│                       (not necessarily the one we take)              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Q-Learning Update Rule

After each transition $(S_t, A_t, R_{t+1}, S_{t+1})$, Q-Learning applies:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a) - Q(S_t, A_t) \Big]$$

where:
- $Q(S_t, A_t)$ is the current estimate of the action-value for state-action pair $(S_t, A_t)$
- $\alpha \in (0, 1]$ is the **step size** (learning rate)
- $R_{t+1}$ is the **immediate reward** received after taking action $A_t$ in state $S_t$
- $\gamma \in [0, 1]$ is the **discount factor**
- $\max_a Q(S_{t+1}, a)$ is the maximum action-value over **all** actions available in the next state $S_{t+1}$

### 2.1 Decomposing the Update

**TD Target** — the one-step estimated return under the greedy policy:

$$\text{TD Target} = R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a)$$

**TD Error** — the temporal difference signal:

$$\delta_t = R_{t+1} + \gamma \max_{a} Q(S_{t+1}, a) - Q(S_t, A_t)$$

**Compact form:**

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \, \delta_t$$

### 2.2 Note on Terminal States

When $S_{t+1}$ is a terminal state, the target simplifies to just the immediate reward:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} - Q(S_t, A_t) \Big]$$

In implementation, this is handled by multiplying the bootstrap term by $(1 - \text{terminated})$.

---

## 3. Off-Policy Learning — Two Separate Policies

Q-Learning maintains a crucial separation between two policies:

| Policy | Role | What It Does |
|---|---|---|
| **Behavior policy** $b$ | Generates experience | $\varepsilon$-greedy w.r.t. $Q$ — explores |
| **Target policy** $\pi$ | Policy being learned/evaluated | Greedy w.r.t. $Q$ — $\pi(s) = \arg\max_a Q(s, a)$ |

The behavior policy $b$ selects actions using $\varepsilon$-greedy to ensure exploration. The target policy $\pi$ is implicitly defined as the greedy policy with respect to the current Q-values. The $\max$ in the update rule is what evaluates the target policy $\pi$ without ever needing to follow it.

> **Key Insight:** Off-policy learning means the agent can learn about the optimal policy while following a completely different exploratory policy. The behavior policy just needs to visit all state-action pairs often enough — it doesn't need to be "good."

### 3.1 Why the Max Makes It Off-Policy

Consider what happens when the agent takes a random exploratory action in state $S_{t+1}$:

- **SARSA** uses $Q(S_{t+1}, A_{t+1})$ where $A_{t+1}$ is the random action — the update is "contaminated" by the exploration, so SARSA learns the value of the $\varepsilon$-greedy policy $q_\pi$ rather than the optimal $q_*$
- **Q-Learning** uses $\max_a Q(S_{t+1}, a)$ — the random action doesn't affect the update target at all, so Q-Learning always evaluates the greedy (optimal) policy

This is why Q-Learning converges to $q_*$ even with an exploratory behavior policy, while SARSA converges to $q_\pi$ (the value of the $\varepsilon$-greedy policy).

---

## 4. Connection to the Bellman Optimality Equation

The Q-Learning update is a **sample-based**, **incremental** approximation to the Bellman optimality equation for action values:

$$q_*(s, a) = \mathbb{E}\Big[ R_{t+1} + \gamma \max_{a'} q_*(S_{t+1}, a') \;\Big|\; S_t = s, A_t = a \Big]$$

Compare the Bellman optimality equation with the Q-Learning update:

| Component | Bellman Optimality Equation | Q-Learning Update |
|---|---|---|
| Left side | $q_*(s, a)$ | $Q(S_t, A_t)$ |
| Expected value | $\mathbb{E}[\cdot]$ | Sample-based (single transition) |
| Immediate reward | $R_{t+1}$ | $R_{t+1}$ (observed sample) |
| Future value | $\gamma \max_{a'} q_*(S_{t+1}, a')$ | $\gamma \max_a Q(S_{t+1}, a)$ |
| Solution method | Requires model; solved exactly | Model-free; solved incrementally |

Q-Learning can be understood as performing **stochastic approximation** on the Bellman optimality equation: each update nudges $Q(S_t, A_t)$ a fraction $\alpha$ toward a noisy sample of the true solution. Over many samples, these updates average out to solve the equation.

> **Key Insight:** SARSA approximates the **Bellman equation** for $q_\pi$ (policy evaluation), while Q-Learning approximates the **Bellman optimality equation** for $q_*$ (optimal control). This is the fundamental mathematical reason Q-Learning finds the optimal policy directly.

---

## 5. Q-Learning Backup Diagram

The backup diagram shows the structure of the Q-Learning update — which states and actions contribute to the updated value:

```
          Q-Learning Backup               SARSA Backup
          ─────────────────               ────────────

              (S_t, A_t)                   (S_t, A_t)
                  │                            │
                  ● ─── action A_t             ● ─── action A_t
                  │                            │
                  ○ ─── state S_{t+1}          ○ ─── state S_{t+1}
                 /|\                           │
                / | \                          │
               /  |  \                         │
              ●   ●   ●  ── all actions        ● ─── action A_{t+1}
             a₁  a₂  a₃                          (sampled from π)
              ↑
              │
           max (take best)

    Uses: max_a Q(S_{t+1}, a)       Uses: Q(S_{t+1}, A_{t+1})
    Evaluates: greedy policy        Evaluates: current ε-greedy policy
```

The Q-Learning backup fans out over **all** actions in the next state $S_{t+1}$ and selects the **maximum**. This is in contrast to SARSA, which samples a single next action. The branching over all actions is what implements the $\max$ operator and makes the method off-policy.

---

## 6. Complete Q-Learning Algorithm

```
Algorithm: Q-Learning (Off-Policy TD Control)
──────────────────────────────────────────────────
Input : step size α ∈ (0, 1], discount γ ∈ [0, 1],
        exploration rate ε > 0
Output: Q ≈ q_* (optimal action-value function)

1.  Initialize Q(s, a) for all s ∈ S, a ∈ A(s) arbitrarily
    (except Q(terminal, ·) = 0)

2.  REPEAT (for each episode):
3.  │   Initialize S (observe starting state)
4.  │
5.  │   REPEAT (for each step of episode):
6.  │   │
7.  │   │   Choose A from S using policy derived from Q
8.  │   │     (e.g., ε-greedy):
9.  │   │       with probability ε:  A ← random action
10. │   │       with probability 1−ε: A ← argmax_a Q(S, a)
11. │   │
12. │   │   Take action A, observe R, S'
13. │   │
14. │   │   Update Q:
15. │   │     Q(S, A) ← Q(S, A) + α [ R + γ max_a Q(S', a) − Q(S, A) ]
16. │   │
17. │   │   S ← S'
18. │   │
19. │   UNTIL S is terminal
20. │
21. UNTIL convergence or max episodes reached
```

### 6.1 Key Differences from SARSA Algorithm

Notice that Q-Learning does **not** need to choose $A'$ before the update (unlike SARSA, which requires the full quintuple $(S, A, R, S', A')$). Q-Learning only needs the quadruple $(S, A, R, S')$. The next action for the behavior policy is chosen **after** the update, at the top of the next iteration.

---

## 7. Convergence Theorem

**Theorem (Watkins & Dayan, 1992).** The Q-Learning algorithm converges to the optimal action-value function, $Q(s, a) \to q_*(s, a)$ as $t \to \infty$, with probability 1, provided that:

1. **All state-action pairs continue to be visited:** every $(s, a)$ pair is updated infinitely often
2. **Step sizes satisfy the Robbins-Monro conditions:**

$$\sum_{t=0}^{\infty} \alpha_t(s, a) = \infty \qquad \text{and} \qquad \sum_{t=0}^{\infty} \alpha_t^2(s, a) < \infty$$

3. **The rewards have bounded variance**

### 7.1 Practical Implications

| Condition | Theory | Practice |
|---|---|---|
| Visit all $(s, a)$ infinitely | Required | Use $\varepsilon$-greedy with $\varepsilon > 0$ |
| Robbins-Monro step sizes | Required for proof | Constant $\alpha$ works well in practice |
| Bounded rewards | Mild assumption | Almost always satisfied |
| Tabular representation | Required | Function approximation may diverge |

> **Warning:** With **function approximation** (e.g., neural networks), Q-Learning can diverge — this is the "deadly triad" problem (bootstrapping + off-policy + function approximation). Deep Q-Networks (DQN) use experience replay and target networks to mitigate this.

### 7.2 Why Constant α Works in Practice

A constant step size $\alpha$ violates the Robbins-Monro conditions (since $\sum \alpha^2 = \infty$), so theoretically Q-Learning does not converge to a single value. However, constant $\alpha$ gives the algorithm a **recency bias** — it weights recent transitions more heavily than old ones. In non-stationary environments or during the early learning phase, this is actually desirable.

---

## 8. Cliff Walking — Q-Learning vs SARSA

The **Cliff Walking** environment (Sutton & Barto, Example 6.6) is the classic demonstration of the on-policy vs off-policy difference. The 4×12 grid has a start state (bottom-left), a goal state (bottom-right), and a cliff along the bottom row. Stepping into the cliff gives a reward of $-100$ and sends the agent back to start. All other transitions give $-1$.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CLIFF WALKING ENVIRONMENT                        │
│                        (4 × 12 grid)                                │
│                                                                     │
│   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐               │
│   │   │   │   │   │   │   │   │   │   │   │   │   │  Row 0        │
│   ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤               │
│   │   │   │   │   │   │   │   │   │   │   │   │   │  Row 1        │
│   ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤               │
│   │   │   │   │   │   │   │   │   │   │   │   │   │  Row 2        │
│   ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤               │
│   │ S │ C │ C │ C │ C │ C │ C │ C │ C │ C │ C │ G │  Row 3        │
│   └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘               │
│                                                                     │
│   S = Start (state 36)       G = Goal (state 47)                    │
│   C = Cliff (reward −100, reset to S)                               │
│   All other transitions: reward −1                                  │
│                                                                     │
│   Actions: 0=Up, 1=Right, 2=Down, 3=Left                           │
└─────────────────────────────────────────────────────────────────────┘
```

### 8.1 Paths Learned by Each Algorithm

```
Q-Learning learns the OPTIMAL path (along the cliff edge):
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│   │   │   │   │   │   │   │   │   │   │   │   │
├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
│   │   │   │   │   │   │   │   │   │   │   │   │
├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
│ ↓ │ → │ → │ → │ → │ → │ → │ → │ → │ → │ → │ ↓ │  ← optimal path
├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤     (length 13)
│ S │ C │ C │ C │ C │ C │ C │ C │ C │ C │ C │ G │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
  ↑                                           ↑
  walks right along row 2 (one step above cliff)


SARSA learns the SAFE path (along the top row):
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ → │ → │ → │ → │ → │ → │ → │ → │ → │ → │ → │ ↓ │  ← safe path
├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤     (length ≈ 17)
│ ↑ │   │   │   │   │   │   │   │   │   │   │ ↓ │
├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
│   │   │   │   │   │   │   │   │   │   │   │ ↓ │
├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
│ S │ C │ C │ C │ C │ C │ C │ C │ C │ C │ C │ G │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

### 8.2 Why the Paths Differ

- **Q-Learning (off-policy)** updates toward the greedy policy ($\max$), which values the shortest path. It "knows" the optimal policy would never step into the cliff, so it assigns high value to the cliff-edge path. But during training with $\varepsilon$-greedy, random exploration along the edge causes occasional cliff falls and higher penalties.

- **SARSA (on-policy)** updates toward the $\varepsilon$-greedy policy, which includes random actions. Walking along the cliff edge with occasional random actions is **dangerous** — a random downward step means $-100$. SARSA accounts for this exploration risk and learns to avoid the cliff edge entirely.

| Metric | Q-Learning | SARSA |
|---|---|---|
| Learned policy (greedy) | Optimal, length 13, reward $-13$ | Safe, length $\approx 17$, reward $\approx -17$ |
| Online performance (during training) | Worse — cliff falls during exploration | Better — avoids cliff |
| Asymptotic greedy policy | Optimal | Suboptimal (unless $\varepsilon \to 0$) |

---

## 9. A Worked Numerical Example

Consider a simple 3-state MDP with 2 actions. Let $\alpha = 0.1$, $\gamma = 1.0$, and initial $Q = 0$ for all entries.

**Step 1:** In state $S_0$, take action $a_1$, receive $R = -1$, arrive at $S_1$.

$$Q(S_0, a_1) \leftarrow 0 + 0.1 \Big[ -1 + 1.0 \cdot \max(Q(S_1, a_0), Q(S_1, a_1)) - 0 \Big]$$
$$Q(S_0, a_1) \leftarrow 0 + 0.1 \Big[ -1 + 1.0 \cdot \max(0, 0) - 0 \Big] = -0.1$$

**Step 2:** In state $S_1$, take action $a_0$, receive $R = +5$, arrive at terminal state.

$$Q(S_1, a_0) \leftarrow 0 + 0.1 \Big[ 5 + 1.0 \cdot 0 - 0 \Big] = 0.5$$

**Step 3 (new episode):** In state $S_0$, take action $a_1$ again, receive $R = -1$, arrive at $S_1$.

$$Q(S_0, a_1) \leftarrow -0.1 + 0.1 \Big[ -1 + 1.0 \cdot \max(0.5, 0) - (-0.1) \Big]$$
$$Q(S_0, a_1) \leftarrow -0.1 + 0.1 \Big[ -1 + 0.5 + 0.1 \Big] = -0.1 + 0.1(-0.4) = -0.14$$

Notice how the updated $Q(S_1, a_0) = 0.5$ from Step 2 now influences the update in Step 3 via the $\max$ operator — **bootstrapping** propagates reward information backward through the state space.

---

## 10. Watkins' Q-Learning — Historical Note

Q-Learning was introduced by **Christopher Watkins** in his 1989 PhD thesis at Cambridge University, titled *"Learning from Delayed Rewards."* The convergence proof was later provided by **Watkins and Dayan (1992)** in their landmark paper in *Machine Learning*.

Key historical facts:
- Q-Learning was one of the **earliest** RL algorithms with a formal convergence guarantee
- The name "Q" comes from Watkins' use of "$Q$" to denote the **quality** of a state-action pair
- Before Q-Learning, most RL algorithms could only evaluate a given policy (prediction); Q-Learning was revolutionary because it directly solved the **control** problem
- Watkins' Q-Learning laid the foundation for **Deep Q-Networks (DQN)** by DeepMind in 2013/2015, which achieved superhuman performance on Atari games by combining Q-Learning with deep neural networks

The original tabular Q-Learning algorithm presented here remains exactly as Watkins described it — a testament to the elegance and completeness of his original formulation.

---

## 11. Complete Python Implementation — CliffWalking-v0

```python
"""
Q-Learning on Gymnasium CliffWalking-v0
========================================
Off-policy TD control: learns the optimal policy directly.
Compare with SARSA to see the on-policy vs off-policy difference.
"""
import numpy as np
import gymnasium as gym
import matplotlib.pyplot as plt


# --- Helper: ε-greedy action selection ---
def epsilon_greedy(Q, state, epsilon, n_actions):
    """Select action using ε-greedy policy derived from Q."""
    if np.random.random() < epsilon:
        return np.random.randint(n_actions)
    else:
        return np.argmax(Q[state])


# --- Q-Learning Algorithm ---
def q_learning(env, n_episodes=500, alpha=0.5, gamma=1.0, epsilon=0.1):
    """
    Tabular Q-Learning (off-policy TD control).

    Parameters
    ----------
    env        : Gymnasium environment
    n_episodes : number of training episodes
    alpha      : step size (learning rate)
    gamma      : discount factor
    epsilon    : exploration rate for ε-greedy behavior policy

    Returns
    -------
    Q               : learned action-value function (n_states × n_actions)
    episode_rewards : list of total rewards per episode
    """
    n_states = env.observation_space.n
    n_actions = env.action_space.n
    Q = np.zeros((n_states, n_actions))
    episode_rewards = []

    for ep in range(n_episodes):
        state, _ = env.reset()
        total_reward = 0

        while True:
            # Choose action using ε-greedy BEHAVIOR policy
            action = epsilon_greedy(Q, state, epsilon, n_actions)

            # Take action, observe reward and next state
            next_state, reward, terminated, truncated, _ = env.step(action)
            total_reward += reward

            # Q-Learning update: use max over next state (off-policy)
            best_next_value = np.max(Q[next_state]) * (1 - terminated)
            td_target = reward + gamma * best_next_value
            td_error = td_target - Q[state, action]
            Q[state, action] += alpha * td_error

            state = next_state

            if terminated or truncated:
                break

        episode_rewards.append(total_reward)

    return Q, episode_rewards


# --- Train Q-Learning ---
env = gym.make('CliffWalking-v0')
Q_qlearn, rewards_qlearn = q_learning(env, n_episodes=500)
env.close()


# --- Print learned policy ---
action_symbols = {0: '↑', 1: '→', 2: '↓', 3: '←'}

print("Learned Q-Learning Policy (4×12 grid):")
print("=" * 50)
for row in range(4):
    row_str = ""
    for col in range(12):
        state = row * 12 + col
        if row == 3 and col == 0:
            row_str += "  S "
        elif row == 3 and col == 11:
            row_str += "  G "
        elif row == 3 and 1 <= col <= 10:
            row_str += "  C "
        else:
            best_action = np.argmax(Q_qlearn[state])
            row_str += f"  {action_symbols[best_action]} "
    print(row_str)
print("=" * 50)


# --- Print reward statistics ---
print(f"\nMean reward (last 100 episodes): {np.mean(rewards_qlearn[-100:]):.1f}")
print(f"Mean reward (first 100 episodes): {np.mean(rewards_qlearn[:100]):.1f}")
```

### 11.1 Expected Output

```
Learned Q-Learning Policy (4×12 grid):
==================================================
  →   →   →   →   →   →   →   →   →   →   →   ↓
  →   →   →   →   →   →   →   →   →   →   →   ↓
  ↓   →   →   →   →   →   →   →   →   →   →   ↓
  S   C   C   C   C   C   C   C   C   C   C   G
==================================================

Mean reward (last 100 episodes): -18.7
Mean reward (first 100 episodes): -72.3
```

Note: Q-Learning's **learned** greedy policy is optimal (the cliff-edge path of length 13), but the **online** reward during training is worse than SARSA because exploration with $\varepsilon$-greedy along the cliff edge leads to occasional catastrophic falls ($-100$).

---

## 12. Side-by-Side Comparison — Q-Learning vs SARSA

```python
"""
Side-by-side comparison of Q-Learning and SARSA on CliffWalking-v0.
Reproduces the classic Figure 6.3 from Sutton & Barto.
"""
import numpy as np
import gymnasium as gym
import matplotlib.pyplot as plt


def epsilon_greedy(Q, state, epsilon, n_actions):
    if np.random.random() < epsilon:
        return np.random.randint(n_actions)
    return np.argmax(Q[state])


def sarsa(env, n_episodes=500, alpha=0.5, gamma=1.0, epsilon=0.1):
    """On-policy SARSA for comparison."""
    n_states = env.observation_space.n
    n_actions = env.action_space.n
    Q = np.zeros((n_states, n_actions))
    episode_rewards = []

    for ep in range(n_episodes):
        state, _ = env.reset()
        action = epsilon_greedy(Q, state, epsilon, n_actions)
        total_reward = 0

        while True:
            next_state, reward, terminated, truncated, _ = env.step(action)
            total_reward += reward
            next_action = epsilon_greedy(Q, next_state, epsilon, n_actions)

            td_target = reward + gamma * Q[next_state, next_action] * (1 - terminated)
            Q[state, action] += alpha * (td_target - Q[state, action])

            state = next_state
            action = next_action

            if terminated or truncated:
                break

        episode_rewards.append(total_reward)
    return Q, episode_rewards


def q_learning(env, n_episodes=500, alpha=0.5, gamma=1.0, epsilon=0.1):
    """Off-policy Q-Learning."""
    n_states = env.observation_space.n
    n_actions = env.action_space.n
    Q = np.zeros((n_states, n_actions))
    episode_rewards = []

    for ep in range(n_episodes):
        state, _ = env.reset()
        total_reward = 0

        while True:
            action = epsilon_greedy(Q, state, epsilon, n_actions)
            next_state, reward, terminated, truncated, _ = env.step(action)
            total_reward += reward

            best_next = np.max(Q[next_state]) * (1 - terminated)
            Q[state, action] += alpha * (reward + gamma * best_next - Q[state, action])

            state = next_state
            if terminated or truncated:
                break

        episode_rewards.append(total_reward)
    return Q, episode_rewards


# --- Run both algorithms over multiple runs ---
n_runs = 30
n_episodes = 500
sarsa_all = np.zeros((n_runs, n_episodes))
qlearn_all = np.zeros((n_runs, n_episodes))

for run in range(n_runs):
    env = gym.make('CliffWalking-v0')
    _, sarsa_all[run] = sarsa(env, n_episodes)
    env.close()

    env = gym.make('CliffWalking-v0')
    _, qlearn_all[run] = q_learning(env, n_episodes)
    env.close()

# --- Plot comparison (Figure 6.3 style) ---
window = 10
sarsa_mean = np.mean(sarsa_all, axis=0)
qlearn_mean = np.mean(qlearn_all, axis=0)

sarsa_smooth = np.convolve(sarsa_mean, np.ones(window) / window, mode='valid')
qlearn_smooth = np.convolve(qlearn_mean, np.ones(window) / window, mode='valid')

plt.figure(figsize=(10, 6))
plt.plot(sarsa_smooth, label='SARSA', color='steelblue', linewidth=2)
plt.plot(qlearn_smooth, label='Q-Learning', color='darkorange', linewidth=2)
plt.xlabel('Episode')
plt.ylabel(f'Sum of rewards during episode\n(averaged over {n_runs} runs)')
plt.title('Q-Learning vs SARSA on Cliff Walking')
plt.ylim(-100, 0)
plt.axhline(y=-13, color='green', linestyle='--', alpha=0.5, label='Optimal: −13')
plt.legend(loc='lower right')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('qlearning_vs_sarsa_cliffwalking.png', dpi=150)
plt.show()
print("Comparison plot saved to qlearning_vs_sarsa_cliffwalking.png")
```

### 12.1 Interpreting the Comparison

The resulting plot closely matches **Figure 6.3** from Sutton & Barto (2018):

- **SARSA** (blue) converges to a reward around $-17$ — the safe top-row path
- **Q-Learning** (orange) has more negative rewards during training (around $-25$ to $-40$) because the optimal cliff-edge policy, combined with $\varepsilon$-greedy exploration, causes frequent cliff falls
- **Key takeaway:** SARSA has better **online** performance; Q-Learning has a better **learned** policy

---

## 13. Q-Learning vs SARSA — Detailed Comparison

| Property | Q-Learning | SARSA |
|---|---|---|
| **Policy type** | Off-policy | On-policy |
| **Update target** | $R + \gamma \max_a Q(S', a)$ | $R + \gamma Q(S', A')$ |
| **Next action in update** | $\arg\max_a Q(S', a)$ (greedy) | $A' \sim \varepsilon\text{-greedy}$ (sampled) |
| **What it converges to** | $q_*$ (optimal values) | $q_\pi$ ($\varepsilon$-greedy values) |
| **Behavior policy** | $\varepsilon$-greedy (any exploring policy) | $\varepsilon$-greedy (same as target) |
| **Target policy** | Greedy ($\pi(s) = \arg\max_a Q(s,a)$) | $\varepsilon$-greedy (same as behavior) |
| **Cliff Walking path** | Optimal (cliff edge, reward $-13$) | Safe (top row, reward $\approx -17$) |
| **Online performance** | Worse (exploration causes cliff falls) | Better (avoids dangerous areas) |
| **Asymptotic policy** | Always optimal | Optimal only if $\varepsilon \to 0$ |
| **Risk sensitivity** | Ignores exploration risk | Accounts for exploration risk |
| **Data required per update** | Quadruple $(S, A, R, S')$ | Quintuple $(S, A, R, S', A')$ |
| **Deadly triad risk** | Higher (off-policy + bootstrapping) | Lower (on-policy) |

---

## 14. Advantages and Limitations

### 14.1 Advantages

1. **Learns optimal policy directly** — no need to decay $\varepsilon$ for convergence to $q_*$
2. **Simpler implementation** — does not require $A'$ before the update
3. **Can learn from demonstrations** — because it is off-policy, Q-Learning can learn from data generated by any policy (expert demonstrations, random exploration, etc.)
4. **Foundation for DQN** — Q-Learning's off-policy nature is essential for experience replay in Deep Q-Networks

### 14.2 Limitations

1. **Maximization bias** — the $\max$ operator causes systematic overestimation of action values (see Double Q-Learning for a fix)
2. **Poor online performance during training** — ignoring exploration risk leads to more penalties during learning
3. **Deadly triad with function approximation** — off-policy + bootstrapping + function approximation can cause divergence
4. **Requires exploration** — the behavior policy must still visit all state-action pairs for convergence

### 14.3 Maximization Bias — A Brief Note

The $\max$ in Q-Learning introduces a subtle upward bias:

$$\mathbb{E}\Big[\max_a Q(s, a)\Big] \geq \max_a \mathbb{E}\Big[Q(s, a)\Big]$$

When Q-values are noisy estimates, taking the maximum of noisy values tends to **overestimate** the true maximum. **Double Q-Learning** (Hasselt, 2010) fixes this by maintaining two independent Q-tables and using one to select the action and the other to evaluate it.

---

## 15. The Landscape — Where Q-Learning Fits

```
┌──────────────────────────────────────────────────────────────────┐
│                     TD CONTROL METHODS                           │
│                                                                  │
│   On-Policy                              Off-Policy              │
│   ─────────                              ──────────              │
│                                                                  │
│   SARSA                                  Q-Learning              │
│   Q(S,A) + α[R + γQ(S',A') − Q(S,A)]   Q(S,A) + α[R + γ max Q(S',a) − Q(S,A)]
│   learns q_π                             learns q_*              │
│        │                                      │                  │
│        │                                      │                  │
│        ▼                                      ▼                  │
│   Expected SARSA                         Double Q-Learning       │
│   uses E_π[Q(S',·)]                      fixes max bias          │
│   (lower variance)                                               │
│        │                                      │                  │
│        └─── when π = greedy ──────────────────┘                  │
│             Expected SARSA = Q-Learning                          │
│                                                                  │
│   n-step SARSA                           n-step Q-Learning       │
│   (n-step returns)                       (n-step off-policy)     │
│                                                                  │
│                       Deep RL                                    │
│                         │                                        │
│                    DQN (2015)                                    │
│                    Q-Learning + Neural Networks                  │
│                    + Experience Replay                           │
│                    + Target Networks                             │
└──────────────────────────────────────────────────────────────────┘
```

---

## Summary

| Concept | Key Point |
|---|---|
| **Q-Learning Update** | $Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha[R_{t+1} + \gamma \max_a Q(S_{t+1}, a) - Q(S_t, A_t)]$ |
| **Off-Policy** | Behavior policy (ε-greedy) ≠ target policy (greedy); the $\max$ evaluates greedy |
| **What It Learns** | $q_*$ — the optimal action-value function, regardless of behavior policy |
| **Bellman Connection** | Sample-based incremental approximation of the Bellman optimality equation |
| **TD Error** | $\delta_t = R_{t+1} + \gamma \max_a Q(S_{t+1}, a) - Q(S_t, A_t)$ |
| **Convergence** | Converges to $q_*$ with probability 1 under Robbins-Monro conditions (Watkins & Dayan, 1992) |
| **Cliff Walking** | Learns optimal cliff-edge path (reward $-13$); SARSA learns safe top-row path ($\approx -17$) |
| **Online Performance** | Worse than SARSA during training due to cliff falls from exploration |
| **Data Per Update** | Quadruple $(S, A, R, S')$ — does not need $A'$ |
| **Maximization Bias** | $\max$ causes overestimation; fixed by Double Q-Learning |
| **Backup Diagram** | Branches over all next actions, selects max — wider than SARSA's single-action backup |
| **Historical Origin** | Watkins (1989); convergence proof by Watkins & Dayan (1992) |

---

## Revision Questions

1. **Write out the Q-Learning update rule and identify the TD target and TD error.** Explain how the $\max$ operator changes the semantics compared to SARSA's update.

2. **What does "off-policy" mean, and why does the $\max$ make Q-Learning off-policy?** Identify the behavior policy and the target policy in Q-Learning, and explain why they differ.

3. **State the convergence theorem for Q-Learning and list its required conditions.** Why does Q-Learning converge to $q_*$ while SARSA only converges to $q_\pi$?

4. **Draw the Cliff Walking grid and the paths learned by Q-Learning and SARSA.** Explain why Q-Learning has worse online performance during training despite learning a better policy.

5. **Explain how Q-Learning relates to the Bellman optimality equation.** Why is it accurate to describe Q-Learning as "stochastic approximation" applied to this equation?

6. **What is maximization bias, and how does it arise in Q-Learning?** Sketch how Double Q-Learning addresses this problem.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [SARSA](./02-sarsa.md) | [Unit 5: TD Learning](./README.md) | [Expected SARSA](./04-expected-sarsa.md) |

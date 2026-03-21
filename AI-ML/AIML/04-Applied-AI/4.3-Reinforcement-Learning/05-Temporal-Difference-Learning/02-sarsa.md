# Chapter 2: SARSA — On-Policy TD Control

> **Unit 5: Temporal Difference Learning** — *Learning action values on-policy, one transition at a time.*

---

## Overview

SARSA is the natural extension of TD(0) prediction to the **control** setting: instead of learning a state-value function $V(s)$, SARSA learns an **action-value function** $Q(s, a)$ and improves the policy simultaneously. Because SARSA follows and evaluates the **same** $\varepsilon$-greedy policy it uses to select actions, it is an **on-policy** method. The name SARSA comes from the quintuple of values used in each update: $(S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1})$.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                        SARSA AT A GLANCE                               ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║   Given: nothing — learns while interacting                            ║
║                                                                        ║
║   Goal:  learn Q(s, a) ≈ q_π(s, a) while improving π                  ║
║                                                                        ║
║   Each transition provides the quintuple:                              ║
║                                                                        ║
║          S_t ──(A_t)──► R_{t+1}, S_{t+1} ──(A_{t+1})──►               ║
║                                                                        ║
║   Update:                                                              ║
║          Q(S_t, A_t) ← Q(S_t, A_t)                                    ║
║                        + α [ R_{t+1} + γ Q(S_{t+1}, A_{t+1})          ║
║                              − Q(S_t, A_t) ]                          ║
║                                                                        ║
║   Policy: ε-greedy with respect to Q  (on-policy)                      ║
║                                                                        ║
╚══════════════════════════════════════════════════════════════════════════╝
```

SARSA is the action-value analogue of TD(0). Where TD(0) uses a transition $(S_t, R_{t+1}, S_{t+1})$ to update $V(S_t)$, SARSA uses the full five-element transition to update $Q(S_t, A_t)$. Because the next action $A_{t+1}$ is already selected **before** the update, the method naturally evaluates the behavior policy — making it truly on-policy.

---

## 1. The SARSA Name — State-Action-Reward-State-Action

The name SARSA is an acronym that encodes exactly which quantities are needed for one update:

$$\underbrace{S_t}_{\text{State}}, \quad \underbrace{A_t}_{\text{Action}}, \quad \underbrace{R_{t+1}}_{\text{Reward}}, \quad \underbrace{S_{t+1}}_{\text{next State}}, \quad \underbrace{A_{t+1}}_{\text{next Action}}$$

At every time step the agent:
1. **Observes** the current state $S_t$
2. **Selects** action $A_t$ using the current policy (e.g., $\varepsilon$-greedy w.r.t. $Q$)
3. **Receives** reward $R_{t+1}$ and transitions to $S_{t+1}$
4. **Selects** the next action $A_{t+1}$ using the **same** policy
5. **Updates** $Q(S_t, A_t)$ using all five values

The fact that $A_{t+1}$ is chosen by the **same** policy used to generate $A_t$ is the defining characteristic that makes SARSA on-policy.

---

## 2. The SARSA Update Rule

After each transition, SARSA applies the following update:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \, Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) \Big]$$

where:
- $Q(S_t, A_t)$ is the current estimate of the action-value for state-action pair $(S_t, A_t)$
- $\alpha \in (0, 1]$ is the **step size** (learning rate)
- $R_{t+1}$ is the **immediate reward** received
- $\gamma \in [0, 1]$ is the **discount factor**
- $Q(S_{t+1}, A_{t+1})$ is the estimated value of the **next** state-action pair under the current policy
- $A_{t+1}$ is the action **actually chosen** by the behavior policy in state $S_{t+1}$

### 2.1 Decomposing the Update

**TD Target** — the one-step estimated return for action values:

$$\text{TD Target} = R_{t+1} + \gamma \, Q(S_{t+1}, A_{t+1})$$

**TD Error** — the temporal difference signal:

$$\delta_t = R_{t+1} + \gamma \, Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)$$

**Compact form:**

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \, \delta_t$$

> **Key Insight:** The TD error $\delta_t$ measures how much better (or worse) the transition turned out compared to the current estimate. When $\delta_t > 0$, the pair $(S_t, A_t)$ was more valuable than expected; the update increases $Q(S_t, A_t)$.

### 2.2 Terminal State Handling

When $S_{t+1}$ is a terminal state, there is no future value, so the update simplifies to:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} - Q(S_t, A_t) \Big]$$

In implementation, we define $Q(\text{terminal}, \cdot) = 0$ for all actions.

---

## 3. Relationship to TD(0) — SARSA as TD Control

SARSA is **TD(0) applied to action-value functions**. The parallel is direct:

| Component | TD(0) Prediction | SARSA Control |
|---|---|---|
| What is learned | $V(s) \approx v_\pi(s)$ | $Q(s, a) \approx q_\pi(s, a)$ |
| Update target | $R_{t+1} + \gamma V(S_{t+1})$ | $R_{t+1} + \gamma Q(S_{t+1}, A_{t+1})$ |
| Update variable | $V(S_t)$ | $Q(S_t, A_t)$ |
| Required experience | $(S_t, R_{t+1}, S_{t+1})$ | $(S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1})$ |
| Policy improvement | Separate step needed | Implicit via $\varepsilon$-greedy on $Q$ |
| Bootstrapping | Yes — uses $V(S_{t+1})$ | Yes — uses $Q(S_{t+1}, A_{t+1})$ |

The critical difference is that SARSA learns **action values**, which allow policy improvement without a model. With $Q(s, a)$, the agent can act greedily by selecting $\arg\max_a Q(s, a)$ — no transition probabilities needed.

---

## 4. On-Policy Nature

SARSA is **on-policy** because the action $A_{t+1}$ appearing in the update is drawn from the **same** $\varepsilon$-greedy policy used to select all actions. The behavior policy (exploration) and the target policy (evaluation) are identical:

$$\pi_{\text{behavior}} = \pi_{\text{target}} = \varepsilon\text{-greedy}(Q)$$

This means SARSA evaluates the policy it is actually following, **including** its exploratory actions. If the agent takes a random exploratory action that leads to a poor outcome, SARSA learns from that consequence — leading to more **conservative, risk-aware** policies.

### 4.1 The $\varepsilon$-Greedy Policy

Given current action-value estimates $Q$, the $\varepsilon$-greedy policy in state $s$ is:

$$\pi(a \mid s) = \begin{cases} 1 - \varepsilon + \frac{\varepsilon}{|\mathcal{A}(s)|} & \text{if } a = \arg\max_{a'} Q(s, a') \\ \frac{\varepsilon}{|\mathcal{A}(s)|} & \text{otherwise} \end{cases}$$

where $|\mathcal{A}(s)|$ is the number of available actions in state $s$.

---

## 5. SARSA Backup Diagram

The backup diagram for SARSA shows the information flow used in each update. Unlike TD(0) prediction (which backs up from a state node), SARSA backs up from one **state-action pair** to the next:

```
                  SARSA Backup Diagram
                  ═══════════════════

                     (S_t, A_t)         ← state-action node being updated
                         │
                         │  take action A_t
                         ▼
                        S_t              ← receive R_{t+1}
                         │
                         │  transition to S_{t+1}
                         ▼
                     (S_{t+1})           ← next state
                         │
                         │  choose A_{t+1} ~ π(·|S_{t+1})
                         ▼
                   (S_{t+1}, A_{t+1})   ← bootstrapped value

         The update uses exactly ONE transition and ONE
         subsequent action selection — a single sample path.

         Q(S_t,A_t) ← Q(S_t,A_t) + α[R_{t+1} + γQ(S_{t+1},A_{t+1}) − Q(S_t,A_t)]
```

> **Note:** The backup is one step deep and one sample wide — making it a **sample backup** (as opposed to a full-width DP backup which considers all possible next states).

---

## 6. Complete SARSA Algorithm

The tabular SARSA algorithm for on-policy TD control is presented below. It combines policy evaluation (updating $Q$) and policy improvement ($\varepsilon$-greedy) into a single online loop.

```
Algorithm: SARSA (On-Policy TD Control)
═══════════════════════════════════════════════════════════════

Input:  step size α ∈ (0, 1], small ε > 0, discount γ ∈ [0, 1]
Output: action-value function Q ≈ q_*

1.  Initialize Q(s, a) arbitrarily for all s ∈ S, a ∈ A(s)
    Initialize Q(terminal, ·) = 0

2.  Loop for each episode:

3.      Initialize S (starting state of the episode)
4.      Choose A from S using policy derived from Q (e.g., ε-greedy)

5.      Loop for each step of the episode:

6.          Take action A, observe R, S'

7.          Choose A' from S' using policy derived from Q (e.g., ε-greedy)

8.          Q(S, A) ← Q(S, A) + α [ R + γ Q(S', A') − Q(S, A) ]

9.          S ← S'
10.         A ← A'

11.     until S is terminal

12. end loop
```

### 6.1 Key Implementation Details

- **Action selection at line 4** — the initial action $A$ is chosen **before** the inner loop begins
- **Action selection at line 7** — the next action $A'$ is chosen **before** the update at line 8. This ensures the update uses the action that **will** be taken, not a hypothetical one
- **Line 9–10** — the state and action carry forward, so the $A'$ selected now becomes the $A$ for the next iteration (no need to re-select)
- **Terminal handling** — when $S'$ is terminal, $Q(S', A') = 0$ by initialization

---

## 7. Convergence Properties

SARSA converges to the optimal action-value function $q_*$ under the following conditions:

### 7.1 GLIE Conditions

SARSA converges to $q_*$ with probability 1 if the policy satisfies **GLIE** — **Greedy in the Limit with Infinite Exploration**:

1. **Infinite exploration:** every state-action pair is visited infinitely often:
   $$\forall (s, a): \quad \sum_{t=0}^{\infty} \mathbf{1}\{S_t = s, A_t = a\} = \infty$$

2. **Greedy in the limit:** the policy converges to a greedy policy:
   $$\pi_t(a \mid s) \xrightarrow{t \to \infty} \mathbf{1}\{a = \arg\max_{a'} Q(s, a')\}$$

A common strategy is to decay $\varepsilon$ over time, e.g., $\varepsilon_k = \frac{1}{k}$ where $k$ is the episode number.

### 7.2 Step-Size Conditions (Robbins-Monro)

Additionally, the step-size sequence $\alpha_t$ must satisfy:

$$\sum_{t=0}^{\infty} \alpha_t = \infty \qquad \text{and} \qquad \sum_{t=0}^{\infty} \alpha_t^2 < \infty$$

In practice, a constant $\alpha$ is often used for non-stationary environments and faster adaptation, trading guaranteed convergence for tracking ability.

### 7.3 What SARSA Converges To

Under a fixed $\varepsilon$-greedy policy (constant $\varepsilon$), SARSA converges to $q_\pi$, the action-value function **of that $\varepsilon$-greedy policy** — not $q_*$. This is expected: SARSA evaluates the policy it follows, and that policy includes random exploration. Only with decaying $\varepsilon \to 0$ does SARSA converge to $q_*$.

---

## 8. The Cliff Walking Example

The Cliff Walking environment (Sutton & Barto, Chapter 6, Example 6.6) is the classic demonstration of how SARSA's on-policy nature leads to **safer** policies than off-policy methods like Q-Learning.

### 8.1 The Environment

```
╔══════════════════════════════════════════════════════════════════╗
║                    CLIFF WALKING GRID (4×12)                    ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║   Row 0:  .   .   .   .   .   .   .   .   .   .   .   .        ║
║   Row 1:  .   .   .   .   .   .   .   .   .   .   .   .        ║
║   Row 2:  .   .   .   .   .   .   .   .   .   .   .   .        ║
║   Row 3:  S   C   C   C   C   C   C   C   C   C   C   G        ║
║                                                                  ║
║   S = Start (bottom-left)       G = Goal (bottom-right)         ║
║   C = Cliff (reward = −100, resets to S)                        ║
║   . = Normal cell (reward = −1 per step)                        ║
║                                                                  ║
║   Actions: Up(0), Right(1), Down(2), Left(3)                    ║
║   Episode ends when agent reaches G                             ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

### 8.2 SARSA vs Q-Learning Paths

The crucial insight is the **different paths** learned by SARSA and Q-Learning:

```
     SARSA learns the SAFE path (along the top):
     ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
     │ → │ → │ → │ → │ → │ → │ → │ → │ → │ → │ → │ ↓ │  ← Row 0
     ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
     │   │   │   │   │   │   │   │   │   │   │   │ ↓ │  ← Row 1
     ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
     │   │   │   │   │   │   │   │   │   │   │   │ ↓ │  ← Row 2
     ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
     │ S │ C │ C │ C │ C │ C │ C │ C │ C │ C │ C │ G │  ← Row 3
     └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
          ▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲▲
                    THE CLIFF (reward = −100)

     Q-Learning learns the OPTIMAL path (along the bottom edge):
     ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
     │   │   │   │   │   │   │   │   │   │   │   │   │
     ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
     │   │   │   │   │   │   │   │   │   │   │   │   │
     ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
     │ ↑ │ → │ → │ → │ → │ → │ → │ → │ → │ → │ → │ ↓ │  ← just above cliff
     ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
     │ S │ C │ C │ C │ C │ C │ C │ C │ C │ C │ C │ G │
     └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

### 8.3 Why the Paths Differ

**SARSA (on-policy):** Because SARSA evaluates the $\varepsilon$-greedy policy it actually follows, it accounts for the fact that with probability $\varepsilon$ it will take a **random** action. Walking next to the cliff means a random action could step **into** the cliff (reward $-100$). SARSA learns that the cells adjacent to the cliff have low value (because the actual behavior occasionally falls in), so it prefers the longer but safer path along the top.

**Q-Learning (off-policy):** Q-Learning evaluates the **greedy** policy regardless of what action is actually taken. It assumes the agent will always take the best action, so it doesn't account for exploratory missteps. The cells above the cliff look fine to Q-Learning because the greedy action would never step into the cliff. But during training with $\varepsilon$-greedy exploration, the agent following Q-Learning's bottom path frequently falls off the cliff.

> **Key Insight:** SARSA produces lower average return during training because it takes a longer path, but it avoids catastrophic penalties. The "optimal" path found by Q-Learning is only optimal for the **greedy** policy — during $\varepsilon$-greedy execution, SARSA's policy actually achieves better online performance.

---

## 9. Effect of ε on the Learned Policy

The exploration rate $\varepsilon$ significantly affects what SARSA learns:

| $\varepsilon$ Value | Exploration Level | Policy Learned | Cliff Walking Behavior |
|---|---|---|---|
| $\varepsilon = 0$ | None (greedy) | $q_*$ (optimal) | Shortest path along cliff edge |
| $\varepsilon = 0.1$ | Moderate | $q_\pi$ (safe) | Safe path away from cliff |
| $\varepsilon = 0.3$ | High | $q_\pi$ (very cautious) | Very wide detour from cliff |
| $\varepsilon \to 0$ (decay) | Decreasing | $q_*$ over time | Starts safe, eventually finds optimal |

**With constant $\varepsilon$:** SARSA converges to the action-value function of the $\varepsilon$-greedy policy. Higher $\varepsilon$ means more random actions are factored into the values, so policies become more conservative.

**With decaying $\varepsilon$:** As $\varepsilon \to 0$, the policy becomes greedy, and SARSA converges to $q_*$. The agent initially explores broadly, then gradually commits to the optimal path.

The mathematical relationship:

$$Q^\pi(s, a) = \mathbb{E}_\pi\Big[R_{t+1} + \gamma Q^\pi(S_{t+1}, A_{t+1}) \mid S_t = s, A_t = a\Big]$$

Since $A_{t+1} \sim \pi(\cdot \mid S_{t+1})$ and $\pi$ is $\varepsilon$-greedy, the expected next Q-value is:

$$\mathbb{E}[Q(S_{t+1}, A_{t+1})] = (1 - \varepsilon)\max_{a'} Q(S_{t+1}, a') + \frac{\varepsilon}{|\mathcal{A}|}\sum_{a'} Q(S_{t+1}, a')$$

---

## 10. Mathematical Derivation — Bellman Equation for SARSA

The true action-value function under policy $\pi$ satisfies the **Bellman equation for $q_\pi$**:

$$q_\pi(s, a) = \mathbb{E}_\pi \Big[ R_{t+1} + \gamma \, q_\pi(S_{t+1}, A_{t+1}) \mid S_t = s, A_t = a \Big]$$

Expanding over the transition dynamics:

$$q_\pi(s, a) = \sum_{s'} \sum_{r} p(s', r \mid s, a) \left[ r + \gamma \sum_{a'} \pi(a' \mid s') \, q_\pi(s', a') \right]$$

The SARSA update:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \, Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) \Big]$$

performs **stochastic approximation** to solve this Bellman equation. Each update moves $Q(S_t, A_t)$ toward the sample TD target $R_{t+1} + \gamma Q(S_{t+1}, A_{t+1})$. In expectation over the randomness of transitions and action selections:

$$\mathbb{E}\Big[ R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) \mid S_t, A_t \Big] \approx q_\pi(S_t, A_t)$$

At the fixed point where $Q = q_\pi$, the expected TD error is zero for all $(s, a)$:

$$\mathbb{E}_\pi[\delta_t \mid S_t = s, A_t = a] = 0$$

---

## 11. Python Implementation — SARSA on Cliff Walking

Below is a complete implementation using Gymnasium's `CliffWalking-v0` environment.

```python
import gymnasium as gym
import numpy as np
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt


def epsilon_greedy(Q, state, epsilon, n_actions):
    """Select action using ε-greedy policy derived from Q."""
    if np.random.random() < epsilon:
        return np.random.randint(n_actions)
    else:
        return np.argmax(Q[state])


def sarsa(env, n_episodes=500, alpha=0.5, gamma=1.0, epsilon=0.1):
    """
    Tabular SARSA on-policy TD control.

    Parameters
    ----------
    env       : Gymnasium environment
    n_episodes: number of training episodes
    alpha     : step size (learning rate)
    gamma     : discount factor
    epsilon   : exploration rate for ε-greedy

    Returns
    -------
    Q              : learned action-value table [n_states x n_actions]
    episode_rewards: total reward per episode (for plotting)
    """
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

            # Choose next action A' from S' using ε-greedy (on-policy)
            next_action = epsilon_greedy(Q, next_state, epsilon, n_actions)

            # SARSA update: Q(S,A) ← Q(S,A) + α[R + γQ(S',A') − Q(S,A)]
            td_target = reward + gamma * Q[next_state, next_action] * (1 - terminated)
            td_error = td_target - Q[state, action]
            Q[state, action] += alpha * td_error

            state = next_state
            action = next_action  # A ← A' (carry forward)

            if terminated or truncated:
                break

        episode_rewards.append(total_reward)

    return Q, episode_rewards


# --- Run SARSA on CliffWalking-v0 ---
env = gym.make('CliffWalking-v0')
Q_sarsa, rewards_sarsa = sarsa(env, n_episodes=500, alpha=0.5, gamma=1.0, epsilon=0.1)
env.close()


# --- Display learned policy ---
action_symbols = ['↑', '→', '↓', '←']

print("Learned SARSA Policy (4×12 grid):")
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
            best_action = np.argmax(Q_sarsa[state])
            row_str += f"  {action_symbols[best_action]} "
    print(row_str)
print("=" * 50)


# --- Plot training curve ---
window = 20
smoothed = np.convolve(rewards_sarsa, np.ones(window) / window, mode='valid')

plt.figure(figsize=(10, 5))
plt.plot(smoothed, linewidth=1.5, color='steelblue')
plt.xlabel('Episode')
plt.ylabel(f'Reward (smoothed over {window} episodes)')
plt.title('SARSA on CliffWalking-v0 — Training Curve')
plt.axhline(y=-13, color='green', linestyle='--', label='Optimal (greedy): −13')
plt.axhline(y=-17, color='orange', linestyle='--', label='Safe path (SARSA): ≈ −17')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('sarsa_cliffwalking_training.png', dpi=150)
plt.show()
print("Training plot saved to sarsa_cliffwalking_training.png")


# --- Print reward statistics ---
print(f"\nMean reward (last 100 episodes): {np.mean(rewards_sarsa[-100:]):.1f}")
print(f"Mean reward (first 100 episodes): {np.mean(rewards_sarsa[:100]):.1f}")
```

### 11.1 Expected Output

```
Learned SARSA Policy (4×12 grid):
==================================================
  →   →   →   →   →   →   →   →   →   →   →   ↓
  ↑   →   →   →   →   →   →   →   →   →   →   ↓
  ↑   →   →   →   →   →   →   →   →   →   →   ↓
  S   C   C   C   C   C   C   C   C   C   C   G
==================================================

Mean reward (last 100 episodes): -17.2
Mean reward (first 100 episodes): -62.5
```

### 11.2 Training Curve Interpretation

The training curve shows:
- **Early episodes (0–50):** reward is very low (around $-100$ to $-200$) because the agent frequently falls off the cliff during exploration
- **Mid training (50–200):** rapid improvement as the agent learns to avoid the cliff entirely
- **Late training (200–500):** reward stabilizes around $-17$ to $-20$, reflecting the safe top-row path. This is higher (less negative) than the optimal greedy path length of $-13$ because the safe path is longer, but the agent avoids cliff falls

---

## 12. SARSA with Decaying ε

To achieve convergence to $q_*$, we can decay $\varepsilon$ over episodes:

```python
def sarsa_decaying_epsilon(env, n_episodes=1000, alpha=0.5, gamma=1.0,
                            epsilon_start=1.0, epsilon_min=0.01, decay_rate=0.995):
    """SARSA with exponentially decaying ε for GLIE convergence."""
    n_states = env.observation_space.n
    n_actions = env.action_space.n
    Q = np.zeros((n_states, n_actions))
    episode_rewards = []
    epsilon = epsilon_start

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
        epsilon = max(epsilon_min, epsilon * decay_rate)

    return Q, episode_rewards


env = gym.make('CliffWalking-v0')
Q_decay, rewards_decay = sarsa_decaying_epsilon(env, n_episodes=1000)
env.close()

print(f"Final ε: {max(0.01, 1.0 * 0.995**1000):.4f}")
print(f"Mean reward (last 100 episodes): {np.mean(rewards_decay[-100:]):.1f}")
```

As $\varepsilon$ decays toward zero, the agent transitions from a safe path to the optimal shortest path along the cliff edge — eventually matching Q-Learning's learned policy.

---

## 13. SARSA vs Q-Learning — Detailed Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                  ON-POLICY vs OFF-POLICY TD CONTROL                 │
│                                                                     │
│          SARSA                          Q-Learning                  │
│     ┌────────────┐                 ┌────────────────┐               │
│     │  Behavior:  │                 │   Behavior:     │              │
│     │  ε-greedy   │                 │   ε-greedy      │              │
│     │             │                 │                  │              │
│     │  Target:    │                 │   Target:        │              │
│     │  ε-greedy   │─── SAME ───     │   GREEDY ───     │── DIFFERENT  │
│     │  (same!)    │                 │   (max Q)        │              │
│     └─────┬──────┘                 └───────┬──────────┘              │
│           │                                │                         │
│           ▼                                ▼                         │
│   Q(S,A) + α[R + γQ(S',A') − Q(S,A)]     Q(S,A) + α[R + γ max Q(S',a') − Q(S,A)]
│                      ▲                                    ▲          │
│                      │                                    │          │
│              A' ~ ε-greedy                        max over all a'    │
│                                                                     │
│   Result: learns q_π (ε-greedy)        Result: learns q_* (optimal) │
│           (conservative/safe)                   (optimal/risky)      │
└─────────────────────────────────────────────────────────────────────┘
```

| Property | SARSA | Q-Learning |
|---|---|---|
| Update target | $R + \gamma Q(S', A')$ | $R + \gamma \max_{a'} Q(S', a')$ |
| Next action in update | $A' \sim \pi(\cdot \mid S')$ | $\arg\max_{a'} Q(S', a')$ |
| Policy type | On-policy | Off-policy |
| What it learns | $q_\pi$ | $q_*$ |
| Risk sensitivity | Aware of exploration risk | Ignores exploration risk |
| Online performance | Better (avoids penalties) | Worse (falls off cliffs) |
| Asymptotic policy | Optimal only with $\varepsilon \to 0$ | Always optimal |
| Cliff Walking path | Safe (top row) | Optimal (bottom edge) |

---

## 14. Variants and Extensions

### 14.1 Expected SARSA

Expected SARSA replaces the sampled next action value with the **expected value** under the policy:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[ R_{t+1} + \gamma \sum_{a'} \pi(a' \mid S_{t+1}) Q(S_{t+1}, a') - Q(S_t, A_t) \right]$$

This reduces variance (no randomness from $A_{t+1}$ selection) and generally performs better than standard SARSA. When $\pi$ is greedy ($\varepsilon = 0$), Expected SARSA reduces to Q-Learning.

### 14.2 n-Step SARSA

The one-step return can be extended to $n$ steps before bootstrapping:

$$G_{t:t+n} = R_{t+1} + \gamma R_{t+2} + \cdots + \gamma^{n-1} R_{t+n} + \gamma^n Q(S_{t+n}, A_{t+n})$$

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[ G_{t:t+n} - Q(S_t, A_t) \right]$$

This interpolates between one-step SARSA ($n=1$) and Monte Carlo ($n = \infty$).

---

## Summary

| Concept | Key Point |
|---|---|
| **SARSA Name** | State-Action-Reward-State-Action — the five values $(S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1})$ |
| **Update Rule** | $Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha[R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)]$ |
| **On-Policy** | Behavior policy = target policy = $\varepsilon$-greedy derived from $Q$ |
| **Relation to TD(0)** | SARSA is TD(0) applied to action-value functions for control |
| **TD Error** | $\delta_t = R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)$ |
| **Convergence** | Converges to $q_*$ under GLIE conditions (greedy in limit, infinite exploration) |
| **Cliff Walking** | SARSA learns a safe path along the top; Q-Learning learns the optimal but risky path |
| **Effect of ε** | Higher $\varepsilon$ → more conservative policy; $\varepsilon \to 0$ → optimal policy |
| **Risk Awareness** | On-policy nature makes SARSA account for the cost of exploration |
| **Backup Diagram** | One transition deep — backs up from $(S_t, A_t)$ to $(S_{t+1}, A_{t+1})$ |
| **Expected SARSA** | Uses $\mathbb{E}_\pi[Q(S', A')]$ instead of sampled $Q(S', A')$ — lower variance |

---

## Revision Questions

1. **Write out the SARSA update rule and explain why it requires five values $(S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1})$.** How does the need for $A_{t+1}$ differ from what TD(0) prediction requires?

2. **Explain what "on-policy" means in the context of SARSA.** Why does SARSA's on-policy nature cause it to learn a different path than Q-Learning in the Cliff Walking example?

3. **State the GLIE conditions for SARSA convergence to $q_*$.** Give a concrete $\varepsilon$-schedule that satisfies these conditions and explain why a constant $\varepsilon$ does not.

4. **Draw the Cliff Walking grid and the paths learned by SARSA and Q-Learning.** Which method achieves better online performance during training, and why?

5. **Compare the backup diagrams of SARSA and TD(0) prediction.** What additional node does SARSA's diagram include, and what role does it play in the update?

6. **Explain how Expected SARSA relates to both SARSA and Q-Learning.** Under what setting of the policy $\pi$ does Expected SARSA reduce to Q-Learning?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [TD(0) Prediction](./01-td0-prediction.md) | [Unit 5: TD Learning](./README.md) | [Q-Learning](./03-q-learning.md) |

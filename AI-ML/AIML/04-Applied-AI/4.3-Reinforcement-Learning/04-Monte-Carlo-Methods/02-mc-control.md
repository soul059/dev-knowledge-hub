# Chapter 2: Monte Carlo Control

> **Unit 4: Monte Carlo Methods** — *Finding optimal policies from sampled episodes without a model of the environment.*

---

## Overview

Monte Carlo **prediction** answers "how good is this policy?" by estimating $V^\pi(s)$.
Monte Carlo **control** answers the harder question: "what is the **best** policy?" It
does so by alternating between **policy evaluation** (estimating the value of the current
policy) and **policy improvement** (making the policy greedier). This is the Monte Carlo
instantiation of the powerful **Generalized Policy Iteration (GPI)** framework.

The critical challenge is that, without a model of the environment, we can no longer
improve a policy from $V^\pi$ alone — we need **action-value functions** $Q^\pi(s,a)$.
This chapter develops MC control algorithms step by step: first Monte Carlo with
**Exploring Starts** (MC ES), then practical **on-policy** control using $\varepsilon$-greedy
policies. Each algorithm is accompanied by rigorous pseudocode, mathematical justification,
and a working Python implementation.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║              MC CONTROL AT A GLANCE                             ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • Goal  : Find the optimal policy π* from sampled episodes    ║
 ║  • Method: GPI — alternate evaluation (Q) and improvement (π)  ║
 ║  • Key   : Must estimate Q(s,a), not V(s), for model-free use  ║
 ║  • MC ES : Guarantees exploration via exploring starts          ║
 ║  • On-pol: Uses ε-greedy policies for continual exploration     ║
 ║  • Model : Not required (model-free)                            ║
 ║  • Type  : Control (finding the best policy)                    ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. From Prediction to Control: Why Q(s,a)?

### 1.1 The Model Problem with V(s)

In DP, policy improvement selects the greedy action from $V^\pi$ using:

$$\pi'(s) = \arg\max_a \sum_{s'} P(s' \mid s,a)\left[R(s,a,s') + \gamma V^\pi(s')\right]$$

This requires the **model** — the transition probabilities $P(s' \mid s,a)$ and the
reward function $R(s,a,s')$. Without a model, we cannot evaluate which action leads to
which next state, so $V(s)$ alone is **insufficient** for policy improvement.

### 1.2 Action-Value Function Q(s,a)

The solution is to estimate the **action-value function** instead:

$$\boxed{Q^\pi(s,a) = \mathbb{E}_\pi\left[G_t \mid S_t = s, A_t = a\right]}$$

With $Q^\pi(s,a)$ available, the greedy policy requires **no model**:

$$\pi'(s) = \arg\max_a Q^\pi(s,a)$$

We simply pick the action with the highest estimated Q-value — no need to know where
actions lead. This is the fundamental reason MC control estimates $Q$ rather than $V$.

### 1.3 MC Estimation of Q(s,a)

The first-visit MC estimate of $Q(s,a)$ is the average of returns following the first
time we visit state $s$ and take action $a$ in each episode:

$$Q(s,a) \approx \frac{1}{N(s,a)} \sum_{i=1}^{N(s,a)} G_t^{(i)}$$

By the law of large numbers, $Q(s,a) \to Q^\pi(s,a)$ as $N(s,a) \to \infty$.

The complication: to estimate $Q(s,a)$ for **every** state-action pair, every pair must
be visited infinitely often. This is the **exploration problem**.

---

## 2. Generalized Policy Iteration (GPI)

### 2.1 The GPI Framework

GPI is the general idea of interacting processes of policy evaluation and policy
improvement. Virtually all RL methods can be described as GPI:

```
        ┌──────────────────────────────────────────┐
        │     GENERALIZED POLICY ITERATION (GPI)   │
        │                                          │
        │        π ──── evaluation ────► Q          │
        │        ▲                       │          │
        │        │                       │          │
        │   improvement              estimates     │
        │        │                       │          │
        │        └──── greedy w.r.t. ◄───┘          │
        │                                          │
        │   The policy is always being improved     │
        │   with respect to the current value       │
        │   function, and the value function is     │
        │   always being driven toward the value    │
        │   function for the current policy.        │
        └──────────────────────────────────────────┘
```

### 2.2 Evaluation and Improvement in MC

In DP-based GPI, evaluation runs iterative policy evaluation to convergence, then
improvement makes the policy greedy. In MC-based GPI:

- **Evaluation:** Use MC to estimate $Q^\pi$ from episodes generated under $\pi$
- **Improvement:** Make $\pi$ greedy (or $\varepsilon$-greedy) with respect to $Q$

The two processes push toward the same fixed point — the optimal policy $\pi^*$ and
optimal action-value function $Q^* = Q^{\pi^*}$:

$$\pi_0 \xrightarrow{E} Q^{\pi_0} \xrightarrow{I} \pi_1 \xrightarrow{E} Q^{\pi_1} \xrightarrow{I} \pi_2 \xrightarrow{E} \cdots \xrightarrow{I} \pi^* \xrightarrow{E} Q^*$$

### 2.3 Do We Need Full Evaluation?

In DP, we ran policy evaluation for many sweeps before improving. In MC, running many
episodes for full evaluation before each improvement step is wasteful. Instead, we can
improve the policy **after every episode** (or even within an episode). This works because
GPI does not require exact evaluation — any movement toward the true value function is
sufficient. The extreme case: update $Q$ from one episode, then improve $\pi$, then
generate the next episode under the new $\pi$.

---

## 3. Monte Carlo with Exploring Starts (MC ES)

### 3.1 The Exploring Starts Assumption

To guarantee that all state-action pairs are visited, MC ES assumes:

> **Exploring Starts:** Every state-action pair $(s,a)$ has a non-zero probability of
> being selected as the starting pair for an episode.

This means the environment (or simulator) must allow us to start episodes from any
state-action pair. While restrictive in practice, this assumption enables a clean
theoretical algorithm.

### 3.2 Algorithm Description

MC ES combines one-episode evaluation with immediate greedy improvement:

1. Generate an episode starting from a randomly chosen $(s_0, a_0)$
2. For each $(s,a)$ pair visited in the episode, compute the return $G$
3. Update $Q(s,a)$ using the first-visit average
4. For each state $s$ visited, set $\pi(s) \leftarrow \arg\max_a Q(s,a)$
5. Repeat

### 3.3 Pseudocode

```
Algorithm: Monte Carlo ES (Exploring Starts)
─────────────────────────────────────────────
Initialize:
    π(s) ∈ A(s)             arbitrarily, for all s ∈ S
    Q(s,a) ∈ ℝ              arbitrarily, for all s ∈ S, a ∈ A(s)
    Returns(s,a) ← empty list, for all s ∈ S, a ∈ A(s)

Repeat forever (for each episode):
    Choose S₀ ∈ S, A₀ ∈ A(S₀) such that all pairs have probability > 0
    Generate an episode from S₀, A₀ following π:
        S₀, A₀, R₁, S₁, A₁, R₂, ..., S_{T-1}, A_{T-1}, R_T

    G ← 0
    For t = T-1, T-2, ..., 0:
        G ← γ G + R_{t+1}
        Unless the pair (S_t, A_t) appears in (S₀,A₀), ..., (S_{t-1},A_{t-1}):
            Append G to Returns(S_t, A_t)
            Q(S_t, A_t) ← average(Returns(S_t, A_t))
            π(S_t) ← argmax_a Q(S_t, a)
```

### 3.4 Why MC ES Converges

Under the exploring starts assumption, each state-action pair is visited infinitely often
as the number of episodes grows. Since $Q(s,a)$ converges to $Q^\pi(s,a)$ by the law of
large numbers, and greedy improvement always yields a policy at least as good as the
current one (by the policy improvement theorem), the sequence of policies converges to
$\pi^*$. The formal proof requires care because the policy is changing between episodes,
but convergence has been established under reasonable conditions.

### 3.5 Limitations of Exploring Starts

In practice, exploring starts is often unrealistic:

- In a real robot, we cannot teleport to arbitrary states
- In a card game, we cannot start from an arbitrary hand configuration
- In many environments, the start state distribution is fixed

This motivates the development of methods that ensure exploration **without** the
exploring starts assumption — namely, $\varepsilon$-greedy policies.

---

## 4. On-Policy MC Control with ε-Greedy Policies

### 4.1 On-Policy vs Off-Policy (Preview)

MC control methods fall into two categories:

| Approach | Behavior Policy | Target Policy | Exploration |
|---|---|---|---|
| **On-policy** | Same as target | $\pi$ is both | Built into $\pi$ ($\varepsilon$-greedy) |
| **Off-policy** | Different from target | $\pi$ (greedy) | Via behavior policy $b$ |

On-policy methods are simpler and converge more reliably. Off-policy methods are more
powerful but require importance sampling (covered in the
[next chapter](./03-on-policy-vs-off-policy.md)).

### 4.2 The ε-Greedy Policy

An $\varepsilon$-greedy policy balances exploitation and exploration:

$$\boxed{\pi(a \mid s) = \begin{cases} 1 - \varepsilon + \dfrac{\varepsilon}{|\mathcal{A}(s)|} & \text{if } a = \arg\max_{a'} Q(s, a') \quad \text{(greedy action)} \\[8pt] \dfrac{\varepsilon}{|\mathcal{A}(s)|} & \text{if } a \neq \arg\max_{a'} Q(s, a') \quad \text{(non-greedy action)} \end{cases}}$$

where:
- $\varepsilon \in [0, 1]$ is the exploration parameter
- $|\mathcal{A}(s)|$ is the number of available actions in state $s$

**Properties of $\varepsilon$-greedy:**

- Every action has probability $\geq \varepsilon / |\mathcal{A}(s)| > 0$
- The greedy action gets extra probability mass: $1 - \varepsilon + \varepsilon/|\mathcal{A}(s)|$
- Probabilities sum to $1$: $(1 - \varepsilon + \varepsilon/|\mathcal{A}|) + (|\mathcal{A}| - 1) \cdot \varepsilon/|\mathcal{A}| = 1$
- At $\varepsilon = 0$: purely greedy (no exploration)
- At $\varepsilon = 1$: purely random (uniform over actions)

### 4.3 Probability Distribution Breakdown

For a state with $|\mathcal{A}| = 4$ actions and $\varepsilon = 0.1$:

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║         ε-GREEDY ACTION PROBABILITIES  (ε=0.1, |A|=4)          ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║                                                                 ║
 ║  Action      Type         Probability    Calculation            ║
 ║  ──────      ────         ───────────    ───────────            ║
 ║  a* (best)   greedy       0.925          1-0.1+0.1/4 = 0.925   ║
 ║  a₂          non-greedy   0.025          0.1/4       = 0.025   ║
 ║  a₃          non-greedy   0.025          0.1/4       = 0.025   ║
 ║  a₄          non-greedy   0.025          0.1/4       = 0.025   ║
 ║  ──────                   ───────────                           ║
 ║  Total                    1.000                                 ║
 ║                                                                 ║
 ║  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ █ █  ← a* = 92.5%   ║
 ║  █                                            ← a₂ =  2.5%    ║
 ║  █                                            ← a₃ =  2.5%    ║
 ║  █                                            ← a₄ =  2.5%    ║
 ║                                                                 ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 5. The ε-Greedy Policy Improvement Theorem

### 5.1 Statement

For any $\varepsilon$-greedy policy $\pi$ with respect to $Q^\pi$, the $\varepsilon$-greedy
policy $\pi'$ with respect to $Q^\pi$ satisfies:

$$\boxed{Q^\pi(s, \pi'(s)) \geq V^\pi(s) \quad \text{for all } s \in \mathcal{S}}$$

which by the policy improvement theorem implies $V^{\pi'}(s) \geq V^\pi(s)$ for all $s$.

### 5.2 Proof Sketch

Starting from the definition of $V^\pi$ under the new $\varepsilon$-greedy policy $\pi'$:

$$\sum_a \pi'(a \mid s) \, Q^\pi(s,a)$$

$$= \frac{\varepsilon}{|\mathcal{A}(s)|} \sum_a Q^\pi(s,a) + (1 - \varepsilon) \max_a Q^\pi(s,a)$$

Now, since $\max_a Q^\pi(s,a) \geq \sum_a \frac{\pi(a \mid s) - \varepsilon/|\mathcal{A}(s)|}{1 - \varepsilon} Q^\pi(s,a)$ (because the max is at least as large as any weighted average), we get:

$$\geq \frac{\varepsilon}{|\mathcal{A}(s)|} \sum_a Q^\pi(s,a) + (1 - \varepsilon) \sum_a \frac{\pi(a \mid s) - \varepsilon/|\mathcal{A}(s)|}{1 - \varepsilon} Q^\pi(s,a)$$

$$= \sum_a \pi(a \mid s) \, Q^\pi(s,a) = V^\pi(s)$$

Therefore $\sum_a \pi'(a \mid s) Q^\pi(s,a) \geq V^\pi(s)$, proving the improvement. Equality
holds for all $s$ only when both $\pi$ and $\pi'$ are already optimal among
$\varepsilon$-greedy policies — the **$\varepsilon$-optimal** policy.

### 5.3 What is an ε-Optimal Policy?

An $\varepsilon$-optimal policy is the best policy among all policies that are
$\varepsilon$-soft (i.e., $\pi(a \mid s) \geq \varepsilon / |\mathcal{A}(s)|$ for all
$s, a$). It is not the true optimal policy $\pi^*$, but the best we can achieve while
maintaining continual exploration. As $\varepsilon \to 0$, the $\varepsilon$-optimal
policy approaches $\pi^*$.

---

## 6. On-Policy First-Visit MC Control Algorithm

### 6.1 Complete Pseudocode

```
Algorithm: On-Policy First-Visit MC Control (ε-greedy)
──────────────────────────────────────────────────────
Initialize:
    π ← arbitrary ε-soft policy
    Q(s,a) ∈ ℝ               arbitrarily, for all s ∈ S, a ∈ A(s)
    Returns(s,a) ← empty list, for all s ∈ S, a ∈ A(s)

Parameter: ε > 0 (small)

Repeat forever (for each episode):
    Generate an episode following π:
        S₀, A₀, R₁, S₁, A₁, R₂, ..., S_{T-1}, A_{T-1}, R_T

    G ← 0
    For t = T-1, T-2, ..., 0:
        G ← γ G + R_{t+1}
        Unless the pair (S_t, A_t) appears in (S₀,A₀), ..., (S_{t-1},A_{t-1}):
            Append G to Returns(S_t, A_t)
            Q(S_t, A_t) ← average(Returns(S_t, A_t))
            A* ← argmax_a Q(S_t, a)      (break ties arbitrarily)
            For all a ∈ A(S_t):
                if a = A*:
                    π(a | S_t) ← 1 - ε + ε/|A(S_t)|
                else:
                    π(a | S_t) ← ε/|A(S_t)|
```

### 6.2 Key Differences from MC ES

| Feature | MC ES | On-Policy MC Control |
|---|---|---|
| Exploration mechanism | Exploring starts | $\varepsilon$-greedy policy |
| Exploring starts needed? | Yes | No |
| Policy type | Deterministic greedy | Stochastic $\varepsilon$-greedy |
| Converges to | $\pi^*$ (optimal) | $\pi_\varepsilon^*$ ($\varepsilon$-optimal) |
| Practical? | Often no | Yes |

### 6.3 Incremental Implementation

Storing all returns is memory-intensive. Using the incremental mean update:

$$Q(s,a) \leftarrow Q(s,a) + \frac{1}{N(s,a)}\bigl(G_t - Q(s,a)\bigr)$$

Or with a constant step-size $\alpha$ for non-stationary problems:

$$Q(s,a) \leftarrow Q(s,a) + \alpha\bigl(G_t - Q(s,a)\bigr)$$

---

## 7. Convergence Properties

### 7.1 Convergence of On-Policy MC Control

On-policy first-visit MC control with $\varepsilon$-greedy policies converges to the
$\varepsilon$-optimal policy $\pi_\varepsilon^*$ under the following conditions:

1. **All state-action pairs are visited infinitely often** — guaranteed by the
   $\varepsilon$-soft policy since every action has probability $\geq \varepsilon/|\mathcal{A}|$
2. **The policy converges** — guaranteed by the $\varepsilon$-greedy improvement theorem
3. **Step sizes satisfy Robbins-Monro conditions** (for the incremental version):
   $\sum_n \alpha_n = \infty$ and $\sum_n \alpha_n^2 < \infty$

The $1/N(s,a)$ step size satisfies these conditions. A constant $\alpha$ does not satisfy
the second condition, which means it will not converge to a fixed point but will continue
to fluctuate (useful for tracking non-stationary environments).

### 7.2 From ε-Optimal to Optimal

To achieve the true optimal policy, two practical approaches exist:

1. **GLIE (Greedy in the Limit with Infinite Exploration):** Decay $\varepsilon$ over
   time such that $\varepsilon_k \to 0$ as $k \to \infty$, but slowly enough that all
   pairs are still visited infinitely often (e.g., $\varepsilon_k = 1/k$).

2. **Off-policy methods:** Use a separate exploratory behavior policy while learning
   the optimal target policy (covered in the next chapter).

A GLIE schedule must satisfy:
- All state-action pairs are explored infinitely often: $\lim_{k \to \infty} N_k(s,a) = \infty$
- The policy converges to greedy: $\lim_{k \to \infty} \varepsilon_k = 0$

---

## 8. The GPI Interaction Diagram

The interplay between evaluation and improvement can be visualized as two competing
forces that drive the system toward the optimal solution:

```
           V, Q
            ▲
            │         ● (π*, Q*)  ← optimal fixed point
            │        ╱
            │       ╱
            │      ╱   ← the GPI "zigzag" path
            │     ╱
            │   ╱╱
            │  ╱╱
            │ ╱╱
            │╱╱
            ●  ← arbitrary start (π₀, Q₀)
            └──────────────────────────► π

    Horizontal axis: space of policies
    Vertical axis: space of value functions

    Evaluation pushes Q toward Q^π  (vertical movement)
    Improvement pushes π toward greedy w.r.t. Q  (horizontal movement)

    ┌─────────────────────────────────────────────────────────┐
    │  Evaluation:   Q ──→ Q^π     (make Q accurate for π)   │
    │  Improvement:  π ──→ greedy(Q) (make π better via Q)    │
    │                                                         │
    │  These two processes COMPETE:                           │
    │   • Evaluation tries to make Q match the current π      │
    │   • Improvement changes π, invalidating the current Q   │
    │                                                         │
    │  But they CONVERGE: the only stable point where          │
    │  neither process causes change is π* and Q*             │
    └─────────────────────────────────────────────────────────┘
```

---

## 9. Complete Python Implementation

The following implementation applies on-policy first-visit MC control with
$\varepsilon$-greedy policies to the Blackjack environment from Gymnasium.

```python
import numpy as np
from collections import defaultdict
import gymnasium as gym


def create_epsilon_greedy_policy(Q, epsilon, num_actions):
    """
    Creates an ε-greedy policy from a Q-function.

    Args:
        Q:           Dictionary mapping state -> array of action values
        epsilon:     Exploration parameter
        num_actions: Number of possible actions

    Returns:
        policy_fn: Function mapping state -> action probabilities
    """
    def policy_fn(state):
        action_values = Q[state]
        best_action = np.argmax(action_values)
        # ε/|A| probability for all actions
        probs = np.ones(num_actions) * epsilon / num_actions
        # Extra (1-ε) probability for the greedy action
        probs[best_action] += 1.0 - epsilon
        return probs

    return policy_fn


def generate_episode(env, policy_fn):
    """
    Generates a complete episode using the given policy.

    Returns:
        episode: List of (state, action, reward) tuples
    """
    episode = []
    state, _ = env.reset()
    done = False

    while not done:
        probs = policy_fn(state)
        action = np.random.choice(len(probs), p=probs)
        next_state, reward, terminated, truncated, _ = env.step(action)
        episode.append((state, action, reward))
        done = terminated or truncated
        state = next_state

    return episode


def on_policy_mc_control(env, num_episodes, gamma=1.0,
                         epsilon=0.1, num_actions=2):
    """
    On-policy first-visit MC control with ε-greedy policies.

    Args:
        env:          Gymnasium environment
        num_episodes: Number of episodes to run
        gamma:        Discount factor
        epsilon:      Exploration parameter
        num_actions:  Number of actions

    Returns:
        Q:      Final action-value function (dict: state -> array)
        policy: Final ε-greedy policy function
    """
    # Initialize Q(s,a) arbitrarily
    Q = defaultdict(lambda: np.zeros(num_actions))
    N = defaultdict(lambda: np.zeros(num_actions))  # Visit counts

    for ep in range(num_episodes):
        # Create policy from current Q
        policy_fn = create_epsilon_greedy_policy(Q, epsilon, num_actions)

        # Generate an episode following the current policy
        episode = generate_episode(env, policy_fn)

        # Compute returns and update Q (first-visit)
        G = 0.0
        visited = set()

        for t in range(len(episode) - 1, -1, -1):
            state, action, reward = episode[t]
            G = gamma * G + reward

            sa_pair = (state, action)
            if sa_pair not in visited:
                visited.add(sa_pair)
                N[state][action] += 1.0
                # Incremental mean update
                Q[state][action] += (
                    (G - Q[state][action]) / N[state][action]
                )

    # Return final policy and Q
    final_policy = create_epsilon_greedy_policy(Q, epsilon, num_actions)
    return Q, final_policy


# ========================================
# Run On-Policy MC Control on Blackjack
# ========================================
env = gym.make("Blackjack-v1", sab=True)
num_actions = env.action_space.n  # 2: stick (0) or hit (1)

print("Running on-policy first-visit MC control on Blackjack...")
print(f"Parameters: ε=0.1, γ=1.0, episodes=500,000\n")

Q, policy = on_policy_mc_control(
    env,
    num_episodes=500000,
    gamma=1.0,
    epsilon=0.1,
    num_actions=num_actions,
)

# Display learned policy for some states
print("Learned policy (0=Stick, 1=Hit):")
print("-" * 65)

sample_states = [
    (21, 10, False),
    (20, 6, False),
    (18, 10, False),
    (15, 10, False),
    (12, 3, False),
    (13, 2, True),
]

for s in sample_states:
    if s in Q:
        probs = policy(s)
        best = np.argmax(Q[s])
        action_name = "Stick" if best == 0 else "Hit"
        print(
            f"  State {str(s):>20s}  →  {action_name:>5s}  "
            f"(Q[stick]={Q[s][0]:+.3f}, Q[hit]={Q[s][1]:+.3f})"
        )

# Estimate win rate under the learned policy
print("\nEstimating win rate under learned policy (10,000 test episodes)...")
wins, draws, losses = 0, 0, 0
test_policy = create_epsilon_greedy_policy(Q, 0.0, num_actions)  # greedy

for _ in range(10000):
    ep = generate_episode(env, test_policy)
    final_reward = ep[-1][2]
    if final_reward > 0:
        wins += 1
    elif final_reward == 0:
        draws += 1
    else:
        losses += 1

print(f"  Wins:   {wins / 100:.1f}%")
print(f"  Draws:  {draws / 100:.1f}%")
print(f"  Losses: {losses / 100:.1f}%")

env.close()
```

**Expected output (approximate):**

```
Running on-policy first-visit MC control on Blackjack...
Parameters: ε=0.1, γ=1.0, episodes=500,000

Learned policy (0=Stick, 1=Hit):
-----------------------------------------------------------------
  State     (21, 10, False)  →  Stick  (Q[stick]=+0.872, Q[hit]=-0.563)
  State      (20, 6, False)  →  Stick  (Q[stick]=+0.701, Q[hit]=-0.482)
  State     (18, 10, False)  →  Stick  (Q[stick]=-0.182, Q[hit]=-0.564)
  State     (15, 10, False)  →    Hit  (Q[stick]=-0.465, Q[hit]=-0.398)
  State      (12, 3, False)  →    Hit  (Q[stick]=-0.253, Q[hit]=-0.210)
  State      (13, 2, True)   →    Hit  (Q[stick]=-0.188, Q[hit]=-0.091)

Estimating win rate under learned policy (10,000 test episodes)...
  Wins:   43.5%
  Draws:   8.9%
  Losses: 47.6%
```

### How the Code Connects to the Theory

| Code Element | Theory Concept |
|---|---|
| `Q = defaultdict(lambda: np.zeros(num_actions))` | Initialize $Q(s,a)$ arbitrarily |
| `epsilon / num_actions` and `1.0 - epsilon` | $\varepsilon$-greedy: $\varepsilon/|\mathcal{A}|$ and $1-\varepsilon+\varepsilon/|\mathcal{A}|$ |
| `np.random.choice(len(probs), p=probs)` | Sample action from $\pi(\cdot \mid s)$ |
| `G = gamma * G + reward` | Backward return: $G_t = R_{t+1} + \gamma G_{t+1}$ |
| `if sa_pair not in visited` | First-visit check for $(S_t, A_t)$ |
| `Q[state][action] += (G - Q[state][action]) / N[state][action]` | Incremental mean: $Q \leftarrow Q + \frac{1}{N}(G - Q)$ |
| `create_epsilon_greedy_policy(Q, epsilon, ...)` | Policy improvement: $\pi \leftarrow \varepsilon\text{-greedy}(Q)$ |
| Outer `for ep in range(num_episodes)` loop | GPI: alternate evaluation and improvement each episode |

---

## 10. MC ES Implementation for Blackjack

For comparison, here is a Monte Carlo Exploring Starts implementation:

```python
import numpy as np
from collections import defaultdict
import gymnasium as gym


def mc_exploring_starts(env, num_episodes, gamma=1.0, num_actions=2):
    """
    Monte Carlo ES (Exploring Starts) for Blackjack.

    Uses random initial state-action pairs and greedy improvement.
    """
    Q = defaultdict(lambda: np.zeros(num_actions))
    N = defaultdict(lambda: np.zeros(num_actions))
    # Deterministic policy: π(s) -> action
    pi = defaultdict(lambda: np.random.randint(num_actions))

    for ep in range(num_episodes):
        # Exploring start: first action is chosen randomly
        state, _ = env.reset()
        first_action = np.random.randint(num_actions)

        # Execute first action
        episode = []
        next_state, reward, terminated, truncated, _ = env.step(first_action)
        episode.append((state, first_action, reward))
        done = terminated or truncated
        state = next_state

        # Follow deterministic policy for rest of episode
        while not done:
            action = pi[state]
            next_state, reward, terminated, truncated, _ = env.step(action)
            episode.append((state, action, reward))
            done = terminated or truncated
            state = next_state

        # Update Q and improve policy
        G = 0.0
        visited = set()
        for t in range(len(episode) - 1, -1, -1):
            s, a, r = episode[t]
            G = gamma * G + r
            if (s, a) not in visited:
                visited.add((s, a))
                N[s][a] += 1.0
                Q[s][a] += (G - Q[s][a]) / N[s][a]
                # Greedy improvement (deterministic)
                pi[s] = np.argmax(Q[s])

    return Q, pi


env = gym.make("Blackjack-v1", sab=True)
print("Running MC Exploring Starts on Blackjack (500,000 episodes)...")
Q_es, pi_es = mc_exploring_starts(env, num_episodes=500000)

print("\nLearned policy (MC ES):")
for s in [(20, 10, False), (18, 10, False), (15, 10, False), (12, 3, False)]:
    if s in Q_es:
        action_name = "Stick" if pi_es[s] == 0 else "Hit"
        print(f"  {str(s):>20s} → {action_name}")

env.close()
```

---

## 11. Comparison of MC Control Methods

| Feature | MC ES | On-Policy ($\varepsilon$-greedy) |
|---|---|---|
| **Exploration** | Via exploring starts | Via $\varepsilon$-greedy randomness |
| **Policy type** | Deterministic | Stochastic ($\varepsilon$-soft) |
| **Converges to** | $\pi^*$ | $\pi_\varepsilon^*$ ($\varepsilon$-optimal) |
| **Practical** | Rarely (need start control) | Yes (no assumptions on starts) |
| **Bias** | Unbiased $Q$ | Unbiased $Q$ |
| **Variance** | High (single trajectories) | High (single trajectories) |
| **Can reach $\pi^*$?** | Yes | Only with $\varepsilon \to 0$ (GLIE) |
| **Complexity** | Same | Same |

---

## 12. Common Pitfalls and Practical Tips

### 12.1 Insufficient Exploration

If $\varepsilon$ is too small (or decays too fast), many state-action pairs may never
be explored. The policy can get stuck in a local optimum. Start with a larger $\varepsilon$
(e.g., 0.2–0.5) and decay it gradually.

### 12.2 High Variance of Returns

MC methods have inherently high variance because each return $G_t$ depends on an entire
trajectory of stochastic transitions. Practical remedies:

- **More episodes:** Variance of the mean decreases as $O(1/N)$
- **Discount factor $\gamma < 1$:** Reduces the effective horizon
- **Constant step-size $\alpha$:** Gives more weight to recent episodes (useful when
  the policy is changing rapidly in early training)

### 12.3 Memory Efficiency

The textbook algorithm stores all returns in lists. For large problems, use the
incremental mean update instead — it requires only $Q(s,a)$ and $N(s,a)$ storage.

### 12.4 Tie-Breaking in argmax

When multiple actions have equal Q-values (especially at the start when all are zero),
break ties **randomly** to avoid systematic bias toward a particular action index.

---

## 13. Summary

| Concept | Key Point |
|---|---|
| Why $Q(s,a)$ not $V(s)$ | $V(s)$ requires a model for policy improvement; $Q(s,a)$ does not |
| GPI | Alternate evaluation ($Q \to Q^\pi$) and improvement ($\pi \to \text{greedy}(Q)$) |
| MC ES | Explores via random starting state-action pairs; converges to $\pi^*$ |
| Exploring starts | Assumes every $(s,a)$ pair can start an episode — often impractical |
| $\varepsilon$-greedy | Greedy action gets $1-\varepsilon+\varepsilon/|\mathcal{A}|$; others get $\varepsilon/|\mathcal{A}|$ |
| $\varepsilon$-greedy improvement | $\sum_a \pi'(a \mid s) Q^\pi(s,a) \geq V^\pi(s)$ — guaranteed improvement |
| On-policy MC control | Uses $\varepsilon$-greedy for both behavior and target; no model needed |
| $\varepsilon$-optimal policy | Best policy among all $\varepsilon$-soft policies |
| GLIE | Decay $\varepsilon \to 0$ to converge to true $\pi^*$ |
| Incremental update | $Q(s,a) \leftarrow Q(s,a) + \frac{1}{N}(G - Q(s,a))$ — memory efficient |
| First-visit check | Only update on first occurrence of $(s,a)$ per episode |
| Convergence | To $\pi_\varepsilon^*$ (on-policy) or $\pi^*$ (MC ES / GLIE) |

---

## Revision Questions

1. **Explain why $V(s)$ is insufficient for policy improvement without a model.** Write the DP policy improvement formula and identify which terms require knowledge of $P(s' \mid s,a)$. Then show how using $Q(s,a)$ eliminates this dependency.

2. **Derive the $\varepsilon$-greedy probability distribution.** For a state with 3 actions and $\varepsilon = 0.3$, compute the exact probability of selecting the greedy action and each non-greedy action. Verify the probabilities sum to 1.

3. **Prove the $\varepsilon$-greedy policy improvement theorem.** Starting from $\sum_a \pi'(a \mid s) Q^\pi(s,a)$, show step by step that this quantity is $\geq V^\pi(s)$.

4. **Compare MC ES and on-policy MC control.** What assumption does MC ES make that on-policy methods avoid? What does on-policy sacrifice in return (hint: what does it converge to)?

5. **A colleague suggests setting $\varepsilon = 0$ to get the optimal policy directly.** Explain why this breaks the MC control algorithm. What exploration guarantee is lost, and how does this affect convergence of $Q$ estimates?

6. **Describe the GPI framework and how MC control implements it.** Draw the evaluation-improvement cycle and explain what would happen if you only did evaluation (without improvement) or only did improvement (without evaluation).

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [MC Prediction](./01-mc-prediction.md) | [Unit 4: Monte Carlo Methods](./README.md) | [On-Policy vs Off-Policy](./03-on-policy-vs-off-policy.md) |

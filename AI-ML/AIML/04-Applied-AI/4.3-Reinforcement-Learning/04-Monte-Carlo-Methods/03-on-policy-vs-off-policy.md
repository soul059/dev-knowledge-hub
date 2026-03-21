# Chapter 3: On-Policy vs Off-Policy Methods

> **Unit 4: Monte Carlo Methods** — *Understanding the two fundamental paradigms for learning from experience: learning about the policy you follow versus learning about a different policy entirely.*

---

## Overview

Every reinforcement learning algorithm must answer a deceptively simple question: **is the
policy generating the data the same policy we are trying to improve?** This single design
choice splits the entire landscape of RL algorithms into two paradigms — **on-policy** and
**off-policy** methods. On-policy methods learn the value of the policy currently being used
to make decisions (the **behavior policy** and the **target policy** are identical). Off-policy
methods learn the value of a **different** policy from the one generating the data, enabling
powerful capabilities such as learning from demonstrations, reusing old experience, and
separating exploration from exploitation entirely.

This chapter formalizes both paradigms, establishes the mathematical foundations that make
off-policy learning possible (including the critical **coverage assumption**), and develops
concrete algorithms for each. We compare their trade-offs in depth and build intuition
through worked examples, ASCII diagrams, and side-by-side algorithmic analysis.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║          ON-POLICY  vs  OFF-POLICY  AT A GLANCE                 ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║                                                                 ║
 ║  On-Policy (b = π)              Off-Policy (b ≠ π)             ║
 ║  ─────────────────              ──────────────────              ║
 ║  • Same policy generates        • Behavior policy b generates  ║
 ║    data and is improved            data; target policy π is     ║
 ║  • Simpler algorithms              improved separately         ║
 ║  • Explore via soft policies    • Can learn optimal policy      ║
 ║    (e.g., ε-greedy)               while exploring freely       ║
 ║  • Converges to best soft       • Requires importance sampling ║
 ║    policy, not truly optimal      or other correction methods   ║
 ║  • Examples: SARSA, MC ES       • Examples: Q-learning, MC     ║
 ║                                   with importance sampling     ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. Two Policies: Behavior and Target

### 1.1 Definitions

Every learning algorithm involves at least one policy, but we must distinguish between
**two conceptually different roles** that a policy can play:

| Policy | Symbol | Role |
|---|---|---|
| **Target policy** | $\pi$ | The policy we want to learn about (evaluate or improve) |
| **Behavior policy** | $b$ | The policy actually used to select actions and generate experience |

The **target policy** $\pi$ is the one whose value function we are estimating or optimizing.
The **behavior policy** $b$ is the one the agent actually follows while interacting with the
environment.

### 1.2 The Fundamental Dichotomy

$$\boxed{
\text{On-Policy: } b = \pi \qquad\qquad \text{Off-Policy: } b \neq \pi
}$$

- **On-policy:** The agent learns about the very policy it is using. Every action selection
  simultaneously serves two purposes — generating data and executing the policy under evaluation.

- **Off-policy:** The agent uses one policy $b$ to generate trajectories but evaluates (or
  improves) a **different** policy $\pi$. The behavior and target policies are decoupled.

### 1.3 Data Flow: On-Policy vs Off-Policy

```
  ON-POLICY (b = π)
  ─────────────────────────────────────────────────────────────────

    ┌──────────┐      actions      ┌─────────────┐
    │  Policy  │ ───────────────►  │ Environment │
    │  π = b   │                   │             │
    │          │ ◄───────────────  │             │
    └──────────┘   (s, a, r, s')   └─────────────┘
         │
         │  same data used to
         │  update the same policy
         ▼
    ┌──────────────┐
    │ Update π     │
    │ (evaluation  │
    │  + improve)  │
    └──────────────┘


  OFF-POLICY (b ≠ π)
  ─────────────────────────────────────────────────────────────────

    ┌──────────┐      actions      ┌─────────────┐
    │ Behavior │ ───────────────►  │ Environment │
    │ Policy b │                   │             │
    │ (e.g.,   │ ◄───────────────  │             │
    │ random)  │   (s, a, r, s')   └─────────────┘
    └──────────┘
         │
         │  data from b is used
         │  to update a DIFFERENT
         │  policy (with correction)
         ▼
    ┌──────────────┐     correction     ┌──────────────┐
    │  Experience  │ ─────────────────► │ Update π     │
    │   Buffer     │  (importance       │ (target      │
    │              │   sampling or      │  policy)     │
    └──────────────┘   other method)    └──────────────┘
```

---

## 2. On-Policy Methods

### 2.1 Core Idea

In on-policy learning, the agent evaluates and improves the **same** policy it uses to
interact with the environment:

$$Q^\pi(s, a) = \mathbb{E}_\pi\left[G_t \mid S_t = s, A_t = a\right]$$

Because $b = \pi$, the returns $G_t$ observed under the behavior policy are **directly**
valid estimates of the target policy's value. No correction is needed — the data distribution
matches the target distribution exactly.

### 2.2 The Exploration-Exploitation Dilemma

On-policy methods face an inherent tension. To **estimate** the value of all state-action
pairs accurately, the agent must **explore** — visit every $(s, a)$ pair infinitely often.
But to **exploit** its knowledge and act optimally, the agent should pick the best action
every time. Since the same policy must do both, on-policy methods resolve this tension
by using **soft policies**.

### 2.3 Soft Policies

A policy $\pi$ is called **soft** (or **$\varepsilon$-soft**) if it assigns nonzero
probability to every action in every state:

$$\pi(a \mid s) > 0 \quad \forall\, s \in \mathcal{S}, \; a \in \mathcal{A}(s)$$

More specifically, an **$\varepsilon$-soft** policy satisfies:

$$\pi(a \mid s) \geq \frac{\varepsilon}{|\mathcal{A}(s)|} \quad \forall\, s, a$$

This guarantees continued exploration. The most common $\varepsilon$-soft policy is the
**$\varepsilon$-greedy** policy:

$$\pi(a \mid s) = \begin{cases} 1 - \varepsilon + \dfrac{\varepsilon}{|\mathcal{A}(s)|} & \text{if } a = \arg\max_{a'} Q(s, a') \\[8pt] \dfrac{\varepsilon}{|\mathcal{A}(s)|} & \text{otherwise} \end{cases}$$

### 2.4 On-Policy MC Control (ε-greedy)

The on-policy Monte Carlo control algorithm estimates $Q^\pi$ using first-visit MC and
improves $\pi$ by making it $\varepsilon$-greedy with respect to $Q$:

```
  Algorithm: On-Policy First-Visit MC Control (ε-greedy)
  ───────────────────────────────────────────────────────
  Initialize:
      Q(s, a) ← arbitrary, for all s ∈ S, a ∈ A(s)
      Returns(s, a) ← empty list, for all s, a
      π ← ε-greedy policy derived from Q

  Repeat forever (for each episode):
      1. Generate episode S₀,A₀,R₁,S₁,A₁,R₂,...,S_{T-1},A_{T-1},R_T
         following π
      2. G ← 0
      3. For t = T-1, T-2, ..., 0:
             G ← γG + R_{t+1}
             If (Sₜ, Aₜ) not in {(S₀,A₀), ..., (S_{t-1},A_{t-1})}:
                 Append G to Returns(Sₜ, Aₜ)
                 Q(Sₜ, Aₜ) ← average(Returns(Sₜ, Aₜ))
      4. For each s in the episode:
             A* ← argmax_a Q(s, a)
             For all a ∈ A(s):
                 π(a|s) ← ε/|A(s)| + (1 - ε) · 𝟙(a = A*)
```

### 2.5 On-Policy TD Control: SARSA

SARSA is the prototypical on-policy **temporal-difference** control algorithm. Unlike MC,
it updates after every step (not only at episode end):

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[R_{t+1} + \gamma\, Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)\right]$$

The name **SARSA** comes from the quintuple $(S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1})$ used
in each update. Note that $A_{t+1}$ is selected by the **same** policy being learned —
this is what makes SARSA on-policy.

### 2.6 Convergence of On-Policy Methods

On-policy MC control converges to $\pi_\varepsilon^*$ — the **best $\varepsilon$-soft
policy** — not the truly optimal policy $\pi^*$. This is because the agent can never
fully commit to a greedy action; it must always reserve probability $\varepsilon$ for
exploration:

$$V^{\pi_\varepsilon^*}(s) \leq V^{\pi^*}(s) \quad \forall\, s$$

> **Key Insight:** On-policy methods trade optimality for simplicity. The policy converges
> to the best policy *within the class of $\varepsilon$-soft policies*, which is generally
> sub-optimal compared to the unconstrained optimal policy $\pi^*$.

To approach $\pi^*$, one can use **GLIE** (Greedy in the Limit with Infinite Exploration)
schedules that reduce $\varepsilon \to 0$ over time while ensuring all state-action pairs
are visited infinitely often.

---

## 3. Off-Policy Methods

### 3.1 Core Idea

Off-policy methods decouple the policy that generates experience ($b$) from the policy
being learned ($\pi$):

$$Q^\pi(s, a) = \mathbb{E}_b\left[\rho_{t:T-1}\, G_t \mid S_t = s, A_t = a\right]$$

where $\rho_{t:T-1}$ is the **importance sampling ratio** that corrects for the mismatch
between $b$ and $\pi$.

### 3.2 Why Off-Policy Learning?

Off-policy methods are more complex but offer capabilities that on-policy methods cannot:

| Capability | Explanation |
|---|---|
| **Learn from demonstrations** | Set $b$ = expert policy, learn $\pi$ from expert trajectories |
| **Learn from old data** | Reuse experience collected by previous policies |
| **Parallel learning** | Learn multiple target policies from the same stream of experience |
| **Truly optimal policy** | $\pi$ can be fully greedy — no exploration compromise |
| **Exploration separation** | $b$ handles exploration; $\pi$ handles exploitation |
| **Experience replay** | Store transitions in a buffer, replay to update $\pi$ |

### 3.3 The Coverage Assumption

For off-policy learning to work, the behavior policy must **cover** the target policy.
Formally:

$$\boxed{\pi(a \mid s) > 0 \implies b(a \mid s) > 0 \quad \forall\, s \in \mathcal{S}, \; a \in \mathcal{A}(s)}$$

In words: every action that $\pi$ might take must also have nonzero probability under $b$.
If $\pi$ would select an action that $b$ never takes, we can never observe the consequences
of that action and therefore cannot estimate its value.

> **Key Insight:** Coverage does **not** require $b$ to match $\pi$ — only that $b$'s
> support (the set of actions with nonzero probability) is at least as broad as $\pi$'s.
> A fully random policy $b$ (uniform over all actions) trivially satisfies coverage for
> any target policy $\pi$.

### 3.4 Importance Sampling Ratio

When data is generated by $b$ but we want to evaluate $\pi$, we correct each trajectory
by the **importance sampling ratio**:

$$\rho_{t:T-1} = \prod_{k=t}^{T-1} \frac{\pi(A_k \mid S_k)}{b(A_k \mid S_k)}$$

This ratio re-weights each trajectory to account for the probability mismatch between
the two policies. If $\pi$ and $b$ agree on a trajectory, $\rho = 1$. If $\pi$ is more
likely to take the observed actions, $\rho > 1$; if less likely, $\rho < 1$.

The **ordinary importance sampling** estimator of $V^\pi(s)$ is:

$$V^\pi(s) \approx \frac{1}{N(s)} \sum_{i=1}^{N(s)} \rho_{t_i:T_i-1}\, G_{t_i}$$

The **weighted importance sampling** estimator (lower variance, biased but consistent):

$$V^\pi(s) \approx \frac{\sum_{i=1}^{N(s)} \rho_{t_i:T_i-1}\, G_{t_i}}{\sum_{i=1}^{N(s)} \rho_{t_i:T_i-1}}$$

For details, see [Importance Sampling](./04-importance-sampling.md).

### 3.5 Off-Policy MC Prediction

```
  Algorithm: Off-Policy MC Prediction (Weighted Importance Sampling)
  ──────────────────────────────────────────────────────────────────
  Input: target policy π, behavior policy b (with coverage)
  Initialize:
      Q(s, a) ← arbitrary, for all s, a
      C(s, a) ← 0, for all s, a          (cumulative weight)

  Repeat forever (for each episode):
      1. Generate episode following b:
         S₀,A₀,R₁,...,S_{T-1},A_{T-1},R_T
      2. G ← 0
      3. W ← 1    (importance sampling weight)
      4. For t = T-1, T-2, ..., 0:
             G ← γG + R_{t+1}
             C(Sₜ, Aₜ) ← C(Sₜ, Aₜ) + W
             Q(Sₜ, Aₜ) ← Q(Sₜ, Aₜ) + (W / C(Sₜ,Aₜ)) · [G - Q(Sₜ,Aₜ)]
             W ← W · π(Aₜ|Sₜ) / b(Aₜ|Sₜ)
             If W = 0 then break   (π would not take this action)
```

### 3.6 Off-Policy TD Control: Q-Learning

**Q-learning** is the most famous off-policy algorithm. It learns the optimal action-value
function $Q^*$ directly, regardless of the behavior policy:

$$\boxed{Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \left[R_{t+1} + \gamma \max_a Q(S_{t+1}, a) - Q(S_t, A_t)\right]}$$

The key difference from SARSA: the update uses $\max_a Q(S_{t+1}, a)$ rather than
$Q(S_{t+1}, A_{t+1})$. The **max** operator means the update target always corresponds
to the **greedy** policy (the target policy $\pi$), regardless of which action $b$
actually selects next. This is off-policy because:

- **Target policy** $\pi$: greedy with respect to $Q$ (implicit via the max)
- **Behavior policy** $b$: any exploratory policy (e.g., $\varepsilon$-greedy)

> **Key Insight:** Q-learning does not need explicit importance sampling ratios because
> the one-step max operator implicitly corrects for the off-policy data. This is a special
> property of one-step methods with the max operator — multi-step off-policy methods
> generally do require importance sampling.

---

## 4. Mathematical Comparison

### 4.1 Update Rules Side by Side

**SARSA (On-Policy):**

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \bigl[R_{t+1} + \gamma\, Q(S_{t+1}, \underbrace{A_{t+1}}_{\text{from } \pi}) - Q(S_t, A_t)\bigr]$$

**Q-Learning (Off-Policy):**

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \bigl[R_{t+1} + \gamma\, \underbrace{\max_a Q(S_{t+1}, a)}_{\text{greedy } \pi} - Q(S_t, A_t)\bigr]$$

The **only** difference is in the bootstrap target:
- SARSA uses the action **actually taken** by the current policy → on-policy
- Q-learning uses the action that **maximizes** $Q$ → off-policy (target is greedy)

### 4.2 Expected Value Under Each Paradigm

For the on-policy case, the expected TD target is:

$$\mathbb{E}_\pi[R_{t+1} + \gamma\, Q(S_{t+1}, A_{t+1}) \mid S_t, A_t] = \sum_{s'} P(s' \mid s,a)\left[R(s,a,s') + \gamma \sum_{a'} \pi(a' \mid s')\, Q(s', a')\right]$$

For the off-policy case (Q-learning):

$$\mathbb{E}_b[R_{t+1} + \gamma\, \max_{a'} Q(S_{t+1}, a') \mid S_t, A_t] = \sum_{s'} P(s' \mid s,a)\left[R(s,a,s') + \gamma \max_{a'} Q(s', a')\right]$$

Note that the Q-learning target does **not** depend on $b$ at all — the max operator
removes the dependence on which action $b$ selects.

---

## 5. The Exploration–Exploitation Dilemma Revisited

### 5.1 On-Policy Approach

On-policy methods **merge** exploration and exploitation into a single policy:

```
  ┌──────────────────────────────────────────────────┐
  │          EXPLORATION–EXPLOITATION IN ON-POLICY   │
  │                                                  │
  │   ┌─────────────────────────────────┐            │
  │   │        Single Policy π          │            │
  │   │                                 │            │
  │   │  ┌───────────┐ ┌────────────┐   │            │
  │   │  │ Exploit   │ │  Explore   │   │            │
  │   │  │ (1 - ε)   │ │  (ε)       │   │            │
  │   │  │ greedy    │ │  random    │   │            │
  │   │  └───────────┘ └────────────┘   │            │
  │   │                                 │            │
  │   │  Both roles MUST coexist in     │            │
  │   │  the same policy → compromise   │            │
  │   └─────────────────────────────────┘            │
  │                                                  │
  │  Consequence: π can never be fully greedy        │
  │  Best achievable: π*_ε (optimal ε-soft policy)   │
  └──────────────────────────────────────────────────┘
```

### 5.2 Off-Policy Approach

Off-policy methods **separate** exploration and exploitation into two policies:

```
  ┌──────────────────────────────────────────────────┐
  │         EXPLORATION–EXPLOITATION IN OFF-POLICY   │
  │                                                  │
  │   ┌──────────────┐      ┌──────────────┐         │
  │   │ Behavior     │      │ Target       │         │
  │   │ Policy b     │      │ Policy π     │         │
  │   │              │      │              │         │
  │   │  Explore     │      │  Exploit     │         │
  │   │  (ε-greedy,  │      │  (greedy,    │         │
  │   │   random,    │      │   optimal)   │         │
  │   │   curious)   │      │              │         │
  │   └──────────────┘      └──────────────┘         │
  │         │                      ▲                  │
  │         │   data + correction  │                  │
  │         └──────────────────────┘                  │
  │                                                  │
  │  Consequence: π CAN be fully greedy              │
  │  Best achievable: π* (truly optimal policy)      │
  └──────────────────────────────────────────────────┘
```

---

## 6. Advantages and Disadvantages

### 6.1 On-Policy: Pros and Cons

**Advantages:**
- **Simplicity** — no importance sampling, no coverage requirements, straightforward updates
- **Lower variance** — returns are directly from the policy being evaluated; no ratio products
- **Stability** — tends to be more numerically stable in practice
- **Safer behavior** — the learned policy accounts for its own exploration, yielding safer
  trajectories (important in robotics, autonomous driving)
- **Natural for continuing tasks** — works seamlessly without episode boundaries

**Disadvantages:**
- **Sub-optimal convergence** — converges to $\pi_\varepsilon^*$, not $\pi^*$ (unless $\varepsilon \to 0$)
- **Cannot reuse old data** — past episodes generated by old policies are invalid; data
  efficiency is lower
- **Exploration is constrained** — the same policy must both explore and exploit
- **No learning from demonstrations** — cannot leverage expert data directly

### 6.2 Off-Policy: Pros and Cons

**Advantages:**
- **Optimal convergence** — can converge to $\pi^*$ (the truly optimal policy)
- **Data reuse** — can replay old experience from a buffer (experience replay)
- **Learning from demonstrations** — can learn from expert trajectories, human demonstrations,
  or any pre-recorded data
- **Parallel learning** — can evaluate multiple target policies from a single behavior stream
- **Flexible exploration** — behavior policy can use any exploration strategy independently

**Disadvantages:**
- **Higher variance** — importance sampling ratios can have very high variance, especially
  over long episodes where ratios are products of many terms
- **Coverage requirement** — behavior policy must cover the target policy
- **Complexity** — more complex algorithms with additional hyperparameters
- **Potential instability** — the "deadly triad" (off-policy + function approximation +
  bootstrapping) can cause divergence
- **Slower convergence** — high variance of importance sampling can slow learning significantly

---

## 7. Comprehensive Comparison Table

| Aspect | On-Policy | Off-Policy |
|---|---|---|
| **Relationship** | $b = \pi$ (same policy) | $b \neq \pi$ (different policies) |
| **Target policy** | $\varepsilon$-soft (must explore) | Can be fully greedy (deterministic) |
| **Behavior policy** | Same as target | Any policy with coverage |
| **Convergence target** | $\pi_\varepsilon^*$ (best soft policy) | $\pi^*$ (truly optimal policy) |
| **Importance sampling** | Not needed | Required (MC) or implicit (Q-learning) |
| **Variance** | Lower (direct samples) | Higher (importance sampling products) |
| **Data efficiency** | Lower (cannot reuse old data) | Higher (experience replay possible) |
| **Algorithm complexity** | Simpler | More complex |
| **Stability** | More stable in practice | Risk of divergence (deadly triad) |
| **Exploration strategy** | Embedded in policy ($\varepsilon$-greedy) | Decoupled (can be anything) |
| **Learning from demos** | Not directly possible | Natural capability |
| **Safety** | Learns safe exploratory behavior | Target policy ignores exploration cost |
| **MC example** | On-policy first-visit MC | Off-policy MC with importance sampling |
| **TD example** | SARSA | Q-learning |
| **Deep RL example** | A3C, PPO | DQN, SAC, DDPG |
| **Best for** | Safety-critical, stable learning | Data efficiency, optimal performance |

---

## 8. Concrete Algorithm Examples

### 8.1 Example: On-Policy — ε-greedy SARSA

Consider a gridworld with 4 actions (up, down, left, right), $\varepsilon = 0.1$,
$\alpha = 0.1$, $\gamma = 0.99$.

**Step-by-step update:**

1. Agent is in state $S_t$, selects action $A_t$ using $\varepsilon$-greedy on $Q$
2. Environment returns reward $R_{t+1}$ and next state $S_{t+1}$
3. Agent selects $A_{t+1}$ using $\varepsilon$-greedy on $Q$ (same policy!)
4. Update:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + 0.1 \left[R_{t+1} + 0.99 \cdot Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)\right]$$

**Numerical example:** Suppose $Q(S_t, A_t) = 2.0$, $R_{t+1} = -1$, $Q(S_{t+1}, A_{t+1}) = 3.0$:

$$Q(S_t, A_t) \leftarrow 2.0 + 0.1\left[-1 + 0.99 \times 3.0 - 2.0\right] = 2.0 + 0.1 \times (-0.03) = 1.997$$

Note that $A_{t+1}$ might be a random exploratory action (with probability $\varepsilon$).
Because SARSA uses this **actual** next action, it learns values that **account for** the
occasional random exploration — learning a safer, more conservative policy.

### 8.2 Example: Off-Policy — Q-Learning

Same gridworld, same hyperparameters. The behavior policy $b$ is $\varepsilon$-greedy,
but the target policy $\pi$ is **greedy** (implicit via max).

**Step-by-step update:**

1. Agent is in state $S_t$, selects action $A_t$ using $b$ ($\varepsilon$-greedy)
2. Environment returns reward $R_{t+1}$ and next state $S_{t+1}$
3. Update uses **max** over all actions at $S_{t+1}$ (not the action $b$ would take):

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + 0.1 \left[R_{t+1} + 0.99 \cdot \max_a Q(S_{t+1}, a) - Q(S_t, A_t)\right]$$

**Numerical example:** Same values, but now $\max_a Q(S_{t+1}, a) = 5.0$ (the best action
at $S_{t+1}$ has value 5.0, even though the $\varepsilon$-greedy $b$ might not select it):

$$Q(S_t, A_t) \leftarrow 2.0 + 0.1\left[-1 + 0.99 \times 5.0 - 2.0\right] = 2.0 + 0.1 \times 1.95 = 2.195$$

Q-learning is more optimistic because it always assumes the best possible next action,
ignoring the fact that $b$ sometimes explores randomly.

### 8.3 The Cliff Walking Example

The classic **cliff walking** environment (Sutton & Barto, Example 6.6) beautifully
illustrates the on-policy vs off-policy difference:

```
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │     │     │     │     │     │     │     │     │     │     │
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │     │     │     │     │     │     │     │     │     │     │     │     │
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │     │     │     │     │     │     │     │     │     │     │     │     │
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  │  S  │XXXXX│XXXXX│XXXXX│XXXXX│XXXXX│XXXXX│XXXXX│XXXXX│XXXXX│XXXXX│  G  │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                          The Cliff (reward = -100)

  SARSA (on-policy):   Learns the SAFE path — stays away from cliff edge
                       because it accounts for its own ε-random actions
                       that might push it off the cliff.

  Q-learning (off-pol): Learns the OPTIMAL path — walks right along the
                        cliff edge, because the greedy target policy
                        would never step off. But during training, the
                        ε-greedy behavior policy DOES occasionally fall.
```

- **SARSA** learns higher online reward (safer path, fewer catastrophic falls)
- **Q-learning** learns the optimal policy but suffers more during training

---

## 9. Real-World Use Cases

### 9.1 On-Policy Use Cases

| Domain | Why On-Policy? |
|---|---|
| **Robotics** | Safety-critical — the robot must account for its own exploration noise |
| **Autonomous driving** | Cannot afford to learn a risky optimal policy; must learn a safe driving policy |
| **Game playing (training)** | PPO and A3C are on-policy and achieve state-of-the-art in many games |
| **Dialogue systems** | The conversational agent must learn from its own interactions |
| **Medical treatment** | Ethical constraints — must evaluate the policy actually being applied to patients |

### 9.2 Off-Policy Use Cases

| Domain | Why Off-Policy? |
|---|---|
| **Atari games (DQN)** | Experience replay massively improves data efficiency |
| **Recommendation systems** | Learn from logged user interaction data (historical behavior policy) |
| **Financial trading** | Learn optimal strategy from historical market data |
| **Learning from experts** | Imitate surgeon demonstrations, pilot recordings, etc. |
| **Multi-task learning** | Learn multiple policies from the same experience stream |
| **Sim-to-real transfer** | Learn in simulation (behavior), deploy in reality (target) |

---

## 10. Advanced Considerations

### 10.1 The Deadly Triad

When combining three elements, off-policy methods can become unstable or diverge:

1. **Off-policy learning** — learning about a policy different from the behavior policy
2. **Function approximation** — using neural networks or linear models instead of tables
3. **Bootstrapping** — using estimated values in the update target (as in TD methods)

Any two of these are generally safe; the combination of all three can cause divergence.
Modern deep RL algorithms (DQN, SAC, DDPG) use various stabilization techniques:
- **Target networks** (frozen copy of $Q$ for computing targets)
- **Experience replay** (breaking temporal correlations)
- **Gradient clipping** (bounding update magnitudes)
- **Double Q-learning** (reducing maximization bias)

### 10.2 Bridging On-Policy and Off-Policy

Some modern algorithms blur the boundary:

- **PPO (Proximal Policy Optimization):** Technically on-policy but reuses data for
  multiple gradient steps with a clipped surrogate objective to stay close to the
  behavior policy.

- **V-trace (IMPALA):** Uses importance sampling with truncated ratios — off-policy
  correction that degrades gracefully to on-policy behavior.

- **Retrace($\lambda$):** A safe off-policy return estimator that works with any
  ratio of behavior to target policy.

### 10.3 Importance Sampling Variance

The variance of importance sampling grows exponentially with trajectory length because
the ratio is a **product** of per-step ratios:

$$\text{Var}[\rho_{t:T-1}] \propto \prod_{k=t}^{T-1} \text{Var}\left[\frac{\pi(A_k \mid S_k)}{b(A_k \mid S_k)}\right]$$

This is why off-policy Monte Carlo methods with long episodes can be practically unusable.
Solutions include:
- **Per-decision importance sampling** — apply ratios only up to each reward
- **Weighted importance sampling** — use self-normalizing estimators
- **Truncated ratios** — clip $\rho$ to a maximum value (as in V-trace)
- **One-step methods** — avoid products entirely (Q-learning uses a single max)

---

## Summary

| Concept | Key Point |
|---|---|
| Target policy $\pi$ | The policy we want to evaluate or optimize |
| Behavior policy $b$ | The policy actually used to collect experience |
| On-policy ($b = \pi$) | Same policy generates data and is improved; simpler but sub-optimal |
| Off-policy ($b \neq \pi$) | Different policies; more complex but can converge to $\pi^*$ |
| Coverage assumption | $\pi(a \mid s) > 0 \Rightarrow b(a \mid s) > 0$ — $b$ must cover $\pi$ |
| Soft policies | Assign nonzero probability to all actions; resolve exploration dilemma |
| Importance sampling | Re-weights off-policy returns by $\prod \pi / b$ to correct distribution mismatch |
| $\varepsilon$-greedy | Most common soft policy; explores with probability $\varepsilon$ |
| SARSA (on-policy) | Uses $Q(S_{t+1}, A_{t+1})$ where $A_{t+1}$ comes from the same policy |
| Q-learning (off-policy) | Uses $\max_a Q(S_{t+1}, a)$; target is greedy regardless of behavior |
| Deadly triad | Off-policy + function approx + bootstrapping can cause divergence |
| Cliff walking | SARSA learns safe path; Q-learning learns optimal but risky path |

---

## Revision Questions

1. **Define the behavior policy and target policy.** Explain the difference between on-policy and off-policy methods in terms of these two policies. Why does the distinction matter for algorithm design?

2. **State the coverage assumption formally.** Give an example of a behavior-target policy pair that violates coverage, and explain what goes wrong when the assumption is violated.

3. **Compare the SARSA and Q-learning update rules.** Identify the exact term that differs and explain why this difference makes one on-policy and the other off-policy. What does each method converge to?

4. **In the cliff walking example, why does SARSA learn a safer path than Q-learning?** Explain in terms of how each algorithm treats the next action $A_{t+1}$ in its update.

5. **Why does the variance of importance sampling grow exponentially with episode length?** Write the importance sampling ratio for a trajectory of length $T$ and explain why this is problematic for off-policy Monte Carlo methods.

6. **An engineer proposes using Q-learning (off-policy) for a safety-critical robot controller.** Discuss the trade-offs. Would you recommend on-policy or off-policy learning in this scenario, and why?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [MC Control](./02-mc-control.md) | [Unit 4: Monte Carlo Methods](./README.md) | [Importance Sampling](./04-importance-sampling.md) |

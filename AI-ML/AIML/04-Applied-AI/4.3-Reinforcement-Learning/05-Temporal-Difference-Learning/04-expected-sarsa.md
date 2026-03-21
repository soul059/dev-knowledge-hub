# Chapter 4: Expected SARSA

> **Unit 5: Temporal Difference Learning** — *Reducing variance by taking expectations over next actions.*

---

## Overview

Expected SARSA is a refined TD control algorithm that modifies SARSA's update rule by replacing the **sampled** next-action value $Q(S_{t+1}, A_{t+1})$ with the **expected** value over all possible next actions, weighted by the policy $\pi$. This seemingly small change eliminates the variance introduced by the random selection of $A_{t+1}$, yielding more stable updates while preserving the bootstrapping efficiency of TD methods. Expected SARSA was introduced by van Seijen et al. (2009) and occupies a unique position in the TD landscape — it generalizes both SARSA and Q-Learning, can operate on-policy or off-policy depending on the policy used in the expectation, and consistently demonstrates improved performance over SARSA across a range of environments.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    EXPECTED SARSA AT A GLANCE                          ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║   Given: nothing — learns while interacting                            ║
║                                                                        ║
║   Goal:  learn Q(s, a) while reducing variance of TD updates           ║
║                                                                        ║
║   Each transition provides:                                            ║
║                                                                        ║
║          S_t ──(A_t)──► R_{t+1}, S_{t+1}                               ║
║                                                                        ║
║   Update:                                                              ║
║          Q(S_t, A_t) ← Q(S_t, A_t)                                    ║
║                        + α [ R_{t+1} + γ Σ_a π(a|S_{t+1}) Q(S_{t+1},a)║
║                              − Q(S_t, A_t) ]                          ║
║                                       ▲                                ║
║                              expectation over ALL actions              ║
║                              weighted by policy π                      ║
║                                                                        ║
║   Key:  eliminates variance from sampling A_{t+1}                      ║
║         generalizes SARSA (sample) and Q-Learning (max)                ║
║                                                                        ║
╚══════════════════════════════════════════════════════════════════════════╝
```

The insight behind Expected SARSA is elegant: instead of relying on a **single sample** of the next action (as SARSA does), take the **expected value** under the policy. This trades a small increase in computation — $O(|\mathcal{A}|)$ per update instead of $O(1)$ — for a significant reduction in variance. When the policy $\pi$ used in the expectation is the same $\varepsilon$-greedy policy used for action selection, Expected SARSA is on-policy; when $\pi$ is the greedy policy, Expected SARSA becomes exactly Q-Learning.

---

## 1. From SARSA to Expected SARSA — Removing Sampling Noise

Recall the SARSA update:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \, Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) \Big]$$

Here, $A_{t+1}$ is a **single action sampled** from the policy $\pi(\cdot | S_{t+1})$. The value $Q(S_{t+1}, A_{t+1})$ is a **stochastic estimate** of what we really want:

$$\mathbb{E}_\pi\Big[Q(S_{t+1}, A_{t+1}) \;\Big|\; S_{t+1}\Big] = \sum_{a} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a)$$

Expected SARSA replaces the sample with this expectation directly:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \sum_{a} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a) - Q(S_t, A_t) \Big]$$

> **Key Insight:** SARSA samples $A_{t+1}$ and uses $Q(S_{t+1}, A_{t+1})$; Expected SARSA computes $\sum_a \pi(a|S_{t+1}) Q(S_{t+1}, a)$ deterministically. The expectation is exact (given current Q-values), so the only remaining stochasticity comes from the environment's transition dynamics — not from action selection.

```
┌─────────────────────────────────────────────────────────────────────┐
│               THE CRITICAL DIFFERENCE                               │
│                                                                     │
│   SARSA:                                                            │
│     S_t ─(A_t)─► R_{t+1}, S_{t+1} ─(A_{t+1})─►                    │
│                                      ▲                              │
│                                      │                              │
│                              A_{t+1} ~ π(·|S_{t+1})               │
│                              ONE random sample                      │
│                              → HIGH variance                        │
│                                                                     │
│   Expected SARSA:                                                   │
│     S_t ─(A_t)─► R_{t+1}, S_{t+1}                                  │
│                               ▲                                     │
│                               │                                     │
│                       Σ_a π(a|S_{t+1}) Q(S_{t+1}, a)               │
│                       expectation over ALL actions                  │
│                       → LOW variance                                │
│                                                                     │
│   Q-Learning:                                                       │
│     S_t ─(A_t)─► R_{t+1}, S_{t+1}                                  │
│                               ▲                                     │
│                               │                                     │
│                       max_a Q(S_{t+1}, a)                           │
│                       only the BEST action's value                  │
│                       → ZERO variance (but off-policy)              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. The Expected SARSA Update Rule

### 2.1 The Full Update

After each transition $(S_t, A_t, R_{t+1}, S_{t+1})$, Expected SARSA applies:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \sum_{a \in \mathcal{A}} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a) - Q(S_t, A_t) \Big]$$

where:
- $Q(S_t, A_t)$ is the current estimate of the action-value for the pair $(S_t, A_t)$
- $\alpha \in (0, 1]$ is the **step size** (learning rate)
- $R_{t+1}$ is the **immediate reward** received
- $\gamma \in [0, 1]$ is the **discount factor**
- $\pi(a \mid S_{t+1})$ is the probability of action $a$ under the policy $\pi$ in the next state
- $\mathcal{A}$ is the set of all available actions

### 2.2 Decomposing the Update

**TD Target** — the Expected SARSA target:

$$\text{TD Target} = R_{t+1} + \gamma \sum_{a} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a)$$

**TD Error** — the temporal difference signal:

$$\delta_t = R_{t+1} + \gamma \sum_{a} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a) - Q(S_t, A_t)$$

**Compact form:**

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \, \delta_t$$

### 2.3 The Expectation Under ε-Greedy

For an $\varepsilon$-greedy policy with $|\mathcal{A}|$ actions, the policy assigns probabilities:

$$\pi(a \mid s) = \begin{cases} 1 - \varepsilon + \frac{\varepsilon}{|\mathcal{A}|} & \text{if } a = \arg\max_{a'} Q(s, a') \\[6pt] \frac{\varepsilon}{|\mathcal{A}|} & \text{otherwise} \end{cases}$$

The expected value under this policy becomes:

$$\sum_{a} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a) = \left(1 - \varepsilon + \frac{\varepsilon}{|\mathcal{A}|}\right) \max_a Q(S_{t+1}, a) \;+\; \frac{\varepsilon}{|\mathcal{A}|} \sum_{a \neq a^*} Q(S_{t+1}, a)$$

This can be rewritten more compactly as:

$$\sum_{a} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a) = (1 - \varepsilon) \max_a Q(S_{t+1}, a) \;+\; \frac{\varepsilon}{|\mathcal{A}|} \sum_{a} Q(S_{t+1}, a)$$

> **Note:** This second form is simpler to implement — separate the greedy and exploratory components.

---

## 3. Backup Diagram

The backup diagram for Expected SARSA shows all next actions being considered, weighted by the policy — unlike SARSA (one action) or Q-Learning (max action):

```
           SARSA                Expected SARSA              Q-Learning
                                                            
          (S_t, A_t)             (S_t, A_t)                (S_t, A_t)
              │                      │                          │
              ● A_t                  ● A_t                     ● A_t
              │                      │                          │
              ○ S_{t+1}              ○ S_{t+1}                 ○ S_{t+1}
              │                    / | \                       / | \
              ● A_{t+1}          ●  ●  ●  ●                 ●  ●  ●  ●
           (sampled)         π(a₁) π(a₂) π(a₃) π(a₄)         max
                                                            
          Uses Q(S',A')      Uses Σ π(a|S')Q(S',a)        Uses max_a Q(S',a)
          ONE sample         WEIGHTED average              MAXIMUM value
                                                            
                                                            
  ┌──────────────────────────────────────────────────────────────────┐
  │  Legend:  ● = action node (filled)                               │
  │           ○ = state node (open)                                  │
  │           Arrows show value flow upward into the update          │
  └──────────────────────────────────────────────────────────────────┘
```

The Expected SARSA backup diagram **fans out** to all actions in the next state, weighting each by $\pi(a|S_{t+1})$. This is wider than SARSA's single-branch backup but carries the same information as Q-Learning when $\pi$ is greedy (since the weight concentrates on the max action).

---

## 4. On-Policy vs Off-Policy — The Duality of Expected SARSA

Expected SARSA is uniquely flexible: it can be **on-policy** or **off-policy** depending on the relationship between the policy $\pi$ used in the expectation and the behavior policy $b$ used for action selection.

### 4.1 On-Policy Expected SARSA

When $\pi = b$ (both are the same $\varepsilon$-greedy policy):

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \sum_{a} b(a \mid S_{t+1}) \, Q(S_{t+1}, a) - Q(S_t, A_t) \Big]$$

This evaluates and improves the **current behavior policy**. Expected SARSA computes the exact expected value that SARSA only estimates via sampling. The result: same expected update as SARSA, but with **lower variance**.

### 4.2 Off-Policy Expected SARSA (= Q-Learning)

When $\pi$ is the **greedy** policy while $b$ is $\varepsilon$-greedy:

$$\pi(a \mid s) = \begin{cases} 1 & \text{if } a = \arg\max_{a'} Q(s, a') \\ 0 & \text{otherwise} \end{cases}$$

The expectation collapses:

$$\sum_{a} \pi(a \mid S_{t+1}) \, Q(S_{t+1}, a) = 1 \cdot Q\!\left(S_{t+1}, \arg\max_a Q(S_{t+1}, a)\right) = \max_a Q(S_{t+1}, a)$$

This is precisely the Q-Learning update!

> **Key result:** Q-Learning is a **special case** of Expected SARSA where the expectation is taken under the greedy policy.

### 4.3 The Spectrum

```
┌──────────────────────────────────────────────────────────────────┐
│                  THE EXPECTED SARSA SPECTRUM                     │
│                                                                  │
│   π = ε-greedy (same as b)    ◄────────────►    π = greedy       │
│   ──────────────────────                        ────────         │
│   On-policy                                     Off-policy       │
│   Expected SARSA                                Q-Learning       │
│   evaluates q_π                                 evaluates q_*    │
│                                                                  │
│            ε = 0.1        ε = 0.01       ε = 0                   │
│              │               │              │                    │
│              ▼               ▼              ▼                    │
│         Mostly on-       Mostly off-    Fully off-               │
│         policy           policy         policy                   │
│         (explores a lot) (explores less)(no exploration          │
│                                          in target)              │
│                                                                  │
│   In all cases: variance ≤ SARSA (same expected update,          │
│                  less noise from action sampling)                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 5. Variance Reduction — Why Expected SARSA Outperforms SARSA

### 5.1 Sources of Variance in TD Updates

Every TD update has the form $Q \leftarrow Q + \alpha \, \delta_t$. The variance of the TD error $\delta_t$ determines how noisy the learning signal is. In SARSA, two sources of randomness contribute:

1. **Environment stochasticity:** the transition $(S_{t+1}, R_{t+1})$ given $(S_t, A_t)$
2. **Action sampling:** $A_{t+1} \sim \pi(\cdot | S_{t+1})$

Expected SARSA eliminates source (2) entirely by computing the expectation analytically:

$$\text{Var}[\delta_t^{\text{ExpSARSA}}] \leq \text{Var}[\delta_t^{\text{SARSA}}]$$

### 5.2 Mathematical Justification

Let $V_\pi(S_{t+1}) = \sum_a \pi(a|S_{t+1}) Q(S_{t+1}, a)$ be the expected value used by Expected SARSA.

For SARSA, the TD target uses $Q(S_{t+1}, A_{t+1})$ where $A_{t+1} \sim \pi$. By the law of total variance:

$$\text{Var}[Q(S_{t+1}, A_{t+1})] = \underbrace{\mathbb{E}_{S_{t+1}}[\text{Var}_{A_{t+1}}[Q(S_{t+1}, A_{t+1}) \mid S_{t+1}]]}_{\text{variance from action sampling} \;\geq\; 0} + \text{Var}_{S_{t+1}}[\mathbb{E}_{A_{t+1}}[Q(S_{t+1}, A_{t+1}) \mid S_{t+1}]]$$

For Expected SARSA, since we compute $V_\pi(S_{t+1})$ exactly:

$$\text{Var}[V_\pi(S_{t+1})] = \text{Var}_{S_{t+1}}[V_\pi(S_{t+1})]$$

The first term (variance from action sampling) is **eliminated**, giving:

$$\text{Var}[\delta_t^{\text{ExpSARSA}}] = \text{Var}[\delta_t^{\text{SARSA}}] - \gamma^2 \, \mathbb{E}_{S_{t+1}}\Big[\text{Var}_{A_{t+1}}\big[Q(S_{t+1}, A_{t+1}) \mid S_{t+1}\big]\Big]$$

Since variances are non-negative, Expected SARSA's TD error always has **equal or lower** variance than SARSA's.

### 5.3 When Does It Matter Most?

The variance reduction is most significant when:
- **Large action spaces** — more actions means more variability in the sample $A_{t+1}$
- **High ε** — greater randomness in action selection amplifies sampling noise
- **Diverse Q-values** — if $Q(s, a)$ varies significantly across actions, sampling is noisier

---

## 6. Computational Cost

### 6.1 Per-Update Complexity

| Algorithm | Computation per Update | Complexity |
|---|---|---|
| **SARSA** | Sample $A_{t+1}$, look up $Q(S_{t+1}, A_{t+1})$ | $O(1)$ |
| **Q-Learning** | Find $\max_a Q(S_{t+1}, a)$ | $O(|\mathcal{A}|)$ |
| **Expected SARSA** | Compute $\sum_a \pi(a|S_{t+1}) Q(S_{t+1}, a)$ | $O(|\mathcal{A}|)$ |

Expected SARSA and Q-Learning have the same asymptotic cost — both require iterating over all actions in the next state. SARSA is cheaper per update but noisier.

### 6.2 When Is the Cost Acceptable?

- **Small action spaces** (e.g., 4 directions in grid world): negligible overhead
- **Moderate action spaces** (e.g., dozens of actions): acceptable; the variance reduction is worth the cost
- **Very large / continuous action spaces**: the sum becomes intractable; function approximation or sampling is needed

> **Practical note:** For most tabular problems studied in introductory RL (grid worlds, Cliff Walking, Windy Gridworld), the action space is small ($|\mathcal{A}| = 4$), making the $O(|\mathcal{A}|)$ cost trivially negligible.

---

## 7. Algorithm — Expected SARSA for Control

```
Algorithm: Tabular Expected SARSA (for estimating Q ≈ q_π or q_*)
────────────────────────────────────────────────────────────────────────
Input : step size α ∈ (0, 1]
        discount factor γ ∈ [0, 1]
        exploration rate ε ∈ [0, 1]
        policy π for the expectation (e.g., ε-greedy or greedy)
Output: action-value function Q ≈ q_π (or q_* if π is greedy)

1.  Initialize Q(s, a) arbitrarily for all s ∈ S, a ∈ A
    Initialize Q(terminal, ·) = 0

2.  REPEAT (for each episode):
3.      Initialize S (starting state of episode)
4.      REPEAT (for each step of episode):
5.          Choose A from S using behavior policy derived from Q
            (e.g., ε-greedy)
6.          Take action A, observe R, S'
7.          IF S' is terminal:
8.              Q(S, A) ← Q(S, A) + α [ R − Q(S, A) ]
9.          ELSE:
10.             Compute expected value:
                V̂ ← Σ_a π(a|S') · Q(S', a)
11.             Q(S, A) ← Q(S, A) + α [ R + γ · V̂ − Q(S, A) ]
12.         S ← S'
13.     UNTIL S is terminal
14. UNTIL convergence or maximum episodes reached
```

**Notes on the algorithm:**
- Line 5: the **behavior policy** $b$ selects actions (typically $\varepsilon$-greedy w.r.t. $Q$)
- Line 10: the **target policy** $\pi$ is used for the expectation (can be the same as $b$ or different)
- When $\pi = b = \varepsilon$-greedy: on-policy Expected SARSA
- When $\pi =$ greedy, $b = \varepsilon$-greedy: Expected SARSA becomes Q-Learning
- Unlike SARSA, we do **not** need to select $A_{t+1}$ before the update

---

## 8. Worked Example — Grid World Update

Consider a $3 \times 3$ grid world with 4 actions (Up, Down, Left, Right) and $\varepsilon = 0.2$.

**Current state:** $S_t = (1,1)$, action taken: $A_t = \text{Right}$  
**Transition:** Reward $R_{t+1} = -1$, next state $S_{t+1} = (1,2)$  
**Current Q-values at $S_{t+1} = (1,2)$:**

| Action | $Q(S_{t+1}, a)$ |
|---|---|
| Up | $-2.0$ |
| Down | $-3.5$ |
| Left | $-1.5$ |
| Right | $+1.0$ ← greedy action ($a^*$) |

**Step 1: Compute policy probabilities** ($\varepsilon = 0.2$, $|\mathcal{A}| = 4$):

$$\pi(\text{Right} \mid S_{t+1}) = 1 - \varepsilon + \frac{\varepsilon}{|\mathcal{A}|} = 1 - 0.2 + \frac{0.2}{4} = 0.85$$

$$\pi(\text{Up} \mid S_{t+1}) = \pi(\text{Down} \mid S_{t+1}) = \pi(\text{Left} \mid S_{t+1}) = \frac{\varepsilon}{|\mathcal{A}|} = \frac{0.2}{4} = 0.05$$

**Step 2: Compute the expected value:**

$$\hat{V} = 0.05 \cdot (-2.0) + 0.05 \cdot (-3.5) + 0.05 \cdot (-1.5) + 0.85 \cdot (1.0)$$

$$\hat{V} = -0.10 - 0.175 - 0.075 + 0.85 = 0.50$$

**Step 3: Compute TD error** (with $\alpha = 0.1$, $\gamma = 0.99$):

$$\delta_t = R_{t+1} + \gamma \, \hat{V} - Q(S_t, A_t) = -1 + 0.99 \times 0.50 - Q((1,1), \text{Right})$$

Suppose $Q((1,1), \text{Right}) = -0.5$:

$$\delta_t = -1 + 0.495 - (-0.5) = -0.005$$

**Step 4: Update:**

$$Q((1,1), \text{Right}) \leftarrow -0.5 + 0.1 \times (-0.005) = -0.5005$$

**Compare with SARSA:** If $A_{t+1}$ happened to be $\text{Down}$ (sampled with probability 0.05), SARSA would use $Q(S_{t+1}, \text{Down}) = -3.5$, giving a much larger TD error — a noisier update driven by an unlikely action. Expected SARSA avoids this by averaging over all possibilities.

---

## 9. Cliff Walking Comparison — SARSA vs Expected SARSA vs Q-Learning

### 9.1 The Environment

```
┌──────────────────────────────────────────────────────────────────┐
│                     CLIFF WALKING GRID                           │
│                                                                  │
│   ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐  │
│   │    │    │    │    │    │    │    │    │    │    │    │    │  │
│   ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤  │
│   │    │    │    │    │    │    │    │    │    │    │    │    │  │
│   ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤  │
│   │    │    │    │    │    │    │    │    │    │    │    │    │  │
│   ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤  │
│   │ S  │▓▓▓▓│▓▓▓▓│▓▓▓▓│▓▓▓▓│▓▓▓▓│▓▓▓▓│▓▓▓▓│▓▓▓▓│▓▓▓▓│▓▓▓▓│ G  │  │
│   └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘  │
│                                                                  │
│   S = Start    G = Goal    ▓▓ = Cliff (reward = −100)            │
│   All other steps: reward = −1                                   │
│                                                                  │
│   ─── SARSA path (safe, top row):              reward ≈ −17      │
│   ─── Q-Learning path (optimal, cliff edge):   reward = −13      │
│   ─── Expected SARSA path:                     reward ≈ −13      │
│       (optimal or near-optimal, with better online performance)  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 9.2 Behavior Summary

| Algorithm | Learned Path | Asymptotic Reward | Online Performance | Variance |
|---|---|---|---|---|
| **SARSA** | Safe (top row) | $\approx -17$ | Best (avoids cliff during training) | High |
| **Q-Learning** | Optimal (cliff edge) | $-13$ | Worst (cliff falls from exploration) | Moderate |
| **Expected SARSA** | Optimal (cliff edge) | $-13$ | Better than Q-Learning | Lowest |

### 9.3 Why Expected SARSA Performs Best

1. **Lower variance than SARSA:** Expected SARSA's updates are smoother, leading to faster and more stable convergence
2. **Better online performance than Q-Learning:** On-policy Expected SARSA accounts for the exploration risk, producing fewer catastrophic cliff falls during training
3. **Near-optimal policy:** Even on-policy Expected SARSA tends to learn a near-optimal or optimal path because the reduced variance allows it to more accurately evaluate cliff-edge states

> **Key result from van Seijen et al. (2009):** Expected SARSA achieves the best of both worlds — performance close to Q-Learning's optimal policy, with online training stability comparable to SARSA.

---

## 10. Convergence Properties

### 10.1 Convergence Guarantee

Expected SARSA converges to $q_\pi$ (or $q_*$ when $\pi$ is greedy) under the standard stochastic approximation conditions:

**Condition 1 (Robbins-Monro):** The step sizes satisfy:

$$\sum_{t=0}^{\infty} \alpha_t(s, a) = \infty \qquad \text{and} \qquad \sum_{t=0}^{\infty} \alpha_t^2(s, a) < \infty$$

**Condition 2 (Exploration):** All state-action pairs are visited infinitely often.

### 10.2 Contraction Property

The Expected SARSA operator $\mathcal{T}$ defined by:

$$(\mathcal{T}Q)(s, a) = \sum_{s'} p(s' \mid s, a) \left[ r(s, a, s') + \gamma \sum_{a'} \pi(a' \mid s') Q(s', a') \right]$$

is a **contraction mapping** in the $\ell_\infty$ norm with contraction factor $\gamma$:

$$\| \mathcal{T}Q_1 - \mathcal{T}Q_2 \|_\infty \leq \gamma \| Q_1 - Q_2 \|_\infty$$

By the Banach fixed-point theorem, this guarantees a unique fixed point $Q^*_\pi$ and convergence of iterative application.

### 10.3 Comparison of Convergence Properties

| Property | SARSA | Expected SARSA | Q-Learning |
|---|---|---|---|
| **Converges to** | $q_\pi$ ($\varepsilon$-greedy) | $q_\pi$ or $q_*$ (depends on $\pi$) | $q_*$ (optimal) |
| **Convergence rate** | Slower (higher variance) | Faster (lower variance) | Moderate |
| **Conditions** | Robbins-Monro + GLIE | Robbins-Monro + exploration | Robbins-Monro + exploration |
| **Constant α** | Converges to neighborhood | Converges to smaller neighborhood | Converges to neighborhood |
| **Function approx.** | Stable (on-policy) | Stable (on-policy mode) | Risk of divergence (off-policy) |

---

## 11. Python Implementation

### 11.1 Core Expected SARSA Agent

```python
import numpy as np
from collections import defaultdict


class ExpectedSARSAAgent:
    """Tabular Expected SARSA agent for discrete environments."""

    def __init__(self, n_actions, alpha=0.5, gamma=1.0, epsilon=0.1):
        self.n_actions = n_actions
        self.alpha = alpha
        self.gamma = gamma
        self.epsilon = epsilon
        self.Q = defaultdict(lambda: np.zeros(n_actions))

    def policy_probs(self, state):
        """Return ε-greedy action probabilities for a given state."""
        probs = np.ones(self.n_actions) * (self.epsilon / self.n_actions)
        best_action = np.argmax(self.Q[state])
        probs[best_action] += 1.0 - self.epsilon
        return probs

    def choose_action(self, state):
        """Select action using ε-greedy policy."""
        probs = self.policy_probs(state)
        return np.random.choice(self.n_actions, p=probs)

    def expected_value(self, state):
        """Compute Σ_a π(a|s) Q(s, a) — the expected Q-value under π."""
        probs = self.policy_probs(state)
        return np.dot(probs, self.Q[state])

    def update(self, state, action, reward, next_state, done):
        """Perform the Expected SARSA update."""
        if done:
            td_target = reward
        else:
            td_target = reward + self.gamma * self.expected_value(next_state)

        td_error = td_target - self.Q[state][action]
        self.Q[state][action] += self.alpha * td_error
        return td_error
```

### 11.2 Training Loop

```python
def train_expected_sarsa(env, agent, n_episodes=500):
    """Train Expected SARSA agent and return reward history."""
    rewards_per_episode = np.zeros(n_episodes)

    for episode in range(n_episodes):
        state, _ = env.reset()
        total_reward = 0
        done = False

        while not done:
            action = agent.choose_action(state)
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated

            agent.update(state, action, reward, next_state, done)

            state = next_state
            total_reward += reward

        rewards_per_episode[episode] = total_reward

    return rewards_per_episode
```

### 11.3 Full Comparison on Cliff Walking

```python
import gymnasium as gym
import matplotlib.pyplot as plt


class SARSAAgent:
    """Standard SARSA for comparison."""

    def __init__(self, n_actions, alpha=0.5, gamma=1.0, epsilon=0.1):
        self.n_actions = n_actions
        self.alpha = alpha
        self.gamma = gamma
        self.epsilon = epsilon
        self.Q = defaultdict(lambda: np.zeros(n_actions))

    def choose_action(self, state):
        if np.random.random() < self.epsilon:
            return np.random.randint(self.n_actions)
        return np.argmax(self.Q[state])

    def update(self, state, action, reward, next_state, next_action, done):
        if done:
            td_target = reward
        else:
            td_target = reward + self.gamma * self.Q[next_state][next_action]
        td_error = td_target - self.Q[state][action]
        self.Q[state][action] += self.alpha * td_error


def train_sarsa(env, agent, n_episodes=500):
    rewards_per_episode = np.zeros(n_episodes)
    for episode in range(n_episodes):
        state, _ = env.reset()
        action = agent.choose_action(state)
        total_reward = 0
        done = False

        while not done:
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            next_action = agent.choose_action(next_state) if not done else 0
            agent.update(state, action, reward, next_state, next_action, done)
            state = next_state
            action = next_action
            total_reward += reward

        rewards_per_episode[episode] = total_reward
    return rewards_per_episode


# --- Run comparison ---
n_runs = 50
n_episodes = 500
sarsa_all = np.zeros((n_runs, n_episodes))
exp_sarsa_all = np.zeros((n_runs, n_episodes))

for run in range(n_runs):
    env = gym.make("CliffWalking-v0")

    sarsa_agent = SARSAAgent(env.action_space.n, alpha=0.5, epsilon=0.1)
    sarsa_all[run] = train_sarsa(env, sarsa_agent, n_episodes)

    exp_agent = ExpectedSARSAAgent(env.action_space.n, alpha=0.5, epsilon=0.1)
    exp_sarsa_all[run] = train_expected_sarsa(env, exp_agent, n_episodes)

    env.close()

# --- Plot ---
window = 10
sarsa_mean = np.mean(sarsa_all, axis=0)
exp_sarsa_mean = np.mean(exp_sarsa_all, axis=0)
sarsa_smooth = np.convolve(sarsa_mean, np.ones(window) / window, mode='valid')
exp_smooth = np.convolve(exp_sarsa_mean, np.ones(window) / window, mode='valid')

plt.figure(figsize=(10, 6))
plt.plot(sarsa_smooth, label='SARSA', color='steelblue', linewidth=2)
plt.plot(exp_smooth, label='Expected SARSA', color='darkorange', linewidth=2)
plt.xlabel('Episode')
plt.ylabel(f'Sum of rewards\n(averaged over {n_runs} runs)')
plt.title('Expected SARSA vs SARSA on Cliff Walking')
plt.ylim(-100, 0)
plt.axhline(y=-13, color='green', linestyle='--', alpha=0.5, label='Optimal: −13')
plt.legend(loc='lower right')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('expected_sarsa_comparison.png', dpi=150)
plt.show()
```

---

## 12. SARSA vs Expected SARSA vs Q-Learning — Detailed Comparison

| Property | SARSA | Expected SARSA | Q-Learning |
|---|---|---|---|
| **Update target** | $R + \gamma Q(S', A')$ | $R + \gamma \sum_a \pi(a|S')Q(S',a)$ | $R + \gamma \max_a Q(S', a)$ |
| **Next action used** | $A' \sim \pi$ (sampled) | All actions (weighted by $\pi$) | $\arg\max_a Q(S', a)$ (greedy) |
| **Policy type** | On-policy only | On-policy or off-policy | Off-policy |
| **Behavior policy** | $\varepsilon$-greedy | $\varepsilon$-greedy (any exploring) | $\varepsilon$-greedy (any exploring) |
| **Target policy** | Same $\varepsilon$-greedy | Configurable ($\varepsilon$-greedy or greedy) | Greedy |
| **Converges to** | $q_\pi$ ($\varepsilon$-greedy values) | $q_\pi$ or $q_*$ (depends on $\pi$) | $q_*$ (optimal values) |
| **Variance** | Highest | Lowest | Moderate |
| **Computation / update** | $O(1)$ | $O(|\mathcal{A}|)$ | $O(|\mathcal{A}|)$ |
| **Data per update** | $(S, A, R, S', A')$ — quintuple | $(S, A, R, S')$ — quadruple | $(S, A, R, S')$ — quadruple |
| **Needs $A'$ before update?** | Yes | No | No |
| **Cliff Walking path** | Safe (top row) | Optimal (cliff edge) | Optimal (cliff edge) |
| **Online performance** | Best (safe) | Very good (stable) | Worst (cliff falls) |
| **Maximization bias** | No | No (unless $\pi =$ greedy) | Yes |
| **Historical note** | Rummery & Niranjan (1994) | van Seijen et al. (2009) | Watkins (1989) |

---

## 13. When to Use Expected SARSA

### 13.1 Prefer Expected SARSA When:

- The **action space is small to moderate** — the $O(|\mathcal{A}|)$ cost is acceptable
- You want **lower variance** than SARSA without going fully off-policy
- You need **on-policy safety** (accounting for exploration cost) but want faster convergence
- You want a **unified framework** that can interpolate between SARSA and Q-Learning

### 13.2 Prefer SARSA When:

- The **action space is very large** and computing the expectation is expensive
- You need **$O(1)$ updates** per step
- The environment is **safety-critical** and you want the most conservative on-policy behavior

### 13.3 Prefer Q-Learning When:

- You want to learn the **optimal policy directly** regardless of exploration
- You plan to use **experience replay** (requires off-policy)
- You are building toward **Deep RL** (DQN and its variants)

---

## 14. Connection to Other Methods

### 14.1 Expected SARSA as a Unifying Framework

Expected SARSA unifies the three one-step TD control methods through the choice of $\pi$:

$$Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha \Big[ R_{t+1} + \gamma \sum_a \pi(a|S_{t+1}) Q(S_{t+1}, a) - Q(S_t, A_t) \Big]$$

| Choice of $\pi$ | Result |
|---|---|
| $\pi = b$ ($\varepsilon$-greedy, sampled) | SARSA (when we sample instead of sum) |
| $\pi = b$ ($\varepsilon$-greedy, exact expectation) | On-policy Expected SARSA |
| $\pi =$ greedy (deterministic) | Q-Learning |
| $\pi =$ softmax or Boltzmann | Soft Expected SARSA |

### 14.2 Relationship to the Bellman Equation

Expected SARSA's target is a **sample** of the right-hand side of the Bellman equation for the policy $\pi$:

$$q_\pi(s, a) = \sum_{s', r} p(s', r \mid s, a) \Big[ r + \gamma \sum_{a'} \pi(a' \mid s') \, q_\pi(s', a') \Big]$$

The inner sum $\sum_{a'} \pi(a'|s') q_\pi(s', a')$ is computed exactly; the outer sum over $s'$ is estimated via sampling (the agent's actual experience). This is why Expected SARSA has lower variance — it computes half of the Bellman equation analytically.

---

## Summary

| Concept | Key Point |
|---|---|
| **Expected SARSA Update** | $Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha[R_{t+1} + \gamma \sum_a \pi(a \mid S_{t+1}) Q(S_{t+1}, a) - Q(S_t, A_t)]$ |
| **Core Idea** | Replace the sampled $Q(S', A')$ in SARSA with the expected value $\sum_a \pi(a \mid S') Q(S', a)$ |
| **Variance** | Lower than SARSA — eliminates randomness from action sampling |
| **On-Policy Mode** | When $\pi = $ behavior policy ($\varepsilon$-greedy): evaluates current policy with lower noise |
| **Off-Policy Mode** | When $\pi = $ greedy: Expected SARSA reduces to Q-Learning |
| **Computation** | $O(\lvert\mathcal{A}\rvert)$ per update vs $O(1)$ for SARSA |
| **Cliff Walking** | Learns optimal/near-optimal path with better online stability than Q-Learning |
| **Convergence** | Converges under Robbins-Monro conditions; contraction mapping in $\ell_\infty$ |
| **Data Per Update** | Quadruple $(S, A, R, S')$ — does not need $A'$ before update |
| **Unifying Property** | Generalizes both SARSA (sample) and Q-Learning (max) through choice of $\pi$ |
| **Best Used When** | Action space is small-to-moderate and variance reduction outweighs extra computation |
| **Historical Origin** | van Seijen, van Hasselt, Whiteson, Wiering (2009) |

---

## Revision Questions

1. **Write out the Expected SARSA update rule and compare it to SARSA's.** Identify the exact term that differs and explain why replacing a sample with an expectation reduces variance while preserving the expected update direction.

2. **Show that Q-Learning is a special case of Expected SARSA.** What policy $\pi$ must be used in the expectation, and what happens to the summation $\sum_a \pi(a|S') Q(S', a)$ under that policy?

3. **Compute one Expected SARSA update step** for a state with three actions, Q-values $[2.0, -1.0, 3.5]$, $\varepsilon = 0.3$, $\alpha = 0.1$, $\gamma = 0.95$, and reward $R = +1$. Show all intermediate calculations.

4. **Explain the on-policy vs off-policy duality of Expected SARSA.** Under what conditions is it on-policy? When does it become off-policy? Why is this flexibility useful?

5. **Compare the Cliff Walking performance of SARSA, Expected SARSA, and Q-Learning.** Which algorithm achieves the best online performance during training? Which learns the optimal path? How does Expected SARSA achieve a balance?

6. **Analyze the computational trade-off of Expected SARSA.** When is the $O(|\mathcal{A}|)$ cost per update justified over SARSA's $O(1)$? Give an example where Expected SARSA would be impractical due to action space size.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Q-Learning](./03-q-learning.md) | [Unit 5: TD Learning](./README.md) | [TD vs MC vs DP](./05-td-vs-mc-vs-dp.md) |

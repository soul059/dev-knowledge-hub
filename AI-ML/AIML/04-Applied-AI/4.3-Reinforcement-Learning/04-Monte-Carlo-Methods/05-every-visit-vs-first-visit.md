# Chapter 5: Every-Visit vs First-Visit MC

> **Unit 4: Monte Carlo Methods** — *Two approaches to counting state visits for return averaging.*

---

## Overview

When a Monte Carlo agent collects an episode and computes returns, it must decide
**how to handle states that appear more than once** in the same trajectory. A state
$s$ might be visited at time steps $t_1, t_2, \ldots$ within a single episode — this
is common in environments with loops, stochastic transitions, or large cyclic state
spaces. The two classical strategies are **first-visit MC**, which records only the
return from the *earliest* visit to $s$ in each episode, and **every-visit MC**, which
records the return from *every* visit to $s$. Both estimators converge to $V^\pi(s)$
as the number of episodes grows, but they differ in bias, variance, data efficiency,
and ease of implementation. This chapter provides a rigorous comparison of the two
methods, complete with mathematical analysis, worked examples, and code.

```
 ╔════════════════════════════════════════════════════════════════════════╗
 ║           EVERY-VISIT vs FIRST-VISIT MC AT A GLANCE                  ║
 ╠════════════════════════════════════════════════════════════════════════╣
 ║                                                                      ║
 ║  First-Visit MC                 Every-Visit MC                       ║
 ║  ──────────────                 ──────────────                       ║
 ║  • Count only the FIRST         • Count EVERY occurrence of s       ║
 ║    occurrence of s in episode     in the episode                     ║
 ║  • Unbiased estimator           • Biased (but consistent)           ║
 ║  • Higher variance per episode  • Lower variance per episode        ║
 ║  • Matches classical sampling   • Uses more data per episode        ║
 ║  • Simpler theoretical proofs   • Simpler to implement              ║
 ║  • Both converge to V^π(s)     • Both converge to V^π(s)           ║
 ║                                                                      ║
 ╚════════════════════════════════════════════════════════════════════════╝
```

---

## 1. The Visit-Counting Problem

Consider an episode generated under policy $\pi$:

$$S_0, A_0, R_1, S_1, A_1, R_2, \ldots, S_{T-1}, A_{T-1}, R_T$$

A state $s$ may appear at multiple time steps within this trajectory. Let $\mathcal{T}(s)$
denote the set of all time steps at which state $s$ is visited:

$$\mathcal{T}(s) = \{ t : S_t = s, \; 0 \le t \le T-1 \}$$

For example, in the episode $A \to B \to A \to C \to \text{Terminal}$, we have
$\mathcal{T}(A) = \{0, 2\}$, $\mathcal{T}(B) = \{1\}$, and $\mathcal{T}(C) = \{3\}$.

The question is: when estimating $V^\pi(A)$, should we use the return from **time 0
only** (first visit), or the returns from **both time 0 and time 2** (every visit)?

---

## 2. First-Visit MC Prediction

### 2.1 Definition

First-visit MC uses only the **first** time step at which $s$ appears in each episode.
Across $N$ episodes, let the first-visit time in episode $k$ be:

$$t_k^{\text{first}}(s) = \min \mathcal{T}_k(s)$$

The first-visit MC estimator after $N$ episodes is:

$$V_N^{\text{FV}}(s) = \frac{1}{N(s)} \sum_{k=1}^{N} G_{t_k^{\text{first}}(s)}^{(k)}$$

where $N(s)$ is the number of episodes in which $s$ was visited at least once, and
$G_t^{(k)}$ is the return from time $t$ in episode $k$.

### 2.2 Algorithm

```
 FIRST-VISIT MC PREDICTION
 ─────────────────────────
 Input : policy π, number of episodes N
 Output: V(s) for all s

 Initialize:
     V(s) ← 0 for all s
     Returns(s) ← empty list for all s

 Repeat for each episode:
     Generate episode: S₀,A₀,R₁,S₁,...,S_{T-1},A_{T-1},R_T
     G ← 0
     For t = T-1, T-2, ..., 0:          ← backward pass
         G ← γ·G + R_{t+1}
         If S_t NOT in {S₀, S₁, ..., S_{t-1}}:    ← first-visit check
             Append G to Returns(S_t)
             V(S_t) ← average(Returns(S_t))

 Return V
```

### 2.3 Key Property: Unbiasedness

Each first-visit return $G_{t_k^{\text{first}}}$ is an **independent, identically
distributed (i.i.d.)** sample of the true expected return from $s$ under $\pi$. By
the definition of value:

$$V^\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]$$

Since the first visit in each episode provides one i.i.d. draw from this distribution:

$$\mathbb{E}[V_N^{\text{FV}}(s)] = V^\pi(s) \quad \forall N \ge 1$$

First-visit MC is an **unbiased** estimator of $V^\pi(s)$.

---

## 3. Every-Visit MC Prediction

### 3.1 Definition

Every-visit MC uses **all** time steps at which $s$ appears in each episode. Across $N$
episodes, the total number of visits to $s$ is:

$$M(s) = \sum_{k=1}^{N} |\mathcal{T}_k(s)|$$

The every-visit MC estimator is:

$$V_N^{\text{EV}}(s) = \frac{1}{M(s)} \sum_{k=1}^{N} \sum_{t \in \mathcal{T}_k(s)} G_t^{(k)}$$

### 3.2 Algorithm

```
 EVERY-VISIT MC PREDICTION
 ─────────────────────────
 Input : policy π, number of episodes N
 Output: V(s) for all s

 Initialize:
     V(s) ← 0 for all s
     Returns(s) ← empty list for all s

 Repeat for each episode:
     Generate episode: S₀,A₀,R₁,S₁,...,S_{T-1},A_{T-1},R_T
     G ← 0
     For t = T-1, T-2, ..., 0:          ← backward pass
         G ← γ·G + R_{t+1}
         Append G to Returns(S_t)         ← no check needed!
         V(S_t) ← average(Returns(S_t))

 Return V
```

> **Implementation Note:** Every-visit MC is **simpler to implement** because it does
> not require the conditional check "has $S_t$ been seen earlier in this episode?" The
> inner loop body is shorter by one line.

### 3.3 Bias of Every-Visit MC

Within a single episode, the multiple returns from different visits to the same state
are **not independent**. They share later portions of the trajectory. This dependence
introduces a **bias** into the every-visit estimator:

$$\mathbb{E}[V_N^{\text{EV}}(s)] \neq V^\pi(s) \quad \text{for finite } N$$

However, every-visit MC is **consistent**: as $N \to \infty$, the bias vanishes and the
estimator converges to the true value:

$$V_N^{\text{EV}}(s) \xrightarrow{a.s.} V^\pi(s) \quad \text{as } N \to \infty$$

The bias arises because when a state is revisited in the same episode, the later
returns are computed from a suffix of the trajectory that is shared with (and thus
correlated with) earlier returns. The average of correlated samples is generally
biased relative to the average of i.i.d. samples.

---

## 4. Detailed Example Walkthrough

Consider a single episode in a grid-world with $\gamma = 1$ (undiscounted):

```
 Episode Trajectory
 ════════════════════════════════════════════════════════════════

 Time:    t=0    t=1    t=2    t=3    t=4    t=5
 State:    A  ──→  B  ──→  C  ──→  A  ──→  D  ──→  Terminal
 Reward:      +1       +2       -1       +3       +4
           R₁=+1   R₂=+2   R₃=-1   R₄=+3   R₅=+4

 State A appears at t=0 and t=3  (visited twice)
 State B appears at t=1           (visited once)
 State C appears at t=2           (visited once)
 State D appears at t=4           (visited once)
```

### Step 1: Compute Returns (backward pass)

Working backwards from $t = T-1 = 4$:

| Time $t$ | State $S_t$ | $R_{t+1}$ | Return $G_t = R_{t+1} + \gamma G_{t+1}$ |
|---|---|---|---|
| $t=4$ | D | $+4$ | $G_4 = 4$ |
| $t=3$ | A | $+3$ | $G_3 = 3 + 1 \cdot 4 = 7$ |
| $t=2$ | C | $-1$ | $G_2 = -1 + 1 \cdot 7 = 6$ |
| $t=1$ | B | $+2$ | $G_1 = 2 + 1 \cdot 6 = 8$ |
| $t=0$ | A | $+1$ | $G_0 = 1 + 1 \cdot 8 = 9$ |

### Step 2: First-Visit MC

```
 ┌─────────────────────────────────────────────────────────────┐
 │              FIRST-VISIT MC PROCESSING                      │
 │                                                             │
 │  t=0: S₀ = A → First visit to A?  YES (not seen before)    │
 │                 Record G₀ = 9  ✓                            │
 │                                                             │
 │  t=1: S₁ = B → First visit to B?  YES                      │
 │                 Record G₁ = 8  ✓                            │
 │                                                             │
 │  t=2: S₂ = C → First visit to C?  YES                      │
 │                 Record G₂ = 6  ✓                            │
 │                                                             │
 │  t=3: S₃ = A → First visit to A?  NO (already seen at t=0) │
 │                 SKIP G₃ = 7  ✗                              │
 │                                                             │
 │  t=4: S₄ = D → First visit to D?  YES                      │
 │                 Record G₄ = 4  ✓                            │
 │                                                             │
 │  Result:  V(A) = 9,  V(B) = 8,  V(C) = 6,  V(D) = 4       │
 └─────────────────────────────────────────────────────────────┘
```

### Step 3: Every-Visit MC

```
 ┌─────────────────────────────────────────────────────────────┐
 │              EVERY-VISIT MC PROCESSING                      │
 │                                                             │
 │  t=0: S₀ = A → Record G₀ = 9  ✓                            │
 │  t=1: S₁ = B → Record G₁ = 8  ✓                            │
 │  t=2: S₂ = C → Record G₂ = 6  ✓                            │
 │  t=3: S₃ = A → Record G₃ = 7  ✓   (counted again!)        │
 │  t=4: S₄ = D → Record G₄ = 4  ✓                            │
 │                                                             │
 │  Result:  V(A) = (9 + 7) / 2 = 8                           │
 │           V(B) = 8,  V(C) = 6,  V(D) = 4                   │
 └─────────────────────────────────────────────────────────────┘
```

### Step 4: Comparison

| State | First-Visit $V(s)$ | Every-Visit $V(s)$ | Difference |
|---|---|---|---|
| A | 9.0 | 8.0 | First-visit uses only $G_0$; every-visit averages $G_0$ and $G_3$ |
| B | 8.0 | 8.0 | Same (visited once) |
| C | 6.0 | 6.0 | Same (visited once) |
| D | 4.0 | 4.0 | Same (visited once) |

> **Observation:** The methods differ only for state A, which was visited twice.
> With $\gamma = 1$, the true value $V^\pi(A)$ depends on the full transition dynamics
> — neither single-episode estimate is guaranteed to be closer. Over many episodes,
> both converge to the same true value.

---

## 5. Mathematical Analysis

### 5.1 Bias

**First-Visit MC** produces i.i.d. samples of $G_t \mid S_t = s$, one per episode.
The sample mean of i.i.d. draws is unbiased:

$$\text{Bias}(V_N^{\text{FV}}) = \mathbb{E}[V_N^{\text{FV}}(s)] - V^\pi(s) = 0$$

**Every-Visit MC** averages correlated returns. Let episode $k$ have $n_k$ visits to
state $s$. The within-episode returns $G_{t_1}^{(k)}, G_{t_2}^{(k)}, \ldots$ are
**positively correlated** because later returns are subsequences of earlier ones. The
bias for finite $N$ is:

$$\text{Bias}(V_N^{\text{EV}}) = \mathbb{E}[V_N^{\text{EV}}(s)] - V^\pi(s) \neq 0$$

However, the bias is $O(1/N)$ and vanishes asymptotically:

$$\lim_{N \to \infty} \text{Bias}(V_N^{\text{EV}}) = 0$$

### 5.2 Variance

For a single episode, let $\sigma^2 = \text{Var}(G_t \mid S_t = s)$.

**First-Visit MC** (after $N$ episodes, each contributing one return):

$$\text{Var}(V_N^{\text{FV}}) = \frac{\sigma^2}{N}$$

**Every-Visit MC** (after $N$ episodes, contributing $M$ total returns with
within-episode correlation $\rho > 0$):

$$\text{Var}(V_N^{\text{EV}}) < \frac{\sigma^2}{N}$$

Every-visit MC achieves lower variance because it uses more data points ($M \ge N$).
Even though the returns within an episode are correlated, the additional samples
still reduce total variance. The effective sample size lies between $N$ (fully
correlated) and $M$ (fully independent).

### 5.3 Mean Squared Error (MSE)

The MSE decomposes as:

$$\text{MSE} = \text{Bias}^2 + \text{Variance}$$

| Estimator | Bias² | Variance | MSE |
|---|---|---|---|
| First-Visit | $0$ | $\sigma^2 / N$ | $\sigma^2 / N$ |
| Every-Visit | $O(1/N^2)$ | $< \sigma^2 / N$ | Often $\le \sigma^2 / N$ |

> **Surprising Result:** Despite being biased, every-visit MC often has **lower MSE**
> than first-visit MC, especially when states are revisited frequently. The variance
> reduction more than compensates for the small bias (Singh & Sutton, 1996).

### 5.4 Convergence

Both estimators converge to the true value by the **law of large numbers**:

**First-Visit MC (Strong Law of Large Numbers):**

$$V_N^{\text{FV}}(s) = \frac{1}{N(s)} \sum_{k=1}^{N(s)} G_k \xrightarrow{a.s.} \mathbb{E}[G \mid S = s] = V^\pi(s)$$

This follows directly because $G_1, G_2, \ldots$ are i.i.d. with finite mean.

**Every-Visit MC (Convergence of Correlated Averages):**

$$V_N^{\text{EV}}(s) \xrightarrow{a.s.} V^\pi(s)$$

The proof is more involved. Although the samples are not i.i.d., the inter-episode
independence and the bounded correlation structure are sufficient to guarantee
convergence (see Singh & Sutton, 1996, for the formal argument).

Both estimators also satisfy the **central limit theorem**, converging at rate
$O(1/\sqrt{N})$ to a normal distribution centered at $V^\pi(s)$.

---

## 6. Visual Comparison of Visit Handling

```
 Episode: S₀=A → S₁=B → S₂=A → S₃=C → S₄=A → S₅=D → Terminal
 Rewards:    +1      +2      -3      +1      +2      +5

 ════════════════════════════════════════════════════════════════

 FIRST-VISIT MC view of state A:

   t=0       t=1       t=2       t=3       t=4       t=5
    A ─────── B ─────── A ─────── C ─────── A ─────── D ──→ End
    ↑                   ×                   ×
    │                   │                   │
  RECORD              SKIP                SKIP
  G₀ = 8             (seen)              (seen)

  Returns(A) ← [8]
  V(A) = 8 / 1 = 8.0

 ────────────────────────────────────────────────────────────────

 EVERY-VISIT MC view of state A:

   t=0       t=1       t=2       t=3       t=4       t=5
    A ─────── B ─────── A ─────── C ─────── A ─────── D ──→ End
    ↑                   ↑                   ↑
    │                   │                   │
  RECORD              RECORD              RECORD
  G₀ = 8             G₂ = 5             G₄ = 7

  Returns(A) ← [8, 5, 7]
  V(A) = (8 + 5 + 7) / 3 = 6.67

 ════════════════════════════════════════════════════════════════

 Note: G₀ = 1+2-3+1+2+5 = 8
       G₂ = -3+1+2+5    = 5
       G₄ = 2+5         = 7      (all with γ=1)
```

---

## 7. Python Implementation

The following code implements both methods and compares them on a simple random-walk
environment where states can be revisited.

```python
import numpy as np
from collections import defaultdict

# ── Simple Looping Environment ──────────────────────────────────
# States: 0, 1, 2, 3, 4 (Terminal)
# From states 1-3, agent moves right with p=0.6, loops back to
# state 0 with p=0.4. State 0 always moves to state 1.
# Rewards: +1 on each transition, +10 on reaching terminal state 4.

def generate_episode():
    """Generate one episode in the looping environment."""
    states, rewards = [0], []
    s = 0
    while s != 4:
        if s == 0:
            s_next = 1
            r = 1.0
        elif np.random.random() < 0.6:
            s_next = s + 1      # move right
            r = 1.0 if s + 1 != 4 else 10.0
        else:
            s_next = 0          # loop back to start
            r = 1.0
        states.append(s_next)
        rewards.append(r)
        s = s_next
    return states[:-1], rewards  # exclude terminal from states


def first_visit_mc(num_episodes, gamma=0.9):
    """First-visit MC prediction."""
    V = defaultdict(float)
    returns = defaultdict(list)

    for _ in range(num_episodes):
        states, rewards = generate_episode()
        G = 0.0
        visited = set()

        # Backward pass
        for t in range(len(states) - 1, -1, -1):
            G = gamma * G + rewards[t]
            s = states[t]
            if s not in visited:
                visited.add(s)
                returns[s].append(G)
                V[s] = np.mean(returns[s])

    return dict(V)


def every_visit_mc(num_episodes, gamma=0.9):
    """Every-visit MC prediction."""
    V = defaultdict(float)
    returns = defaultdict(list)

    for _ in range(num_episodes):
        states, rewards = generate_episode()
        G = 0.0

        # Backward pass
        for t in range(len(states) - 1, -1, -1):
            G = gamma * G + rewards[t]
            s = states[t]
            returns[s].append(G)
            V[s] = np.mean(returns[s])

    return dict(V)


# ── Run comparison ──────────────────────────────────────────────
if __name__ == "__main__":
    np.random.seed(42)
    N = 10000

    V_fv = first_visit_mc(N)
    V_ev = every_visit_mc(N)

    print("State | First-Visit V(s) | Every-Visit V(s)")
    print("──────┼──────────────────┼─────────────────")
    for s in sorted(set(V_fv.keys()) | set(V_ev.keys())):
        fv = V_fv.get(s, 0.0)
        ev = V_ev.get(s, 0.0)
        print(f"  {s}   │     {fv:8.4f}      │     {ev:8.4f}")
```

### 7.1 Convergence Comparison Code

```python
import numpy as np
from collections import defaultdict

def mc_convergence_comparison(num_episodes=5000, gamma=0.9, target_state=0):
    """Track how V(target_state) evolves for both methods."""
    fv_returns = []
    ev_returns = []
    fv_estimates = []
    ev_estimates = []

    for ep in range(1, num_episodes + 1):
        states, rewards = generate_episode()
        G = 0.0
        first_visit_found = False
        episode_returns = []

        # Backward pass to compute returns
        for t in range(len(states) - 1, -1, -1):
            G = gamma * G + rewards[t]
            if states[t] == target_state:
                episode_returns.append(G)

        # First-visit: only use the return from earliest visit
        if episode_returns:
            fv_returns.append(episode_returns[-1])  # earliest = last in reversed list

        # Every-visit: use all returns
        ev_returns.extend(episode_returns)

        # Record running estimates
        if fv_returns:
            fv_estimates.append(np.mean(fv_returns))
        if ev_returns:
            ev_estimates.append(np.mean(ev_returns))

    return fv_estimates, ev_estimates


# Example usage:
# fv_curve, ev_curve = mc_convergence_comparison(num_episodes=5000)
# Both curves converge to the same value; every-visit typically
# converges faster initially due to more data per episode.
```

---

## 8. When to Use Each Method

### 8.1 Prefer First-Visit MC When

- **Theoretical guarantees matter:** First-visit MC has cleaner statistical properties
  (unbiased, i.i.d. samples) that simplify confidence intervals and hypothesis tests.
- **States are rarely revisited:** If the environment is acyclic or episodes are short,
  the two methods produce identical results and first-visit is standard.
- **You need provably unbiased estimates:** In off-policy settings with importance
  sampling, first-visit avoids compounding bias issues.
- **Formal analysis is required:** Most textbook proofs and convergence rate bounds
  are stated for the first-visit variant.

### 8.2 Prefer Every-Visit MC When

- **Data efficiency is critical:** In environments with frequent state revisits,
  every-visit MC extracts more information from each episode.
- **Simplicity of implementation matters:** No need to track which states have been
  seen earlier in the episode.
- **Function approximation is used:** With neural networks or linear function
  approximation, the distinction between first-visit and every-visit becomes less
  sharp; every-visit naturally fits the stochastic gradient descent framework.
- **Lower MSE is preferred over unbiasedness:** When the practical goal is accurate
  estimates rather than theoretical purity, every-visit often wins.

### 8.3 Decision Guide

```
              Need unbiased estimator?
             /                        \
           YES                        NO
            │                          │
     Use First-Visit MC         States revisited often?
                               /                      \
                             YES                       NO
                              │                         │
                       Use Every-Visit MC       Either works
                       (better MSE)             (identical results)
```

---

## 9. Practical Recommendations

1. **Default to first-visit** for tabular settings when following textbook algorithms
   (e.g., Sutton & Barto). This keeps your implementation aligned with standard proofs.

2. **Switch to every-visit** when working with function approximation. Most deep RL
   implementations implicitly use every-visit updates since each $(s, G_t)$ pair is
   treated as a training sample.

3. **Use incremental updates** in either case to avoid storing all returns:

$$V(s) \leftarrow V(s) + \frac{1}{N(s)}\big(G_t - V(s)\big)$$

4. **Monitor convergence** by tracking the running estimate of $V(s)$ across episodes.
   If every-visit MC converges noticeably faster, the environment likely has frequent
   revisits and every-visit is the better choice.

5. **In off-policy learning** with importance sampling, prefer first-visit MC. The
   importance sampling ratio for every-visit MC is harder to compute correctly and
   can introduce additional variance.

6. **For Monte Carlo control** (policy improvement), the choice matters less because
   the policy changes between episodes, invalidating the i.i.d. assumption of
   first-visit MC anyway.

---

## 10. Summary Table

| Concept | Key Point |
|---|---|
| First-visit definition | Use only the return from the earliest visit to $s$ in each episode |
| Every-visit definition | Use the return from every visit to $s$ in each episode |
| Bias (first-visit) | Unbiased — each return is an i.i.d. sample of $V^\pi(s)$ |
| Bias (every-visit) | Biased for finite $N$ due to within-episode correlation; consistent |
| Variance (first-visit) | $\sigma^2 / N$ — standard i.i.d. sample mean variance |
| Variance (every-visit) | Less than $\sigma^2 / N$ — more data points reduce variance |
| MSE comparison | Every-visit often has lower MSE despite bias |
| Convergence | Both converge to $V^\pi(s)$ by the law of large numbers |
| Convergence rate | Both $O(1/\sqrt{N})$; every-visit may converge faster in practice |
| Implementation | Every-visit is simpler (no visit-tracking logic needed) |
| Off-policy use | First-visit preferred for cleaner importance sampling |
| Function approximation | Every-visit is natural fit for SGD-based updates |
| Acyclic environments | Both methods are identical (no repeated visits) |
| Data efficiency | Every-visit uses more data per episode |

---

## Revision Questions

1. **Define first-visit and every-visit MC prediction.** Given the episode
   $A \to B \to A \to C \to A \to \text{Terminal}$ with rewards $[+2, +1, -1, +3, +5]$
   and $\gamma = 1$, compute $V(A)$ under each method for this single episode.

2. **Prove that first-visit MC is an unbiased estimator of $V^\pi(s)$.** What
   property of the first-visit returns makes the standard proof of unbiasedness
   apply directly? Why does this property fail for every-visit returns?

3. **Explain why every-visit MC is biased but consistent.** What is the source of
   correlation among within-episode returns, and why does this correlation vanish
   in its effect as $N \to \infty$?

4. **Compare the MSE of both methods.** Under what conditions can a biased estimator
   (every-visit) achieve lower MSE than an unbiased one (first-visit)? Relate your
   answer to the bias-variance decomposition $\text{MSE} = \text{Bias}^2 + \text{Var}$.

5. **When would you prefer every-visit MC over first-visit MC in practice?** Give two
   concrete scenarios and justify your choice using the properties discussed.

6. **Modify the first-visit MC algorithm to use incremental updates** instead of
   storing all returns. Write the update rule and explain why it produces the same
   result as the batch-average version.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Importance Sampling](./04-importance-sampling.md) | [Unit 4: Monte Carlo Methods](./README.md) | [TD(0) Prediction](../05-Temporal-Difference-Learning/01-td0-prediction.md) |

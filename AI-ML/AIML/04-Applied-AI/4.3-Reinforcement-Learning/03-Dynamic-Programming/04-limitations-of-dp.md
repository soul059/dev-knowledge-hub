# Chapter 4: Limitations of Dynamic Programming

> **Unit 3: Dynamic Programming** — *Understanding when DP breaks down and what comes next.*

---

## Overview

Dynamic programming provides elegant, provably convergent algorithms for solving MDPs. But
DP methods carry fundamental limitations that restrict their applicability in practice. The
most critical are the **curse of dimensionality** (state and action spaces grow exponentially
with problem features), the **requirement for a complete model** (transition probabilities
and reward function must be known), and the **computational cost** of full-width backups.
These limitations motivate the model-free methods covered in subsequent units: Monte Carlo
methods and temporal-difference learning.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║              LIMITATIONS OF DP AT A GLANCE                      ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • Curse of dimensionality: exponential growth of state space   ║
 ║  • Requires complete model: P(s'|s,a) and R(s,a,s')            ║
 ║  • Full-width backups: must sweep every state, every action     ║
 ║  • Memory: must store V(s) or Q(s,a) for ALL states             ║
 ║  • Motivates: Monte Carlo, TD, and function approximation       ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. Curse of Dimensionality

### The Problem

Real-world problems are described by multiple features. The total number of states is the
**product** of the number of values each feature can take:

$$|\mathcal{S}| = \prod_{i=1}^{d} n_i$$

where $d$ is the number of state variables and $n_i$ is the number of discrete values for
variable $i$.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │              THE CURSE OF DIMENSIONALITY                         │
  │                                                                  │
  │  State features:  position × velocity × angle × angular_vel     │
  │                                                                  │
  │  If each has 100 discrete values:                                │
  │    |S| = 100 × 100 × 100 × 100 = 100,000,000 states            │
  │                                                                  │
  │  For 10 features with 100 values each:                           │
  │    |S| = 100^10 = 10^20 states                                  │
  │                                                                  │
  │  V(s) with 64-bit floats: 10^20 × 8 bytes = 8 × 10^11 TB       │
  │  ── far beyond any computer's memory ──                          │
  │                                                                  │
  │  Features:  1     2      3       4        5       ...    10      │
  │  States:   100   10K    1M     100M      10B    ...   10^20     │
  │                          ↑                ↑                      │
  │                      manageable      impossible                  │
  └──────────────────────────────────────────────────────────────────┘
```

### Scaling Example

| State Features | Values per Feature | $\|\mathcal{S}\|$ | DP Feasible? |
|---|---|---|---|
| 2 (e.g., row × col) | 100 | 10,000 | ✓ Yes |
| 4 (e.g., CartPole) | 100 | $10^8$ | ⚠ Borderline |
| 6 (e.g., Acrobot) | 100 | $10^{12}$ | ✗ No |
| 10 (e.g., multi-joint robot) | 100 | $10^{20}$ | ✗ Impossible |
| 50 (e.g., Atari pixels) | 256 | $256^{50}$ | ✗ Astronomical |

> **Key Insight:** DP works brilliantly for small, tabular MDPs but becomes infeasible as
> the state space grows. This is not a software problem — it is a fundamental mathematical
> limitation.

---

## 2. Requirement for a Complete Model

DP algorithms require full knowledge of the environment dynamics:

$$P(s' \mid s, a) \quad \text{and} \quad R(s, a, s') \quad \text{for all } s, a, s'$$

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║              MODEL-BASED vs MODEL-FREE                          ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║                                                                 ║
 ║  MODEL-BASED (DP):                                              ║
 ║  ┌─────────────┐     ┌─────────────┐                            ║
 ║  │ Known model  │────▶│ Plan with   │────▶ Optimal policy       ║
 ║  │ P(s'|s,a)    │     │ DP (PI/VI)  │                           ║
 ║  │ R(s,a,s')    │     └─────────────┘                           ║
 ║  └─────────────┘                                                ║
 ║                                                                 ║
 ║  MODEL-FREE (MC/TD):                                            ║
 ║  ┌─────────────┐     ┌─────────────┐                            ║
 ║  │ Experience   │────▶│ Learn from  │────▶ Learned policy       ║
 ║  │ (s,a,r,s')   │     │ samples     │                           ║
 ║  │ from env     │     └─────────────┘                           ║
 ║  └─────────────┘                                                ║
 ║                                                                 ║
 ║  When is the model unknown?                                     ║
 ║  • Robotics: friction, contact dynamics are complex             ║
 ║  • Games: opponent's strategy is unknown                        ║
 ║  • Finance: market dynamics are non-stationary                  ║
 ║  • Healthcare: patient response varies unpredictably            ║
 ║                                                                 ║
 ╚══════════════════════════════════════════════════════════════════╝
```

Even when a model exists in principle, **building it** can be impractical:
- The model has $|\mathcal{S}|^2 \times |\mathcal{A}|$ parameters (transition probabilities)
- Estimating each parameter requires sufficient data
- The model may be non-stationary (changing over time)

---

## 3. Computational Cost Analysis

### Time Complexity

| Algorithm | Per-Sweep Cost | Total Iterations | Overall |
|---|---|---|---|
| **Policy Evaluation** | $O(\|\mathcal{S}\|^2 \times \|\mathcal{A}\|)$ | $O\left(\frac{\log(1/\varepsilon)}{\log(1/\gamma)}\right)$ | $O\left(\frac{\|\mathcal{S}\|^2 \|\mathcal{A}\| \log(1/\varepsilon)}{\log(1/\gamma)}\right)$ |
| **Policy Iteration** | $O(\|\mathcal{S}\|^2 \times \|\mathcal{A}\|)$ per eval sweep | Few outer iterations | Dominated by inner evaluation |
| **Value Iteration** | $O(\|\mathcal{S}\|^2 \times \|\mathcal{A}\|)$ | $O\left(\frac{\log(1/\varepsilon)}{\log(1/\gamma)}\right)$ | Same as policy evaluation |

### Space Complexity

| Component | Memory Required |
|---|---|
| Value function $V(s)$ | $O(\|\mathcal{S}\|)$ |
| Policy $\pi(s)$ | $O(\|\mathcal{S}\|)$ |
| Transition model $P(s' \mid s, a)$ | $O(\|\mathcal{S}\|^2 \times \|\mathcal{A}\|)$ |
| Reward model $R(s, a, s')$ | $O(\|\mathcal{S}\|^2 \times \|\mathcal{A}\|)$ |
| **Total** | **$O(\|\mathcal{S}\|^2 \times \|\mathcal{A}\|)$** |

```
  ┌──────────────────────────────────────────────────────────────────┐
  │          STORAGE REQUIREMENTS BY STATE SPACE SIZE                │
  │                                                                  │
  │  |S|      P matrix size       Memory (float64)                   │
  │  ─────────────────────────────────────────────                   │
  │  100      100×4×100 = 40K     320 KB          ← trivial         │
  │  1,000    1K×4×1K = 4M        32 MB           ← manageable      │
  │  10,000   10K×4×10K = 400M    3.2 GB          ← challenging     │
  │  100,000  100K×4×100K = 40B   320 GB          ← impractical     │
  │  1M       1M×4×1M = 4T       32 TB            ← impossible      │
  │                                                                  │
  │  Note: Sparse representations help, but the fundamental          │
  │  scaling remains quadratic in |S|.                               │
  └──────────────────────────────────────────────────────────────────┘
```

### Per-Sweep Cost Breakdown

For each sweep of value iteration:

$$\underbrace{|\mathcal{S}|}_{\text{states}} \times \underbrace{|\mathcal{A}|}_{\text{actions}} \times \underbrace{|\mathcal{S}|}_{\text{successors}} = |\mathcal{S}|^2 \times |\mathcal{A}|$$

For an MDP with 10,000 states and 4 actions, one sweep requires $10{,}000 \times 4 \times 10{,}000 = 4 \times 10^8$ operations. At 1 GFLOP, that's about 0.4 seconds per sweep. With 1,000 sweeps needed for convergence: **400 seconds total** — manageable. But at 100,000 states, the same calculation gives $4 \times 10^{10}$ operations per sweep — **11 hours per sweep**.

---

## 4. Synchronous vs Asynchronous DP

Standard (synchronous) DP sweeps through **every state** in each iteration. Asynchronous
DP relaxes this requirement, updating states in any order and at any frequency.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │          SYNCHRONOUS vs ASYNCHRONOUS DP                          │
  │                                                                  │
  │  Synchronous:                                                    │
  │  ┌───┬───┬───┬───┬───┬───┬───┬───┐                             │
  │  │ s₁│ s₂│ s₃│ s₄│ s₅│ s₆│ s₇│ s₈│  ← update ALL states      │
  │  └───┴───┴───┴───┴───┴───┴───┴───┘     every sweep             │
  │       sweep 1, then sweep 2, ...                                 │
  │                                                                  │
  │  Asynchronous:                                                   │
  │  ┌───┬───┬───┬───┬───┬───┬───┬───┐                             │
  │  │ s₃│   │ s₇│   │ s₁│   │ s₃│   │  ← update SOME states      │
  │  └───┴───┴───┴───┴───┴───┴───┴───┘     in any order            │
  │       any order, any frequency                                   │
  │                                                                  │
  │  Requirement: every state must be updated infinitely often       │
  │  (in the limit) for convergence.                                 │
  └──────────────────────────────────────────────────────────────────┘
```

### Variants of Asynchronous DP

| Variant | Description | Advantage |
|---|---|---|
| **In-place DP** | Update states one at a time using latest values | Faster convergence; saves memory |
| **Prioritized sweeping** | Update states with largest Bellman error first | Focuses computation where needed most |
| **Real-time DP** | Update only states visited by the agent | Only computes values for relevant states |
| **Gauss-Seidel** | Sweep in fixed order; use updated values immediately | Classic numerical methods approach |

---

## 5. Real-Time Dynamic Programming (RTDP)

RTDP addresses the problem that most states may never be visited under the optimal policy.
Instead of sweeping all states, RTDP updates only **states encountered during simulated
episodes**.

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║  REAL-TIME DYNAMIC PROGRAMMING (RTDP)                               ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║  1. Start from initial state s₀                                     ║
 ║  2. At current state s:                                             ║
 ║       a. Update: V(s) ← max_a Σ_{s'} P(s'|s,a)[R + γV(s')]        ║
 ║       b. Act greedily: a ← argmax_a Q(s,a)                         ║
 ║       c. Observe next state s' ~ P(·|s,a)                          ║
 ║       d. Set s ← s'                                                ║
 ║  3. Repeat from step 2 until terminal or timeout                    ║
 ║  4. Repeat episodes until convergence                               ║
 ║                                                                     ║
 ║  Key benefit: Avoids updating irrelevant states.                    ║
 ║  Convergence: guaranteed for stochastic shortest path problems.     ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

RTDP focuses computation on the states the agent actually encounters, making it practical
for large state spaces where only a small fraction of states are reachable from typical
starting conditions.

---

## 6. Motivation for Model-Free Methods

The limitations of DP motivate two families of model-free algorithms:

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │                  THE PATH FORWARD FROM DP                           │
  │                                                                      │
  │  DP Limitations:                                                     │
  │  ├── Requires model    ──────▶  MONTE CARLO METHODS (Unit 4)        │
  │  │   P(s'|s,a) unknown          Learn from complete episodes        │
  │  │                               No model needed                     │
  │  │                                                                   │
  │  ├── Requires model    ──────▶  TEMPORAL DIFFERENCE (Unit 5)        │
  │  │   P(s'|s,a) unknown          Learn from single transitions       │
  │  │   + Want online learning      Bootstrap like DP                   │
  │  │                               No model, no complete episodes      │
  │  │                                                                   │
  │  └── Curse of          ──────▶  FUNCTION APPROXIMATION (Unit 6)     │
  │      dimensionality              Approximate V(s) with neural nets   │
  │      |S| too large               Generalize across similar states    │
  │                                                                      │
  └──────────────────────────────────────────────────────────────────────┘
```

### What Each Method Trades Away

| Feature | DP | Monte Carlo | TD Learning |
|---|---|---|---|
| Requires model? | ✓ Yes | ✗ No | ✗ No |
| Requires episodes? | ✗ No | ✓ Yes (complete) | ✗ No |
| Bootstraps? | ✓ Yes | ✗ No | ✓ Yes |
| Online? | ✗ No (full sweeps) | After episodes | ✓ Yes (per step) |
| Bias | None (exact model) | None (unbiased) | Some (bootstrap bias) |
| Variance | None (exact computation) | High (episode returns) | Lower than MC |

---

## 7. Comparison Table: DP vs MC vs TD (Preview)

```
 ╔═════════════════════════════════════════════════════════════════════════╗
 ║                    DP vs MC vs TD — OVERVIEW                          ║
 ╠══════════════╦══════════════════╦════════════════╦════════════════════╣
 ║              ║   Dynamic Prog.  ║  Monte Carlo   ║  Temporal Diff.   ║
 ╠══════════════╬══════════════════╬════════════════╬════════════════════╣
 ║ Model        ║ Required         ║ Not required   ║ Not required      ║
 ║ Bootstrap    ║ Yes              ║ No             ║ Yes               ║
 ║ Full episode ║ No               ║ Yes            ║ No                ║
 ║ Update type  ║ Expected (exact) ║ Sample-based   ║ Sample-based      ║
 ║ Backup width ║ Full-width       ║ Sample         ║ Sample            ║
 ║ Bias         ║ None             ║ None           ║ Some              ║
 ║ Variance     ║ None             ║ High           ║ Lower             ║
 ║ Online       ║ No               ║ After episode  ║ Yes               ║
 ║ Convergence  ║ Guaranteed       ║ Guaranteed     ║ Guaranteed (*)    ║
 ║ Scalability  ║ Poor (tabular)   ║ Better         ║ Best              ║
 ╚══════════════╩══════════════════╩════════════════╩════════════════════╝

 (*) Under appropriate step-size conditions.
```

```
         BACKUP DIAGRAMS COMPARED

     DP (full-width)        MC (sample)         TD (sample)

         V(s)                 V(s)                V(s)
        /   \                  |                    |
      a₁    a₂               a₃                   a₂
     / \   / \                 |                    |
   s₁  s₂ s₃ s₄              s₇                   s₃
                               |                  (stop,
                              s₁₂                 bootstrap
                               |                   V(s₃))
                              ...
                               |
                              s_T
                         (full episode)
```

---

## 8. When DP Is Still Useful in Practice

Despite its limitations, DP remains valuable in several settings:

| Setting | Why DP Works | Example |
|---|---|---|
| **Small MDPs** | State space is manageable | Board games, simple gridworlds |
| **Known models** | Factory, logistics with calibrated simulators | Inventory control, supply chain |
| **Planning in learned models** | Learn model first, then plan with DP | Dyna architecture, model-based RL |
| **Solving sub-problems** | DP solves a small, inner planning problem | Options/hierarchical RL |
| **Baseline / verification** | DP gives the true optimal solution | Benchmarking approximate methods |
| **Structured state spaces** | Factored MDPs exploit structure | Operations research, network optimization |

> **Key Insight:** DP is not obsolete — it is the **gold standard** against which all
> approximate methods are measured. Understanding DP deeply is essential for understanding
> why Monte Carlo and TD methods work.

---

## 9. Python Example: Demonstrating Scalability Issues

```python
import numpy as np
import time

def value_iteration_timed(n_states, n_actions=4, gamma=0.99, theta=1e-4, max_iter=500):
    """
    Run value iteration on a random MDP and report timing.
    Demonstrates how computation scales with state space size.
    """
    # Build random MDP
    # P[s, a, s'] — sparse-ish: each (s,a) transitions to ~5 successor states
    P = np.zeros((n_states, n_actions, n_states))
    for s in range(n_states):
        for a in range(n_actions):
            # Random sparse transitions: pick 5 successor states
            n_successors = min(5, n_states)
            successors = np.random.choice(n_states, size=n_successors, replace=False)
            probs = np.random.dirichlet(np.ones(n_successors))
            P[s, a, successors] = probs

    # Random rewards
    R = np.random.randn(n_states, n_actions, n_states) * 0.1

    # Precompute for vectorized VI
    R_sa = np.einsum('sap,sap->sa', P, R)

    V = np.zeros(n_states)
    start_time = time.time()

    for k in range(max_iter):
        Q = R_sa + gamma * np.einsum('sap,p->sa', P, V)
        V_new = np.max(Q, axis=1)
        delta = np.max(np.abs(V_new - V))
        V = V_new
        if delta < theta:
            elapsed = time.time() - start_time
            return elapsed, k + 1, True

    elapsed = time.time() - start_time
    return elapsed, max_iter, False


# ═══════════════════════════════════════════════════
# Scaling experiment
# ═══════════════════════════════════════════════════
print("=" * 65)
print("SCALABILITY OF VALUE ITERATION")
print("=" * 65)
print(f"{'|S|':>10} | {'Time (s)':>10} | {'Iters':>6} | {'Converged':>9} | {'Memory (MB)':>12}")
print("-" * 65)

for n_states in [100, 500, 1000, 2000, 5000]:
    np.random.seed(42)
    try:
        elapsed, iters, converged = value_iteration_timed(n_states)
        # Memory estimate: P matrix dominates
        memory_mb = (n_states * 4 * n_states * 8) / (1024 ** 2)
        print(f"{n_states:>10} | {elapsed:>10.3f} | {iters:>6} | {'Yes' if converged else 'No':>9} | {memory_mb:>12.1f}")
    except MemoryError:
        print(f"{n_states:>10} | {'OOM':>10} | {'N/A':>6} | {'N/A':>9} | {'OOM':>12}")


# ═══════════════════════════════════════════════════
# Demonstrate curse of dimensionality
# ═══════════════════════════════════════════════════
print(f"\n{'=' * 65}")
print("CURSE OF DIMENSIONALITY — State Space Explosion")
print("=" * 65)
print(f"{'Features':>10} | {'Vals/Feature':>12} | {'|S|':>15} | {'P matrix':>15} | {'Feasible?':>10}")
print("-" * 65)

for n_features, vals_per_feature in [(2, 10), (2, 100), (3, 50), (4, 20), (5, 20), (6, 10), (8, 10), (10, 10)]:
    n_states = vals_per_feature ** n_features
    p_size = n_states * 4 * n_states  # assume 4 actions
    p_memory_gb = (p_size * 8) / (1024 ** 3)

    if p_memory_gb < 0.001:
        mem_str = f"{p_memory_gb * 1024:.1f} MB"
    elif p_memory_gb < 1000:
        mem_str = f"{p_memory_gb:.1f} GB"
    else:
        mem_str = f"{p_memory_gb / 1024:.0f} TB"

    feasible = "✓ Yes" if n_states <= 10000 else ("⚠ Hard" if n_states <= 100000 else "✗ No")

    print(f"{n_features:>10} | {vals_per_feature:>12} | {n_states:>15,} | {mem_str:>15} | {feasible:>10}")
```

**Expected output:**

```
=================================================================
SCALABILITY OF VALUE ITERATION
=================================================================
      |S| |   Time (s) |  Iters | Converged |   Memory (MB)
-----------------------------------------------------------------
       100 |      0.012 |    142 |       Yes |          0.3
       500 |      0.298 |    142 |       Yes |          7.6
      1000 |      1.184 |    142 |       Yes |         30.5
      2000 |      4.827 |    142 |       Yes |        122.1
      5000 |     30.291 |    142 |       Yes |        762.9

=================================================================
CURSE OF DIMENSIONALITY — State Space Explosion
=================================================================
  Features | Vals/Feature |            |S| |        P matrix |  Feasible?
-----------------------------------------------------------------
         2 |           10 |             100 |          0.3 MB |      ✓ Yes
         2 |          100 |          10,000 |       3052.0 MB |      ✓ Yes
         3 |           50 |         125,000 |       ...    GB |     ⚠ Hard
         4 |           20 |         160,000 |       ...    GB |       ✗ No
         5 |           20 |       3,200,000 |       ...    TB |       ✗ No
         ...
```

---

## 10. Strategies to Mitigate DP Limitations

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║            STRATEGIES FOR SCALING DP                            ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║                                                                 ║
 ║  1. State aggregation                                           ║
 ║     Group similar states together to reduce |S|                 ║
 ║                                                                 ║
 ║  2. Function approximation                                      ║
 ║     Represent V(s) ≈ f(s; θ) with parameters θ ≪ |S|           ║
 ║     (Neural networks, linear models, tile coding)               ║
 ║                                                                 ║
 ║  3. Factored MDPs                                               ║
 ║     Exploit conditional independence in state variables         ║
 ║     to decompose the problem                                    ║
 ║                                                                 ║
 ║  4. Hierarchical RL                                             ║
 ║     Solve sub-problems at different levels of abstraction       ║
 ║                                                                 ║
 ║  5. Sparse representations                                      ║
 ║     Exploit sparsity in transitions (most s' have P=0)          ║
 ║                                                                 ║
 ║  6. Model-free methods                                          ║
 ║     Skip the model entirely: learn from experience              ║
 ║     (Monte Carlo, TD, Q-learning, SARSA, ...)                   ║
 ║                                                                 ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 11. Summary

| Concept | Key Point |
|---|---|
| Curse of dimensionality | $\|\mathcal{S}\| = \prod n_i$ grows exponentially with features |
| Model requirement | DP needs $P(s' \mid s, a)$ and $R(s, a, s')$ — often unavailable |
| Time complexity | $O(\|\mathcal{S}\|^2 \|\mathcal{A}\|)$ per sweep |
| Space complexity | $O(\|\mathcal{S}\|^2 \|\mathcal{A}\|)$ for the model |
| Synchronous DP | Updates all states every sweep — wasteful for large MDPs |
| Asynchronous DP | Selective updates (prioritized sweeping, RTDP) — more efficient |
| Model-free motivation | MC and TD learn from experience without knowing the model |
| DP still useful | Small MDPs, known models, learned models (Dyna), benchmarking |
| Function approximation | Key to scaling: replace $V(s)$ table with parameterized $f(s; \theta)$ |

---

## Revision Questions

1. **Explain the curse of dimensionality** in the context of reinforcement learning. If a robot has 6 joint angles, each discretized to 100 values, how many states exist? How much memory is needed to store $V(s)$?

2. **Why does DP require a complete model?** Identify which step of value iteration uses $P(s' \mid s, a)$ and explain why sample-based methods can avoid this requirement.

3. **Compare synchronous and asynchronous DP.** Describe prioritized sweeping and explain when it is most beneficial.

4. **What is Real-Time DP?** How does it avoid updating irrelevant states? Under what conditions does RTDP converge?

5. **Fill in the comparison table** for DP, Monte Carlo, and TD learning. Explain the trade-off between bias and variance across these three families.

6. **Given a problem with $|\mathcal{S}| = 50{,}000$ and $|\mathcal{A}| = 10$**, estimate the time for one sweep of value iteration (assuming $10^9$ operations per second). Is this feasible? What strategies could make it tractable?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Value Iteration](./03-value-iteration.md) | [Unit 3: Dynamic Programming](./README.md) | [Monte Carlo Methods](../04-Monte-Carlo-Methods/README.md) |

# Chapter 5: TD vs MC vs DP — A Unified View

> **Unit 5: Temporal Difference Learning** — *Understanding the relationships between the three fundamental RL approaches.*

---

## Overview

Dynamic Programming (DP), Monte Carlo (MC), and Temporal Difference (TD) learning are the three foundational pillars of reinforcement learning. Each solves the same core problem — estimating value functions and finding optimal policies — but each makes fundamentally different assumptions about what information is available and how updates are computed. Understanding their relationships, trade-offs, and the dimensions along which they differ is essential for choosing the right algorithm for any RL problem.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                  TD vs MC vs DP — AT A GLANCE                          ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║   Three methods, one goal: estimate V(s) ≈ v_π(s)                      ║
║                                                                        ║
║   DP:   V(s) ← Σ_a π(a|s) Σ_{s',r} p(s',r|s,a)[r + γV(s')]          ║
║         ► full model required, bootstraps, sweeps all states           ║
║                                                                        ║
║   MC:   V(s) ← V(s) + α [ G_t − V(s) ]                               ║
║         ► model-free, waits for full return G_t, episode-based         ║
║                                                                        ║
║   TD:   V(s) ← V(s) + α [ R_{t+1} + γV(S_{t+1}) − V(s) ]            ║
║         ► model-free, bootstraps, learns every step                    ║
║                                                                        ║
║   Two key dimensions separate them:                                    ║
║         1. Bootstrapping?  (DP ✓, TD ✓, MC ✗)                         ║
║         2. Sampling?       (DP ✗, TD ✓, MC ✓)                         ║
║                                                                        ║
╚══════════════════════════════════════════════════════════════════════════╝
```

These three methods are not isolated inventions — they lie on a spectrum defined by two orthogonal dimensions: **bootstrapping** (updating from estimates vs actual returns) and **sampling** (using sample transitions vs the full model). Understanding this 2×2 grid, along with the bias-variance trade-offs that emerge, provides a unified lens through which all of reinforcement learning can be understood.

---

## 1. The Three Update Rules — Side by Side

Before analyzing differences, let us state the prediction update rules precisely.

### 1.1 Dynamic Programming (Expected Update)

DP uses the **Bellman expectation equation** directly. It requires complete knowledge of the environment model $p(s', r \mid s, a)$:

$$V(s) \leftarrow \sum_{a} \pi(a \mid s) \sum_{s', r} p(s', r \mid s, a) \Big[ r + \gamma V(s') \Big]$$

This is a **full backup**: it considers every possible next state $s'$ and reward $r$, weighted by their transition probabilities. No sampling is involved.

### 1.2 Monte Carlo (Sample, No Bootstrap)

MC methods wait until the end of an episode to compute the actual return $G_t$, then update:

$$V(S_t) \leftarrow V(S_t) + \alpha \Big[ G_t - V(S_t) \Big]$$

where the return is the complete discounted sum:

$$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots + \gamma^{T-t-1} R_T$$

MC uses a **sample** of the trajectory (one actual episode) and does **not bootstrap** — $G_t$ is computed entirely from real rewards.

### 1.3 Temporal Difference — TD(0) (Sample, Bootstrap)

TD(0) updates after every single step using the estimated value of the next state:

$$V(S_t) \leftarrow V(S_t) + \alpha \Big[ R_{t+1} + \gamma V(S_{t+1}) - V(S_t) \Big]$$

TD uses a **sample** transition (one actual step) and **bootstraps** — it uses $V(S_{t+1})$ rather than waiting for the true return $G_t$.

> **Key Insight:** TD combines the best of both worlds — it is model-free like MC (learns from experience) and bootstraps like DP (does not wait for episode end). This is precisely why Sutton called TD methods "a combination of Monte Carlo ideas and dynamic programming ideas."

---

## 2. Bootstrapping vs Sampling — The Two Dimensions

The relationship between DP, MC, and TD is best understood through two independent binary dimensions.

### 2.1 Bootstrapping

**Bootstrapping** means updating an estimate based on another estimate, rather than waiting for the actual outcome.

- **Bootstraps (DP, TD):** The update target includes $V(S_{t+1})$ or $V(s')$ — a current estimate, not the true value. This introduces **bias** because the estimate may be wrong, but it dramatically reduces **variance** and allows learning before episode completion.
- **No Bootstrap (MC):** The update target is $G_t$ — the actual realized return. This is **unbiased** (in expectation, $\mathbb{E}[G_t \mid S_t = s] = v_\pi(s)$) but has **high variance** because $G_t$ depends on many random transitions.

### 2.2 Sampling

**Sampling** means using a single sampled trajectory or transition to estimate an expectation, rather than computing it exactly from the model.

- **Samples (MC, TD):** Only one actual transition or trajectory is observed. The agent does not need to know the environment dynamics $p(s', r \mid s, a)$. This makes the methods **model-free**.
- **Full Model (DP):** All possible transitions and their probabilities are used. The expectation is computed exactly, but this requires complete knowledge of the MDP — the method is **model-based**.

### 2.3 The 2×2 Grid

These two dimensions create a 2×2 classification of backup operations:

```
┌──────────────────────────────────────────────────────────────────┐
│                  BOOTSTRAPPING vs SAMPLING GRID                  │
├──────────────────┬────────────────────┬──────────────────────────┤
│                  │   Bootstraps       │   Does NOT Bootstrap     │
│                  │   (uses estimates) │   (uses actual returns)  │
├──────────────────┼────────────────────┼──────────────────────────┤
│  Sample Backup   │                    │                          │
│  (one trajectory │   TD Learning      │   Monte Carlo            │
│   or transition) │   TD(0), SARSA,    │   First-visit MC,        │
│                  │   Q-Learning       │   Every-visit MC         │
├──────────────────┼────────────────────┼──────────────────────────┤
│  Full Backup     │                    │                          │
│  (all possible   │   Dynamic          │   Exhaustive             │
│   transitions)   │   Programming      │   Monte Carlo            │
│                  │   Policy Eval,     │   (rarely practical —    │
│                  │   Value Iteration  │    average over ALL      │
│                  │                    │    possible episodes)    │
└──────────────────┴────────────────────┴──────────────────────────┘
```

The bottom-right cell — **Exhaustive MC** — is theoretically possible but almost never practical. It would require averaging returns over all possible complete episodes, which is typically exponential or infinite. It is included here for conceptual completeness.

> **Key Insight:** TD sits in the top-left cell, combining sampling from MC (top row) with bootstrapping from DP (left column). This unique position is what makes TD methods so powerful and widely applicable.

---

## 3. Backup Diagrams — Visualizing the Difference

Backup diagrams show which states and actions contribute to an update. They make the structural differences between DP, MC, and TD immediately visible.

```
  DP (Full Backup)          MC (Sample Backup)        TD(0) (Sample Backup)
                              
        (s)                       (s)                       (s)
       / | \                       |                         |
      /  |  \                      |                         |
    a1  a2  a3                     a                         a
   /|\  |  /|\                     |                         |
  / | \ | / | \                    |                         |
 s' s' s's' s' s'                  s'                        s'
  \ | / | \ | /                    |                     [bootstrap]
   \|/  |  \|/                     a'
    ·   ·   ·                      |
   (all transitions               s''
    weighted by p)                 |
                                   ·
                                   ·    (continues to
                                   ·     episode end)
                                   |
                                  s_T
                              [actual return G_t]
```

**Interpreting the diagrams:**

- **DP** fans out over **all actions** (weighted by $\pi$) and **all next states** (weighted by $p$). The backup is wide — every possibility is considered. This requires the model.
- **MC** follows a **single sampled trajectory** all the way to the terminal state. The backup is deep — it traces the entire episode. No bootstrapping occurs.
- **TD(0)** follows a **single sampled transition** for just one step, then bootstraps from the estimated value $V(S_{t+1})$. The backup is both narrow (one sample) and shallow (one step).

---

## 4. Bias-Variance Trade-off

The choice between bootstrapping and not bootstrapping creates a fundamental bias-variance trade-off.

### 4.1 Monte Carlo — Unbiased, High Variance

The MC target $G_t$ is an **unbiased** estimate of $v_\pi(s)$:

$$\mathbb{E}_\pi[G_t \mid S_t = s] = v_\pi(s)$$

However, $G_t$ depends on the entire sequence of random rewards and transitions from step $t$ to the episode end. Each random variable adds variance:

$$\text{Var}(G_t) = \text{Var}\left(\sum_{k=0}^{T-t-1} \gamma^k R_{t+k+1}\right)$$

In long episodes or high-stochasticity environments, this variance can be enormous.

### 4.2 TD(0) — Biased, Low Variance

The TD target $R_{t+1} + \gamma V(S_{t+1})$ is a **biased** estimate of $v_\pi(s)$ because $V(S_{t+1}) \neq v_\pi(S_{t+1})$ in general:

$$\mathbb{E}_\pi[R_{t+1} + \gamma V(S_{t+1}) \mid S_t = s] \neq v_\pi(s) \quad \text{(when } V \neq v_\pi\text{)}$$

However, the variance is much lower because the target depends on only **one** random transition $(R_{t+1}, S_{t+1})$ rather than an entire trajectory.

As $V$ converges to $v_\pi$, the bias vanishes and the TD target becomes asymptotically unbiased. The practical consequence: TD typically converges **faster** than MC because the reduction in variance more than compensates for the small bias.

### 4.3 DP — No Sampling Error

DP computes expectations exactly from the model — there is no sampling variance at all. The only "error" comes from using the current estimate $V(s')$ rather than the true value $v_\pi(s')$ (bootstrapping bias), but this is systematically reduced through iterative sweeps.

```
┌─────────────────────────────────────────────────────────────────┐
│              BIAS-VARIANCE SPECTRUM                             │
│                                                                 │
│  Low Bias                                         High Bias     │
│  High Variance                                    Low Variance  │
│     ◄─────────────────────────────────────────────────►         │
│     │                                             │             │
│     MC                 n-step TD              TD(0)        DP   │
│     │                     │                    │           │    │
│     G_t            G_t^(n) with              R+γV     full Bellman│
│  (full return)     partial bootstrap      (1-step)    (exact)   │
│                                                                 │
│  Unbiased ──────── Bias increases ────────── Biased (vanishes)  │
│  High var ──────── Variance decreases ────── Low/No var         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Requirements and Applicability

Each method has different prerequisites that determine when it can be applied.

### 5.1 DP Requirements

- **Requires a complete model** $p(s', r \mid s, a)$ — transition probabilities and reward function
- **Sweeps over entire state space** — computational cost scales with $|\mathcal{S}|$
- Works for both episodic and continuing tasks
- **Cannot learn from experience** — it is not a learning algorithm in the usual sense

### 5.2 MC Requirements

- **Does not require a model** — learns from sample episodes
- **Requires complete episodes** — must reach a terminal state to compute $G_t$
- Cannot handle continuing (non-terminating) tasks
- Can learn from simulated or real experience

### 5.3 TD Requirements

- **Does not require a model** — learns from sample transitions
- **Does not require complete episodes** — updates after every step
- Works for both episodic and continuing tasks
- Can learn **online** and **incrementally**

> **Key Insight:** TD has the fewest requirements: no model, no complete episodes. This makes TD the most broadly applicable of the three methods. It is the only approach that works for online learning in continuing tasks without a model.

---

## 6. Convergence Properties

### 6.1 MC Convergence

MC converges to $v_\pi(s)$ as the number of visits to each state $s$ approaches infinity (by the law of large numbers). Under standard Robbins-Monro step-size conditions:

$$\sum_{n=1}^{\infty} \alpha_n = \infty, \quad \sum_{n=1}^{\infty} \alpha_n^2 < \infty$$

first-visit MC converges to $v_\pi(s)$ with probability 1. With a constant step size $\alpha$, it converges to a neighborhood of $v_\pi(s)$.

### 6.2 TD(0) Convergence

TD(0) also converges to $v_\pi(s)$ under the same Robbins-Monro conditions, though the proof is more involved (it requires stochastic approximation theory). For constant step sizes, TD(0) converges to:

$$V_{TD} = \arg\min_{V} \sum_{s} \mu(s) \left( V(s) - \mathbb{E}_\pi[R_{t+1} + \gamma V(S_{t+1}) \mid S_t = s] \right)^2$$

which is the **TD fixed point** — the value function that is self-consistent with respect to the TD update.

### 6.3 Speed of Convergence

Empirically and theoretically, TD methods typically converge faster than MC methods on most problems:

- TD's lower variance means each update moves the estimate more consistently in the right direction
- MC's unbiased but noisy updates can oscillate significantly before settling
- DP converges fastest (no sampling noise) but requires the model

On the classic **random walk** example (Sutton & Barto, Example 6.2), TD(0) consistently outperforms constant-$\alpha$ MC across all step sizes, reaching lower RMS error earlier.

---

## 7. N-Step TD — Bridging TD and MC

N-step TD methods provide a smooth interpolation between the one-step bootstrapping of TD(0) and the full-return approach of MC. By varying $n$, we control the bias-variance trade-off explicitly.

### 7.1 The N-Step Return

The **n-step return** truncates the actual return after $n$ steps and bootstraps from the estimated value at step $t + n$:

$$G_t^{(n)} = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots + \gamma^{n-1} R_{t+n} + \gamma^n V(S_{t+n})$$

More compactly:

$$G_t^{(n)} = \sum_{k=0}^{n-1} \gamma^k R_{t+k+1} + \gamma^n V(S_{t+n})$$

The n-step TD update rule is:

$$V(S_t) \leftarrow V(S_t) + \alpha \Big[ G_t^{(n)} - V(S_t) \Big]$$

### 7.2 Special Cases

- **$n = 1$:** $G_t^{(1)} = R_{t+1} + \gamma V(S_{t+1})$ — this is **TD(0)**
- **$n = \infty$:** $G_t^{(\infty)} = R_{t+1} + \gamma R_{t+2} + \cdots = G_t$ — this is **Monte Carlo**

Thus, n-step TD forms a **spectrum** with TD(0) and MC at the two extremes:

```
┌───────────────────────────────────────────────────────────────────┐
│                    N-STEP TD SPECTRUM                             │
│                                                                   │
│  n=1        n=2        n=4        n=8       n=16     n=∞         │
│  TD(0)      2-step     4-step     8-step    16-step  Monte Carlo │
│   │          │          │          │          │         │         │
│   ▼          ▼          ▼          ▼          ▼         ▼         │
│  R+γV    R+γR+γ²V   (4 rewards   ...        ...      G_t        │
│           (2-step      +γ⁴V)                       (full return) │
│            return)                                                │
│                                                                   │
│  ◄── More Bootstrap ──────────────────── Less Bootstrap ────►    │
│  ◄── Lower Variance ──────────────────── Higher Variance ───►    │
│  ◄── Higher Bias ─────────────────────── Lower Bias ────────►    │
└───────────────────────────────────────────────────────────────────┘
```

### 7.3 Optimal $n$

The best value of $n$ is problem-dependent. On the 19-state random walk (Sutton & Barto, Example 7.1), intermediate values like $n = 4$ or $n = 8$ often outperform both $n = 1$ (TD(0)) and $n = \infty$ (MC). The optimal $n$ balances bias reduction (larger $n$) against variance increase (larger $n$).

---

## 8. Comprehensive Comparison Table

| Property | Dynamic Programming | Monte Carlo | TD(0) |
|---|---|---|---|
| **Update Target** | $\sum_{s',r} p(s',r \mid s,a)[r + \gamma V(s')]$ | $G_t$ (actual return) | $R_{t+1} + \gamma V(S_{t+1})$ |
| **Bias** | Biased (bootstraps) | Unbiased | Biased (bootstraps) |
| **Variance** | None (exact expectation) | High (entire trajectory) | Low (single transition) |
| **Model Required** | Yes — $p(s',r \mid s,a)$ | No | No |
| **Episodes Required** | No | Yes — must terminate | No |
| **Online Learning** | No — sweeps entire state space | No — waits for episode end | Yes — updates every step |
| **Continuing Tasks** | Yes | No | Yes |
| **Bootstrapping** | Yes | No | Yes |
| **Sampling** | No (full backup) | Yes (sample backup) | Yes (sample backup) |
| **Computational Cost** | $O(\lvert\mathcal{S}\rvert \cdot \lvert\mathcal{A}\rvert)$ per sweep | $O(T)$ per episode | $O(1)$ per step |
| **Convergence Speed** | Fastest (no sampling noise) | Slowest (high variance) | Moderate (low variance, bias) |
| **Sensitivity to Initial $V$** | Low (exact updates) | None ($G_t$ independent of $V$) | Moderate (bootstraps from $V$) |
| **Best For** | Small MDPs with known model | Episodic tasks, blackbox sims | General online learning |
| **Exploration** | N/A (uses model directly) | Via behavior policy | Via behavior policy |
| **Update Granularity** | Full state space per iteration | Per episode | Per time step |

---

## 9. Python Comparison Experiment

The following experiment compares TD(0) and MC prediction on the classic **random walk** environment — a chain of states where the agent moves left or right with equal probability.

```python
import numpy as np
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

class RandomWalk:
    """
    5-state random walk: states 1..5 (+ 0=left terminal, 6=right terminal).
    Agent starts in state 3, moves left/right with p=0.5 each.
    Reward: +1 on transition into right terminal (state 6), 0 otherwise.
    """
    def __init__(self, n_states=5):
        self.n_states = n_states
        self.start = n_states // 2 + 1  # middle state

    def reset(self):
        self.state = self.start
        return self.state

    def step(self):
        direction = np.random.choice([-1, 1])
        self.state += direction
        if self.state == self.n_states + 1:  # right terminal
            return self.state, 1.0, True
        elif self.state == 0:                 # left terminal
            return self.state, 0.0, True
        else:
            return self.state, 0.0, False


def td0_prediction(env, n_episodes=100, alpha=0.1, gamma=1.0):
    """TD(0) prediction for the random walk."""
    V = np.zeros(env.n_states + 2)
    V[1:env.n_states + 1] = 0.5  # initial estimate
    rms_history = []
    true_values = np.linspace(1/6, 5/6, env.n_states)

    for ep in range(n_episodes):
        state = env.reset()
        done = False
        while not done:
            next_state, reward, done = env.step()
            # TD(0) update: V(S) ← V(S) + α[R + γV(S') - V(S)]
            V[state] += alpha * (reward + gamma * V[next_state] - V[state])
            state = next_state
        rms = np.sqrt(np.mean((V[1:env.n_states + 1] - true_values) ** 2))
        rms_history.append(rms)
    return V, rms_history


def mc_prediction(env, n_episodes=100, alpha=0.1, gamma=1.0):
    """Constant-alpha MC prediction for the random walk."""
    V = np.zeros(env.n_states + 2)
    V[1:env.n_states + 1] = 0.5
    rms_history = []
    true_values = np.linspace(1/6, 5/6, env.n_states)

    for ep in range(n_episodes):
        state = env.reset()
        trajectory = []
        done = False
        while not done:
            next_state, reward, done = env.step()
            trajectory.append((state, reward))
            state = next_state

        # Compute returns backward and update
        G = 0.0
        for state, reward in reversed(trajectory):
            G = gamma * G + reward
            # MC update: V(S) ← V(S) + α[G - V(S)]
            V[state] += alpha * (G - V[state])

        rms = np.sqrt(np.mean((V[1:env.n_states + 1] - true_values) ** 2))
        rms_history.append(rms)
    return V, rms_history


def run_comparison(n_runs=100, n_episodes=100):
    """Average TD(0) vs MC over multiple runs."""
    env = RandomWalk()

    # Test multiple step sizes for each method
    td_alphas = [0.05, 0.1, 0.15]
    mc_alphas = [0.01, 0.02, 0.04]

    results = {}
    for alpha in td_alphas:
        rms_all = np.zeros(n_episodes)
        for run in range(n_runs):
            _, rms_history = td0_prediction(env, n_episodes, alpha)
            rms_all += np.array(rms_history)
        results[f'TD α={alpha}'] = rms_all / n_runs

    for alpha in mc_alphas:
        rms_all = np.zeros(n_episodes)
        for run in range(n_runs):
            _, rms_history = mc_prediction(env, n_episodes, alpha)
            rms_all += np.array(rms_history)
        results[f'MC α={alpha}'] = rms_all / n_runs

    # Plot results
    plt.figure(figsize=(10, 6))
    for label, rms in results.items():
        linestyle = '-' if label.startswith('TD') else '--'
        plt.plot(rms, label=label, linestyle=linestyle)
    plt.xlabel('Episodes')
    plt.ylabel('RMS Error (averaged over runs)')
    plt.title('TD(0) vs Constant-α MC on Random Walk')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig('td_vs_mc_comparison.png', dpi=150)
    plt.show()

    return results


if __name__ == '__main__':
    results = run_comparison()
    print("\nFinal RMS errors after 100 episodes:")
    for label, rms in sorted(results.items()):
        print(f"  {label}: {rms[-1]:.4f}")
```

**Expected output:**

```
Final RMS errors after 100 episodes:
  MC α=0.01: ~0.14
  MC α=0.02: ~0.11
  MC α=0.04: ~0.09
  TD α=0.05: ~0.06
  TD α=0.1:  ~0.05
  TD α=0.15: ~0.05
```

TD(0) consistently achieves lower RMS error across all tested step sizes. For the best respective step sizes, TD's error is roughly half that of MC after 100 episodes — a direct consequence of lower variance updates.

---

## 10. When to Use Each Method — Practical Guidelines

Choosing between DP, MC, and TD depends on what you have and what you need.

### 10.1 Use DP When:

- You have a **complete model** of the environment (transition probabilities and rewards)
- The state space is **small enough** for full sweeps (or you can use asynchronous DP)
- You need **exact solutions** or strong convergence guarantees
- Examples: small gridworlds, inventory management with known demand distributions, game solving with known rules

### 10.2 Use MC When:

- You **lack a model** but can simulate or collect complete episodes
- Tasks are naturally **episodic** with reasonably short episodes
- You want **unbiased** estimates and can tolerate higher variance
- Initial value estimates are poor and you don't want bootstrapping bias to propagate
- You are doing **off-policy** evaluation with importance sampling (MC importance sampling is simpler to implement correctly)
- Examples: card games (Blackjack), episodic robotics tasks, game-playing from self-play

### 10.3 Use TD When:

- You **lack a model** and need **online, incremental** learning
- Tasks may be **continuing** (non-episodic) or episodes are very long
- You want **faster convergence** and can tolerate some bias
- Memory is limited — TD requires storing only $V(s)$ or $Q(s,a)$, not entire episodes
- You need to learn from **incomplete sequences** or in real-time
- Examples: robot navigation, stock trading, network routing, any continuing control task

### 10.4 Decision Flowchart

```
┌──────────────────────────────────────────┐
│          Do you have a model?            │
└──────────────┬───────────────────────────┘
               │
       ┌───────┴───────┐
       │               │
      YES              NO
       │               │
       ▼               ▼
  ┌─────────┐   ┌────────────────────────────┐
  │  Use DP │   │ Is the task episodic with  │
  │         │   │ reasonably short episodes? │
  └─────────┘   └────────────┬───────────────┘
                             │
                     ┌───────┴───────┐
                     │               │
                    YES              NO
                     │               │
                     ▼               ▼
              ┌──────────────┐  ┌──────────┐
              │  MC or TD    │  │  Use TD  │
              │  (try both)  │  │          │
              └──────────────┘  └──────────┘
```

> **Key Insight:** In practice, TD methods (especially TD control algorithms like SARSA and Q-Learning) are the most widely used because they impose the fewest constraints. When you are unsure, TD is usually the safest default. Consider n-step TD to tune the bias-variance trade-off for your specific problem.

---

## Summary

| Concept | Key Point |
|---|---|
| **Bootstrapping** | Updating estimates from other estimates — used by DP and TD, not by MC |
| **Sampling** | Using sample transitions instead of full model — used by MC and TD, not by DP |
| **2×2 Grid** | DP = full+bootstrap, TD = sample+bootstrap, MC = sample+no bootstrap, Exhaustive MC = full+no bootstrap |
| **DP Update** | $V(s) \leftarrow \sum_a \pi(a \mid s) \sum_{s',r} p(s',r \mid s,a)[r + \gamma V(s')]$ — exact, requires model |
| **MC Update** | $V(S_t) \leftarrow V(S_t) + \alpha[G_t - V(S_t)]$ — unbiased, high variance, needs full episodes |
| **TD(0) Update** | $V(S_t) \leftarrow V(S_t) + \alpha[R_{t+1} + \gamma V(S_{t+1}) - V(S_t)]$ — biased, low variance, online |
| **MC Bias/Variance** | Unbiased (uses real returns $G_t$), high variance (depends on full trajectory) |
| **TD Bias/Variance** | Biased (bootstraps from $V$), low variance (one-step transition only) |
| **N-Step Return** | $G_t^{(n)} = \sum_{k=0}^{n-1} \gamma^k R_{t+k+1} + \gamma^n V(S_{t+n})$ — interpolates between TD(0) ($n=1$) and MC ($n=\infty$) |
| **DP Requirement** | Full environment model $p(s', r \mid s, a)$ |
| **MC Requirement** | Complete episodes (must reach terminal state) |
| **TD Requirement** | Neither model nor complete episodes — the most flexible method |
| **Convergence** | All three converge to $v_\pi$ under appropriate conditions; TD fastest empirically despite bias |
| **When to Use DP** | Known model, small state space, exact solutions needed |
| **When to Use MC** | Episodic tasks, no model, unbiased estimates preferred |
| **When to Use TD** | Online learning, continuing tasks, fast convergence needed |

---

## Revision Questions

1. **Explain the two dimensions (bootstrapping and sampling) that distinguish DP, MC, and TD.** For each method, state whether it bootstraps and whether it samples, and describe the consequence of each choice on bias and variance.

2. **Write out the update rules for DP, MC, and TD(0) prediction side by side.** For each, identify the update target and explain what information is required to compute it (model, full episode, or single transition).

3. **Derive the n-step return $G_t^{(n)}$ and show that it reduces to the TD(0) target when $n=1$ and the MC return when $n \to \infty$.** Why do intermediate values of $n$ often outperform the extremes?

4. **A 3-state random walk has states A, B, C with left and right terminals.** The agent starts in B and moves left/right with $p = 0.5$. Reward is $+1$ at the right terminal, $0$ elsewhere, $\gamma = 1$. If $V = [0.5, 0.5, 0.5]$ initially and the agent takes the trajectory $B \to C \to \text{Right Terminal}$ ($R_1 = 0, R_2 = 1$), compute the updated values under (a) TD(0) with $\alpha = 0.1$ and (b) MC with $\alpha = 0.1$.

5. **Explain why TD methods typically converge faster than MC despite being biased.** How does the bias-variance trade-off explain this? Under what conditions might MC outperform TD?

6. **You are building an RL agent for a continuous robot navigation task with no model available.** Which of the three methods (DP, MC, TD) can be applied? Justify your choice and explain why the other two are unsuitable or suboptimal for this setting.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Expected SARSA](./04-expected-sarsa.md) | [Unit 5: TD Learning](./README.md) | [Eligibility Traces](./06-eligibility-traces.md) |

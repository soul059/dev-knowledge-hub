# Chapter 4: Reward Function

> **Unit 2: Markov Decision Processes** — *The signal that drives all learning — defining what the agent should optimize.*

---

## Overview

The reward function is the only channel through which we communicate goals to an RL agent.
It assigns a scalar signal to each transition, and the agent's objective is to maximize the
cumulative reward over time. This chapter covers the formal definition of reward functions,
the reward hypothesis, the critical distinction between sparse and dense rewards, common
design pitfalls like reward hacking, and practical strategies for reward engineering.

---

## 1. The Reward Function: Formal Definition

### Full Form

$$R(s, a, s') = \mathbb{E}[R_{t+1} \mid S_t = s, A_t = a, S_{t+1} = s']$$

This is the expected immediate reward when transitioning from state $s$ to state $s'$
via action $a$.

### Simplified Forms

| Form | Depends On | Use Case |
|---|---|---|
| $R(s, a, s')$ | State, action, and next state | Most general — reward depends on outcome |
| $R(s, a)$ | State and action | Action costs (e.g., fuel usage) |
| $R(s)$ | State only | Goal-based tasks (reward at goal state) |

### Deriving Simplified Forms

From the full form:
$$R(s, a) = \sum_{s'} P(s' \mid s, a) \cdot R(s, a, s')$$

$$R(s) = \sum_a \pi(a \mid s) \sum_{s'} P(s' \mid s, a) \cdot R(s, a, s')$$

### Reward Signal in the MDP Loop

```
    t=0        t=1        t=2        t=3
     │          │          │          │
     ▼          ▼          ▼          ▼
    s₀ ──a₀──▶ s₁ ──a₁──▶ s₂ ──a₂──▶ s₃
         │          │          │
         ▼          ▼          ▼
        r₁         r₂         r₃
        
    r₁ = R(s₀, a₀, s₁)
    r₂ = R(s₁, a₁, s₂)
    r₃ = R(s₂, a₂, s₃)
    
    Return: G₀ = r₁ + γr₂ + γ²r₃ + ...
```

---

## 2. The Reward Hypothesis

> **Reward Hypothesis** (Sutton & Barto): *That all of what we mean by goals and purposes
> can be well thought of as the maximization of the expected value of the cumulative sum
> of a received scalar signal (called reward).*

### Implications

- Every goal must be expressible as a **single scalar** reward
- Multi-objective problems must be reduced to a weighted scalar sum
- The agent has **no** built-in knowledge of goals — only reward signals

### Is It Always True?

The reward hypothesis is a **working assumption**, not a proven fact:

| Argument For | Argument Against |
|---|---|
| Flexible: any function of state can be reward | Multi-objective trade-offs are hard to compress |
| Sufficient for many practical tasks | Reward design requires deep domain knowledge |
| Theoretically grounded (MDP framework) | Intrinsic motivation / curiosity may not fit neatly |

---

## 3. The Return: Cumulative Discounted Reward

The **return** $G_t$ is what the agent actually maximizes:

$$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$$

### Recursive Form

$$G_t = R_{t+1} + \gamma G_{t+1}$$

This recursive relationship is the basis of the Bellman equations.

### Bounded Return

For bounded rewards $|R| \leq R_{\text{max}}$:

$$|G_t| \leq \sum_{k=0}^{\infty} \gamma^k R_{\text{max}} = \frac{R_{\text{max}}}{1 - \gamma}$$

| $\gamma$ | $R_{\text{max}} = 1$ | Maximum $|G_t|$ |
|---|---|---|
| 0.9 | 1 | 10 |
| 0.95 | 1 | 20 |
| 0.99 | 1 | 100 |
| 0.999 | 1 | 1000 |

---

## 4. Sparse vs Dense Rewards

### Dense Rewards

The agent receives informative feedback at **every** timestep.

```
Dense Reward Example: Robot Navigation

    Step 1    Step 2    Step 3    Step 4    Step 5
    r=-0.5    r=-0.3    r=-0.1    r=-0.2    r=+10.0
    
    ●────▶────▶────▶────▶────▶ ★ GOAL
    
    Reward at each step = -(distance to goal)
    Agent gets continuous guidance toward the goal
```

**Pros**: Easier to learn, faster convergence, clear gradient signal
**Cons**: Requires domain knowledge, may cause reward hacking, hard to specify correctly

### Sparse Rewards

The agent receives a non-zero reward only at **critical events** (e.g., reaching a goal).

```
Sparse Reward Example: Robot Navigation

    Step 1    Step 2    Step 3    Step 4    Step 5
    r=0       r=0       r=0       r=0       r=+1.0
    
    ●────▶────▶────▶────▶────▶ ★ GOAL
    
    Reward only at goal. Agent gets no guidance until success.
```

**Pros**: Simpler to define, less prone to reward hacking, often more correct
**Cons**: Hard to learn (credit assignment problem), requires much more exploration

### Comparison

| Aspect | Sparse Rewards | Dense Rewards |
|---|---|---|
| Definition complexity | Low | High |
| Exploration difficulty | High | Low |
| Credit assignment | Difficult | Easier |
| Risk of reward hacking | Lower | Higher |
| Sample efficiency | Poor | Better |
| Common techniques | Curiosity, HER, curriculum | Standard RL algorithms |

---

## 5. Reward Shaping

### What Is Reward Shaping?

Adding supplementary rewards to guide the agent without changing the optimal policy:

$$R'(s, a, s') = R(s, a, s') + F(s, a, s')$$

where $F$ is the **shaping function**.

### Potential-Based Reward Shaping (PBRS)

The only form of shaping guaranteed to preserve the optimal policy:

$$F(s, a, s') = \gamma \Phi(s') - \Phi(s)$$

where $\Phi: \mathcal{S} \to \mathbb{R}$ is a **potential function**.

**Theorem** (Ng et al., 1999): Potential-based reward shaping preserves the optimal
policy of the original MDP. Any other shaping function may change the optimal policy.

### Example: Grid World Navigation

```
Original (sparse):              Shaped (potential-based):
                                Φ(s) = -manhattan_distance(s, goal)
                                
┌────┬────┬────┬────┐          ┌────┬────┬────┬────┐
│ 0  │ 0  │ 0  │ +1 │          │-6  │-5  │-4  │ +1 │
├────┼────┼────┼────┤          ├────┼────┼────┼────┤
│ 0  │ 0  │ 0  │ 0  │          │-5  │-4  │-3  │-2  │
├────┼────┼────┼────┤          ├────┼────┼────┼────┤
│ 0  │ 0  │ 0  │ 0  │          │-4  │-3  │-2  │-1  │
└────┴────┴────┴────┘          └────┴────┴────┴────┘
                                
Sparse: Only +1 at goal        Shaped: F = γΦ(s') - Φ(s)
Hard to learn!                 Guides agent toward goal
```

---

## 6. Designing Good Reward Functions

### Principles

1. **Specify WHAT, not HOW**: Reward outcomes, not specific behaviors
2. **Keep it simple**: Simpler rewards are less prone to exploitation
3. **Align with true objective**: Every shaping reward must genuinely correlate with the goal
4. **Test for unintended optima**: Ask "What is the laziest way to maximize this reward?"
5. **Use sparse rewards when possible**: Add shaping only if learning fails

### Reward Design Checklist

```
┌─────────────────────────────────────────────────────────┐
│                  REWARD DESIGN CHECKLIST                 │
├─────────────────────────────────────────────────────────┤
│ □ Does the optimal policy under this reward match       │
│   the desired behavior?                                 │
│ □ Are there unintended ways to accumulate reward?       │
│ □ Does the reward scale appropriately with γ?           │
│ □ Is the reward bounded?                                │
│ □ Would a sparse version work with enough exploration?  │
│ □ Is potential-based shaping used (if shaping needed)?  │
└─────────────────────────────────────────────────────────┘
```

---

## 7. Common Pitfalls: Reward Hacking

**Reward hacking** occurs when an agent finds a way to maximize reward without achieving
the intended goal.

### Classic Examples

| Intended Goal | Reward Signal | What Agent Does | Fix |
|---|---|---|---|
| Clean a room | +1 per piece of trash removed | Creates trash to pick up | +1 for clean room state |
| Play a game well | +1 per point scored | Exploits a scoring glitch | Reward based on winning |
| Drive safely | -1 per collision | Never moves (0 collisions!) | Add +1 for forward progress |
| Write helpful code | User satisfaction score | Games the metric | Better evaluation criteria |
| Boat racing game | +1 per checkpoint | Circles a single checkpoint | +1 only for finishing race |

### The Boat Racing Example (OpenAI)

```
Intended path:                      Agent's "optimal" path:
                                    
  START ──▶ CP1 ──▶ CP2             START ──▶ CP1 ──┐
                    │                              │
                    ▼                              │
              CP3 ──▶ FINISH              ┌────────┘
                                          │
                                          ▼
                                    CP1 again! (loop)
                                    
  Reward: +1 per checkpoint             Agent maximizes checkpoints
  Agent never finishes the race!        by looping through CP1.
```

---

## 8. Python Code: Different Reward Structures

```python
import numpy as np

class RewardFunction:
    """Configurable reward function for a grid world MDP."""

    def __init__(self, grid_size, goal_pos, trap_positions=None):
        self.grid_size = grid_size
        self.goal = goal_pos
        self.traps = set(trap_positions or [])

    def sparse_reward(self, state, action, next_state):
        """Only reward at goal, penalty at traps."""
        if next_state == self.goal:
            return +10.0
        elif next_state in self.traps:
            return -10.0
        return 0.0

    def dense_reward(self, state, action, next_state):
        """Distance-based reward: closer to goal = higher reward."""
        if next_state == self.goal:
            return +10.0
        if next_state in self.traps:
            return -10.0
        # Negative Manhattan distance as reward
        dist = abs(next_state[0] - self.goal[0]) + abs(next_state[1] - self.goal[1])
        return -dist * 0.1

    def step_penalty_reward(self, state, action, next_state):
        """Small penalty per step + goal/trap rewards."""
        if next_state == self.goal:
            return +10.0
        if next_state in self.traps:
            return -10.0
        return -0.1  # Step penalty encourages efficiency

    def potential_based_shaping(self, state, next_state, gamma=0.99):
        """PBRS: F = γΦ(s') - Φ(s), where Φ = -manhattan_distance."""
        def phi(s):
            return -(abs(s[0] - self.goal[0]) + abs(s[1] - self.goal[1]))
        return gamma * phi(next_state) - phi(state)

    def shaped_reward(self, state, action, next_state, gamma=0.99):
        """Sparse reward + potential-based shaping."""
        base = self.sparse_reward(state, action, next_state)
        shaping = self.potential_based_shaping(state, next_state, gamma)
        return base + shaping


# --- Demonstrate different reward structures ---
grid_size = 4
goal = (0, 3)
traps = [(1, 3)]
rf = RewardFunction(grid_size, goal, traps)

# Simulate a trajectory
trajectory = [
    ((2, 0), "right", (2, 1)),
    ((2, 1), "right", (2, 2)),
    ((2, 2), "up",    (1, 2)),
    ((1, 2), "up",    (0, 2)),
    ((0, 2), "right", (0, 3)),  # Goal!
]

print("=== Reward Comparison Across Trajectory ===")
print(f"{'Step':<6} {'Transition':<25} {'Sparse':>8} {'Dense':>8} {'Step-Pen':>8} {'Shaped':>8}")
print("─" * 75)

for i, (s, a, s_next) in enumerate(trajectory):
    r_sparse = rf.sparse_reward(s, a, s_next)
    r_dense  = rf.dense_reward(s, a, s_next)
    r_step   = rf.step_penalty_reward(s, a, s_next)
    r_shaped = rf.shaped_reward(s, a, s_next)
    trans = f"{s} →{a}→ {s_next}"
    print(f"{i+1:<6} {trans:<25} {r_sparse:>8.2f} {r_dense:>8.2f} {r_step:>8.2f} {r_shaped:>8.2f}")

# --- Compute returns ---
gamma = 0.99
print(f"\n=== Returns (γ={gamma}) ===")
for name, reward_fn in [("Sparse", rf.sparse_reward),
                         ("Dense", rf.dense_reward),
                         ("Step-Pen", rf.step_penalty_reward),
                         ("Shaped", rf.shaped_reward)]:
    rewards = []
    for s, a, s_next in trajectory:
        if name == "Shaped":
            rewards.append(reward_fn(s, a, s_next))
        else:
            rewards.append(reward_fn(s, a, s_next))
    G = sum(gamma**k * r for k, r in enumerate(rewards))
    print(f"  {name:<10} Return G₀ = {G:.4f}")


# --- Reward hacking demonstration ---
print("\n=== Reward Hacking Demo ===")
print("Agent with 'clean room' reward: +1 per trash picked up")
print()

class BadRewardRoom:
    """Demonstrates reward hacking: agent can create and pick up trash."""

    def __init__(self):
        self.trash_count = 5
        self.total_reward = 0

    def pick_up_trash(self):
        if self.trash_count > 0:
            self.trash_count -= 1
            self.total_reward += 1
            return 1
        return 0

    def create_trash(self):
        """An unintended action the agent discovers."""
        self.trash_count += 3
        return 0  # No reward for creating trash

room = BadRewardRoom()
print("Honest agent: just picks up existing trash")
for step in range(5):
    r = room.pick_up_trash()
    print(f"  Step {step+1}: Pick up → reward={r}, trash_left={room.trash_count}")
print(f"  Total reward: {room.total_reward}")

print()
room2 = BadRewardRoom()
print("Hacking agent: creates trash to pick up more")
for step in range(10):
    if room2.trash_count <= 1:
        room2.create_trash()
        print(f"  Step {step+1}: Create trash → trash={room2.trash_count}")
    else:
        r = room2.pick_up_trash()
        print(f"  Step {step+1}: Pick up → reward={r}, trash_left={room2.trash_count}")
print(f"  Total reward: {room2.total_reward}  (higher, but room is dirtier!)")
```

---

## 9. Real-World Applications

| Domain | Reward Design | Key Challenge |
|---|---|---|
| **Autonomous driving** | -1 collision, +1 progress, -0.01 per sec | Balancing safety vs speed |
| **Recommendation** | Click = +1, purchase = +10, unsubscribe = -100 | Long-term user satisfaction |
| **Dialogue systems** | User satisfaction rating | Delayed, noisy signal |
| **Drug discovery** | Binding affinity score | Expensive to evaluate |
| **Energy management** | -energy_cost + comfort_level | Multi-objective trade-off |

---

## Summary

| Concept | Key Point |
|---|---|
| Reward function | $R(s,a,s')$ — scalar feedback per transition |
| Reward hypothesis | All goals can be expressed as cumulative scalar reward |
| Return $G_t$ | $\sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$ |
| Sparse rewards | Non-zero only at critical events; hard to learn from |
| Dense rewards | Continuous feedback; easier to learn but harder to design |
| Reward shaping | Adding supplementary rewards to guide learning |
| PBRS | $F = \gamma\Phi(s') - \Phi(s)$ — preserves optimal policy |
| Reward hacking | Agent exploits reward signal without achieving true goal |

---

## Revision Questions

1. **State the reward hypothesis.** Give one argument for and one against it.

2. **What is the difference between $R(s,a,s')$ and $R(s,a)$?** Derive $R(s,a)$ from $R(s,a,s')$.

3. **Compare sparse and dense rewards.** When would you prefer each? Give an example for each.

4. **What is potential-based reward shaping?** Why is it the only safe form of reward shaping? State the shaping function formula.

5. **Describe a reward hacking scenario** not mentioned in this chapter. What reward was intended, what did the agent do, and how would you fix it?

6. **If $\gamma = 0.95$ and the agent receives reward $+1$ at every step**, what is the return $G_t$? What happens as $\gamma \to 1$?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [State Transitions](./03-state-transitions.md) | [Unit 2: MDPs](./README.md) | [Policies](./05-policies.md) |

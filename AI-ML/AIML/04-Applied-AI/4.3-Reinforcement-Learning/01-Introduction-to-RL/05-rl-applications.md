# Chapter 5: RL Applications

> **Unit 1 · Introduction to RL** — File 5 of 5

---

## 1. Overview

Reinforcement learning has moved far beyond academic toy problems. Today it powers
game-playing AIs that defeat world champions, trains robots to walk, aligns large
language models, and optimises real-world systems from data centres to hospitals.

```
 ╔════════════════════════════════════════════════════════════════════╗
 ║             REINFORCEMENT LEARNING IN THE REAL WORLD              ║
 ╠════════════════════════════════════════════════════════════════════╣
 ║                                                                    ║
 ║  🎮 Games ──── Atari · Go · Chess · StarCraft · Dota 2           ║
 ║  🤖 Robotics ─ Manipulation · Locomotion · Sim-to-Real           ║
 ║  🚗 Driving ── Lane keeping · Path planning · Traffic signals    ║
 ║  📊 RecSys ─── Content ranking · Ad placement · Playlists        ║
 ║  💰 Finance ── Portfolio · Trading · Risk management             ║
 ║  💊 Health ─── Drug dosing · Treatment plans · Clinical trials   ║
 ║  🗣️ LLMs ───── RLHF · Instruction following · Safety            ║
 ║  🏭 Industry ─ HVAC · Supply chain · Chip design                 ║
 ║                                                                    ║
 ╚════════════════════════════════════════════════════════════════════╝
```

---

## 2. Games

### 2.1 Atari — Deep Q-Networks (DQN)

In 2013, DeepMind showed that a single RL agent could learn to play dozens of Atari
2600 games at human-level performance directly from raw pixels.

```
  ┌────────────────────────────────────────────────────────────┐
  │                    DQN ARCHITECTURE                        │
  │                                                            │
  │  Raw Pixels    Conv Layers      Fully Connected    Q-values│
  │  (84 × 84)                                                │
  │  ┌─────┐      ┌────────┐       ┌──────┐       ┌────────┐ │
  │  │Frame│─────▶│ Conv2D │──────▶│  FC  │──────▶│Q(s, a₁)│ │
  │  │Stack│      │  ×3    │       │ 512  │       │Q(s, a₂)│ │
  │  │ ×4  │      └────────┘       └──────┘       │  ...   │ │
  │  └─────┘                                       │Q(s, aₙ)│ │
  │                                                └────────┘ │
  │  Loss: (r + γ max_a' Q(s',a'; θ⁻) − Q(s,a; θ))²         │
  └────────────────────────────────────────────────────────────┘
```

| Component           | Detail                                       |
|---------------------|----------------------------------------------|
| **State**           | Stack of 4 greyscale frames (84×84×4)        |
| **Actions**         | Discrete joystick + button (varies by game)  |
| **Reward**          | Change in game score                         |
| **Key innovations** | Experience replay, target network            |
| **Algorithm**       | DQN (Deep Q-Network)                         |

### 2.2 Go — AlphaGo & AlphaZero

```
  ┌────────────────────────────────────────────────────────────┐
  │              ALPHAGO / ALPHAZERO TIMELINE                   │
  │                                                            │
  │  2016  AlphaGo      Beat Lee Sedol 4-1 (SL + RL + MCTS)  │
  │  2017  AlphaGo Zero  Self-play only, no human data         │
  │  2017  AlphaZero     Go + Chess + Shogi from scratch       │
  │  2019  MuZero        Learns the environment model too      │
  │                                                            │
  │  Key Idea: MCTS (Monte Carlo Tree Search) guided by        │
  │            a policy network π(a|s) and value network V(s)  │
  └────────────────────────────────────────────────────────────┘
```

**Mathematical formulation:**

- Policy network: $\pi_\theta(a \mid s) \approx \pi^*(a \mid s)$
- Value network: $V_\theta(s) \approx V^*(s)$
- Self-play generates training data; MCTS improves the policy.
- Loss: $\mathcal{L} = (z - V_\theta(s))^2 - \pi^T \log \pi_\theta(s) + c\|\theta\|^2$

### 2.3 StarCraft II — AlphaStar

| Aspect          | Detail                                          |
|-----------------|-------------------------------------------------|
| **State**       | Spatial features, unit lists, game stats        |
| **Actions**     | Structured (function ID + arguments)            |
| **Complexity**  | ~10²⁶ possible actions per step                |
| **Training**    | Population-based self-play, multi-agent league  |
| **Algorithm**   | Transformer + LSTM + RL (IMPALA variant)        |
| **Result**      | Grandmaster level in all three StarCraft races  |

---

## 3. Robotics

### 3.1 Overview

```
  ┌────────────────────────────────────────────────────────────┐
  │              RL IN ROBOTICS PIPELINE                        │
  │                                                            │
  │   ┌──────────┐     ┌──────────┐     ┌──────────┐         │
  │   │ Train in │     │ Transfer │     │ Deploy   │         │
  │   │   SIM    │────▶│ Sim→Real │────▶│ on ROBOT │         │
  │   │(millions │     │ (domain  │     │(real     │         │
  │   │ of eps)  │     │  randomi-│     │ world)   │         │
  │   └──────────┘     │  zation) │     └──────────┘         │
  │                     └──────────┘                           │
  │   Algorithms: PPO, SAC, TD3, DDPG                         │
  └────────────────────────────────────────────────────────────┘
```

### 3.2 Application Areas

| Task               | State                        | Action                  | Algorithm     |
|--------------------|------------------------------|-------------------------|---------------|
| **Grasping**       | RGB-D image, joint angles    | Gripper position        | SAC, HER      |
| **Locomotion**     | Joint angles, velocities     | Joint torques           | PPO, TRPO     |
| **Dexterous manip**| Finger positions, object pose| Finger joint torques    | PPO + DR      |
| **Drone flight**   | Position, velocity, IMU      | Rotor speeds            | TD3           |

### 3.3 Sim-to-Real Transfer

**Domain Randomisation**: Train with randomised physics parameters (friction, mass,
delays) so the policy is robust to the real-world "domain."

$$\pi^* = \arg\max_\pi \; \mathbb{E}_{\xi \sim \mathcal{P}(\xi)} \left[ \mathbb{E}_\pi \left[ \sum_t \gamma^t r_t \mid \xi \right] \right]$$

where $\xi$ represents randomised environment parameters.

---

## 4. Autonomous Driving

```
  ┌────────────────────────────────────────────────────────────┐
  │           RL FOR AUTONOMOUS DRIVING                        │
  │                                                            │
  │   Perception ──▶ Planning ──▶ Control                     │
  │                     ▲                                      │
  │                     │ RL can handle all three               │
  │                     │ or just planning/control              │
  │                                                            │
  │   State : Camera images, LIDAR, vehicle speed, GPS        │
  │   Action: Steering angle, acceleration, braking            │
  │   Reward: +progress, -collision, -lane deviation           │
  │                                                            │
  │   Challenges: Safety constraints, sim-to-real gap,         │
  │               multi-agent traffic, rare events              │
  └────────────────────────────────────────────────────────────┘
```

| Company / Project    | RL Application                          | Technique              |
|----------------------|-----------------------------------------|------------------------|
| Waymo                | Motion planning                         | Imitation + RL         |
| Tesla                | Neural network planner training         | Large-scale simulation |
| CARLA simulator      | RL research benchmark                   | PPO, SAC               |

---

## 5. Recommendation Systems

RL models recommendation as a sequential decision problem: each recommendation is an
action; user engagement (click, watch time, purchase) is the reward.

```
  ┌────────────────────────────────────────────────────────────┐
  │          RL FOR RECOMMENDATIONS                            │
  │                                                            │
  │  User arrives ──▶ Agent recommends item ──▶ User reacts   │
  │       │                                         │          │
  │       │           ┌───────────────────┐         │          │
  │       └───────────│ State: user       │◀────────┘          │
  │                   │ history, context  │  Reward:           │
  │                   └───────────────────┘  click / skip      │
  │                                                            │
  │  Key: Optimise long-term engagement, not just click-rate  │
  └────────────────────────────────────────────────────────────┘
```

| Platform  | RL Technique                    | Objective                      |
|-----------|---------------------------------|--------------------------------|
| YouTube   | REINFORCE for ranking           | Long-term user satisfaction    |
| Netflix   | Contextual bandits              | Artwork personalisation        |
| Spotify   | Multi-armed bandits             | Playlist generation            |
| E-commerce| Q-Learning for page layout      | Purchase conversion            |

---

## 6. Finance

### 6.1 Portfolio Optimisation

$$a_t = \text{portfolio weights} \quad w_t \in \Delta^n \quad (\text{simplex})$$

$$r_{t+1} = \log\left(\frac{\text{portfolio value}_{t+1}}{\text{portfolio value}_t}\right)$$

### 6.2 Algorithmic Trading

```
  ┌────────────────────────────────────────────────────────────┐
  │           RL FOR TRADING                                   │
  │                                                            │
  │  State : Price history, indicators, position, P&L         │
  │  Action: {BUY, SELL, HOLD} or continuous position size    │
  │  Reward: Realised P&L, risk-adjusted return (Sharpe)      │
  │                                                            │
  │  Challenges:                                               │
  │    • Non-stationary markets                                │
  │    • Transaction costs                                     │
  │    • Partial observability                                 │
  │    • Market impact of agent's own actions                  │
  └────────────────────────────────────────────────────────────┘
```

| Application              | State Features                 | Algorithm          |
|--------------------------|--------------------------------|--------------------|
| Portfolio management     | Returns, volatility, macro     | PPO, A2C           |
| Order execution          | Order book, inventory          | DQN, DDPG          |
| Market making            | Bid-ask spread, inventory      | Multi-agent RL     |

---

## 7. Healthcare

### 7.1 Treatment Policy Optimisation

RL can learn optimal treatment policies from observational clinical data.

$$s_t = \text{patient vitals, lab values, history}$$
$$a_t = \text{drug, dose, procedure}$$
$$r_t = \text{patient outcome (survival, recovery)}$$

### 7.2 Examples

| Application              | State                          | Action              | Impact           |
|--------------------------|--------------------------------|---------------------|------------------|
| Sepsis treatment         | Vitals, lab tests              | IV fluid, vasopressor| Reduced mortality|
| Radiation therapy        | Tumour size, organ doses       | Beam intensity       | Better outcomes  |
| Diabetes management      | Blood glucose, meal, exercise  | Insulin dose         | Stable glucose   |
| Clinical trial design    | Interim results, patient data  | Continue / stop trial| Efficiency       |

### 7.3 Challenges

- **Off-policy evaluation**: Cannot experiment freely with patients.
- **Safety constraints**: Must guarantee "do no harm."
- **Data quality**: Observational data has confounders.

---

## 8. RLHF for Large Language Models

### 8.1 The RLHF Pipeline

```
  ┌────────────────────────────────────────────────────────────────┐
  │              RLHF PIPELINE (e.g., ChatGPT)                     │
  │                                                                │
  │  Step 1: Supervised Fine-Tuning (SFT)                          │
  │  ┌──────────┐    human-written     ┌──────────┐               │
  │  │ Pre-     │    demonstrations    │  SFT     │               │
  │  │ trained  │ ──────────────────▶  │  Model   │               │
  │  │ LLM      │                      └────┬─────┘               │
  │  └──────────┘                           │                      │
  │                                          ▼                      │
  │  Step 2: Train Reward Model                                    │
  │  ┌──────────┐    human preferences  ┌──────────┐              │
  │  │ SFT      │    (A > B rankings)   │  Reward  │              │
  │  │ outputs  │ ──────────────────▶   │  Model   │              │
  │  └──────────┘                       └────┬─────┘              │
  │                                          │                      │
  │  Step 3: RL Fine-Tuning (PPO)            ▼                      │
  │  ┌──────────┐    reward signal     ┌──────────┐               │
  │  │ Policy   │ ◀─────────────────── │  Reward  │               │
  │  │ (LLM)    │                      │  Model   │               │
  │  │          │ ──────────────────▶  │          │               │
  │  └──────────┘    generated text    └──────────┘               │
  │                                                                │
  │  Result: LLM that follows instructions and is aligned          │
  └────────────────────────────────────────────────────────────────┘
```

### 8.2 RLHF Formulation

- **State** $s_t$: tokens generated so far.
- **Action** $a_t$: next token to generate.
- **Reward**: $R(x, y) = R_\phi(x, y) - \beta \, \text{KL}[\pi_\theta \| \pi_{\text{ref}}]$

The KL penalty prevents the model from diverging too far from the SFT model.

$$\max_\theta \; \mathbb{E}_{x \sim D,\; y \sim \pi_\theta(\cdot|x)} \big[ R_\phi(x, y) - \beta \, \text{KL}(\pi_\theta \| \pi_{\text{ref}}) \big]$$

### 8.3 Impact

| System        | RL Technique | Purpose                                   |
|---------------|--------------|-------------------------------------------|
| ChatGPT       | PPO (RLHF)  | Instruction following, helpfulness        |
| Claude        | RLHF + CAI   | Harmlessness, honesty                     |
| Gemini        | RLHF         | Alignment, safety                         |
| Llama 2       | PPO (RLHF)  | Safety, helpfulness                       |

---

## 9. Industrial Applications

```
  ┌────────────────────────────────────────────────────────────┐
  │           MORE RL APPLICATIONS                             │
  │                                                            │
  │  🏭 Google Data Centre Cooling                            │
  │     → RL reduced cooling energy by 40%                     │
  │     → State: sensor readings; Action: cooling settings     │
  │                                                            │
  │  📦 Supply Chain / Inventory Management                    │
  │     → Optimal ordering policies under uncertainty          │
  │                                                            │
  │  🔌 Power Grid Management                                 │
  │     → Balance supply and demand in real time               │
  │                                                            │
  │  🧬 Drug Discovery / Molecular Design                     │
  │     → Generate molecules with desired properties           │
  │                                                            │
  │  📡 Network Routing                                        │
  │     → Optimise packet routing in real time                 │
  │                                                            │
  │  🎨 Chip Design (Google TPU)                              │
  │     → RL places chip components on a floorplan             │
  └────────────────────────────────────────────────────────────┘
```

---

## 10. Comprehensive Application Table

| Domain                | Problem                    | State                         | Action                    | Reward                        | Algorithm         |
|-----------------------|----------------------------|-------------------------------|---------------------------|-------------------------------|-------------------|
| **Games (Atari)**     | Play from pixels           | Raw pixels (84×84×4)          | Discrete (joystick)       | Score change                  | DQN               |
| **Games (Go)**        | Board game                 | Board position                | Stone placement           | +1 win / -1 loss              | AlphaZero (MCTS)  |
| **Games (StarCraft)** | Real-time strategy         | Spatial + unit features       | Structured actions        | Win / loss                    | IMPALA + PBT      |
| **Robotics**          | Object grasping            | Joint angles + camera         | Gripper pose              | +1 grasp success              | SAC + HER         |
| **Robotics**          | Bipedal walking            | Joint angles + IMU            | Joint torques             | Forward velocity              | PPO               |
| **Autonomous Driving**| Lane following             | Camera, LIDAR, speed          | Steering, throttle        | Progress − collision penalty  | PPO, SAC          |
| **Recommendation**    | Content ranking            | User history                  | Item to show              | Click / engagement            | REINFORCE         |
| **Finance**           | Portfolio optimisation     | Returns, indicators           | Portfolio weights          | Risk-adjusted return          | PPO, A2C          |
| **Healthcare**        | Sepsis treatment           | Patient vitals                | Drug dosage               | Patient survival              | Off-policy RL     |
| **LLM Alignment**     | RLHF                      | Token sequence                | Next token                | Human preference score        | PPO               |
| **Data Centre**       | Cooling optimisation       | Temperature sensors           | Cooling parameters        | −Energy consumption           | Deep RL           |
| **Chip Design**       | Floorplanning              | Partial placement             | Component position        | Wire length + congestion      | Policy Gradient   |

---

## 11. Simple Game-Playing Code Example

A complete example: training a Q-learning agent to play **FrozenLake**.

```python
import gymnasium as gym
import numpy as np

# === Environment Setup ===
env = gym.make("FrozenLake-v1", is_slippery=False, map_name="4x4")
n_states  = env.observation_space.n   # 16
n_actions = env.action_space.n        # 4

# === Q-Table Initialisation ===
Q = np.zeros((n_states, n_actions))

# === Hyperparameters ===
alpha   = 0.8    # learning rate
gamma   = 0.95   # discount factor
epsilon = 1.0    # exploration rate
epsilon_decay = 0.995
epsilon_min   = 0.01
n_episodes    = 2000

# === Training Loop ===
rewards_per_episode = []

for episode in range(n_episodes):
    state, _ = env.reset()
    total_reward = 0
    done = False

    while not done:
        # Epsilon-greedy action selection
        if np.random.rand() < epsilon:
            action = env.action_space.sample()
        else:
            action = np.argmax(Q[state])

        next_state, reward, terminated, truncated, _ = env.step(action)
        done = terminated or truncated

        # Q-Learning update
        best_next = np.max(Q[next_state]) if not terminated else 0
        Q[state, action] += alpha * (reward + gamma * best_next - Q[state, action])

        state = next_state
        total_reward += reward

    # Decay exploration
    epsilon = max(epsilon_min, epsilon * epsilon_decay)
    rewards_per_episode.append(total_reward)

# === Results ===
avg_last_100 = np.mean(rewards_per_episode[-100:])
print(f"Average reward (last 100 episodes): {avg_last_100:.2f}")
print(f"Final epsilon: {epsilon:.4f}")

# === Display learned policy ===
action_symbols = ["←", "↓", "→", "↑"]
print("\nLearned Policy (4×4 grid):")
for row in range(4):
    row_str = "| "
    for col in range(4):
        state = row * 4 + col
        best_action = np.argmax(Q[state])
        row_str += f"{action_symbols[best_action]} | "
    print(row_str)
```

### Q-Learning Pseudocode

```
Algorithm: Q-Learning (Off-Policy TD Control)
──────────────────────────────────────────────────
Input : env, learning rate α, discount γ, exploration ε
Output: Q-table Q(s, a)

1.  Initialise Q(s, a) ← 0 for all s, a
2.  FOR each episode DO
3.      s ← env.reset()
4.      WHILE episode not done DO
5.          a ← ε-greedy(Q, s, ε)
6.          s', r, done ← env.step(a)
7.          Q(s,a) ← Q(s,a) + α [r + γ max_a' Q(s',a') − Q(s,a)]
8.          s ← s'
9.      END WHILE
10.     Decay ε
11. END FOR
```

---

## 12. Summary

```
╔════════════════════════════════════════════════════════════════════╗
║  CHAPTER 5 — KEY TAKEAWAYS                                       ║
╠════════════════════════════════════════════════════════════════════╣
║  1. RL excels when the problem involves sequential decisions     ║
║  2. Game-playing AIs (DQN, AlphaGo) were early breakthroughs    ║
║  3. Robotics uses sim-to-real transfer with domain randomisation ║
║  4. RLHF aligns LLMs with human preferences using PPO           ║
║  5. Finance, healthcare, and industry all benefit from RL        ║
║  6. Key challenge: safety, sample efficiency, and reward design  ║
║  7. The field is rapidly expanding into new domains              ║
╚════════════════════════════════════════════════════════════════════╝
```

| Domain         | Key Technique          | Challenge                        |
|----------------|------------------------|----------------------------------|
| Games          | DQN, MCTS + RL         | Compute cost                     |
| Robotics       | PPO + Sim-to-Real      | Reality gap                      |
| Driving        | PPO, Imitation + RL    | Safety guarantees                |
| Finance        | Policy Gradient        | Non-stationarity                 |
| Healthcare     | Off-policy RL          | Cannot experiment freely         |
| LLMs (RLHF)   | PPO + Reward Model     | Reward hacking, alignment tax    |
| Industry       | Deep RL                | Deployment risk, interpretability|

---

## 13. Revision Questions

1. **Name three key innovations** in DQN that enabled it to play Atari games from raw
   pixels.

2. **How does AlphaZero differ from the original AlphaGo?** Why is self-play sufficient
   for learning?

3. **What is sim-to-real transfer?** Explain domain randomisation and why it helps.

4. **Describe the three steps of the RLHF pipeline.** What role does the KL penalty
   play in Step 3?

5. **Why is RL particularly suited for recommendation systems** compared to supervised
   learning? What does RL optimise that SL cannot?

6. **Choose one healthcare application of RL.** Define the state, action, and reward.
   What ethical concerns arise?

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [Episodes and Trajectories](04-episodes-trajectories.md) | [Unit 1 Index](../README.md) | [Markov Decision Processes →](../../02-Markov-Decision-Processes/01-markov-property.md) |

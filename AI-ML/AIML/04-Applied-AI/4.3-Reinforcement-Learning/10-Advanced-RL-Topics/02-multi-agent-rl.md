# Multi-Agent Reinforcement Learning (MARL)

## Overview

**Multi-Agent RL** extends RL to settings with multiple interacting agents, each learning its own policy. Agents may cooperate, compete, or do both. MARL introduces challenges like non-stationarity (each agent's optimal policy depends on others' changing policies), credit assignment (who contributed to team success?), and communication (how to coordinate?).

---

## Problem Settings

```
1. COOPERATIVE: All agents share a common goal
   Example: Robot team assembling a product
   Reward: shared (team reward)
   Challenge: credit assignment

2. COMPETITIVE: Zero-sum, one wins, others lose
   Example: Chess, Go, poker
   Reward: r_agent = -r_opponent
   Challenge: opponent modeling

3. MIXED (General-sum): Complex interactions
   Example: Traffic, economics, social dilemmas
   Reward: independent objectives
   Challenge: equilibrium selection

  ┌──────────────────────────────────────────┐
  │  Cooperative    Competitive     Mixed     │
  │   🤝  🤝         ⚔️  ⚔️        🤝⚔️     │
  │  Team game      Zero-sum      Real world │
  │  Shared R      R₁ = -R₂     Independent │
  └──────────────────────────────────────────┘
```

---

## Key Challenges

```
1. NON-STATIONARITY:
   Each agent is learning → environment changes
   Agent A's optimal policy depends on Agent B's policy
   But B is also learning → moving target!
   
   Single-agent: env is fixed (Markov)
   Multi-agent:  env includes other agents (non-stationary)

2. SCALABILITY:
   Joint action space grows exponentially
   2 agents, 10 actions each → 100 joint actions
   10 agents, 10 actions each → 10^10 joint actions!

3. CREDIT ASSIGNMENT:
   Team gets reward +1... who contributed?
   Agent 1 did the right thing
   Agent 2 did nothing
   Agent 3 interfered
   All get the same team reward!

4. PARTIAL OBSERVABILITY:
   Each agent sees only its local observation
   Can't see other agents' states or intentions
   Need to communicate or infer
```

---

## Learning Paradigms

```
1. INDEPENDENT LEARNERS (IL):
   Each agent treats others as part of environment
   Each runs standard RL (DQN, PPO, etc.)
   
   Pros: Simple, scalable
   Cons: Non-stationary (others are learning too)
         No convergence guarantees

2. CENTRALIZED TRAINING, DECENTRALIZED EXECUTION (CTDE):
   Training: access to all agents' data (centralized critic)
   Execution: each agent only uses its own observation
   
   ┌────────────────────────────────────────┐
   │  Training:                              │
   │   All observations → Central Critic     │
   │   Individual obs → Each Agent's Actor   │
   │                                          │
   │  Execution:                             │
   │   Agent 1: obs₁ → π₁(a₁|o₁)           │
   │   Agent 2: obs₂ → π₂(a₂|o₂)           │
   │   No communication needed!              │
   └────────────────────────────────────────┘

3. FULLY CENTRALIZED:
   One "meta-agent" controls all agents
   Joint action space → computationally infeasible for many agents
   
4. COMMUNICATION LEARNING:
   Agents learn to send messages to each other
   CommNet, TarMAC, etc.
```

---

## Key Algorithms

```
1. INDEPENDENT PPO / DQN:
   Each agent runs its own PPO/DQN
   Simplest approach, often works surprisingly well
   Buffer: each agent stores own transitions

2. MADDPG (Multi-Agent DDPG):
   CTDE framework
   Centralized critic: Q_i(s, a₁, ..., aₙ)  ← sees all actions
   Decentralized actor: π_i(a_i|o_i)          ← sees own obs
   Each agent has own critic with global info

3. QMIX:
   For cooperative tasks
   Each agent has local Q_i(o_i, a_i)
   Global Q = mixing_network(Q₁, Q₂, ..., Qₙ)
   Constraint: ∂Q_tot/∂Q_i ≥ 0 (monotonic)
   → argmax of Q_tot = argmax of each Q_i independently!

4. MAPPO (Multi-Agent PPO):
   Shared PPO with centralized value function
   V(s) sees global state, π(a|o) sees local obs
   Simple, strong baseline for cooperative tasks
```

---

## Self-Play

```
For competitive games:

  Agent plays against copies of itself
  As it improves, opponent improves → curriculum!
  
  AlphaGo/AlphaZero: self-play from scratch
    Random → learns basic moves
    → learns tactics → learns strategy
    → superhuman!

  Population-based training:
    Maintain diverse pool of policies
    Each agent trains against pool members
    Prevents overfitting to single opponent
    
  ┌──────────────────────────────────────┐
  │  Self-play training loop:             │
  │                                       │
  │  π₀ (random)                          │
  │    ↓ play against self               │
  │  π₁ (basic)                           │
  │    ↓ play against π₀, π₁             │
  │  π₂ (intermediate)                    │
  │    ↓ play against π₀, π₁, π₂         │
  │  π₃ (advanced)                        │
  │    ↓ ...                             │
  │  π_n (superhuman)                     │
  └──────────────────────────────────────┘
```

---

## Python: Independent PPO for Multi-Agent

```python
import torch
import torch.nn as nn

class MAAgent(nn.Module):
    """Single agent in multi-agent setting."""
    def __init__(self, obs_dim, act_dim, global_dim=None):
        super().__init__()
        # Decentralized actor (local observation)
        self.actor = nn.Sequential(
            nn.Linear(obs_dim, 128), nn.ReLU(),
            nn.Linear(128, 64), nn.ReLU(),
            nn.Linear(64, act_dim))
        
        # Centralized critic (global state)
        critic_input = global_dim if global_dim else obs_dim
        self.critic = nn.Sequential(
            nn.Linear(critic_input, 128), nn.ReLU(),
            nn.Linear(128, 64), nn.ReLU(),
            nn.Linear(64, 1))
    
    def get_action(self, obs):
        logits = self.actor(obs)
        dist = torch.distributions.Categorical(logits=logits)
        action = dist.sample()
        return action, dist.log_prob(action)
    
    def get_value(self, global_state):
        return self.critic(global_state).squeeze(-1)

class MAPPO:
    """Multi-Agent PPO with shared reward."""
    def __init__(self, n_agents, obs_dim, act_dim, global_dim):
        self.agents = [MAAgent(obs_dim, act_dim, global_dim)
                       for _ in range(n_agents)]
        self.optimizers = [
            torch.optim.Adam(agent.parameters(), lr=3e-4)
            for agent in self.agents]
        self.n_agents = n_agents
    
    def act(self, observations):
        """Each agent selects action from its own observation."""
        actions, log_probs = [], []
        for i, obs in enumerate(observations):
            a, lp = self.agents[i].get_action(torch.FloatTensor(obs))
            actions.append(a.item())
            log_probs.append(lp)
        return actions, log_probs
    
    def update(self, agent_idx, states, actions, advantages, returns,
               old_log_probs, global_states, clip_eps=0.2):
        agent = self.agents[agent_idx]
        opt = self.optimizers[agent_idx]
        
        # PPO clipped update
        logits = agent.actor(states)
        dist = torch.distributions.Categorical(logits=logits)
        new_log_probs = dist.log_prob(actions)
        
        ratio = (new_log_probs - old_log_probs).exp()
        surr1 = ratio * advantages
        surr2 = torch.clamp(ratio, 1 - clip_eps, 1 + clip_eps) * advantages
        
        policy_loss = -torch.min(surr1, surr2).mean()
        value_loss = (agent.get_value(global_states) - returns).pow(2).mean()
        entropy = dist.entropy().mean()
        
        loss = policy_loss + 0.5 * value_loss - 0.01 * entropy
        opt.zero_grad()
        loss.backward()
        opt.step()
```

---

## Applications

```
  Application              Type          Example
  ───────────              ────          ───────
  Game AI                  Competitive   OpenAI Five (Dota 2)
  Autonomous vehicles      Mixed         Traffic coordination
  Robotics                 Cooperative   Warehouse robots
  Network routing          Cooperative   Packet scheduling
  Trading                  Competitive   Market making
  Swarm control            Cooperative   Drone swarms
  Social simulation        Mixed         Economic modeling
```

---

## Revision Questions

1. **What makes multi-agent RL harder than single-agent RL?**
2. **What is CTDE and why is it useful?**
3. **How does QMIX decompose team value into individual values?**
4. **How does self-play create a natural curriculum?**
5. **What is the credit assignment problem in cooperative MARL?**

---

[Previous: 01-model-based-rl.md](01-model-based-rl.md) | [Next: 03-hierarchical-rl.md](03-hierarchical-rl.md)

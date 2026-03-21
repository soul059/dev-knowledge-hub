# 4.3 Reinforcement Learning

## Course Overview

Reinforcement Learning (RL) is the branch of machine learning where agents learn optimal behavior through interaction with an environment, receiving rewards as feedback. This subject covers the full RL landscape — from foundational concepts (MDPs, Bellman equations) through tabular methods (DP, MC, TD) to modern deep RL algorithms (DQN, PPO, SAC) and cutting-edge topics like RLHF.

---

## Table of Contents

### Unit 1: Introduction to RL
- [What is Reinforcement Learning?](01-Introduction-to-RL/01-what-is-rl.md)
- [RL vs Supervised vs Unsupervised](01-Introduction-to-RL/02-rl-vs-sl-vs-ul.md)
- [Agent, Environment, State, Action, Reward](01-Introduction-to-RL/03-agent-environment.md)
- [Episodes and Trajectories](01-Introduction-to-RL/04-episodes-trajectories.md)
- [RL Applications](01-Introduction-to-RL/05-rl-applications.md)

### Unit 2: Markov Decision Processes (MDP)
- [Markov Property](02-Markov-Decision-Processes/01-markov-property.md)
- [MDP Components](02-Markov-Decision-Processes/02-mdp-components.md)
- [State Transitions](02-Markov-Decision-Processes/03-state-transitions.md)
- [Reward Function](02-Markov-Decision-Processes/04-reward-function.md)
- [Policies](02-Markov-Decision-Processes/05-policies.md)
- [Value Functions](02-Markov-Decision-Processes/06-value-functions.md)
- [Bellman Equations](02-Markov-Decision-Processes/07-bellman-equations.md)

### Unit 3: Dynamic Programming
- [Policy Evaluation](03-Dynamic-Programming/01-policy-evaluation.md)
- [Policy Iteration](03-Dynamic-Programming/02-policy-iteration.md)
- [Value Iteration](03-Dynamic-Programming/03-value-iteration.md)
- [Limitations of DP](03-Dynamic-Programming/04-limitations-of-dp.md)

### Unit 4: Monte Carlo Methods
- [Monte Carlo Prediction](04-Monte-Carlo-Methods/01-mc-prediction.md)
- [MC for Control](04-Monte-Carlo-Methods/02-mc-control.md)
- [On-Policy vs Off-Policy](04-Monte-Carlo-Methods/03-on-policy-vs-off-policy.md)
- [Importance Sampling](04-Monte-Carlo-Methods/04-importance-sampling.md)
- [Every-Visit vs First-Visit](04-Monte-Carlo-Methods/05-every-visit-vs-first-visit.md)

### Unit 5: Temporal Difference Learning
- [TD(0) Prediction](05-Temporal-Difference-Learning/01-td0-prediction.md)
- [SARSA](05-Temporal-Difference-Learning/02-sarsa.md)
- [Q-Learning](05-Temporal-Difference-Learning/03-q-learning.md)
- [Expected SARSA](05-Temporal-Difference-Learning/04-expected-sarsa.md)
- [TD vs MC vs DP](05-Temporal-Difference-Learning/05-td-vs-mc-vs-dp.md)
- [Eligibility Traces TD(λ)](05-Temporal-Difference-Learning/06-eligibility-traces.md)

### Unit 6: Function Approximation
- [Why Function Approximation?](06-Function-Approximation/01-why-function-approximation.md)
- [Linear Function Approximation](06-Function-Approximation/02-linear-function-approximation.md)
- [Neural Network Approximation](06-Function-Approximation/03-neural-network-approximation.md)
- [Gradient Descent Methods](06-Function-Approximation/04-gradient-descent-methods.md)
- [Semi-Gradient Methods](06-Function-Approximation/05-semi-gradient-methods.md)

### Unit 7: Deep Q-Networks (DQN)
- [DQN Architecture](07-Deep-Q-Networks/01-dqn-architecture.md)
- [Experience Replay](07-Deep-Q-Networks/02-experience-replay.md)
- [Target Networks](07-Deep-Q-Networks/03-target-networks.md)
- [Double DQN](07-Deep-Q-Networks/04-double-dqn.md)
- [Dueling DQN](07-Deep-Q-Networks/05-dueling-dqn.md)
- [Prioritized Experience Replay](07-Deep-Q-Networks/06-prioritized-experience-replay.md)

### Unit 8: Policy Gradient Methods
- [Policy Gradient Theorem](08-Policy-Gradient-Methods/01-policy-gradient-theorem.md)
- [REINFORCE Algorithm](08-Policy-Gradient-Methods/02-reinforce.md)
- [Baseline and Variance Reduction](08-Policy-Gradient-Methods/03-baseline-variance-reduction.md)
- [Actor-Critic Methods](08-Policy-Gradient-Methods/04-actor-critic.md)
- [A2C/A3C](08-Policy-Gradient-Methods/05-a2c-a3c.md)
- [Advantage Estimation (GAE)](08-Policy-Gradient-Methods/06-gae.md)

### Unit 9: Advanced Policy Gradient
- [Trust Region Policy Optimization (TRPO)](09-Advanced-Policy-Gradient/01-trpo.md)
- [Proximal Policy Optimization (PPO)](09-Advanced-Policy-Gradient/02-ppo.md)
- [Soft Actor-Critic (SAC)](09-Advanced-Policy-Gradient/03-sac.md)
- [Deterministic Policy Gradient (DDPG)](09-Advanced-Policy-Gradient/04-ddpg.md)
- [TD3](09-Advanced-Policy-Gradient/05-td3.md)

### Unit 10: Advanced RL Topics
- [Model-Based RL](10-Advanced-RL-Topics/01-model-based-rl.md)
- [Multi-Agent RL](10-Advanced-RL-Topics/02-multi-agent-rl.md)
- [Hierarchical RL](10-Advanced-RL-Topics/03-hierarchical-rl.md)
- [Inverse RL](10-Advanced-RL-Topics/04-inverse-rl.md)
- [Offline RL](10-Advanced-RL-Topics/05-offline-rl.md)
- [RL from Human Feedback (RLHF)](10-Advanced-RL-Topics/06-rlhf.md)
- [Exploration Strategies](10-Advanced-RL-Topics/07-exploration-strategies.md)

---

## Key Resources

| Resource | Type | Coverage |
|----------|------|----------|
| Sutton & Barto (2018) | Textbook | Definitive RL textbook |
| David Silver's RL Course | Video | Theory + intuition |
| OpenAI Spinning Up | Tutorial | Deep RL implementation |
| Gymnasium (Gym) | Library | RL environments |
| Stable Baselines3 | Library | RL algorithm implementations |

---

## Prerequisites

- Linear algebra (vectors, matrices)
- Probability and statistics
- Python programming
- Basic neural networks (for deep RL units)

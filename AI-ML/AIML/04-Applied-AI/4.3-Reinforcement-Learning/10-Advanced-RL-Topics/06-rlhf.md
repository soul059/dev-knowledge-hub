# Reinforcement Learning from Human Feedback (RLHF)

## Overview

**RLHF** trains AI systems to align with human preferences by using human feedback as a reward signal. Instead of defining a mathematical reward function, humans rank model outputs, and a reward model is trained on these rankings. This reward model then guides RL training. RLHF is the key technique behind ChatGPT, Claude, and other aligned language models.

---

## Why RLHF?

```
Problem: How do you define "helpful, harmless, honest" as a reward?

  Traditional RL:  R = score (game points, distance, etc.)
  Language:        R = ??? (what makes a "good" response?)

  Reward hacking examples without RLHF:
    Objective: "Generate helpful text"
    → Model: repeats "I'll help!" 1000 times (high reward!)
    
    Objective: "Maximize engagement"
    → Model: generates controversial/clickbait content
    
    Objective: "Answer questions correctly"
    → Model: always says "I don't know" (never wrong!)

  Solution: Let HUMANS judge quality → learn reward from humans

  ┌──────────────────────────────────────────────────┐
  │ Human preferences capture nuances that are       │
  │ impossible to specify in a mathematical formula: │
  │                                                    │
  │ • Helpfulness (actually answers the question)    │
  │ • Truthfulness (factually correct)               │
  │ • Safety (no harmful content)                    │
  │ • Style (appropriate tone and format)            │
  │ • Nuance (handles edge cases well)               │
  └──────────────────────────────────────────────────┘
```

---

## RLHF Pipeline

```
Three stages:

  Stage 1: Supervised Fine-Tuning (SFT)
  ┌──────────────────────────────────────┐
  │  Pre-trained LLM                      │
  │       ↓ fine-tune on                 │
  │  Human-written demonstrations        │
  │       ↓                              │
  │  SFT Model (π_SFT)                  │
  └──────────────────────────────────────┘

  Stage 2: Reward Model Training
  ┌──────────────────────────────────────┐
  │  For each prompt x:                   │
  │    Generate pairs: (y₁, y₂) from π   │
  │    Human rates: y₁ > y₂ or y₂ > y₁  │
  │                                       │
  │  Train reward model R(x, y):         │
  │    P(y₁ > y₂) = σ(R(x,y₁) - R(x,y₂))│
  │    (Bradley-Terry model)              │
  │                                       │
  │  Loss: -E[log σ(R(x,y_w) - R(x,y_l))]│
  │    y_w = human-preferred (winner)     │
  │    y_l = human-rejected (loser)       │
  └──────────────────────────────────────┘

  Stage 3: RL Fine-Tuning (PPO)
  ┌──────────────────────────────────────┐
  │  Policy: π_θ (initialized from SFT)  │
  │                                       │
  │  For each prompt x:                   │
  │    y ~ π_θ(·|x)  (generate response)│
  │    r = R(x, y)    (reward model score)│
  │    KL penalty: -β KL(π_θ || π_SFT)  │
  │                                       │
  │  Total reward:                        │
  │    r_total = R(x,y) - β KL(π_θ||π_ref)│
  │                                       │
  │  Update π_θ with PPO                 │
  └──────────────────────────────────────┘
```

---

## Reward Model

```
Architecture: Same as LLM but with scalar output head

  Input: prompt x + response y
  Output: scalar reward R(x, y)

  Training data: human comparisons
    (x, y_win, y_lose) × 100K+ pairs

  Loss (Bradley-Terry):
    L = -log σ(R(x, y_w) - R(x, y_l))

  This learns: "response A is better than B"
  Not: "response A gets score 7.3"
  
  Comparisons are MUCH easier for humans than absolute scores!

  ┌────────────────────────────────────────┐
  │ Prompt: "What is the capital of France?"│
  │                                         │
  │ Response A: "Paris is the capital."    │
  │ Response B: "France is in Europe."     │
  │                                         │
  │ Human: A > B                           │
  │                                         │
  │ R(x, A) > R(x, B) after training      │
  └────────────────────────────────────────┘
```

---

## KL Penalty

```
Why constrain π_θ to stay near π_SFT?

  Without KL penalty:
    Policy finds reward hacking exploits
    Generates text that "fools" reward model
    But is gibberish or adversarial to humans
    
  KL(π_θ || π_ref):
    Measures how far policy has drifted from reference
    β controls constraint strength
    
    High β: stay very close to SFT (conservative)
    Low β:  allow more deviation (risk reward hacking)

  Reward with KL:
    r = R(x, y) - β × [log π_θ(y|x) - log π_ref(y|x)]
    
    Per-token KL: Σ_t [log π_θ(y_t|x,y_{<t}) - log π_ref(y_t|x,y_{<t})]

  Think of it as: "be good (high R) but don't be weird (low KL)"
```

---

## Direct Preference Optimization (DPO)

```
DPO simplifies RLHF by eliminating the reward model AND PPO!

  Key insight: the optimal policy under RLHF has a closed form:
    π*(y|x) ∝ π_ref(y|x) exp(R(x,y)/β)
    
  Rearranging:
    R(x,y) = β log(π*(y|x) / π_ref(y|x)) + const.

  Substitute into Bradley-Terry loss:
    L_DPO = -E[log σ(β log(π_θ(y_w|x)/π_ref(y_w|x))
                     - β log(π_θ(y_l|x)/π_ref(y_l|x)))]

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  RLHF: Train reward model → Train policy with PPO│
  │        (complex, unstable, expensive)              │
  │                                                    │
  │  DPO:  Directly optimize policy on preferences    │
  │        (simple supervised loss!)                   │
  │                                                    │
  │  DPO = "the reward model is implicit in the       │
  │         policy itself"                             │
  │                                                    │
  │  Same theoretical optimum, much simpler training! │
  └──────────────────────────────────────────────────┘

  Intuition of DPO gradient:
    ↑ probability of preferred response
    ↓ probability of rejected response
    Weighted by how "surprising" the preferences are
```

---

## Python: RLHF Components

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class RewardModel(nn.Module):
    """Simplified reward model (normally uses full LLM)."""
    def __init__(self, hidden_dim):
        super().__init__()
        self.backbone = nn.Sequential(
            nn.Linear(hidden_dim, 256), nn.ReLU(),
            nn.Linear(256, 128), nn.ReLU())
        self.head = nn.Linear(128, 1)
    
    def forward(self, hidden_states):
        # hidden_states: last hidden state of LLM
        return self.head(self.backbone(hidden_states)).squeeze(-1)

def reward_model_loss(reward_model, chosen_hidden, rejected_hidden):
    """Bradley-Terry pairwise loss."""
    r_chosen = reward_model(chosen_hidden)
    r_rejected = reward_model(rejected_hidden)
    return -F.logsigmoid(r_chosen - r_rejected).mean()


def dpo_loss(policy_logps_chosen, policy_logps_rejected,
             ref_logps_chosen, ref_logps_rejected, beta=0.1):
    """Direct Preference Optimization loss."""
    chosen_rewards = beta * (policy_logps_chosen - ref_logps_chosen)
    rejected_rewards = beta * (policy_logps_rejected - ref_logps_rejected)
    
    return -F.logsigmoid(chosen_rewards - rejected_rewards).mean()


def compute_rlhf_reward(reward_model, policy, ref_policy,
                         prompt, response, beta=0.01):
    """Total RLHF reward = R(x,y) - β × KL."""
    # Reward model score
    rm_score = reward_model(prompt, response)
    
    # Per-token KL divergence
    policy_logp = policy.log_prob(response, given=prompt)
    ref_logp = ref_policy.log_prob(response, given=prompt)
    kl = policy_logp - ref_logp  # per-token
    
    # Total reward
    reward = rm_score - beta * kl.sum()
    return reward
```

---

## RLHF in Practice

```
  Component          Details
  ─────────          ───────
  SFT data           10K-100K high-quality demonstrations
  Comparison data    100K-1M+ human preference pairs
  Reward model       Same arch as LLM, scalar output head
  RL algorithm       PPO (most common) or DPO (simpler)
  KL coefficient     β = 0.01-0.2 (tuned carefully)
  Training           Iterative (collect → label → train → repeat)
  
  Scaling challenges:
  • Expensive: human labelers cost $15-30/hour
  • Quality: labeler disagreement, biases
  • Coverage: can't label all possible outputs
  • Gaming: model may exploit reward model weaknesses
  
  Alternatives to RLHF:
  • DPO: no separate reward model needed
  • RLAIF: AI provides feedback instead of humans
  • Constitutional AI: AI self-critiques against principles
  • Best-of-N: generate N, pick best by reward model (no RL)
```

---

## Revision Questions

1. **Why can't traditional reward functions capture what makes a good LLM response?**
2. **What are the three stages of the RLHF pipeline?**
3. **How is the reward model trained from human comparisons (Bradley-Terry)?**
4. **Why is the KL penalty necessary during PPO fine-tuning?**
5. **How does DPO simplify RLHF and what is the key mathematical insight?**

---

[Previous: 05-offline-rl.md](05-offline-rl.md) | [Next: 07-exploration-strategies.md](07-exploration-strategies.md)

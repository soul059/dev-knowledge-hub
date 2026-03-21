# Prioritized Experience Replay (PER)

## Overview

**Prioritized Experience Replay** (Schaul et al., 2016) replaces uniform random sampling from the replay buffer with **priority-based sampling**. Transitions with higher TD error (more "surprising") are sampled more frequently, enabling faster and more efficient learning.

---

## Motivation

```
Uniform replay: Every transition equally likely to be sampled

  ┌────────────────────────────────────────┐
  │  Buffer: 1M transitions                │
  │                                        │
  │  95% have small TD error (well-learned)│
  │   5% have large TD error (informative) │
  │                                        │
  │  Uniform: Sample 32 → expect ~1.6      │
  │  informative transitions per batch     │
  │                                        │
  │  Wasteful! Most samples teach little.  │
  └────────────────────────────────────────┘

PER: Sample informative transitions more often

  Priority = |δ| = |TD error|
  
  Large |δ|: Prediction was far off → lots to learn
  Small |δ|: Prediction was close → already learned
  
  Higher priority → sampled more frequently
```

---

## Priority Schemes

```
1. Proportional prioritization:
   P(i) = p_i^α / Σ_k p_k^α
   
   p_i = |δ_i| + ε     (small ε > 0 prevents zero priority)
   α = exponent (0 = uniform, 1 = fully proportional)
   
   α = 0: uniform sampling (no prioritization)
   α = 1: strictly proportional to |TD error|
   Typical: α = 0.6

2. Rank-based prioritization:
   Sort transitions by |δ|, assign rank
   P(i) = (1/rank(i))^α / Σ_k (1/rank(k))^α
   
   More robust to outliers than proportional
   But: requires maintaining sorted order

  Priority distribution:
  P(i)
   ↑
   │ ██
   │ ██ █
   │ ██ ██ █
   │ ██ ██ ██ █ █
   │ ██ ██ ██ ██ ██ █ █ █ █ █
   └──────────────────────────→ transitions (sorted by |δ|)
     high error        low error
```

---

## Importance Sampling Correction

```
Problem: Prioritized sampling introduces BIAS
  We're no longer sampling uniformly from the data distribution
  → Gradient estimates are biased
  → Can cause convergence issues

Solution: Importance sampling weights

  w_i = (1 / (N × P(i)))^β
  Normalize: w_i ← w_i / max_j(w_j)
  
  β starts at β₀ (e.g., 0.4) and anneals to 1.0 over training
  
  β = 0: no correction (pure prioritized)
  β = 1: full correction (unbiased)
  
  Anneal β → 1 because:
    Early training: bias is okay (learning fast matters)
    Late training: need unbiased for convergence

  Apply weights to loss:
    Loss = w_i × (y_i - Q(s_i, a_i; θ))²
    
  High-priority (frequently sampled) → lower weight
  Low-priority (rarely sampled) → higher weight
  → Corrects for non-uniform sampling
```

---

## Data Structure: Sum Tree

```
Efficient sampling with O(log N) complexity:

  Sum Tree: binary tree where each leaf stores a priority
  Parent = sum of children

  Example (8 transitions):
  
                    [42]                    ← total priority
                 /        \
              [29]        [13]
             /    \      /    \
          [12]   [17]  [5]   [8]
          / \    / \   / \   / \
         3  9  12  5  2  3  1  7        ← leaf priorities
         
  To sample: generate random number r ∈ [0, 42)
  Traverse tree: go left if r < left child, else subtract and go right
  
  r = 25: 25 < 29 → left, 25 > 12 → right (r=13), 13 > 12 → right (r=1)
  → Sample transition with priority 5 (4th leaf)
  
  O(log N) per sample (vs O(N) for linear scan)
  O(log N) for priority update
```

---

## Algorithm

```
Initialize replay buffer D with sum tree, β₀, α

For each step:
  1. Interact: take action, get (s, a, r, s', done)
  
  2. Store with max priority:
     p_new = max(existing priorities)  (ensures new transitions get sampled)
     Insert (s, a, r, s', done) into D with priority p_new
  
  3. Sample: Draw minibatch of B transitions using priorities
     P(i) = p_i^α / Σ p_k^α
  
  4. Compute IS weights:
     w_i = (N × P(i))^(-β) / max_j(w_j)
     β ← β₀ + (1 - β₀) × (step / total_steps)    (anneal to 1)
  
  5. Compute TD errors:
     δ_i = r_i + γ max Q(s'_i, ·; θ⁻) - Q(s_i, a_i; θ)
  
  6. Update priorities:
     p_i ← |δ_i| + ε
  
  7. Update network:
     Loss = (1/B) Σ w_i × δ_i²
     Gradient descent on θ
```

---

## Python Implementation

```python
import numpy as np

class SumTree:
    def __init__(self, capacity):
        self.capacity = capacity
        self.tree = np.zeros(2 * capacity - 1)
        self.data = [None] * capacity
        self.write = 0
        self.size = 0
    
    def _propagate(self, idx, change):
        parent = (idx - 1) // 2
        self.tree[parent] += change
        if parent != 0:
            self._propagate(parent, change)
    
    def update(self, idx, priority):
        change = priority - self.tree[idx]
        self.tree[idx] = priority
        self._propagate(idx, change)
    
    def add(self, priority, data):
        idx = self.write + self.capacity - 1
        self.data[self.write] = data
        self.update(idx, priority)
        self.write = (self.write + 1) % self.capacity
        self.size = min(self.size + 1, self.capacity)
    
    def get(self, s):
        idx = 0
        while idx < self.capacity - 1:
            left = 2 * idx + 1
            if s <= self.tree[left]:
                idx = left
            else:
                s -= self.tree[left]
                idx = left + 1
        data_idx = idx - self.capacity + 1
        return idx, self.tree[idx], self.data[data_idx]
    
    @property
    def total(self):
        return self.tree[0]

class PrioritizedReplayBuffer:
    def __init__(self, capacity, alpha=0.6, beta=0.4):
        self.tree = SumTree(capacity)
        self.alpha = alpha
        self.beta = beta
        self.epsilon = 1e-6
        self.max_priority = 1.0
    
    def push(self, transition):
        self.tree.add(self.max_priority ** self.alpha, transition)
    
    def sample(self, batch_size):
        indices, priorities, batch, weights = [], [], [], []
        segment = self.tree.total / batch_size
        
        for i in range(batch_size):
            s = np.random.uniform(segment * i, segment * (i + 1))
            idx, priority, data = self.tree.get(s)
            indices.append(idx)
            priorities.append(priority)
            batch.append(data)
        
        probs = np.array(priorities) / self.tree.total
        weights = (self.tree.size * probs) ** (-self.beta)
        weights /= weights.max()
        
        return batch, np.array(indices), np.array(weights)
    
    def update_priorities(self, indices, td_errors):
        for idx, td_error in zip(indices, td_errors):
            priority = (abs(td_error) + self.epsilon) ** self.alpha
            self.tree.update(idx, priority)
            self.max_priority = max(self.max_priority, priority)
```

---

## Revision Questions

1. **Why is uniform sampling from replay buffer inefficient?**
2. **How does TD error determine transition priority?**
3. **Why is importance sampling correction needed and how does β control it?**
4. **How does a Sum Tree enable efficient O(log N) prioritized sampling?**
5. **Why are new transitions added with maximum priority?**

---

[Previous: 05-dueling-dqn.md](05-dueling-dqn.md) | [Back to README](../README.md)

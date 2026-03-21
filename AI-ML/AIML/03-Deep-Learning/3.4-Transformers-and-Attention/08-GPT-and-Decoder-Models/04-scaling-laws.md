# Neural Scaling Laws

> **Navigation:** [← GPT-2 and GPT-3](03-gpt2-and-gpt3.md) | [In-Context Learning →](05-in-context-learning.md)

---

## Overview

**Neural scaling laws** describe the empirical observation that language model performance (measured by loss) improves predictably as a **power law** function of model size, dataset size, and compute budget. Kaplan et al. (2020) at OpenAI first characterized these relationships quantitatively, showing that loss decreases smoothly with scale across many orders of magnitude. Later, Hoffmann et al. (2022) at DeepMind refined these findings with the **Chinchilla** study, demonstrating that most large language models are significantly **over-parameterized and under-trained**, and proposed an optimal ratio of approximately 20 training tokens per parameter. These scaling laws have become essential tools for planning model training, allocating compute budgets, and predicting model performance before expensive training runs.

---

## 1. Power Law Relationships

Kaplan et al. (2020) discovered that test loss follows a **power law** with respect to three independent variables:

### 1.1 Loss vs. Model Size (Parameters N)

```
L(N) = (N_c / N)^α_N

where:
  L(N) = test loss
  N    = number of non-embedding parameters
  N_c  = a constant (≈ 8.8 × 10¹³)
  α_N  = 0.076
```

### 1.2 Loss vs. Dataset Size (Tokens D)

```
L(D) = (D_c / D)^α_D

where:
  D    = number of training tokens
  D_c  = a constant (≈ 5.4 × 10¹³)
  α_D  = 0.095
```

### 1.3 Loss vs. Compute (FLOPs C)

```
L(C) = (C_c / C)^α_C

where:
  C    = compute in FLOPs
  C_c  = a constant (≈ 3.1 × 10⁸)
  α_C  = 0.050
```

### Combined Scaling Law

When both model size and dataset size vary:

```
L(N, D) = [ (N_c / N)^(α_N / α_D) + (D_c / D) ]^α_D

This captures the irreducible loss when both N and D are finite.
```

---

## 2. ASCII Diagram: Scaling Curves

```
  Test Loss (L)
  ▲
  │
  3.5│●
     │ ●
  3.0│  ●
     │   ●
  2.5│    ●●
     │      ●●
  2.0│        ●●●
     │           ●●●●
  1.5│               ●●●●●●●
     │                      ●●●●●●●●●●●
  1.0│                                  ●●●●●●●●●●  ← diminishing returns
     │
  0.5│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  (irreducible loss)
     │
     └────────────────────────────────────────────► log(Scale)
      10⁶   10⁷   10⁸   10⁹   10¹⁰  10¹¹  10¹²
              (Parameters / Tokens / FLOPs)

  Key observations:
  • Loss decreases as a smooth power law
  • Relationship is linear on a log-log plot
  • There is an irreducible loss floor (entropy of language)
  • Returns diminish but never fully plateau in observed range
```

### Log-Log Plot (Linear Relationship)

```
  log(Loss)
  ▲
  │●
  │ ●
  │  ●
  │   ●
  │    ●
  │     ●                 slope = -α
  │      ●
  │       ●
  │        ●
  │         ●
  └──────────────────────► log(N)

  On a log-log plot, a power law L = (N_c/N)^α
  becomes a straight line with slope -α.
```

---

## 3. Worked Example: Predicting Loss

### Problem

Given a model with 1B parameters achieves a loss of 2.5, predict the loss for a 10B parameter model using the scaling exponent α_N = 0.076.

### Solution

```
Using L(N) = (N_c / N)^α_N:

L(1B) / L(10B) = (N_c/1B)^0.076 / (N_c/10B)^0.076
               = (10B/1B)^0.076
               = 10^0.076
               = 1.191

L(10B) = L(1B) / 1.191
       = 2.5 / 1.191
       = 2.10

Scaling from 1B to 10B (10× more parameters) reduces loss by ~16%.
```

### Table: Expected Loss Reduction with Scale

```
Scale Increase    Loss Multiplier (α=0.076)    Loss Reduction
    2×            2^(-0.076) = 0.949              ~5.1%
    5×            5^(-0.076) = 0.885              ~11.5%
   10×            10^(-0.076) = 0.840             ~16.0%
  100×            100^(-0.076) = 0.706            ~29.4%
 1000×            1000^(-0.076) = 0.593           ~40.7%
```

---

## 4. Chinchilla: Compute-Optimal Training

### 4.1 The Chinchilla Finding (Hoffmann et al., 2022)

The key insight from DeepMind's Chinchilla paper ("Training Compute-Optimal Large Language Models"):

> **For a given compute budget, most large language models are significantly over-parameterized and under-trained.**

### 4.2 Optimal Scaling Ratio

```
For compute-optimal training:

  D_opt ≈ 20 × N

where:
  D_opt = optimal number of training tokens
  N     = number of model parameters

Equivalently:
  N and D should be scaled equally with compute:
  N ∝ C^0.5    (parameters scale as square root of compute)
  D ∝ C^0.5    (tokens scale as square root of compute)
```

### 4.3 Chinchilla vs. Gopher: A Case Study

```
  Model          Parameters    Training Tokens    Compute       Loss
  ┌────────────┬────────────┬─────────────────┬───────────┬──────────┐
  │ Gopher     │   280B     │     300B         │   C       │   2.01   │
  │ Chinchilla │    70B     │    1400B (1.4T)  │   C       │   1.94   │
  └────────────┴────────────┴─────────────────┴───────────┴──────────┘

  Same compute budget, but Chinchilla:
  • Uses 4× FEWER parameters
  • Trains on ~4.7× MORE tokens
  • Achieves BETTER performance!

  Gopher ratio:   300B / 280B = 1.07 tokens/param  (under-trained!)
  Chinchilla ratio: 1.4T / 70B = 20 tokens/param   (compute-optimal!)
```

### 4.4 Visualization: Iso-FLOP Curves

```
  Loss
  ▲
  │                     Under-     Compute-    Over-
  │                     trained    optimal     trained
  │                        │          │          │
  │     ●                  │          │          │
  │      ●                 │          │          │
  │       ●                │          │          │
  │        ●               │          │          │
  │         ●              │     ★    │          │
  │          ●             │   (min)  │    ●     │
  │           ●            │          │     ●    │
  │            ●●●         │          │      ●●  │
  │                ●●●●    │          │          │
  │                        │          │          │
  └────────────────────────┼──────────┼──────────┼──► Model Size (N)
                           │ ← small  │  large → │
                           │  models  │  models  │

  For a fixed compute budget (iso-FLOP curve):
  • Too-small models: under-parameterized, can't learn enough
  • Too-large models: under-trained, not enough data to converge
  • Sweet spot ★: the compute-optimal model size
```

---

## 5. Implications for Model Design

### 5.1 Training Budget Planning

Given a compute budget C (in FLOPs), the optimal allocation is:

```
Step 1: Choose model size  N_opt = k₁ · C^0.5
Step 2: Choose token count D_opt = k₂ · C^0.5
Step 3: Verify ratio       D_opt / N_opt ≈ 20
```

### 5.2 Comparison of Scaling Philosophies

| Approach           | Philosophy                          | Example Models         |
|--------------------|-------------------------------------|------------------------|
| **Kaplan (2020)**  | Scale parameters faster than data   | GPT-3 (175B, 300B tok) |
| **Chinchilla**     | Scale parameters and data equally   | Chinchilla (70B, 1.4T) |
| **LLaMA (2023)**   | Over-train smaller models on more data | LLaMA-7B on 1T tokens |

### 5.3 Real Model Analysis

```
Model         Params    Tokens    Tokens/Param    Status
──────────────────────────────────────────────────────────
GPT-3 175B     175B      300B        1.7          Under-trained
Gopher 280B    280B      300B        1.1          Under-trained
Chinchilla     70B      1400B       20.0          Optimal
LLaMA 7B       7B       1000B      142.9          Over-trained (by design)
LLaMA 65B      65B      1400B       21.5          Near-optimal
Llama 2 70B    70B      2000B       28.6          Slightly over-trained
```

LLaMA intentionally over-trains because **inference cost** depends on model size, not training tokens — smaller models are cheaper to deploy.

---

## 6. The Compute-Loss Relationship

### 6.1 Estimating Compute (FLOPs)

A rough approximation for transformer training compute:

```
C ≈ 6 · N · D

where:
  C = total floating-point operations
  N = model parameters
  D = training tokens

The factor 6 comes from:
  • 2× for forward pass (multiply-accumulate)
  • 2× because gradients cost ~2× forward
  • Total: ~6 FLOPs per parameter per token
```

### 6.2 Worked Example

```
GPT-3 training compute:
  N = 175 × 10⁹ parameters
  D = 300 × 10⁹ tokens
  C ≈ 6 × 175 × 10⁹ × 300 × 10⁹
    = 315 × 10²¹
    = 3.15 × 10²³ FLOPs
    ≈ 3,640 petaflop-days

On A100 GPUs (312 TFLOPS each):
  Time ≈ 3.15 × 10²³ / (312 × 10¹² × 3600 × 24)
       ≈ 11,700 GPU-days
       ≈ 1,024 GPUs for ~11.4 days
```

---

## 7. Code: Plotting Scaling Law Curves

```python
import numpy as np
import matplotlib.pyplot as plt


def power_law_loss(scale, alpha, constant):
    """Compute loss using power law: L = (constant / scale)^alpha."""
    return (constant / scale) ** alpha


def plot_scaling_laws():
    """Plot scaling law curves for model size, data size, and compute."""
    fig, axes = plt.subplots(1, 3, figsize=(18, 5))

    # --- Loss vs Model Size ---
    N = np.logspace(6, 12, 100)        # 1M to 1T parameters
    alpha_N = 0.076
    N_c = 8.8e13
    L_N = power_law_loss(N, alpha_N, N_c)

    axes[0].loglog(N, L_N, 'b-', linewidth=2)
    axes[0].set_xlabel("Model Size (Parameters N)", fontsize=12)
    axes[0].set_ylabel("Test Loss L(N)", fontsize=12)
    axes[0].set_title("Loss vs. Model Size", fontsize=14)
    axes[0].grid(True, which="both", alpha=0.3)

    # Mark GPT models
    gpt_sizes = {"GPT-1\n117M": 117e6, "GPT-2\n1.5B": 1.5e9, "GPT-3\n175B": 175e9}
    for label, size in gpt_sizes.items():
        loss = power_law_loss(size, alpha_N, N_c)
        axes[0].plot(size, loss, 'ro', markersize=8)
        axes[0].annotate(label, (size, loss), textcoords="offset points",
                         xytext=(10, 10), fontsize=9)

    # --- Loss vs Dataset Size ---
    D = np.logspace(8, 13, 100)        # 100M to 10T tokens
    alpha_D = 0.095
    D_c = 5.4e13
    L_D = power_law_loss(D, alpha_D, D_c)

    axes[1].loglog(D, L_D, 'g-', linewidth=2)
    axes[1].set_xlabel("Dataset Size (Tokens D)", fontsize=12)
    axes[1].set_ylabel("Test Loss L(D)", fontsize=12)
    axes[1].set_title("Loss vs. Dataset Size", fontsize=14)
    axes[1].grid(True, which="both", alpha=0.3)

    # --- Loss vs Compute ---
    C = np.logspace(15, 25, 100)       # 10^15 to 10^25 FLOPs
    alpha_C = 0.050
    C_c = 3.1e8
    L_C = power_law_loss(C, alpha_C, C_c)

    axes[2].loglog(C, L_C, 'r-', linewidth=2)
    axes[2].set_xlabel("Compute (FLOPs C)", fontsize=12)
    axes[2].set_ylabel("Test Loss L(C)", fontsize=12)
    axes[2].set_title("Loss vs. Compute", fontsize=14)
    axes[2].grid(True, which="both", alpha=0.3)

    plt.tight_layout()
    plt.savefig("scaling_laws.png", dpi=150, bbox_inches="tight")
    plt.show()
    print("Plot saved to scaling_laws.png")


def chinchilla_optimal(compute_flops):
    """Given a compute budget in FLOPs, compute the optimal N and D."""
    # C ≈ 6 * N * D  and  D = 20 * N
    # C = 6 * N * 20N = 120 * N^2
    # N = sqrt(C / 120)
    N_opt = np.sqrt(compute_flops / 120)
    D_opt = 20 * N_opt
    return N_opt, D_opt


def plot_chinchilla_comparison():
    """Compare Chinchilla-optimal vs over-parameterized models."""
    compute_budgets = np.logspace(18, 24, 50)

    N_optimal = []
    D_optimal = []
    N_overparameterized = []
    D_underdata = []

    for C in compute_budgets:
        N_opt, D_opt = chinchilla_optimal(C)
        N_optimal.append(N_opt)
        D_optimal.append(D_opt)
        # Over-parameterized: 10× more params, 10× less data
        N_overparameterized.append(N_opt * 10)
        D_underdata.append(D_opt / 10)

    fig, ax = plt.subplots(figsize=(10, 6))
    ax.loglog(compute_budgets, N_optimal, 'b-', label="Chinchilla-optimal N", lw=2)
    ax.loglog(compute_budgets, D_optimal, 'g-', label="Chinchilla-optimal D", lw=2)
    ax.loglog(compute_budgets, N_overparameterized, 'r--',
              label="Over-parameterized N", lw=1.5)
    ax.loglog(compute_budgets, D_underdata, 'm--',
              label="Under-trained D", lw=1.5)

    ax.set_xlabel("Compute Budget (FLOPs)", fontsize=12)
    ax.set_ylabel("Scale", fontsize=12)
    ax.set_title("Chinchilla-Optimal vs Over-Parameterized Scaling", fontsize=14)
    ax.legend(fontsize=11)
    ax.grid(True, which="both", alpha=0.3)

    plt.tight_layout()
    plt.savefig("chinchilla_comparison.png", dpi=150, bbox_inches="tight")
    plt.show()


# --- Main ---
if __name__ == "__main__":
    plot_scaling_laws()

    # Compute Chinchilla-optimal for various budgets
    print("\nChinchilla-Optimal Model Configurations:")
    print(f"{'Compute (FLOPs)':<20} {'Params (N)':<15} {'Tokens (D)':<15} {'D/N Ratio':<10}")
    print("-" * 60)

    for exp in [18, 19, 20, 21, 22, 23, 24]:
        C = 10 ** exp
        N, D = chinchilla_optimal(C)
        print(f"10^{exp:<17} {N:>12.2e}   {D:>12.2e}   {D/N:>8.1f}")

    # Verify with known models
    print("\nModel Verification:")
    print(f"GPT-3:      N=175B, D=300B → D/N = {300/175:.1f} (under-trained)")
    print(f"Chinchilla: N=70B,  D=1.4T → D/N = {1400/70:.1f} (optimal)")
    print(f"LLaMA-7B:   N=7B,   D=1T   → D/N = {1000/7:.1f} (over-trained)")
```

---

## 8. Real-World Applications

| Application                     | How Scaling Laws Help                              |
|---------------------------------|----------------------------------------------------|
| **Training budget planning**    | Predict optimal model size for a given compute budget |
| **Performance forecasting**     | Estimate model quality before training              |
| **Hardware procurement**        | Determine GPU/TPU requirements                      |
| **Model architecture search**   | Decide layer count, hidden dim for target loss       |
| **Cost-benefit analysis**       | Quantify diminishing returns of scaling              |
| **Research prioritization**     | Identify whether to scale or improve algorithms      |

---

## 9. Summary Table

| Concept                    | Key Point                                                  |
|----------------------------|------------------------------------------------------------|
| Power law                  | L(N) ∝ N^(-α); loss vs scale is linear on log-log plot    |
| α_N (model size)           | ~0.076; need 10× params for ~16% loss reduction           |
| α_D (data size)            | ~0.095; data scaling is slightly more efficient than params|
| α_C (compute)              | ~0.050; compute scaling has smallest exponent              |
| Kaplan et al. finding      | Scale parameters faster than data                          |
| Chinchilla finding         | Scale parameters and data equally (D ≈ 20N)               |
| Compute formula            | C ≈ 6 · N · D FLOPs for transformer training              |
| GPT-3 was under-trained    | 300B tokens for 175B params (D/N = 1.7, optimal is 20)    |
| Over-training rationale    | Smaller models cheaper at inference even if training costs more |
| Irreducible loss           | Floor set by entropy of natural language                   |

---

## 10. Revision Questions

1. **State the three power law relationships** discovered by Kaplan et al. What are the approximate exponents for each?

2. **What is the Chinchilla-optimal ratio** of training tokens to parameters? Why was GPT-3 considered under-trained by this criterion?

3. **Given a compute budget of 10²² FLOPs**, calculate the Chinchilla-optimal number of parameters and training tokens.

4. **Explain the iso-FLOP curve.** Why does a U-shaped loss curve emerge when varying model size at fixed compute?

5. **Why might a team choose to over-train a smaller model** (like LLaMA-7B with 1T tokens) even though it's not compute-optimal for training?

6. **How would you use scaling laws to decide** between training a 10B model on 200B tokens vs. a 5B model on 400B tokens, given the same compute budget?

---

> **Navigation:** [← GPT-2 and GPT-3](03-gpt2-and-gpt3.md) | [In-Context Learning →](05-in-context-learning.md)

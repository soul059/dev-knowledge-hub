# 📊 Computational Graphs

> **Chapter 4.2 — Visualizing Neural Network Computation for Backpropagation**

| Topic | Details |
|---|---|
| **Subject** | Backpropagation — Computational Graphs |
| **Prerequisites** | Chapter 4.1 (Chain Rule) |
| **Key Insight** | Computational graphs make the chain rule systematic — each node computes a local gradient, and backprop is just multiplying along reverse paths |

---

## 📌 Overview

A **computational graph** is a directed acyclic graph (DAG) that represents a mathematical computation as a sequence of elementary operations. Every modern deep learning framework (PyTorch, TensorFlow, JAX) builds computational graphs internally. Understanding them is the key to understanding how backpropagation works, why certain operations are faster than others, and how automatic differentiation (autograd) functions.

---

## 1. What Is a Computational Graph?

### 1.1 Definition

```
    A computational graph is a DAG where:
    
    • NODES represent operations (+, ×, exp, log, ReLU, etc.)
      or input variables (x, w, b)
    
    • EDGES represent data flow (tensors flowing between operations)
    
    • The graph has a single output: the LOSS value L
    
    • No cycles! Data only flows forward (DAG = Directed Acyclic Graph)
```

### 1.2 Simple Example: f = (x + y) × z

```
    Forward Pass:

    x = 2 ──┐
             ├──► [+] ──► q = 5 ──┐
    y = 3 ──┘                      ├──► [×] ──► f = -20
                                   │
    z = -4 ────────────────────────┘

    The computation breaks down into elementary operations:
    1. q = x + y = 2 + 3 = 5
    2. f = q × z = 5 × (-4) = -20
```

```
    Backward Pass (chain rule at each node):

    Node [×]:
        ∂f/∂q = z = -4        (local gradient)
        ∂f/∂z = q = 5         (local gradient)

    Node [+]:
        ∂q/∂x = 1             (local gradient)
        ∂q/∂y = 1             (local gradient)

    Chain rule:
        ∂f/∂x = ∂f/∂q · ∂q/∂x = (-4)(1) = -4
        ∂f/∂y = ∂f/∂q · ∂q/∂y = (-4)(1) = -4
        ∂f/∂z = 5
```

### 1.3 Annotated Diagram with Gradients

```
    FORWARD PASS (→)                    BACKWARD PASS (←)
    ═══════════════                     ══════════════════

    x = 2 ──┐                          ∂f/∂x = -4 ◄──┐
             ├──[+]──► q = 5 ──┐                       ├── ∂f/∂q = -4 ◄──┐
    y = 3 ──┘                   │       ∂f/∂y = -4 ◄──┘                   │
                                ├──[×]──► f = -20       ∂f/∂f = 1 (start) │
    z = -4 ────────────────────┘        ∂f/∂z = 5  ◄─────────────────────┘

    At each node:
    upstream_gradient × local_gradient = downstream_gradient
```

---

## 2. Computational Graph for a Neuron

### 2.1 Single Neuron: z = wx + b, a = σ(z), L = (y - a)²

```
    Forward Pass:

    x = 2.0 ──┐
               ├──[×]──► p = wx = 1.0 ──┐
    w = 0.5 ──┘                          ├──[+]──► z = 1.1 ──[σ]──► a = 0.750
                                         │
    b = 0.1 ────────────────────────────┘
                                                                      │
                                                                      ├──[-]──► d = y-a = 0.250
                                                                      │
    y = 1.0 ──────────────────────────────────────────────────────────┘
                                                                              │
                                                                              ├──[²]──► L = 0.0625
```

### 2.2 Complete Forward + Backward Trace

```
    OPERATION BREAKDOWN:

    Step  │ Operation │ Formula       │ Value   │ Local Gradient(s)
    ──────┼───────────┼───────────────┼─────────┼────────────────────────
    1     │ multiply  │ p = w·x       │ 1.0     │ ∂p/∂w = x = 2.0
          │           │               │         │ ∂p/∂x = w = 0.5
    2     │ add       │ z = p + b     │ 1.1     │ ∂z/∂p = 1, ∂z/∂b = 1
    3     │ sigmoid   │ a = σ(z)      │ 0.750   │ ∂a/∂z = a(1-a) = 0.1875
    4     │ subtract  │ d = y - a     │ 0.250   │ ∂d/∂a = -1, ∂d/∂y = 1
    5     │ square    │ L = d²        │ 0.0625  │ ∂L/∂d = 2d = 0.500

    BACKWARD PASS (right to left):

    ∂L/∂L = 1.0                                          (seed)
    ∂L/∂d = 2d = 2(0.250) = 0.500                       (step 5)
    ∂L/∂a = ∂L/∂d · ∂d/∂a = 0.500 · (-1) = -0.500      (step 4)
    ∂L/∂z = ∂L/∂a · ∂a/∂z = -0.500 · 0.1875 = -0.09375 (step 3)
    ∂L/∂p = ∂L/∂z · ∂z/∂p = -0.09375 · 1 = -0.09375    (step 2)
    ∂L/∂b = ∂L/∂z · ∂z/∂b = -0.09375 · 1 = -0.09375    (step 2)
    ∂L/∂w = ∂L/∂p · ∂p/∂w = -0.09375 · 2.0 = -0.1875   (step 1)
    ∂L/∂x = ∂L/∂p · ∂p/∂x = -0.09375 · 0.5 = -0.046875 (step 1)

    ┌─────────────────────────────────────────────┐
    │  Gradients we need for learning:            │
    │  ∂L/∂w = -0.1875  → w should increase      │
    │  ∂L/∂b = -0.09375 → b should increase      │
    │                                             │
    │  Update: w ← w - η·(∂L/∂w)                 │
    │          w ← 0.5 - 0.1·(-0.1875) = 0.519   │
    └─────────────────────────────────────────────┘
```

---

## 3. Building Blocks — Common Node Types

### 3.1 Local Gradients for Elementary Operations

```
    ┌──────────────────────────────────────────────────────────┐
    │   OPERATION    │   FORWARD      │   LOCAL GRADIENTS      │
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = a + b    │   c = a + b    │   ∂c/∂a = 1           │
    │   (addition)   │                │   ∂c/∂b = 1           │
    │                │                │   "Gradient distributer"│
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = a × b    │   c = a × b    │   ∂c/∂a = b           │
    │   (multiply)   │                │   ∂c/∂b = a           │
    │                │                │   "Gradient swapper"    │
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = max(0,a) │   c = ReLU(a)  │   ∂c/∂a = 1 if a>0   │
    │   (ReLU)       │                │   ∂c/∂a = 0 if a≤0   │
    │                │                │   "Gradient gate"       │
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = σ(a)     │   c = 1/(1+e⁻ᵃ)│  ∂c/∂a = c(1-c)      │
    │   (sigmoid)    │                │   "Gradient squeezer"   │
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = a²       │   c = a²       │   ∂c/∂a = 2a          │
    │   (square)     │                │                        │
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = eᵃ       │   c = eᵃ       │   ∂c/∂a = eᵃ = c      │
    │   (exp)        │                │   Gradient = output!    │
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = log(a)   │   c = ln(a)    │   ∂c/∂a = 1/a         │
    │   (log)        │                │                        │
    ├────────────────┼────────────────┼────────────────────────┤
    │   c = 1/a      │   c = a⁻¹      │   ∂c/∂a = -1/a²       │
    │   (reciprocal) │                │                        │
    └────────────────┴────────────────┴────────────────────────┘
```

### 3.2 Backprop Rule at Each Node

```
    At every node in the graph:

    ┌──────────────────────────────────────────────────────┐
    │                                                      │
    │   incoming_gradient = upstream_gradient × local_grad  │
    │                                                      │
    │   "upstream" = gradient flowing FROM the loss         │
    │   "local"    = ∂(output)/∂(input) at this node       │
    │   "incoming" = gradient flowing TO the inputs         │
    │                                                      │
    └──────────────────────────────────────────────────────┘

    If a node has MULTIPLE outputs (fan-out):
    → The incoming gradients from all outputs are SUMMED

    If a node has MULTIPLE inputs:
    → Each input gets its own gradient (independent computation)
```

---

## 4. Full Network Computational Graph

### 4.1 2-Layer Network Graph

```
    L = MSE(σ(W²·ReLU(W¹·x + b¹) + b²), y)

    x ──┐                                           y ──┐
        ├──[W¹·]──► p₁ ──┐                              │
    W¹──┘                  ├──[+]──► z₁ ──[ReLU]──► a₁  │
                           │                              │
    b¹ ────────────────────┘                              │
                                                          │
    a₁ ──┐                                               │
         ├──[W²·]──► p₂ ──┐                              │
    W² ──┘                  ├──[+]──► z₂ ──[σ]──► ŷ ──┐  │
                           │                           │  │
    b² ────────────────────┘                           │  │
                                                       ├──[MSE]──► L
                                                       │
    y ─────────────────────────────────────────────────┘

    Total operations: 8 nodes
    Forward: Left to right (compute L from x)
    Backward: Right to left (compute ∂L/∂W¹, ∂L/∂W², ∂L/∂b¹, ∂L/∂b²)
```

---

## 5. PyTorch's Autograd — Automatic Computational Graphs

### 5.1 Dynamic Computation Graphs

```python
import torch

# PyTorch builds the graph DYNAMICALLY as operations are performed

x = torch.tensor(2.0, requires_grad=True)
w = torch.tensor(0.5, requires_grad=True)
b = torch.tensor(0.1, requires_grad=True)
y = torch.tensor(1.0)

# Forward pass — graph is built automatically
p = w * x           # MulBackward node created
z = p + b            # AddBackward node created
a = torch.sigmoid(z) # SigmoidBackward node created
d = y - a            # SubBackward node created
L = d ** 2           # PowBackward node created

print(f"Loss: {L.item():.6f}")
print(f"Gradient function chain: {L.grad_fn}")

# Backward pass — traverse graph in reverse
L.backward()

print(f"\n∂L/∂w = {w.grad.item():.6f}")
print(f"∂L/∂b = {b.grad.item():.6f}")
print(f"∂L/∂x = {x.grad.item():.6f}")

# Verify with our manual computation
print(f"\nManual ∂L/∂w = -0.1875 vs autograd = {w.grad.item():.4f}")
```

### 5.2 Inspecting the Graph

```python
# PyTorch lets you inspect the computational graph

def print_graph(tensor, indent=0):
    """Print the computational graph leading to this tensor."""
    prefix = "  " * indent
    if tensor.grad_fn is not None:
        print(f"{prefix}{tensor.grad_fn}")
        for child in tensor.grad_fn.next_functions:
            if child[0] is not None:
                # Create a dummy tensor to recurse
                print_graph_fn(child[0], indent + 1)

# Simpler approach — just check grad_fn
x = torch.tensor(2.0, requires_grad=True)
w = torch.tensor(0.5, requires_grad=True)

z = w * x + 0.1
a = torch.sigmoid(z)
L = (1.0 - a) ** 2

print(f"L.grad_fn: {L.grad_fn}")
print(f"  → {L.grad_fn.next_functions}")
```

### 5.3 Detaching from the Graph

```python
# Sometimes you DON'T want gradients (inference, fixed targets, etc.)

with torch.no_grad():
    # No graph is built → faster, less memory
    y_pred = model(x)

# Or detach a specific tensor
target = some_model(x).detach()  # Stop gradient flow here
loss = criterion(predictions, target)  # Gradients don't flow into some_model
```

---

## 6. Applications of Computational Graph Thinking

| Application | How Computational Graphs Help |
|---|---|
| **Debugging gradients** | Trace which node produces NaN/zero gradients |
| **Memory optimization** | Graph shows which tensors need to be cached |
| **Custom autograd** | Implement new operations with forward + backward |
| **Gradient checkpointing** | Trade compute for memory by recomputing activations |
| **Mixed precision** | Some nodes use float16, others float32 |
| **Pruning** | Remove graph edges (connections) with zero gradients |

---

## 📊 Summary Table

| Concept | Key Point |
|---|---|
| **Computational graph** | DAG of operations: nodes = ops, edges = data flow |
| **Forward pass** | Traverse left→right, compute output from inputs |
| **Backward pass** | Traverse right→left, multiply local gradients (chain rule) |
| **Local gradient** | ∂(output)/∂(input) at a single node |
| **Upstream gradient** | Gradient flowing from the loss toward this node |
| **Fan-out rule** | If a value is used multiple times, SUM the incoming gradients |
| **Addition node** | Distributes gradient equally to both inputs |
| **Multiplication node** | Swaps: ∂(ab)/∂a = b, ∂(ab)/∂b = a |
| **ReLU node** | Gates: passes gradient if input > 0, blocks if ≤ 0 |
| **PyTorch autograd** | Builds graph dynamically, runs backward automatically |

---

## ❓ Revision Questions

1. **Draw the computational graph for L = (wx + b - y)². Label every node with its operation, and annotate with forward values given w=2, x=3, b=-1, y=4.**

2. **Using the graph from Q1, compute all gradients ∂L/∂w, ∂L/∂x, ∂L/∂b by applying the chain rule node by node. Show the upstream gradient at each node.**

3. **What is the difference between static and dynamic computational graphs? Which approach does PyTorch use?**

4. **Explain the "gradient swapper" property of the multiplication node. Why does ∂(ab)/∂a = b?**

5. **A variable x is used in three different places in the graph. How do you compute ∂L/∂x? Draw a small example.**

6. **Why is `torch.no_grad()` important for inference? What does "detaching" a tensor mean in terms of the computational graph?**

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Chain Rule Review](01-chain-rule-review.md) | [🏠 Neural Networks Fundamentals](../README.md) | [Backward Pass Step-by-Step →](03-backward-pass-step-by-step.md) |

---

*© 2024 AIML Study Notes. Built for deep understanding, not shallow memorization.*

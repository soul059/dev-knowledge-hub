# 3. Framework Comparison

> **Unit 9 · Deep Learning Frameworks** — TensorFlow vs PyTorch vs JAX: making the right choice

---

## Chapter Overview

The deep learning framework landscape in 2024 is dominated by three players: **TensorFlow** (Google), **PyTorch** (Meta), and the rising **JAX** (Google DeepMind). Choosing the right framework impacts your productivity, debugging experience, deployment options, and career. This chapter provides a comprehensive, fair comparison across every dimension that matters — from execution models and debugging to deployment and community — and gives a clear decision guide for different use cases.

---

## 1. The Big Picture

```
  Deep Learning Framework Landscape (2024):
  
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  RESEARCH DOMINANCE:          PRODUCTION DOMINANCE:          │
  │  ████████████████ PyTorch     ████████████████ TensorFlow   │
  │  ████████ TensorFlow          ████████ PyTorch               │
  │  ████ JAX                     ██ JAX                         │
  │                                                              │
  │  PyTorch: ~70% of research papers (2024)                    │
  │  TensorFlow: ~70% of production deployments                 │
  │  JAX: Growing fast in cutting-edge research labs            │
  │                                                              │
  │  The gap is CLOSING — PyTorch deployment is improving       │
  │  and TensorFlow research usage is stable.                    │
  └──────────────────────────────────────────────────────────────┘
```

---

## 2. Execution Models

### Eager vs Graph Execution

```
  EAGER EXECUTION (PyTorch default, TF2 default):
  ┌──────────────────────────────────────────────────────────────┐
  │  x = tensor([1, 2, 3])                                      │
  │  y = x * 2                # executes IMMEDIATELY            │
  │  print(y)                 # tensor([2, 4, 6]) — value known!│
  │                                                              │
  │  ✓ Natural Python debugging (print, pdb, breakpoints)      │
  │  ✓ Dynamic control flow (if/else, loops based on values)   │
  │  ✗ Less optimizable (can't see the full computation)       │
  └──────────────────────────────────────────────────────────────┘
  
  GRAPH EXECUTION (TF1 default, @tf.function, torch.compile):
  ┌──────────────────────────────────────────────────────────────┐
  │  @tf.function                                                │
  │  def compute(x):                                             │
  │      y = x * 2            # builds graph, doesn't execute  │
  │      return y                                                │
  │                                                              │
  │  result = compute(x)      # graph is compiled & optimized  │
  │                                                              │
  │  ✓ Can optimize the full computation (fusion, reordering)  │
  │  ✓ Easier to deploy (graph is self-contained)              │
  │  ✗ Harder to debug (graph ≠ Python code)                   │
  │  ✗ Not all Python works in graph mode                      │
  └──────────────────────────────────────────────────────────────┘
```

### Framework Approaches

```
  ┌────────────────┬──────────────┬────────────────┬──────────────┐
  │                │ TensorFlow 2 │ PyTorch        │ JAX          │
  ├────────────────┼──────────────┼────────────────┼──────────────┤
  │ Default mode   │ Eager        │ Eager          │ Eager        │
  │ Graph mode     │ @tf.function │ torch.compile  │ jax.jit      │
  │ Philosophy     │ Both         │ Eager-first    │ Functional   │
  │ Graph maturity │ Very mature  │ Improving (2.0)│ Core design  │
  └────────────────┴──────────────┴────────────────┴──────────────┘
```

---

## 3. Debugging Experience

```
  DEBUGGING COMPARISON:
  
  ┌──────────────────────────────────────────────────────────────┐
  │  PYTORCH (Best debugging experience):                       │
  │                                                              │
  │  def forward(self, x):                                      │
  │      h = self.layer1(x)                                     │
  │      import pdb; pdb.set_trace()  # ← works perfectly!    │
  │      print(h.shape, h.mean())     # ← immediate values!   │
  │      h = self.layer2(h)                                     │
  │      return h                                               │
  │                                                              │
  │  ✓ Standard Python debugger works                          │
  │  ✓ print() shows actual values                             │
  │  ✓ Can inspect intermediate tensors                        │
  │  ✓ torch.autograd.set_detect_anomaly(True) finds NaN source│
  ├──────────────────────────────────────────────────────────────┤
  │  TENSORFLOW 2 (Good in eager mode):                         │
  │                                                              │
  │  # Eager mode — similar to PyTorch                         │
  │  x = tf.constant([1.0, 2.0])                               │
  │  y = x * 2                                                  │
  │  print(y)   # tf.Tensor([2. 4.], ...)  ← works!           │
  │                                                              │
  │  # Inside @tf.function — harder:                           │
  │  @tf.function                                               │
  │  def compute(x):                                            │
  │      tf.print(x)   # must use tf.print, not print!        │
  │      return x * 2                                           │
  │                                                              │
  │  ⚠ @tf.function transforms Python code into a graph       │
  │  ⚠ Some Python constructs don't work inside @tf.function  │
  ├──────────────────────────────────────────────────────────────┤
  │  JAX (Functional — different mental model):                 │
  │                                                              │
  │  # JAX is purely functional — no side effects              │
  │  # print() doesn't work inside jit-compiled functions!     │
  │  @jax.jit                                                   │
  │  def compute(x):                                            │
  │      # print(x)  ← WON'T WORK! (traced, not executed)    │
  │      jax.debug.print("{x}", x=x)  # ← use this instead   │
  │      return x * 2                                           │
  └──────────────────────────────────────────────────────────────┘
```

---

## 4. Deployment

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    DEPLOYMENT OPTIONS                        │
  │                                                              │
  │  TENSORFLOW:                                                 │
  │  ┌──────────────────────────────────────────────┐           │
  │  │  TF Serving    → Production server (gRPC/REST)│          │
  │  │  TF Lite       → Mobile & IoT                 │          │
  │  │  TF.js         → Browser (JavaScript)         │          │
  │  │  SavedModel    → Standard format              │          │
  │  │  TFX           → Full ML pipeline             │          │
  │  │  Google Cloud   → Vertex AI integration       │          │
  │  └──────────────────────────────────────────────┘           │
  │  ✓ Most mature deployment ecosystem                        │
  │  ✓ Battle-tested at Google scale                           │
  │                                                              │
  │  PYTORCH:                                                    │
  │  ┌──────────────────────────────────────────────┐           │
  │  │  TorchServe    → Production server            │          │
  │  │  PyTorch Mobile→ iOS & Android                │          │
  │  │  ExecuTorch    → On-device (new, 2024)       │          │
  │  │  ONNX Export   → Cross-framework format       │          │
  │  │  torch.compile → Optimized inference          │          │
  │  │  AWS SageMaker → Cloud integration            │          │
  │  └──────────────────────────────────────────────┘           │
  │  ⚠ Deployment improving rapidly but less mature than TF    │
  │                                                              │
  │  JAX:                                                        │
  │  ┌──────────────────────────────────────────────┐           │
  │  │  jax2tf         → Convert JAX → TF SavedModel│          │
  │  │  ONNX (via jax) → Cross-framework            │          │
  │  │  Google TPUs    → First-class support         │          │
  │  └──────────────────────────────────────────────┘           │
  │  ⚠ Limited native deployment — usually converts to TF     │
  └──────────────────────────────────────────────────────────────┘
```

### Mobile Deployment

```
  ┌──────────────────────────────────────────────────────────────┐
  │  Mobile AI Framework Comparison:                            │
  │                                                              │
  │  TF Lite:                         PyTorch Mobile/ExecuTorch:│
  │  • Very mature                    • Newer but improving     │
  │  • Tiny binary size               • Larger binary           │
  │  • Excellent quantization         • Good quantization       │
  │  • Delegate system for            • Custom backends for     │
  │    hardware acceleration            GPU/NPU acceleration    │
  │  • Huge ecosystem                 • Growing ecosystem       │
  │  • Used in billions of devices    • Growing adoption        │
  │                                                              │
  │  Winner for mobile: TF Lite (maturity)                      │
  │  But ExecuTorch is closing the gap rapidly                  │
  └──────────────────────────────────────────────────────────────┘
```

---

## 5. Community and Research Usage

```
  Research Paper Framework Usage (2020-2024):
  
  2020:  PyTorch ██████████████████████ 55%
         TensorFlow ████████████████ 40%
         JAX ██ 5%
  
  2022:  PyTorch ██████████████████████████ 65%
         TensorFlow ██████████████ 28%
         JAX ████ 7%
  
  2024:  PyTorch ████████████████████████████ 70%
         TensorFlow ██████████ 20%
         JAX ████████ 10%
  
  Key trend: PyTorch dominates research.
  TensorFlow dominates production.
  JAX is the choice of Google DeepMind.
  
  
  Industry Usage:
  ┌─────────────────────────────────────────────────────┐
  │  Google/DeepMind:  TensorFlow + JAX               │
  │  Meta/Facebook:    PyTorch                          │
  │  OpenAI:           PyTorch (GPT models)            │
  │  Microsoft:        PyTorch + ONNX                  │
  │  Amazon:           PyTorch + TensorFlow            │
  │  Hugging Face:     PyTorch (primary)               │
  │  Tesla:            PyTorch                          │
  │  Apple:            Core ML (imports from both)     │
  └─────────────────────────────────────────────────────┘
```

---

## 6. Learning Curve

```
  Learning Curve Comparison:
  
  Productivity ↑
               │                         ╱── JAX (powerful when mastered
               │                    ╱───╱     but steep learning curve)
               │               ╱───╱
               │          ╱───╱
  Keras ──────→│─────────╱────────────── ← Keras (fastest start)
               │        ╱     
               │   ╱───╱
  PyTorch ────→│──╱──────────────────── ← PyTorch (moderate start,
               │ ╱                        full control quickly)
               │╱
               └──────────────────────→ Time invested
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Keras:    Fastest to "hello world" (3 lines to train)      │
  │            But walls appear when you need custom behavior   │
  │                                                              │
  │  PyTorch:  Moderate start (need to write training loop)     │
  │            But very consistent — same approach for simple   │
  │            and complex models                               │
  │                                                              │
  │  JAX:      Steep start (functional programming paradigm)    │
  │            But incredibly powerful for research once mastered│
  │            (automatic vectorization, differentiation, JIT)  │
  └──────────────────────────────────────────────────────────────┘
```

---

## 7. ONNX Interoperability

```
  ONNX = Open Neural Network Exchange
  
  A standard format for exchanging models between frameworks:
  
  ┌──────────┐     ┌──────┐     ┌──────────────┐
  │ PyTorch  │────→│      │────→│ TensorFlow   │
  └──────────┘     │      │     └──────────────┘
                   │ ONNX │
  ┌──────────┐     │      │     ┌──────────────┐
  │ TF/Keras │────→│      │────→│ ONNX Runtime │
  └──────────┘     └──────┘     └──────────────┘
                                        │
                              ┌─────────┼─────────┐
                              ↓         ↓         ↓
                           CPU       GPU       Edge
                         inference inference  devices
```

```python
# PyTorch → ONNX export:
import torch

model = MyModel()
model.eval()
dummy_input = torch.randn(1, 1, 28, 28)

torch.onnx.export(
    model,
    dummy_input,
    "model.onnx",
    input_names=['input'],
    output_names=['output'],
    dynamic_axes={'input': {0: 'batch_size'},
                  'output': {0: 'batch_size'}},
    opset_version=17
)

# Run with ONNX Runtime (fast inference):
import onnxruntime as ort

session = ort.InferenceSession("model.onnx")
result = session.run(None, {"input": input_data.numpy()})
```

---

## 8. JAX as an Alternative

```
  JAX = Autograd + XLA (Accelerated Linear Algebra)
  
  Key Philosophy: Functional transformations of functions
  
  ┌──────────────────────────────────────────────────────────────┐
  │  JAX TRANSFORMATIONS:                                        │
  │                                                              │
  │  jax.grad(f)     → automatic differentiation               │
  │  jax.jit(f)      → just-in-time compilation (XLA)          │
  │  jax.vmap(f)     → automatic vectorization (batch!)        │
  │  jax.pmap(f)     → automatic parallelization (multi-GPU!)  │
  │                                                              │
  │  These COMPOSE:                                              │
  │  jax.jit(jax.vmap(jax.grad(loss_fn)))                      │
  │  = Compiled, batched gradient computation!                  │
  └──────────────────────────────────────────────────────────────┘
```

```python
# JAX example:
import jax
import jax.numpy as jnp

def loss_fn(params, x, y):
    pred = jnp.dot(x, params['w']) + params['b']
    return jnp.mean((pred - y) ** 2)

# Automatic differentiation
grad_fn = jax.grad(loss_fn)

# JIT compile for speed
fast_grad_fn = jax.jit(grad_fn)

# Compute gradients
params = {'w': jnp.ones(784), 'b': jnp.zeros(1)}
grads = fast_grad_fn(params, x_batch, y_batch)

# With libraries like Flax or Haiku:
# import flax.linen as nn
# class MLP(nn.Module):
#     @nn.compact
#     def __call__(self, x):
#         x = nn.Dense(256)(x)
#         x = nn.relu(x)
#         x = nn.Dense(10)(x)
#         return x
```

---

## 9. Comprehensive Comparison Table

```
  ┌──────────────────────┬─────────────────┬─────────────────┬───────────────┐
  │  Feature             │ TensorFlow 2    │ PyTorch          │ JAX          │
  ├──────────────────────┼─────────────────┼─────────────────┼───────────────┤
  │  Creator             │ Google          │ Meta (Facebook)  │ Google       │
  │  Initial Release     │ 2015            │ 2016             │ 2018         │
  │  Execution           │ Eager + Graph   │ Eager (+ compile)│ Functional   │
  │  High-level API      │ Keras           │ nn.Module        │ Flax / Haiku │
  │  Debugging           │ Good (eager)    │ Best             │ Different    │
  │  Deployment          │ Best            │ Good (improving) │ Limited      │
  │  Mobile              │ TF Lite (best)  │ ExecuTorch       │ Via TF       │
  │  Research Usage      │ Declining       │ Dominant (70%)   │ Growing      │
  │  Production Usage    │ Dominant        │ Growing fast     │ Niche        │
  │  TPU Support         │ First-class     │ Limited          │ First-class  │
  │  GPU Support         │ Excellent       │ Excellent        │ Excellent    │
  │  Community Size      │ Large           │ Largest          │ Smaller      │
  │  Job Market          │ Many jobs       │ Most jobs        │ Few jobs     │
  │  Learning Curve      │ Easy (Keras)    │ Moderate         │ Steep        │
  │  Customization       │ Moderate        │ High             │ Highest      │
  │  Auto Differentiation│ tf.GradientTape │ autograd         │ jax.grad     │
  │  Distributed Training│ tf.distribute   │ DDP (excellent)  │ pmap/pjit   │
  │  Model Hub           │ TF Hub          │ Hugging Face     │ HF (growing)│
  │  ONNX Support        │ Via converter   │ Native export    │ Via jax2tf   │
  └──────────────────────┴─────────────────┴─────────────────┴───────────────┘
```

---

## 10. Decision Guide

```
  WHICH FRAMEWORK SHOULD YOU CHOOSE?
  
  ┌──────────────────────────────────────────────────────────────┐
  │  YOU ARE...              │  RECOMMENDATION                  │
  ├──────────────────────────┼──────────────────────────────────┤
  │  A STUDENT learning DL   │  PyTorch (most tutorials/courses)│
  │  A RESEARCHER            │  PyTorch (most papers use it)    │
  │  An ML ENGINEER (deploy) │  TensorFlow (best deployment)   │
  │  Working at GOOGLE       │  TensorFlow or JAX              │
  │  Working at META/OpenAI  │  PyTorch                         │
  │  Building MOBILE apps    │  TensorFlow (TF Lite is best)   │
  │  Doing CUTTING-EDGE      │  JAX (if at Google/DeepMind)    │
  │  research                │  PyTorch (everywhere else)      │
  │  Need FASTEST iteration  │  Keras (least boilerplate)      │
  │  Need MOST CONTROL       │  PyTorch or JAX                 │
  │  Using HUGGING FACE      │  PyTorch (primary framework)    │
  │  Training on TPUs        │  JAX or TensorFlow              │
  ├──────────────────────────┼──────────────────────────────────┤
  │                                                              │
  │  GENERAL RECOMMENDATION:                                     │
  │                                                              │
  │  Learn PyTorch FIRST — it's the lingua franca of DL.       │
  │  Learn TF/Keras if you need production deployment.          │
  │  Learn JAX if you're doing advanced research.               │
  │                                                              │
  │  Knowing BOTH PyTorch and TF/Keras is ideal for industry.  │
  └──────────────────────────────────────────────────────────────┘
```

---

## 11. Worked Example: Same Model in All Three

```python
# ─── KERAS VERSION ─── (~15 lines)
import tensorflow as tf
model = tf.keras.Sequential([
    tf.keras.layers.Flatten(input_shape=(28, 28)),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dense(10, activation='softmax'),
])
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
model.fit(X_train, y_train, epochs=5, validation_split=0.1)


# ─── PYTORCH VERSION ─── (~35 lines)
import torch, torch.nn as nn
class MLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(nn.Flatten(), nn.Linear(784, 128), nn.ReLU(), nn.Linear(128, 10))
    def forward(self, x):
        return self.net(x)

model = MLP()
opt = torch.optim.Adam(model.parameters())
for epoch in range(5):
    for X, y in train_loader:
        opt.zero_grad()
        nn.CrossEntropyLoss()(model(X), y).backward()
        opt.step()


# ─── JAX/FLAX VERSION ─── (~30 lines)
import jax, jax.numpy as jnp, flax.linen as nn, optax
class MLP(nn.Module):
    @nn.compact
    def __call__(self, x):
        x = x.reshape((x.shape[0], -1))
        x = nn.Dense(128)(x)
        x = nn.relu(x)
        return nn.Dense(10)(x)

model = MLP()
params = model.init(jax.random.PRNGKey(0), jnp.ones((1, 28, 28)))
tx = optax.adam(1e-3)
opt_state = tx.init(params)

@jax.jit
def train_step(params, opt_state, x, y):
    def loss_fn(p):
        logits = model.apply(p, x)
        return optax.softmax_cross_entropy_with_integer_labels(logits, y).mean()
    grads = jax.grad(loss_fn)(params)
    updates, opt_state_new = tx.update(grads, opt_state)
    return optax.apply_updates(params, updates), opt_state_new
```

---

## 12. Summary Table

| Dimension | TensorFlow/Keras | PyTorch | JAX |
|-----------|:----------------:|:-------:|:---:|
| **Ease of use** | ★★★★★ | ★★★★ | ★★★ |
| **Debugging** | ★★★★ | ★★★★★ | ★★★ |
| **Deployment** | ★★★★★ | ★★★★ | ★★ |
| **Research adoption** | ★★★ | ★★★★★ | ★★★★ |
| **Production adoption** | ★★★★★ | ★★★★ | ★★ |
| **Flexibility** | ★★★★ | ★★★★★ | ★★★★★ |
| **Documentation** | ★★★★★ | ★★★★★ | ★★★ |
| **Mobile** | ★★★★★ | ★★★ | ★ |
| **TPU support** | ★★★★★ | ★★ | ★★★★★ |

---

## 13. Revision Questions

1. **Compare eager execution and graph execution.** What are the advantages and disadvantages of each? How do TensorFlow 2 and PyTorch handle this differently?

2. **Why has PyTorch become the dominant framework for research while TensorFlow leads in production?** What technical and ecosystem factors drive this split?

3. **What is ONNX and why is it important?** How would you export a PyTorch model to ONNX format? Write the code.

4. **Describe JAX's four key transformations (grad, jit, vmap, pmap).** Why is JAX considered more "functional" than PyTorch?

5. **A company needs to train a model in Python and deploy it on both a cloud server and mobile devices. Which framework would you recommend and why?**

6. **Compare TF Lite and PyTorch Mobile/ExecuTorch for mobile deployment.** Which is more mature, and what are the tradeoffs?

---

| [← 02 PyTorch Basics](02-pytorch-basics.md) | [Unit 9 Home](README.md) | [04 → GPU Training](04-gpu-training.md) |
|:---|:---:|---:|

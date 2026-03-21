# 5. Model Saving and Loading

> **Unit 9 · Deep Learning Frameworks** — Persisting trained models for inference, checkpointing, and deployment

---

## Chapter Overview

After hours of training, your model's learned weights exist only in GPU/CPU memory — one power outage or crash and they're gone. **Model saving** preserves your trained model to disk so it can be loaded later for inference, fine-tuning, or deployment. This chapter covers all the ways to save and load models in both PyTorch and TensorFlow, checkpoint strategies for resuming training, ONNX export for cross-framework deployment, and model versioning best practices.

---

## 1. Why Save Models?

```
  ┌──────────────────────────────────────────────────────────────┐
  │  USE CASES FOR SAVING MODELS:                                │
  │                                                              │
  │  1. INFERENCE / DEPLOYMENT                                   │
  │     Train once → save → load in production server           │
  │                                                              │
  │  2. CHECKPOINTING                                            │
  │     Save periodically during training                       │
  │     → Resume if training crashes                            │
  │     → Go back to best epoch                                 │
  │                                                              │
  │  3. TRANSFER LEARNING                                        │
  │     Save pretrained model → load → fine-tune on new task    │
  │                                                              │
  │  4. SHARING / REPRODUCIBILITY                                │
  │     Share trained model with collaborators                   │
  │     Publish alongside research paper                        │
  │                                                              │
  │  5. MODEL SELECTION                                          │
  │     Save multiple models → compare → deploy best one       │
  └──────────────────────────────────────────────────────────────┘
```

---

## 2. PyTorch: Saving and Loading

### Method 1: Save/Load state_dict (Recommended)

```python
import torch
import torch.nn as nn

# ── Define model ──
class MLP(nn.Module):
    def __init__(self, input_size=784, hidden_size=256, num_classes=10):
        super().__init__()
        self.net = nn.Sequential(
            nn.Flatten(),
            nn.Linear(input_size, hidden_size),
            nn.ReLU(),
            nn.Dropout(0.3),
            nn.Linear(hidden_size, num_classes)
        )
    
    def forward(self, x):
        return self.net(x)

model = MLP()
# ... train the model ...

# ═══════════════════════════════════════════════
#  SAVE: state_dict only (weights and biases)
# ═══════════════════════════════════════════════
torch.save(model.state_dict(), 'model_weights.pth')
# File contains: {'net.1.weight': tensor(...), 'net.1.bias': tensor(...), ...}

# ═══════════════════════════════════════════════
#  LOAD: must create model first, then load weights
# ═══════════════════════════════════════════════
model = MLP()  # create same architecture!
model.load_state_dict(torch.load('model_weights.pth'))
model.eval()   # set to evaluation mode

# Make predictions
with torch.no_grad():
    predictions = model(test_input)
```

```
  state_dict is an OrderedDict mapping layer names to tensors:
  
  {
    'net.1.weight': tensor of shape (256, 784),
    'net.1.bias':   tensor of shape (256,),
    'net.3.weight': tensor of shape (10, 256),
    'net.3.bias':   tensor of shape (10,),
  }
  
  ✓ Small file size (just weights)
  ✓ Flexible (can load into different code versions)
  ✓ Safe (no arbitrary code execution)
  ✗ Must have model class definition to load
```

### Method 2: Save Entire Model

```python
# SAVE entire model (architecture + weights)
torch.save(model, 'full_model.pth')

# LOAD — no need to define the class!
model = torch.load('full_model.pth')
model.eval()
```

```
  ┌──────────────────────────────────────────────────────────────┐
  │  ⚠ WARNING: torch.save(model) uses Python's pickle         │
  │                                                              │
  │  Risks:                                                      │
  │  • Pickle is NOT secure — can execute arbitrary code!       │
  │  • Breaks if you move/rename the model class file           │
  │  • Tied to Python version and PyTorch version               │
  │  • Larger file size                                         │
  │                                                              │
  │  Recommendation: ALWAYS use state_dict for production!      │
  │  Use full model save only for quick prototyping.            │
  └──────────────────────────────────────────────────────────────┘
```

### Method 3: Save Complete Checkpoint (Best for Training)

```python
# ═══════════════════════════════════════════════
#  SAVE CHECKPOINT (everything needed to resume)
# ═══════════════════════════════════════════════

def save_checkpoint(model, optimizer, scheduler, epoch, loss, path):
    """Save complete training state."""
    torch.save({
        'epoch': epoch,
        'model_state_dict': model.state_dict(),
        'optimizer_state_dict': optimizer.state_dict(),
        'scheduler_state_dict': scheduler.state_dict() if scheduler else None,
        'loss': loss,
        'best_val_acc': best_val_acc,
    }, path)
    print(f"Checkpoint saved: epoch {epoch}, loss {loss:.4f}")

# During training:
for epoch in range(num_epochs):
    train_loss = train_one_epoch(model, optimizer, train_loader)
    val_acc = evaluate(model, val_loader)
    
    scheduler.step()
    
    # Save every 5 epochs
    if (epoch + 1) % 5 == 0:
        save_checkpoint(model, optimizer, scheduler, epoch, train_loss,
                       f'checkpoint_epoch{epoch+1}.pth')
    
    # Save best model
    if val_acc > best_val_acc:
        best_val_acc = val_acc
        save_checkpoint(model, optimizer, scheduler, epoch, train_loss,
                       'best_checkpoint.pth')


# ═══════════════════════════════════════════════
#  LOAD CHECKPOINT (resume training)
# ═══════════════════════════════════════════════

def load_checkpoint(path, model, optimizer=None, scheduler=None):
    """Load checkpoint and resume training."""
    checkpoint = torch.load(path)
    
    model.load_state_dict(checkpoint['model_state_dict'])
    
    if optimizer:
        optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
    
    if scheduler and checkpoint.get('scheduler_state_dict'):
        scheduler.load_state_dict(checkpoint['scheduler_state_dict'])
    
    epoch = checkpoint['epoch']
    loss = checkpoint['loss']
    
    print(f"Loaded checkpoint from epoch {epoch}, loss {loss:.4f}")
    return epoch

# Resume training:
model = MLP().to(device)
optimizer = optim.Adam(model.parameters(), lr=1e-3)
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=10)

start_epoch = load_checkpoint('checkpoint_epoch50.pth', model, optimizer, scheduler)

for epoch in range(start_epoch + 1, num_epochs):
    train_one_epoch(model, optimizer, train_loader)
    scheduler.step()
```

### Loading on Different Device

```python
# Model saved on GPU, loading on CPU:
model = MLP()
model.load_state_dict(
    torch.load('model_weights.pth', map_location=torch.device('cpu'))
)

# Model saved on GPU 0, loading on GPU 1:
model.load_state_dict(
    torch.load('model_weights.pth', map_location='cuda:1')
)

# Model saved on CPU, loading on GPU:
model = MLP()
model.load_state_dict(torch.load('model_weights.pth'))
model.to(device)  # then move to GPU
```

---

## 3. TensorFlow/Keras: Saving and Loading

### SavedModel Format (Recommended for Deployment)

```python
import tensorflow as tf

# Build and train model
model = tf.keras.Sequential([
    tf.keras.layers.Flatten(input_shape=(28, 28)),
    tf.keras.layers.Dense(256, activation='relu'),
    tf.keras.layers.Dense(10, activation='softmax')
])
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy')
model.fit(X_train, y_train, epochs=10)

# ═══════════════════════════════════════════════
#  SAVE as SavedModel (TF's standard format)
# ═══════════════════════════════════════════════
model.save('saved_model_dir')
# Creates a directory:
# saved_model_dir/
#   ├── saved_model.pb        (computation graph)
#   ├── variables/
#   │   ├── variables.data    (weights)
#   │   └── variables.index
#   └── assets/               (optional extra files)

# LOAD
loaded_model = tf.keras.models.load_model('saved_model_dir')
predictions = loaded_model.predict(X_test)
```

### Keras Format (.keras)

```python
# Modern Keras format (TF 2.13+)
model.save('model.keras')

# Load
loaded_model = tf.keras.models.load_model('model.keras')
```

### H5 Format (Legacy)

```python
# HDF5 format (older, still widely used)
model.save('model.h5')

# Or save weights only
model.save_weights('weights.h5')

# Load
loaded_model = tf.keras.models.load_model('model.h5')

# Load weights only (must have same architecture)
model = build_model()
model.load_weights('weights.h5')
```

### TF Format Comparison

```
  ┌───────────────────┬────────────────┬──────────────┬──────────────┐
  │  Format           │ SavedModel     │ .keras       │ .h5 (HDF5)   │
  ├───────────────────┼────────────────┼──────────────┼──────────────┤
  │  Includes graph   │ Yes            │ Yes          │ Yes          │
  │  Includes weights │ Yes            │ Yes          │ Yes          │
  │  Includes config  │ Yes            │ Yes          │ Yes          │
  │  TF Serving       │ ✓ (native)    │ Convert      │ Convert      │
  │  TF Lite          │ ✓ (convert)   │ Convert      │ Convert      │
  │  TF.js            │ ✓ (convert)   │ Convert      │ Convert      │
  │  Custom objects    │ ✓             │ ✓            │ ✓            │
  │  Recommended      │ Deployment     │ General use  │ Legacy       │
  └───────────────────┴────────────────┴──────────────┴──────────────┘
```

---

## 4. ONNX Export

### PyTorch to ONNX

```python
import torch
import torch.onnx

model = MLP()
model.load_state_dict(torch.load('best_model.pth'))
model.eval()

# Create dummy input matching your model's expected input
dummy_input = torch.randn(1, 1, 28, 28)

# Export to ONNX
torch.onnx.export(
    model,                          # model to export
    dummy_input,                    # example input
    'model.onnx',                   # output file
    export_params=True,             # include weights
    opset_version=17,               # ONNX version
    do_constant_folding=True,       # optimize constants
    input_names=['input'],          # input tensor names
    output_names=['output'],        # output tensor names
    dynamic_axes={                  # allow variable batch size
        'input': {0: 'batch_size'},
        'output': {0: 'batch_size'}
    }
)

print("Model exported to model.onnx")
```

### Running ONNX Models

```python
# pip install onnxruntime  (CPU)
# pip install onnxruntime-gpu  (GPU)

import onnxruntime as ort
import numpy as np

# Create inference session
session = ort.InferenceSession('model.onnx')

# Get input/output names
input_name = session.get_inputs()[0].name
output_name = session.get_outputs()[0].name

# Run inference
test_input = np.random.randn(1, 1, 28, 28).astype(np.float32)
result = session.run([output_name], {input_name: test_input})
predictions = result[0]  # numpy array of shape (1, 10)
print(f"Predicted class: {predictions.argmax()}")
```

### TensorFlow to ONNX

```python
# pip install tf2onnx

import tf2onnx
import tensorflow as tf

model = tf.keras.models.load_model('saved_model_dir')

# Convert
spec = (tf.TensorSpec((None, 28, 28), tf.float32, name="input"),)
output_path = "model.onnx"
model_proto, _ = tf2onnx.convert.from_keras(model, input_signature=spec, 
                                              output_path=output_path)
```

```
  ONNX Ecosystem:
  
  ┌──────────┐    ┌──────────┐    ┌──────────────────────┐
  │ PyTorch  │───→│          │───→│ ONNX Runtime (fast!) │
  └──────────┘    │          │    │ TensorRT (NVIDIA)     │
                  │  ONNX    │    │ OpenVINO (Intel)      │
  ┌──────────┐    │  Model   │    │ Core ML (Apple)       │
  │ TF/Keras │───→│ (.onnx)  │───→│ DirectML (Windows)    │
  └──────────┘    │          │    │ Web (ONNX.js)         │
                  │          │    │ Mobile (various)      │
  ┌──────────┐    │          │    └──────────────────────┘
  │ JAX      │───→│          │
  └──────────┘    └──────────┘
```

---

## 5. Model Versioning

### File Naming Conventions

```
  GOOD naming:
  ┌──────────────────────────────────────────────────────────────┐
  │  model_v1.0_mnist_mlp_acc9812_20240115.pth                 │
  │  ├─── version  ├─── dataset  ├── metric   ├── date         │
  │       ├─── architecture                                     │
  │                                                              │
  │  checkpoint_epoch050_valloss0234.pth                        │
  │  best_model_exp42_acc9856.pth                               │
  └──────────────────────────────────────────────────────────────┘
  
  BAD naming:
  ┌──────────────────────────────────────────────────────────────┐
  │  model.pth              → which model? when?               │
  │  best.pth               → best of what experiment?         │
  │  final_final_v2_new.pth → chaos!                           │
  └──────────────────────────────────────────────────────────────┘
```

### Saving Metadata with the Model

```python
import json
from datetime import datetime

def save_model_with_metadata(model, config, metrics, path):
    """Save model with complete metadata for reproducibility."""
    save_dict = {
        'model_state_dict': model.state_dict(),
        'config': config,
        'metrics': metrics,
        'timestamp': datetime.now().isoformat(),
        'pytorch_version': torch.__version__,
        'model_class': model.__class__.__name__,
    }
    torch.save(save_dict, path)
    
    # Also save metadata as readable JSON
    metadata = {k: v for k, v in save_dict.items() if k != 'model_state_dict'}
    json_path = path.replace('.pth', '_metadata.json')
    with open(json_path, 'w') as f:
        json.dump(metadata, f, indent=2, default=str)
    
    print(f"Model saved to {path}")
    print(f"Metadata saved to {json_path}")

# Usage:
config = {
    'architecture': 'MLP',
    'hidden_size': 256,
    'dropout': 0.3,
    'learning_rate': 1e-3,
    'batch_size': 128,
    'epochs_trained': 30,
}

metrics = {
    'train_loss': 0.0234,
    'val_loss': 0.0567,
    'val_accuracy': 0.9821,
    'test_accuracy': 0.9815,
}

save_model_with_metadata(model, config, metrics, 'models/mlp_v1.0.pth')
```

---

## 6. Deployment Considerations

```
  ┌──────────────────────────────────────────────────────────────┐
  │  FROM TRAINING TO DEPLOYMENT                                 │
  │                                                              │
  │  Training Environment          Production Environment       │
  │  ─────────────────────         ────────────────────────      │
  │  • GPU (NVIDIA V100)          • Cloud server (CPU/GPU)     │
  │  • PyTorch + CUDA             • TorchServe / TF Serving    │
  │  • Python 3.10                • Docker container            │
  │  • Full dependencies          • Minimal dependencies       │
  │                                                              │
  │  Steps:                                                      │
  │  1. Train model                                             │
  │  2. Save best checkpoint                                    │
  │  3. Export (ONNX / TorchScript / SavedModel)               │
  │  4. Optimize (quantization, pruning)                        │
  │  5. Containerize (Docker)                                   │
  │  6. Deploy to serving platform                              │
  │  7. Monitor in production                                   │
  └──────────────────────────────────────────────────────────────┘
```

### PyTorch TorchScript Export

```python
# TorchScript: serialize PyTorch model for C++ runtime (no Python needed)

model.eval()

# Method 1: Tracing (works for most models)
traced_model = torch.jit.trace(model, torch.randn(1, 1, 28, 28))
traced_model.save('model_traced.pt')

# Method 2: Scripting (works with control flow)
scripted_model = torch.jit.script(model)
scripted_model.save('model_scripted.pt')

# Load in Python
loaded = torch.jit.load('model_traced.pt')
output = loaded(test_input)

# Load in C++ (for production):
# torch::jit::script::Module model;
# model = torch::jit::load("model_traced.pt");
```

### Model Registry Basics

```
  MODEL REGISTRY: Central store for trained models
  
  ┌──────────────────────────────────────────────────────────────┐
  │  Model Name    │ Version │ Stage      │ Metrics            │
  ├────────────────┼─────────┼────────────┼────────────────────┤
  │  mnist-mlp     │ 1.0     │ Archived   │ acc: 97.8%         │
  │  mnist-mlp     │ 2.0     │ Production │ acc: 98.2%         │
  │  mnist-mlp     │ 3.0     │ Staging    │ acc: 98.5%         │
  │  cifar-resnet  │ 1.0     │ Production │ acc: 93.1%         │
  └────────────────┴─────────┴────────────┴────────────────────┘
  
  Tools: MLflow, Weights & Biases, DVC, Neptune
  
  MLflow example:
  import mlflow
  
  with mlflow.start_run():
      mlflow.log_params(config)
      mlflow.log_metrics(metrics)
      mlflow.pytorch.log_model(model, "model")
      # Registers model in MLflow Model Registry
```

---

## 7. Summary Table

| Method | Framework | Saves | File Size | Best For |
|--------|-----------|-------|-----------|----------|
| **state_dict** | PyTorch | Weights only | Small | ✓ Production, sharing |
| **torch.save(model)** | PyTorch | Full model (pickle) | Large | Quick prototyping |
| **Checkpoint** | PyTorch | Model+optimizer+epoch | Large | Resume training |
| **TorchScript** | PyTorch | Serialized model | Medium | C++ deployment |
| **ONNX** | Both | Cross-framework | Medium | Cross-platform deploy |
| **SavedModel** | TensorFlow | Full model | Medium | TF Serving, TF Lite |
| **.keras** | TensorFlow | Full model | Medium | General Keras use |
| **.h5** | TensorFlow | Full model | Medium | Legacy/compatibility |

---

## 8. Complete Example: Checkpoint and Resume

```python
import torch
import torch.nn as nn
import torch.optim as optim
from pathlib import Path

class Trainer:
    def __init__(self, model, optimizer, scheduler, criterion, device, save_dir='checkpoints'):
        self.model = model.to(device)
        self.optimizer = optimizer
        self.scheduler = scheduler
        self.criterion = criterion
        self.device = device
        self.save_dir = Path(save_dir)
        self.save_dir.mkdir(exist_ok=True)
        self.best_val_acc = 0
        self.start_epoch = 0
    
    def save_checkpoint(self, epoch, val_acc, is_best=False):
        state = {
            'epoch': epoch,
            'model_state_dict': self.model.state_dict(),
            'optimizer_state_dict': self.optimizer.state_dict(),
            'scheduler_state_dict': self.scheduler.state_dict(),
            'best_val_acc': self.best_val_acc,
        }
        # Save latest
        torch.save(state, self.save_dir / 'latest.pth')
        # Save best
        if is_best:
            torch.save(state, self.save_dir / 'best.pth')
            print(f"  ★ New best model saved (acc: {val_acc:.4f})")
    
    def load_checkpoint(self, path='latest'):
        ckpt_path = self.save_dir / f'{path}.pth'
        if not ckpt_path.exists():
            print("No checkpoint found, starting from scratch.")
            return
        
        ckpt = torch.load(ckpt_path, map_location=self.device)
        self.model.load_state_dict(ckpt['model_state_dict'])
        self.optimizer.load_state_dict(ckpt['optimizer_state_dict'])
        self.scheduler.load_state_dict(ckpt['scheduler_state_dict'])
        self.start_epoch = ckpt['epoch'] + 1
        self.best_val_acc = ckpt['best_val_acc']
        print(f"Resumed from epoch {self.start_epoch}, best_acc={self.best_val_acc:.4f}")
    
    def train(self, train_loader, val_loader, num_epochs):
        for epoch in range(self.start_epoch, num_epochs):
            # Train
            self.model.train()
            for X, y in train_loader:
                X, y = X.to(self.device), y.to(self.device)
                self.optimizer.zero_grad()
                loss = self.criterion(self.model(X), y)
                loss.backward()
                self.optimizer.step()
            
            self.scheduler.step()
            
            # Validate
            val_acc = self.evaluate(val_loader)
            is_best = val_acc > self.best_val_acc
            if is_best:
                self.best_val_acc = val_acc
            
            self.save_checkpoint(epoch, val_acc, is_best)
            print(f"Epoch {epoch+1}/{num_epochs} | Val Acc: {val_acc:.4f} | "
                  f"Best: {self.best_val_acc:.4f}")
    
    def evaluate(self, loader):
        self.model.eval()
        correct = total = 0
        with torch.no_grad():
            for X, y in loader:
                X, y = X.to(self.device), y.to(self.device)
                correct += (self.model(X).argmax(1) == y).sum().item()
                total += y.size(0)
        return correct / total

# Usage:
model = MLP()
optimizer = optim.AdamW(model.parameters(), lr=1e-3)
scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=50)

trainer = Trainer(model, optimizer, scheduler, nn.CrossEntropyLoss(), device)
trainer.load_checkpoint('latest')  # resume if exists
trainer.train(train_loader, val_loader, num_epochs=50)
```

---

## 9. Revision Questions

1. **Compare saving `state_dict` vs saving the entire model in PyTorch.** What are the advantages and risks of each approach?

2. **What should a training checkpoint contain to fully resume training?** List all components and explain why each is necessary.

3. **How do you load a PyTorch model that was saved on GPU onto a CPU machine?** Write the code and explain the `map_location` parameter.

4. **Explain the ONNX format and its purpose.** Write PyTorch code to export a model to ONNX and then run inference using ONNX Runtime.

5. **Compare TensorFlow's SavedModel, .keras, and .h5 formats.** When would you use each one?

6. **Design a model versioning strategy for a team of 5 ML engineers working on the same project.** Include naming conventions, what metadata to save, and which tools to use.

---

| [← 04 GPU Training](04-gpu-training.md) | [Unit 9 Home](README.md) | [Unit 10 → Building Neural Networks](../10-Building-Neural-Networks/) |
|:---|:---:|---:|

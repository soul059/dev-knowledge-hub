# 1. TensorFlow & Keras Basics

> **Unit 9 · Deep Learning Frameworks** — Building, training, and evaluating neural networks with the TensorFlow/Keras ecosystem

---

## Chapter Overview

**TensorFlow** is Google's open-source deep learning framework, and **Keras** is its high-level API that makes building neural networks feel like stacking LEGO blocks. Together, they power a huge portion of production AI systems worldwide — from Google Search to recommendation engines. This chapter covers the TensorFlow ecosystem, three ways to build models in Keras (Sequential, Functional, and Subclassing), the compile-fit-evaluate workflow, essential callbacks, and a complete end-to-end example.

---

## 1. TensorFlow Ecosystem Overview

```
  ┌──────────────────────────────────────────────────────────────┐
  │                   TensorFlow Ecosystem                       │
  │                                                              │
  │  ┌──────────────────────────────────────────────────────┐   │
  │  │                  tf.keras (High-Level API)            │   │
  │  │  Sequential │ Functional │ Subclassing               │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                         ↓                                    │
  │  ┌──────────────────────────────────────────────────────┐   │
  │  │              TensorFlow Core (Low-Level)              │   │
  │  │  tf.Tensor │ tf.Variable │ tf.GradientTape           │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                         ↓                                    │
  │  ┌──────────────────────────────────────────────────────┐   │
  │  │              Execution Engine                         │   │
  │  │  Eager Mode │ Graph Mode │ tf.function                │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                         ↓                                    │
  │  ┌──────────────────────────────────────────────────────┐   │
  │  │              Deployment                               │   │
  │  │  TF Serving │ TF Lite │ TF.js │ SavedModel           │   │
  │  └──────────────────────────────────────────────────────┘   │
  │                                                              │
  │  Supporting Libraries:                                       │
  │  • TensorFlow Hub (pretrained models)                       │
  │  • TensorFlow Datasets (ready-to-use datasets)              │
  │  • TensorBoard (visualization)                               │
  │  • TensorFlow Extended (TFX) (ML pipelines)                 │
  └──────────────────────────────────────────────────────────────┘
```

### Installation and Basic Import

```python
# pip install tensorflow

import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

print(f"TensorFlow version: {tf.__version__}")
print(f"GPU available: {tf.config.list_physical_devices('GPU')}")

# Tensors — similar to PyTorch/NumPy
x = tf.constant([[1.0, 2.0], [3.0, 4.0]])
print(x.shape)   # (2, 2)
print(x.dtype)   # float32
print(x.device)  # /job:localhost/replica:0/task:0/device:CPU:0
```

---

## 2. Keras Sequential API

The **simplest** way to build a model — stack layers linearly.

```
  Sequential Model = Linear Stack of Layers
  
  Input → [Layer 1] → [Layer 2] → [Layer 3] → Output
  
  Each layer has exactly ONE input and ONE output.
  No branching, no merging, no skip connections.
```

```python
from tensorflow.keras import Sequential
from tensorflow.keras.layers import Dense, Dropout, Flatten

# Method 1: Pass list of layers
model = Sequential([
    Flatten(input_shape=(28, 28)),       # 28×28 → 784
    Dense(256, activation='relu'),        # 784 → 256
    Dropout(0.3),                         # regularization
    Dense(128, activation='relu'),        # 256 → 128
    Dropout(0.3),                         # regularization
    Dense(10, activation='softmax'),      # 128 → 10 (probabilities)
])

# Method 2: Add layers one by one
model = Sequential()
model.add(Flatten(input_shape=(28, 28)))
model.add(Dense(256, activation='relu'))
model.add(Dropout(0.3))
model.add(Dense(128, activation='relu'))
model.add(Dropout(0.3))
model.add(Dense(10, activation='softmax'))

# View model summary
model.summary()
```

```
  model.summary() output:
  ┌──────────────────────────────────────────────────────────────┐
  │  Model: "sequential"                                         │
  │  ──────────────────────────────────────────────────────────  │
  │  Layer (type)               Output Shape       Param #      │
  │  ═══════════════════════════════════════════════════════════ │
  │  flatten (Flatten)          (None, 784)        0            │
  │  dense (Dense)              (None, 256)        200,960      │
  │  dropout (Dropout)          (None, 256)        0            │
  │  dense_1 (Dense)            (None, 128)        32,896       │
  │  dropout_1 (Dropout)        (None, 128)        0            │
  │  dense_2 (Dense)            (None, 10)         1,290        │
  │  ═══════════════════════════════════════════════════════════ │
  │  Total params: 235,146                                       │
  │  Trainable params: 235,146                                   │
  │  Non-trainable params: 0                                     │
  └──────────────────────────────────────────────────────────────┘
  
  Parameter calculation:
  Dense(256): 784 × 256 + 256 = 200,960  (weights + biases)
  Dense(128): 256 × 128 + 128 = 32,896
  Dense(10):  128 × 10  + 10  = 1,290
```

---

## 3. Keras Functional API

For models with **non-linear topology** — multiple inputs, multiple outputs, shared layers, skip connections.

```
  Functional API enables complex architectures:
  
  Example: Multi-input model
  
  Text Input ──→ [Embedding] ──→ [LSTM] ──┐
                                            ├──→ [Concat] → [Dense] → Output
  Meta Input ──→ [Dense] ─────────────────┘
```

```python
from tensorflow.keras import Model, Input
from tensorflow.keras.layers import Dense, Dropout, Concatenate

# Define inputs
inputs = Input(shape=(784,), name='input_features')

# Define forward pass
x = Dense(256, activation='relu', name='hidden1')(inputs)
x = Dropout(0.3)(x)
x = Dense(128, activation='relu', name='hidden2')(x)
x = Dropout(0.3)(x)
outputs = Dense(10, activation='softmax', name='output')(x)

# Create model
model = Model(inputs=inputs, outputs=outputs, name='mlp_functional')
model.summary()
```

### Multi-Input Example

```python
# Two different input sources merged together
input_image = Input(shape=(784,), name='image_input')
input_meta = Input(shape=(10,), name='metadata_input')

# Image branch
x1 = Dense(128, activation='relu')(input_image)
x1 = Dropout(0.3)(x1)
x1 = Dense(64, activation='relu')(x1)

# Metadata branch
x2 = Dense(32, activation='relu')(input_meta)

# Merge branches
merged = Concatenate()([x1, x2])
x = Dense(64, activation='relu')(merged)
outputs = Dense(10, activation='softmax')(x)

model = Model(inputs=[input_image, input_meta], outputs=outputs)

# Training with multiple inputs:
# model.fit([X_image, X_meta], y, ...)
```

### Skip Connection Example (Residual)

```python
inputs = Input(shape=(256,))
x = Dense(256, activation='relu')(inputs)
x = Dense(256)(x)  # no activation yet

# Skip connection!
from tensorflow.keras.layers import Add
x = Add()([x, inputs])  # residual: F(x) + x
x = tf.keras.activations.relu(x)

outputs = Dense(10, activation='softmax')(x)
model = Model(inputs, outputs)
```

---

## 4. Model Subclassing

Maximum flexibility — define model as a Python class with custom `call()` method.

```python
class MyMLP(tf.keras.Model):
    def __init__(self, hidden_size=256, num_classes=10, dropout_rate=0.3):
        super().__init__()
        self.flatten = Flatten()
        self.dense1 = Dense(hidden_size, activation='relu')
        self.dropout1 = Dropout(dropout_rate)
        self.dense2 = Dense(hidden_size // 2, activation='relu')
        self.dropout2 = Dropout(dropout_rate)
        self.classifier = Dense(num_classes, activation='softmax')
    
    def call(self, inputs, training=False):
        x = self.flatten(inputs)
        x = self.dense1(x)
        x = self.dropout1(x, training=training)  # dropout needs training flag!
        x = self.dense2(x)
        x = self.dropout2(x, training=training)
        return self.classifier(x)

model = MyMLP(hidden_size=256, num_classes=10)

# Must call model once to build it (creates weights)
dummy = tf.random.normal((1, 28, 28))
model(dummy)
model.summary()
```

### Comparison of Three APIs

```
  ┌──────────────────┬─────────────┬──────────────┬─────────────────┐
  │  Feature         │ Sequential  │ Functional   │ Subclassing     │
  ├──────────────────┼─────────────┼──────────────┼─────────────────┤
  │  Topology        │ Linear only │ Any DAG      │ Any (even loops)│
  │  Multiple I/O    │ No          │ Yes          │ Yes             │
  │  Skip connections│ No          │ Yes          │ Yes             │
  │  Custom logic    │ No          │ Limited      │ Full Python     │
  │  Debugging       │ Easy        │ Medium       │ Flexible        │
  │  Serialization   │ Easy        │ Easy         │ Requires work   │
  │  model.summary() │ Yes         │ Yes          │ Limited         │
  │  Best for        │ Simple MLPs │ Most models  │ Research/custom │
  ├──────────────────┼─────────────┼──────────────┼─────────────────┤
  │  Recommendation  │ Beginners   │ MOST CASES   │ Advanced/custom │
  └──────────────────┴─────────────┴──────────────┴─────────────────┘
```

---

## 5. Compiling the Model

Before training, you must **compile** the model — specify optimizer, loss, and metrics.

```python
model.compile(
    optimizer='adam',                           # or keras.optimizers.Adam(lr=1e-3)
    loss='sparse_categorical_crossentropy',    # for integer labels
    metrics=['accuracy']                        # track during training
)
```

### Common Loss Functions

```
  ┌──────────────────────────────────────────────────────────────┐
  │  TASK                    │ LOSS FUNCTION                     │
  ├──────────────────────────┼───────────────────────────────────┤
  │  Multi-class (int labels)│ 'sparse_categorical_crossentropy'│
  │  Multi-class (one-hot)   │ 'categorical_crossentropy'       │
  │  Binary classification   │ 'binary_crossentropy'            │
  │  Regression              │ 'mse' or 'mae'                   │
  └──────────────────────────┴───────────────────────────────────┘
  
  Note: If output activation is 'softmax', use from_logits=False (default).
  If output has NO activation (raw logits), use from_logits=True:
  
  loss = keras.losses.SparseCategoricalCrossentropy(from_logits=True)
```

### Optimizer Options

```python
# String shorthand (default settings):
model.compile(optimizer='adam', ...)

# Custom configuration:
model.compile(
    optimizer=keras.optimizers.Adam(
        learning_rate=1e-3,
        beta_1=0.9,
        beta_2=0.999,
        epsilon=1e-7,
        weight_decay=1e-4,  # if using AdamW
    ),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Other optimizers:
# keras.optimizers.SGD(learning_rate=0.01, momentum=0.9)
# keras.optimizers.RMSprop(learning_rate=0.001)
# keras.optimizers.AdamW(learning_rate=0.001, weight_decay=0.01)
```

---

## 6. Training with model.fit()

```python
history = model.fit(
    X_train, y_train,                  # Training data
    validation_data=(X_val, y_val),    # Validation data
    epochs=30,                          # Number of epochs
    batch_size=128,                     # Mini-batch size
    callbacks=[...],                    # Callbacks (see below)
    verbose=1,                          # Progress bar
)

# history.history is a dict with metrics per epoch:
# {
#   'loss': [2.30, 1.85, 1.21, ...],
#   'accuracy': [0.12, 0.45, 0.67, ...],
#   'val_loss': [2.28, 1.90, 1.35, ...],
#   'val_accuracy': [0.13, 0.42, 0.62, ...],
# }
```

### Plotting Training History

```python
import matplotlib.pyplot as plt

def plot_history(history):
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))
    
    # Loss
    ax1.plot(history.history['loss'], label='Train Loss')
    ax1.plot(history.history['val_loss'], label='Val Loss')
    ax1.set_xlabel('Epoch')
    ax1.set_ylabel('Loss')
    ax1.set_title('Loss Curves')
    ax1.legend()
    ax1.grid(True, alpha=0.3)
    
    # Accuracy
    ax2.plot(history.history['accuracy'], label='Train Accuracy')
    ax2.plot(history.history['val_accuracy'], label='Val Accuracy')
    ax2.set_xlabel('Epoch')
    ax2.set_ylabel('Accuracy')
    ax2.set_title('Accuracy Curves')
    ax2.legend()
    ax2.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('training_history.png', dpi=150)
    plt.show()

plot_history(history)
```

---

## 7. Essential Callbacks

Callbacks execute actions at specific points during training.

```
  Training Flow with Callbacks:
  
  on_train_begin()
  │
  for each epoch:
  │  on_epoch_begin()
  │  │
  │  for each batch:
  │  │  on_train_batch_begin()
  │  │  → forward + backward + update
  │  │  on_train_batch_end()
  │  │
  │  on_epoch_end()  ← most callbacks act here
  │
  on_train_end()
```

### EarlyStopping

```python
from tensorflow.keras.callbacks import EarlyStopping

early_stop = EarlyStopping(
    monitor='val_loss',      # metric to monitor
    patience=10,             # epochs to wait before stopping
    restore_best_weights=True,  # go back to best epoch's weights!
    min_delta=0.001,         # minimum change to count as improvement
    verbose=1
)

# Effect:
# Epoch 15: val_loss improved from 0.234 to 0.231 ✓
# Epoch 16: val_loss improved from 0.231 to 0.228 ✓
# Epoch 17: val_loss = 0.229 (no improvement)
# Epoch 18: val_loss = 0.230 (no improvement)
# ...
# Epoch 26: val_loss = 0.235 (no improvement for 10 epochs)
# → STOP! Restore weights from epoch 16.
```

### ModelCheckpoint

```python
from tensorflow.keras.callbacks import ModelCheckpoint

checkpoint = ModelCheckpoint(
    filepath='best_model.keras',   # save path
    monitor='val_accuracy',        # metric to monitor
    save_best_only=True,           # only save when metric improves
    save_weights_only=False,       # save full model (not just weights)
    verbose=1
)
```

### TensorBoard Callback

```python
from tensorflow.keras.callbacks import TensorBoard

tensorboard = TensorBoard(
    log_dir='./logs',
    histogram_freq=1,        # log weight histograms every epoch
    write_graph=True,        # log computation graph
    write_images=True,       # log weight images
    update_freq='epoch',     # log metrics per epoch
)

# Launch TensorBoard:
# tensorboard --logdir=./logs --port=6006
```

### ReduceLROnPlateau

```python
from tensorflow.keras.callbacks import ReduceLROnPlateau

reduce_lr = ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.5,             # multiply LR by this factor
    patience=5,             # epochs to wait before reducing
    min_lr=1e-7,            # minimum learning rate
    verbose=1
)
```

### Using All Callbacks Together

```python
callbacks = [
    EarlyStopping(monitor='val_loss', patience=10, restore_best_weights=True),
    ModelCheckpoint('best_model.keras', monitor='val_accuracy', save_best_only=True),
    ReduceLROnPlateau(monitor='val_loss', factor=0.5, patience=5, min_lr=1e-7),
    TensorBoard(log_dir='./logs'),
]

history = model.fit(
    X_train, y_train,
    validation_data=(X_val, y_val),
    epochs=100,  # set high — early stopping will handle it
    batch_size=128,
    callbacks=callbacks
)
```

---

## 8. Evaluating the Model

```python
# Evaluate on test set
test_loss, test_accuracy = model.evaluate(X_test, y_test, verbose=0)
print(f"Test Loss: {test_loss:.4f}")
print(f"Test Accuracy: {test_accuracy:.4f}")

# Predictions
predictions = model.predict(X_test)          # probabilities (batch_size, 10)
predicted_classes = predictions.argmax(axis=1)  # class labels

# Confusion matrix
from sklearn.metrics import classification_report, confusion_matrix
print(classification_report(y_test, predicted_classes))
```

---

## 9. Complete Example: MNIST MLP

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras.callbacks import EarlyStopping, ModelCheckpoint, ReduceLROnPlateau
import numpy as np

# ═══════════════════════════════════════════════════════════
#  1. LOAD AND PREPARE DATA
# ═══════════════════════════════════════════════════════════

(X_train_full, y_train_full), (X_test, y_test) = keras.datasets.mnist.load_data()

# Normalize to [0, 1]
X_train_full = X_train_full.astype('float32') / 255.0
X_test = X_test.astype('float32') / 255.0

# Split train into train + validation
X_train, X_val = X_train_full[:50000], X_train_full[50000:]
y_train, y_val = y_train_full[:50000], y_train_full[50000:]

print(f"Training:   {X_train.shape}, {y_train.shape}")
print(f"Validation: {X_val.shape}, {y_val.shape}")
print(f"Test:       {X_test.shape}, {y_test.shape}")

# ═══════════════════════════════════════════════════════════
#  2. BUILD MODEL (Functional API)
# ═══════════════════════════════════════════════════════════

inputs = keras.Input(shape=(28, 28), name='digit_image')
x = layers.Flatten()(inputs)
x = layers.Dense(256, activation='relu')(x)
x = layers.BatchNormalization()(x)
x = layers.Dropout(0.3)(x)
x = layers.Dense(128, activation='relu')(x)
x = layers.BatchNormalization()(x)
x = layers.Dropout(0.3)(x)
outputs = layers.Dense(10, activation='softmax', name='predictions')(x)

model = keras.Model(inputs=inputs, outputs=outputs, name='mnist_mlp')
model.summary()

# ═══════════════════════════════════════════════════════════
#  3. COMPILE
# ═══════════════════════════════════════════════════════════

model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# ═══════════════════════════════════════════════════════════
#  4. CALLBACKS
# ═══════════════════════════════════════════════════════════

callbacks = [
    EarlyStopping(
        monitor='val_loss', patience=10, 
        restore_best_weights=True, verbose=1
    ),
    ModelCheckpoint(
        'mnist_best.keras', monitor='val_accuracy',
        save_best_only=True, verbose=1
    ),
    ReduceLROnPlateau(
        monitor='val_loss', factor=0.5,
        patience=5, min_lr=1e-6, verbose=1
    ),
]

# ═══════════════════════════════════════════════════════════
#  5. TRAIN
# ═══════════════════════════════════════════════════════════

history = model.fit(
    X_train, y_train,
    validation_data=(X_val, y_val),
    epochs=100,
    batch_size=128,
    callbacks=callbacks,
    verbose=1
)

# ═══════════════════════════════════════════════════════════
#  6. EVALUATE
# ═══════════════════════════════════════════════════════════

test_loss, test_acc = model.evaluate(X_test, y_test, verbose=0)
print(f"\nTest Loss:     {test_loss:.4f}")
print(f"Test Accuracy: {test_acc:.4f}")

# ═══════════════════════════════════════════════════════════
#  7. PREDICTIONS
# ═══════════════════════════════════════════════════════════

predictions = model.predict(X_test[:5])
for i in range(5):
    pred_class = np.argmax(predictions[i])
    confidence = predictions[i][pred_class]
    true_class = y_test[i]
    status = "✓" if pred_class == true_class else "✗"
    print(f"  {status} Predicted: {pred_class} (conf: {confidence:.3f}), True: {true_class}")
```

---

## 10. Summary Table

| Concept | Keras Implementation | Key Point |
|---------|---------------------|-----------|
| **Build (Sequential)** | `Sequential([Dense(...), ...])` | Linear stack, simplest |
| **Build (Functional)** | `Model(inputs, outputs)` | Any DAG topology |
| **Build (Subclass)** | `class MyModel(tf.keras.Model)` | Full flexibility |
| **Compile** | `model.compile(optimizer, loss, metrics)` | Required before training |
| **Train** | `model.fit(X, y, epochs, callbacks)` | Returns history object |
| **Evaluate** | `model.evaluate(X_test, y_test)` | Returns loss + metrics |
| **Predict** | `model.predict(X)` | Returns probabilities |
| **EarlyStopping** | `EarlyStopping(patience=10)` | Prevents overfitting |
| **ModelCheckpoint** | `ModelCheckpoint(save_best_only=True)` | Saves best model |
| **ReduceLROnPlateau** | `ReduceLROnPlateau(factor=0.5)` | Adaptive LR |

---

## 11. Revision Questions

1. **Compare the three Keras model-building APIs (Sequential, Functional, Subclassing).** When would you use each one? Which is recommended for most cases?

2. **What does `model.compile()` do, and what are the three required arguments?** Why must compilation happen before training?

3. **Explain the EarlyStopping callback.** What does `restore_best_weights=True` do, and why is it important?

4. **Write a complete Keras program** that builds an MLP with 2 hidden layers (256 and 128 neurons), trains it on MNIST, uses EarlyStopping and ModelCheckpoint, and evaluates on the test set.

5. **What is the difference between `sparse_categorical_crossentropy` and `categorical_crossentropy`?** When do you use each?

6. **Explain how the ReduceLROnPlateau callback works.** What are `factor`, `patience`, and `min_lr`? Give an example scenario showing LR changes over epochs.

---

| [← Unit 8: Practical Training](../08-Practical-Training/) | [Unit 9 Home](README.md) | [02 → PyTorch Basics](02-pytorch-basics.md) |
|:---|:---:|---:|

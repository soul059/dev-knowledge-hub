# Chapter 4: Debugging in Jupyter Notebooks

> **Unit 6 · Jupyter Notebooks** — Techniques and tools for finding and fixing bugs in interactive notebook environments.

---

## 1 · The Debugging Challenge in Notebooks

Notebooks introduce unique debugging challenges compared to scripts:

| Challenge | Why It Matters |
|---|---|
| **Non-linear execution** | Cells can be run in any order, creating hidden state |
| **Persistent variables** | Deleted cells still leave variables in memory |
| **No traditional breakpoints** | You can't set breakpoints like in an IDE (without tools) |
| **Long-running cells** | A bug deep in a pipeline wastes minutes of re-execution |

---

## 2 · Print Debugging (Level 1)

The simplest approach — add `print()` statements to inspect values at key points.

### 2.1 — Basic Print Debugging

```python
def process_data(df):
    print(f"Input shape: {df.shape}")                     # Check input
    
    df = df.dropna()
    print(f"After dropna: {df.shape}")                    # Check transformation
    
    df['ratio'] = df['revenue'] / df['costs']
    print(f"Ratio range: {df['ratio'].min():.2f} – {df['ratio'].max():.2f}")  # Check values
    print(f"Any infinities? {df['ratio'].isin([float('inf')]).any()}")         # Check edge cases
    
    return df
```

### 2.2 — f-string Debugging Shortcut (Python 3.8+)

```python
x = 42
y = [1, 2, 3]
name = "model_v2"

# The = sign after the variable prints both the name and value
print(f"{x = }")          # x = 42
print(f"{y = }")          # y = [1, 2, 3]
print(f"{name = }")       # name = 'model_v2'
print(f"{len(y) = }")     # len(y) = 3
```

### 2.3 — Conditional Printing for Loops

```python
for i, batch in enumerate(data_loader):
    loss = train_step(batch)
    
    if i % 100 == 0:           # Print every 100 steps
        print(f"Step {i:5d} | Loss: {loss:.4f}")
    
    if loss > 10:              # Alert on anomalies
        print(f"⚠️ HIGH LOSS at step {i}: {loss:.4f}")
        print(f"  Batch stats: mean={batch.mean():.4f}, std={batch.std():.4f}")
```

---

## 3 · IPython's `%debug` Magic (Level 2)

### 3.1 — Post-Mortem Debugging

When a cell raises an exception, run `%debug` in the **next cell** to open an interactive debugger **at the point of failure**.

```python
# Cell 1 — This raises an error
def divide_values(a, b):
    return a / b

result = divide_values(10, 0)   # ZeroDivisionError!
```

```python
# Cell 2 — Debug the error
%debug
```

This drops you into an interactive `pdb` session at the exact line that failed. You can inspect variables, move up/down the call stack, and understand the error.

### 3.2 — `%%debug` Cell Magic

Debug a cell **as it runs** (not post-mortem). This sets a breakpoint at the first line.

```python
%%debug
x = 10
y = compute_something(x)   # Step through this line by line
z = x + y
```

---

## 4 · Automatic Debugging with `%pdb`

Toggle automatic post-mortem debugging — whenever an exception occurs, `pdb` launches automatically.

```python
%pdb on     # Enable: auto-debug on any exception
%pdb off    # Disable: return to normal behavior
%pdb        # Toggle current state
```

```python
%pdb on

# Now ANY unhandled exception will drop you into the debugger automatically
result = 1 / 0    # → Drops straight into pdb
```

---

## 5 · Essential `pdb` Commands

Once inside the debugger (`ipdb>` or `pdb>` prompt), use these commands:

| Command | Full Name | Action |
|---|---|---|
| `n` | next | Execute the next line (step **over** function calls) |
| `s` | step | Step **into** a function call |
| `c` | continue | Continue execution until the next breakpoint or end |
| `p expr` | print | Print the value of an expression |
| `pp expr` | pretty-print | Pretty-print the value |
| `q` | quit | Quit the debugger |
| `l` | list | Show source code around the current line |
| `l 1, 20` | list range | Show lines 1 through 20 |
| `w` | where | Print the full call stack (traceback) |
| `u` | up | Move up one frame in the call stack |
| `d` | down | Move down one frame in the call stack |
| `b N` | breakpoint | Set a breakpoint at line N |
| `a` | args | Print the arguments of the current function |
| `r` | return | Continue until the current function returns |
| `h` | help | Show help |

### 5.1 — Step-by-Step Debugging Example

```python
def compute_metrics(y_true, y_pred):
    n = len(y_true)
    errors = [t - p for t, p in zip(y_true, y_pred)]
    mse = sum(e**2 for e in errors) / n
    rmse = mse ** 0.5
    return rmse

# This crashes if lists are different lengths
compute_metrics([1, 2, 3], [1, 2])
```

```
# In the next cell:
%debug

ipdb> w                    # Show call stack
ipdb> p y_true             # [1, 2, 3]
ipdb> p y_pred             # [1, 2]
ipdb> p len(y_true)        # 3
ipdb> p len(y_pred)        # 2  ← Found the bug! Different lengths
ipdb> p errors             # [0, 0]  ← zip silently truncated
ipdb> q                    # Quit debugger
```

---

## 6 · Rich Output with `IPython.display`

Use `IPython.display` to visualize data during debugging.

```python
from IPython.display import display, HTML, Markdown, Image, JSON

# Display multiple DataFrames side by side
display(HTML("<h3>Training Data</h3>"))
display(train_df.head())
display(HTML("<h3>Test Data</h3>"))
display(test_df.head())

# Render Markdown programmatically
display(Markdown("## ✅ All checks passed!"))
display(Markdown(f"- Rows: **{len(df):,}**\n- Columns: **{df.shape[1]}**"))

# Show an image
display(Image(filename="confusion_matrix.png", width=400))

# Display formatted JSON
display(JSON({"accuracy": 0.95, "f1": 0.93, "params": {"lr": 0.01}}))
```

### 6.1 — Displaying Multiple Objects

```python
# By default, only the LAST expression in a cell is displayed
# Enable displaying all expressions:
from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"

# Now all these are displayed:
df.shape
df.dtypes
df.head()
```

---

## 7 · Logging in Notebooks

For production-quality debugging, use Python's `logging` module instead of `print()`.

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s [%(levelname)s] %(message)s',
    datefmt='%H:%M:%S'
)
logger = logging.getLogger(__name__)

def train_model(X, y, epochs=10):
    logger.info(f"Starting training: {X.shape[0]} samples, {epochs} epochs")
    
    for epoch in range(epochs):
        loss = fit_one_epoch(X, y)
        
        if loss < 0:
            logger.warning(f"Negative loss at epoch {epoch}: {loss}")
        
        logger.debug(f"Epoch {epoch}: loss={loss:.4f}")
    
    logger.info("Training complete")
```

### 7.1 — Log Levels

| Level | When to Use |
|---|---|
| `DEBUG` | Detailed diagnostic info (variable values, loop iterations) |
| `INFO` | General progress updates (start/end of major steps) |
| `WARNING` | Something unexpected but not fatal |
| `ERROR` | A failure that prevents part of the task |
| `CRITICAL` | A failure that stops everything |

---

## 8 · Debugging Data Pipelines

### 8.1 — Shape Tracking Pattern

```python
def debug_pipeline(df):
    """Track DataFrame shape through each transformation."""
    steps = []
    
    def track(step_name, dataframe):
        steps.append((step_name, dataframe.shape))
        print(f"  [{step_name}] → {dataframe.shape}")
        return dataframe
    
    print("Pipeline execution:")
    df = track("raw", df)
    df = track("drop_nulls", df.dropna())
    df = track("filter_active", df[df['status'] == 'active'])
    df = track("drop_dupes", df.drop_duplicates())
    
    return df, steps
```

### 8.2 — Assertion-Based Debugging

```python
def validate_dataframe(df, step_name=""):
    """Run sanity checks after each transformation."""
    prefix = f"[{step_name}] " if step_name else ""
    
    assert not df.empty, f"{prefix}DataFrame is empty!"
    assert not df.columns.duplicated().any(), f"{prefix}Duplicate column names!"
    
    null_pct = df.isnull().mean()
    high_null = null_pct[null_pct > 0.5]
    if len(high_null) > 0:
        print(f"⚠️ {prefix}Columns with >50% nulls: {list(high_null.index)}")
    
    print(f"✅ {prefix}Shape: {df.shape}, Nulls: {df.isnull().sum().sum()}")

# Usage
df = pd.read_csv("data.csv")
validate_dataframe(df, "after_load")

df = df.dropna(subset=['target'])
validate_dataframe(df, "after_dropna")
```

---

## 9 · Common Pitfalls and How to Avoid Them

### 9.1 — Hidden State

```python
# ❌ Problem: You delete a cell that defined `temp_var`, but it still exists in memory
# ✅ Solution: Regularly run Kernel → Restart & Run All

# ❌ Problem: You run Cell 5 before Cell 3, and get unexpected results
# ✅ Solution: Check In[n] numbers — they should be sequential
```

### 9.2 — Execution Order Bugs

```python
# Cell 1 — In [1]
threshold = 0.5

# Cell 2 — In [3] (ran AFTER Cell 3!)
results = df[df['score'] > threshold]   # Uses threshold=0.8, not 0.5!

# Cell 3 — In [2]
threshold = 0.8    # Overrides threshold, but Cell 2 was already run with 0.5
```

**Best practice:** After finishing your analysis, always run **Kernel → Restart & Run All** to verify everything works in order.

### 9.3 — Mutable Default Arguments

```python
# ❌ Bug: default list is shared across calls
def add_item(item, items=[]):
    items.append(item)
    return items

# ✅ Fix: use None as default
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

---

## 10 · Variable Inspector

### 10.1 — Built-in Approaches

```python
# Quick check of all variables
%whos

# Check if a specific variable exists
'my_var' in dir()

# Check memory usage of variables
import sys
for var_name in dir():
    var = eval(var_name)
    size = sys.getsizeof(var)
    if size > 1_000_000:    # >1MB
        print(f"{var_name}: {size / 1e6:.1f} MB")
```

### 10.2 — JupyterLab Variable Inspector Extension

```bash
# Install the variable inspector extension
pip install lckr-jupyterlab-variableinspector
```

Once installed, right-click in a notebook and select **"Open Variable Inspector"** — it shows a live table of all variables, their types, sizes, and values.

### 10.3 — Memory Cleanup

```python
# Delete a specific variable
del large_dataframe

# Delete all variables (nuclear option)
%reset -f

# Garbage collection
import gc
gc.collect()
```

---

## Summary Table

| Technique | When to Use | Command / Pattern |
|---|---|---|
| Print debugging | Quick inspection | `print(f"{var = }")` |
| `%debug` | After an exception | Run `%debug` in next cell |
| `%%debug` | Step through a cell | Put `%%debug` at cell top |
| `%pdb on` | Auto-debug all errors | Toggle at notebook start |
| pdb commands | Inside debugger | `n`, `s`, `c`, `p`, `q`, `l`, `w` |
| `IPython.display` | Rich visual debugging | `display(df)`, `display(HTML(...))` |
| `logging` | Production-quality logs | `logger.info(...)`, `logger.debug(...)` |
| Shape tracking | Data pipeline debugging | Print shape after each step |
| Assertions | Validate assumptions | `assert not df.empty` |
| `%whos` | Variable inspection | Shows all variables with types |
| Restart & Run All | Verify correctness | Kernel menu or shortcut |

---

## Revision Questions

1. **Explain the difference between `%debug` and `%pdb`. When would you use each?**
2. **You are inside the `pdb` debugger. What do the commands `n`, `s`, and `c` do? How are they different?**
3. **What is 'hidden state' in a notebook, and how can it cause bugs? How do you prevent it?**
4. **Write a pipeline validation function that checks DataFrame shape, null counts, and duplicate rows after each transformation step.**
5. **Why is `logging` preferred over `print()` for debugging in production notebooks?**
6. **How can you display all expression results in a cell, not just the last one?**

---

[← Chapter 3: Magic Commands](./03-magic-commands.md) · [Next → Chapter 5: Best Practices](./05-best-practices.md)

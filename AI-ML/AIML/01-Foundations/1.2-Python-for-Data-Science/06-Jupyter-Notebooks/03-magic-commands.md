# Chapter 3: Magic Commands

> **Unit 6 · Jupyter Notebooks** — Leveraging IPython magic commands and shell integration to supercharge your notebook workflow.

---

## 1 · What Are Magic Commands?

Magic commands are special commands provided by **IPython** (the kernel behind Jupyter Python notebooks). They are prefixed with `%` or `%%` and provide shortcuts for common tasks that would otherwise require boilerplate code.

| Type | Prefix | Scope | Example |
|---|---|---|---|
| **Line magic** | `%` | Operates on a single line | `%timeit x = 1 + 1` |
| **Cell magic** | `%%` | Operates on the entire cell | `%%timeit` (times the whole cell) |

> 💡 **Tip:** If `%automagic` is on (default), you can omit the `%` for line magics — e.g., just type `timeit x = 1`. But explicit `%` is recommended for clarity.

---

## 2 · Discovering Available Magics

### 2.1 — List All Magics

```python
%lsmagic
```

**Output:**
```
Available line magics:
%alias  %cd  %debug  %env  %load  %matplotlib  %pip  %pwd  %run  %time  %timeit  %who  %whos  ...

Available cell magics:
%%bash  %%capture  %%html  %%javascript  %%timeit  %%writefile  ...
```

### 2.2 — Get Help on Any Magic

```python
%timeit?          # Show docstring
%timeit??         # Show source code (if available)
```

---

## 3 · Essential Line Magics (`%`)

### 3.1 — `%timeit` — Benchmark a Single Line

Runs the statement multiple times and reports the **average ± standard deviation**.

```python
%timeit sorted(range(1000))
# 45.2 µs ± 1.3 µs per loop (mean ± std. dev. of 7 runs, 10,000 loops each)

%timeit -n 100 -r 3 sorted(range(1000))
# 100 loops, 3 runs — useful for expensive operations
```

### 3.2 — `%time` — Time a Single Execution

Runs the statement **once** and reports wall time and CPU time.

```python
%time result = sum(range(10_000_000))
# CPU times: user 312 ms, sys: 15.6 ms, total: 328 ms
# Wall time: 330 ms
```

**When to use which:**
| Command | Use When |
|---|---|
| `%timeit` | You need a reliable average (micro-benchmarks) |
| `%time` | You want a quick one-shot measurement |

### 3.3 — `%run` — Execute a Python Script

```python
# Run a script as if you typed it into the notebook
%run my_script.py

# Run with arguments
%run train_model.py --epochs 10 --lr 0.001

# Run and make variables available in notebook namespace
%run -i preprocessing.py
```

### 3.4 — `%load` — Load File Contents into a Cell

```python
# Loads the content of the file INTO the current cell (replaces this line)
%load utils.py

# Load specific lines
%load -r 10-25 utils.py
```

After execution, the `%load` line is commented out and replaced with the file contents.

### 3.5 — `%who` and `%whos` — Inspect Variables

```python
%who             # List all variables
# x    y    df    model    scaler

%who str         # List only string variables
# name    city

%whos            # Detailed table with type and value
# Variable   Type         Data/Info
# --------   ----         ---------
# x          int          42
# df         DataFrame    100 rows × 5 columns
# model      LogisticReg  LogisticRegression()
```

### 3.6 — `%pwd` and `%cd` — Directory Navigation

```python
%pwd
# 'C:\\Users\\keval\\projects\\ml-project'

%cd data/
# C:\Users\keval\projects\ml-project\data

%cd ..
# C:\Users\keval\projects\ml-project
```

### 3.7 — `%env` — Environment Variables

```python
# List all environment variables
%env

# Get a specific variable
%env HOME

# Set an environment variable
%env MY_API_KEY=abc123
```

### 3.8 — `%matplotlib` — Plot Backend Configuration

```python
# Render plots inline (most common for notebooks)
%matplotlib inline

# Interactive plots (zoom, pan) in JupyterLab
%matplotlib widget

# Separate window (rare in notebooks)
%matplotlib qt
```

> ⚠️ Run `%matplotlib inline` **before** any plotting code — typically in the first cell.

### 3.9 — `%pip` — Install Packages

```python
# Install into the SAME environment as the running kernel
%pip install scikit-learn

# Upgrade a package
%pip install --upgrade pandas

# Install from requirements
%pip install -r requirements.txt
```

> 💡 Prefer `%pip install` over `!pip install` — the magic version ensures the package goes into the correct kernel environment.

---

## 4 · Essential Cell Magics (`%%`)

### 4.1 — `%%timeit` — Benchmark an Entire Cell

```python
%%timeit
data = list(range(10000))
sorted_data = sorted(data)
total = sum(sorted_data)
# 1.23 ms ± 45.6 µs per loop (mean ± std. dev. of 7 runs, 1,000 loops each)
```

### 4.2 — `%%writefile` — Write Cell Contents to a File

```python
%%writefile utils.py
def clean_text(text):
    """Remove extra whitespace and lowercase."""
    return ' '.join(text.lower().split())

def load_config(path):
    import json
    with open(path) as f:
        return json.load(f)
```

```
Writing utils.py
```

This is perfect for creating helper modules directly from your notebook.

### 4.3 — `%%bash` — Run Shell Commands

```python
%%bash
echo "Current directory:"
pwd
echo "Python version:"
python --version
echo "Disk usage:"
df -h | head -5
```

> On Windows, use `%%cmd` or `%%powershell` instead, or prefix with `!`.

### 4.4 — `%%html` — Render HTML

```python
%%html
<div style="text-align:center; padding:20px; background:#f0f8ff; border-radius:8px;">
    <h2 style="color:#2c3e50;">🎯 Model Performance Dashboard</h2>
    <p style="font-size:24px; color:#27ae60;">Accuracy: <b>93.2%</b></p>
</div>
```

### 4.5 — `%%javascript` — Execute JavaScript

```python
%%javascript
// Show a browser alert
alert("Training complete!");

// Access notebook metadata
console.log(Jupyter.notebook.get_cells().length + " cells in this notebook");
```

### 4.6 — `%%capture` — Suppress or Capture Output

```python
%%capture captured_output
import warnings
warnings.warn("This is a test warning")
print("This output is captured, not displayed")

# Later, access the captured output:
print(captured_output.stdout)
print(captured_output.stderr)
```

Useful for silencing verbose library imports or capturing logs for later analysis.

---

## 5 · The Autoreload Extension

When you edit external `.py` files, normally you have to restart the kernel. The **autoreload** extension reloads modules automatically.

```python
# Load the extension
%load_ext autoreload

# Set autoreload mode
%autoreload 2    # Reload ALL modules before every cell execution
```

| Mode | Behavior |
|---|---|
| `%autoreload 0` | Disable autoreload |
| `%autoreload 1` | Reload only modules imported with `%aimport` |
| `%autoreload 2` | Reload ALL imported modules (recommended) |
| `%autoreload 3` | Reload all modules AND update existing objects in place |

### Typical Workflow

```python
# Cell 1 — run once at the top of the notebook
%load_ext autoreload
%autoreload 2

# Cell 2
from mymodule import preprocess, train_model

# Now edit mymodule.py in your editor...
# Next time you run a cell, changes are picked up automatically!
```

---

## 6 · Shell Commands with `!`

Prefix any line with `!` to run it as a shell command.

```python
# List files
!ls -la data/

# Check Python version
!python --version

# Capture shell output into a Python variable
files = !ls data/*.csv
print(files)
# ['data/train.csv', 'data/test.csv', 'data/validation.csv']

# Use Python variables in shell commands (with $)
filename = "output.txt"
!echo "Hello" > {filename}
!cat {filename}
```

### `!` vs `%` — When to Use Which

| Need | Use | Example |
|---|---|---|
| Run a shell command | `!` | `!ls`, `!git status` |
| Install a package | `%pip` | `%pip install numpy` |
| Change directory (persistent) | `%cd` | `%cd data/` |
| Change directory (one-off) | `!` | `!cd data && ls` (does NOT persist) |
| Benchmark code | `%timeit` | `%timeit sorted(data)` |

> ⚠️ `!cd some_dir` does **NOT** change the notebook's working directory — use `%cd` instead.

---

## Summary Table

| Magic | Type | Purpose |
|---|---|---|
| `%timeit` / `%%timeit` | Line / Cell | Benchmark execution time (averaged) |
| `%time` | Line | Single-run timing |
| `%run` | Line | Execute an external `.py` script |
| `%load` | Line | Load file contents into a cell |
| `%who` / `%whos` | Line | List / inspect workspace variables |
| `%pwd` / `%cd` | Line | Print / change working directory |
| `%env` | Line | Get or set environment variables |
| `%matplotlib inline` | Line | Enable inline plotting |
| `%pip install` | Line | Install packages in kernel environment |
| `%%writefile` | Cell | Save cell contents to a file |
| `%%bash` | Cell | Run cell as a bash script |
| `%%html` | Cell | Render cell as HTML |
| `%%javascript` | Cell | Execute cell as JavaScript |
| `%%capture` | Cell | Capture/suppress cell output |
| `%load_ext autoreload` | Line | Auto-reload edited external modules |
| `%lsmagic` | Line | List all available magic commands |

---

## Revision Questions

1. **What is the difference between `%timeit` and `%time`? When would you choose one over the other?**
2. **How does `%pip install` differ from `!pip install`? Why is the magic version recommended?**
3. **Write a cell that saves a Python function to a file called `helpers.py` using a magic command.**
4. **Explain what `%autoreload 2` does and describe a workflow where it is essential.**
5. **How can you capture shell command output into a Python variable?**
6. **Why does `!cd some_directory` not change the notebook's working directory? What should you use instead?**

---

[← Chapter 2: Markdown & Documentation](./02-markdown-and-documentation.md) · [Next → Chapter 4: Debugging](./04-debugging.md)

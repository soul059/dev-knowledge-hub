# Chapter 1: The Jupyter Notebook Environment

> **Unit 6 · Jupyter Notebooks** — Setting up, navigating, and mastering the interactive computing environment that powers modern data science.

---

## 1 · What Is Jupyter Notebook?

Jupyter Notebook is an open-source web application that lets you create documents containing **live code, equations, visualizations, and narrative text**. The name "Jupyter" comes from three core languages: **Ju**lia, **Py**thon, and **R**.

| Feature | Description |
|---|---|
| Interactive execution | Run code cell-by-cell and see results immediately |
| Rich output | Display plots, tables, images, HTML, and LaTeX inline |
| Reproducibility | Share `.ipynb` files that anyone can re-run |
| Language-agnostic | Supports 40+ languages via different kernels |

---

## 2 · Installation

### 2.1 — Using pip

```bash
# Install classic notebook
pip install notebook

# Install JupyterLab (recommended modern interface)
pip install jupyterlab
```

### 2.2 — Using conda

```bash
# Classic notebook
conda install -c conda-forge notebook

# JupyterLab
conda install -c conda-forge jupyterlab
```

### 2.3 — Verify Installation

```bash
jupyter --version
# Should show jupyter core, notebook, lab versions
```

---

## 3 · Starting the Notebook Server

```bash
# Classic Notebook
jupyter notebook

# JupyterLab
jupyter lab

# Specify port and no-browser
jupyter notebook --port 8890 --no-browser

# Open a specific notebook
jupyter notebook my_analysis.ipynb
```

The server starts at `http://localhost:8888` by default. A **token** is printed in the terminal for authentication.

---

## 4 · Interface Overview

### 4.1 — Classic Notebook UI

```
┌──────────────────────────────────────────────────┐
│  File  Edit  View  Insert  Cell  Kernel  Help    │  ← Menu Bar
├──────────────────────────────────────────────────┤
│  💾  ✂️  📋  ▶️  ⏹  ⟳  │ Code ▼│              │  ← Toolbar
├──────────────────────────────────────────────────┤
│  In [1]: import pandas as pd                     │  ← Code Cell
│          df = pd.read_csv("data.csv")            │
├──────────────────────────────────────────────────┤
│  Out[1]: <DataFrame 100 rows × 5 columns>       │  ← Output
├──────────────────────────────────────────────────┤
│  ## Analysis Summary                             │  ← Markdown Cell
│  This section covers ...                         │
└──────────────────────────────────────────────────┘
```

### 4.2 — Dashboard (File Browser)

When you launch `jupyter notebook`, the dashboard shows your file system. You can:
- Create new notebooks (`New → Python 3`)
- Upload files
- Open terminals
- Manage running notebooks under the **Running** tab

---

## 5 · Cell Types

### 5.1 — Code Cells

```python
# This is a code cell — it runs Python (or whichever kernel is active)
import numpy as np
data = np.random.randn(1000)
print(f"Mean: {data.mean():.4f}, Std: {data.std():.4f}")
```

### 5.2 — Markdown Cells

Used for documentation, explanations, and formatting (see Chapter 2 for full details).

```markdown
## Section Title
This cell uses **Markdown** to create formatted text.
```

### 5.3 — Raw Cells

Raw cells are **not executed** and **not rendered**. They pass content directly to `nbconvert` for format-specific output (e.g., LaTeX commands for PDF export).

```
\chapter{Introduction}    ← Only meaningful during nbconvert to LaTeX
```

---

## 6 · Kernel Management

The **kernel** is the computational engine that executes your code. For Python notebooks, the kernel is an IPython process.

| Action | Menu Path | What It Does |
|---|---|---|
| **Interrupt** | Kernel → Interrupt | Stops the currently running cell (like Ctrl+C) |
| **Restart** | Kernel → Restart | Kills the kernel and starts fresh — all variables are lost |
| **Restart & Clear** | Kernel → Restart & Clear Output | Restart + remove all cell outputs |
| **Restart & Run All** | Kernel → Restart & Run All | Clean-slate execution of every cell top-to-bottom |
| **Change kernel** | Kernel → Change kernel | Switch to another installed kernel (e.g., R, Julia) |

### Example: Installing a New Kernel

```bash
# Create a virtual environment and register its kernel
python -m venv myenv
myenv\Scripts\activate          # Windows
pip install ipykernel
python -m ipykernel install --user --name=myenv --display-name "My Project Env"
```

---

## 7 · Keyboard Shortcuts

Jupyter has two modes: **Command mode** (press `Esc`) and **Edit mode** (press `Enter`).

### 7.1 — Essential Shortcuts

| Shortcut | Mode | Action |
|---|---|---|
| `Shift + Enter` | Both | Run cell, move to next |
| `Ctrl + Enter` | Both | Run cell, stay in place |
| `Alt + Enter` | Both | Run cell, insert new cell below |
| `Esc` | Edit → Cmd | Enter command mode |
| `Enter` | Cmd → Edit | Enter edit mode |
| `A` | Command | Insert cell **above** |
| `B` | Command | Insert cell **below** |
| `DD` | Command | Delete cell |
| `Z` | Command | Undo cell delete |
| `M` | Command | Change cell to Markdown |
| `Y` | Command | Change cell to Code |
| `L` | Command | Toggle line numbers |
| `Shift + M` | Command | Merge selected cells |
| `Ctrl + Shift + -` | Edit | Split cell at cursor |
| `H` | Command | Show all shortcuts |

### 7.2 — Productivity Tip

```
Press Esc → A → Enter       →  Quickly insert a cell above and start typing
Press Esc → B → Enter       →  Quickly insert a cell below and start typing
Press Esc → D → D           →  Delete current cell (double-tap D)
```

---

## 8 · Cell Execution Order and Kernel State

### 8.1 — Execution Numbers

Each code cell shows `In [n]:` where `n` is the **execution order**. This number tells you *when* the cell was last run, not its position.

```python
# Cell 1 — run first        In [1]:
x = 10

# Cell 2 — run second       In [2]:
y = x + 5

# Cell 1 — run again        In [3]:  ← execution number updated!
x = 99

# Cell 2 — still shows      In [2]:  ← y is still 15, NOT 104!
```

### 8.2 — Hidden State Pitfall

> ⚠️ Variables persist in memory even if you delete the cell that created them. Always use **Kernel → Restart & Run All** to verify your notebook works top-to-bottom.

---

## 9 · JupyterLab vs Classic Notebook

| Feature | Classic Notebook | JupyterLab |
|---|---|---|
| Interface | Single document | Multi-panel IDE |
| Tabs | ❌ | ✅ Multiple notebooks side-by-side |
| File browser | Basic dashboard | Integrated sidebar |
| Terminal | Separate tab | Embedded panel |
| Extensions | nbextensions | Lab extensions (richer API) |
| Drag & drop | ❌ | ✅ Cells and tabs |
| CSV viewer | ❌ | ✅ Built-in |
| Dark mode | Via extension | ✅ Built-in |

**Recommendation:** Use **JupyterLab** for new projects — it is the officially recommended interface going forward.

---

## 10 · Google Colab Overview

Google Colaboratory is a **free, cloud-hosted** Jupyter environment provided by Google.

| Feature | Detail |
|---|---|
| **Free GPU/TPU** | Access to NVIDIA T4, A100, and TPUs |
| **No setup** | Runs entirely in the browser |
| **Google Drive** | Notebooks saved to Drive automatically |
| **Pre-installed libs** | NumPy, Pandas, TensorFlow, PyTorch, etc. |
| **Sharing** | Share like Google Docs with link permissions |

### Quick Start in Colab

```python
# Mount Google Drive
from google.colab import drive
drive.mount('/content/drive')

# Install a package not available by default
!pip install transformers

# Check GPU availability
!nvidia-smi
```

### Colab vs Local Jupyter

| Aspect | Google Colab | Local Jupyter |
|---|---|---|
| Cost | Free tier available | Your hardware |
| GPU | ✅ Free (limited) | Only if you have one |
| Customization | Limited | Full control |
| Session length | ~12 hrs max, may disconnect | Unlimited |
| Privacy | Data goes to Google servers | Fully local |

---

## Summary Table

| Topic | Key Takeaway |
|---|---|
| Installation | `pip install jupyterlab` or `conda install jupyterlab` |
| Starting | `jupyter lab` or `jupyter notebook` |
| Cell types | Code (execute), Markdown (document), Raw (passthrough) |
| Kernel | Manage via Interrupt, Restart, Change; state persists across cells |
| Shortcuts | `Shift+Enter` (run), `A/B` (insert), `DD` (delete) |
| Execution order | `In [n]` tracks *when* run, not position — beware hidden state |
| JupyterLab | Modern multi-panel IDE — preferred over classic |
| Colab | Free cloud Jupyter with GPU — great for learning and prototyping |

---

## Revision Questions

1. **What are the three cell types in Jupyter Notebook, and when would you use each?**
2. **Explain the difference between `Shift+Enter`, `Ctrl+Enter`, and `Alt+Enter`.**
3. **Why is cell execution order important? Give an example of how out-of-order execution can cause bugs.**
4. **How do you register a virtual environment as a Jupyter kernel?**
5. **Compare JupyterLab and the Classic Notebook interface — list at least four differences.**
6. **What are two advantages and two limitations of Google Colab compared to a local Jupyter setup?**

---

[Next → Chapter 2: Markdown & Documentation](./02-markdown-and-documentation.md)

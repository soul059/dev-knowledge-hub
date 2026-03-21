# Chapter 5: Best Practices for Jupyter Notebooks

> **Unit 6 · Jupyter Notebooks** — Professional conventions for writing clean, reproducible, and collaborative notebooks.

---

## 1 · Notebook Organization

### 1.1 — Standard Structure

Every notebook should follow a consistent top-to-bottom layout:

```
┌─────────────────────────────────────────────┐
│  Title & Description (Markdown)             │  ← What this notebook does
├─────────────────────────────────────────────┤
│  Imports & Configuration (Code)             │  ← ALL imports in one cell
├─────────────────────────────────────────────┤
│  Constants & Parameters (Code)              │  ← Hyperparams, file paths
├─────────────────────────────────────────────┤
│  Data Loading (Code + Markdown)             │  ← Load and describe data
├─────────────────────────────────────────────┤
│  Analysis / Processing (Code + Markdown)    │  ← Main work, well-sectioned
├─────────────────────────────────────────────┤
│  Results & Visualization (Code + Markdown)  │  ← Outputs and plots
├─────────────────────────────────────────────┤
│  Conclusion (Markdown)                      │  ← Summary of findings
└─────────────────────────────────────────────┘
```

### 1.2 — Imports at the Top

```python
# ── Standard Library ──────────────────────────────────
import os
import sys
import json
from pathlib import Path
from datetime import datetime

# ── Third-Party ───────────────────────────────────────
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

# ── Project Modules ───────────────────────────────────
from src.preprocessing import clean_data
from src.features import build_features

# ── Configuration ─────────────────────────────────────
%matplotlib inline
plt.style.use('seaborn-v0_8-whitegrid')
pd.set_option('display.max_columns', 50)
pd.set_option('display.max_rows', 100)

print(f"Python {sys.version}")
print(f"NumPy {np.__version__}, Pandas {pd.__version__}")
```

**Why?** Grouping all imports makes dependencies obvious and avoids `NameError` surprises when cells run out of order.

### 1.3 — Narrative Flow

Write notebooks like a **story**: each section should logically follow the previous one.

```markdown
## 1. Problem Statement
We want to predict customer churn based on usage patterns.

## 2. Data Overview
The dataset contains 7,043 customers with 21 features...

## 3. Key Finding
Month-to-month contracts have 3× higher churn than annual contracts.
```

---

## 2 · Reproducibility

### 2.1 — Restart & Run All

> 🏆 **Golden Rule:** A notebook that fails on "Restart & Run All" is **broken**, even if individual cells work.

Before sharing or committing, always run **Kernel → Restart & Run All** and verify that every cell completes without error.

### 2.2 — Pin Dependencies

```python
# Cell 1: Generate requirements from the current environment
%pip freeze > requirements.txt

# Or create a minimal requirements file manually:
```

```python
%%writefile requirements.txt
numpy==1.24.3
pandas==2.0.3
scikit-learn==1.3.0
matplotlib==3.7.2
seaborn==0.12.2
```

### 2.3 — Set Random Seeds

```python
import random
import numpy as np

SEED = 42

random.seed(SEED)
np.random.seed(SEED)

# For PyTorch
import torch
torch.manual_seed(SEED)
torch.cuda.manual_seed_all(SEED)

# For TensorFlow
import tensorflow as tf
tf.random.set_seed(SEED)

# For scikit-learn (pass to individual functions)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=SEED
)
```

### 2.4 — Parameterize File Paths

```python
# ❌ Hard-coded paths — breaks on other machines
df = pd.read_csv("C:/Users/keval/data/train.csv")

# ✅ Relative paths with Path
from pathlib import Path
DATA_DIR = Path("../data")
df = pd.read_csv(DATA_DIR / "train.csv")

# ✅ Or use environment variables
import os
DATA_DIR = Path(os.getenv("DATA_DIR", "../data"))
```

---

## 3 · Version Control

### 3.1 — The Problem with `.ipynb` in Git

Notebook files are JSON with embedded outputs (images, HTML, large data). This causes:
- **Huge diffs** that are unreadable
- **Merge conflicts** that are nearly impossible to resolve
- **Large repo size** from embedded images and data

### 3.2 — `nbstripout` — Strip Output Before Committing

```bash
# Install
pip install nbstripout

# Set up as a Git filter (run once per repo)
nbstripout --install

# Now all notebook outputs are automatically stripped on git add
# The notebook itself keeps its outputs — only the committed version is clean
```

### 3.3 — `jupytext` — Pair Notebooks with Scripts

Jupytext keeps a synced `.py` (or `.md`) copy of your notebook that diffs cleanly.

```bash
pip install jupytext

# Convert notebook to a paired .py script
jupytext --set-formats ipynb,py:percent notebook.ipynb

# This creates notebook.py with cells delimited by:
# %% [markdown]
# # Title
# %% 
# import pandas as pd
```

### 3.4 — `.gitignore` for Notebooks

```gitignore
# Ignore checkpoint files
.ipynb_checkpoints/

# Ignore large output notebooks (keep only stripped versions)
*-output.ipynb

# Ignore data files
data/*.csv
data/*.parquet
```

### 3.5 — Best Practice Summary for Git

| Strategy | Tool | When to Use |
|---|---|---|
| Strip outputs | `nbstripout` | Always — minimal setup |
| Pair with scripts | `jupytext` | When code review matters |
| Review notebooks | `nbdime` | For visual notebook diffs |

---

## 4 · Notebooks vs. Scripts

### 4.1 — When to Use Notebooks

| ✅ Use Notebooks For | ❌ Avoid Notebooks For |
|---|---|
| Exploratory data analysis | Production pipelines |
| Prototyping and experimentation | Long-running batch jobs |
| Visualization and reporting | Reusable library code |
| Teaching and tutorials | Unit-tested modules |
| Quick ad-hoc analysis | Complex multi-file applications |

### 4.2 — The Notebook-to-Script Pipeline

```
 Explore in Notebook  →  Extract to .py modules  →  Call from notebook
 ┌─────────────────┐     ┌──────────────────┐      ┌────────────────┐
 │ Prototype code   │ ──► │ src/preprocess.py │ ◄──  │ Clean notebook │
 │ Iterate quickly  │     │ src/model.py      │      │ imports modules │
 │ Visualize results│     │ src/evaluate.py   │      │ runs pipeline   │
 └─────────────────┘     └──────────────────┘      └────────────────┘
```

---

## 5 · Modularizing Code

### 5.1 — When to Extract to `.py`

Extract code from notebooks when:
- A function is **used in multiple notebooks**
- A function is **longer than ~20 lines**
- You need **unit tests** for the logic
- The code is **stable and well-defined**

### 5.2 — Step-by-Step Extraction

```python
# Step 1: Write and test in the notebook
def clean_text(text):
    text = text.lower().strip()
    text = re.sub(r'[^\w\s]', '', text)
    text = re.sub(r'\s+', ' ', text)
    return text

# Test it
assert clean_text("  Hello, WORLD!  ") == "hello world"
```

```python
# Step 2: Move to a .py file
%%writefile src/text_utils.py
import re

def clean_text(text):
    """Remove punctuation, extra whitespace, and lowercase."""
    text = text.lower().strip()
    text = re.sub(r'[^\w\s]', '', text)
    text = re.sub(r'\s+', ' ', text)
    return text
```

```python
# Step 3: Import and use
%load_ext autoreload
%autoreload 2

from src.text_utils import clean_text
df['clean'] = df['raw_text'].apply(clean_text)
```

### 5.3 — Project Structure

```
my_project/
├── notebooks/
│   ├── 01-eda.ipynb
│   ├── 02-features.ipynb
│   └── 03-modeling.ipynb
├── src/
│   ├── __init__.py
│   ├── data.py           # Data loading and cleaning
│   ├── features.py       # Feature engineering
│   ├── model.py           # Model training and evaluation
│   └── utils.py           # Shared utilities
├── tests/
│   ├── test_data.py
│   └── test_features.py
├── data/
│   ├── raw/
│   └── processed/
├── requirements.txt
└── README.md
```

---

## 6 · Naming Conventions

### 6.1 — Notebook Files

```
# Prefix with numbers for ordering
01-data-exploration.ipynb
02-feature-engineering.ipynb
03-model-training.ipynb
04-evaluation-report.ipynb

# Use descriptive, lowercase, hyphen-separated names
# ❌ Bad:  Untitled1.ipynb, Copy of analysis.ipynb, final_FINAL_v3.ipynb
# ✅ Good: customer-churn-eda.ipynb, lstm-sentiment-analysis.ipynb
```

### 6.2 — Variables and Functions

```python
# Follow PEP 8 inside notebooks too
train_data = pd.read_csv("train.csv")    # snake_case for variables
LEARNING_RATE = 0.001                     # UPPER_CASE for constants
def compute_metrics(y_true, y_pred):      # snake_case for functions
    pass
```

---

## 7 · Documentation Standards

### 7.1 — Every Notebook Needs

| Element | Where | Example |
|---|---|---|
| **Title** | First cell (Markdown) | `# Customer Churn Analysis` |
| **Purpose** | Second cell (Markdown) | One-paragraph description |
| **Author & Date** | Header area | `Author: Keval · Date: 2024-01-15` |
| **Data source** | Before data loading | Link/path to data + description |
| **Section headers** | Between logical blocks | `## 3. Feature Engineering` |
| **Key findings** | After analysis cells | Markdown cells explaining results |
| **Conclusion** | Last section | Summary of insights and next steps |

### 7.2 — Docstrings for Functions

```python
def train_and_evaluate(X_train, y_train, X_test, y_test, model_params=None):
    """
    Train a model and return evaluation metrics.
    
    Parameters
    ----------
    X_train : pd.DataFrame — Training features
    y_train : pd.Series — Training labels
    X_test : pd.DataFrame — Test features
    y_test : pd.Series — Test labels
    model_params : dict, optional — Hyperparameters for the model
    
    Returns
    -------
    dict : {'accuracy': float, 'f1': float, 'model': fitted model}
    """
    pass
```

---

## 8 · Collaboration Tips

### 8.1 — Code Review Checklist for Notebooks

```markdown
- [ ] Runs top-to-bottom with Restart & Run All
- [ ] All imports at the top
- [ ] No hard-coded file paths
- [ ] Random seeds set
- [ ] Outputs are meaningful (not leftover debugging)
- [ ] Markdown explains the "why", not just the "what"
- [ ] No sensitive data (API keys, passwords) in cells
```

### 8.2 — Clearing Sensitive Output

```python
# Before sharing, clear outputs that may contain sensitive info
# Menu: Cell → All Output → Clear

# Or use nbstripout to automate this
```

### 8.3 — Environment Sharing

```bash
# Share exact environment
pip freeze > requirements.txt

# Or use conda
conda env export > environment.yml

# Minimal approach: list only direct dependencies
pip install pipreqs
pipreqs . --force    # Scans imports and creates minimal requirements.txt
```

---

## 9 · Notebook Templates for ML Projects

### 9.1 — EDA Template Skeleton

```markdown
# 📊 Exploratory Data Analysis: [Dataset Name]

## 1. Setup
(imports, configuration, data loading)

## 2. Data Overview
- Shape, dtypes, memory usage
- First/last rows
- Statistical summary (describe)

## 3. Missing Values Analysis
- Heatmap of nulls
- Missing percentage per column

## 4. Univariate Analysis
- Distribution of each feature
- Target variable distribution

## 5. Bivariate Analysis
- Correlations
- Feature vs target relationships

## 6. Key Findings
- Bullet-point summary

## 7. Recommendations for Next Steps
```

### 9.2 — Model Training Template Skeleton

```markdown
# 🤖 Model Training: [Task Name]

## 1. Setup & Data Loading
## 2. Train/Validation/Test Split
## 3. Baseline Model
## 4. Feature Engineering
## 5. Model Selection & Hyperparameter Tuning
## 6. Final Model Training
## 7. Evaluation on Test Set
## 8. Error Analysis
## 9. Model Export & Next Steps
```

---

## 10 · Converting Notebooks with `nbconvert`

### 10.1 — Command-Line Conversion

```bash
# Convert to HTML (great for sharing reports)
jupyter nbconvert --to html analysis.ipynb

# Convert to PDF (requires LaTeX installation)
jupyter nbconvert --to pdf analysis.ipynb

# Convert to Python script
jupyter nbconvert --to script analysis.ipynb

# Convert to Markdown
jupyter nbconvert --to markdown analysis.ipynb

# Convert to slides (reveal.js presentation)
jupyter nbconvert --to slides analysis.ipynb --post serve

# Execute notebook and save with outputs
jupyter nbconvert --to notebook --execute analysis.ipynb --output analysis-executed.ipynb
```

### 10.2 — Convert Without Output

```bash
# Strip outputs during conversion
jupyter nbconvert --to html --no-input analysis.ipynb    # Hide code cells
jupyter nbconvert --to html --no-prompt analysis.ipynb   # Hide In[]/Out[] prompts
jupyter nbconvert --ClearOutputPreprocessor.enabled=True --to notebook analysis.ipynb
```

### 10.3 — Programmatic Conversion

```python
import nbformat
from nbconvert import HTMLExporter

# Load notebook
with open('analysis.ipynb') as f:
    nb = nbformat.read(f, as_version=4)

# Convert to HTML
exporter = HTMLExporter()
html_body, resources = exporter.from_notebook_node(nb)

# Save
with open('analysis.html', 'w', encoding='utf-8') as f:
    f.write(html_body)
```

---

## Summary Table

| Practice | Key Recommendation |
|---|---|
| **Organization** | Imports at top → Constants → Data → Analysis → Results → Conclusion |
| **Reproducibility** | Restart & Run All, pin deps, set random seeds, use relative paths |
| **Version control** | Use `nbstripout` + `jupytext`; add `.ipynb_checkpoints/` to `.gitignore` |
| **Notebooks vs scripts** | Notebooks for exploration; scripts for reusable/production code |
| **Modularization** | Extract stable functions to `.py` files; use `%autoreload 2` |
| **Naming** | Number-prefixed, lowercase, hyphen-separated: `01-eda.ipynb` |
| **Documentation** | Title, purpose, author, section headers, findings, conclusion |
| **Collaboration** | Clear outputs before sharing; use environment files; no secrets |
| **Templates** | Maintain EDA and modeling templates for consistency |
| **Conversion** | `nbconvert` for HTML, PDF, slides, scripts |

---

## Revision Questions

1. **Why should all imports be grouped in the first code cell? What problems does this prevent?**
2. **Explain the "Restart & Run All" rule. Why is a notebook that only works with selective cell execution considered broken?**
3. **Compare `nbstripout` and `jupytext` — what problem does each solve, and when would you use both together?**
4. **When should you extract notebook code into a `.py` module? List at least three criteria.**
5. **Design a `.gitignore` file for a Jupyter-based ML project. What should be excluded and why?**
6. **How would you convert a notebook into an HTML report that hides all code cells? Write the command.**

---

[← Chapter 4: Debugging](./04-debugging.md) · [Back to Unit 6 Index →](./README.md)

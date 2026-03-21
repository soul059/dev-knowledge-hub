# Chapter 2: Markdown & Documentation in Notebooks

> **Unit 6 · Jupyter Notebooks** — Writing rich, well-structured documentation alongside your code using Markdown, LaTeX, tables, and HTML.

---

## 1 · Why Document Your Notebooks?

A notebook without documentation is just a script with extra steps. Good documentation turns a notebook into a **reproducible, self-explanatory report** that your future self (and colleagues) can understand months later.

**Rule of thumb:** If someone can read your notebook *without* running it and still understand the analysis, your documentation is good enough.

---

## 2 · Markdown Basics

Change any cell to Markdown by pressing `M` in command mode, or selecting **Cell → Cell Type → Markdown**.

### 2.1 — Headers

```markdown
# H1 — Title (use once per notebook)
## H2 — Section
### H3 — Subsection
#### H4 — Sub-subsection
```

### 2.2 — Text Formatting

```markdown
**bold text**
*italic text*
***bold and italic***
~~strikethrough~~
`inline code`
> Blockquote — useful for callouts and notes
```

**Rendered:**  
**bold text**, *italic text*, ***bold and italic***, ~~strikethrough~~, `inline code`

### 2.3 — Lists

```markdown
# Unordered list
- Item one
- Item two
  - Sub-item (indent with 2 spaces)
  - Another sub-item

# Ordered list
1. First step
2. Second step
3. Third step

# Task list (renders in JupyterLab)
- [x] Clean data
- [ ] Train model
- [ ] Evaluate results
```

### 2.4 — Links and Images

```markdown
# Hyperlink
[Pandas Documentation](https://pandas.pydata.org/docs/)

# Image from URL
![Alt text](https://example.com/chart.png)

# Image from local file
![My Plot](./images/plot.png)

# Image with HTML (for sizing control)
<img src="./images/plot.png" width="400" alt="My Plot">
```

---

## 3 · Code Blocks in Markdown

Use triple backticks for fenced code blocks. These are **display only** — they do not execute.

````markdown
```python
import pandas as pd
df = pd.read_csv("data.csv")
df.head()
```

```sql
SELECT name, age FROM users WHERE age > 25;
```

```bash
pip install scikit-learn
```
````

Use inline code with single backticks: `` Use `df.describe()` to get summary stats. ``

---

## 4 · LaTeX Math

Jupyter supports LaTeX math rendering via MathJax.

### 4.1 — Inline Math

Wrap with single dollar signs:

```markdown
The loss function is $L = -\sum_{i} y_i \log(\hat{y}_i)$.
```

**Renders as:** The loss function is $L = -\sum_{i} y_i \log(\hat{y}_i)$.

### 4.2 — Display Math (Block)

Wrap with double dollar signs for centered, full-line equations:

```markdown
$$
\nabla_\theta J(\theta) = \frac{1}{m} \sum_{i=1}^{m} \nabla_\theta L(f_\theta(x^{(i)}), y^{(i)})
$$
```

### 4.3 — Common LaTeX Symbols for Data Science

| Notation | LaTeX | Renders As |
|---|---|---|
| Fraction | `\frac{a}{b}` | $\frac{a}{b}$ |
| Square root | `\sqrt{x}` | $\sqrt{x}$ |
| Summation | `\sum_{i=1}^{n} x_i` | $\sum_{i=1}^{n} x_i$ |
| Partial derivative | `\frac{\partial L}{\partial w}` | $\frac{\partial L}{\partial w}$ |
| Matrix | `\begin{bmatrix} a & b \\ c & d \end{bmatrix}` | matrix |
| Greek letters | `\alpha, \beta, \theta, \sigma` | $\alpha, \beta, \theta, \sigma$ |
| Hat notation | `\hat{y}` | $\hat{y}$ |
| Expectation | `\mathbb{E}[X]` | $\mathbb{E}[X]$ |

### 4.4 — Aligned Equations

```markdown
$$
\begin{aligned}
z &= w^T x + b \\
a &= \sigma(z) \\
L &= -(y \log a + (1-y) \log(1-a))
\end{aligned}
$$
```

---

## 5 · Tables

### 5.1 — Markdown Tables

```markdown
| Model       | Accuracy | F1 Score | Training Time |
|-------------|----------|----------|---------------|
| Logistic Reg| 0.85     | 0.83     | 2.1s          |
| Random Forest| 0.91    | 0.89     | 15.3s         |
| XGBoost     | 0.93     | 0.92     | 8.7s          |
```

**Alignment:**

```markdown
| Left-aligned | Center-aligned | Right-aligned |
|:-------------|:--------------:|--------------:|
| Data         | Data           | Data          |
```

### 5.2 — When to Use Code Instead

For dynamic tables, generate them in code cells:

```python
import pandas as pd

results = pd.DataFrame({
    'Model': ['Logistic Reg', 'Random Forest', 'XGBoost'],
    'Accuracy': [0.85, 0.91, 0.93],
    'F1 Score': [0.83, 0.89, 0.92]
})
results.style.highlight_max(axis=0, color='lightgreen')
```

---

## 6 · HTML in Markdown Cells

Jupyter Markdown cells support raw HTML for advanced formatting.

### 6.1 — Colored Text and Alerts

```html
<span style="color:red">⚠️ Warning: This step is irreversible!</span>

<div style="background-color:#e7f3fe; padding:12px; border-left:4px solid #2196F3;">
  <strong>ℹ️ Note:</strong> Ensure your data is normalized before training.
</div>

<div style="background-color:#fff3cd; padding:12px; border-left:4px solid #ffc107;">
  <strong>⚠️ Caution:</strong> Large datasets may cause memory issues.
</div>
```

### 6.2 — Collapsible Sections

```html
<details>
<summary><b>Click to expand: Data Dictionary</b></summary>

| Column | Type | Description |
|--------|------|-------------|
| age    | int  | Customer age |
| income | float| Annual income |

</details>
```

### 6.3 — Side-by-Side Layout

```html
<div style="display:flex; gap:20px;">
  <div style="flex:1;">
    <h4>Before Cleaning</h4>
    <p>Missing values: 342</p>
  </div>
  <div style="flex:1;">
    <h4>After Cleaning</h4>
    <p>Missing values: 0</p>
  </div>
</div>
```

---

## 7 · Documentation-First Notebooks

### 7.1 — Structure Template

```markdown
# 📊 Project Title

## 1. Objective
Clearly state the goal of the analysis.

## 2. Data Source
- Source, date, size, license
- Link to raw data

## 3. Setup & Imports
(code cell with all imports)

## 4. Data Loading & Exploration
- Shape, dtypes, missing values
- Key statistics

## 5. Data Cleaning & Preprocessing

## 6. Exploratory Data Analysis (EDA)

## 7. Modeling

## 8. Results & Evaluation

## 9. Conclusions & Next Steps
```

### 7.2 — README-Style Notebooks

For team projects, create a `00-README.ipynb` that serves as the entry point:

```markdown
# Project: Customer Churn Prediction

## 📁 Notebook Index
| Notebook | Description | Status |
|----------|-------------|--------|
| `01-eda.ipynb` | Exploratory analysis | ✅ Complete |
| `02-features.ipynb` | Feature engineering | ✅ Complete |
| `03-modeling.ipynb` | Model training | 🔄 In progress |

## 🛠 Setup
\```bash
pip install -r requirements.txt
\```

## 📊 Key Findings
- Churn rate is 26.5%
- Top predictor: contract type
```

---

## 8 · Adding Structure to Analysis Notebooks

### 8.1 — Section Dividers

```markdown
---
## 🔹 Section 3: Feature Engineering
---
```

### 8.2 — Callout Patterns

```markdown
> **📌 Key Insight:** Customers on month-to-month contracts churn 3x more.

> **🔧 TODO:** Add interaction features between tenure and contract type.

> **⚠️ Assumption:** We assume missing values are MCAR (Missing Completely at Random).
```

### 8.3 — Inline Annotations in Code Cells

```python
# ── 1. Load Data ──────────────────────────────────────────────
df = pd.read_csv("data.csv")

# ── 2. Quick Sanity Check ────────────────────────────────────
assert df.shape[0] > 0, "DataFrame is empty!"
print(f"Loaded {df.shape[0]:,} rows, {df.shape[1]} columns")
```

---

## Summary Table

| Topic | Key Takeaway |
|---|---|
| Headers | Use `#` to `####` to create a clear hierarchy |
| Formatting | `**bold**`, `*italic*`, `` `code` ``, `> blockquote` |
| Lists | `-` for unordered, `1.` for ordered, `- [x]` for tasks |
| LaTeX inline | `$E = mc^2$` for inline math |
| LaTeX display | `$$..$$` for centered block equations |
| Tables | Pipe-delimited with `|` and alignment via `:---:` |
| HTML | Use for colors, alerts, collapsible sections, layouts |
| Structure | Follow a consistent template: Objective → Data → Analysis → Results |
| README notebooks | Create an index notebook linking all project notebooks |

---

## Revision Questions

1. **How do you switch a cell to Markdown mode using a keyboard shortcut?**
2. **Write the LaTeX syntax for the mean squared error formula: MSE = (1/n) Σ(yᵢ − ŷᵢ)².**
3. **How do you create a table with right-aligned numbers in Markdown?**
4. **What HTML element can you use to create collapsible/expandable sections in a notebook?**
5. **Describe the "documentation-first" approach to writing notebooks. What sections should a well-structured analysis notebook include?**
6. **Why might you prefer an HTML `<img>` tag over Markdown `![]()`  syntax for images?**

---

[← Chapter 1: Notebook Environment](./01-notebook-environment.md) · [Next → Chapter 3: Magic Commands](./03-magic-commands.md)

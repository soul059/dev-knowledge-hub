# 🗂️ Chapter 4: Subplots and Layouts

> **Unit 4 — Data Visualization** · Python for Data Science · AIML Foundations

---

## 📋 Overview

Real-world reports and dashboards rarely contain a single chart. This chapter covers how to arrange multiple plots in a single figure using `plt.subplots`, `GridSpec`, shared axes, and Seaborn's figure-level API (`FacetGrid`).

---

## 1 · `plt.subplots()` — The Workhorse

```python
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns

tips = sns.load_dataset('tips')
x = np.linspace(0, 2 * np.pi, 100)
```

### Simple Grid

```python
fig, axes = plt.subplots(nrows=2, ncols=2, figsize=(10, 8))

axes[0, 0].plot(x, np.sin(x))
axes[0, 0].set_title("sin(x)")

axes[0, 1].plot(x, np.cos(x), color='orange')
axes[0, 1].set_title("cos(x)")

axes[1, 0].plot(x, np.tan(x), color='green')
axes[1, 0].set_ylim(-5, 5)
axes[1, 0].set_title("tan(x)")

axes[1, 1].plot(x, np.exp(-x), color='red')
axes[1, 1].set_title("exp(-x)")

fig.suptitle("Trig & Exponential Functions", fontsize=16, fontweight='bold')
plt.tight_layout()
plt.show()
```

### Single Row or Column

```python
# 1 row, 3 columns — axes is a 1-D array
fig, axes = plt.subplots(1, 3, figsize=(14, 4))
for i, ax in enumerate(axes):
    ax.plot(x, np.sin(x + i), label=f'phase={i}')
    ax.legend()
plt.tight_layout()
plt.show()
```

---

## 2 · Sharing Axes

When subplots share the same range, use `sharex` / `sharey` to keep them aligned and reduce clutter.

```python
fig, axes = plt.subplots(2, 1, figsize=(8, 6), sharex=True, sharey=True)

sns.histplot(tips[tips['sex'] == 'Male']['total_bill'],
             bins=20, color='steelblue', ax=axes[0])
axes[0].set_title("Male")

sns.histplot(tips[tips['sex'] == 'Female']['total_bill'],
             bins=20, color='salmon', ax=axes[1])
axes[1].set_title("Female")

fig.suptitle("Total Bill Distribution by Sex")
plt.tight_layout()
plt.show()
```

| Parameter | Effect |
|---|---|
| `sharex=True` | All subplots share the same x-axis range; only bottom row shows x labels |
| `sharey=True` | All subplots share the same y-axis range; only left column shows y labels |
| `sharex='col'` | Columns share x-axes independently |
| `sharey='row'` | Rows share y-axes independently |

---

## 3 · `fig.add_subplot()` — Adding Plots One at a Time

```python
fig = plt.figure(figsize=(10, 6))
ax1 = fig.add_subplot(2, 2, 1)   # 2 rows, 2 cols, position 1
ax1.plot(x, np.sin(x)); ax1.set_title("Plot 1")
ax2 = fig.add_subplot(2, 2, 2)
ax2.bar(['A', 'B', 'C'], [3, 7, 5]); ax2.set_title("Plot 2")
ax3 = fig.add_subplot(2, 1, 2)   # spans full bottom row
ax3.plot(x, np.cos(x), 'r--'); ax3.set_title("Plot 3 — full width")
plt.tight_layout()
plt.show()
```

---

## 4 · `GridSpec` — Complex Layouts

```python
from matplotlib.gridspec import GridSpec

fig = plt.figure(figsize=(12, 8))
gs = GridSpec(3, 3, figure=fig, hspace=0.4, wspace=0.3)

ax_main = fig.add_subplot(gs[0:2, 0:2])       # top-left: 2 rows × 2 cols
ax_main.scatter(tips['total_bill'], tips['tip'], alpha=0.6)
ax_main.set_title("Main Scatter")

ax_right = fig.add_subplot(gs[0:2, 2])         # top-right column
ax_right.hist(tips['tip'], bins=20, orientation='horizontal', color='coral')

ax_bottom = fig.add_subplot(gs[2, :])           # full bottom row
sns.boxplot(data=tips, x='day', y='total_bill', palette='Set2', ax=ax_bottom)
plt.show()
```

GridSpec slicing: `gs[row, col]`, `gs[0:2, 0:2]` (block), `gs[2, :]` (full row), `gs[:, 2]` (full col).

---

## 5 · Spacing Adjustments

### `plt.tight_layout()`
Automatically adjusts padding so nothing overlaps. Call it **after** adding all plots.

```python
plt.tight_layout(pad=1.5)  # pad controls overall padding
```

### `plt.subplots_adjust()`
Manual control over margins and spacing.

```python
plt.subplots_adjust(
    left=0.1,     # left margin
    right=0.95,   # right margin
    top=0.9,      # top margin
    bottom=0.1,   # bottom margin
    hspace=0.4,   # vertical spacing between rows
    wspace=0.3    # horizontal spacing between columns
)
```

### `constrained_layout` (Modern Alternative)

```python
fig, axes = plt.subplots(2, 2, figsize=(10, 8), layout='constrained')
# Automatically adjusts — no need for tight_layout()
```

---

## 6 · Subplot Titles

Use `ax.set_title()` per subplot and `fig.suptitle()` for the overall figure title.

---

## 7 · Seaborn: Figure-Level vs Axes-Level

| Aspect | Axes-Level | Figure-Level |
|---|---|---|
| Returns | Matplotlib `Axes` | Seaborn `FacetGrid` / `PairGrid` |
| Accepts `ax=` | ✅ Yes | ❌ No (creates its own figure) |
| Faceting (col/row) | Must be done manually | ✅ Built-in |
| Size control | `figsize` on figure | `height` + `aspect` |
| Examples | `sns.boxplot(ax=ax)` | `sns.catplot(kind='box')` |

```python
# Axes-level — fits into any subplot
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
sns.histplot(data=tips, x='total_bill', ax=axes[0])
sns.boxplot(data=tips, x='day', y='tip', ax=axes[1])
plt.tight_layout()
plt.show()

# Figure-level — creates its own grid
g = sns.catplot(data=tips, x='day', y='total_bill',
                col='sex', kind='box', height=4, aspect=1.2)
g.fig.suptitle("Figure-Level Example", y=1.02)
plt.show()
```

---

## 8 · FacetGrid — Faceted Plots

`FacetGrid` maps a plotting function across subsets of a DataFrame.

```python
g = sns.FacetGrid(tips, col='time', row='smoker',
                  height=3.5, aspect=1.3, margin_titles=True)
g.map_dataframe(sns.scatterplot, x='total_bill', y='tip', alpha=0.6)
g.add_legend()
g.set_titles(col_template="{col_name}", row_template="{row_name}")
g.set_axis_labels("Total Bill ($)", "Tip ($)")
plt.show()
```

### Key `FacetGrid` Methods

| Method | Purpose |
|---|---|
| `g.map()` / `g.map_dataframe()` | Apply a plotting function |
| `g.add_legend()` | Add a unified legend |
| `g.set_titles()` / `g.set_axis_labels()` | Customize titles and labels |
| `g.set(xlim=, ylim=)` | Set axis limits globally |

---

## 9 · Putting It All Together

Combine `GridSpec` with Seaborn axes-level functions to build dashboards in a single figure (see Section 4 for `GridSpec` details).

---

## 📝 Summary Table

| Technique | Code | Best For |
|---|---|---|
| Simple grid | `plt.subplots(nrows, ncols)` | Uniform grids |
| Dynamic add | `fig.add_subplot(rows, cols, idx)` | Mixed dimensions |
| Complex layout | `GridSpec` | Non-uniform cell spanning |
| Shared axes | `sharex=True, sharey=True` | Consistent scales |
| Auto spacing | `plt.tight_layout()` | Quick fix |
| Manual spacing | `plt.subplots_adjust()` | Fine-tuned margins |
| Faceted seaborn | `FacetGrid` / `catplot(col=)` | Subset comparisons |

---

## ❓ Revision Questions

1. How does `plt.subplots(2, 3)` arrange axes and how do you access each one?
2. What is the difference between `sharex=True` and `sharex='col'`?
3. Write code using `GridSpec` to create a layout with a large plot on the left (2 rows) and two smaller plots stacked on the right.
4. Why can't you pass `ax=` to a Seaborn figure-level function like `sns.catplot()`?
5. What is `FacetGrid` and when is it useful?
6. Compare `tight_layout()` and `constrained_layout` — when would you choose one over the other?

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Plot Types](03-plot-types.md) | [Unit 4 — Data Visualization](../) | [Customization & Styling →](05-customization-and-styling.md) |

# 📈 Chapter 2: Seaborn — Statistical Plots

> **Unit 4 — Data Visualization** · Python for Data Science · AIML Foundations

---

## 📋 Overview

Seaborn is a high-level statistical visualization library built on top of Matplotlib. It provides beautiful defaults, built-in themes, and tight integration with pandas DataFrames. This chapter covers its four major plot families: relational, distribution, categorical, and regression.

---

## 1 · Setup

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# Load built-in datasets
tips = sns.load_dataset('tips')
iris = sns.load_dataset('iris')
flights = sns.load_dataset('flights')

# Set global theme
sns.set_theme(style='whitegrid', palette='muted', font_scale=1.1)
```

### Available Themes

| Style | Description |
|---|---|
| `darkgrid` | Grey background with grid (default) |
| `whitegrid` | White background with grid |
| `dark` | Grey background, no grid |
| `white` | White background, no grid |
| `ticks` | White background with tick marks |

```python
sns.set_theme(style='ticks')
```

---

## 2 · Relational Plots

Show the relationship between two numeric variables.

### Scatter Plot

```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.scatterplot(data=tips, x='total_bill', y='tip',
                hue='smoker', style='time', size='size',
                palette='Set2', ax=ax)
ax.set_title("Tips vs Total Bill")
plt.show()
```

### Line Plot

```python
fmri = sns.load_dataset('fmri')
fig, ax = plt.subplots(figsize=(8, 5))
sns.lineplot(data=fmri, x='timepoint', y='signal',
             hue='region', style='event',
             markers=True, dashes=True, ax=ax)
ax.set_title("fMRI Signal Over Time")
plt.show()
```

---

## 3 · Distribution Plots

Visualize the distribution of one or two variables.

### Histogram

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

sns.histplot(data=tips, x='total_bill', bins=20, kde=True, ax=axes[0])
axes[0].set_title("Histogram with KDE overlay")

sns.histplot(data=tips, x='total_bill', hue='sex', multiple='stack', ax=axes[1])
axes[1].set_title("Stacked by Sex")
plt.tight_layout()
plt.show()
```

### KDE Plot (Kernel Density Estimate)

```python
fig, ax = plt.subplots()
sns.kdeplot(data=tips, x='total_bill', hue='time',
            fill=True, alpha=0.4, linewidth=2, ax=ax)
ax.set_title("KDE: Dinner vs Lunch Bills")
plt.show()
```

### ECDF Plot (Empirical Cumulative Distribution)

```python
fig, ax = plt.subplots()
sns.ecdfplot(data=tips, x='total_bill', hue='sex', ax=ax)
ax.set_title("ECDF of Total Bill")
plt.show()
```

---

## 4 · Categorical Plots

Compare values across categories.

### Box Plot — shows quartiles and outliers

```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.boxplot(data=tips, x='day', y='total_bill', hue='sex',
            palette='pastel', ax=ax)
ax.set_title("Total Bill by Day and Sex")
plt.show()
```

### Violin Plot — box plot + KDE

```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.violinplot(data=tips, x='day', y='total_bill', hue='sex',
               split=True, inner='quart', palette='Set2', ax=ax)
ax.set_title("Violin Plot — Split by Sex")
plt.show()
```

### Swarm Plot — individual points (no overlap)

```python
fig, ax = plt.subplots(figsize=(8, 5))
sns.swarmplot(data=tips, x='day', y='total_bill',
              hue='sex', dodge=True, size=4, ax=ax)
ax.set_title("Swarm Plot")
plt.show()
```

### Bar Plot — mean + confidence interval

```python
fig, ax = plt.subplots(figsize=(7, 4))
sns.barplot(data=tips, x='day', y='total_bill', hue='sex',
            estimator=np.mean, errorbar='ci', palette='Blues_d', ax=ax)
ax.set_title("Mean Total Bill by Day")
plt.show()
```

### Count Plot — frequency counts

```python
fig, ax = plt.subplots()
sns.countplot(data=tips, x='day', hue='sex', palette='rocket', ax=ax)
ax.set_title("Number of Records per Day")
plt.show()
```

---

## 5 · Regression Plots

### `regplot()` — axes-level

```python
fig, ax = plt.subplots(figsize=(7, 5))
sns.regplot(data=tips, x='total_bill', y='tip',
            scatter_kws={'alpha': 0.5}, line_kws={'color': 'red'}, ax=ax)
ax.set_title("Linear Regression: Tip ~ Total Bill")
plt.show()
```

---

## 6 · The hue / style / size Trifecta

These parameters add extra dimensions to any relational or categorical plot:

| Parameter | Maps to | Best for |
|---|---|---|
| `hue` | Color | Categorical grouping |
| `style` | Marker / line style | Distinguishing categories |
| `size` | Marker size | Numeric variable |

```python
sns.scatterplot(data=tips, x='total_bill', y='tip',
                hue='day',          # color by day
                style='smoker',     # marker shape by smoker
                size='size',        # marker size by party size
                palette='deep')
plt.show()
```

---

## 7 · Color Palettes

```python
# Built-in palette names
sns.color_palette('deep')       # default qualitative
sns.color_palette('pastel')     # soft
sns.color_palette('Set2')       # colorbrewer
sns.color_palette('viridis', 8) # sequential (8 colors)
sns.color_palette('coolwarm', as_cmap=True)  # diverging

# Preview a palette
sns.palplot(sns.color_palette('husl', 10))
plt.show()

# Set globally
sns.set_palette('colorblind')
```

| Palette Type | Use Case | Examples |
|---|---|---|
| Qualitative | Distinct categories | `deep`, `Set2`, `colorblind` |
| Sequential | Ordered values | `viridis`, `Blues`, `rocket` |
| Diverging | Values around a center | `coolwarm`, `RdBu`, `vlag` |

---

## 📝 Summary Table

| Plot Family | Axes-Level | Figure-Level |
|---|---|---|
| Relational | `scatterplot()`, `lineplot()` | `relplot()` |
| Distribution | `histplot()`, `kdeplot()`, `ecdfplot()` | `displot()` |
| Categorical | `boxplot()`, `violinplot()`, `swarmplot()`, `barplot()`, `countplot()` | `catplot()` |
| Regression | `regplot()` | `lmplot()` |

---

## ❓ Revision Questions

1. What is the difference between an **axes-level** function (e.g., `sns.boxplot()`) and a **figure-level** function (e.g., `sns.catplot()`)?
2. How do you overlay a KDE curve on a histogram in Seaborn?
3. Explain the roles of `hue`, `style`, and `size` in `sns.scatterplot()`.
4. When would you use a **violin plot** instead of a **box plot**?
5. Write code to create a faceted regression plot split by the `time` column using `lmplot()`.
6. How do you switch Seaborn's color palette to a colorblind-friendly one globally?

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Matplotlib Basics](01-matplotlib-basics.md) | [Unit 4 — Data Visualization](../) | [Plot Types →](03-plot-types.md) |

# 📊 Chapter 3: Plot Types — Choosing the Right Visualization

> **Unit 4 — Data Visualization** · Python for Data Science · AIML Foundations

---

## 📋 Overview

Choosing the correct plot type is as important as knowing how to code it. This chapter provides a practical guide to the most common chart types, when to use each one, and a decision flowchart for quick reference.

---

## 🔀 Decision Flowchart: Choosing a Plot

```
What do you want to show?
│
├─ Trend over time?          ──▶  Line Plot / Area Plot
├─ Relationship (x vs y)?    ──▶  Scatter Plot / Regression Plot
├─ Distribution of one var?  ──▶  Histogram / KDE / Box Plot / Violin
├─ Comparison across groups? ──▶  Bar Plot / Grouped Bar / Box Plot
├─ Composition / proportion? ──▶  Pie Chart (small n) / Stacked Bar
├─ Correlation matrix?       ──▶  Heatmap
├─ All pairwise relations?   ──▶  Pair Plot
└─ Spread & outliers?        ──▶  Box Plot / Violin Plot
```

---

## 1 · Line Plot — Trends Over Time

**Use when:** data is ordered (time series, sequences).

```python
import matplotlib.pyplot as plt
import numpy as np
import seaborn as sns
import pandas as pd

dates = pd.date_range('2024-01', periods=12, freq='M')
revenue = [20, 22, 25, 28, 35, 40, 38, 42, 50, 55, 60, 65]

fig, ax = plt.subplots(figsize=(9, 4))
ax.plot(dates, revenue, marker='o', linewidth=2, color='teal')
ax.fill_between(dates, revenue, alpha=0.1, color='teal')  # area effect
ax.set_title("Monthly Revenue 2024")
ax.set_ylabel("Revenue ($K)")
plt.tight_layout()
plt.show()
```

| ✅ Good for | ❌ Avoid when |
|---|---|
| Time series, sequential data | Categories with no natural order |
| Showing rate of change | Too many overlapping lines (>5) |

---

## 2 · Scatter Plot — Relationships

**Use when:** comparing two continuous variables.

```python
tips = sns.load_dataset('tips')

fig, ax = plt.subplots(figsize=(7, 5))
sns.scatterplot(data=tips, x='total_bill', y='tip', hue='smoker',
                style='time', alpha=0.7, ax=ax)
ax.set_title("Tip vs Total Bill")
plt.show()
```

| ✅ Good for | ❌ Avoid when |
|---|---|
| Correlation, clusters | Very few data points (<10) |
| Identifying outliers | Overplotting with thousands of points (use hexbin) |

---

## 3 · Histogram — Distribution

**Use when:** understanding the shape and spread of a single variable.

```python
fig, ax = plt.subplots(figsize=(7, 4))
sns.histplot(data=tips, x='total_bill', bins=25, kde=True, color='coral', ax=ax)
ax.set_title("Distribution of Total Bill")
plt.show()
```

### Choosing Bin Count
Use `bins='auto'`, `int(np.sqrt(len(data)))` (square root rule), or `int(1 + np.log2(len(data)))` (Sturges' rule).

---

## 4 · Bar Plot — Comparisons

**Use when:** comparing quantities across discrete categories.

```python
languages = ['Python', 'R', 'SQL', 'Java', 'Scala']
users = [72, 45, 68, 30, 15]

fig, ax = plt.subplots(figsize=(7, 4))
ax.bar(languages, users, color=sns.color_palette('viridis', 5), edgecolor='black')
ax.set_ylabel("Users (K)")
ax.set_title("Language Usage in ML")
plt.tight_layout()
plt.show()
```

### Grouped Bar Plot

```python
df = pd.DataFrame({
    'Language': ['Python', 'R', 'SQL'] * 2,
    'Year': [2022]*3 + [2024]*3,
    'Users': [60, 40, 55, 75, 42, 70]
})

fig, ax = plt.subplots(figsize=(7, 4))
sns.barplot(data=df, x='Language', y='Users', hue='Year', palette='Blues', ax=ax)
ax.set_title("Language Growth")
plt.show()
```

---

## 5 · Box Plot — Spread & Outliers

**Use when:** comparing distributions and spotting outliers across groups.

```python
fig, ax = plt.subplots(figsize=(7, 5))
sns.boxplot(data=tips, x='day', y='total_bill', hue='sex',
            palette='Set2', ax=ax)
ax.set_title("Bill Distribution by Day")
plt.show()
```

**Anatomy:** min → Q1 → median → Q3 → max; dots = outliers (>1.5×IQR).

---

## 6 · Violin Plot — Distribution Shape

**Use when:** you need box-plot information **plus** the shape of the distribution.

```python
fig, ax = plt.subplots(figsize=(7, 5))
sns.violinplot(data=tips, x='day', y='total_bill',
               inner='quart', palette='muted', ax=ax)
ax.set_title("Violin Plot — Total Bill by Day")
plt.show()
```

---

## 7 · Heatmap — Correlation / Matrix Data

**Use when:** displaying correlation matrices, confusion matrices, or any 2-D grid.

```python
iris = sns.load_dataset('iris')
corr = iris.select_dtypes(include='number').corr()

fig, ax = plt.subplots(figsize=(6, 5))
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm',
            vmin=-1, vmax=1, linewidths=0.5, ax=ax)
ax.set_title("Iris Correlation Matrix")
plt.tight_layout()
plt.show()
```

---

## 8 · Pie Chart — Proportions (Use with Caution)

**Use when:** showing parts of a whole with ≤5 categories.

```python
sizes = [45, 25, 20, 10]
labels = ['Python', 'R', 'SQL', 'Other']
explode = (0.05, 0, 0, 0)

fig, ax = plt.subplots(figsize=(5, 5))
ax.pie(sizes, labels=labels, explode=explode,
       autopct='%1.1f%%', startangle=90,
       colors=sns.color_palette('pastel'))
ax.set_title("Market Share")
plt.show()
```

> ⚠️ **Caution:** Humans are poor at comparing angles. Prefer bar charts for accuracy. Use pie charts only for very simple compositions (≤5 slices).

---

## 9 · Area Plot — Cumulative Trends

```python
x = np.arange(1, 8)
y1, y2 = [3,5,4,6,7,8,7], [2,3,4,3,5,6,5]
fig, ax = plt.subplots(figsize=(8, 4))
ax.stackplot(x, y1, y2, labels=['Product A', 'Product B'], alpha=0.8)
ax.legend(loc='upper left')
ax.set_title("Stacked Area Plot")
plt.show()
```

---

## 10 · Pair Plot — All Pairwise Relationships

**Use when:** exploring multivariate data at a glance.

```python
sns.pairplot(iris, hue='species', diag_kind='kde',
             palette='husl', plot_kws={'alpha': 0.6})
plt.suptitle("Iris Pair Plot", y=1.02)
plt.show()
```

---

## 📝 Plot Selection Summary Table

| Plot | Variables | Data Type | Primary Purpose |
|---|---|---|---|
| **Line** | 1–2 numeric | Ordered / time | Trends |
| **Scatter** | 2 numeric | Continuous | Relationships |
| **Histogram** | 1 numeric | Continuous | Distribution shape |
| **KDE** | 1 numeric | Continuous | Smooth distribution |
| **Bar** | 1 categorical + 1 numeric | Mixed | Comparisons |
| **Box** | 1 categorical + 1 numeric | Mixed | Spread & outliers |
| **Violin** | 1 categorical + 1 numeric | Mixed | Distribution + spread |
| **Heatmap** | Matrix | Numeric grid | Correlations |
| **Pie** | 1 categorical | Proportions | Parts of whole (≤5) |
| **Area** | 1–2 numeric | Ordered | Cumulative composition |
| **Pair plot** | Multiple numeric | Continuous | Multivariate overview |

---

## ❓ Revision Questions

1. You have monthly sales data for 3 years. Which plot type would you choose and why?
2. When is a **violin plot** more informative than a **box plot**?
3. Why are pie charts generally discouraged? What is a better alternative?
4. You want to explore correlations among 8 numeric features. Which plot type is most efficient?
5. What is `sns.pairplot()` and when would you use it?
6. How do you decide between a **histogram** and a **KDE plot**?

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Seaborn Statistical Plots](02-seaborn-statistical-plots.md) | [Unit 4 — Data Visualization](../) | [Subplots & Layouts →](04-subplots-and-layouts.md) |

# 📊 Chapter 1: Matplotlib Basics

> **Unit 4 — Data Visualization** · Python for Data Science · AIML Foundations

---

## 📋 Overview

Matplotlib is the foundational plotting library in Python. Nearly every other visualization library (Seaborn, Plotly, pandas plots) builds on or interoperates with it. This chapter covers the two interfaces (pyplot and OOP), core plot types, and essential customization.

---

## 1 · Installing & Importing

```python
# pip install matplotlib
import matplotlib.pyplot as plt
import numpy as np
```

---

## 2 · Figure and Axes Objects

Every Matplotlib graphic lives inside a **Figure** (the window/canvas) that contains one or more **Axes** (individual plots).

```python
# Create a figure with one axes
fig, ax = plt.subplots()          # recommended OOP approach
ax.plot([1, 2, 3], [4, 5, 6])
plt.show()

# Create a figure with custom size (width, height in inches)
fig, ax = plt.subplots(figsize=(10, 5))
```

### pyplot vs OOP Interface

| Feature | pyplot (`plt.plot()`) | OOP (`ax.plot()`) |
|---|---|---|
| Quick exploration | ✅ Great | Slightly more code |
| Multiple subplots | Awkward | ✅ Clean & explicit |
| Fine-grained control | Limited | ✅ Full control |
| Production code | ❌ Avoid | ✅ Recommended |

```python
# pyplot style (implicit state machine)
plt.plot([1, 2, 3], [10, 20, 30])
plt.title("pyplot style")
plt.show()

# OOP style (explicit axes)
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [10, 20, 30])
ax.set_title("OOP style")
plt.show()
```

---

## 3 · Basic Line Plot

```python
x = np.linspace(0, 2 * np.pi, 100)
y = np.sin(x)

fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, y, color='royalblue', linewidth=2, linestyle='-', label='sin(x)')
ax.set_title("Sine Wave", fontsize=14)
ax.set_xlabel("x (radians)")
ax.set_ylabel("sin(x)")
ax.legend()
ax.grid(True, linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```

### Multiple Lines on One Plot

```python
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, np.sin(x), label='sin(x)')
ax.plot(x, np.cos(x), label='cos(x)', linestyle='--')
ax.plot(x, np.sin(x) + np.cos(x), label='sin+cos', linestyle=':')
ax.legend(loc='upper right')
ax.set_title("Multiple Lines")
plt.show()
```

---

## 4 · Scatter Plot

```python
np.random.seed(42)
x = np.random.rand(50)
y = 2 * x + np.random.randn(50) * 0.3
colors = np.random.rand(50)
sizes = np.random.rand(50) * 200

fig, ax = plt.subplots()
scatter = ax.scatter(x, y, c=colors, s=sizes, alpha=0.6, cmap='viridis', edgecolors='k')
ax.set_title("Scatter Plot with Color & Size")
ax.set_xlabel("X"); ax.set_ylabel("Y")
fig.colorbar(scatter, ax=ax, label="Color value")
plt.show()
```

---

## 5 · Bar Plot

```python
categories = ['Python', 'R', 'SQL', 'Java', 'Julia']
values = [85, 55, 70, 40, 25]

fig, ax = plt.subplots(figsize=(7, 4))
bars = ax.bar(categories, values, color=['#3776AB', '#276DC3', '#E38C00', '#F89820', '#9558B2'],
              edgecolor='black', linewidth=0.8)
ax.set_title("Language Popularity in Data Science")
ax.set_ylabel("Score")

# Add value labels on bars
for bar in bars:
    ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 1,
            str(bar.get_height()), ha='center', fontsize=10)
plt.tight_layout()
plt.show()
```

### Horizontal Bar Plot

```python
fig, ax = plt.subplots()
ax.barh(categories, values, color='steelblue')
ax.set_xlabel("Score")
plt.show()
```

---

## 6 · Titles, Labels, and Legends

```python
fig, ax = plt.subplots()
ax.plot(x, np.sin(x), label='sin')
ax.set_title("Title Here", fontsize=16, fontweight='bold')
ax.set_xlabel("X-axis", fontsize=12)
ax.set_ylabel("Y-axis", fontsize=12)
ax.legend(loc='best', fontsize=10, frameon=True, shadow=True)
```

---

## 7 · Grid and Annotations

```python
fig, ax = plt.subplots(figsize=(8, 4))
ax.plot(x, np.sin(x))
ax.grid(True, which='both', linestyle='--', linewidth=0.5, alpha=0.7)

# Annotate a specific point
ax.annotate('Peak', xy=(np.pi/2, 1), xytext=(np.pi/2 + 1, 0.8),
            arrowprops=dict(arrowstyle='->', color='red'),
            fontsize=12, color='red')
plt.show()
```

---

## 8 · Saving Figures

```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, np.sin(x))
ax.set_title("Save Me")

# Save in different formats
fig.savefig('plot.png', dpi=150, bbox_inches='tight')     # raster
fig.savefig('plot.pdf', bbox_inches='tight')               # vector
fig.savefig('plot.svg', bbox_inches='tight')               # vector (web)
```

| Parameter | Purpose | Example |
|---|---|---|
| `dpi` | Resolution (dots per inch) | `dpi=300` for print |
| `bbox_inches` | Trim whitespace | `'tight'` |
| `facecolor` | Background color | `'white'` |
| `transparent` | Transparent background | `True` |

---

## 9 · Common Pitfalls & Best Practices

| ✅ Do | ❌ Don't |
|---|---|
| Use OOP interface for anything beyond quick exploration | Mix pyplot and OOP calls on the same figure |
| Call `plt.tight_layout()` before `plt.show()` | Forget to adjust layout (labels get cut off) |
| Set `figsize` early for readable proportions | Rely on default tiny figures |
| Use `fig.savefig()` before `plt.show()` | Save after `plt.show()` — the figure may be cleared |
| Add labels, title, and legend for clarity | Create unlabeled plots |

---

## 📝 Summary Table

| Concept | Key Function / Object |
|---|---|
| Create figure + axes | `fig, ax = plt.subplots()` |
| Line plot | `ax.plot(x, y)` |
| Scatter plot | `ax.scatter(x, y)` |
| Bar plot | `ax.bar(categories, values)` |
| Title / labels | `ax.set_title()`, `set_xlabel()`, `set_ylabel()` |
| Legend | `ax.legend()` |
| Grid | `ax.grid(True)` |
| Annotation | `ax.annotate()` |
| Save | `fig.savefig('file.png', dpi=150)` |
| Display | `plt.show()` |

---

## ❓ Revision Questions

1. What is the difference between a **Figure** and an **Axes** in Matplotlib?
2. Why is the OOP interface (`fig, ax = plt.subplots()`) preferred over `plt.plot()` for production code?
3. Write code to create a scatter plot with a colorbar and labeled axes.
4. How do you save a figure at 300 DPI with no extra whitespace?
5. What does `ax.annotate()` do, and what are its key parameters?
6. Why should you call `fig.savefig()` **before** `plt.show()`?

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| — | [Unit 4 — Data Visualization](../) | [Seaborn Statistical Plots →](02-seaborn-statistical-plots.md) |

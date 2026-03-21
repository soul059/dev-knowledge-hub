# 🎨 Chapter 5: Customization and Styling

> **Unit 4 — Data Visualization** · Python for Data Science · AIML Foundations

---

## 📋 Overview

Default plots are fine for exploration, but presentations, papers, and dashboards demand polished visuals. This chapter covers colors, markers, fonts, axes, themes, and the techniques needed to produce publication-quality figures.

---

## 1 · Colors

### Specifying Colors

```python
import matplotlib.pyplot as plt
import matplotlib.colors as mcolors
import numpy as np
import seaborn as sns

x = np.linspace(0, 10, 100)

fig, ax = plt.subplots()
ax.plot(x, np.sin(x), color='steelblue')         # named color
ax.plot(x, np.cos(x), color='#FF6347')            # hex
ax.plot(x, np.sin(x+1), color=(0.2, 0.6, 0.3))   # RGB tuple (0-1)
ax.plot(x, np.cos(x+1), color='C3')               # default cycle color
plt.show()
```

### Colormaps

```python
data = np.random.rand(10, 10)
fig, ax = plt.subplots()
im = ax.imshow(data, cmap='viridis')
fig.colorbar(im, ax=ax)
plt.show()
```

| Category | Colormaps | Use Case |
|---|---|---|
| Sequential | `viridis`, `plasma`, `Blues`, `Reds` | Ordered numeric data |
| Diverging | `coolwarm`, `RdBu`, `seismic` | Data with a meaningful center |
| Qualitative | `Set1`, `Set2`, `tab10` | Categorical data |
| Perceptually uniform | `viridis`, `inferno`, `magma` | Accurate perception, colorblind safe |

### Custom Color Palette in Seaborn

```python
my_palette = ['#264653', '#2a9d8f', '#e9c46a', '#f4a261', '#e76f51']
sns.set_palette(my_palette)

tips = sns.load_dataset('tips')
sns.barplot(data=tips, x='day', y='total_bill', hue='sex')
plt.show()
```

---

## 2 · Line Styles and Markers

```python
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, np.sin(x), linestyle='-', marker='o', markevery=10, label='solid + circle')
ax.plot(x, np.cos(x), linestyle='--', marker='s', markevery=10, label='dashed + square')
ax.legend()
plt.show()
```

| Parameter | Values |
|---|---|
| `linestyle` | `'-'` solid, `'--'` dashed, `'-.'` dash-dot, `':'` dotted |
| `marker` | `'o'` circle, `'s'` square, `'^'` triangle, `'D'` diamond |

---

## 3 · Font Sizes

```python
fig, ax = plt.subplots()
ax.plot(x, np.sin(x))
ax.set_title("Title", fontsize=18, fontweight='bold')
ax.set_xlabel("X axis", fontsize=14)
ax.tick_params(axis='both', labelsize=12)

# Global font settings
plt.rcParams.update({'font.size': 12, 'axes.titlesize': 16, 'axes.labelsize': 14})
```

---

## 4 · Tick Formatting

```python
import matplotlib.ticker as ticker
fig, ax = plt.subplots()
ax.plot(x, np.exp(x / 3))
ax.set_xticks([0, np.pi, 2*np.pi, 3*np.pi])
ax.set_xticklabels(['0', 'π', '2π', '3π'])
ax.yaxis.set_major_formatter(ticker.StrMethodFormatter('{x:,.0f}'))
ax.tick_params(axis='x', rotation=45)
plt.tight_layout(); plt.show()
```

---

## 5 · Axis Limits and Scales

```python
fig, ax = plt.subplots()
ax.plot(x, np.exp(x / 3))
ax.set_xlim(0, 10); ax.set_ylim(0, 50)    # custom limits
ax.set_yscale('log')                        # log scale: 'linear', 'log', 'symlog'
plt.show()
```

---

## 6 · Spines

Spines are the lines forming the border of the plot area.

```python
fig, ax = plt.subplots()
ax.plot(x, np.sin(x))

# Remove top and right spines
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

# Move bottom spine to y=0
ax.spines['bottom'].set_position(('data', 0))

# Thicken left spine
ax.spines['left'].set_linewidth(1.5)

ax.set_title("Clean Spines")
plt.show()
```

---

## 7 · Matplotlib Built-in Styles

```python
# List all available styles
print(plt.style.available)
# ['seaborn-v0_8', 'ggplot', 'fivethirtyeight', 'dark_background', ...]
```

```python
# Apply a style globally
plt.style.use('seaborn-v0_8-whitegrid')

# Temporary style for one block
with plt.style.context('ggplot'):
    fig, ax = plt.subplots()
    ax.plot(x, np.sin(x))
    plt.show()
```

### Popular Styles

| Style | Description |
|---|---|
| `'seaborn-v0_8-whitegrid'` | Clean, academic look |
| `'ggplot'` | R ggplot2 aesthetic |
| `'fivethirtyeight'` | Bold, journalistic |
| `'dark_background'` | Dark mode |
| `'bmh'` | Bayesian Methods for Hackers |
| `'tableau-colorblind10'` | Colorblind-friendly palette |

---

## 8 · Seaborn Themes

```python
sns.set_theme(style='whitegrid', palette='colorblind', font_scale=1.2)
sns.set_context('talk')  # options: 'paper', 'notebook', 'talk', 'poster'
```

---

## 9 · Publication-Quality Figures

```python
plt.rcParams.update({
    'figure.dpi': 150, 'savefig.dpi': 300,
    'font.size': 11, 'font.family': 'serif',
    'axes.linewidth': 0.8, 'lines.linewidth': 1.5,
})

fig, ax = plt.subplots(figsize=(6, 4))
ax.plot(x, np.sin(x), label='sin(x)', color='#1f77b4')
ax.set_xlabel('x (radians)'); ax.set_ylabel('Amplitude')
ax.set_title('Publication-Quality Plot')
ax.legend(frameon=False)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
fig.savefig('plot.pdf', bbox_inches='tight')
plt.show()
```

| Tip | Why |
|---|---|
| Save as PDF/SVG | Vector formats scale without pixelation |
| 300+ DPI for PNG | Meets journal requirements |
| `bbox_inches='tight'` | Eliminates excess whitespace |
| Remove unnecessary spines | Cleaner, less ink |
| Limit to 3–5 colors | Easier to distinguish |

---

## 10 · DPI Settings

```python
plt.rcParams['figure.dpi'] = 100    # screen rendering
plt.rcParams['savefig.dpi'] = 300   # saved files
fig.savefig('high_res.png', dpi=600) # per-save override
```

---

## 📝 Summary Table

| Customization | Key API |
|---|---|
| Colors | Named, hex `'#FF6347'`, RGB tuple, `cmap=` |
| Line styles | `linestyle=`, `marker=`, `linewidth=` |
| Fonts | `fontsize=`, `plt.rcParams['font.size']` |
| Ticks | `set_xticks()`, `ticker.StrMethodFormatter` |
| Axis limits/scale | `set_xlim()`, `set_yscale('log')` |
| Spines | `ax.spines['top'].set_visible(False)` |
| Matplotlib styles | `plt.style.use('ggplot')` |
| Seaborn themes | `sns.set_theme(style=, palette=, context=)` |
| DPI | `plt.rcParams['savefig.dpi'] = 300` |

---

## ❓ Revision Questions

1. How do you specify a color using a hex code vs. an RGB tuple in Matplotlib?
2. What is the difference between a **sequential** and a **diverging** colormap? Give an example of each.
3. Write code to remove the top and right spines from a plot.
4. How does `sns.set_context('talk')` differ from `sns.set_context('paper')`?
5. List three best practices for creating publication-quality figures.
6. How do you apply a Matplotlib style temporarily for a single plot without affecting others?

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Subplots & Layouts](04-subplots-and-layouts.md) | [Unit 4 — Data Visualization](../) | [Interactive Plots — Plotly →](06-interactive-plots-plotly.md) |

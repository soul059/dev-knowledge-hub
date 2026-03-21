# 🌐 Chapter 6: Interactive Plots with Plotly

> **Unit 4 — Data Visualization** · Python for Data Science · AIML Foundations

---

## 📋 Overview

Plotly creates interactive, web-based visualizations with zoom, pan, hover tooltips, and animations — all from Python. This chapter covers Plotly Express (high-level) and Graph Objects (low-level), layout customization, subplots, animations, and how Plotly compares to Matplotlib.

---

## 1 · Installation & Import

```python
# pip install plotly
import plotly.express as px          # high-level API
import plotly.graph_objects as go    # low-level API
from plotly.subplots import make_subplots
import pandas as pd
import numpy as np
```

---

## 2 · Plotly Express — Quick Interactive Plots

Plotly Express (`px`) works like Seaborn — pass a DataFrame and column names.

### Scatter Plot

```python
df = px.data.tips()

fig = px.scatter(df, x='total_bill', y='tip',
                 color='smoker', size='size',
                 hover_data=['day', 'time'],
                 title='Tips vs Total Bill',
                 labels={'total_bill': 'Total Bill ($)', 'tip': 'Tip ($)'})
fig.show()
```

### Line Plot

```python
gapminder = px.data.gapminder()
india = gapminder[gapminder['country'] == 'India']
fig = px.line(india, x='year', y='gdpPercap', title='India GDP Per Capita', markers=True)
fig.show()
```

### Bar Plot

```python
medals = px.data.medals_long()
fig = px.bar(medals, x='nation', y='count', color='medal', barmode='group',
             color_discrete_map={'gold': '#FFD700', 'silver': '#C0C0C0', 'bronze': '#CD7F32'})
fig.show()
```

### Histogram

```python
fig = px.histogram(df, x='total_bill', nbins=30, color='sex', marginal='box')
fig.show()
```

### Additional Express Plots

```python
fig = px.box(df, x='day', y='total_bill', color='smoker', title='Bill by Day')
fig.show()

fig = px.violin(df, x='day', y='total_bill', color='sex', box=True, title='Violin')
fig.show()

# Heatmap
corr = px.data.iris().select_dtypes(include='number').corr()
fig = px.imshow(corr, text_auto='.2f', color_continuous_scale='RdBu_r')
fig.show()
```

---

## 3 · Hover Tooltips

```python
fig = px.scatter(df, x='total_bill', y='tip', color='day',
                 hover_name='day',
                 hover_data={'total_bill': ':.2f', 'tip': ':.2f', 'size': True})
fig.show()
```

### Custom Hover Template

```python
fig = px.scatter(df, x='total_bill', y='tip')
fig.update_traces(
    hovertemplate='<b>Bill:</b> $%{x:.2f}<br><b>Tip:</b> $%{y:.2f}<extra></extra>'
)
fig.show()
```

---

## 4 · Plotly Graph Objects — Full Control

```python
fig = go.Figure()
fig.add_trace(go.Scatter(
    x=df['total_bill'], y=df['tip'], mode='markers',
    marker=dict(size=10, color=df['size'], colorscale='Viridis', showscale=True),
    text=df['day'],
    hovertemplate='Bill: $%{x:.2f}<br>Tip: $%{y:.2f}<br>Day: %{text}',
    name='Tips'
))
fig.update_layout(title='Tips Scatter', xaxis_title='Total Bill ($)',
                  yaxis_title='Tip ($)', template='plotly_white')
fig.show()
```

### Adding Multiple Traces

```python
x = np.linspace(0, 2 * np.pi, 100)
fig = go.Figure()
fig.add_trace(go.Scatter(x=x, y=np.sin(x), mode='lines', name='sin(x)'))
fig.add_trace(go.Scatter(x=x, y=np.cos(x), mode='lines+markers',
                         name='cos(x)', line=dict(dash='dash')))
fig.update_layout(template='plotly_white')
fig.show()
```

---

## 5 · Layout Customization

```python
fig.update_layout(
    title=dict(text='My Plot', font=dict(size=20), x=0.5),
    xaxis=dict(title='X', showgrid=True),
    yaxis=dict(title='Y', zeroline=True),
    plot_bgcolor='white',
    font=dict(family='Arial', size=14),
    legend=dict(x=0.01, y=0.99),
    width=800, height=500
)
```

### Templates: `'plotly'`, `'plotly_white'`, `'plotly_dark'`, `'ggplot2'`, `'seaborn'`, `'simple_white'`

---

## 6 · Subplots in Plotly

```python
fig = make_subplots(rows=1, cols=2, subplot_titles=('Scatter', 'Histogram'))
fig.add_trace(go.Scatter(x=df['total_bill'], y=df['tip'], mode='markers'), row=1, col=1)
fig.add_trace(go.Histogram(x=df['total_bill'], nbinsx=25), row=1, col=2)
fig.update_layout(height=400, width=900, template='plotly_white')
fig.show()
```

For mixed types, use the `specs` parameter with `colspan`/`rowspan`.

---

## 7 · Animations

### Animated Scatter (Gapminder)

```python
fig = px.scatter(gapminder, x='gdpPercap', y='lifeExp',
                 size='pop', color='continent', hover_name='country',
                 log_x=True, size_max=60,
                 animation_frame='year', animation_group='country',
                 range_x=[100, 100000], range_y=[25, 90])
fig.show()
```

---

## 8 · Saving Plots

```python
fig.write_html('my_plot.html')                        # interactive HTML
fig.write_html('plot.html', include_plotlyjs='cdn')   # smaller file
# Static images (requires: pip install kaleido)
fig.write_image('plot.png', scale=2)                  # raster
fig.write_image('plot.pdf')                           # vector
```

---

## 9 · Plotly in Jupyter

Plotly works seamlessly in Jupyter Notebook, JupyterLab, and VS Code — `fig.show()` renders inline automatically. For JupyterLab, install `jupyterlab-plotly` extension.

---

## 10 · Matplotlib vs Plotly Comparison

| Feature | Matplotlib | Plotly |
|---|---|---|
| Type | Static | Interactive |
| Hover/zoom | ❌ No | ✅ Built-in |
| Animation | Complex (`FuncAnimation`) | Easy (`animation_frame=`) |
| Export HTML | ❌ | ✅ `write_html()` |
| Static export | ✅ Excellent | Needs kaleido |
| Large data (>100K) | ✅ Fast | Slower |
| Journals | ✅ Standard | Less common |
| Dashboards | Needs Streamlit | ✅ Dash |

### When to Use Which: Plotly for exploration/dashboards/sharing; Matplotlib+Seaborn for publications/large data.

---

## 📝 Summary Table

| Concept | Key API |
|---|---|
| Quick scatter / line / bar | `px.scatter()`, `px.line()`, `px.bar()` |
| Histogram with marginals | `px.histogram(marginal='box')` |
| Hover tooltips | `hover_data=`, `hovertemplate=` |
| Full control traces | `go.Figure()`, `go.Scatter()`, `go.Bar()` |
| Layout | `fig.update_layout()` |
| Themes | `template='plotly_white'` |
| Subplots | `make_subplots(rows, cols)` |
| Animation | `animation_frame=` |
| Save HTML | `fig.write_html('file.html')` |
| Save image | `fig.write_image('file.png')` |

---

## ❓ Revision Questions

1. What is the difference between **Plotly Express** and **Graph Objects**? When would you use each?
2. How do you add custom hover tooltips using `hovertemplate`?
3. Write code to create an animated scatter plot using the Gapminder dataset.
4. How do you create subplots with mixed chart types in Plotly?
5. What are two ways to save a Plotly figure, and how do they differ?
6. In what scenarios would you choose Plotly over Matplotlib, and vice versa?

---

## 🧭 Navigation

| Previous | Up | Next |
|---|---|---|
| [← Customization & Styling](05-customization-and-styling.md) | [Unit 4 — Data Visualization](../) | — |

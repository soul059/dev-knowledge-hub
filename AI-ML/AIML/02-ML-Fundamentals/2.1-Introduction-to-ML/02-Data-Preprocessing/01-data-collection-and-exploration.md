# 📊 Chapter 1: Data Collection and Exploration

> **Module:** Data Preprocessing · **Topic:** Data Collection & EDA
> **Objective:** Learn to gather, explore, and assess data quality before building ML models.

---

## 📌 1. Why Data Collection & Exploration Matter

```
  Raw Data ──▶ Collection ──▶ Exploration ──▶ Cleaning ──▶ Feature Eng. ──▶ Model
                  │                │
                  │                └── Identify issues EARLY
                  └── Choose reliable sources
```

## 📌 2. Data Sources

| Source | Description | Example |
|---|---|---|
| **Databases** | Structured data from SQL/NoSQL stores | MySQL, PostgreSQL, MongoDB |
| **APIs** | Programmatic access to live data | Twitter API, OpenWeather API |
| **Web Scraping** | Extracting data from web pages | BeautifulSoup, Scrapy |
| **Surveys / Forms** | Manually collected responses | Google Forms, SurveyMonkey |
| **Public Datasets** | Open repositories for research | Kaggle, UCI ML Repository, data.gov |
| **Sensors / IoT** | Real-time streaming data | Temperature sensors, wearables |

**Key questions:** Is it relevant? Reliable? Recent? Sufficient volume? Legally accessible (GDPR)?

---

## 📌 3. EDA Techniques

**Univariate** (one variable): histograms, box plots, summary stats, value counts
**Bivariate** (two variables): scatter plots, correlation, grouped box plots, chi-square
**Multivariate** (3+ variables): pair plots, correlation heatmaps, PCA

---

## 📌 4. Summary Statistics

| Statistic | Formula | What It Tells You |
|---|---|---|
| **Mean (μ)** | `μ = (Σxᵢ) / n` | Central tendency; sensitive to outliers |
| **Median** | Middle value when sorted | Central tendency; robust to outliers |
| **Mode** | Most frequently occurring value | Most common category/value |
| **Std Dev (σ)** | `σ = √[ Σ(xᵢ - μ)² / n ]` | Spread around the mean |
| **Skewness** | `γ = E[(X-μ)³] / σ³` | Asymmetry of distribution |
| **Kurtosis** | `κ = E[(X-μ)⁴] / σ⁴` | Tail heaviness (outlier proneness) |

### Interpreting Skewness
```
  Left-Skewed (neg)          Normal (zero)         Right-Skewed (pos)
       ___                      ___                       ___
      /   \                    /   \                     /   \
     /     |                  /     \                   |     \
  __/      |___           ___/       \___           ___|      \__
  mean < median           mean = median             mean > median
```

---

## 📌 5. Distribution Analysis

| Distribution | Shape | Example |
|---|---|---|
| **Normal (Gaussian)** | Symmetric bell curve | Heights, test scores |
| **Right-Skewed** | Long tail to the right | Income, house prices |
| **Left-Skewed** | Long tail to the left | Age at retirement |
| **Uniform** | Flat, equal probability | Fair dice roll |
| **Bimodal** | Two peaks | Mixed populations |

---

## 📌 6. Visualization Techniques

### Box Plot Anatomy
```
         ┌─────────┐
    ─────┤         ├─────    ○  ○   (outliers)
         └─────────┘
    min  Q1   Q2   Q3  max
    IQR = Q3 - Q1 │ Outlier if < Q1-1.5×IQR or > Q3+1.5×IQR
```

### Python Visualizations
```python
import pandas as pd, seaborn as sns, matplotlib.pyplot as plt

# Histogram — distribution of a single variable
df['age'].hist(bins=30, edgecolor='black', color='skyblue')
plt.title('Age Distribution'); plt.show()

# Box Plot — outlier detection
sns.boxplot(x='species', y='petal_length', data=df); plt.show()

# Scatter Plot — relationship between two variables
plt.scatter(df['sqft'], df['price'], alpha=0.5, c='coral')
plt.xlabel('Square Footage'); plt.ylabel('Price ($)'); plt.show()

# Correlation Heatmap — multivariate relationships
sns.heatmap(df.corr(), annot=True, cmap='coolwarm', vmin=-1, vmax=1)
plt.title('Feature Correlation Matrix'); plt.show()
```

**Correlations:** `+1` = perfect positive · `-1` = perfect negative · `0` = no linear relationship

---

## 📌 7. Identifying Patterns and Anomalies

| Technique | Purpose |
|---|---|
| **Z-score** | Flag values where `|z| > 3` as outliers |
| **IQR method** | Flag values outside `[Q1 - 1.5×IQR, Q3 + 1.5×IQR]` |
| **Isolation Forest** | ML-based anomaly detection for multivariate data |

---

## 📌 8. Data Quality Assessment Checklist

```
  ✅ Completeness  — Are there missing values? What percentage?
  ✅ Consistency   — Do formats match across records?
  ✅ Accuracy      — Do values fall within expected ranges?
  ✅ Timeliness    — Is the data current and relevant?
  ✅ Uniqueness    — Are there duplicate records?
  ✅ Validity      — Do values conform to business rules?
```

---

## 📌 9. Python — Complete EDA Workflow

```python
import pandas as pd, numpy as np, seaborn as sns, matplotlib.pyplot as plt

df = pd.read_csv('housing.csv')

# ── Basic info ──
print("Shape:", df.shape)
print(df.info())                  # dtypes, non-null counts
print(df.describe())              # count, mean, std, min, quartiles, max

# ── Missing values ──
print("\nMissing values:\n", df.isnull().sum())
print("Missing %:\n", (df.isnull().sum() / len(df) * 100).round(2))
print("Duplicate rows:", df.duplicated().sum())

# ── Skewness and Kurtosis ──
print("Skewness:\n", df.skew())
print("Kurtosis:\n", df.kurtosis())

# ── Correlation heatmap ──
plt.figure(figsize=(10, 8))
sns.heatmap(df.corr(), annot=True, fmt='.2f', cmap='coolwarm')
plt.title('Correlation Matrix'); plt.tight_layout(); plt.show()

# ── All distributions ──
df.hist(bins=25, figsize=(14, 10), edgecolor='black')
plt.suptitle('Feature Distributions'); plt.tight_layout(); plt.show()
```

---

## 📌 10. Real-World ML Applications

| Domain | EDA Insight | Model Impact |
|---|---|---|
| **Healthcare** | Skewed lab values → log transform | Better regression convergence |
| **Finance** | Outliers in transactions → fraud signals | Anomaly detection thresholds |
| **E-commerce** | Bimodal purchase patterns → segments | Clustering model design |
| **NLP** | Word frequency distributions | Embedding layer dimensions |

---

## 📋 Summary Table

| Concept | Key Takeaway |
|---|---|
| Data Sources | Choose reliable, relevant, and legal sources |
| Univariate EDA | Understand each feature's distribution individually |
| Bivariate EDA | Discover correlations and relationships |
| Summary Stats | Mean, median, std, skewness, kurtosis |
| Visualization | Histograms, box plots, scatter plots, heatmaps |
| Anomaly Detection | Z-score, IQR, Isolation Forest |
| Quality Checklist | Completeness, consistency, accuracy, uniqueness |

---

## ❓ Revision Questions

1. **What is the difference between univariate and bivariate analysis?** Give an example of each.
2. **A dataset has skewness = +2.3. What does this indicate?** Mean or median — which is better here?
3. **Q1 = 20 and Q3 = 80. What are the IQR outlier boundaries?** Is a value of 110 an outlier?
4. **Why is `df.describe()` not sufficient for a complete EDA?** What additional steps are needed?
5. **Name three public dataset repositories** and one advantage of each.
6. **When would you prefer a box plot over a histogram** for exploring a numerical feature?

---

## 🧭 Navigation

| ⬅️ Previous | ➡️ Next |
|---|---|
| — | [02 · Handling Missing Values](02-handling-missing-values.md) |

---
> **Pro Tip:** Always perform EDA _before_ any data cleaning or feature engineering. Understanding your
> raw data prevents you from making incorrect assumptions during preprocessing.

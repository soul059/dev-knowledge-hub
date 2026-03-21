# Chapter 8: Time Series Basics

> **Unit 3 – Pandas** | Working with dates, times, and temporal data — essential for ML feature engineering.

---

## 1. Overview

Pandas was originally built for financial time series analysis and has best-in-class datetime support. Key concepts: converting to datetime, creating date ranges, resampling frequencies, rolling windows, and time-based slicing.

```python
import pandas as pd
import numpy as np
```

---

## 2. Converting to Datetime — `pd.to_datetime()`

```python
# From strings
pd.to_datetime('2024-01-15')
pd.to_datetime('January 15, 2024')
pd.to_datetime('15/01/2024', dayfirst=True)

# From a Series
dates = pd.Series(['2024-01-01', '2024-02-15', '2024-03-30'])
dates = pd.to_datetime(dates)

# Handle errors
pd.to_datetime(['2024-01-01', 'not_a_date', '2024-03-01'], errors='coerce')
# NaT for unparseable values (NaT = Not a Time, like NaN for dates)

# Specify format for faster parsing
pd.to_datetime('20240115', format='%Y%m%d')
pd.to_datetime(dates, format='%Y-%m-%d')    # Faster when format is known

# From multiple columns
df = pd.DataFrame({'year': [2024, 2024], 'month': [1, 6], 'day': [15, 30]})
df['date'] = pd.to_datetime(df[['year', 'month', 'day']])

# From Unix timestamp
pd.to_datetime(1705276800, unit='s')    # Seconds since epoch
pd.to_datetime(1705276800000, unit='ms')
```

### Common Format Codes

| Code | Meaning | Example |
|---|---|---|
| `%Y` | 4-digit year | 2024 |
| `%m` | Month (01-12) | 03 |
| `%d` | Day (01-31) | 15 |
| `%H` | Hour (00-23) | 14 |
| `%M` | Minute (00-59) | 30 |
| `%S` | Second (00-59) | 45 |
| `%A` | Day name | Monday |
| `%B` | Month name | January |

---

## 3. DatetimeIndex and Date Ranges

```python
# Create a DatetimeIndex
dates = pd.DatetimeIndex(['2024-01-01', '2024-01-02', '2024-01-03'])

# Generate date ranges with pd.date_range()
pd.date_range(start='2024-01-01', end='2024-12-31', freq='D')     # Daily
pd.date_range(start='2024-01-01', periods=12, freq='MS')          # Monthly start
pd.date_range(start='2024-01-01', periods=52, freq='W')           # Weekly
pd.date_range(start='2024-01-01', periods=24, freq='h')           # Hourly
pd.date_range(start='2024-01-01', periods=4, freq='QS')           # Quarterly start

# Common frequency aliases
# 'D' = Day, 'W' = Week, 'MS' = Month Start, 'ME' = Month End
# 'QS' = Quarter Start, 'YS' = Year Start, 'h' = Hour, 'min' = Minute
# 'B' = Business day, 'BMS' = Business month start

# Create a time series DataFrame
ts = pd.DataFrame({
    'date': pd.date_range('2024-01-01', periods=365, freq='D'),
    'value': np.random.randn(365).cumsum()
})
ts = ts.set_index('date')
```

---

## 4. Extracting Date Components — `.dt` Accessor

```python
df = pd.DataFrame({
    'timestamp': pd.date_range('2024-01-01', periods=100, freq='D')
})

# Extract components
df['year'] = df['timestamp'].dt.year
df['month'] = df['timestamp'].dt.month
df['day'] = df['timestamp'].dt.day
df['day_of_week'] = df['timestamp'].dt.dayofweek      # 0=Monday, 6=Sunday
df['day_name'] = df['timestamp'].dt.day_name()         # 'Monday', 'Tuesday', ...
df['month_name'] = df['timestamp'].dt.month_name()
df['quarter'] = df['timestamp'].dt.quarter
df['week'] = df['timestamp'].dt.isocalendar().week     # ISO week number
df['hour'] = df['timestamp'].dt.hour
df['is_month_end'] = df['timestamp'].dt.is_month_end
df['is_month_start'] = df['timestamp'].dt.is_month_start

# Day of year (useful for seasonality features)
df['day_of_year'] = df['timestamp'].dt.dayofyear
```

---

## 5. Period and Timedelta

### 5.1 Period — A Span of Time

```python
# A Period represents a span (e.g., "January 2024")
p = pd.Period('2024-01', freq='M')     # January 2024
p.start_time                            # 2024-01-01
p.end_time                              # 2024-01-31

# Period arithmetic
p + 3                                   # April 2024 (3 months later)

# PeriodIndex
periods = pd.period_range('2024-01', periods=12, freq='M')

# Convert between Timestamp and Period
ts = pd.Timestamp('2024-06-15')
ts.to_period('M')                       # Period('2024-06', 'M')
```

### 5.2 Timedelta — Duration Between Times

```python
# Create timedeltas
td = pd.Timedelta('5 days')
td = pd.Timedelta(days=5, hours=3, minutes=30)
td = pd.to_timedelta('2 days 6 hours')

# Timedelta arithmetic
pd.Timestamp('2024-01-01') + pd.Timedelta('30 days')    # 2024-01-31

# Calculate duration between dates
df['days_since'] = (pd.Timestamp('today') - df['timestamp']).dt.days

# Timedelta Series operations
df['next_week'] = df['timestamp'] + pd.Timedelta('7 days')
```

---

## 6. Time Zone Handling

```python
# Localize a naive timestamp (assign time zone)
ts = pd.Timestamp('2024-06-15 10:00')
ts_utc = ts.tz_localize('UTC')

# Convert between time zones
ts_est = ts_utc.tz_convert('US/Eastern')
ts_ist = ts_utc.tz_convert('Asia/Kolkata')

# For a Series or DatetimeIndex
dates = pd.date_range('2024-01-01', periods=5, freq='D', tz='UTC')
dates_est = dates.tz_convert('US/Eastern')

# Common time zones: 'UTC', 'US/Eastern', 'US/Pacific', 'Europe/London', 'Asia/Kolkata'

# List available time zones
import pytz
# pytz.all_timezones  # Very long list
```

---

## 7. Time-Based Indexing and Slicing

When the index is a `DatetimeIndex`, you get powerful string-based slicing.

```python
ts = pd.DataFrame({
    'value': np.random.randn(365).cumsum()
}, index=pd.date_range('2024-01-01', periods=365, freq='D'))

# Select a specific date
ts.loc['2024-06-15']

# Select an entire month
ts.loc['2024-03']

# Select a year
ts.loc['2024']

# Date range slicing (inclusive on both ends)
ts.loc['2024-03-01':'2024-06-30']

# Filter by day of week
ts[ts.index.dayofweek < 5]    # Weekdays only

# Filter by month
ts[ts.index.month.isin([6, 7, 8])]    # Summer months
```

---

## 8. Resampling — `resample()`

Change the frequency of time series data. Think of it as `groupby()` for time.

```python
# Create daily data
daily = pd.DataFrame({
    'sales': np.random.randint(100, 500, 365),
    'visitors': np.random.randint(50, 200, 365)
}, index=pd.date_range('2024-01-01', periods=365, freq='D'))

# Downsample: daily → monthly (aggregate)
monthly = daily.resample('ME').sum()
monthly_avg = daily.resample('ME').mean()

# Weekly average
weekly = daily.resample('W').mean()

# Quarterly total
quarterly = daily.resample('QE').sum()

# Multiple aggregations
monthly_stats = daily['sales'].resample('ME').agg(['sum', 'mean', 'std', 'count'])

# Upsample: monthly → daily (needs fill)
monthly_data = pd.DataFrame({
    'value': [100, 150, 200]
}, index=pd.date_range('2024-01', periods=3, freq='MS'))

daily_upsampled = monthly_data.resample('D').ffill()       # Forward fill
daily_upsampled = monthly_data.resample('D').interpolate()  # Interpolate

# Custom aggregation
daily['sales'].resample('ME').apply(lambda x: x.max() - x.min())
```

---

## 9. Rolling Windows — `rolling()`

Compute statistics over a sliding window of fixed size.

```python
# 7-day rolling average
daily['sales_7d_avg'] = daily['sales'].rolling(window=7).mean()

# 30-day rolling standard deviation
daily['sales_30d_std'] = daily['sales'].rolling(window=30).std()

# Rolling sum
daily['sales_7d_sum'] = daily['sales'].rolling(window=7).sum()

# Minimum periods — allow partial windows at the start
daily['sales_7d_avg'] = daily['sales'].rolling(window=7, min_periods=1).mean()

# Centered rolling window
daily['sales_centered'] = daily['sales'].rolling(window=7, center=True).mean()

# Exponentially weighted moving average (EWMA) — more weight to recent values
daily['sales_ewm'] = daily['sales'].ewm(span=7).mean()

# Custom rolling function
daily['sales_range'] = daily['sales'].rolling(window=7).apply(lambda x: x.max() - x.min())
```

---

## 10. Shift and Diff

```python
# shift() — move data forward or backward in time
daily['sales_yesterday'] = daily['sales'].shift(1)      # Previous day
daily['sales_tomorrow'] = daily['sales'].shift(-1)       # Next day
daily['sales_last_week'] = daily['sales'].shift(7)       # 7 days ago

# diff() — difference between consecutive values
daily['sales_change'] = daily['sales'].diff()            # Today - yesterday
daily['sales_change_7d'] = daily['sales'].diff(7)        # Change over 7 days

# pct_change() — percentage change
daily['sales_pct_change'] = daily['sales'].pct_change()  # Daily % change

# Lag features for ML (common in time series forecasting)
for i in range(1, 8):
    daily[f'sales_lag_{i}'] = daily['sales'].shift(i)
```

---

## 11. Practical Time Series Patterns for ML

```python
# Pattern 1: Feature engineering from dates
def create_date_features(df, date_col='date'):
    df['year'] = df[date_col].dt.year
    df['month'] = df[date_col].dt.month
    df['day'] = df[date_col].dt.day
    df['dayofweek'] = df[date_col].dt.dayofweek
    df['is_weekend'] = df[date_col].dt.dayofweek >= 5
    df['quarter'] = df[date_col].dt.quarter
    df['dayofyear'] = df[date_col].dt.dayofyear
    df['is_month_start'] = df[date_col].dt.is_month_start
    df['is_month_end'] = df[date_col].dt.is_month_end
    return df

# Pattern 2: Rolling statistics as features
def create_rolling_features(df, col, windows=[7, 14, 30]):
    for w in windows:
        df[f'{col}_rolling_mean_{w}'] = df[col].rolling(w, min_periods=1).mean()
        df[f'{col}_rolling_std_{w}'] = df[col].rolling(w, min_periods=1).std()
    return df

# Pattern 3: Train/test split for time series (NEVER random split!)
train = daily.loc[:'2024-09-30']    # First 9 months
test = daily.loc['2024-10-01':]     # Last 3 months

# Pattern 4: Detect seasonality with resampling
monthly_pattern = daily['sales'].groupby(daily.index.month).mean()
```

---

## 12. Best Practices

| Practice | Recommendation |
|---|---|
| Parse dates early | Use `parse_dates` in `read_csv()` or convert immediately |
| Set DatetimeIndex | Enables string slicing and `resample()` |
| Time series splits | **Never** use random train/test split — always use temporal split |
| Lag features | Create lag and rolling features for ML forecasting |
| Time zones | Always store in UTC, convert for display only |
| Format specifier | Use `format=` in `to_datetime()` for 5-10× faster parsing |

---

## 13. Summary Table

| Function | Purpose | Example |
|---|---|---|
| `pd.to_datetime()` | Convert to datetime | `pd.to_datetime('2024-01-15')` |
| `pd.date_range()` | Generate date sequence | `pd.date_range('2024-01', periods=12, freq='MS')` |
| `.dt` accessor | Extract components | `df['date'].dt.month` |
| `resample()` | Change frequency | `daily.resample('ME').mean()` |
| `rolling()` | Sliding window stats | `df['x'].rolling(7).mean()` |
| `shift()` | Lag / lead values | `df['x'].shift(1)` |
| `diff()` | Consecutive differences | `df['x'].diff()` |
| `ewm()` | Exponential weighting | `df['x'].ewm(span=7).mean()` |

---

## 14. Revision Questions

1. How do you convert a string column to datetime, handling invalid values gracefully?
2. What is the difference between `resample()` and `rolling()`?
3. Write code to create a DatetimeIndex with business days only for the year 2024.
4. Why should you never use random train/test splitting for time series data?
5. Create lag features (lag 1, 3, 7) and a 7-day rolling average for a `sales` column.
6. How do you extract the day of the week and check if a date is a weekend in Pandas?

---

[← Previous: Merging & Joining](07-merging-and-joining.md) | [Back to Series & DataFrames →](01-series-and-dataframes.md)

# 🛒 Supermarket Pulse: A Data-Driven Operations Analysis of Q1 2019

> *Cleaning the noise. Finding the signal. Telling the story behind every transaction — now with pandas DataFrames.*

---

## 📌 Project Overview

This branch reimplements the **Supermarket Pulse** capstone entirely in Python using a **pandas DataFrame** as the unified data structure. Where the original project used Excel Power Query and formulas, this version performs every step — data loading, cleaning, statistical computation, probability modeling, and visualization — programmatically inside a Jupyter Notebook.

Unlike the `supermarket_pulse_using_panda_series` branch (which extracts each column into its own named Series), this implementation keeps all data consolidated in a single working DataFrame — `supermarket` — and accesses columns by name throughout. This reflects how most real-world pandas workflows are structured: one clean table, operated on directly.

The same five analytical stages are preserved: data cleaning, descriptive statistics, probability modeling, visual dashboarding, and hypothesis formulation. Every figure computed has a business question behind it; every chart answers something management actually needs to know.

---

## 🗂️ Dataset

| Attribute | Detail |
|---|---|
| **Period** | Q1 2019 (January – March) |
| **Branches** | Alexandria (A), Cairo (B), Giza (C) |
| **Total Rows** | ~974 (after cleaning) |
| **Source** | CSV loaded directly from GitHub via URL |
| **Tool Used** | Python 3 — `pandas`, `numpy`, `matplotlib`, `scipy.stats` |

**Key columns:** Invoice ID, Branch, City, Customer Type, Gender, Product Line, Unit Price, Quantity, Tax, Sales, Date, Time, Payment Method, COGS, Gross Margin %, Gross Income, Rating.

---

## 🐍 Why a Single DataFrame?

Rather than splitting columns into standalone Series objects, this branch loads the dataset into `supermarket_raw`, immediately copies it into a working DataFrame named `supermarket`, and operates on columns using bracket notation throughout:

```python
supermarket_raw = pd.read_csv("...", index_col="Invoice ID")
supermarket = supermarket_raw.copy()
```

Working from a copy preserves the raw data as an unmodified reference — a clean separation between source and analysis that mirrors production data pipelines. All cleaning, transformations, and computations operate on `supermarket`, never on `supermarket_raw`.

---

## 🔧 Part 1 — Data Cleaning (In-Place DataFrame Operations)

The raw dataset arrived with the same structural issues as the original — addressed here programmatically rather than through Power Query.

### Split Column
`City_CustType` was a merged column containing two distinct attributes. Split using the `|` delimiter and inserted as two new columns, then the original was dropped — all in one block:

```python
if 'City_CustType' in supermarket.columns:
    city_custtype = supermarket["City_CustType"]
    supermarket['City']          = city_custtype.str.split("|").str[0]
    supermarket['Customer Type'] = city_custtype.str.split("|").str[1]
    supermarket = supermarket.drop(columns=['City_CustType'])
```

The guard clause (`if 'City_CustType' in supermarket.columns`) makes the cell idempotent — safe to re-run without error.

### Text Standardization
`Product line` had inconsistent capitalization. Normalized in place using `.str.strip().str.title()`:

```python
supermarket['Product line'] = supermarket['Product line'].str.strip().str.title()
```

### Null Handling
Rows with blank `Rating` values were dropped in a single operation using `dropna(subset=...)` — no manual index tracking required:

```python
supermarket = supermarket.dropna(subset=['Rating'])
```

The shape of both `supermarket_raw` and the cleaned `supermarket` were displayed side by side to confirm the drop count.

### Data Types
Column types were verified using `supermarket.info()`, which prints dtypes and non-null counts for all columns simultaneously — a more efficient diagnostic than checking each column individually.

---

## 📊 Part 2 — Baseline Metrics & Statistical Summary

Descriptive statistics were computed directly from DataFrame column accessors.

| Metric | Value |
|---|---|
| **Mean Total Sales** | $322.26 |
| **Median Total Sales** | $253.39 |
| **Rating Variance** | ~2.95 |
| **Rating Std Dev** | ~1.72 |
| **Margin of Error (95% CI, n=60)** | Calculated via `1.96 × supermarket['Sales'].sample(60).sem()` |

```python
print(f"The Mean of the Total Sales is: {supermarket['Sales'].mean()}")
print(f"The Median of the Total Sales is: {supermarket['Sales'].median()}")
print(f"The Mode of the Total Sales are: {supermarket['Sales'].mode().to_list()}")
```

### 🔍 Observation 1 — Skewness in Sales Revenue

The mean ($322.26) sits noticeably above the median ($253.39) — a gap of ~$68.87. This is the classic fingerprint of a **right-skewed distribution**: a small number of exceptionally large purchases pulling the mean upward while most transactions cluster at lower values.

**Implication:** The median is a more reliable measure of a *typical* transaction. Management should not anchor operational expectations to the mean.

---

## 📈 Part 3 — Visual Dashboard (matplotlib)

Four charts were built using `matplotlib.pyplot`, each sourced directly from DataFrame columns via bracket notation.

### Scatter Plot — Quantity vs. Total Sales

```python
X = supermarket['Quantity']
Y = supermarket['Sales']

z = np.polyfit(X, Y, 1)
p = np.poly1d(z)
ax.plot(X, p(X), color='red', linestyle='--', label='Linear Trendline')
```

A positive linear relationship exists between quantity purchased and total sales (R² ≈ 0.4957). Roughly 49.57% of the variation in total sales is explained by quantity alone — a moderate but meaningful correlation.

---

### Bar Chart — Revenue by Branch Location

```python
Z = supermarket['Sales'].groupby(supermarket['Branch']).sum()
```

| Branch | Total Revenue |
|---|---|
| Alexandria (A) | $103,013 |
| Cairo (B) | $102,875 |
| **Giza (C)** | **$107,993** |

Giza leads by a slim but consistent margin. Revenue values were annotated directly above each bar using `ax.text()`.

---

### Pie Chart — Payment Method Breakdown

```python
Z = supermarket['Payment'].value_counts()
ax.pie(Z, labels=Z.index, autopct='%1.1f%%', textprops={'color': 'white'})
```

Customer payment preferences were nearly evenly split:

- **Ewallet:** 35%
- **Cash:** 34%
- **Credit Card:** 31%

The near-parity of all three channels signals a diverse, digitally-engaged customer base.

---

### Histogram — Distribution of Customer Ratings

```python
hist = ax.hist(supermarket['Rating'], bins=12, edgecolor='black')
```

Customer ratings (scale: 4–10) showed a relatively **uniform distribution**, with frequency counts annotated above each bin. No strong concentration of very high ratings — indicating consistent but unremarkable satisfaction across the customer base.

---

### 🔍 Observation 2 — Ewallet Promotion Strategy

**Giza should be the primary target for any Ewallet promotion.** It generated the highest total revenue ($107,993) and, at 35% Ewallet share, an estimated Ewallet revenue of ~$37,797 — the highest across all branches. A focused promotion here reinforces existing high-value Ewallet behavior and delivers the greatest revenue impact per marketing dollar.

---

## 🎲 Part 4 — Probability Modeling (scipy.stats)

All distributions were computed using `scipy.stats` modules, with parameters derived live from the `supermarket` DataFrame.

```python
from scipy.stats import norm, binom, poisson, expon, randint
```

### Normal Distribution — Sales Revenue

```python
mean_sales = supermarket['Sales'].mean()
std_sales  = supermarket['Sales'].std()
```

| Scenario | Probability |
|---|---|
| Customer spends **≤ $500** | **76.53%** |
| Customer spends **> $800** | **2.59%** |

**Observation 3:** Over three-quarters of customers spend under $500, and high-ticket purchases above $800 are statistically rare (≈1 in 39 customers). The data strongly favors a **volume-over-premium strategy**.

---

### Binomial Distribution — Credit Card Usage

The baseline credit card probability was computed from the `Payment` column:

```python
payment_probabilities   = supermarket['Payment'].value_counts(normalize=True)
p_success_credit_card   = payment_probabilities['Credit card']
```

| Scenario | Probability |
|---|---|
| Exactly 20 of 50 customers pay by credit card | **4.63%** |
| Exactly 0 of 10 customers pay by credit card | **2.45%** |

Both outcomes represent statistical edge cases — reinforcing that credit card usage is consistent but not dominant.

---

### Poisson Distribution — Foot Traffic

Average daily transaction rate derived from the `Date` column:

```python
date_datetime          = pd.to_datetime(supermarket['Date'], format='%m/%d/%Y')
avg_daily_transactions = date_datetime.value_counts().mean()
```

| Scenario | Probability |
|---|---|
| Exactly 85 transactions tomorrow | Computed via `poisson.pmf(85, λ)` |
| Exactly 120 transactions tomorrow | Very low — not a credible staffing baseline |

Extreme outlier volumes (e.g., 120/day) carry very low probability and should not drive shift scheduling decisions.

---

### Exponential Distribution — Equipment Failure

```python
prob_jam_within_20  = expon.cdf(20, scale=45)      # ~36%
prob_smooth_60_min  = 1 - expon.cdf(60, scale=45)
```

With printer jams averaging every 45 minutes, a jam within 20 minutes carries a non-trivial probability (~36%). Preventive maintenance windows should account for this.

---

### Uniform Distribution — Voucher Draw

```python
uniform_probability = randint.pmf(142, 1, 301)   # = 1/300 ≈ 0.33%
```

Each of 300 receipts carries an equal 1/300 probability of winning. No receipt number holds any statistical advantage.

---

---

## 🔄 DataFrame vs. Series Branch — Key Differences

| Aspect | Series Branch | **This Branch (DataFrame)** |
|---|---|---|
| Data structure | 15 standalone `pd.Series` objects | One unified `supermarket` DataFrame |
| Column access | `sales.mean()` | `supermarket['Sales'].mean()` |
| Null dropping | `.drop(null_rating)` on every Series | `supermarket.dropna(subset=['Rating'])` |
| Column splitting | New Series assigned by name | New columns inserted into DataFrame; original dropped |
| Type inspection | `.dtype` per Series | `supermarket.info()` for all columns at once |
| Groupby syntax | `sales.groupby(branch).sum()` | `supermarket['Sales'].groupby(supermarket['Branch']).sum()` |

Both branches produce identical analytical results. The DataFrame approach is more concise for multi-column operations and closer to standard production pandas practice.

---

## 🛠️ Tools & Techniques

| Category | Tool / Method |
|---|---|
| Data Loading | `pd.read_csv()` via GitHub raw URL |
| Working Copy | `supermarket_raw.copy()` |
| Data Cleaning | `.str.split()`, `.str.strip().str.title()`, `.dropna()`, `.drop(columns=[])` |
| Type Inspection | `supermarket.info()` |
| Descriptive Statistics | `.mean()`, `.median()`, `.mode()`, `.var()`, `.std()`, `.sem()` |
| Probability Modeling | `scipy.stats` — `norm`, `binom`, `poisson`, `expon`, `randint` |
| Visualizations | `matplotlib.pyplot` — Scatter, Bar, Pie, Histogram |
| Trendline | `numpy.polyfit()` + `numpy.poly1d()` |

---


---

*This branch extends the original Excel capstone by reproducing every analytical step in Python using a unified DataFrame workflow — demonstrating that the same business insights can be derived programmatically with greater reproducibility, transparency, and scalability.*

---

> *"Without data, you're just another person with an opinion."* — W. Edwards Deming

# 🛒 Supermarket Pulse: A Data-Driven Operations Analysis of Q1 2019

> *Cleaning the noise. Finding the signal. Telling the story behind every transaction — now in Python.*

---

## 📌 Project Overview

This branch reimplements the **Supermarket Pulse** capstone entirely in Python using **pandas Series** as the core data structure. Where the original project used Excel Power Query and formulas, this version performs every step — data loading, cleaning, statistical computation, probability modeling, and visualization — programmatically inside a Jupyter Notebook.

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

## 🐍 Why Pandas Series?

Rather than loading the dataset into a single DataFrame and operating on it column-by-column, this implementation extracts each column into its own named **pandas Series** (indexed by `Invoice ID`). This design choice makes every transformation explicit, keeps column operations self-contained, and mirrors how an analyst would reason about one variable at a time.

```python
sales    = pd.Series(supermarket_raw["Sales"],    index=supermarket_raw.index, name="Sales")
rating   = pd.Series(supermarket_raw["Rating"],   index=supermarket_raw.index, name="Rating")
payment  = pd.Series(supermarket_raw["Payment"],  index=supermarket_raw.index, name="Payment")
# ...and so on for all 15 columns
```

---

## 🔧 Part 1 — Data Cleaning (pandas Series Operations)

The raw dataset arrived with the same structural issues as the original — addressed here programmatically rather than through Power Query.

### Split Column
`City_CustType` was a merged column. Split using the `|` delimiter via `.str.split()`:

```python
city          = city_custtype.str.split("|").str[0]
customer_type = city_custtype.str.split("|").str[1]
```

### Text Standardization
`Product line` had inconsistent capitalization. Normalized using `.str.strip().str.title()`:

```python
product_line = product_line.str.strip().str.title()
```

### Null Handling
Transactions with blank `Rating` values were identified via a list comprehension and dropped from **all** Series simultaneously to preserve index alignment:

```python
null_rating = [x for x in rating.index if pd.isnull(rating[x])]
rating = rating.drop(null_rating)
sales  = sales.drop(null_rating)
# ...applied across all 16 Series
```

### Data Types
`unit_price`, `quantity`, and `sales` were verified as `float64`, `int64`, and `float64` respectively using `.dtype`.

---

## 📊 Part 2 — Baseline Metrics & Statistical Summary

With clean Series in hand, descriptive statistics were computed directly from pandas built-in methods.

| Metric | Value |
|---|---|
| **Mean Total Sales** | $322.26 |
| **Median Total Sales** | $253.39 |
| **Rating Variance** | ~2.95 |
| **Rating Std Dev** | ~1.72 |
| **Margin of Error (95% CI, n=60)** | Calculated via `1.96 × sales.sample(60).sem()` |

```python
print(f"The Mean of the Total Sales is: {sales.mean()}")
print(f"The Median of the Total Sales is: {sales.median()}")
print(f"The Mode of the Total Sales are: {sales.mode().to_list()}")
```

### 🔍 Observation 1 — Skewness in Sales Revenue

The mean ($322.26) sits noticeably above the median ($253.39) — a gap of ~$68.87. This is the classic fingerprint of a **right-skewed distribution**: a small number of exceptionally large purchases pulling the mean upward while most transactions cluster at lower values.

**Implication:** The median is a more reliable measure of a *typical* transaction. Management should not anchor operational expectations to the mean.

---

## 📈 Part 3 — Visual Dashboard (matplotlib)

Four charts were built using `matplotlib.pyplot`, each derived directly from the cleaned Series.

### Scatter Plot — Quantity vs. Total Sales

```python
z = np.polyfit(quantity, sales, 1)
p = np.poly1d(z)
ax.plot(quantity, p(quantity), color='red', linestyle='--', label='Linear Trendline')
```

A positive linear relationship exists between quantity purchased and total sales (R² ≈ 0.4957). Roughly 49.57% of the variation in total sales is explained by quantity alone — a moderate but meaningful correlation.

---

### Bar Chart — Revenue by Branch Location

```python
Z = sales.groupby(branch).sum()
```

| Branch | Total Revenue |
|---|---|
| Alexandria (A) | $103,013 |
| Cairo (B) | $102,875 |
| **Giza (C)** | **$107,993** |

Giza leads by a slim but consistent margin. Bar values were annotated directly on each bar using `ax.text()`.

---

### Pie Chart — Payment Method Breakdown

```python
Z = payment.value_counts()
ax.pie(Z, labels=Z.index, autopct='%1.1f%%')
```

Customer payment preferences were nearly evenly split:

- **Ewallet:** 35%
- **Cash:** 34%
- **Credit Card:** 31%

The near-parity of all three channels signals a diverse, digitally-engaged customer base.

---

### Histogram — Distribution of Customer Ratings

```python
hist = ax.hist(rating, bins=12, edgecolor='black')
```

Customer ratings (scale: 4–10) showed a relatively **uniform distribution**, with frequency counts annotated above each bin. No strong concentration of very high ratings — indicating consistent but unremarkable satisfaction across the customer base.

---

### 🔍 Observation 2 — Ewallet Promotion Strategy

**Giza should be the primary target for any Ewallet promotion.** It generated the highest total revenue ($107,993) and, at 35% Ewallet share, an estimated Ewallet revenue of ~$37,797 — the highest across all branches. A focused promotion here reinforces existing high-value Ewallet behavior and delivers the greatest revenue impact per marketing dollar.

---

## 🎲 Part 4 — Probability Modeling (scipy.stats)

All distributions were computed using `scipy.stats` modules imported at runtime.

```python
from scipy.stats import norm, binom, poisson, expon, randint
```

### Normal Distribution — Sales Revenue

Parameters derived live from the cleaned `sales` Series:

```python
mean_sales = sales.mean()
std_sales  = sales.std()
```

| Scenario | Probability |
|---|---|
| Customer spends **≤ $500** | **76.53%** |
| Customer spends **> $800** | **2.59%** |

**Observation 3:** Over three-quarters of customers spend under $500, and high-ticket purchases above $800 are statistically rare (≈1 in 39 customers). The data strongly favors a **volume-over-premium strategy**.

---

### Binomial Distribution — Credit Card Usage

The baseline credit card probability was computed directly from the `payment` Series:

```python
p_success_credit_card = payment.value_counts(normalize=True)['Credit card']
```

| Scenario | Probability |
|---|---|
| Exactly 20 of 50 customers pay by credit card | **4.63%** |
| Exactly 0 of 10 customers pay by credit card | **2.45%** |

Both outcomes represent statistical edge cases — reinforcing that credit card usage is consistent but not dominant.

---

### Poisson Distribution — Foot Traffic

Average daily transaction rate derived from the `date` Series:

```python
date_datetime = pd.to_datetime(date, format='%m/%d/%Y')
avg_daily_transactions = date_datetime.value_counts().mean()
```

| Scenario | Probability |
|---|---|
| Exactly 85 transactions tomorrow | Computed via `poisson.pmf(85, λ)` |
| Exactly 120 transactions tomorrow | Very low — not a credible baseline |

Extreme outlier volumes (e.g., 120/day) carry very low probability and should not drive staffing decisions.

---

### Exponential Distribution — Equipment Failure

```python
prob_jam_within_20 = expon.cdf(20, scale=45)   # ~36%
prob_smooth_60_min = 1 - expon.cdf(60, scale=45)
```

With printer jams averaging every 45 minutes, a jam within 20 minutes has a non-trivial probability (~36%). Preventive maintenance windows should account for this.

---

### Uniform Distribution — Voucher Draw

```python
uniform_probability = randint.pmf(142, 1, 301)  # = 1/300
```

Each of 300 receipts carries an equal 1/300 probability (~0.33%) of winning. No receipt number has any statistical advantage.

---

---

## 🛠️ Tools & Techniques

| Category | Tool / Method |
|---|---|
| Data Loading | `pd.read_csv()` via GitHub raw URL |
| Series Construction | `pd.Series()` with `Invoice ID` as index |
| Data Cleaning | `.str.split()`, `.str.strip().str.title()`, `.drop()` |
| Descriptive Statistics | `.mean()`, `.median()`, `.mode()`, `.var()`, `.std()`, `.sem()` |
| Probability Modeling | `scipy.stats` — `norm`, `binom`, `poisson`, `expon`, `randint` |
| Visualizations | `matplotlib.pyplot` — Scatter, Bar, Pie, Histogram |
| Trendline | `numpy.polyfit()` + `numpy.poly1d()` |

---


*This branch extends the original Excel capstone by reproducing every analytical step in Python, demonstrating that the same business insights can be derived programmatically — with greater reproducibility, transparency, and scalability.*

---

> *"Without data, you're just another person with an opinion."* — W. Edwards Deming

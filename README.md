# 🛒 Supermarket Performace Exploratory Data Analysis (EDA) : Q1 2019 Operations: Data & Insights

> *Extracting clarity from data noise to uncover the signal. Revealing the story behind the data.*

---

## 📌 Strategic Overview

Worked as the **Lead Data Analyst** for a global supermarket chain to optimize operations across three major Egyptian branches **(Alexandria, Cairo, and Giza)** . Built a unified analytical workflow that transformed a chaotic, raw extract of **~1,000 transactions** for Quarter 1 of the year 2019 into structured, actionable business intelligence.

The project spans five distinct analytical stages :

1. **Data Cleaning**  
   *Filtering out the noise and preparing raw transactional data for analysis.*
2. **Descriptive Statistics**  
   *Summarizing baseline KPIs and operational performance across branches.*
3. **Visual Dashboarding**  
   *Transforming complex data points into intuitive, executive-facing charts.*
4. **Probability Modeling**  
   *Analyzing purchase patterns and predicting transactional likelihoods.*
5. **Hypothesis Formulation**  
   *Testing data-backed assumptions to drive strategic decision-making.*

> 💡 Every figure computed has a business question behind it; every chart answers something management actually needs to know.

---

## 🗂️ Dataset

| Attribute | Detail |
|---|---|
| **Period** | Q1 2019 (January – March) |
| **Branches** | Alexandria (A), Cairo (B), Giza (C) |
| **Total Rows** | ~974 (after cleaning) |
| **Tool Used** | Microsoft Excel (Power Query + Formulas + Charts) |

**Key columns:** Invoice ID, Branch, City, Customer Type, Gender, Product Line, Unit Price, Quantity, Tax, Total, Date, Time, Payment Method, COGS, Gross Margin, Gross Income, Customer Rating.

---

## 🔧 Part 1 — Data Cleaning (Power Query)

The raw dataset arrived with several structural issues that would corrupt any downstream analysis if left uncorrected:

- **Split columns:** `City_CustType` was a merged column containing two distinct attributes. Separated into `City` and `Customer Type` using the correct delimiter.
- **Text standardization:** The `Product Line` column had inconsistent capitalization. Applied *Capitalize Each Word* transformation to normalize all entries.
- **Null handling:** Transactions with blank `Rating` values were filtered out — these could not contribute to satisfaction analysis.
- **Data types:** `Unit Price` and `Total` were cast to Currency; `Quantity` was cast to Whole Number.

The cleaned result was loaded into a dedicated `Clean_Data` sheet — the single source of truth for all subsequent analysis.

---

## 📊 Part 2 — Baseline Metrics & Statistical Summary

With clean data in hand, the focus shifted to establishing the financial and behavioral baseline.

| Metric | Value |
|---|---|
| **Mean Total Bill** | $322.26 |
| **Median Total Bill** | $253.39 |
| **Standard Deviation (Rating)** | ~1.72 |
| **Margin of Error (95% CI, n=60)** | Calculated via Z = 1.96 |

### 🔍 Observation 1 — Skewness in Sales Revenue


<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/109b180c-8710-4175-8162-b4ef85102970" />


The mean ($322.26) sits noticeably above the median ($253.39) — a gap of $68.87. This is the classic fingerprint of a **right-skewed distribution**: a small number of exceptionally large purchases pulling the mean upward while most transactions cluster at lower values. With a standard deviation of $245.65, transaction values are highly dispersed.

**Implication:** The median is a more reliable measure of a *typical* transaction. Management should not anchor operational expectations to the mean.

---

## 📈 Part 3 — Visual Dashboard

Four visualizations were built to give management an at-a-glance picture of operations.

### Scatter Plot — Quantity vs. Sales


<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/33a4a626-2693-4a05-99fb-f2880c98356d" />


A positive linear relationship exists between quantity purchased and sales (R² = 0.4957). Roughly **49.57%** of the variation in sales can be explained by quantity alone — a moderate but meaningful correlation. The relationship is real, not random.

---

### Bar Chart — Revenue by Branch Location


<img width="900" height="500" alt="image" src="https://github.com/user-attachments/assets/cca90860-f829-4c14-9806-b36e63cafb65" />


All three branches performed within a tight revenue band in Q1 2019:

| Branch | Total Revenue |
|---|---|
| Alexandria | $103,013 |
| Cairo | $102,875 |
| **Giza** | **$107,993** |

Giza leads by a slim but consistent margin.

---

### Pie Chart — Payment Method Breakdown

<img width="900" height="500" alt="pie_payment_type" src="https://github.com/user-attachments/assets/b0d06e8f-d836-4698-bcc3-142674815efa" />

<img width="952" height="614" alt="image" src="https://github.com/user-attachments/assets/cd68b92a-e828-4889-bf45-f561a4fb5ae6" />


Customer payment preferences were nearly evenly split:

- **Ewallet:** 35%
- **Cash:** 34%
- **Credit Card:** 31%

The near-parity of all three channels signals a diverse, digitally-engaged customer base — and an opportunity for targeted payment promotions.

---

### Histogram — Distribution of Customer Ratings

<img width="900" height="500" alt="histogram_customer_rating" src="https://github.com/user-attachments/assets/68166cd5-8c68-4095-b308-840e0498af88" />


Customer ratings (scale: 4–10) showed a relatively **uniform distribution** across the range, with a slight spike at the [4, 4.5] bin (99 occurrences) and a dip at the upper end ([9.5, 10] = 68). There is no strong concentration of very high ratings — indicating consistent but unremarkable satisfaction across the customer base.

---

### 🔍 Observation 2 — Ewallet Promotion Strategy

<img width="900" height="500" alt="observation_2_ewallet" src="https://github.com/user-attachments/assets/c18f0935-31fd-4680-ac28-890c3800c939" />


**Giza should be the primary target for any Ewallet promotion.** It generated the highest total revenue ($107,993), the widest Ewallet transaction spread (SD gap of $97.27), and an estimated Ewallet revenue of ~$37,797 (35% × $107,993) — the highest across all branches. A focused promotion here reinforces existing high-value Ewallet behavior and delivers the greatest revenue impact per dollar of marketing spend.

---

## 🎲 Part 4 — Probability Modeling

### Normal Distribution — Sales Revenue

<img width="900" height="500" alt="probability_distributions" src="https://github.com/user-attachments/assets/22fd4fd2-00b6-44f0-a3c9-34b1a7128b84" />


Using Mean = $322.26 and SD = $245.65:

| Scenario | Probability |
|---|---|
| Customer spends **≤ $500** | **76.53%** |
| Customer spends **≥ $800** | **2.59%** |

**Observation 3:** Over three-quarters of customers spend under $500, and high-ticket purchases above $800 are statistically rare (≈1 in 39 customers). The data strongly favors a **volume-over-premium strategy** — stock for smaller, frequent baskets rather than high-ticket inventory that sits on shelves.

---

### Binomial Distribution — Credit Card Usage (p = 0.31)

| Scenario | Probability |
|---|---|
| Exactly 20 of 50 customers pay by credit card | **4.63%** |
| Exactly 0 of 10 customers pay by credit card | **2.45%** |

Both outcomes represent statistical edge cases — reinforcing that credit card usage is consistent but not dominant. Planning staffing or POS terminals exclusively around credit card traffic would be a miscalculation.

---

### Poisson, Exponential & Uniform Distributions

Additional probability models were applied to operational scenarios:

- **Poisson (Foot Traffic):** Modeled daily transaction probability to guide shift scheduling. Extreme outlier volumes (e.g., 120 transactions/day) carry very low probability and should not drive baseline staffing decisions.
- **Exponential (Equipment Failure):** With printer jams averaging every 45 minutes, the probability of a jam within 20 minutes is non-trivial (~36%). Preventive maintenance windows should account for this.
- **Uniform (Voucher Draw):** Each of 300 receipts carries an equal 1/300 probability of winning — no receipt number has any advantage.

---

## 🧪 Part 5 — Hypothesis Formulation

### Test 1 — Member vs. Normal Customer Spend

- **H₀:** The average total spend of Member customers = the average total spend of Normal customers.
- **H₁:** The average total spend of Member customers > the average total spend of Normal customers.
- **Type:** One-tailed (directional) — the marketing team suspects Members spend *more*, not merely *differently*.

### Test 2 — Customer Rating by Gender

- **H₀:** The average customer rating for Male shoppers = the average customer rating for Female shoppers.
- **H₁:** There is a difference in average rating between Male and Female shoppers.
- **Type:** Two-tailed (non-directional) — no prior assumption is made about the direction of the difference.

---

## 🛠️ Tools & Techniques

| Category | Tool / Method |
|---|---|
| Data Cleaning | Excel Power Query |
| Descriptive Statistics | Excel (AVERAGE, MEDIAN, MODE, STDEV.S, VAR.S) |
| Visualizations | Excel Charts (Scatter, Bar, Pie, Histogram) |
| Probability Modeling | NORM.DIST, BINOM.DIST, POISSON.DIST, EXPON.DIST |
| Hypothesis Framing | One-tailed and two-tailed T-test structure |

---


*This project was completed as part of an Excel for Data Analysis course capstone, applying statistical and analytical techniques to a real-world retail operations scenario.*

---

> *"Without data, you're just another person with an opinion."* — W. Edwards Deming

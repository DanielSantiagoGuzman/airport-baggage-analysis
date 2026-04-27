# Airport Baggage Delivery Time: EDA & Probability Analysis

> **TL;DR:** Analyzed ~99K real airline baggage handling records to characterize delivery time distributions and estimate the probability of on-time delivery. After removing 18K+ invalid records, the clean dataset shows a mean delivery time of 15.5 minutes with an ~82% probability of delivering all bags within 21 minutes of landing.

---

## Business Problem

Airlines and airports use baggage delivery time as a core operational KPI. Slow delivery frustrates passengers, creates missed connections, and signals process inefficiencies in ramp operations. Understanding the *distribution* of delivery times, not just the average, enables better SLA setting, staffing decisions, and bottleneck identification.

**Key question:** What is the probability that a random flight delivers all bags within 21 minutes of landing?

---

## Dataset Overview

| Field | Description |
|---|---|
| `ActualArrival` | Timestamp of flight arrival at gate |
| `FlightNumber` | Numeric flight identifier |
| `Origin` / `Destination` | 3-letter airport codes |
| `ExpectedBagsCount` | Number of bags expected on this flight |
| `FirstBagDropTime` | Timestamp of first bag reaching baggage claim |
| `LastBagDropTime` | Timestamp of last bag reaching baggage claim |

**Derived variable:** `BaggageDeliveryTime` = `LastBagDropTime − ActualArrival` (minutes)

**Raw data:** 99,174 rows across multiple airports and dates  
**After cleaning:** 80,922 rows retained (18,252 records removed, ~18.4%)

---

## Methodology

### 1. Data Quality Assessment
Six categories of data issues were identified and quantified before removing any records:

| Issue | Count | % of Raw Data |
|---|---|---|
| Duplicate rows | 8,380 | 8.45% |
| Same origin & destination | 35 | 0.04% |
| Zero expected bags with drop times | 81 | 0.08% |
| Zero or negative delivery time | 117 | 0.12% |
| First == Last bag time (multi-bag flights) | 8,885 | 8.96% |
| Bag drop rate > 45 bags/min | 2,514 | 2.53% |

### 2. Cleaning Pipeline
Fixes applied sequentially to preserve auditability. Z-score outlier removal (±2.5 SD) applied last, after structural issues were resolved, ensuring outlier detection isn't skewed by duplicates or impossible records.

### 3. Distribution Analysis
The clean data is right-skewed (skewness ≈ 0.4), with most flights completing delivery in 11–19 minutes but a tail extending to ~32 minutes. A log-normal distribution was selected as the theoretical model because:
- Delivery time is strictly positive (log-normal is bounded at 0)
- The right-skewed shape matches log-normal geometry
- Log-transformed values are approximately normally distributed

### 4. Probability Estimation

| Approach | P(Delivery Time < 21 min) |
|---|---|
| Empirical (relative frequency) | **82.2%** |
| Theoretical (log-normal CDF) | **80.0%** |
| Difference | 2.2 percentage points |

The ~2 pp gap indicates the log-normal is a reasonable but slightly conservative fit; the model slightly underestimates the probability at the 21-minute threshold.

---

## Key Findings

- **Average delivery time:** 15.5 minutes post-landing
- **Typical range (IQR):** 11–19 minutes
- **A 21-minute SLA would be met ~80–82% of the time** under current operations
- **Hour-of-day variation exists** — peak arrival hours show marginally longer delivery times, suggesting staffing levels may not scale proportionally with flight volume
- **Data quality matters:** 18% of raw records were invalid; analysis on uncleaned data would have produced misleading results

---

## Visualizations

| Figure | Description |
|---|---|
| `fig_distribution.png` | Histogram + box plot of clean delivery times |
| `fig_before_after.png` | Distribution comparison: raw vs. cleaned dataset |
| `fig_lognormal_fit.png` | Log-normal fit assessment (PDF, log-transform, CDF comparison) |
| `fig_hourly.png` | Delivery time by hour of day (median, IQR band) |

---

## Tools & Libraries

- **Python 3**: core analysis
- **pandas**: data loading, cleaning, datetime parsing
- **NumPy / SciPy**: statistical measures, Z-scores, distribution fitting
- **Matplotlib / Seaborn**: visualization

---

## How to Run

```bash
# 1. Clone the repo
git clone https://github.com/DanielSantiagoGuzman/airport-baggage-analysis.git
cd airport-baggage-analysis
```

### 1.5. Get the data
The dataset is not tracked in this repo due to file size.
Download `Project_Data_2025.csv` and place it in the `data/` folder before running the notebook.

```bash
# 2. Install dependencies
pip install -r requirements.txt

# 3. Launch the notebook
jupyter notebook notebooks/01_eda_cleaning.ipynb
```

The notebook is self-contained; run all cells top to bottom. Output figures are saved to `outputs/`.

---

## Repository Structure

```
airport-baggage-analysis/
├── data/
│   └── Project_Data_2025.csv       # Raw dataset
├── notebooks/
│   └── 01_eda_cleaning.ipynb       # Full EDA, cleaning, and probability analysis
├── outputs/
│   ├── fig_distribution.png
│   ├── fig_before_after.png
│   ├── fig_lognormal_fit.png
│   └── fig_hourly.png
├── requirements.txt
├── .gitignore
└── README.md
```

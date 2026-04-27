# Airport Baggage Delivery Time: EDA & Performance Analysis

> **TL;DR:** Analyzed ~99K real airline baggage handling records across 108 destination airports to characterize delivery time distributions, estimate on-time delivery probability, and identify which airports are driving performance gaps. After removing 18K+ invalid records, the clean dataset shows a mean delivery time of 15.5 minutes, but a 4–5x performance gap exists between the best and worst airports in the network.

---

## Business Problem

Airlines and airports use baggage delivery time as a core operational KPI. Slow delivery frustrates passengers, creates missed connections, and signals process inefficiencies in ramp operations. Understanding the *distribution* of delivery times — not just the average enables better SLA setting, staffing decisions, and bottleneck identification.

**Key questions:**
1. What is the probability that a random flight delivers all bags within 21 minutes of landing?
2. Which airports are the best and worst performers, and by how much?
3. Does higher bag volume meaningfully slow down delivery?

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

**Raw data:** 99,174 rows · 108 destination airports · July 2021 to August 2022  
**After cleaning:** 80,922 rows retained (18,252 records removed, ~18.4%)

**Data:** The dataset was provided as part of a university course project and is not publicly redistributable. To reproduce the analysis, substitute any similarly structured flight operations dataset with the columns listed above.

---

## Notebooks

### `01_eda_cleaning.ipynb`: EDA, Cleaning & Probability Estimation
End-to-end data quality assessment, cleaning pipeline, and distribution analysis.

| Section | Content |
|---|---|
| Data Quality Assessment | Six issue categories identified and quantified before removing any records |
| Cleaning Pipeline | Sequential fixes with row count logged at each step |
| Descriptive Statistics | Mean, median, IQR, skewness, kurtosis |
| Distribution Visualization | Histogram, box plot, before/after comparison, hourly patterns |
| Probability Estimation | Empirical (82.2%) vs. theoretical log-normal (80.0%) approaches |

### `02_airport_analysis.ipynb`: Airport Performance Analysis
Airport-level rankings, tier classification, volume effects, and SQL replication with DuckDB.

| Section | Content |
|---|---|
| Airport Rankings | Median, p90, and SLA compliance for 91 qualifying airports (≥100 flights) |
| Performance Tiers | Fast / Average / Slow classification via terciles; SLA compliance by tier |
| Bag Volume Effect | Correlation analysis (r = 0.48), OLS regression, binned delivery time trends |
| DuckDB SQL | Full ranking reproduced using `RANK()`, `PERCENTILE_CONT`, `NTILE()` window functions |

---

## Key Findings

| Finding | Detail |
|---|---|
| **Average delivery time** | 15.5 minutes post-landing |
| **Typical range (IQR)** | 11–19 minutes |
| **P(< 21 minutes)** | 82.2% empirical / 80.0% log-normal theoretical |
| **Airport performance gap** | Best airports: median ~5–6 min. Worst: ~27 min — a 4–5x spread |
| **SLA compliance by tier** | Fast-tier airports: ~97% under 21 min. Slow-tier: ~55% |
| **Bag volume effect** | r = 0.48; each additional bag adds ~0.17 min to delivery time |
| **Day-of-week effect** | Minimal, weekday vs. weekend difference is ~1 minute median |
| **Data quality impact** | 18.4% of raw records were invalid; analysis on uncleaned data would produce misleading results |

---

## Methodology

### Cleaning Pipeline (Notebook 01)

| Step | Records Removed | Reason |
|---|---|---|
| Duplicate rows | 8,380 | Double-counts flights and skews all statistics |
| Same origin & destination | 35 | Operationally impossible; data entry error |
| Zero expected bags with drop times | 81 | No bags → no drop time should exist |
| Zero/negative delivery time | 117 | Bag logged before or at arrival — physically impossible |
| First == Last bag time (multi-bag) | 8,885 | Multiple bags can't all scan at the same second |
| Z-score outliers (±2.5 SD) | 734 | Statistically extreme delivery times |

### Distribution Fitting (Notebook 01)
Log-normal selected over normal because delivery time is strictly positive, the distribution is right-skewed, and log-transformed values are approximately normally distributed. The ~2 percentage point gap between empirical and theoretical estimates confirms a reasonable fit.

### Airport Analysis (Notebook 02)
Airports with fewer than 100 flights were excluded to ensure statistical reliability. Performance tiers assigned via terciles of median delivery time. Bag volume effect quantified via Pearson correlation and OLS regression across 80,922 flights.

### SQL Replication (Notebook 02)
DuckDB used to run ANSI SQL window functions directly on the pandas DataFrame `RANK()` for airport ordering, `PERCENTILE_CONT` for exact p90 computation, `NTILE(3)` for tier assignment. Results validated against pandas output. The same queries translate directly to Snowflake or Redshift in a production environment.

---

## Visualizations

| Figure | Notebook | Description |
|---|---|---|
| `fig_distribution.png` | 01 | Histogram + box plot of clean delivery times |
| `fig_before_after.png` | 01 | Distribution comparison: raw vs. cleaned dataset |
| `fig_lognormal_fit.png` | 01 | Log-normal fit assessment (PDF, log-transform, CDF comparison) |
| `fig_hourly.png` | 01 | Delivery time by hour of day with IQR band |
| `fig_airport_rankings.png` | 02 | Top 10 / Bottom 10 airports — median + p90 + SLA % |
| `fig_tiers.png` | 02 | Tier distribution and SLA compliance by tier |
| `fig_volume_effect.png` | 02 | Bag count vs. delivery time — scatter, regression, binned medians |
| `fig_heatmap.png` | 02 | All 91 airports across all four metrics simultaneously |

---

## Tools & Libraries

| Tool | Role |
|---|---|
| **pandas** | Data loading, cleaning pipeline, aggregation |
| **NumPy / SciPy** | Statistical measures, Z-scores, OLS regression, distribution fitting |
| **DuckDB** | In-process SQL — window functions, percentile computation, tier classification |
| **Matplotlib / Seaborn** | All visualizations |

---

## How to Run

```bash
# 1. Clone the repo
git clone https://github.com/DanielSantiagoGuzman/airport-baggage-analysis.git
cd airport-baggage-analysis

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add the dataset
# Place Project_Data_2025.csv in the data/ folder
# (Dataset is not tracked in this repo — see Data section above)

# 4. Run the notebooks in order
jupyter notebook notebooks/01_eda_cleaning.ipynb
jupyter notebook notebooks/02_airport_analysis.ipynb
```

Both notebooks are self-contained and run all cells top to bottom. Output figures are saved to `outputs/`.

---

## Repository Structure

```
airport-baggage-analysis/
├── data/
│   └── Project_Data_2025.csv       # Raw dataset (not tracked — see above)
├── notebooks/
│   ├── 01_eda_cleaning.ipynb       # EDA, cleaning pipeline, probability analysis
│   └── 02_airport_analysis.ipynb  # Airport rankings, tiers, volume effects, DuckDB SQL
├── outputs/
│   ├── fig_distribution.png
│   ├── fig_before_after.png
│   ├── fig_lognormal_fit.png
│   ├── fig_hourly.png
│   ├── fig_airport_rankings.png
│   ├── fig_tiers.png
│   ├── fig_volume_effect.png
│   └── fig_heatmap.png
├── requirements.txt
├── .gitignore
└── README.md
```

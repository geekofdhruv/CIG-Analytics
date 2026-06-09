# CIG-Analytics — Ride-Hailing Surge Pricing Strategy Analysis

## Executive Summary

This project delivers a comprehensive analytical framework for a leading Indian ride-hailing platform to evaluate and optimize its dynamic surge pricing strategy. Operating across **Mumbai, Bangalore, and Delhi** with **50,000 trip-level records** over a full calendar year, the analysis addresses a critical strategic inflection point: while surge pricing has driven a **9% topline revenue increase**, it has simultaneously triggered a **14% decline in customer satisfaction**, rising cancellation rates, and early churn signals.

With a **Series D fundraise** planned within 18 months, the leadership team requires evidence-based pricing recommendations that demonstrate sustainable unit economics, customer retention, and long-term enterprise value — not just short-term revenue extraction.

**Key Deliverable:** A single Jupyter notebook (`analysis.ipynb`) containing all data preparation, statistical modeling, scenario simulation, and strategic recommendation logic, with professional Excel workbooks and chart exports stored in the `output/` folder.

---

## Business Context

### The Pricing Paradox

Dynamic surge pricing is a double-edged sword in ride-hailing:

| Dimension | Current Impact | Risk Level |
|---|---|---|
| **Revenue** | +9% surge-driven uplift | Low — immediate benefit |
| **Satisfaction** | -14% QoQ decline | High — leading indicator of churn |
| **Cancellations** | Elevated in extreme surge bands | Medium — immediate revenue loss |
| **LTV/CAC** | Below 3.0x benchmark at ALL bands | Critical — fundraising risk |
| **Churn** | Incremental +0.5-2.0% per surge band | High — compounding over time |

The core challenge is identifying the **surge pricing thresholds** where incremental revenue gains begin to erode customer trust, retention, and long-term economic value — and designing a pricing architecture that optimizes the trade-off.

### Strategic Mandate

As the strategy and analytics function, the mandate is threefold:

1. **Diagnose** — Quantify where and when surge pricing damages customer behavior
2. **Model** — Estimate demand elasticity, churn sensitivity, and unit economics across segments
3. **Recommend** — Simulate pricing scenarios and identify the policy that maximizes long-term enterprise value

---

## Dataset

### Overview

The analysis uses a **trip-level dataset** representing ride-hailing activity across three major Indian metros over a full calendar year.

- **Records:** 50,000 trips
- **Cities:** Mumbai, Bangalore, Delhi
- **Time Period:** Full calendar year
- **Granularity:** Trip-level with customer, financial, and operational dimensions

### Data Source

| Source | Location | Access |
|---|---|---|
| **Google Sheets (Reference)** | [India Rideshare Dataset](https://docs.google.com/spreadsheets/d/1saBuyrZ9JqZ4gMfUS9XPXMFbl9BN5Erj/edit?usp=sharing&ouid=116405649123253020989&rtpof=true&sd=true) | Online view/download |
| **Local Copy** | `Dataset/Copy of india_rideshare_dataset.xlsx` | Working directory |

> **Note:** The dataset is excluded from version control via `.gitignore` to keep the repository lightweight. Download the dataset from the Google Sheets link above and place it in the `Dataset/` directory.

### Data Dictionary

The dataset contains **41 fields** organized into four categories:

#### Operational Fields
| Field | Description | Type |
|---|---|---|
| `trip_id` | Unique trip identifier | String |
| `customer_id` | Unique customer identifier | String |
| `city` | Metro city (Mumbai, Bangalore, Delhi) | Categorical |
| `source_zone` | Pickup zone identifier | String |
| `destination_zone` | Drop-off zone identifier | String |
| `zone_type` | Zone classification (residential, commercial, transit_hub) | Categorical |
| `cab_type` | Vehicle category | Categorical |
| `service_tier` | Service level (economy, premium, etc.) | Categorical |
| `datetime` | Trip timestamp | DateTime |
| `hour` | Hour of day (0-23) | Integer |
| `day` | Day of month | Integer |
| `month` | Month (1-12) | Integer |
| `weekday` | Day of week (Mon=0 .. Sun=6) | Integer |
| `is_peak_hour` | Flag for peak demand hours | Binary |
| `is_weekend` | Flag for Saturday/Sunday | Binary |
| `latitude` / `longitude` | Geographic coordinates | Float |
| `distance_km` | Trip distance in kilometers | Float |
| `surge_multiplier` | Dynamic pricing multiplier | Float |
| `fare_inr` | Total fare charged to customer | Float |
| `ride_completed` | Completion status (1=completed, 0=cancelled) | Binary |

#### Customer Fields
| Field | Description | Type |
|---|---|---|
| `customer_segment` | Customer classification | Categorical |
| `loyalty_tier` | Membership tier (Gold, Silver, Bronze, None) | Categorical |
| `satisfaction_score` | Post-trip satisfaction rating (1-5) | Float |
| `days_since_last_ride` | Recency metric | Integer |
| `churn_risk_flag` | ML-predicted churn probability flag | Binary |

#### Financial Fields
| Field | Description | Type |
|---|---|---|
| `driver_payout_inr` | Amount paid to driver | Float |
| `platform_revenue_inr` | Platform's share of fare | Float |
| `platform_take_rate` | Platform's percentage cut | Float |
| `cost_per_trip_inr` | Operational cost per trip | Float |
| `contribution_margin_inr` | `platform_revenue - driver_payout - cost_per_trip` | Float |
| `cac_inr` | Customer acquisition cost | Float |

#### External Variables
| Field | Description | Type |
|---|---|---|
| `temperature_c` | Ambient temperature | Float |
| `apparent_temperature_c` | Feels-like temperature | Float |
| `humidity` | Relative humidity | Float |
| `wind_speed_kmh` | Wind speed | Float |
| `precip_probability` | Chance of precipitation | Float |
| `precip_intensity` | Precipitation intensity | Float |
| `cloud_cover` | Cloud coverage percentage | Float |
| `visibility_km` | Visibility distance | Float |
| `weather_condition` | Categorical weather state | Categorical |

### Data Quality

- **Missing Values:** Only `loyalty_tier` has missing values (18,852 of 50,000 = 37.7%). These are imputed as `'None'` (valid customers without loyalty membership) and a `loyalty_missing` flag is created.
- **Outliers:** IQR capping applied to fare, distance, contribution margin, driver payout, and platform revenue (2,454-3,296 outliers per column).
- **Sample Size Validation:** 12 sparse groups (city x zone x time x surge band with <30 trips) flagged as unreliable for cancellation rate analysis.

---

## Core Business Question

> **What pricing strategy maximizes long-term enterprise value while balancing revenue growth, contribution profitability, customer retention, and competitive positioning?**

More specifically:

1. At what **surge multiplier thresholds** does demand suppression become statistically significant?
2. What is the **price elasticity of demand** across customer segments, zone types, and time conditions?
3. How does **surge-driven churn** impact customer lifetime value (LTV) and the LTV/CAC ratio?
4. What is the **optimal surge pricing architecture** (caps, segmentation, loyalty overlays) that maximizes contribution margin while keeping LTV/CAC above the 3.0x benchmark?

---

## Analytical Framework

The analysis is organized into four workstreams, each building on the previous:

### Workstream 1 — Demand Suppression and Customer Response Analysis

**Objective:** Identify city-zone-time combinations where surge pricing causes measurable deterioration in customer behavior.

**Key Analyses:**
- Baseline metrics under non-surge conditions (surge_multiplier = 1.0)
- Comparative metrics across surge bands (Low 1.01-1.3x, Medium 1.31-1.7x, High 1.71-2.5x, Extreme >2.5x)
- Delta analysis: surge vs. baseline for completion rates, satisfaction, churn risk
- Statistical significance testing (Chi-square for completion rates, t-tests for satisfaction and churn)
- Logistic regression for cancellation drivers (odds ratios by feature)

**Key Output:** Heatmaps, bar charts, violin plots, and an odds-ratio plot showing that surge multiplier is the strongest predictor of cancellation (OR = 12.75, p << 0.001).

### Workstream 2 — Demand Sensitivity and Pricing Elasticity Assessment

**Objective:** Estimate demand sensitivity to fare increases across segments, zones, and conditions.

**Key Analyses:**
- Logistic regression with interactions (log_fare x zone_type, log_fare x peak, log_fare x weekend)
- Linear Probability Model (LPM) for direct percentage-point interpretation
- Marginal effects: impact of 10% fare increase on completion probability
- Segment-specific elasticities by customer segment, loyalty tier, zone x peak, and zone x weekday

**Key Output:** 5 charts showing predicted completion vs. fare, elasticity by segment, and fare distributions. **Critical finding:** All 19 price elasticities are statistically insignificant (p > 0.4), likely due to 96% overall completion rate leaving minimal variance.

### Workstream 3 — Unit Economics and LTV Stress Test

**Objective:** Assess the financial and retention economics of surge pricing at the band level.

**Key Analyses:**
- Contribution margin decomposition by surge band (fare, revenue, driver payout, cost, CM)
- Churn sensitivity analysis using a proxy indicator (churn_risk_flag OR days_since_last_ride > 60 + low satisfaction)
- Breakeven churn threshold: maximum churn rate where LTV_surge >= LTV_baseline
- LTV/CAC sensitivity across 5 churn scenarios (+0%, +10%, +20%, +50%, +100%)

**Key Output:** Professional Excel workbook (`unit_economics_workbook.xlsx`) with 6 sections, and 4 visualizations.

**Critical Finding:** LTV/CAC < 3.0x at **ALL** surge bands and **ALL** churn scenarios. The business is below the VC health benchmark. Root cause: CAC (374.77 INR) is high relative to per-trip contribution margin.

### Workstream 4 — Pricing Scenario Simulation

**Objective:** Simulate the impact of alternative surge pricing policies and identify the optimal strategy.

**Scenarios Tested:**

| Scenario | Description | Surge Cap |
|---|---|---|
| **Status Quo** | No policy change | None |
| **Calibrated Optimum** | Universal cap at 2.0x | 2.0x all zones |
| **Segmented Caps** | Zone-specific limits | Res 1.5x / Com 2.2x / Trn 1.8x |
| **Loyalty Protection** | Gold/Silver capped at 1.3x | 1.3x for loyalty tiers |
| **Retention Focus** | Aggressive cap at 1.5x | 1.5x all zones |

**Simulation Engine:**
- Demand response: `D_new = D0 x (P_new/P0)^elasticity` (elasticities from WS2)
- Churn response: `churn = 0.03 + 0.0015 x (avg_surge - 1)` (slope from WS3)
- Financial recomputation: new fare, new revenue, new CM, new LTV
- Zone-level detail for residential, commercial, and transit_hub

**Key Output:** Professional Excel workbook (`pricing_simulation.xlsx`) with scenario comparison, zone details, and strategic recommendation.

**Recommended Strategy:** Cap surge at **2.0x universally** + protect loyalty tier customers at **1.3x**. This preserves 85%+ of surge CM uplift while reducing churn risk and keeping LTV/CAC closest to the 3.0x benchmark.

---

## Project Structure

```
CIG-Analytics/
├── Dataset/                           # Raw data (gitignored)
│   └── Copy of india_rideshare_dataset.xlsx
├── output/                            # All generated outputs
│   ├── unit_economics_workbook.xlsx   # WS3: Professional Excel export
│   ├── pricing_simulation.xlsx        # WS4: Professional Excel export
│   ├── workstream1_heatmaps.png       # WS1: Surge vs baseline heatmaps
│   ├── workstream1_cancellation_bars.png
│   ├── workstream1_satisfaction_violin.png
│   ├── workstream1_completion_by_city.png
│   ├── workstream1_churn_heatmap.png
│   ├── workstream1_odds_ratios.png
│   ├── workstream2_completion_vs_fare_by_zone.png
│   ├── workstream2_peak_vs_weekend.png
│   ├── workstream2_elasticity_bar.png
│   ├── workstream2_segment_elasticity.png
│   ├── workstream2_fare_distribution.png
│   ├── workstream3_cm_by_band.png
│   ├── workstream3_ltv_decomposition.png
│   ├── workstream3_churn_vs_cm.png
│   ├── workstream3_ltv_cac_heatmap.png
│   ├── workstream4_cm_vs_surge.png
│   ├── workstream4_ltv_cac_vs_churn.png
│   ├── workstream4_demand_vs_revenue.png
│   ├── workstream4_multi_kpi.png
│   ├── workstream4_zone_cm_comparison.png
│   └── workstream4_zone_surge_comparison.png
├── venv/                              # Python virtual environment
├── analysis.ipynb                     # Main analysis notebook (all workstreams)
├── requirements.txt                   # Python dependencies
├── .gitignore                         # Git exclusions
└── README.md                          # This file
```

---

## Technical Implementation

### Environment Setup

```bash
# 1. Clone the repository
git clone <repo-url>
cd CIG-Analytics

# 2. Create virtual environment
python -m venv venv

# 3. Activate (Windows)
venv\Scripts\activate
#    Activate (macOS/Linux)
source venv/bin/activate

# 4. Install dependencies
pip install -r requirements.txt

# 5. Register Jupyter kernel
python -m ipykernel install --user --name=cig-analytics

# 6. Download dataset and place in Dataset/ folder
#    Use the Google Sheets link in the Dataset section above

# 7. Launch Jupyter
jupyter notebook analysis.ipynb
#    Or execute directly via nbconvert:
#    jupyter nbconvert --to notebook --execute --ExecutePreprocessor.kernel_name=cig-analytics analysis.ipynb
```

### Dependencies

All dependencies are pinned in `requirements.txt` and installed in the `venv/` directory:

- **pandas** — Data manipulation and analysis
- **numpy** — Numerical computing
- **matplotlib** — Static charting (inline `plt.show()` — no `Agg` backend)
- **seaborn** — Statistical visualization
- **openpyxl** — Excel file generation
- **scipy** — Statistical tests (chi-square, t-tests)
- **statsmodels** — Logistic regression and linear models
- **jupyter** — Notebook execution environment

### Key Technical Decisions

| Decision | Rationale |
|---|---|
| **Single notebook** | Simplifies execution, review, and version control. All 4 workstreams in one file. |
| **Inline `plt.show()`** | Charts render inline for notebook viewing; `plt.savefig()` exports to `output/` for external use. |
| **Compact chart sizes** | All charts are ~40% smaller than matplotlib defaults for dense notebooks. |
| **IQR outlier capping** | 1.5x IQR rule applied to 5 financial columns. Preserves ~95% of data. |
| **MIN_SURGE_N = 30** | Groups below 30 trips excluded from WS1 displays. Prevents noisy cancellation rates. |
| **LPM alongside logit** | Linear Probability Model provides direct percentage-point interpretation. |
| **3% baseline churn** | Consistent with industry benchmarks for Indian ride-hailing. |
| **Single CAC average** | Dataset mean of 374.77 INR. Simplification; real CAC varies by segment. |

### Notebook Execution

The notebook is designed for **sequential execution** from top to bottom:

1. **Cell 1 (Data Preparation)** — Loads, cleans, engineers features, saves `df_clean`
2. **Cells 2-8 (WS1)** — Baseline, surge behavior, significance tests, logistic regression
3. **Cells 9-14 (WS2)** — Elasticity estimation, segment analysis, visualizations
4. **Cells 15-20 (WS3)** — CM analysis, churn sensitivity, LTV/CAC, Excel export
5. **Cells 21-25 (WS4)** — Baseline metrics, scenario engine, visualizations, Excel export

**Critical:** All cells must be executed in order. Later workstreams depend on variables (`df_clean`, `ELASTICITY`, `CHURN_SLOPE`, `CAC`, etc.) created in earlier cells.

---

## Key Findings and Recommendations

### 1. Surge Pricing Has Significant Negative Externalities

- **Surge multiplier** is the strongest predictor of cancellation (OR = 12.75 in logistic regression)
- Completion rates drop from ~96% baseline to ~85% in Extreme surge (>2.5x)
- Satisfaction scores decline monotonically with surge band

### 2. Price Elasticity Is Extremely Low

- All 19 segment-specific price elasticities are **statistically insignificant** (p > 0.4)
- This is likely due to the **96% overall completion rate** leaving very little variance to model
- Implication: customers are relatively **price-inelastic** for ride-hailing in Indian metros

### 3. LTV/CAC Is Below Benchmark at ALL Surge Levels

- **LTV/CAC < 3.0x** across all surge bands and all churn scenarios
- Industry benchmark for VC-backed platforms: **LTV/CAC > 3.0x**
- Root cause: **CAC is too high** (374.77 INR) relative to per-trip contribution margin
- **Strategic implication:** The real lever is **reducing CAC** or **improving retention**, not surge optimization

### 4. Recommended Pricing Architecture

| Policy | Recommendation | Rationale |
|---|---|---|
| **Universal Cap** | **2.0x maximum** | Preserves 85%+ of surge CM; eliminates extreme surge churn |
| **Loyalty Overlay** | **1.3x for Gold/Silver** | Protects highest-LTV segment with minimal revenue impact |
| **Segmentation** | Simple, not complex | Zone-specific caps are operationally burdensome for marginal gain |
| **Retention Floor** | Avoid 1.5x universal | Sacrifices too much margin for limited churn benefit |

### 5. Immediate Actions for Series D Readiness

1. **Implement 2.0x surge cap** with 1.3x loyalty overlay
2. **Investigate CAC reduction** — referral programs, organic growth, lower-funnel marketing efficiency
3. **Improve baseline retention** — satisfaction recovery programs, driver quality incentives
4. **A/B test** the recommended policy in one city before national rollout
5. **Build longitudinal churn tracking** — the current proxy is cross-sectional and approximate

---

## Limitations and Caveats

1. **Churn is a proxy**, not observed longitudinal behavior. The indicator combines churn_risk_flag + recency/satisfaction signals.
2. **Surge is endogenous** — it activates during high demand, creating attenuation bias in elasticity estimates.
3. **LTV model assumes constant CM and churn**; reality is path-dependent and customer-specific.
4. **Single CAC average** masks significant segment variation (new vs. returning customers, city differences).
5. **Cross-sectional data** — no time-series panel structure to track the same customer over time.
6. **96% completion rate** compresses variance, making elasticity estimation statistically underpowered.

---

## License and Usage

This analysis is proprietary and intended for internal strategic decision-making. The dataset is synthetic/purpose-built for the case and should not be used for production model training without validation against real operational data.

---

## Contact

For questions or issues with the analysis notebook, reach out to the strategy and analytics team.

**Project:** CIG-Analytics  
**Date:** June 2026  
**Status:** Complete — all 4 workstreams executed, Excel exports generated, recommendations delivered

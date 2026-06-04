# CIG-Analytics — Ride-Hailing Pricing Strategy Analysis

## Background

A leading ride-hailing platform operating across Mumbai, Bangalore, and Delhi is facing a strategic pricing inflection point. Over the last two quarters, dynamic surge pricing has contributed to a **9% increase in topline revenue**. However, the same period has seen a **14% decline in customer satisfaction**, higher cancellation rates, and early indications of customer churn.

With a Series D fundraise planned within the next 18 months, leadership must demonstrate not just revenue growth, but sustainable unit economics, customer retention, and long-term business quality. The executive team has engaged the strategy and analytics function to assess whether the current pricing model is maximizing enterprise value—or creating hidden structural risk.

As part of the strategy and analytics team, the mandate is to diagnose the pricing problem and recommend a commercially viable pricing strategy.

## Dataset

The analysis is conducted using a purpose-built trip-level dataset representing ride-hailing activity across Mumbai, Bangalore, and Delhi over a full calendar year (50,000 records).

**Dataset Reference (Google Sheets):** [India Rideshare Dataset](https://docs.google.com/spreadsheets/d/1saBuyrZ9JqZ4gMfUS9XPXMFbl9BN5Erj/edit?usp=sharing&ouid=116405649123253020989&rtpof=true&sd=true)

**Local Copy:** `Dataset/Copy of india_rideshare_dataset.xlsx`

> **Note:** The dataset is excluded from version control via `.gitignore`. Use the Google Sheets link above to access the data.

### Available Data Fields

| Category | Fields |
|---|---|
| **Operational** | Trip timestamps, route information, city, zone type, distance, ride completion status, surge multiplier, fare |
| **Customer** | Customer segment, loyalty tier, days since last ride, churn risk indicators, satisfaction scores |
| **Financial** | Driver payout, platform revenue, take rate, trip costs, contribution margin, CAC |
| **External Variables** | Weather conditions — precipitation, visibility, humidity, temperature |

The dataset requires preprocessing, cleaning, and feature engineering before analysis.

## Core Business Question

> What pricing strategy maximizes long-term enterprise value while balancing revenue growth, contribution profitability, customer retention, and competitive positioning?

Specifically — determine the **surge pricing thresholds** at which incremental revenue gains begin to erode customer trust, retention, and long-term economic value.

---

## Workstreams

### Workstream 1 — Demand Suppression and Customer Response Analysis

Identify the city, zone, and time-window combinations where surge pricing is associated with measurable deterioration in customer behavior:

- Increased cancellations
- Reduced ride completion rates
- Worsening satisfaction scores
- Elevated churn indicators

The analysis should distinguish **normal demand fluctuations** from **pricing-driven behavioral deterioration** using comparable baseline conditions.

### Workstream 2 — Demand Sensitivity and Pricing Elasticity Assessment

Estimate demand sensitivity to fare increases across customer segments, zone types, and operating conditions. Assess how ride demand responds to surge-driven fare increases across:

- Residential, commercial, and transit zones
- Peak vs off-peak periods
- Weekday vs weekend conditions

Where relevant, **control for observable external factors** such as weather conditions.

> All assumptions, methodology, and limitations must be clearly stated.

---

## Project Structure

```
CIG-Analytics/
├── Dataset/                        # Raw data (gitignored)
├── notebooks/                      # Jupyter notebooks for analysis
├── src/                            # Python modules and utilities
├── output/                         # Generated results and visualizations
├── .gitignore
└── README.md
```

## Getting Started

1. Clone the repository
2. Download the dataset from the [Google Sheets link](https://docs.google.com/spreadsheets/d/1saBuyrZ9JqZ4gMfUS9XPXMFbl9BN5Erj/edit?usp=sharing&ouid=116405649123253020989&rtpof=true&sd=true) and place it in the `Dataset/` directory
3. Create a virtual environment: `python -m venv venv`
4. Activate: `venv\Scripts\activate` (Windows) or `source venv/bin/activate` (macOS/Linux)
5. Install dependencies: `pip install -r requirements.txt`

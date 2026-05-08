# Generation Interconnection Queue Viability

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rohanmyers/queue-viability/blob/main/notebooks/01_data_inventory.ipynb)
![Python](https://img.shields.io/badge/Python-3.10+-blue)
![License](https://img.shields.io/badge/License-MIT-green)

A machine learning tool that predicts whether a proposed U.S. power plant will actually get built by scoring every active project in the national interconnection queue with a calibrated probability and regional confidence rating.

**Headline finding: Of 1,703 GW of announced generation capacity outside Texas, the model predicts 92% will never reach commercial operation.**

---

## The Problem

Every power plant seeking to connect to the U.S. electrical grid must join an interconnection queue managed by a regional grid operator (ISO/RTO). These queues have exploded in size. As of 2024, they contain roughly 2,300 GW of proposed capacity, more than double the country's entire existing generating fleet. But historically, only 13% of projects entering a queue between 2000 and 2019 ever reached commercial operation (LBNL, 2025). The other 87% were withdrawn or remain indefinitely "active" despite having no realistic prospect of being built.

This phantom capacity problem has real consequences. Utilities plan generation and transmission infrastructure around announced queue projects. When those projects evaporate, ratepayers are left holding the cost of infrastructure built for demand that never materialized. At the same time, the rapid growth of AI data centers is driving unprecedented electricity load growth, making it more critical than ever to distinguish real supply additions from speculative filings.

Regulators at FERC, state public utility commissions, and ISO planning teams currently have no standardized, evidence-based method to assess which queued projects are credible. This project attempts to creat one.

---

## What the Model Does

For each active generation interconnection request, the model outputs:

- **Viability score (0–1):** A calibrated probability that the project will reach commercial operation, based on patterns learned from 22,587 historical queue outcomes
- **Confidence flag:** `validated` (model performance confirmed via bootstrap CI) or `unvalidated` (insufficient test data in that region)

The top 10% of projects ranked by the model capture **31% of all operational outcomes** — a 3.1x lift over random selection.

---

## Key Findings

### 1. The queue overstates buildable capacity by 87–92%
- **Scenario A (all regions):** 261 GW expected of 2,061 GW announced — 87% attrition
- **Scenario B (validated regions only):** 61 GW expected of 844 GW — 93% attrition
- **Scenario C (ex-ERCOT, most defensible):** 132 GW expected of 1,703 GW — **92% attrition**

ERCOT predictions are flagged as unreliable due to sparse training data; Scenarios B and C are the more trustworthy estimates.

### 2. Developer track record is the strongest predictor
Developers with established completion histories complete projects at roughly **2x the baseline rate** (34% vs. 17%). Projects from unknown or first-time developers, 76% of the active queue, are scored on fuel type, region, size, and queue conditions alone.

### 3. Zombie projects inflate announced capacity
Projects entering the queue before 2010 and still technically "active" today score near 0.0 viability with near certainty. Five such projects alone (a 418 MW facility in Florida, three Oregon wind projects, and a South Dakota wind project) represent over 1,500 MW of phantom capacity that FERC's Order 2023 deposit requirements were designed to flush out.

### 4. Smaller projects complete more often
Projects under 50 MW complete at 26% vs. 11% for projects over 500 MW, which is a clean monotonic relationship the model exploits across both the continuous log-MW feature and a binned capacity class.

### 5. Completion rates have been falling
The test cohort (2018–2019) had a 13.4% positive rate vs. 21.7% in training (2010–2017), reflecting real-world queue congestion growth — not a modeling artifact.

---

## Model Performance

| Metric | Value |
|---|---|
| Algorithm | XGBoost binary classifier + isotonic calibration |
| Training set | 12,100 rows, 2010–2017 cohort, 21.7% positive rate |
| Test set | ~3,180 rows, 2018–2019 cohort, 13.4% positive rate |
| Test AUC | **0.748** |
| Calibrated Brier score | **0.103** (vs. 0.214 uncalibrated) |
| Top-decile lift | **3.1x** (top 10% captures 31% of operational outcomes) |
| Baseline AUC (region only) | 0.610 |
| Baseline AUC (logistic regression) | 0.718 |

### Regional reliability (bootstrap 95% CI)

| Region | n (test) | AUC | 95% CI | Status |
|---|---|---|---|---|
| Southeast | 497 | 0.878 | [0.834, 0.922] | ✅ Validated |
| West | 604 | 0.780 | [0.723, 0.837] | ✅ Validated |
| ISO-NE | 162 | 0.769 | [0.682, 0.839] | ✅ Validated |
| PJM | 824 | 0.679 | [0.622, 0.725] | ✅ Validated |
| MISO | 344 | 0.541 | [0.430, 0.671] | ⚠️ Inconclusive |
| ERCOT | 219 | 0.499 | [0.423, 0.582] | ⚠️ Inconclusive |
| NYISO | 223 | 0.428 | [0.264, 0.612] | ⚠️ Inconclusive |
| CAISO | 166 | 0.405 | [0.168, 0.613] | ⚠️ Inconclusive |
| SPP | 139 | N/A | — | ⚠️ No test positives |

Inconclusive CIs span 0.5 — data is consistent with random performance, not necessarily reliable bad performance. More data needed.

---

## Data Sources

| Dataset | Source | Use |
|---|---|---|
| LBNL Queued Up 2025 Edition | [emp.lbl.gov/queues](https://emp.lbl.gov/queues) | Training labels, features, live scoring set |
| EIA Table 4 — Average Retail Price by State | [eia.gov/electricity/sales_revenue_price](https://www.eia.gov/electricity/sales_revenue_price/) | Industrial electricity price feature |

All datasets are free and publicly available. No authentication required.

---

## Repository Structure

```
queue-viability/
├── notebooks/
│   ├── 01_data_inventory.ipynb          # Download and verify all raw data
│   ├── 02_eda_queue_outcomes.ipynb      # EDA, label feasibility, go/no-go
│   ├── 03_feature_engineering.ipynb     # Internal feature construction
│   ├── 04_external_joins.ipynb          # EIA price join, train/test split
│   ├── 05_baseline_and_xgboost.ipynb    # Logistic regression + XGBoost
│   ├── 06_shap_calibration.ipynb        # SHAP, calibration, regional eval
│   └── 07_score_live_queue.ipynb        # Score all active projects
├── models/
│   ├── xgboost_v1.json                  # Saved XGBoost model
│   ├── isotonic_calibrator.pkl          # Calibration model
│   ├── feature_list_v1.json             # Feature metadata
│   └── model_card_v1.md                 # Model card with limitations
├── data/
│   ├── raw/                             # Downloaded source files (not committed)
│   ├── interim/                         # Cleaned intermediate files
│   └── processed/                       # Model-ready parquets + scored CSV
├── reports/
│   ├── figures/                         # All saved charts and SHAP plots
│   ├── executive_summary.md             # 2-page non-technical summary
│   ├── feasability.md                   # Determine whether or not there is a real use case
│   ├── day5_results.md                  # Model training results
│   ├── day6_results.md                  # Evaluation and calibration results
│   └── day7_results.md                  # Live queue scoring results
```

---

## How to Run

All notebooks are designed for Google Colab + Google Drive. Each notebook mounts Drive for persistent storage and installs required packages on startup.

**1. Clone the repo**
```bash
git clone https://github.com/rohanmyers/queue-viability.git
```

**2. Download data manually** (see `data/README.md` for links and filenames) and place in your Google Drive at `My Drive/queue-viability/data/raw/`

**3. Run notebooks in order** using the Open in Colab badges at the top of each notebook

**4. Score the live queue** — run notebook 07 to apply the trained model to all active projects and generate `data/processed/active_queue_scored.csv`

### Requirements
```
pandas>=2.0
numpy>=1.24
xgboost>=2.0
scikit-learn>=1.3
shap>=0.44
geopandas>=0.14
pyarrow>=14.0
openpyxl>=3.1
matplotlib>=3.7
seaborn>=0.13
```

---

## Deliverable

The primary output is `data/processed/active_queue_scored.csv` — a scored, ranked list of all ~10,430 active and suspended generation interconnection requests with:

- `viability`: Calibrated probability of reaching commercial operation (0–1)
- `confidence`: `validated` or `unvalidated` based on regional model reliability
- `region`, `state`, `fuel_bucketed`, `mw_total`, `developer`, `q_year`

---

## Known Limitations

See `models/model_card_v1.md` for full documentation. Key points:

- **ERCOT predictions are unreliable.** LBNL training data for ERCOT is sparse (~150 training rows, primarily 2015–2017) and skewed toward completed projects. ERCOT viability scores are flagged `unvalidated`.
- **Post-2019 projects have immature labels.** Labels are based on observed outcomes (operational or withdrawn). Projects entering 2020–2024 are in the live scoring set, not the training set, because most haven't resolved yet.
- **Developer name matching is approximate.** Developer names in LBNL data are inconsistent. Unknown or first-time developers receive NaN for track-record features; the model routes these via its missing-value path.
- **Industrial electricity price is underestimated in deregulated states.** EIA Table 4 published prices are used, which correctly handle regulated + competitive supplier reporting. Earlier Form 861-only estimates for Ohio, Pennsylvania, and Illinois were replaced with the official published values.
- **No transmission proximity feature.** Distance to nearest high-voltage substation was excluded due to data availability constraints (HIFLD Open Data decommissioned). This would likely improve regional predictions.
- **LBNL queue IDs are not globally unique.** The same numeric ID appears in multiple ISO regions. All project-level analysis uses a composite identifier (queue_id + region + state + queue_year).

---

## References

- Rand et al. (2025). *Queued Up: 2025 Edition.* Lawrence Berkeley National Laboratory. https://emp.lbl.gov/queues
- Gorman et al. (2024). Grid connection barriers to renewable energy deployment in the United States. *Joule.* https://doi.org/10.1016/j.joule.2024.101791
- FERC Order No. 2023. Generator Interconnection Process Reform. https://www.ferc.gov/explainer-interconnection-final-rule-2023-A
- MIT CEEPR (2023). FERC Order 2023: Will It Make a Difference? https://ceepr.mit.edu/wp-content/uploads/2023/12/MIT-CEEPR-RC-2023-06.pdf
- Grid Strategies (2025). National Load Growth Report. https://gridstrategiesllc.com

---

## License

MIT — see `LICENSE` for details.

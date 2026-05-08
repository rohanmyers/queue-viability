# Model Card: Generation Interconnection Queue Viability Model v1

**Model ID:** `xgboost_v1.json` + `isotonic_calibrator.pkl`
**Version:** 1.0
**Date:** May 2026
**Author:** Rohan Myers
**Repository:** github.com/rohanmyers/queue-viability

---

## Model Description

### What this model does

This model predicts the probability that a U.S. generation interconnection request, a proposed power plant seeking permission to connect to the electrical grid, will reach commercial operation. It outputs a calibrated probability score from 0 to 1 for each active project in a national interconnection queue dataset.

### Algorithm

XGBoost binary classifier (`xgboost_v1.json`) with post-hoc isotonic probability calibration (`isotonic_calibrator.pkl`). The model was trained using `scale_pos_weight=3.6` to handle class imbalance (approximately 22% positive rate in training). Probabilities are calibrated using isotonic regression fit on a held-out calibration split of the test set.

### Intended use

Decision support for utility regulators, ISO/RTO planning staff, state public utility commission analysts, FERC analysts, and ratepayer advocates evaluating the credibility of announced generation capacity. The model is designed to supplement, not replace, expert judgment. It should not be used as the sole basis for regulatory decisions.

### Out-of-scope uses

- **Do not use as the sole basis for denying a developer's interconnection request.** The model identifies population-level patterns; any individual project may deviate from those patterns for legitimate reasons.
- **Do not use in ERCOT for high-stakes decisions.** ERCOT predictions are explicitly flagged as unreliable (see Regional Performance section).
- **Do not use for load-side (data center) interconnection requests.** The model was trained exclusively on generation interconnection data. Load queue viability is a different prediction problem requiring different data.

---

## Training Data

### Source
Lawrence Berkeley National Laboratory Queued Up 2025 Edition, Sheet "03. Complete Queue Data."
Dataset URL: https://emp.lbl.gov/queues

U.S. EIA has state-level industrial retail prices
Dataset URL: https://www.eia.gov/electricity/sales_revenue_price/

### Scope
All generation and surplus interconnection requests across 7 U.S. ISOs and 49 non-ISO balancing authorities, representing approximately 97% of U.S. installed generating capacity.

### Filtering
- Kept only `project_type` = "Generation" or "Surplus" (excluded Upgrade and Replacement)
- Restricted to `q_year` 2010–2017 for training (outcomes mature enough to label reliably)
- `q_year` 2018–2019 held out as test set
- `q_year` 2020+ reserved as live scoring set (outcomes not yet mature)

### Label construction
- **Y = 1 (Operational):** `q_status` = "operational" — project reached commercial operation
- **Y = 0 (Withdrawn):** `q_status` = "withdrawn" — project formally withdrew from queue
- Excluded: `q_status` = "active", "suspended", "unknown" — outcome not yet observed

### Training set statistics
- Total rows: 12,100
- Positive (operational): 2,628 (21.7%)
- Negative (withdrawn): 9,472 (78.3%)
- Queue years: 2010–2017
- ISOs represented: All 9 (PJM, West, Southeast, MISO, CAISO, ISO-NE, NYISO, ERCOT, SPP)

---

## Features

| Feature | Type | Source | Description |
|---|---|---|---|
| `fuel_bucketed` | Categorical | LBNL | Primary fuel type (Solar, Wind, Battery, Gas, Hydro, etc.). Rare types grouped into "Other_combined." |
| `region` | Categorical | LBNL | ISO/RTO region (PJM, MISO, CAISO, West, Southeast, ISO-NE, NYISO, ERCOT, SPP) |
| `service` | Categorical | LBNL | Interconnection service type: ERIS (energy-only), NRIS (network), NRIS/ERIS (hybrid), Other |
| `project_type` | Categorical | LBNL | Generation or Surplus. Surplus projects scored via model's missing-value path (see Known Issues). |
| `capacity_class` | Categorical | LBNL (derived) | Binned MW: small (<50), medium (50–200), large (200–500), huge (>500) |
| `log_mw` | Numeric | LBNL (derived) | Log-transformed total capacity: log(1 + mw_total). Reduces right-skew. |
| `q_year` | Numeric | LBNL | Calendar year of queue entry |
| `q_year_region_volume` | Numeric | LBNL (derived) | Count of all projects entering the same ISO in the same year (proxy for queue congestion) |
| `cluster_size` | Numeric | LBNL (derived) | Number of projects in the same interconnection cluster. Projects with no cluster assignment receive cluster_size=1. |
| `dev_prior_n` | Numeric | LBNL (derived) | Number of prior interconnection requests by this developer that resolved before this project's queue date |
| `dev_prior_completion_rate` | Numeric | LBNL (derived) | Developer's historical completion rate among prior resolved projects; Bayesian-smoothed toward global mean when n<3. NaN for unknown developers. |
| `has_dev_history` | Binary | LBNL (derived) | 1 if developer had any prior resolved projects; 0 otherwise |
| `industrial_price_cents_kwh` | Numeric | EIA Table 4 | State-level average industrial retail electricity price (cents/kWh), 2024 values |

### Feature importance (mean absolute SHAP, test set)

| Rank | Feature | Mean |SHAP| |
|---|---|---|
| 1 | region | 0.111 |
| 2 | fuel_bucketed | 0.107 |
| 3 | log_mw | 0.094 |
| 4 | service | 0.079 |
| 5 | q_year_region_volume | 0.049 |
| 6 | capacity_class | 0.048 |
| 7 | dev_prior_completion_rate | 0.047 |
| 8 | has_dev_history | 0.043 |
| 9 | dev_prior_n | 0.041 |
| 10 | q_year | 0.038 |

Developer track record features collectively account for ~13% of mean absolute SHAP despite being available for only ~11% of rows, indicating strong signal where data exists.

---

## Evaluation

### Test set
- 2018–2019 queue cohort
- n = 3,178 projects
- Positive rate: 13.4% (lower than training's 21.7%, reflecting real-world declining completion rates)

### Overall performance

| Metric | Value | Notes |
|---|---|---|
| Test AUC | 0.748 | Model correctly ranks 75% of project pairs |
| Calibrated Brier score | 0.103 | After isotonic calibration |
| Uncalibrated Brier score | 0.214 | Before calibration |
| Average Precision (AP) | 0.337 | 2.5x lift over naive baseline |
| Top-10% lift | 3.1x | Top decile captures 31% of all operational outcomes |
| Logistic regression AUC | 0.718 | Baseline for comparison |
| Region-only baseline AUC | 0.610 | Weakest meaningful baseline |

### Regional performance (bootstrap 95% CIs, n=500 resamples)

| Region | n | AUC | 95% CI | Validated? |
|---|---|---|---|---|
| Southeast | 497 | 0.878 | [0.834, 0.922] | ✅ Yes |
| West | 604 | 0.780 | [0.723, 0.837] | ✅ Yes |
| ISO-NE | 162 | 0.769 | [0.682, 0.839] | ✅ Yes |
| PJM | 824 | 0.679 | [0.622, 0.725] | ✅ Yes |
| MISO | 344 | 0.541 | [0.430, 0.671] | ⚠️ Inconclusive |
| ERCOT | 219 | 0.499 | [0.423, 0.582] | ⚠️ Inconclusive |
| NYISO | 223 | 0.428 | [0.264, 0.612] | ⚠️ Inconclusive |
| CAISO | 166 | 0.405 | [0.168, 0.613] | ⚠️ Inconclusive |
| SPP | 139 | N/A | — | ⚠️ No test positives |

"Inconclusive" means the 95% CI spans 0.50. The data is consistent with random performance, not necessarily reliably poor performance. This reflects small test cohort sizes, not necessarily model failure.

### Calibration
The uncalibrated model substantially overpredicts completion probability due to (a) scale_pos_weight upweighting during training and (b) declining positive rates between the training cohort (21.7%) and test cohort (13.4%). Isotonic regression calibration on a held-out calibration split reduces the Brier score from 0.214 to 0.103. AUC is unchanged by calibration (it is a ranking metric). Calibrated probabilities should be used for any probability-based application. Viability scores of exactly 0.0 (821 projects) and 1.0 (8 projects) reflect isotonic regression behavior at the tails and should be interpreted as ">95% predicted to fail" and ">95% predicted to complete" respectively.

---

## Known Issues and Limitations

### Training data limitations

**Temporal scope:** The model was trained on 2010–2017 queue cohorts and tested on 2018–2019. Energy market conditions have changed substantially since 2017 (FERC Order 2023 and the data center load boom)  and the model may be systematically optimistic or pessimistic about some project types as a result. The model should be retrained annually as new outcome data matures.

**Post-2019 labels are immature:** Projects entering the queue 2020–2024 have not had sufficient time to resolve (complete or withdraw). Their current "active" status cannot be used as a label. These projects form the live scoring set. This produces a train-test base-rate shift (21.7% → 13.4%) that is real and documented, not a modeling error.

**LBNL queue ID non-uniqueness:** The same numeric queue ID appears in multiple ISO regions in the LBNL dataset. All project-level operations use a composite identifier (queue_id + region + state + queue_year). Aggregate statistics are unaffected.

**Developer name inconsistency:** Developer names in LBNL data are not standardized — the same company may appear under multiple name variants. "Masked" entries (LBNL-redacted developer names) are treated as missing. Approximately 76% of active queue projects have no identifiable developer track record and receive NaN for developer features.

### Feature-level limitations

**cluster_size:** The cluster_size feature was initially computed incorrectly (NaN cluster IDs were grouped into a single mega-cluster of 24,379). After correction (unclustered projects receive cluster_size=1), the feature became less dominant but more interpretable. Post-fix XGBoost AUC improved from 0.729 to 0.748, suggesting the corrected version is more informative.

**IA_status_clean excluded:** The interconnection agreement study phase field was excluded due to leakage: 87% of withdrawn projects and 77% of operational projects have their IA_status_clean field overwritten with the outcome value ("Withdrawn" or "Operational"), making it a near-direct mirror of the label for most training rows.

**proposed_dev_years excluded:** Developer-stated timeline (proposed in-service date minus queue date) was excluded after preprocessing revealed that date columns were cast to strings during an earlier save step, producing all-zero values. Empirical evidence from LBNL also suggests developer-stated timelines are weak predictors of actual completion.

**Surplus project type:** Active queue projects with project_type = "Surplus" were not present in the training data (the training set contained only operational and withdrawn projects, which were overwhelmingly Generation type). At inference, Surplus projects are routed through the model's missing-value path rather than remapped to Generation. Predictions for Surplus projects are flagged as lower confidence.

**Industrial electricity price in deregulated states (resolved):** Initial feature construction from EIA Form 861 utility-level data underestimated prices in retail-deregulated states (Ohio, Pennsylvania, Illinois) by ~50% because Form 861 only captures the regulated wires-and-poles portion. Replaced with EIA's published Table 4 values, which correctly aggregate regulated and competitive supplier reporting.

**No transmission proximity feature:** Distance to nearest high-voltage substation was planned but excluded after the HIFLD Open Data hub was decommissioned and replacement data sources produced only a legacy 1999 dataset with no voltage attributes. Including this feature would likely improve regional predictions, particularly in the West and SPP.

### ERCOT-specific warning

ERCOT predictions are flagged as unreliable and should not be used for high-stakes decisions. The LBNL training data contains approximately 150 ERCOT rows concentrated in 2015–2017, with a completion rate of 49%, which is substantially above the national average and likely reflecting selection bias in LBNL's ERCOT coverage rather than genuine ERCOT market characteristics. Additionally, 25 ERCOT rows had placeholder queue dates of 1970 (Unix epoch artifacts) and were excluded from training. ERCOT's bootstrap confidence interval on test AUC spans [0.42, 0.58], consistent with random performance.

### SPP-specific note

SPP's 2018–2019 test cohort contained zero operational projects, making AUC undefined. SPP operational projects from this era entered the queue in earlier years (2010–2016) and fall in the training set. This is a consequence of the temporal split, not a data error.

---

## Maintenance

**Recommended update cadence:** Annual, as new LBNL Queued Up editions are released. Each new edition adds one year of matured outcome data, extending the trainable cohort.

**Monitoring:** Track calibration stability by comparing predicted viability distributions for new scoring cohorts against historical distributions. A significant shift (e.g., mean viability dropping from 0.15 to 0.05) may indicate distributional shift or a labeling change in the LBNL dataset.

**Known next improvements:**
1. Add transmission proximity features (distance to nearest 230+ kV substation) when HIFLD data is restored or replaced
2. Validate model in ERCOT, MISO, CAISO, NYISO, SPP with additional test data
3. Standardize developer name matching across LBNL editions using fuzzy matching or FERC entity IDs
4. Add year-stratified cross-validation to produce per-year reliability estimates

---

## Citation

Primary data source:
Rand et al. (2025). *Queued Up: 2025 Edition, Characteristics of Power Plants Seeking Transmission Interconnection As of the End of 2024.* Lawrence Berkeley National Laboratory. https://emp.lbl.gov/queues

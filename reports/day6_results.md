# Day 6 Results

## SHAP analysis
Top 5 features by mean |SHAP|:
             feature  mean_abs_shap
              region       0.110718
       fuel_bucketed       0.106774
              log_mw       0.094191
             service       0.078825
q_year_region_volume       0.049300

Compared to gain-based importance, SHAP confirms the top features.

See `figures/shap_summary.png` for full visualization and
`figures/shap_waterfall_*.png` for individual case studies.

## Calibration
Uncalibrated Brier score: 0.2137
Calibrated Brier score (isotonic): 0.1026

The uncalibrated model overpredicts completion probability due to (a) `scale_pos_weight=3.6`
during training, and (b) declining base rate between train (21.7%) and test (13.4%) cohorts.
Isotonic regression on a held-out calibration split corrects this; AUC unchanged.

## Per-region performance
               n  positive_rate    auc  mean_pred
region                                           
PJM        824.0          0.167  0.679      0.492
West       604.0          0.116  0.780      0.424
Southeast  497.0          0.123  0.878      0.414
MISO       344.0          0.093  0.541      0.470
NYISO      223.0          0.063  0.428      0.422
ERCOT      219.0          0.283  0.499      0.591
CAISO      166.0          0.048  0.405      0.436
ISO-NE     162.0          0.259  0.769      0.533
SPP        139.0          0.000    NaN      0.440

## Per-fuel performance
                     n  positive_rate    auc
fuel_bucketed                               
Solar           1728.0          0.124  0.734
Battery          443.0          0.072  0.489
Solar+Battery    356.0          0.056  0.509
Wind             293.0          0.133  0.756
Gas              163.0          0.325  0.734
Other_combined    95.0          0.158  0.860
Other             59.0          0.458  0.551
Hydro             37.0          0.649  0.827
Coal               4.0          0.750  0.167

## Headline regulator-facing metric
The top 10% of test projects (ranked by model) capture 31% of
all operational outcomes — a 3.1x lift over random ranking.

## Decisions for Day 7
- Apply calibrated model to active queue (~10,400 projects)
- Aggregate scores by region and fuel for headline finding
- Build case study writeups for 3-5 highest-scored active projects

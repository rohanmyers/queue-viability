# Day 5 Results

## Headline numbers
- **Naive baseline (predict mean rate):** AUC 0.500
- **Region-only baseline:** AUC 0.6096
- **Fuel-only baseline:** AUC 0.5762
- **Logistic regression:** AUC 0.7183, AP 0.3176
- **XGBoost (untuned):** AUC 0.7476, AP 0.3373

XGBoost beats the strongest naive baseline by 0.138 AUC.
XGBoost beats logistic regression by 0.029 AUC.

## Train vs. test gap
- LR:      train 0.7029, test 0.7183 (gap -0.015)
- XGBoost: train 0.7733, test 0.7476 (gap +0.026)

## Top features (by XGBoost gain)
                  feature  importance
            fuel_bucketed    0.145269
                   region    0.115171
                   log_mw    0.109558
                  service    0.101510
                   q_year    0.080285
           capacity_class    0.078201
          has_dev_history    0.076616
              dev_prior_n    0.076419
     q_year_region_volume    0.072275
dev_prior_completion_rate    0.068075

## Decisions for Day 6
- Calibration: model uses scale_pos_weight, so predicted probabilities are biased.
  Apply isotonic calibration on test set or refit with scale_pos_weight=1 + class_weight in cv.
- SHAP analysis to validate feature importances and check for surprises
- Per-region performance breakdown (especially flag ERCOT low-confidence)
- Hyperparameter tuning if needed


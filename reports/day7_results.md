# Day 7 Results — Live Queue Scoring

## Coverage
Scored 10,430 active and suspended generation interconnection requests.
- Validated regions: 4,948 projects
- Unvalidated regions: 5,482 projects

## Headline finding
**Of 2,061 GW of generation in the active U.S. interconnection queue,
the model predicts ~261 GW will reach commercial operation —
an expected attrition of 87%.**

## By region
           n_projects  mean_viability    total_mw  expected_mw  expected_completion_rate
region                                                                                  
CAISO             633           0.074  272530.388    19154.973                  0.070286
ERCOT            1706           0.384  358716.800   129354.166                  0.360602
ISO-NE            306           0.262   34503.347     4332.137                  0.125557
MISO             2129           0.112  389187.610    38557.925                  0.099073
NYISO             373           0.071   69391.780     4371.728                  0.063001
PJM              2069           0.152  198938.574    23121.246                  0.116223
SPP               641           0.081  127519.352     8649.659                  0.067830
Southeast         856           0.164  132895.542    12585.504                  0.094702
West             1717           0.077  477560.540    20924.867                  0.043816

## By fuel
                n_projects  mean_viability    total_mw  expected_mw  expected_completion_rate
fuel_bucketed                                                                                
Battery               2805           0.179  493751.002    84260.226                  0.170653
Coal                    12           0.318     686.900       97.617                  0.142112
Gas                    381           0.231  119173.870    13882.415                  0.116489
Hydro                   34           0.293    4061.665      429.693                  0.105792
Other                   78           0.136    8828.464      962.538                  0.109027
Other_combined         353           0.132  169011.270    10926.648                  0.064650
Solar                 4187           0.159  574261.840    90650.706                  0.157856
Solar+Battery         1680           0.148  497682.813    36644.314                  0.073630
Wind                   900           0.126  193786.110    23198.047                  0.119710

## Case studies saved to figures/
- Highest-viability project: PIS096
  (West,
  Battery,
  200 MW, viability 0.44)
- Lowest-viability project: Q376
  (West,
  Solar+Battery,
  400 MW, viability 0.00)

## Deliverables
- `data/processed/active_queue_scored.csv` — 10,430 scored projects
- `data/processed/active_queue_scored.parquet` — same, in parquet
- `figures/queue_forecast_by_region.png` — announced vs. expected capacity
- `figures/case_study_*.png` — SHAP waterfall plots for case studies

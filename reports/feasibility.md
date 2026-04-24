# Feasibility Assessment

## Dataset
- Source: LBNL Queued Up 2025 Edition, sheet `03. Complete Queue Data`
- Total rows: 36,441
- Generation + Surplus rows: 33,566

## Label availability (generation only)
- Operational (Y=1): 3,747
- Withdrawn (Y=0): 18,840
- Suspended: 546
- Active (to score): 10,430
- Raw trainable rows: 22,587

## Temporal label reliability
Operational rates by queue-entry year show steep drop after 2019:
- 2010-2019: 13-30% operational rates (mature outcomes)
- 2020: 7.6%
- 2021: 2.5%
- 2022: 1.1%
- 2024: 0.9%

Projects entering the queue after 2019 haven't had time to either complete or
formally withdraw, so their "withdrawn" labels are unreliable — a project still
in study phase is not the same as a project that failed.

## Training cutoff decision
**Use q_year <= 2019 for training.** Post-2019 requests held out for scoring only.

- Clean training set (2010-2019): 15,662 rows
  - Operational (Y=1): 3,265
  - Withdrawn (Y=0): 12,397
  - Class balance: 20.8% positive

## Regional coverage

- PJM: 8099
- West: 7647
- Southeast: 3893
- MISO: 3282
- ERCOT: 2582
- CAISO: 2534
- SPP: 2432
- NYISO: 1816
- ISO-NE: 1281

## Fuel type coverage (top 10 after Unicode cleanup)

- Solar: 13270
- Battery: 5354
- Wind: 4926
- Solar+Battery: 3569
- Gas: 2945
- Other: 1448
- Hydro: 456
- Coal: 303
- Offshore Wind: 230
- Geothermal: 158

## Baseline completion rate to beat
- Overall (full trainable set): 16.6%
- 2010-2019 cohort only: 20.8%

## Go/no-go decision
[x] Proceed with generation queue viability
- Scope: predict whether a queued generation request will reach commercial operation
- Training set: 2010-2019 queue entries only
- Scoring set: 2020+ requests (including all ~10,430 active projects)
- Features planned: capacity, fuel type, region, utility, developer track record,
  queue vintage, IA_status_clean (ordinal study phase), geographic features
  (substation distance from HIFLD), retail electricity price (EIA 861)
- Framing: motivated by FERC Order 2023 interconnection backlog and related
  data center / phantom-load concerns

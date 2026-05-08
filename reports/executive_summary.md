# Executive Summary: Generation Interconnection Queue Viability Model

**Rohan Myers | May 2026 | github.com/rohanmyers/queue-viability**

---

## The Problem in One Sentence

The U.S. electrical grid's interconnection queue contains 2,300 gigawatts of proposed power plants, but historically, only 13% of projects ever get built, and regulators have no systematic way to tell which ones are real.

## Why It Matters Now

Two forces are colliding on the U.S. grid. On the supply side, developers are filing interconnection requests at unprecedented rates. AI data centers are projected to nearly triple their electricity consumption by 2030, requiring massive new generation to serve them. Grid planners, utility regulators, and FERC analysts are caught in the middle: they must decide today how much generation and transmission infrastructure to build, but the queue data they rely on is dominated by projects that will never materialize.

When utilities overbuild infrastructure for phantom queue projects, residential and commercial ratepayers absorb those costs through higher electricity bills. FERC's Order 2023 introduced deposit requirements and withdrawal penalties to address this problem, but the reforms are still new and their effects are not yet measurable. No publicly available tool exists to quantify how much of any queue is real.

## What Was Built

A machine learning model that assigns every active generation interconnection request a **viability score** or a calibrated probability from 0 to 1 representing the likelihood that the project will reach commercial operation. The model was trained on 22,587 historical queue outcomes (2010–2017), evaluated on a held-out 2018–2019 test cohort, and then applied to score all 10,430 currently active and suspended projects in the national queue.

The model uses 13 features available for every queued project: fuel type (Solar, Wind, Battery, Gas, etc.), ISO/RTO region, proposed capacity in MW, service type (ERIS/NRIS), queue year, queue congestion at time of entry, developer track record (historical completion rate and number of prior projects), and state-level industrial electricity price. No proprietary or non-public data is required.

## Key Numbers

**Model performance:** Test AUC of 0.748. The model correctly ranks 75% of (operational, withdrawn) project pairs in the right order on a held-out test set. The top 10% of projects ranked by the model capture 31% of all eventual completions, a 3.1x improvement over random selection. Predicted probabilities are well-calibrated after isotonic correction (Brier score 0.103).

**Headline forecast:** Of 1,703 GW of active generation queue capacity outside Texas, the model predicts **92% attrition** or approximately 132 GW is expected to reach commercial operation. Including ERCOT (where training data is limited), the estimate is 87% attrition across 2,061 GW.

**Developer track record:** Projects from developers with established completion histories complete at roughly twice the rate of projects from unknown developers (34% vs. 17%). The 76% of active queue projects from developers with no identifiable track record are significantly more likely to be phantom.

**Zombie projects:** Projects entering the queue before 2010 and still technically "active" today receive near-zero viability scores. Five such projects alone, including a 418 MW facility in Florida idle since 2006 and three Oregon wind projects idle since 2010, represent over 1,500 MW of phantom capacity still counted in published queue totals.

## Who Should Use This

**State public utility commission staff** reviewing utility integrated resource plans can use the model to independently challenge load forecasts built around speculative queue projects. **ISO/RTO planning teams** can use viability scores to tier their queue and prioritize study resources toward projects most likely to complete. **FERC analysts** can use the regional performance breakdown to identify where Order 2023's reforms are having the most impact. **Ratepayer advocates and journalists** can use the scored CSV to quantify how much announced generation in their region is likely phantom.

## Regional Reliability

The model's performance is validated with statistical confidence in four regions: Southeast (AUC 0.88), West (AUC 0.78), ISO-NE (AUC 0.77), and PJM (AUC 0.68). In MISO, ERCOT, NYISO, CAISO, and SPP, the test cohort is too small to confirm performance; predictions in those regions are flagged as unvalidated and should be treated as exploratory. Importantly, several of the unvalidated regions: ERCOT (Texas), CAISO (California), and MISO (Ohio, Indiana, Illinois) are among the fastest-growing data center markets, making model validation there a high-priority future improvement.

## Limitations and Honest Caveats

This model reflects historical queue dynamics from 2010–2019. The energy landscape has changed substantially (FERC Order 2023, the IRA, the data center boom) and the model may be systematically over- or under-optimistic about certain project types as a result. It should be updated annually as new outcome data becomes available. ERCOT predictions are particularly unreliable and are clearly flagged in all outputs. The model does not incorporate non-public information (financing status, offtake agreements, permitting progress) that a sophisticated developer or ISO analyst would also consider.

## How to Access

All code, data pipelines, trained model, and the full scored queue output are available at:

**github.com/rohanmyers/queue-viability**

The primary deliverable — `data/processed/active_queue_scored.csv` — contains calibrated viability scores for all 10,430 active projects and can be opened in Excel, filtered by region or fuel type, and sorted by predicted viability without any programming knowledge.

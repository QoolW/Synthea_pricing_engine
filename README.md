# Synthea Healthcare Claims Risk Model
An actuarial pricing pipeline for group insurance: a claims-cost model
built on synthetic patient data, calibrated against real, published Malaysian occupational
risk statistics.

---

## Objectives

- **Claims model**: Model claims cost via GLM/XGBoost on synthetic patient data.
- **Data cleaning**: Clean and window claims data via SQLite pipeline.
- **Risk calibration**: Calibrate industry risk using real DOSM injury statistics.
- **Deliverables**: Deliver quote schedule format and dashboard via Excel PivotCharts.

## Data Sources

- **[Synthea](https://synthea.mitre.org/)**: An open-source synthetic patient generator.
  Every patient, encounter, and claim cost in this project is synthetic. No real patient or
  employer data is used anywhere.
- **DOSM, *National Occupational Injury and Disease Statistics***: a real, published
  Malaysian government report. Industry risk multipliers are calibrated directly against its
  injury-rate-per-1,000-workers figures
  (refer to Table 2.b - Jumlah Column in `/data/dosm_industry_risk.xlsx`).

## Methodology
1. **Clean and window the data:** Filter to a working-age active census, then aggregate
   claims within a single policy year.
2. **Derive smoking status:** From coded clinical observations (LOINC `72166-2`).
3. **Fit a claims-cost model:** A Poisson GLM on age, gender, and smoker status, benchmarked
   against a Tweedie GLM and XGBoost.
4. **Calibrate industry risk:** As a continuous ratio of each sector's real DOSM injury rate to
   the national average (`risk_multiplier = dosm_injury_rate_per_1k x national_avg_injury_rate`).
5. **Validate the model choice statistically:** GLM vs. XGBoost is decided by a paired
   significance test (Wilcoxon signed-rank + paired t-test) on held-out data.
6. **Roll up to employer groups:** Rescale to
   a Malaysian Ringgit benchmark, and export to two Excel deliverables: a formatted
   quote schedule and a PivotTable/PivotChart dashboard.

## Outcomes
- **Smoking status:** Before the fix, `is_smoker` came out negative. On the corrected 1,919-row panel: `is_smoker = 0.0713, p < 0.001` (positive and significant).
- **Gender (female):** `is_female = 1.0573` is a large, significant effect (exp(1.057) ≈ 2.9x). Plausibly real reproductive/maternity care utilization in Synthea's clinical modules (worth investigating).
- **GLM Poisson vs. Tweedie (Nearly identical):** RMSE `$34,343.77` vs `$34,389.20` -- MAE `$15,324.60` vs `$15,334.1` 
- **In-sample, XGBoost looked better:** MAE `$14,631.82` vs. the GLM's `$15,324.60`. On the actual held-out test split: GLM's MAE was `$15,725.06` -- XGBoost's was `$15,789.90`, worse.
- **Wilcoxon Signed-Rank Test (`p = 0.008`):** Proves XGBoost is statistically significantly worse than GLM on median/rank-based error.
- **Paired t-test (`p = 0.89`):** Indicates no statistically significant mean difference under normal distribution assumptions.
- **Final model (Poisson GLM):** Because XGBoost failed to demonstrate a superior error reduction, the automated decision logic correctly rejected XGBoost and retained the simpler, more interpretable Poisson GLM baseline.

## Limitations 
- **The quote schedule is currently built for a single company (COMP001), not the full
  book.** It works solely as a proof of concept and a formatting template.
- **The dashboard is Excel PivotTable/PivotChart-based, not Power BI.** While Power BI is not
  under the scope of this project, dashboard could use some improvement.
- **The extended feature set (income, marital status, etc) can be tested and potentially promoted** 
  into the live pipeline. Not a main consideration in this project due to the problem of overfitting and etc.
- **Sample size** of ~1,900 working-age members after filtering is workable for the modeling
  shown here, but thin for the extended feature set and for any future subgroup analysis
  (e.g. by industry).
- **This is more of a data science approach to an actuarial problem, not full ratemaking practice.**
  As someone from a Statistics background, some real actuarial intricacies are under-addressed in this project.
---

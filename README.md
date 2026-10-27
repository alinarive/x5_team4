# X5 team 4: EDA and weekly turnover forecasting

This repository contains the team's exploratory analysis and a reproducible
comparison of forecasting methods for the X5 university case.

The task is to forecast weekly retail turnover for every requested store–week
from **11 May to 6 July 2026**. The supplied `target` measures turnover in
**conventional units**, not a percentage or an amount in rubles. Values above
one are possible; forecasts are not capped at one.

## Start here

- [EDA_x5.ipynb](EDA_x5.ipynb): the team's exploratory analysis.
- [simple_modeling_v2.ipynb](simple_modeling_v2.ipynb): the earlier forecasting
  notebook. It compares `Latest`, `Mean4`, `Ridge` and `Boosted trees` on the
  original feature set and freezes `Latest` as the chosen method. The saved
  tables and plots can be read without running it. It is kept as a reference
  for the initial benchmark and the donor-median reserve rule.
- [simple_modeling_v3.ipynb](simple_modeling_v3.ipynb): the current forecasting
  notebook. It builds on v2, replaces long-memory lags with short-memory lags,
  adds individual and global seasonality, and compares the revised
  `CatBoost`-based methods. The saved tables and plots can be read without
  running it.
- [v2 methodology and results](results/modeling_v2/report_sections.md):
  English text for the v2 benchmark (Latest, Mean4, Ridge, Boosted trees).
- [v3 methodology and results](results/modeling_v3/report_sections.md):
  English text for the current pipeline (CatBoost, CatBoost_two_stage,
  CatBoost_hybrid, seasonality).
- [v2 validation metrics](results/modeling_v2/validation_metrics.csv) and
  [v2 group metrics](results/modeling_v2/group_metrics.csv): the comparison
  tables behind the v2 conclusions.
- [v3 validation metrics](results/modeling_v3/validation_metrics.csv) and
  [v3 group metrics](results/modeling_v3/group_metrics.csv): the comparison
  tables behind the v3 conclusions.
- [v2 figures](results/modeling_v2/figures/) and
  [v3 figures](results/modeling_v3/figures/): PNG charts for the presentation.

## Data and local folder layout

The original data are distributed separately with the case materials and are not
included in this repository. To rerun the forecasting notebook, place the data
folder next to your cloned repository:

```text
work_folder/
├── MSC Case data 2026/
│   ├── train.parquet
│   └── test_template.parquet
└── x5_team4/
    ├── EDA_x5.ipynb
    ├── simple_modeling_v2.ipynb
    ├── simple_modeling_v3.ipynb
    ├── requirements_modeling_v2.txt
    ├── requirements_modeling_v3.txt
    ├── README.md
    └── results/
        ├── modeling_v2/
        └── modeling_v3/

The notebook reads `DATA = Path('../MSC Case data 2026')`. If your data are stored
elsewhere, change only this line in the first code cell. Keep the original files
unchanged. The forecasting notebook does not require `sample.csv`, the briefing
PDFs or the Q&A workbook to execute. The EDA retains its own original data paths.

## Run the forecasting notebook

The saved analysis was run with **Python 3.12**. Package versions are recorded in
[requirements_modeling_v2.txt](requirements_modeling_v2.txt).

1. Open a terminal in the repository folder and install the dependencies into
   the Python environment you intend to use:

   ```text
   python -m pip install -r requirements_modeling_v2.txt
   ```

2. Open `simple_modeling_v2.ipynb` in VS Code, Jupyter or another notebook editor.
   Select a kernel using that same Python environment.
3. Set the notebook's working directory to the repository folder containing
   `simple_modeling_v2.ipynb`. Run all cells from top to bottom.

Calculations, model definitions and plots are contained in the notebook; no
external modelling scripts are needed. The notebook writes to
`results/modeling_v2/` and overwrites its generated outputs on a rerun.
Individual forecast CSVs are local outputs and are excluded from Git. The final
file is `results/modeling_v2/final_forecast.csv`, with 31,586 rows in template
order and columns `store_pseudo_id`, `week` and `target`. The saved notebook
shows summary checks rather than a preview of these future predictions.
Data checks use printed messages: inspect them for missing targets, duplicate
store–weeks, unexpected row counts or non-finite predictions.

Calculations, model definitions and plots are contained in each notebook; no
external modelling scripts are needed. simple_modeling_v2.ipynb writes to
`results/modeling_v2/` and simple_modeling_v3.ipynb writes to
`results/modeling_v3/`. Each notebook overwrites its own generated outputs on a
rerun. Individual forecast CSVs are local outputs and are excluded from Git. The
final file of each run is `results/modeling_v2/final_forecast.csv` or
`results/modeling_v3/final_forecast.csv`, with 31,586 rows in template order and
columns store_pseudo_id, week and `target`. The saved notebooks show summary
checks rather than a preview of these future predictions. Data checks use printed
messages: inspect them for missing targets, duplicate store–weeks, unexpected row
counts or non-finite predictions.

The report text is written Markdown rather than generated interpretation. If you
change the data, dates, features or model settings in a notebook, update both
that notebook's written interpretation and the corresponding
report_sections.md after checking the new results.

## Data and features (v3)

The supplied training data contain 321,531 observations, 3,561 stores and
148 weeks, from 10 July 2023 to 4 May 2026. Every row represents one store in
one week. The final template includes 168 stores without training history,
accounting for 702 requested rows. This data description is shared by v2 and v3.

The v3 feature set differs from v2 in three ways.

Short-memory lags. Long-memory lags (lag8, roll_median_8, n_obs_total)
were removed: they pull the model towards the volatile 2024–2025 regime and are
not informative for a nine-week horizon in the current regime. The retained
history features are last_value, lag1, lag2, lag4, mean_4w,
roll_mean_2, roll_mean_4, roll_std_4, n_recent, has_history,
gap_weeks.

### Seasonality. Two signals are added:

seasonal_lag_52 — the store's own turnover in the same calendar week 52
weeks before the forecast week. Available for Established stores with a
full year of history; NaN for Limited and New.
global_seasonal_factor — network-level seasonality, computed as the network
mean turnover in week w − 52 divided by the recent 52-week network mean.
Available for every store, including new ones.
global_seasonal_level = last_value × global_seasonal_factor — a store-level
combination of the current level and the network seasonal factor.
has_seasonal_lag — a binary flag that lets tree-based models decide when to
rely on the store's own seasonal signal and when to fall back to the global
one.
All seasonal features are computed from data up to origin only. No future
turnover enters the seasonal computation.

Profile and calendar features. region, log_age, log_area,
forecast_woy, forecast_quarter, forecast_month, and the event-type
categories feature24_cat, feature26_cat, feature28_cat with missing encoded
as 'missing'. horizon is kept for diagnostics but excluded from the model
features.


## Methods and current results

### v2 benchmark (original feature set)

Four methods are compared:

- **Latest:** repeat the last available turnover.
- **Mean4:** repeat the mean of observations in the latest four calendar weeks.
- **Ridge:** a pooled linear regression with regularisation.
- **Boosted trees:** histogram gradient boosting using the same predictor set.

Latest and Mean4 share a reserve rule for stores without history: use past donor
medians by region and store-age band, with broader groups when needed. Details
are explained in the notebook.

The two selection origins are **3 March and 5 May 2025**. Each predicts the next
nine weeks using only turnover available at its origin. Model choice uses mean
MAE with equal weight for the two periods; lower is better.

| Method | Mean selection MAE |
| --- | ---: |
| Latest | **0.017624** |
| Mean4 | 0.018419 |
| Boosted trees | 0.019158 |
| Ridge | 0.019471 |

The selected method is **Latest**, including the donor-median reserve rule.
After freezing the choice, a separate retrospective control forecasts
**9 March–4 May 2026** from **2 March 2026**. Latest's control MAE is **0.016064**,
RMSE is **0.023941**, and Bias (prediction minus actual) is **−0.004451**.
These errors use the target's conventional units; they are not percentage errors.

### Limits and discussion points

- Two selection periods provide limited evidence of performance across seasons.
  Mean4 wins the first period and Latest wins the second.
- The control period was explored in earlier work. It is a retrospective check,
  not a blind test, and does not determine the selected method.
- For stores without history, control MAE is **0.046959**, about three times the
  established-store MAE of **0.015390**.
- Region, area, opening information and the requested future store lists are
  assumed available at forecast time.
- Fixed Ridge and boosting settings do not beat Latest in this comparison. This
  does not establish that all implementations of these methods would lose.
- No prediction intervals or quantified monetary benefits are estimated. These
  are point forecasts for a university case, not a guarantee of future accuracy.

Useful next steps are more historical evaluation periods, a seasonal reference
forecast and better treatment of stores without history.

## v3 pipeline (revised feature set)

Seven methods are compared on the revised feature set in
simple_modeling_v3.ipynb:

- **Latest**: repeat the last available turnover.
- **Mean4**: repeat the mean of observations in the latest four calendar weeks.
- **SeasonalNaive**: repeat the turnover of the same store in the same week one
year earlier; falls back to Latest when no such observation exists.
- **Ridge**: a pooled linear regression with regularisation, median imputation,
standard scaling for numeric inputs and one-hot encoding for categories.
- **CatBoost**: a single boosted-tree model on all stores, with native
categorical handling and MAE loss.
- **CatBoost_two_stage**: two separate CatBoost models — model_hist on stores
with has_history == 1 using the full feature set, and model_new on stores
with has_history == 0 using only profile and calendar features plus the
global seasonality block (no short lags, no rolling statistics, no
seasonal_lag_52).
- **CatBoost_hybrid**: a deterministic rule that uses CatBoost for stores with
history and Latest (donor median) for stores without history.
Latest, Mean4 and the New branch of the hybrid share the same donor-median
reserve rule as in v2. The selection origins are 3 March and 5 May 2025; the
control origin is 2 March 2026. Model choice uses mean MAE with equal weight for
the two selection periods.

### Method	v3 mean selection MAE
CatBoost_hybrid	0.019389
CatBoost_two_stage	0.019389
Latest	0.020126
CatBoost	0.020279
Mean4	0.022243
Ridge	0.022520
SeasonalNaive	0.056654
The selected method is CatBoost_hybrid. On the revised feature set it is
the only method that beats Latest on the selection periods: 0.019389 vs
0.020126, a 3.66% relative improvement. This reverses the v2 result,
where no method beat Latest on selection.

On the retrospective control period (9 March–4 May 2026, forecast from 2 March
2026), CatBoost_hybrid achieves MAE = 0.015696, RMSE = 0.023741, and
Bias = −0.006652 over all 30,375 control store–weeks.

Compared with the v2 baseline Latest (control MAE 0.016064, RMSE 0.023941):

MAE: −0.000368, a 2.29% relative reduction;
RMSE: −0.000200, a 0.84% relative reduction;
Bias: −0.006652 vs −0.004451, an increase of 0.002201 in absolute bias.
Compared with the earlier v3 candidate CatBoost_two_stage (control MAE
0.018415, RMSE 0.026271):

MAE: −0.002719, a 14.76% relative reduction;
RMSE: −0.002530, a 9.63% relative reduction;
Bias: −0.006652 vs −0.013626, a reduction of 0.006974 in absolute bias.
The per-group control results are:

Established: CatBoost = 0.015281, Latest = 0.015390,
Ridge = 0.014260, CatBoost_two_stage = 0.018217.
Limited history: CatBoost = 0.013982, Latest = 0.015798,
Ridge = 0.013717, CatBoost_two_stage = 0.015881.
New / no history: Latest = 0.046959, CatBoost = 0.052573,
CatBoost_two_stage = 0.052363, Ridge = 0.048605. The hybrid reproduces
Latest here.

The control results show a clear regime dependence. On Established, the pooled
CatBoost and Ridge have lower MAE than CatBoost_two_stage, while Ridge
is also the best method on Limited history. CatBoost_two_stage is
competitive on the volatile 2024–2025 selection periods but loses on the stable
2026 control. This is the reason the hybrid uses CatBoost rather than
CatBoost_two_stage for stores with history: the control period is closer to
the test horizon than the selection periods, and the pooled model is the more
stable of the two CatBoost variants across regimes.

These errors use the target's conventional units; they are not percentage
errors. MAE and RMSE are not 95% prediction intervals. Full tables are in
results/modeling_v3/.

## Limits and discussion points

- Two selection periods and one control period provide limited evidence of
performance across seasons. In v3, Mean4 wins the first selection period,
Latest wins the second, and CatBoost_two_stage is competitive on both.
CatBoost_hybrid combines the two regimes by using the pooled CatBoost for
stores with history and the Latest donor median for new stores.
- The control period was explored in earlier work. It is a retrospective check,
not a blind test, and does not determine the selected method.
- For stores without history, the donor-median fallback produces a control MAE
of about 0.046959, several times the established-store MAE. CatBoost and
CatBoost_two_stage are worse than the donor median on this segment, which is
why the v3 hybrid falls back to Latest for new stores.
- Region, area, opening information and the requested future store lists are
assumed available at forecast time.
- The hybrid improves over both Latest and CatBoost_two_stage on the control
period. The improvement over Latest is modest (2.29% MAE, 0.84% RMSE) and is
obtained mainly on established and limited stores. The new-store segment is
unchanged relative to Latest, so the overall gain is bounded by the
new-store share of the template.
- Ridge has the lowest control MAE on established and limited stores
(0.014260 and 0.013717), but it is also the least stable across selection
periods (0.022520 and 0.025226). The hybrid uses CatBoost because it is the
more stable CatBoost variant across regimes, not because it is the best method
on the control period in absolute terms.
- The hybrid under-predicts by 0.006652 on average (Bias), compared with
0.004451 for Latest. The increase of 0.0022 in absolute bias is small
relative to the MAE gain, but a bias correction fitted on the selection
periods is a reasonable next step.
- No prediction intervals or quantified monetary benefits are estimated. These
are point forecasts for a university case, not a guarantee of future accuracy.

## Shared and local output files

The repository includes concise metrics, report text, figures,
`run_summary.json` and `frozen_selection.json`. The JSON files record the saved
run and the method choice made before control evaluation.

The notebook generates the following files only for local use:
`final_forecast.csv`, `baseline_forecast.csv`, `validation_predictions.csv`
(about 55 MB) and `final_forecast_audit.csv`. These
are excluded by `.gitignore`, along with Python environments, caches and earlier
local prototype notebooks. Historical examples remain in the notebook to explain
and assess the methods. Rerun the notebook to obtain the final prediction CSV.

## Sources

The supplied X5 case data, `MSC_Briefing.pdf`, the project assignment criteria,
the X5 Q&A workbook and the team's `EDA_x5.ipynb` provide the project context.
The case documents and source data are shared separately from this repository.

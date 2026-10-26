# X5 team 4: EDA and weekly turnover forecasting

This repository contains the team's exploratory analysis and a reproducible
comparison of forecasting methods for the X5 university case.

The task is to forecast weekly retail turnover for every requested store–week
from **11 May to 6 July 2026**. The supplied `target` measures turnover in
**conventional units**, not a percentage or an amount in rubles. Values above
one are possible; forecasts are not capped at one.

## Start here

- [EDA_x5.ipynb](EDA_x5.ipynb): the team's exploratory analysis.
- [simple_modeling_v2.ipynb](simple_modeling_v2.ipynb): the main forecasting notebook.
  Its saved tables and plots can be read without running it. It builds on the EDA
  but does not need an active EDA session.
- [Methodology, model descriptions and results](results/modeling_v2/report_sections.md):
  English text for the project report or presentation.
- [Validation metrics](results/modeling_v2/validation_metrics.csv) and
  [metrics by history group](results/modeling_v2/group_metrics.csv): the comparison
  tables behind the conclusions.
- [Figures](results/modeling_v2/figures/): five PNG charts for the presentation.

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
    ├── requirements_modeling_v2.txt
    ├── README.md
    └── results/
        └── modeling_v2/
```

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

The report text is written Markdown rather than generated interpretation. If you
change the data, dates, features or model settings, update both the notebook's
written interpretation and `report_sections.md` after checking the new results.

## Methods and current results

The supplied training data contain **321,531 observations**, **3,561 stores** and
**148 weeks**, from 10 July 2023 to 4 May 2026. Every row represents one store in
one week. The final template includes 168 stores without training history,
accounting for 702 requested rows.

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

## Limits and discussion points

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

# Football Player Market Value Prediction

Estimating football player market values from on-field performance, then examining where observed valuations differ from model expectations. The project combines FBref statistics with Transfermarkt valuations and compares four regression approaches to support data-informed scouting and recruitment analysis.

**[Read the research report](football_player_market_value_report.pdf)** · **[Explore the modeling analysis](player_market_value_analysis.Rmd)**

## Results

The research report records the following results on a held-out test set:

| Model | RMSE on log values | RMSE (€ millions) | Reported R²* |
| --- | ---: | ---: | ---: |
| XGBoost | **0.6029** | **10.2381** | **0.6311** |
| Random Forest | 0.6207 | 11.3049 | 0.6152 |
| Elastic Net | 0.6527 | 11.1186 | 0.5671 |
| Linear Regression | 0.6535 | 11.1477 | 0.5661 |

*The analysis computes this metric as the squared correlation between predicted and observed `log1p` market values. It is distinct from the residual-based coefficient of determination on euro values.*

XGBoost achieved the lowest error in the reported comparison. Its predictions feed player-level valuation tables, actual-versus-predicted plots, and residual summaries by position and league.

## Approach

1. **Prepare the data.** Normalize player names, align FBref season-end years with Transfermarkt season-start years, and join the sources. The modeling analysis retains players with at least 450 minutes and positive recorded valuations.
2. **Engineer features.** Combine age, playing time, goals, assists, expected goals, progressive actions, discipline, per-90 statistics, position, and league. Model the target as `log1p(market value in € millions)`.
3. **Compare models.** Use an 80/20 split with seed 42. Train linear regression, elastic net with 10-fold cross-validation, a 500-tree random forest, and XGBoost with five-fold cross-validation and early stopping.
4. **Analyze valuation gaps.** Calculate observed value minus predicted value. The analysis labels gaps above +25% as “Overvalued,” below −25% as “Undervalued,” and the remainder as “Fairly Valued,” relative to the model. Percentage gaps use a €0.1M floor on predicted value.

These labels describe model residuals, not proven transfer-market mispricing. Transfermarkt values are estimates, and the model does not capture every factor affecting a player's value.

## Data and files

The included snapshots contain **30,505 FBref rows** and **40,520 Transfermarkt rows** before filtering and joining. FBref covers the five major European leagues with season-end years 2014–2024. Transfermarkt uses season-start years 2014–2024 and also includes some lower-division records; the modeling join determines the final sample.

| File | Purpose |
| --- | --- |
| [football_player_market_value_report.pdf](football_player_market_value_report.pdf) | Research narrative, model comparison, visualizations, and interpretation |
| [player_market_value_analysis.Rmd](player_market_value_analysis.Rmd) | Feature engineering, model training, evaluation, and residual analysis |
| [predicting_player_market_value.Rmd](predicting_player_market_value.Rmd) | Exploratory analysis and commented data-collection code |
| [all_leagues_stats.csv](all_leagues_stats.csv) | FBref performance snapshot |
| [market_values.csv](market_values.csv) | Transfermarkt valuation snapshot |

## Reproduce the analysis

Use **R and RStudio**, or R with Pandoc available. The CSV snapshots are included, so the documented workflow uses them directly.

```bash
git clone https://github.com/anikethk1/player_market_value_prediction.git
cd player_market_value_prediction
```

From an R session with the repository root as the working directory, install the dependencies:

```r
install.packages(c(
  "rmarkdown", "knitr", "tidyverse", "caret", "xgboost",
  "randomForest", "glmnet", "corrplot", "Metrics", "scales",
  "kableExtra", "gridExtra", "ggrepel", "reshape2"
), repos = "https://cloud.r-project.org")
```

Render the main analysis to HTML:

```r
rmarkdown::render(
  "player_market_value_analysis.Rmd",
  output_format = "html_document",
  knit_root_dir = getwd(),
  envir = new.env()
)
```

This trains the four models and writes:

- `player_market_value_analysis.html` — the rendered analysis
- `model_comparison.csv` — model evaluation metrics
- `player_valuations.csv` — held-out player valuations and residuals
- `efficiency_by_position.csv` and `efficiency_by_league.csv` — grouped residual summaries

To render the separate exploratory analysis:

```r
rmarkdown::render(
  "predicting_player_market_value.Rmd",
  output_format = "html_document",
  knit_root_dir = getwd(),
  envir = new.env()
)
```

The main document caches computations. Remove `player_market_value_analysis_cache/` before rerunning after changes to the input data or dependencies. Its final `sessionInfo()` section records the software environment; exact metrics can vary across package versions. Model serialization examples are in the final R Markdown appendix and are disabled by default.

## Project team and sources

Aniketh Kalagara, Palash, Jovan, and Kelly, as credited in the analysis.

Data sources: [FBref](https://fbref.com/en/) and [Transfermarkt](https://www.transfermarkt.com/). The exploratory document includes the original collection approach using [worldfootballR](https://github.com/JaseZiv/worldfootballR).

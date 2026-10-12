# December 2026 Revenue Forecast: Model Evaluation and Planning Recommendation

An end-to-end forecasting study that tests whether a $4.1M monthly revenue target for December 2026 is realistic. Nine methods (statistical, neural, and foundation-model) are evaluated on the same out-of-sample year, and the results are translated into a planning recommendation for a non-technical executive audience.

**Bottom line:** December 2026 revenue is most likely about **$3.98M** (80% interval $3.50M–$4.45M), and the probability of reaching $4.1M is about **37%**. The recommendation is to plan at $3.9M–$4.0M and treat $4.1M as a stretch goal.

## Context

The $4.1M target, an implied 18% year-over-year growth rate, came from an unvalidated generative-AI forecast run on a data export that was later found to overstate recent revenue. This study re-forecasts December 2026 from the corrected 60-month series (January 2021 to December 2025).

## Results

Accuracy is measured on 2025, using models trained only on data through December 2024. December 2026 forecasts come from models refit on all 60 months.

| Model | 2025 MAE | Dec 2026 forecast | 80% interval | P(≥ $4.1M) |
|---|---|---|---|---|
| **SARIMAX(0,1,0)(1,1,1,12)** | **$68K** | **$3.98M** | $3.50M–$4.45M | 37% |
| 6-month moving average | $140K | $3.55M | — | — |
| N-BEATS (quantile regression) | $154K | $3.90M | $3.66M–$4.12M | 13% |
| 12-month moving average | $184K | $3.44M | — | — |
| Chronos-Bolt (zero-shot) | $185K | $3.99M | $3.46M–$4.44M | 38% |
| Naive | $225K | $3.83M | — | — |
| ARIMA(1,1,0) | $225K | $3.83M | $3.16M–$4.50M | 31% |
| Linear trend | $271K | $3.98M | — | — |
| Darts ARIMA(2,1,1) | $360K | $4.19M | $3.65M–$4.74M | 58% |

The notebook also reports 95% intervals for every model that supports them.

## Highlights

- **Selection on out-of-sample error, not in-sample fit.** The three ARIMA candidates were within 6 AIC points of each other, while their 2025 errors differed by nearly 2x.
- **Convergence checks.** The reference SARIMAX(1,1,1)(1,1,1,12) failed to converge. The selected specification converged, with a lower AIC and equivalent error.
- **Residual diagnostics without the differencing burn-in.** Including the start-up residual masked autocorrelation entirely (Ljung-Box p = 1.00 for a random walk with drift). With it removed, the best ARIMA shows significant leftover autocorrelation (p = 0.0003), while the SARIMAX residuals are consistent with white noise (Ljung-Box p = 0.43, Jarque-Bera p = 0.37).
- **Like-for-like uncertainty.** Models are compared at 80%, the widest interval Chronos-Bolt can produce, alongside 95% where available.
- **Reproducible.** The environment is locked with uv and stochastic models are seeded, so repeated runs give identical results.

## Approach

1. **Baselines:** naive, 6- and 12-month moving averages, and a linear trend.
2. **Classical models:** an ARIMA specification search and seasonal SARIMAX, compared on convergence, AIC, and holdout error; residual diagnostics (Ljung-Box, Jarque-Bera); refit on the full series with 95% and 80% intervals and the probability of reaching the target.
3. **Modern models:** N-BEATS with quantile regression, zero-shot Chronos-Bolt (`amazon/chronos-bolt-small`), and an ARIMA(2,1,1) built in Darts as a like-for-like reference.
4. **Comparison and recommendation:** one table covering all nine methods, followed by a written recommendation for executives.

## Limitations

- **One holdout year.** Model selection rests on 2025 alone; a rolling-origin backtest would show how stable each model's accuracy is.
- **Short history and no external drivers.** Sixty monthly observations of revenue alone; pricing, pipeline, or macroeconomic regressors are a natural extension.
- **Approximate target probabilities.** For the classical models and Chronos-Bolt, they assume normally distributed forecast errors.

## Running the Analysis

Prerequisites: [uv](https://docs.astral.sh/uv/), which installs Python 3.12 if needed. The first run downloads the `amazon/chronos-bolt-small` weights from Hugging Face.

```bash
uv sync
uv run jupyter lab notebooks/revenue_forecast_evaluation.ipynb
```

To execute every cell from the command line and save the outputs:

```bash
uv run jupyter nbconvert --to notebook --execute --inplace notebooks/revenue_forecast_evaluation.ipynb
```

The full notebook runs on CPU in under a minute once the model weights are cached.

## Repository Structure

```
.
├── data/
│   ├── revenue.csv                            # monthly revenue, Jan 2021 – Dec 2025
│   └── LICENSE.txt                            # license for the case-study data
├── notebooks/
│   └── revenue_forecast_evaluation.ipynb      # full analysis and recommendation
├── pyproject.toml                             # dependencies
├── uv.lock                                    # exact, tested versions
└── .python-version                            # Python 3.12
```

## Tech Stack

| Package | Version | Role |
|---|---|---|
| statsmodels | 0.15.0 | ARIMA, SARIMAX, Ljung-Box test |
| darts[torch] | 0.47.0 | N-BEATS, Darts ARIMA, MAE/RMSE |
| chronos-forecasting | 2.3.2 | Chronos-Bolt zero-shot model |
| torch | 2.14.1 | Neural network backend |
| pandas, numpy, scipy | 3.0.6, 2.3.5, 1.18.1 | Data handling, statistics |
| matplotlib | 3.11.2 | Charts |

To upgrade to the newest compatible versions, run `uv lock --upgrade` and then `uv sync`, re-run the notebook, and update this table.

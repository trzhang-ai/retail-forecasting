# Retail Revenue Forecasting: Approach Evaluation and Recommendation

Your VP of Sales projected $4.1M in monthly revenue by December 2026 — 18% year-over-year growth. The data engineering team found bugs in the original data that inflated recent figures. They've handed you clean data. Your job: figure out what the numbers actually say.

## Project Structure

```
.
├── data/revenue.csv        # 60 months of monthly revenue (Jan 2021 – Dec 2025)
├── starter/starter.ipynb   # the analysis notebook (all four phases)
├── pyproject.toml          # project dependencies
├── uv.lock                 # exact, tested package versions
└── .python-version         # Python 3.12
```

## Getting Started

### Prerequisites

- [uv](https://docs.astral.sh/uv/). It installs Python 3.12 automatically if you don't have it.
- Internet access on the first run: the notebook downloads the `amazon/chronos-bolt-small` model from Hugging Face.

### Installation

From the project root:

```bash
uv sync
```

This creates `.venv` with the exact versions in `uv.lock`, including Jupyter.

### Running the Notebook

Open it in Jupyter Lab:

```bash
uv run jupyter lab starter/starter.ipynb
```

In VS Code, open `starter/starter.ipynb` and select the `.venv` interpreter as the kernel.

To run every cell from the command line and save the outputs into the notebook:

```bash
uv run jupyter nbconvert --to notebook --execute --inplace starter/starter.ipynb
```

The notebook runs on CPU and needs no GPU.

### Dependencies

Tested versions, as locked in `uv.lock`:

| Package | Version | Used for |
|---|---|---|
| Python | 3.12 | |
| pandas | 3.0.6 | Data handling |
| numpy | 2.3.5 | Numerical work |
| matplotlib | 3.11.2 | Charts |
| statsmodels | 0.15.0 | ARIMA, SARIMAX, Ljung-Box test |
| scipy | 1.18.1 | Jarque-Bera test, normal probabilities |
| scikit-learn | 1.9.1 | Required by Darts |
| darts[torch] | 0.47.0 | N-BEATS, Darts ARIMA, MAE/RMSE metrics |
| torch | 2.14.1 | Installed with `darts[torch]` |
| chronos-forecasting | 2.3.2 | Chronos-Bolt zero-shot model |
| transformers | 5.19.0 | Installed with `chronos-forecasting` |
| jupyter, ipykernel | 1.1.1, 7.4.0 | Development tools (`dev` group) |

To upgrade everything to the newest compatible versions, run `uv lock --upgrade` and then `uv sync`, re-run the notebook, and update this table.

## Project Instructions

Work through the notebook in order:

1. **Phase 1: Baseline Forecasts.** Naive, 6- and 12-month moving averages, and a linear trend, each compared with the $4.1M target.
2. **Phase 2: Classical Models.** Three ARIMA and two SARIMAX specifications compared by convergence, AIC, and 2025 test error (MAE, RMSE); residual diagnostics (Ljung-Box, Jarque-Bera); the best models refit on all data with 95% prediction intervals and the probability of reaching $4.1M.
3. **Phase 3: Modern Models.** N-BEATS with quantile regression (Darts), a zero-shot Chronos-Bolt forecast (`amazon/chronos-bolt-small` through `chronos-forecasting`), and a Darts ARIMA(2,1,1) baseline.
4. **Phase 4: Comparison and Recommendation.** A comparison table of every model's December 2026 forecast, prediction intervals, gap to $4.1M, test error, and probability of reaching the target, followed by a written recommendation.

### Notes on Chronos

- The rubric specifies `amazon/chronos-bolt-small`. Chronos-Bolt has a different architecture from Chronos-2, and darts' `Chronos2Model` (darts 0.47.0) cannot load it, so the notebook uses Amazon's `chronos-forecasting` package instead.
- Chronos-Bolt predicts nine fixed percentiles, from the 10th to the 90th, so its widest prediction interval is 80%. The notebook therefore compares all models at 80% and also reports 95% intervals where a model can produce them.

### Deliverables

1. Completed notebook with all phases implemented
2. Comparison table showing all models, December 2026 forecasts, prediction intervals, and accuracy metrics
3. At least three charts: baseline comparison, classical model forecasts with intervals, modern model forecasts
4. Written recommendation (500-1000 words) answering:
   - What is the realistic range for December 2026 revenue?
   - What should the company plan for instead of $4.1M?
   - Which forecasting approach would you recommend for ongoing use, and why?

## Data

- `data/revenue.csv` — 60 months of monthly revenue (January 2021 through December 2025), cleaned by the data engineering team

## License

[License](LICENSE.txt)

# S&P 500 EVT Tail-Risk Analysis

## Data

| Item | Value |
|---|---:|
| Series | S&P 500 daily closes |
| Period | 1985-01-02 to 2025-04-22 |
| Observations | 10,154 |
| Return unit | percent log-return |
| Train/test split | before 2018-01-01 / 2018 onward |

## What Is Modeled

Let `X_t` be the S&P 500 closing price on trading day `t`.

```text
R_t = 100 * log(X_t / X_{t-1})
```

`R_t` is the daily percent log-return. The project models the conditional volatility of `R_t`.

The GARCH(1,1)-t model is:

```text
R_t = mu + epsilon_t
epsilon_t = sigma_t * z_t
z_t ~ standardized Student-t(nu)

sigma_t^2 = omega + alpha1 * epsilon_{t-1}^2 + beta1 * sigma_{t-1}^2
```

The estimated object in the volatility stage is `sigma_t`, the conditional standard deviation of returns on day `t`.

After estimating `sigma_t`, returns are standardized:

```text
z_t = (R_t - mu) / sigma_t
```

Then EVT models the left tail of `z_t`. For a threshold `u < 0`, exceedances are:

```text
y_t = u - z_t,  for z_t < u
```

The GPD estimates the tail shape `xi` and scale `sigma` of these positive excesses.

## GARCH(1,1)-t Fit

| Parameter | Estimate |
|---|---:|
| mu | 0.0764 |
| omega | 0.0120 |
| alpha1 | 0.0944 |
| beta1 | 0.9001 |
| nu | 5.763 |
| alpha1 + beta1 | 0.9944 |

## EVT Tail Fit

Left tail convention: exceedances are `z < u`; positive excess is `y = u - z`.

| Threshold method | u | xi | sigma | exceedance rate | z_99 | z_95 |
|---|---:|---:|---:|---:|---:|---:|
| Preliminary 1% | -2.783 | 0.256 | 0.733 | 1.0% | -9.209 | n/a |
| Mean-excess | -1.468 | 0.142 | 0.600 | 7.4% | -5.279 | -3.637 |
| Weissman AMSE | -1.560 | 0.146 | 0.608 | 6.4% | -5.470 | -3.781 |

## Return-Scale VaR

| Threshold method | VaR 99% one-step | VaR 95% one-step | VaR 99% avg sigma | VaR 95% avg sigma |
|---|---:|---:|---:|---:|
| Preliminary 1% | -26.13% | n/a | -9.32% | n/a |
| Mean-excess | -14.95% | -10.27% | -5.31% | -3.64% |
| Weissman AMSE | -15.49% | -10.68% | -5.51% | -3.78% |

## Model Stability Result

GARCH was the more stable volatility filter for the EVT pipeline.

| Model | Parameters | Train observations | Test observations | Ljung-Box p, squared z lag 10 | Ljung-Box p, squared z lag 22 | 99% VaR violations | Expected 99% violations |
|---|---:|---:|---:|---:|---:|---:|---:|
| GARCH(1,1)-t | 7 | 8,288 | 1,833 | 0.0021 | 0.0686 | 1 | 18.33 |

## Files

| Path | Contents |
|---|---|
| `notebooks/01_time_series_modeling.ipynb` | GARCH fit and standardized residual export |
| `notebooks/02_evt_threshold_analysis.ipynb` | POT/GPD thresholds and VaR |
| `notebooks/04_neural_gpd.ipynb` | Neural GPD scale experiment |
| `notebooks/05_lstm_evt.ipynb` | Return-only LSTM volatility filter |
| `notebooks/06_paper_inspired_lstm_evt.ipynb` | OHLCV feature LSTM volatility filter |
| `notebooks/07_model_comparison_stability.ipynb` | Model stability comparison |
| `notebooks/spy.csv` | Raw S&P 500 OHLCV data |
| `notebooks/standardized_residuals.csv` | GARCH-t standardized residuals |
| `notebooks/garch_params.csv` | GARCH handoff parameters |

## Requirements

```bash
pip install -r requirements.txt
```

## Run

```bash
jupyter notebook notebooks/01_time_series_modeling.ipynb
jupyter notebook notebooks/02_evt_threshold_analysis.ipynb
jupyter notebook notebooks/07_model_comparison_stability.ipynb
```

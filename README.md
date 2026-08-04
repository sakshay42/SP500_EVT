# Extreme Value Analysis of S&P 500 Returns

A statistical analysis of tail risk in the S&P 500 index using **Extreme Value Theory (EVT)**, combining GARCH time series modeling with Generalized Pareto Distribution (GPD) fitting to estimate Value at Risk (VaR) for extreme market crashes.

---

## Overview

Standard risk models often underestimate the probability of rare, extreme market events (e.g., Black Monday 1987, the 2008 Financial Crisis, COVID-19 crash 2020). This project addresses that by applying a two-stage approach:

1. **GARCH(1,1) modeling** — captures volatility clustering in S&P 500 log-returns and extracts standardized residuals
2. **GPD fitting via EVT** — fits a Generalized Pareto Distribution to the left tail of standardized residuals, using two principled threshold selection methods

---

## Data

- **Source:** S&P 500 daily closing prices (downloaded from https://stooq.com/q/d/?s=%5Espx)
- **Period:** January 2, 1985 – April 22, 2025
- **Observations:** N = 10,154 trading days
- **Target variable:** Log-returns $R_n = \log(X_n / X_{n-1})$

---

## Methodology

### Stage 1 — GARCH Time Series Analysis

- Diagnosed volatility clustering via ACF/PACF of squared returns
- Selected GARCH(1,1) with **t-distributed innovations** (ν ≈ 5.70) using AIC/BIC over a grid of GARCH(p,q) models with 1 ≤ p, q ≤ 5
- Estimated parameters via MLE; all parameters statistically significant

| Parameter | Estimate |
|-----------|----------|
| μ         | 0.0764   |
| ω         | 0.0120   |
| α₁        | 0.0944   |
| β₁        | 0.9001   |
| ν (df)    | 5.763    |

*(Corrected from an earlier version of this table, which showed μ and ω off by several orders of magnitude and somewhat different α₁/β₁ — those numbers don't match what `01_time_series_modeling.ipynb`'s GARCH(1,1)-t fit actually produces, on this repo's data or the original. The values above are read directly from that notebook's own `arch` model summary output.)*

### Stage 2 — Extreme Value Analysis (Left Tail)

Applied **Peaks Over Threshold (POT)** to the left tail of GARCH standardized residuals. Two threshold selection methods were compared:

#### Method 1: Mean-Excess Plot
- Identifies the threshold beyond which empirical mean excess is approximately linear, via an automatic two-segment (piecewise-linear) changepoint search over the mean-excess curve
- Selected threshold: **u = −1.468**
- GPD fit: ξ = 0.142 (heavy-tailed), σ = 0.600, 7.4% of residuals exceed the threshold

#### Method 2: Weissman-Style Double-Bootstrap AMSE
- Selects the threshold minimizing a genuine bias/variance ("AMSE") estimate of the tail-index estimator, via subsampling *without* replacement at two sizes `n1` and `n2 = n1²/n` (Danielsson, de Haan, Peng, de Vries 2001), combined as `k0 = k1*²/k2*` — validated against synthetic data with a known true shape parameter before being trusted on real residuals
- Selected threshold: **u = −1.560**
- GPD fit: ξ = 0.146 (heavy-tailed), σ = 0.608, 6.4% of residuals exceed the threshold

#### Threshold Comparison

| Method        | Threshold | Shape ξ | Scale σ | q (exceedance %) | VaR 99% | VaR 95% |
|---------------|-----------|---------|---------|-------------------|---------|---------|
| Preliminary   | −2.783    | 0.256   | 0.733   | 1.0%              | −9.209  | −6.071  |
| Mean-Excess   | −1.468    | 0.142   | 0.600   | 7.4%              | −5.279  | −3.637  |
| Weissman AMSE | −1.560    | 0.146   | 0.608   | 6.4%              | −5.470  | −3.781  |

**Trade-off:** the two principled methods now agree closely (ξ within 0.004, thresholds within 0.1 of each other), which cross-validates both. A useful diagnostic surfaced along the way: a Hall-rule pilot fit only on the most extreme ~2% of residuals gives ξ≈0.30 — closer to the preliminary estimate — showing the tail-index estimate is genuinely sensitive to how much of the distribution is included, which is exactly the bias/variance tradeoff threshold selection is meant to navigate.

> An earlier version of this analysis reported u=−0.911 (mean-excess) and u=−0.267 (Weissman) with near-zero shape estimates. Neither number had any code behind it — they were hardcoded with no derivation anywhere in the repo, and the *described* Weissman procedure (bootstrap-resampling a threshold's own exceedances and comparing to that same sample's own estimate) can only ever measure variance, never bias, so it never had a real minimum. The numbers above are the first ones actually computed by code in this repo, with a synthetic-data validation gate checking the method recovers a known ground-truth shape parameter before trusting it here.

---

## Key Results

- S&P 500 log-returns exhibit heavy tails (ξ > 0), consistent with findings in the quantitative risk management literature
- All three major historical crashes (Black Monday 1987, GFC 2008, COVID-19 2020) appear as exceedances beyond the 99% VaR threshold
- The GARCH-EVT pipeline substantially improves on naive GPD fitting by first filtering out volatility clustering
- The mean-excess and double-bootstrap AMSE methods now agree closely with each other, which wasn't true of the earlier (uncoded, hardcoded) version of this analysis

---

## Repository Structure

```
├── notebooks/
│   ├── 01_time_series_modeling.ipynb      # EDA, ACF/PACF, GARCH(1,1) model selection + fit, residual diagnostics
│   ├── 02_evt_threshold_analysis.ipynb    # POT/GPD tail fit: preliminary, mean-excess, and Weissman AMSE thresholds, VaR
│   ├── spy.csv                            # Raw S&P 500 daily closes (input to notebook 1)
│   └── standardized_residuals.csv         # t-GARCH standardized residuals (output of notebook 1, input to notebook 2)
├── EVT_report.pdf           # Written report with figures and methodology
├── requirements.txt
└── README.md
```

The two notebooks run in sequence: `01_time_series_modeling.ipynb` fits the GARCH(1,1)-t model and saves its standardized residuals to `standardized_residuals.csv`; `02_evt_threshold_analysis.ipynb` loads that file and does the EVT/threshold-selection work independently, so it can be re-run (e.g. to try different threshold-selection settings) without refitting GARCH each time.

---

## Requirements

```
numpy
pandas
scipy
matplotlib
seaborn
statsmodels   # ACF/PACF plots
arch          # GARCH modeling
yfinance      # S&P 500 data download
jupyter
```

Install with:
```bash
pip install -r requirements.txt
```

---

## Running the Analysis

```bash
git clone https://github.com/sakshay42/SP500_EVT.git
cd SP500_EVT
pip install -r requirements.txt
jupyter notebook notebooks/01_time_series_modeling.ipynb   # run first, end to end
jupyter notebook notebooks/02_evt_threshold_analysis.ipynb # then this one
```

---

## References

McNeil, A. J., Frey, R., & Embrechts, P. (2015). *Quantitative Risk Management: Concepts, Techniques and Tools* (Revised Edition). Princeton University Press.

---

## Author

**Akshay Sakanaveeti**  
Department of Statistics and Operations Research  
University of North Carolina at Chapel Hill  
sakshay@unc.edu

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
- Selected GARCH(1,1) with **t-distributed innovations** (ν ≈ 5.76) using AIC/BIC over a grid of GARCH(p,q) models with 1 ≤ p, q ≤ 5
- Estimated parameters via MLE; all parameters statistically significant

| Parameter | Estimate |
|-----------|----------|
| μ         | 0.0764   |
| ω         | 0.0120   |
| α₁        | 0.0944   |
| β₁        | 0.9001   |
| ν (df)    | 5.763    |

**Volatility persistence:** α₁ + β₁ = 0.9944, with a 95% CI (from the model's parameter covariance matrix) of [0.9878, 1.0010] — this straddles 1, so a literal unit root in volatility (IGARCH) can't be ruled out at the 95% level. Practically: the implied volatility shock half-life is ~124 trading days, and the closed-form unconditional variance `ω/(1-α₁-β₁)` is unstable this close to the boundary, which is why the "average-sigma" VaR below uses the mean of the realized conditional volatility series instead of that formula.

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

`z_99`/`z_95` are GPD-based quantiles of the *standardized* residuals — how many standard deviations into the tail, not a return-scale loss. `z_95` is left blank for the preliminary threshold: at q=1.0% exceedance, a 95% VaR would need a 5% tail probability, which is less extreme than the threshold itself — the GPD approximation isn't valid there, so it's reported as not applicable rather than a misleading extrapolated number.

| Method        | Threshold | Shape ξ | Scale σ | q (exceedance %) | z_99   | z_95   |
|---------------|-----------|---------|---------|-------------------|--------|--------|
| Preliminary   | −2.783    | 0.256   | 0.733   | 1.0%              | −9.209 | n/a    |
| Mean-Excess   | −1.468    | 0.142   | 0.600   | 7.4%              | −5.279 | −3.637 |
| Weissman AMSE | −1.560    | 0.146   | 0.608   | 6.4%              | −5.470 | −3.781 |

**Trade-off:** the two principled methods agree closely (ξ within 0.004, thresholds within 0.1 of each other), which cross-validates both. A useful diagnostic: a Hall-rule pilot fit only on the most extreme ~2% of residuals gives ξ≈0.30 — closer to the preliminary estimate — showing the tail-index estimate is genuinely sensitive to how much of the distribution is included, which is exactly the bias/variance tradeoff threshold selection is meant to navigate.

#### Return-Scale VaR

`z_q` above is in standardized-residual units. Actual VaR in log-return terms is `VaR = μ̂ + σ̂ · z_q`, with two versions of σ̂: the one-step-ahead GARCH forecast (μ̂=0.000764, σ̂=0.028456 — the relevant number for "tomorrow's VaR"), and the mean of the realized in-sample conditional volatility (σ̂=0.010205 — a smoother, less date-specific figure).

| Method        | VaR 99% (one-step-ahead) | VaR 95% (one-step-ahead) | VaR 99% (avg-σ) | VaR 95% (avg-σ) |
|---------------|---------------------------|---------------------------|------------------|------------------|
| Preliminary   | −26.13%                   | n/a                        | −9.32%           | n/a              |
| Mean-Excess   | −14.95%                   | −10.27%                    | −5.31%           | −3.64%           |
| Weissman AMSE | −15.49%                   | −10.68%                    | −5.51%           | −3.78%           |

The one-step-ahead figures are large because they're conditional on the GARCH model's forecast volatility at the end of the sample (April 2025), which was elevated; the average-σ figures reflect typical, unconditional daily risk instead.

#### t-Distribution Consistency Check

For t-distributed GARCH innovations with ν degrees of freedom, EVT implies the tail index should satisfy ξ ≈ 1/ν. Here, 1/ν̂ = 1/5.763 = 0.1735, compared to ξ = 0.142 (mean-excess, 95% CI [0.061, 0.224]) and ξ = 0.146 (Weissman, 95% CI [0.057, 0.234]) — 1/ν̂ falls inside both CIs (shape-parameter CIs from the standard GPD-MLE asymptotic variance, (1+ξ)²/n_exceedances). Since ν̂ comes purely from the GARCH stage and ξ purely from the EVT stage fit only to the extreme tail, this agreement is a genuine cross-check that the two stages are telling a consistent story about tail heaviness.

---

## Key Results

- S&P 500 log-returns exhibit heavy tails (ξ > 0), consistent with findings in the quantitative risk management literature
- All three major historical crashes (Black Monday 1987, GFC 2008, COVID-19 2020) appear as exceedances beyond the 99% VaR threshold
- The GARCH-EVT pipeline substantially improves on naive GPD fitting by first filtering out volatility clustering
- The mean-excess and double-bootstrap AMSE methods agree closely with each other, cross-validating both threshold choices
- The fitted GPD shape parameter is consistent with the GARCH-t model's own degrees-of-freedom estimate (ξ ≈ 1/ν̂), a cross-check between the two independently-fit stages of the pipeline
- Volatility persistence (α₁+β₁ ≈ 0.994) is close enough to a unit root that a literal IGARCH process can't be ruled out at 95% confidence, which is why this analysis reports both a one-step-ahead and an average-σ VaR rather than relying on the (unstable, near this boundary) closed-form unconditional variance

---

## Repository Structure

```
├── notebooks/
│   ├── 01_time_series_modeling.ipynb      # EDA, ACF/PACF, GARCH(1,1) model selection + fit, residual diagnostics
│   ├── 02_evt_threshold_analysis.ipynb    # POT/GPD tail fit: preliminary, mean-excess, and Weissman AMSE thresholds, VaR
│   ├── spy.csv                            # Raw S&P 500 daily closes (input to notebook 1)
│   ├── standardized_residuals.csv         # t-GARCH residuals + conditional volatility + mu (output of notebook 1, input to notebook 2)
│   └── garch_params.csv                   # One-step-ahead sigma forecast, mu, nu, alpha1, beta1 (output of notebook 1, input to notebook 2)
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

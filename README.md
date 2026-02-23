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
| μ         | 3.39e+04 |
| ω         | 2.67e+10 |
| α₁        | 0.1135   |
| β₁        | 0.8674   |
| ν (df)    | 5.697    |

### Stage 2 — Extreme Value Analysis (Left Tail)

Applied **Peaks Over Threshold (POT)** to the left tail of GARCH standardized residuals. Two threshold selection methods were compared:

#### Method 1: Mean-Excess Plot
- Identifies the threshold beyond which empirical mean excess is approximately linear
- Selected threshold: **u = −0.911**
- GPD fit: ξ = 0.039 (slightly heavy-tailed), σ = 0.678

#### Method 2: Weissman Bootstrap-MSE
- Selects threshold minimizing bootstrap-estimated MSE of the tail-index estimator
- Selected threshold: **u = −0.267**
- GPD fit: ξ = 0.001 (near-exponential tail), σ = 0.734

#### Threshold Comparison

| Method        | Threshold | Shape ξ | Scale σ | VaR 99% | VaR 95% |
|---------------|-----------|---------|---------|---------|---------|
| Preliminary   | −2.783    | 0.256   | 0.733   | −9.112  | −6.006  |
| Mean-Excess   | −0.911    | 0.039   | 0.678   | −4.290  | −3.027  |
| Weissman-MSE  | −0.267    | 0.001   | 0.734   | −3.617  | −2.431  |

**Trade-off:** Mean-excess gives lower variance; Weissman gives lower bias at the extreme tail.

---

## Key Results

- S&P 500 log-returns exhibit heavy tails (ξ > 0), consistent with findings in the quantitative risk management literature
- All three major historical crashes (Black Monday 1987, GFC 2008, COVID-19 2020) appear as exceedances beyond the 99% VaR threshold
- The GARCH-EVT pipeline substantially improves on naive GPD fitting by first filtering out volatility clustering

---

## Repository Structure

```
├── notebooks/
│   └── analysis.ipynb       # Full analysis: EDA, GARCH fitting, GPD, VaR plots
├── EVT_report.pdf           # Written report with figures and methodology
├── requirements.txt
└── README.md
```

---

## Requirements

```
numpy
pandas
scipy
matplotlib
arch          # GARCH modeling
yfinance      # S&P 500 data download
```

Install with:
```bash
pip install -r requirements.txt
```

---

## Running the Analysis

```bash
git clone https://github.com/<your-username>/extreme-value-analysis-sp500.git
cd extreme-value-analysis-sp500
pip install -r requirements.txt
jupyter notebook notebooks/analysis.ipynb
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

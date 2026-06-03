# PCA Eigen-Portfolio with Cryptocurrencies

> Exam project — *N.F.*
> All rights reserved. See `LICENSE` for details.

---

## Overview

This project applies **Principal Component Analysis (PCA)** to the top 50 cryptocurrencies by market capitalization to extract latent risk factors and construct **eigen-portfolios** — portfolios whose weights are the eigenvectors of the return covariance matrix. The analysis includes covariance **denoising via Marchenko-Pastur (Random Matrix Theory)**, K-Means clustering on factor loadings, Markowitz efficient frontier optimization, and economic attribution of principal components using external factors (Bitcoin, VIX, gold, geopolitical risk index).

All model estimation is performed on **in-sample data (June 2022 – June 2024)**; out-of-sample performance is evaluated on the **residual year (June 2024 – June 2025)**, with Bitcoin (BTC) as the external benchmark.

---

## Project Structure

```
.
├── crypto_eigenportfolio.ipynb   # Main Jupyter notebook (full analysis)
└── crypto_dataset.csv            # Cryptocurrency universe with tickers and market cap
```

> **External data required (not included):** The notebook uses the Geopolitical Risk Index (GPR) daily series by Caldara & Iacoviello. Download `data_gpr_daily_recent.xls` from [matteoiacoviello.com/gpr.htm](https://www.matteoiacoviello.com/gpr.htm) and place it in the same folder as the notebook before running.

---

## Methodology

### 1. Data Collection and Universe Construction
The cryptocurrency universe is built from `crypto_dataset.csv`, which lists tickers ranked by market capitalization. Stablecoins and assets with insufficient price history are filtered out automatically, retaining the **top 50 cryptocurrencies** (excluding BTC, which is kept as a separate benchmark). Daily adjusted prices are downloaded via `yfinance`. The risk-free rate is proxied by the 13-Week T-Bill yield (`^IRX`) from Yahoo Finance, annualized and converted to a daily rate.

Daily **log-returns** are computed and excess returns are obtained by subtracting the daily risk-free rate. Bitcoin excess returns are computed separately and used as an external benchmark throughout.

### 2. In-Sample / Out-of-Sample Split

| Period | Dates |
|--------|-------|
| **In-Sample** | June 2022 → June 2024 |
| **Out-of-Sample** | June 2024 → June 2025 |

All PCA estimation, covariance matrix computation, cluster assignment, and portfolio weight construction are performed exclusively on in-sample data.

### 3. PCA on Excess Returns
PCA is applied to the standardized in-sample excess return matrix. The number of statistically significant components is determined via two criteria: the **scree plot** (Kaiser threshold: eigenvalue > 1) and the **Marchenko-Pastur upper bound** from Random Matrix Theory. Two components are retained.

### 4. Eigen-Portfolio Construction
Each eigen-portfolio is defined by an eigenvector of the covariance matrix, with weights normalized so that the sum of absolute values equals 1 (allowing both long and short positions). For each eigen-portfolio, in-sample Sharpe ratio, mean return, and standard deviation are computed and compared.

An **ensemble eigen-portfolio** is constructed as a Sharpe-weighted combination of the top two eigen-portfolios, to reduce the overfitting risk associated with selecting a single component.

### 5. Efficient Frontier and Portfolio Positioning
Using `PyPortfolioOpt`, the Markowitz Efficient Frontier is computed on in-sample excess returns and the sample covariance matrix. The **Global Minimum Variance (MVP)** and **Tangency (Maximum Sharpe Ratio)** portfolios are identified. Eigen-portfolios are projected onto the risk-return plane to assess how closely they approximate the efficient frontier.

### 6. K-Means Clustering on Factor Loadings
K-Means clustering is applied to the factor loadings of the first two principal components, grouping cryptocurrencies by their latent factor profile. The optimal number of clusters (K = 4) is selected via Elbow method and Silhouette score. An **equal-weighted portfolio** is constructed for each cluster and evaluated out-of-sample against BTC.

Notable finding: Cluster 1 (composed entirely of gold-backed tokens: PAXG and XAUT) achieves a CAGR of ~+20% with Sharpe 1.362 OOS — structurally decorrelated from the rest of the crypto market. The remaining clusters underperform BTC, with idiosyncratic small-cap tokens driving the worst drawdowns.

### 7. Covariance Matrix Denoising (Marchenko-Pastur)
The sample covariance matrix is denoised using **Random Matrix Theory**. Eigenvalues below the Marchenko-Pastur upper bound (λ_max = (1 + 1/√q)², where q = T/N) are treated as noise and replaced with their average. The denoised covariance matrix is reconstructed as Q × Λ_denoised × Qᵀ, rescaled to original asset variances. The Efficient Frontier and tangency portfolio are recomputed on the denoised matrix and compared to the sample version in terms of weight concentration, frontier shape, and OOS performance.

### 8. Economic Attribution of Principal Components
The PCA scores (time series of each component) are regressed on a set of external factors via OLS:

| Factor | Source | Transformation |
|--------|--------|---------------|
| Bitcoin (BTC) | Yahoo Finance | Log-return |
| VIX | Yahoo Finance (`^VIX`) | Day-on-day % change |
| Gold (GLD) | Yahoo Finance | Day-on-day % change |
| US 10Y Treasury (`^TNX`) | Yahoo Finance | Day-on-day diff (bps) |
| Geopolitical Risk Index (GPR) | Caldara & Iacoviello | Level + day-on-day diff |

A **loading-factor attribution table** is produced to map each principal component to its dominant economic driver.

### 9. Portfolio Rotation
Based on the economic attribution, a rotated portfolio is constructed by combining eigen-portfolios in proportion to their BTC beta exposure, with a hedge against the geopolitical risk factor. OOS performance is compared to EP0 and the BTC benchmark.

---

## Requirements

```bash
pip install numpy pandas matplotlib seaborn plotly yfinance fmfinance quantstats \
            statsmodels scipy scikit-learn pypfopt openpyxl xlrd
```

```bash
pip install https://github.com/FraMedda/fmfinance/archive/refs/heads/main.zip
```

Python 3.9+ recommended. Run in Jupyter Notebook or JupyterLab.

---

## Usage

1. Clone the repository
2. Install the required packages (see above)
3. Download `data_gpr_daily_recent.xls` from [matteoiacoviello.com/gpr.htm](https://www.matteoiacoviello.com/gpr.htm) and place it in the repo folder
4. Open and run `crypto_eigenportfolio.ipynb` sequentially from top to bottom

---

## License

This project is protected by copyright. See the [`LICENSE`](LICENSE) file for full terms.
**Reusing, copying, modifying, or redistributing** the code or data without explicit written permission from the author is not permitted.

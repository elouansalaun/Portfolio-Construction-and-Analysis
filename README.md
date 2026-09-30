# Portfolio Construction and Analysis with Python

A step-by-step series of Jupyter notebooks that goes from computing simple returns to **Monte Carlo simulation of dynamic risk budgeting** between a Performance-Seeking Portfolio (PSP) and a Goal-Hedging Portfolio (GHP).

All reusable functions are collected in a single toolkit, [`risk_kit.py`](risk_kit.py), which grows as the notebooks progress. It covers risk metrics, Markowitz optimisation, CPPI, stochastic simulations (GBM and CIR), bond pricing and allocation backtesting.



---

## Notebooks overview

| # | Notebook | Summary |
|---|----------|---------|
| 1 | [Bases of returns](1_Bases_of_returns.ipynb) | Computing simple returns from prices (`pct_change`, `shift`), compounding returns and annualising monthly, quarterly and daily returns. |
| 2 | [Volatility and risk](2_Volatilty_and_risk.ipynb) | Standard deviation from first principles, annualised volatility, and risk-adjusted returns (return/vol ratio and Sharpe ratio) for small-cap vs large-cap US stocks. |
| 3 | [Drawdown](3_Drawdown.ipynb) | Building a wealth index, tracking previous peaks and computing the maximum drawdown. Large caps lost **-85.9%** in 1929–1932 and **-55.3%** in 2008–2009. |
| 4 | [Deviation from normality](4_Deviation_from_normality.ipynb) | Skewness, kurtosis and the Jarque-Bera test on EDHEC hedge fund indices. Only *CTA Global* passes the normality test. |
| 5 | [Downside measures](5_Downside_measures.ipynb) | Semi-deviation, historic VaR, Gaussian VaR, Cornish-Fisher (modified) VaR and CVaR (expected shortfall). |
| 6 | [Efficient frontier](6_Efficient_frontier.ipynb) | Portfolio return and volatility in matrix form, the 2-asset frontier, then the N-asset efficient frontier with `scipy.optimize` on the 30 Fama-French industry portfolios. |
| 7 | [Max Sharpe ratio](7_Max_sharpe_ratio.ipynb) | Finding the tangency (MSR) portfolio and plotting the Capital Market Line. |
| 8 | [Markowitz robustness & GMV](8_Markowitz_robustess_and_gmv.ipynb) | Shows how sensitive Markowitz weights are to errors in expected returns, and introduces the Equally-Weighted and Global Minimum Variance portfolios as more robust alternatives. |
| 9 | [Limits of diversification](9_Limits_of_diversification.ipynb) | Building a cap-weighted total market index and computing rolling 36-month correlations. Correlations rise when markets fall (ρ ≈ **-0.28** between trailing returns and average correlation). |
| 10 | [CPPI & drawdown constraints](10_CPPI_and_drawdown_constraints.ipynb) | Backtesting Constant Proportion Portfolio Insurance (CPPI) on industries and on the market index, including a variant with an explicit maximum-drawdown constraint. |
| 11 | [Random walk & Monte Carlo](11_Random_walk_and_Monte_Carlo.ipynb) | Simulating stock prices with Geometric Brownian Motion, vectorising the generator and fixing the drift bias. |
| 12 | [Interactive Monte Carlo CPPI](12_Interactive_plot_and_MonteCarlo_CPPI_sim.ipynb) | `ipywidgets` dashboards for GBM paths and Monte Carlo CPPI, with terminal-wealth histograms and floor-violation statistics. |
| 13 | [Liabilities & funding ratio](13_Liabilities_and_funding%20ratio.ipynb) | Present value of liabilities, zero-coupon discounting and the funding ratio. Even cash is risky relative to liabilities when rates move. |
| 14 | [Interest rates & liability hedging](14_Interest_rate_changes_and_liability_hedging.ipynb) | The Cox-Ingersoll-Ross (CIR) short-rate model, simulated zero-coupon bond prices and terminal funding ratios when hedging with cash vs zero-coupon bonds. |
| 15 | [GHP construction](15_GHP_construction.ipynb) | Coupon bond pricing, Macaulay duration and duration matching with a short and a long bond to immunise a liability stream. |
| 16 | [Monte Carlo with CIR](16_Monte_Carlo_sim_using_CIR.ipynb) | Pricing coupon bonds along CIR rate paths, computing bond total returns and simulating a 70/30 equity/bond portfolio. |
| 17 | [Naive risk budgeting](17_Naive_risk_budgeting_psp_ghp.ipynb) | A generic `bt_mix` backtester with fixed-mix and glide-path allocators, and terminal-wealth statistics (breach probability and expected shortfall). |
| 18 | [Dynamic risk budgeting](18_Monte_Carlo_sim_of_dynamic_risk_budj.ipynb) | CPPI-style floor and max-drawdown allocators between PSP and GHP, tested on 5,000 Monte Carlo scenarios and backtested on the US market since 1990. |

---

## Main results

The last notebooks answer one practical question: **how do you split wealth between a risky PSP and a safe GHP, so that you capture upside while protecting a minimum level of wealth?**

### 1. Liability hedging: cash is not a safe asset (notebooks 13–15)

- When performance is measured by the **funding ratio** (assets / PV of liabilities) rather than asset value, cash becomes risky. With assets of 5 and liabilities worth 6.23, the funding ratio falls from **80.2% to 77.2%** when rates drop from 3% to 2%, even though the assets did not lose value.
- Across 10,000 CIR scenarios, hedging a 10-year liability with **zero-coupon bonds** gives a nearly constant terminal funding ratio. Holding cash leaves it widely dispersed.
- **Duration matching** with a 10y and a 20y bond (48.3% / 51.7%) matches the liability duration of 10.96 years. The funding ratio barely moves when rates shift by ±1%:

| Hedge portfolio | Δ funding ratio if rates → 3% | Δ funding ratio if rates → 5% |
|---|---|---|
| Long bond only | +2.75% | -2.23% |
| Short bond only | -2.61% | +2.72% |
| **Duration-matched** | **+0.16%** | **+0.16%** |

### 2. Naive risk budgeting: static mixes and glide paths (notebook 17)

10-year horizon, 500 scenarios (equities: GBM μ = 7%, σ = 15%; bonds: CIR-driven 10y/30y coupon bonds). Terminal wealth per $1 invested, floor = 0.80:

| Strategy | Mean | Std | P(breach) | E(shortfall) |
|---|---|---|---|---|
| 100% Bonds | 1.38 | 0.10 | 0% | – |
| 100% Equities | 1.92 | 0.88 | 4.0% | 11.1% |
| 70/30 fixed mix | 1.76 | 0.55 | 0.8% | 8.3% |
| Glide path 80% → 20% | 1.65 | 0.41 | 0.4% | 6.2% |

Static mixes and glide paths **reduce downside risk, but they cost expected return and never guarantee the floor**.

### 3. Dynamic risk budgeting: floor and drawdown allocators (notebook 18)

5,000 scenarios over 10 years. The GHP is a zero-coupon bond, the floor is 75% of initial wealth, and *m* is the CPPI multiplier:

| Strategy | Mean | Std | P(breach) | E(shortfall) |
|---|---|---|---|---|
| Zero-coupon (GHP) | 1.34 | 0.00 | 0% | – |
| 100% Equities (PSP) | 1.98 | 1.00 | 3% | 12% |
| 70/30 fixed mix | 1.76 | 0.60 | 1% | 6% |
| **Floor 75%, m = 3** | **1.95** | 1.00 | **0%** | – |
| Floor 75%, m = 1 | 1.63 | 0.44 | 0% | – |
| Floor 75%, m = 5 | 1.97 | 1.00 | 0% | – |
| Floor 75%, m = 10 | 1.97 | 1.01 | 3% | ≈ 0% |
| Max-drawdown 25% (cash GHP) | 1.63 | 0.55 | 0% | – |

- The **floor allocator (m = 3)** keeps almost all of the equity upside (1.95 vs 1.98) and **never breaches the floor**, whereas the 70/30 mix gives up much more return and still breaches it.
- **Gap risk:** with a large multiplier (m = 10), the portfolio cannot de-risk fast enough between rebalancing dates, and floor breaches come back (3%).
- The **drawdown allocator** caps the maximum drawdown at 25% in every scenario (worst case: **-23.2%**).

### 4. Historical backtest on the US market, 1990–2018 (notebook 18)

Max-drawdown allocator (25%, m = 5) between the cap-weighted total market index and cash at 3%:

| | Ann. return | Ann. vol | Sharpe | Max drawdown | CF VaR (5%) |
|---|---|---|---|---|---|
| Market | 9.61% | 14.54% | 0.44 | **-49.99%** | 6.69% |
| MaxDD 25% strategy | 9.01% | 11.28% | **0.52** | **-24.42%** | 5.00% |

Giving up about **0.6% of annual return halves the maximum drawdown** and improves the Sharpe ratio, and the drawdown constraint holds on real data.

---

## Repository structure

```
.
├── 1_ … 18_*.ipynb   # Notebooks, to be read in order
├── risk_kit.py       # Toolkit of reusable functions built along the notebooks
└── data/             # Datasets
    ├── Portfolios_Formed_on_ME_monthly_EW.csv   # Fama-French size portfolios
    ├── edhec-hedgefundindices.csv               # EDHEC hedge fund indices
    ├── ind30_m_*.csv / ind49_m_*.csv            # Fama-French industry portfolios (returns, size, nb firms)
    ├── F-F_Research_Data_Factors*.csv           # Fama-French factors
    └── ...                                      # S&P 500, CAC 40, BRK-A, sample prices
```

### Main `risk_kit` functions

| Category | Functions |
|---|---|
| Data loading | `get_ffme_returns`, `get_hfi_returns`, `get_ind_returns`, `get_total_market_index_returns` |
| Risk metrics | `annualize_rets`, `annualize_vol`, `sharpe_ratio`, `drawdown`, `skewness`, `kurtosis`, `is_normal`, `semideviation`, `var_historic`, `var_gaussian`, `cvar_historic`, `summary_stats` |
| Optimisation | `portfolio_return`, `portfolio_vol`, `minimize_vol`, `msr`, `gmv`, `plot_ef` |
| Insurance | `run_cppi` |
| Simulation | `gbm`, `cir` |
| Fixed income & ALM | `discount`, `pv`, `funding_ratio`, `bond_cash_flows`, `bond_price`, `bond_total_return`, `macaulay_duration`, `match_durations` |
| Allocation backtesting | `bt_mix`, `fixedmix_allocator`, `glidepath_allocator`, `floor_allocator`, `drawdown_allocator`, `terminal_values`, `terminal_stats` |

---

## Getting started

```bash
git clone <repo-url>
cd Portfolio-Construction-and-Analysis
pip install numpy pandas scipy matplotlib seaborn ipywidgets jupyter
jupyter notebook
```



The work follows the EDHEC *Introduction to Portfolio Construction and Analysis with Python* curriculum.

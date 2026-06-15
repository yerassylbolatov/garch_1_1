# GARCH Volatility Modeling — 2004–2009

A Python-based quantitative risk analysis project that models time-varying volatility using GARCH(1,1) and demonstrates why standard parametric VaR — which assumes constant volatility — systematically fails during periods of market stress.

The core question this project answers: **if volatility changes every day, why do standard risk models treat it as a constant?**

---

## Motivation

Standard parametric VaR uses a single, fixed volatility estimate for the entire sample period. This works reasonably well in calm markets — but during the 2008 financial crisis, volatility didn't just increase, it **clustered**: large moves were followed by more large moves, and the elevated regime persisted for months.

GARCH (Generalized Autoregressive Conditional Heteroskedasticity) captures exactly this behavior. By modeling volatility as a process that evolves daily based on recent shocks, GARCH produces a time-varying VaR that adapts to the market regime — rising ahead of large losses and falling as calm returns.

This project runs GARCH(1,1) across the full 2004–2009 period, spanning both the pre-crisis calm and the crisis itself, making the contrast between adaptive and static risk models visually and statistically clear.

---

## Tickers Analyzed

| Ticker | Description | Story |
|--------|-------------|-------|
| `SPY`  | S&P 500 ETF | Broad market — persistence near 0.99 |
| `XLF`  | Financials sector ETF | Highest volatility spike in 2008 |
| `GLD`  | Gold ETF | Flight to safety — different volatility dynamics |
| `XLP`  | Consumer staples ETF | Defensive — low volatility clustering |
| `BAC`  | Bank of America | Single-stock — most dramatic GARCH response |

---

## Project Structure

### `garch_var_report(ticker, start_date, end_date, confidence, portfolio_value)`

The core computation function. Fits a GARCH(1,1) model to log returns and returns a full analysis dictionary.

**Parameters:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `ticker` | — | Stock/ETF symbol (e.g. `"SPY"`) |
| `start_date` | — | Start date in `"YYYY-MM-DD"` format |
| `end_date` | — | End date in `"YYYY-MM-DD"` format |
| `confidence` | `0.95` | VaR confidence level |
| `portfolio_value` | `100_000` | Portfolio size in dollars |

**Returns a dict containing:**

| Key | Description |
|-----|-------------|
| `returns` | Log returns series |
| `omega` | GARCH baseline variance parameter |
| `alpha` | Reaction coefficient — sensitivity to recent shocks |
| `beta` | Persistence coefficient — inertia of volatility |
| `persistence` | `alpha + beta` — how slowly shocks decay |
| `conditional_vol` | Daily conditional volatility σ_t series |
| `garch_var_series` | Daily time-varying VaR in dollars |
| `mean_garch_var` | Average GARCH VaR over the period |
| `param_var` | Standard parametric VaR for comparison |
| `confidence` | Confidence level used |
| `portfolio_value` | Portfolio value used |

---

### `plot_garch_report(results)`

Takes a results dict and produces a 2×2 figure:

- **Top left** — GARCH VaR (daily, time-varying) vs flat parametric VaR. The gap between them during 2008 is the main visual result.
- **Top right** — Conditional volatility σ_t vs constant std. Shows volatility clustering — elevated regimes that persist for months.
- **Bottom left** — Daily returns as bars with the GARCH VaR threshold overlaid. Shows how many returns breached the threshold and when.
- **Bottom right** — GARCH(1,1) parameter bar chart with persistence (α + β) annotated.

---

## Key Concepts

### GARCH(1,1) Variance Equation

```
σ²_t = ω + α * ε²_(t-1) + β * σ²_(t-1)
```

| Term | Meaning |
|------|---------|
| `ω` | Long-run baseline variance |
| `α * ε²_(t-1)` | Reaction to yesterday's shock |
| `β * σ²_(t-1)` | Persistence of yesterday's volatility |

### Persistence (α + β)

The sum `α + β` measures how slowly volatility shocks decay:

- Close to 1 → shocks persist for weeks or months (typical for financial returns)
- Far from 1 → shocks dissipate quickly

For SPY over 2004–2009, `α + β ≈ 0.989` — meaning a volatility spike in October 2008 takes weeks to fade, not days.

### Why Standard VaR Fails

Parametric VaR computes:

```
VaR = |μ + z * σ| * portfolio_value
```

Where `σ` is a single constant estimated over the full sample. During the crisis this constant is pulled upward by extreme days — but it reacts too slowly and doesn't capture day-to-day volatility dynamics.

GARCH VaR computes the same formula but with `σ_t` — a different value every day. In calm periods it stays low. In stressed periods it rises quickly. This makes it a far more accurate risk signal in real time.

---

## Example Usage

```python
# Full period spanning calm and crisis
g = garch_var_report("SPY", "2004-01-01", "2009-12-31")

# View key parameters
print(f"Alpha:       {g['alpha']:.4f}")
print(f"Beta:        {g['beta']:.4f}")
print(f"Persistence: {g['persistence']:.4f}")
print(f"Mean GARCH VaR:  ${g['mean_garch_var']:,.0f}")
print(f"Parametric VaR:  ${g['param_var']:,.0f}")

# Visualize
plot_garch_report(g)
```

---

## Sample Output — SPY 2004–2009

```
Alpha:           0.0737
Beta:            0.9150
Persistence:     0.9888
Mean GARCH VaR:  $1,837
Parametric VaR:  $2,104
```

The mean GARCH VaR is lower than parametric VaR because GARCH correctly identifies that most of the sample period was calm — the constant parametric estimate is inflated by the crisis months.

---

## Connection to Previous Project

This project is a direct extension of the **Financial Crisis VaR Simulation (2007–2009)** project:

- That project showed *that* standard VaR models failed during the crisis
- This project shows *why* — constant volatility assumption breaks down when volatility clusters
- Together they form a complete narrative: failure diagnosis → model improvement

---

## Dependencies

```
yfinance
numpy
pandas
matplotlib
scipy
arch
```

Install with:
```bash
pip install yfinance numpy pandas matplotlib scipy arch
```

---

## Author

Built as a quantitative finance portfolio project focusing on volatility modeling, time-varying risk estimation, and the limitations of static risk models during market stress periods.

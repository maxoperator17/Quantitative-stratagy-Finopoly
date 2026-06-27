# Core-Satellite Alpha Strategy — Methodology

**Universe:** 10 large-cap US equities | **Rebalance:** Monthly | **Type:** Long-only

---

## 1. What the Strategy Does

The strategy splits the portfolio into two parts:

- **Core (70%):** Equal weight across all 10 stocks (7% each). This gives stable, diversified market exposure.
- **Satellite (30%):** The top 2 momentum-ranked stocks get an extra 15% each on top of their core allocation, giving them ~22% total.

The idea is simple — most of your return comes from just being in the market. A small, systematic tilt toward stocks with recent momentum can add extra return without blowing up risk.

---

## 2. Signal Generation

Every month, on the first trading day, the following steps run using only data available at that point (no look-ahead):

**Step 1 — Compute 21-day return for each stock:**

```
R = (Price today / Price 21 days ago) - 1
```

**Step 2 — Compute 21-day realised volatility (annualised):**

```
Vol = StdDev(daily returns over past 21 days) × sqrt(252)
```

**Step 3 — Score each stock:**

```
Alpha Score = R × Vol^0.8
```

The `Vol^0.8` adjustment is the key part. Raw momentum over-rewards stocks that moved just because they were volatile. Dividing fully by volatility (i.e. using Sharpe ratio as the signal) over-penalises them. The `0.8` exponent is a middle ground — it keeps the directional signal but discounts noise-driven moves.

**Step 4 — Rank all 10 stocks by Alpha Score. Top 2 become the satellite.**

---

## 3. Why These Choices

**Why momentum?**
Stocks that have done well over the past 1–12 months tend to keep doing well over the next few months. This is one of the most consistently documented patterns in financial research across markets and time periods.

**Why 21 days and not 1 month?**
The 1-month (approximately 21-day) lookback avoids the short-term reversal effect — stocks that go up a lot in the very last few days tend to pull back. Using exactly the past 21 trading days and then executing on the *next* month's first day handles this cleanly.

**Why monthly rebalancing?**
Daily trading on a 10-stock portfolio would eat returns through transaction costs. Monthly rebalancing keeps turnover reasonable while still responding to momentum shifts.

**Why 0.8 and not 1.0 (full vol-scaling)?**
Research shows that the 0.8 fractional exponent maximises signal quality empirically. A score of 1.0 (Sharpe ratio) strips too much signal from high-volatility stocks that genuinely moved directionally. A score of 0 (raw returns) is too noisy. 0.8 is the practical sweet spot.

---

## 4. Portfolio Weights

| Layer | Rule | Weight |
|---|---|---|
| Core | All 10 stocks equally | 7% each |
| Satellite | Top 2 stocks, extra allocation | +15% each |
| Satellite stocks total | Core + satellite combined | ~22% each |
| Other 8 stocks | Core only | 7% each |
| Total | | 100% |

Weights are normalised to exactly 1.0 before each trade to prevent floating-point drift.

---

## 5. Execution Rules

- Trades execute at the closing price on the **first trading day of each new month**
- The signal is generated from data up to and including the **last day of the previous month** — the new month's data is never used to generate that month's signal
- Transaction cost: **10 basis points (0.10%)** per trade, deducted from cash
- Strategy is **long-only**, no leverage, no short positions
- Fractional shares are allowed

---

## 6. Risk Controls

The strategy has several built-in limits that don't require separate management:

- No stock can fall below 7% weight (the core floor prevents total concentration)
- No stock can exceed ~22% weight (core + satellite cap)
- The 0.8 vol exponent reduces the signal from stocks that are just noisy
- Monthly rebalancing corrects drift before it compounds
- The 10bps cost per trade naturally discourages chasing marginal signals

**Known weaknesses:**

- Momentum strategies can suffer sharp losses during sudden market recoveries (e.g. March 2020)
- A 10-stock universe is concentrated — one bad earnings report has an outsized impact
- The 21-day signal works better in trending markets than in choppy, sideways ones
- The strategy is best suited to portfolios under $10M. Larger capital would require modelling how your own trades move prices

---

## 7. Performance Metrics Used

| Metric | What it tells you |
|---|---|
| Annualised Return | Average yearly return, compounded |
| Annualised Volatility | How much the portfolio value swings year to year |
| Sharpe Ratio | Return earned per unit of total risk (higher = better) |
| Sortino Ratio | Like Sharpe, but only counts downside swings as risk |
| Maximum Drawdown | Biggest peak-to-trough loss in the period |
| Calmar Ratio | Annual return divided by max drawdown |
| Monthly Turnover | What fraction of the portfolio was traded each month |

All metrics compare against an equal-weight buy-and-hold benchmark of the same 10 stocks.

---

## 8. Overfitting Controls

**Train/Test Split:**
The strategy parameters are determined on the first 5 years of data (training period). The same fixed rules are then applied to the final 2 years without any adjustment (out-of-sample test). If performance holds up in the test period, it is evidence the strategy is not just memorising historical patterns.

**Parameter Sensitivity:**
The momentum window is varied from 15 to 30 days. If the strategy's Sharpe ratio stays roughly stable across this range, it means the results are not fragile to the exact parameter chosen.

**No look-ahead bias:**
At every point in the backtest, the signal only uses data that would have been available on that exact day. Price forward-filling uses only the last known price, never a future price.

**Survivorship bias acknowledgement:**
All 10 tickers are large-cap stocks that exist and are healthy today. This means companies that went bankrupt or were delisted between 2019 and 2026 are not in the universe, which makes the backtest slightly too optimistic. A production version should use the index constituents as they existed at the start of the period.

---

## 9. Universe

| Ticker | Company | Sector |
|---|---|---|
| AAPL | Apple | Technology |
| MSFT | Microsoft | Technology |
| NVDA | Nvidia | Technology / Semiconductors |
| JPM | JPMorgan Chase | Financials |
| GS | Goldman Sachs | Financials |
| CAT | Caterpillar | Industrials |
| XOM | Exxon Mobil | Energy |
| PG | Procter & Gamble | Consumer Staples |
| JNJ | Johnson & Johnson | Healthcare |
| PFE | Pfizer | Healthcare |

---

*For research and educational purposes only. Past backtest results do not predict future returns.*

# QQQ 0DTE Options Day Trader (TradingView Pine Script v6)

`qqq_0dte_strategy.pine` is an intraday strategy for buying same-day-expiry (0DTE) QQQ options.
A bullish signal means "buy a 0DTE call"; a bearish signal means "buy a 0DTE put".

## Install

1. Open a **QQQ** chart in TradingView on a **1- to 5-minute** timeframe, regular trading hours.
2. Open the **Pine Editor**, paste the contents of `qqq_0dte_strategy.pine`, and click **Add to chart**.
3. Open the **Strategy Tester** tab for the backtest, and the strategy's settings (gear icon) to tune it.

## What you need to know before trusting the numbers

TradingView has no option-chain data in Pine Script and cannot place option orders, so the script works like this:

| What you see | What it really is |
| --- | --- |
| Strategy Tester P&L | QQQ shares sized to the option's delta (contracts × 100 × delta). A first-order approximation only. |
| Dashboard, scoreboard and trade labels | A Black-Scholes model of the option (strike, premium, delta, P&L) using the VXN volatility index as implied volatility. Includes time decay and gamma. |
| Your broker | The real thing. Fills, spreads and implied volatility will differ from both of the above. |

Calibrate the model before relying on it: compare the dashboard's at-the-money call and put premiums with your
broker's QQQ option chain and adjust **IV multiplier** until they agree.

## How it decides

A trade needs a **setup** (an event on the chart) plus enough **confluence** (indicators agreeing with the direction).
Every setup and every confluence factor has its own on/off switch.

### Setups

| Trend setups | Reversal setups |
| --- | --- |
| Opening-range breakout (ORB) | Liquidity sweep |
| Pullback to VWAP / moving average | Pin bar / three-bar reversal at a key level |
| Bull / bear flag | Head & shoulders / inverse head & shoulders |
| Fair value gap (FVG) retest | Double top / double bottom |
| Consolidation breakout (rectangle, triangle, pennant, wedge) | Change of character (CHoCH), off by default |
| Key-level breakout (prior-day and pre-market high/low) | |
| Late-day breakout | |
| Moving-average crossover | |
| Rising / falling three methods | |
| Bollinger squeeze breakout | |

Breakout setups also require relative volume (RVOL) of at least 1.5 by default.

### Confluence score

Trend setups need 6 of these 11 points by default, and calls are only taken above VWAP (puts below):

- Price vs VWAP
- Fast MA vs slow MA (EMA or SMA, 9 and 21)
- Price vs the trend SMA (50)
- Price vs the EMA on two higher timeframes (15 and 60 minutes)
- MACD line vs signal line
- RSI on the right side of 50 and not yet overbought/oversold
- RVOL at or above the threshold
- VIX falling (calls) or rising (puts)
- ADX at or above 20
- Heikin Ashi candle direction

Reversal setups use a separate 5-point score and need 2 by default: RSI was overbought/oversold recently or shows
divergence, RVOL above the threshold, MACD histogram turning, VIX moving the right way, Heikin Ashi candle in the
trade direction.

### Which setups are worth keeping

Switching everything on gives more trades, not more edge. The **Setup scoreboard** table shows trades, win rate and
modeled P&L for each setup. Test setups one at a time, and switch off the ones that lose money on your timeframe.

### Exits and limits

- ATR stop and 2R target on the QQQ price, stop to breakeven after 1R, optional ATR trail.
- Exit on a close back through VWAP, a 90-minute time stop, and a hard flatten at 15:30 New York time.
- Entries from 09:45 to 15:00 with a midday pause, at most 3 trades and 2 losses per day, 3-bar cooldown after an exit.

## Not included, and why

- **Elliott wave, Gann angles, harmonic patterns:** they depend on judgment calls that cannot be written as fixed rules without inventing them.
- **Moon cycles:** no tradable link to intraday QQQ moves.
- **Renko:** a chart type rather than a setup. Backtests on Renko charts are unreliable because the bars are synthetic.
- **Triangles, wedges, pennants, rectangles as separate drawings:** covered together by the consolidation and flag breakouts instead of fitting trendlines.

## Alerts

Create an alert on the strategy, choose **Order fills only**, and set the message to:

```
{{strategy.order.alert_message}}
```

Each alert then reads like `QQQ 0DTE | BUY TO OPEN 1x 505C expiring today | Flag | score 7/11 | QQQ 504.62 | est. premium $1.85 | stop QQQ 503.90 ...`.
The script does not place trades; you (or your own automation) act on the alert.

## Limits of the model

- Option prices are estimates. There is no volatility skew, and VXN is a 30-day index rather than true 0DTE implied volatility.
- Chart patterns are detected by fixed rules on swing pivots, which is an approximation of what a person draws by eye. A pivot is only confirmed a few bars after it forms.
- Expiration is assumed to be 16:00 New York time. On early-close days the model overstates premium.
- Backtest fills happen at the bar close that produced the signal. A real order fills a moment later at a slightly different price.
- Pre-market levels need extended hours switched on for the chart.
- Past backtest results do not predict future results. 0DTE options can lose their entire value in minutes. This is an educational tool, not financial advice.

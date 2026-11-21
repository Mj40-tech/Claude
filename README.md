# TradingView scripts

| Script | What it is |
| --- | --- |
| `qqq_0dte_strategy.pine` | QQQ 0DTE options day trader. Documented first, below. |
| `ny_4am_sweep_ifvg_strategy.pine` | 4AM range sweep into a 15-minute PD array with an IFVG entry. Documented in the [second half](#4am-range-sweep--15m-pd-array--ifvg) of this page. |

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

# 4AM Range Sweep + 15m PD Array + IFVG

`ny_4am_sweep_ifvg_strategy.pine` automates this checklist (all times New York):

| Checklist step | What the script does |
| --- | --- |
| Wake up at 03:55 | Sends a wake-up alert at 03:55. |
| Mark the nearest 15-minute PD array | Draws the nearest bearish 15m array above price and the nearest bullish 15m array below price. |
| Wait for 04:00, mark the 03:00-04:00 high and low | Draws the range box and extends its high and low as dashed lines. |
| Wait for a sweep of the high or low into a marked array | Labels the sweep, then labels the moment price trades into a marked array. |
| Enter after the first 30-second IFVG | Enters at the close of the first bar that inverts a fair value gap on the chart timeframe. |
| Target opposing liquidity, max 1:3 RR | Stop beyond the sweep extreme. Target at the other side of the range, moved in to 3R if the range is further away. |

## Install

1. Open a chart of a market that trades overnight (futures such as NQ or ES, forex, gold, crypto) and set it to **30 seconds**.
2. Open the **Pine Editor**, paste the contents of `ny_4am_sweep_ifvg_strategy.pine`, and click **Add to chart**.
3. The **Strategy Tester** tab shows the backtest. The status table on the chart shows which step of the checklist today is on.

Second-based charts need a TradingView plan that includes them. On a 1-minute chart the script still runs, and the
entry IFVG is then a 1-minute one. US stocks and ETFs have no bars between 03:00 and 04:00, so the script has nothing to
work with there.

## How each step is defined

- **15m PD array:** a fair value gap, an order block or an inverse fair value gap on the 15-minute chart that no
  15-minute candle has closed through. Each type has its own switch. Only arrays from closed 15m candles are used, so
  a marked array never changes after the fact.
- **Nearest:** at 03:55 the script takes the closest bearish array above price and the closest bullish array below
  price. With **Skip arrays that sit inside the range** on (the default), an array has to reach beyond the range so
  far, because a sweep of the range cannot end inside an array that is inside the range.
- **Sweep:** after 04:00 a bar trades beyond the range high or low.
- **Into the array:** from the sweep bar on, a bar overlaps a marked array that is still valid. A marked array stops
  being valid when a 15m candle closes through it.
- **IFVG:** for a short, a bar closes below a bullish fair value gap on the chart timeframe. For a long, a bar closes
  above a bearish one. The gap must have formed within the last 60 bars.
- **One trade a day.** The first side to complete every step is the trade.

## Choices the checklist leaves open

These are the script's defaults. Each one is a setting.

| Question | Default |
| --- | --- |
| When does the script stop looking for an entry? | 07:00 New York |
| Where is the stop? | One tick beyond the highest (short) or lowest (long) price of the sweep |
| What if the other side of the range is very close? | The trade is skipped when the target is less than 1R away |
| How many arrays are marked? | One above price and one below |
| Is an open trade closed at a set time? | No. It stays open until the stop or the target is hit |
| Position size | 1 contract or unit. Risk-based sizing is available |

## Alerts

Create an alert on the strategy, choose **Order fills and alert() function calls**, and set the message to:

```
{{strategy.order.alert_message}}
```

You then get the 03:55 wake-up with the marked arrays, the 04:00 range, the sweep into an array, and the entry and exit
with prices. The script runs when a bar closes, so each alert arrives at the close of the bar that caused it.

## Limits

- The script was checked by review, not by TradingView's compiler, which was not reachable where it was written. If the
  Pine Editor reports an error when you add it to the chart, the error message and line number are enough to fix it.
- A 30-second chart loads about a week of history on most plans, so the backtest covers only a handful of trades.
  That is too few to judge the model. Use bar replay to study more days.
- Order blocks have no single agreed definition. The script uses the last opposite candle before a candle that closes
  beyond it. Switch order blocks off if your definition differs.
- Backtest entries fill at the close of the bar that inverts the gap. A live order fills a moment later at a slightly
  different price.
- Past backtest results do not predict future results. This is an educational tool, not financial advice.

# QQQ 0DTE Options Day Trader (TradingView Pine Script v6)

`qqq_0dte_strategy.pine` is an intraday strategy for buying same-day-expiry (0DTE) QQQ options.
A bullish setup means "buy a 0DTE call"; a bearish setup means "buy a 0DTE put".

## Install

1. Open a **QQQ** chart in TradingView on a **1- to 5-minute** timeframe, regular trading hours.
2. Open the **Pine Editor**, paste the contents of `qqq_0dte_strategy.pine`, and click **Add to chart**.
3. Open the **Strategy Tester** tab for the backtest, and the strategy's settings (gear icon) to tune it.

## What you need to know before trusting the numbers

TradingView has no option-chain data in Pine Script and cannot place option orders, so the script works like this:

| What you see | What it really is |
| --- | --- |
| Strategy Tester P&L | QQQ shares sized to the option's delta (contracts × 100 × delta). A first-order approximation only. |
| Dashboard and trade labels | A Black-Scholes model of the option (strike, premium, delta, P&L) using the VXN volatility index as implied volatility. Includes time decay and gamma. |
| Your broker | The real thing. Fills, spreads and implied volatility will differ from both of the above. |

Calibrate the model before relying on it: compare the dashboard's at-the-money call and put premiums with your
broker's QQQ option chain and adjust **IV multiplier** until they agree.

## How it trades

- **Bias:** price above the 09:30-anchored VWAP, fast EMA above slow EMA, and price above the 15-minute EMA favours calls. The mirror image favours puts.
- **Entries:** an opening-range breakout (first 15 minutes, with a volume check), or a trend pullback that reclaims the fast EMA.
- **Exits:** ATR stop and 2R target on the QQQ price, stop to breakeven after 1R, optional ATR trail, exit on a close through VWAP, a 90-minute time stop, and a hard flatten at 15:30 New York time.
- **Limits:** entries from 09:45 to 14:45 with a midday pause, at most 3 trades and 2 losses per day, 3-bar cooldown after an exit.

Everything above is an input you can change.

## Alerts

Create an alert on the strategy, choose **Order fills only**, and set the message to:

```
{{strategy.order.alert_message}}
```

Each alert then reads like `QQQ 0DTE | BUY TO OPEN 1x 505C expiring today | ORB | QQQ 504.62 | est. premium $1.85 | stop QQQ 503.90 ...`.
The script does not place trades; you (or your own automation) act on the alert.

## Limits of the model

- Option prices are estimates. There is no volatility skew, and VXN is a 30-day index rather than true 0DTE implied volatility.
- Expiration is assumed to be 16:00 New York time. On early-close days the model overstates premium.
- Backtest fills happen at the bar close that produced the signal. A real order fills a moment later at a slightly different price.
- Past backtest results do not predict future results. 0DTE options can lose their entire value in minutes. This is an educational tool, not financial advice.

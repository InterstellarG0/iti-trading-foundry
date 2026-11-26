# Trading Foundry — Agent methodology and risk veto
Date: 2026-11-26
Status: Research / proposed paper-test rules; **not implemented as automated entry gates**.

## Mental model
The agent is a deterministic decision process, not a predictor. Observe market and account data; classify regime; evaluate a candidate signal; authorize risk; execute a simulated order; monitor the position; record outcomes and compare strategy versions. A working bot is not proof of a profitable strategy.

**Signals suggest; risk decides.** A valid setup may be rejected. After rejection the scanner continues without forcing an entry. The administrator can review logs and approve separately tested changes.

## Candidate long setup — to backtest
1. Use completed candles and record the timeframe.
2. RSI was below 30, turned upward and crossed back above 30.
3. Price has stopped making lower lows and reclaimed a defined level; RSI rising while price makes lower lows is not sufficient.
4. Volume supports the recovery; a volume spike alone may reflect heavy selling.
5. All gates must be evaluated on historical and out-of-sample data, including failed signals. Avoid assuming RSI, stochastic and Stoch RSI are independent confirmations.

## Portfolio authorization
Before entering, evaluate:
- Existing positions and correlated BTC/ETH/SOL/ADA/XRP exposure; diversification by coin count alone is misleading.
- Total basket exposure cap, position risk based on stop distance, drawdown and session loss rules.
- Fees, spread, liquidity, slippage, order size and possibility of stop gaps.
- Whether the proposed trade would violate any cap. If yes, **BLOCK AND KEEP SCANNING**.
- Log reason codes: `NO_PRICE_CONFIRMATION`, `RSI_NOT_CONFIRMED`, `VOLUME_AMBIGUOUS`, `CORRELATED_EXPOSURE`, `RISK_LIMIT`, `COSTS_TOO_HIGH`, `DATA_STALE`.

## Evaluation protocol
Record baseline version, candidate version, date range, timeframe, regime, sample size, average win/loss, net expectancy after fees/slippage, maximum drawdown, profit factor, rejected signals, fill quality and out-of-sample performance. Compare one change at a time. Roll back if evidence degrades. Paper fills are simulations and cannot guarantee real fills or profit.

## Architecture / separation of duties
- Market-data adapter: validate candles, prices, timestamps and freshness.
- Regime classifier: trend, volatility, basket co-movement.
- Signal scanner: propose candidate setups.
- Portfolio risk manager: veto based on aggregate risk and permissions.
- Execution simulator: model fees, partial fills and price impact.
- Position manager: monitor invalidation and exits.
- Journal and loop tuner: keep an auditable log and compare versions.
- Human administrator: review decisions and approve changes; no automatic code deployment or real-money execution.

## Existing implementation versus proposed
The existing public HTML Foundry page has scanners, simulated fills, a basket cap and challenge-style loss controls. This note does **not** claim that RSI-30-cross, correlated-exposure veto, reason codes, or out-of-sample analysis have been wired into the running bot. They are next implementation and test tasks.

## Test cases
- RSI crosses above 30, price makes lower lows -> WAIT.
- RSI recovery + price reclaim + volume support, but ETH/SOL positions breach aggregate correlated risk -> BLOCK and keep scanning.
- Three consecutive rejected entries -> no forced trade; log each rejection.
- Incomplete or stale candle -> do not authorize.
- Signal qualifies but expected trading costs erase estimated edge -> reject.

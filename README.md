# Order Book Imbalance as a Short-Horizon Price Predictor

A market-microstructure research project studying whether **order book imbalance** predicts
the direction of the next price move in crypto markets — and, critically, whether any such
edge survives the bid-ask spread.

> This project is deliberately scoped to test a single, falsifiable hypothesis honestly.
> The goal is not a flashy backtest. The goal is to show research discipline: a clear
> question, a clean methodology, and an honest report of what the data actually says —
> including if the edge is zero after costs.

## Hypothesis

**H1:** At the top of the book, when bid-side resting volume substantially exceeds ask-side
resting volume (positive order book imbalance), the mid-price is more likely to tick *up*
over the next short horizon (e.g. 1–10 seconds) than down. The reverse holds for negative
imbalance.

**H0 (null):** Order book imbalance carries no information about the sign of the next
mid-price change.

**The real test — H2:** Even if H1 holds, the predicted moves are smaller than the
bid-ask spread, so a taker strategy that crosses the spread to act on the signal earns
**zero or negative** expected value after costs. Demonstrating *whether the signal is
tradable* is the point of the project, not just whether it is statistically present.

### Why this question is worth asking

- It is the canonical example of **adverse selection** and **order flow** — the exact
  concepts a market-making / trading desk cares about.
- It forces an honest reckoning with **transaction costs**, which is where most naive
  "alpha" projects quietly die.
- The answer is genuinely uncertain and the result is interesting either way.

### Definitions

Order book imbalance over the top *N* levels:

```
imbalance = (bid_volume - ask_volume) / (bid_volume + ask_volume)   ∈ [-1, +1]
```

where `bid_volume` / `ask_volume` are the summed resting sizes across the best *N* levels.

Mid price: `mid = (best_bid + best_ask) / 2`.
Spread (in bps): `10_000 * (best_ask - best_bid) / mid`.

## Why crypto

Crypto is the only liquid asset class offering **free, tick-level order book data** through
public exchange REST/WebSocket APIs (here: Binance). Equity microstructure data of this
granularity is expensive and licensed. Collecting the data myself — rather than downloading
a pre-cleaned dataset — is part of the project.

## Methodology (planned)

1. **Collect** — snapshot the live order book at a fixed cadence and log imbalance, spread,
   and mid price to disk. *(implemented — `src/collector.py`)*
2. **Label** — for each snapshot, compute the forward mid-price return at several horizons.
3. **Measure signal** — relationship between imbalance and the sign/magnitude of the
   forward return (binned hit-rates, regression, information coefficient).
4. **Apply costs** — net the predicted moves against the spread and fees to test H2.
5. **Report honestly** — including null results, lookahead checks, and limitations.

## Repository structure

```
trading/
├── README.md            # this file — the hypothesis and plan
├── requirements.txt     # dependencies
├── .gitignore
├── src/
│   └── collector.py     # order book snapshot collector (step one)
└── data/                # captured snapshots (gitignored)
```

## Status

- [x] Step 1 — Hypothesis defined; order book collector built.
- [ ] Step 2 — Capture a multi-hour dataset.
- [ ] Step 3 — Signal analysis.
- [ ] Step 4 — Cost-adjusted EV test.
- [ ] Step 5 — Write-up of results.

## Usage

```bash
pip install -r requirements.txt
python src/collector.py --symbol BTCUSDT --levels 20 --interval 1 --duration 600
```

Writes timestamped snapshots to `data/orderbook_<symbol>_<date>.csv`.

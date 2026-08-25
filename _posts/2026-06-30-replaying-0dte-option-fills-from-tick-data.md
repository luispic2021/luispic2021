---
layout: post
title: "When My Backtest Changed Its Mind"
date: 2026-06-30
categories: backtesting market-data python
description: Replaying ordered option trades reversed an optimistic 0DTE backtest and exposed why candle fills fail for path-dependent exits in live comparisons.
---

My candle-based backtest liked a 0DTE options strategy. The live system did not.

On matched trades, the gap was too large to explain away as ordinary slippage. The backtest had turned a losing period into a winning one because it was answering two questions that minute candles cannot answer reliably:

1. Which exit condition happened first inside the bar?
2. What price could the order actually receive?

For a path-dependent exit such as a trailing stop, those are not details. They are the result.

## A candle has no sequence

Suppose one minute has both a low below the stop and a high above the next trailing level. OHLC records both extremes but not their order.

My engine resolved that ambiguity too favorably. It could register a new high, ratchet the trail, and book a profitable exit even when the live path had touched the stop first.

The candle model also filled at theoretical levels without observing the market’s actual prints. The combined bias was directional, not random, so optimizing trail width against it meant optimizing against a broken measuring instrument.

## Replay the tape, in order

I rebuilt the exit simulation around timestamped option trades. For each historical position, the engine now:

1. loads the contract’s prints during the holding window
2. sorts them in timestamp order
3. updates the running high on every print
4. applies stop, target, trail, and time rules in sequence
5. lets the first satisfied condition win

The candle engine remains available as a lower-fidelity baseline. Tick replay is opt-in so I can compare the two models on the same entries.

The result changed direction. The optimistic candle run became a losing tick-replay run and moved much closer to the observed live behavior. That did not prove the live strategy was sound. It showed that the previous backtest had overstated it.

## Trades delivered more value than missing quotes

I expected NBBO quotes to be essential for realistic spread modeling. The data plan available to this experiment included trade prints but not the historical quote endpoint.

That limitation forced a useful test: were ordered prints enough to remove the dominant bias?

On the matched panel, they were. The residual difference between tick replay and live fills was small relative to the correction from intrabar ordering. Prints also contain some realized spread information: the trade that triggers a sell stop often occurs near the bid rather than at a frictionless theoretical level.

This is a conclusion about this sample, not a universal claim that quotes do not matter.

## I deleted the correction that looked more realistic

I briefly tested a synthetic spread calibrated from a limited month of quote captures. Applying a flat half-spread made one comparison look better.

It was also the wrong model:

- trade prints already embedded some spread, so the adjustment double-counted it
- a flat value ignored volatility, time of day, liquidity, and time to expiry
- the calibration could not be validated in periods without quotes

A visible limitation was preferable to an untestable correction tuned to one window. Real quotes can support a quote-based fill mode later; a constant chosen because it improves a chart should not quietly become historical truth.

## Instrument the instrument

Tick data has its own failure mode: sparse prints. Each replayed trade now records the number of prints in its actual holding window and a simple fill-quality status:

```text
ok | sparse | no_trades
```

That flag does not repair thin data. It prevents the engine from presenting a low-confidence path as equally trustworthy.

The review caught two important implementation errors before the change landed: a timestamp fallback that could misread epoch units, and a quality metric that counted the broader scan window instead of the position’s hold window. The full test suite now covers both.

The strategy still has to earn confidence through new evidence. But the backtest now distinguishes what it observed, what it inferred, and where the tape was too thin to know.

*This is an educational backtesting case study, not investment advice. Tick replay and historical results do not predict future profitability.*

---
layout: post
title: "The Trail Wasn’t the Only Problem"
date: 2026-06-29
categories: automated-trading backtesting market-data
description: A 0DTE trailing-stop experiment became a lesson in small samples, path-dependent exits, and validating the tools used to validate a strategy.
---

I started with a narrow question: was my trailing stop too tight for a fast-moving 0DTE option?

In live runs, the trail sometimes triggered faster than my polling loop could explain. So I recorded the option’s tick-by-tick path and measured pullbacks that recovered before the next high.

The first result was useful: nearly half of the recovered pullbacks in the sample were deeper than the live trail. The stop was sitting inside ordinary movement for those trades.

But “make it wider” was not a complete answer.

## Typical trades and rare runners wanted different things

When I replayed several trail widths, the median trade favored a tighter exit. Total return favored a wider one because a few large moves dominated the aggregate.

That is a product decision disguised as a parameter choice:

- optimize the common trade and cut give-back
- or tolerate more give-back to remain exposed to rare runners

A single fixed trail cannot maximize both objectives. Before changing live behavior, I needed to know whether I could identify the regime early enough.

## The smarter gates arrived too late

I tested several prototypes: option-path confirmation, entry-time volatility and trend filters, and early follow-through in the underlying.

None improved on a simple fixed trail in this small sample.

The entry-time features did not separate the regimes. The feature that did show promise—whether the underlying continued in the trade’s direction—resolved too slowly. By the time the signal became informative, the option had often completed the move I wanted the trail to manage.

The underlying was explaining the regime on a minutes-long clock. The option was repricing on a seconds-long clock.

That mismatch is easy to miss when every observation ends up in the same spreadsheet row.

## Then the validators disagreed

I compared matched trades across three sources:

| Source | What it represented | Directional result |
|---|---|---|
| Candle backtest | Simulated historical exits | Strongly positive |
| Paper experiment | Forward test with simulated fills | Negative |
| Live behavior | Actual broker orders and fills | Negative |

The gap was too large to call slippage. The backtest and the live system were measuring different paths.

My initial hypothesis focused on spread and fill assumptions. That was plausible, but not yet proved. Minute candles also hid the order of events inside each bar, which is decisive for a trailing stop: did the option hit the stop first, or make a new high first?

I left the live parameter unchanged and treated the wider trail as a paper experiment. More importantly, I stopped treating the candle backtest’s absolute result as evidence until I could replay the actual sequence of prints.

That next test changed the diagnosis. [The follow-up rebuilds fills from ordered tick data]({% post_url 2026-06-30-replaying-0dte-option-fills-from-tick-data %}) and separates the largest source of bias from the costs I had only inferred here.

## What I kept

Three lessons survived the revision:

1. Measure the path, not only the outcome. Path-dependent exits need path-level data.
2. Do not out-clever a small sample. A more adaptive rule is not automatically a better one.
3. Validate the validator. Backtest, paper, and live results are different evidence and should stay labeled that way.

This was one month and a few dozen matched trades. The findings were provisional—but provisional and measured was still better than confident and guessed.

*This experiment is educational and is not investment advice. Backtests and paper results do not establish future profitability.*

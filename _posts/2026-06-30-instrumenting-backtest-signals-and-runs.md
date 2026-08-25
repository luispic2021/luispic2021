---
layout: post
title: "Teaching My Backtester Which Trades Not to Take"
date: 2026-06-30
categories: backtesting automated-trading python
description: I added market context to every signal and deterministic run identities so I could test trade filters without changing strategy behavior at all.
---

Aggregate backtest results tell me whether a strategy deserves another question. They rarely tell me which question to ask.

While reviewing losing trades in a 0DTE divergence strategy, two patterns kept bothering me. Some entries appeared to fight a nearby moving average. Others fired when the stochastic oscillator had only just qualified instead of reaching a deeper extreme.

Those were hypotheses, not filters. I wanted to test them without changing the entry logic.

## Add context before adding rules

The signal detector already knew which bar qualified. I extended its output with the market state at that moment:

- the oscillator value at the signal
- signed distance from price to several moving averages
- direction, so “near resistance” and “near support” are analyzed separately

Each value is calculated once and written to the signals CSV. The strategy takes exactly the same trades as before.

An analysis script then joins signals to completed trades and reports linear and rank correlations alongside bucketed win rates. The first pass suggested that signals deeper in the oscillator’s extreme zone performed differently from those that barely qualified.

That is not enough evidence to promote a rule. It is enough to define the next experiment: test a threshold out of sample and compare the filtered variant against the unchanged baseline.

The distinction matters. Instrumentation produced a hypothesis; it did not prove an edge.

## The bigger failure was run bookkeeping

While adding the new columns, I ran into a more basic trust problem. I had accumulated manual variant labels, handwritten commands, and output files whose configuration was not recoverable from the filename.

Worse, a forgotten label could let two configurations share a checkpoint. A restarted backtest might resume from signals generated under different parameters without saying so.

I replaced the label with a deterministic identity derived from the full configuration:

```text
canonical configuration
  → SHA-256 fingerprint
  → short collision-aware run ID
  → indexed outputs and checkpoints
```

The same configuration receives the same ID. Change anything that can affect the result and the ID changes. Each invocation is recorded with its configuration, output summary, and reproducible command.

This removed a human step and made accidental checkpoint reuse much harder.

## Cold review found the quiet bugs

An independent review of the diff found three gaps I had missed:

1. The list of hashed parameters could drift from the function signature.
2. An exit path changed candle resolution based on which file existed, but that input was missing from the fingerprint.
3. Resuming an older checkpoint could mix valid new feature values with missing ones.

None changed the strategy logic. All could change what I believed about a result.

I added guard tests for parameter parity, included hidden data dependencies in the fingerprint, and made incompatible checkpoints fail visibly.

I also simplified the run index. Instead of expanding every strategy parameter into a growing set of CSV columns, each row stores one canonical JSON configuration. The index stays stable while individual strategies evolve.

## Trust is a backtest feature

This work did not improve the reported performance of the strategy. It improved the chain of evidence:

- the signal row preserves the context I want to study
- the trade row records the outcome
- the run index records how both were produced
- the identifier prevents unlike runs from sharing state

That is less glamorous than a new entry rule. It is also what makes the next rule testable.

*This is an educational account of backtest tooling, not investment advice. Historical results do not establish future performance.*

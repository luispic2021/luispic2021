---
layout: post
title: "A Backtest Needs Two IDs"
date: 2026-07-07
categories: backtesting data-modeling python
description: Separating strategy variants from evaluation runs made my backtests easier to reproduce, compare across windows, and migrate safely over time.
---

When a strategy has been backtested hundreds of times, “which run was that?” stops being an administrative question. It becomes part of the evidence.

My backtester already assigned a deterministic ID to every full configuration. Same inputs, same ID; change an input, get a different ID. Output files and crash-resume checkpoints were keyed to it.

That solved accidental overwrites. It also hid a modeling mistake: I was treating a strategy and one evaluation of that strategy as the same object.

## The window changed the identity of the edge

The fingerprint included everything, including the start and end dates. When I reran a promoted configuration on newer data, it received an unrelated ID. There was no durable link saying, “this is the same strategy definition measured in a different period.”

The comparison I cared about most was the one the data model could not express.

So I split one identity into two:

- A **variant** is the strategy definition: entry logic and exit configuration.
- A **run** is one evaluation of that variant, including its data window, account-sizing assumption, and fill model.

Re-evaluating the same edge on a new period keeps the variant ID and receives a new run ID.

## Partition the configuration once

The implementation starts with one canonical payload. Context keys are excluded only when producing the variant fingerprint:

```python
_CONTEXT_KEYS = frozenset({
    "file_path",
    "account_size",
    "start_date",
    "end_date",
    "tick_replay",
    "fill_pricing_mode",
    "exit_candles_path",
})


def variant_payload(payload: dict) -> dict:
    return {key: value for key, value in payload.items()
            if key not in _CONTEXT_KEYS}
```

Both IDs come from the same serialization and fingerprinting path. That is important: two parallel implementations would eventually disagree about a default or nested field.

Fill modeling belongs to run context. Switching from candles to tick replay can materially change measured results, but it does not create a new trading idea. It creates a more faithful evaluation of the same idea.

## Preserve old run IDs during migration

Historical filenames and checkpoints already depended on the original full-configuration hash. I kept that hash byte-for-byte as the run fingerprint and added only the variant fingerprint.

The migration reconstructs every historical payload through the same production functions used for a new run, then backfills the variant columns. Afterward, a simple filter returns every evaluation of one edge across dates and fill assumptions.

The migration also exposed how much apparent experimentation was repeated measurement. Several historical runs collapsed into fewer distinct variants. That is useful information in itself.

## Small abstraction, quiet bugs

An independent review found that parameters for one strategy were being threaded into the fingerprint of unrelated strategies. Those unused values could fragment a variant even though they had no effect on behavior.

It also found a schema hazard: appending new fields to an unmigrated CSV could place values under the wrong headers. Both cases now fail loudly, and guard tests keep the migration’s defaults aligned with the live configuration.

The design lesson is broader than backtesting. When an object is evaluated repeatedly, the thing being evaluated and the conditions of one evaluation need separate identities.

Without that split, reproducibility tells you how to recreate a file. With it, you can also ask whether the same idea held up somewhere else.

*This post covers backtest system design for educational purposes, not investment advice. Historical evaluations do not guarantee future results.*

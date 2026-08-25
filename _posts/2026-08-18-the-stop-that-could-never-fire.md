---
layout: post
title: "The Stop That Could Never Fire"
date: 2026-08-18
categories: automated-trading incident-review reliability
description: Three correct components still abandoned a late-session option position because its exit deadline landed after the trading loop ended for the day.
---

My 0DTE bot opened a position late enough that its configured time stop landed after the market close.

At the bell, the main loop did exactly what it was designed to do: it ended the session. The resting profit-taking order remained open, the time stop never had a chance to fire, and the option expired worthless.

No single component was obviously broken.

## The bug lived between correct components

The time stop was calculated correctly from the fill. The session loop stopped correctly at the market close. The exit order was placed correctly and waited for a price the market never reached.

The failure existed in the composition:

- the deadline code did not know when the process would stop listening
- the session-close code did not know a position still needed management
- the summary counted only journaled closes, so it omitted the abandoned trade

This is the kind of incident unit tests can miss when each function’s local contract is correct. The system-level contract—never end while a position may still be open—was not encoded.

## Reuse the safety behavior already present

The frustrating part was that the bot already knew how to handle this condition.

Its emergency-halt path checked for an active position, attempted to flatten it, refused to end until the close was confirmed, and raised an urgent alert when confirmation failed. That reviewed behavior sat near an unconditional session-close branch that simply called `break`.

The fix reused the existing close primitive instead of adding another order path. Session end and emergency halt now share the same invariant: do not declare the session finished while the broker may still hold the position.

## Derive the entry cutoff

A fixed “no new trades after this clock time” rule would solve only the current configuration. Different hold periods need different cutoffs, and shortened sessions move the close.

The entry gate is derived instead:

```text
session_exit_deadline = market_close - operational_buffer

accept entry only if:
expected_fill_time + configured_hold_period <= session_exit_deadline
```

The gate uses the expected fill, not the signal timestamp. A completed signal bar can lag wall-clock time, and a submitted order can take time to confirm. The exit timer begins from the fill, so the admission decision must reason about that same clock.

The calculation also uses the session calendar, which means the cutoff moves automatically on early-close days.

## Prevention and containment are different

The entry gate prevents this specific late trade. It does not cover every way a position can become stranded.

A separate authentication incident had already produced the same dangerous state through a different path. That changed the scope of the fix:

- the derived gate prevents an exit deadline beyond the session
- an end-of-session sweep contains any open position, regardless of cause
- failure to confirm the sweep keeps the recovery path alive and the alerts active

The clever rule handles one cause. The boring sweep handles the category.

An independent review found that my first retry still had a hole: it returned early while the time stop was in the future, which was precisely the original scenario. A second review corrected my explanation of broker behavior and exposed the real issue—the fallback close could not proceed until the resting exit was cancelled.

The check I am keeping is simple: whenever code computes a deadline, ask whether the program will still be alive to observe it.

Timers assume someone is around to hear them. Mine was not.

*This incident review is educational and is not investment advice.*

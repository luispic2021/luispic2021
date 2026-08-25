---
layout: post
title: "The Bot That Walked Away"
date: 2026-08-17
categories: automated-trading incident-review reliability
description: A filled order went unmanaged after authentication failed. The fix was to treat unknown order state as risk, recover it safely, and keep watching.
---

My 0DTE bot submitted an entry, spent about a minute failing to confirm the fill, logged `Aborting`, and went back to scanning for signals.

The order had filled. The bot was no longer managing the position. No time-based exit, no profit target, no recovery loop. The option later expired worthless.

My first theory was wrong in a useful way.

## The retry existed, but the error never reached it

I suspected that entry confirmation was missing the authentication recovery used elsewhere in the bot.

The polling loop did call the retry wrapper. One layer below it, however, the broker method caught every exception and returned a polite `success=False` object. The underlying `401 Unauthorized` never propagated to the code designed to refresh credentials.

The retry wrapper was not bypassed. It was starved.

The polling loop then discarded the error detail it did receive. The logs repeated “failed to fetch order” while the actual status code sat unused in the response object.

This is the danger of broad exception handling in a lower layer: it can make a function look resilient while disabling the recovery policy above it.

## Shared authentication made failures routine

Multiple bot processes shared token state. When one process refreshed, another could keep an older access token in memory while its local expiry timestamp still looked valid.

The next broker call would fail, trigger another refresh, and rotate the token again. The logs showed a cascade across processes.

Instead of assuming those bursts could be eliminated immediately, I changed the system to survive them.

## Unknown is a position state

The entry path now distinguishes three outcomes:

- **filled** — adopt and manage the position
- **dead** — confirmed cancelled or rejected with no fill
- **unknown** — neither state is proved

Unknown no longer means abort. The bot recovers authentication, checks the order again, cancels any remaining open quantity, and queries the broker’s actual positions.

If the contract is held, the bot adopts the broker quantity and begins normal exit management. If state is still ambiguous, it keeps reconciling and alerting through the session instead of returning to signal scanning.

Authentication recovery also moved to the broker layer, where every request passes through one policy. On a `401`, a process first adopts the newer token already written by a sibling. It creates a new token only if adoption fails.

## Review the fix in a different mental model

The sharpest review finding was a partial fill. My first “dead order” branch treated a cancelled order as flat without checking whether some quantity had filled before cancellation.

The review also caught two related mistakes:

- adopted size came from configuration instead of the position actually held
- stacked retry layers multiplied the refresh behavior I was trying to contain

Those bugs were inside the recovery code because I was still thinking in the all-filled-or-not-filled shape of the original incident.

As an independent tripwire, the bot now periodically compares its tracked positions with the broker’s positions and alerts on drift. It is intentionally alert-only. Reconciliation detects disagreement; it does not improvise a trade.

## The operational lesson

An urgent notification is not risk management. I received the original alert and read it too late. Anything the system must do—such as continue managing a possibly live position—belongs in the system’s recovery behavior, not in a request for a human to notice.

The absence of fill confirmation is not evidence that no fill occurred. When the order state is unclear, ask the broker what the account actually owns and keep watching until the ambiguity is resolved.

*This incident review is educational and is not investment advice.*

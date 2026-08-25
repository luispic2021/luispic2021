---
layout: post
title: "A Kill Switch That Doesn’t Abandon the Trade"
date: 2026-07-16
categories: automated-trading reliability aws
description: How a local halt file became a scoped S3 policy system that can flatten an autonomous trading bot without opening an inbound service to the bot.
---

An autonomous trading bot is convenient until I decide, mid-session and away from my computer, that it should stop.

The obvious controls are unsafe or ineffective. Changing an environment variable does not mutate the environment of a running process. Killing the process can orphan an open position with no code left to manage its exit.

The requirement was sharper: a running bot needed to observe a remote instruction, stop taking risk, and never abandon a position.

## Poll a control the process can see

The bot already has a loop, so each iteration can read external state. The first version used a local sentinel file:

```python
def is_trading_halted(*scopes):
    if halt_file_exists():
        return True
    return any(scoped_halt_exists(scope) for scope in scopes if scope)
```

The local file has one excellent property: it works without a network. It is the offline emergency brake. Its weakness is reachability—I still need shell access to create it.

For remote control, I added small JSON policy documents in S3. The bot makes outbound reads; my phone can update the document through a separately permissioned path. The trading machine does not expose a new web server or inbound port.

## A halt is one field in a policy

A boolean would solve today’s problem and force a new format for tomorrow’s. I used a document whose fields are optional:

```json
{
  "halt": true,
  "direction_filter": null
}
```

The current safety action is `halt`. Other fields can constrain future entries, but no remote field is allowed to increase configured risk.

Policies can exist at global, strategy, and instance scopes:

```text
control/policy.json
control/policy.<strategy>.json
control/policy.<instance-label>.json
```

Ordinary fields merge from broad to specific. `halt` is different: it is OR-ed across every layer. A narrower document cannot undo a global halt. To resume, every active halt must be cleared.

That asymmetry is deliberate. Safety-critical state should ratchet toward less activity, not be relaxed accidentally by a more specific override.

## Remote input is untrusted input

A policy edited from a phone is easy to mistype. The resolver therefore:

- accepts only known fields
- validates values before constructing the policy
- ignores an invalid document rather than applying half of it
- preserves the bot’s configured risk limits
- catches its own failures so it cannot crash the trading loop

Remote reads are cached for a short interval. The local file is checked on every iteration so the offline brake remains immediate.

Fetch failure and “no policy exists” are separate states. Repeated remote failures raise an alert because the control path may be unavailable, but they do not invent a new policy.

## Halt means flatten, then confirm

My first version stopped new entries and allowed an open trade to reach its normal exit. That is a pause button, not an emergency stop.

The current behavior attempts to cancel the resting exit and market-close the position. The session ends only after the broker confirms the account is flat.

If the close cannot be confirmed, the bot does not simply exit. It continues managing the existing order path and raises urgent alerts. “I sent a close request” and “the position is closed” are different states.

Other policy changes affect future entries only. A direction filter should not mutate the terms of a trade already in progress.

## The S3 missing-object trap

The least-obvious failure came from IAM. With object-read permission but no bucket-list permission, S3 can return `403 AccessDenied` for a missing key rather than `404 Not Found`.

A missing scoped policy is normal. A `403` looks like a broken control channel and triggers false alarms.

The fix was to grant narrowly scoped bucket-list access for the control prefix in addition to object reads. Least privilege is still the goal, but the smallest-looking permission set is not correct if it changes normal absence into an error.

The final design has two brakes: a network-independent local halt and an outbound-only remote policy. Both share one invariant: the bot may stop only after it knows what happened to the position.

*This is an educational systems-design example, not investment advice.*

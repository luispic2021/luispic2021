---
layout: post
title: "One Strategy, Two Accounts, Zero Silent Fallbacks"
date: 2026-06-20
categories: automated-trading reliability python
description: How I isolated journals, logs, credentials, and alerts before letting one automated trading strategy run safely across multiple brokerage accounts.
---

Running one strategy against two brokerage accounts sounds like a configuration change. In a live trading system, it is an isolation problem.

My bot originally assumed one process, one account, and one trade journal. Launching it twice would have sent both processes into the same CSV ledger. Positions, P&L, and recovery state could be mixed before I noticed anything was wrong.

The feature was not “run the bot twice.” It was “make every side effect belong to exactly one account.”

## Give each process an identity

Each bot instance now receives an explicit account selection and derives a non-secret label for operational output. That label scopes three things:

- its trade journal
- its log file
- every notification it sends

The filenames follow a pattern like this:

```text
2026_perma_journal_<account-label>.csv
permabot_<account-label>.log
```

Separate journals also mean separate file locks. One process cannot block or corrupt another account’s write, and cumulative results are calculated from the correct history.

The label is useful for observability, but it is not authorization. Credentials stay in environment variables, and neither the label nor a real account identifier belongs in source control or public logs.

## Fail loudly when identity is missing

The most important behavior is what the bot refuses to do.

If an operator selects an account whose credentials are not configured, startup fails. It does not fall back to the primary account. A silent fallback could place the intended trades twice in one account while leaving the other untouched.

For software that can move money, an explicit failure is safer than a plausible guess.

I applied the same rule to alerts. My first implementation added the account label inside the notification helper. It worked, but the text at each call site no longer matched the message that was actually sent. I moved the label into the call sites so the source shows the complete alert.

That is a small readability choice until something breaks near the close. Then “what you read is what gets sent” becomes an operational feature.

## Migrate history without gambling on it

Changing the journal convention created a data migration. Existing files had to move to account-scoped names without losing prior trades or overwriting a newer target.

I used a small idempotent migration:

1. Resolve the old and new paths.
2. Stop if the target already exists.
3. Preserve the original until the move succeeds.
4. Make a second run a no-op.

“Solo project” does not make historical state disposable. If a system resumes from files, renaming those files is a production change.

## Isolation revealed the next shared dependency

The account-level files are now independent, but the processes still depend on shared brokerage authentication state. A token refresh in one process can invalidate the in-memory token held by another.

That is the next boundary to fix: one component should own refresh, while the workers adopt the latest valid token instead of racing to rotate it.

The broader lesson is simple. Multi-account support is not duplication. It is identity carried consistently through configuration, persistence, observability, and failure behavior.

When software trades unattended, “it runs twice” and “the two runs cannot confuse each other” are very different standards.

*This is an engineering case study for educational purposes, not investment advice.*

---
layout: post
title: "From Disk to S3: A Safer Trade-Journal Pipeline"
date: 2026-06-28
categories: automated-trading data-engineering aws
description: A practical design for syncing trading journals to S3 without turning a cloud outage during sync into a false local-write failure or lost history.
---

Every completed trade in my bot becomes a row in a CSV journal. For a long time, those files lived on the trading machine and were copied through Git as an informal backup.

That was enough until I wanted the data available for analysis. The useful change was not simply “upload CSVs to S3.” It was deciding which failure should be allowed to affect which part of the system.

## Local persistence remains the source of truth

After a journal row is written successfully, the bot calls a small S3 upload helper. The destination bucket comes from an environment variable, and the upload is a no-op when cloud sync is not configured.

The sequence matters:

```text
close trade
  → append journal row locally
  → confirm local write
  → attempt cloud sync
  → warn, but preserve the row, if sync fails
```

The upload sits outside the local write’s exception handler. Otherwise, an S3 outage could produce a “journal write failed” error even though the row was safely on disk. That message would be both alarming and wrong.

Local durability and remote availability are separate outcomes, so the code reports them separately.

The S3 client is initialized lazily and reused by the long-running process. The object key also separates raw inputs from future analytics outputs:

```text
landing/journals/<year>/<filename>
```

The bot writes only to the landing prefix. An analytics job can read from there and write curated tables elsewhere under a different role. The boundary keeps permissions and ownership understandable.

## Historical data was the harder part

New uploads solved only the next trade. The existing archive contained several journal schemas from different stages of the bot:

- an early, smaller schema with one symbol and one entry price
- a transitional options schema
- the current account-scoped schema with separate underlying and option fields

I built the migration around the data that actually existed, not the schema I wished had existed. It detects the source shape, maps fields conservatively, calculates only derivable values, and leaves unavailable fields empty.

The natural deduplication key combines strategy, entry time, and contract. Existing annual rows win over imported archive rows. The migration writes backups before modifying an annual journal and produces the same result when run again.

That idempotence matters because migrations rarely happen once. They get interrupted, reviewed, rerun, and occasionally resumed months later by someone who has forgotten the original assumptions.

## Git was not the first thing to remove

Once S3 existed, removing journals from Git looked obvious. Operationally, it was premature.

When a tracked file is deleted in a commit, another machine that pulls the change also deletes its local copy. That is correct Git behavior and a poor surprise for a stateful production host.

So I kept the existing backup path until the S3 flow was validated end to end. The sequence is now:

1. Upload new rows reliably.
2. Reconcile local journals against S3.
3. Test restoration.
4. Only then remove the files from version control in a coordinated change.

The takeaway is not that object storage is complicated. It is that adding a new durable path does not make the old one disposable on day one.

Design the handoff for partial success: a trade journal can be safe locally even when cloud sync is down, and a migration can stop halfway without destroying the history it is trying to preserve.

*This post describes system design, not investment advice.*

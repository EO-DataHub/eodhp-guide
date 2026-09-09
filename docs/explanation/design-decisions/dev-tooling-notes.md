---
title: Dev tooling notes
doc_status: unreviewed
tags:
  - pulsar
last_reviewed:
reviewed_by:
review_notes: "Working notes rather than a decision record; migrated from the standalone accounting and billing repository; not yet reviewed in the guide."
---
# Dev tooling notes

Working notes for the `accounting-service` development environment. Separate from
the ledger design documents, which record decisions rather than work in progress.

## Open: taming Pulsar client log output

`pulsar.Client(..., logger=...)` rejects Loguru's logger. The cause is a strict
type check in the installed client, `pulsar/__init__.py:959-966`, which accepts
only three things:

- `logging.Logger`
- `pulsar.ConsoleLogger`
- `pulsar.FileLogger`

Anything else raises `ValueError`. Loguru's logger is `loguru._logger.Logger`,
which is not a `logging.Logger` subclass, so duck typing cannot get past the
check.

The bridge behind that parameter also looks broken. `Client._prepare_logger`
(`pulsar/__init__.py:1002-1009`) calls:

    logger.log(logging.getLevelName(level), message)

`logging.getLevelName` returns a string, and `logging.Logger.log` requires an
integer level, so it raises `TypeError: level must be an integer`. Confirmed by
calling the bridge directly with both an integer level and a `LoggerLevel`
member. The level names it produces are also wrong, because the C++ layer uses
its own scale:

| `LoggerLevel` | Value | `getLevelName` returns    |
|---------------|-------|---------------------------|
| `Debug`       | 0     | `NOTSET`                  |
| `Info`        | 1     | `Level LoggerLevel.Info`  |
| `Warn`        | 2     | `Level LoggerLevel.Warn`  |
| `Error`       | 3     | `Level LoggerLevel.Error` |

So the documented `logger=logging.getLogger('pulsar')` example appears not to
work either. Verify the installed `pulsar-client` version before spending more
time here, and check whether a later release fixes it.

Two candidate ways forward, neither yet tested against a broker:

1. `pulsar.ConsoleLogger(pulsar.LoggerLevel.Warn)`. Filters in C++ before
   anything reaches Python, so it is the cheapest option and needs no
   workaround. It writes to stderr and therefore bypasses Loguru formatting and
   sinks.
2. A small adapter subclassing `logging.Logger` to satisfy the type check, with
   `log` overridden to ignore the broken integer contract and forward to Loguru,
   mapping the level strings above. Untested: a run was started and cancelled.

## Dekaf UI

An Apache 2.0 Pulsar UI, in the official `apache/pulsar-helm-chart`. It can
produce messages, but only one hand-typed payload at a time, so it does not
replace the script. It is worth having to read topics, above all the
`dead-letter-accounting-ingester` topic, which currently has no other way to
inspect it.

Against a shared cluster it reads through its own subscription, so it observes a
copy and cannot divert messages from the deployed ingester. Three cautions: it
can create topics and edit policies, it has no multi-user support, and an
abandoned subscription would hold a backlog and retention on the topic. Whether
its consume sessions are durable subscriptions or non-durable readers is
unconfirmed.

To divert messages to a local service, use the takeover mode already in
`eodhp_utils`: `--takeover` on the ingester CLI. Never against production, since
it suspends the real consumer too.

## Done

Alembic is in place: `alembic.ini`, `alembic/env.py`, the baseline revision, and
`7c3d5e9a1f42`, which renames constraints to the metadata naming convention on
databases created before Alembic owned the schema. That migration is needed,
because production will not be recreated.

The tests run against a throwaway PostgreSQL container started by
`tests/conftest.py`, so `uv run pytest` needs only Docker. `tests/test_migrations.py`
has been removed: migrations are applied and reviewed by hand. The one thing it
covered that no tool reports is recorded on `UNCOMPARED_INDEXES` in
`alembic/env.py`.

`dev/inject.py` sends fake Pulsar messages and is wired up as a console script, so
`uv run inject` works. It has `billing-event` and `workspace-settings` commands;
`tests/send_test_message.py` has been folded into it. Consumption rate samples are
not covered yet, so the one-hour windowing in
`ConsumptionSampleRateIngesterMessager` is still only reachable through the tests.

The Loguru question above is answered in practice: `dev/inject.py` and
`catalogue-control` both pass `pulsar.ConsoleLogger(pulsar.LoggerLevel.Warn)`, which
filters in the C++ client. Routing Pulsar's output into Loguru is still unsolved and
still not worth solving.

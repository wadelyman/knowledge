---
type: Concept
title: Request logs
description: Router-recorded per-request logs on a Firestore bus with native 7-day TTL — the hot path never blocks on logging.
tags: [operations, logging, firestore, router]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
sources:
  - id: firestore-ttl
    resource: https://cloud.google.com/firestore/docs/ttl
    title: Firestore time-to-live policies
    author: team:platform
---

# Request logs

M8 gave users failure visibility (404, 502, gate 403) with zero new infra: the
router sees every app request, and its
accesslog middleware records them through a buffered channel with a single
writer goroutine and drop-on-full semantics. A full buffer degrades to
no-logs, never no-traffic — the hot path never blocks on logging.

## The bus

Firestore is the log bus (`projects/<name>/logs`) with native Firestore TTL on
an `expire_at` field (7 days) — no sweeper, no retention code. The API
endpoints (`GET …/logs` and SSE `…/logs/stream`) are the stable contract; the
bus is replaceable behind them if Cloud Logging wins later. There is no
delete-cascade for logs — the TTL reaps a deleted project's entries within 7
days.

## Privacy rules

Query strings are never logged (tokens live there) — verified live with a
`?token=…` request. Bodies are never logged. Non-project traffic (bare domain,
`/api/*` proxy calls) is captured by the middleware but not persisted, because
no per-project collection exists for it.

## Consumers

The web console's LogsCard polls `…/logs` every 3 seconds while visible; SSE
(replay, 2s poll, 25s heartbeat, 10-minute close) exists for
`mycelia logs --follow`. For container stdout/stderr rather than request
lines, see [runtime logs](/domains/operations/runtime-logs.md).

## E2E-found fix

The router SA needed `roles/datastore.user` (it had viewer) — the writer got
PermissionDenied and correctly degraded to no-logs rather than breaking
traffic.

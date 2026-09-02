---
type: Concept
title: Audit log
description: Append-only per-project audit trail in Firestore — written after mutations succeed, never gating them.
tags: [operations, audit, firestore, observability]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Audit log

M7 added an append-only audit trail at `projects/<name>/audit` in Firestore.
Every mutating route writes an entry after the mutation succeeds — never
before, and never gating the mutation. Audit is observability, not a control.

## Properties

- Owner-only read via `GET …/audit`, surfaced in the web console's Activity
  card.
- Secret values are never logged — the entry records the mutation, not the
  payload.
- There is deliberately no `project.delete` event: the trail dies with the
  project. Audit entries are not part of the delete cascade because the
  cascade removes the project document itself.

## Contrast with request logs

The audit log records platform mutations (deploys, secret writes, CI link
changes, deletes). Per-request traffic visibility — 404s, 502s, gate 403s —
lives in [request logs](/domains/operations/request-logs.md), a separate
Firestore-backed bus with native TTL. Runtime stdout/stderr from container
apps is a third stream, proxied from Cloud Logging as runtime logs.

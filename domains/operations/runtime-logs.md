---
type: Concept
title: Runtime logs
description: Container app stdout/stderr via a read-only Cloud Logging proxy — no bus, no duplication, no retention code.
tags: [operations, logging, cloud-logging, containers]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Runtime logs

M9: container apps' stdout/stderr already exists in Cloud Logging, so the API
proxies a scoped `entries:list` — read-only, filter pinned to
`resource.labels.service_name="app-<project>"` plus the stdout/stderr streams.
No Firestore duplication, no retention code, no new bus: the design cites a
prior platform's retrospective that Firestore-as-log-bus ages poorly.
Request logs keep their Firestore bus
because they need per-project user-visible failure records, not bulk streams.

## Endpoint behavior

- `GET …/logs/runtime` is owner-only, like the other log surfaces.
- Static projects get a 400, not a fake empty list — distinct error codes as
  UX.
- Severity is whitelisted before it lands in a filter clause (injection
  guard), and pagination uses an RFC3339 `before` cursor.
- No SSE: Cloud Logging has no cheap push path, so the web console polls at 3s
  and `mycelia logs --runtime --follow` polls at 2s. Each poll costs a Logging
  API call — fine at one operator.

## Terraform footprint

The API service account holds `roles/logging.viewer` and the Logging API is
enabled. Nothing else — the feature is a proxy, not a pipeline. E2E verified
with an nginx container app: access and error lines arrive live with revision
labels.

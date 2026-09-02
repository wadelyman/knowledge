---
type: Concept
title: Loam overview
description: Loam is Mycelia's knowledge node — an OKF v0.2 spine inside mycelia-api with a derived confidence model, MCP surface, and vector sidecar.
tags: [knowledge, loam, okf, m11]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Loam overview

M11 phase 1 (implemented 2026-09-02) added Loam, the platform's knowledge
system. It ingests OKF v0.2 bundles — this corpus is one — projects them into
Firestore, derives a confidence score per concept, and serves search and
traversal over HTTP and MCP. The canonical bundle is the `wadelyman/knowledge`
repository (decision O-7).

## Where it lives

The OKF spine is new files inside `api/` served by `mycelia-api`, not a
separate service: phase 1 needs nothing the API lacks (Firestore access,
verified identity, browser and CLI reachability, test harness), and a separate
service would buy a second backend, IAP audience, service account, and image
for zero capability. The split is deferred until the phase 4 agent runtime
needs its own failure domain.

## Components

- `api/okf` — pure package: OKF v0.2 parse/serialize (unknown-key raw
  preservation), link extraction, conformance lint plus secret scan,
  structural chunker. See [OKF bundle format](/domains/knowledge/okf-bundle-format.md).
- `OKFStore` — its own interface (`api/okf_store.go`) with `FakeOKFStore`
  beside it in production code; the two-pass idempotent ingestor sits on top.
- [Confidence model](/domains/knowledge/confidence-model.md) — derived at read
  time, never stored; every score carries its model version.
- The MCP surface — hand-rolled JSON-RPC 2.0
  over HTTP.
- loam-index — Python/FastAPI +
  turbovec vector sidecar, internal-only, single writer.

## Authorization

OKF read routes require only a verified caller — the bundle is org-wide by
construction. `POST /okf/ingest` is gated on an `OKF_OPERATORS` env allowlist,
deliberately not a tier. Writes from agents go
through the [proposal flow](/domains/knowledge/proposal-flow.md), and
[agent directives](/domains/knowledge/agent-directives.md) (not yet written)
will govern phase 4 agent behavior as concepts in the bundle itself.

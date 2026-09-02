---
type: Concept
title: Tiers
description: Per-user deploy caps — the tier lives in users/<email>, caps are computed by capsForTier, and the operator flips tiers via gcloud.
tags: [operations, tiers, quotas, firestore]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Tiers

M7 moved the platform's deploy clamps into `capsForTier` (`api/tiers.go`).
Tier 1 is the default and equals the pre-M7 values — e.g. 1 container project
per owner, 0.5 vCPU / 256Mi / max 2 instances per app. The tier lives in a
Firestore `users/<email>` document.

## Administration

There is no self-serve and no billing: the operator flips tiers directly with
a gcloud-token curl against the Firestore REST API (exact command in RUNBOOK),
patching `users/<email>` with an `updateMask` on the `tier` field.

## Where caps apply

The upload body cap is per-caller (tier) — auth runs before the cap is chosen.
Preview counts share a per-tier cap across static and container previews.

## Deliberate non-goals

OKF ingest authorization is deliberately NOT a tier — it is an
`OKF_OPERATORS` env allowlist on the API, because
tiers are per-user deploy caps, not a general permission system. Per-domain
ACLs for knowledge content are likewise deferred.

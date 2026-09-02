---
type: Concept
title: Confidence model
description: Confidence is derived at read time, never stored — every score carries its model version (okf-confidence/v1) and decomposition.
tags: [knowledge, confidence, trust, okf]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Confidence model

OKF §5.1 refuses a stored score, so Loam derives confidence at read time and
never writes it into frontmatter or Firestore as authoritative state. Every
score carries its model version (`okf-confidence/v1`) and its decomposition,
so a consumer can see why a concept scored as it did and detect when the model
changes underneath.

## Inputs

The v1 decomposition draws on frontmatter trust signals — `verified` by a
`human:` actor, `status` (draft, stable, deprecated), `stale_after`, and
`sources` — plus graph signals from the link structure. `usage_count` is
banded and moves the score by at most 0.1, so popular-but-unverified content
cannot outrank verified content on traffic alone.

## Hard rule

Agents can never write a `human:` verification (Rule 3). Human attestation is
the top of the trust ladder precisely because no automated path can mint it;
agent contributions arrive through the
proposal flow and stay machine-graded
until a human verifies them.

## Why derived, not stored

A stored score rots silently: the underlying signals change (a source goes
stale, a verification ages) while the number stays put. Deriving at read time
keeps the score a function of current evidence. The filter grammar (type,
tags, status, and friends) pushes down into Firestore queries so filtered
recall stays cheap.

---
type: Concept
title: Proposal flow
description: Agent-originated knowledge changes are proposals — validated against OKF conformance, queued for review, never committed directly.
tags: [knowledge, proposals, agents, review]
status: draft
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Proposal flow

Draft: the read side and proposal-queue schema shipped in M11 phase 1, but the
phase 4 agent runtime that produces proposals is not built yet, and the E3
live drills plus the 30-concept human audit are still pending.

## Model

Agents never write the bundle directly. A change arrives as a proposal —
create, update, or deprecate — which is validated against OKF conformance
before it is queued. Updates are patches, not replacements, and unknown
frontmatter keys are preserved through the patch path (the OKF format's
preservation rule
applies to agents most of all, since they are the most frequent rewriters).

## Writing an edge is a body edit

Linking is a prose change: `okf_link` edits the document body to add a
sentence containing the markdown link, because the surrounding sentence is the
relation's only evidence — a frontmatter patch cannot carry it.

## Trust constraints

- Proposals are reviewed under a review policy before application; nothing an
  agent writes lands silently.
- Agents can never write a `human:` verification — the
  confidence model treats human
  attestation as unforgeable-by-construction.
- Agent opinion (flags, doubts, suggested relations) lives in Firestore side
  channels (`okf/flags`, `okf/agent_runs`), not in bundle frontmatter —
  opinion is not knowledge, and putting it in the bundle pollutes it for every
  consumer.

## Access path

Proposals arrive over the [MCP surface](/domains/knowledge/mcp-surface.md)
(`okf_propose_create`, `okf_propose_update`, `okf_propose_deprecate`), subject
to the same verified-caller requirement as all OKF routes.

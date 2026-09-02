---
type: Concept
title: Previews
description: Branch deploys at branch--project hosts — static previews with lazy TTL, container previews as per-branch Cloud Run services, and the CI PR lane.
tags: [operations, previews, branches, cloud-run]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
---

# Previews

Preview hosts are `<branch>--<project>.<domain>` — a single DNS label the
wildcard certificate already covers. `--` is banned in new project names to
keep the split unambiguous, and a whole-label project name wins over the
preview parsing.

## Static previews (M7)

A `branch` field on upload creates a static preview under a per-tier preview
cap. Expiry is a lazy 14-day TTL reaped on `listDeployments` reads — no
scheduler. At one operator, an unread project's orphaned previews cost pennies
and a sweeper can land later without a format change.

## Container previews (M10)

Branch uploads on container projects deploy per-branch Cloud Run services
`app-<project>-<sanitize(branch)>` through the unchanged
container deploy pipeline (min instances 0, by digest). Sanitization respects the 63-char service-name
cap and trims trailing hyphens — GCP rejects them. The router resolves the
live preview deployment doc's `url` and proxies with the same per-host
identity-token pattern as main; it never reconstructs service names. The
reaper and the delete cascade tear down preview Cloud Run services along with
everything else — the router never checks TTL itself, so an expired preview
404s because its doc is gone.

## CI PR previews

PR events deploy previews through the second WIF provider (`github-pr`) and a
preview-only `ci-pr-<project>` service account — see
[CI workload identities](/domains/identity/ci-wif-identities.md). CI authz is
split on the tgz endpoint only (`POST /api/v1/deployments`); the zip endpoint
stays owner-only, and CI identities use tgz by contract.

## Trust note

Previews inherit app secrets — the trust boundary is repo write access. A
secret-free preview lane is a noted follow-up if outside collaborators ever
push branches.

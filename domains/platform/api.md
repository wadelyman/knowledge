---
type: Concept
title: API service
description: The mycelia-api Cloud Run service — projects, deployments, secrets, audit, tiers, and the Loam OKF spine.
tags: [platform, api, firestore, cloud-run]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# API service

The API (`api/`, Go module `mycelia/api`, now on go 1.25) runs as Cloud Run
service `mycelia-api` with min 0 / max 2 instances and LB-only ingress. In M1
it answered only `GET /healthz`; it has since grown into the platform's
control plane.

## Responsibilities

Projects and deployments (zip and tgz uploads, CI-originated deploys),
write-only secrets, the per-project audit log,
per-user tiers, CI link/unlink, and log reads.
Firestore is the routing table. Since M11 phase 1 the same service also hosts
the Loam OKF spine (`api/okf` package, `/api/v1/okf/*` routes, and the MCP
endpoint) — a separate service would have bought a second backend, IAP
audience, service account, and image for zero capability.

## Identity handling

Every request except `GET /healthz` passes
[JWT verification](/domains/identity/jwt-verification.md) of the IAP
assertion; an unconfigured verifier fails closed. Identity comes only from the
verified `X-Goog-IAP-JWT-Assertion`, or a verified `X-Mycelia-User-JWT`
forwarded by the router's service account. Per-route authorization is
owner-only by default; CI identities get deploy-only access to their own
project, and OKF ingest is gated on an `OKF_OPERATORS` env allowlist.

## Notable implementation constraints

- Uploads that trigger container builds return 201 `status: building`
  immediately and state advances by reconcile-on-read — no background
  goroutines, because Cloud Run throttles CPU outside requests.
- The run v2 API traps are documented in `api/gcp_apis.go` comments:
  `UpdateService` is not upsert, CPU throttling is the proto field
  `resources.cpuIdle` (the v1 annotation is ignored), and readiness lives in
  `terminalCondition`, not the `conditions` list.
- The verifier is stdlib-only crypto/ecdsa (~40 lines); the module's only
  direct non-GCP dependency is `gopkg.in/yaml.v3`, promoted for OKF
  frontmatter handling.

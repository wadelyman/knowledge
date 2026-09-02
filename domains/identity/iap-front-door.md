---
type: Decision Record
title: IAP front door
description: IAP is the only auth perimeter — both backends, no app-level login system, no unauthenticated endpoints except health probes.
tags: [identity, iap, security, perimeter]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
---

# IAP front door

Decided 2026-08-25 (M1): IAP is the only auth perimeter. One front door —
every request arrives with a Google-verified identity, and there is no login
system to build or breach. The decision survived the later IdP change intact;
only the provider behind IAP changed (see the auth mode toggle).

## Mechanics

Both backends (router and api) have IAP enabled with their own configuration,
and Cloud Run ingress is LB-only so the `run.app` URLs cannot bypass the
perimeter. After authentication, IAP injects `X-Goog-IAP-JWT-Assertion` — a
Google-signed ES256 JWT regardless of which IdP the human used. Since M4 the
API [verifies that JWT](/domains/identity/jwt-verification.md) on every
request; before M4, identity was trusted from the unsigned email header.

## Hard rules

- **No unauthenticated endpoints.** `GET /healthz` is the only path that may
  answer without a verified identity, and only because LB health probes carry
  no JWT (Google prober allowance, by platform design). A human curl of
  `/healthz` without credentials gets a 302 — that is the perimeter working,
  not a bug.
- **No long-lived secrets.** No API keys, no deploy keys, no SA keys anywhere
  in the request path.
- **Fail closed.** Unconfigured verifier, missing header, unknown caller,
  missing group claim — all reject. There is no "allow anyway" path.

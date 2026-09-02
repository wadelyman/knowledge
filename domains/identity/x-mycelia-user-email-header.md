---
type: Decision Record
title: X-Mycelia-User-Email trust header
description: M5's forwarded-identity trust rule — the API honored X-Mycelia-User-Email only from the router SA. Superseded by M4 JWT verification.
tags: [identity, router, api, superseded]
status: deprecated
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# X-Mycelia-User-Email trust header

Deprecated: superseded 2026-08-30 by
JWT verification (M4 phase 1). The
header is gone; router-SA forwarded identity now requires a verified
`X-Mycelia-User-JWT`.

## What it was

M5 (2026-08-28) needed the web UI — same origin as the router — to call the
API, which sits behind its own IAP gate the browser cannot pass interactively.
The router proxied `/api/*` with its own SA identity token and forwarded the
real user in an `X-Mycelia-User-Email` header.

## The trust rule

The API honored `X-Mycelia-User-Email` ONLY when the IAP caller was exactly
`ROUTER_SA_EMAIL` (env-configured; empty meant the feature was off). Any other
caller setting the header was evaluated by their own identity — the header was
ignored. The proxy stripped all inbound auth, cookie, and IAP headers and
never followed upstream redirects; the upstream was the fixed env-derived
`https://api.<domain>` with no user input in the URL.

## Why it was always temporary

The rule was caller-match trust, not proof. The M5 spec explicitly deferred to
M4's cryptographic verification, and M4 deleted both the header and the
bare-email `callerIdentity` second shape it had introduced. Kept here as
historical context for anyone reading pre-M4 commits.

---
type: Concept
title: JWT verification (M4 front door)
description: The API verifies the IAP assertion on every request — stdlib ES256 verifier, exact backend-service audience, fail-closed.
tags: [identity, jwt, iap, es256]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
---

# JWT verification (M4 front door)

M4 phase 1 (deployed 2026-08-30) replaced all header trust with cryptographic
proof. API identity now comes ONLY from a verified `X-Goog-IAP-JWT-Assertion`;
the M2-era loose header parsing is deleted, and the M5
[X-Mycelia-User-Email trust header](/domains/identity/x-mycelia-user-email-header.md)
is gone.

## Verification rules

- Algorithm: ES256, Google-signed regardless of which IdP the human
  authenticated with — the verifier is IdP-agnostic across
  auth modes.
- Audience: pinned exactly to
  `aud = /projects/<num>/global/backendServices/<api-backend-id>` — the
  signed-headers doc form, NOT the `iap_web/compute/services` name the
  settings API uses. Never a prefix match.
- Forwarded user identity from the router SA requires a verified
  `X-Mycelia-User-JWT` against the **router** backend audience — a separate
  audience from the API's own.
- ES256 for IAP assertions, RS256 for GitHub OIDC (M6) — separate verifiers,
  never routed through each other.

## Implementation

The verifier is stdlib-only — about 40 lines of crypto/ecdsa, no new module
dependency. An unconfigured verifier fails closed: everything except
`/healthz` returns 401, and terraform always sets the env vars. Router serving
stays header-gated (read-only, LB-only ingress); router-side JWT verification
is a deferred defense-in-depth step.

## Scope note

M4's remaining hardening phases (2+) are not started. Per-app service accounts
and write-only secrets, originally bundled into M4, shipped with M3 where
their consumer (container apps) exists.

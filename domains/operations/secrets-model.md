---
type: Concept
title: Write-only secrets model
description: App secrets live in Secret Manager — the platform can write and bind them but can never read versions back.
tags: [operations, secrets, secret-manager, security]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
sources:
  - id: sm-docs
    resource: https://cloud.google.com/secret-manager/docs/access-control
    title: Secret Manager access control
    author: team:platform
---

# Write-only secrets model

Container apps consume platform-managed secrets from Secret Manager, injected
as `secretKeyRef: latest` environment variables on the per-app Cloud Run
service. The model is write-only by design: the platform's role denies
`versions.access` — verified by E2E impersonation of the API service account.

## API surface

The API exposes PUT (create/update), DELETE, and GET-names-only. Secret values
never appear in any response, and they are never written to the
[audit log](/domains/operations/audit-log.md) — the audit records the
mutation, not the value.

## Two subtle requirements

- The write-only role still needs `secrets.getIamPolicy`, because the flow
  that grants the app's service account accessor binding reads the existing
  policy first.
- Changing a secret alone does not re-resolve it in a running app — the
  container deploy pipeline
  forces a fresh revision per deploy via a `mycelia.dev/deploy-id` annotation,
  so redeploying picks up new values even on an identical image digest
  (E2E-verified).

## Boundary for previews

Previews inherit app secrets — the trust boundary is repo write access. A secret-free preview lane is a noted follow-up
if outside collaborators ever push branches.

## What this is not

Platform-level operator secrets (the Entra client secret, the IAP OAuth client
secret) are a separate concern: they live only in terraform state, injected
via `TF_VAR_…` and never committed. AUTH_FLOWS.md's hard rule stands: no API
keys, no deploy keys, no SA keys.

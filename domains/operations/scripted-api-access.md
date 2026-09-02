---
type: Playbook
title: Scripted API and E2E access
description: Operator curl/E2E access via a throwaway impersonated service account — user gcloud cannot mint custom-audience ID tokens.
tags: [operations, playbook, iap, impersonation]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
stale_after: 2026-08-01T00:00:00Z
---

# Scripted API and E2E access

Stale-flagged: the IAM propagation timings in this note were measured once
(2026-09-01) and drift; re-verify before relying on them in a hurry.

## Why a service account is required

`gcloud auth print-identity-token --audiences=...` rejects user accounts
("requires valid service account"). Operator curl and E2E therefore go through
a throwaway SA, impersonated. Three requirements, all enforced by
IAP: the token's `aud` must equal the
programmatic IAP OAuth client ID (`iap_programmatic_client_id` in
prod.tfvars); that client must be registered in the backend's IAP settings
(`programmaticClients` — terraform `google_iap_settings`, both backends); and
the calling identity must hold `roles/iap.httpsResourceAccessor` on the
backend.

## Recipe (from RUNBOOK)

```bash
SA=mycelia-e2e@${PROJECT_ID}.iam.gserviceaccount.com
gcloud iam service-accounts create mycelia-e2e
gcloud iam service-accounts add-iam-policy-binding "$SA" \
  --member="user:you@example.com" --role="roles/iam.serviceAccountTokenCreator"
# grant roles/iap.httpsResourceAccessor on BOTH backend services, then:
TOKEN=$(gcloud auth print-identity-token --impersonate-service-account="$SA" \
  --audiences="$CLIENT_ID" --include-email)
curl -H "Authorization: Bearer $TOKEN" https://api.<domain>/api/v1/projects
```

Delete the SA when done — it is unmanaged standing access otherwise. Tokens
expire in about an hour.

## IAM propagation timings (measured 2026-09-01)

- Impersonation on a **newly created** SA can take ~45 minutes to start
  working (`getOpenIdToken` 403s until then — even `roles/owner` doesn't help;
  it lacks `iam.serviceAccounts.getOpenIdToken`).
- Fresh role grants propagate in minutes; impatient retries 401 or
  PERMISSION_DENIED. Sleep 60s after granting IAP roles before minting.
- When running E2E: create the SA and grant roles FIRST, then do other work
  while IAM propagates.

Symptom-to-cause mapping for 302/401 failures belongs in the (not yet written)
[debugging guide](/domains/operations/debugging-guide.md) concept.

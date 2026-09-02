---
type: Concept
title: Workforce Identity Federation (Entra)
description: How Microsoft Entra ID sign-in bridges to GCP through an org-level workforce pool, OIDC provider, and group-claim mapping.
tags: [identity, entra, workforce-federation, oidc]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
sources:
  - id: wif-docs
    resource: https://cloud.google.com/iam/docs/workforce-identity-federation
    title: Workforce Identity Federation documentation
    author: team:platform
  - id: iap-workforce-docs
    resource: https://cloud.google.com/iap/docs/workforce-identity-federation
    title: IAP with Workforce Identity Federation
    author: team:platform
---

# Workforce Identity Federation (Entra)

User requirement (2026-08-25): the organization authenticates with Microsoft
Entra ID, not Google accounts. Workforce Identity Federation bridges Entra to
Google; IAP stays as the enforcement point unchanged.

## The moving parts

All terraform-managed pieces live in `workforce.tf`, count-gated on
`auth_mode = "entra"`:

- Workforce pool `mycelia-entra` — an **org-level** container; the operator
  needs org-level IAM, which is why google mode exists for the demo
  environment (see [auth mode toggle](/domains/identity/auth-mode-toggle.md)).
- Pool provider `entra-oidc` — OIDC link to the Entra tenant
  (`login.microsoftonline.com/<tenant>/v2.0`), authorization CODE flow.
- Entra app registration — client ID, client secret, and groups claim;
  created manually in the Entra portal (SETUP.md §0). Its redirect URI is the
  terraform output `entra_redirect_uri`.
- Access grant — `roles/iap.httpsResourceAccessor` on both backends bound to
  `principalSet:…/workforcePools/mycelia-entra/attribute.groups/<entra_allowed_group_id>`.

## Sign-in flow

IAP's identity source is `WORKFORCE_IDENTITY_FEDERATION`, so an
unauthenticated browser redirects to the workforce sign-in and lands on the
Entra login page (Microsoft branding, conditional access, MFA — all Entra
policy applies). Entra issues an ID token; Google exchanges it and maps
attributes (`google.subject=assertion.oid`,
`google.display_name=preferred_username`,
`google.groups=assertion.groups`). IAP then checks the caller's mapped groups
against the allowed Entra group.

## Group claims are load-bearing

The Entra app registration MUST emit group claims. If it stops, every user
loses access — fail-closed and correct, but a total lockout. Rotation of the
Entra client secret is similarly critical: an expired secret locks everyone
out, so rotate before expiry (RUNBOOK).

---
type: Decision Record
title: Auth mode toggle
description: Identity is one variable — auth_mode = "google" (default) or "entra"; the IdP is config, not code.
tags: [identity, auth-mode, terraform, entra]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Auth mode toggle

Decided 2026-08-25: identity is a one-variable toggle, `auth_mode` in
`variables.tf`, because both targets are real — the operator's demo
environment (acadiasun.com, Google Workspace, no Entra admin access) and the
production target that requires Microsoft Entra ID.

## The two modes

| | `google` (default) | `entra` |
|---|---|---|
| Sign-in authority | Google accounts | Microsoft Entra ID via [workforce federation](/domains/identity/workforce-federation.md) |
| Access grant | `google_iap_member` (e.g. `domain:acadiasun.com`) | Entra security group via mapped group claims |
| Org-level resources | none | workforce pool + provider (org-level) |
| OAuth client | IAP's Google-managed client for browsers; manual client only for scripted access | one manual OAuth client as IAP's channel to the workforce service |

In google mode there are no org-level resources, so the org-level IAM
requirement drops away entirely.

## Consequences

Nothing app-side changes between modes — the downstream
`X-Goog-IAP-JWT-Assertion` is Google-signed either way, and
JWT verification is IdP-agnostic. The
Entra path is fully built but dormant until `auth_mode = "entra"` plus the
manual portal steps in SETUP.md §0. Switching modes later means
`terraform apply` tears down and recreates the relevant resources, and IAP
sessions reset.

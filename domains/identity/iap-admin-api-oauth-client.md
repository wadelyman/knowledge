---
type: Decision Record
title: IAP Admin API OAuth client creation
description: Creating IAP OAuth clients via the deprecated OAuth Admin API or identityAwareProxyClients REST — superseded by manual console clients.
tags: [identity, iap, oauth, superseded]
status: deprecated
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# IAP Admin API OAuth client creation

Deprecated: Google deprecated the IAP OAuth Admin API in July 2025, and the
`identityAwareProxyClients` REST workaround was abandoned for interactive
flows after a 2026-09-01 spike. OAuth clients are now created manually in the
console and supplied via terraform vars.

## What it was

Early milestones created IAP OAuth clients two programmatic ways:

- `google_iap_brand` / `google_iap_client` terraform resources — dead since
  the Admin API deprecation; the provider pin was raised to `~> 6.10` partly
  because of this.
- `identityAwareProxyClients.create` (REST) — worked for creating the M2 demo
  scripted-access client (brand `orgInternalOnly=true`, auto-internal).

## Why it was abandoned

Clients created via `identityAwareProxyClients` have no registrable redirect
URIs and reject loopback redirects (`redirect_uri_mismatch`), which makes them
unusable for the CLI's PKCE flow. The
CLI uses a manually created Desktop-app client instead (one-time console step,
RUNBOOK). The remaining manual clients — the workforce-channel client in entra
mode and the scripted-access client — are supplied via vars
(`iap_oauth_client_id` / `iap_oauth_client_secret`, the secret via `TF_VAR`
only, never committed).

## What survives

Creating the client is only half the story: IAP rejects service-account
identity tokens unless the token audience is listed in the backend's
`accessSettings.oauthSettings.programmaticClients`. The terraform
`google_iap_settings` resource (google-beta) is the working management path —
raw REST PATCHes of that field kept returning INVALID_ARGUMENT.

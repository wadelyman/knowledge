---
type: Concept
title: CLI OAuth (PKCE)
description: The mycelia CLI's sign-in — PKCE browser flow against a Desktop-app OAuth client, ID-token-only cache, direct to api.<domain>.
tags: [identity, cli, oauth, pkce]
status: draft
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# CLI OAuth (PKCE)

Draft: the pipeline is deployed, but CLI login E2E is pending the one-time
manual creation of the Desktop OAuth client (RUNBOOK), so this flow is not yet
verified end to end.

## Flow

The `mycelia` CLI (M8, stdlib-only Go module at go 1.22 — the 1.22 floor is
deliberate) talks to `https://api.<domain>/…` **directly**, never through the
router's `/api/*` proxy: the proxy requires a browser-session IAP assertion
and strips inbound `Authorization`, while the api host's own IAP accepts
programmatic bearer tokens (the same path as
[scripted access](/domains/operations/scripted-api-access.md)).

Sign-in is a PKCE (S256) browser flow against a **Desktop-app OAuth client**
whose client ID is registered as a programmatic client on the api backend's
IAP settings (`iap_cli_client_id` in terraform). The resulting ID token's
`aud` is that client ID, so IAP admits it and the API then authorizes
per-route as usual.

## Why a Desktop client

OAuth clients created via the `identityAwareProxyClients` API have no
registrable redirect URIs and reject loopback redirects
(`redirect_uri_mismatch`, spiked 2026-09-01) — see the deprecated
[IAP Admin API OAuth client](/domains/identity/iap-admin-api-oauth-client.md)
concept. Desktop-app clients allow RFC 8252 loopback redirects by design, and
their "secret" is not a secret (installed-app convention, same as gcloud's own
client) — it ships embedded in the binary.

## Token cache

`~/.config/mycelia/token.json` (directory 0700, file 0600, ownership-checked)
holds only the ~1-hour ID token; the refresh token is discarded in memory at
exchange time. On a 401 the CLI clears the cache and re-auths exactly once.
`mycelia logout` deletes the cache; the token itself stays valid until expiry.

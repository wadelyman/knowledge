---
type: Reference
title: MCP surface
description: Loam's hand-rolled JSON-RPC 2.0 over HTTP MCP server — no SDK, injection-delimited payloads, stdlib encoding/json only.
tags: [knowledge, mcp, json-rpc, api]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# MCP surface

Loam exposes an MCP server as hand-rolled JSON-RPC 2.0 over HTTP, built with
`encoding/json` only. An MCP SDK would have been the api module's first
non-GCP dependency with no justification yet — the M9 yaml.v3 debate is the
precedent for how conservatively dependencies enter this codebase. The server
shares the API service's routes, identity, and IAP
perimeter rather than standing up its own.

## Design points

- **Injection-delimited payloads.** Content returned to tool-callers is
  wrapped in delimiters so retrieved knowledge cannot masquerade as
  instructions to the calling agent — a prompt-injection guard at the
  protocol boundary.
- **Reads are the contract.** Search, fetch, and traversal map onto the same
  org-wide read authorization as `/api/v1/okf/*`; there is no separate MCP
  permission model.
- **Writes go through proposals.** Agent-originated creates and updates are
  validated against OKF conformance before queueing in the
  proposal flow — propose, never
  commit directly.

## Companion surface

The CLI mirrors the same functionality as `mycelia okf` (ten subcommands),
implemented as a thin server-side client — notably `okf lint` is
`POST /api/v1/okf/lint`, not a local token-less command, because importing
`api/okf` from the CLI would force the CLI module's go directive from 1.22 to
1.25 and the 1.22 floor is deliberate. The fix path (a nested go.mod for
`api/okf` at 1.22) is deferred.

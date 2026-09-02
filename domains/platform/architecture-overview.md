---
type: Concept
title: Architecture overview
description: End-to-end request path and component layout of the Mycelia platform on GCP Cloud Run.
tags: [platform, architecture, gcp, cloud-run]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
sources:
  - id: arch-doc
    resource: ARCHITECTURE.md (mycelia repo)
    title: Mycelia — Architecture
    author: team:platform
  - id: iap-docs
    resource: https://cloud.google.com/iap/docs/concepts-overview
    title: IAP concepts overview
    author: team:platform
---

# Architecture overview

Mycelia is an internal app-hosting platform: users deploy static sites and
container apps to `<name>.<domain>` behind a single SSO front door. The
platform metaphor is mycorrhizal — apps are trees, the platform is the fungal
substrate delivering identity, secrets, storage, and traffic.

## Request path

Browser traffic hits a global HTTPS load balancer (EXTERNAL scheme) fronted by
a Certificate Manager wildcard certificate. The URL map splits on host:
`api.<domain>` goes to the API backend, everything else to the
[router](/domains/platform/router.md) backend. Both backends have IAP on with
their own OAuth clients — see [IAP front door](/domains/identity/iap-front-door.md).
Both Cloud Run services use `INGRESS_TRAFFIC_INTERNAL_LOAD_BALANCER`, so the
`run.app` URLs cannot bypass the LB.

## What lives where

- Router (`router/`, Go module `mycelia/router`) — Cloud Run `mycelia-router`,
  min 1 / max 2, service account holds GCS objectViewer on the deploys bucket only.
- API (`api/`, Go module `mycelia/api`) — Cloud Run
  `mycelia-api`, min 0 / max 2.
- Loam index (`index/`, Python/FastAPI) — Cloud Run `loam-index`,
  internal-ingress, min=max=1, deliberately off the load balancer.
- Infra — the terraform layout, google and
  google-beta providers pinned `~> 6.10`, applied by the operator.

## Security boundaries

Five layers: network (HTTPS-only LB, LB-only ingress, private bucket with
public-access prevention), identity (IAP on both backends — one front door, no
anonymous endpoint except LB health probes), authorization (Entra group or
`google_iap_member` depending on auth mode), runtime (distroless nonroot
images, per-service least-privilege service accounts), and supply chain
(images tagged with immutable git SHAs, committed provider lock file,
linux/amd64 builds only).

## Load-bearing edge constraints

The [edge load balancer](/domains/platform/edge-load-balancer.md) has four
constraints that silently break the platform if changed casually: Certificate
Manager for wildcard TLS (never `google_compute_managed_ssl_certificate`),
EXTERNAL scheme (never EXTERNAL_MANAGED — IAP requires it), serverless NEGs
referencing string-literal service names (never resource references — circular
dependency between IAP audience and backend service ID), and the HTTPS proxy
depending on both certificate map entries.

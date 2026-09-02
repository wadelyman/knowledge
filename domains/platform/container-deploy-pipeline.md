---
type: Concept
title: Container deploy pipeline
description: The M3 async pipeline — zip upload to internal-ingress Cloud Run via staging bucket, Cloud Build, and Artifact Registry, with reconcile-on-read status.
tags: [platform, containers, cloud-build, cloud-run]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
---

# Container deploy pipeline

A zip upload with a root Dockerfile takes the container path: staging tgz →
Cloud Build → Artifact Registry `mycelia-apps` → internal-ingress Cloud Run
service `app-<project>`. Deploys are by digest, and a
`mycelia.dev/deploy-id` revision annotation forces a fresh revision per deploy
so changed secrets re-resolve even on identical digests.

## Async by necessity

Container builds take minutes and the LB backend timeout is 30 seconds, so the
API returns 201 `status: building` immediately. `listDeployments` and
`getProject` advance state (`building → deploying → live|failed`, plus
`superseded`) with one cheap API call per read — reconcile-on-read. There are
no callbacks and no background goroutines: Cloud Run throttles CPU outside
requests.

## Runtime shape

Apps run as per-app service accounts (`app-<project>`; project names cap at 26
chars so the SA account ID fits the 30-char limit) with
[write-only secrets](/domains/operations/secrets-model.md) injected as
`secretKeyRef: latest` env vars. Internal-ingress services are reachable only
over Google's internal network, which is why the
router egresses through a VPC connector and a
private `run.app` DNS zone.

## Platform-side permissions found by E2E

- Cloud Run validates at deploy time that the caller can pull the image, so
  the API SA holds `artifactregistry.reader` on `mycelia-apps`.
- The API SA holds project-level `iam.serviceAccountUser` instead of per-app-SA
  grants — an accepted hygiene trade-off (fewer moving parts; the API SA is
  already near-total if compromised).
- The platform secrets role is write-only and still needs
  `secrets.getIamPolicy` because the accessor binding flow reads the policy
  first.

## Delete cascade

Deleting a container project tears down, in order: deployment docs → GCS
prefix → Cloud Run service → app secrets → app SA → project doc. AR images and
staging objects are reaped separately by bucket/registry lifecycle rules.
[Previews](/domains/operations/previews.md) extend the same pipeline to
per-branch services (`app-<project>-<sanitized-branch>`, min instances 0).

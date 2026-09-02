---
type: Concept
title: Deploy pipeline
description: How code reaches Cloud Run — deploy.sh builds git-SHA-tagged images, pushes to Artifact Registry, and applies terraform.
tags: [platform, deploy, artifact-registry, terraform]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Deploy pipeline

`deploy.sh` is the only supported ship path: it builds the router and API
images (`linux/amd64`, distroless `static-debian12:nonroot` runtime), tags each
with `$(git rev-parse --short HEAD)`, pushes to the Artifact Registry
repository `mycelia`, and runs `terraform apply`. It also builds and rsyncs
the web UI to the `_web/` GCS prefix. The full flow is in the
ship and rollback playbook.

## Why git-SHA tags, never :latest

`:latest` plus terraform is a silent no-op: an identical image variable
produces no plan diff, so Cloud Run never rolls a new revision and redeploys
appear to succeed while changing nothing. Git-SHA tagging makes every deploy a
distinct, immutable var value — which also means uncommitted work cannot be
deployed; commit first.

## Supply-chain posture

Builder images match each module's go directive (golang:1.25 for router and
api; the CLI is stdlib-only at go 1.22
and built with `go build`, not deployed as a container). The terraform
provider lock file is committed. Digest pinning (`@sha256` plus Renovate) is
an accepted, still-open follow-up from M1.

## Interaction with terraform

Because the image vars live in `prod.tfvars` as pins, a bare
`terraform apply` without `-var-file=prod.tfvars` resets the services to the
variable defaults — the M1 "It's running!" hello placeholder. This has
happened once in production; the pins exist to prevent recurrence. See the
terraform layout for where the pins
and image variables live.

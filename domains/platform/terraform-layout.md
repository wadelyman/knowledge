---
type: Reference
title: Terraform layout
description: File-by-file map of the terraform/ module that provisions the whole platform.
tags: [platform, terraform, infrastructure]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Terraform layout

All infrastructure lives in `terraform/`, a single root module applied by the
operator with `prod.tfvars` (gitignored — never commit it). Providers are
google and google-beta pinned `~> 6.10`; 6.10 is the floor because
`google_iap_settings`, the only terraform surface for workforce-identity IAP
config, does not exist before it. `terraform validate` is the schema gate on
any provider bump.

## File map

- `main.tf` / `variables.tf` / `outputs.tf` — provider config, the `auth_mode`
  toggle and other vars, outputs like `entra_redirect_uri` and `name_servers`.
- `lb.tf` — global HTTPS load balancer, URL map, serverless NEGs, IAP member
  bindings.
- `cert.tf` — Certificate Manager wildcard certificate and map.
- `dns.tf` — Cloud DNS zone; the parent zone delegates NS records to it.
- `run.tf` — the two Cloud Run services and their scaling blocks.
- `buckets.tf` — deploys bucket and build-staging bucket with lifecycle rules
  (AR images and staging objects are reaped by 30-day/7-day lifecycles).
- `firestore.tf` — Firestore database and the TTL policy on log `expire_at`.
- `iam.tf` — service accounts and least-privilege bindings.
- `workforce.tf` — the Entra
  [workforce federation](/domains/identity/workforce-federation.md) pool and
  OIDC provider, count-gated on `auth_mode = "entra"`.
- `wif.tf` — the GitHub CI workload identity
  pool (`mycelia-github`) and providers (`github`, `github-pr`).
- `vpc.tf` — VPC connector, private `run.app` DNS zone, Cloud Router + Cloud
  NAT (the NAT exists because ALL_TRAFFIC egress routed API-proxy calls into
  the VPC with no internet path — a 2026-08-30 outage).
- `apps.tf` — per-app Cloud Run support resources.
- `monitoring.tf` — uptime check, 5xx alerts, email channel.
- `loam.tf` — the loam-index sidecar service and snapshot bucket.

## Cost notes

Steady state is a few dollars per month for the M1 baseline (min-instance
router plus LB); the M3 connectivity additions (VPC connector floor plus Cloud
NAT) add roughly $50–70/month. A fuller breakdown belongs in the (not yet
written) [cost model](/domains/operations/cost-model.md) concept.

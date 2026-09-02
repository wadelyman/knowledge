---
type: Concept
title: Monitoring and alerts
description: Terraform-only monitoring — uptime check accepting IAP's 302, 5xx alerts on router and api, operator email channel.
tags: [operations, monitoring, alerts, uptime]
status: draft
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Monitoring and alerts

Draft: the alert email path itself was never verified with a deliberate outage
(the operator did not want downtime during M7 E2E). The first real alert is
the test.

## Shape (M7)

Single-operator platform, so monitoring is terraform-only
(`terraform/monitoring.tf`) with no new runtime infrastructure:

- Uptime check against the LB that accepts IAP's 302 redirect as healthy — the
  perimeter redirects unauthenticated probes by design.
- 5xx alert policies on the router and api Cloud Run services.
- One email notification channel to the operator, plus a small dashboard.

## Manual checks that predate M7

The two numbers that matter when something looks wrong:

```bash
# 5xx on the LB
gcloud logging read "resource.type=https_load_balancer AND severity>=ERROR" --limit=20
# Cloud Run instance count / CPU — console: Cloud Run → mycelia-router → Metrics
```

## Reading router logs

Router 404s log `download <key>: <err>` — that line distinguishes "file
missing" from "GCS outage / IAM broken". Response-level detail for app traffic
lives in request logs, and container
stdout/stderr in runtime logs. Symptom-
to-cause mapping beyond this belongs in the (not yet written)
[debugging guide](/domains/operations/debugging-guide.md) concept.

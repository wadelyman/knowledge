---
type: Decision Record
title: Edge load balancer constraints
description: Four load-bearing constraints on the global HTTPS load balancer — Certificate Manager, EXTERNAL scheme, NEG string literals, cert-map dependency.
tags: [platform, load-balancer, iap, tls]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
---

# Edge load balancer constraints

Decided 2026-08-25 (M1). These four constraints are load-bearing; changing any
one silently breaks the edge. Recorded in DECISIONS.md and enforced by the
terraform layout.

## The constraints

1. **Certificate Manager for wildcard TLS.** `google_compute_managed_ssl_certificate`
   cannot do the wildcard-plus-map pattern the platform needs (`*.<domain>` and
   bare `<domain>` in one certificate map).
2. **EXTERNAL scheme, never EXTERNAL_MANAGED.** IAP simply does not work on
   EXTERNAL_MANAGED load balancers.
3. **Serverless NEGs reference `local.*_service_name` string literals** shared
   with the Cloud Run services — never `google_cloud_run_v2_service.*.name`.
   Referencing the resource creates a circular dependency between the IAP
   audience and the backend service ID.
4. **The HTTPS proxy `depends_on` both certificate map entries.** Attaching a
   zero-entry map fails.

## Context

The URL map splits `api.<domain>` to the API backend and everything else to
the router backend, each with IAP enabled — the
The IAP front door depends on this edge
shape. LB health probes to `/healthz` are the only traffic allowed through
without an identity, by GCP platform design.

---
type: Concept
title: Router service
description: The mycelia-router Cloud Run service — wildcard host parsing, static serving from GCS, container proxying, and access logging.
tags: [platform, router, cloud-run, gcs]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# Router service

The router (`router/`, Go 1.25 module `mycelia/router`) runs as Cloud Run
service `mycelia-router` with min 1 / max 2 instances and ingress restricted
to the internal load balancer. Its service account holds exactly one role:
GCS objectViewer on the deploys bucket.

## Static serving

The router parses the project label from the request host (`parseProjectHost`)
and serves the corresponding prefix from the private deploys bucket
(`<project>-mycelia-deploys`, UBLA and public-access prevention enforced).
Directory requests resolve to `/index.html`; any object read error returns 404
with a log line — `download <key>: <err>` distinguishes "file missing" from
"GCS outage or IAM broken".

## Container proxying

For container projects the router reverse-proxies to internal-ingress
[per-app Cloud Run services](/domains/platform/container-deploy-pipeline.md)
using a hand-rolled proxy: a per-app-host SA identity token, all inbound auth
material stripped, upstream redirects never followed. Reaching internal apps
requires a Serverless VPC Access connector with ALL_TRAFFIC egress and a
private DNS zone mapping `run.app` to the `private.googleapis.com` VIP
(199.36.153.8/30). The same proxy pattern serves
previews by resolving the live preview
deployment doc's `url` — the router never reconstructs service names.

## API proxy and access logs

The router also proxies `/api/*` from the web console to the API with its own
SA identity token; that path requires a browser-session IAP assertion and
strips inbound `Authorization`. Every app request passes through the accesslog
middleware (buffered channel, single writer goroutine, drop-on-full), which
feeds [request logs](/domains/operations/request-logs.md) — the hot path never
blocks on logging. The web console served by the router is documented
separately in the (not yet written)
[web console](/domains/platform/web-console.md) concept.

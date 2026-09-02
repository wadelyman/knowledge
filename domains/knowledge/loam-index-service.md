---
type: Concept
title: loam-index service
description: The Python/FastAPI + turbovec vector sidecar — internal-only, single writer, GCS snapshots with model and dimension sidecar.
tags: [knowledge, loam-index, turbovec, embeddings]
status: draft
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
---

# loam-index service

Draft: implemented and locally green (pytest, 11 tests — the repo's third test
runner, contained to `index/`), but the E3 live drills (ingest E2E,
reindex-verify, filtered recall, restart drill) are still pending.

## Why a separate service

turbovec has no Go bindings, and CGo would break the api/router distroless
static builds — so the vector index is a Python FastAPI sidecar (`index/`),
the platform's only Python service. It runs as Cloud Run `loam-index` with
internal ingress, min=max=1 (a deliberate single writer), and is never on the
load balancer: no NEG, no backend, no IAP. The
[API](/domains/platform/api.md) invokes it with a per-host identity token —
the same pattern the router uses for container apps.

## Index semantics

- Persistence is full-write `write()`/`load()`, not incremental sync.
- Allowlisted search KeyErrors on absent ids; the server intersects first.
- `/add` skips duplicates.
- Snapshots live in a GCS bucket with a model+dimension sidecar, so an
  embedding-model change is detected rather than silently mixing vector
  spaces. The service account holds objectAdmin on its snapshot bucket and
  `aiplatform.user` for Vertex AI embeddings.
- The design doc's "crash-safe at any byte" claim holds only for the local
  atomic rename, not the GCS upload.

## The index is disposable

It is never the source of truth for anything: if the snapshot is lost or the
embedding model changes, rebuild from the git bundle. `mycelia okf reindex`
must be a routine operation, not an incident. Chunk ids are uint64 from a
counter (never a hash — silent collision is the worst failure mode in a
retrieval system), with the id-to-concept map in Firestore.

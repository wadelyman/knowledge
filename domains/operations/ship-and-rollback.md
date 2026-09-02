---
type: Playbook
title: Ship and roll back
description: The only supported deploy path — commit, run deploy.sh, verify — and how to roll back to a previous image tag.
tags: [operations, deploy, rollback, playbook]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
verified: { by: human:wlyman, at: 2026-09-01T00:00:00Z }
---

# Ship and roll back

## Ship a change

```bash
git commit -am "..."        # image tag = git short SHA; uncommitted work won't deploy
./deploy.sh                 # build + push + terraform apply (review the plan)
```

The [deploy pipeline](/domains/platform/deploy-pipeline.md) tags images with
the git short SHA, so committing first is mandatory. After deploy, verify with
the three curl checks in SETUP.md §6 — most importantly, an unauthenticated
`https://api.<domain>/healthz` must return 302, not `ok`.

## Roll back

Redeploy the previous image tag, found in git history:

```bash
PREV=$(git rev-parse --short HEAD~1)
terraform -chdir=terraform apply -var-file=prod.tfvars \
  -var="router_image=${AR}/router:${PREV}" \
  -var="api_image=${AR}/api:${PREV}"
```

Old image SHAs remain in Artifact Registry unless manually deleted, so any
previous revision is redeployable.

## Known footguns

- **Never run a bare `terraform apply`** without `-var-file=prod.tfvars` —
  the image variables revert to defaults and the services reset to the M1
  "It's running!" placeholder. prod.tfvars pins both images to prevent
  recurrence; if it happens, re-apply with the pins.
- If a deploy breaks after a terraform change, `git stash` the tf change,
  `./deploy.sh` back to the last good state, then investigate.
- First apply on a fresh project can 403 on API propagation — re-run
  `./deploy.sh`; ordering is declared, propagation is eventually consistent.

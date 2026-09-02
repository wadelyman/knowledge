---
type: Concept
title: CI workload identities (WIF)
description: GitHub Actions deploy via workload identity federation — per-project CI service accounts, repo-scoped principalSets, provider-side branch restrictions.
tags: [identity, ci, wif, github-actions]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
stale_after: 2026-08-01T00:00:00Z
sources:
  - id: wli-docs
    resource: https://cloud.google.com/iam/docs/workload-identity-federation
    title: Workload identity federation documentation
    author: team:platform
  - id: gh-oidc-docs
    resource: https://docs.github.com/en/actions/concepts/security/openid-connect
    title: OpenID Connect in GitHub Actions
    author: team:platform
---

# CI workload identities (WIF)

M6 (2026-08-30) gave GitHub Actions a deploy path with no long-lived secrets:
GitHub OIDC → workload identity federation → per-project CI service account →
ID token → IAP. Stale-flagged because the IAM propagation timings below drift
and should be re-measured periodically.

## Why per-project service accounts

After WIF token exchange, the impersonated ID token carries no repo or ref
claims, so a shared deploy SA could not be authorized per project. Each linked
project gets `ci-<project>`, whose token identity maps 1:1 to the project; the
API grants that identity deploy-only access to its own project, and every
other route stays owner-only.

## Where the restrictions live

- **Repo scoping** comes from the member itself:
  `principalSet://…/attribute.repository/<owner>/<repo>`. SA allow policies
  reject principal-attribute conditions ("undeclared reference to
  'attribute'"), so conditions were abandoned in favor of the principalSet.
- **Main-only** lives in the WIF provider's `attribute_condition` (pool-wide);
  PR runs and side branches die at token exchange with "rejected by the
  attribute condition".
- The pool `mycelia-github` and provider additionally restrict
  `repository_owner` to the configured GitHub owner as defense in depth.

## The PR lane (M10)

Container previews needed PR builds, so M10
added a second provider `github-pr` that drops only the ref clause, plus a
preview-only `ci-pr-<project>` SA admitted on non-default branches. The
load-bearing guarantee stays at token exchange; the server-side 403 on main is
defense in depth. Fork PRs are denied for free — the fork's `repository` claim
never matches the repo-scoped principalSet.

## Operational gotchas (E2E-found)

- The Actions auth step needs `id_token_include_email: true` — IAP 401s ID
  tokens without an email claim.
- The principalSet references the WIF **pool** name while workflow YAML needs
  the **provider** name; the API splits `CI_WIF_PROVIDER` at `/providers/`
  and is startup-fatal on misconfig.
- First deploy after linking can fail on IAM propagation lag (~1–5 minutes);
  rerun the Action. Link's create-SA → grant-IAP sequence races new-principal
  propagation — retry the link. Deploys from CI use the same
  deploy pipeline image tagging rules.

---
type: Reference
title: OKF bundle format (v0.2)
description: The on-disk knowledge bundle layout Loam ingests — index/log furniture, domain directories, concept frontmatter, and preservation rules.
tags: [knowledge, okf, format, reference]
generated: { by: loam-bootstrap/1.0, at: 2026-09-02T00:00:00Z }
sources:
  - id: okf-pkg
    resource: api/okf/ (mycelia repo)
    title: api/okf — parser, linter, chunker
    author: team:platform
  - id: design-doc
    resource: ~/.opencode/2026-09-02-mycelia-m11-loam-okf-knowledge-design.md
    title: M11 Loam OKF knowledge design
    author: team:platform
---

# OKF bundle format (v0.2)

An OKF bundle is a git repository of markdown files. The directory structure
is domain-independent: a root `index.md` and `log.md` (furniture, not
concepts), then `domains/<domain>/` with one subdirectory per knowledge
domain, each with its own frontmatter-free `index.md` orientation file and one
markdown file per concept. This bundle follows that shape.

## Frontmatter rules

- Every concept carries YAML frontmatter with a required `type`; `title`,
  `description`, and `tags` are the indexing surface — they are prepended to
  every embedded chunk so mid-document chunks know their document.
- The bundle-root `index.md` carries exactly one frontmatter key,
  `okf_version: "0.2"`. Frontmatter in any other `index.md` is a lint error
  (`E_INDEX_FRONTMATTER`), and `index.md`/`log.md` used as a concept is
  `E_RESERVED_FILENAME`.
- Timestamps are ISO 8601 with an explicit UTC offset; actors take the forms
  `<producer>/<version>`, `human:<id>`, or `process:<id>`.
- `verified` accepts a bare mapping and normalizes to a one-element list;
  absent `status` means `stable`.

## Preservation is not optional

Consumers SHOULD preserve unknown frontmatter keys when round-tripping and
MUST NOT reject documents with unrecognized fields. The parser preserves
unrecognized keys as raw source lines rather than re-marshaling YAML (yaml.v3
normalizes formatting), which is how parse → serialize achieves byte identity.
A projection that drops unknown keys silently destroys them the first time an
agent rewrites a file — called out in the design as the single most likely
correctness bug in the system.

## Links

Markdown links between concepts are edges; the surrounding sentence is the
relation's only evidence. A link whose target does not exist is not malformed
— it is a gap signal (`W_DANGLING_LINK`, a warning permanently) that feeds the
research queue. The linter also scans for credential shapes (`E_SECRET`) so
pasted tokens never reach the git repo.

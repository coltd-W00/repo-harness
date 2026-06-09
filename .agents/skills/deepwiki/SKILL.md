---
name: deepwiki
description: >-
  Generate or refresh a DeepWiki-style Markdown wiki for the repo under
  docs/wiki/ (overview + per-area pages with architecture diagrams, summaries,
  and links to sources). Use when onboarding a repo, when the structure or tech
  stack changed, or when wiki pages drift from the current tree.
---

# DeepWiki

Standalone skill (under trial — not yet wired into the Harness workflow). It
generates a DeepWiki-style Markdown wiki: an overview page plus one page per
**Area**, each with an architecture diagram (Mermaid), a summary, and links to
the real source paths — mirroring what Devin's hosted DeepWiki produces, but as
versioned files in the repo.

DeepWiki is a _consumer_ of `docs/KNOWLEDGE_INDEX.md` (the Orient map) and the
source-of-truth Hierarchy: read them as a frame, then expand each Area into its
own page. Do **not** duplicate the index or glossary — link to them. On
conflict, the source in the Hierarchy wins.

## Input (read before running)

- `docs/KNOWLEDGE_INDEX.md` — Purpose, Top-Level Structure, Key Technologies,
  Key Concepts (the page frame + Area list). If it is stale, refresh it first.
- `docs/ARCHITECTURE.md`, `docs/GLOSSARY.md`, `README.md` — for summaries,
  diagrams, and terms (link the glossary, don't copy it).
- The real tree: `git ls-files` (source of truth for file lists + links).
- `.devin/wiki.json` if present: honor `pages` (create exactly those, with
  `title`/`purpose`/`parent`) and `repo_notes` (priority/emphasis).

## Steps

1. **Orient — fix the page list.** If `.devin/wiki.json` has `pages`, create
   exactly those. Otherwise cluster the `KNOWLEDGE_INDEX.md` Top-Level Structure
   into Areas (one page each). Every page must map to a real path.
2. **Author `docs/wiki/README.md`:** (a) Purpose, 1–3 sentences matching the
   index; (b) a `mermaid` architecture diagram of how the Areas relate; (c) a
   "Pages" table linking every child page; (d) a "Sources" list linking the
   top-level directories.
3. **Author each `docs/wiki/<area>.md`:** summary (the Area's purpose); "Key
   files" listing + linking real source paths; flow/interactions (Mermaid when
   useful); "Related concepts" linking `docs/GLOSSARY.md`; a "Sources" footer of
   repo-relative links; and a `[← Wiki](./README.md)` back-link.
4. **Cross-link:** the index links to every page; every page links back to the
   index; reference `KNOWLEDGE_INDEX.md` / `GLOSSARY.md` instead of copying.
5. **Format** (repo uses Prettier `proseWrap: always`):

   ```bash
   npx prettier --write "docs/wiki/**/*.md"
   ```

## Verify (mechanical gate — each must pass)

```bash
test -f docs/wiki/README.md
npx prettier --check "docs/wiki/**/*.md"
! grep -rIn -e 'TODO' -e '<area>' -e 'TBD' docs/wiki
# no broken intra-wiki .md links:
! grep -rhoE '\]\(([^):#]+\.md)' docs/wiki | sed -E 's/^\]\(//' \
  | while read -r l; do [ -e "docs/wiki/$l" ] || echo "BROKEN $l"; done | grep .
```

## Done when

`docs/wiki/README.md` plus every Area page exist, the index links to all pages
and every page links back, no placeholders or broken intra-wiki `.md` links
remain, and Prettier is clean.

<!--
Status: standalone/trial. Once validated, promote into the Harness workflow by
moving the canonical procedure to `_harness/skills/generate-deepwiki.md`, adding
a row to the `_harness/04-SKILLS.md` registry, and wiring an on-demand step into
`_harness/01-WORKFLOW.md` (GĐ2/GĐ6) — mirroring `generate-knowledge-index`.
-->

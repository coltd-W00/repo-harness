---
name: deepwiki
description: >-
  Generate a real DeepWiki for any repository: a navigable Markdown wiki under
  docs/wiki/ with a home/overview page, an architecture diagram, and one page
  per major component — each with diagrams, key files linked to source, public
  interfaces, and dependencies. Use to onboard or document an unfamiliar
  codebase, or to refresh the wiki after the structure changes.
---

# DeepWiki Generator

Standalone, repo-agnostic skill. It produces a **DeepWiki** for the current
repository — the same shape Devin's hosted DeepWiki gives you (architecture
diagrams, summaries, links to sources, navigable pages) but committed as
Markdown under `docs/wiki/`. No dependency on any project-specific files.

A real DeepWiki is **derived from the code, not guessed**. Every claim, file
link, and diagram must reflect what is actually in the tree at generation time.
Prefer linking source over paraphrasing it.

## 0. Steering (optional)

If `.devin/wiki.json` exists at the repo root, honor it (this is Devin's native
wiki-steering format):

- `pages` — if present, create **exactly** these pages (each has `title`,
  `purpose`, optional `parent` for hierarchy). Skip auto page-planning.
- `repo_notes` — free-form guidance on priorities / emphasis. Always respect.

If there is no `.devin/wiki.json`, plan the pages yourself in step 2.

## 1. Discover (build a mental model of the repo)

Gather facts mechanically before writing anything:

- **Inventory:** `git ls-files` for the real tree. Identify top-level
  directories and group source files by language/extension.
- **Stack & tooling:** detect manifests and derive the tech stack + commands —
  e.g. `package.json` (scripts), `pyproject.toml`/`requirements.txt`,
  `Cargo.toml`, `go.mod`, `pom.xml`/`build.gradle`, `Gemfile`, `composer.json`,
  `Dockerfile`, `docker-compose.yml`, CI under `.github/workflows/`.
- **Entry points:** `main`/`index`/`cmd/`/`app`/`server` files, CLI definitions,
  HTTP route registrations, exported package roots.
- **Components:** the cohesive subsystems — usually top-level packages/modules
  (e.g. `frontend/`, `backend/`, `api/`, `core/`, `services/<x>`, a crate, a Go
  package). One wiki page per major component.
- **Relationships & data:** how components call/import each other; key data
  models/schemas/migrations; external integrations (DBs, queues, 3rd-party
  APIs).
- **Read the README** and any existing `docs/` to anchor the project's purpose —
  but verify against the code; code wins on conflict.

## 2. Plan pages

- Home page: `docs/wiki/README.md`.
- One page per major component from step 1 (or exactly the `.devin/wiki.json`
  `pages`). Use clear filenames, e.g. `docs/wiki/backend.md`,
  `docs/wiki/frontend.md`, `docs/wiki/data-model.md`. Keep the set focused —
  merge trivial dirs, split only genuinely large subsystems.

## 3. Author the home page (`docs/wiki/README.md`)

- **Overview** — 2–4 sentences: what the project does and who/what it's for.
- **Architecture diagram** — a `mermaid` `flowchart` (or `graph`) showing the
  major components and how they connect (requests, data, deploy boundaries).
- **Tech stack** — languages, frameworks, datastores, key libs (from step 1).
- **Getting started** — the detected install / build / run / test commands.
- **Pages** — a table linking every component page with a one-line purpose.
- **Repository map** — top-level directories with one-line descriptions, each
  linked to the directory.

## 4. Author each component page (`docs/wiki/<component>.md`)

Every component page includes:

- **Summary** — the component's responsibility and boundaries.
- **Key files** — the most important files/modules, each a link to the real path
  (optionally `path#Lstart-Lend` for a specific symbol).
- **Internals** — a `mermaid` diagram when it clarifies: `flowchart` for module
  structure, `sequenceDiagram` for an important request/operation flow, or
  `erDiagram`/`classDiagram` for data models.
- **Public interface** — exported functions/types, HTTP/RPC endpoints, CLI
  commands, or config the rest of the system depends on.
- **Dependencies** — what it depends on (in) and what depends on it (out),
  linking sibling wiki pages.
- **Footer** — a `[← Home](./README.md)` back-link.

## 5. Cross-link & format

- Home links to every page; every page links back to Home; components link to
  the sibling pages they reference. Use relative `./*.md` links.
- If the repo has a Markdown formatter (e.g. Prettier config present), run it on
  `docs/wiki/`. Otherwise skip formatting — do **not** add tooling the repo
  doesn't already use.

## Verify (mechanical gate — each must pass)

````bash
# home exists
test -f docs/wiki/README.md

# at least one architecture diagram was produced
grep -rIlq '```mermaid' docs/wiki

# no placeholders left behind
! grep -rIn -e 'TODO' -e 'TBD' -e '<component>' -e 'FIXME' docs/wiki

# every planned page is reachable from Home (no orphans)
for f in docs/wiki/*.md; do
  b=$(basename "$f"); [ "$b" = "README.md" ] && continue
  grep -q "($b\|/$b\|./$b)" docs/wiki/README.md || { echo "ORPHAN $b"; exit 1; }
done

# no broken intra-wiki .md links
! grep -rhoE '\]\(([^):#]+\.md)' docs/wiki | sed -E 's/^\]\(//' \
  | while read -r l; do [ -e "docs/wiki/$l" ] || echo "BROKEN $l"; done | grep .
````

## Done when

`docs/wiki/README.md` plus every planned component page exist; the home page has
an architecture diagram and links to every page; every page links back; there
are no placeholders, orphan pages, or broken intra-wiki links; and the facts,
file links, and diagrams reflect the current tree.

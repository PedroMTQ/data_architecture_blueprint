# Architecture blueprint slides (Slidev)

Presentation deck distilled from [`../docs/`](../docs/). ~43 slides across medallion layers, diagrams, governance, and APIs.

## Local development

From the repo root:

```bash
cd slides
npm install
npm run dev
```

Or: `npm run slides:dev` from the repository root (after `npm install` in `slides/` once).

Opens the Slidev dev server with live reload.

## Export PDF

```bash
cd slides
npx playwright install chromium   # first time only
npm run export
```

Output: `slides/dist/data_architecture_blueprint.pdf`

On push to `main`, CI also exports the PDF and publishes it alongside the HTML deck (see [`.github/workflows/publish-docs.yml`](../.github/workflows/publish-docs.yml)).

## Build static site

```bash
npm run slides:build
```

Output: `slides/dist/` — deployed with docs to GitHub Pages at:

- **HTML:** https://pedromtq.github.io/data_architecture_blueprint/slides/
- **PDF:** https://pedromtq.github.io/data_architecture_blueprint/slides/data_architecture_blueprint.pdf

## Structure

| Path | Content |
|------|---------|
| `slides.md` | Entry + section imports |
| `pages/00-intro.md` | Cover, agenda, principles |
| `pages/01-organization.md` | S3, data contract, logical org |
| `pages/02-high-level.md` | Full pipeline diagram |
| `pages/03-bronze.md` | Landing → Bronze |
| `pages/04-silver.md` | Bronze → Silver |
| `pages/05-clinical.md` | openEHR / OMOP |
| `pages/06-gold.md` | Serving, OMOP, APIs |
| `pages/07-linking.md` | Entity linking, crypto-shred, cascade |
| `pages/08-ops.md` | Observability, stack, RBAC/ABAC |
| `public/diagrams/` | Copies of [`../diagram/`](../diagram/) — run `npm run sync-diagrams` after edits |

## Diagrams

Export SVGs to [`../docs/assets/diagrams/`](../docs/assets/diagrams/) (gold/OMOP) and/or [`../diagram/`](../diagram/). Then run `npm run sync-diagrams` to copy into `public/diagrams/`.

Large SVGs on a slide: use `{.diagram}` and frontmatter `class: diagram-slide` (see `styles/index.css`). Smaller in-slide diagrams: `{.diagram-sm}`. Tighter cap: add `diagram-sm` on a `diagram-slide` or lower `max-height` in CSS.

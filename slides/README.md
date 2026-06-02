# Architecture blueprint slides (Slidev)

Presentation slides for  [`../docs/`](../docs/). ~35 slides across medallion layers, diagrams, governance, and APIs. Uses the [Seriph](https://sli.dev/themes/gallery.html#seriph) theme (`@slidev/theme-seriph`).

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

Requires `playwright-chromium` (listed in `package.json` devDependencies).

```bash
cd slides
npm install
npx playwright install chromium   # first time only; use --with-deps on fresh Linux/CI
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

## Diagrams

Slides reference the same SVGs as MkDocs via a symlink: `pages/diagrams` → `docs/assets/diagrams`. Use `./diagrams/<name>.svg` in slide markdown. Edit diagrams once in `docs/assets/diagrams/` — docs and slides stay in sync.

Large SVGs: set frontmatter `class: diagram-slide`. Tighter cap: lower `max-height` in `.slidev-layout.diagram-slide img` in CSS.

Draw.io SVGs use `light-dark()` for OS dark mode; slides force `color-scheme: light` on diagram images so backgrounds stay white regardless of system theme.

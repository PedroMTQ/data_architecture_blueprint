# Conceptual data architecture blueprint

This blueprint describes a data platform architecture for a healthcare research institute across the medallion layers (Bronze, Silver, Gold), covering ingestion, processing, serving, observability, and access control.

- [Documentation](https://pedromtq.github.io/data_architecture_blueprint/)
- [Slides (HTML)](https://pedromtq.github.io/data_architecture_blueprint/slides/)
- [Slides (PDF)](https://pedromtq.github.io/data_architecture_blueprint/slides/data_architecture_blueprint.pdf)

## Documentation site

Browse the full blueprint on **GitHub Pages** (after enabling Pages → **GitHub Actions** in repo settings):

**https://pedromtq.github.io/data_architecture_blueprint/**

### Local preview (MkDocs)

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Open http://127.0.0.1:8000 — live reload on edits under `docs/`. Click any diagram to open the lightbox (scroll/pinch to zoom, drag to pan).

### Slides (Slidev)

```bash
cd slides && npm install && npm run dev
```

Or from the repo root: `npm run slides:dev` (after installing dependencies in `slides/`).

Published at **https://pedromtq.github.io/data_architecture_blueprint/slides/** — PDF at [`slides/data_architecture_blueprint.pdf`](https://pedromtq.github.io/data_architecture_blueprint/slides/data_architecture_blueprint.pdf) (built on every push via GitHub Actions). See [`slides/README.md`](slides/README.md).

### Source files

| Topic | File |
|-------|------|
| [S3 Data Lake Hierarchy & Organizational Strategy](docs/s3_data_lake_hierarchy_organizational_strategy.md) | `docs/s3_data_lake_hierarchy_organizational_strategy.md` |
| [Data ingestion & bronze layer](docs/data_ingestion_bronze_layer.md) | `docs/data_ingestion_bronze_layer.md` |
| [Data processing & silver layer](docs/data_processing_silver_layer.md) | `docs/data_processing_silver_layer.md` |
| [Data serving & gold layer](docs/data_serving_gold_layer.md) | `docs/data_serving_gold_layer.md` |
| [Observability, Metrics & Operational Health](docs/observability_metrics_operational_health.md) | `docs/observability_metrics_operational_health.md` |
| [Access control](docs/access_control.md) | `docs/access_control.md` |
| [Infrastructure Stack Summary](docs/tech_stack.md) | `docs/tech_stack.md` |

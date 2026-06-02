# Data architecture blueprint

This blueprint describes the data platform architecture across the medallion layers (**Bronze**, **Silver**, **Gold**), covering ingestion, processing, serving, observability, and access control.

## How to read this site

| Section | Topics |
|---------|--------|
| [S3 hierarchy & strategy](s3_data_lake_hierarchy_organizational_strategy.md) | Physical vs logical organization, naming, lifecycle |
| [Bronze — ingestion](data_ingestion_bronze_layer.md) | Ingestion paths, validation, WORM vault |
| [Silver — processing](data_processing_silver_layer.md) | openEHR, identity bridge, quarantine |
| [Gold — serving](data_serving_gold_layer.md) | OMOP, FHIR, sandboxes, Submission API |
| [Observability](observability_metrics_operational_health.md) | Pipeline health, data quality, storage metrics |
| [Access control](access_control.md) | RBAC/ABAC, pseudonymization, crypto-shredding |
| [Infrastructure stack](tech_stack.md) | Proposed tools and technologies |

## Design principles

- **Security by design** — WORM immutability, per-file DEK encryption, pseudonymization at the boundary
- **GDPR by architecture** — Crypto-shredding and consent withdrawal cascade via the identity bridge
- **Decouple persistence from analytics** — quality checked data and openEHR in Silver, derived data, data marts, and OMOP in Gold
- **Logical over physical organization** — Flat Bronze/Silver; structure deferred to Gold and metadata

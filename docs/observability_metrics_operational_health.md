
## Observability, metrics & operational health

Row-level entity linking (Identity Bridge, openEHR, OpenMetadata, and GDPR cascade) is documented with a diagram in the [Gold layer — Row level entity linking](data_serving_gold_layer.md#row-level-entity-linking) section.

### Access Control & cryptographic erasure

![Crypto-shredding and DEK erasure](assets/diagrams/crypto_shredding.svg){ width="100%" }

* **Ingestion isolation:** Tenants retain write-only privileges to their specific landing perimeter via strict S3 IAM policies; they possess zero direct read permissions over any of the S3 buckets.
* **Crypto-shredding protocol:** To reconcile Write-Once-Read-Many (WORM) immutability with GDPR's Article 17 ("Right to be Forgotten"), the system leverages native MinIO Enterprise SSE-KMS. DEKs are generated per file in-memory, and the raw payload is encrypted natively by MinIO.
* **Erasure execution:** To fulfill a GDPR deletion request, an administrative process calls the KMS API to destroy the specific file DEK (e.g., Clinnova’s consent management system). This adds a new event in the ledger with the status: 'CRYPTO_SHRED_COMPLETED'. Purging the key renders the underlying immutable file irrecoverable, achieving legal compliance without breaking the physical storage chain of custody. (Note: This deletion event must subsequently trigger downstream deletions across the Silver and Gold medallion layers).***


### Pipeline observability (Airflow)

Airflow is the primary dashboard for pipeline health. Every DAG execution emits structured logs with task-level success/failure states, duration metrics, and retry counts. For production deployments, metrics should be exported to a centralised monitoring stack to enable cross-pipeline visibility and alerting beyond what Airflow's UI provides natively.

### Data quality observability

Data quality is automatically tracked at two distinct points. Great Expectations audits openEHR compositions at the Silver → Gold boundary, surfacing distribution anomalies, missing field rates, and schema drift.
The OHDSI Data Quality Dashboard (DQD) runs native SQL assertions over completed OMOP schemas, validating cross-table clinical plausibility against international OHDSI conformance standards. Additional data quality metrics could be exposed via e.g., Pydantic EDC validation, and 2+. the gold processing layers.

### Storage observability

MinIO AIStor exposes native S3-compatible metrics via a Prometheus endpoint. These cover bucket-level storage consumption, object count, request throughput, and error rates. Lifecycle policy execution (transitions to warm/cold storage) should be tracked to validate that archival policies are running as expected and that storage costs remain predictable.

### Unified data metrics visibility via OpenMetadata

[OpenMetadata](https://docs.open-metadata.org/v1.12.x/how-to-guides/data-quality-observability/alerts-notifications) provides metrics aggregation and an alerting system; in fact it also offers direct integration of Great Expectations audits and Airflow pipelines. For custom metrics (e.g., DQD), we could e.g., expose metrics to a relational database which can then be served to OpenMetadata.

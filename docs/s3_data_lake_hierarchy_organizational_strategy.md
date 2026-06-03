
## S3 Data Lake Hierarchy & Organizational Strategy

This platform decouples physical data storage from logical organization across the entire ingestion and processing layers (bronze  and silver).

* **Physical tenant isolation (Bronze & Silver):** At both the bronze and silver layers, the object storage enforces a flat, tenant-specific bucket (e.g., s3://<tenant_id>/bronze/ and s3://<tenant_id>/silver/). This separation allows for bucket-wide security policies, reducing the risk for human IAM policies misconfiguration.
* **Processing (Silver):** The silver layer acts as a standardization layer. Files are validated, cleaned, and compressed, but they remain a flat pool of logically independent assets under the tenant bucket. The platform still doesn’t enforce any hierarchical data organization (by project, domain, or cohort) at this stage.
* **Downstream materialization (Gold):** All structural grouping (whether by data domain, research project, or cohort) is deferred to the Gold layer, which will be described in the [respective section](data_serving_gold_layer.md). Here the file storage system is human readable and more malleable to allow for fine-grained IAM configuration.


## Design principles

- **Security by design** — WORM immutability, per-file DEK encryption, pseudonymization at the boundary
- **GDPR by architecture** — Crypto-shredding and consent withdrawal cascade via the identity bridge
- **Decouple persistence from analytics** — quality checked data and openEHR in Silver, derived data, data marts, and OMOP in Gold
- **Logical over physical organization** — Flat Bronze/Silver; structure deferred to Gold and metadata


## End-to-end pipeline

![End-to-end medallion pipeline](assets/diagrams/full_pipeline.svg){ width="100%" }

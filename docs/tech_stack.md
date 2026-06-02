
# Infrastructure Stack Summary

## Storage & Database

* **Storage engine:** [Distributed MinIO](https://github.com/minio/minio/blob/master/docs/distributed/README.md) or ([MinIO AIStor](https://www.min.io/product/aistor)) - High-performance, on-prem S3-compatible object storage. Enterprise solution (AIStor) provides many of the necessary compliance and security features (compliance and governance modes, KMS, fined-tuned IAM).
* **PostgreSQL:** The relational engine for the audit ledgers, metadata catalogues, the openEHR clinical data repository (managed by EHRBase) and the multi-tenant OMOP CDMs / Sandboxes (likely separate clusters). 
* **OLAP DB:** we could replace PostgreSQL with an analytic driven DB (TBD)
* **Staging buffer:** Apache Kafka - Handles small-file streaming ingestion and batching.
* **Elasticsearch:** OpenMetadata’s search engine

### Storage notes

In the landing, bronze, and silver buckets, there is no physical organization of the files; it is done logically through the audit ledger. In the gold layer, to promote human readability and easier to manage security policies, we organize the data physically (s3://gold__common/ and s3://gold__my_group/). We also enforce role-based, attribute-based and prefix-matching IAM policies for fine-grained control.

The audit ledger is a persistent append-only transactional ledger which also preserves payload metadata (could be potentially split into audit/metadata).

Regarding storage modes:

- landing is unlocked
- bronze is locked, immutable, compliance mode, WORM. This serves as the source of truth of all downstream assets
- silver is governance mode. This serves as the storage for versioned, standardized unstructured assets (Omics, Bio-imaging, and Multimedia) and supports deterministic pipeline re-runs.
- gold uses prefix-based physical layout (`gold__common/`, `gold__my_group/`) for readability and IAM policy alignment

## Gateways, identity & security

* **Ingestion gateway:** FastAPI - Stateless REST microservice managing tenant authentication, pseudonymization, and payload routings.
* **Serving gateway:** FastAPI - Downstream delivery for researchers: small assets via API, large assets (e.g., omics) via time-bound pre-signed URLs; also fronts analytical tooling access patterns (e.g., ATLAS/Athena) where applicable.
* **Submission gateway:** FastAPI - Accepts researcher-derived data into Gold with automated provenance and lineage registration in OpenMetadata; enforces identity-bridge linkage for GDPR cascade (see [Gold — serving](data_serving_gold_layer.md)).
* **Pseudonymization (PSDS):** Clinnova’s PSDS service - Replaces raw PII with secure research pseudonyms at ingestion (bypassed if the source node has already pseudonymized data).
* **Secrets & Key Management service (KMS):** [MinIO’s KMS](https://docs.min.io/kms/) - SSE-KMS with per-file Data Encryption Keys (DEKs); supports crypto-shredding on bronze WORM objects for GDPR erasure without breaking the physical chain of custody.
* **Identity:** MPI + Splink/Deterministic rules - Entity resolution and patient identification.

## Orchestration & Processing

* **Apache Airflow:** The central orchestrator managing event-driven Directed Acyclic Graphs (DAGs) across the platform — triggers per-file bronze pre-processing, Silver pipelines (via MinIO webhooks, containerized workers, and EHRbase API calls), dbt model execution, data validation tasks, and other cross-layer processes.
* **EHRbase (openEHR CDR):** An open-source Clinical Data Repository that implements the openEHR specifications. It functions strictly as a write-heavy, source-faithful semantic persistence layer, exposing standard APIs that automatically execute real-time validation of inbound payloads against clinical templates.
* **Clinical Knowledge Manager (CKM) & Archetype Designer:** Low-code modeling and governance frameworks utilized by clinical data managers to design, import, version, and compile standardized openEHR archetypes and templates.
* **Pydantic:** The runtime validation engine deployed within the ingestion microservices to enforce strict row-level structural types and schema constraints on incoming data payloads.

## Gold, Analytics & Quality

* **dbt (Data Build Tool):** The core transformation engine executing scheduled, version-controlled SQL pipelines to map granular openEHR structures into the flattened OMOP canonical tables.
* **OMOPHub:** Generates vocabulary crosswalks and term-resolution mappings for openEHR → OMOP pipelines; outputs are validated human-in-the-loop and loaded into PostgreSQL as static dbt sources (see [Gold — serving](data_serving_gold_layer.md)).
* **OHDSI ATLAS & Athena:** The standardized analytical suite. Athena provides the unified interface to download and manage standard medical ontologies (e.g., SNOMED, LOINC), while ATLAS serves as the web-based visual cohort builder, phenotyping engine, and population-level statistical analyzer for downstream researchers.
* **OHDSI Data Quality Dashboard (DQD):** The validation engine that runs native SQL assertions over completed OMOP schemas to verify cross-table clinical plausibility and compliance with international OHDSI standards.
* **Great Expectations:** The data quality auditing framework used to audit data (including openEHR → Gold checks), detect data distribution anomalies, and track other data qualitys metrics.
* **OpenFHIR (optional):** [OpenFHIR](https://open-fhir.com/) - FHIR API layer generated from EHRbase for clinical interoperability.s
* **MLflow:** Model registry and artifact serving backed by S3-compatible storage for research ML workflows.

## Governance, Metadata & AI

* **OpenMetadata:** The centralized, API-first Knowledge Graph control plane for the platform. It natively hosts a Model Context Protocol (MCP) server for autonomous AI agents, maps unified data lineage (Airflow/dbt pipelines plus researcher uploads via the Submission API), glossary and data-quality hooks, data governance, and acts as a central marketplace for data discovery.

## Metrics & Observability

* **Apache Airflow:** Operational UI for DAG runs, task health, and pipeline troubleshooting (complements metrics backends below).
* **Prometheus:** e.g., used for API gateways and MinIO where instrumented; not for ephemeral jobs (not well supported)
* **Grafana/Superset:** Dashboarding for metrics
* **OpenTelemetry:** Distributed tracing where adopted

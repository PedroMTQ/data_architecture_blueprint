
# Infrastructure Stack Summary

## Storage & Database

* **Storage engine:** [Distributed MinIO](https://github.com/minio/minio/blob/master/docs/distributed/README.md) or ([MinIO AIStor](https://www.min.io/product/aistor)) - High-performance, on-prem S3-compatible object storage. Enterprise solution (AIStor) provides many of the necessary compliance and security features.
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

## Ingestion & Gateway

* **Ingestion gateway:** FastAPI (e.g.,) gateway - Stateless containerized REST microservice managing tenant authentication, automated pseudonymization, and transaction routing.
* **Secrets & Key Management service (KMS):** [MinIO’s KMS](https://docs.min.io/kms/).
* **Identity:** MPI + Splink/Deterministic rules - Entity resolution and patient identification.

## Orchestration & Processing

* **Apache Airflow:** The central orchestrator managing event-driven Directed Acyclic Graphs (DAGs) across the platform — triggers per-file bronze pre-processing, Silver pipelines (via MinIO webhooks, containerized workers, and EHRbase API calls), dbt model execution, data validation tasks, and other cross-layer processes.
* **EHRbase (openEHR CDR):** An open-source Clinical Data Repository that implements the openEHR specifications. It functions strictly as a write-heavy, source-faithful semantic persistence layer, exposing standard APIs that automatically execute real-time validation of inbound payloads against clinical templates.
* **Clinical Knowledge Manager (CKM) & Archetype Designer:** Low-code modeling and governance frameworks utilized by clinical data managers to design, import, version, and compile standardized openEHR archetypes and templates.
* **Pydantic:** The runtime validation engine deployed within the ingestion microservices to enforce strict row-level structural types and schema constraints on incoming data payloads.

## Gold, Analytics & Quality

* **dbt (Data Build Tool):** The core transformation engine executing scheduled, version-controlled SQL pipelines to map granular openEHR structures into the flattened OMOP canonical tables.
* **OHDSI ATLAS & Athena:** The standardized analytical suite. Athena provides the unified interface to download and manage standard medical ontologies (e.g., SNOMED, LOINC), while ATLAS serves as the web-based visual cohort builder, phenotyping engine, and population-level statistical analyzer for downstream researchers.
* **OHDSI Data Quality Dashboard (DQD):** The validation engine that runs native SQL assertions over completed OMOP schemas to verify cross-table clinical plausibility and compliance with international OHDSI standards.
* **Great Expectations:** The data quality auditing framework used to audit data, surface distribution anomalies, and track data metrics.

## Governance, Metadata & AI

* **OpenMetadata:** The centralized, API-first Knowledge Graph control plane for the platform. It natively hosts a Model Context Protocol (MCP) server for autonomous AI agents, maps unified data lineage (controlled and un-supervised), data governance, and acts as a central marketplace for data discovery.

## Metrics & Observability

* **Prometheus:** e.g., used for the API gateways but not for ephemeral jobs (not well supported)
* **Grafana/Superset:** Dashboarding for metrics
* **OpenTelemetry**

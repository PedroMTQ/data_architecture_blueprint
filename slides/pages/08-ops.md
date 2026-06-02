---
class: compact-slide
---

# Infrastructure stack (1/2)

**Storage & databases**

- **MinIO AIStor** - S3-compatible lake (landing, bronze WORM, silver governance, gold buckets)
- **PostgreSQL** - per-layer audit ledgers, EHRbase (openEHR), OMOP CDMs (or OLAP-centric DB), research sandbox schemas
- **Kafka** - small EDC file staging
- **Elasticsearch** - OpenMetadata search index

**Ingestion, identity & security**

- **FastAPI gateways** - ingestion, serving, submission
- **PSDS** - boundary pseudonymization
- **MPI / Splink** - patient identity & linkage
- **MinIO Enterprise KMS** - SSE-KMS, per-file DEKs, crypto-shredding

**Orchestration & Silver processing**

- **Apache Airflow** - bronze pre-processing, silver DAGs, validation orchestration
- **MinIO webhooks** + **container workers** - event-driven, deterministic replays
- **EHRbase**, **CKM**,  **Archetype Designer** - clinical ingest & modelling
- **Pydantic** - data parsing, validationn, and transformation

---
class: compact-slide
---

# Infrastructure stack (2/2)

**Gold, analytics & quality**

- **dbt** - openEHR → OMOP SQL pipelines
- **OMOPHub** - vocabulary crosswalks (HITL)
- **OHDSI Athena** (standard ontologies) & **ATLAS** (data sampling)
- **DQD** - OMOP auditing
- **Great Expectations** - openEHR audits (and other tabular data checks)
- *Optional:* **OpenFHIR** (clinical API)
- **MLflow** (model artifacts on S3 - registry & serving)

**Governance, metadata & AI**

- **OpenMetadata** - catalogue, hybrid lineage (pipelines + Submission API), glossary, quality alerts, **MCP**

**Observability**

- **Airflow** - pipeline health UI · **Prometheus** - MinIO & gateway metrics
- **Grafana / Superset** - dashboards · **OpenTelemetry** - distributed traces (where adopted)




---
class: compact table-dense
---

# Human Roles

| Role | Ingress, Landing & Bronze | Silver | Gold (ABAC enforced where noted) | Control Plane |
| :--- | :--- | :--- | :--- | :--- |
| **Data provider** | W (Landing S3) | NONE | NONE | NONE |
| **Data engineer** | R (Bronze S3, Kafka, Bronze Ledger) | R (EHRbase, Silver S3, Silver Ledger, Metadata Cat.) | W (OMOP deploy, Vocab deploy)<br>R (Identity Bridge, FHIR) | **ADMIN** (OpenMetadata) |
| **Clinical data modeler** | NONE | W (EHRbase templates) | R/W (Vocab mappings) | W (OpenMetadata glossary) |
| **Researcher** | NONE | NONE | R (Identity Bridge, OMOP, FHIR, Vocab) — *ABAC enforced*<br>R/W (Derived S3) — *ABAC enforced* | R (OpenMetadata) |
| **Security & compliance**| R/W (Bronze Ledger crypto-shred) | R (Silver Ledger) | NONE | R/W (KMS key deletion)<br>R (OpenMetadata) |

---
class: compact table-dense
---

# Machine Roles

| Service(s) | Ingress, Landing & Bronze | Silver | Gold (ABAC enforced where noted) | Control Plane |
| :--- | :--- | :--- | :--- | :--- |
| **Landing to bronze** | R/W (Landing S3), W (Bronze S3) <br>R (Kafka)<br>W (Bronze Ledger) | NONE | NONE | R/W (KMS wrap DEK) |
| **Bronze to silver** | R (Bronze S3) | W (EHRbase, Silver S3, Silver Ledger, Metadata Cat.) | NONE | R (KMS unwrap DEK) |
| **Silver to gold** | NONE | R (EHRbase, Silver S3, Silver Ledger, Metadata Cat.) | W (Identity Bridge, OMOP, FHIR)<br>R (Vocab) | W (OpenMetadata) |
| **Ingestion gateway** | W (Landing S3, Kafka, Bronze Ledger) | NONE | NONE | NONE |
| **Serving gateway** | R/W (Bronze Ledger drops) | NONE | R (Identity Bridge, OMOP, FHIR, Derived S3, Vocab) | NONE |
| **Submission gateway** | NONE | NONE | W (Identity Bridge, Derived S3) | W (OpenMetadata un-supervised) |
| **AI agents** | NONE | NONE | R (Identity Bridge, OMOP, FHIR, Derived S3, Vocab) | R (OpenMetadata via MCP) |

---
layout: cover
---

# Thank you

Full documentation:

https://pedromtq.github.io/data_architecture_blueprint/

Slides: `/slides/`

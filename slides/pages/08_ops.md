---
layout: section
class: section-slide optional-slide
transition: fade
---

# Platform & access

---
class: compact-slide optional-slide
---

# Discovery, analytics & observability

- **ATLAS** (OMOP) - cohorts, phenotypes, population-level analytics (ABAC-scoped schemas)
- **OpenMetadata** (read-only governance graph — not operational identity):
  - **Catalogue & search** - Silver/Gold assets, schemas, modality metadata (offloads heavy discovery from internal DBs)
  - **Lineage** - automated from Airflow/dbt; researcher uploads via **Submission API**
  - **Quality & observability** - GX + pipeline integration, alerts/dashboards; DQD OMOP metrics
  - **Governance** - business glossary, tags; discovery marketplace (RBAC/ABAC-scoped)
- **Airflow** - DAGs health
- **MCP** - stewards & AI agents query the graph (e.g. *omics from visit V for patient P?*) without raw SQL
- **GX + DQD** hosting (clinical data quality)
- Ops metrics with Grafana/Superset


---
class: compact-slide optional-slide
---

# Infrastructure stack (1/2)

###### Storage & databases

- **MinIO AIStor** - S3-compatible lake (landing, bronze WORM, silver governance, gold buckets)
- **PostgreSQL** - per-layer audit ledgers, EHRbase (openEHR), OMOP CDMs (or OLAP-centric DB), research sandbox schemas
- **Kafka** - small EDC file staging
- **Elasticsearch** - OpenMetadata search index

###### Ingestion, identity & security

- **FastAPI gateways** - ingestion, serving, submission
- **PSDS** - boundary pseudonymization
- **MPI / Splink** - patient identity & linkage
- **MinIO Enterprise KMS** - SSE-KMS, per-file DEKs, crypto-shredding

###### Orchestration & Silver processing

- **Apache Airflow** - bronze pre-processing, silver DAGs, validation orchestration
- **MinIO webhooks** + **container workers** - event-driven, deterministic replays
- **EHRbase**, **CKM**,  **Archetype Designer** - clinical ingest & modelling
- **Pydantic** - data parsing, validation, and transformation

---
class: compact-slide optional-slide

---

# Infrastructure stack (2/2)

###### Gold, analytics & quality

- **dbt** - openEHR → OMOP SQL pipelines
- **OMOPHub** - vocabulary crosswalks (HITL)
- **OHDSI Athena** (standard ontologies) & **ATLAS** (cohort builder / phenotyping)
- **DQD** - OMOP auditing
- **Great Expectations** - openEHR audits (and other tabular data checks)
- *Optional:* **OpenFHIR** (clinical API) · **MLflow** (model artifacts on S3)

###### Governance, metadata & AI

- **OpenMetadata** - catalogue, hybrid lineage (pipelines + Submission API), glossary, quality alerts, **MCP**

###### Observability

- **Airflow** - pipeline health UI · **Prometheus** - MinIO & gateway metrics
- **Grafana / Superset** - dashboards · **OpenTelemetry** - distributed traces (where adopted)

---
class: compact table-dense optional-slide
---

# Human roles

| Role | Ingress, Landing & Bronze | Silver | Gold (ABAC enforced where noted) | Control Plane |
| :--- | :--- | :--- | :--- | :--- |
| **Data provider** | W (Landing S3) | NONE | NONE | NONE |
| **Data engineer** | R (Bronze S3, Kafka, Bronze Ledger) | R (EHRbase, Silver S3, Silver Ledger, Metadata Cat.) | W (OMOP deploy, Vocab deploy)<br>R (Identity Bridge, FHIR) | **ADMIN** (OpenMetadata) |
| **Clinical data modeler** | NONE | W (EHRbase templates) | R/W (Vocab mappings) | W (OpenMetadata glossary) |
| **Researcher** | NONE | NONE | R bridge & OMOP *(ABAC)* · R FHIR & vocab<br>R/W Derived S3 *(ABAC)* | R (OpenMetadata) |
| **Security & compliance**| R/W (Bronze Ledger crypto-shred) | R (Silver Ledger) | NONE | R/W (KMS key deletion)<br>R (OpenMetadata) |

---
class: compact table-dense optional-slide
---

# Machine roles

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

[Documentation](https://pedromtq.github.io/data_architecture_blueprint/)

[Slides](https://pedromtq.github.io/data_architecture_blueprint/slides/)

[PDF](https://pedromtq.github.io/data_architecture_blueprint/slides/data_architecture_blueprint.pdf)

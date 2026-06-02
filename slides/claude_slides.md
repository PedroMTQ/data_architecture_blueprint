---
theme: default
title: Data Architecture Blueprint
subtitle: Secure, GDPR-Compliant Healthcare Data Platform
author: Data Architecture Team
layout: cover
colorSchema: dark
transition: fade
---

# Data Architecture Blueprint

## A Secure, GDPR-Compliant Healthcare Data Platform

Medallion architecture | Bronze → Silver → Gold

---
layout: center
---

# Design Principles

---
layout: two-cols
---


## Data Platform Philosophy

Decouple **data persistence** from **data analytics**

- **Silver**: OpenEHR for granular, long-term storage with complete clinical context
- **Gold**: OMOP CDM for fast analytics, flat tables, and research workflows
- **Result**: Incremental analytics capability without re-processing raw data

---
layout: cover
background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%)
---

# Bronze Layer
## Ingestion & Immutable Vault

---

## Ingestion Principles

### Data Immutability & Idempotent Processing

Raw data locked at storage level (**WORM**). Pipelines guarantee idempotency via SHA-256 checksums — network retries never create duplicates.

### Regulatory Viability & Federated Custodianship

- Strict IAM access controls
- GDPR lifecycle management (crypto-shredding)
- Data lake as secure replica; original custodians maintain source data

---

## Ingestion Architecture

```
Secure Ingestion Gateway
        ↓
   [Authenticate]
   [Pseudonymize via PSDS]
   [Route by payload size]
        ↓
┌───────┼───────┬──────────┐
│       │       │          │
Small   Standard  Large Blobs
Streams (Normal)   (Omics)
(EDC)   Data      & Video
  ↓       ↓         ↓
Kafka   PUT      Pre-signed
Buffer  Direct   URLs
  ↓       ↓         ↓
Aggregator ───→ MinIO Landing Zone
                   ↓
            Demultiplex & Validate
                   ↓
            MinIO Bronze Vault
             (WORM, Locked)
```

---

## Bronze Data Flow (Detailed)

1. **Secure Gateway** — FastAPI validates tenant, pseudonymizes PII via Clinnova PSDS
2. **Payload Routing** — Size-dependent paths (Kafka buffer, direct PUT, pre-signed URLs)
3. **Demultiplexing** — Airflow detects multiplex flag, splits multi-patient files
4. **Validation & Scanning** — Malware scans, structural validation (headerless CSV, unparseable FASTA)
5. **Encryption** — MinIO KMS wraps each file with unique DEK
6. **Immutable Commit** — Files locked in Bronze Vault (Compliance Mode WORM)

---

## Identity Layer & Pseudonymization

### Master Patient Index (MPI)

Clinnova's MPI provides baseline identification across:
- Electronic Data Captures (EDCs)
- Unstructured data
- Omics data

**Goal**: Patient A from CHL = Patient A from LIH via secure linkage

### Entity Resolution (Optional)

For legacy datasets lacking native IDs:
- Deterministic matching rules
- Probabilistic methods (e.g., Splink)
- Deployed at Silver layer

---

## Audit Ledger (Bronze)

```
event_id (UUID, PK)
event_type (INGEST_LANDING, VALIDATED, REJECTED, CRYPTO_SHRED_COMPLETED)
event_timestamp (TIMESTAMP)
patient_id (secure research pseudonym)
sha256_checksum (indexed for deduplication)
s3_uri (object location)
payload_metadata (JSONB):
  - validation_error
  - raw_file_name
  - file_extension
  - manual_release (boolean)
  - project, cohort, partner, data domain
```

**Critical**: INSERT-only ledger with disaster recovery (replication + backups)

---

## Bronze Storage Tiers

| Layer | Path | Mode | Purpose |
|-------|------|------|---------|
| **Landing** | `s3://<tenant>/bronze_landing/<uuid>/` | Hot / Unlocked | Ingress point |
| **Bronze** | `s3://<tenant>/bronze/<uuid>/` | Hot-to-Warm / Compliance WORM | Source of truth |
| **Archive** | `s3://<tenant>/bronze_archive/<uuid>/` | Warm-to-Cold / Compliance WORM | Multiplex archives |

---
layout: cover
background: linear-gradient(135deg, #1e293b 0%, #334155 100%)
---

# Silver Layer
## Processing & Standardization

---

## Silver Processing Principles

### Governance Mode Storage

Versioned (not locked) to support replayable pipelines and service updates while protecting against accidental deletion.

### Event-Driven Architecture

Triggered by MinIO ObjectCreated Webhooks or Airflow Dataset Sensors — no cron jobs or polling.

### Deterministic Processing

Stateless, version-controlled containers. All software versions logged in audit ledger.

---

## Silver Layer Strategy

**Decouple persistence from analytics**

- **EDC Pipeline**: OpenEHR archetypes for complete granularity, zero schema migrations
- **Unstructured Pipeline**: Validation, compression, metadata extraction for omics/imaging/multimedia

```
Bronze (Raw, Format-Agnostic)
        ↓
   ┌────┴────┐
   ↓         ↓
 EDC      Unstructured
   ↓         ↓
OpenEHR  Compressed
 CDR     + Metadata
   ↓         ↓
  Silver Layer (Standardized)
        ↓
  Gold Layer (Analytics)
```

---

## EDC Pipeline: OpenEHR Strategy

### Why OpenEHR (not OMOP only)?

- **Long-term persistence** — Stable Reference Model never changes
- **Granular context** — Preserves clinical nuance ML models need
- **Zero schema migrations** — Universal structure adapts to any workflow
- **Automated validation** — Templates enforce constraints at API level

### Why not OpenEHR only?

- **Analytical overhead** — Nested hierarchies slow aggregations, survival analyses
- **Format mismatch** — Data scientists expect flat tables, not JSON compositions

### The Dual-Model Solution

OpenEHR (**Silver**) → complete granularity, long-term source of truth  
OMOP CDM (**Gold**) → fast analytics, flat tables, incremental per-group builds

---

## EHRbase & Clinical Data Modeling

### Core Components

- **EHRbase** — Production-ready Clinical Data Repository implementing openEHR specs
- **Pydantic** — Runtime validation before API ingestion (data transformation + error handling)
- **Clinical Knowledge Manager (CKM)** — Reuse pre-built archetypes from global ecosystem
- **Archetype Designer** — Low-code tool for novel clinical trial metrics

**Workflow**: Payload → Pydantic validation → EHRbase API ingestion → Versioned storage

---

## Unstructured/Omics Pipeline

### Processing Steps

1. **Validation** — FASTQ integrity checks, BAM sanity checks, DICOM header verification
2. **Identity Linkage** — Extract patient ID from metadata, verify against MPI/openEHR
3. **Compression** — bgzip (FASTQ/BAM), OME-TIFF with Deflate, JPEG-LS for medical imaging
4. **Metadata Extraction** — Technical attributes, quality metrics, file characteristics
5. **Catalogue Publication** — Features indexed in Silver Metadata Catalogue (OpenMetadata)

---

## Validation & Failure Handling

### Structural Validation per Modality

**Genomics**: FASTQ gzip -t, BAM samtools quickcheck  
**Imaging**: DICOM header tags, format compliance  
**Documents**: Page count, authoring app, language tags

### Logical Quarantine (No Physical Movement)

- File fails validation → event appended to audit ledger
- State mutates: `VALIDATION_FAILED` or `ORPHAN_PATIENT_ID`
- Scheduled retry loop with TTL (e.g., 1-day retry, 90-day max)
- Re-validation triggered when upstream systems resolve (MPI catch-up)

---

## Silver Storage & Identity Bridge

### Storage Tiers

```
s3://<tenant>/silver/<uuid>/<filename.ext>
├── Mode: Governance (versioned, replayable)
├── Versions: Retained per lifecycle policy
└── Archives: Older versions → cold storage
```

### Identity Bridge Table

```
patient_id
s3_uri
modality
file_type
sha_256_checksum
```

**Purpose**: Cross-layer GDPR compliance. Enables selective deletion cascades from Bronze → Silver → Gold

---
layout: cover
background: linear-gradient(135deg, #334155 0%, #475569 100%)
---

# Gold Layer
## Analytics, Serving & User-Facing Data

---

## Gold Layer Overview

**Purpose**: User-facing analytics, research-ready datasets, incremental transformations

**Architecture**: Structured organization by domain, research group, and use case

```
Silver (Flat, Standardized)
        ↓
    dbt Pipeline
        ↓
┌───────┴───────┬──────────┬────────┐
│               │          │        │
OMOP CDM    FHIR        Identity   Derived
(Analytics) (Standards)  Bridge    Data
                                  (Sandbox)
```

---

## OMOP: Observations Medical Outcomes Partnership

### Purpose

Fast analytics via flattened, canonical data model optimized for research

### ETL Strategy

- **Incremental builds** — Per research group, not monolithic re-processing
- **Powered by openEHR** — dbt queries clean Silver archetypes, not raw Bronze data
- **Faster, maintainable** — Heavily vetted intermediate data eliminates repetitive raw ETL

### Tools

- **dbt** — Version-controlled SQL pipelines deploying schema updates
- **OHDSI ATLAS** — Web-based cohort builder, phenotyping, population analysis
- **OHDSI Athena** — Standard ontology download (SNOMED, LOINC, RxNorm)

---

## FHIR: Fast Healthcare Interoperability Resources

### Purpose

External tool integration, standards-based interoperability

### Role in Platform

- Written to Gold layer from Silver pipelines
- Serves downstream FHIR-compatible tools
- Researcher READ access (ABAC enforced)
- AI agents READ access

---

## Derived Data & Research Sandboxes

### Gold Derived Data Layer

```
s3://gold__my_group/
├── cohort_analysis_2024/
├── ml_training_dataset/
└── temp_workspace/
```

### Access Model

- **Researchers**: READ/WRITE within their group prefix only (ABAC enforced)
- **Serving Gateway**: Delivers pre-signed URLs for data download
- **Submission Gateway**: Routes researcher uploads here

---

## Identity Bridge (Cross-Layer GDPR)

### Architecture

Maintained across **Bronze → Silver → Gold**

```
Bronze Audit Ledger
├── File A (patient_id: p001, checksum: abc123)
├── File B (patient_id: p001, checksum: def456)
└── File C (patient_id: p002, checksum: ghi789)
        ↓
Silver Identity Bridge
├── p001 → [FileA_uri, FileB_uri]
└── p002 → [FileC_uri]
        ↓
Gold Tables
├── OMOP_patient_p001
└── OMOP_patient_p002
```

### GDPR Deletion Cascade

1. **Right to Erasure Request** → p001 revokes consent
2. **KMS**: Destroy DEK for p001 files → Bronze files unrecoverable
3. **Audit Ledger**: Log `CRYPTO_SHRED_COMPLETED` event
4. **Silver**: Logical deletion via identity bridge
5. **Gold**: Drop p001 OMOP tables, derived datasets

---
layout: cover
background: linear-gradient(135deg, #475569 0%, #64748b 100%)
---

# Access Control
## RBAC + ABAC + Crypto-Shredding

---

## Access Control Framework

### RBAC (Role-Based Access Control)

**Human Roles**:
- Data Provider
- Researcher
- Clinical Data Modeler
- Data Engineer
- Security & Compliance Officer

**Machine Roles/Services**:
- Landing to Bronze Processor
- Bronze to Silver Processor
- Silver to Gold Processor
- Ingestion Gateway
- Serving Gateway
- Submission Gateway
- AI Agents

---

## Bronze Layer Access

| Resource | Data Provider | Data Engineer | Others |
|----------|---------------|---------------|--------|
| **Landing Bucket** | WRITE | NONE | NONE |
| **Bronze Bucket** | NONE | READ (debug) | NONE |
| **Kafka Buffer** | NONE | READ (debug) | NONE |
| **Audit Ledger** | NONE | READ | Security: READ/WRITE |

### Key Services

- **Landing→Bronze Processor**: WRITE to Bronze, WRITE to audit ledger
- **Ingestion Gateway**: WRITE to Kafka buffer, WRITE audit (INGEST status)

---

## Silver Layer Access

| Resource | Data Modeler | Data Engineer | Researcher |
|----------|--------------|---------------|------------|
| **EHRbase CDR** | WRITE (templates) | READ | NONE |
| **Silver Bucket** | NONE | READ | NONE |
| **Metadata Catalogue** | NONE | READ | NONE |
| **Audit Ledger** | NONE | READ | NONE |

### Key Services

- **Bronze→Silver Processor**: WRITE (files, audit events), READ MPI/KMS
- **Silver→Gold Processor**: READ (features for OMOP transformation)

---

## Gold Layer Access (ABAC Enforced)

| Resource | Researcher | Data Modeler | Data Engineer |
|----------|-----------|--------------|---------------|
| **OMOP CDM** | READ (schema-based) | NONE | WRITE (dbt) |
| **FHIR** | READ | NONE | READ |
| **Derived Data** | READ/WRITE (own prefix) | NONE | NONE |
| **Vocabulary Maps** | READ | READ/WRITE | WRITE (dbt) |

**ABAC Examples**:
- Researcher can only query `rg__oncology` schema
- Researcher can only R/W `s3://gold__my_group/`

---

## Crypto-Shredding: GDPR Right to Erasure

### Challenge

WORM (Write-Once-Read-Many) immutability conflicts with GDPR Article 17 deletion rights.

### Solution: Per-File Data Encryption Keys (DEK)

1. **Ingestion**: MinIO KMS generates unique DEK per file
2. **Storage**: File encrypted with DEK, locked in WORM vault
3. **Deletion Request**: Destroy specific file's DEK via KMS API
4. **Result**: File physically intact but cryptographically irrecoverable

### Cascade Effect

```
DEK Destruction (Bronze)
        ↓
Audit Ledger: CRYPTO_SHRED_COMPLETED
        ↓
Delete Silver identity bridge entries
        ↓
Drop Gold OMOP/FHIR/derived tables
```

---

## Control Plane: OpenMetadata & KMS

| System | Access Type | Purpose |
|--------|------------|---------|
| **OpenMetadata KG** | Admin: Data Engineer | Lineage, glossary, discovery |
| | Read: Researchers | Search cohorts/datasets |
| | Read/Write: Data Modelers | Update business terms |
| **MinIO KMS** | Read/Write: DEK Management | Key generation, rotation |
| | Read/Write: Security Officer | Right-to-Erasure deletion |

---
layout: cover
background: linear-gradient(135deg, #64748b 0%, #78909c 100%)
---

# Observability & Metrics
## Pipeline Health, Data Quality, Storage

---

## Observability Layers

### 1. Pipeline Health (Airflow)

- DAG execution status, duration, retry counts
- Per-task success/failure
- Export to centralized monitoring (Prometheus → Grafana)

### 2. Data Quality

- **Great Expectations** — Audit Silver→Gold boundary for distribution anomalies, schema drift
- **OHDSI DQD** — SQL assertions on OMOP tables for clinical plausibility (international standards)
- **Pydantic** — EDC validation failures at ingestion

### 3. Storage Observability

- **MinIO Prometheus endpoint** — Bucket consumption, object count, throughput, error rates
- **Lifecycle tracking** — Validate warm/cold transitions, cost predictability

---

## OpenMetadata: Unified Metrics Visibility

```
Airflow Pipelines
├── DAG status, duration
├── Task-level metrics
└── Structured logs
        ↓
Great Expectations Audits
├── Data profile snapshots
├── Quality anomalies
└── Schema drift alerts
        ↓
Custom Metrics (DQD, etc.)
├── Clinical plausibility
├── OHDSI conformance
└── Domain-specific rules
        ↓
OpenMetadata Aggregation
├── Alerting & notifications
├── Metrics dashboard
└── Lineage + quality view
```

---

## Storage Observability

### MinIO AIStor Metrics

```
Bucket-Level:
├── Storage consumption (GB/TB)
├── Object count
├── Request throughput (ops/sec)
└── Error rates (%)

Lifecycle Policy Execution:
├── Transitions to warm/cold
├── Archival completion rates
└── Cost trending
```

### Expected Metrics

- Bronze ingestion rate (files/day)
- Silver reprocessing time
- Gold OMOP build duration
- Storage cost per modality

---
layout: cover
background: linear-gradient(135deg, #78909c 0%, #90a4ae 100%)
---

# Infrastructure Stack
## Technology Choices & Justification

---

## Storage & Database

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Object Storage** | MinIO / AIStor | S3-compatible, on-prem, enterprise security |
| **Relational DB** | PostgreSQL | Audit ledgers, metadata, EHRbase backend, OMOP CDM |
| **Staging Buffer** | Apache Kafka | Small-file aggregation, event streaming |
| **Search Engine** | Elasticsearch | OpenMetadata discovery |

---

## Ingestion & Gateway

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Ingestion Gateway** | FastAPI | Stateless REST, tenant auth, pseudonymization routing |
| **KMS** | MinIO KMS | Per-file DEK generation, key lifecycle |
| **Identity & ER** | MPI + Splink | Patient identification, entity resolution |

---

## Orchestration & Processing

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Orchestrator** | Apache Airflow | DAG scheduling, event-driven pipelines, cross-layer coordination |
| **Clinical CDR** | EHRbase | OpenEHR semantic storage, API validation, version control |
| **Validation** | Pydantic | Payload transformation, runtime type checking |
| **Modeling** | CKM & Designer | Archetype curation, low-code template design |

---

## Gold, Analytics & Quality

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Transformation** | dbt | Version-controlled SQL, OMOP ETL, incremental builds |
| **OMOP Suite** | OHDSI ATLAS | Cohort builder, phenotyping, population analysis |
| **Ontologies** | OHDSI Athena | Standard vocab download (SNOMED, LOINC, RxNorm) |
| **Quality Engine** | OHDSI DQD | SQL-based OMOP validation, clinical plausibility checks |
| **Auditing** | Great Expectations | Data profiling, distribution anomaly detection, quality metrics |

---

## Governance & Metadata

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Knowledge Graph** | OpenMetadata | Lineage, glossary, discovery, MCP server for AI agents |
| **Metrics** | Prometheus | API + system metrics |
| **Dashboarding** | Grafana / Superset | Metrics visualization |
| **Observability** | OpenTelemetry | Structured tracing |

---

## Architecture Diagram: Full Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    Data Providers (External)                │
└──────────────────────────────────────────────────────────────┘
                            ↓
        ┌───────────────────────────────────┐
        │   Secure Ingestion Gateway        │
        │   (FastAPI, PSDS, Auth)           │
        └───────────────────────────────────┘
         ↙                    ↓                      ↖
    Kafka Buffer        S3 Direct               Pre-signed URLs
         ↓                    ↓                      ↓
    ┌────────────────────────────────────────────────────┐
    │     MinIO Landing Zone (Hot, Unlocked)           │
    └────────────────────────────────────────────────────┘
                            ↓
         Airflow: Demultiplex, Validate, Scan
                            ↓
    ┌────────────────────────────────────────────────────┐
    │     MinIO Bronze Vault (WORM, Immutable)          │
    │ + PostgreSQL Audit Ledger (DEK, checksums)        │
    └────────────────────────────────────────────────────┘
                            ↓
         ┌──────────────────┬──────────────────┐
         ↓                  ↓                  ↓
    ┌─────────┐        ┌──────────┐      ┌──────────┐
    │ EHRbase │        │ Compress │      │ Validate │
    │ (EDC)   │        │  Metadata│      │ Unstructured
    └─────────┘        │ Extract  │      └──────────┘
         ↓              │ Omics    │           ↓
         ↓              └──────────┘           ↓
         └──────────────────┬──────────────────┘
                            ↓
    ┌────────────────────────────────────────────────────┐
    │ MinIO Silver (Governance, Versioned, Replayable) │
    │ + Identity Bridge + Metadata Catalogue            │
    └────────────────────────────────────────────────────┘
                            ↓
            ┌───────────────┼───────────────┐
            ↓               ↓               ↓
        ┌────────┐     ┌──────────┐   ┌──────────┐
        │ dbt    │     │OHDSI DQD │   │OpenMeta  │
        │OMOP ETL│     │Validation│   │Governance│
        └────────┘     └──────────┘   └──────────┘
            ↓               ↓               ↓
    ┌────────────────────────────────────────────────────┐
    │          MinIO Gold (Structured, Physical)        │
    │  s3://gold__common/  s3://gold__my_group/         │
    │  OMOP | FHIR | Derived | Vocabulary Mappings     │
    └────────────────────────────────────────────────────┘
                            ↓
        ┌───────────────────────────────────┐
        │   Serving Gateway & Researcher    │
        │   Access (ABAC, Pre-signed URLs)  │
        └───────────────────────────────────┘
```

---
layout: cover
---

# Key Takeaways

---

## Design Philosophy

✓ **Security by design** — WORM + crypto-shredding + pseudonymization  
✓ **GDPR by architecture** — Federated custody, consent cascades, audit trail  
✓ **Decouple persistence from analytics** — OpenEHR (Silver) ≠ OMOP (Gold)  
✓ **Logical over physical** — Flat Bronze/Silver, structure in Gold + metadata

---

## Why This Works

1. **Immutability + Encryption** solves compliance & audit trails
2. **OpenEHR in Silver** enables zero schema migrations & granular ML data
3. **OMOP in Gold** delivers fast analytics without re-processing Bronze
4. **Identity Bridge** connects layers for selective GDPR deletion
5. **Event-driven + Deterministic** ensures reliability & reproducibility

---

## Next Steps

- **Design clinical archetypes** — CKM + domain experts
- **Prototype ingestion gateway** — FastAPI + PSDS integration
- **Build initial Bronze/Silver pipelines** — Airflow DAGs + EHRbase
- **Plan incremental OMOP builds** — Per research group, not monolithic
- **Implement observability** — Prometheus + Grafana dashboards
- **Test crypto-shredding** — GDPR compliance validation

---
layout: center
---

# Questions?

Data Architecture Blueprint  
A Reference Implementation for Secure, Scalable, GDPR-Compliant Healthcare Analytics

---

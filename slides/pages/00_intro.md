---
layout: cover
---

# Data Architecture Blueprint

Healthcare research data platform blueprint

---
layout: default
---

# Agenda

*~35 slides · ℹ = information slide*

- Introduction — challenges & medallion architecture
- Background & design principles
- Organization — S3 layout & data contract
- End-to-end pipeline overview
- Landing → Bronze → Silver
- Clinical data models — openEHR & OMOP
- Gold serving layer & APIs
- Identity bridge, linking & GDPR cascade
- Discovery & observability
- Infrastructure stack
- Access control — human & machine roles


---
class: compact-slide optional-slide
---

# A bit about my background

- **On-Premises data architecture**
    - Vendor-agnostic medallion lakehouse over S3(Hetzner)/MinIO
    - Big emphasis on creating custom on-prem scalable solutions
    - Orchestration
    - Relational and non-releational storage and data serving

- **Healthcare+Bioinformatics background with industry standards** 
    - BSc in Nutritional Sciences -> MSc and Phd in Bioinformatics
    - PhD at University of Luxembourg (genomics research), open-source tooling for large-scale protein function annotation (data integration of multiple sources).
    - ~4 years in academia, ~4 years in industry - research background, and delivery and product-oriented

- **Data integration & entity resolution**
    - 8 years building data pipelines
    - Deterministic + probabilistic record linkage
    - Data + processes standardization across multiple modalities and domains


---
class: slide challenges-slide
---


# Challenges

*Four intersecting challenges that define the architecture*

<div class="grid grid-cols-2 gap-4 challenges-grid">
<div class="challenge-card">
<div class="challenge-card-title">Multiple Data Modalities</div>
<div class="challenge-card-body">Managing highly disparate data (clinical records, omics, multimedia) across incompatible formats, scale (KB to TB), and validation rules.</div>
</div>
<div class="challenge-card">
<div class="challenge-card-title">The GDPR Paradox: Right-to-erasure vs. WORM compliance</div>
<div class="challenge-card-body">Right to erasure conflicts with WORM compliance. Resolved via per-file KMS-managed DEKs; destroying the key achieves cryptographic erasure without breaking the immutable storage chain.</div>
</div>
<div class="challenge-card">
<div class="challenge-card-title">Zero-Trust data ingestion</div>
<div class="challenge-card-body">Federated custodianship prevents enforcing data contracts at the source. We follow a zero-trust paradigm executing automated malware scanning, multiple validation steps, and payload demultiplexing.</div>
</div>
<div class="challenge-card">
<div class="challenge-card-title">Granularity vs. Analytical Performance</div>
<div class="challenge-card-body">Clinical systems require nested, granular semantics, while analytics often require flat tabular data. Resolved via a dual-model paradigm decoupling storage (openEHR) from serving (OMOP).</div>
</div>
</div>



---
class: compact-slide medallion-slide
---

# Medallion architecture


### Landing + Bronze - ingestion & compliance

Secure entry boundary that locks raw data as-is into flat, tenant-isolated, immutable WORM buckets ("Compliance Mode"). It enforces a strict **"One Object, One Patient"** contract to enable individual file crypto-shredding for GDPR compliance.

### Silver-  processing & validation

Validates, aggregates, compresses, and extracts semantic context within a versioned ("Governance Mode") layer built for deterministic pipeline replays. It isolates clinical data into an openEHR repository while anchoring unstructured/omics data via a central identity bridge table.


### Gold + Serving - serving & analytics

"Sandbox-like" research-oriented serving layer. openEHR data is exposed as OMOP for analytics. Data is organized by research group and feeds a centralized OpenMetadata knowledge graph for semantic discovery and AI workflows.


---
class: compact-slide optional-slide principles-slide
---

# Design principles

- *Data quality & incremental standardization preservation of ground-truth data is the baseline*. To balance immediate operational ingestion velocity with the long-term evolution of institutional knowledge, data standards and structural feature extraction are enforced incrementally through modular pipelines.

- *Decoupled persistence & analytical layers*. Isolate ingestion and processing volatility by maintaining data-type-agnostic, flat bronze and silver storage layers. Leverage openEHR to handle source-faithful clinical granularity at the persistence layer (silver), deferring human-readable, structured organization to the gold/serving layer mapped to the OMOP CDM for research and analytics.

- *Security by design*. Enforce WORM immutability, per-file encryption, and physical multi-tenant isolation via dedicated per-tenant buckets (landing, bronze, silver). Automate pseudonymization at the system boundary and log all events through a per-layer ledger.

- *Data linking & compliance by design*. Secure the full clinical chain across all modalities by utilizing an identity bridge table, enabling automated compliance actions such as consent withdrawal cascades.

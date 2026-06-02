---
layout: cover
---

# Data Architecture Blueprint

Healthcare research data platform blueprint

---
layout: default
---

# Agenda

*~35 slides · ℹ = information sldie*

- Introduction — challenges & medallion architecture
- Background & design principles
- Organization — S3 layout & data contract
- End-to-end pipeline overview
- Landing → Bronze → Silver
- Clinical models — openEHR & OMOP
- Gold serving layer & APIs
- Identity bridge, linking & GDPR cascade
- Discovery & observability
- Infrastructure stack
- Access control — human & machine roles


---
class: compact-slide
---

# A bit about my background

- **On-Premises Data Architecture**
    - Vendor-agnostic medallion lakehouse over S3(Hetzner)/MinIO
    - Big emphasis on creating custom on-prem scalable solutions
    - Orchestration
    - Relational and non-releational storage and data serving

- **Healthcare+Bioinformatics background with industry standards** 
    - BSc in Nutritional Sciences -> MSc and Phd in Bioinformatics
    - PhD at University of Luxembourg (genomics research), open-source tooling for large-scale protein function annotation and genomics data integration.
    - ~4 years in academia, ~4 years in industry - research background, and delivery and product-oriented

- **Data Governance & Entity Resolution**
    - 8 years building data pipelines
    - Deterministic + probabilistic record linkage
    - Data standardization across multiple modalities and domains


---
class: slide
---


# Challenges

*Four intersecting challenges that define the architecture*

<div class="grid grid-cols-2 gap-x-6 gap-y-4 text-sm text-left">
<div>
<div class="font-bold text-slate-800">Multiple data modalities</div>
<div class="mt-1 text-slate-600 leading-snug">Clinical data records, omics data, multimedia, text documents - each with incompatible formats, scales (KB to 300GB+), and validation rules.</div>
</div>
<div>
<div class="font-bold text-slate-800">GDPR in an immutable store</div>
<div class="mt-1 text-slate-600 leading-snug">Right to erasure conflicts directly with fully auditable and WORM immutability -> Crypto-shredding with per-file DEKs (KMS managed) </div>
</div>
<div>
<div class="font-bold text-slate-800">Federated custodianship</div>
<div class="mt-1 text-slate-600 leading-snug">CHL, IBBL and other partners are primary legal data custodians. The platform acts as a secure replica for downstream use cases. Pseudonymization must happen upstream but verified throughout the workflow.
</div>
</div>
<div>
<div class="font-bold text-slate-800">Research ≠ clinical IT</div>
<div class="mt-1 text-slate-600 leading-snug">Downstream consumers data analytics driven data formats, but clinical granularity requires semantics preservation. One data model cannot serve both without compromise. Multiple opinions, ideas, and requirements -> no one size fits all solution</div>
</div>
</div>

---
class: compact-slide
---

# Medallion architecture


### Landing + Bronze - ingestion & compliance

Secure entry boundary that locks raw data as-is into flat, tenant-isolated, immutable WORM buckets ("Compliance Mode"). It enforces a strict **"One Object, One Patient"** contract to enable individual file crypto-shredding for GDPR compliance.

### Silver-  processing & validation

Validates, aggregates, compresses, and extracts semantic context within a versioned ("Governance Mode") layer built for deterministic pipeline replays. It isolates clinical data into an openEHR repository while anchoring unstructured/omics data via a central identity bridge table.


### Gold + Serving - serving & analytics

"Sandbox-like" research-oriented serving layer. openEHR data is exposed as OMOP for analytics. Data is organized by research group and feeds a centralized OpenMetadata knowledge graph for semantic discovery and AI workflows.


---
class: slide optional-slide
---

# Design principles

- *Data quality and standardization as the driving principle* - Prioritize iterative and collaborative development while maintaining the preservation of ground-truth data as the non-negotiable architectural baseline.  

- *Security by design* - WORM immutability, per-file encryption, physical multi-tenant isolation via dedicated per-tenant buckets (landing, bronze, silver), automated pseudonymization at the system boundary. Automated logging of events through a per-layer ledger.  

- *Data linking by design* - Secure the full clinical chain across all modalities by utilizing an identity bridge table, enabling e.g., consent withdrawal cascades to meet strict regulatory compliance.  

- *Decoupled persistence from analytics* - Leverage openEHR to handle source-faithful clinical granularity (silver), mapping to the OMOP CDM (gold) for research and analytics.

- *Logical over physical organization* - Maintain data-type-agnostic, flat bronze and silver storage layers to isolate ingestion and processing volatility, deferring human-readable structure to gold/serving layer.  

- *Incremental process standardization* - Enforce data standards and structural feature extraction incrementally through modular pipelines, balancing immediate operational ingestion velocity with the long-term evolution of institutional knowledge.



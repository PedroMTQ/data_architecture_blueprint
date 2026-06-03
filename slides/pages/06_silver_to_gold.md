---
layout: section
class: section-slide
transition: fade
---

# Gold serving

---
layout: default
class: diagram-slide  optional-slide
---

# Gold serving layer

![Silver to gold](./diagrams/silver_to_gold_to_serving.svg)

---
class: compact-slide

---

# Gold objectives

- Convert **openEHR → OMOP** for analytics-driven serving
- Serve **omics, multimedia, & unstructured data*** — e.g. cohort requests assemble multi-modal assets in Gold and grant researcher access via the API
- Integrate **researcher** derived data (i.e., analysis) for downstream use cases (AI workflows, sharing with other researchers, etc)
- Richer **catalogue & semantics** - discoverability, knowledge base
- **Cross-modal links** between all data assets (various modalities + derived data)



<p class="gold-callout">After silver processing, <strong>clinical data</strong> is reshaped into analysis-ready gold datasets (OMOP CDMs), and <strong>omics, multimedia, documents, and researcher outputs</strong> are published to gold storage for discovery and reuse</p>


<p class="slide-footnote">* Serving API for standard requests; pre-signed URLs for large files</p>

---
layout: default
class: diagram-slide
---

# openEHR → OMOP pipeline

![openEHR to OMOP](./diagrams/openehr_to_omop.svg)

---
class: compact-slide optional-slide

---

# Submission API (upload)

Gateway for **derived / un-supervised** researcher outputs - strict compliance **before** data enters Gold

- **Automated:** infer schema & size; cross-reference Silver/Gold hashes; technical lineage (e.g., potentially derived from SLURM/HPC jobs)
- **Minimize manual input:** project ID, study/cohort description, **`patient_id`** - essential for **identity bridge** & GDPR cascade
- **On success:** `s3://gold_<group>/…` · append **gold audit ledger**· syncs to **OpenMetadata**
- Exposure per **RBAC/ABAC** - defined by the research group head

---
class: compact-slide optional-slide

---

# Serving API (read)

**Forked delivery** (mirrors ingestion) for downstream consumers

- **Small / standard files** → **Serving API** gateway (REST)
- **Large omics & binaries** → time-bound **pre-signed URLs** (direct MinIO, no API streaming)
- **Tabular analytics:** OMOP CDM + **ATLAS**
- **Optional later:** feature store for ML features; **MLflow** registry/serving on S3 artifacts

Security needs to be scoped.
Provision for common gold bucket → `s3://gold_lih/**`

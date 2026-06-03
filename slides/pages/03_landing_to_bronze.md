---
layout: section
class: section-slide
transition: fade
---

# Landing → Bronze

---
layout: default
class: diagram-slide optional-slide
---

# Landing → Bronze

![Landing to bronze](./diagrams/landing_to_bronze.svg)

---
class: compact-slide
---

# Payload routing

```mermaid
flowchart LR
  EDC[Small / EDC]
  STD[Standard PUT]
  OMICS[Large / Omics]

  GW[Ingestion Gateway]

  K[Kafka]
  Agg[Aggregator]
  PS[Pre-signed URL]
  LZ[Landing zone]

  EDC --> GW
  STD --> GW
  OMICS --> GW

  GW -->|EDC| K
  K --> Agg
  Agg --> LZ

  GW -->|PUT| LZ

  GW -->|Omics| PS
  PS --> LZ

  linkStyle 0,3,4,5 stroke:#0d9488,stroke-width:3px
  linkStyle 1,6 stroke:#2c5282,stroke-width:3px
  linkStyle 2,7,8 stroke:#cd7f32,stroke-width:3px
```

---
class: compact-slide optional-slide
---

## Ingestion gateway

- Authenticated stateless ingestion gateway
- Pseudonymization at the boundary for all inbound PII (calls Clinnova's **PSDS**).
- Routes payload by size and modality

## Patient ID & identity linkage 

- Master Patient Index (MPI) links all modalities: EDC, unstructured, and omics to one patient identity
- Data is re-verified during downstream silver processing from file headers (e.g., FASTQ tags)

## Demultiplexing

Multiplexed files **must not** enter WORM bronze intact.

1. Land in `bronze_landing` with `is_multiplexed: true`
2. Demultiplexer splits into single-patient files
3. Each file goes to landing layer → normal validation → bronze vault
4. Original multiplex → `bronze_archive`

---
class: compact-slide optional-slide
---

# Pre-validation (landing)

- Malware scan + file structure checks (generic and file-type specifics)
- Failures → **logical quarantine** in audit ledger (`VALIDATION_FAILED`)
- Airflow sensor retries with TTL - no physical move required
- Clean files → per-file **DEK** → immutable **bronze** bucket (encrypted and WORM)

**Once files are established as valid they move to the bronze bucket**

---
class: compact-slide optional-slide
---

# Structural validation & compression

*Examples by modality — built incrementally per use case*

**Structural validation (landing / bronze)**
- **Genomics:** `gzip -t` on `.fastq.gz`; `samtools quickcheck` on BAM/CRAM
- **Imaging:** DICOM mandatory tags (modality, SOP Class UID) — format compliance, not deep image QA
- **Documents / tabular:** headerless CSV, unparseable FASTA/FASTQ magic bytes
- **All modalities:** malware scan; `patient_id` cross-check vs MPI where extractable from headers


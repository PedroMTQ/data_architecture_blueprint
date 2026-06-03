---
layout: section
class: section-slide
transition: fade
---

# Identity & compliance

---
class: compact-slide diagram-slide
---

# Identity bridge table

![Identity bridge](./diagrams/identity_bridge.svg)

`patient_id` · `s3_uri` · `modality` · `file_type` · `sha_256_checksum`

**Operational source of truth** at Silver ingest — Submission API must anchor derived assets here.

---
class: compact-slide optional-slide
---

# Row-level entity linking — mechanism

```mermaid
flowchart LR
  MPI[MPI / Splink] --> Ingest[Silver ingest]
  Ingest --> Bridge[Identity bridge]
  Ledger[Audit ledger] -.immutable fallback.- Bridge
  Bridge --> EHR[EHRbase]
  Bridge --> Omop[OMOP CDMs, FHIR]
  Bridge --> Gold[Gold data assets, derived data]
  Sub[Submission API] --> Bridge

  style Bridge fill:#76608A,stroke:#76608A,color:#fff
```

- **Identity bridge** — operational source of truth (`patient_id` + `s3_uri` at Silver)
- **Audit ledger** — immutable fallback for MPI drift / merges
- Others are derivatives

---
class: compact-slide diagram-slide optional-slide
---

# GDPR erasure & cascade

![Crypto shredding](./diagrams/crypto_shredding.svg)

- **Bronze:** destroy per-file **DEK** (SSE-KMS) - WORM object is preserved, ciphertext unreadable; ledger `CRYPTO_SHRED_COMPLETED`
- **Cascade** (via **identity bridge** + `patient_id`): Silver S3, Gold OMOP / FHIR / derived, OpenMetadata catalogue
- **Derived uploads** must use **Submission API** with `patient_id`, otherwise cascade may miss derived data assets


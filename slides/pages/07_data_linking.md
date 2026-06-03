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
  Ingest --> Bridge[Identity bridge · operational SoT]
  Ledger[Bronze audit ledger] -.immutable fallback.- Bridge
  Bridge --> EHR[EHRbase · derivative clinical]
  Bridge --> Gold[Gold · OMOP · FHIR · derived]
  Sub[Submission API] --> Bridge
  Pipelines[Airflow / dbt / Submission] -.read-only.- OM[OpenMetadata · governance]
```

1. **Identity bridge** — operational SoT (`patient_id` + `s3_uri` at Silver)
2. **Bronze audit ledger** — immutable fallback for MPI drift / merges
3. **EHRbase** — derivative clinical context (pre-resolved `patient_id`, no late join)
4. **OpenMetadata** — derivative lineage graph (MCP / stewards; not runtime identity validation)

---
class: compact-slide diagram-slide optional-slide
---

# GDPR erasure & cascade

![Crypto shredding](./diagrams/crypto_shredding.svg)

- **Bronze:** destroy per-file **DEK** (SSE-KMS) - WORM object stays, ciphertext unreadable; ledger `CRYPTO_SHRED_COMPLETED`
- **Cascade** (via **identity bridge** + `patient_id`): Silver S3 · Gold OMOP / FHIR / derived · OpenMetadata catalogue
- **Derived uploads** must use **Submission API** with `patient_id` - otherwise cascade may miss researcher assets


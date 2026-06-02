---
layout: default
class: diagram-slide
---

# Landing → Bronze

![Landing to bronze](./diagrams/landing_to_bronze.svg)

---
class: compact-slide optional-slide
---

# Payload routing

```mermaid
flowchart LR
  SM[small files - EDC] --> GW[Gateway] --> K[Kafka] --> Agg[Aggregator] --> LZ[Landing zone]
  STD[Standard PUT] --> GW[Gateway]--> LZ
  BIG[Omics large] --> GW[Gateway] --> PS[Pre-signed URL] --> LZ
```

---
class: compact-slide optional-slide
---

## Ingestion gateway

- Authenticated stateless ingestion gateway
- Pseudonymization at the boundary for all inbound PII (calls Clinnova's **PSDS**). Outputs secure research **patient_id** for ledger (cross-reference with Clinnova's MPI)
- Routes payload by size and modality

## Patient ID & identity linkage 

- Master Patient Index (MPI) links all modalities: EDC, unstructured, and omics to one patient identity
- Data is re-verified during downstream silver processing from file headers (e.g., FASTQ tags)
- Orphan files → `ORPHAN_PATIENT_ID` in ledger + retry loop (temporal decoupling of upstream MPI patient registration)


---
class: compact-slide optional-slide
---

# Demultiplexing

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



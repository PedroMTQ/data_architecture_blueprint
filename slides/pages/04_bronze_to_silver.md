---
layout: section
class: section-slide
transition: fade
---

# Bronze → Silver

---
layout: default
class: diagram-slide optional-slide
---

# Bronze → Silver

![Bronze to silver](./diagrams/bronze_to_silver.svg)

---
class: compact-slide
---

# Bronze vs silver file locking and immutability

| Bronze | Silver |
|--------|--------|
| Compliance mode (WORM) | Governance mode (versioned) |
| Immutable source of truth | Replayable; authorized overwrite on pipeline replay |
| SSE-KMS (per-object DEK) | SSE-KMS (per-object DEK) |
| Erasure: destroy DEK (WORM) | Erasure: delete object/versions |
| **Bronze** audit ledger (ingest, payload metadata) | **Silver** audit ledger (processing, pipeline fingerprint) |

*Silver pipelines fingerprint is preserved to allow for deterministic replays*

---
class: compact-slide

---

# Silver pipelines (fork by modality)

```mermaid
flowchart LR
  O[Bronze ObjectCreated]
  subgraph EDC [Clinical data / EDC → openEHR]
    direction LR
    subgraph PYD [Pydantic]
      direction LR
      P1[Parse] --> P2[Validate] --> P3[Transform]
    end
    E4[EHRbase API]
    P3 --> E4
  end
  subgraph UNST [Unstructured / omics]
    direction LR
    U1[Validate] --> U2[MPI check] --> U3[Metadata extract] --> U4[Compress] --> U5[Silver S3 + catalogue]
  end
  O --> P1
  O --> U1
```

*OpenEHR archetypes and templates to be defined by data modeller and mapped to data collection system*


Per-modality feature extraction (genomics, imaging, documents) → **OpenMetadata** catalogue

- Extracts superficial semantics: Reference genome, coverage, Q30%, modality, slice thickness, page count, etc
- Deferred deeper semantics (e.g., AI embeddings for documents deferred to **Gold**)

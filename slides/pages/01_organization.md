---
layout: section
class: section-slide
transition: fade
---

# Organization

---
class: compact-slide
---

# Storage, isolation & organization

**Tenant isolation:** one S3 bucket per tenant at bronze/silver; gold uses `gold_<group>/` buckets plus per-group **OMOP** schemas in PostgreSQL.

| Layer | Storage | Main Outcomes |
|-------|----------------------|---------|
| Landing | `bronze_landing/<uuid>/` - hot, unlocked | Validation, **logical quarantine** |
| Bronze | `bronze/<uuid>/` - WORM | Flat files |
| Archive | `bronze_archive/<uuid>/` - WORM | archived files, e.g., multiplex data |
| Silver | S3 `silver/<uuid>/`, (unstructured),  **EHRbase / Postgres** (EDC) | Files validated, compress, metadata |
| Gold | S3 `gold_<group>/…` , **OMOP / Postgres** | **Organized by** group, etc; `_` as word spacer, `__` / `/` as namespace; OMOP CDM, data serving, deeper metadata |

## Data contract

**One object, one patient**

- Each file contains data for **one** patient; a patient may own many files
- Enables per-patient crypto-shredding without collateral loss

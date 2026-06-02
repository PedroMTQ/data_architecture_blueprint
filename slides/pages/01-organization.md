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
| Gold | S3 `gold_<group>/…` , **OMOP / Postgres** | **Organized by** group, project, cohort, domain; OMOP CDM, data serving, deeper metadata |

Each layer has its own **append-only PostgreSQL audit ledger**.
**Logical quarantine:** failed assets stay in place; certain ledger states block downstream work until retry/TTL.
**DEKs** in MinIO KMS.
Naming (`_` / `__` / `/`) at **Gold** only.

---
class: compact-slide
---

# Data contract

**One object, one patient**

- Each file contains data for **one** patient; a patient may own many files
- Enables per-patient crypto-shredding without collateral loss

**Paths & idempotency**

- UUID at gateway: `s3://<tenant>/bronze_landing/<uuid>/<file>`
- SHA-256 checksum in landing audit ledger -> duplicate uploads rejected

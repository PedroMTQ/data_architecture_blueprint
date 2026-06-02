
## Access control

This is a high level RBAC matrix, but in a production system it would have much more depth.  Additionally we would need to combine it with ABAC e.g., for role-specific permissions in the gold layer.

### Ingress & landing resources

#### S3 landing bucket

* Human roles:
  * Data provider — WRITE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — NONE
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — READ/WRITE
  * Bronze to silver processor — NONE
  * Silver to gold processor — NONE
  * Ingestion gateway — WRITE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE

### Bronze resources

#### S3 bronze and archive bucket

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — READ *(admin debug only)*
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — WRITE
  * Bronze to silver processor — READ
  * Silver to gold processor — NONE
  * Ingestion gateway — NONE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE

#### Bronze Kafka buffer

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — READ *(admin debug only)*
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — READ
  * Bronze to silver processor — NONE
  * Silver to gold processor — NONE
  * Ingestion gateway — WRITE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE

#### Bronze audit ledger

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — READ
  * Security & compliance officer — READ/WRITE *(triggers crypto-shredding events and deletes DEK)*
* Machine roles/services:
  * Landing to bronze processor — WRITE
  * Bronze to silver processor — NONE
  * Silver to gold processor — NONE
  * Ingestion gateway — WRITE *(Logs INGEST status)*
  * Serving gateway — READ/WRITE *(Reads for hash duplicates, writes initial drop event)*
  * Submission gateway — NONE
  * AI agents — NONE

### Silver resources

#### Silver EHRbase

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — WRITE *(Configures/Uploads openEHR Templates via CKM)*
  * Data engineer — READ
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — WRITE *(Ingests clinical payloads)*
  * Silver to gold processor — READ *(Extracts for OMOP dbt pipeline)*
  * Ingestion gateway — NONE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE

#### S3 silver bucket

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — READ
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — WRITE *(Saves compressed/validated files)*
  * Silver to gold processor — READ
  * Ingestion gateway — NONE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE

#### Silver audit ledger

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — READ
  * Security & compliance officer — READ
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — WRITE (writes events)
  * Silver to gold processor — READ (checks status)
  * Ingestion gateway — NONE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE

#### Silver metadata catalogue

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — READ
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — WRITE
  * Silver to gold processor — READS *(pulls features to include in Gold)*
  * Ingestion gateway — NONE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE

### Gold resources

#### Gold identity bridge

* Human roles:
  * Data provider — NONE
  * Researcher — READ *(ABAC enforced)*
  * Clinical data modeler — NONE
  * Data engineer — READ
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — NONE
  * Silver to gold processor — WRITE
  * Ingestion gateway — NONE
  * Serving gateway — READ
  * Submission gateway — WRITE
  * AI agents — READ

#### Gold OMOP

* Human roles:
  * Data provider — NONE
  * Researcher — READ (ABAC enforced: e.g., rg__oncology schema only)
  * Clinical data modeler — NONE
  * Data engineer — WRITE *(Deploys dbt schema updates)*
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — NONE
  * Silver to gold processor — WRITE *(dbt executes flat table upserts)*
  * Ingestion gateway — NONE
  * Serving gateway — READ *(ATLAS/Athena)*
  * Submission gateway — NONE
  * AI agents — READ

#### Gold FHIR

* Human roles:
  * Data provider — NONE
  * Researcher — READ
  * Clinical data modeler — NONE
  * Data engineer — READ
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — NONE
  * Silver to gold processor — WRITE
  * Ingestion gateway — NONE
  * Serving gateway — READ
  * Submission gateway — NONE
  * AI agents — READ

#### Gold derived data

* Human roles:
  * Data provider — NONE
  * Researcher — READ/WRITE (ABAC enforced: Strictly inside s3://gold__my_group/)
  * Clinical data modeler — NONE
  * Data engineer — NONE
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — NONE
  * Silver to gold processor — NONE
  * Ingestion gateway — NONE
  * Serving gateway — READ *(Delivers Pre-Signed URLs)*
  * Submission gateway — WRITE *(Routes researcher uploads here)*
  * AI agents — READ

#### Gold vocabulary mapping

* Human roles:
  * Data provider — NONE
  * Researcher — READ
  * Clinical data modeler — READ/WRITE *(Curates mappings via OMOPHub/Athena)*
  * Data engineer — WRITE *(Deploys static lookup tables via dbt)*
  * Security & compliance officer — NONE
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — NONE
  * Silver to gold processor — READ *(Uses lookups during OMOP transformation)*
  * Ingestion gateway — NONE
  * Serving gateway — READ
  * Submission gateway — NONE
  * AI agents — READ

### Control plane

#### OpenMetadata Knowledge Graph

* Human roles:
  * Data provider — NONE
  * Researcher — READ *(Searches for cohorts/datasets)*
  * Clinical data modeler — WRITE *(Updates business glossary/terms)*
  * Data engineer — ADMIN
  * Security & compliance officer — READ *(Views Lineage & Quality Dashboards)*
* Machine roles/services:
  * Landing to bronze processor — NONE
  * Bronze to silver processor — NONE
  * Silver to gold processor — WRITE
  * Ingestion gateway — NONE
  * Serving gateway — NONE
  * Submission gateway — WRITE *(Registers un-supervised researcher uploads)*
  * AI agents — READ *(Via Model Context Protocol / MCP)*

#### MinIO Enterprise KMS (Secrets & Keys)

* Human roles:
  * Data provider — NONE
  * Researcher — NONE
  * Clinical data modeler — NONE
  * Data engineer — NONE
  * Security & compliance officer — READ/WRITE *(Executes key deletion for Right-to-Erasure)*
* Machine roles/services:
  * Landing to bronze processor — READ/WRITE *(Generates & wraps DEK)*
  * Bronze to silver processor — READ *(Unwraps DEK to process file)*
  * Silver to gold processor — NONE
  * Ingestion gateway — NONE
  * Serving gateway — NONE
  * Submission gateway — NONE
  * AI agents — NONE


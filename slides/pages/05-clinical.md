---
class: compact-slide optional-slide

---

# openEHR as clinical source of truth

- **Granular and rich clinical vocabulary**
- **Long-term persistence** - no reference model schema migration
- **AQL** queries clinical concepts, not physical columns
- Automated archetype validation at ingestion


<div class="grid grid-cols-2 gap-4">

<div>

OMOP-only:
- Lossy by definition
- Flattening strips ML-relevant context
- Rigid vocabularies lead to slower data collection (OMOP needs to be defined first) → i.e., waterfall

</div>

<div>

openEHR-only:
- Deep hierarchies → difficult aggregation
- Analytics expect flat tables

</div>

</div>

<div class="flex justify-center mt-4">
  <strong>Dual model: persist in openEHR, serve via OMOP</strong>
</div>

---
class: compact-slide optional-slide

---

# Tooling: Pydantic, CKM, Archetype Designer

- **Pydantic** - transform to EHRbase contract; fix or reject bad EDC rows
- **OpenEHR Clinical Knowledge Manager (CKM)** - reuse international archetypes and templates
- **OpenEHR Archetype Designer** - low-code templates for trial-specific metrics 
- **EHRbase** - fully featured production CDR

---
class: compact-slide
---

# openEHR + OMOP synergy

- **Decouple persistence from analytics** - **EHRbase** (Silver) = clinical source of truth; **OMOP** (Gold) = research serving
- **Gold per research group:** **AQL** extract → **dbt** → staging OMOP → **DQD** → publish (schemas defined and isolated per group)
- **Serving:** **Athena** standard vocabularies + **ATLAS** OMOP CDM exploration to define cohorts
- **Incremental OMOP** - jobs read vetted openEHR, not raw Bronze; build CDMs when a cohort needs them

# S12 – Data Modelling: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `platform/enterprise-data-models.md` — Modelling approach, thin ontology, data contracts, domain-aligned models
- `platform/medallion-architecture.md` — Medallion zones, dimensional zone, naming conventions
- `governance/edap-tagging-strategy.md` — Data contract tags, data product tier tags
- `platform/edap-access-model.md` — Domain catalog structure, product schema, reference catalog
- `specifications/edap-pipeline-framework.md` — Zone transitions, configuration schema

---

## Feature S12-F1: Standards Review and Update

**Description:** Review Water Corporation's existing Data Modelling Standards and Guidelines document (Appendix 7), conduct a gap analysis against modern lakehouse best practices and Databricks capabilities, and produce updated standards endorsed by WC.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S12-F1-US01 | Enterprise Data Architect | review WC's existing Data Modelling Standards against current Databricks and lakehouse best practices | gaps between legacy standards and the EDAP platform capabilities are identified |
| S12-F1-US02 | Enterprise Data Architect | produce a gap analysis documenting where existing standards are insufficient, outdated, or incompatible with EDAP | WC has a clear understanding of what needs to change and why |
| S12-F1-US03 | Data Governance Council | receive updated standards with best practice recommendations for endorsement | the organisation has a current, endorsed set of data modelling standards aligned to EDAP |
| S12-F1-US04 | Data Engineer | reference a single, authoritative modelling standards document | I have clear guidance on naming, typing, history management, and structural patterns when building models in Databricks |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S12-F1-AC01 | WC's existing Data Modelling Standards document is available | the review is completed | a gap analysis document is produced listing each gap with: current standard, identified issue, recommended change, and rationale referencing Databricks/lakehouse best practice |
| S12-F1-AC02 | The gap analysis is complete | updated standards are drafted | the updated document covers: naming conventions, data types (Delta Lake types), SCD handling, surrogate key strategy, liquid clustering guidance, Unity Catalog registration, and column-level documentation requirements |
| S12-F1-AC03 | Updated standards are drafted | the document is presented to the Data Governance Council | the council reviews and endorses (or provides feedback for revision within one PI cycle) |
| S12-F1-AC04 | Standards are endorsed | a data engineer builds a new model in EDAP | the modelling standards document provides unambiguous guidance for the task, and the resulting model can be assessed for compliance against the documented standards |
| S12-F1-AC05 | Modelling tools are evaluated | a recommendation is made to WC | tools that do not require additional licence procurement (e.g. MS Visio, dbdiagram.io, or equivalent) are identified, with rationale for the recommendation |

### Technical Notes
- The enterprise data models wiki advocates domain-aligned models over monolithic EDMs; updated standards should reflect this shift.
- Standards must address Delta Lake-specific considerations: liquid clustering (replacing Z-ordering), deletion vectors, VARIANT columns, schema evolution, and UniForm.
- Modelling tools must not require additional licence procurement per the scope of work; MS Visio, MS Word, or alternatives subject to WC endorsement.
- Standards should reference the EDAP medallion architecture wiki naming conventions for schema-as-layer patterns.

---

## Feature S12-F2: Dimensional Modelling Framework

**Description:** Establish a dimensional modelling framework for the Gold layer, defining patterns for star and snowflake schemas, fact and dimension table design, surrogate key management, and SCD handling in Databricks Delta Lake.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S12-F2-US01 | Data Engineer | follow a standardised framework for building Gold-layer dimensional models | I produce consistent star/snowflake schemas that align with WC's modelling standards |
| S12-F2-US02 | BI Analyst | consume Gold-layer dimensional models with clear facts, dimensions, and hierarchies | I can build dashboards and reports efficiently using well-structured, business-friendly data |
| S12-F2-US03 | Data Engineer | implement SCD Type 2 handling in dimension tables using Delta Lake | I can track historical changes to dimension attributes with effective/expiry timestamps |
| S12-F2-US04 | Data Engineer | use a standardised surrogate key generation approach | dimension tables have consistent, performant surrogate keys decoupled from business keys |
| S12-F2-US05 | Data Engineer | apply pre-defined templates for common dimensional patterns (date dimensions, geographic hierarchies, conformed dimensions) | I can reuse proven patterns rather than building from scratch |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S12-F2-AC01 | The dimensional modelling framework is documented | a data engineer reviews the framework | it provides patterns, examples, and implementation guidance for: star schemas, snowflake schemas, fact tables (transactional, periodic snapshot, accumulating snapshot), and dimension tables |
| S12-F2-AC02 | SCD handling guidance is documented | a data engineer implements an SCD Type 2 dimension in Databricks | the implementation follows the framework's prescribed approach using Delta Lake merge operations or Lakeflow AUTO CDC, with effective/expiry timestamps and current-row indicators |
| S12-F2-AC03 | Surrogate key strategy is defined | dimension tables are built using the framework | each dimension table uses a consistent surrogate key approach (e.g. SHA-256 hash of business key or monotonically increasing ID) documented in the standards |
| S12-F2-AC04 | Dimensional model templates are provided | a data engineer begins building a Gold-layer model for a new use case | they can start from a template that includes: dim_ and fact_ prefixes, standard audit columns, SCD columns, and naming conventions per the medallion wiki |
| S12-F2-AC05 | Conformed dimensions are identified | cross-domain dimensional models are reviewed | shared conformed dimensions (e.g. date, geography, organisational unit) are registered in the `prod_reference` catalog and reused across domain models |
| S12-F2-AC06 | Gold Dimensional zone guidance is documented | a model is deployed to the `product` schema | it follows the dimensional zone specifications from the medallion architecture wiki: surrogate keys, SCD Type 2, pre-aggregated measures, predefined hierarchies |

### Technical Notes
- Gold Dimensional zone follows the medallion architecture wiki: star/snowflake schemas with surrogate keys, SCD Type 2, pre-aggregated measures, and predefined hierarchies.
- Table naming uses `dim_`, `fact_`, `agg_` prefixes per the medallion architecture wiki naming conventions.
- Conformed dimensions reside in `prod_reference` catalog per the access model wiki and enterprise data models wiki.
- Liquid clustering should be applied to large fact tables on frequently filtered columns per the medallion architecture wiki.
- Materialized views are recommended for frequently accessed Gold aggregations.

---

## Feature S12-F3: Conceptual and Logical Models

**Description:** Develop domain-aligned conceptual and logical data models for each of WC's seven data domains, define shared business entity definitions, and maintain a thin enterprise ontology in the `prod_reference` catalog.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S12-F3-US01 | Domain Data Steward | have a conceptual model for my domain that defines core business entities and their relationships | I have a shared understanding of my domain's data landscape for governance and communication |
| S12-F3-US02 | Enterprise Data Architect | maintain a thin enterprise ontology defining critical shared concepts (Customer, Asset, Product, Location) and their key relationships | cross-domain discovery and interoperability are enabled without a monolithic enterprise data model |
| S12-F3-US03 | Data Engineer | reference logical models that define entity attributes, data types, relationships, and business rules for each domain | I have clear specifications for implementing physical models in Databricks |
| S12-F3-US04 | Business Analyst | access business entity definitions in Alation (enterprise catalogue) | I can discover and understand data assets using business-meaningful terminology |
| S12-F3-US05 | Domain Data Steward | contribute to and maintain the business glossary terms for my domain | business definitions are accurate, current, and owned by the people closest to the data |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S12-F3-AC01 | Conceptual models are created | each of WC's seven data domains is reviewed | a conceptual model exists for each domain showing core entities (minimum 5 per domain), their key relationships, and a textual description of each entity |
| S12-F3-AC02 | The thin enterprise ontology is defined | the `prod_reference` catalog is reviewed | it contains cross-domain reference data (shared dimensions, code tables, geographic hierarchies) with documented entity definitions and ownership by Architecture & Strategy |
| S12-F3-AC03 | Logical models are produced | a data engineer reviews the logical model for a domain | it includes entity attributes with data types, primary/foreign key relationships, business rules, and mapping to source system fields |
| S12-F3-AC04 | Models are created using approved tools | all conceptual and logical models are delivered | they are produced using tools that do not require additional licence procurement (e.g. MS Visio, MS Word) as specified in the scope of work |
| S12-F3-AC05 | Business glossary terms are defined | domain stewards review the glossary for their domain | each core entity and its key attributes have approved business definitions registered in Alation |
| S12-F3-AC06 | Models align to water utility standards | domain models are reviewed | consideration is given to water utility industry standard common data models and best practices as required by the scope of work |

### Technical Notes
- The enterprise data models wiki explicitly advocates a thin enterprise ontology over a monolithic EDM: "a shared set of business terms, domain boundaries, and critical entity definitions" kept "conceptual and lightweight."
- `prod_reference` catalog implements the thin enterprise ontology per the access model wiki Section 3.3 and the enterprise data models wiki Section 8.
- Domain models crystallise at Silver (domain-aligned) and Gold (consumption-aligned) per the enterprise data models wiki Section 5.2: "Bronze stays source-aligned; Silver is domain-aligned."
- Business glossary integration with Alation supports the enterprise data models wiki's principle that "the data catalogue becomes the connective tissue that the EDM was supposed to be."

---

## Feature S12-F4: Physical Implementation in Databricks

**Description:** Implement physical data models in Databricks using Delta tables with liquid clustering, register all tables in Unity Catalog with comprehensive metadata (column descriptions, table comments, governed tags), and enforce modelling standards at the platform level.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S12-F4-US01 | Data Engineer | create Delta tables that implement the approved logical models with correct data types, constraints, and liquid clustering | physical models are performant, queryable, and aligned to the logical design |
| S12-F4-US02 | Data Engineer | register all tables in Unity Catalog with table comments and column descriptions | data assets are discoverable and understandable by both human users and AI assistants |
| S12-F4-US03 | Domain Data Steward | verify that governed tags are applied to all registered tables | ABAC policies and classification governance are enforced consistently |
| S12-F4-US04 | Data Analyst | discover tables in Unity Catalog with clear descriptions of what each table and column contains | I can find and understand the right data without needing to ask the engineering team |
| S12-F4-US05 | Platform Administrator | enforce modelling standards programmatically (e.g. naming convention checks, mandatory tag presence) | non-compliant models are detected before deployment to production |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S12-F4-AC01 | A logical model has been approved | the physical model is implemented in Databricks | Delta tables are created with correct column data types matching the logical model, NOT NULL constraints where specified, and CHECK constraints where applicable |
| S12-F4-AC02 | Delta tables are created | liquid clustering is applied | large tables (>1M rows) have liquid clustering configured on frequently filtered columns, replacing legacy Z-ordering |
| S12-F4-AC03 | Tables are registered in Unity Catalog | a table is queried in Catalog Explorer | it has a table comment describing its purpose and business context, and every column has a description |
| S12-F4-AC04 | Governed tags are required | a table is deployed to a production catalog | it has at minimum `sensitivity`, `domain`, and `data_product_tier` tags applied per the tagging strategy |
| S12-F4-AC05 | Naming convention standards are defined | a CI/CD pipeline runs before production deployment | a validation step checks table and column names against the modelling standards and rejects non-compliant names |
| S12-F4-AC06 | Predictive optimisation is enabled | a production catalog is monitored over 30 days | OPTIMIZE, VACUUM, and ANALYZE TABLE operations are executed automatically by predictive optimisation without manual scheduling |

### Technical Notes
- Liquid clustering replaces Z-ordering as the recommended data layout optimisation per the medallion architecture wiki.
- Predictive optimisation handles OPTIMIZE, VACUUM, and ANALYZE automatically per the medallion architecture wiki.
- Table and column comments are critical for AI/BI assistants (Genie spaces) and Alation integration per the enterprise data models wiki Section 7.
- Governed tags must follow the tagging strategy wiki's allowed values; tag application is explicit per object, never automatically inherited.
- Delta Lake UniForm may be enabled to provide Iceberg interoperability per the medallion architecture wiki.

---

## Feature S12-F5: Data Contracts and Product Definitions

**Description:** Implement data contracts for Gold-layer data products defining schema, SLA tiers, quality thresholds, and ownership, aligned to the data contract tags in the tagging strategy and the data product tier classification.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S12-F5-US01 | Data Product Owner | define a data contract for my data product specifying schema, freshness SLA, quality thresholds, and ownership | consumers have clear expectations and I have a defined standard to maintain |
| S12-F5-US02 | Data Consumer | discover data products in the Unity Catalog with their contract terms | I can evaluate whether a data product meets my needs before building a dependency on it |
| S12-F5-US03 | Data Product Owner | classify my data product using the `data_product_tier` tag (certified, provisional, experimental, deprecated) | consumers can assess the maturity and reliability of the product |
| S12-F5-US04 | Technical Data Steward | monitor data products against their contracted SLAs and quality thresholds | SLA breaches are detected and escalated automatically |
| S12-F5-US05 | Data Product Owner | version my data contract and manage schema evolution with backward compatibility | consumers are protected from breaking changes and have time to adapt to non-breaking changes |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S12-F5-AC01 | A data contract template is defined | a data product owner creates a contract for a new product | the contract includes: schema definition (field names, types, nullability, primary/foreign keys), quality expectations (completeness, uniqueness, referential integrity thresholds), freshness SLA (update frequency, maximum staleness), ownership (producing team, contact, escalation path), and versioning policy |
| S12-F5-AC02 | A data product is published to the `product` schema | its Unity Catalog registration is reviewed | it carries `data_product_tier`, `sensitivity`, and `domain` governed tags per the tagging strategy |
| S12-F5-AC03 | SLA tiers are defined | the SLA monitoring framework is configured | each SLA tier (e.g. Tier 1: <1hr freshness, Tier 2: <4hr, Tier 3: <24hr) has automated monitoring with alerting on breach |
| S12-F5-AC04 | Quality thresholds are defined in a contract | the data quality monitoring runs | metrics (completeness ≥99%, uniqueness on key columns = 100%, null rate on mandatory columns = 0%) are measured and results stored in `prod_platform` |
| S12-F5-AC05 | Schema versioning is implemented | a non-breaking schema change is applied to a data product (e.g. adding a nullable column) | the change is applied without consumer disruption and the contract version is incremented |
| S12-F5-AC06 | A data product is deprecated | the `data_product_tier` tag is set to `deprecated` | consumers are notified, a deprecation timeline is communicated, and the product is removed after the agreed notice period |

### Technical Notes
- Data contracts replace upfront schema agreement per the enterprise data models wiki Section 5.4: "explicit, versioned agreements between data producers and consumers."
- Data product tiers (certified, provisional, experimental, deprecated) are governed tags per the tagging strategy wiki.
- The `product` schema in each domain catalog is the Gold layer for certified data products per the access model wiki Section 3.3.
- Quality thresholds should be enforced using Lakeflow expectations and monitored via system tables per the pipeline framework spec.
- Data contracts should be stored as structured metadata (e.g. Delta table in `prod_platform`) to enable programmatic validation and reporting.
- The FAUQD test (referenced in the access model wiki) should be the certification gate for data products moving from provisional to certified status.

# Enterprise Data Analytics Platform

## Configurable Pipeline Framework â Requirements Specification

| | |
|---|---|
| **Document ID** | EDAP-FWK-001 |
| **Version** | 0.4 (Draft) |
| **Date** | March 2026 |
| **Author** | IT Architecture & Strategy |
| **Classification** | Internal |
| **Status** | Draft for Review |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Architectural Context](#2-architectural-context)
3. [Generic Engine Requirements](#3-generic-engine-requirements)
4. [Zone-Specific Processing Rules](#4-zone-specific-processing-rules)
5. [Non-Functional Requirements](#5-non-functional-requirements)
6. [Configuration Schema](#6-configuration-schema)
7. [Processing Flow](#7-processing-flow)
8. [Acceptance Criteria](#8-acceptance-criteria)
9. [Open Questions and Decisions](#9-open-questions-and-decisions)
10. [Appendix A â DLT-META Gap Analysis](#appendix-a--dlt-meta-gap-analysis)
11. [Document History](#11-document-history)

---

## 1. Introduction

### 1.1 Purpose

This document defines the functional and non-functional requirements for a configurable pipeline framework that automates the movement and transformation of data across the layers and zones of the Enterprise Data Analytics Platform (EDAP).

The framework must provide a metadata-driven, zero-code or low-code mechanism to onboard new source entities through configuration rather than bespoke pipeline development. It will be implemented using Databricks Lakeflow Streaming Declarative Pipelines (SDP) and governed through Unity Catalog.

While the framework is designed to be general-purpose across the EDAP medallion architecture, the **primary delivery focus** is the Raw Zone (Bronze) to Base Zone (Silver) transition. This zone transition is the most common, most standardised, and highest-volume processing pattern in the platform. Subsequent zone transitions (Base-to-Enriched, Enriched-to-Gold) will be delivered incrementally, reusing the same engine with zone-specific configuration.

### 1.2 Scope

The framework covers configurable pipeline processing across the following zone transitions:

| Zone Transition | Source â Target | Delivery Priority |
|---|---|---|
| Raw â Base | Bronze Raw â Silver Base | **Phase 1 (primary)** |
| Base â Enriched | Silver Base â Silver Enriched | Phase 2 |
| Enriched â Exploratory | Silver Enriched â Gold Exploratory | Phase 3 |
| Enriched â BI | Silver Enriched â Gold BI | Phase 3 |

The framework addresses the following capabilities as a generic engine:

- Reading from source Delta tables using streaming or batch reads, with support for multiple ingestion methods (file-based, managed connector, streaming).
- Applying configurable data type casting, field renaming, null handling, and structural flattening.
- Performing deduplication and change detection using hash-based comparison or CDC event processing.
- Applying configurable data quality rules and annotating records with DQ status columns.
- Managing record history via configurable SCD Type 1 or Type 2 processing.
- Populating zone-appropriate audit columns.
- Writing output to target Delta tables registered in Unity Catalog.
- Applying standard tags as defined in the EDAP tagging strategy.

**Out of scope:** Landing Zone file arrival, Bronze Raw Zone ingestion (Lakeflow Connect/Auto Loader), Sandbox provisioning, and BI semantic layer configuration.

### 1.3 Audience

This document is intended for the IT Architecture Review Board (ARB), data platform engineers, solution architects, and delivery teams within the SAFe delivery environment responsible for implementing and consuming the EDAP.

### 1.4 Definitions and Acronyms

| Acronym | Definition |
|---|---|
| EDAP | Enterprise Data Analytics Platform |
| SDP | Streaming Declarative Pipeline (Lakeflow) |
| DQ | Data Quality |
| SCD | Slowly Changing Dimension |
| CDC | Change Data Capture |
| PI | Personal Information |
| ARB | Architecture Review Board |
| UC | Unity Catalog |
| SAFe | Scaled Agile Framework |
| DABs | Databricks Asset Bundles |
| AUTO CDC | Lakeflow API for applying change data capture (replaces APPLY CHANGES) |
| Dataflowspec | A Delta table that holds runtime pipeline specifications, as defined in the DLT-META pattern |
| PRIS Act | Privacy and Responsible Information Sharing Act 2024 (WA) |
| SOCI Act | Security of Critical Infrastructure Act 2018 (Cth) |

### 1.5 Referenced Documents

| Document | Reference |
|---|---|
| EDAP Medallion Architecture Specification | EDAP-ARCH-001 |
| EDAP Data Governance Tagging Framework | EDAP-GOV-001 |
| Development Environment Data Strategy | ADR-EDP-001 |
| DLT-META Documentation | https://databrickslabs.github.io/dlt-meta/ |
| Lakeflow AUTO CDC APIs | https://learn.microsoft.com/en-us/azure/databricks/ldp/cdc |
| Lakeflow DQ Expectations | https://docs.databricks.com/aws/en/ldp/expectations |

---

## 2. Architectural Context

### 2.1 Medallion Architecture Alignment

The framework operates across the EDAP medallion architecture, processing data through progressively refined layers:

- **Bronze (Raw Zone):** Immutable, append-only system of record. Source columns may be loosely typed (STRING, VARIANT, BINARY) for file-based ingestion or natively typed for connector-based ingestion.
- **Silver (Base Zone):** Cleansed, validated, deduplicated, and historised data structurally aligned with source systems. First layer of enterprise-grade quality.
- **Silver (Enriched Zone):** Integrated, cross-source data with business logic applied. Combines multiple Base Zone entities with documented join logic.
- **Gold (Exploratory Zone):** Wide, denormalised datasets for data science and ad hoc analysis.
- **Gold (BI Zone):** Dimensional models (facts and dimensions) with KPIs for BI and reporting.

The framework provides a single configurable engine that handles the pipeline processing for each zone transition. The engine's behaviour is parameterised by the zone transition being performed â the same framework code executes different transformation, DQ, audit, and history logic depending on the configured source and target zones.

### 2.2 Unity Catalog Namespace Convention

The framework must adhere to the EDAP naming conventions for Unity Catalog objects:

| Zone Transition | Source Pattern | Target Pattern |
|---|---|---|
| Raw â Base | `<env>_bronze.<source_system>_raw.<entity>` | `<env>_silver.<source_system>_base.<entity>` |
| Base â Enriched | `<env>_silver.<source_system>_base.<entity>` | `<env>_silver.<domain>_enriched.<entity>` |
| Enriched â BI | `<env>_silver.<domain>_enriched.<entity>` | `<env>_gold.<domain>_bi.<entity>` |
| Enriched â Exploratory | `<env>_silver.<domain>_enriched.<entity>` | `<env>_gold.<domain>_exploratory.<entity>` |

### 2.3 Pipeline Technology

The framework must be implemented using Databricks Lakeflow Streaming Declarative Pipelines (formerly Delta Live Tables). Pipeline logic is expressed using `CREATE STREAMING TABLE`, `CREATE MATERIALIZED VIEW`, and the AUTO CDC APIs in SQL or Python â not custom YAML or imperative Spark code.

**API migration note:** Databricks now recommends the AUTO CDC APIs (`AUTO CDC INTO`, `create_auto_cdc_flow`) as replacements for the APPLY CHANGES APIs. Both have the same syntax. The EDAP framework should target the AUTO CDC APIs for new development while remaining compatible with environments where only the APPLY CHANGES APIs are available.

**Compute model:** Lakeflow Declarative Pipelines supports both serverless compute and classic (job cluster) compute. Serverless compute eliminates cluster management overhead, provides faster startup times, and simplifies cost attribution. For the EDAP framework, serverless compute is the **recommended default** for new pipelines, subject to the following considerations:

- Serverless is appropriate for the majority of Raw-to-Base and Base-to-Enriched pipelines where compute requirements are standard.
- Classic compute should be used where pipelines require specific Spark configurations not available in serverless mode, custom library dependencies, or deterministic cluster sizing for cost-sensitive workloads.
- The choice between serverless and classic should be configurable per data flow group in the Dataflowspec, defaulting to serverless.
- Cost observability (NFR-CST-01, NFR-CST-02) applies equally to both compute models.

**Pipeline chaining constraint:** Lakeflow SDP currently does not allow a streaming table created via AUTO CDC (SCD Type 2) to be used as a streaming source in a downstream pipeline without enabling `skipChangeCommits`. This constraint affects Base â Enriched pipeline chaining where the Base Zone table uses SCD Type 2. The framework must account for this â either by setting `skipChangeCommits`, reading the target as a batch source, or structuring pipelines to avoid streaming from SCD Type 2 targets.

### 2.4 Raw Zone Ingestion Patterns

The framework must accommodate variation in how data arrives in the Raw Zone, as this affects the transformation logic required for the Raw â Base transition:

| Ingestion Method | Typical Column Types | File Audit Columns | Examples |
|---|---|---|---|
| Auto Loader (file-based) | STRING, VARIANT, BINARY | Present (edap.file_name, edap.file_path, etc.) | CSV, JSON, Parquet, XML file drops |
| Lakeflow Connect (managed connector) | Native types from source schema (INTEGER, TIMESTAMP, DECIMAL, etc.) | Not present â no file-based origin | SAP, Salesforce, database CDC connectors |
| Streaming / event-based | STRING or VARIANT (serialised payloads) | May or may not be present | Kafka, Event Hubs, SCADA telemetry |

The framework's type casting logic must introspect the source schema at runtime and apply casting only where the source type differs from the configured target type, rather than unconditionally casting all columns from STRING.

### 2.5 Design Decision â Internal Pipeline Staging vs Separate Zone

A common pattern in medallion architectures is to introduce an intermediate zone (sometimes called "Standardised" or "Conformed") between Raw and Base to handle structural transformations such as flattening nested JSON/VARIANT payloads, exploding arrays, and normalising field names before DQ and SCD processing.

This framework does **not** introduce a separate published zone for this purpose. The rationale is:

- **Many sources arrive flat or natively typed.** Entities ingested via Lakeflow Connect, flat CSV/Parquet files, or tabular CDC streams require no structural transformation. A separate zone would be a pass-through copy for these entities â adding storage cost, latency, governance overhead, and lineage noise with no value.
- **The Raw Zone already provides the replay checkpoint.** Since Bronze is immutable and append-only, any reprocessing can replay from Raw. A separate intermediate zone does not add meaningful replay capability beyond what Raw already provides.
- **A separate zone increases the configuration and operational surface.** Every entity would require an additional target table, retention policy, monitoring configuration, and access control definition, even when no structural transformation is needed.

Instead, the framework supports **internal pipeline staging** â where entities that require complex structural transformation (deep nesting, array explosion, multi-level flattening) can declare intermediate `CREATE STREAMING TABLE` steps *within the same SDP pipeline*. These internal staging tables are managed within the pipeline graph and visible in Lakeflow lineage, but are not published to consumers.

### 2.6 Relationship to DLT-META

DLT-META is an open-source metadata-driven framework from Databricks Labs that automates Bronze and Silver pipeline generation using Lakeflow Declarative Pipelines. It provides a proven reference pattern for several capabilities this framework requires. A detailed gap analysis is provided in [Appendix A](#appendix-a--dlt-meta-gap-analysis).

**DLT-META capabilities that align with EDAP requirements:**

- **Dataflowspec pattern:** Translates onboarding JSON into a Delta "Dataflowspec" table, providing both version control (JSON files) and runtime discoverability (Delta table). This two-stage pattern should be adopted by the EDAP framework.
- **Data flow grouping:** The `data_flow_group` property allows multiple entities to be co-executed within a single Lakeflow Declarative Pipeline, balancing operational efficiency with failure isolation.
- **CDC support:** `create_auto_cdc_flow` and `create_auto_cdc_from_snapshot_flow` APIs with SCD Type 1 and Type 2, `apply_as_deletes`, `sequence_by`, `except_column_list`, and `track_history_column_list`.
- **DQ expectations:** Both Bronze and Silver layers support Lakeflow expectations (`expect`, `expect_or_fail`, `expect_or_drop`, `expect_or_quarantine`).
- **Quarantine table support:** Routes DQ failures to separate quarantine tables at both Bronze and Silver layers (since v0.0.10).
- **Append flows:** Multiple source streams writing to the same target table via Lakeflow `append_flow` API.
- **Sink API support:** Lakeflow `create_sink` for writing processed data to external Delta tables or Kafka topics.
- **Custom transformation hooks:** Python functions injectable at both Bronze and Silver layers, with ordered execution (`transformation_functions` array).
- **Source metadata extraction:** Configurable extraction of Auto Loader `_metadata` columns (file_name, file_path) into Bronze target columns.
- **Snapshot CDC:** `apply_changes_from_snapshot` API for full extract sources where CDC feeds are not available.
- **Tooling and deployment:** CLI (`databricks labs dlt-meta onboard`, `databricks labs dlt-meta deploy`), Lakehouse App UI, and Databricks Asset Bundles support.

**Gaps where EDAP must extend beyond DLT-META:**

- **EDAP-specific audit columns.** DLT-META does not prescribe edap_-prefixed audit columns, effectivity tracking conventions, or SHA-512 hash-based change detection.
- **Per-record DQ annotation model.** DLT-META uses native Lakeflow expectations which operate at the pipeline level (pass/warn/fail/drop/quarantine). EDAP requires per-record DQ annotation columns (dq_status, dq_errors, dq_warnings, dq_checked_ts) that propagate all records with their individual quality assessment.
- **Column-level configuration depth.** DLT-META's Silver transformation uses SQL `select_exp` arrays applied via `selectExpr()`. EDAP requires richer column-level configuration including per-column type casting, null handling rules, flatten paths, and per-column DQ rules.
- **Configuration validation.** DLT-META's onboarding process has limited schema validation â malformed JSON can produce unclear runtime errors.
- **Unity Catalog tagging.** DLT-META does not manage UC tags.
- **Multi-zone scope.** DLT-META covers Bronze and Silver. EDAP requires the same engine to handle Silver-to-Gold transitions.
- **Table and column comments.** DLT-META supports table comments; the EDAP framework should extend this to column-level comments for Unity Catalog discoverability.
- **No formal SLA.** DLT-META is a Databricks Labs project provided AS-IS without SLAs.

The design phase should determine whether to adopt, reference, or build independently. This is captured as OQ-01. See [Appendix A](#appendix-a--dlt-meta-gap-analysis) for the full analysis and adoption strategy options.

---

## 3. Generic Engine Requirements

These requirements apply to the framework engine regardless of which zone transition is being performed.

### 3.1 Configuration-Driven Pipeline Generation

| ID | Requirement | Priority |
|---|---|---|
| FR-CFG-01 | The framework shall accept a declarative configuration file (JSON or YAML) that defines the source-to-target mapping for each entity, including source catalog/schema/table, target catalog/schema/table, column mappings, data type casts, business key definition, and zone transition type. | Must Have |
| FR-CFG-02 | The framework shall translate the declarative configuration into a runtime Dataflowspec Delta table (following the DLT-META two-stage pattern) that serves as the authoritative runtime specification for the pipeline engine. This provides both version control (source files) and runtime discoverability (Delta table with full query, lineage, and audit capability). | Must Have |
| FR-CFG-03 | The framework shall generate valid Lakeflow SDP SQL or Python pipeline code from the Dataflowspec at runtime, without requiring manual code changes per entity. | Must Have |
| FR-CFG-04 | The configuration shall support environment parameterisation (dev, test, prod) to resolve Unity Catalog catalog names dynamically, using environment-suffixed properties (e.g. `source_catalog_dev`, `source_catalog_prod`) or a templating mechanism. | Must Have |
| FR-CFG-05 | The framework shall support a configuration versioning mechanism so that changes to entity mappings are auditable and traceable. The Dataflowspec table shall include version, import_author, and import_timestamp metadata. | Must Have |
| FR-CFG-06 | The configuration shall support defining the source extract pattern for each entity: full extract (snapshot), incremental (CDC/append), or event-based. | Must Have |
| FR-CFG-07 | The configuration shall support specifying the ingestion method (file_based, managed_connector, streaming) for Raw â Base transitions, to determine which audit columns to expect and whether type casting is required. | Must Have |
| FR-CFG-08 | The configuration shall support a data flow grouping mechanism (`data_flow_group`) that allows multiple entities to be assigned to the same Lakeflow Declarative Pipeline, enabling co-execution while preserving the option for per-entity pipeline isolation where required. | Must Have |
| FR-CFG-09 | The framework shall validate the configuration schema at onboarding time (required fields, correct types, referential integrity between entity and column definitions, valid zone transition types). Malformed or incomplete configuration shall produce clear, actionable error messages and shall not generate invalid pipelines. | Must Have |
| FR-CFG-10 | The configuration shall support specifying reader options per entity (e.g. multiline, header, rescued data column, infer column types) to be passed through to the underlying Spark reader or Auto Loader configuration. | Should Have |
| FR-CFG-11 | The configuration shall support specifying columns to exclude from the edap_hash calculation. | Must Have |
| FR-CFG-12 | The configuration shall identify the zone transition type for each entity (e.g. raw_to_base, base_to_enriched, enriched_to_bi, enriched_to_exploratory), which determines the applicable audit columns, DQ behaviour, and history management pattern. | Must Have |
| FR-CFG-13 | The configuration shall support specifying table properties per entity (e.g. `pipelines.reset.allowed`, `pipelines.autoOptimize.managed`) to be passed through to the Lakeflow `CREATE STREAMING TABLE` declaration. | Should Have |
| FR-CFG-14 | The configuration shall support specifying a table comment per entity, to be published to Unity Catalog for data catalogue discoverability. | Should Have |
| FR-CFG-15 | The framework shall provide an onboarding workflow (CLI command, notebook, or API) that validates the configuration, translates it into the Dataflowspec Delta table, and optionally triggers pipeline deployment. Adding or modifying an entity shall not require downtime for existing pipeline entities. | Must Have |

### 3.2 Data Type Casting and Structural Transformation

| ID | Requirement | Priority |
|---|---|---|
| FR-TYP-01 | The framework shall introspect the source table schema at pipeline initialisation and determine which columns require type casting based on the difference between the source type and the configured target type. | Must Have |
| FR-TYP-02 | Where the source column type already matches the configured target type (e.g. connector-ingested data or Base â Enriched transitions), the framework shall pass the column through without casting. | Must Have |
| FR-TYP-03 | Where the source column type differs from the target type, the framework shall cast the column and handle cast failures gracefully â records with cast errors shall be annotated via DQ columns and propagated, not dropped. | Must Have |
| FR-TYP-04 | The framework shall flatten nested VARIANT/JSON structures into individual typed columns as defined in the configuration mapping. | Must Have |
| FR-TYP-05 | The framework shall support configurable null handling rules per column (e.g. replace null with default value, flag as DQ warning, flag as DQ error). | Must Have |
| FR-TYP-06 | The framework shall apply standardised field naming conventions to all output columns, converting source field names to consistent, clear, and descriptive names as specified in configuration. | Must Have |
| FR-TYP-07 | The framework shall handle source schema evolution gracefully. If a source table contains columns not present in the configuration, the framework shall log a warning and either ignore or capture the unmapped columns, based on a configurable policy (ignore, capture, warn). | Should Have |
| FR-TYP-08 | For entities requiring complex structural transformation (multi-level VARIANT flattening, array explosion, multi-step normalisation), the framework shall support declaring one or more internal staging tables within the SDP pipeline using `CREATE STREAMING TABLE`. These are internal to the pipeline, not published to consumers, and serve as debuggable checkpoints. | Must Have |
| FR-TYP-09 | Internal staging tables shall only be created for entities where the configuration explicitly declares them. Flat or simple entities proceed directly through the core transformation steps. | Must Have |
| FR-TYP-10 | Internal staging tables shall be visible in Lakeflow pipeline lineage for engineer inspection but shall not appear in consumer-facing Unity Catalog schemas. | Should Have |
| FR-TYP-11 | The framework shall support a `_rescued_data` column pattern. Where Auto Loader captures unmatched columns into a rescue column, the framework shall propagate this column to the target (or flag it via DQ annotation) rather than silently dropping it. | Should Have |
| FR-TYP-12 | The framework shall support configurable SQL `select` expressions per entity (analogous to DLT-META's `select_exp` array), enabling transformation logic beyond simple column mapping. Multiple `select_exp` arrays shall be supported where multi-step `selectExpr` evaluation is needed (e.g. explode followed by column selection â a known DLT-META limitation). | Should Have |
| FR-TYP-13 | The framework shall support optional `where_clause` filter conditions per entity. Where applied, filtered records shall be logged for auditability. | Could Have |
| FR-TYP-14 | The framework shall support column-level comments in configuration, to be published to Unity Catalog for data catalogue discoverability. | Should Have |

### 3.3 Deduplication and Change Detection

| ID | Requirement | Priority |
|---|---|---|
| FR-DDP-01 | The framework shall compare the edap_hash of incoming records against the current version in the target table to detect genuine data changes. | Must Have |
| FR-DDP-02 | For full extract (snapshot) sources, the framework shall detect records present in the previous snapshot but absent in the current snapshot and mark them as soft-deleted (edap_is_deleted = TRUE). | Must Have |
| FR-DDP-03 | For incremental (CDC) sources, the framework shall process change events using the Lakeflow AUTO CDC API (`create_auto_cdc_flow` or `AUTO CDC INTO`). Where only the APPLY CHANGES API is available, the framework shall fall back transparently. | Must Have |
| FR-DDP-04 | The framework shall handle duplicate records within the same batch by retaining only one record per business key per batch. | Must Have |
| FR-DDP-05 | The framework shall carry the edap_batch column forward from the source for lineage traceability. | Must Have |
| FR-DDP-06 | The framework shall support the Lakeflow `create_auto_cdc_from_snapshot_flow` API for full extract (snapshot) sources where CDC feeds are not available, enabling efficient snapshot-to-snapshot diff processing without manual diff logic. | Should Have |
| FR-DDP-07 | The framework shall support append flow patterns (`append_flow` API), allowing multiple source streams to write to the same target table. | Could Have |
| FR-DDP-08 | For initial hydration of a CDC-enabled entity, the framework shall support loading the initial full dataset using a `once` flow, followed by continuous CDC processing. | Should Have |

### 3.4 SCD History Management

| ID | Requirement | Priority |
|---|---|---|
| FR-SCD-01 | The framework shall implement SCD Type 2 history tracking using edap_eff_from, edap_eff_to, and edap_is_current columns, configurable per entity. | Must Have |
| FR-SCD-02 | When a record changes (different edap_hash for the same business key), the framework shall close the current version and insert a new version with appropriate effectivity timestamps. | Must Have |
| FR-SCD-03 | When a record is soft-deleted, the framework shall close the current version and insert a new version with edap_is_deleted = TRUE and edap_is_current = TRUE. | Must Have |
| FR-SCD-04 | If a previously deleted record reappears, the framework shall create a new current version with edap_is_deleted = FALSE. | Must Have |
| FR-SCD-05 | The framework shall use the Lakeflow AUTO CDC APIs with SCD Type 2 tracking mode where applicable. The EDAP audit columns (edap_eff_from, edap_eff_to, edap_is_current) must be reconciled with or mapped from the Lakeflow-managed `__START_AT` and `__END_AT` columns. | Must Have |
| FR-SCD-06 | The framework shall support configurable `sequence_by` column(s) per entity for CDC conflict resolution, including handling of out-of-order events. | Must Have |
| FR-SCD-07 | The configuration shall support `except_column_list` and/or `track_history_column_list` to control which columns are included in or excluded from change tracking. | Must Have |
| FR-SCD-08 | The framework shall support SCD Type 1 (overwrite, no history) as a configurable alternative to SCD Type 2, selectable per entity. When SCD Type 1 is selected, effectivity columns are not populated. | Should Have |
| FR-SCD-09 | The configuration shall support an `apply_as_deletes` expression to identify which CDC events represent logical deletes. | Must Have |
| FR-SCD-10 | The framework shall handle the `skipChangeCommits` constraint when downstream pipelines read from SCD Type 2 targets as streaming sources. The configuration or pipeline generation logic shall manage this automatically where applicable. | Should Have |

### 3.5 Data Quality Annotation

| ID | Requirement | Priority |
|---|---|---|
| FR-DQ-01 | The framework shall evaluate DQ rules from configuration and populate dq_status (PASS, WARN, FAIL), dq_errors (ARRAY\<STRING\>), dq_warnings (ARRAY\<STRING\>), and dq_checked_ts on every output record. This per-record annotation model operates **in addition to** (not instead of) Lakeflow pipeline-level expectations. | Must Have |
| FR-DQ-02 | DQ rules shall be configurable per entity and per column, supporting: not null, data type validity, value range, allowed values, regex pattern match, cross-column consistency, uniqueness, and SQL expression. | Must Have |
| FR-DQ-03 | Each DQ rule shall be classifiable as error (contributes to FAIL) or warning (contributes to WARN). No errors or warnings = PASS. | Must Have |
| FR-DQ-04 | The framework shall populate dq_checked_ts with the evaluation timestamp. | Must Have |
| FR-DQ-05 | All records shall propagate to the target regardless of DQ status. No records filtered or quarantined by default. This is the EDAP "annotate, never filter" principle. | Must Have |
| FR-DQ-06 | The framework shall support referencing Unity Catalog tables or views for allowed-value lookups. | Should Have |
| FR-DQ-07 | The framework shall support defining DQ rules as Lakeflow SDP expectations (`expect`, `expect_or_fail`, `expect_or_drop`) in parallel with per-record annotation, to leverage built-in DQ dashboards and event log metrics. | Should Have |
| FR-DQ-08 | The framework shall optionally support a quarantine table pattern using the `expect_or_quarantine` expectation type. When enabled, quarantined records are routed to a separate Delta table and still annotated with DQ columns. This extends the default annotate-all pattern, not replaces it. | Could Have |
| FR-DQ-09 | DQ rules shall support SQL expression syntax (e.g. `amount > 0`) in addition to structured rule types, aligning with Lakeflow expectations syntax. | Should Have |
| FR-DQ-10 | The framework shall support querying DQ metrics from the Lakeflow pipeline event log for programmatic reporting and alerting. | Should Have |

### 3.6 Audit Column Management

| ID | Requirement | Priority |
|---|---|---|
| FR-AUD-01 | The framework shall carry forward edap_batch from the source record. | Must Have |
| FR-AUD-02 | The framework shall set edap_eff_from and edap_eff_to as part of SCD Type 2 processing, reconciling with the Lakeflow AUTO CDC `__START_AT` and `__END_AT` columns. | Must Have |
| FR-AUD-03 | The framework shall set edap_is_current and edap_is_deleted as part of history and soft-delete processing. | Must Have |
| FR-AUD-04 | The framework shall populate edap_inserted_ts with the current timestamp when a new record version is created. | Must Have |
| FR-AUD-05 | The framework shall populate edap_modified_ts with the current timestamp on every insert or update. | Must Have |
| FR-AUD-06 | The framework shall populate edap_user with the identity of the executing service principal. | Must Have |
| FR-AUD-07 | The framework shall not fail if file-related audit columns are absent from the source table (connector-ingested entities). | Must Have |
| FR-AUD-08 | The specific audit columns populated shall be determined by the zone transition type, as defined in Section 4. | Must Have |

### 3.7 Tagging

| ID | Requirement | Priority |
|---|---|---|
| FR-TAG-01 | The framework shall apply standard Unity Catalog tags to all target tables, including: medallion_layer, medallion_sublayer, retention_days, and data_type. | Must Have |
| FR-TAG-02 | The framework shall support applying tags at catalog, schema, and table level. Shared values should be set at the highest applicable level. | Should Have |
| FR-TAG-03 | Tag values shall be sourced from the entity configuration, with defaults derived from the zone transition type. | Must Have |

### 3.8 Data Contract Enforcement

| ID | Requirement | Priority |
|---|---|---|
| FR-CTR-01 | The framework shall validate pipeline output against the declared data contract schema for each entity, where a contract is defined. Validation shall confirm that all contracted columns are present, data types match, and nullable constraints are honoured. Schema violations shall be logged and optionally configured to fail the pipeline. | Should Have |
| FR-CTR-02 | The framework shall emit contract compliance metrics for each entity where a data contract is defined, including: schema match result (pass/fail), freshness (time since last successful update vs contracted SLA), and quality score (DQ pass rate vs contracted threshold). These metrics shall be queryable from the pipeline event log or a dedicated contract compliance table. | Should Have |

### 3.9 Sink and Output Routing

| ID | Requirement | Priority |
|---|---|---|
| FR-SNK-01 | The framework shall support the Lakeflow `create_sink` API to publish processed data to external targets (e.g. external Delta tables, Kafka topics) in addition to the primary Unity Catalog target table. | Could Have |
| FR-SNK-02 | Sink configuration shall be per-entity, with sink format and connection options specified in the entity configuration. | Could Have |

---

## 4. Zone-Specific Processing Rules

The generic engine (Section 3) provides the configurable machinery. This section defines the specific behaviours, audit columns, and constraints that apply to each zone transition. These rules are encoded as defaults within the framework for each zone transition type.

### 4.1 Raw â Base (Bronze â Silver)

**Delivery priority: Phase 1**

This is the most standardised and highest-volume transition. It transforms loosely typed, append-only Raw Zone records into cleansed, validated, and historised Base Zone tables.

**Zone-specific characteristics:**

- Source columns may be STRING/VARIANT (file-based) or natively typed (connector-based). Type casting and schema introspection apply.
- SCD Type 2 is the default history management pattern.
- All records propagate regardless of DQ status (annotate, never filter).
- Structural flattening of nested payloads is common.
- Deduplication is based on edap_hash comparison.
- Business semantics are retained â no business logic is applied beyond cleansing and standardisation.
- For snapshot sources without CDC feeds, `create_auto_cdc_from_snapshot_flow` is the preferred mechanism.

**Source audit columns (carried forward where present):**

| Column | Type | Description |
|---|---|---|
| edap_batch | BIGINT | Epoch (milliseconds) identifying the ingestion batch. Carried forward from Raw Zone. |

Note: File-related audit columns (edap.file_name, edap.file_path, edap.file_size, edap.file_modification_time) may not be present for connector-ingested entities. The framework must not assume their presence.

**Target audit columns (Base Zone):**

| Column | Type | Description |
|---|---|---|
| edap_batch | BIGINT | Carried forward from Raw Zone. |
| edap_eff_from | TIMESTAMP | When this version of the record became effective. |
| edap_eff_to | TIMESTAMP | When this version was superseded. 9999-12-31 indicates the current version. |
| edap_is_current | BOOLEAN | TRUE for the current version of the record. |
| edap_is_deleted | BOOLEAN | TRUE if the record has been soft-deleted or absent from a full extract. |
| edap_inserted_ts | TIMESTAMP | When the record was inserted. |
| edap_modified_ts | TIMESTAMP | Last modified timestamp. |
| edap_user | STRING | Service principal that executed the pipeline. |
| dq_status | STRING | Overall DQ status: PASS, WARN, FAIL. |
| dq_errors | ARRAY\<STRING\> | Failed rules. Empty array if no errors. |
| dq_warnings | ARRAY\<STRING\> | Warning rules. Empty array if no warnings. |
| dq_checked_ts | TIMESTAMP | When DQ rules were evaluated. |

**Target tags (defaults):** medallion_layer = silver, medallion_sublayer = base, retention_days = (from config), data_type = (from config).

### 4.2 Base â Enriched (Silver â Silver)

**Delivery priority: Phase 2**

Integrates Base Zone data from multiple source systems and applies business logic to create reusable, cross-domain datasets.

**Zone-specific characteristics:**

- Source data is already typed and validated â type casting is generally not required.
- Joins across multiple Base Zone tables are the primary transformation pattern.
- SCD Type 2 effectivity is calculated from Base Zone source records.
- DQ rules focus on cross-source consistency, referential integrity, and business rule compliance.
- Business logic is applied (calculations, derivations, scoring, geospatial joins).
- Reading from SCD Type 2 Base Zone tables as streaming sources requires `skipChangeCommits` handling (see Section 2.3).

**Target audit columns (Enriched Zone):** Same structure as Base Zone, with edap_batch, edap_eff_from, edap_eff_to, and edap_is_current calculated from Base Zone source records.

**Target tags (defaults):** medallion_layer = silver, medallion_sublayer = enriched, retention_days = (from config), data_type = (from config).

### 4.3 Enriched â BI (Silver â Gold)

**Delivery priority: Phase 3**

Transforms Enriched Zone data into dimensional models (facts, dimensions, metric views) for BI and reporting.

**Zone-specific characteristics:**

- Dimensional modelling patterns: star schemas, snowflake schemas, SCD dimensions.
- Surrogate keys generated for dimension tables.
- Pre-aggregated measures, calculated KPIs, and business-friendly terminology.
- edap_ system columns are generally **not** carried into the BI Zone. SCD history is expressed through dimensional attributes (eff_from, eff_to, is_current) without the edap_ prefix.
- DQ rules focus on referential integrity between facts and dimensions, KPI boundary validation, and completeness.
- `CREATE MATERIALIZED VIEW` may be preferred over `CREATE STREAMING TABLE` for aggregated Gold outputs.

**Target tags (defaults):** medallion_layer = gold, medallion_sublayer = bi, retention_days = (from config).

### 4.4 Enriched â Exploratory (Silver â Gold)

**Delivery priority: Phase 3**

Produces wide, denormalised datasets optimised for data science, exploration, and feature engineering.

**Zone-specific characteristics:**

- Wide tables combining multiple subject areas â minimal joins required downstream.
- Schema evolution is more permissive than other zones.
- DQ is lighter â minimum quality thresholds rather than strict validation.
- Liquid clustering and Z-ordering are critical for performance.
- edap_ system columns may be carried forward or omitted depending on use case.

**Target tags (defaults):** medallion_layer = gold, medallion_sublayer = exploratory, retention_days = (from config).

---

## 5. Non-Functional Requirements

### 5.1 Idempotency and Reprocessing

| ID | Requirement | Priority |
|---|---|---|
| NFR-IDP-01 | The framework shall produce identical output when the same input batch is processed multiple times (idempotent). | Must Have |
| NFR-IDP-02 | The framework shall support full reprocessing of an entity from its source zone without manual intervention. | Must Have |
| NFR-IDP-03 | The framework shall support initial hydration of a target table from a full source dataset before transitioning to incremental CDC processing. | Should Have |

### 5.2 Scalability and Performance

| ID | Requirement | Priority |
|---|---|---|
| NFR-SCL-01 | The framework shall support onboarding new entities through configuration only, without code deployment. | Must Have |
| NFR-SCL-02 | The framework shall process entities with record volumes from thousands to hundreds of millions of rows. | Must Have |
| NFR-SCL-03 | The framework shall leverage Databricks auto-scaling and Photon acceleration where available. | Should Have |
| NFR-SCL-04 | The framework shall support liquid clustering on target tables, configurable per entity. | Should Have |
| NFR-SCL-05 | The framework shall support partition column specification per entity where liquid clustering is not appropriate. | Could Have |

### 5.3 Observability and Monitoring

| ID | Requirement | Priority |
|---|---|---|
| NFR-OBS-01 | The framework shall emit structured logs for each pipeline run: entity name, zone transition, record counts (read, inserted, updated, deleted, errored), start/end timestamps, and DQ summary statistics. | Must Have |
| NFR-OBS-02 | The framework shall integrate with Databricks Lakeflow pipeline monitoring, including DQ expectation dashboards. | Should Have |
| NFR-OBS-03 | The framework shall support alerting on pipeline failures and DQ threshold breaches, configurable per entity. | Should Have |
| NFR-OBS-04 | Unity Catalog lineage shall be automatically captured from source to target for all framework-managed entities. | Must Have |
| NFR-OBS-05 | The framework shall support querying DQ expectation metrics from the Lakeflow pipeline event log for programmatic monitoring. | Should Have |

### 5.4 Security and Governance

| ID | Requirement | Priority |
|---|---|---|
| NFR-SEC-01 | The framework shall execute under a dedicated service principal with least-privilege access to source (read) and target (read/write) Unity Catalog objects. | Must Have |
| NFR-SEC-02 | Configuration files and the Dataflowspec Delta table shall be access-controlled to prevent unauthorised modification. | Must Have |
| NFR-SEC-03 | The framework shall not expose or log sensitive data values (PI, credentials) in pipeline logs or error messages. | Must Have |
| NFR-SEC-04 | The framework shall support integration with the Protected Zone pattern where source entities contain PI. | Should Have |

### 5.5 Testability

| ID | Requirement | Priority |
|---|---|---|
| NFR-TST-01 | The framework shall be deployable to the dev environment using shallow-cloned or synthetic data, consistent with ADR-EDP-001. | Must Have |
| NFR-TST-02 | The framework shall support dry-run mode where pipeline logic is validated without writing to target tables. | Should Have |
| NFR-TST-03 | Individual entity configurations shall be independently testable without running the full pipeline suite. | Must Have |
| NFR-TST-04 | The framework shall support integration test patterns that validate record counts across source and target tables after pipeline execution. | Should Have |

### 5.6 Maintainability and Extensibility

| ID | Requirement | Priority |
|---|---|---|
| NFR-MNT-01 | The framework shall follow a modular architecture, separating configuration parsing, onboarding/validation, transformation logic, DQ evaluation, SCD processing, and audit column management into distinct, testable components. | Must Have |
| NFR-MNT-02 | The framework shall support custom transformation hooks via Python functions, injectable at both Bronze and Silver layers with configurable execution order. | Should Have |
| NFR-MNT-03 | The framework shall be version-controlled and deployed via CI/CD pipelines aligned with the SAFe delivery cadence. | Must Have |
| NFR-MNT-04 | Internal staging tables shall be automatically managed by the framework without separate operational overhead. | Must Have |
| NFR-MNT-05 | The framework shall support deployment via Databricks Asset Bundles (DABs) for declarative, repeatable infrastructure-as-code deployment. | Should Have |
| NFR-MNT-06 | The framework shall provide an onboarding workflow (CLI, notebook, or API) that translates configuration into the Dataflowspec Delta table. | Must Have |
| NFR-MNT-07 | Adding or modifying an entity and re-onboarding shall not require downtime for existing pipeline entities. The onboarding process shall support both overwrite and append modes. | Should Have |

### 5.7 Data Observability

| ID | Requirement | Priority |
|---|---|---|
| NFR-DOB-01 | The framework shall integrate with Databricks Lakehouse Monitoring for automated data profiling of target tables, enabling drift detection, statistical summaries, and anomaly identification without custom instrumentation. | Should Have |
| NFR-DOB-02 | The framework shall support configurable freshness SLA monitoring per entity. When a target table's last update timestamp exceeds the configured freshness threshold, the framework shall emit an alert via the standard alerting mechanism. Freshness thresholds shall be configurable per entity in the Dataflowspec. | Should Have |
| NFR-DOB-03 | The framework shall support volume anomaly detection per entity. When the record count for a pipeline run deviates from historical norms by more than a configurable threshold (e.g. percentage deviation from the rolling average), the framework shall emit a warning alert. This guards against silent data loss, upstream source failures, and unexpected volume spikes. | Should Have |

### 5.8 Cost Management

| ID | Requirement | Priority |
|---|---|---|
| NFR-CST-01 | The framework shall support cost attribution per entity or data flow group, enabling the platform team to allocate compute and storage costs to the owning domain or data product. Cost attribution may be implemented through cluster tagging, pipeline tagging, or integration with Databricks system tables for billing analysis. | Should Have |
| NFR-CST-02 | The framework shall provide DBU consumption observability per pipeline, enabling the platform team to monitor and optimise compute spend. Pipeline-level DBU metrics shall be queryable from Databricks system tables or the pipeline event log. | Should Have |

### 5.9 Error Handling and Recovery

| ID | Requirement | Priority |
|---|---|---|
| NFR-ERR-01 | The framework shall handle transient failures (network timeouts, temporary resource unavailability) with configurable retry logic before failing. | Should Have |
| NFR-ERR-02 | When a pipeline run fails, the framework shall produce diagnostic information sufficient to identify the failing entity, the processing step, and the root cause. | Must Have |
| NFR-ERR-03 | The framework shall support the `pipelines.reset.allowed` table property to control whether full refresh is permitted for a given entity, protecting against accidental data loss on production tables. | Should Have |
| NFR-ERR-04 | Pipeline failures for one entity within a data flow group shall not prevent other independent entities from completing, where Lakeflow supports partial completion. | Should Have |

---

## 6. Configuration Schema

This section provides the indicative structure for the entity configuration. The final schema will be determined during design. The configuration follows a two-stage lifecycle: authored as JSON/YAML files in source control, then translated via the onboarding workflow into the Dataflowspec Delta table for runtime consumption.

### 6.1 Entity-Level Configuration

| Property | Type | Description |
|---|---|---|
| data_flow_id | STRING | Unique identifier for the pipeline data flow. |
| data_flow_group | STRING | Logical grouping for co-execution within a single Lakeflow Pipeline. |
| zone_transition | STRING | raw_to_base \| base_to_enriched \| enriched_to_bi \| enriched_to_exploratory. |
| source_format | STRING | Source format: delta \| cloudFiles \| eventhub \| kafka \| snapshot. |
| source_details | MAP\<STRING,STRING\> | Source connection details (schema path, source path per env, catalog, database, table). |
| source_metadata | OBJECT | Optional. Auto Loader `_metadata` extraction config. |
| target_catalog_{env} | STRING | Target catalog per environment. |
| target_schema | STRING | Target schema. |
| target_table | STRING | Target table. |
| target_table_comment | STRING | Optional. Table comment for Unity Catalog. |
| business_keys | ARRAY\<STRING\> | Business key columns for deduplication and SCD. |
| extract_pattern | STRING | full_snapshot \| incremental_cdc \| incremental_append. |
| ingestion_method | STRING | file_based \| managed_connector \| streaming (for raw_to_base). |
| scd_type | STRING | 1 \| 2. Default: 2. |
| sequence_by | STRING | CDC ordering column(s). |
| apply_as_deletes | STRING | Optional. SQL expression for delete identification. |
| except_column_list | ARRAY\<STRING\> | Optional. Columns excluded from SCD tracking. |
| track_history_column_list | ARRAY\<STRING\> | Optional. Columns included in SCD tracking (alternative to except_column_list). |
| select_exp | ARRAY\<STRING\> or ARRAY\<ARRAY\<STRING\>\> | Optional. SQL select expressions. Supports single or multi-stage evaluation. |
| where_clause | STRING | Optional. SQL filter condition. |
| reader_options | MAP\<STRING,STRING\> | Optional. Spark reader options. |
| hash_exclude_columns | ARRAY\<STRING\> | Columns excluded from edap_hash. |
| liquid_cluster_keys | ARRAY\<STRING\> | Optional. Liquid clustering columns. |
| partition_columns | ARRAY\<STRING\> | Optional. Partition columns (where liquid clustering not used). |
| table_properties | MAP\<STRING,STRING\> | Optional. Lakeflow table properties. |
| tags | MAP\<STRING,STRING\> | Tags to apply (merged with zone defaults). |
| custom_transform_fn | ARRAY\<STRING\> | Optional. Ordered list of custom Python transformation function names. |
| unmapped_column_policy | STRING | Optional. ignore \| capture \| warn. Default: warn. |
| staging_steps | ARRAY\<OBJECT\> | Optional. Internal staging table definitions. |
| dq_expectations_json | STRING | Optional. Path to external DQ rules JSON (Lakeflow expectations format). |
| quarantine_table | STRING | Optional. Quarantine table name. |
| quarantine_table_properties | MAP\<STRING,STRING\> | Optional. Quarantine table properties. |
| sink | OBJECT | Optional. Lakeflow sink configuration. |
| append_flows | ARRAY\<OBJECT\> | Optional. Additional source streams for append flow. |

### 6.2 Column-Level Configuration

| Property | Type | Description |
|---|---|---|
| source_column | STRING | Column name in the source table. |
| target_column | STRING | Renamed column name in the target table. |
| target_type | STRING | Target data type. |
| source_type_override | STRING | Optional. Forces source type interpretation. |
| nullable | BOOLEAN | Whether the column allows nulls. |
| default_value | STRING | Optional. Default for null replacement. |
| flatten_path | STRING | Optional. JSON path for VARIANT flattening. |
| column_comment | STRING | Optional. Column comment for Unity Catalog. |
| dq_rules | ARRAY\<OBJECT\> | DQ rule definitions (see 6.3). |

### 6.3 DQ Rule Configuration

| Property | Type | Description |
|---|---|---|
| rule_name | STRING | Unique rule name (used in dq_errors/dq_warnings arrays). |
| rule_type | STRING | not_null \| type_check \| range \| allowed_values \| regex \| unique \| custom_sql \| sql_expression. |
| severity | STRING | error \| warning. |
| parameters | MAP\<STRING,STRING\> | Rule-specific parameters. |

---

## 7. Processing Flow

The following describes the logical processing sequence. The same sequence applies to all zone transitions; specific steps may be no-ops depending on the transition type and entity configuration.

### 7.1 High-Level Pipeline Steps

| Step | Operation | Description |
|---|---|---|
| 1 | **Read Source** | Read from the source Delta table using streaming or batch read. Apply reader options. Extract Auto Loader `_metadata` columns if configured. |
| 2 | **Schema Introspection** | Inspect source schema. Compare against configured target types to build a casting plan. For transitions where types already match, casting is skipped. |
| 3 | **Flatten & Rename** | Flatten nested structures and rename columns. For complex entities, writes to internal staging tables. May be a no-op for non-Raw sources. |
| 4 | **Type Cast** | Cast columns where types differ. No-op for natively typed sources or Silver â Gold transitions. Capture cast failures in DQ annotations. |
| 5 | **Custom Transforms** | Apply configured SQL select expressions and/or custom Python transformation functions in configured execution order. |
| 6 | **Null Handling** | Apply null handling rules per column. |
| 7 | **DQ Evaluation** | Evaluate DQ rules. Populate per-record annotation columns. Apply Lakeflow expectations in parallel. Route to quarantine if configured. All records propagate to main target. |
| 8 | **Deduplication** | Compare edap_hash against current target. For snapshots, detect deletes. For CDC, process via AUTO CDC API. |
| 9 | **SCD Processing** | Apply SCD Type 1 or 2 as configured. Map Lakeflow `__START_AT`/`__END_AT` to EDAP effectivity columns. For BI Zone targets, use dimensional conventions (no edap_ prefix). |
| 10 | **Audit Columns** | Populate zone-appropriate audit columns per Section 4. |
| 11 | **Write Target** | Write to target Delta table. Apply liquid clustering/partitioning. Set table and column comments. |
| 12 | **Apply Tags** | Apply Unity Catalog tags, merged with zone defaults. |
| 13 | **Sink Output** | If configured, publish to external targets via Lakeflow sink API. |

---

## 8. Acceptance Criteria

### Phase 1 â Raw â Base

- A new source entity can be onboarded by adding a configuration entry and running the onboarding workflow â no new pipeline code required.
- The framework correctly handles both file-ingested (STRING/VARIANT) and connector-ingested (natively typed) source entities.
- All Base Zone audit columns are correctly populated for every record.
- All Base Zone DQ columns are correctly populated. No records filtered or quarantined by default.
- SCD Type 2 history is correctly maintained: changed records produce a new version, deleted records are soft-deleted, reappearing records create new current versions. SCD Type 1 is available as a configurable alternative.
- The EDAP audit column model (edap_eff_from/edap_eff_to/edap_is_current) is correctly reconciled with Lakeflow's `__START_AT`/`__END_AT` columns.
- Running the same pipeline twice with the same input produces identical output (idempotent).
- The framework operates correctly in dev, test, and prod environments using parameterised catalog names.
- Unity Catalog lineage is visible from source to target.
- Standard tags are applied to all framework-managed target tables.
- Pipeline logs provide sufficient detail for operational monitoring and troubleshooting.
- The framework does not fail when file-related audit columns are absent.
- Entities requiring complex structural transformation can declare internal staging steps visible in Lakeflow lineage.
- Configuration is validated at onboarding with clear error messages.
- The onboarding workflow successfully translates configuration into a Dataflowspec Delta table.
- Multiple entities can be co-executed within a single Lakeflow Pipeline via data flow grouping.
- Snapshot sources are processable via `create_auto_cdc_from_snapshot_flow` without manual diff logic.
- The framework is deployable via Databricks Asset Bundles.

### Phase 2 â Base â Enriched

- The same framework engine processes Base â Enriched transitions using configuration.
- Custom transformation hooks and SQL select expressions support multi-table joins and business logic.
- Enriched Zone audit columns are correctly populated, with effectivity derived from Base Zone source records.
- DQ rules covering cross-source consistency and referential integrity are evaluated and annotated.
- The `skipChangeCommits` constraint is handled correctly when reading from SCD Type 2 Base Zone targets.

### Phase 3 â Enriched â Gold

- The framework produces BI Zone dimensional models with surrogate keys and business-friendly naming.
- The framework produces Exploratory Zone wide tables with appropriate clustering.
- edap_ system columns are omitted from BI Zone targets as per the architecture specification.
- `CREATE MATERIALIZED VIEW` is supported for aggregated Gold outputs.

---

## 9. Open Questions and Decisions

| ID | Question | Decision Owner | Status |
|---|---|---|---|
| OQ-01 | Should the EDAP framework adopt DLT-META as its foundation, use it as a reference architecture, or build independently? See Appendix A for the full gap analysis and adoption strategy options. | Architecture / Platform Team | Open |
| OQ-02 | How should the framework reconcile Lakeflow AUTO CDC managed columns (`__START_AT`, `__END_AT`) with the EDAP convention (edap_eff_from, edap_eff_to, edap_is_current)? Options: (a) map in post-processing, (b) custom SCD logic bypassing AUTO CDC, (c) carry both. **Recommendation:** Option (a) â map Lakeflow-managed columns to EDAP columns in a post-processing view or within the pipeline definition. Retain `__START_AT` and `__END_AT` as the physical columns managed by Lakeflow AUTO CDC, and expose `edap_eff_from`, `edap_eff_to`, and `edap_is_current` as logical columns via a view layer or computed columns in the pipeline output step. This preserves Lakeflow's native SCD management (avoiding the maintenance burden of option (b)) while presenting a consistent EDAP interface to consumers. The view layer also provides a natural point for deriving `edap_is_current` (which Lakeflow does not natively manage) from `__END_AT IS NULL` or equivalent logic. | Architecture / Data Engineering | Open â Recommendation provided |
| OQ-03 | How should the framework handle schema evolution in the Raw Zone? Automatic propagation or explicit configuration required? | Architecture / Data Engineering | Open |
| OQ-04 | What is the DQ threshold for pipeline alerting? Configurable per entity? | Data Governance / Product Owner | Open |
| OQ-05 | Should the framework support partial reprocessing (specific date ranges or batches) or only full entity reprocessing? | Data Engineering | Open |
| OQ-06 | How should the framework handle late-arriving data for incremental sources? | Architecture / Data Engineering | Open |
| OQ-07 | What is the retention policy for Base Zone SCD history? | Data Governance / Compliance | Open |
| OQ-08 | What is the default data flow grouping strategy? What are the failure isolation trade-offs of co-execution? | Platform Team / Data Engineering | Open |
| OQ-09 | How should the Protected Zone integration work for PI-containing entities? | Security / Data Governance | Open |
| OQ-10 | For Lakeflow Connect-ingested entities, should the framework validate source schema against configuration and fail fast on mismatches? | Architecture / Data Engineering | Open |
| OQ-11 | What is the retention/cleanup policy for internal staging tables? | Architecture / Platform Team | Open |
| OQ-12 | For Base â Enriched joins across multiple Base Zone tables, how should join definitions be expressed in configuration? | Architecture / Data Engineering | Open |
| OQ-13 | For BI Zone dimensional models, should surrogate key generation be managed by the framework or delegated to transformation logic? | Architecture / Data Engineering | Open |
| OQ-14 | Should entities using the quarantine pattern also retain annotated records in the main target (quarantine as copy) or divert them (quarantine as removal)? | Data Governance / Architecture | Open |

---

## Appendix A â DLT-META Gap Analysis

This appendix provides a structured comparison between DLT-META capabilities and EDAP requirements to inform the build-vs-adopt decision (OQ-01).

### A.1 Feature Alignment Matrix

| Capability | DLT-META | EDAP Requirement | Gap |
|---|---|---|---|
| Metadata-driven pipeline generation | Yes â Dataflowspec pattern | FR-CFG-01 to FR-CFG-03 | Aligned. Adopt the Dataflowspec pattern. |
| Data flow grouping | Yes â `data_flow_group` | FR-CFG-08 | Aligned. |
| Environment parameterisation | Yes â `{env}` suffix | FR-CFG-04 | Aligned. |
| Configuration validation | Limited â runtime errors on bad JSON | FR-CFG-09 | **Gap.** EDAP needs rigorous pre-validation. |
| CDC / SCD Type 1 & 2 | Yes â AUTO CDC, snapshot CDC | FR-SCD-01 to FR-SCD-09, FR-DDP-03, FR-DDP-06 | Partial. DLT-META uses `__START_AT`/`__END_AT`; EDAP needs edap_ column mapping. |
| DQ expectations | Yes â expect, expect_or_fail, expect_or_drop, expect_or_quarantine | FR-DQ-01 to FR-DQ-10 | **Gap.** Pipeline-level only. EDAP needs per-record annotation. |
| Quarantine tables | Yes â Bronze and Silver (v0.0.10+) | FR-DQ-08 | Aligned (as optional pattern). |
| Custom transformations | Yes â Python functions, ordered execution | FR-TYP-12, NFR-MNT-02 | Aligned. |
| SQL select expressions | Yes â `select_exp` array | FR-TYP-12 | Partial. Single-stage only; EDAP needs multi-stage. |
| Append flows | Yes â `append_flow` API | FR-DDP-07 | Aligned. |
| Sink API | Yes â Delta and Kafka | FR-SNK-01 | Aligned. |
| Liquid clustering | Yes â Bronze, Quarantine, Silver | NFR-SCL-04 | Aligned. |
| Auto Loader metadata | Yes â `source_metadata` config | FR-AUD-07 | Aligned. |
| Table properties | Yes â `table_properties` map | FR-CFG-13 | Aligned. |
| Table comments | Yes â `table_comment` properties | FR-CFG-14 | Aligned. |
| UC tagging | No | FR-TAG-01 to FR-TAG-03 | **Gap.** |
| EDAP audit columns | No â no `edap_` convention | FR-AUD-01 to FR-AUD-08 | **Gap.** |
| Per-column configuration | No â entity-level SQL only | FR-TYP-01 to FR-TYP-06 | **Gap.** |
| Hash-based deduplication | No â relies on CDC/SCD | FR-DDP-01 | **Gap.** |
| Multi-zone (Gold) | No â Bronze and Silver only | Section 4.3, 4.4 | **Gap.** |
| DABs deployment | Yes | NFR-MNT-05 | Aligned. |
| CLI tooling | Yes â `databricks labs dlt-meta` | NFR-MNT-06 | Aligned. |
| Lakehouse App UI | Yes | NFR-MNT-06 | Aligned (bonus). |
| Formal SLA / support | No â Databricks Labs, AS-IS | All NFRs | **Risk.** |

### A.2 Adoption Strategy Options

| Strategy | Description | Pros | Cons |
|---|---|---|---|
| **Adopt & Extend** | Use DLT-META as the foundation; build EDAP-specific extensions on top. | Fastest time-to-value; proven patterns; community updates. | Dependency on unsupported project; extension points may not be sufficient; version upgrade risk. |
| **Reference & Build** | Use DLT-META as a reference architecture; build independently following the same patterns. | Full control; no external dependency; can optimise for EDAP. | Longer build time; must re-implement proven patterns. |
| **Fork & Customise** | Fork the DLT-META repository and customise for EDAP. | Full control with a head start; can cherry-pick upstream updates. | Merge conflict risk; maintenance burden; inherits architectural choices that may not fit EDAP. |

---

## 11. Document History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1 | March 2026 | IT Architecture & Strategy | Initial draft for ARB review (scoped to Raw â Base). |
| 0.2 | March 2026 | IT Architecture & Strategy | Incorporated DLT-META review findings: data flow grouping, config validation, SCD Type 1, CDC configuration, append flows, quarantine option, rescued data, select expressions, DABs support. |
| 0.3 | March 2026 | IT Architecture & Strategy | Broadened scope to EDAP Configurable Pipeline Framework covering all zone transitions. Restructured into generic engine requirements and zone-specific processing rules. Added phased delivery and acceptance criteria. |
| 0.4 | March 2026 | IT Architecture & Strategy | Comprehensive quality uplift following deep DLT-META analysis. Key additions: AUTO CDC API migration guidance and pipeline chaining constraints (skipChangeCommits); Dataflowspec two-stage pattern promoted to Must Have; per-record DQ annotation clarified alongside pipeline expectations; source_metadata extraction; snapshot CDC via create_auto_cdc_from_snapshot_flow; initial hydration pattern; __START_AT/__END_AT reconciliation requirement (OQ-02); multi-stage select_exp support; error handling and recovery section (5.7); DQ event log metrics; table and column comments; integration test patterns; Sink API section (3.8); referenced documents section; full DLT-META gap analysis appendix with feature alignment matrix and adoption strategy options. |

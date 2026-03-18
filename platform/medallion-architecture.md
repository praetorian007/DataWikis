# Medallion Architecture â Enterprise Data Platform Design Guide

## Overview

The medallion architecture is a data design pattern used to logically organise data in a lakehouse. Its goal is to incrementally and progressively improve the structure and quality of data as it flows through each layer of the architecture. Also referred to as a "multi-hop" architecture, it structures data pipelines into a series of increasingly refined layers â typically **Bronze**, **Silver**, and **Gold** â each denoting a higher level of data quality, structure, and business readiness.

This architecture guarantees atomicity, consistency, isolation, and durability (ACID) as data passes through multiple layers of validation and transformation before being stored in a layout optimised for efficient analytics. By structuring data this way, organisations can streamline data processing, enhance data governance, simplify troubleshooting, and build a single source of truth for enterprise data products.

### Key Principles

- **Progressive refinement**: Each layer builds on the previous, improving structure and quality incrementally.
- **ACID compliance**: Ensured through open table formats (Delta Lake, Apache Iceberg) at every persistent layer. Delta Lake **UniForm** (Universal Format) enables Delta tables to be read natively as Iceberg or Hudi tables without data duplication, providing broad interoperability with external engines.
- **Reproducibility**: All downstream layers should be regenerable from the Bronze layer (the system of record).
- **Separation of concerns**: Each layer has a distinct purpose and clear boundaries â ingestion, validation, enrichment, and consumption are not conflated.
- **Governance by design**: Access controls, lineage, and data quality checks are embedded at every tier, not bolted on afterwards.

### Flexibility and Adaptability

The medallion architecture is not a rigid prescription. It represents a spectrum of possibilities that can be adapted to unique organisational circumstances. Each of the three core layers can be further segmented into one or more zones to support data management, processing, access controls, or other specific requirements. Some organisations introduce extra layers (e.g. a pre-Bronze landing area or a Platinum layer for specialised data products), and the architecture is compatible with both Data Mesh (domain-specific medallion stacks) and traditional centralised governance models.

> **Note:** See the **Zone Comparison** table at the end of this document for a quick summary of all zones.

---

## Unity Catalog Alignment

In a Databricks environment, the medallion architecture maps naturally to Unity Catalog's three-level namespace: **Catalog â Schema â Table/View/Volume**. The recommended approach is to represent medallion layers as schemas (not table name prefixes), enabling clean access control, lineage tracking, and discoverability.

### Recommended Namespace Pattern

| Approach | Pattern | Example |
|----------|---------|---------|
| Layer-as-Schema | `<catalog>.<layer>.<table>` | `edap_prod.bronze.sap_work_orders` |
| Layer + Source-as-Schema | `<catalog>.<layer>_<source>.<table>` | `edap_prod.bronze_sap.work_orders` |

- **Catalogs** should reflect environment (e.g. `edap_dev`, `edap_test`, `edap_prod`) or domain boundaries (e.g. `assets`, `finance`).
- **Schemas** should encode the medallion layer, optionally combined with source system or subject area.
- **Table names** should be descriptive and business-meaningful â avoid repeating layer or source in the table name when these are already encoded in the schema.

### Governance Features

- **Fine-grained access controls**: Unity Catalog supports table, row, and column-level security.
- **Automatic data lineage**: Unity Catalog captures lineage across layers, providing a clear view of data flow from Bronze through to Gold.
- **Column-level security**: Protect sensitive data (e.g. personal information) with column-level masking and access policies.
- **Audit logging**: All data access and changes are logged via system tables for compliance and investigation.

---

## Bronze Layer

**Purpose:** Capture and persist raw data from source systems with minimal transformation for audit, traceability, and reprocessing.

The Bronze layer is the system of record for all ingested data. It preserves a complete historical archive of data as it arrived, enabling full traceability and the ability to replay or reprocess pipelines at any point. No business logic or heavy transformation is applied here â the focus is on reliable capture.

### Landing Zone

**Purpose:** Temporarily stage data before ingestion into the Raw Zone.

**Characteristics:**

- Transient â cleared after successful ingestion; not a persistence layer.
- Contains structured, unstructured, or semi-structured data in its native format.
- Supports any file format (CSV, JSON, XML, Avro, Parquet, binary, images, PDFs, etc.).
- Acts as a buffer between file arrival and processing.

**Guidance:**

- Automate cleanup post-ingestion to reduce storage costs and improve security.
- Validate file-level expectations before promoting to Raw: naming conventions, expected format, encoding, file size within expected range.
- Consider a pre-Bronze quarantine mechanism for virus scanning or validating potentially malicious or corrupted files before they persist in the lakehouse.
- Extracts are typically:
  - **Full extract** â entire dataset loaded each time.
  - **Delta extract** â changes (inserts, updates, and deletions) captured between specific points in time. Always ensure soft and hard deletions are captured.
- Prefer formats with embedded schema (e.g. Parquet over JSON over CSV) to reduce downstream schema inference issues.

**Folder Structure:**

```
/bronze/landing/
    âââ <source_system>/
          âââ YYYY/
                âââ MM/
                      âââ DD/
                            âââ <edp_batch>/
                                  âââ file1.parquet
                                  âââ ...
```

### Raw Zone

**Purpose:** Persist the original, unaltered data for long-term retention, traceability, and replay capability.

**Characteristics:**

- Contains structured, unstructured, or semi-structured data in its native format.
- Immutable, append-only storage â data is never modified or overwritten.
- Preserves full fidelity of source data as received.

**Guidance:**

- Retain data for regulatory, audit, or reprocessing purposes.
- All downstream layers must be regenerable from this zone.
- The ingestion framework may output directly into Raw using an efficient format (e.g. Parquet or Delta).
- For Databricks implementations, storing Bronze as Delta tables (rather than raw files in cloud storage) is recommended. Delta provides ACID transactions, time travel for debugging, schema evolution support, liquid clustering for performance (replacing the now-legacy Z-ordering approach), and full Unity Catalog governance including lineage tracking.
- Use **Auto Loader** for efficient, incremental ingestion from cloud object storage, with built-in support for schema inference and evolution.
- Consider ingesting semi-structured data (JSON, XML) as a single **VARIANT** column for maximum robustness against upstream schema changes â this ensures no data is dropped due to unexpected field additions or type changes.

**Folder Structure:**

```
/bronze/raw/
    âââ <source_system>/
          âââ YYYY/
                âââ MM/
                      âââ DD/
                            âââ <edp_batch>/
                                  âââ file1.json
                                  âââ ...
```

### Processed Zone

**Purpose:** Apply schema validation and standardise formats to prepare data for downstream cleansing and transformation.

**Characteristics:**

- Data format standardised to an open table format (Delta Lake or Apache Iceberg).
- Schema validation applied â data types and basic structure are enforced.
- Retains original business semantics without altering business logic.
- Immutable, append-only storage.

**Guidance:**

- Validate data types and basic structure without altering business logic.
- Optimise partitioning for downstream consumption.
- Create processes for handling schema validation failures â route invalid records to a **quarantine table** rather than silently dropping them.
- Include audit columns for lineage and change detection.
- Use the **rescued data column** (`_rescued_data`) pattern from Auto Loader to capture any data that fails parsing or doesn't match the expected schema, preventing data loss.
- Leverage **Lakeflow Spark Declarative Pipelines** (formerly Delta Live Tables) with **expectations** to define data quality constraints. Expectations can retain, drop, or fail on invalid records, providing built-in quarantine capabilities.

**Audit Columns:**

| Column | Description |
|--------|-------------|
| `edp_batch` | Epoch in milliseconds describing when the batch arrived (e.g. `1720071745635`) |
| `edp_hash` | SHA-512 hash of all non-`edp_` prefixed fields (separated by `\|` joiner, including secret salt) â supports change detection and deduplication |
| `_metadata.file_name` | Source file provenance (automatically captured by Auto Loader) |

A configuration file specifies any fields to be excluded from the calculation of the `edp_hash`.

**Folder Structure:**

```
/bronze/processed/
    âââ <source_system>/
          âââ <table_name>/
                âââ year=YYYY/
                      âââ month=MM/
                            âââ day=DD/
                                 âââ [data files]
```

---

## Silver Layer

**Purpose:** Transform and validate raw data into consistent, structured, and integrated datasets. This layer focuses on improving data quality and making data usable for analysis, while providing an "enterprise view" of key business entities.

The Silver layer is where the ELT methodology applies "just enough" transformation â data is cleansed, conformed, and integrated without over-engineering for specific consumption patterns. It serves as the foundation for Data Engineers, Data Analysts, and Data Scientists to build further analytical products.

### Protected Zone

**Purpose:** Securely store and isolate protected information (e.g. Personal Information) in a controlled manner.

**Characteristics:**

- Access is controlled by role-based access policies (RBAC) via Unity Catalog â users require the appropriate role to access specific subject areas (e.g. customer PI).
- Temporary privileged access is provided via PAM (Privileged Access Management) when necessary and revoked automatically after task completion.
- Column-level security and dynamic data masking may be applied to restrict sensitive field visibility.
- May contain derived datasets that associate personal information with demographic or analytical attributes. An anonymised ID and non-sensitive demographic fields are then used in other layers rather than the personal information itself.

**Guidance:**

- Apply column-level masking and row-level security via Unity Catalog to enforce least-privilege access.
- Ensure compliance with relevant privacy regulations (e.g. GDPR, Privacy Act, SOCI Act).
- Use views to abstract underlying sensitive data and provide secure access layers for different user groups.
- Implement "right to be forgotten" capabilities using Delta Lake features (Change Data Feed, deletion vectors, and VACUUM with defined retention).
- **Deletion vectors** are a general Delta Lake performance feature, not limited to privacy compliance. They significantly improve the performance of MERGE, UPDATE, and DELETE operations by marking rows as deleted without immediately rewriting data files. Deletion vectors are enabled by default on new Delta tables.

### Base Zone

**Purpose:** Apply data quality checks and transformations to create clean, trustworthy datasets.

**Characteristics:**

- Cleansed, validated, and deduplicated data.
- Flattened nested structures (e.g. exploded JSON arrays).
- Nulls handled according to documented business rules.
- Standardised field names ensuring consistent, clear, and descriptive names across all datasets.
- Version history is tracked (SCD Type 2 or change data feed as appropriate).
- Structurally aligned with the data source â maintains a recognisable mapping to source entities.
- Acts as a reliable source for further enrichment and analysis.

**Guidance:**

- Define clear, documented data validation and cleansing rules.
- Create processes for handling validation and cleansing failures â quarantine failed records with metadata describing the issue.
- Design for **idempotent reprocessing** â rerunning a pipeline should produce the same result.
- Use **streaming tables** for append-only source data and **materialized views** for data requiring updates/merges.
- Apply **AutoCDC** (available in Lakeflow Declarative Pipelines) for handling change data capture with SCD Type 1 and Type 2 patterns without complex manual merge logic.
- Include audit columns (e.g. record effectivity dates, record deletion flags, source batch reference).
- In the Silver layer, explicitly define expected schemas rather than relying on schema inference â this ensures consistent data types, early detection of schema drift, and predictable downstream behaviour.

### Enriched Zone

**Purpose:** Integrate base data and apply business logic to create reusable datasets that support creation of the Gold Layer.

**Characteristics:**

- Combines data from multiple subject areas using well-defined, documented join logic.
- Supports both normalised (modular, reusable) and denormalised (performance-optimised) structures, depending on analytical requirements.
- Applies advanced business rules, calculations, and contextual enrichments (e.g. geospatial joins, time-series interpolation, demographic tagging, entity resolution).
- Aligns to subject areas or business use cases (e.g. Customer 360, Asset Health).
- Focuses on preparing consistent, high-quality inputs for Gold.
- Designed for reusability across multiple downstream analytical products.

**Guidance:**

- Define and document reusable join structures (e.g. reusable views or intermediate tables).
- Balance normalisation for modularity and denormalisation for performance.
- Validate all transformations through data quality checks and lineage tracking.
- Provide common enriched views that can be consumed directly or extended in Gold.
- Ensure naming conventions, field standardisation, and metadata are consistent with the rest of the Silver layer.
- Monitor join operations to prevent mismatches or lost records in merged datasets.
- This zone is where data begins to transition from source-aligned to business-aligned structures.

---

## Gold Layer

**Purpose:** Provide trusted, high-performance datasets tailored for business consumption, decision-making, analytics, and reporting.

The Gold layer is where semantic meaning, business logic, and consumability converge. Data here is fully refined, aggregated, and optimised for end-user queries, dashboards, AI agents, Databricks Apps, machine learning, and business-critical operations. Optimising Gold-layer tables for performance is a best practice because these datasets are frequently queried â large amounts of historical data are typically accessed in the Silver layer and not materialised in Gold.

Gold-layer datasets increasingly serve as the foundation for **data products** — certified, contracted, and discoverable outputs that carry explicit schema definitions, quality guarantees, ownership, and SLAs. Treating Gold outputs as data products ensures they are governed, versioned, and consumable by both human users and automated agents.

### Exploratory Zone

**Purpose:** Provide broad, denormalised, analysis-ready datasets optimised for discovery, data science, and ad hoc analysis.

**Characteristics:**

- Wide, denormalised datasets combining multiple subject areas into a single unified view.
- Includes enriched fields from reference and master data.
- Fewer joins required, enabling faster iteration and discovery.
- Suited for data science, exploration, and self-service analytics.

**Guidance:**

- Build tables around use-case themes (e.g. Customer Activity, Asset Performance).
- Include all relevant context in a single table to reduce dependency on joins.
- Ensure important metadata (e.g. source, timestamp, version, lineage) is included.
- Partition and optimise large tables for performance. Use **liquid clustering** on frequently filtered columns â this is the modern replacement for Z-ordering (now considered legacy for Delta tables) and Hive-style partitioning, providing automatic, adaptive data layout without manual maintenance.
- Maintain documentation on assumptions and enrichments applied.
- Use standardised naming conventions to maintain readability.
- Allow flexibility in schema evolution, but enforce minimum data quality thresholds.

**Typical Use Cases:**

- Exploratory data analysis (EDA)
- Feature engineering for ML models
- Rapid prototyping and hypothesis testing
- Trend analysis across domains (e.g. customer, operations, billing)

### Dimensional Zone

**Purpose:** Support traditional BI, reporting, and metrics through well-structured dimensional models such as star and snowflake schemas. Provide consistent, governed KPIs and dimensions optimised for semantic querying.

**Characteristics:**

- Dimensional models (star schemas, snowflake schemas â facts and dimensions).
- Surrogate keys used in dimension tables to uniquely identify records, decoupling from business keys.
- Slowly changing dimensions (SCD Type 2) to handle changes over time, with effective/expiry timestamps.
- Pre-aggregated measures and consistent KPIs.
- Predefined hierarchies (e.g. geography, product, time).
- Aggregated data to support various levels of analysis (daily, monthly, etc.).
- Optimised for performance in reporting tools, BI semantic layers, and OLAP engines.
- Consistent, trusted business logic applied across all measures.

**Guidance:**

- Collaborate with business SMEs for metric definitions, KPIs, facts, and dimensions.
- Use business-friendly terminology â convert technical data fields and structures into terms understood by the business.
- Add calculated fields to provide additional insight (e.g. ratios, percentages, derived metrics).
- Structure hierarchical data to support drill-down analysis.
- Capture metric lineage and definitions for transparency.
- Track dimensional changes with effective/expiry timestamps.
- Apply appropriate surrogate key management strategies.
- Use **materialized views** for frequently accessed aggregations to improve query performance.
- Define refresh frequencies and enforce SLAs on data recency.
- Provide semantic metadata for BI tools (e.g. Databricks AI/BI Dashboards, Power BI, Tableau), AI agents, and Databricks Apps.
- Organise Gold tables by business domain (e.g. sales, operations, finance).

---

## Sandbox Layer

**Purpose:** Provide an isolated, flexible environment for data experimentation, exploration, and innovation.

**Characteristics:**

- Dedicated user-specific or team-specific environments for ad hoc analyses, model prototyping, and data discovery.
- Does not affect production data or impact other users' workspaces.
- Temporary data stores and experimental datasets are supported.
- Ideal for exploratory data analysis (EDA), feature engineering, and ML experimentation.

**Guidance:**

- Data is private by default.
- Sharing access must be explicitly granted when collaboration is needed.
- Access controls should align with security and data sensitivity policies.
- Monitor for shadow pipelines â ensure governance guardrails are applied even in experimental workspaces.
- Provide clear policies on data retention and cleanup of sandbox environments.
- Sandbox data should never be promoted directly to Gold â it must flow through the standard pipeline layers.

---

## Quarantine Pattern

**Purpose:** Isolate records that fail validation, quality checks, or schema enforcement at any layer, enabling investigation and reprocessing without blocking the main pipeline.

**Characteristics:**

- Quarantine tables exist alongside the layer where the failure occurred (e.g. a quarantine schema per layer: `bronze_quarantine`, `silver_quarantine`).
- Each quarantined record includes metadata describing the failure reason, source batch, timestamp, and the original record.
- Invalid records are never silently dropped â they are routed to quarantine for investigation.

**Guidance:**

- Implement quarantine at every layer transition (Landing â Raw, Raw â Processed, Processed â Base, Base â Enriched).
- Use Lakeflow Declarative Pipeline **expectations** to automatically quarantine records that violate data quality rules.
- Use the Auto Loader **rescued data column** (`_rescued_data`) to capture records that fail schema parsing.
- Define clear processes for reviewing, correcting, and replaying quarantined records.
- Alert data stewards when quarantine volumes exceed defined thresholds.
- Track quarantine metrics as a KPI for pipeline health.

---

## Data Quality Framework

Data quality is not a single gate but a progressive concern embedded at every layer. The cost of finding and fixing data issues increases exponentially the later they are discovered (the 1:10:100 rule).

| Layer | Quality Focus | Mechanisms |
|-------|--------------|------------|
| **Landing** | File-level validation (format, naming, size, encoding) | Pre-ingestion checks, quarantine of corrupted/malicious files |
| **Bronze (Processed)** | Schema validation, type enforcement, completeness | Auto Loader schema inference, rescued data column, expectations |
| **Silver (Base)** | Business rule validation, deduplication, null handling, referential integrity | Expectations, custom validation rules, quarantine tables |
| **Silver (Enriched)** | Join validation, transformation stability, distribution drift detection | Row count reconciliation, statistical profiling, lineage checks |
| **Gold** | KPI accuracy, compliance, aggregation correctness, SLA adherence | Business rule assertions, metric lineage, automated anomaly detection |

---

## Pipeline Implementation

### Lakeflow Spark Declarative Pipelines

Lakeflow Spark Declarative Pipelines (SDP) â formerly Delta Live Tables (DLT) â is the recommended framework for implementing medallion pipelines in Databricks. Key capabilities include:

- **Streaming tables**: For continuous, append-only ingestion (ideal for Bronze).
- **Materialized views**: For incremental batch transformations that automatically track upstream changes (ideal for Silver and Gold).
- **AutoCDC**: Handles out-of-order CDC events and supports SCD Type 1 and Type 2 patterns without complex manual merge logic.
- **Expectations**: Declarative data quality constraints that can retain, drop, or quarantine invalid records.
- **Automatic lineage**: Full dependency graph managed by the framework.

### Auto Loader

Auto Loader is the recommended ingestion tool for streaming file ingestion from cloud object storage into Bronze. Key features:

- Incremental and cost-efficient ingestion (no unnecessary file listing).
- Schema inference and evolution handled automatically.
- VARIANT column support for maximum resilience against schema changes.
- Rescued data column for capturing records that fail parsing.
- Native integration with Lakeflow Declarative Pipelines.

### Predictive Optimisation

Databricks **predictive optimisation** automatically handles OPTIMIZE, VACUUM, and ANALYZE TABLE operations for managed Delta tables in Unity Catalog. Rather than manually scheduling maintenance, predictive optimisation uses historical access patterns to determine when and how to optimise each table. It is recommended to enable predictive optimisation at the catalog or schema level to reduce operational overhead and ensure consistent table performance across all medallion layers.

### Processing Patterns

| Pattern | Description | Best For |
|---------|-------------|----------|
| **Streaming (append)** | Continuously ingest new files/events as they arrive | Bronze ingestion, event data |
| **Triggered batch** | Process all available new data on a schedule | Most Silver/Gold transformations |
| **Materialized view** | Incrementally refresh based on upstream changes | Silver enrichment, Gold aggregations |
| **AutoCDC** | Apply CDC events with SCD Type 1 or 2 semantics | Base zone from transactional sources |

---

## Zone Comparison

| Zone | Layer | Purpose | Primary Consumers | Structure | Typical Use Cases |
|------|-------|---------|-------------------|-----------|-------------------|
| Landing | Bronze | Temporarily stage raw extracts before ingestion | Platform ingestion framework | Mixed formats (any) | Ingestion buffering, arrival tracking |
| Raw | Bronze | Persist original, unaltered data for traceability and reprocessing | Data Engineers, Auditors | Append-only, immutable, original or Delta format | System of record, audit replay |
| Processed | Bronze | Schema validation, format standardisation | Data Engineers | Append-only, immutable, Delta/Iceberg (UniForm enables cross-format reads) | Schema enforcement, deduplication prep |
| Protected | Silver | Secure and isolate sensitive information (e.g. PI) | Privileged users (via PAM/RBAC) | Structured, access-controlled | Managing regulated or protected data |
| Base | Silver | Cleanse and standardise; apply quality rules | Data Engineers, Analysts, Scientists | Structured with history | Trusted foundation for enrichment |
| Enriched | Silver | Integrate cleansed data with reference/master data; add business context | Data Engineers, Data Scientists | Structured, lightly (de)normalised | Cross-source joins, calculated attributes |
| Exploratory | Gold | Wide, denormalised datasets for ad hoc analysis and data science | Data Analysts, Data Scientists | Denormalised, wide tables | EDA, ML feature prep, cross-domain analysis |
| Dimensional | Gold | Formal semantic models with KPIs and hierarchies for reporting | Business Users, BI Analysts | Star/snowflake schemas | Dashboards, KPI reporting, slice-and-dice |
| Quarantine | Cross-layer | Isolate records failing validation at any layer | Data Engineers, Data Stewards | Structured with failure metadata | Error investigation, reprocessing |
| Sandbox | Separate | Isolated environment for experimentation | Data Scientists, Analysts | Flexible, temporary | Prototyping, EDA, ML experimentation |

---

## Naming Conventions

Maintain consistency, clarity, and usability across datasets, tables, and fields in the platform.

### General Guidelines

- Apply naming conventions uniformly across the entire EDAP.
- Choose names that clearly describe the purpose and contents of the object.
- Avoid complex or cryptic names â favour understandable terminology.
- Use underscores (`_`) to separate words (e.g. `customer_address`).
- Avoid SQL or programming reserved words (e.g. `select`, `order`, `table`).
- Only use common, widely accepted abbreviations where they do not reduce clarity.
- Do not encode medallion layer information in table names when layers are represented as schemas.

### Prefix/Suffix Usage

- Prefix names with the source of the data where appropriate (e.g. `sap_` for SAP, `max_` for Maximo).
- This is particularly relevant in Bronze and Silver Base zones where source alignment is maintained.

### Field Naming

- Use meaningful names that reflect the data stored (e.g. `customer_id`, `order_date`).
- Where abbreviations are used, keep them consistent (e.g. `id` for identifier, `amt` for amount).
- Use business-friendly terminology in Gold layer tables â convert technical field names to terms understood by the business.

### Table Naming

- Tables should follow the pattern: `<subject_area>_<entity>_<granularity>` (e.g. `customer_transaction_daily`).
- In Gold Dimensional zone, use standard prefixes: `dim_` for dimensions, `fact_` for fact tables, `agg_` for aggregates.
- Table and column comments should be maintained in Unity Catalog to support discoverability and AI-powered assistants.

---

## Folder Structure

A well-defined folder structure is crucial for managing the data lifecycle, ensuring traceability, and simplifying operational tasks.

### Common Principles

- Use scripts or orchestration tools to create and manage folder structures automatically, reducing manual errors.
- Apply folder-level access controls, especially for sensitive data, to enforce security policies across all zones.
- Standardise naming conventions across zones to ensure smooth data transitions and improve governance.
- Ensure key metadata (e.g. source, batch, timestamps) is embedded in folder or file names to enhance traceability.
- Implement monitoring for folder usage and automated cleanup procedures for transient data, particularly in the Landing Zone.

### Landing and Raw Zones

```
/bronze/<landing|raw>/
    âââ <source_system>/
          âââ YYYY/
                âââ MM/
                      âââ DD/
                            âââ <edp_batch>/
                                  âââ file1.parquet
                                  âââ ...
```

- Separate folders for each source system or business unit for isolation and security.
- Time-based partitioning (YYYY/MM/DD) for traceability and cleanup.
- Batch identification (`edp_batch`) for detailed tracking.
- Landing zone supports automated cleanup after successful ingestion.
- Raw zone is append-only and immutable.

### Processed Zone

```
/bronze/processed/
    âââ <source_system>/
          âââ <table_name>/
                âââ year=YYYY/
                      âââ month=MM/
                            âââ day=DD/
                                 âââ [Delta/Iceberg data files]
```

- Data structured into logical tables using open table formats (Delta Lake or Iceberg).
- Hive-style partitioning for optimised query performance. Note that for Delta tables, **liquid clustering** is now preferred over Hive-style partitioning â it provides automatic, adaptive data layout and eliminates the need to choose partition columns upfront.
- Partitioning strategy should align with common query patterns and downstream processing needs.

---

## Security Considerations

### Layer-Level Isolation

For maximum security, consider dedicated service principals per medallion layer, each with least-privilege access. This ensures that if Bronze processing is compromised, it cannot read or corrupt Silver and Gold data. Key patterns include:

- Separate compute clusters per layer (Bronze, Silver, Gold) for workload isolation and cost optimisation.
- Dedicated storage credentials and external locations per layer in Unity Catalog.
- Layer-specific cluster policies for right-sizing, autoscaling, and spend control.

### Access Control Model

| Layer | Typical Access |
|-------|---------------|
| Bronze | Data Engineers, Platform team |
| Silver | Data Engineers, Data Analysts, Data Scientists |
| Gold | Business Users, BI Analysts, AI Agents, Databricks Apps, Application services |
| Sandbox | Individual users (private by default) |
| Quarantine | Data Engineers, Data Stewards |

---

## Anti-Patterns to Avoid

- **Writing directly to Silver from ingestion**: Always land in Bronze first. Bypassing Bronze introduces fragility from schema changes and corrupt source records.
- **Silently dropping bad data**: Always quarantine invalid records with failure metadata â never discard them.
- **Over-materialising in Gold**: Large historical datasets should remain in Silver; Gold should contain aggregated, optimised views.
- **Encoding layer names in table names**: Use schemas to represent layers; table names should describe business content.
- **Monolithic pipelines**: Break pipelines into per-layer (or per-source) jobs for isolation, independent scaling, and easier debugging.
- **Shadow pipelines from Sandbox**: Never promote sandbox data directly to Gold â all data must flow through the standard pipeline layers.
- **Treating medallion as a one-time migration**: Each layer needs ongoing ownership, maintenance, monitoring, and continuous improvement.
- **Ignoring schema evolution**: Plan for upstream schema changes from the start â use Auto Loader's schema evolution, VARIANT columns, and rescued data patterns.

---

*This document provides guidance on the information architecture of the Enterprise Data Platform aligned with the medallion methodology. For implementation details on specific pipelines, refer to the Data Engineering Lifecycle, Business Intelligence Lifecycle, and Data Science Lifecycle companion documents.*

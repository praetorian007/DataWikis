# Medallion Architecture — Enterprise Data & Analytics Platform (EDAP) Design Guide

**Mark Shaw** | Principal Data Architect

---

## Overview

The medallion architecture is a data design pattern used to logically organise data in a lakehouse. Its goal is to incrementally and progressively improve the structure and quality of data as it flows through each layer of the architecture. Also referred to as a "multi-hop" architecture, it structures data pipelines into a series of increasingly refined layers — typically **Bronze**, **Silver**, and **Gold** — each denoting a higher level of data quality, structure, and business readiness.

This architecture guarantees atomicity, consistency, isolation, and durability (ACID) as data passes through multiple layers of validation and transformation before being stored in a layout optimised for efficient analytics. By structuring data this way, organisations can streamline data processing, enhance data governance, simplify troubleshooting, and build a single source of truth for enterprise data products.

### Key Principles

- **Progressive refinement**: Each layer builds on the previous, improving structure and quality incrementally.
- **ACID compliance**: Ensured through open table formats (Delta Lake, Apache Iceberg) at every persistent layer. Delta Lake **UniForm** (Universal Format) enables Delta tables to be read natively as Iceberg or Hudi tables without data duplication, providing broad interoperability with external engines.
- **Reproducibility**: All downstream layers should be regenerable from the Bronze layer (the system of record).
- **Separation of concerns**: Each layer has a distinct purpose and clear boundaries — ingestion, validation, enrichment, and consumption are not conflated.
- **Governance by design**: Access controls, lineage, and data quality checks are embedded at every tier, not bolted on afterwards.
- **Bronze immutability**: Bronze is immutable. Never update or delete records in Bronze (outside of legally required right-to-be-forgotten operations — via de-identification). If a source system sends bad data, capture it faithfully; the correction happens in Silver and Gold.

### Flexibility and Adaptability

The medallion architecture is not a rigid prescription. It represents a spectrum of possibilities that can be adapted to unique organisational circumstances. Each of the three core layers can be further segmented into one or more zones to support data management, processing, access controls, or other specific requirements. Some organisations introduce extra layers (e.g. a pre-Bronze landing area or a Platinum layer for specialised data products), and the architecture is compatible with both Data Mesh (domain-specific medallion stacks) and traditional centralised governance models.

> **Note:** See the **Zone Comparison** table at the end of this document for a quick summary of all zones.

---

## Unity Catalog Alignment

In a Databricks environment, the medallion architecture maps naturally to Unity Catalog's three-level namespace: **Catalog → Schema → Table/View/Volume**. The recommended approach is to represent medallion layers as catalogs and zones as schemas, enabling clean access control, lineage tracking, and discoverability.

### Namespace Convention

| Level | Pattern | Example |
|-------|---------|---------|
| **Catalog** | `<environment>_<layer>` | `prod_bronze`, `prod_silver`, `prod_gold` |
| **Schema** | `<source_system\|functional_area\|domain>_<zone_suffix>` | `scada_raw`, `scada_base`, `asset_ops_enriched`, `maintenance_bi` |
| **Table** | `<entity_name>` | `work_orders`, `dim_asset`, `fact_consumption` |

**Zone suffixes:**

| Zone | Suffix | Example Schema |
|------|--------|----------------|
| Raw | `_raw` | `sap_raw` |
| Base | `_base` | `sap_base` |
| Enriched | `_enriched` | `asset_ops_enriched` |
| Exploratory | `_exploratory` | `customer_exploratory` |
| BI | `_bi` | `maintenance_bi` |

**Volume path (Landing Zone):**

```
/Volumes/<environment>_<layer>/<zone>/<source_system>/<year>/<month>/<day>/<file_name>
```

Example: `/Volumes/dev_bronze/landing/grange/2026/01/23/20260123T081500_grange_b.json`

### Governance Features

- **Fine-grained access controls**: Unity Catalog supports table, row, and column-level security via privilege grants (GRANT/REVOKE).
- **Attribute-Based Access Control (ABAC)**: Tag-driven access policies defined at the catalog level and inherited by all child objects. ABAC enables dynamic row filtering and column masking based on governed tags (e.g. `pi_contained`, `pi_type`, `soci_critical`, `access_model`) — replacing per-table policy management with scalable, classification-driven governance across all medallion layers. See the companion **EDAP Access Model** for WC's ABAC policy design.
- **Automatic data lineage**: Unity Catalog captures column-level lineage across layers, providing a clear view of data flow from Bronze through to Gold.
- **Column-level security**: Protect sensitive data (e.g. personal information) with column-level masking and access policies, enforced via ABAC or direct column mask functions.
- **Audit logging**: All data access and changes are logged via system tables for compliance and investigation.

---

## Bronze Layer

**Purpose:** Capture and persist raw data from source systems with minimal transformation for audit, traceability, and reprocessing.

The Bronze layer is the system of record for all ingested data. It preserves a complete historical archive of data as it arrived, enabling full traceability and the ability to replay or reprocess pipelines at any point. No business logic or heavy transformation is applied here — the focus is on reliable capture.

> **Immutability rule:** Bronze is immutable. Never update or delete records in Bronze (outside of legally required right-to-be-forgotten operations — via de-identification). If a source system sends bad data, capture it faithfully; the correction happens in Silver and Gold.

### Landing Zone

**Purpose:** Temporarily stage data before ingestion into the Raw Zone.

**Characteristics:**

- Transient — cleared after successful ingestion; not a persistence layer.
- Contains structured, unstructured, or semi-structured data in its native format.
- Supports any file format (CSV, JSON, XML, Avro, Parquet, binary, images, PDFs, etc.).
- Acts as a buffer between file arrival and processing.

**Guidance:**

- Automate cleanup post-ingestion to reduce storage costs and improve security.
- Validate file-level expectations before promoting to Raw: naming conventions, expected format, encoding, file size within expected range.
- Consider a pre-Bronze quarantine mechanism for virus scanning or validating potentially malicious or corrupted files before they persist in the lakehouse.
- Extracts are typically:
  - **Full extract** — entire dataset loaded each time.
  - **Delta extract** — changes (inserts, updates, and deletions) captured between specific points in time. Always ensure soft and hard deletions are captured.
- Prefer formats with embedded schema (e.g. Parquet over JSON over CSV) to reduce downstream schema inference issues.

**Volume Path:**

```
/Volumes/<environment>_bronze/landing/<source_system>/<year>/<month>/<day>/<file_name>
```

### Raw Zone

**Purpose:** Persist the original, unaltered data for long-term retention, traceability, and replay capability.

**Characteristics:**

- Immutable, append-only storage — data is never modified or overwritten.
- Preserves full fidelity of source data as received.
- Stored as Delta tables with STRING, VARIANT, or BINARY column typing to maximise resilience against upstream schema changes.
- Schema validation and type enforcement are deferred to Silver — Bronze preserves data exactly as received.

**Guidance:**

- Retain data for regulatory, audit, or reprocessing purposes.
- All downstream layers must be regenerable from this zone.
- For Databricks implementations, storing Bronze as Delta tables (rather than raw files in cloud storage) is recommended. Delta provides ACID transactions, time travel for debugging, schema evolution support, liquid clustering for performance (replacing the now-legacy Z-ordering approach), and full Unity Catalog governance including lineage tracking.
- Use **Auto Loader** for efficient, incremental ingestion from cloud object storage, with built-in support for schema inference and evolution.
- Consider ingesting semi-structured data (JSON, XML) as a single **VARIANT** column for maximum robustness against upstream schema changes — this ensures no data is dropped due to unexpected field additions or type changes.
- Use the **rescued data column** (`_rescued_data`) pattern from Auto Loader to capture any data that fails parsing or doesn't match the expected schema, preventing data loss.
- Leverage **Lakeflow Spark Declarative Pipelines** (formerly Delta Live Tables) with **expectations** to define data quality constraints. Expectations can retain, drop, or fail on invalid records, providing built-in quarantine capabilities.

**Audit Columns:**

| Column | Description |
|--------|-------------|
| `edap.file_name` | Source file name (Auto Loader metadata) |
| `edap.file_path` | Source file path (Auto Loader metadata) |
| `edap.file_size` | Source file size in bytes (Auto Loader metadata) |
| `edap.file_modification_time` | Source file last-modified timestamp (Auto Loader metadata) |
| `edap_batch` | Epoch in milliseconds describing when the batch arrived (e.g. `1720071745635`) |
| `edap_hash` | SHA-512 hash of all non-`edap_` prefixed fields (separated by `\|` joiner, including secret salt) — supports change detection and deduplication |
| `edap_inserted_ts` | Timestamp when the record was inserted into the Raw Zone |

A configuration file specifies any fields to be excluded from the calculation of the `edap_hash`.

**Standard Tags:**

| Tag | Description |
|-----|-------------|
| `medallion_layer` | `bronze` |
| `medallion_sublayer` | `raw` |
| `source_system` | Originating system (e.g. `sap`, `maximo`, `scada`) |
| `retention_days` | Retention period in days |
| `data_type` | Nature of the data (e.g. `structured`, `semi_structured`, `time_series`) |
| `pi_contained` | Whether personal information is present (`true`/`false`) |
| `waicp_classification` | WAICP classification (e.g. `OFFICIAL`, `OFFICIAL:Sensitive`) |
| `data_domain` | Business domain (e.g. `asset`, `customer`, `operations`) |
| `data_owner` | Accountable data owner |
| `regulatory_scope` | Applicable regulatory framework(s) (e.g. `soci_act`, `pris_act`, `privacy_act`) |

> **Note:** Tag key names align with the governed tag taxonomy defined in the **EDAP Tagging Strategy**. That document is the authoritative source for allowed values, inheritance rules, and change management procedures.

---

## Silver Layer

**Purpose:** Transform and validate raw data into consistent, structured, and integrated datasets. This layer focuses on improving data quality and making data usable for analysis, while providing an "enterprise view" of key business entities.

The Silver layer is where the ELT methodology applies "just enough" transformation — data is cleansed, conformed, and integrated without over-engineering for specific consumption patterns. It serves as the foundation for Data Engineers, Data Analysts, and Data Scientists to build further analytical products.

### Protected Zone (Optional)

**Purpose:** Securely store and isolate protected information (e.g. Personal Information) in a controlled manner.

> **Note:** The Protected Zone is optional. Unity Catalog governed tags and column-level security can be used as an alternative to physical zone separation — tagging PI columns and applying dynamic masking policies achieves the same access control outcome without requiring a dedicated zone.

**Characteristics:**

- Access is controlled by role-based access policies (RBAC) via Unity Catalog — users require the appropriate role to access specific subject areas (e.g. customer PI).
- Temporary privileged access is provided via PAM (Privileged Access Management) when necessary and revoked automatically after task completion.
- Column-level security and dynamic data masking may be applied to restrict sensitive field visibility.
- May contain derived datasets that associate personal information with demographic or analytical attributes. An anonymised ID and non-sensitive demographic fields are then used in other layers rather than the personal information itself.

**Guidance:**

- Apply column-level masking and row-level security via Unity Catalog to enforce least-privilege access.
- Ensure compliance with relevant privacy regulations: Privacy Act 1988 (Cth) including the Notifiable Data Breaches scheme (Part IIIC), PRIS Act 2024, SOCI Act 2018 (incl. 2024 SOCI Rules amendments expanding positive security obligations), WAICP, and State Records Act 2000.
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
- Structurally aligned with the data source — maintains a recognisable mapping to source entities.
- Acts as a reliable source for further enrichment and analysis.

**Guidance:**

- Define clear, documented data validation and cleansing rules.
- Create processes for handling validation and cleansing failures — records failing quality checks are annotated with DQ columns (see below) rather than routed to separate quarantine tables.
- Design for **idempotent reprocessing** — rerunning a pipeline should produce the same result.
- Use **streaming tables** for append-only source data and **materialized views** for data requiring updates/merges.
- Apply **AutoCDC** (available in Lakeflow Spark Declarative Pipelines) for handling change data capture with SCD Type 1 and Type 2 patterns without complex manual merge logic.
- In the Silver layer, explicitly define expected schemas rather than relying on schema inference — this ensures consistent data types, early detection of schema drift, and predictable downstream behaviour.

**Audit Columns:**

| Column | Description |
|--------|-------------|
| `edap_eff_from` | Record effective-from timestamp (maps to `__START_AT` in SCD Type 2) |
| `edap_eff_to` | Record effective-to timestamp (maps to `__END_AT` in SCD Type 2) |
| `edap_is_current` | Whether this is the current version of the record (`true`/`false`) |
| `edap_is_deleted` | Whether the record has been soft-deleted (`true`/`false`) |
| `edap_inserted_ts` | Timestamp when the record was first inserted |
| `edap_modified_ts` | Timestamp when the record was last modified |
| `edap_user` | Identity of the pipeline or user that last modified the record |
| `dq_status` | Data quality status: `pass`, `warn`, or `fail` |
| `dq_errors` | Array of critical DQ rule violations |
| `dq_warnings` | Array of non-critical DQ observations |
| `dq_checked_ts` | Timestamp of the most recent DQ evaluation |

**Standard Tags:**

| Tag | Description |
|-----|-------------|
| `medallion_layer` | `silver` |
| `medallion_sublayer` | `base` |
| `source_system` | Originating system |
| `retention_days` | Retention period in days |
| `data_type` | Nature of the data |
| `pi_contained` | Whether personal information is present |
| `pi_lawful_basis` | Lawful basis for PI processing (where `pi_contained=true`) |
| `waicp_classification` | WAICP classification |
| `data_domain` | Business domain |
| `data_owner` | Accountable data owner |
| `regulatory_scope` | Applicable regulatory framework(s) |

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

**Audit Columns:** Same effectivity and DQ columns as the Base Zone are carried forward.

**Standard Tags:**

| Tag | Description |
|-----|-------------|
| `medallion_layer` | `silver` |
| `medallion_sublayer` | `enriched` |
| `source_system` | Originating system(s) |
| `retention_days` | Retention period in days |
| `data_type` | Nature of the data |
| `pi_contained` | Whether personal information is present |
| `pi_lawful_basis` | Lawful basis for PI processing (where `pi_contained=true`) |
| `waicp_classification` | WAICP classification |
| `data_domain` | Business domain |
| `data_owner` | Accountable data owner |
| `regulatory_scope` | Applicable regulatory framework(s) |

---

## Gold Layer

**Purpose:** Provide trusted, high-performance datasets tailored for business consumption, decision-making, analytics, and reporting.

The Gold layer is where semantic meaning, business logic, and consumability converge. Data here is fully refined, aggregated, and optimised for end-user queries, dashboards, AI agents, Databricks Apps, machine learning, and business-critical operations. Optimising Gold-layer tables for performance is a best practice because these datasets are frequently queried — large amounts of historical data are typically accessed in the Silver layer and not materialised in Gold.

Gold-layer datasets increasingly serve as the foundation for **data products** — certified, contracted, and discoverable outputs that carry explicit schema definitions, quality guarantees, ownership, and SLAs. Treating Gold outputs as data products ensures they are governed, versioned, and consumable by both human users and automated agents.

**Data product lifecycle:** Gold data products follow a four-state lifecycle managed via the `data_product_tier` governed tag:

| State | Tag Value | Description |
|---|---|---|
| **Incubation** | `experimental` | Under development or in pilot. SLAs are best-effort. Limited consumers. |
| **Active** | `certified` | In production with defined SLAs, data contracts, and monitoring. |
| **Deprecation** | `deprecated` | Scheduled for retirement. Consumers notified. No new consumers onboarded. |
| **Retirement** | *(removed)* | Decommissioned. Tables archived or deleted per retention policy. |

**Additional Gold consumption patterns:** Beyond BI dashboards and Genie spaces, Gold-layer tables serve as inputs to:
- **Feature tables** — Standard Delta tables with primary key and timestamp metadata, registered in Unity Catalog for ML model training and inference. Feature tables in Gold enable point-in-time lookups and automatic feature serving via Online Tables.
- **Vector Search indexes** — Embedding-based similarity search for RAG pipelines, automatically synchronised from Gold Delta tables.
- **Delta Sharing** — External data sharing with governed, zero-copy access. See the companion **Domain Governance Across Systems** document for the six-step governance process applied to external sharing.
- **Databricks Apps** — Custom data applications (Streamlit, Dash, Gradio) consuming Gold tables directly.

> **Note:** In the Gold layer, `edap_` system columns are NOT carried forward. SCD is expressed via `eff_from`, `eff_to`, and `is_current` without the `edap_` prefix, keeping Gold tables clean and business-friendly.

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
- Partition and optimise large tables for performance. Use **liquid clustering** on frequently filtered columns — this is the modern replacement for Z-ordering (now considered legacy for Delta tables) and Hive-style partitioning, providing automatic, adaptive data layout without manual maintenance.
- Maintain documentation on assumptions and enrichments applied.
- Use standardised naming conventions to maintain readability.
- Allow flexibility in schema evolution, but enforce minimum data quality thresholds.

**Typical Use Cases:**

- Exploratory data analysis (EDA)
- Feature engineering for ML models
- Rapid prototyping and hypothesis testing
- Trend analysis across domains (e.g. customer, operations, billing)

### BI Zone

**Purpose:** Support traditional BI, reporting, and metrics through well-structured dimensional models such as star and snowflake schemas. Provide consistent, governed KPIs and dimensions optimised for semantic querying.

Facts, dimensions, and aggregates co-reside in the BI Zone, distinguished by naming convention:

| Prefix | Object Type | Example |
|--------|-------------|---------|
| `fact_` | Fact table | `fact_consumption`, `fact_work_orders` |
| `dim_` | Dimension table | `dim_asset`, `dim_customer`, `dim_date` |
| `agg_` | Pre-aggregated table | `agg_monthly_consumption`, `agg_daily_incidents` |

**Characteristics:**

- Dimensional models (star schemas, snowflake schemas — facts and dimensions).
- Surrogate keys used in dimension tables to uniquely identify records, decoupling from business keys.
- Slowly changing dimensions (SCD Type 2) to handle changes over time, with `eff_from`/`eff_to`/`is_current` columns (no `edap_` prefix).
- Pre-aggregated measures and consistent KPIs.
- Predefined hierarchies (e.g. geography, product, time).
- Aggregated data to support various levels of analysis (daily, monthly, etc.).
- Optimised for performance in reporting tools, BI semantic layers, and OLAP engines.
- Consistent, trusted business logic applied across all measures.

**Guidance:**

- Collaborate with business SMEs for metric definitions, KPIs, facts, and dimensions.
- Use business-friendly terminology — convert technical data fields and structures into terms understood by the business.
- Add calculated fields to provide additional insight (e.g. ratios, percentages, derived metrics).
- Structure hierarchical data to support drill-down analysis.
- Capture metric lineage and definitions for transparency.
- Track dimensional changes with `eff_from`/`eff_to` timestamps.
- Apply appropriate surrogate key management strategies.
- Use **materialized views** for frequently accessed aggregations to improve query performance.
- Define refresh frequencies and enforce SLAs on data recency.
- Provide semantic metadata for BI tools (e.g. Databricks AI/BI Dashboards, Power BI, Tableau), AI agents, and Databricks Apps.
- Organise Gold tables by business domain (e.g. sales, operations, finance).

**Standard Tags (Gold — applies to Exploratory and BI zones):**

| Tag | Description |
|-----|-------------|
| `medallion_layer` | `gold` |
| `medallion_sublayer` | `exploratory` or `bi` |
| `source_system` | Originating system(s) |
| `retention_days` | Retention period in days |
| `data_type` | Nature of the data |
| `pi_contained` | Whether personal information is present |
| `pi_lawful_basis` | Lawful basis for PI processing (where `pi_contained=true`) |
| `waicp_classification` | WAICP classification |
| `data_domain` | Business domain |
| `data_owner` | Accountable data owner |
| `quality_tier` | Steward-certified quality level (`certified`, `provisional`, `uncertified`) |
| `regulatory_scope` | Applicable regulatory framework(s) |
| `model_risk_tier` | For feature tables serving ML models: `high`, `medium`, `low` |
| `ai_governance_level` | For tables serving AI agents: `autonomous`, `human_in_loop`, `human_on_loop`, `human_in_command` |

### Metric Views and UC Metrics

**Purpose:** Provide lightweight, governed metric definitions enabling consistent KPI reporting across tools. Two complementary approaches are available:

**SQL Metric Views** — Traditional SQL views over BI Zone tables. Each view encapsulates a single metric or a cohesive set of related metrics. Consistent grain, filters, and business logic ensure all consumers see the same numbers. Discoverable via Unity Catalog with comments describing the metric definition.

**UC Metrics (Public Preview)** — First-class Unity Catalog objects that define business metrics as catalog-level entities. UC Metrics are defined once and consumed across AI/BI Dashboards, Genie spaces, notebooks, SQL, and (via upcoming integrations) external tools such as Tableau, Hex, Sigma, and ThoughtSpot. UC Metrics carry semantic metadata (display names, formats, descriptions) that improve the accuracy of natural-language queries in Genie. Where available, UC Metrics are the preferred approach for new metric definitions as they provide richer semantic context and broader consumption than SQL views.

**Guidance:**

- Name metric views descriptively (e.g. `metric_monthly_water_loss`, `metric_customer_satisfaction`).
- Document the metric formula, grain, and any filters in the view comment or UC Metric definition.
- Reference `fact_` and `dim_` tables from the BI Zone — do not bypass the dimensional model.
- Register metric views in the data catalogue (Alation) for enterprise discoverability.
- For new metrics, prefer UC Metrics where the capability meets requirements; retain SQL views for complex metrics that exceed UC Metric expression capabilities.

---

## Sandbox Layer

**Purpose:** Provide an isolated, flexible environment for data experimentation, exploration, and innovation.

**Namespace:** `edap_sandbox.<user_or_team>.<table>`

**Characteristics:**

- Dedicated user-specific or team-specific environments for ad hoc analyses, model prototyping, and data discovery.
- Does not affect production data or impact other users' workspaces.
- Temporary data stores and experimental datasets are supported.
- Ideal for exploratory data analysis (EDA), feature engineering, and ML experimentation.

**Guidance:**

- Data is private by default.
- Sharing access must be explicitly granted when collaboration is needed.
- Access controls should align with security and data sensitivity policies.
- When sampling production data into Sandbox, ensure PI is masked or removed — Sandbox environments must not contain unprotected personal information.
- Monitor for shadow pipelines — ensure governance guardrails are applied even in experimental workspaces. Shadow pipelines that bypass standard ingestion and quality processes undermine data trust and governance.
- Provide clear policies on data retention and cleanup of sandbox environments.
- Sandbox data should never be promoted directly to Gold — it must flow through the standard pipeline layers.

---

## Data Quality Approach

Data quality is not a single gate but a progressive concern embedded at every layer. The cost of finding and fixing data issues increases exponentially the later they are discovered (the 1:10:100 rule).

Rather than routing failed records to separate quarantine tables, EDAP uses **DQ annotation columns** (`dq_status`, `dq_errors`, `dq_warnings`, `dq_checked_ts`) embedded in Silver tables. This keeps data co-located, simplifies lineage, and allows consumers to filter by quality status.

| Layer | Quality Focus | Mechanisms |
|-------|--------------|------------|
| **Landing** | File-level validation (format, naming, size, encoding) | Pre-ingestion checks, quarantine of corrupted/malicious files |
| **Bronze (Raw)** | Data capture fidelity, schema resilience | Auto Loader schema inference, rescued data column, VARIANT columns |
| **Silver (Base)** | Business rule validation, deduplication, null handling, referential integrity | Expectations, DQ annotation columns, custom validation rules |
| **Silver (Enriched)** | Join validation, transformation stability, distribution drift detection | Row count reconciliation, statistical profiling, DQ columns carried forward |
| **Gold** | KPI accuracy, compliance, aggregation correctness, SLA adherence | Business rule assertions, metric lineage, automated anomaly detection |

### Data Observability

Beyond inline DQ checks, data observability provides continuous, automated monitoring of data health across all medallion layers. The five pillars of data observability complement the DQ annotation approach:

| Pillar | What It Monitors | Medallion Relevance |
|---|---|---|
| **Freshness** | When was the table last updated? Is it within SLA? | Critical for Gold data products with defined refresh SLAs |
| **Volume** | Are we receiving the expected number of rows? | Detects upstream source failures at Bronze; detects join fan-outs at Silver |
| **Schema** | Have columns been added, removed, or changed type? | Auto Loader and VARIANT columns handle this at Bronze; explicit schema enforcement at Silver catches drift |
| **Distribution** | Are value distributions consistent with historical patterns? | Detects data quality degradation, upstream business changes, or encoding errors |
| **Lineage** | Where did data come from and where does it flow? | Unity Catalog captures column-level lineage automatically across all layers |

Databricks **Data Quality Monitoring** (formerly Lakehouse Monitoring) provides automated anomaly detection for freshness and volume, data profiling, and custom metric tracking — see the companion **Databricks End-to-End Platform** document for details.

**DQ Column Usage:**

- `dq_status = 'pass'` — record passed all quality checks.
- `dq_status = 'warn'` — record passed but has non-critical observations (e.g. unusual but valid values).
- `dq_status = 'fail'` — record failed one or more critical quality rules. Downstream layers should exclude or flag these records.
- Use Lakeflow Spark Declarative Pipeline **expectations** to populate DQ columns automatically.
- Use the Auto Loader **rescued data column** (`_rescued_data`) to capture records that fail schema parsing in Bronze.
- Alert data stewards when `dq_status = 'fail'` volumes exceed defined thresholds.
- Track DQ metrics as a KPI for pipeline health.

---

## Pipeline Implementation

### Lakeflow Spark Declarative Pipelines

Lakeflow Spark Declarative Pipelines (SDP) — formerly Delta Live Tables (DLT) — is the recommended framework for implementing medallion pipelines in Databricks. Key capabilities include:

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
- Native integration with Lakeflow Spark Declarative Pipelines.

### Predictive Optimisation

Databricks **predictive optimisation** automatically handles OPTIMIZE, VACUUM, and ANALYZE TABLE operations for managed Delta tables in Unity Catalog. Rather than manually scheduling maintenance, predictive optimisation uses historical access patterns to determine when and how to optimise each table. It is recommended to enable predictive optimisation at the catalog or schema level to reduce operational overhead and ensure consistent table performance across all medallion layers.

### Lakehouse Federation and the Medallion Architecture

**Lakehouse Federation** enables Unity Catalog to query external data sources (SQL Server, PostgreSQL, Snowflake, BigQuery, and others) without data movement. Federated tables exist outside the medallion pipeline — they are not ingested through Bronze, cleansed in Silver, or aggregated in Gold. Instead, they appear as read-only foreign catalog objects in Unity Catalog, governed by the same ABAC policies and lineage tracking as native tables.

Federated data should be used for:
- **Real-time enrichment** — Joining Gold-layer tables with live operational system data that does not need to be replicated into the lakehouse.
- **Migration transitions** — Querying legacy data warehouse tables during migration without requiring full replication.
- **Low-volume reference lookups** — Accessing external reference data that changes infrequently and does not justify a full ingestion pipeline.

Federated data should **not** replace the standard medallion pipeline for core analytical workloads. High-volume, frequently-queried data should flow through Bronze → Silver → Gold for performance, quality assurance, and historical retention.

Foreign catalogs follow the naming convention `<env>_<source>_federated` (e.g. `prod_sqlserver_federated`), using the `_federated` suffix defined in the EDAP Tagging Strategy.

### Processing Patterns

| Pattern | Description | Best For |
|---------|-------------|----------|
| **Streaming (append)** | Continuously ingest new files/events as they arrive | Bronze ingestion, event data |
| **Triggered batch** | Process all available new data on a schedule | Most Silver/Gold transformations |
| **Materialized view** | Incrementally refresh based on upstream changes | Silver enrichment, Gold aggregations |
| **AutoCDC** | Apply CDC events with SCD Type 1 or 2 semantics | Base zone from transactional sources |

---

## Zone Comparison

| Zone | Layer | Suffix | Purpose | Primary Consumers | Structure |
|------|-------|--------|---------|-------------------|-----------|
| Landing | Bronze | — | Temporarily stage raw extracts before ingestion | Platform ingestion framework | Mixed formats (any) |
| Raw | Bronze | `_raw` | Persist original, unaltered data for traceability and reprocessing | Data Engineers, Auditors | Append-only, immutable, Delta (STRING/VARIANT/BINARY) |
| Protected | Silver | — | Securely isolate sensitive information (e.g. PI) — optional, tags may be used instead | Privileged users (via PAM/RBAC) | Structured, access-controlled |
| Base | Silver | `_base` | Cleanse and standardise; apply quality rules | Data Engineers, Analysts, Scientists | Structured with history and DQ columns |
| Enriched | Silver | `_enriched` | Integrate cleansed data with reference/master data; add business context | Data Engineers, Data Scientists | Structured, lightly (de)normalised |
| Exploratory | Gold | `_exploratory` | Wide, denormalised datasets for ad hoc analysis and data science | Data Analysts, Data Scientists | Denormalised, wide tables |
| BI | Gold | `_bi` | Formal dimensional models with KPIs and hierarchies for reporting | Business Users, BI Analysts | Star/snowflake schemas (fact_, dim_, agg_) |
| Metric Views | Gold | `_bi` | Governed SQL views over BI tables for consistent KPI reporting | Business Users, BI Analysts | SQL views |
| Sandbox | Separate | — | Isolated environment for experimentation | Data Scientists, Analysts | Flexible, temporary |

---

## Naming Conventions

Maintain consistency, clarity, and usability across datasets, tables, and fields in the platform.

### General Guidelines

- Apply naming conventions uniformly across the entire EDAP.
- Choose names that clearly describe the purpose and contents of the object.
- Avoid complex or cryptic names — favour understandable terminology.
- Use underscores (`_`) to separate words (e.g. `customer_address`).
- Avoid SQL or programming reserved words (e.g. `select`, `order`, `table`).
- Only use common, widely accepted abbreviations where they do not reduce clarity.
- Do not encode medallion layer information in table names when layers are represented as catalogs.

### Source System Prefixes

Prefix names with the source of the data where appropriate, particularly in Bronze and Silver Base zones where source alignment is maintained.

| Prefix | Source System |
|--------|--------------|
| `sap_` | SAP |
| `maximo_` | Maximo |
| `scada_` | SCADA |

### Field Naming

- Use meaningful names that reflect the data stored (e.g. `customer_id`, `order_date`).
- Where abbreviations are used, keep them consistent (e.g. `id` for identifier, `amt` for amount).
- Use business-friendly terminology in Gold layer tables — convert technical field names to terms understood by the business.

**Field Key Suffixes:**

| Suffix | Meaning | Example |
|--------|---------|---------|
| `_bk` | Business key | `customer_bk`, `asset_bk` |
| `_sk` | Surrogate key | `customer_sk`, `asset_sk` |
| `_fk` | Foreign key | `customer_fk`, `asset_fk` |

### Table Naming

- Tables should follow the pattern: `<subject_area>_<entity>_<granularity>` (e.g. `customer_transaction_daily`).
- In Gold BI Zone, use standard prefixes: `fact_` for fact tables, `dim_` for dimensions, `agg_` for aggregates.
- Table and column comments should be maintained in Unity Catalog to support discoverability and AI-powered assistants.

---

## Tagging Strategy

Tags provide metadata for governance, cost management, and operational visibility across the EDAP. Apply tags consistently to all taggable assets.

### What to Tag

| Asset Type | Examples |
|------------|----------|
| **Jobs** | Ingestion pipelines, transformation workflows |
| **Instance Pools** | Shared compute pools |
| **Unity Catalog Objects** | Catalogs, schemas, tables, views, volumes |
| **Compute** | Clusters, SQL warehouses |
| **Workspace Assets** | Notebooks, queries, dashboards |
| **ML Assets** | Models, experiments, feature tables |
| **Queries** | Saved SQL queries |

### Standard Tags

The following tags should be applied to Unity Catalog objects (tables, views, volumes) across all medallion layers:

| Tag Key | Layer | Description | Example Values |
|---------|-------|-------------|----------------|
| `waicp_classification` | 1 | WAICP classification | `OFFICIAL`, `OFFICIAL:Sensitive`, `OFFICIAL:Sensitive-Personal` |
| `pi_contained` | 2 | Whether personal information is present | `true`, `false` |
| `pi_type` | 2 | PI sub-classification (column-level, where `pi_contained=true`) | `direct_identifier`, `indirect_identifier`, `sensitive_pi` |
| `regulatory_scope` | 2 | Applicable regulatory framework(s) | `soci_act`, `pris_act`, `privacy_act`, `state_records_act` |
| `sensitivity_type` | 2 | Nature of sensitivity beyond WAICP sublabels | `personal_information`, `infrastructure_vulnerability`, `commercial_in_confidence` |
| `pi_lawful_basis` | 2 | Lawful basis for PI processing (where `pi_contained=true`) | `consent`, `legitimate_interest`, `legal_obligation`, `contractual_necessity` |
| `access_model` | 3 | Access restriction level | `open`, `controlled`, `restricted`, `privileged` |
| `masking_required` | 3 | Column masking behaviour (where `pi_type` set) | `none`, `partial`, `full`, `hash`, `redact` |
| `medallion_layer` | 4 | Layer in the medallion architecture | `bronze`, `silver`, `gold` |
| `medallion_sublayer` | 4 | Zone within the layer | `raw`, `base`, `enriched`, `exploratory`, `bi` |
| `source_system` | 4 | Originating source system | `sap_ecc`, `maximo`, `scada`, `grange` |
| `data_domain` | 4 | Business data domain | `customer`, `asset`, `operations`, `finance` |
| `data_owner` | 4 | Accountable data owner (role or team) | `asset_management`, `customer_services` |
| `soci_critical` | 4 | SOCI Act critical infrastructure flag | `true`, `false` |
| `quality_tier` | 4 | Steward-certified quality level (Silver, Gold) | `certified`, `provisional`, `uncertified` |
| `retention_days` | 3 | Data retention period in days | `365`, `2555` |
| `data_type` | 4 | Nature of the data | `structured`, `semi_structured`, `time_series`, `geospatial` |

> **Note:** For the full tagging taxonomy — including AI/ML governance tags (`model_risk_tier`, `ai_governance_level`), cost allocation tags, operational tags, allowed values, inheritance rules, and change management procedures — refer to the **EDAP Tagging Strategy** companion document, which is the authoritative source for all governed tag definitions.

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

---

## Anti-Patterns to Avoid

- **Writing directly to Silver from ingestion**: Always land in Bronze first. Bypassing Bronze introduces fragility from schema changes and corrupt source records.
- **Silently dropping bad data**: Always annotate invalid records with DQ columns — never discard them.
- **Over-materialising in Gold**: Large historical datasets should remain in Silver; Gold should contain aggregated, optimised views.
- **Encoding layer names in table names**: Use catalogs and schemas to represent layers; table names should describe business content.
- **Monolithic pipelines**: Break pipelines into per-layer (or per-source) jobs for isolation, independent scaling, and easier debugging.
- **Shadow pipelines from Sandbox**: Never promote sandbox data directly to Gold — all data must flow through the standard pipeline layers.
- **Treating medallion as a one-time migration**: Each layer needs ongoing ownership, maintenance, monitoring, and continuous improvement.
- **Ignoring schema evolution**: Plan for upstream schema changes from the start — use Auto Loader's schema evolution, VARIANT columns, and rescued data patterns.
- **Applying business logic in Bronze**: Bronze captures data faithfully — corrections and transformations belong in Silver and Gold.

---

## Companion Documents

| Document | Relationship |
|---|---|
| **EDAP Tagging Strategy** | Authoritative source for the full governed tag taxonomy, allowed values, and change management |
| **EDAP Access Model** | Workspace topology, catalog bindings, ABAC policies, and federated ownership model |
| **Domain Governance Across Systems** | Three-layer governance architecture, data contracts, Delta Sharing governance, Lakehouse Federation governance |
| **Databricks End-to-End Platform** | Platform capabilities reference covering Lakeflow, Unity Catalog, Mosaic AI, and AI/BI |
| **Enterprise Data Models** | Rationale for domain-aligned modelling over monolithic enterprise data models |
| **Data Engineering Lifecycle** | Lifecycle framework for building and operating data pipelines across medallion layers |
| **Business Intelligence Lifecycle** | Lifecycle framework for analytics and reporting consuming Gold-layer data products |
| **Data Science Lifecycle** | Lifecycle framework for ML/AI development consuming Gold-layer feature tables |
| **Data Governance Lifecycle** | End-to-end governance lifecycle from classification through compliance |

---

*This document provides guidance on the information architecture of the Enterprise Data & Analytics Platform aligned with the medallion methodology.*

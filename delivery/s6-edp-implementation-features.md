# S6 – EDP Implementation: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:** Medallion Architecture, EDAP Access Model, Databricks End-to-End Platform, Data Engineering Lifecycle

---

## Feature S6-F1: Source Data Lands and Is Preserved in Its Raw Form

**Description:** Source system data arrives in the platform and is stored exactly as received — append-only, fully traceable to its origin file and batch — so that engineers always have a complete, unmodified record of what was ingested and when.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F1-US01 | Data Engineer | land source system files into Unity Catalog Volumes following the defined folder structure (source_system/YYYY/MM/DD/edp_batch/) | incoming data is staged in governed, access-controlled storage with traceability by source and batch |
| S6-F1-US02 | Data Engineer | ingest files from Landing Zone Volumes into Raw zone Delta tables using Auto Loader with schema inference and evolution | new and changed files are processed incrementally without manual file listing, and schema changes are handled gracefully |
| S6-F1-US03 | Data Engineer | ingest data from supported database sources (e.g. SQL Server) using Lakeflow Connect managed connectors | database ingestion is handled by managed infrastructure with automatic cursor tracking, retry, and incremental reads |
| S6-F1-US04 | Data Engineer | capture semi-structured data (JSON, XML) into Raw zone tables using a VARIANT column | upstream schema changes do not cause data loss or pipeline failures at the ingestion layer |
| S6-F1-US05 | Data Engineer | implement automated cleanup of Landing Zone files after successful ingestion into Raw | storage costs are controlled and stale files do not accumulate in the Landing Zone |
| S6-F1-US06 | Data Engineer | capture file-level metadata (_metadata.file_name, edp_batch) as audit columns in Raw zone tables | every record is traceable to its source file and ingestion batch |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F1-AC01 | A source system file (e.g. Parquet) is placed in the Landing Zone Volume /Volumes/dev_asset/raw/landing/sap/2026/03/19/batch_001/ | Auto Loader processes the file | the data appears in the corresponding raw schema streaming table with _metadata.file_name and edp_batch columns populated |
| S6-F1-AC02 | Auto Loader is configured with schema inference | a new column appears in a subsequent source file | the schema evolves automatically and the new column is captured; no data is silently dropped |
| S6-F1-AC03 | A record fails schema parsing during Auto Loader ingestion | the pipeline completes | the failed record is captured in the _rescued_data column and is available for investigation |
| S6-F1-AC04 | Lakeflow Connect is configured for a SQL Server source | incremental changes are made in the source database | only changed records are ingested into the Raw zone Delta table on the next pipeline run |
| S6-F1-AC05 | A Landing Zone file is successfully ingested into Raw | the cleanup process runs | the source file is removed from the Landing Zone Volume and the cleanup event is logged |
| S6-F1-AC06 | Raw zone tables are inspected | a DELETE or UPDATE operation is attempted on a Raw zone table | the operation is rejected or blocked by pipeline design, confirming append-only semantics |

### Technical Notes
- Landing Zone is implemented using Unity Catalog Volumes within each domain catalog's raw schema, per the access model wiki.
- Raw zone tables are append-only Delta tables preserving source fidelity; they are the system of record for all ingested data.
- Auto Loader's cloud_files format is the recommended ingestion mechanism for file-based sources; Lakeflow Connect for database sources.
- VARIANT columns provide maximum resilience against upstream schema changes for semi-structured data.
- Folder structure follows the medallion architecture wiki: /bronze/landing/<source_system>/YYYY/MM/DD/<edp_batch>/.

---

## Feature S6-F2: Real-Time Data Changes Captured and Available Within Minutes

**Description:** Changes in source databases and event streams are captured continuously and land in the platform within minutes, so that downstream consumers work with near-current data without waiting for overnight batch windows.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F2-US01 | Data Engineer | ingest CDC events from source databases into Bronze streaming tables using Lakeflow Connect's CDC capability | changes (inserts, updates, deletes) are captured incrementally with minimal source system impact |
| S6-F2-US02 | Data Engineer | apply AUTO CDC to produce SCD Type 1 and Type 2 outputs from CDC event streams | historical change tracking is handled automatically without complex manual merge logic |
| S6-F2-US03 | Data Engineer | create streaming tables for continuous, append-only event data (e.g. IoT sensor data, application logs) | event data is ingested with low latency and processed incrementally |
| S6-F2-US04 | Data Engineer | configure event-driven triggers (e.g. S3 event notifications, table update triggers) to activate downstream pipelines | pipelines run only when new data arrives, reducing unnecessary compute costs |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F2-AC01 | Lakeflow Connect CDC is configured for a source database | a row is inserted, updated, and deleted in the source | all three change events are captured in the Bronze streaming table with correct operation type metadata |
| S6-F2-AC02 | AUTO CDC is configured with SCD Type 2 semantics | an existing record is updated in the source | the Silver target table contains both the previous version (with an expiry timestamp) and the current version (with a null expiry timestamp) |
| S6-F2-AC03 | AUTO CDC is configured with SCD Type 1 semantics | an existing record is updated in the source | the Silver target table contains only the latest version of the record |
| S6-F2-AC04 | A streaming table is configured for an event source | 1,000 events are produced to the source within a 5-minute window | all 1,000 events are present in the streaming table after the next micro-batch completes |
| S6-F2-AC05 | An event-driven trigger is configured for a table update | the upstream Bronze table receives new data | the downstream pipeline is triggered automatically within the configured latency threshold |
| S6-F2-AC06 | CDC ingestion is running | out-of-order events arrive (an older update arrives after a newer one) | AUTO CDC handles the ordering correctly using the configured sequence column and the final table state reflects the correct current value |

### Technical Notes
- AUTO CDC INTO (SQL) / APPLY CHANGES INTO (Python) syntax handles out-of-order CDC events and supports both SCD Type 1 and Type 2 patterns.
- Streaming tables are the preferred construct for append-only ingestion in Lakeflow Declarative Pipelines.
- Lakeflow Connect uses Arcion-based CDC technology for low-impact, log-based change capture from databases.
- Event-driven triggers (file arrival, table update) are configured within Lakeflow Jobs to chain ingestion and transformation.

---

## Feature S6-F3: Clean, Conformed Data Available for Analysis

**Description:** Raw source data is cleansed, deduplicated, quality-checked, and enriched into trustworthy Silver-layer tables, so that analysts and downstream pipelines can work with structurally consistent, business-contextualised data instead of raw extracts.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F3-US01 | Data Engineer | build Base zone (silver) transformations using Lakeflow Declarative Pipelines that cleanse, deduplicate, and type-cast data from Raw | the base schema contains trustworthy, structurally consistent data suitable for downstream enrichment |
| S6-F3-US02 | Data Engineer | define data quality expectations (EXPECT, EXPECT OR DROP, EXPECT OR FAIL) on Silver pipeline tables | invalid records are handled according to defined rules — retained with flags, dropped, or cause pipeline failure — with all outcomes tracked |
| S6-F3-US03 | Data Engineer | route records that fail data quality expectations to quarantine tables with failure metadata | no data is silently dropped; failed records are available for investigation and reprocessing |
| S6-F3-US04 | Data Engineer | build Enriched zone (curated schema) transformations that join Base data across sources and apply business logic | the curated schema contains integrated, business-contextualised data ready for Gold layer consumption |
| S6-F3-US05 | Data Engineer | include audit columns (edp_hash for change detection, record effectivity dates, source batch reference) in Base zone tables | change detection, deduplication, and lineage traceability are supported at the record level |
| S6-F3-US06 | Data Engineer | explicitly define expected schemas for Silver layer tables rather than relying on schema inference | data types are consistent, schema drift is detected early, and downstream behaviour is predictable |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F3-AC01 | A Base zone pipeline is deployed for a source entity | new data arrives in the Raw zone | the Base zone table is updated with cleansed, deduplicated, correctly typed records within the defined SLA |
| S6-F3-AC02 | Data quality expectations are defined on a Base zone table (e.g. EXPECT amount > 0 ON VIOLATION DROP ROW) | records with amount <= 0 arrive in Raw | those records are dropped from the Base table and the expectation metrics (rows passed, rows dropped) are recorded in the pipeline event log |
| S6-F3-AC03 | Quarantine tables are configured | records fail a data quality expectation | failed records appear in the corresponding quarantine table with columns for failure_reason, source_batch, and original_record |
| S6-F3-AC04 | An Enriched zone pipeline joins data from multiple Base tables | both source Base tables contain data | the curated schema table contains correctly joined records with no unexpected row duplication or loss; row counts are reconciled |
| S6-F3-AC05 | edp_hash is computed for Base zone records | the same source record is ingested in two consecutive batches with no changes | the edp_hash values match and the record is not duplicated |
| S6-F3-AC06 | Schema is explicitly defined for a Silver table | a source file arrives with an unexpected column type change | the pipeline raises an alert or fails gracefully rather than silently accepting incorrect types |

### Technical Notes
- Base zone maps to the `base` schema; Enriched zone maps to the `curated` schema per the access model wiki.
- Use streaming tables for append-only source data and materialised views for data requiring updates/merges, as recommended in the medallion architecture wiki.
- edp_hash is computed as SHA-512 of all non-edp_ prefixed fields with a secret salt, per the medallion architecture wiki.
- Lakeflow Declarative Pipeline expectations provide built-in quarantine capabilities; expectation metrics are stored in the pipeline event log as Delta tables.
- Pipelines should be designed for idempotent reprocessing as per the Data Engineering Lifecycle wiki.

---

## Feature S6-F4: Business-Ready Data Products Available for Consumption

**Description:** Analysts and BI tools can query well-structured dimensional models, pre-aggregated summaries, and centrally defined metrics in the Gold layer, so that common business questions are answerable immediately without complex joins or tribal knowledge.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F4-US01 | Data Engineer | build Gold layer dimensional models (star schemas with fact and dimension tables) in the product schema using Lakeflow Declarative Pipelines | business users can query well-structured data with standard BI patterns including drill-down, filtering, and slicing |
| S6-F4-US02 | Data Engineer | implement SCD Type 2 handling for dimension tables with surrogate keys, effective/expiry timestamps | historical analysis reflects accurate point-in-time dimension states |
| S6-F4-US03 | Data Engineer | create pre-aggregated KPI and summary tables in the product schema | common business queries execute quickly without requiring complex joins or on-the-fly aggregation |
| S6-F4-US04 | Data Engineer | define UC Metrics (Unity Catalog Metrics) as first-class catalog objects for core business measures | metric definitions are centralised, consistent, and reusable across AI/BI Dashboards, Genie spaces, and SQL |
| S6-F4-US05 | Data Engineer | use materialised views for Gold layer aggregations to enable incremental refresh | Gold tables are refreshed efficiently without full recomputation, reducing compute costs and refresh latency |
| S6-F4-US06 | Data Analyst | query Gold layer tables using business-friendly terminology (e.g. dim_customer, fact_work_order, agg_daily_revenue) | I can understand and use the data without needing to interpret technical field names or source system codes |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F4-AC01 | A dimensional model is built in the product schema | the schema is inspected | it contains appropriately named fact tables (fact_*), dimension tables (dim_*), and aggregate tables (agg_*) following naming conventions |
| S6-F4-AC02 | A dimension table is configured with SCD Type 2 | a dimension attribute changes in the source | the dimension table contains both the prior record (with expiry timestamp set) and the new record (with null expiry timestamp), each with distinct surrogate keys |
| S6-F4-AC03 | UC Metrics are defined for a core business measure | the metric is queried from a Genie space | the metric returns a consistent value that matches the definition and can be sliced by its declared dimensions |
| S6-F4-AC04 | A materialised view is defined for a Gold aggregation | upstream curated data changes | the materialised view is incrementally refreshed and reflects the updated data without full recomputation (confirmed via pipeline metrics) |
| S6-F4-AC05 | Gold layer tables are built | column names and table comments are inspected | all columns use business-friendly names, and table and column comments are populated with clear descriptions |
| S6-F4-AC06 | Gold layer tables are built | a BI tool (e.g. Power BI) connects to a serverless SQL warehouse and queries a Gold table | the query returns correct results within acceptable response times (under 10 seconds for standard dashboard queries) |

### Technical Notes
- Gold layer tables reside in the `product` schema per the access model wiki; they represent certified data products.
- Use dim_ prefix for dimensions, fact_ for fact tables, agg_ for aggregates as specified in the medallion architecture naming conventions.
- UC Metrics (Public Preview) are first-class catalog objects that centralise metric definitions — consume them across Dashboards, Genie, and SQL.
- Materialised views are the preferred Gold construct for aggregations; they support incremental refresh.
- Gold tables should be optimised for consumption — large historical datasets remain in Silver.

---

## Feature S6-F5: Table Maintenance Runs Automatically Without Operator Intervention

**Description:** Delta tables across all medallion layers are automatically optimised, compacted, and vacuumed based on access patterns, so that query performance stays fast and storage costs stay controlled without anyone manually running maintenance commands.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F5-US01 | Data Engineer | apply liquid clustering on frequently filtered columns in Silver and Gold tables | query performance is optimised through adaptive data layout without manual maintenance or upfront partition column selection |
| S6-F5-US02 | Platform Engineer | enable predictive optimisation at the catalog or schema level for managed Delta tables | OPTIMIZE, VACUUM, and ANALYZE TABLE operations are handled automatically based on historical access patterns, reducing operational overhead |
| S6-F5-US03 | Data Engineer | verify that deletion vectors are enabled on Delta tables to improve MERGE, UPDATE, and DELETE performance | write operations on Silver and Gold tables are efficient, with rows marked as deleted without immediate file rewrites |
| S6-F5-US04 | Platform Engineer | configure VACUUM retention policies per medallion layer aligned to regulatory and operational requirements | orphaned data files are cleaned up to reclaim storage while preserving the minimum retention required for time travel and compliance |
| S6-F5-US05 | Data Engineer | monitor table optimisation effectiveness using system tables and table properties | I can identify underperforming tables and validate that optimisation settings are delivering expected improvements |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F5-AC01 | Liquid clustering is applied to a Gold table on its primary filter column(s) | a query filters on the clustered column | query execution time is measurably faster compared to the same query without liquid clustering (confirmed via query profile) |
| S6-F5-AC02 | Predictive optimisation is enabled at the catalog level | a managed Delta table in the catalog has not been manually optimised for 7 days | predictive optimisation has automatically run OPTIMIZE and/or ANALYZE TABLE based on the table's access patterns (confirmed via system tables) |
| S6-F5-AC03 | Deletion vectors are enabled on a Silver table | a MERGE operation updates 10,000 rows | the MERGE completes without rewriting the entire data files; deletion vectors are used (confirmed via table history) |
| S6-F5-AC04 | VACUUM is configured with a retention threshold of 7 days on Silver tables | VACUUM runs (either manually or via predictive optimisation) | data files older than the retention threshold that are no longer referenced by the current table version are removed and storage usage decreases |
| S6-F5-AC05 | Optimisation settings are applied across environments | the Dev environment table optimisation configuration is compared to Prod | Dev uses minimal optimisation settings (reduced clustering, shorter retention) to control non-production costs, while Prod has full optimisation enabled |

### Technical Notes
- Liquid clustering replaces the now-legacy Z-ordering approach and Hive-style partitioning per the medallion architecture wiki.
- Predictive optimisation is recommended at catalog or schema level to ensure all managed tables benefit without per-table configuration.
- Deletion vectors are enabled by default on new Delta tables; verify they are active on existing tables.
- VACUUM retention must consider time travel requirements, regulatory retention (SOCI Act, PRIS Act, State Records Act), and downstream reprocessing needs.
- Dev/Test environments should use reduced optimisation to keep non-production costs within the 10% of production target (S10 requirement).

---

## Feature S6-F6: Every Data Asset Discoverable with Context and Lineage

**Description:** Every table, view, and volume in the platform has a human-readable description, governed classification tags, clear ownership, and automatically captured lineage — so that anyone can find data, understand what it means, know who owns it, and trace where it came from.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F6-US01 | Data Engineer | add table-level and column-level comments/descriptions to all registered tables in Unity Catalog | data assets are discoverable and understandable by analysts, stewards, and AI-powered assistants |
| S6-F6-US02 | Data Steward | apply governed tags (sensitivity, pi_category, soci_critical, domain, data_product_tier) to tables and columns | classification is explicit per object and can drive ABAC policies for access control |
| S6-F6-US03 | Data Engineer | verify that Unity Catalog lineage automatically captures column-level data flow across pipeline layers | data provenance is transparent from Bronze through to Gold without manual lineage documentation |
| S6-F6-US04 | Data Steward | assign table and schema ownership to the appropriate domain steward group | federated ownership is established and domain teams can manage access within their catalogs |
| S6-F6-US05 | Data Analyst | search for data assets in Catalog Explorer using business terms and descriptions | I can discover relevant tables and understand their contents without contacting the data engineering team |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F6-AC01 | A table is registered in the product schema | the table is inspected in Catalog Explorer | table-level and column-level comments are populated and describe the business meaning in clear, non-technical language |
| S6-F6-AC02 | Governed tags are applied to a table containing personal information | the table's tags are inspected | the sensitivity tag is set to restricted and the pi_category tag is set to the appropriate category (e.g. billing, contact) |
| S6-F6-AC03 | A pipeline processes data from Raw through Base to Product | the lineage graph is viewed in Catalog Explorer | column-level lineage is visible tracing data from the Raw source table through Base and Curated to the Product table |
| S6-F6-AC04 | Table ownership is assigned to a domain steward group | the owner of prod_asset.product.dim_asset is queried | the owner is the domain_asset_stewards group |
| S6-F6-AC05 | Comments and tags are applied to all product schema tables | a search for "customer" is performed in Catalog Explorer | relevant customer-related tables are returned with their descriptions, tags, and ownership visible |
| S6-F6-AC06 | Governed tags are applied | a compliance report is generated listing all objects without a sensitivity tag | zero objects in the product schema are returned as untagged; any gaps in base or curated schemas are flagged for remediation |

### Technical Notes
- Governed tags are defined at the account level using tag policies; only allowed values can be assigned per the access model wiki tag taxonomy.
- Tags are assessed and applied explicitly per object — they are never automatically inherited from parent objects.
- Production catalogs and schemas must be owned by groups, never individual users, per the access model ownership rules.
- AI-powered documentation in Unity Catalog can assist with auto-generating descriptions but human review is required.
- Lineage is captured automatically for all queries on Databricks compute (notebooks, jobs, SQL warehouses, SDP pipelines).

---

## Feature S6-F7: Sensitive Data Protected Automatically Based on Classification

**Description:** Personal information is masked and critical infrastructure data is row-filtered automatically based on governed tags, so that sensitive data is protected by default and only visible to explicitly authorised users — without engineers building bespoke access logic per table.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F7-US01 | Platform Engineer | implement RBAC grants following the principle of least privilege (SELECT on product/curated for analysts, MODIFY on raw/base/curated for engineers, MANAGE on domain catalogs for stewards) | each role has precisely the access required and no more |
| S6-F7-US02 | Platform Engineer | implement ABAC policies that mask columns tagged with pi_category values other than none, unless the querying user is in the corresponding pris_authorised_<category> group | personal information is masked by default and only visible to explicitly authorised users |
| S6-F7-US03 | Platform Engineer | implement ABAC row filter policies for tables tagged soci_critical=true, restricting access to members of the soci_critical_access group | SOCI Act critical infrastructure data is only accessible to authorised personnel |
| S6-F7-US04 | Platform Engineer | create dynamic views where complex cross-table access logic is required beyond what ABAC policies can express | advanced access control scenarios are handled without duplicating data or creating bespoke solutions per table |
| S6-F7-US05 | Data Analyst | query a table containing PI-tagged columns without being a member of the authorised group | I see masked values (e.g. ***MASKED***) for PI columns while still being able to work with non-sensitive data in the same table |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F7-AC01 | RBAC grants are configured | a user in domain_asset_analysts queries prod_asset.product.dim_asset | the query succeeds and returns data |
| S6-F7-AC02 | RBAC grants are configured | a user in domain_asset_analysts attempts to INSERT into prod_asset.base.sap_work_orders | the operation is denied with a permission error |
| S6-F7-AC03 | ABAC PI masking policy is applied to prod_customer catalog | a user NOT in pris_authorised_billing queries a table with columns tagged pi_category=billing | the PI-tagged columns return ***MASKED*** for all rows |
| S6-F7-AC04 | ABAC PI masking policy is applied to prod_customer catalog | a user IN pris_authorised_billing queries the same table | the PI-tagged columns return the actual values |
| S6-F7-AC05 | ABAC SOCI row filter policy is applied | a user NOT in soci_critical_access queries a table tagged soci_critical=true | zero rows are returned |
| S6-F7-AC06 | ABAC policies are applied to production catalogs | a developer accesses prod_* data in read-only mode from wc-edap-dev | the same ABAC masking and row filtering policies are enforced regardless of the originating workspace |
| S6-F7-AC07 | All access policies are configured | compute runtime version is checked | all compute accessing ABAC-protected tables runs Databricks Runtime 16.4 or above, or serverless compute |

### Technical Notes
- ABAC policies are defined at the catalog level and inherit downward to all child schemas and tables, per the access model wiki.
- ABAC requires Databricks Runtime 16.4+ or serverless compute.
- PI masking UDFs are defined in the prod_platform.governance schema for centralised management.
- ABAC replaces the previous practice of applying row filters and column masks individually per table.
- Dynamic views provide an optional additional layer for complex cross-table access logic.

---

## Feature S6-F8: Data Quality Certified and Tracked Over Time

**Description:** Data stewards can review, certify, and track the quality of data products against defined standards, and the enterprise catalogue stays synchronised — so that consumers know which data products are trustworthy and classifications drive access control automatically.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F8-US01 | Data Steward | conduct periodic metadata reviews to verify that table descriptions, tags, and ownership are accurate and current | data catalogue quality is maintained and does not degrade over time |
| S6-F8-US02 | Data Steward | certify data products in the product schema using the FAUQD test criteria and update the data_product_tier tag accordingly | consumers can trust that certified data products meet defined quality, accuracy, and usability standards |
| S6-F8-US03 | Platform Engineer | configure the Alation OCF connector to harvest metadata, lineage, and pipeline information from Databricks Unity Catalog | Alation maintains a complete, synchronised view of all EDAP data assets |
| S6-F8-US04 | Data Steward | manage data classifications and tags in Alation and have them synchronised back to Unity Catalog governed tags | tag management is centralised in Alation and ABAC policies in Databricks are driven by Alation classifications |
| S6-F8-US05 | Data Analyst | browse EDAP data assets in Alation with complete metadata, descriptions, lineage, and certification status | I can discover and understand data using the enterprise catalogue without needing direct Databricks access |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F8-AC01 | A metadata review schedule is defined | the review period arrives (e.g. quarterly) | a review checklist is generated listing all product schema tables, their current descriptions, tags, ownership, and last review date |
| S6-F8-AC02 | A data product passes the FAUQD test | the data_product_tier tag is updated | the tag value changes to certified and the certification date is recorded |
| S6-F8-AC03 | The Alation OCF connector is configured | metadata harvest runs | Alation contains accurate, current metadata for all Unity Catalog schemas, tables, columns, descriptions, and tags |
| S6-F8-AC04 | Lineage is harvested by Alation | the lineage view for a Gold table is inspected in Alation | end-to-end lineage from source system through Bronze, Silver, and Gold is visible in Alation |
| S6-F8-AC05 | Alation-to-Unity Catalog tag synchronisation is configured | a sensitivity tag is updated in Alation for a table | the corresponding governed tag in Unity Catalog is updated and the associated ABAC policy takes effect |

### Technical Notes
- Alation OCF (Open Connector Framework) connector for Databricks is the out-of-box integration path.
- Bidirectional metadata flow: Databricks→Alation (harvest) and Alation→Databricks (tag sync for ABAC).
- FAUQD test criteria are defined in the EDAP Data Products wiki (Findable, Accessible, Understandable, Quality, Dependable).
- Data product certification is a stewardship responsibility; the data_product_tier governed tag tracks certification status.

---

## Feature S6-F9: External Systems Can Consume EDAP Data Securely

**Description:** Downstream operational systems, regulators, and partner organisations can receive governed data from the platform through documented, access-controlled integration points — so that EDAP data drives value beyond the analytics platform without bypassing security or audit controls.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S6-F9-US01 | Data Engineer | implement integrations from EDAP to approved external consuming systems per the solution architecture | downstream operational systems and partner platforms receive data from EDAP reliably |
| S6-F9-US02 | Data Engineer | configure Delta Sharing for sharing data products with external parties (regulators, contractors, partner utilities) | external recipients can access governed data without needing a Databricks account, using the open sharing protocol |
| S6-F9-US03 | Data Engineer | implement reverse ETL patterns to push Gold layer data back to operational systems where required | operational systems are enriched with analytics-derived insights without building custom point-to-point integrations |
| S6-F9-US04 | Security Analyst | verify that all external integration points enforce the same access control and audit policies as internal consumption | data shared externally complies with classification, PI masking, and audit requirements |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S6-F9-AC01 | An external integration is configured per the approved architecture | data is produced in the product schema | the external system receives the data within the defined SLA and the delivery is logged in audit tables |
| S6-F9-AC02 | Delta Sharing is configured for an external recipient | the recipient queries the shared data using an open sharing client (e.g. Spark, pandas) | the recipient receives the data successfully and does not require a Databricks account |
| S6-F9-AC03 | Delta Sharing is configured for a table with ABAC policies | the shared data is inspected | PI columns are masked in the shared output and SOCI-restricted rows are filtered, consistent with internal ABAC enforcement |
| S6-F9-AC04 | All external integrations are deployed | the integration inventory is reviewed | each integration has a documented data contract specifying schema, SLA, freshness, and owner |

### Technical Notes
- Delta Sharing provides read-only, time-limited, audit-logged access using the open sharing protocol; recipients do not need Databricks.
- All external integrations must comply with the EDAP access model — ABAC policies apply to shared data.
- Integration patterns should follow the approved solution architecture; custom point-to-point integrations are discouraged.
- Each integration should have a data contract defining schema, quality, freshness, and SLAs per the Data Engineering Lifecycle wiki.

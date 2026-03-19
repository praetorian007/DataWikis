# S13 – Data Integration: Feature Breakdown

**Scope Area:** EDP Detailed Design
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `specifications/edap-pipeline-framework.md` — Metadata-driven framework, Dataflowspec, DLT-META, zone transitions, DQ expectations
- `platform/medallion-architecture.md` — Landing Zone, Raw Zone, Bronze/Silver/Gold layers, quarantine pattern, Auto Loader
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, PI tags, classification lifecycle
- `platform/edap-access-model.md` — Domain catalogues, Unity Catalog Volumes, Landing Zone implementation
- `platform/enterprise-data-models.md` — Data contracts, domain-aligned models

---

## Feature S13-F1: Every Source System Has a Documented, Costed Connectivity Path

**Description:** Solution architects and data engineers can look up any source system and immediately see how it connects to EDAP — which connector to use, what it costs, what its limitations are, and whether it needs custom ingestion — so delivery teams make informed choices without trial-and-error and procurement surprises are eliminated before implementation begins.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S13-F1-US01 | Solution Architect | look up the documented connection method for any data source type (database, file, API, streaming) | the delivery team has a clear reference for how each source system connects to EDAP |
| S13-F1-US02 | Solution Architect | see an evaluation of Lakeflow Connect managed connectors for WC's source systems (SAP, Salesforce, databases) | I can recommend managed connectors where they meet requirements and identify where custom ingestion is needed |
| S13-F1-US03 | Data Engineer | access a connector inventory listing capabilities, limitations, licensing requirements, and associated costs | I can select the appropriate connector for each source without trial-and-error |
| S13-F1-US04 | Platform Administrator | understand external connector licensing requirements and costs | budget and procurement decisions are informed before implementation begins |
| S13-F1-US05 | Solution Architect | review documented limitations of each connectivity method | the team can plan workarounds and avoid discovering limitations during implementation |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S13-F1-AC01 | Source systems for initial use cases are identified | the integration architecture design is completed | a connectivity design document exists mapping each source system to its connection method (Lakeflow Connect, Auto Loader, JDBC, API, streaming), with rationale for the choice |
| S13-F1-AC02 | Lakeflow Connect connectors are evaluated | the connector inventory is published | each available managed connector is listed with: supported source systems, data types, CDC capability, known limitations, licensing model, and estimated cost |
| S13-F1-AC03 | Custom ingestion patterns are required for some sources | the design document is reviewed | alternative approaches are documented for sources where managed connectors are unavailable or insufficient (e.g. JDBC with Auto Loader, API polling, file-based extraction) |
| S13-F1-AC04 | The integration architecture is documented | the architecture is reviewed by the ARB | the design covers: network connectivity (VPC peering, PrivateLink), authentication methods (service principals, secrets management), and data flow diagrams showing source-to-landing-to-raw paths |
| S13-F1-AC05 | Licensing and cost implications are documented | the project manager reviews the integration design | all external connector licensing requirements and estimated costs are listed, with no undocumented licensing dependencies |

### Technical Notes
- The pipeline framework spec identifies three ingestion method categories: Auto Loader (file-based), Lakeflow Connect (managed connector), and streaming/event-based — each producing different column types in the Raw Zone.
- Lakeflow Connect provides native types from source schema (INTEGER, TIMESTAMP, etc.) while Auto Loader produces STRING/VARIANT; the pipeline framework must handle both.
- Landing Zone is implemented using Unity Catalog Volumes per the access model wiki Section 3.3.
- Network connectivity design must consider VPC peering for on-premises sources and PrivateLink for AWS service connectivity.

---

## Feature S13-F2: New Source Entities Onboarded via Configuration, Not Code

**Description:** Data engineers can onboard a new source entity by adding a configuration entry — not by writing custom Spark or SQL — so bulk ingestion scales through configuration, data quality rules are enforced at every zone transition, and the platform team can monitor ingestion health from a single observability layer.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S13-F2-US01 | Data Engineer | onboard a new source entity by adding a configuration entry (Dataflowspec) rather than writing pipeline code | bulk ingestion scales through configuration, not custom development, as required by the scope of work |
| S13-F2-US02 | Data Engineer | use Auto Loader for file-based batch ingestion with schema inference and evolution | new file formats and schema changes are handled automatically without pipeline modifications |
| S13-F2-US03 | Data Engineer | configure data quality expectations at the Bronze Processed zone transition | invalid records are quarantined with failure metadata rather than silently dropped |
| S13-F2-US04 | Data Engineer | group multiple source entities into a single Lakeflow Declarative Pipeline using data flow groups | operational efficiency is balanced with failure isolation per the pipeline framework spec |
| S13-F2-US05 | Platform Administrator | monitor ingestion pipeline health and throughput via Lakeflow observability | I can detect and resolve ingestion failures before they impact downstream consumers |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S13-F2-AC01 | The pipeline framework is implemented | a new source entity is onboarded | the onboarding is completed by adding a JSON configuration file and running the onboarding process — no custom Spark or SQL code is written for the entity |
| S13-F2-AC02 | Auto Loader is configured for a file-based source | source files arrive in the Landing Zone (Unity Catalog Volume) | files are incrementally ingested into the Raw Zone Delta table with `_metadata` columns (file_name, file_path) captured |
| S13-F2-AC03 | The Dataflowspec pattern is implemented | a configuration entry specifies a source entity with type casting, null handling, and DQ expectations | the framework applies all configured transformations and DQ rules without custom code |
| S13-F2-AC04 | DQ expectations are configured for the Bronze Processed zone | a record fails a DQ expectation | the record is routed to the quarantine table with failure metadata (rule name, failed value, timestamp, source batch reference) |
| S13-F2-AC05 | Data flow grouping is configured | 10 source entities from the same system are grouped into a single pipeline | all 10 entities are processed within a single Lakeflow SDP execution, with individual entity failures not blocking other entities in the group |
| S13-F2-AC06 | EDAP audit columns are configured | a batch ingestion pipeline writes to the Raw Zone | each record includes `edp_batch` (epoch milliseconds) and `edp_hash` (SHA-512 hash of non-edp fields) per the pipeline framework spec |
| S13-F2-AC07 | Pipeline observability is enabled | the ingestion pipeline completes (success or failure) | pipeline metrics (rows processed, duration, DQ pass/fail counts, error details) are available in Lakeflow monitoring and `prod_platform` catalogue |

### Technical Notes
- Framework must align to the pipeline framework spec (EDAP-FWK-001): Dataflowspec pattern, data flow grouping, zone-specific processing rules.
- The two-stage onboarding pattern (JSON files for version control, Delta Dataflowspec table for runtime) is recommended per the pipeline framework spec Section 2.6.
- Auto Loader with rescued data column (`_rescued_data`) prevents data loss from schema parsing failures per the medallion architecture wiki.
- VARIANT column support should be used for semi-structured data (JSON, XML) per the medallion architecture wiki.
- Serverless compute is the recommended default for Lakeflow SDP pipelines per the pipeline framework spec Section 2.3.

---

## Feature S13-F3: Database Changes Captured and Reflected in EDAP Within Minutes

**Description:** Inserts, updates, and deletes from transactional source systems are captured incrementally — using Lakeflow AUTO CDC for native change feeds or snapshot comparison for systems without CDC support — so the Silver Base zone reflects current and historical source state without full-table reloads, with out-of-order events handled gracefully and both soft and hard deletes detected.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S13-F3-US01 | Data Engineer | configure CDC ingestion using Lakeflow AUTO CDC for transactional sources | inserts, updates, and deletes from source systems are captured and applied incrementally without full-table reloads |
| S13-F3-US02 | Data Engineer | implement SCD Type 2 historisation for dimension-relevant source entities | a complete change history is maintained with effective/expiry timestamps for audit and time-travel analysis |
| S13-F3-US03 | Data Engineer | handle out-of-order CDC events gracefully | late-arriving events are applied in the correct sequence using the configured `sequence_by` column |
| S13-F3-US04 | Data Engineer | configure snapshot-based CDC for sources that do not provide native change feeds | full-extract sources are compared between snapshots to detect inserts, updates, and deletes |
| S13-F3-US05 | Data Engineer | detect and process both soft and hard deletes from source systems | the Silver Base zone accurately reflects the current and historical state of source data including deleted records |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S13-F3-AC01 | AUTO CDC is configured for a transactional source | the source system inserts, updates, and deletes records | all changes are applied to the Silver Base zone table in the correct order, with the target table reflecting the current state (SCD Type 1) or full history (SCD Type 2) |
| S13-F3-AC02 | SCD Type 2 is configured for a dimension entity | a source record attribute is updated | the existing target record is closed (expiry timestamp set) and a new record is inserted with the updated attribute and a new effective timestamp |
| S13-F3-AC03 | Out-of-order events arrive | a CDC event with an earlier sequence number arrives after a later event has been processed | the event is applied correctly based on `sequence_by` ordering, and the target table reflects the correct state |
| S13-F3-AC04 | Snapshot-based CDC is configured | two consecutive full extracts are processed for the same source entity | inserts, updates, and deletes between the two snapshots are detected and applied to the target table |
| S13-F3-AC05 | Delete detection is configured | a source record is soft-deleted (flagged) or hard-deleted (removed from extract) | the Silver Base zone table reflects the deletion: soft deletes are flagged via a deletion indicator column; hard deletes are detected via snapshot comparison |
| S13-F3-AC06 | CDC processing is configured via the pipeline framework | a new CDC source is onboarded | the onboarding is achieved through Dataflowspec configuration (specifying `sequence_by`, `apply_as_deletes`, `except_column_list`, SCD type) without custom merge code |

### Technical Notes
- Lakeflow AUTO CDC APIs (`AUTO CDC INTO`, `create_auto_cdc_flow`) replace APPLY CHANGES APIs per the pipeline framework spec Section 2.3.
- `apply_changes_from_snapshot` API handles full-extract sources without native CDC per the pipeline framework spec Section 2.6 (DLT-META snapshot CDC capability).
- Pipeline chaining constraint: SCD Type 2 streaming tables cannot be used as streaming sources downstream without `skipChangeCommits` per the pipeline framework spec Section 2.3.
- Both soft and hard deletes must be captured per the medallion architecture wiki Landing Zone guidance.
- `edp_hash` (SHA-512) supports change detection for sources without native CDC per the pipeline framework spec.

---

## Feature S13-F4: Real-Time Event Data Available as Queryable Tables

**Description:** Real-time data from streaming sources (Kafka, Event Hubs, SCADA telemetry) is continuously ingested into queryable Delta tables — with the same DQ expectations and quarantine handling as batch data — so analysts and operational systems can access near-real-time information within minutes of generation at source.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S13-F4-US01 | Data Engineer | configure streaming tables for continuous ingestion from real-time sources (e.g. Kafka, Event Hubs, SCADA telemetry) | data is available for analysis within minutes of generation at the source |
| S13-F4-US02 | Data Engineer | configure event-driven triggers (e.g. S3 event notifications) to initiate pipeline processing | pipelines execute automatically when new data arrives without polling or scheduled intervals |
| S13-F4-US03 | Data Engineer | apply DQ expectations to streaming data in-flight | invalid records from streaming sources are quarantined in the same manner as batch records |
| S13-F4-US04 | Platform Administrator | monitor streaming pipeline health including lag, throughput, and error rates | streaming workloads are observable and issues are detected before they cause data staleness |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S13-F4-AC01 | A streaming table is configured for a real-time source | events are produced at the source | events are ingested into the Raw Zone streaming table within the configured latency window (default: <5 minutes end-to-end) |
| S13-F4-AC02 | Event-driven triggers are configured | a new file lands in the Landing Zone S3 path | an Auto Loader pipeline is triggered to process the file without manual intervention or scheduled polling |
| S13-F4-AC03 | DQ expectations are applied to a streaming pipeline | a malformed event arrives on the stream | the event is quarantined with failure metadata, and valid events continue processing without interruption |
| S13-F4-AC04 | Streaming pipeline monitoring is configured | a streaming pipeline is running | lag metrics (time from event generation to target table availability), throughput (events/second), and error rates are visible in the Lakeflow monitoring dashboard |
| S13-F4-AC05 | Streaming source schema evolution occurs | the source schema adds a new field | the streaming pipeline handles the change gracefully using Auto Loader schema evolution or VARIANT column patterns without pipeline failure |

### Technical Notes
- Streaming tables are the recommended approach for append-only ingestion per the medallion architecture wiki and pipeline framework spec.
- Auto Loader handles streaming file ingestion with schema inference and evolution per the medallion architecture wiki.
- VARIANT column approach provides maximum resilience against upstream schema changes for JSON/XML streaming sources per the medallion architecture wiki.
- Event-driven triggers (S3 event notifications) align with the DataOps scope item (S7) orchestration requirements.
- Streaming sources may produce STRING or VARIANT column types per the pipeline framework spec Section 2.4.

---

## Feature S13-F5: PI and Sensitive Data Identified and Tagged Automatically During Ingestion

**Description:** Newly ingested tables are automatically scanned for Personal Information using Unity Catalog Data Classification, with results presented to domain stewards for review and approval — so PI columns are tagged and ABAC-protected as part of the standard onboarding workflow, not as an afterthought, and the Data Protection Officer has a complete PI inventory across all production catalogues.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S13-F5-US01 | Domain Data Steward | have PI automatically detected in newly ingested tables using Unity Catalog Data Classification | I receive recommendations for PI tagging rather than manually inspecting every column |
| S13-F5-US02 | Domain Data Steward | review and approve automated PI detection results before tags are applied | automated classification informs but does not bypass the explicit classification assessment required by governance policy |
| S13-F5-US03 | Technical Data Steward | apply `pii_contained`, `pii_type`, and `pi_category` governed tags to confirmed PI columns | ABAC masking policies are automatically enforced on the tagged columns |
| S13-F5-US04 | Data Protection Officer | generate a report of all PI-tagged columns across production catalogues | I have a complete inventory of PI in the platform for PRIS Act compliance |
| S13-F5-US05 | Data Engineer | integrate PI classification into the ingestion pipeline | newly onboarded source entities are flagged for PI review as part of the standard onboarding workflow |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S13-F5-AC01 | Unity Catalog Data Classification is enabled | a new table is created in a production catalogue | automated PI detection runs and generates classification recommendations (columns likely containing names, addresses, emails, phone numbers, identifiers) |
| S13-F5-AC02 | Automated classification results are available | a domain steward reviews the recommendations | they can approve, reject, or modify each recommendation before governed tags are applied |
| S13-F5-AC03 | Governed tags are applied to confirmed PI columns | a `pi_category=contact` tag is applied to a column | the ABAC PI masking policy automatically masks the column for users not in `pris_authorised_contact` |
| S13-F5-AC04 | PI inventory reporting is configured | a monthly PI report is generated | the report lists all columns tagged with any `pi_category` value, grouped by domain and catalogue, with the tagging steward and date |
| S13-F5-AC05 | Classification is integrated into onboarding | a new source entity is onboarded via the pipeline framework | the onboarding workflow includes a PI classification step that flags the entity for steward review with an initial classification status of `unclassified` per the tagging strategy |
| S13-F5-AC06 | Classification coverage is monitored | a weekly classification coverage report runs | the report shows the percentage of Silver and Gold tables with completed PI classification (target: 100% for production tables within 30 days of creation) |

### Technical Notes
- Unity Catalog Data Classification is in Public Preview with GA for compliance profiles expected mid-March 2026 per the access model wiki Section 12.
- WC uses PI (Personal Information) terminology per PRIS Act 2024, not PII, per the access model wiki Section 2.
- Classification must be explicit per object; auto-detection provides recommendations but does not auto-tag per the access model wiki Section 6.2.
- Tagging aligns to the tagging strategy wiki Layer 2 tags (`pii_contained`, `pii_type`) and the access model wiki governed tag taxonomy (`pi_category`).
- The classification lifecycle (unclassified -> provisional -> classified) from the tagging strategy wiki governs cross-domain access during the assessment period.

---

## Feature S13-F6: Documents and Files Governed Alongside Structured Data

**Description:** Unstructured files (PDFs, images, Word documents) are ingested into Unity Catalog Volumes within domain catalogues — classified, tagged, and access-controlled with the same governance as structured Delta tables — so users can discover documents through the catalogue and security officers can verify that sensitive files are protected consistently.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S13-F6-US01 | Data Engineer | ingest unstructured files (PDFs, images, Word documents) into Unity Catalog Volumes within domain catalogues | unstructured data is governed with the same access control and audit trail as structured Delta tables |
| S13-F6-US02 | Domain Data Steward | classify and tag unstructured data Volumes with the same governed tags used for structured data | WAICP classification, sensitivity, and PI indicators are applied consistently across all data types |
| S13-F6-US03 | Data Analyst | discover and access unstructured data through the Unity Catalog and Alation | I can find relevant documents and files without relying on ad-hoc file shares |
| S13-F6-US04 | Security Officer | verify that access controls on Volumes prevent unauthorised access to sensitive documents | unstructured data containing PI or commercially sensitive information is protected per governance policy |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S13-F6-AC01 | Unity Catalog Volumes are created within domain catalogues | unstructured files are uploaded to a Volume | the files are accessible via the Volume path (e.g. `/Volumes/prod_asset/raw/landing/`) and governed by Unity Catalog access controls |
| S13-F6-AC02 | Governed tags are applied to Volumes | a Volume containing sensitive documents is tagged with `sensitivity=restricted` | only users with appropriate grants can read files from the Volume |
| S13-F6-AC03 | Volume access is audited | a user reads a file from a Unity Catalog Volume | the access event is captured in the Unity Catalog audit log with user, timestamp, and file path |
| S13-F6-AC04 | Unstructured data is discoverable | Volumes are registered in Unity Catalog | they appear in Catalog Explorer and Alation with descriptions, tags, and ownership metadata |
| S13-F6-AC05 | File-level validation is applied | files arrive in a Landing Zone Volume | pre-ingestion checks validate naming conventions, expected format, file size, and encoding per the medallion architecture wiki Landing Zone guidance |

### Technical Notes
- Unity Catalog Volumes replace legacy DBFS mounts per the access model wiki Section 3.3: "Volumes provide governed, access-controlled file storage."
- Landing Zone is implemented as Volumes, not schemas per the access model wiki: files land in managed or external Volumes before being ingested into Raw Zone tables.
- Volumes support the `essential_eight` governed tag per the access model wiki tag taxonomy.
- READ FILES and WRITE FILES are never granted on external locations to end users; all file access uses Volumes per the access model wiki Section 8.
- Consider virus scanning or quarantine for potentially malicious files per the medallion architecture wiki Landing Zone guidance.

---

## Feature S13-F7: All Architecture Patterns Proven End-to-End Before Use Case Delivery

**Description:** Every integration pattern (batch, CDC, streaming, API, unstructured) is implemented and validated as a technical use case in the staging environment — with end-to-end lineage confirmed, the full Dev-to-Prod promotion path exercised, and documented implementation guidance produced — so delivery teams can follow proven patterns with confidence when building business use cases.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S13-F7-US01 | Solution Architect | prove each integration architecture pattern (batch, CDC, streaming, API, unstructured) end-to-end in the staging environment | the architecture is validated before business use case delivery depends on it |
| S13-F7-US02 | Data Engineer | verify that end-to-end lineage is captured from source through Landing, Raw, Base, Enriched, and Product zones | data lineage is traceable across the full medallion architecture for governance and debugging |
| S13-F7-US03 | Platform Administrator | validate that technical patterns work correctly across the full Dev -> Staging -> Prod promotion path | the deployment pipeline is proven for each pattern type before business use cases go live |
| S13-F7-US04 | Solution Architect | document the validated patterns with implementation guidance for the delivery team | future use case development can follow proven patterns without re-discovery |
| S13-F7-US05 | Data Engineer | validate the metadata-driven pipeline framework against representative source system data | the framework handles real-world data volumes, schema variations, and edge cases reliably |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S13-F7-AC01 | Architecture patterns from Appendix 6 are identified | each pattern type is implemented as a technical use case | a working end-to-end pipeline exists in the staging environment for each pattern (batch file ingestion, managed connector ingestion, CDC ingestion, streaming ingestion, API ingestion, unstructured data ingestion) |
| S13-F7-AC02 | A technical pattern pipeline runs end-to-end | lineage is queried in Unity Catalog | the lineage graph shows the complete data flow from source (Landing/Raw) through all intermediate zone transitions to the final Gold target |
| S13-F7-AC03 | A technical pattern is validated in staging | the pattern is promoted to production | the promotion follows the standard DABs deployment pipeline (dev -> staging -> prod) and the pattern functions identically in production |
| S13-F7-AC04 | Technical patterns are validated | pattern documentation is produced | each validated pattern has a documented implementation guide covering: architecture diagram, configuration example (Dataflowspec), DQ expectations, known constraints, and performance characteristics |
| S13-F7-AC05 | The pipeline framework is validated against real data | representative source system data is processed | the framework handles: schema variations between sources, type casting from STRING to native types, nested JSON/VARIANT flattening, and DQ quarantining without pipeline failure |
| S13-F7-AC06 | All required patterns are validated | the validation report is reviewed by the ARB | all patterns listed in Appendix 6 are confirmed as proven, with any limitations or workarounds documented and accepted |

### Technical Notes
- Appendix 6 (referenced in the scope of work) defines the full list of architecture patterns to be validated; this feature ensures each is proven.
- End-to-end lineage uses Unity Catalog system tables (`system.access.table_lineage`) per the access model wiki Section 10.2.
- DABs (Databricks Asset Bundles) are the deployment mechanism for pipeline promotion per the pipeline framework spec.
- Pattern validation should include the pipeline framework's Dataflowspec configuration approach to verify zero-code onboarding works for each pattern type.
- Internal pipeline staging (for complex structural transformations) should be validated as part of the pattern proving per the pipeline framework spec Section 2.5.

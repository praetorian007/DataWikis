# S2 – Enterprise Data Platform Design: Feature Breakdown

**Scope Area:** EDP Detailed Design
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `platform/medallion-architecture.md` — Zone decomposition, liquid clustering, predictive optimisation, quarantine pattern
- `platform/edap-access-model.md` — Workspace topology, domain catalogs, schema naming (raw/base/curated/product/sandbox), ABAC
- `platform/databricks-end-to-end-platform.md` — Lakeflow Connect, SDP, Unity Catalog, AI/BI, Mosaic AI, DABs
- `platform/enterprise-data-models.md` — Thin enterprise ontology, data contracts, domain-aligned modelling
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, classification lifecycle
- `specifications/edap-pipeline-framework.md` — Metadata-driven framework, Dataflowspec, DQ expectations
- `lifecycles/data-engineering-lifecycle.md` — Ingestion, storage, transformation, serving patterns

---

## Feature S2-F1: Approved Environment and Workspace Topology

**Description:** The delivery team and WC SMEs have an agreed, documented environment structure – Dev, Test, Prod, and Sandbox – with clear workspace boundaries, network isolation, and promotion paths so that every subsequent scope item builds on a common foundation.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S2-F1-US01 | Solution Architect | define the workspace topology (number of workspaces, purpose of each, network boundaries) with WC infrastructure SMEs | the environment structure is agreed before any provisioning begins |
| S2-F1-US02 | Data Engineer | understand which workspace I develop in, which I test in, and how code promotes to production | I follow a clear path from development to production without ambiguity |
| S2-F1-US03 | Platform Administrator | see documented network isolation between environments (VPC peering, PrivateLink, security groups) | I can provision environments knowing the security boundaries are defined |
| S2-F1-US04 | Security Officer | verify that production is isolated from non-production with no direct data access paths | regulatory and security requirements are met from the architecture level |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S2-F1-AC01 | WC SMEs and the delivery team have workshopped the environment structure | the Data Platform Design document is reviewed | it contains a workspace topology diagram showing Dev, Test, Prod, and Sandbox with network boundaries, purpose statements, and user access profiles for each |
| S2-F1-AC02 | The environment structure is documented | a reviewer checks for promotion paths | the document defines how assets move from Dev → Test → Prod, including approval gates and the role of DABs |
| S2-F1-AC03 | The environment structure is documented | a reviewer checks for sandbox governance | sandbox workspace rules are defined: retention policy, data access scope, promotion path for successful experiments, and cost guardrails |
| S2-F1-AC04 | The Data Platform Design is submitted for approval | WC stakeholders review it | the environment topology section is approved and signed off before S5 configuration begins |

### Technical Notes
- Workspace topology should align to the access model wiki: production workspace restricted to stewards, service principals, and platform admins; development open to engineers.
- Sandbox workspace provides exploratory access with 90-day retention and no production write access.
- Network design should document VPC peering for on-premises sources and PrivateLink for AWS service connectivity.

---

## Feature S2-F2: Medallion Architecture and Storage Layout Agreed

**Description:** A detailed storage layout is documented and approved – covering Landing, Bronze (Raw/Processed), Silver (Base/Enriched), and Gold (Dimensional/Exploratory) zones – with file formats, naming conventions, and retention policies so that all implementation teams build consistently.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S2-F2-US01 | Data Engineer | reference a single design document that defines each medallion zone, its purpose, schema naming, and data quality expectations | I know exactly where data belongs at each stage of the pipeline |
| S2-F2-US02 | Solution Architect | define the mapping between medallion zones and Unity Catalog schemas (raw, base, curated, product, sandbox) | the logical architecture maps cleanly to the physical catalog structure |
| S2-F2-US03 | Data Engineer | understand file format standards (Delta as default, Parquet for external sharing) and when exceptions apply | I don't make ad-hoc format decisions during implementation |
| S2-F2-US04 | Data Governance Lead | see documented retention and purge policies for each zone | compliance with State Records Act 2000 and operational requirements is designed in from the start |
| S2-F2-US05 | Data Engineer | understand the quarantine pattern for records that fail data quality expectations | I know how rejected records are handled without losing data |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S2-F2-AC01 | The medallion architecture is designed | the Data Platform Design is reviewed | it defines each zone (Landing, Raw, Processed, Protected, Base, Enriched, Exploratory, Dimensional, Sandbox, Quarantine) with purpose, schema naming convention, and data quality expectations |
| S2-F2-AC02 | Schema naming is defined | a reviewer checks alignment with the access model | schema names (raw, base, curated, product, sandbox) map explicitly to medallion zones with a published mapping table |
| S2-F2-AC03 | Retention policies are documented | a reviewer checks for regulatory alignment | each zone has a defined retention period, purge mechanism, and regulatory justification (State Records Act, SOCI Act, PRIS Act where applicable) |
| S2-F2-AC04 | The quarantine pattern is documented | a reviewer checks for completeness | the design specifies how quarantined records are stored, what metadata is captured (failure reason, timestamp, source), and the remediation workflow |
| S2-F2-AC05 | File format standards are documented | a reviewer checks for exceptions | Delta Lake is the default format with documented exceptions (e.g., Parquet for Delta Sharing to non-Databricks consumers), and liquid clustering is specified as the default optimisation strategy |

### Technical Notes
- Storage layout should align to the medallion architecture wiki's zone decomposition.
- Landing Zone implemented via Unity Catalog Volumes (managed or external) per the access model wiki.
- Liquid clustering replaces Z-ordering and Hive-style partitioning as the default optimisation strategy.
- Predictive optimisation should be specified as the default for OPTIMIZE, VACUUM, and ANALYZE TABLE.

---

## Feature S2-F3: Ingestion Patterns Documented for Every Source Type

**Description:** Every ingestion pattern the platform will use – batch, CDC, streaming, file-based, API-based – is documented with technology choices, failure handling, and schema validation rules so that data engineers can implement any source without designing from scratch.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S2-F3-US01 | Data Engineer | reference documented ingestion patterns for each source type (database, file, API, streaming) | I select the right pattern for each source without reinventing the approach |
| S2-F3-US02 | Solution Architect | see a decision matrix mapping source characteristics to ingestion tools (Lakeflow Connect, Auto Loader, Lakeflow AUTO CDC, custom JDBC) | technology selection is consistent and justified |
| S2-F3-US03 | Data Engineer | understand the landing zone interface – what is platform-owned ingestion vs externally landed data | I know whether to build an ingestion pipeline or expect data to arrive via an external process |
| S2-F3-US04 | Data Engineer | see documented failure handling and retry patterns for each ingestion method | I handle failures consistently across all pipelines |
| S2-F3-US05 | Data Engineer | understand schema validation and drift handling at the point of ingestion | I know how schema evolution is managed and when manual intervention is required |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S2-F3-AC01 | Ingestion patterns are designed | the Data Platform Design is reviewed | it documents at least four ingestion patterns: batch file (Auto Loader), managed connector (Lakeflow Connect), CDC (Lakeflow AUTO CDC), and streaming (Kafka/Kinesis) |
| S2-F3-AC02 | A decision matrix is published | a data engineer needs to ingest from a new source | they can look up the source type and see the recommended ingestion tool, expected latency, CDC capability, and known limitations |
| S2-F3-AC03 | Landing zone interfaces are documented | a reviewer checks for boundary clarity | the design clearly distinguishes platform-owned ingestion (Databricks pulls data) from externally ingested data (source system pushes to S3/Volumes) with ownership and SLA for each |
| S2-F3-AC04 | Failure handling is documented | a reviewer checks each ingestion pattern | each pattern specifies: retry strategy, dead-letter/quarantine handling, alerting mechanism, and manual recovery procedure |
| S2-F3-AC05 | Schema validation is documented | a reviewer checks for drift handling | the design specifies how schema evolution is handled (Auto Loader rescue columns, SDP expectations, manual approval for breaking changes) |

### Technical Notes
- Ingestion patterns should align to the pipeline framework spec (EDAP-FWK-001): Auto Loader for file-based, Lakeflow Connect for managed connectors, AUTO CDC for database CDC.
- External ingestion tools (Glue, DMS, AppFlow) should be documented as landing zone sources with clear handoff points.
- The metadata-driven framework (Dataflowspec) should be referenced as the configuration mechanism for bulk onboarding.

---

## Feature S2-F4: Metadata-Driven Transformation Framework Designed

**Description:** The transformation framework design is documented – specifying how data flows from Bronze through Silver to Gold using Lakeflow Declarative Pipelines and metadata-driven configuration – so that implementation teams have a blueprint for building scalable, consistent pipelines.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S2-F4-US01 | Data Engineer | understand the transformation framework design: how Dataflowspec configuration drives pipeline behaviour | I can implement transformations by extending configuration rather than writing bespoke pipeline code |
| S2-F4-US02 | Data Engineer | see documented patterns for each zone transition (Raw→Base, Base→Enriched, Enriched→Dimensional) | I apply the correct transformation logic at each layer |
| S2-F4-US03 | Solution Architect | see the reverse ETL design for pushing data back to target applications | outbound data flows are architecturally consistent with inbound patterns |
| S2-F4-US04 | BI Developer | understand how Gold-layer outputs are designed to support Power BI, AI/BI Dashboards, and Databricks SQL | I know the serving layer structure and can design semantic models against it |
| S2-F4-US05 | Data Product Owner | see how data products are consumed via Databricks SQL, Delta Sharing, and REST APIs | I can plan how downstream consumers will access my data product |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S2-F4-AC01 | The transformation framework is designed | the Data Platform Design is reviewed | it describes the Dataflowspec-driven approach using Lakeflow Declarative Pipelines (SDP) with zone-transition patterns for Raw→Base, Base→Enriched, and Enriched→Dimensional |
| S2-F4-AC02 | Zone transition patterns are documented | a reviewer checks each transition | each transition specifies: DQ expectations applied, SCD handling approach (Type 1/Type 2), audit column population, and quarantine rules |
| S2-F4-AC03 | The reverse ETL pattern is documented | a reviewer checks for completeness | the design specifies the approach for pushing data back to target applications (Lakeflow Connect reverse sync, API-based, file export) with governance and audit requirements |
| S2-F4-AC04 | The serving layer is documented | a reviewer checks consumption patterns | the design defines how Gold-layer data is consumed: Databricks SQL Warehouses for interactive queries, Delta Sharing for external consumers, REST APIs for application integration, and UC Metrics for governed business metrics |
| S2-F4-AC05 | Data sharing mechanisms are documented | a reviewer checks for security | the design specifies Delta Sharing configuration, recipient management, audit logging, and data product registration for cross-team and external sharing |

### Technical Notes
- Transformation framework should align to the pipeline framework spec (EDAP-FWK-001) and its Dataflowspec configuration model.
- Lakeflow Declarative Pipelines (SDP) is the primary transformation engine; the design should specify streaming tables vs materialised views decision criteria.
- UC Metrics should be specified as the governed semantic layer for business metric definitions.
- Delta Sharing should be specified for external data product distribution.

---

## Feature S2-F5: Unity Catalog Structure, Access Model, and Lineage Designed

**Description:** The Unity Catalog structure is documented – covering metastore, catalog naming (domain-based), schema layout, access policies (RBAC/ABAC), and lineage model – so that security and governance are designed in from the start, not bolted on during implementation.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S2-F5-US01 | Security Officer | see the Unity Catalog structure with domain-based catalogs and environment separation | I can verify that data isolation meets regulatory requirements before implementation |
| S2-F5-US02 | Domain Data Steward | understand how ABAC policies will enforce access based on governed tags from Alation | I know how my classification decisions translate into technical access control |
| S2-F5-US03 | Platform Administrator | see the Entra ID integration design (SCIM/SSO) and group structure | I can plan identity provisioning alongside platform configuration |
| S2-F5-US04 | Data Engineer | understand the lineage model – what is captured automatically vs what requires annotation | I know my obligations for maintaining lineage integrity |
| S2-F5-US05 | Compliance Officer | see documented encryption standards, secrets management approach, and audit logging design | I can verify alignment with Essential Eight, SOCI Act, and PRIS Act requirements |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S2-F5-AC01 | The Unity Catalog structure is designed | the Data Platform Design is reviewed | it defines domain-based catalogs (`prod_asset`, `prod_customer`, etc.), environment prefixing (`dev_`, `test_`, `prod_`), and schema layout (raw, base, curated, product, sandbox) with a complete catalog inventory |
| S2-F5-AC02 | The access model is designed | a reviewer checks for RBAC/ABAC coverage | the design defines group naming conventions (`domain_<domain>_<role>`), RBAC grant patterns per schema, and ABAC policies driven by governed tags (sensitivity, pi_category, soci_critical) |
| S2-F5-AC03 | Entra ID integration is designed | a reviewer checks for identity lifecycle | the design specifies SCIM provisioning from Entra ID, group mapping, service principal strategy, and deprovisioning behaviour |
| S2-F5-AC04 | Encryption and secrets management are designed | a reviewer checks against Essential Eight | the design specifies SSE-KMS with customer-managed keys, TLS 1.2+ for transit, Databricks secret scopes backed by AWS Secrets Manager, and key rotation policy |
| S2-F5-AC05 | Audit logging is designed | a reviewer checks for completeness | the design specifies Databricks system tables for audit, Splunk integration for SIEM, retention period, and alerting rules for security events |
| S2-F5-AC06 | Data retention and purge policies are designed | a reviewer checks regulatory alignment | each data zone has retention periods aligned to State Records Act 2000, SOCI Act 2018, and PRIS Act 2024, with automated purge mechanisms specified |

### Technical Notes
- Unity Catalog structure should align to the access model wiki: domain-based catalogs with federated ownership via MANAGE privilege.
- ABAC design should reference the tagging strategy wiki's 4-layer model (WAICP Classification, Sensitivity Reason, Access & Handling, Data Management).
- Lineage is captured automatically by Unity Catalog for SQL and SDP pipelines; the design should document gaps (e.g., notebook-based transformations) and annotation requirements.
- Break-glass access procedures should be included per the access model wiki Section 10.4.

---

## Feature S2-F6: Metadata Tagging Standards and Catalogue Integration Designed

**Description:** Metadata tagging standards are documented and the Alation–Unity Catalog integration design is agreed – covering tag taxonomy, bidirectional sync, and business glossary alignment – so that data assets are consistently classified and discoverable across both platforms.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S2-F6-US01 | Domain Data Steward | understand the tagging taxonomy and my responsibilities for classifying data assets | I know which tags I need to apply and what values are valid |
| S2-F6-US02 | Data Engineer | see how Delta schema descriptions align with existing conceptual and logical models | I write column descriptions that are consistent with the business glossary |
| S2-F6-US03 | Data Governance Lead | see the Alation–Unity Catalog integration design (bidirectional metadata sync) | I know how business classifications in Alation flow to technical enforcement in Databricks |
| S2-F6-US04 | Data Analyst | understand how to discover data assets using Alation and UC Discover | I can find the data I need without asking the platform team |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S2-F6-AC01 | The tagging taxonomy is designed | the Data Platform Design is reviewed | it defines the 4-layer tag model (WAICP Classification, Sensitivity Reason, Access & Handling, Data Management) with allowed values for each governed tag |
| S2-F6-AC02 | Alation integration is designed | a reviewer checks for bidirectional sync | the design specifies: Alation→UC (business classifications drive ABAC), UC→Alation (metadata harvesting via OCF connector), and conflict resolution rules |
| S2-F6-AC03 | Business glossary alignment is designed | a reviewer checks for coverage | the design specifies how Alation business glossary terms map to Unity Catalog column descriptions and how consistency is maintained |
| S2-F6-AC04 | Discovery experience is designed | a reviewer checks for user journeys | the design documents when users use Alation (enterprise-wide discovery, business context) vs UC Discover (Databricks-native discovery, technical context) |

### Technical Notes
- Tagging standards should align to the tagging strategy wiki's 4-layer model, including AI/ML governance tags and data contract tags.
- Alation OCF connector for Databricks handles UC→Alation metadata harvesting; the reverse sync (Alation→UC tags) requires custom integration.
- The architectural relationship between Alation, Unity Catalog, and UC Discover should be documented per the tagging strategy wiki Section 12.2.
- Column descriptions should reference the business glossary but not duplicate it – Alation is the glossary of record.

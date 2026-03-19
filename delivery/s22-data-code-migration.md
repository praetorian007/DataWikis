# S22 – Data & Code Migration: Feature Breakdown

**Scope Area:** PMO & Support
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `lifecycles/data-engineering-lifecycle.md` (Ingestion, Transformation, Storage stages; DataOps undercurrent)
- `lifecycles/data-science-lifecycle.md` (Deploy stage; MLOps discipline; Mosaic AI Model Serving)
- `lifecycles/data-governance-lifecycle.md` (Stage 9 – AI and Model Governance; Unity Catalog Model Registry)
- `governance/data-governance-roles.md` (Data Platform Owner, Technical Data Steward)

---

## Feature S22-F1: Migration Strategy Document

**Description:** Develop the overarching data and code migration strategy document covering the approach for migrating all legacy platforms (AWS Glue, Azure Data Factory, SAP Data Services, AWS SageMaker) to the Databricks-based EDAP, including tooling assessment, licensing models, and risk mitigation.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F1-US01 | Data Platform Owner | have a documented migration strategy covering all legacy platforms | I can plan and prioritise migration activities with clear scope and sequencing |
| S22-F1-US02 | Solution Architect | understand the tooling landscape and licensing requirements for migration | I can make informed decisions about build-vs-buy and avoid unexpected costs |
| S22-F1-US03 | Project Manager | have a risk assessment and mitigation plan for each migration path | I can proactively manage delivery risks and communicate them to the steering committee |
| S22-F1-US04 | Data Platform Owner | have licensing estimates for any migration tooling required | I can secure budget approval and avoid procurement delays |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F1-AC01 | The migration strategy document is drafted | it is reviewed by stakeholders | it covers all four legacy platforms (AWS Glue, ADF, SAP DS, SageMaker) with a defined approach for each |
| S22-F1-AC02 | Tooling options have been assessed | the strategy is finalised | at least two migration tooling options per platform are evaluated with a recommendation and rationale |
| S22-F1-AC03 | Licensing requirements are identified | the strategy is approved | each tool or service requiring a licence includes the licensing model (perpetual, subscription), estimated cost, and procurement lead time |
| S22-F1-AC04 | Risk assessment is conducted | the strategy is accepted | each migration path has at least three identified risks with likelihood, impact, and mitigation actions documented |
| S22-F1-AC05 | The strategy is submitted for review | Water Corporation evaluates the document | the strategy includes a sequencing and dependency map across all four migration paths |

### Technical Notes
- Align migration approach with the medallion architecture (Bronze/Silver/Gold) and Unity Catalog namespace conventions defined in the Data Engineering Lifecycle wiki.
- Consider Databricks Lakeflow as the primary target orchestration and pipeline framework across all migration paths.
- Licensing assessment should cover Databricks Lakeflow Connect, any third-party migration accelerators, and Mosaic AI components.
- Strategy should reference the DataOps undercurrent (CI/CD, testing, version control) to ensure migrated assets adopt EDAP engineering standards from day one.

---

## Feature S22-F2: AWS Glue Migration Path

**Description:** Define and document the migration path for AWS Glue ETL jobs, PySpark algorithms, and Glue workflows to Databricks Lakeflow Declarative Pipelines and Databricks notebooks/jobs, including code conversion patterns and testing strategies.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F2-US01 | Data Engineer | have a documented mapping from Glue ETL job patterns to Lakeflow Declarative Pipeline equivalents | I can systematically convert existing jobs without redesigning business logic from scratch |
| S22-F2-US02 | Data Engineer | have conversion patterns for PySpark algorithms running in Glue to Databricks notebooks and jobs | I can migrate statistical and transformation logic with confidence that outputs remain consistent |
| S22-F2-US03 | Technical Data Steward | understand how Glue Data Catalog assets map to Unity Catalog | I can ensure metadata continuity and lineage is preserved during migration |
| S22-F2-US04 | DataOps Engineer | have a testing strategy for validating migrated Glue jobs | I can verify functional equivalence between source and target before cutover |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F2-AC01 | An inventory of existing Glue ETL jobs is compiled | the migration path is documented | every Glue job type (ETL, streaming, crawler) has a corresponding Databricks target pattern identified |
| S22-F2-AC02 | PySpark code from Glue is analysed | conversion patterns are defined | at least one worked example per major pattern (batch transform, incremental load, statistical algorithm) is provided |
| S22-F2-AC03 | Glue Data Catalog metadata is reviewed | the Unity Catalog mapping is completed | a table-level mapping document shows how databases, tables, partitions, and classifications translate to Unity Catalog catalogs, schemas, and tags |
| S22-F2-AC04 | A migrated pipeline is tested | validation results are produced | row counts, schema alignment, and a sample of 1,000 records per table are compared between Glue output and Databricks output with zero unresolved discrepancies |
| S22-F2-AC05 | Glue workflow scheduling is reviewed | the Lakeflow Jobs equivalent is documented | job dependencies, retry logic, and alerting configurations are mapped to Lakeflow Jobs equivalents |

### Technical Notes
- Glue crawlers have no direct equivalent in Databricks; recommend using Auto Loader with schema inference for similar discovery behaviour.
- Lakeflow Declarative Pipelines (formerly Delta Live Tables) provide built-in data quality expectations that should replace any custom Glue quality checks.
- Ensure Spark version compatibility is assessed (Glue Spark versions may lag Databricks Runtime).
- Reference the Data Engineering Lifecycle Ingestion and Transformation stages for target-state pipeline design patterns.

---

## Feature S22-F3: Azure Data Factory Migration Path

**Description:** Define and document the migration path for Azure Data Factory pipelines, linked services, and dataflows to Databricks Lakeflow Jobs, Lakeflow Connect, and the Structured Data Processing (SDP) framework.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F3-US01 | Data Engineer | have a mapping from ADF pipeline activities to Lakeflow Jobs task types | I can reconstruct orchestration logic in the target platform |
| S22-F3-US02 | Data Engineer | understand how ADF linked services translate to Lakeflow Connect connectors | I can re-establish source system connectivity without custom integration work |
| S22-F3-US03 | Data Engineer | have a conversion approach for ADF dataflows to SDP or Spark-based transformations | I can migrate transformation logic that currently runs in ADF's mapping dataflows |
| S22-F3-US04 | Data Platform Owner | understand which ADF capabilities have no direct Databricks equivalent | I can plan workarounds or accept scope exclusions with full visibility |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F3-AC01 | An inventory of ADF pipelines is compiled | the migration path is documented | every pipeline activity type in use (Copy, Lookup, ForEach, Web, Stored Procedure, Dataflow) has a documented Databricks equivalent or workaround |
| S22-F3-AC02 | ADF linked services are catalogued | the Lakeflow Connect mapping is completed | each linked service type is mapped to a Lakeflow Connect connector, a JDBC/ODBC connection, or an alternative with any gaps explicitly identified |
| S22-F3-AC03 | ADF dataflows are analysed | the conversion approach is documented | transformation logic categories (joins, aggregations, pivots, derived columns, conditional splits) each have a Spark SQL or PySpark equivalent pattern documented |
| S22-F3-AC04 | ADF trigger and scheduling configurations are reviewed | the Lakeflow Jobs scheduling approach is defined | all trigger types (schedule, tumbling window, event-based) have equivalent Lakeflow Jobs configurations documented |
| S22-F3-AC05 | A representative ADF pipeline is migrated as a proof of concept | validation is performed | the migrated pipeline produces identical output datasets (row count, schema, data content verified on a sample of 1,000 records) compared to the ADF original |

### Technical Notes
- ADF's Copy Activity for bulk data movement maps to Lakeflow Connect for supported sources or Auto Loader / COPY INTO for file-based ingestion.
- ADF dataflows run on a Spark back-end; conversion to native PySpark on Databricks is generally straightforward but requires testing for behavioural differences.
- ADF integration runtimes have no equivalent; connectivity is handled via Databricks networking (VPC peering, PrivateLink, or public endpoints with IP allowlisting).
- Align migrated pipelines with the medallion architecture zones defined in the Data Engineering Lifecycle wiki.

---

## Feature S22-F4: SAP Data Services Migration Path

**Description:** Define and document the migration path for SAP Data Services workflows, transformations, and job scheduling to Databricks Lakeflow, including extraction patterns for SAP source systems and transformation mapping.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F4-US01 | Data Engineer | have a documented approach for migrating SAP DS workflows to Lakeflow pipelines | I can plan the conversion effort with realistic estimates |
| S22-F4-US02 | Data Engineer | understand how SAP DS transformation constructs (queries, scripts, data flows) map to PySpark or Spark SQL | I can convert transformation logic systematically |
| S22-F4-US03 | Data Engineer | have a recommended approach for SAP source system extraction in Databricks | I can replace SAP DS's native SAP connectors with a supported alternative |
| S22-F4-US04 | Technical Data Steward | understand how SAP DS data quality rules and audit trails translate to the Databricks platform | I can ensure governance controls are maintained post-migration |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F4-AC01 | An inventory of SAP DS workflows is compiled | the migration path is documented | every workflow is categorised by complexity (simple, moderate, complex) with estimated conversion effort per category |
| S22-F4-AC02 | SAP DS transformation constructs are analysed | the mapping document is produced | each transformation type (query, script, lookup, merge, case/conditional, hierarchy flattening) has a PySpark or Spark SQL equivalent documented |
| S22-F4-AC03 | SAP source system connectivity is assessed | the extraction approach is defined | a recommended SAP extraction method is documented (e.g., Lakeflow Connect for SAP, CDS views with JDBC, or third-party connector) with licensing and performance considerations |
| S22-F4-AC04 | SAP DS job scheduling is reviewed | the Lakeflow Jobs equivalent is defined | all scheduling patterns (time-based, dependency-based, event-based) are mapped to Lakeflow Jobs with retry and alerting configurations |
| S22-F4-AC05 | SAP DS data quality rules are catalogued | the Databricks equivalent is documented | each data quality rule type has a corresponding Lakeflow Declarative Pipeline expectation or Lakehouse Monitoring check defined |

### Technical Notes
- SAP Data Services has proprietary transformation constructs (e.g., hierarchy flattening, ABAP-based lookups) that require careful analysis; not all will have one-to-one Spark equivalents.
- SAP extraction via Lakeflow Connect for SAP (if available) or CDS views with JDBC is the preferred approach; assess whether SAP BW or S/4HANA direct extraction is required.
- SAP DS audit and error-handling patterns should be mapped to Lakeflow Declarative Pipeline expectations and Databricks alerting.
- Reference the Data Engineering Lifecycle Transformation stage for target-state transformation patterns.

---

## Feature S22-F5: AWS SageMaker Migration Path

**Description:** Define and document the migration path for AWS SageMaker machine learning models to Mosaic AI Model Serving, training pipelines to Databricks ML Runtime, and model registry assets to Unity Catalog, with specific reference to models such as the Hardship Propensity XGBoost model.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F5-US01 | Data Scientist | have a documented approach for migrating SageMaker training pipelines to Databricks ML Runtime | I can retrain existing models on the new platform without rewriting core algorithm logic |
| S22-F5-US02 | Data Scientist | understand how SageMaker model registry artefacts map to Unity Catalog Model Registry | I can preserve model versioning, lineage, and metadata during migration |
| S22-F5-US03 | ML Engineer | have a migration plan for SageMaker endpoints to Mosaic AI Model Serving | I can ensure inference continuity with minimal downtime during cutover |
| S22-F5-US04 | AI/ML Governance Lead | have a plan for preserving model governance metadata (model cards, approval history, risk classification) during migration | I can maintain governance compliance throughout the transition |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F5-AC01 | An inventory of SageMaker models is compiled | the migration path is documented | every model is catalogued with its framework (XGBoost, TensorFlow, PyTorch, sklearn), training data source, inference pattern (batch/real-time), and current SLA |
| S22-F5-AC02 | SageMaker training pipelines are analysed | the Databricks ML Runtime migration approach is defined | a step-by-step conversion guide is provided covering compute configuration, library dependencies, feature engineering, hyperparameter tuning, and experiment tracking (MLflow) |
| S22-F5-AC03 | SageMaker model registry entries are reviewed | the Unity Catalog Model Registry migration plan is defined | each model's versions, stages, metadata, and lineage are mapped to Unity Catalog equivalents with a migration script or procedure documented |
| S22-F5-AC04 | SageMaker endpoints are catalogued | the Mosaic AI Model Serving migration plan is defined | each endpoint has a target serving configuration (real-time, batch, or streaming) with expected latency, throughput, and autoscaling parameters documented |
| S22-F5-AC05 | The Hardship Propensity XGBoost model is selected as a pilot | migration validation is completed | the migrated model produces prediction outputs within 1% tolerance of the SageMaker baseline on an identical test dataset, and inference latency meets or exceeds the existing SLA |

### Technical Notes
- SageMaker uses its own model registry; Unity Catalog Model Registry provides equivalent versioning, staging, and lineage but requires re-registration of model artefacts.
- XGBoost models are framework-portable; the primary migration effort is in the training pipeline, feature engineering, and serving infrastructure rather than the model itself.
- Align with the Data Science Lifecycle Deploy stage and MLOps discipline for target-state model deployment patterns.
- Ensure model cards and datasheets are created or migrated per the AI and Model Governance stage (Stage 9) of the Data Governance Lifecycle.
- Reference the AI/ML Governance Lead role for governance oversight of migrated models.

---

## Feature S22-F6: Data Migration Execution for Business Use Cases

**Description:** Execute the actual data and code migration for the S17 business use cases, including pipeline conversion, data validation, parallel running, and production cutover with rollback planning.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F6-US01 | Data Engineer | execute the migration of S17 business use case pipelines to the EDAP | the use cases run on the target platform and can be validated against legacy outputs |
| S22-F6-US02 | Data Product Owner | have a validation report confirming data accuracy post-migration | I can certify the migrated data product for business use |
| S22-F6-US03 | Data Platform Owner | have a cutover plan with rollback procedures | I can approve the production switch with confidence that rollback is possible if issues arise |
| S22-F6-US04 | Data Consumer | experience no disruption to data availability during the migration cutover | I can continue my analytical work without interruption |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F6-AC01 | S17 business use case pipelines are identified | migration execution begins | every pipeline has a documented conversion plan referencing the applicable migration path (F2, F3, F4, or F5) |
| S22-F6-AC02 | Migrated pipelines are deployed to the test environment | data validation is executed | row counts match within 0.1% tolerance, schema validation passes with zero errors, and a record-level comparison of 10,000 sampled records per table shows zero unresolved discrepancies |
| S22-F6-AC03 | Parallel running is initiated | both legacy and EDAP pipelines run concurrently for a minimum of two sprint cycles | output comparison reports are generated daily and reviewed, with all discrepancies logged, triaged, and resolved or accepted |
| S22-F6-AC04 | The cutover plan is drafted | it is reviewed by Water Corporation | the plan includes a go/no-go checklist, rollback procedure with a defined rollback window (minimum 48 hours), communication plan, and named responsible parties |
| S22-F6-AC05 | Production cutover is executed | post-cutover validation is completed within 24 hours | all migrated data products are accessible via Unity Catalog, downstream consumers confirm data availability, and Lakehouse Monitoring checks pass |

### Technical Notes
- Migration execution is scoped to S17 business use cases only; broader migration is addressed by the strategy document (F1).
- Parallel running in test and production environments must account for non-prod cost constraints (refer to S10 – non-prod costs not exceeding 10% of production).
- Cutover should follow the DataOps promotion strategy (Dev to Test to Prod) defined in S7.
- Post-migration, all data products must be registered in Unity Catalog and harvested by Alation (per S14/S15 integration requirements).
- Validation approach should use the data quality dimensions (freshness, accuracy, completeness, uniqueness) from the Data Engineering Lifecycle.

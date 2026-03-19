# S22 – Data & Code Migration: Feature Breakdown

**Scope Area:** PMO & Support
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `lifecycles/data-engineering-lifecycle.md` (Ingestion, Transformation, Storage stages; DataOps undercurrent)
- `lifecycles/data-science-lifecycle.md` (Deploy stage; MLOps discipline; Mosaic AI Model Serving)
- `lifecycles/data-governance-lifecycle.md` (Stage 9 – AI and Model Governance; Unity Catalog Model Registry)
- `governance/data-governance-roles.md` (Data Platform Owner, Technical Data Steward)

---

## Feature S22-F1: Clear, Costed Migration Path Documented for Every Legacy Platform

**Description:** Any stakeholder can open a single strategy document and immediately understand how each legacy platform (AWS Glue, Azure Data Factory, SAP Data Services, AWS SageMaker) will reach the Databricks-based EDAP — what it will cost, which tools are recommended, what risks to plan for, and in what order — so that funding, procurement, and sequencing decisions are made with confidence rather than assumptions.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F1-US01 | Data Platform Owner | see, in one document, the migration approach for every legacy platform with a clear sequence and dependencies | I can make prioritisation decisions and present a credible plan to the steering committee |
| S22-F1-US02 | Solution Architect | compare at least two evaluated tooling options per platform with licensing models and costs | I can recommend the best-fit option without hidden procurement surprises |
| S22-F1-US03 | Project Manager | review a risk register with likelihood, impact, and mitigations for each migration path | I can proactively manage delivery risks and set realistic stakeholder expectations |
| S22-F1-US04 | Finance Business Partner | see licensing estimates broken down by model (perpetual vs subscription), cost, and procurement lead time | I can approve budget and avoid delays caused by unplanned procurement cycles |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F1-AC01 | A stakeholder opens the migration strategy | they look for their platform | all four legacy platforms (AWS Glue, ADF, SAP DS, SageMaker) have a defined migration approach, and none is missing |
| S22-F1-AC02 | A decision-maker reviews tooling options | they need to choose a migration tool for a given platform | at least two options per platform are evaluated with a recommendation and clear rationale |
| S22-F1-AC03 | A procurement officer checks licensing requirements | they need to begin purchasing | every tool or service requiring a licence shows the licensing model, estimated cost, and procurement lead time |
| S22-F1-AC04 | A project manager reviews migration risks | they need to update the risk register | each migration path has at least three identified risks with likelihood, impact, and mitigation actions |
| S22-F1-AC05 | A delivery lead plans the migration sequence | they look for dependencies between paths | a sequencing and dependency map across all four migration paths is included |

### Technical Notes
- Align migration approach with the medallion architecture (Bronze/Silver/Gold) and Unity Catalog namespace conventions defined in the Data Engineering Lifecycle wiki.
- Consider Databricks Lakeflow as the primary target orchestration and pipeline framework across all migration paths.
- Licensing assessment should cover Databricks Lakeflow Connect, any third-party migration accelerators, and Mosaic AI components.
- Strategy should reference the DataOps undercurrent (CI/CD, testing, version control) to ensure migrated assets adopt EDAP engineering standards from day one.

---

## Feature S22-F2: AWS Glue Workloads Have a Proven Migration Path to Databricks

**Description:** A data engineer can take any existing AWS Glue ETL job, PySpark algorithm, or Glue workflow and follow a documented, tested conversion path to produce an equivalent Databricks pipeline — with worked examples, a metadata mapping to Unity Catalog, and a validation approach that proves output consistency before cutover — so that migration proceeds systematically rather than through ad-hoc rework.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F2-US01 | Data Engineer | follow a documented mapping from each Glue ETL job pattern to its Lakeflow Declarative Pipeline equivalent | I can convert existing jobs systematically without redesigning business logic from scratch |
| S22-F2-US02 | Data Engineer | reference worked conversion examples for PySpark algorithms (batch transform, incremental load, statistical algorithm) | I can migrate transformation logic confidently, knowing outputs will remain consistent |
| S22-F2-US03 | Technical Data Steward | see exactly how Glue Data Catalog databases, tables, partitions, and classifications translate to Unity Catalog | I can verify that metadata continuity and lineage are preserved during migration |
| S22-F2-US04 | DataOps Engineer | run a validation suite that compares Glue outputs to Databricks outputs at row-count, schema, and record level | I can prove functional equivalence before approving cutover |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F2-AC01 | A data engineer needs to migrate a Glue job | they consult the migration path document | every Glue job type in use (ETL, streaming, crawler) has a corresponding Databricks target pattern identified |
| S22-F2-AC02 | A data engineer needs to convert PySpark code | they look for conversion guidance | at least one worked example per major pattern (batch transform, incremental load, statistical algorithm) is provided and runnable |
| S22-F2-AC03 | A technical data steward checks metadata continuity | they review the Unity Catalog mapping | a table-level mapping shows how databases, tables, partitions, and classifications translate to Unity Catalog catalogs, schemas, and tags |
| S22-F2-AC04 | A migrated pipeline is deployed and tested | validation results are reviewed | row counts, schema alignment, and a sample of 1,000 records per table are compared between Glue output and Databricks output with zero unresolved discrepancies |
| S22-F2-AC05 | A data engineer checks scheduling equivalence | they review the Lakeflow Jobs mapping | job dependencies, retry logic, and alerting configurations from Glue workflows are mapped to Lakeflow Jobs equivalents |

### Technical Notes
- Glue crawlers have no direct equivalent in Databricks; recommend using Auto Loader with schema inference for similar discovery behaviour.
- Lakeflow Declarative Pipelines (formerly Delta Live Tables) provide built-in data quality expectations that should replace any custom Glue quality checks.
- Ensure Spark version compatibility is assessed (Glue Spark versions may lag Databricks Runtime).
- Reference the Data Engineering Lifecycle Ingestion and Transformation stages for target-state pipeline design patterns.

---

## Feature S22-F3: ADF Pipelines Have a Proven Migration Path to Databricks

**Description:** A data engineer can take any existing ADF pipeline, linked service, or dataflow and follow a documented, tested conversion path to produce an equivalent Databricks workflow — with activity-level mappings to Lakeflow Jobs, connector mappings to Lakeflow Connect, and a proof-of-concept demonstrating identical output — so that migration is predictable and no pipeline is left without a clear path forward.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F3-US01 | Data Engineer | look up any ADF pipeline activity type and find its Lakeflow Jobs equivalent or workaround | I can reconstruct orchestration logic in the target platform without guesswork |
| S22-F3-US02 | Data Engineer | see how each ADF linked service translates to a Lakeflow Connect connector, JDBC/ODBC connection, or documented alternative | I can re-establish source system connectivity without custom integration work |
| S22-F3-US03 | Data Engineer | follow documented Spark SQL or PySpark equivalents for every ADF dataflow transformation category | I can migrate transformation logic that currently runs in ADF's mapping dataflows |
| S22-F3-US04 | Data Platform Owner | see explicitly which ADF capabilities have no direct Databricks equivalent and what the workaround is | I can accept scope exclusions or plan workarounds with full visibility |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F3-AC01 | A data engineer encounters an ADF activity type during migration | they consult the migration path | every pipeline activity type in use (Copy, Lookup, ForEach, Web, Stored Procedure, Dataflow) has a documented Databricks equivalent or workaround |
| S22-F3-AC02 | A data engineer needs to reconnect a source system | they check the Lakeflow Connect mapping | each linked service type is mapped to a Lakeflow Connect connector, a JDBC/ODBC connection, or an alternative, with any gaps explicitly identified |
| S22-F3-AC03 | A data engineer converts a dataflow transformation | they reference the conversion patterns | transformation logic categories (joins, aggregations, pivots, derived columns, conditional splits) each have a Spark SQL or PySpark equivalent pattern documented |
| S22-F3-AC04 | A data engineer migrates trigger and scheduling configurations | they review the scheduling mapping | all trigger types (schedule, tumbling window, event-based) have equivalent Lakeflow Jobs configurations documented |
| S22-F3-AC05 | A representative ADF pipeline has been migrated as a proof of concept | validation is performed | the migrated pipeline produces identical output datasets (row count, schema, data content verified on a sample of 1,000 records) compared to the ADF original |

### Technical Notes
- ADF's Copy Activity for bulk data movement maps to Lakeflow Connect for supported sources or Auto Loader / COPY INTO for file-based ingestion.
- ADF dataflows run on a Spark back-end; conversion to native PySpark on Databricks is generally straightforward but requires testing for behavioural differences.
- ADF integration runtimes have no equivalent; connectivity is handled via Databricks networking (VPC peering, PrivateLink, or public endpoints with IP allowlisting).
- Align migrated pipelines with the medallion architecture zones defined in the Data Engineering Lifecycle wiki.

---

## Feature S22-F4: SAP Data Services Workflows Have a Proven Migration Path

**Description:** A data engineer can take any existing SAP Data Services workflow — including its transformations, SAP source extractions, scheduling, and data quality rules — and follow a documented conversion path to produce an equivalent Databricks pipeline, with complexity-based effort estimates, transformation mappings, and governance controls preserved — so that SAP DS retirement can proceed on a credible timeline with no loss of business logic or quality controls.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F4-US01 | Data Engineer | see every SAP DS workflow categorised by complexity with a realistic conversion effort estimate | I can plan the migration with credible timelines and resource requests |
| S22-F4-US02 | Data Engineer | look up any SAP DS transformation construct and find its PySpark or Spark SQL equivalent | I can convert transformation logic systematically without reinventing business rules |
| S22-F4-US03 | Data Engineer | follow a recommended approach for SAP source system extraction in Databricks (Lakeflow Connect for SAP, CDS views with JDBC, or third-party connector) | I can replace SAP DS's native connectors with a supported, licensed alternative |
| S22-F4-US04 | Technical Data Steward | see how SAP DS data quality rules and audit trails translate to Lakeflow Declarative Pipeline expectations and Lakehouse Monitoring checks | I can verify that governance controls are maintained post-migration |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F4-AC01 | A delivery lead needs to estimate migration effort | they review the workflow inventory | every SAP DS workflow is categorised by complexity (simple, moderate, complex) with estimated conversion effort per category |
| S22-F4-AC02 | A data engineer needs to convert a transformation | they consult the mapping document | each transformation type (query, script, lookup, merge, case/conditional, hierarchy flattening) has a PySpark or Spark SQL equivalent documented |
| S22-F4-AC03 | A data engineer needs to extract from an SAP source system | they check the extraction approach | a recommended SAP extraction method is documented (e.g., Lakeflow Connect for SAP, CDS views with JDBC, or third-party connector) with licensing and performance considerations |
| S22-F4-AC04 | A data engineer migrates scheduling configurations | they review the Lakeflow Jobs mapping | all scheduling patterns (time-based, dependency-based, event-based) are mapped to Lakeflow Jobs with retry and alerting configurations |
| S22-F4-AC05 | A technical data steward audits governance controls post-migration | they check the data quality mapping | each SAP DS data quality rule type has a corresponding Lakeflow Declarative Pipeline expectation or Lakehouse Monitoring check defined |

### Technical Notes
- SAP Data Services has proprietary transformation constructs (e.g., hierarchy flattening, ABAP-based lookups) that require careful analysis; not all will have one-to-one Spark equivalents.
- SAP extraction via Lakeflow Connect for SAP (if available) or CDS views with JDBC is the preferred approach; assess whether SAP BW or S/4HANA direct extraction is required.
- SAP DS audit and error-handling patterns should be mapped to Lakeflow Declarative Pipeline expectations and Databricks alerting.
- Reference the Data Engineering Lifecycle Transformation stage for target-state transformation patterns.

---

## Feature S22-F5: SageMaker Models Have a Proven Migration Path to Mosaic AI

**Description:** A data scientist or ML engineer can take any existing SageMaker model — including its training pipeline, model registry entry, and serving endpoint — and follow a documented, tested migration path to Databricks, preserving model versioning, governance metadata, and inference SLAs, with the Hardship Propensity XGBoost model as a validated pilot — so that model migration is repeatable and no model loses its governance trail or production reliability in transit.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F5-US01 | Data Scientist | follow a step-by-step guide to retrain an existing SageMaker model on Databricks ML Runtime | I can retrain models on the new platform without rewriting core algorithm logic |
| S22-F5-US02 | Data Scientist | see how SageMaker model registry artefacts (versions, stages, metadata, lineage) map to Unity Catalog Model Registry | I can preserve model versioning and lineage during migration |
| S22-F5-US03 | ML Engineer | follow a migration plan that moves SageMaker endpoints to Mosaic AI Model Serving with defined latency and throughput targets | I can ensure inference continuity with minimal downtime during cutover |
| S22-F5-US04 | AI/ML Governance Lead | verify that model governance metadata (model cards, approval history, risk classification) is preserved or recreated during migration | I can maintain governance compliance throughout the transition |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F5-AC01 | An ML engineer needs to plan a model migration | they review the model inventory | every SageMaker model is catalogued with its framework (XGBoost, TensorFlow, PyTorch, sklearn), training data source, inference pattern (batch/real-time), and current SLA |
| S22-F5-AC02 | A data scientist needs to migrate a training pipeline | they follow the conversion guide | the guide covers compute configuration, library dependencies, feature engineering, hyperparameter tuning, and experiment tracking (MLflow) step by step |
| S22-F5-AC03 | A data scientist needs to migrate model registry entries | they follow the migration procedure | each model's versions, stages, metadata, and lineage are mapped to Unity Catalog equivalents with a migration script or procedure documented |
| S22-F5-AC04 | An ML engineer needs to migrate a serving endpoint | they follow the serving migration plan | each endpoint has a target serving configuration (real-time, batch, or streaming) with expected latency, throughput, and autoscaling parameters documented |
| S22-F5-AC05 | The Hardship Propensity XGBoost model has been migrated as a pilot | validation is completed | the migrated model produces prediction outputs within 1% tolerance of the SageMaker baseline on an identical test dataset, and inference latency meets or exceeds the existing SLA |

### Technical Notes
- SageMaker uses its own model registry; Unity Catalog Model Registry provides equivalent versioning, staging, and lineage but requires re-registration of model artefacts.
- XGBoost models are framework-portable; the primary migration effort is in the training pipeline, feature engineering, and serving infrastructure rather than the model itself.
- Align with the Data Science Lifecycle Deploy stage and MLOps discipline for target-state model deployment patterns.
- Ensure model cards and datasheets are created or migrated per the AI and Model Governance stage (Stage 9) of the Data Governance Lifecycle.
- Reference the AI/ML Governance Lead role for governance oversight of migrated models.

---

## Feature S22-F6: Business Use Case Data Migrated, Validated, and Serving from EDAP

**Description:** S17 business use case data products are migrated from legacy platforms to the EDAP, validated to prove accuracy, run in parallel for two sprint cycles, and cut over to production — so that data consumers experience no disruption, the migrated products are certified for business use, and legacy platforms can be decommissioned with confidence.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S22-F6-US01 | Data Engineer | execute the pipeline migration using the proven paths from F2–F5 and see each pipeline running on the EDAP | the use cases are operational on the target platform and ready for validation |
| S22-F6-US02 | Data Product Owner | review a validation report confirming row counts, schema alignment, and record-level accuracy | I can certify the migrated data product for business use with evidence, not assumptions |
| S22-F6-US03 | Data Platform Owner | approve a cutover plan that includes a go/no-go checklist, rollback procedure, and named responsible parties | I can authorise the production switch knowing rollback is possible within 48 hours if issues arise |
| S22-F6-US04 | Data Consumer | continue accessing my data and reports without interruption during the migration cutover | my analytical work is not disrupted by the platform change |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S22-F6-AC01 | A data engineer begins migrating an S17 pipeline | they check the conversion plan | every pipeline has a documented conversion plan referencing the applicable migration path (F2, F3, F4, or F5) |
| S22-F6-AC02 | Migrated pipelines are deployed to the test environment | data validation is executed | row counts match within 0.1% tolerance, schema validation passes with zero errors, and a record-level comparison of 10,000 sampled records per table shows zero unresolved discrepancies |
| S22-F6-AC03 | Parallel running is initiated | both legacy and EDAP pipelines have run concurrently for a minimum of two sprint cycles | output comparison reports are generated daily and reviewed, with all discrepancies logged, triaged, and resolved or accepted |
| S22-F6-AC04 | The cutover plan is submitted for review | Water Corporation evaluates it | the plan includes a go/no-go checklist, rollback procedure with a defined rollback window (minimum 48 hours), communication plan, and named responsible parties |
| S22-F6-AC05 | Production cutover is executed | post-cutover validation is completed within 24 hours | all migrated data products are accessible via Unity Catalog, downstream consumers confirm data availability, and Lakehouse Monitoring checks pass |

### Technical Notes
- Migration execution is scoped to S17 business use cases only; broader migration is addressed by the strategy document (F1).
- Parallel running in test and production environments must account for non-prod cost constraints (refer to S10 – non-prod costs not exceeding 10% of production).
- Cutover should follow the DataOps promotion strategy (Dev to Test to Prod) defined in S7.
- Post-migration, all data products must be registered in Unity Catalog and harvested by Alation (per S14/S15 integration requirements).
- Validation approach should use the data quality dimensions (freshness, accuracy, completeness, uniqueness) from the Data Engineering Lifecycle.

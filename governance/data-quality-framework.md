# Data Quality Framework

**Mark Shaw** | Principal Data Architect

---

## 1. Purpose

Water Corporation is a critical infrastructure utility responsible for delivering water, wastewater, and drainage services across Western Australia. The decisions that keep this service running — asset maintenance prioritisation, network capacity planning, regulatory compliance reporting, customer billing, and public safety management — all depend on data. When data quality fails, the consequences are not abstract: incorrect asset condition data delays maintenance on ageing infrastructure; incomplete customer records cause billing errors; stale SCADA telemetry obscures operational risk; and inaccurate compliance datasets create regulatory exposure under the Security of Critical Infrastructure Act 2018 (SOCI Act) and the Personal Information (Relevant Regulators) Act 2024 (PRIS Act).

A Data Quality Framework provides the structure, standards, and mechanisms to ensure that data across the Enterprise Data & Analytics Platform (EDAP) is systematically measured, monitored, and improved. It defines what quality means, how it is assessed, where it is enforced, who is accountable, and how issues are detected and resolved.

Without a framework, quality management devolves into reactive firefighting — ad hoc fixes applied when dashboards break or reports are challenged. With a framework, quality becomes a continuous, embedded discipline: proactive, measurable, and governed.

This document establishes that framework for EDAP. It is designed to be practical, not aspirational — grounded in the platform's architecture, aligned with existing governance roles, and integrated with the pipeline framework that processes data every day.

---

## 2. What Is Data Quality?

Data quality is **fitness for purpose** — the degree to which data is suitable for the decisions and processes it supports. This is a critical distinction: quality is not an absolute property of data itself. The same dataset may be perfectly adequate for one use case and wholly inadequate for another. A customer address with suburb-level granularity may be sufficient for regional demand forecasting but insufficient for field crew dispatch. An asset condition score updated weekly may be adequate for long-term capital planning but unacceptable for real-time operational decisions.

This means quality cannot be assessed in isolation from use. It must be defined in the context of consumers and their requirements — which is why quality rules are defined per data product, by the people who understand the business context, and encoded in data contracts that make quality expectations explicit.

The EDAP Data Quality Framework draws on two foundational standards:

- **DAMA-DMBOK2** (Data Management Body of Knowledge, 2nd edition) — the industry reference for data quality management, defining quality dimensions, assessment methods, and organisational structures for quality governance.
- **ISO 8000** (Data Quality) — the international standard for data quality management, providing a formal vocabulary and process framework for measuring and improving data quality.

Both standards reinforce the same core principle: quality is a managed discipline, not a byproduct of good intentions. It requires defined dimensions, measurable rules, assigned accountability, continuous monitoring, and a culture of improvement.

---

## 3. Quality Dimensions

Quality is not a single attribute. It is decomposed into dimensions — each describing a different aspect of what makes data trustworthy. The following dimensions form the standard quality vocabulary for EDAP. Every quality rule maps to one of these dimensions; every quality dashboard reports against them.

| Dimension | Definition | How Measured | Water Corporation Example |
|---|---|---|---|
| **Completeness** | Required fields are populated; no unexpected gaps in the data. | Percentage of non-null values for mandatory columns; record count vs expected count. | Customer address fields populated for all active billing accounts. |
| **Uniqueness** | No duplicate records exist for the same business entity. | Primary key uniqueness assertion; duplicate detection on business keys. | One record per asset in `dim_asset`; no duplicate work orders in `fact_work_order`. |
| **Accuracy** | Values conform to business rules and faithfully represent real-world truth. | Rule-based validation; cross-reference checks against authoritative sources. | Asset GPS coordinates fall within Water Corporation's service area boundaries. |
| **Consistency** | The same data is represented the same way across systems and contexts. | Cross-system reconciliation; format standardisation checks. | Customer ID format is consistent between SAP, CRM, and the billing system. |
| **Timeliness** | Data is available when needed, within the service level agreement (SLA) for freshness. | Time elapsed since last refresh compared to the defined SLA. | SCADA telemetry available in EDAP within 15 minutes of capture. |
| **Validity** | Values conform to defined formats, schemas, and allowed ranges. | Regex pattern matching; range checks; enumeration validation. | Work order status is in the allowed set (`open`, `in_progress`, `completed`, `cancelled`). |
| **Referential Integrity** | Foreign key relationships hold; related records exist where expected. | Foreign key lookup validation; orphan record detection. | Every work order references a valid asset ID that exists in the asset dimension. |

These dimensions are not equally important for every dataset. A SCADA telemetry table may prioritise timeliness above all else; a financial reporting dimension may prioritise accuracy and completeness. The relative weighting of dimensions is defined per data product in its data contract.

---

## 4. The EDAP Quality Architecture

### 4.1 The "Annotate, Never Filter" Philosophy

The EDAP's foundational quality principle is **"Annotate, Never Filter"** (also expressed as "Propagate, Don't Drop" in the Medallion Architecture document). This means:

- Records that fail business data quality rules are **never silently removed** from the pipeline.
- Instead, every record is annotated with its quality assessment via embedded DQ columns, and **all records propagate** to the target table regardless of their quality status.
- Consumers decide their own quality tolerance. A data scientist exploring patterns may want all records including those flagged as `FAIL`. A BI dashboard serving executive KPIs may filter to `PASS` and `WARN` only. The data is the same; the quality context travels with it.

This approach delivers three critical benefits:

1. **No invisible data loss.** Dropping records silently creates false trust — consumers believe their data is complete when it is not. Annotating preserves completeness and makes quality problems visible.
2. **Lineage integrity.** Co-locating quality annotations with the data they describe means lineage is preserved end-to-end. There are no orphaned records in separate quarantine tables that lose their connection to the pipeline.
3. **Consumer autonomy.** Different consumers have different quality tolerances. Embedding quality status in the data allows each consumer to apply the threshold appropriate to their use case.

**Important distinction:** The "Annotate, Never Filter" principle applies to **business data quality issues** — invalid codes, unexpected nulls, referential integrity violations, out-of-range values. **Structural and technical failures** (unparseable records, binary corruption, missing mandatory system fields) are handled differently: these are routed to the quarantine path during the Raw-to-Base pipeline transition because they cannot be processed at all. This is the only point where records are removed from the standard pipeline flow.

### 4.2 DQ Annotation Columns

Every Silver (Base and Enriched) and Gold table in EDAP includes four DQ annotation columns. These columns are introduced at the Base Zone and carried forward through all downstream zones.

| Column | Type | Description |
|---|---|---|
| `dq_status` | `STRING` | Overall quality verdict for this record: `PASS`, `WARN`, or `FAIL`. |
| `dq_errors` | `ARRAY<STRING>` | List of DQ rule names that failed with error severity. Empty array if no errors. |
| `dq_warnings` | `ARRAY<STRING>` | List of DQ rule names that triggered with warning severity. Empty array if no warnings. |
| `dq_checked_ts` | `TIMESTAMP` | Timestamp of when the DQ evaluation was performed for this record. |

**Status derivation logic:**

| Condition | `dq_status` |
|---|---|
| No errors AND no warnings | `PASS` |
| No errors AND one or more warnings | `WARN` |
| One or more errors (regardless of warnings) | `FAIL` |

**Rules:**

- All records propagate to the target table regardless of `dq_status`. No records are filtered or quarantined by default.
- `dq_errors` and `dq_warnings` contain the **names** of the rules that triggered, not free-text messages. This enables programmatic querying (e.g. `array_contains(dq_errors, 'pk_unique')`).
- `dq_checked_ts` records when the evaluation occurred, not when the source data was created. This distinguishes "we checked this record yesterday and it passed" from "we checked it today and it failed."
- DQ columns are carried forward through Silver Enriched and Gold zones. If additional quality rules are evaluated at downstream zones, the DQ columns are updated to reflect the cumulative assessment.

### 4.3 Lakeflow Expectations (Pipeline-Level)

Lakeflow Spark Declarative Pipelines (SDP) provide a complementary quality mechanism: **expectations**. Expectations operate at the pipeline level rather than per-record — they evaluate conditions across the dataset and record aggregate pass/fail counts in the pipeline event log (`system.pipelines.event_log`).

Expectations do **not** add columns to the target table. They provide batch-level visibility, not row-level annotation. Both mechanisms are needed — they are complementary, not alternatives.

**Expectation types:**

| Type | Behaviour | When to Use |
|---|---|---|
| `EXPECT` | Log violations in the event log; retain all rows. | **EDAP default.** Use for most quality rules. Aligns with the "Annotate, Never Filter" principle. Provides batch-level monitoring without affecting data flow. |
| `EXPECT OR DROP` | Drop rows that violate the expectation; log the drop count. | Use sparingly. Appropriate only when there is a documented business decision to exclude specific records (e.g. test records with a known sentinel value). |
| `EXPECT OR FAIL` | Fail the entire pipeline if the expectation is violated. | Use as a **circuit breaker** for critical invariants that, if violated, indicate a systemic problem: primary key uniqueness failures, schema integrity violations, zero-row output when rows are expected. |

**EDAP guidance:**

- Default to `EXPECT` for business quality rules. This records violations for monitoring and alerting without altering data flow.
- Use `EXPECT OR FAIL` for invariants where a violation means something is fundamentally wrong with the pipeline or source data — not merely that a business rule was broken.
- Use `EXPECT OR DROP` only when a Data Product Owner has explicitly documented the exclusion requirement in the data product's data contract. Every dropped record must be accounted for.

### 4.4 Three Complementary DQ Observability Layers

The EDAP provides quality observability at three granularities. All three are needed — they serve different audiences and different questions.

| Layer | Mechanism | What It Tells You | Audience |
|---|---|---|---|
| **Row-level** | Physical `dq_status`, `dq_errors`, `dq_warnings`, `dq_checked_ts` columns | Which specific rules each individual record violated. | Data consumers filtering by quality; Data Domain Stewards investigating specific records. |
| **Pipeline/batch-level** | Lakeflow SDP `EXPECT` expectations via `system.pipelines.event_log` | How many records failed each rule in each pipeline run. | Data Engineers monitoring pipeline health; Technical Data Stewards tracking rule pass rates. |
| **Table/trend-level** | Databricks Lakehouse Monitoring (Data Profiling) | Column statistics over time — null rates, value distributions, drift, freshness anomalies. | Data Domain Stewards monitoring quality trends; Data Product Owners tracking SLA compliance. |

### 4.5 The 1:10:100 Rule

The cost of addressing data quality issues increases exponentially the later they are discovered in the data lifecycle:

| Stage | Relative Cost | Example |
|---|---|---|
| **Prevention at source** (1x) | 1 | Validating data entry at the point of capture in SAP. Correct once, correct everywhere. |
| **Detection at ingestion** (10x) | 10 | Catching an invalid asset status code in the Base Zone pipeline. Requires annotation, investigation, and source system remediation. |
| **Remediation in Gold** (100x) | 100 | Discovering incorrect customer billing data after it has been aggregated into KPIs, published to dashboards, and used to generate invoices. Requires root cause analysis, data correction, report republication, and potentially customer communication. |

This principle drives EDAP's approach: embed quality checks as early as possible, push quality accountability upstream to source system owners, and invest in prevention over remediation.

---

## 5. Quality by Medallion Zone

Quality is not a single gate — it is a progressive concern embedded at every layer of the medallion architecture. Each zone has a distinct quality focus, uses specific mechanisms, and evaluates different types of rules.

| Zone | Layer | Quality Focus | Key Mechanisms | Example Rules |
|---|---|---|---|---|
| **Landing** | Bronze | File integrity and format validation | Pre-ingestion checks; file format validation; size, encoding, and naming convention checks | File is valid CSV/JSON/Parquet; file matches expected naming convention; file is not empty; file size within expected range. |
| **Raw** | Bronze | Capture fidelity — all source data preserved without loss or truncation | Auto Loader schema inference; rescued data column (`_rescued_data`); VARIANT columns for schema resilience | All source columns captured; no silent truncation of values; rescued data column is empty (no parsing failures). |
| **Base** | Silver | Business rule validation — the primary DQ enforcement point | DQ annotation columns (`dq_status`, `dq_errors`, `dq_warnings`, `dq_checked_ts`); Lakeflow expectations; deduplication; type casting and validation | Primary key uniqueness; not-null on mandatory fields; type validity; allowed value checks; range checks; referential integrity. |
| **Enriched** | Silver | Integration quality — ensuring joins and enrichments are sound | Join validation; row count reconciliation; distribution monitoring; DQ columns carried forward and updated | No unmatched foreign key joins beyond threshold; row counts within expected range of source; no unexpected fan-out from joins. |
| **BI** | Gold | KPI accuracy and data product readiness | Business rule assertions; metric lineage validation; anomaly detection; data contract compliance checks | Aggregations reconcile to source totals; KPI values within expected bounds; freshness SLA met; completeness thresholds satisfied. |
| **Exploratory** | Gold | Transparency — quality context visible to consumers | DQ columns carried forward; quality scores visible in metadata | Consumers can see the `dq_status` of underlying records and make informed decisions about data fitness. |

**Key principle:** DQ annotation columns are introduced at the **Base Zone** and carried forward through every downstream zone. They are never stripped. Gold-layer consumers always have access to the quality assessment of the records they are consuming.

---

## 6. Quality Rules — Definition and Configuration

### 6.1 Rule Structure

DQ rules are defined per entity in the EDAP pipeline framework metadata configuration. Each rule specifies what to check, where to check it, and how severe a violation is.

Every rule has the following attributes:

| Attribute | Description |
|---|---|
| `name` | Unique rule name within the entity. Used in `dq_errors` and `dq_warnings` arrays. Must be descriptive (e.g. `pk_unique`, `status_valid`, `install_date_reasonable`). |
| `type` | The rule type: `not_null`, `uniqueness`, `range`, `allowed_values`, `regex`, `sql_expression`, `referential_integrity`, `cross_column`, `data_type`. |
| `severity` | `error` or `warning`. Errors contribute to `FAIL` status; warnings contribute to `WARN` status. |
| `column` / `columns` | The column(s) the rule applies to. Some types (e.g. `uniqueness`, `cross_column`) operate on multiple columns. |
| `parameters` | Type-specific parameters (e.g. `values` for `allowed_values`, `min`/`max` for `range`, `pattern` for `regex`, `expression` for `sql_expression`). |

### 6.2 Supported Rule Types

| Rule Type | Description | Example Use |
|---|---|---|
| `not_null` | Column must not be null. | `asset_id` must be populated on every record. |
| `uniqueness` | Column(s) must be unique across the dataset (primary key check). | `asset_id` is unique in the asset base table. |
| `range` | Column value must fall within a defined range (inclusive). | `install_date` between `1900-01-01` and `current_date()`. |
| `allowed_values` | Column value must be in a defined set of allowed values. | `status` in (`active`, `inactive`, `decommissioned`). |
| `regex` | Column value must match a regular expression pattern. | `email` matches `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`. |
| `sql_expression` | A SQL expression that must evaluate to true for each record. | `end_date >= start_date` or `amount > 0`. |
| `referential_integrity` | Column value must exist in a reference table/column. | `asset_type_code` exists in `prod_silver.reference_base.asset_type.code`. |
| `cross_column` | A validation that depends on multiple columns in the same record. | If `status = 'completed'` then `completion_date IS NOT NULL`. |
| `data_type` | Column value must be castable to the expected data type. | `latitude` is a valid DOUBLE. |

### 6.3 Example Configuration

Quality rules are configured in YAML as part of the entity's pipeline metadata:

```yaml
entity: asset
source_schema: prod_bronze.sap_raw
target_schema: prod_silver.asset_base

quality_rules:
  - name: "pk_unique"
    type: "uniqueness"
    columns: ["asset_id"]
    severity: "error"

  - name: "asset_id_not_null"
    type: "not_null"
    column: "asset_id"
    severity: "error"

  - name: "status_valid"
    type: "allowed_values"
    column: "status"
    values: ["active", "inactive", "decommissioned"]
    severity: "error"

  - name: "install_date_reasonable"
    type: "range"
    column: "install_date"
    min: "1900-01-01"
    max: "current_date()"
    severity: "warning"

  - name: "gps_within_wa"
    type: "sql_expression"
    expression: "latitude BETWEEN -35.5 AND -13.5 AND longitude BETWEEN 112.0 AND 129.0"
    severity: "warning"

  - name: "asset_type_exists"
    type: "referential_integrity"
    column: "asset_type_code"
    reference_table: "prod_silver.reference_base.asset_type"
    reference_column: "code"
    severity: "error"

  - name: "completion_date_when_completed"
    type: "cross_column"
    expression: "NOT (status = 'completed' AND completion_date IS NULL)"
    severity: "error"
```

### 6.4 Lakeflow Expectations Configuration

In addition to per-record DQ annotation, entities can define Lakeflow SDP expectations for pipeline-level monitoring:

```yaml
expectations:
  - name: "pk_unique_expect"
    expression: "COUNT(*) = COUNT(DISTINCT asset_id)"
    action: "EXPECT OR FAIL"

  - name: "status_valid_expect"
    expression: "status IN ('active', 'inactive', 'decommissioned')"
    action: "EXPECT"

  - name: "no_null_asset_id"
    expression: "asset_id IS NOT NULL"
    action: "EXPECT OR FAIL"
```

---

## 7. Quality Tiers and Certification

### 7.1 The `quality_tier` Governed Tag

Every Silver and Gold table in EDAP carries a `quality_tier` tag in Unity Catalog. This tag communicates the level of quality assurance applied to the data, enabling consumers to make informed trust decisions.

| Tier | Tag Value | Meaning | Consumer Guidance |
|---|---|---|---|
| **Certified** | `certified` | The data asset has passed FAUQD certification. DQ rules are defined, automated, monitored, and meeting SLAs. | Trusted for production decisions, regulatory reporting, and customer-facing outputs. |
| **Provisional** | `provisional` | An initial quality assessment has been completed. DQ rules are defined but not yet fully validated or meeting all SLAs. | Suitable for operational use with awareness of known quality limitations. Not yet approved for regulatory reporting. |
| **Uncertified** | `uncertified` | No formal quality assessment has been performed. Quality rules may not exist or may not be automated. | Use at own risk. Suitable for exploration and analysis only. Must not be used for regulatory reporting or customer-facing outputs. |

The `quality_tier` tag defaults to `uncertified` at initial ingestion and is progressed through steward review and certification processes.

### 7.2 FAUQD Certification

FAUQD is the certification test that determines whether a data asset qualifies as a certified data product. The acronym stands for:

| Letter | Criterion | Quality Relevance |
|---|---|---|
| **F** | **Findable** | The data asset is registered in the catalogue (Alation and Unity Catalog) and is discoverable by authorised consumers. |
| **A** | **Addressable** | Consumers can access the data through governed, self-service channels with clear access request processes. |
| **U** | **Understandable** | The data asset has complete business metadata — descriptions, glossary term linkages, semantic definitions, and documented lineage. |
| **Q** | **Quality-assured** | Automated DQ rules exist, are running in the pipeline, are monitored, and are meeting defined SLAs. Quality scores are published. |
| **D** | **Discoverable & Described** | The data asset has a data contract that specifies schema, freshness SLA, quality thresholds, and semantic definitions. |

The **Quality-assured** criterion is the focus of this framework. It requires:

1. **DQ rules exist** — Business quality rules have been defined by the Data Domain Steward and are documented in the entity configuration.
2. **DQ rules are automated** — Rules are implemented in the pipeline framework and execute on every pipeline run, populating DQ annotation columns and Lakeflow expectations.
3. **DQ rules are monitored** — Quality dashboards are in place, alerting is configured, and quality scores are published in the catalogue.
4. **SLAs are met** — Quality scores meet the thresholds defined in the data product's data contract. SLA breaches are investigated and resolved within defined timeframes.

### 7.3 Certification Process

| Step | Activity | Responsible Role | Outcome |
|---|---|---|---|
| 1 | **Propose certification** — Identify a data asset that is ready for quality assessment. | Data Product Owner | Certification candidate identified. |
| 2 | **Define business quality rules** — Specify the quality dimensions, rules, and thresholds that apply to this data asset based on its business context and consumer requirements. | Data Domain Steward | Quality rules documented; severity levels assigned; SLA thresholds defined. |
| 3 | **Implement quality rules** — Encode the defined rules in the pipeline framework configuration (per-record DQ annotation and Lakeflow expectations). Configure monitoring and alerting. | Technical Data Steward / Data Engineer | Rules implemented, tested, and deployed to production. |
| 4 | **Validation period** — Run the quality rules in production for a defined period (minimum 4 weeks). Monitor pass rates, identify false positives, and tune rules. | Technical Data Steward + Data Domain Steward | Rules validated; false positives resolved; quality scores stabilised. |
| 5 | **Review and certify** — Review quality scores, confirm SLAs are met, verify catalogue metadata is complete, and approve certification. Set `quality_tier = certified`. | Data Product Owner | Data asset certified; `quality_tier` tag updated; certification recorded in governance register. |
| 6 | **Ongoing monitoring** — Continuously monitor quality scores. If SLAs are breached, investigate and remediate. Certification is not permanent — it must be maintained. | Data Product Owner + Data Domain Steward | Quality maintained; certification sustained or revoked if standards slip. |

**Certification revocation:** If a certified data asset's quality scores fall below contracted SLA thresholds for a sustained period (defined in the data contract, typically 5 consecutive business days), the Data Product Owner must triage the issue. If the issue cannot be resolved within the defined response time, the `quality_tier` is downgraded to `provisional` and a quality advisory is published to consumers in the catalogue. Certification is restored only after quality scores return to SLA compliance and the root cause has been addressed.

---

## 8. Quality Observability and Monitoring

### 8.1 The Five Pillars of Data Observability

Beyond rule-based assertions, data observability provides continuous, automated monitoring of data health across all medallion layers. The five pillars complement the DQ annotation approach by detecting issues that pre-defined rules would miss.

| Pillar | What It Monitors | Medallion Relevance |
|---|---|---|
| **Freshness** | When was the table last updated? Is it within SLA? | Critical for Gold data products with defined refresh SLAs. Stale Gold-layer data may indicate pipeline failure or source system issues. |
| **Volume** | Are we receiving the expected number of rows? | Detects upstream source failures at Bronze; detects join fan-outs or unexpected row loss at Silver; validates aggregation completeness at Gold. |
| **Schema** | Have columns been added, removed, or changed type? | Auto Loader and VARIANT columns handle schema evolution at Bronze; explicit schema enforcement at Silver catches drift that may break data contracts. |
| **Distribution** | Are value distributions consistent with historical patterns? | Detects data quality degradation, upstream business changes, encoding errors, or — for AI/ML use cases — training data drift that affects model performance. |
| **Lineage** | Where did data come from and where does it flow? | Unity Catalog captures column-level lineage automatically. Broken lineage is detected before it manifests as a quality issue in downstream data products. |

### 8.2 Lakehouse Monitoring

Databricks Lakehouse Monitoring (Data Profiling) provides automated, continuous monitoring capabilities that complement per-record DQ annotation:

| Capability | Description |
|---|---|
| **Profile metrics** | Automated computation of column-level statistics — null counts, distinct counts, mean, median, min, max, standard deviation — computed on a schedule or triggered by pipeline completion. |
| **Drift detection** | Statistical comparison of current profile metrics against a baseline or previous time window. Flags significant distribution shifts automatically. |
| **Custom metrics** | User-defined SQL metrics that extend the built-in profile with domain-specific health indicators (e.g. percentage of records with `dq_status = 'FAIL'`, average days since last asset inspection). |
| **Alerting** | Threshold-based alerts on any profile metric or custom metric. Alerts route to Slack channels, PagerDuty, or email. |

Lakehouse Monitoring operates at the **table/trend level** — it does not annotate individual rows. It provides the anomaly detection layer that catches issues rule-based assertions miss: gradual distribution drift, subtle volume changes, unexpected schema evolution.

### 8.3 Quality Dashboards

A data quality dashboard is essential for making quality visible and actionable. EDAP quality dashboards should provide the following views:

| View | Content | Audience |
|---|---|---|
| **Enterprise quality summary** | Quality scores by data domain; aggregate pass rates; number of certified vs provisional vs uncertified assets; overall quality trend. | Data Governance Council, CDO, Data Governance Manager. |
| **Domain quality detail** | Per-data-product quality scores by dimension; trend over time; top failing rules; SLA breach history; open quality issues. | Data Domain Steward, Data Owner. |
| **Product quality card** | Single data product view: current quality score; DQ rule pass rates by dimension; freshness status; volume trend; schema change history; consumer count. | Data Product Owner, Data Consumers. |
| **Pipeline health** | Per-pipeline Lakeflow expectation pass rates; batch-level failure trends; quarantine path volume; processing latency. | Data Engineers, Technical Data Stewards. |
| **Quality incident tracker** | Open incidents by severity; mean time to detect (MTTD); mean time to resolve (MTTR); incident trend by domain. | Data Governance Manager, Data Domain Stewards. |

Quality scores and freshness status should be published in both Unity Catalog (as tags or properties) and Alation (as quality badges), ensuring that consumers encounter quality signals at the point of data discovery.

### 8.4 Alerting

Quality alerts ensure that issues are detected and escalated promptly. The following alert categories should be configured:

| Alert Category | Trigger | Channel | Severity |
|---|---|---|---|
| **Pipeline freshness SLA breach** | Table not refreshed within the SLA defined in the data contract. | Slack (domain channel) + PagerDuty (for SOCI-critical or Tier 1 data products). | High or Critical (depending on data product tier). |
| **DQ rule failure spike** | `dq_status = 'FAIL'` rate exceeds the threshold defined in the data contract (e.g. >5% of records). | Slack (domain channel) + PagerDuty (if threshold exceeds critical level). | High. |
| **Pipeline expectation failure** | `EXPECT OR FAIL` expectation triggers, halting the pipeline. | Slack (engineering channel) + PagerDuty. | Critical. |
| **Schema drift detected** | Column added, removed, or type-changed in a source table. | Slack (domain channel) — informational for Auto Loader resilient ingestion; escalated if it breaks a data contract. | Medium. |
| **Row count anomaly** | Volume deviates beyond expected bounds (e.g. >30% change from trailing 7-day average). | Slack (domain channel). | Medium. |
| **Distribution drift** | Lakehouse Monitoring detects statistically significant distribution change. | Slack (domain channel). | Medium. |
| **Quality score degradation** | Composite quality score drops below the SLA threshold for a certified data product. | Slack (domain channel) + email to Data Product Owner. | High. |
| **Consumer usage trends** | Weekly digest of data product consumption patterns, consumer counts, and query volumes. | Email (weekly digest to Data Product Owner). | Informational. |

---

## 9. Quality Roles and Accountability

Data quality is a shared responsibility, but accountability must be clearly assigned. The following table maps quality responsibilities to the governance roles defined in the companion Core Data Governance Roles document.

| Role | Quality Responsibility |
|---|---|
| **Data Domain Steward** | Defines business quality rules and thresholds based on domain expertise. Validates that implemented rules accurately reflect business meaning. Triages quality issues and prioritises remediation. Reviews quality dashboards for their domain. Coordinates with source system owners on upstream quality improvement. |
| **Technical Data Steward** | Implements DQ rules in pipeline configurations and Unity Catalog. Configures Lakeflow expectations. Manages DQ annotation column logic. Sets up Lakehouse Monitoring, alerting, and quality dashboards. Tunes rules to eliminate false positives. |
| **Data Product Owner** | Owns quality SLAs for their data product as defined in the data contract. Responds to quality incidents affecting their product. Drives remediation and root cause resolution. Decides when quality is sufficient for certification. Communicates quality status to consumers. |
| **Data Engineer** | Implements quality assertions in pipeline code. Builds and maintains the quality monitoring infrastructure. Ensures DQ annotation columns are correctly populated. Manages pipeline-level expectations. |
| **Data Governance Manager** | Sets enterprise quality policy and standards. Reports on quality KPIs to the Data Governance Council. Manages the quality improvement programme. Ensures quality processes are consistent across domains. Tracks quality maturity progression. |
| **Data Consumer** | Reports quality issues through defined channels (catalogue feedback, Slack, or incident management). Provides feedback on whether data quality is fit for their specific purpose. Uses `dq_status` columns and quality scores to make informed trust decisions. |

**RACI for quality rule implementation:**

| Activity | Data Domain Steward | Technical Data Steward | Data Product Owner | Data Engineer |
|---|---|---|---|---|
| Define business quality rules | **A/R** | C | C | I |
| Implement rules in pipeline | C | **A** | I | **R** |
| Validate rule accuracy | **R** | C | **A** | I |
| Monitor quality dashboards | **R** | **R** | **A** | I |
| Triage quality incidents | **R** | C | **A** | I |
| Remediate pipeline issues | I | **A** | I | **R** |
| Remediate source data issues | **R** | I | **A** | I |
| Certify data product quality | C | C | **A/R** | I |

**A** = Accountable, **R** = Responsible, **C** = Consulted, **I** = Informed

---

## 10. Quality Incident Management

### 10.1 Incident Lifecycle

When a data quality issue is detected, it follows a defined lifecycle to ensure consistent, timely resolution:

```
Detection → Triage → Root Cause Analysis → Remediation → Verification → Post-Incident Review
```

| Phase | Activity | Responsible |
|---|---|---|
| **Detection** | Quality issue identified through automated assertion failure, observability alert, Lakehouse Monitoring anomaly, or consumer report. | Automated systems or Data Consumer. |
| **Triage** | Assess severity, scope, and impact. Determine whether this is a pipeline issue, source system issue, or business rule change. Assign to the appropriate resolver. | Technical Data Steward. |
| **Root Cause Analysis** | Use data lineage (Unity Catalog column-level lineage) to trace the issue to its origin. Identify whether the root cause is upstream (source system), midstream (pipeline logic), or downstream (business rule definition). | Technical Data Steward + Data Domain Steward. |
| **Remediation** | Fix the root cause. This may involve correcting source data, adjusting pipeline logic, updating business rules, or flagging the data product as degraded with a quality advisory in the catalogue. | Data Engineer (pipeline), Source System Owner (source data), Data Domain Steward (business rules). |
| **Verification** | Re-run quality assertions to confirm the fix. Validate that quality scores return to SLA compliance. Update the data product's quality status in the catalogue. | Technical Data Steward. |
| **Post-Incident Review** | For High and Critical incidents: conduct a blameless post-incident review. Document the root cause, timeline, resolution, and prevention measures. Add or strengthen assertions to prevent recurrence. Update data contracts if quality specifications were insufficient. | Data Product Owner (convenes), all involved roles. |

### 10.2 Severity Levels

| Severity | Definition | Response Time | Resolution Target | Example |
|---|---|---|---|---|
| **Critical** | Data product unusable; regulatory or safety impact; SOCI or PRIS compliance at risk. | 1 hour | 4 hours | SOCI compliance dataset missing critical records; customer PI exposed due to masking failure. |
| **High** | Data product materially degraded; significant business impact; SLA breach on Tier 1 product. | 4 hours | 1 business day | Customer billing dimension has >5% null addresses; Gold KPI dashboard showing incorrect aggregations. |
| **Medium** | Data quality below SLA but product still usable; minor business impact. | 1 business day | 3 business days | Asset condition fact table freshness SLA breached by 2 hours; warning-level DQ rule spike on non-critical dimension. |
| **Low** | Minor quality issue; no material business impact; cosmetic or edge-case concern. | 5 business days | 10 business days | Warning-level DQ rule triggering on a non-critical field; minor referential integrity issue in low-usage exploratory table. |

### 10.3 Escalation Path

| Condition | Escalation |
|---|---|
| Critical incident not acknowledged within 1 hour | Escalate to Data Owner and Data Governance Manager. |
| High incident not resolved within 1 business day | Escalate to Data Product Owner and Data Domain Steward. |
| Recurring incident (same root cause, third occurrence) | Escalate to Data Governance Council for systemic review. |
| Incident with regulatory impact (SOCI, PRIS) | Immediate notification to Data Protection Officer and CISO. |

---

## 11. Continuous Improvement

### 11.1 Quality Maturity Model

Data quality management matures through defined stages. The maturity model provides a roadmap for Water Corporation's progression from reactive quality management to predictive, self-healing quality systems.

| Level | Name | Characteristics | EDAP Indicators |
|---|---|---|---|
| **1** | **Reactive** | Quality issues discovered when reports break or consumers complain. No systematic quality rules. Ad hoc fixes. | No DQ annotation columns populated. No Lakeflow expectations. `quality_tier = uncertified` on all assets. Quality issues raised informally. |
| **2** | **Managed** | Quality rules defined for critical data products. DQ annotation columns populated. Basic monitoring in place. Issues tracked. | DQ rules configured for Tier 1 data products. `quality_tier = provisional` for key assets. Quality dashboards exist but are manually reviewed. |
| **3** | **Defined** | Quality rules defined across all governed data products. Automated monitoring and alerting. Formal incident process. Quality scores published. | DQ rules for all Tier 1 and Tier 2 products. FAUQD certification process operational. Quality scores in the catalogue. Incident lifecycle followed consistently. |
| **4** | **Measured** | Quality metrics feed governance KPIs. Trend analysis informs improvement priorities. Quality SLAs in all data contracts. Root cause patterns analysed systematically. | Quality KPIs reported to Data Governance Council monthly. MTTR tracked and improving. Quality rule coverage >90% for certified products. SLA adherence >95%. |
| **5** | **Optimising** | Predictive quality management. Anomaly detection catches issues before they breach SLAs. Quality rules evolve based on consumer feedback and usage patterns. Self-healing pipelines retry and correct known failure patterns. | Lakehouse Monitoring drift detection operational. Predictive alerts reduce incident volume. Quality improvement programme delivers measurable business value. Consumer satisfaction with data quality tracked and trending upward. |

### 11.2 Quality Metrics

The following metrics drive continuous improvement and are reported through governance channels:

| Metric | Definition | Target | Reporting Cadence |
|---|---|---|---|
| **Quality score** | Composite score per data product based on DQ rule pass rates weighted by dimension importance. | >95% for certified products. | Continuous (per pipeline run). |
| **DQ rule coverage** | Percentage of columns with at least one DQ rule defined, for governed data products. | >90% for certified products. | Monthly. |
| **SLA adherence rate** | Percentage of pipeline runs where quality SLAs (freshness, completeness, accuracy thresholds) were met. | >95%. | Weekly. |
| **Mean time to detect (MTTD)** | Average time from quality issue occurrence to detection. | <1 hour for Critical/High; <4 hours for Medium. | Monthly. |
| **Mean time to resolve (MTTR)** | Average time from quality issue detection to resolution. | Within severity response targets. | Monthly. |
| **Certification coverage** | Percentage of Tier 1 and Tier 2 data products with `quality_tier = certified`. | 100% of Tier 1; >80% of Tier 2. | Monthly. |
| **Quarantine path volume** | Number of records routed to the quarantine path (structural failures only). Sustained growth indicates a systemic source system issue. | Trending stable or decreasing. | Weekly. |
| **Consumer-reported issues** | Number of quality issues reported by consumers (i.e. not caught by automated rules). | Trending downward as rule coverage improves. | Monthly. |

### 11.3 Improvement Cycle

Quality improvement is not a one-off project. It follows a continuous cycle:

1. **Measure** — Collect quality metrics from DQ annotation columns, Lakeflow event logs, and Lakehouse Monitoring.
2. **Analyse** — Identify patterns: Which domains have the lowest quality scores? Which rules fail most frequently? Which source systems generate the most issues? Where are consumers reporting problems that automated rules miss?
3. **Prioritise** — Focus improvement effort where it delivers the most business value. A quality issue affecting a Tier 1 regulatory reporting product takes priority over one affecting an exploratory dataset.
4. **Act** — Implement improvements: add missing rules, tune existing rules, engage source system owners on upstream fixes, strengthen data contracts, improve monitoring.
5. **Review** — Assess the impact of improvements. Have quality scores improved? Has MTTR decreased? Are consumer-reported issues declining? Report outcomes to the Data Governance Council.

---

## 12. Companion Documents

This Data Quality Framework operates within the broader EDAP governance and architecture ecosystem. The following companion documents provide additional context and detail.

| Document | Relevance to Data Quality |
|---|---|
| **Medallion Architecture** | Defines the zone-by-zone quality approach, the "Annotate, Never Filter" principle, DQ annotation columns, data observability pillars, and the quarantine path for structural failures. |
| **EDAP Pipeline Framework** | Specifies the technical requirements for DQ annotation (FR-DQ-01 through FR-DQ-10), including per-record annotation, rule types, Lakeflow expectation integration, and zone-specific processing rules. |
| **EDAP Tagging Strategy** | Defines the `quality_tier` governed tag (`certified`, `provisional`, `uncertified`) and its relationship to the classification lifecycle and cross-domain access model. |
| **Core Data Governance Roles** | Defines the roles accountable for quality: Data Domain Steward, Technical Data Steward, Data Product Owner, Data Engineer, Data Governance Manager, and Data Consumer. |
| **Data Governance Lifecycle** | Stage 4 (Data Quality Management) establishes quality as a continuous governance discipline, defines the quality remediation process, and positions quality scoring as a governance KPI. |
| **EDAP Data Products** | Defines the FAUQD certification test, data product tiers, and data contracts — all of which embed quality requirements as a first-class concern. |
| **Databricks End-to-End Platform** | Details Lakehouse Monitoring (Data Profiling) capabilities, Lakeflow SDP expectations, Unity Catalog lineage, and the technical infrastructure that supports quality observability. |

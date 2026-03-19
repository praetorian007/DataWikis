# S23.1 – Data & Analytics Governance Advice: Data Product SDLC and Quality Assurance: Feature Breakdown

**Scope Area:** PMO & Support
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `lifecycles/data-engineering-lifecycle.md` (all stages; DataOps undercurrent; Software Engineering undercurrent)
- `lifecycles/data-governance-lifecycle.md` (Stage 4 – Data Quality and Observability; Stage 10 – Continuous Improvement)
- `governance/data-governance-roles.md` (Data Product Owner, Data Domain Steward, Technical Data Steward)

---

## Feature S23.1-F1: Current State Assessment

**Description:** Review Water Corporation's existing data product lifecycle, governance practices, and quality assurance processes to establish a clear baseline of maturity, strengths, and areas requiring improvement.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.1-F1-US01 | Data Governance Manager | have a documented assessment of our current data product lifecycle practices | I can understand our baseline maturity and communicate it to stakeholders |
| S23.1-F1-US02 | Data Product Owner | understand how our existing SDLC processes compare to industry expectations | I can identify where my team's practices need to evolve |
| S23.1-F1-US03 | Data Platform Owner | have visibility of current QA processes across all data products | I can assess the consistency and effectiveness of quality assurance across the platform |
| S23.1-F1-US04 | Data Domain Steward | understand documented vs actual governance practices for data products | I can identify where governance is operating informally and needs formalisation |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.1-F1-AC01 | Stakeholder interviews are planned | the assessment commences | interviews are conducted with at least one representative from each of the following: data engineering, data science, BI/analytics, data governance, and business domain ownership |
| S23.1-F1-AC02 | Existing documentation is collected | the review is completed | all existing SDLC, QA, and governance documents are catalogued with a currency assessment (current, outdated, or missing) for each |
| S23.1-F1-AC03 | Current practices are analysed | the assessment report is produced | the report rates maturity across at least six dimensions: design, build, test, deploy, monitor, and retire, using a defined maturity scale (e.g., Initial, Managed, Defined, Measured, Optimised) |
| S23.1-F1-AC04 | QA processes are assessed | findings are documented | each existing QA process is evaluated against the five data quality dimensions (freshness, accuracy, uniqueness, completeness, distribution) with coverage gaps identified |
| S23.1-F1-AC05 | The assessment is finalised | it is presented to Water Corporation | the report includes an executive summary of no more than two pages and a detailed appendix with supporting evidence |

### Technical Notes
- Use the Data Engineering Lifecycle stages (Generation, Ingestion, Storage, Transformation, Serving) as a framework for assessing pipeline development practices.
- Assess current data quality practices against the Data Governance Lifecycle Stage 4 (Data Quality and Observability) expectations.
- Evaluate governance role clarity against the roles defined in `governance/data-governance-roles.md`, particularly Data Product Owner and Technical Data Steward.
- Consider alignment with the DataOps undercurrent (automated testing, CI/CD, monitoring) as a maturity indicator.

---

## Feature S23.1-F2: Best Practice Framework

**Description:** Define an industry best practice framework for the data product software development lifecycle covering design, build, test, deploy, monitor, and retire stages, contextualised for Water Corporation's Databricks lakehouse platform.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.1-F2-US01 | Data Engineer | have a defined SDLC framework for data products that covers all lifecycle stages | I can follow a consistent, repeatable process from design through to retirement |
| S23.1-F2-US02 | Data Product Owner | understand industry best practices for each SDLC stage | I can set appropriate standards and expectations for my data product teams |
| S23.1-F2-US03 | Data Governance Manager | have a reference framework that connects SDLC practices to governance requirements | I can ensure governance is embedded in the lifecycle rather than bolted on |
| S23.1-F2-US04 | Technical Data Steward | understand testing and quality best practices for data pipelines | I can implement appropriate quality gates at each lifecycle stage |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.1-F2-AC01 | The framework document is drafted | it is reviewed by stakeholders | it covers all six lifecycle stages: design, build, test, deploy, monitor, and retire, with defined activities, artefacts, and quality gates for each |
| S23.1-F2-AC02 | Industry references are compiled | the framework is finalised | at least five industry sources are referenced (e.g., DAMA-DMBOK2, Databricks best practices, DataOps Manifesto, relevant Australian standards) |
| S23.1-F2-AC03 | The framework addresses governance integration | the document is reviewed | each SDLC stage identifies the governance touchpoints, responsible roles (per `governance/data-governance-roles.md`), and required approvals |
| S23.1-F2-AC04 | The framework is contextualised for Databricks | the document is accepted | each stage includes platform-specific guidance referencing Databricks capabilities (Unity Catalog, Lakeflow, Lakehouse Monitoring, Asset Bundles) |
| S23.1-F2-AC05 | The retire stage is defined | the framework is reviewed | retirement criteria include data product deprecation notice periods, consumer impact assessment, archive requirements, and catalogue deregistration procedures |

### Technical Notes
- The Design stage should reference data contract patterns (schema, SLA, quality thresholds) aligned with the Data Product Owner role definition.
- The Build stage should reference the medallion architecture (Bronze/Silver/Gold), pipeline patterns, and coding standards from the Data Engineering Lifecycle.
- The Test stage should incorporate unit testing, integration testing, and data validation patterns from the DataOps undercurrent.
- The Deploy stage should reference the CI/CD and promotion strategy (Dev to Test to Prod) from S7 DataOps Enablement.
- The Monitor stage should reference Lakehouse Monitoring and the Data Governance Lifecycle Stage 4 (Data Quality and Observability).
- The Retire stage should align with data product lifecycle management responsibilities of the Data Product Owner.

---

## Feature S23.1-F3: Gap Analysis and Recommendations

**Description:** Analyse the gaps between Water Corporation's current state (F1) and the target best practice framework (F2), and produce a prioritised improvement roadmap with actionable recommendations.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.1-F3-US01 | Data Governance Manager | have a clear gap analysis comparing our current practices to the target framework | I can understand the scale of change required and communicate it to leadership |
| S23.1-F3-US02 | Data Platform Owner | have a prioritised roadmap of improvements | I can sequence investment and effort across sprints and PIs |
| S23.1-F3-US03 | Project Manager | understand the dependencies and effort estimates for each recommendation | I can incorporate improvements into delivery planning |
| S23.1-F3-US04 | Data Product Owner | have specific, actionable recommendations for my domain | I can start implementing improvements within my team without waiting for enterprise-wide changes |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.1-F3-AC01 | The current state assessment (F1) and best practice framework (F2) are complete | the gap analysis is performed | every SDLC stage has gaps documented with a severity rating (critical, high, medium, low) based on impact to data product quality and governance compliance |
| S23.1-F3-AC02 | Gaps are identified | recommendations are drafted | each gap has at least one actionable recommendation with an estimated effort (T-shirt size: S, M, L, XL), responsible role, and dependency on other recommendations |
| S23.1-F3-AC03 | Recommendations are prioritised | the roadmap is produced | recommendations are sequenced into three horizons: quick wins (achievable within current PI), medium-term (1-2 PIs), and strategic (3+ PIs), with rationale for prioritisation |
| S23.1-F3-AC04 | The gap analysis is reviewed | Water Corporation accepts the document | the analysis includes a visual maturity heatmap showing current vs target state across all SDLC stages and quality dimensions |
| S23.1-F3-AC05 | Recommendations are finalised | the roadmap is accepted | the roadmap identifies at least three quick wins that can be implemented immediately to demonstrate value |

### Technical Notes
- Prioritisation should consider dependencies on other EDAP scope items (e.g., S7 DataOps Enablement, S10 Testing, S6 EDP Implementation).
- Quick wins should focus on areas where Databricks platform capabilities (Lakehouse Monitoring, Unity Catalog tagging, Lakeflow expectations) can address gaps with minimal custom development.
- The roadmap should align with SAFe PI planning cadence for integration into delivery planning.
- Reference the Continuous Improvement stage (Stage 10) of the Data Governance Lifecycle for ongoing governance maturity improvement patterns.

---

## Feature S23.1-F4: Quality Assurance Framework

**Description:** Define a comprehensive data quality assurance framework covering the five core quality dimensions (freshness, accuracy, uniqueness, completeness, distribution), testing patterns for data pipelines, and integration with Databricks Lakehouse Monitoring.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.1-F4-US01 | Data Engineer | have defined testing patterns for each quality dimension | I can implement consistent quality checks across all data pipelines |
| S23.1-F4-US02 | Technical Data Steward | have a framework for setting quality thresholds per data product | I can configure meaningful quality gates that catch genuine issues without excessive false positives |
| S23.1-F4-US03 | Data Product Owner | have quality SLAs defined for my data products | I can commit to measurable quality standards and track compliance over time |
| S23.1-F4-US04 | Data Consumer | have visibility into the quality status of the data I consume | I can make informed decisions about whether to trust the data for my analysis |
| S23.1-F4-US05 | Data Domain Steward | have an alerting and escalation framework for quality breaches | I can respond to quality issues promptly and ensure they are resolved within agreed timeframes |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.1-F4-AC01 | The framework document is drafted | it is reviewed | all five quality dimensions are defined with measurable metrics: freshness (time since last update vs SLA), accuracy (error rate against validated source), uniqueness (duplicate record percentage), completeness (null/missing value percentage), and distribution (statistical drift from baseline) |
| S23.1-F4-AC02 | Testing patterns are defined | the framework is reviewed | each quality dimension has at least two testing patterns documented (e.g., row count assertions, schema validation, referential integrity checks, statistical profiling, anomaly detection) |
| S23.1-F4-AC03 | Lakehouse Monitoring integration is specified | the framework is accepted | the framework includes configuration guidance for Databricks Lakehouse Monitoring covering: profile monitoring (statistical summaries), drift monitoring (distribution changes), and custom metric monitoring (business-specific rules) |
| S23.1-F4-AC04 | Alerting and escalation rules are defined | the framework is reviewed | quality breaches are classified into three severity tiers (informational, warning, critical) with defined response timeframes and escalation paths for each |
| S23.1-F4-AC05 | Quality reporting is specified | the framework is accepted | the framework defines a quality dashboard specification showing per-data-product quality scores across all five dimensions with trend lines over a rolling 90-day window |
| S23.1-F4-AC06 | The framework addresses pipeline testing | the document is reviewed | testing patterns cover unit tests (individual transformation logic), integration tests (end-to-end pipeline correctness), and regression tests (post-deployment validation) with guidance on when each test type applies |

### Technical Notes
- Freshness monitoring should leverage Unity Catalog table metadata (last modified timestamps) and Lakehouse Monitoring freshness profiles.
- Accuracy testing should include referential integrity checks across medallion layers (Bronze to Silver reconciliation, Silver to Gold reconciliation).
- Uniqueness checks should use primary key validation at the Silver and Gold layers, leveraging Delta Lake constraints where applicable.
- Completeness monitoring should be configurable per column with different thresholds (e.g., critical fields require 100% completeness, optional fields may tolerate 5% nulls).
- Distribution monitoring using Lakehouse Monitoring can detect statistical drift, which is critical for both data quality and ML model performance monitoring.
- Align with Lakeflow Declarative Pipeline expectations for inline quality checks (expect, expect or drop, expect or fail).
- Reference the Data Governance Lifecycle Stage 4 (Data Quality and Observability) for governance context.

# S10 – Testing and Non-Prod Environments: Feature Breakdown

**Scope Area:** EDP Detailed Design
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `platform/edap-access-model.md` — Workspace topology, catalog bindings, dev/staging/prod separation, sandbox governance
- `governance/edap-tagging-strategy.md` — Classification lifecycle, PI handling in non-prod
- `platform/medallion-architecture.md` — Data quality framework, quarantine patterns, layer structure
- `specifications/edap-pipeline-framework.md` — Pipeline framework, DQ expectations, Lakeflow SDP

---

## Feature S10-F1: Test Environment Architecture

**Description:** Design and configure Dev and Test (staging) workspace environments with cost-optimised compute, clear catalog bindings, and infrastructure controls that ensure ongoing non-production platform costs do not exceed 10% of production environment costs.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S10-F1-US01 | Platform Administrator | configure the `wc-edap-dev` and `wc-edap-staging` workspaces with appropriate catalog bindings | developers and testers have access to the correct catalogs (dev read-write, prod read-only) per the defined workspace topology |
| S10-F1-US02 | Platform Administrator | implement cluster policies with aggressive autoscaling limits, auto-termination, and instance type restrictions for non-prod workspaces | compute costs are constrained to meet the 10% cost target |
| S10-F1-US03 | Data Engineer | use serverless compute for development and testing workloads by default | I avoid idle cluster costs and the platform only incurs charges during active use |
| S10-F1-US04 | Finance Analyst | view a monthly cost comparison between production and non-production environments | I can verify that the 10% cost target is being maintained and identify cost anomalies |
| S10-F1-US05 | Platform Administrator | define separate storage quotas and retention policies for dev and staging catalogs | non-production storage does not grow unbounded and is cleaned up automatically |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S10-F1-AC01 | Dev and staging workspaces are configured | catalog bindings are reviewed | `wc-edap-dev` has read-write access to `dev_*` catalogs and read-only access to selected `prod_*` catalogs; `wc-edap-staging` has read-write access to `staging_*` catalogs and read-only access to `prod_*` catalogs |
| S10-F1-AC02 | Cluster policies are applied to non-prod workspaces | a user creates a cluster in the dev workspace | the cluster is constrained to approved instance types, maximum 4 workers, auto-terminates after 30 minutes of inactivity, and uses spot instances |
| S10-F1-AC03 | Serverless compute is the default for non-prod | a user runs a notebook or SQL query in the dev workspace without specifying compute | the workload executes on serverless compute |
| S10-F1-AC04 | Cost monitoring is configured | the monthly cost report is generated for Month 2 onwards | non-production DBU and storage costs are itemised separately and verified to be at or below 10% of production costs |
| S10-F1-AC05 | Retention policies are configured for dev/staging catalogs | a dev catalog table exceeds the configured retention period (e.g. 30 days since last update) | the table is flagged for cleanup and data stewards are notified |
| S10-F1-AC06 | Cost alerting is configured | non-prod costs reach 80% of the 10% budget threshold in a billing period | an automated alert is sent to the platform team |

### Technical Notes
- Workspace topology follows the access model wiki Section 3.2: `wc-edap-dev`, `wc-edap-staging`, `wc-edap-prod`.
- Serverless compute eliminates idle cluster costs and is recommended for non-prod to minimise spend.
- Cluster policies should enforce spot instances for non-prod workloads where supported.
- The 10% cost target is a contractual requirement from the scope of work; it should be validated with actual use case workloads and reported monthly.
- Consider using Databricks Budgets (account-level) to enforce spending limits.

---

## Feature S10-F2: Test Data Management

**Description:** Implement test data provisioning for non-production environments using masked production data samples, synthetic data generation, and PI-compliant anonymisation to ensure realistic testing without exposing sensitive information.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S10-F2-US01 | Data Engineer | access realistic, masked test data in the dev workspace | I can develop and test pipelines against data that reflects production schema and distribution without accessing real PI |
| S10-F2-US02 | Platform Administrator | automate the refresh of non-production datasets from production | test data is kept current without manual intervention |
| S10-F2-US03 | Data Protection Officer | verify that no unmasked PI exists in dev or staging catalogs | PRIS Act compliance is maintained across all non-production environments |
| S10-F2-US04 | Data Scientist | generate synthetic datasets for domains where production data sampling is insufficient or prohibited | I can develop models against statistically representative data |
| S10-F2-US05 | QA Analyst | access edge-case and boundary-condition test data | I can validate pipeline behaviour against known difficult data patterns (nulls, special characters, schema variations) |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S10-F2-AC01 | Production data masking is configured | a test data refresh pipeline runs | all columns tagged with `pi_category` in the source production table are masked or anonymised in the target dev/staging table |
| S10-F2-AC02 | Data sampling is configured | a test data refresh pipeline runs for a large production table (>10M rows) | the dev/staging target contains a statistically representative sample (configurable, default 5% or 500K rows, whichever is smaller) |
| S10-F2-AC03 | Synthetic data generation is available | a domain team requests synthetic data for a new use case | a synthetic dataset is generated matching the target schema, data types, and distribution profile within 5 business days |
| S10-F2-AC04 | PI compliance scanning is configured | a weekly scan runs across all dev and staging catalogs | zero tables contain unmasked direct identifiers (as tagged by `pii_type=direct_identifier` in the tagging strategy) |
| S10-F2-AC05 | Automated refresh is scheduled | the monthly test data refresh completes | dev/staging catalogs contain data no older than 30 days from the production source |
| S10-F2-AC06 | Edge-case test data is provisioned | a QA analyst queries the test data catalogue | dedicated test datasets exist for null handling, schema drift, special characters, and boundary values |

### Technical Notes
- PI terminology: WC uses Personal Information (PI) per the PRIS Act 2024, not PII, per the access model wiki Section 2.
- Masking in non-prod aligns with ADR-EDP-001 (Development Environment Data Strategy) referenced in the access model wiki.
- ABAC policies may be relaxed on dev catalogs using anonymised data since the underlying data does not contain real PI per the access model wiki Section 6.4.
- Sandbox schemas must not contain unmasked PI per the access model wiki Section 3.3 (Sandbox Governance).
- Synthetic data should preserve referential integrity and statistical distributions to be useful for pipeline testing and ML model development.

---

## Feature S10-F3: Automated Test Framework

**Description:** Implement an automated testing framework covering unit tests for pipeline logic, integration tests across datasets, data quality validation using Lakeflow expectations, and regression testing to catch pipeline regressions during iterative development.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S10-F3-US01 | Data Engineer | run unit tests for my pipeline transformation logic using pytest | I can validate individual functions and transformations in isolation before committing code |
| S10-F3-US02 | Data Engineer | run integration tests that validate end-to-end pipeline execution from Bronze to Gold | I can confirm that data flows correctly across zone transitions with the expected schema and row counts |
| S10-F3-US03 | Data Engineer | define data quality expectations as part of my pipeline configuration | DQ rules are enforced automatically at each zone transition, with failures quarantined per the medallion architecture |
| S10-F3-US04 | QA Analyst | run regression tests after each pipeline code change | I can detect unintended changes in output schema, row counts, or data values |
| S10-F3-US05 | Platform Administrator | view test results in the CI/CD pipeline (GitHub Actions) | test pass/fail status gates pull request merges and environment promotions |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S10-F3-AC01 | Unit test framework is configured with pytest | a developer runs `pytest` against a pipeline module | all transformation functions execute with Spark session mocking and assert expected outputs for given inputs |
| S10-F3-AC02 | Integration tests are defined for a pipeline | the integration test suite runs in the staging workspace | the pipeline executes end-to-end against test data, producing output tables that match expected schema, row counts (within ±1% tolerance), and key column values |
| S10-F3-AC03 | Lakeflow expectations are configured per the pipeline framework spec | a pipeline runs with DQ expectations active | records violating expectations are routed to the quarantine table with failure metadata (rule name, failed value, timestamp) |
| S10-F3-AC04 | Regression test baselines are established | a pipeline code change is deployed to the staging workspace | the regression suite compares output against the baseline and reports any differences in schema, row count, or aggregate values |
| S10-F3-AC05 | CI/CD integration is configured | a pull request is submitted in GitHub | unit tests execute automatically via GitHub Actions; the PR cannot be merged if tests fail |
| S10-F3-AC06 | Test coverage reporting is enabled | a test suite completes | a coverage report is generated showing the percentage of pipeline logic covered by unit and integration tests (target: ≥80% for core transformation logic) |

### Technical Notes
- Unit tests use pytest with PySpark session mocking per the DataOps scope item (S7).
- Integration tests execute against the staging workspace using staging catalogs with test data.
- DQ expectations use Lakeflow Declarative Pipeline `expect`, `expect_or_fail`, `expect_or_drop`, and `expect_or_quarantine` per the pipeline framework spec.
- Quarantine pattern follows the medallion architecture wiki: separate quarantine tables per layer with failure metadata.
- Regression baselines should be stored as Delta tables in the staging catalog for comparison.
- CI/CD pipeline (GitHub Actions) runs unit tests on every PR; integration tests run on merge to the dev/staging branch.

---

## Feature S10-F4: Performance Testing

**Description:** Design and execute performance tests for pipeline throughput, SQL Warehouse query performance, concurrent user workloads, and compute tuning to ensure the platform meets operational SLAs and can handle expected data volumes.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S10-F4-US01 | Data Engineer | benchmark pipeline throughput for batch ingestion at expected production volumes | I can verify that pipelines complete within defined SLA windows (e.g. nightly batch within 4 hours) |
| S10-F4-US02 | Data Analyst | benchmark SQL Warehouse query response times for typical Gold-layer analytical queries | I can confirm that dashboards and reports load within acceptable response times (e.g. <10 seconds for standard queries) |
| S10-F4-US03 | Platform Administrator | test concurrent user scenarios on SQL Warehouses | I can right-size warehouse configurations to handle peak analytical workloads |
| S10-F4-US04 | Data Engineer | identify and resolve performance bottlenecks in pipeline processing | pipelines are tuned using liquid clustering, optimised joins, and appropriate compute sizing |
| S10-F4-US05 | Platform Administrator | document performance baselines and tuning recommendations | the operations team has clear guidance for ongoing performance management |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S10-F4-AC01 | Performance test data is provisioned at production-scale volumes (or extrapolated from sample) | the batch ingestion pipeline for the largest source system runs | the pipeline completes within the defined SLA window (to be agreed per use case, default: 4 hours for nightly batch) |
| S10-F4-AC02 | A representative set of Gold-layer analytical queries is defined | queries are executed against a production-scale dataset on a SQL Warehouse | 90th percentile query response time is below 10 seconds for standard queries and below 60 seconds for complex cross-domain queries |
| S10-F4-AC03 | Concurrent user testing is configured | 20 concurrent users execute queries against a SQL Warehouse simultaneously | the warehouse auto-scales to handle the load with no query failures and p95 response time degrades by no more than 50% compared to single-user baseline |
| S10-F4-AC04 | Performance bottlenecks are identified | tuning recommendations are applied (e.g. liquid clustering, partition pruning, materialized views) | a measurable improvement (≥20%) is demonstrated in the affected metric (throughput or query time) |
| S10-F4-AC05 | Performance testing is complete | a performance test report is produced | the report documents: test scenarios, data volumes, measured throughput/latency, tuning actions taken, and baseline metrics for ongoing monitoring |

### Technical Notes
- Liquid clustering is the modern replacement for Z-ordering per the medallion architecture wiki; performance tests should validate clustering key selection.
- Materialized views should be used for frequently accessed Gold aggregations per the medallion wiki guidance.
- SQL Warehouse sizing should consider serverless SQL Warehouses for cost-effective auto-scaling.
- Performance baselines should be stored in `prod_platform` for ongoing comparison.
- Predictive optimisation (auto-OPTIMIZE, auto-VACUUM, auto-ANALYZE) should be enabled and its impact measured.

---

## Feature S10-F5: UAT Enablement

**Description:** Create UAT test scripts, business user test scenarios, a defect tracking process, and a formal sign-off procedure to support Water Corporation's user acceptance testing of the EDAP platform and business use cases.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S10-F5-US01 | Business User | receive clear, step-by-step UAT test scripts for each business use case | I can execute tests without requiring deep technical knowledge of the platform |
| S10-F5-US02 | Business User | log defects found during UAT with sufficient detail for the development team to reproduce and resolve them | defects are tracked transparently and resolved before sign-off |
| S10-F5-US03 | Domain Data Steward | validate that data quality, access controls, and classifications are correctly applied in the production environment | I can confirm governance requirements are met before accepting delivery |
| S10-F5-US04 | Project Manager | track UAT progress, defect resolution, and sign-off status | I have clear visibility of readiness for production deployment |
| S10-F5-US05 | Business User | formally sign off on completed UAT cycles | there is documented acceptance that the delivered functionality meets business requirements |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S10-F5-AC01 | UAT test scripts are authored for each business use case | a business user receives the test scripts | each script includes: test objective, preconditions, step-by-step instructions, expected results, and pass/fail recording fields |
| S10-F5-AC02 | UAT environment is configured | a business user logs into the staging workspace to execute UAT | they can access the required test data, dashboards, and reports without assistance from the platform team |
| S10-F5-AC03 | Defect tracking is established in JIRA | a business user identifies a defect during UAT | they can log it via a standard JIRA template that captures: steps to reproduce, expected vs actual result, severity, and screenshots |
| S10-F5-AC04 | UAT defects are being tracked | a critical or high severity defect is logged | it is triaged by the development team within 1 business day and a resolution plan is communicated to the UAT tester |
| S10-F5-AC05 | All UAT test scripts have been executed | the UAT cycle is complete | a sign-off document is produced listing: total tests executed, passed, failed, deferred, outstanding defects, and formal acceptance (or rejection with rationale) by the business owner |
| S10-F5-AC06 | UAT governance scenarios are included | the business user executes access control test scripts | they can verify that RBAC and ABAC restrictions are correctly applied (e.g. PI masking is active, restricted data is inaccessible to unauthorised users) |

### Technical Notes
- Water Corporation conducts UAT per the scope of work; the SI facilitates UAT preparation and defect resolution.
- UAT environment uses the staging workspace with test data per S10-F2.
- Test scripts should cover platform capabilities (pipeline execution, data quality, access controls) and business use case functionality (reports, dashboards, data products).
- Defect tracking uses JIRA aligned with the project's SAFe delivery cadence.
- Governance-specific UAT scenarios should validate ABAC masking, RBAC restrictions, and break-glass procedures per the access model wiki.
- Sign-off process must align with the scope criteria: "access control UAT passed" and "UAT enablement complete".

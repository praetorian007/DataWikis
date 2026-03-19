# S7 – DataOps Enablement: Feature Breakdown

**Scope Area:** EDP Detailed Design
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:** Data Engineering Lifecycle (DataOps undercurrent), Databricks End-to-End Platform (DABs, Lakeflow Jobs), EDAP Access Model (naming, tagging), Medallion Architecture

---

## Feature S7-F1: Governed Code Promotion from Development to Production

**Description:** Engineers can collaborate on pipeline code through a structured Git workflow with mandatory reviews, automated checks, and protected branches — so that every change reaching production has been peer-reviewed, tested, and approved through a predictable promotion path.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S7-F1-US01 | Platform Engineer | define a Git repository structure that separates notebooks, pipeline code, job configurations, DABs deployment definitions, and infrastructure templates | assets are logically organised, independently deployable, and easy to navigate for engineers across domains |
| S7-F1-US02 | Data Engineer | follow a defined branching strategy (main, develop, feature/*, release/*, hotfix/*) with clear merge rules | code changes flow through a predictable lifecycle from development to production with appropriate review gates |
| S7-F1-US03 | Data Engineer | create a pull request for my feature branch with mandatory code review by at least one peer and one WC DataOps team member | all code is reviewed for quality, standards compliance, and alignment with EDAP patterns before merging |
| S7-F1-US04 | Data Engineer | see automated checks (linting, unit tests, security scans) run on my pull request before merge is permitted | code quality issues are caught early and consistently without relying solely on manual review |
| S7-F1-US05 | WC DataOps Engineer | participate in code review for all pull requests as part of the knowledge transfer process | the WC team builds capability and familiarity with the codebase throughout the project |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S7-F1-AC01 | The repository structure is defined | the repository root is inspected | it contains clearly separated directories for pipeline code (src/), notebooks (notebooks/), DABs bundles (bundles/), tests (tests/), and infrastructure (infra/) with a documented README |
| S7-F1-AC02 | The branching strategy is documented | a Data Engineer creates a feature branch | the branch follows the naming convention feature/<ticket-id>-<short-description> and is created from the develop branch |
| S7-F1-AC03 | Branch protection rules are configured on main and develop | a direct push to main is attempted | the push is rejected; changes can only be merged via pull request |
| S7-F1-AC04 | Pull request workflows are configured | a pull request is opened | automated checks (linting, unit tests, security scan) run and the PR requires at least two approvals (one peer, one WC DataOps) before merge is permitted |
| S7-F1-AC05 | A pull request is merged to develop | the merge commit is inspected | it uses a squash merge with a conventional commit message format that references the work item |

### Technical Notes
- Water Corporation GitHub organisation is the version control platform.
- WC DataOps team participation in PR reviews is a contractual knowledge transfer requirement per S19.
- Branch protection rules enforce the review process; no bypasses for automated or admin accounts without explicit approval.
- Conventional commit messages (e.g. feat:, fix:, refactor:) support automated changelog generation.

---

## Feature S7-F2: All Platform Assets Deployed as Code

**Description:** Every Databricks asset — jobs, pipelines, clusters, permissions — is declared in version-controlled YAML bundles that deploy identically across environments, so that nothing in the platform exists as a manual, undocumented snowflake.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S7-F2-US01 | Data Engineer | define Databricks assets (jobs, pipelines, notebooks, clusters, permissions) as YAML declarations within a DABs bundle | all platform assets are version-controlled, reviewable, and deployable as code |
| S7-F2-US02 | Data Engineer | use environment-specific variable overrides in DABs (e.g. catalog names, warehouse sizes, compute policies) to deploy the same bundle to Dev, Staging, and Prod | a single codebase deploys correctly to each environment without manual configuration changes |
| S7-F2-US03 | Data Engineer | validate a DABs bundle locally before pushing to CI/CD | I catch configuration errors early without consuming CI/CD pipeline time |
| S7-F2-US04 | Platform Engineer | define permissions and access control within DABs bundles | asset permissions are consistent, auditable, and deployed alongside the assets they govern |
| S7-F2-US05 | Data Engineer | include Lakeflow Declarative Pipeline definitions, Lakeflow Jobs, and Lakeflow Connect ingestion configurations within DABs bundles | all pipeline and orchestration assets are deployed from a single declarative definition |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S7-F2-AC01 | A DABs bundle is defined for a data pipeline | the bundle YAML is inspected | it declares the pipeline, job, compute configuration, permissions, and Unity Catalog target schema as code |
| S7-F2-AC02 | Environment-specific overrides are configured | the same bundle is deployed to Dev and Prod | the Dev deployment targets dev_* catalogs with reduced compute; the Prod deployment targets prod_* catalogs with production compute settings |
| S7-F2-AC03 | A Data Engineer runs `databricks bundle validate` locally | the bundle contains a syntax error | the validation fails with a clear error message identifying the issue before any deployment occurs |
| S7-F2-AC04 | A DABs bundle is deployed to the Dev workspace | the deployed assets are inspected in the Databricks workspace | all jobs, pipelines, and permissions match the YAML definitions exactly |
| S7-F2-AC05 | DABs bundle definitions are stored in the Git repository | a change to a bundle YAML is submitted | the change goes through the standard PR review process before deployment |

### Technical Notes
- DABs is the primary IaC deployment model for all Databricks assets, replacing legacy approaches (dbx, manual CLI scripts) per the Databricks End-to-End Platform wiki.
- DABs supports environment-aware deployment through variable substitution and target environment definitions in databricks.yml.
- Terraform is used complementarily for AWS infrastructure (VPCs, IAM roles, S3 buckets); DABs handles Databricks-specific assets.
- `databricks bundle validate` and `databricks bundle deploy` are the core CLI commands for local validation and deployment.

---

## Feature S7-F3: Every Pipeline Change Validated Automatically Before Deployment

**Description:** Unit tests, data validation, schema checks, integration tests, and security scans run automatically on every code change, so that defects, data quality regressions, and security issues are caught before they reach any environment — not after.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S7-F3-US01 | Data Engineer | write unit tests for transformation logic using pytest with Spark session mocking/testing | I can verify business logic in isolation without needing live data or a running cluster |
| S7-F3-US02 | Data Engineer | write data validation tests that assert row counts, schema conformance, null handling, and referential integrity | data quality issues are detected automatically after each pipeline run |
| S7-F3-US03 | Data Engineer | write integration tests that validate end-to-end pipeline logic across datasets (Bronze → Silver → Gold) | the full data flow is verified against expected outputs before promotion to higher environments |
| S7-F3-US04 | Platform Engineer | enable Data Quality Monitoring (Lakehouse Monitoring) on key tables for automated anomaly detection, freshness tracking, and data profiling | ongoing data health is monitored without manual inspection, and anomalies are surfaced proactively |
| S7-F3-US05 | Platform Engineer | run security and compliance scans on IaC templates and code to detect open permissions, secrets in code, and policy violations | security issues are caught in CI before they reach any environment |
| S7-F3-US06 | Data Engineer | run the full test suite as part of the CI/CD pipeline with clear pass/fail reporting | no code is promoted to a higher environment without passing all automated quality gates |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S7-F3-AC01 | Unit tests are written for a Silver transformation function | pytest is executed | all unit tests pass and test coverage for transformation logic is reported at 80% or above |
| S7-F3-AC02 | Data validation tests are configured for a Base zone table | the pipeline processes new data | validation tests assert expected row count (within tolerance), schema matches the defined specification, and critical columns contain no unexpected nulls |
| S7-F3-AC03 | Integration tests are configured for a pipeline | the end-to-end test suite runs against a test dataset | the Gold output matches expected results for row counts, key metric values, and dimensional relationships |
| S7-F3-AC04 | Data Quality Monitoring is enabled on a product schema table | 7 days of data have been ingested | Data Quality Monitoring generates a profile dashboard showing freshness, completeness, distribution statistics, and any detected anomalies |
| S7-F3-AC05 | Security scanning is configured in the CI pipeline | a PR contains a hardcoded secret (e.g. an AWS access key) | the security scan fails the PR check and identifies the offending file and line number |
| S7-F3-AC06 | The full test suite runs in CI | a test fails | the CI pipeline stops, reports the failure clearly, and the PR cannot be merged until the issue is resolved |

### Technical Notes
- pytest is the standard testing framework; Spark session testing uses pyspark.testing or databricks-connect for local execution.
- Data Quality Monitoring (formerly Lakehouse Monitoring) is a serverless, Unity Catalog-integrated service for ongoing table health tracking.
- Security scans should detect secrets in code, overly permissive IAM policies in IaC templates, and non-compliant configurations.
- Test data for unit and integration tests should be deterministic and version-controlled; production data must not be used directly in tests.

---

## Feature S7-F4: One-Click Promotion from Dev Through Test to Prod

**Description:** A tested release can be promoted from Dev to Test to Prod through a single CI/CD pipeline with manual approval gates, full audit trail, and rollback capability — so that deployments are fast, repeatable, and never require someone to manually configure a workspace.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S7-F4-US01 | Data Engineer | trigger a CI pipeline automatically when I open or update a pull request | my code changes are tested and validated before review |
| S7-F4-US02 | Data Engineer | deploy my changes to the Dev environment automatically after merging to the develop branch | the latest code is always running in Dev for testing and experimentation |
| S7-F4-US03 | Platform Engineer | promote a release from Dev to Test (Staging) via a manual approval step in the CI/CD pipeline | the Staging environment reflects only intentionally promoted releases, enabling controlled integration testing |
| S7-F4-US04 | Platform Engineer | promote a release from Test to Prod via a manual approval step requiring at least two approvers | production deployments are deliberate, auditable, and authorised by responsible parties |
| S7-F4-US05 | Data Engineer | deploy Databricks assets using DABs within the CI/CD pipeline (databricks bundle deploy) | all Databricks assets (jobs, pipelines, permissions) are deployed consistently and automatically, not manually |
| S7-F4-US06 | Platform Engineer | roll back a production deployment to the previous version if a critical issue is detected | production can be restored to a known-good state quickly without manual reconfiguration |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S7-F4-AC01 | A pull request is opened | the CI pipeline runs | linting, unit tests, security scans, and DABs bundle validation all execute and results are reported on the PR |
| S7-F4-AC02 | A PR is merged to the develop branch | the CD pipeline runs | DABs bundles are deployed to the wc-edap-dev workspace targeting dev_* catalogs, and a deployment log is recorded |
| S7-F4-AC03 | A release is tagged for promotion to Staging | the promotion pipeline requires approval | a designated approver must approve in GitHub Actions before the deployment to wc-edap-staging proceeds |
| S7-F4-AC04 | A release is approved for Prod promotion | the production deployment pipeline runs | DABs bundles are deployed to wc-edap-prod targeting prod_* catalogs, and the deployment is recorded with approver identity and timestamp |
| S7-F4-AC05 | A production deployment completes | the deployment audit trail is inspected | it records the Git commit SHA, branch/tag, deploying service principal, approver(s), timestamp, and deployed asset list |
| S7-F4-AC06 | A critical issue is detected in production | a rollback is initiated | the previous DABs bundle version is redeployed to production and services are restored within the target rollback SLA (under 30 minutes) |

### Technical Notes
- GitHub Actions is the CI/CD platform; workflows are defined in the repository's .github/workflows/ directory.
- DABs deployment uses `databricks bundle deploy --target <environment>` within the GitHub Actions workflow.
- Service principals are used for CI/CD deployments; credentials are stored in GitHub Actions secrets.
- Manual approval gates use GitHub Actions environments with required reviewers.
- Rollback is achieved by redeploying the previous Git tag/release using DABs.

---

## Feature S7-F5: Pipeline Dependencies Orchestrated with Automatic Retry and Alerting

**Description:** Data pipelines execute in the correct order with automatic retry on transient failures and immediate alerting on persistent issues, so that end-to-end data processing runs reliably without manual monitoring or intervention.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S7-F5-US01 | Data Engineer | define Lakeflow Jobs with multi-task DAGs chaining ingestion, transformation, and Gold layer refresh | end-to-end data processing executes as a single orchestrated workflow with clear dependency ordering |
| S7-F5-US02 | Data Engineer | configure retry policies (maximum retries, backoff intervals) per task within a Lakeflow Job | transient failures are handled automatically without manual intervention |
| S7-F5-US03 | Data Engineer | configure event-driven triggers (file arrival on S3, table update in Unity Catalog) to start Lakeflow Jobs | pipelines run when data is available rather than on fixed schedules, reducing latency and unnecessary compute |
| S7-F5-US04 | Data Engineer | configure scheduled (cron-based) triggers for jobs that require fixed-interval execution | batch processing jobs run at predictable times aligned with business requirements |
| S7-F5-US05 | Data Engineer | configure notifications (email, Slack, Microsoft Teams) for job success, failure, and SLA breach | the team is alerted promptly to pipeline issues requiring attention |
| S7-F5-US06 | Data Engineer | use branching (if/else) and looping (for each) control flow within Lakeflow Jobs | complex orchestration patterns (conditional processing, iterating over source systems) are handled natively |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S7-F5-AC01 | A Lakeflow Job is defined with three tasks (ingestion → Silver transformation → Gold refresh) | the job is triggered | tasks execute in dependency order; the Silver task starts only after ingestion completes; Gold starts only after Silver completes |
| S7-F5-AC02 | A retry policy is configured (3 retries with exponential backoff) on an ingestion task | the task fails on the first attempt due to a transient error | the task retries up to 3 times with increasing delay; if the third retry succeeds, the job continues; if all retries fail, the job fails and an alert is sent |
| S7-F5-AC03 | A file arrival trigger is configured for an S3 path | a new file is uploaded to the monitored S3 path | the Lakeflow Job is triggered within 5 minutes of file arrival |
| S7-F5-AC04 | A table update trigger is configured | the upstream table is updated by a preceding pipeline | the downstream Lakeflow Job is triggered automatically |
| S7-F5-AC05 | Notifications are configured | a production Lakeflow Job fails | a notification is sent to the configured channel(s) within 5 minutes, including job name, failure reason, and a link to the run |
| S7-F5-AC06 | Lakeflow Jobs are deployed | job run metrics are queried from system tables (system.lakeflow.jobs, system.lakeflow.job_run_timeline) | execution durations, queue times, and task-level metrics are available for performance monitoring and SLA tracking |

### Technical Notes
- Lakeflow Jobs (formerly Databricks Workflows) is the native orchestrator per the Databricks End-to-End Platform wiki.
- Task types include: notebook, Python script, SQL, SDP pipeline, Lakeflow Connect ingestion, dbt, JAR, and AI/BI dashboard refresh.
- Event-driven triggers (file arrival, table update) are preferred over scheduled triggers where data arrival is unpredictable.
- Job definitions should be included in DABs bundles for version-controlled, repeatable deployment (S7-F2).
- System tables provide queryable job run metrics for building operational dashboards and tracking SLAs.

---

## Feature S7-F6: Non-Prod Environments Reflect Production at 10% or Less Cost

**Description:** Dev and Test environments contain realistic, privacy-compliant data that mirrors production schemas and distributions, while cost controls ensure non-production spend stays at or below 10% of production — so that engineers develop against representative data without blowing the budget or exposing real personal information.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S7-F6-US01 | Platform Engineer | implement automated data refresh for Dev and Test environments using masked or sampled datasets from Production | non-production environments contain realistic data for testing while complying with data privacy standards |
| S7-F6-US02 | Platform Engineer | apply PI masking, anonymisation, or synthetic data generation to all personal information before it enters non-production environments | non-production environments never contain unmasked PI, ensuring compliance with the PRIS Act and the access model wiki |
| S7-F6-US03 | Platform Engineer | implement cost controls (reduced compute sizing, shorter retention, minimal optimisation) for non-production environments | ongoing non-production platform costs remain at or below 10% of production costs |
| S7-F6-US04 | Platform Engineer | monitor non-production environment costs using system tables and billing data | cost trends are visible and breaches of the 10% threshold are detected early |
| S7-F6-US05 | Data Engineer | work with representative data in the Dev environment that reflects production schema, volume characteristics, and data distributions | my development and testing produces reliable results that translate correctly to production |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S7-F6-AC01 | The non-prod data refresh process is configured | the refresh runs for a domain | dev_* catalog tables are populated with masked/sampled data that matches the production schema and contains representative data distributions |
| S7-F6-AC02 | PI masking is applied during non-prod refresh | a dev_customer table is inspected | all columns tagged with pi_category values other than none contain masked, anonymised, or synthetic values — no real PI is present |
| S7-F6-AC03 | Cost controls are in place | monthly billing is reviewed for the preceding period | total non-production environment costs (Dev + Staging) are at or below 10% of production environment costs |
| S7-F6-AC04 | Cost monitoring is configured | a non-production cost dashboard is accessed | it displays daily and monthly cost trends for Dev and Staging, compared against the 10% threshold, with alerting configured when costs exceed 8% (early warning) |
| S7-F6-AC05 | Non-prod data sampling is configured | the sample dataset is compared to production | the sample represents no more than 10% of production volume but preserves key data distributions, referential integrity, and edge cases |

### Technical Notes
- Non-production data strategy aligns with ADR-EDP-001 (Development Environment Data Strategy) referenced in the access model wiki.
- Sandbox schemas must not contain unmasked PI; anonymised or synthetic datasets are required per the access model sandbox governance rules.
- Cost control mechanisms: reduced cluster policies, shorter auto-termination, smaller SQL warehouses, minimal predictive optimisation, shorter VACUUM retention.
- System table system.billing.usage is the source for cost attribution and environment cost comparison.
- The 10% non-prod cost target is a contractual requirement from S10.

---

## Feature S7-F7: Consistent Naming and Tagging Across All Assets

**Description:** Every catalog, schema, table, column, job, pipeline, and group follows a documented naming convention, and every asset carries the required governed tags — so that the platform is navigable by convention, searchable by classification, and auditable for compliance without manual inventory exercises.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S7-F7-US01 | Platform Engineer | define and document naming conventions for all EDAP objects: catalogs, schemas, tables, columns, views, volumes, jobs, pipelines, clusters, secret scopes, and groups | all assets follow predictable, consistent naming patterns that support discoverability and governance |
| S7-F7-US02 | Platform Engineer | define and document the tagging strategy covering governed tags (sensitivity, pi_category, soci_critical, domain, data_product_tier, records_class, essential_eight) and operational tags (cost centre, project, team) | all assets are classified consistently for access control, compliance, and cost attribution |
| S7-F7-US03 | Data Engineer | reference a naming convention guide when creating new tables, columns, and pipeline assets | I can create assets that are compliant without needing to remember all convention details |
| S7-F7-US04 | Platform Engineer | enforce naming conventions through automated validation in the CI/CD pipeline | non-compliant names are caught before deployment rather than after |
| S7-F7-US05 | Data Engineer | follow coding standards for notebooks and pipeline code (formatting, documentation, error handling, logging patterns) | code across the platform is readable, maintainable, and consistent regardless of author |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S7-F7-AC01 | Naming conventions are documented | the convention document is reviewed | it specifies patterns for catalogs (<env>_<domain>), schemas (raw/base/curated/product/sandbox), tables (<source>_<entity> for Silver, dim_/fact_/agg_ for Gold), columns (snake_case, business-meaningful), jobs, pipelines, and groups |
| S7-F7-AC02 | Tagging standards are documented | the standard is reviewed | it specifies required tags per object type, allowed values per governed tag, and the process for requesting new tag values |
| S7-F7-AC03 | Naming convention validation is integrated into CI | a PR contains a DABs bundle with a table named CUSTOMER_DIM (violating snake_case and prefix conventions) | the CI check fails with a message identifying the naming violation and the correct pattern (dim_customer) |
| S7-F7-AC04 | Coding standards are documented | the standards document is reviewed | it covers Python code formatting (Black/Ruff), SQL formatting, notebook structure (header, imports, parameters, logic, output), error handling, logging patterns, and documentation requirements |
| S7-F7-AC05 | Tagging standards are enforced | a compliance scan runs across all prod_* catalogs | it reports the percentage of objects with all required tags applied and flags any objects missing mandatory tags for remediation |
| S7-F7-AC06 | Naming and tagging standards are published | an engineer accesses the standards | they are available as a searchable wiki or quick reference guide, integrated into onboarding materials |

### Technical Notes
- Naming conventions align to the medallion architecture wiki (no layer encoding in table names when layers are schemas) and the access model wiki (catalog naming: <env>_<domain>).
- Governed tag taxonomy is defined at the Databricks account level per the access model wiki; only allowed values can be assigned.
- Automated naming validation can be implemented as a pre-commit hook, a CI linting step, or both.
- Coding standards should reference Databricks notebook best practices and be consistent with training materials (S19).
- The dim_/fact_/agg_ prefix convention for Gold tables is specified in the medallion architecture wiki naming section.

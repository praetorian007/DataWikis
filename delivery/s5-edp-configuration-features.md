# S5 – EDP Configuration: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:** EDAP Access Model, Medallion Architecture, Databricks End-to-End Platform, Data Engineering Lifecycle

---

## Feature S5-F1: Workspace Deployment

**Description:** Provision and configure Databricks workspaces for Dev, Test, and Prod environments with account-level setup, networking, and workspace topology aligned to the EDAP access model.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F1-US01 | Platform Engineer | provision three Databricks workspaces (wc-edap-dev, wc-edap-staging, wc-edap-prod) in the AWS Sydney (ap-southeast-2) region | the platform has distinct lifecycle environments for development, testing, and production workloads |
| S5-F1-US02 | Platform Engineer | configure account-level settings including billing, network configuration, and workspace administration | all workspaces operate under a consistent account-level governance model |
| S5-F1-US03 | Platform Engineer | establish workspace-level network security (PrivateLink, VPC peering, security groups) per environment | data never traverses the public internet and each environment is network-isolated |
| S5-F1-US04 | Platform Engineer | configure catalog bindings per workspace (dev_* read-write to Dev, staging_* read-write to Staging, prod_* read-write to Prod, prod_* read-only to Dev/Staging) | workspace isolation enforces that production data cannot be modified from non-production environments |
| S5-F1-US05 | Data Domain Steward | access production data in read-only mode from the development workspace for exploratory analysis | I can investigate data issues without needing production workspace access |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F1-AC01 | The Databricks account is provisioned in AWS Sydney (ap-southeast-2) | workspace deployment is complete | three workspaces (wc-edap-dev, wc-edap-staging, wc-edap-prod) are accessible and respond to API health checks |
| S5-F1-AC02 | All three workspaces are deployed | a user attempts to access a prod_* catalog from wc-edap-dev | the catalog is accessible in read-only mode only; any write operation returns a permission error |
| S5-F1-AC03 | All three workspaces are deployed | a user attempts to access a dev_* catalog from wc-edap-prod | the request is denied because dev_* catalogs are not bound to the production workspace |
| S5-F1-AC04 | Network configuration is applied | connectivity is tested between each workspace and required AWS services (S3, Secrets Manager, CloudWatch) | all connections succeed via private endpoints without traversing the public internet |
| S5-F1-AC05 | Workspace deployment is complete | the deployment configuration is reviewed | all workspace infrastructure is defined as Infrastructure as Code (CloudFormation or Terraform) and stored in source control |

### Technical Notes
- Workspace topology follows the access model wiki: environment workspaces (Dev, Staging, Prod) with optional domain-specific workspaces (e.g. wc-edap-prod-customer) provisioned later if PRIS Act isolation requirements demand it.
- Catalog bindings are configured via the Account Console or Terraform; prod_reference and prod_platform catalogs are read-only from all non-production workspaces.
- All workspace provisioning must be codified in IaC to support repeatable deployment and disaster recovery.

---

## Feature S5-F2: Unity Catalog Configuration

**Description:** Configure the Unity Catalog metastore, create domain catalogs for each environment, and establish the schema structure (raw/base/curated/product/sandbox) aligned to the EDAP medallion architecture and access model.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F2-US01 | Platform Engineer | create a single Unity Catalog metastore in the AWS Sydney region and attach it to all three workspaces | all data assets are governed under one metastore with cross-workspace visibility |
| S5-F2-US02 | Platform Engineer | create domain catalogs following the <env>_<domain> naming convention (e.g. prod_asset, prod_customer, prod_network, prod_operations, prod_workforce, prod_finance, prod_environment, prod_reference, prod_platform) for each environment | data is isolated by domain and environment as the primary unit of governance |
| S5-F2-US03 | Platform Engineer | create schemas within each domain catalog following the medallion structure: raw, base, curated, product, and sandbox | data flows through well-defined layers with clear boundaries for access control and lineage |
| S5-F2-US04 | Platform Engineer | configure Unity Catalog Volumes within each domain catalog for the Landing Zone (e.g. /Volumes/prod_asset/raw/landing/) | file-based ingestion has governed, access-controlled staging areas replacing legacy DBFS mounts |
| S5-F2-US05 | Data Domain Steward | view the catalog, schema, and table structure for my domain in Catalog Explorer | I can understand and manage the data assets under my stewardship |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F2-AC01 | The metastore is created | a user queries the metastore from any of the three workspaces | the metastore is accessible and returns the expected catalog list based on workspace bindings |
| S5-F2-AC02 | Domain catalogs are created for production | the catalog list is inspected | prod_asset, prod_customer, prod_network, prod_operations, prod_workforce, prod_finance, prod_environment, prod_reference, and prod_platform catalogs all exist with correct naming |
| S5-F2-AC03 | Domain catalogs are created for all environments | schemas within prod_asset are listed | the schemas raw, base, curated, product, and sandbox are present |
| S5-F2-AC04 | Corresponding dev_* and staging_* catalogs are created | a count of catalogs is performed | each of the seven data domains plus reference and platform has catalogs for dev, staging, and prod (27 catalogs total) |
| S5-F2-AC05 | Unity Catalog Volumes are configured | a file is uploaded to /Volumes/dev_asset/raw/landing/ | the file is accessible via the Volumes API and visible in Catalog Explorer |
| S5-F2-AC06 | Sandbox schemas are configured | a sandbox object retention policy check is performed | sandbox schemas have a documented 90-day retention policy and storage quota as defined in the access model |

### Technical Notes
- Schema names (raw, base, curated, product, sandbox) map to medallion zones as per the access model wiki: raw = Raw, base = Silver Base, curated = Silver Enriched, product = Gold, sandbox = experimentation.
- Landing Zone is implemented via Unity Catalog Volumes, not as a schema. Volumes reside within the raw schema path.
- Catalog and schema creation should be scripted and version-controlled for repeatability across environments.
- Predictive optimisation should be enabled at the catalog or schema level for managed Delta tables.

---

## Feature S5-F3: Compute Governance

**Description:** Define and enforce cluster policies, configure serverless SQL warehouses, establish compute settings for Lakeflow pipeline execution, and set up instance pools to govern cost, performance, and security across all workloads.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F3-US01 | Platform Engineer | create cluster policies that enforce instance types, autoscaling ranges, automatic termination, and Databricks Runtime versions per workload type | compute costs are controlled and all clusters meet minimum security and governance standards |
| S5-F3-US02 | Platform Engineer | configure serverless SQL warehouses for each environment with appropriate sizing | analysts and BI tools can query data with fast start-up times and pay-for-work-done billing |
| S5-F3-US03 | Platform Engineer | configure compute settings for Lakeflow Declarative Pipeline execution including serverless mode | pipeline compute is right-sized, cost-efficient, and managed without manual cluster provisioning |
| S5-F3-US04 | Platform Engineer | create instance pools for non-serverless compute workloads with appropriate instance types and warm pool sizes | cluster start-up times are reduced and instance costs are optimised through reuse |
| S5-F3-US05 | Data Engineer | select from pre-defined cluster policies when creating compute resources | I am guided toward compliant, cost-effective configurations without needing to understand all infrastructure options |
| S5-F3-US06 | Platform Engineer | enforce that all compute attached to Unity Catalog uses Databricks Runtime 16.4 or above | ABAC policies are enforceable on all compute resources as required by the access model |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F3-AC01 | Cluster policies are defined | a Data Engineer attempts to create a cluster outside policy constraints (e.g. oversized instance type, disabled auto-termination) | the cluster creation is rejected with a clear policy violation message |
| S5-F3-AC02 | Serverless SQL warehouses are configured | an analyst runs a SQL query against a prod_* catalog table | the query executes successfully on serverless compute with start-up time under 15 seconds |
| S5-F3-AC03 | Lakeflow pipeline compute is configured | a Lakeflow Declarative Pipeline is executed in serverless mode | the pipeline runs, scales automatically, and completes without manual compute provisioning |
| S5-F3-AC04 | Instance pools are created | a cluster using a pool is started | the cluster starts within 60 seconds using pre-provisioned instances from the pool |
| S5-F3-AC05 | Cluster policies are in place | the policy definitions are reviewed | each policy enforces Databricks Runtime 16.4+ minimum, auto-termination within 30 minutes of inactivity, and autoscaling with defined min/max worker counts |
| S5-F3-AC06 | Compute governance is configured | system table billing data is queried for the past 7 days | compute costs are attributable by workspace, cluster policy, and job/pipeline |

### Technical Notes
- Cluster policies should be defined per workload type: interactive engineering, scheduled jobs, Lakeflow pipelines, and analyst SQL queries.
- Serverless compute is the default for SQL warehouses, Lakeflow Declarative Pipelines, and jobs where supported.
- Runtime version minimum of 16.4 is mandatory for ABAC enforcement per the access model wiki.
- Spot/on-demand ratios should be specified in cluster policies to optimise cost for non-latency-sensitive workloads.

---

## Feature S5-F4: Secrets and Credential Management

**Description:** Integrate Databricks secret scopes with AWS Secrets Manager to securely manage credentials, API keys, and connection strings used by pipelines and notebooks.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F4-US01 | Platform Engineer | configure Databricks secret scopes backed by AWS Secrets Manager | all credentials are stored centrally in AWS and accessed securely at runtime without hardcoding |
| S5-F4-US02 | Data Engineer | retrieve database connection credentials from a secret scope within a notebook or pipeline | I can connect to source systems securely without embedding credentials in code |
| S5-F4-US03 | Platform Engineer | restrict secret scope access to specific groups and service principals using ACLs | only authorised users and pipelines can access sensitive credentials |
| S5-F4-US04 | Security Analyst | audit which secrets were accessed, by whom, and when | I can verify compliance with credential access policies and detect anomalous usage |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F4-AC01 | AWS Secrets Manager is configured with test credentials | a Databricks secret scope is created and linked to the Secrets Manager backend | the secret scope is accessible from all three workspaces and returns stored secret values |
| S5-F4-AC02 | Secret scope ACLs are configured | an unauthorised user attempts to read a secret | the request is denied with an access error |
| S5-F4-AC03 | Secrets are used in a pipeline | the pipeline execution log is inspected | secret values are redacted in all logs and UI outputs |
| S5-F4-AC04 | Secret access auditing is enabled | secret access events are queried from audit logs | each access event records the user/service principal identity, secret scope, secret key, timestamp, and workspace |
| S5-F4-AC05 | Secret scopes are configured | the configuration is reviewed | all secret scope definitions are managed as IaC and stored in source control (secret values excluded) |

### Technical Notes
- AWS Secrets Manager-backed scopes are preferred over Databricks-backed scopes for centralised credential management and automatic rotation.
- Service principals used for pipeline execution should have dedicated secret scope access separate from interactive user access.
- Secret values must never appear in notebook outputs, pipeline logs, or version-controlled code. Scanning for secrets in code is covered under S7 (DataOps).

---

## Feature S5-F5: Identity and Access Foundation

**Description:** Configure SCIM provisioning from Azure AD / Entra ID to synchronise users and groups at the Databricks account level, establish workspace roles, and create service principals for automated workloads.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F5-US01 | Platform Engineer | configure account-level SCIM provisioning from Entra ID to Databricks | users and groups are automatically synchronised and managed centrally in the identity provider |
| S5-F5-US02 | Platform Engineer | create account-level groups following the naming convention (domain_<domain>_stewards, domain_<domain>_engineers, domain_<domain>_analysts, domain_<domain>_scientists) | group membership maps to the federated domain ownership model and can be granted Unity Catalog privileges |
| S5-F5-US03 | Platform Engineer | assign groups to workspaces based on role (engineers to Dev/Staging, analysts via SQL Warehouses, stewards and service principals to Prod) | workspace access follows the principle of least privilege |
| S5-F5-US04 | Platform Engineer | create service principals for pipeline execution, CI/CD, and administrative automation | automated workloads run with dedicated identities that are auditable and do not depend on individual user accounts |
| S5-F5-US05 | Platform Engineer | disable workspace-level SCIM provisioning and workspace-local groups | identity management is consistent and governed exclusively at the account level as per Databricks best practice |
| S5-F5-US06 | Platform Engineer | create PRIS Act authorised groups (pris_authorised_<category>) and SOCI critical access groups | ABAC policies for PI masking and SOCI row filtering can reference these groups as policy exceptions |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F5-AC01 | SCIM provisioning is configured | a new user is added to the domain_asset_engineers group in Entra ID | the user appears in the corresponding Databricks account-level group within the SCIM sync interval (maximum 40 minutes) |
| S5-F5-AC02 | SCIM provisioning is configured | a user is removed from a group in Entra ID | the user's group membership is removed in Databricks on the next sync cycle |
| S5-F5-AC03 | Groups are assigned to workspaces | a user in domain_asset_analysts attempts to access wc-edap-prod directly | access is denied because analyst groups are not assigned to the production workspace |
| S5-F5-AC04 | Service principals are created | a Lakeflow pipeline runs in production using a service principal | the pipeline executes successfully and all audit logs record the service principal identity |
| S5-F5-AC05 | Workspace-level SCIM is disabled | the workspace SCIM configuration is inspected | no workspace-level SCIM endpoints are active; all identity provisioning operates at the account level |
| S5-F5-AC06 | Domain groups are created | a GRANT statement is executed assigning MANAGE on prod_asset to domain_asset_stewards | the grant succeeds and the steward group can manage access within the prod_asset catalog |

### Technical Notes
- Account-level SCIM only; workspace-level SCIM is explicitly disabled per the access model wiki and Databricks best practice.
- All groups must be account-level groups — workspace-local groups cannot be granted Unity Catalog privileges.
- The MANAGE privilege enables federated domain ownership without granting implicit data access (SELECT).
- Production catalogs and schemas must be owned by groups, never individual users.

---

## Feature S5-F6: Git Integration and Repos

**Description:** Configure GitHub connectivity, establish the branching strategy, and set up Databricks Repos for source-controlled notebook and code development across all workspaces.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F6-US01 | Platform Engineer | configure Git integration between Databricks workspaces and the Water Corporation GitHub organisation | engineers can use Databricks Repos to develop code that is version-controlled in GitHub |
| S5-F6-US02 | Data Engineer | clone a GitHub repository into my Databricks workspace and work on a feature branch | I can develop, test, and commit code changes using standard Git workflows without leaving the Databricks environment |
| S5-F6-US03 | Platform Engineer | configure Git credentials using personal access tokens or OAuth for each workspace | authentication to GitHub is secure and does not require manual credential entry for each session |
| S5-F6-US04 | Platform Engineer | define and document the branching strategy (main, develop, feature/*, release/*, hotfix/*) | all engineers follow a consistent branching model that supports CI/CD promotion through Dev, Test, and Prod |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F6-AC01 | Git integration is configured | an engineer clones a GitHub repository in the wc-edap-dev workspace | the repository is cloned successfully and all branches are visible |
| S5-F6-AC02 | Git integration is configured | an engineer commits and pushes changes from Databricks Repos to GitHub | the commit appears in the GitHub repository with correct author attribution |
| S5-F6-AC03 | Branching strategy is defined | the strategy document is reviewed | it specifies branch naming conventions, merge rules (feature→develop→main), required reviewers, and alignment with the CI/CD promotion path (Dev→Test→Prod) |
| S5-F6-AC04 | Git integration is configured | the production workspace Repos configuration is inspected | the production workspace is configured for read-only Repos access (code is deployed via CI/CD, not edited directly in Prod) |

### Technical Notes
- Databricks Repos enables Git-backed notebooks, Python modules, and SQL files within the workspace.
- The production workspace should not permit direct code editing; all code arrives via CI/CD deployment (covered in S7-F4).
- Git integration supports GitHub, Azure DevOps, GitLab, and Bitbucket; Water Corporation uses GitHub.
- Branching strategy should align with the DABs deployment model detailed in S7-F2.

---

## Feature S5-F7: Monitoring and Alerting Foundation

**Description:** Establish monitoring and alerting infrastructure using AWS CloudWatch, Databricks system tables, and native alerting to provide visibility into platform health, compute usage, pipeline status, and security events.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F7-US01 | Platform Engineer | configure AWS CloudWatch integration for Databricks workspace metrics (cluster utilisation, API latency, error rates) | infrastructure health is visible through WC's existing monitoring tools |
| S5-F7-US02 | Platform Engineer | enable and configure Databricks system tables (audit logs, billing/usage, lineage, job runs, pipeline metrics) | operational metadata is queryable as Delta tables for governance dashboards, cost attribution, and compliance reporting |
| S5-F7-US03 | Platform Engineer | create alerting rules for critical platform events (cluster failures, pipeline failures, excessive compute spend, security anomalies) | the platform team is notified promptly of issues requiring intervention |
| S5-F7-US04 | Platform Engineer | build an initial platform health dashboard using Databricks SQL and system tables | platform health, compute costs, and pipeline status are visible at a glance for the platform team and stakeholders |
| S5-F7-US05 | Security Analyst | receive alerts for suspicious access patterns on restricted or privileged data | potential security incidents are detected and investigated within the target response window |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F7-AC01 | CloudWatch integration is configured | a Databricks cluster runs for 10 minutes | CloudWatch metrics (CPU utilisation, memory, instance count) are populated within 5 minutes of the run |
| S5-F7-AC02 | System tables are enabled | the system.billing.usage table is queried | it returns billing records attributable by workspace, cluster, job, and user for the preceding 24 hours |
| S5-F7-AC03 | System tables are enabled | the system.access.audit table is queried | it returns audit events including data access, privilege changes, and workspace login events |
| S5-F7-AC04 | Alerting rules are configured | a Lakeflow pipeline fails in the production workspace | an alert notification is sent to the configured channel (email, Slack, or Teams) within 5 minutes of the failure |
| S5-F7-AC05 | The platform health dashboard is built | the dashboard is accessed by a platform engineer | it displays current compute spend (daily/weekly trend), active cluster count, pipeline success/failure rates, and audit event volume |
| S5-F7-AC06 | Security alerting is configured | a user queries a table tagged sensitivity=privileged more than 10 times in an hour | a security alert is raised for investigation |

### Technical Notes
- System tables are the primary source for cost attribution, audit, and lineage; they are queried via Databricks SQL and surfaced in AI/BI Dashboards.
- Key system tables: system.billing.usage, system.access.audit, system.access.column_lineage, system.access.table_lineage, system.lakeflow.jobs, system.lakeflow.job_run_timeline.
- CloudWatch provides infrastructure-level monitoring; system tables provide Databricks-specific operational metadata. Both are required for comprehensive observability.
- Alert destinations should include email and at least one team messaging channel (Slack or Microsoft Teams).

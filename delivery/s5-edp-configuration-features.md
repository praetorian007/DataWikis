# S5 – EDP Configuration: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:** EDAP Access Model, Medallion Architecture, Databricks End-to-End Platform, Data Engineering Lifecycle

---

## Feature S5-F1: Three Isolated Environments Ready for Development

**Description:** Engineers can develop, test, and release data pipelines across three fully isolated Databricks workspaces (Dev, Test, Prod), each network-secured and governed so that production data is protected from non-production modification.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F1-US01 | Data Engineer | start building pipelines in a Dev workspace that is fully provisioned, network-secured, and connected to AWS services | I can begin development immediately without waiting for infrastructure setup or raising access requests |
| S5-F1-US02 | Data Engineer | deploy my tested pipeline to a Staging workspace that mirrors Prod configuration | I can validate production-like behaviour before releasing to end users |
| S5-F1-US03 | Data Domain Steward | explore production data in read-only mode from the Dev workspace | I can investigate data issues and answer business questions without needing production workspace access or risking accidental modification |
| S5-F1-US04 | Platform Engineer | verify that all workspace infrastructure is defined as code (CloudFormation or Terraform) | I can rebuild or recover any environment repeatably without manual steps |
| S5-F1-US05 | Security Analyst | confirm that no data traffic between workspaces and AWS services traverses the public internet | I can assure stakeholders that the platform meets Water Corporation's network security requirements |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F1-AC01 | The Databricks account is provisioned in AWS Sydney (ap-southeast-2) | workspace deployment is complete | three workspaces (wc-edap-dev, wc-edap-staging, wc-edap-prod) are accessible and respond to API health checks |
| S5-F1-AC02 | All three workspaces are deployed | a user attempts to write to a prod_* catalog from wc-edap-dev | the write operation returns a permission error; read-only access succeeds |
| S5-F1-AC03 | All three workspaces are deployed | a user attempts to access a dev_* catalog from wc-edap-prod | the request is denied because dev_* catalogs are not bound to the production workspace |
| S5-F1-AC04 | Network configuration is applied | connectivity is tested between each workspace and required AWS services (S3, Secrets Manager, CloudWatch) | all connections succeed via private endpoints without traversing the public internet |
| S5-F1-AC05 | Workspace deployment is complete | the deployment configuration is reviewed | all workspace infrastructure is defined as Infrastructure as Code (CloudFormation or Terraform) and stored in source control |

### Technical Notes
- Workspace topology follows the access model wiki: environment workspaces (Dev, Staging, Prod) with optional domain-specific workspaces (e.g. wc-edap-prod-customer) provisioned later if PRIS Act isolation requirements demand it.
- Catalog bindings are configured via the Account Console or Terraform; prod_reference and prod_platform catalogs are read-only from all non-production workspaces.
- All workspace provisioning must be codified in IaC to support repeatable deployment and disaster recovery.

---

## Feature S5-F2: Domain-Isolated Data Catalogues with Governed Discovery

**Description:** Every data domain has its own isolated catalogue with a clear medallion-layer schema structure, so domain teams can find, manage, and govern their data independently while the metastore provides cross-domain discovery.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F2-US01 | Data Domain Steward | browse my domain's catalogue in Catalog Explorer and see all schemas, tables, and volumes organised by medallion layer | I can understand the full inventory of data assets under my stewardship at a glance |
| S5-F2-US02 | Data Engineer | land files into a governed Volume within my domain's raw schema | file-based ingestion flows into access-controlled staging areas with clear domain ownership |
| S5-F2-US03 | Data Analyst | discover tables across multiple domains from a single metastore | I can find relevant data from any domain without needing separate access to each domain's tools or documentation |
| S5-F2-US04 | Data Engineer | work in a sandbox schema with a documented 90-day retention policy | I can experiment and prototype without affecting governed data layers, knowing temporary assets will be cleaned up automatically |
| S5-F2-US05 | Platform Engineer | verify that catalogue and schema creation is scripted and version-controlled | I can recreate the full catalogue structure across environments repeatably |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F2-AC01 | The metastore is created | a user queries the metastore from any of the three workspaces | the metastore is accessible and returns the expected catalogue list based on workspace bindings |
| S5-F2-AC02 | Domain catalogues are created for production | the catalogue list is inspected | prod_asset, prod_customer, prod_network, prod_operations, prod_workforce, prod_finance, prod_environment, prod_reference, and prod_platform catalogues all exist with correct naming |
| S5-F2-AC03 | Domain catalogues are created for all environments | schemas within prod_asset are listed | the schemas raw, base, curated, product, and sandbox are present |
| S5-F2-AC04 | Corresponding dev_* and staging_* catalogues are created | a count of catalogues is performed | each of the seven data domains plus reference and platform has catalogues for dev, staging, and prod (27 catalogues total) |
| S5-F2-AC05 | Unity Catalog Volumes are configured | a file is uploaded to /Volumes/dev_asset/raw/landing/ | the file is accessible via the Volumes API and visible in Catalog Explorer |
| S5-F2-AC06 | Sandbox schemas are configured | a sandbox object retention policy check is performed | sandbox schemas have a documented 90-day retention policy and storage quota as defined in the access model |

### Technical Notes
- Schema names (raw, base, curated, product, sandbox) map to medallion zones as per the access model wiki: raw = Raw, base = Silver Base, curated = Silver Enriched, product = Gold, sandbox = experimentation.
- Landing Zone is implemented via Unity Catalog Volumes, not as a schema. Volumes reside within the raw schema path.
- Catalogue and schema creation should be scripted and version-controlled for repeatability across environments.
- Predictive optimisation should be enabled at the catalogue or schema level for managed Delta tables.

---

## Feature S5-F3: Right-Sized Compute Available on Demand

**Description:** Engineers and analysts get the right compute for their workload — interactive clusters, serverless SQL, or pipeline execution — without needing to understand infrastructure details, while policies enforce cost and governance guardrails automatically.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F3-US01 | Data Engineer | select from pre-defined cluster policies when creating compute resources | I am guided toward compliant, cost-effective configurations without needing to understand all infrastructure options |
| S5-F3-US02 | Data Analyst | run a SQL query and get results in under 15 seconds without provisioning anything | I can explore data interactively with minimal wait time and pay only for work done |
| S5-F3-US03 | Data Engineer | execute a Lakeflow Declarative Pipeline without manually provisioning or sizing compute | pipeline compute scales automatically and I can focus on transformation logic, not infrastructure |
| S5-F3-US04 | Platform Engineer | attribute compute costs by workspace, cluster policy, and job/pipeline | I can identify cost drivers and ensure each workload type stays within budget |
| S5-F3-US05 | Platform Engineer | enforce that all compute uses Databricks Runtime 16.4 or above | ABAC policies are enforceable on all compute resources as required by the access model |
| S5-F3-US06 | Data Engineer | start a cluster within 60 seconds using pre-provisioned instances from a pool | I can begin interactive work quickly without waiting for cold-start instance provisioning |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F3-AC01 | Cluster policies are defined | a Data Engineer attempts to create a cluster outside policy constraints (e.g. oversized instance type, disabled auto-termination) | the cluster creation is rejected with a clear policy violation message |
| S5-F3-AC02 | Serverless SQL warehouses are configured | an analyst runs a SQL query against a prod_* catalogue table | the query executes successfully on serverless compute with start-up time under 15 seconds |
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

## Feature S5-F4: Secrets Securely Available to Pipelines Without Hardcoding

**Description:** Data engineers can connect to source systems and external services from their pipelines by referencing named secrets, without ever seeing, storing, or hardcoding credentials in code, notebooks, or logs.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F4-US01 | Data Engineer | retrieve database connection credentials from a secret scope within a notebook or pipeline | I can connect to source systems securely without embedding credentials in code |
| S5-F4-US02 | Data Engineer | reference a named secret in my pipeline code and have it resolved at runtime | my code is portable across environments and never contains sensitive values |
| S5-F4-US03 | Security Analyst | audit which secrets were accessed, by whom, and when | I can verify compliance with credential access policies and detect anomalous usage |
| S5-F4-US04 | Platform Engineer | restrict secret scope access to specific groups and service principals using ACLs | only authorised users and pipelines can access sensitive credentials |

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

## Feature S5-F5: Users Provisioned Automatically from Corporate Identity

**Description:** When someone is added to a domain group in Entra ID, they automatically receive the correct Databricks access — right workspace, right catalogue permissions, right role — without any manual provisioning steps.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F5-US01 | Data Domain Steward | add a new team member to my domain group in Entra ID and have them appear in Databricks with the correct permissions within 40 minutes | I can onboard team members without raising infrastructure tickets or waiting for manual provisioning |
| S5-F5-US02 | Platform Engineer | manage all identity and group membership exclusively through Entra ID | there is a single source of truth for who has access to what, and deprovisioning happens automatically when someone leaves a group |
| S5-F5-US03 | Platform Engineer | assign domain steward groups the MANAGE privilege on their catalogues | domain teams can independently manage access within their own data without escalating to the platform team |
| S5-F5-US04 | Platform Engineer | create service principals for pipeline execution, CI/CD, and administrative automation | automated workloads run with dedicated, auditable identities that do not depend on individual user accounts |
| S5-F5-US05 | Platform Engineer | create PRIS Act authorised groups and SOCI critical access groups | ABAC policies for PI masking and SOCI row filtering can reference these groups as policy exceptions |
| S5-F5-US06 | Security Analyst | verify that no workspace-level SCIM or workspace-local groups exist | I can confirm that all identity governance operates at the account level as required by Databricks best practice |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S5-F5-AC01 | SCIM provisioning is configured | a new user is added to the domain_asset_engineers group in Entra ID | the user appears in the corresponding Databricks account-level group within the SCIM sync interval (maximum 40 minutes) |
| S5-F5-AC02 | SCIM provisioning is configured | a user is removed from a group in Entra ID | the user's group membership is removed in Databricks on the next sync cycle |
| S5-F5-AC03 | Groups are assigned to workspaces | a user in domain_asset_analysts attempts to access wc-edap-prod directly | access is denied because analyst groups are not assigned to the production workspace |
| S5-F5-AC04 | Service principals are created | a Lakeflow pipeline runs in production using a service principal | the pipeline executes successfully and all audit logs record the service principal identity |
| S5-F5-AC05 | Workspace-level SCIM is disabled | the workspace SCIM configuration is inspected | no workspace-level SCIM endpoints are active; all identity provisioning operates at the account level |
| S5-F5-AC06 | Domain groups are created | a GRANT statement is executed assigning MANAGE on prod_asset to domain_asset_stewards | the grant succeeds and the steward group can manage access within the prod_asset catalogue |

### Technical Notes
- Account-level SCIM only; workspace-level SCIM is explicitly disabled per the access model wiki and Databricks best practice.
- All groups must be account-level groups — workspace-local groups cannot be granted Unity Catalog privileges.
- The MANAGE privilege enables federated domain ownership without granting implicit data access (SELECT).
- Production catalogues and schemas must be owned by groups, never individual users.
- Group naming convention: domain_<domain>_stewards, domain_<domain>_engineers, domain_<domain>_analysts, domain_<domain>_scientists.

---

## Feature S5-F6: Code Changes Flow from IDE to Databricks via Git

**Description:** Engineers can develop pipeline code in Databricks or their local IDE, commit to GitHub, and have their changes version-controlled and reviewable — with production locked to read-only so that all code arrives via CI/CD.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F6-US01 | Data Engineer | clone a GitHub repository into my Databricks workspace and work on a feature branch | I can develop, test, and commit code changes using standard Git workflows without leaving the Databricks environment |
| S5-F6-US02 | Data Engineer | commit and push changes from Databricks Repos back to GitHub with correct author attribution | my work is version-controlled and visible to reviewers immediately |
| S5-F6-US03 | Data Engineer | follow a documented branching strategy (main, develop, feature/*, release/*, hotfix/*) | code changes flow through a predictable lifecycle with clear merge rules aligned to the CI/CD promotion path |
| S5-F6-US04 | Platform Engineer | verify that the production workspace has read-only Repos access | no one can edit code directly in production; all code arrives via CI/CD deployment |

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

## Feature S5-F7: Platform Health Visible and Alertable in Real Time

**Description:** The platform team and stakeholders can see compute costs, pipeline status, and security events at a glance on a live dashboard, and receive automatic alerts when something goes wrong or looks anomalous.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S5-F7-US01 | Platform Engineer | open a dashboard and see current compute spend, active cluster count, pipeline success/failure rates, and audit event volume | I can assess platform health at a glance without running ad-hoc queries |
| S5-F7-US02 | Platform Engineer | receive an alert within 5 minutes when a production pipeline fails | I can investigate and resolve issues before they impact downstream consumers |
| S5-F7-US03 | Platform Engineer | query system tables for billing, audit, lineage, and job run data as standard Delta tables | I can build custom reports, track trends, and answer governance questions using SQL |
| S5-F7-US04 | Security Analyst | receive alerts for suspicious access patterns on restricted or privileged data | potential security incidents are detected and investigated within the target response window |
| S5-F7-US05 | Platform Engineer | see infrastructure metrics (CPU utilisation, memory, instance count) in CloudWatch alongside Water Corporation's existing monitoring | Databricks infrastructure health is visible through the organisation's standard monitoring tools |

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

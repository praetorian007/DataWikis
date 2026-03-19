# EDAP Access Model — Databricks Unity Catalog & Federated Domain Governance

**Mark Shaw** | Principal Data Architect

---

## 1. Purpose

This document defines Water Corporation's access model for the Enterprise Data & Analytics Platform (EDAP), built on Databricks Unity Catalog. It describes how workspaces, catalogs, identity, and attribute-based access control (ABAC) are structured to support a federated, domain-based data governance model across WC's seven data domains (Customer, Asset, Operations, Finance, Legal & Compliance, People, and Technology & Digital).

This wiki should be read alongside:

- **EDAP-FWK-001** — Pipeline Framework Requirements Specification
- **EDAP Tagging Strategy** — Four-layer governed tag taxonomy for Unity Catalog (authoritative source for all tag definitions)
- **Domain Governance Across Systems** — Three-layer governance architecture, data contracts, Delta Sharing governance, and Lakehouse Federation governance
- **Databricks End-to-End Platform** — Platform capabilities reference covering Lakeflow, Unity Catalog, Mosaic AI, and AI/BI
- **Medallion Architecture** — Zone decomposition and data flow conventions across Bronze, Silver, and Gold layers
- **Data Governance Lifecycle** — End-to-end governance lifecycle from classification through compliance

---

## 2. Guiding Principles

The access model is grounded in the following principles, established through WC's enterprise data governance framework:

| Principle | Description |
|---|---|
| **Open by default, restricted by exception** | All data is accessible unless a classification assessment determines otherwise. Restrictions are applied through governed tags and ABAC policies, not blanket denial. |
| **Explicit classification per object** | Every table, schema, and catalog receives a deliberate classification assessment. Classification is never automatically inherited from parent objects — it must be explicitly set. |
| **Personal Information (PI) terminology** | WC uses the term Personal Information (PI) per the PRIS Act 2024, not PII. All tagging, policies, and documentation must reflect this. |
| **Four-level access model** | Data access is categorised as **Open**, **Controlled**, **Restricted**, or **Privileged**, aligned to WAICP (WA Information Classification Policy) and WC's data sensitivity framework. |
| **Federated domain ownership** | Each of WC's seven data domains has an executive owner who governs data within their domain, operating within guardrails set by the central Architecture & Strategy function. |
| **Least privilege** | Users receive the minimum access required for their role. Unity Catalog enforces this natively. |
| **Define once, secure everywhere** | Access policies are defined centrally in Unity Catalog and enforced consistently across all workspaces. |

---

## 3. Architecture Overview

### 3.1 Single Metastore

WC operates a **single Unity Catalog metastore** in the AWS Sydney (ap-southeast-2) region. This metastore serves all EDAP workspaces. Databricks recommends one metastore per region — it is not intended as a unit of data isolation, but as a regional boundary for metadata.

All data and AI assets cataloged in this metastore are accessible from any workspace attached to it, subject to catalog bindings, privilege grants, and ABAC policies.

### 3.2 Workspace Topology

Workspaces are separated along two axes: **environment lifecycle** and **domain/team boundary**.

#### Environment Workspaces

| Workspace | Purpose | Catalog Bindings |
|---|---|---|
| `wc-edap-dev` | Development, experimentation, feature engineering | `dev_*` catalogs (read-write), `prod_*` catalogs (read-only, select domains) |
| `wc-edap-staging` | Integration testing, QA, pre-production validation | `staging_*` catalogs (read-write), `prod_*` catalogs (read-only) |
| `wc-edap-prod` | Production workloads, scheduled pipelines, serving | `prod_*` catalogs (read-write) |

#### Domain-Specific Workspaces (Where Required)

For domains with strict isolation requirements, particularly Customer data subject to the PRIS Act 2024, a dedicated workspace may be provisioned. This provides an additional layer of network and compute isolation beyond what catalog bindings alone deliver.

| Workspace | Justification |
|---|---|
| `wc-edap-prod-customer` | PRIS Act 2024 requirements for PI processing. Dedicated compute with fine-grained access control. Catalog binding restricted to customer-related schemas across the layer catalogs (e.g. `prod_silver.customer_enriched`, `prod_gold.customer_bi`) and approved cross-domain reference catalogs only. |

> **Decision criteria for domain-specific workspaces:** A separate workspace is warranted only when regulatory, network isolation, or compute isolation requirements cannot be met through catalog bindings and ABAC alone. Most domains will operate within the shared environment workspaces.

### 3.3 Layer-Based Catalog Layout

Catalogs follow a `<env>_<layer>` naming convention, aligned with the medallion architecture. Catalogs represent the **primary unit of environment and layer isolation**, while domain governance is enforced at schema level via the MANAGE privilege.

#### Production Catalogs

| Catalog | Layer | Owner |
|---|---|---|
| `prod_bronze` | Bronze | Architecture & Strategy |
| `prod_silver` | Silver | Architecture & Strategy |
| `prod_gold` | Gold | Architecture & Strategy |
| `prod_reference` | Cross-domain reference data | Architecture & Strategy |
| `prod_platform` | EDAP platform metadata, audit, lineage | Architecture & Strategy |

Each layer also has corresponding `dev_<layer>` and `staging_<layer>` catalogs (e.g. `dev_bronze`, `dev_silver`, `staging_gold`).

#### Schema Structure Within Each Catalog

Schemas within each layer catalog align to the EDAP medallion architecture zones. Domain and source system boundaries are expressed at the schema level:

| Catalog | Schema Pattern | Zone | Example |
|---|---|---|---|
| `prod_bronze` | `<source_system>_raw` | Raw | `sap_raw`, `maximo_raw`, `scada_raw` |
| `prod_silver` | `<source_system>_base` | Base | `sap_base`, `maximo_base` |
| `prod_silver` | `<domain>_enriched` | Enriched | `asset_enriched`, `customer_enriched` |
| `prod_gold` | `<domain>_exploratory` | Exploratory | `asset_exploratory`, `operations_exploratory` |
| `prod_gold` | `<domain>_bi` | BI | `asset_bi`, `customer_bi`, `finance_bi` |
| `edap_sandbox` | `<user_or_team>` | Sandbox | `jsmith`, `asset_analytics_team` |

> **Landing Zone implementation:** The Landing Zone is not a schema but is implemented using **Unity Catalog Volumes** within the Bronze catalog. Files from source systems are staged in managed or external Volumes (e.g. `/Volumes/prod_bronze/landing/sap/`) before being ingested into Raw zone schema tables via Auto Loader or Lakeflow Connect. Volumes provide governed, access-controlled file storage that replaces legacy DBFS mounts.

#### Sandbox Governance

The `edap_sandbox` catalog provides a dedicated space for experimentation, with user-specific or team-specific schemas. It is subject to the following guardrails:

- **Retention policy:** Objects in sandbox schemas are subject to a 90-day retention policy. Objects older than 90 days are flagged for deletion and removed after a 14-day grace period unless explicitly extended by the domain steward.
- **Read access to production:** Sandbox users may be granted read-only access to Silver and Gold schemas within their domain to enable experimentation against real data. This access is governed by the same ABAC policies as any other query.
- **Storage quotas:** Each sandbox schema has a storage quota (set via cluster policies and monitoring) to prevent runaway experimentation from consuming excessive storage. Quotas are reviewed quarterly.
- **Promotion path:** Successful sandbox experiments that should become production assets must follow the standard pipeline promotion path: code is committed to source control, deployed via DABs through dev, staging, and prod, and outputs land in the appropriate medallion schema. Direct promotion from sandbox to production is not permitted.
- **No PI in sandbox:** Sandbox schemas must not contain unmasked Personal Information. If PI is required for experimentation, anonymised or synthetic datasets must be used.

#### Schema-to-Medallion Zone Mapping

The following table provides an explicit mapping between EDAP schema patterns and the medallion architecture zones described in the companion Medallion Architecture document:

| Medallion Zone | Catalog | Schema Pattern | Implementation |
|---|---|---|---|
| Landing Zone | `prod_bronze` | *(Unity Catalog Volumes)* | Files land in managed or external Volumes within the Bronze catalog (e.g. `/Volumes/prod_bronze/landing/sap/`). Not persisted as tables. |
| Raw | `prod_bronze` | `<source_system>_raw` | Append-only Delta tables preserving source data as received. |
| Base | `prod_silver` | `<source_system>_base` | Cleansed, typed, conformed data with SCD and `edap_hash` change detection. |
| Enriched | `prod_silver` | `<domain>_enriched` | Business logic applied, cross-source joins, conformed dimensions and facts. |
| Exploratory | `prod_gold` | `<domain>_exploratory` | Wide, denormalised datasets for ad hoc analysis and data science. |
| BI | `prod_gold` | `<domain>_bi` | Certified data products published via data contracts. Dimensional models. |
| Sandbox | `edap_sandbox` | `<user_or_team>` | Isolated experimentation space. Not promoted to production. |

---

## 4. Federated Ownership Model

### 4.1 Central vs. Domain Responsibilities

WC's federated model divides governance responsibilities between the central Architecture & Strategy function and domain teams.

#### Central Team (Architecture & Strategy)

- Metastore administration and configuration
- Account-level identity provisioning (SCIM)
- Catalog creation and lifecycle management
- Governed tag definitions (account-level tag policies)
- ABAC policy authorship for cross-cutting concerns (PI masking, SOCI classification)
- Platform catalog (`prod_platform`) ownership
- Reference data catalog (`prod_reference`) ownership
- Data product certification standards and the FAUQD test
- Audit log monitoring and compliance reporting

#### Domain Teams (via Executive Owner / Data Steward Group)

- Schema and table creation within their domain schemas
- Grant management within their domain (via `MANAGE` privilege at schema level)
- Data contract authorship and maintenance
- Classification assessment of domain objects (applying governed tags)
- Data product development, testing, and promotion
- Domain-specific ABAC policy proposals (implemented by central team)

### 4.2 The MANAGE Privilege

The `MANAGE` privilege is the key mechanism enabling federated ownership in Unity Catalog. It is granted at schema level, allowing domain teams to:

- Manage grants on objects within their schemas (grant/revoke `SELECT`, `MODIFY`, etc.)
- Drop and alter objects (schema evolution)
- Handle day-to-day governance without requiring central team involvement

Critically, `MANAGE` does **not** implicitly grant `SELECT` or other data access privileges — these must be explicitly assigned. This ensures that a domain steward can administer access without necessarily being able to read all data in their domain (important for PI-restricted datasets).

**Implementation:**

```sql
-- Central team grants MANAGE on domain schemas to domain steward group
GRANT MANAGE ON SCHEMA prod_silver.asset_base TO `domain_asset_stewards`;
GRANT MANAGE ON SCHEMA prod_silver.asset_enriched TO `domain_asset_stewards`;
GRANT MANAGE ON SCHEMA prod_gold.asset_bi TO `domain_asset_stewards`;
GRANT MANAGE ON SCHEMA prod_gold.asset_exploratory TO `domain_asset_stewards`;

-- Domain steward can then independently manage access
GRANT SELECT ON SCHEMA prod_gold.asset_bi TO `asset_analysts`;
GRANT USE SCHEMA ON SCHEMA prod_gold.asset_bi TO `asset_analysts`;
```

### 4.3 Ownership Rules

- **Production catalogs and schemas** must be owned by groups, never individual users.
- **Catalog ownership** is held by `edap_platform_admins` (since catalogs are layer-based, not domain-specific).
- **Schema ownership** is held by domain steward groups (e.g. `domain_asset_stewards` owns `prod_silver.asset_base`, `prod_gold.asset_bi`, etc.).
- **Platform and reference catalogs** are owned by the `edap_platform_admins` group.
- **Service principals** are used for pipeline execution and own the objects created by automated processes.

---

## 5. Identity & Access Management

### 5.1 Account-Level Identity Provisioning

All identity management is performed at the Databricks account level via SCIM synchronisation from WC's identity provider (Azure AD / Entra ID).

- **No workspace-level SCIM provisioning.** This is explicitly disabled.
- **All groups are account-level groups**, not workspace-local groups. Workspace-local groups cannot be granted Unity Catalog privileges and must not be used.
- **Group membership is managed in Azure AD** and synchronised automatically.

### 5.2 Group Structure

Groups are structured to reflect both domain membership and functional role:

| Group Pattern | Example | Purpose |
|---|---|---|
| `domain_<domain>_stewards` | `domain_asset_stewards` | Data stewardship and governance for the domain. Receives `MANAGE` on domain schemas. |
| `domain_<domain>_engineers` | `domain_asset_engineers` | Data engineering within the domain. `MODIFY` on raw/base/enriched schemas. |
| `domain_<domain>_analysts` | `domain_asset_analysts` | Analytical consumers. `SELECT` on enriched/exploratory/bi schemas. |
| `domain_<domain>_scientists` | `domain_customer_scientists` | Data science and ML workloads. Broader read access plus model registry permissions. |
| `edap_platform_admins` | — | Central platform team. Account admin and metastore configuration. |
| `edap_platform_engineers` | — | Central engineering team. Pipeline framework development. |
| `pris_authorised_<category>` | `pris_authorised_billing` | Users explicitly authorised to access specific categories of PI under the PRIS Act. Used in ABAC policy exceptions. |
| `soci_critical_access` | — | Users authorised to access SOCI Act critical infrastructure data. |

Domain group examples across the seven domains:

| Domain | Stewards | Engineers | Analysts |
|---|---|---|---|
| Asset | `domain_asset_stewards` | `domain_asset_engineers` | `domain_asset_analysts` |
| Customer | `domain_customer_stewards` | `domain_customer_engineers` | `domain_customer_analysts` |
| Operations | `domain_operations_stewards` | `domain_operations_engineers` | `domain_operations_analysts` |
| Finance | `domain_finance_stewards` | `domain_finance_engineers` | `domain_finance_analysts` |
| Legal & Compliance | `domain_legal_compliance_stewards` | `domain_legal_compliance_engineers` | `domain_legal_compliance_analysts` |
| People | `domain_people_stewards` | `domain_people_engineers` | `domain_people_analysts` |
| Technology & Digital | `domain_technology_digital_stewards` | `domain_technology_digital_engineers` | `domain_technology_digital_analysts` |

### 5.3 Workspace Access

Groups are assigned to workspaces based on their role:

- **Production workspace:** Domain stewards, production service principals, platform admins. Analysts access production data via SQL Warehouses, not direct workspace access where possible.
- **Development workspace:** Engineers, scientists, and analysts who need to develop or test.
- **Staging workspace:** Engineers and service principals performing integration testing.

---

## 6. Attribute-Based Access Control (ABAC)

### 6.1 Overview

ABAC is WC's primary mechanism for enforcing fine-grained, classification-driven access control at scale. It complements the privilege model (GRANT/REVOKE) by allowing policies to be defined based on **governed tags** applied to data assets.

ABAC is now available in Public Preview on AWS and is Databricks' recommended approach for centralised, scalable governance — replacing the previous practice of applying row filters and column masks individually per table.

### 6.2 Governed Tags

Governed tags are defined at the Databricks account level using tag policies. Only the values specified in the tag policy can be assigned — this prevents inconsistent or ad-hoc classification.

#### WC Governed Tag Taxonomy

| Tag Key | Layer | Allowed Values | Source Framework | Applied To |
|---|---|---|---|---|
| `waicp_classification` | 1 | `UNOFFICIAL`, `OFFICIAL`, `OFFICIAL:Sensitive`, `OFFICIAL:Sensitive-Personal`, `OFFICIAL:Sensitive-Commercial`, `OFFICIAL:Sensitive-Legal`, `OFFICIAL:Sensitive-Cabinet` | WAICP | Tables, views, schemas, catalogs |
| `pi_contained` | 2 | `true`, `false` | PRIS Act 2024 | Tables, columns |
| `pi_type` | 2 | `direct_identifier`, `indirect_identifier`, `sensitive_pi` | PRIS Act 2024 | Columns (where `pi_contained = true`) |
| `regulatory_scope` | 2 | `pris_act`, `soci_act`, `state_records_act`, `privacy_act`, `foi_act`, `none` | Multiple | Tables |
| `sensitivity_type` | 2 | `personal_information`, `commercial_in_confidence`, `legal_privilege`, `infrastructure_vulnerability`, `operational_security`, `geospatial_sensitive`, `indigenous_cultural`, `financial_sensitive`, `environmental_sensitive` | WAICP sublabels / WC extensions | Tables, columns |
| `pi_lawful_basis` | 2 | `consent`, `legitimate_interest`, `legal_obligation`, `vital_interest`, `public_interest`, `contractual_necessity`, `not_assessed` | Privacy Act 1988 (Cth) / PRIS Act 2024 | Tables (where `pi_contained = true`) |
| `access_model` | 3 | `open`, `controlled`, `restricted`, `privileged` | WAICP / WC Data Classification | Tables, schemas |
| `masking_required` | 3 | `none`, `partial`, `full`, `hash`, `redact` | PRIS Act 2024 / WC PI Framework | Columns (where `pi_type` set) |
| `sharing_permitted` | 3 | `internal_only`, `interagency`, `delta_sharing`, `public` | WC Sharing Framework | Tables, schemas |
| `soci_critical` | 4 | `true`, `false` | SOCI Act 2018 (incl. 2024 amendments) | Schemas, tables |
| `data_domain` | 4 | `asset`, `customer`, `operations`, `finance`, `legal_compliance`, `people`, `technology_digital`, `enterprise_reference` | WC Domain Model | Schemas, tables |
| `quality_tier` | 4 | `certified`, `provisional`, `uncertified` | EDAP Data Products Framework | Tables (Silver, Gold) |
| `model_risk_tier` | — | `high`, `medium`, `low` | ISO/IEC 42001:2023 / NIST AI RMF | Models, endpoints |
| `ai_governance_level` | — | `autonomous`, `human_in_loop`, `human_on_loop`, `human_in_command` | Voluntary AI Safety Standard (2024) | Models, endpoints, agents |

> **Important:** Tags are assessed and applied explicitly per object. A table in the `prod_silver` catalog is not automatically tagged `pi_contained=true` — it must be assessed and tagged based on its actual content. The complete tag taxonomy — including all four layers, allowed values, inheritance rules, and change management procedures — is defined in the companion **EDAP Tagging Strategy**. The table above shows the access-relevant subset; the tagging strategy is the authoritative source.

### 6.3 ABAC Policies

Policies are defined at the highest applicable level (typically the catalog) and inherit downward to all child schemas and tables. WC defines the following policy categories:

#### PI Masking Policy

Applied to all production catalogs. Masks columns tagged with any `pi_type` value (`direct_identifier`, `indirect_identifier`, `sensitive_pi`), unless the querying user is a member of the corresponding `pris_authorised_<category>` group. The `masking_required` tag on each column determines the masking behaviour (`full`, `partial`, `hash`, or `redact`). Where a `pi_lawful_basis` tag is present on the parent table, the policy also validates that the user's access purpose aligns with the documented lawful basis — supporting compliance with the Privacy Act 1988 (Cth) reform programme's automated decision-making obligations and the Notifiable Data Breaches scheme (Part IIIC).

```sql
-- Example: Mask PI columns unless user is in authorised group
-- UDF defined in prod_platform.governance schema
CREATE OR REPLACE FUNCTION prod_platform.governance.mask_pi(value STRING)
RETURNS STRING
RETURN CASE
  WHEN is_account_group_member('pris_authorised_billing') THEN value
  ELSE '***MASKED***'
END;
```

The ABAC policy is then attached at the catalog level, matching columns tagged with `pi_type` and applying the appropriate masking UDF based on the `masking_required` tag. All current and future tables within that catalog are automatically covered.

#### SOCI Critical Row Filter

For tables tagged `soci_critical=true`, a row filter policy ensures only members of `soci_critical_access` can query the data. This supports compliance with SOCI Act 2018 requirements including the 2024 SOCI Rules amendments, which expand positive security obligations for critical infrastructure entities.

#### Domain Isolation Policy

While catalog bindings and workspace bindings provide the primary isolation, ABAC policies can provide an additional layer — for example, ensuring that cross-domain reference data shared via `prod_reference` respects the originating domain's classification.

### 6.4 ABAC and the Development Lifecycle

ABAC policies on production catalogs apply regardless of which workspace the query originates from. When developers access production data in read-only mode from the development workspace (via catalog binding), the same masking and filtering policies are enforced. This ensures PI and SOCI protections are maintained even during development.

For development catalogs using anonymised or synthetic data, ABAC policies may be relaxed since the underlying data does not contain real PI.

---

## 7. Catalog Binding & Workspace Isolation

### 7.1 Binding Rules

| Catalog Pattern | Bound To | Access Mode |
|---|---|---|
| `prod_*` | `wc-edap-prod` | Read-Write |
| `prod_*` (selected) | `wc-edap-dev`, `wc-edap-staging` | Read-Only |
| `dev_*` | `wc-edap-dev` | Read-Write |
| `staging_*` | `wc-edap-staging` | Read-Write |
| `prod_reference` | All workspaces | Read-Only (except in prod) |
| `prod_platform` | All workspaces | Read-Only (except in prod) |

### 7.2 Binding Implementation

```sql
-- Bind production layer catalogs to production workspace only (read-write)
ALTER CATALOG prod_bronze SET OWNER TO `edap_platform_admins`;
ALTER CATALOG prod_silver SET OWNER TO `edap_platform_admins`;
ALTER CATALOG prod_gold SET OWNER TO `edap_platform_admins`;
-- Workspace binding configured via Account Console or Terraform

-- Bind production layer catalogs to dev workspace as read-only
-- (configured via Account Console — catalog binding with READ_ONLY mode)
```

> **Note:** Catalog bindings override user-level permissions. Even if a user has `SELECT` on a table, they cannot access it from an unbound workspace.

---

## 8. Lakehouse Federation Access Governance

Lakehouse Federation enables Unity Catalog to query external data sources (SQL Server, PostgreSQL, MySQL, Snowflake, BigQuery, Amazon Redshift, and others) via **connections** and **foreign catalogs**. The access model for federated data sources follows these rules:

- **Connection creation** is restricted to `edap_platform_admins` only. Connections store credentials for external systems and must be managed centrally.
- **Foreign catalogs** follow the naming convention `<env>_<source>_federated` (e.g. `prod_sqlserver_federated`), using the `_federated` suffix defined in the EDAP Tagging Strategy to distinguish federated assets from native lakehouse data.
- **Workspace binding** applies to foreign catalogs in the same way as native catalogs — foreign catalogs for production external systems are bound to the production workspace only.
- **ABAC policies** apply to federated tables. Where external source data contains PI or SOCI-classified information, the relevant governed tags must be applied to the foreign catalog objects and ABAC policies will enforce masking and filtering at query time.
- **Privilege grants** on foreign catalog schemas follow the same domain steward delegation model as native schemas — `MANAGE` is granted to the relevant domain steward group.
- **Lineage** is captured for federated queries, providing visibility into cross-platform data flows.

> **Important:** Federated queries execute against live external systems. Query governance is enforced by Unity Catalog, but performance and availability depend on the external source. Federated access should complement, not replace, the standard ingestion-to-medallion pipeline for core analytical workloads.

See the companion **Domain Governance Across Systems** document for the broader governance framework applied to Lakehouse Federation.

---

## 9. External Locations & Storage

- `CREATE EXTERNAL LOCATION` is restricted to `edap_platform_admins` only.
- Domain teams access data through managed tables and volumes — not through direct external location permissions.
- `READ FILES` and `WRITE FILES` are never granted on external locations to end users. All file-based access uses Unity Catalog volumes.
- External locations are bound to specific workspaces where required.

---

## 10. Compute & Access Modes

Unity Catalog requires compute that supports its access control model:

| Compute Type | Access Mode | Use Case |
|---|---|---|
| SQL Warehouse (Serverless) | Shared | Analyst queries, BI tool connections, ad-hoc SQL |
| Shared Cluster | Shared | Multi-user notebooks, lightweight engineering |
| Dedicated Cluster | Single User | ML training, PI-authorised workloads requiring fine-grained access |
| Serverless Compute | Shared | Lakeflow Spark Declarative Pipelines, jobs |
| Model Serving Endpoints | Serverless | Real-time ML inference, AI Gateway |
| Online Tables | Serverless | Low-latency key-value lookups for feature serving |

> **ABAC requirement:** ABAC policies require Databricks Runtime 16.4 or above, or serverless compute. Older runtimes cannot access ABAC-protected tables.

> **Online Tables and Model Serving:** Online Tables provide millisecond-latency key-value lookups for real-time ML feature serving. Access to Online Tables is governed by Unity Catalog privileges on the underlying Delta table. Model Serving endpoints are secured via endpoint-level ACLs and, for external-facing endpoints, via AI Gateway rate limiting and guardrails.

---

## 11. AI/ML Asset Governance

Unity Catalog governs AI and ML assets as first-class securables alongside data assets. The access model for ML assets follows these rules:

### 11.1 Governed Asset Types

| Asset Type | Unity Catalog Object | Governance |
|---|---|---|
| Registered models | `catalog.schema.model` | Versioned, tagged, lineage-tracked. Access via GRANT on the model object. |
| Model serving endpoints | Endpoint ACLs | Per-endpoint access control. Query permissions govern who can invoke the endpoint. |
| Feature tables | Standard Delta tables with PK/timestamp metadata | Governed identically to any Unity Catalog table. |
| Online Tables | Read-only sync from Delta tables | Access governed by privileges on the source Delta table. |
| Vector Search indexes | Unity Catalog objects | Inherit access controls from the source Delta table. |
| AI agents | Logged and served via Model Serving | Governed as registered models with additional `ai_governance_level` tagging. |
| Functions (UDFs) | `catalog.schema.function` | Access via GRANT EXECUTE. Used for ABAC masking and tool-use in agents. |

### 11.2 ML-Specific Group Permissions

| Group Pattern | Permissions |
|---|---|
| `domain_<domain>_scientists` | `SELECT` on Gold/Enriched schemas, `CREATE MODEL` in domain model schemas, `USE` on Feature Store tables, experiment tracking access |
| `edap_ml_engineers` | `MODIFY` on model registry schemas, deploy to Model Serving endpoints, manage AI Gateway configurations |
| `edap_platform_admins` | Create and manage AI Gateway routes, manage model serving endpoint ACLs, configure guardrails |

### 11.3 AI Gateway Access Control

AI Gateway provides centralised governance for all foundation model access (Databricks-hosted and external providers). Access control is enforced at three levels:

- **Route-level ACLs** — Control which teams can access which model providers and model sizes (e.g., restricting large-model access to approved use cases).
- **Rate limiting and token budgets** — Enforce per-team or per-use-case consumption limits to control GenAI costs.
- **Guardrails** — Input/output content filtering, PII detection, and topic restrictions applied consistently across all consuming applications.

All AI Gateway requests are logged to system tables, providing a single audit trail for GenAI usage, cost attribution, and compliance reporting.

### 11.4 ML Asset Tagging

ML assets must be tagged using the governed tag taxonomy:

- `model_risk_tier` — `high`, `medium`, or `low` based on the model's impact and risk profile (aligned to ISO/IEC 42001:2023 and NIST AI RMF)
- `ai_governance_level` — `autonomous`, `human_in_loop`, `human_on_loop`, or `human_in_command` (aligned to the Australian Government Voluntary AI Safety Standard)
- `data_domain` — The business domain the model serves
- `data_product_tier` — The maturity tier of the model as a data product

---

## 12. Audit & Compliance

### 12.1 Audit Logging

Unity Catalog automatically captures user-level audit logs for all data access events. These logs feed into WC's centralised security monitoring solution and support:

- PRIS Act 2024 compliance — tracking who accessed PI and when
- Privacy Act 1988 (Cth) compliance — supporting Notifiable Data Breaches scheme (Part IIIC) obligations by enabling rapid identification of affected records and individuals in the event of a breach; supporting automated decision-making transparency requirements under the reform programme
- SOCI Act 2018 reporting (incl. 2024 SOCI Rules amendments) — monitoring access to critical infrastructure data with expanded positive security obligations
- State Records Act 2000 — demonstrating appropriate records management
- WAICP — validating that access patterns align with information classification levels
- Essential Eight (ACSC) — supporting access control and audit trail requirements

### 12.2 System Tables

Unity Catalog system tables provide queryable access to:

- **Audit logs** — all access and modification events
- **Billable usage** — cost attribution by domain/workspace
- **Lineage** — data flow tracking across the medallion layers
- **Granted privileges** — current access state for compliance reporting

### 12.3 Monitoring

The central platform team monitors for:

- Excessive or suspicious access patterns on restricted/privileged data
- Anomalous create/alter/delete operations on securables
- Classification gaps — objects without `waicp_classification` or `pi_contained` tags
- ABAC policy conflicts or evaluation errors

### 12.4 Break-Glass (Emergency) Access

In incident scenarios requiring urgent access to data outside normal privilege grants, the following break-glass procedure applies:

1. **Request:** The requester raises an emergency access request through the IT service management tool, citing the incident reference and the specific data assets required.
2. **Approval:** A member of `edap_platform_admins` or the relevant domain executive owner approves the request. Two-person approval is required for `privileged`-classified data.
3. **Provisioning:** Temporary access is granted via time-limited group membership (managed through Azure AD / Entra ID with an automatic expiry, typically 4 to 8 hours).
4. **Audit:** All break-glass access events are flagged in the audit log with a dedicated tag. The platform team reviews all break-glass events weekly.
5. **Revocation:** Access is automatically revoked at expiry. Manual early revocation is performed if the incident is resolved sooner.

> **Important:** Break-glass access does not bypass ABAC policies or Unity Catalog governance. It grants temporary membership in an authorised group, ensuring all access is still logged and filtered.

---

## 13. Service Principal Governance

Service principals are machine identities used for automated pipeline execution, ML model serving, embedded dashboard publishing, and AI Gateway access. They are critical infrastructure and must be governed with the same rigour as human identities.

### 13.1 Naming Convention

Service principals follow the pattern `sp_<env>_<purpose>` (e.g. `sp_prod_asset_pipeline`, `sp_prod_model_serving`, `sp_dev_integration_tests`).

### 13.2 Lifecycle Management

| Stage | Responsibility | Process |
|---|---|---|
| **Creation** | `edap_platform_admins` | Provisioned via Terraform or Account Console. Registered in Azure AD / Entra ID. |
| **Scoping** | `edap_platform_admins` + domain steward | Granted minimum privileges required for the specific workload. Scoped to specific catalogs/schemas. |
| **Credential rotation** | Automated | OAuth tokens are the preferred authentication method. Where secrets are used, rotation is enforced on a 90-day cycle via automated pipeline. |
| **Review** | `edap_platform_admins` | Quarterly review of all service principal privileges. Unused service principals are flagged for decommissioning. |
| **Decommissioning** | `edap_platform_admins` | Privileges revoked, secrets rotated to invalid values, service principal disabled. Retained in audit log for 12 months. |

### 13.3 Least Privilege for Service Principals

- Pipeline service principals receive `MODIFY` only on the target schemas they write to and `SELECT` only on the schemas they read from.
- Model serving service principals receive `EXECUTE` on the model and `SELECT` on feature tables required for inference.
- Dashboard embedding service principals receive `SELECT` on the datasets backing the dashboard — their permissions define the data visible to embedded viewers.
- Service principals must not be members of `edap_platform_admins` or domain steward groups.

---

## 14. Data Sharing

### 14.1 Cross-Domain Sharing (Internal)

Cross-domain data sharing within WC is achieved through Unity Catalog's native privilege model. Because all domains share a single metastore, a user with appropriate grants can query across domain schemas using the three-level namespace:

```sql
-- An analyst with access to both domain schemas can join cross-domain
SELECT a.asset_id, c.customer_name
FROM prod_gold.asset_bi.asset_instance a
JOIN prod_gold.customer_bi.customer_asset_link c
  ON a.asset_id = c.asset_id;
```

ABAC policies on the relevant schemas are enforced at query time, ensuring PI masking and row filtering apply regardless of how the data is accessed.

### 14.2 External Sharing (Delta Sharing)

For sharing data with external parties (regulators, contractors, partner utilities), WC uses **Delta Sharing**. This avoids direct access to Unity Catalog and provides:

- Read-only, time-limited access to specific shares
- No requirement for the recipient to have a Databricks account (open sharing protocol)
- Audit logging of all recipient access events
- ABAC policies can be applied to shared data

External Delta Sharing follows the six-step governance process defined in the companion **Domain Governance Across Systems** document: request, classification review, approval, share creation, monitoring, and revocation.

### 14.3 Privacy-Preserving Collaboration (Clean Rooms)

For scenarios where data must be analysed collaboratively without physical data movement or exposure of raw records, WC can use **Databricks Clean Rooms**. Participants define their own tables and the approved computations (aggregations, joins, ML training) that can run against them — only computation results are returned, never the underlying data. Clean Rooms are governed by Unity Catalog and use serverless compute.

Clean Rooms are appropriate for:
- Sharing anonymised operational data with government agencies or regulators
- Joint analysis with partner utilities or suppliers
- Cross-organisation analytics where PI or commercially sensitive data is involved

---

## 15. Roadmap Alignment

| Databricks Capability | Status (Mar 2026) | WC Relevance |
|---|---|---|
| **ABAC (Governed Tags + Policies)** | Public Preview (AWS, Azure, GCP) | Core to WC's classification-driven access model |
| **Data Classification (auto-detect)** | GA (compliance profiles) | Automate PI detection during ingestion. Evaluate for integration with `pi_contained` / `pi_type` tagging workflow |
| **Legacy feature deprecation** (DBFS, Hive Metastore) | Enforced for new accounts (Dec 2025+) | WC should plan to disable legacy access on existing workspaces |
| **Catalog Explorer improvements** (Govern tab, Suggested tab) | GA | Improved discovery and governance UX for domain stewards |
| **Delta Sharing with ABAC** | Public Preview | Enables externally shared data to retain ABAC protections |
| **Lakehouse Federation** | GA | Federated queries against external sources under Unity Catalog governance |
| **Clean Rooms** | GA | Privacy-preserving collaboration for regulatory and cross-organisation scenarios |
| **AI Gateway** | GA | Centralised GenAI governance, routing, rate limiting, and audit |
| **Online Tables** | GA | Low-latency feature serving for real-time ML inference |
| **Predictive Optimization** | GA | Automated table maintenance — reduces operational overhead |

---

## 16. Decision Register

| Decision | Rationale |
|---|---|
| Single metastore for all workspaces | Databricks best practice. Simplifies cross-domain sharing. One metastore per region. |
| Catalogs as `<env>_<layer>` | Aligns with medallion architecture. Layer-based catalogs provide clear environment and zone separation. Domain governance is enforced at schema level via MANAGE privilege. |
| MANAGE privilege for domain delegation | Enables domain autonomy without transferring full ownership. Does not grant implicit data access. Granted at schema level to domain steward groups. |
| ABAC for classification enforcement | Scalable, tag-driven. Policies defined once at catalog level and inherited. Replaces per-table row filters. |
| Account-level SCIM only | Databricks recommendation. Workspace-level SCIM disabled. Consistent identity across all workspaces. |
| Open by default with ABAC exceptions | Maximises data utility. Restrictions applied only where classification demands it. |
| PI terminology (not PII) | PRIS Act 2024 compliance. Consistent with WC's regulatory obligations. |
| Dedicated workspace for Customer domain | PRIS Act requirements for PI processing isolation. Workspace binding restricted to customer-related schemas across the layer catalogs. May not be needed if ABAC + catalog binding is sufficient — to be validated. |
| Federated catalog naming with `_federated` suffix | Distinguishes federated (live external) data from native lakehouse data. Aligned with EDAP Tagging Strategy naming convention. |
| AI Gateway for centralised GenAI governance | Single control point for model access, rate limiting, guardrails, and audit. Prevents ungoverned direct-to-provider access. |
| Service principal lifecycle management | Critical for pipeline, ML serving, and embedded dashboard security. OAuth preferred over secrets. 90-day rotation enforced. |
| Clean Rooms for external privacy-preserving analytics | Provides a governed alternative to Delta Sharing where raw data must not be exposed. Supports regulatory and cross-organisation collaboration. |

---

## Appendix A: Unity Catalog Access Control Layers

The following layers work together to enforce WC's access model. Each layer addresses a different dimension of control:

1. **Workspace bindings** — Controls *where* users can access data (which workspace can see which catalogs).
2. **Privileges and ownership** — Controls *who* can access *what* (GRANT/REVOKE on securables using ANSI SQL).
3. **ABAC policies (governed tags)** — Controls *what data* users can see within tables (row filtering, column masking driven by classification tags).
4. **Dynamic views** — Optional additional layer for complex cross-table access logic.

These layers are complementary and cumulative. A query must pass all applicable layers to return data.

---

## Appendix B: References

- [Databricks Unity Catalog Best Practices](https://docs.databricks.com/aws/en/data-governance/unity-catalog/best-practices)
- [Unity Catalog ABAC Documentation](https://docs.databricks.com/aws/en/data-governance/unity-catalog/abac/)
- [Federated Data Catalog Ownership (Databricks Community)](https://community.databricks.com/t5/technical-blog/federated-data-catalog-ownership-balance-governance-and-autonomy/ba-p/103026)
- [Unity Catalog Access Control](https://docs.databricks.com/aws/en/data-governance/unity-catalog/access-control)
- [Lakehouse Federation](https://docs.databricks.com/aws/en/query-federation/)
- [AI Gateway](https://docs.databricks.com/aws/en/ai-gateway/)
- [Clean Rooms](https://docs.databricks.com/aws/en/clean-rooms/)
- [Online Tables](https://docs.databricks.com/aws/en/machine-learning/feature-store/online-tables)
- [Model Serving](https://docs.databricks.com/aws/en/machine-learning/model-serving/)
- [Databricks Workspace Best Practices](https://www.databricks.com/blog/2022/03/10/functional-workspace-organization-on-databricks.html)

---

*This document is maintained by Architecture & Strategy, Digital & Technology, Water Corporation. For questions or change requests, contact the EDAP governance team.*

# EDAP Access Model â Databricks Unity Catalog & Federated Domain Governance

**Document Owner:** Architecture & Strategy, Digital & Technology  
**Classification:** OFFICIAL  
**Status:** Draft  
**Last Updated:** March 2026  

---

## 1. Purpose

This document defines Water Corporation's access model for the Enterprise Data Analytics Platform (EDAP), built on Databricks Unity Catalog. It describes how workspaces, catalogs, identity, and attribute-based access control (ABAC) are structured to support a federated, domain-based data governance model across WC's seven data domains.

This wiki should be read alongside:

- **EDAP-FWK-001** â Pipeline Framework Requirements Specification
- **ADR-EDP-001** â Development Environment Data Strategy
- **EDAP Data Products Wiki** â Data contracts, FAUQD test, and certification processes
- **WC Data Classification & Tagging Strategy** â WAICP, PRIS Act 2024, SOCI Act 2018, and State Records Act 2000 alignment

---

## 2. Guiding Principles

The access model is grounded in the following principles, established through WC's enterprise data governance framework:

| Principle | Description |
|---|---|
| **Open by default, restricted by exception** | All data is accessible unless a classification assessment determines otherwise. Restrictions are applied through governed tags and ABAC policies, not blanket denial. |
| **Explicit classification per object** | Every table, schema, and catalog receives a deliberate classification assessment. Classification is never automatically inherited from parent objects â it must be explicitly set. |
| **Personal Information (PI) terminology** | WC uses the term Personal Information (PI) per the PRIS Act 2024, not PII. All tagging, policies, and documentation must reflect this. |
| **Three-level access model** | Data access is categorised as **Open**, **Restricted**, or **Privileged**, aligned to WC's data sensitivity framework. |
| **Federated domain ownership** | Each of WC's seven data domains has an executive owner who governs data within their domain, operating within guardrails set by the central Architecture & Strategy function. |
| **Least privilege** | Users receive the minimum access required for their role. Unity Catalog enforces this natively. |
| **Define once, secure everywhere** | Access policies are defined centrally in Unity Catalog and enforced consistently across all workspaces. |

---

## 3. Architecture Overview

### 3.1 Single Metastore

WC operates a **single Unity Catalog metastore** in the AWS Sydney (ap-southeast-2) region. This metastore serves all EDAP workspaces. Databricks recommends one metastore per region â it is not intended as a unit of data isolation, but as a regional boundary for metadata.

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

For domains with strict isolation requirements â particularly Customer & Billing data subject to the PRIS Act 2024 â a dedicated workspace may be provisioned. This provides an additional layer of network and compute isolation beyond what catalog bindings alone deliver.

| Workspace | Justification |
|---|---|
| `wc-edap-prod-customer` | PRIS Act 2024 requirements for PI processing. Dedicated compute with fine-grained access control. Catalog binding restricted to `prod_customer` and approved cross-domain reference catalogs only. |

> **Decision criteria for domain-specific workspaces:** A separate workspace is warranted only when regulatory, network isolation, or compute isolation requirements cannot be met through catalog bindings and ABAC alone. Most domains will operate within the shared environment workspaces.

### 3.3 Catalog Layout â Domain Ã Environment

Catalogs follow a `<env>_<domain>` naming convention and represent the **primary unit of data isolation**.

#### Production Catalogs

| Catalog | Domain | Executive Owner |
|---|---|---|
| `prod_asset` | Asset Management | *[Domain Executive]* |
| `prod_customer` | Customer & Billing | *[Domain Executive]* |
| `prod_network` | Network & Infrastructure | *[Domain Executive]* |
| `prod_operations` | Operations | *[Domain Executive]* |
| `prod_workforce` | Workforce & Safety | *[Domain Executive]* |
| `prod_finance` | Finance & Commercial | *[Domain Executive]* |
| `prod_environment` | Environment & Compliance | *[Domain Executive]* |
| `prod_reference` | Cross-domain reference data | Architecture & Strategy |
| `prod_platform` | EDAP platform metadata, audit, lineage | Architecture & Strategy |

Each domain also has corresponding `dev_<domain>` and `staging_<domain>` catalogs.

#### Schema Structure Within Each Catalog

Schemas within each domain catalog align to the EDAP medallion architecture:

| Schema | Medallion Layer | Description |
|---|---|---|
| `raw` | Raw | Ingested data in original form. Append-only. |
| `base` | Base / Bronze | Cleansed, typed, and conformed. SCD Type 1/2 applied. `edap_hash` change detection. |
| `curated` | Silver | Business logic applied. Conformed dimensions and facts. Domain-specific transformations. |
| `product` | Gold | Certified data products. Published via data contracts. FAUQD-tested. |
| `sandbox` | N/A | Domain team experimentation space. Not promoted to production. |

> **Landing Zone implementation:** The Landing Zone is not a schema but is implemented using **Unity Catalog Volumes** within each domain catalog. Files from source systems are staged in managed or external Volumes (e.g. `/Volumes/prod_asset/raw/landing/`) before being ingested into `raw` schema tables via Auto Loader or Lakeflow Connect. Volumes provide governed, access-controlled file storage that replaces legacy DBFS mounts.

#### Sandbox Governance

The `sandbox` schema in each domain catalog provides a space for experimentation, but is subject to the following guardrails:

- **Retention policy:** Objects in sandbox schemas are subject to a 90-day retention policy. Objects older than 90 days are flagged for deletion and removed after a 14-day grace period unless explicitly extended by the domain steward.
- **Read access to production:** Sandbox users may be granted read-only access to `base`, `curated`, and `product` schemas within their domain to enable experimentation against real data. This access is governed by the same ABAC policies as any other query.
- **Storage quotas:** Each domain's sandbox schema has a storage quota (set via cluster policies and monitoring) to prevent runaway experimentation from consuming excessive storage. Quotas are reviewed quarterly.
- **Promotion path:** Successful sandbox experiments that should become production assets must follow the standard pipeline promotion path: code is committed to source control, deployed via DABs through dev â staging â prod, and outputs land in the appropriate medallion schema (`base`, `curated`, or `product`). Direct promotion from sandbox to production is not permitted.
- **No PI in sandbox:** Sandbox schemas must not contain unmasked Personal Information. If PI is required for experimentation, anonymised or synthetic datasets (per ADR-EDP-001) must be used.

#### Schema-to-Medallion Zone Mapping

The following table provides an explicit mapping between EDAP schema names and the medallion architecture zones described in the companion Medallion Architecture document:

| Medallion Zone | EDAP Schema | Implementation |
|---|---|---|
| Landing Zone | *(Unity Catalog Volumes)* | Files land in managed or external Volumes within the domain catalog (e.g. `prod_asset.raw` volume paths). Not persisted as tables. |
| Raw | `raw` | Append-only Delta tables preserving source data as received. |
| Silver Base | `base` | Cleansed, typed, conformed data with SCD and `edap_hash` change detection. |
| Silver Enriched | `curated` | Business logic applied, cross-source joins, conformed dimensions and facts. |
| Gold | `product` | Certified data products published via data contracts. |

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

- Schema and table creation within their domain catalogs
- Grant management within their domain (via `MANAGE` privilege)
- Data contract authorship and maintenance
- Classification assessment of domain objects (applying governed tags)
- Data product development, testing, and promotion
- Domain-specific ABAC policy proposals (implemented by central team)

### 4.2 The MANAGE Privilege

The `MANAGE` privilege is the key mechanism enabling federated ownership in Unity Catalog. It allows domain teams to:

- Manage grants on objects within their catalog (grant/revoke `SELECT`, `MODIFY`, etc.)
- Drop and alter objects (schema evolution)
- Handle day-to-day governance without requiring central team involvement

Critically, `MANAGE` does **not** implicitly grant `SELECT` or other data access privileges â these must be explicitly assigned. This ensures that a domain steward can administer access without necessarily being able to read all data in their domain (important for PI-restricted datasets).

**Implementation:**

```sql
-- Central team grants MANAGE on domain catalog to domain steward group
GRANT MANAGE ON CATALOG prod_asset TO `domain_asset_stewards`;

-- Domain steward can then independently manage access
GRANT SELECT ON SCHEMA prod_asset.product TO `asset_analysts`;
GRANT USE SCHEMA ON SCHEMA prod_asset.product TO `asset_analysts`;
```

### 4.3 Ownership Rules

- **Production catalogs and schemas** must be owned by groups, never individual users.
- **Catalog ownership** is held by the domain's data steward group (e.g. `domain_asset_stewards`).
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
| `domain_<domain>_stewards` | `domain_asset_stewards` | Data stewardship and governance for the domain. Receives `MANAGE` on domain catalogs. |
| `domain_<domain>_engineers` | `domain_asset_engineers` | Data engineering within the domain. `MODIFY` on raw/base/curated schemas. |
| `domain_<domain>_analysts` | `domain_asset_analysts` | Analytical consumers. `SELECT` on curated/product schemas. |
| `domain_<domain>_scientists` | `domain_customer_scientists` | Data science and ML workloads. Broader read access plus model registry permissions. |
| `edap_platform_admins` | â | Central platform team. Account admin and metastore configuration. |
| `edap_platform_engineers` | â | Central engineering team. Pipeline framework development. |
| `pris_authorised_<category>` | `pris_authorised_billing` | Users explicitly authorised to access specific categories of PI under the PRIS Act. Used in ABAC policy exceptions. |
| `soci_critical_access` | â | Users authorised to access SOCI Act critical infrastructure data. |

### 5.3 Workspace Access

Groups are assigned to workspaces based on their role:

- **Production workspace:** Domain stewards, production service principals, platform admins. Analysts access production data via SQL Warehouses, not direct workspace access where possible.
- **Development workspace:** Engineers, scientists, and analysts who need to develop or test.
- **Staging workspace:** Engineers and service principals performing integration testing.

---

## 6. Attribute-Based Access Control (ABAC)

### 6.1 Overview

ABAC is WC's primary mechanism for enforcing fine-grained, classification-driven access control at scale. It complements the privilege model (GRANT/REVOKE) by allowing policies to be defined based on **governed tags** applied to data assets.

ABAC is now available in Public Preview on AWS and is Databricks' recommended approach for centralised, scalable governance â replacing the previous practice of applying row filters and column masks individually per table.

### 6.2 Governed Tags

Governed tags are defined at the Databricks account level using tag policies. Only the values specified in the tag policy can be assigned â this prevents inconsistent or ad-hoc classification.

#### WC Governed Tag Taxonomy

| Tag Key | Allowed Values | Source Framework | Applied To |
|---|---|---|---|
| `sensitivity` | `open`, `restricted`, `privileged` | WC Data Classification | Catalogs, schemas, tables |
| `pi_category` | `contact`, `billing`, `identity`, `health`, `location`, `none` | PRIS Act 2024 | Tables, columns |
| `soci_critical` | `true`, `false` | SOCI Act 2018 | Catalogs, schemas, tables |
| `records_class` | `permanent`, `temporary`, `vital` | State Records Act 2000 | Tables |
| `domain` | `asset`, `customer`, `network`, `operations`, `workforce`, `finance`, `environment` | WC Domain Model | Catalogs, schemas |
| `data_product_tier` | `certified`, `provisional`, `experimental`, `deprecated` | EDAP Data Products Framework | Schemas, tables |
| `essential_eight` | `compliant`, `non_compliant`, `assessment_required` | Essential Eight | Tables, volumes |

> **Important:** Tags are assessed and applied explicitly per object. A table in the `prod_customer` catalog is not automatically tagged `pi_category=billing` â it must be assessed and tagged based on its actual content.

### 6.3 ABAC Policies

Policies are defined at the highest applicable level (typically the catalog) and inherit downward to all child schemas and tables. WC defines the following policy categories:

#### PI Masking Policy

Applied to all production catalogs. Masks columns tagged with any `pi_category` value other than `none`, unless the querying user is a member of the corresponding `pris_authorised_<category>` group.

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

The ABAC policy is then attached at the catalog level, matching columns tagged `pi_category` and applying the masking UDF. All current and future tables within that catalog are automatically covered.

#### SOCI Critical Row Filter

For tables tagged `soci_critical=true`, a row filter policy ensures only members of `soci_critical_access` can query the data.

#### Domain Isolation Policy

While catalog bindings and workspace bindings provide the primary isolation, ABAC policies can provide an additional layer â for example, ensuring that cross-domain reference data shared via `prod_reference` respects the originating domain's classification.

### 6.4 ABAC and the Development Lifecycle

ABAC policies on production catalogs apply regardless of which workspace the query originates from. When developers access production data in read-only mode from the development workspace (via catalog binding), the same masking and filtering policies are enforced. This ensures PI and SOCI protections are maintained even during development.

For development catalogs using anonymised or synthetic data (per ADR-EDP-001), ABAC policies may be relaxed since the underlying data does not contain real PI.

---

## 7. Catalog Binding & Workspace Isolation

### 7.1 Binding Rules

| Catalog Pattern | Bound To | Access Mode |
|---|---|---|
| `prod_*` | `wc-edap-prod` (and domain-specific prod workspaces if applicable) | Read-Write |
| `prod_*` (selected) | `wc-edap-dev`, `wc-edap-staging` | Read-Only |
| `dev_*` | `wc-edap-dev` | Read-Write |
| `staging_*` | `wc-edap-staging` | Read-Write |
| `prod_reference` | All workspaces | Read-Only (except in prod) |
| `prod_platform` | All workspaces | Read-Only (except in prod) |

### 7.2 Binding Implementation

```sql
-- Bind production asset catalog to production workspace only (read-write)
ALTER CATALOG prod_asset SET OWNER TO `domain_asset_stewards`;
-- Workspace binding configured via Account Console or Terraform

-- Bind production asset catalog to dev workspace as read-only
-- (configured via Account Console â catalog binding with READ_ONLY mode)
```

> **Note:** Catalog bindings override user-level permissions. Even if a user has `SELECT` on a table, they cannot access it from an unbound workspace.

---

## 8. External Locations & Storage

- `CREATE EXTERNAL LOCATION` is restricted to `edap_platform_admins` only.
- Domain teams access data through managed tables and volumes â not through direct external location permissions.
- `READ FILES` and `WRITE FILES` are never granted on external locations to end users. All file-based access uses Unity Catalog volumes.
- External locations are bound to specific workspaces where required.

---

## 9. Compute & Access Modes

Unity Catalog requires compute that supports its access control model:

| Compute Type | Access Mode | Use Case |
|---|---|---|
| SQL Warehouse (Serverless) | Shared | Analyst queries, BI tool connections, ad-hoc SQL |
| Shared Cluster | Shared | Multi-user notebooks, lightweight engineering |
| Dedicated Cluster | Single User | ML training, PI-authorised workloads requiring fine-grained access |
| Serverless Compute | Shared | Lakeflow Declarative Pipelines, jobs |

> **ABAC requirement:** ABAC policies require Databricks Runtime 16.4 or above, or serverless compute. Older runtimes cannot access ABAC-protected tables.

---

## 10. Audit & Compliance

### 10.1 Audit Logging

Unity Catalog automatically captures user-level audit logs for all data access events. These logs feed into WC's centralised security monitoring solution and support:

- PRIS Act 2024 compliance â tracking who accessed PI and when
- SOCI Act 2018 reporting â monitoring access to critical infrastructure data
- State Records Act 2000 â demonstrating appropriate records management
- Essential Eight â supporting access control and audit trail requirements

### 10.2 System Tables

Unity Catalog system tables provide queryable access to:

- **Audit logs** â all access and modification events
- **Billable usage** â cost attribution by domain/workspace
- **Lineage** â data flow tracking across the medallion layers
- **Granted privileges** â current access state for compliance reporting

### 10.3 Monitoring

The central platform team monitors for:

- Excessive or suspicious access patterns on restricted/privileged data
- Anomalous create/alter/delete operations on securables
- Classification gaps â objects without `sensitivity` or `pi_category` tags
- ABAC policy conflicts or evaluation errors

### 10.4 Break-Glass (Emergency) Access

In incident scenarios requiring urgent access to data outside normal privilege grants, the following break-glass procedure applies:

1. **Request:** The requester raises an emergency access request through the IT service management tool, citing the incident reference and the specific data assets required.
2. **Approval:** A member of `edap_platform_admins` or the relevant domain executive owner approves the request. Two-person approval is required for `privileged`-classified data.
3. **Provisioning:** Temporary access is granted via time-limited group membership (managed through Azure AD / Entra ID with an automatic expiry, typically 4â8 hours).
4. **Audit:** All break-glass access events are flagged in the audit log with a dedicated tag. The platform team reviews all break-glass events weekly.
5. **Revocation:** Access is automatically revoked at expiry. Manual early revocation is performed if the incident is resolved sooner.

> **Important:** Break-glass access does not bypass ABAC policies or Unity Catalog governance. It grants temporary membership in an authorised group, ensuring all access is still logged and filtered.

---

## 11. Data Sharing

### 11.1 Cross-Domain Sharing (Internal)

Cross-domain data sharing within WC is achieved through Unity Catalog's native privilege model. Because all domains share a single metastore, a user with appropriate grants can query across domain catalogs using the three-level namespace:

```sql
-- An analyst with access to both domains can join cross-domain
SELECT a.asset_id, c.customer_name
FROM prod_asset.product.asset_instance a
JOIN prod_customer.product.customer_asset_link c
  ON a.asset_id = c.asset_id;
```

ABAC policies on both catalogs are enforced at query time, ensuring PI masking and row filtering apply regardless of how the data is accessed.

### 11.2 External Sharing

For sharing data with external parties (regulators, contractors, partner utilities), WC uses **Delta Sharing**. This avoids direct access to Unity Catalog and provides:

- Read-only, time-limited access to specific shares
- No requirement for the recipient to have a Databricks account (open sharing protocol)
- Audit logging of all recipient access events
- ABAC policies can be applied to shared data

---

## 12. Roadmap Alignment

| Databricks Capability | Status | WC Relevance |
|---|---|---|
| **ABAC (Governed Tags + Policies)** | Public Preview (AWS, Azure, GCP) | Core to WC's classification-driven access model |
| **Data Classification (auto-detect)** | Public Preview; GA for compliance profiles mid-March 2026 | Potential to automate PI detection during ingestion |
| **Legacy feature deprecation** (DBFS, Hive Metastore) | GA â enforced for new accounts from Dec 2025 | WC should plan to disable legacy access on existing workspaces |
| **Catalog Explorer improvements** (Govern tab, Suggested tab) | Rolling out March 2026 | Improved discovery and governance UX for domain stewards |
| **Delta Sharing with ABAC** | Public Preview | Enables externally shared data to retain ABAC protections |

---

## 13. Decision Register

| Decision | Rationale |
|---|---|
| Single metastore for all workspaces | Databricks best practice. Simplifies cross-domain sharing. One metastore per region. |
| Catalogs as `<env>_<domain>` | Primary isolation unit. Supports federated ownership and clear workspace binding. |
| MANAGE privilege for domain delegation | Enables domain autonomy without transferring full ownership. Does not grant implicit data access. |
| ABAC for classification enforcement | Scalable, tag-driven. Policies defined once at catalog level and inherited. Replaces per-table row filters. |
| Account-level SCIM only | Databricks recommendation. Workspace-level SCIM disabled. Consistent identity across all workspaces. |
| Open by default with ABAC exceptions | Maximises data utility. Restrictions applied only where classification demands it. |
| PI terminology (not PII) | PRIS Act 2024 compliance. Consistent with WC's regulatory obligations. |
| Dedicated workspace for Customer domain | PRIS Act requirements for PI processing isolation. May not be needed if ABAC + catalog binding is sufficient â to be validated. |

---

## Appendix A: Unity Catalog Access Control Layers

The following layers work together to enforce WC's access model. Each layer addresses a different dimension of control:

1. **Workspace bindings** â Controls *where* users can access data (which workspace can see which catalogs).
2. **Privileges and ownership** â Controls *who* can access *what* (GRANT/REVOKE on securables using ANSI SQL).
3. **ABAC policies (governed tags)** â Controls *what data* users can see within tables (row filtering, column masking driven by classification tags).
4. **Dynamic views** â Optional additional layer for complex cross-table access logic.

These layers are complementary and cumulative. A query must pass all applicable layers to return data.

---

## Appendix B: References

- [Databricks Unity Catalog Best Practices](https://docs.databricks.com/aws/en/data-governance/unity-catalog/best-practices)
- [Unity Catalog ABAC Documentation](https://docs.databricks.com/aws/en/data-governance/unity-catalog/abac/)
- [Federated Data Catalog Ownership (Databricks Community)](https://community.databricks.com/t5/technical-blog/federated-data-catalog-ownership-balance-governance-and-autonomy/ba-p/103026)
- [Unity Catalog Access Control](https://docs.databricks.com/aws/en/data-governance/unity-catalog/access-control)
- [Databricks Workspace Best Practices](https://www.databricks.com/blog/2022/03/10/functional-workspace-organization-on-databricks.html)

---

*This document is maintained by Architecture & Strategy, Digital & Technology, Water Corporation. For questions or change requests, contact the EDAP governance team.*

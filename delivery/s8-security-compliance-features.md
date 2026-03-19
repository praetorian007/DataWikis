# S8 – Security & Compliance: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `platform/edap-access-model.md` — ABAC, catalog structure, break-glass access, workspace topology
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, governed tags, WAICP alignment
- `governance/data-governance-roles.md` — Roles, RACI, federated governance model
- `platform/medallion-architecture.md` — Layer-level security, access control model

---

## Feature S8-F1: RBAC Implementation

**Description:** Implement role-based access control across Databricks workspaces, mapping IAM roles to workspace entitlements, configuring service principals for pipeline execution, and enforcing least-privilege access aligned to WC's domain-based group structure.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F1-US01 | Platform Administrator | define Databricks workspace roles (Admin, User, Data Engineer, Data Analyst) mapped to WC's functional requirements | each persona has appropriate workspace entitlements without over-provisioning |
| S8-F1-US02 | Platform Administrator | map AWS IAM roles to Databricks service principals and SCIM groups | platform identity is consistent across AWS and Databricks layers |
| S8-F1-US03 | Data Engineer | use a dedicated service principal for production pipeline execution | pipelines run with least-privilege credentials and are auditable to a non-human identity |
| S8-F1-US04 | Domain Data Steward | manage grants within my domain catalog using the MANAGE privilege | I can independently govern access within my domain without central team involvement |
| S8-F1-US05 | Security Officer | verify that no individual user accounts own production catalogs or schemas | ownership is always held by groups, reducing key-person risk |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F1-AC01 | Workspace roles are defined in the Databricks account console | a new user is provisioned via SCIM | they receive only the workspace entitlements matching their assigned role (Admin, User, Data Engineer, or Data Analyst) |
| S8-F1-AC02 | AWS IAM roles are mapped to Databricks service principals | a Lakeflow pipeline executes in the production workspace | it authenticates using its designated service principal, not a personal user identity |
| S8-F1-AC03 | Domain group structure follows `domain_<domain>_<role>` pattern (e.g. `domain_asset_engineers`) | a domain engineer is assigned to the asset domain | they receive MODIFY on raw/base/curated schemas and SELECT on product schemas within `prod_asset` only |
| S8-F1-AC04 | The MANAGE privilege is granted to domain steward groups | a domain steward grants SELECT on a schema within their catalog | the grant succeeds without requiring platform admin intervention |
| S8-F1-AC05 | Production catalogs and schemas are configured | an audit query checks ownership of all production securables | zero objects are owned by individual user accounts; all are owned by groups or service principals |
| S8-F1-AC06 | Least-privilege is enforced | an analyst attempts to MODIFY a Bronze table they have SELECT-only access to | the operation is denied with an appropriate permissions error |

### Technical Notes
- Group structure aligns to the access model wiki: `domain_<domain>_stewards`, `domain_<domain>_engineers`, `domain_<domain>_analysts`, `domain_<domain>_scientists`, plus cross-cutting groups (`edap_platform_admins`, `pris_authorised_<category>`, `soci_critical_access`).
- MANAGE privilege grants domain autonomy without implicit data access per the access model wiki Section 4.2.
- Service principals must be used for all automated pipeline execution; personal tokens must not be used in production.
- Workspace access assignments follow the topology in the access model wiki Section 5.3: production workspace restricted to stewards, service principals, and platform admins.

---

## Feature S8-F2: ABAC Implementation

**Description:** Implement attribute-based access control using Unity Catalog governed tags and ABAC policies to enforce classification-driven column masking, row-level filtering, and dynamic access decisions aligned to WC's 4-layer tagging strategy.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F2-US01 | Platform Administrator | define governed tag policies at the Databricks account level with restricted allowed values | only approved classification values can be applied to data assets, preventing ad-hoc or inconsistent tagging |
| S8-F2-US02 | Data Analyst | query a table containing Personal Information (PI) columns | PI columns are automatically masked unless I am a member of the corresponding `pris_authorised_<category>` group |
| S8-F2-US03 | Domain Data Steward | apply governed tags (`sensitivity`, `pi_category`, `soci_critical`) to tables and columns within my domain | ABAC policies automatically enforce the correct access controls without per-table manual configuration |
| S8-F2-US04 | Security Officer | ensure SOCI-critical data is only accessible to authorised users | a row filter policy on tables tagged `soci_critical=true` restricts access to `soci_critical_access` group members |
| S8-F2-US05 | Data Engineer | access production data from the development workspace in read-only mode | the same ABAC masking and filtering policies apply regardless of which workspace I query from |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F2-AC01 | Governed tag policies are created for `sensitivity`, `pi_category`, `soci_critical`, `records_class`, `domain`, `data_product_tier`, and `essential_eight` | a steward attempts to apply a value not in the allowed set | the operation is rejected by Unity Catalog |
| S8-F2-AC02 | A PI masking UDF is deployed in `prod_platform.governance` schema | a user without `pris_authorised_billing` membership queries a column tagged `pi_category=billing` | the column value is returned as `***MASKED***` |
| S8-F2-AC03 | A PI masking UDF is deployed in `prod_platform.governance` schema | a user who is a member of `pris_authorised_billing` queries a column tagged `pi_category=billing` | the column value is returned unmasked |
| S8-F2-AC04 | A SOCI row filter policy is attached at catalog level | a user without `soci_critical_access` membership queries a table tagged `soci_critical=true` | zero rows are returned |
| S8-F2-AC05 | ABAC policies are applied to production catalogs | a developer queries production data via read-only catalog binding from the dev workspace | masking and row filter policies are enforced identically to production workspace queries |
| S8-F2-AC06 | Compute is configured for ABAC | all clusters and SQL Warehouses used for querying ABAC-protected tables | run Databricks Runtime 16.4 or above, or serverless compute |
| S8-F2-AC07 | ABAC policies are defined at catalog level | a new table is created within an ABAC-protected catalog | the table automatically inherits the catalog-level ABAC policy without additional configuration |

### Technical Notes
- ABAC is in Public Preview on AWS; requires DBR 16.4+ or serverless compute per the access model wiki Section 9.
- Governed tag taxonomy must match the tagging strategy wiki Layer 2 and Layer 3 tags precisely.
- Dynamic views may be used as an additional layer for complex cross-table access logic per the access model wiki Appendix A.
- PI masking UDFs should handle all `pi_category` values (contact, billing, identity, health, location) per the access model wiki Section 6.3.
- Tags are assessed and applied explicitly per object; they are never automatically inherited from parent objects per the access model wiki Section 6.2.

---

## Feature S8-F3: SCIM Provisioning

**Description:** Configure account-level SCIM provisioning from Microsoft Entra ID (Azure AD) to synchronise users and groups into Databricks, automate group lifecycle management, and disable workspace-level SCIM to ensure a single source of identity truth.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F3-US01 | Platform Administrator | configure account-level SCIM provisioning from Entra ID | user and group identities are automatically synchronised to Databricks without manual provisioning |
| S8-F3-US02 | IT Security Analyst | ensure workspace-level SCIM is explicitly disabled | there is a single authoritative identity source and no workspace-local groups can be created that bypass Unity Catalog governance |
| S8-F3-US03 | HR Administrator | deactivate a departing employee in Entra ID | their Databricks access is automatically revoked within the next SCIM sync cycle |
| S8-F3-US04 | Platform Administrator | map Entra ID security groups to the WC domain group naming convention | group membership changes in Entra ID are reflected in Databricks group assignments automatically |
| S8-F3-US05 | Domain Data Steward | rely on group membership for access governance | when a team member changes roles, their Databricks group membership updates via Entra ID without manual Databricks intervention |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F3-AC01 | SCIM provisioning is configured at Databricks account level | a new user is added to an Entra ID security group mapped to `domain_asset_engineers` | the user appears in the Databricks account console within the SCIM sync interval (maximum 40 minutes) with correct group membership |
| S8-F3-AC02 | Workspace-level SCIM is disabled | an administrator attempts to create a workspace-local group | the operation is blocked or the group cannot be used for Unity Catalog grants |
| S8-F3-AC03 | A user is deactivated in Entra ID | the next SCIM sync cycle completes | the user's Databricks account is deactivated and they can no longer authenticate |
| S8-F3-AC04 | Entra ID groups follow the naming pattern required by EDAP | all Databricks account-level groups | match the naming patterns defined in the access model wiki (`domain_<domain>_<role>`, `edap_platform_admins`, `pris_authorised_<category>`, `soci_critical_access`) |
| S8-F3-AC05 | SCIM provisioning is operational | a monthly reconciliation report is generated | it confirms zero orphaned Databricks accounts (accounts not present in Entra ID) and zero unsynced groups |

### Technical Notes
- Account-level SCIM only per Databricks best practice and the access model wiki Section 5.1.
- All groups must be account-level groups, not workspace-local groups, to ensure Unity Catalog privilege compatibility.
- Group membership managed in Entra ID and synchronised automatically; no manual group management in Databricks.
- SCIM sync interval should be monitored and reconciled regularly to detect drift.

---

## Feature S8-F4: Encryption and Key Management

**Description:** Enable encryption at rest using AWS SSE-KMS with customer-managed keys (CMK), configure TLS for data in transit, and implement cluster-level encryption to ensure all data in the EDAP is protected per WC's security requirements and the Essential Eight framework.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F4-US01 | Security Officer | enable S3 server-side encryption (SSE-KMS) with customer-managed keys for all EDAP storage buckets | data at rest is encrypted under WC-controlled keys, not default AWS-managed keys |
| S8-F4-US02 | Platform Administrator | configure cluster-level encryption for Databricks compute | data on local disks and shuffle storage is encrypted during processing |
| S8-F4-US03 | Security Officer | enforce TLS 1.2 or higher for all data in transit | network communications between Databricks components, AWS services, and client connections are encrypted |
| S8-F4-US04 | Platform Administrator | implement key rotation policies for customer-managed keys | encryption keys are rotated annually (or on demand) without service disruption |
| S8-F4-US05 | Data Custodian | restrict key management permissions to authorised roles only | CMK creation, rotation, and deletion require `edap_platform_admins` group membership |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F4-AC01 | SSE-KMS is configured on all EDAP S3 buckets | a new Delta table file is written to S3 | the object is encrypted with the designated customer-managed key (verifiable via S3 object metadata) |
| S8-F4-AC02 | Cluster encryption is enabled | a Databricks cluster starts | local disk encryption is active (confirmed via cluster configuration audit) |
| S8-F4-AC03 | TLS enforcement is configured | a connection is attempted using TLS 1.1 or lower | the connection is rejected |
| S8-F4-AC04 | Key rotation policy is in place | a scheduled key rotation event occurs | the new key version is used for new writes; existing data remains readable via key version history |
| S8-F4-AC05 | IAM policies restrict KMS key management | a non-admin user attempts to create, rotate, or delete a CMK | the operation is denied by AWS IAM |
| S8-F4-AC06 | All encryption configurations are applied | an encryption compliance audit is run against EDAP infrastructure | all S3 buckets, EBS volumes, and network endpoints report compliant encryption status |

### Technical Notes
- AWS KMS customer-managed keys provide key rotation and access control via IAM policies.
- Databricks workspace encryption should use a workspace-specific CMK; Unity Catalog metastore storage should use a separate CMK.
- Cluster encryption covers both EBS volumes and local NVMe instance storage.
- TLS 1.2 minimum aligns with Essential Eight and SOCI Act requirements.

---

## Feature S8-F5: Audit Logging and SIEM Integration

**Description:** Enable comprehensive Databricks audit logging via Unity Catalog system tables, configure log delivery to Splunk for SIEM integration, and establish queryable audit infrastructure to support compliance reporting, incident investigation, and access monitoring.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F5-US01 | Security Officer | access Unity Catalog audit logs capturing all data access events | I can investigate who accessed what data and when, supporting PRIS Act and SOCI Act compliance |
| S8-F5-US02 | Platform Administrator | configure system tables for audit logs, billable usage, lineage, and granted privileges | audit data is queryable via SQL for self-service compliance analysis |
| S8-F5-US03 | Security Operations Analyst | receive Databricks audit logs in Splunk | EDAP events are correlated with broader enterprise security events in the existing SIEM |
| S8-F5-US04 | Data Custodian | query billable usage system tables by domain and workspace | cost can be attributed to specific domains for chargeback and budget monitoring |
| S8-F5-US05 | Security Officer | track all privilege grant and revoke operations | changes to the access control posture are fully auditable and reviewable |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F5-AC01 | Unity Catalog audit logging is enabled | a user queries a table in any production catalog | the access event (user, table, timestamp, action, workspace) is captured in the system audit table within 15 minutes |
| S8-F5-AC02 | Splunk integration is configured | audit log events are generated in Databricks | events appear in the designated Splunk index within 30 minutes |
| S8-F5-AC03 | System tables are enabled | a platform admin runs a query against `system.access.audit` | the query returns structured audit data for the configured retention period |
| S8-F5-AC04 | Lineage system tables are enabled | a pipeline writes data from Bronze to Silver | the lineage event is captured in `system.access.table_lineage` and reflects the source-to-target relationship |
| S8-F5-AC05 | Privilege system tables are enabled | a domain steward grants SELECT on a schema | the grant event is captured in both the audit log and `system.information_schema.catalog_privileges` |
| S8-F5-AC06 | Billable usage system tables are enabled | a monthly cost report is generated | the report attributes DBU consumption to each workspace and catalog (domain) |

### Technical Notes
- System tables referenced in the access model wiki Section 10.2: audit logs, billable usage, lineage, granted privileges.
- Splunk integration may use Databricks audit log delivery (account-level configuration) or a streaming pipeline from system tables to Splunk via HEC (HTTP Event Collector).
- Audit log retention should align with WC's records management requirements and the State Records Act 2000.
- Consider creating a dedicated dashboard in `prod_platform` for security monitoring.

---

## Feature S8-F6: Compliance Reporting and Alerting

**Description:** Implement periodic compliance reporting, automated security alerts for anomalous access patterns, SOCI Act compliance monitoring, and break-glass emergency access procedures aligned to the access model wiki's defined process.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F6-US01 | Security Officer | receive a weekly compliance report summarising access patterns on restricted and privileged data | I can identify anomalous or excessive access and take corrective action |
| S8-F6-US02 | Security Operations Analyst | receive automated alerts when unauthorised access attempts or privilege escalations are detected | I can respond to potential security incidents in near real-time |
| S8-F6-US03 | Data Protection Officer | generate a PRIS Act compliance report showing who accessed PI data over a specified period | I can demonstrate regulatory compliance during audits |
| S8-F6-US04 | Platform Administrator | implement the break-glass emergency access procedure | urgent data access can be granted with two-person approval, time-limited group membership, and full audit trail |
| S8-F6-US05 | Security Officer | receive a weekly review summary of all break-glass access events | every emergency access event is tracked, reviewed, and documented |
| S8-F6-US06 | Security Officer | monitor for classification gaps across production catalogs | objects without `sensitivity` or `pi_category` tags are identified and reported for remediation |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F6-AC01 | Compliance reporting is configured | the weekly reporting schedule triggers | a report is generated listing: number of access events on restricted/privileged data, top 10 accessing users, any denied access attempts, and classification coverage percentage |
| S8-F6-AC02 | Alerting rules are configured | a user attempts to access data tagged `sensitivity=privileged` without being in an authorised group | an alert is raised in Splunk and the platform team is notified within 10 minutes |
| S8-F6-AC03 | Alerting rules are configured | a privilege escalation occurs (e.g. GRANT MANAGE by a non-admin) | an alert is raised immediately and flagged for investigation |
| S8-F6-AC04 | The break-glass procedure is implemented | an emergency access request is raised and approved by a platform admin or domain executive | temporary group membership is provisioned in Entra ID with automatic expiry (4-8 hours) and the event is flagged with a dedicated audit tag |
| S8-F6-AC05 | Break-glass access has been provisioned | the expiry time is reached | group membership is automatically revoked without manual intervention |
| S8-F6-AC06 | Break-glass access has been provisioned | the weekly break-glass review is conducted | all break-glass events from the previous week are listed with: requester, approver, incident reference, data assets accessed, and duration |
| S8-F6-AC07 | Classification gap monitoring is configured | a weekly scan runs against production catalogs | all tables and schemas missing `sensitivity` or `pi_category` tags are reported to the relevant domain steward for remediation |
| S8-F6-AC08 | SOCI Act compliance monitoring is configured | an access event occurs on data tagged `soci_critical=true` | the event is logged with enhanced metadata and included in the SOCI compliance report |

### Technical Notes
- Break-glass procedure follows the 5-step process defined in the access model wiki Section 10.4: request, approval (two-person for `privileged` data), provisioning (time-limited Entra ID group membership), audit, revocation.
- Break-glass access does not bypass ABAC policies or Unity Catalog governance; it grants temporary group membership so all access remains logged and policy-filtered.
- Classification gap monitoring aligns with the tagging strategy wiki's requirement for explicit classification per object.
- SOCI Act reporting must align with mandatory incident reporting requirements to the Cyber and Infrastructure Security Centre (CISC) as referenced in the governance roles wiki.
- Compliance reports should be stored in `prod_platform` catalog for retention and audit purposes.

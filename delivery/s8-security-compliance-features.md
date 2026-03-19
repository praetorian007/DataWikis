# S8 – Security & Compliance: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `platform/edap-access-model.md` — ABAC, catalog structure, break-glass access, workspace topology
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, governed tags, WAICP alignment
- `governance/data-governance-roles.md` — Roles, RACI, federated governance model
- `platform/medallion-architecture.md` — Layer-level security, access control model

---

## Feature S8-F1: Domain Teams Independently Manage Their Own Data Access

**Description:** Domain stewards and engineers can grant, revoke, and audit access within their own domain catalogues without waiting for a central platform team — while the platform enforces least-privilege boundaries that prevent accidental over-provisioning.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F1-US01 | Domain Data Steward | grant a new analyst SELECT access to my domain's curated schema myself | they can start querying data today instead of waiting for a platform admin to action my request |
| S8-F1-US02 | Domain Data Steward | see exactly who has access to what within my domain catalogue | I can answer audit questions and spot excessive privileges without raising a support ticket |
| S8-F1-US03 | Data Engineer | run production pipelines under a service principal identity | my personal credentials are never used in production, and every pipeline action is traceable to a non-human identity |
| S8-F1-US04 | Security Officer | confirm that no individual user account owns a production catalogue or schema | key-person risk is eliminated because ownership is always held by groups or service principals |
| S8-F1-US05 | Data Analyst | attempt an action I'm not entitled to and receive a clear, immediate denial | I understand my access boundaries without guesswork, and least-privilege is enforced consistently |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F1-AC01 | Workspace roles are defined in the Databricks account console | a new user is provisioned via SCIM | they receive only the workspace entitlements matching their assigned role (Admin, User, Data Engineer, or Data Analyst) |
| S8-F1-AC02 | AWS IAM roles are mapped to Databricks service principals | a Lakeflow pipeline executes in the production workspace | it authenticates using its designated service principal, not a personal user identity |
| S8-F1-AC03 | Domain group structure follows `domain_<domain>_<role>` pattern (e.g. `domain_asset_engineers`) | a domain engineer is assigned to the asset domain | they receive MODIFY on raw/base/curated schemas and SELECT on product schemas within `prod_asset` only |
| S8-F1-AC04 | The MANAGE privilege is granted to domain steward groups | a domain steward grants SELECT on a schema within their catalogue | the grant succeeds without requiring platform admin intervention |
| S8-F1-AC05 | Production catalogues and schemas are configured | an audit query checks ownership of all production securables | zero objects are owned by individual user accounts; all are owned by groups or service principals |
| S8-F1-AC06 | Least-privilege is enforced | an analyst attempts to MODIFY a Bronze table they have SELECT-only access to | the operation is denied with an appropriate permissions error |

### Technical Notes
- Group structure aligns to the access model wiki: `domain_<domain>_stewards`, `domain_<domain>_engineers`, `domain_<domain>_analysts`, `domain_<domain>_scientists`, plus cross-cutting groups (`edap_platform_admins`, `pris_authorised_<category>`, `soci_critical_access`).
- MANAGE privilege grants domain autonomy without implicit data access per the access model wiki Section 4.2.
- Service principals must be used for all automated pipeline execution; personal tokens must not be used in production.
- Workspace access assignments follow the topology in the access model wiki Section 5.3: production workspace restricted to stewards, service principals, and platform admins.

---

## Feature S8-F2: Sensitive Data Automatically Protected Based on Its Classification Tags

**Description:** Once a steward tags a column or table with its sensitivity classification, the platform automatically enforces the right protection — masking PI columns, filtering SOCI-critical rows, and applying these controls identically across every workspace — without anyone writing per-table security rules.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F2-US01 | Domain Data Steward | tag a column as `pi_category=contact` and have it automatically masked for unauthorised users | I don't need to configure individual masking rules — the platform enforces protection from the tag alone |
| S8-F2-US02 | Data Analyst | query a table containing PI columns and see masked values where I'm not authorised | I can work with the dataset safely without accidentally viewing sensitive data I shouldn't see |
| S8-F2-US03 | Data Analyst (authorised) | query the same PI column and see unmasked values because I'm in the `pris_authorised_contact` group | I can perform my authorised analytical work without restrictions slowing me down |
| S8-F2-US04 | Security Officer | know that SOCI-critical data is invisible to users outside the `soci_critical_access` group | regulatory obligations are met automatically, not through manual access reviews |
| S8-F2-US05 | Data Engineer | query production data from the development workspace in read-only mode | the same masking and filtering policies apply regardless of which workspace I query from — no security gaps between environments |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F2-AC01 | Governed tag policies are created for `sensitivity`, `pi_category`, `soci_critical`, `records_class`, `domain`, `data_product_tier`, and `essential_eight` | a steward attempts to apply a value not in the allowed set | the operation is rejected by Unity Catalog |
| S8-F2-AC02 | A PI masking UDF is deployed in `prod_platform.governance` schema | a user without `pris_authorised_billing` membership queries a column tagged `pi_category=billing` | the column value is returned as `***MASKED***` |
| S8-F2-AC03 | A PI masking UDF is deployed in `prod_platform.governance` schema | a user who is a member of `pris_authorised_billing` queries a column tagged `pi_category=billing` | the column value is returned unmasked |
| S8-F2-AC04 | A SOCI row filter policy is attached at catalogue level | a user without `soci_critical_access` membership queries a table tagged `soci_critical=true` | zero rows are returned |
| S8-F2-AC05 | ABAC policies are applied to production catalogues | a developer queries production data via read-only catalogue binding from the dev workspace | masking and row filter policies are enforced identically to production workspace queries |
| S8-F2-AC06 | Compute is configured for ABAC | all clusters and SQL Warehouses used for querying ABAC-protected tables | run Databricks Runtime 16.4 or above, or serverless compute |
| S8-F2-AC07 | ABAC policies are defined at catalogue level | a new table is created within an ABAC-protected catalogue | the table automatically inherits the catalogue-level ABAC policy without additional configuration |

### Technical Notes
- ABAC is in Public Preview on AWS; requires DBR 16.4+ or serverless compute per the access model wiki Section 9.
- Governed tag taxonomy must match the tagging strategy wiki Layer 2 and Layer 3 tags precisely.
- Dynamic views may be used as an additional layer for complex cross-table access logic per the access model wiki Appendix A.
- PI masking UDFs should handle all `pi_category` values (contact, billing, identity, health, location) per the access model wiki Section 6.3.
- Tags are assessed and applied explicitly per object; they are never automatically inherited from parent objects per the access model wiki Section 6.2.

---

## Feature S8-F3: Users Provisioned and Deprovisioned in Sync with Corporate Directory

**Description:** When someone joins, moves, or leaves Water Corporation in Entra ID, their Databricks access automatically follows — the right groups, the right workspace entitlements, and immediate revocation when they depart — with no manual Databricks administration required.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F3-US01 | New Team Member | be added to my Entra ID security group and automatically receive the correct Databricks access | I can start working on my first day without waiting for a separate provisioning request |
| S8-F3-US02 | IT Security Analyst | know that a departing employee's Databricks access is revoked automatically when they're deactivated in Entra ID | there are no orphaned accounts with lingering access to data assets |
| S8-F3-US03 | Domain Data Steward | trust that group membership changes in Entra ID are reflected in Databricks automatically | when someone moves to a different team, their data access adjusts without me raising a ticket |
| S8-F3-US04 | Platform Administrator | manage all identity in Entra ID as the single source of truth | no workspace-local groups exist that could bypass Unity Catalog governance |
| S8-F3-US05 | Security Officer | run a monthly reconciliation showing zero orphaned accounts and zero unsynced groups | I can demonstrate to auditors that identity governance is continuous and automated |

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

## Feature S8-F4: All Data Encrypted at Rest and in Transit with Customer-Managed Keys

**Description:** Every byte of data in EDAP — at rest in S3, on cluster local disks, and in transit between services — is encrypted using Water Corporation-controlled keys, so WC retains full custody of its encryption posture independent of AWS defaults.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F4-US01 | Security Officer | confirm that all EDAP storage is encrypted under WC-controlled customer-managed keys | we are not dependent on AWS default keys and can rotate or revoke keys on our own terms |
| S8-F4-US02 | Security Officer | confirm that all network communication uses TLS 1.2 or higher | no data traverses the wire unencrypted, meeting Essential Eight and SOCI Act requirements |
| S8-F4-US03 | Platform Administrator | rotate customer-managed keys annually without service disruption | existing data remains readable via key version history while new writes use the rotated key |
| S8-F4-US04 | Data Custodian | know that only `edap_platform_admins` can create, rotate, or delete encryption keys | key management permissions are tightly restricted, preventing accidental or unauthorised changes |
| S8-F4-US05 | Platform Administrator | run an encryption compliance audit and see a fully green report | every S3 bucket, EBS volume, and network endpoint in EDAP reports compliant encryption status |

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

## Feature S8-F5: Every Data Access Event Auditable and Searchable

**Description:** Security officers, stewards, and analysts can answer "who accessed what data, when, and from where" within minutes — querying audit logs directly in Databricks or correlating EDAP events with enterprise security data in Splunk.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F5-US01 | Security Officer | search audit logs by user, table, or time range and get results in seconds | I can investigate a suspected data breach or answer a regulator's question without waiting for someone to extract logs |
| S8-F5-US02 | Security Operations Analyst | see Databricks audit events in Splunk alongside our other enterprise security data | I can correlate EDAP access patterns with broader security events in one place |
| S8-F5-US03 | Domain Data Steward | trace the lineage of a data quality issue from Gold back through Silver and Bronze | I can pinpoint where a problem was introduced and which upstream tables were involved |
| S8-F5-US04 | Data Custodian | attribute DBU consumption to each domain and workspace in a monthly report | I can run cost chargebacks and spot domains consuming disproportionate resources |
| S8-F5-US05 | Security Officer | see every privilege grant and revoke event in the audit trail | I can verify that access control changes were authorised and appropriate |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F5-AC01 | Unity Catalog audit logging is enabled | a user queries a table in any production catalogue | the access event (user, table, timestamp, action, workspace) is captured in the system audit table within 15 minutes |
| S8-F5-AC02 | Splunk integration is configured | audit log events are generated in Databricks | events appear in the designated Splunk index within 30 minutes |
| S8-F5-AC03 | System tables are enabled | a platform admin runs a query against `system.access.audit` | the query returns structured audit data for the configured retention period |
| S8-F5-AC04 | Lineage system tables are enabled | a pipeline writes data from Bronze to Silver | the lineage event is captured in `system.access.table_lineage` and reflects the source-to-target relationship |
| S8-F5-AC05 | Privilege system tables are enabled | a domain steward grants SELECT on a schema | the grant event is captured in both the audit log and `system.information_schema.catalog_privileges` |
| S8-F5-AC06 | Billable usage system tables are enabled | a monthly cost report is generated | the report attributes DBU consumption to each workspace and catalogue (domain) |

### Technical Notes
- System tables referenced in the access model wiki Section 10.2: audit logs, billable usage, lineage, granted privileges.
- Splunk integration may use Databricks audit log delivery (account-level configuration) or a streaming pipeline from system tables to Splunk via HEC (HTTP Event Collector).
- Audit log retention should align with WC's records management requirements and the State Records Act 2000.
- Consider creating a dedicated dashboard in `prod_platform` for security monitoring.

---

## Feature S8-F6: Compliance Posture Visible on Demand with Proactive Alerting

**Description:** Security officers and data protection staff can see the organisation's compliance posture at any moment — classification coverage, access anomalies, SOCI status, PRIS Act adherence — and receive proactive alerts when something drifts out of policy, including a proven break-glass process for emergency access.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S8-F6-US01 | Security Officer | open a compliance dashboard and see this week's access patterns on restricted and privileged data | I can spot anomalies without running manual queries or waiting for a periodic report |
| S8-F6-US02 | Security Operations Analyst | receive an automated alert within 10 minutes when someone attempts to access privileged data without authorisation | I can respond to potential incidents before they escalate |
| S8-F6-US03 | Data Protection Officer | generate a PRIS Act compliance report showing who accessed PI data over any specified period | I can satisfy a regulator's request on the same day it's received |
| S8-F6-US04 | Platform Administrator | invoke the break-glass procedure and grant time-limited emergency access with two-person approval and a full audit trail | urgent operational needs are met without permanently weakening the security posture |
| S8-F6-US05 | Security Officer | see a weekly summary of all break-glass events with requester, approver, incident reference, and duration | every emergency access event is reviewed and documented — nothing slips through |
| S8-F6-US06 | Domain Data Steward | receive a notification listing my domain's tables that are missing classification tags | I can remediate classification gaps before they become audit findings |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S8-F6-AC01 | Compliance reporting is configured | the weekly reporting schedule triggers | a report is generated listing: number of access events on restricted/privileged data, top 10 accessing users, any denied access attempts, and classification coverage percentage |
| S8-F6-AC02 | Alerting rules are configured | a user attempts to access data tagged `sensitivity=privileged` without being in an authorised group | an alert is raised in Splunk and the platform team is notified within 10 minutes |
| S8-F6-AC03 | Alerting rules are configured | a privilege escalation occurs (e.g. GRANT MANAGE by a non-admin) | an alert is raised immediately and flagged for investigation |
| S8-F6-AC04 | The break-glass procedure is implemented | an emergency access request is raised and approved by a platform admin or domain executive | temporary group membership is provisioned in Entra ID with automatic expiry (4–8 hours) and the event is flagged with a dedicated audit tag |
| S8-F6-AC05 | Break-glass access has been provisioned | the expiry time is reached | group membership is automatically revoked without manual intervention |
| S8-F6-AC06 | Break-glass access has been provisioned | the weekly break-glass review is conducted | all break-glass events from the previous week are listed with: requester, approver, incident reference, data assets accessed, and duration |
| S8-F6-AC07 | Classification gap monitoring is configured | a weekly scan runs against production catalogues | all tables and schemas missing `sensitivity` or `pi_category` tags are reported to the relevant domain steward for remediation |
| S8-F6-AC08 | SOCI Act compliance monitoring is configured | an access event occurs on data tagged `soci_critical=true` | the event is logged with enhanced metadata and included in the SOCI compliance report |

### Technical Notes
- Break-glass procedure follows the 5-step process defined in the access model wiki Section 10.4: request, approval (two-person for `privileged` data), provisioning (time-limited Entra ID group membership), audit, revocation.
- Break-glass access does not bypass ABAC policies or Unity Catalog governance; it grants temporary group membership so all access remains logged and policy-filtered.
- Classification gap monitoring aligns with the tagging strategy wiki's requirement for explicit classification per object.
- SOCI Act reporting must align with mandatory incident reporting requirements to the Cyber and Infrastructure Security Centre (CISC) as referenced in the governance roles wiki.
- Compliance reports should be stored in `prod_platform` catalogue for retention and audit purposes.

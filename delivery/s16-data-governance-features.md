# S16 – Data Governance: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, classification lifecycle
- `governance/domain-governance-across-systems.md` — three-layer governance, data contracts, domain accountability
- `governance/data-governance-roles.md` — roles, RACI, stewardship responsibilities
- `lifecycles/data-governance-lifecycle.md` — governance lifecycle stages, AI model governance, regulatory alignment
- `platform/edap-access-model.md` — ABAC policies, audit logging, system tables, break-glass procedures

---

## Feature S16-F1: Audit and Monitoring Framework

**Description:** Establish a comprehensive audit and monitoring framework using Unity Catalog system tables and Databricks platform capabilities to track data access, metadata changes, and anomalous activity across the EDAP.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S16-F1-US01 | Data Protection Officer | query a centralised audit log showing who accessed PI-tagged data, when, and from which workspace | I can respond to PRIS Act 2024 enquiries with specific, timestamped evidence |
| S16-F1-US02 | Data Platform Owner | monitor for anomalous data access patterns (e.g. bulk exports, unusual query volumes, off-hours access to restricted data) | potential security incidents are detected early and escalated |
| S16-F1-US03 | Data Domain Steward | view a log of all metadata changes (tag additions, description updates, schema alterations) within my domain catalog | I can track governance activity and identify unauthorised or accidental metadata changes |
| S16-F1-US04 | Data Custodian | integrate Unity Catalog audit logs with WC's Splunk instance | security events from the EDAP are correlated with events from other corporate systems in a single monitoring platform |
| S16-F1-US05 | Data Platform Engineer | configure automated alerts for specific audit events (e.g. privilege escalation, break-glass access, ABAC policy failures) | the platform team is notified of critical events without needing to manually review logs |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S16-F1-AC01 | Unity Catalog system tables (`system.access.audit`) are enabled and populated | a platform engineer queries `system.access.audit` for data access events in the past 24 hours | the query returns records including user identity, action type, target object, timestamp, workspace, and source IP |
| S16-F1-AC02 | A user queries a PI-masked column in `prod_customer` | the audit log is reviewed | a record exists showing the user, the table/column accessed, whether masking was applied, and the ABAC policy that triggered |
| S16-F1-AC03 | Audit logs are configured to stream to Splunk | a data access event occurs in Databricks | the event appears in Splunk within 15 minutes, with all relevant fields indexed and searchable |
| S16-F1-AC04 | Anomaly detection rules are configured (e.g. >1000 rows exported from a restricted table in a single query) | a user executes a query matching the anomaly pattern | an alert is generated and sent to the security monitoring team within 30 minutes |
| S16-F1-AC05 | Break-glass access is granted to a user following the emergency access procedure | the break-glass access is used | the audit log contains a flagged entry for the break-glass event, and the weekly review process surfaces it automatically |

### Technical Notes
- Unity Catalog system tables are the primary data source: `system.access.audit`, `system.billing.usage`, `system.access.table_lineage`.
- Audit log streaming to Splunk uses Databricks' audit log delivery configuration (account-level setting delivering to S3, then ingested by Splunk).
- Anomaly detection can be implemented using Databricks SQL alerts on audit log queries, or via Splunk correlation rules for more complex patterns.
- Align anomaly detection thresholds to the governance lifecycle wiki's monitoring stage guidance.
- Break-glass access events should be identifiable by the temporary group membership pattern (time-limited `pris_authorised_*` or `soci_critical_access` membership).

---

## Feature S16-F2: Regulatory Compliance Framework

**Description:** Implement governance controls and reporting capabilities aligned to Water Corporation's regulatory obligations under the SOCI Act 2018, PRIS Act 2024, State Records Act 2000, and the Essential Eight maturity model.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S16-F2-US01 | Data Protection Officer | generate a report listing all data assets classified as containing personal information, their PI categories, applied masking policies, and access groups | I can demonstrate PRIS Act 2024 compliance to the Office of the Information Commissioner |
| S16-F2-US02 | Data Custodian | confirm that all SOCI-critical data assets have the `soci_critical=true` tag applied and row filter policies active | WC meets its critical infrastructure reporting obligations under the SOCI Act 2018 |
| S16-F2-US03 | Data Domain Steward | verify that data assets under my domain have records classification tags (`records_class`) applied per the State Records Act 2000 | retention and disposal obligations are traceable to individual data assets |
| S16-F2-US04 | Data Platform Owner | assess the EDAP's alignment to the Essential Eight maturity model and identify gaps | I can report the platform's security posture to the WC CISO and governance council |
| S16-F2-US05 | Data Governance Council member | review a quarterly compliance summary showing regulatory tag coverage, policy enforcement rates, and identified gaps | the council can make informed decisions about governance investment priorities |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S16-F2-AC01 | All production tables containing PI have been tagged with `pi_category` values | a PRIS compliance report is generated | the report lists every PI-tagged table and column, the applied masking policy, the authorised access groups, and confirms zero unmasked PI columns are accessible to non-authorised users |
| S16-F2-AC02 | All SOCI-critical data assets have been identified and tagged | a SOCI compliance query is executed | the query confirms that 100% of identified SOCI-critical assets have `soci_critical=true` tags and active row filter ABAC policies |
| S16-F2-AC03 | Records classification tags have been applied to data product tables in Gold layer | a State Records Act compliance report is generated | the report shows each data product's records class (permanent, temporary, vital) and flags any Gold-layer tables missing a `records_class` tag |
| S16-F2-AC04 | An Essential Eight self-assessment has been completed for the EDAP | the assessment is documented | a gap analysis is produced identifying areas where the EDAP does not meet the target maturity level, with remediation actions assigned and tracked |
| S16-F2-AC05 | Quarterly compliance reporting is scheduled | the end of a quarter is reached | a consolidated compliance report covering PRIS, SOCI, State Records, and Essential Eight is produced and presented to the Data Governance Council within 10 business days of quarter end |

### Technical Notes
- Compliance reports should query UC `information_schema` views for tag coverage and `system.access.audit` for enforcement evidence.
- Implement reports as parameterised Databricks SQL dashboards stored in `prod_platform` workspace, accessible to governance roles.
- The Essential Eight assessment should cover: application control (cluster policies), patching (runtime versions), MFA (Entra ID SSO), admin privilege restriction, and audit logging — all as they apply to the EDAP.
- Align to the governance lifecycle wiki's Stage 4 (Compliance and Regulatory Alignment) and Stage 6 (Monitoring and Auditing).
- SOCI Act mandatory reporting timescales (12-hour and 72-hour) should be documented in operational runbooks, not just dashboards.

---

## Feature S16-F3: Automated ABAC Provisioning

**Description:** Implement a streamlined, tag-driven access control provisioning workflow where stewardship classifications in Alation drive UC governed tags and ABAC policies with minimal manual intervention, reducing the time from classification decision to enforcement.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S16-F3-US01 | Data Domain Steward | classify a new data asset in Alation and have the appropriate access controls automatically enforced in Databricks within 24 hours | there is no manual access control provisioning step required after I complete my classification |
| S16-F3-US02 | Data Platform Engineer | define ABAC policies once at the catalog level and have them automatically apply to any new table that receives a governed tag | I do not need to create individual row filters or column masks for each new table |
| S16-F3-US03 | Data Platform Owner | track the provisioning pipeline's success rate and mean time from classification to enforcement | I can report on the automation's effectiveness and identify bottlenecks |
| S16-F3-US04 | Data Protection Officer | confirm that newly ingested data containing PI has masking enforced before any analyst can query it | there is no window of unprotected access for newly landed sensitive data |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S16-F3-AC01 | A steward classifies a new table as `sensitivity=restricted` and `pi_category=contact` in Alation | the Alation-to-UC sync pipeline and ABAC policies are in place | within 24 hours, the table is tagged in UC and queries from unauthorised users return masked PI columns |
| S16-F3-AC02 | A new table is ingested into `prod_customer.raw` by an automated pipeline | the ingestion pipeline applies default tags (e.g. `sensitivity=open`, `pi_category=none`) as part of the pipeline framework | the table has governed tags from the moment of creation, ensuring it is covered by ABAC policies from first query |
| S16-F3-AC03 | The automated provisioning pipeline has been running for 30 days | a metrics report is generated | the mean time from Alation classification to UC ABAC enforcement is less than 24 hours, and the success rate is at or above 99% |
| S16-F3-AC04 | A bulk reclassification is performed (e.g. 50 tables reclassified from `open` to `restricted`) | the sync pipeline processes the batch | all 50 tables are updated in UC within 48 hours, and ABAC policies are enforced on all of them |

### Technical Notes
- The pipeline framework should apply default governed tags at table creation time (per the tagging strategy wiki's "classify at birth" principle), ensuring no table exists without at least a baseline classification.
- The end-to-end automation chain is: pipeline creates table with default tags → steward reviews and updates classification in Alation → sync pipeline pushes updated tags to UC → ABAC policies evaluate at query time.
- ABAC policies at catalog level apply to all current and future tables — no per-table policy creation is needed.
- Monitor provisioning latency using the `prod_platform.governance` audit tables and the reconciliation outputs from S15-F3.
- Align to the governance lifecycle wiki's Stage 3 (Access and Security Governance) automation objectives.

---

## Feature S16-F4: Governance Reporting and Dashboards

**Description:** Build a suite of governance dashboards and scheduled reports covering compliance status, access audit summaries, data quality scorecards, and classification coverage, enabling the Data Governance Council and domain stewards to monitor governance health.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S16-F4-US01 | Data Governance Council member | view a dashboard showing classification coverage (% of tables with sensitivity, PI, SOCI, and records tags applied) across all domains | I can identify domains that are lagging in their classification obligations |
| S16-F4-US02 | Data Domain Steward | view a domain-specific dashboard showing data quality scores, metadata completeness, and certification status for my domain's data products | I can prioritise stewardship activities where they are most needed |
| S16-F4-US03 | Data Protection Officer | receive a weekly email report summarising PI access events, masking policy activations, and any anomalies detected | I have a regular compliance pulse without needing to log into the platform |
| S16-F4-US04 | Data Platform Owner | view a dashboard showing ABAC policy evaluation counts, sync pipeline success rates, and reconciliation status | I can monitor the operational health of the governance automation |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S16-F4-AC01 | Governance dashboards have been deployed in Databricks SQL | a Data Governance Council member navigates to the governance dashboard | they see panels for: classification coverage by domain, PI tag coverage, SOCI tag coverage, records class coverage, and trend lines over the past 90 days |
| S16-F4-AC02 | A domain-specific dashboard is configured for `prod_asset` | the Asset domain steward opens the dashboard | they see: table count, metadata completeness (% with descriptions), data quality scores (% passing DQ rules), data product certification status, and tables missing required tags |
| S16-F4-AC03 | A weekly PI access report is configured | one week elapses | an email is delivered to the DPO containing: total PI access events, unique users accessing PI data, masking activations, any anomalies flagged, and a comparison to the prior week |
| S16-F4-AC04 | Dashboards query system tables and governance audit tables | the platform team validates data freshness | all dashboard panels reflect data no older than 24 hours |
| S16-F4-AC05 | Governance dashboards have been deployed for at least 60 days | the Data Governance Council reviews the dashboards | classification coverage across all domains is measurable, with a target of ≥95% of Gold-layer tables having all four tag layers applied |

### Technical Notes
- Build dashboards using Databricks SQL Dashboards, sourcing data from `system.access.audit`, `system.information_schema.*`, and custom governance tables in `prod_platform.governance`.
- Use Databricks SQL alerts for the weekly email reports (or integrate with an email/notification service).
- Dashboard access should be restricted to governance roles — use workspace-level permissions on the SQL dashboard objects.
- Align data quality scorecards to the governance lifecycle wiki's quality framework (DQ rules, anomaly detection, freshness monitoring).
- Classification coverage metrics should track all four tagging layers from the tagging strategy wiki.

---

## Feature S16-F5: Data Stewardship Operationalisation

**Description:** Establish operational stewardship workflows in Alation and supporting processes for metadata review cadence, data product certification, and stewardship task management, aligned to the governance roles defined in WC's governance framework.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S16-F5-US01 | Data Domain Steward | follow a defined workflow in Alation to review, classify, and certify data assets within my domain on a regular cadence | stewardship is a structured, repeatable process rather than ad-hoc activity |
| S16-F5-US02 | Data Owner | see a summary of stewardship activity in my domain — reviews completed, certifications granted, outstanding items | I can hold my stewards accountable and report governance progress to the Data Governance Council |
| S16-F5-US03 | Data Domain Steward | use Alation's certification mechanism to mark data products as "Certified" after they pass the FAUQD test | data consumers can distinguish trusted, certified data products from provisional or experimental ones |
| S16-F5-US04 | Technical Data Steward | receive notifications when new tables are created in my domain catalog that require classification review | newly ingested data does not remain unclassified beyond the defined review window |
| S16-F5-US05 | Data Governance Council member | review stewardship KPIs (e.g. time to classify new assets, certification rate, metadata completeness) across all domains | I can assess whether the stewardship operating model is effective |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S16-F5-AC01 | A stewardship review cadence has been defined (e.g. monthly per domain) | the review period arrives | the Domain Steward receives a task list in Alation of new or modified assets requiring review, with clear instructions on the review actions required |
| S16-F5-AC02 | A data product in `prod_asset.product` has passed FAUQD testing | the Domain Steward certifies it in Alation | the certification status is visible to all Alation users, and the `data_product_tier=certified` tag is synced to Unity Catalog via the S15-F1 sync pipeline |
| S16-F5-AC03 | A new table is created in `prod_operations.raw` | 5 business days elapse | the Operations Domain Steward has been notified, and the table has been reviewed and classified — or flagged as overdue if not |
| S16-F5-AC04 | Stewardship KPIs have been configured | the Data Governance Council reviews the quarterly report | the report includes: average days to classify new assets, percentage of Gold-layer tables certified, metadata completeness by domain, and stewardship task completion rate |
| S16-F5-AC05 | A stewardship operating model document has been produced | the document is reviewed by the Data Governance Council | it defines: review cadence per domain, escalation paths for overdue classifications, certification prerequisites (FAUQD), and RACI for stewardship tasks aligned to the governance roles wiki |

### Technical Notes
- Leverage Alation's built-in stewardship features: trust flags (Endorsed, Warning, Deprecated), custom workflows, and curation tasks.
- Map Alation trust flags to UC `data_product_tier` governed tag values: Endorsed → certified, Warning → provisional, Deprecated → deprecated.
- New-asset notifications can be triggered by the metadata harvest (S14-F4) — when new objects appear in Alation, alert the relevant domain steward group.
- The FAUQD test (Findable, Accessible, Understandable, Quality, Dependable) is the certification prerequisite per the access model wiki — implement a checklist in Alation or as a linked artefact.
- Stewardship KPIs should feed into the governance dashboards (S16-F4).
- Align the RACI for stewardship tasks to the governance roles wiki: Data Owner (Accountable), Domain Steward (Responsible), Technical Steward (Consulted), Data Platform Owner (Informed).

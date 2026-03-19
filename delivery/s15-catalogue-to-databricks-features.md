# S15 – Integration from Data Catalogue to Databricks: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `platform/edap-access-model.md` — ABAC policies, governed tags, PI masking, SOCI row filters
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, Alation-to-UC tag sync direction
- `governance/domain-governance-across-systems.md` — three-layer governance, classification lifecycle
- `governance/data-governance-roles.md` — Data Domain Steward, Technical Data Steward, RACI

---

## Feature S15-F1: Business Classifications from Alation Drive Access Control in Databricks

**Description:** When a steward classifies a data asset in Alation — applying sensitivity, PI category, SOCI, or records tags — those classifications automatically propagate to Unity Catalog as governed tags. Alation is the single place stewards work; UC enforces their decisions without a separate engineering request.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S15-F1-US01 | Data Domain Steward | apply a sensitivity classification (e.g. `restricted`) to a table in Alation and have it automatically propagate to the corresponding Unity Catalog governed tag | I can perform classification in a single tool without needing Databricks access |
| S15-F1-US02 | Data Domain Steward | apply a PI category tag (e.g. `pi_category=billing`) to a column in Alation and have it sync to the UC column-level governed tag | column-level masking policies in UC are activated based on my stewardship decisions |
| S15-F1-US03 | Data Platform Owner | ensure that only tags from the approved WC governed tag taxonomy can be synced to UC | ad-hoc or non-standard Alation fields do not create ungoverned tags in Unity Catalog |
| S15-F1-US04 | Data Platform Engineer | rely on an automated sync pipeline that maps Alation custom fields to UC governed tag keys and values | tag propagation does not depend on manual intervention or ad-hoc scripts |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S15-F1-AC01 | A Data Domain Steward sets `sensitivity=restricted` on a table in Alation | the sync pipeline executes | the corresponding table in Unity Catalog has the governed tag `sensitivity` set to `restricted`, verifiable via `SELECT * FROM system.information_schema.table_tags` |
| S15-F1-AC02 | A Data Domain Steward sets `pi_category=contact` on a column in Alation | the sync pipeline executes | the corresponding column in Unity Catalog has the governed tag `pi_category` set to `contact` |
| S15-F1-AC03 | A steward attempts to set a tag value in Alation that is not in the UC governed tag policy's allowed values | the sync pipeline processes the change | the sync rejects the invalid value, logs an error, and does not create or modify any UC tag |
| S15-F1-AC04 | Tags have been synced for all tables in `prod_customer` catalogue | a reconciliation query is run comparing Alation tag values to UC governed tag values | 100% of synced tags match between the two systems with no discrepancies |
| S15-F1-AC05 | A tag value is changed in Alation (e.g. `sensitivity` updated from `open` to `restricted`) | the next sync cycle completes | the UC governed tag reflects the updated value, and the change is captured in the UC audit log |

### Technical Notes
- The sync mechanism should use the Databricks REST API or Unity Catalog SQL commands (`ALTER TABLE ... SET TAGS`) to apply governed tags. Alation's API provides the source of truth.
- Tag mapping must align to the 4-layer tagging model: Layer 1 (WAICP classification), Layer 2 (Regulatory — PRIS, SOCI, State Records), Layer 3 (Platform — sensitivity, data_product_tier), Layer 4 (Business — domain-specific).
- Only tags defined in the UC tag policy are valid sync targets. The sync pipeline must validate values against the tag policy before attempting to write.
- The sync service principal requires `APPLY TAG` privilege on all production catalogues.
- Consider using Lakeflow Jobs to orchestrate the sync as a scheduled Databricks notebook or Python task, ensuring it is monitored and logged within the EDAP operational framework.

---

## Feature S15-F2: Access Restrictions Enforce Automatically When Stewards Classify Data

**Description:** A steward's classification decision in Alation has immediate, enforceable security outcomes. When a classification propagates to Unity Catalog, ABAC policies evaluate the governed tags at query time — masking PI columns and filtering SOCI-critical rows — without a separate engineering request.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S15-F2-US01 | Data Domain Steward | know that when I classify a column as containing personal information in Alation, the data is automatically masked for unauthorised users in Databricks | my classification decisions have immediate, enforceable security outcomes without requiring a separate engineering request |
| S15-F2-US02 | Data Protection Officer | verify that PI masking is active for all columns tagged with a `pi_category` value other than `none` | I can confirm regulatory compliance with the PRIS Act 2024 across the entire platform |
| S15-F2-US03 | Data Consumer | query a table containing masked columns and receive masked values for PI columns I am not authorised to see | I can still use the table for analytics without being exposed to sensitive data |
| S15-F2-US04 | Data Platform Engineer | define ABAC policies once at the catalogue level and have them automatically apply to any table or column that receives the relevant tag | policies are defined once and automatically apply to any current or future tagged asset |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S15-F2-AC01 | A column has been tagged `pi_category=billing` via Alation-to-UC sync and a PI masking ABAC policy is active on the catalogue | a user who is NOT a member of `pris_authorised_billing` queries the column | the column returns `***MASKED***` for all rows |
| S15-F2-AC02 | The same column is queried by a user who IS a member of `pris_authorised_billing` | the user executes a SELECT on the column | the column returns unmasked values |
| S15-F2-AC03 | A table is tagged `soci_critical=true` via Alation-to-UC sync and a SOCI row filter ABAC policy is active | a user who is NOT a member of `soci_critical_access` queries the table | the query returns zero rows (row filter applied) |
| S15-F2-AC04 | A new table is created, and within 24 hours a steward tags it with `sensitivity=privileged` in Alation | the tag syncs to UC and the ABAC policy evaluates on next query | access is restricted per the `privileged` sensitivity policy — only members of approved groups can query the table |
| S15-F2-AC05 | ABAC policies are deployed across all `prod_*` catalogues | the platform team queries `system.access.audit` for ABAC evaluation events | policy evaluations are logged for every query against tagged objects, confirming enforcement is active |

### Technical Notes
- ABAC policies must be defined at the catalogue level using `CREATE ATTRIBUTE ACCESS POLICY` in Unity Catalog, referencing governed tag keys (`sensitivity`, `pi_category`, `soci_critical`).
- The PI masking UDF (`prod_platform.governance.mask_pi`) should be deployed per the access model wiki's specification, checking `is_account_group_member()` against the relevant `pris_authorised_<category>` group.
- ABAC requires Databricks Runtime 16.4+ or serverless compute. Cluster policies must enforce this minimum runtime version.
- The end-to-end flow is: steward classifies in Alation → tag syncs to UC → ABAC policy evaluates tag at query time → masking/filtering applied. No additional engineering intervention should be required after the initial policy setup.
- Test with both shared clusters and SQL Warehouses to confirm ABAC enforcement is consistent across compute types.

---

## Feature S15-F3: Tag Sync Verified End-to-End with Discrepancies Flagged

**Description:** The platform team and governance officers can verify at any time that every tag in Alation has a matching governed tag in Unity Catalog and that the corresponding ABAC policy is enforcing correctly. Mismatches are flagged automatically so that compliance gaps cannot go undetected.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S15-F3-US01 | Data Platform Owner | run a reconciliation report comparing all tag values in Alation against their corresponding governed tags in Unity Catalog | I can detect and remediate any sync drift or failures |
| S15-F3-US02 | Data Domain Steward | receive an alert if a tag I applied in Alation has not propagated to UC within the expected timeframe | I can escalate sync issues before they create compliance gaps |
| S15-F3-US03 | Data Protection Officer | review a compliance report showing that all PI-tagged columns in Alation have corresponding masking policies active in UC | I can demonstrate to auditors that classification and enforcement are aligned |
| S15-F3-US04 | Data Platform Engineer | run an automated end-to-end test that applies a tag in Alation, triggers a sync, and validates the ABAC policy outcome in Databricks | I can confirm the full pipeline works correctly after any configuration change |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S15-F3-AC01 | Tags have been synced from Alation to UC for all production catalogues | a reconciliation job runs comparing Alation custom field values to UC `information_schema.table_tags` and `information_schema.column_tags` | a report is generated listing any mismatches, with zero mismatches constituting a pass |
| S15-F3-AC02 | A tag sync fails for a specific table (e.g. due to a renamed object) | the reconciliation job detects the discrepancy | the platform team receives an alert identifying the specific object and the nature of the mismatch within one reconciliation cycle |
| S15-F3-AC03 | An end-to-end validation test is executed | the test applies `pi_category=identity` to a test column in Alation, triggers sync, and queries the column as an unauthorised user | the query returns masked values, confirming the full chain (Alation → sync → UC tag → ABAC policy → masked result) is operational |
| S15-F3-AC04 | A monthly compliance reconciliation is scheduled | the report is generated | it includes: total objects tagged in Alation, total governed tags in UC, match percentage, list of discrepancies, and ABAC policy coverage (percentage of tagged objects with an active policy) |
| S15-F3-AC05 | Reconciliation has been running for 30 days | the platform team reviews reconciliation history | tag sync accuracy is at or above 99.5% across all production catalogues |

### Technical Notes
- Reconciliation should query both Alation's API (to retrieve current tag values) and UC's `information_schema` views (to retrieve current governed tag values), then compare.
- Implement as a scheduled Databricks notebook or Lakeflow Job that writes results to a governance audit table in `prod_platform.governance`.
- The end-to-end validation test should be part of the CI/CD pipeline for any changes to the sync mechanism or ABAC policy definitions.
- Reconciliation reports should feed into the governance dashboards (S16-F4).
- Align to the three-layer governance model: the reconciliation validates that Layer 1 (Alation stewardship) is correctly reflected in Layer 2 (UC enforcement).

# S14 – EDP to Data Catalogue: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, Alation vs UC Discover roles
- `governance/domain-governance-across-systems.md` — three-layer governance, cross-system lineage
- `platform/edap-access-model.md` — catalog layout, governed tags, system tables for audit/lineage

---

## Feature S14-F1: All EDAP Data Assets Discoverable in Alation with Full Metadata

**Description:** Any WC staff member can search Alation and find every table, view, column, description, governed tag, and data product tier across all EDAP domain catalogues — evaluating data assets from a single enterprise catalogue without needing Databricks access.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S14-F1-US01 | Data Consumer | search Alation and find every EDAP data product with its descriptions, column-level metadata, and data product tier | I can discover and evaluate data assets without needing a Databricks login |
| S14-F1-US02 | Data Domain Steward | see all tables and views from my domain catalogue in Alation with accurate schema and column details | I can perform stewardship activities (descriptions, classifications, certifications) in one place |
| S14-F1-US03 | Data Platform Owner | confirm that every production catalogue and schema in Unity Catalog is represented in Alation | I know the enterprise catalogue gives a complete picture of the EDAP |
| S14-F1-US04 | Data Custodian | verify that the metadata harvesting pathway is encrypted and does not traverse untrusted networks | I can confirm the integration meets WC's Essential Eight and network segmentation requirements |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S14-F1-AC01 | The Alation OCF connector for Databricks is installed and configured with a dedicated service principal | a user searches for a known table in Alation | the table appears with correct catalogue, schema, table name, all columns, column data types, and column descriptions matching Unity Catalog |
| S14-F1-AC02 | Tables in Unity Catalog have governed tags applied (e.g. `sensitivity=restricted`, `pi_category=contact`) | a metadata harvest completes | the corresponding Alation data source objects display these tag values as custom fields, with values matching UC exactly |
| S14-F1-AC03 | A new table is created in `prod_customer.product` schema in Unity Catalog | the next scheduled metadata harvest runs | the new table appears in Alation within one harvest cycle, with all columns, descriptions, and tags |
| S14-F1-AC04 | A table is dropped from Unity Catalog | the next metadata harvest runs | the table is flagged as deprecated or removed in Alation (per configured stale object handling), not left as a phantom entry |
| S14-F1-AC05 | Metadata harvesting is executed across all seven domain production catalogues plus `prod_reference` and `prod_platform` | the harvest completes | the total object count in Alation matches the total object count in Unity Catalog's `information_schema` views, with zero discrepancies |

### Technical Notes
- Use a dedicated Databricks service principal for the Alation connector, registered at the account level and synchronised via SCIM per the access model wiki.
- The connector must target the production workspace (`wc-edap-prod`) with read-only catalogue bindings for all `prod_*` catalogues.
- Alation OCF connectors for Databricks support Unity Catalog natively — ensure the connector version supports governed tags and system table lineage.
- The service principal requires only `SELECT` and `USE` privileges — no `MODIFY`, `MANAGE`, or `CREATE` privileges.
- Network path must comply with WC's Essential Eight controls (TLS 1.2+, no public internet traversal if Alation is hosted on-premises or in a peered VPC).
- Harvest scope must include all `prod_*` catalogues, including `prod_reference` and `prod_platform`.
- Map UC governed tags to Alation custom fields using the 4-layer tagging model from the tagging strategy wiki: Layer 1 (WAICP), Layer 2 (Regulatory), Layer 3 (Platform), Layer 4 (Business).
- Configure stale metadata handling: objects removed from UC should be flagged, not silently retained in Alation.
- Table and column comments in Unity Catalog (`COMMENT ON`) must be harvested into Alation's description fields.
- The `data_product_tier` tag (certified, provisional, experimental, deprecated) should map to Alation's trust flag or certification mechanism.

---

## Feature S14-F2: End-to-End Data Lineage Visible from Source Through to Gold Layer

**Description:** A steward, analyst, or governance officer can trace any data product back through its full pipeline lineage — from source system ingestion through Bronze, Silver, and Gold layers — including column-level transformations and cross-component flows involving non-Databricks AWS services, all visible in Alation.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S14-F2-US01 | Data Domain Steward | view end-to-end lineage in Alation from source system ingestion through Bronze/Silver/Gold layers to the published data product | I can trace data provenance for regulatory enquiries and impact analysis |
| S14-F2-US02 | Data Consumer | see which upstream tables and columns feed into a Gold-layer data product | I can assess the reliability and provenance of data before using it in reports or models |
| S14-F2-US03 | Data Platform Owner | see lineage that spans both Databricks components and non-Databricks AWS services (e.g. S3, Glue, Lambda) in a single Alation lineage view | the enterprise catalogue provides a complete picture of data flow, not just the Databricks portion |
| S14-F2-US04 | Data Protection Officer | trace how personal information flows from source systems through to consumption endpoints | I can respond to PRIS Act 2024 enquiries with specific, documented data flows |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S14-F2-AC01 | A Lakeflow Declarative Pipeline writes data from `raw` to `base` to `curated` to `product` schemas within a domain catalogue | lineage harvesting completes | Alation displays table-level lineage showing the full pipeline flow across all four schemas |
| S14-F2-AC02 | Unity Catalog has captured column-level lineage for a transformation that derives `full_name` from `first_name` and `last_name` | lineage harvesting completes | Alation displays column-level lineage showing both source columns contributing to the derived column |
| S14-F2-AC03 | An S3 bucket is configured as an external data source feeding into a landing zone Volume in Unity Catalog | lineage is harvested | Alation displays the S3 source as an upstream node in the lineage graph, connected to the corresponding `raw` schema table |
| S14-F2-AC04 | A non-Databricks AWS service (e.g. AWS Glue job or Lambda function) produces data consumed by an EDAP pipeline | lineage configuration is complete | the AWS service appears as a lineage node in Alation, with manual or API-driven lineage links bridging the Databricks and non-Databricks components |
| S14-F2-AC05 | Lineage harvesting is configured for all production catalogues | a full lineage harvest completes | at least 90% of tables in Silver and Gold layers have at least one upstream lineage link visible in Alation |

### Technical Notes
- Unity Catalog system tables (`system.access.table_lineage`, `system.access.column_lineage`) are the primary source for Databricks lineage. The OCF connector should harvest from these tables.
- For non-Databricks AWS services, Alation's open API or manual lineage capabilities will be required to create lineage nodes and edges. Document the process for registering external lineage sources.
- Cross-component lineage aligns to the three-layer governance model (domain governance wiki): source system → EDAP → consumption layer.
- Column-level lineage requires that Unity Catalog lineage capture is enabled and that pipelines run on compute that supports lineage (shared access mode or serverless).
- Lineage completeness should be measured and reported as a governance KPI.

---

## Feature S14-F3: Metadata Stays Current Automatically Without Manual Refresh

**Description:** Stewards and consumers always see current metadata and lineage in Alation because synchronisation runs automatically on a scheduled cadence with delta extraction — and the platform team is alerted immediately if synchronisation fails or falls behind.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S14-F3-US01 | Data Domain Steward | trust that the metadata I see in Alation reflects the current state of Unity Catalog | I can make stewardship decisions based on accurate, up-to-date information |
| S14-F3-US02 | Data Platform Owner | know that a scheduled harvest runs daily and completes automatically without manual intervention | the enterprise catalogue is never more than 24 hours behind the platform |
| S14-F3-US03 | Data Platform Engineer | rely on delta (incremental) synchronisation rather than full extraction on each run | harvest jobs complete efficiently without placing unnecessary load on Databricks or Alation |
| S14-F3-US04 | Data Platform Owner | receive automated alerts when a scheduled harvest fails or completes with errors | I can remediate sync issues before metadata staleness affects stewardship or discovery |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S14-F3-AC01 | A daily metadata harvest schedule has been configured in Alation | 24 hours elapse after a new table is created in Unity Catalog | the new table is visible in Alation without manual intervention |
| S14-F3-AC02 | Delta sync is enabled for the Databricks OCF connector | a scheduled harvest runs after only minor changes in Unity Catalog | the harvest completes in less than 50% of the time required for a full extraction, and only changed objects are updated |
| S14-F3-AC03 | A harvest job fails due to a connectivity or authentication error | the failure is detected | an alert is sent to the platform team's notification channel (email or messaging integration) within 15 minutes of failure |
| S14-F3-AC04 | Harvest schedules are running in production | a platform engineer queries harvest job history | a log of all harvest runs is available showing start time, end time, objects processed, objects added, objects updated, objects removed, and any errors |
| S14-F3-AC05 | Metadata freshness monitoring is configured | a domain steward views a data source object in Alation | a "last synced" timestamp is visible, and objects not refreshed within the expected window (e.g. >36 hours) are flagged |

### Technical Notes
- Schedule metadata harvesting to run daily during a low-activity window (e.g. 02:00–04:00 AWST) to minimise compute contention.
- Lineage harvesting may run on a separate, less frequent schedule (e.g. weekly) if lineage data changes less frequently than structural metadata.
- Delta sync should leverage Unity Catalog's `information_schema` timestamps or Alation's built-in incremental extraction capability.
- Alerting should integrate with WC's existing monitoring and notification tooling (e.g. email, Microsoft Teams, or Splunk alerts).
- Document the expected harvest duration and establish a performance baseline to detect degradation over time.

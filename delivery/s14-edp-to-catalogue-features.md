# S14 – Integration from EDP to Data Catalogue: Feature Breakdown

**Scope Area:** EDP Implementation
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `governance/edap-tagging-strategy.md` — 4-layer tagging model, Alation vs UC Discover roles
- `governance/domain-governance-across-systems.md` — three-layer governance, cross-system lineage
- `platform/edap-access-model.md` — catalog layout, governed tags, system tables for audit/lineage

---

## Feature S14-F1: Alation OCF Connector Setup

**Description:** Install and configure the Alation Open Connector Framework (OCF) connector for Databricks Unity Catalog, establishing authenticated connectivity between Alation and the EDAP production environment.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S14-F1-US01 | Data Platform Engineer | install and configure the Alation OCF connector for Databricks | Alation can connect to Unity Catalog and harvest metadata from all EDAP domain catalogs |
| S14-F1-US02 | Data Platform Engineer | configure service principal authentication between Alation and Databricks | the connector uses a secure, least-privilege identity that does not rely on individual user credentials |
| S14-F1-US03 | Data Platform Owner | verify that the OCF connector can enumerate all production catalogs and schemas | I can confirm that the connector has appropriate visibility across the EDAP catalog layout |
| S14-F1-US04 | Data Custodian | ensure the connector's network path complies with WC's network segmentation and encryption requirements | metadata harvesting traffic is encrypted in transit and does not traverse untrusted networks |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S14-F1-AC01 | The Alation OCF connector for Databricks is installed in the Alation instance | the connector is configured with the EDAP production workspace URL and a dedicated service principal | the connector successfully authenticates and returns a list of all `prod_*` catalogs |
| S14-F1-AC02 | The service principal used by the connector has been provisioned in Unity Catalog | the connector attempts to access catalog metadata | only `SELECT` and `USE` privileges are granted — no `MODIFY`, `MANAGE`, or `CREATE` privileges exist on the service principal |
| S14-F1-AC03 | Network connectivity between Alation and Databricks has been configured | a test connection is executed from the Alation admin console | the connection succeeds over TLS with no certificate warnings, and the round-trip latency is under 2 seconds |
| S14-F1-AC04 | The OCF connector is configured | a manual metadata extraction is triggered | the connector completes without errors and logs are available in Alation's job history |

### Technical Notes
- Use a dedicated Databricks service principal for the Alation connector, registered at the account level and synchronised via SCIM per the access model wiki.
- The connector must target the production workspace (`wc-edap-prod`) with read-only catalog bindings for all `prod_*` catalogs.
- Alation OCF connectors for Databricks support Unity Catalog natively — ensure the connector version supports governed tags and system table lineage.
- Network path must comply with WC's Essential Eight controls (TLS 1.2+, no public internet traversal if Alation is hosted on-premises or in a peered VPC).

---

## Feature S14-F2: Metadata Harvesting

**Description:** Configure and validate the harvesting of structural and descriptive metadata from Unity Catalog into Alation, covering tables, views, schemas, columns, descriptions, governed tags, and data product tier classifications.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S14-F2-US01 | Data Domain Steward | see all tables and views from my domain catalog in Alation with accurate schema and column details | I can perform stewardship activities (descriptions, classifications, certifications) in a single enterprise catalogue |
| S14-F2-US02 | Data Consumer | find EDAP data products in Alation with their descriptions, column-level metadata, and data product tier | I can discover and evaluate data products without needing direct access to Databricks |
| S14-F2-US03 | Data Platform Engineer | configure the harvester to capture Unity Catalog governed tags and surface them as custom fields in Alation | business classifications (sensitivity, PI category, SOCI critical) applied in UC are visible in the enterprise catalogue |
| S14-F2-US04 | Data Domain Steward | see table and column descriptions authored in Unity Catalog reflected in Alation | metadata authored by engineers during pipeline development is not lost and does not need to be re-entered |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S14-F2-AC01 | A metadata harvest has completed for `prod_asset` catalog | a user searches for a known table in Alation | the table appears with correct catalog, schema, table name, all columns, column data types, and column descriptions matching Unity Catalog |
| S14-F2-AC02 | Tables in Unity Catalog have governed tags applied (e.g. `sensitivity=restricted`, `pi_category=contact`) | a metadata harvest completes | the corresponding Alation data source objects display these tag values as custom fields, with values matching UC exactly |
| S14-F2-AC03 | A new table is created in `prod_customer.product` schema in Unity Catalog | the next scheduled metadata harvest runs | the new table appears in Alation within one harvest cycle, with all columns, descriptions, and tags |
| S14-F2-AC04 | A table is dropped from Unity Catalog | the next metadata harvest runs | the table is flagged as deprecated or removed in Alation (per configured stale object handling), not left as a phantom entry |
| S14-F2-AC05 | Metadata harvesting is executed across all seven domain production catalogs plus `prod_reference` and `prod_platform` | the harvest completes | the total object count in Alation matches the total object count in Unity Catalog's `information_schema` views, with zero discrepancies |

### Technical Notes
- Harvest scope must include all `prod_*` catalogs, including `prod_reference` and `prod_platform`.
- Map UC governed tags to Alation custom fields using the 4-layer tagging model from the tagging strategy wiki: Layer 1 (WAICP), Layer 2 (Regulatory), Layer 3 (Platform), Layer 4 (Business).
- Configure stale metadata handling: objects removed from UC should be flagged, not silently retained in Alation.
- Table and column comments in Unity Catalog (`COMMENT ON`) must be harvested into Alation's description fields.
- The `data_product_tier` tag (certified, provisional, experimental, deprecated) should map to Alation's trust flag or certification mechanism.

---

## Feature S14-F3: Lineage Harvesting

**Description:** Configure harvesting of pipeline lineage and column-level lineage from Unity Catalog system tables into Alation, including cross-component lineage for non-Databricks AWS services that form part of the end-to-end data flow.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S14-F3-US01 | Data Domain Steward | view end-to-end lineage in Alation from source system ingestion through Bronze/Silver/Gold layers to the published data product | I can trace data provenance for regulatory enquiries and impact analysis |
| S14-F3-US02 | Data Platform Engineer | configure Alation to harvest column-level lineage captured by Unity Catalog | stewards and analysts can trace individual column transformations across pipeline stages |
| S14-F3-US03 | Data Platform Owner | see lineage that spans both Databricks components and non-Databricks AWS services (e.g. S3, Glue, Lambda) in a single Alation lineage view | the enterprise catalogue provides a complete picture of data flow, not just the Databricks portion |
| S14-F3-US04 | Data Consumer | understand which upstream tables and columns feed into a Gold-layer data product | I can assess the reliability and provenance of data before using it in reports or models |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S14-F3-AC01 | A Lakeflow Declarative Pipeline writes data from `raw` to `base` to `curated` to `product` schemas within a domain catalog | lineage harvesting completes | Alation displays table-level lineage showing the full pipeline flow across all four schemas |
| S14-F3-AC02 | Unity Catalog has captured column-level lineage for a transformation that derives `full_name` from `first_name` and `last_name` | lineage harvesting completes | Alation displays column-level lineage showing both source columns contributing to the derived column |
| S14-F3-AC03 | An S3 bucket is configured as an external data source feeding into a landing zone Volume in Unity Catalog | lineage is harvested | Alation displays the S3 source as an upstream node in the lineage graph, connected to the corresponding `raw` schema table |
| S14-F3-AC04 | A non-Databricks AWS service (e.g. AWS Glue job or Lambda function) produces data consumed by an EDAP pipeline | lineage configuration is complete | the AWS service appears as a lineage node in Alation, with manual or API-driven lineage links bridging the Databricks and non-Databricks components |
| S14-F3-AC05 | Lineage harvesting is configured for all production catalogs | a full lineage harvest completes | at least 90% of tables in Silver and Gold layers have at least one upstream lineage link visible in Alation |

### Technical Notes
- Unity Catalog system tables (`system.access.table_lineage`, `system.access.column_lineage`) are the primary source for Databricks lineage. The OCF connector should harvest from these tables.
- For non-Databricks AWS services, Alation's open API or manual lineage capabilities will be required to create lineage nodes and edges. Document the process for registering external lineage sources.
- Cross-component lineage aligns to the three-layer governance model (domain governance wiki): source system → EDAP → consumption layer.
- Column-level lineage requires that Unity Catalog lineage capture is enabled and that pipelines run on compute that supports lineage (shared access mode or serverless).
- Lineage completeness should be measured and reported as a governance KPI.

---

## Feature S14-F4: Automated Sync Schedule

**Description:** Configure recurring metadata and lineage harvesting schedules with delta synchronisation, freshness monitoring, and alerting to ensure Alation remains current with Unity Catalog.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S14-F4-US01 | Data Platform Owner | configure an automated daily metadata harvest from Unity Catalog to Alation | the enterprise catalogue is never more than 24 hours behind the platform |
| S14-F4-US02 | Data Platform Engineer | implement delta (incremental) synchronisation rather than full extraction on each run | harvest jobs complete efficiently without placing unnecessary load on Databricks or Alation |
| S14-F4-US03 | Data Platform Owner | receive automated alerts when a scheduled harvest fails or completes with errors | I can remediate sync issues before metadata staleness affects stewardship or discovery |
| S14-F4-US04 | Data Domain Steward | view a freshness indicator on data source objects in Alation showing when metadata was last synchronised | I can trust that the metadata I see reflects the current state of Unity Catalog |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S14-F4-AC01 | A daily metadata harvest schedule has been configured in Alation | 24 hours elapse after a new table is created in Unity Catalog | the new table is visible in Alation without manual intervention |
| S14-F4-AC02 | Delta sync is enabled for the Databricks OCF connector | a scheduled harvest runs after only minor changes in Unity Catalog | the harvest completes in less than 50% of the time required for a full extraction, and only changed objects are updated |
| S14-F4-AC03 | A harvest job fails due to a connectivity or authentication error | the failure is detected | an alert is sent to the platform team's notification channel (email or messaging integration) within 15 minutes of failure |
| S14-F4-AC04 | Harvest schedules are running in production | a platform engineer queries harvest job history | a log of all harvest runs is available showing start time, end time, objects processed, objects added, objects updated, objects removed, and any errors |
| S14-F4-AC05 | Metadata freshness monitoring is configured | a domain steward views a data source object in Alation | a "last synced" timestamp is visible, and objects not refreshed within the expected window (e.g. >36 hours) are flagged |

### Technical Notes
- Schedule metadata harvesting to run daily during a low-activity window (e.g. 02:00–04:00 AWST) to minimise compute contention.
- Lineage harvesting may run on a separate, less frequent schedule (e.g. weekly) if lineage data changes less frequently than structural metadata.
- Delta sync should leverage Unity Catalog's `information_schema` timestamps or Alation's built-in incremental extraction capability.
- Alerting should integrate with WC's existing monitoring and notification tooling (e.g. email, Microsoft Teams, or Splunk alerts).
- Document the expected harvest duration and establish a performance baseline to detect degradation over time.

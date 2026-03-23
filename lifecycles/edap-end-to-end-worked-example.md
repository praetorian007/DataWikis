# EDAP End-to-End Worked Example — From Business Request to Live Dashboard

**Mark Shaw** | Principal Data Architect

---

## Introduction

This document traces a single business request — a dashboard showing water main break rates — from initial ask through to a live production dashboard in the Enterprise Data & Analytics Platform (EDAP). The journey touches every medallion layer, every governance role, and every major platform capability. It is intended as both a teaching tool for new team members and a reference for how the pieces of the EDAP fit together in practice.

The scenario is realistic. It involves cross-domain data, a new source system onboarding, dimensional modelling, data product certification, and a Power BI dashboard — the kind of request that exercises the full breadth of the platform.

---

## 1. The Business Request

Water Corporation manages thousands of kilometres of water mains across Western Australia. When a main breaks, the consequences are immediate and costly: service disruption to customers, water loss, road damage, emergency crew mobilisation, and potential public safety risks. Over time, ageing infrastructure increases break frequency, and the capital replacement programme must prioritise which mains to replace first.

The General Manager of Operations wants a monthly dashboard that answers a deceptively simple question: **where are our mains breaking, and why?**

Specifically, the GM wants to see:

- **Break rate** (breaks per kilometre per year) by asset material type, age band, and geographic area
- **Mean time to repair** and **repair cost** trends
- **Repeat break rate** — mains that have broken more than once in a rolling twelve-month window
- A **geographic heat map** of break hotspots by pressure zone and suburb
- **Trend analysis** — monthly, quarterly, and year-over-year comparisons to identify seasonal patterns and long-term deterioration

The capital team will use this dashboard to build a data-driven replacement priority matrix, shifting from reactive maintenance to planned asset renewal.

The request is logged through the BI team's intake process — a structured form that captures the business question, the intended audience, the desired refresh frequency (monthly, in this case), and the expected data sources. This intake step ensures every request is triaged, scoped, and tracked before any development begins.

---

## 2. Requirements and Discovery

**Roles involved:** BI Developer, Data Architect, Data Domain Steward (Operations), Data Domain Steward (Asset)

### 2.1 Requirements Workshop

The BI Developer facilitates a requirements workshop with the GM's team, a Data Architect, and the Data Domain Stewards for Operations and Asset. The workshop produces a clear specification.

**Metrics required:**

| Metric | Definition |
|---|---|
| Break rate | Number of pipe break events per kilometre of main per year |
| Mean time to repair (MTTR) | Average hours from break reported to break repaired |
| Repeat break rate | Percentage of mains with more than one break in a rolling 12-month window |
| Break cost | Total repair cost per break event |
| Compliance % | Percentage of breaks repaired within SLA timeframe |

**Dimensions required:**

| Dimension | Attributes |
|---|---|
| Asset material type | Cast iron, ductile iron, PVC, asbestos cement, copper, polyethylene, etc. |
| Age band | 0–10 years, 11–20 years, 21–40 years, 41–60 years, 60+ years |
| Diameter | Nominal pipe diameter in millimetres |
| Pressure zone | Hydraulic pressure zone identifier |
| Geographic area | Suburb, town, region |
| Time | Month, quarter, financial year |

**Data sources identified:**

| Source System | Domain | Data |
|---|---|---|
| Maximo | Operations | Work orders for pipe break events — break type, reported date, repair date, cost, crew |
| SAP ECC | Asset | Asset register — pipe material, install date, diameter, asset class, length |
| ESRI GIS | Asset | Spatial boundaries — pressure zones, suburbs, regions, pipe route geometry |

### 2.2 Cross-Domain Dependency

This request spans two data domains. The Operations domain owns the break events (work orders from Maximo). The Asset domain owns the pipe attributes (SAP equipment register) and spatial data (ESRI GIS). The enriched dataset that joins them will require a **cross-domain data contract** — a formal agreement between the two domains on the join logic, the business rules, and the ownership of the resulting product.

### 2.3 Regulatory Context

The Data Architect flags that some asset data may be SOCI-critical under the Security of Critical Infrastructure Act 2018. Precise infrastructure locations — particularly pressure zone boundaries and pipeline routes — could reveal vulnerabilities in the water supply network. This will need to be assessed during classification.

No Personal Information (PI) is involved in this request. The work orders contain crew identifiers, but these will not be carried into the analytical dataset.

### 2.4 Data Catalogue Discovery

The Data Architect checks the Data Catalogue (Alation) to determine what already exists in the EDAP.

| Asset | Status in EDAP | Location |
|---|---|---|
| Maximo work orders | Ingested to Bronze Raw | `prod_bronze.maximo_raw.work_order` |
| SAP equipment register | Ingested to Bronze Raw and processed to Silver Base | `prod_bronze.sap_raw.equipment`, `prod_silver.sap_base.base_equipment` |
| ESRI GIS spatial boundaries | **Not yet in EDAP** | Needs onboarding |

The SAP equipment table in Silver Base (`prod_silver.sap_base.base_equipment`) is confirmed to contain `material_type`, `install_date`, `diameter`, `asset_class`, and `pipe_length_m`, and is SCD Type 2-tracked with `edap_eff_from`, `edap_eff_to`, and `edap_is_current` columns.

Maximo work orders are in Bronze Raw but have not yet been processed to Silver Base — this pipeline needs to be built.

GIS spatial data is not in the platform at all — this requires a full onboarding from source.

---

## 3. Data Onboarding — GIS Spatial Data

**Roles involved:** Data Engineer, Technical Data Steward, Data Domain Steward (Asset)

The GIS spatial data is the missing piece. It must be onboarded into the EDAP before any analytical work can proceed.

### 3.1 Ingestion Design

The Data Engineer designs the ingestion pipeline in consultation with the Asset domain's Technical Data Steward.

| Design Decision | Detail |
|---|---|
| Source | ESRI REST API (ArcGIS feature service) |
| Landing | Files extracted via API and staged in Unity Catalog Volumes: `/Volumes/prod_bronze/landing/esri/` |
| Pipeline framework | Lakeflow Spark Declarative Pipelines (SDP) with Auto Loader |
| Target | `prod_bronze.gis_raw.spatial_boundary` |
| Refresh | Daily incremental (feature service change tracking) |
| Format | GeoJSON converted to Delta table with geometry stored as WKT (Well-Known Text) |

### 3.2 Pipeline Implementation

The data flows as follows:

```
ESRI GIS API → /Volumes/prod_bronze/landing/esri/ → prod_bronze.gis_raw.spatial_boundary
```

The pipeline applies the standard Bronze Raw conventions:

- **Auto Loader** ingests files from the landing volume, inferring schema and capturing any unexpected fields in the `_rescued_data` column
- All source fields are preserved as received — no transformation at this stage
- **Audit columns** are appended:
  - `edap_batch` — epoch milliseconds when the batch arrived (e.g. `1742425200000`)
  - `edap_hash` — SHA-512 hash of all non-`edap_` prefixed fields for change detection
  - `edap_inserted_ts` — timestamp of record insertion into the Raw zone
- The `_rescued_data` column captures any schema evolution from the GIS source, ensuring no data is silently dropped

### 3.3 Governed Tags Applied at Bronze

The Technical Data Steward applies the standard Raw zone tags:

| Tag | Value |
|---|---|
| `medallion_layer` | `bronze` |
| `medallion_sublayer` | `raw` |
| `source_system` | `esri` |
| `data_domain` | `asset` |
| `ingestion_method` | `autoloader` |
| `classification_status` | `unclassified` |
| `waicp_classification` | `OFFICIAL` (default until classified) |
| `pi_contained` | `false` |

At this point, `classification_status` is `unclassified` — the table has been ingested but not yet assessed for sensitivity. Under the EDAP tagging strategy, unclassified tables default to `OFFICIAL` as a provisional holding value, and cross-domain access is governed by the classification lifecycle rules.

### 3.4 Classification Assessment

The Technical Data Steward performs the initial sensitivity classification:

| Tag | Value | Rationale |
|---|---|---|
| `waicp_classification` | `OFFICIAL` | Spatial boundaries are general operational data, not sensitive |
| `pi_contained` | `false` | No personal information in spatial boundary data |
| `soci_critical` | `true` | Infrastructure location data — pressure zone boundaries could reveal network vulnerability |
| `sensitivity_type` | `infrastructure_vulnerability` | Precise infrastructure geometry |
| `regulatory_scope` | `soci_act` | Subject to SOCI Act 2018 obligations |
| `access_model` | `controlled` | Not open — requires membership in `soci_critical_access` group |
| `classification_status` | `provisional` | Initial assessment complete, pending steward confirmation |

The Data Domain Steward (Asset) reviews the provisional classification. They confirm:

- The `soci_critical` designation is appropriate — pressure zone boundaries reveal the network's hydraulic structure
- The `controlled` access model is correct — analysts in the `domain_asset_analysts` and `domain_operations_analysts` groups who are also members of `soci_critical_access` can query this data
- The classification is accurate and complete

The steward advances the status: `classification_status` moves from `provisional` to `classified`. The table is now fully governed.

---

## 4. Bronze to Silver — Base Zone Processing

**Roles involved:** Data Engineer, Technical Data Steward

The Base zone applies type casting, deduplication, SCD Type 2 tracking, and data quality annotation to produce clean, trustworthy datasets that are structurally aligned with their source systems.

### 4.1 Maximo Work Orders (Raw to Base)

**Pipeline:** `prod_bronze.maximo_raw.work_order` → `prod_silver.maximo_base.base_work_order`

The Lakeflow SDP pipeline applies the following transformations:

| Transformation | Detail |
|---|---|
| Type casting | `reported_date` STRING → TIMESTAMP, `completed_date` STRING → TIMESTAMP, `repair_cost` STRING → DECIMAL(12,2), `work_order_id` STRING → STRING (retained as natural key) |
| Deduplication | `edap_hash` comparison — duplicate records with identical hash values within the same batch are collapsed |
| Primary key | `work_order_id` (natural key from Maximo) |
| SCD Type 2 | Track history of work order status changes (e.g. OPEN → IN_PROGRESS → COMPLETE) using `edap_eff_from`, `edap_eff_to`, `edap_is_current` |
| Null handling | `location_id` permitted as NULL (flagged as warning), `work_order_id` must not be NULL (flagged as error) |
| Field standardisation | Column names lowercased, spaces replaced with underscores, consistent naming applied |

**Data quality rules applied:**

| Rule | Severity | DQ Column |
|---|---|---|
| `work_order_id IS NOT NULL` | Error | `dq_errors` |
| `status IN ('OPEN', 'IN_PROGRESS', 'COMPLETE', 'CANCELLED', 'CLOSED')` | Error | `dq_errors` |
| `reported_date <= CURRENT_DATE` | Error | `dq_errors` |
| `reported_date IS NOT NULL` | Error | `dq_errors` |
| `location_id IS NOT NULL` | Warning | `dq_warnings` |
| `repair_cost >= 0` | Warning | `dq_warnings` |
| `completed_date >= reported_date` | Warning | `dq_warnings` |

Every record in the Base table carries the DQ annotation columns:

- `dq_status` — `PASS`, `WARN`, or `FAIL` (FAIL if any error rule is violated; WARN if only warning rules are violated)
- `dq_errors` — array of violated error rules (e.g. `['status_not_in_allowed_values']`)
- `dq_warnings` — array of violated warning rules (e.g. `['location_id_is_null']`)
- `dq_checked_ts` — timestamp of the most recent DQ evaluation

**Example record with `dq_status = WARN`:**

| work_order_id | status | reported_date | location_id | dq_status | dq_errors | dq_warnings | dq_checked_ts |
|---|---|---|---|---|---|---|---|
| WO-2025-041872 | COMPLETE | 2025-11-14 | *NULL* | WARN | [] | ['location_id_is_null'] | 2026-03-20T06:00:00Z |

This record passes all error rules but is flagged as a warning because `location_id` is missing. The record is retained in the Base table — not quarantined — but the warning is visible to downstream consumers and DQ dashboards.

**Governed tags applied:**

| Tag | Value |
|---|---|
| `medallion_layer` | `silver` |
| `medallion_sublayer` | `base` |
| `source_system` | `maximo` |
| `data_domain` | `operations` |
| `classification_status` | `classified` |
| `waicp_classification` | `OFFICIAL` |
| `pi_contained` | `false` |
| `quality_tier` | `provisional` |

### 4.2 SAP Equipment (Already in Base)

The SAP equipment register already exists at `prod_silver.sap_base.base_equipment`. The Data Engineer confirms with the Technical Data Steward that this table contains all required fields:

- `equipment_id` — the asset identifier, used to join to Maximo work orders via `asset_id`
- `material_type` — pipe material (e.g. CAST_IRON, DUCTILE_IRON, PVC, AC)
- `install_date` — date the pipe was installed
- `diameter_mm` — nominal pipe diameter in millimetres
- `asset_class` — equipment class (e.g. WATER_MAIN, SERVICE_PIPE)
- `pipe_length_m` — length of the pipe segment in metres

The table is SCD Type 2-tracked, with `edap_eff_from`, `edap_eff_to`, and `edap_is_current` columns. Its `classification_status` is `classified`, `quality_tier` is `certified`, and `data_domain` is `asset`.

No additional work is required for this source.

### 4.3 GIS Spatial Boundaries (Raw to Base)

**Pipeline:** `prod_bronze.gis_raw.spatial_boundary` → `prod_silver.gis_base.base_spatial_boundary`

| Transformation | Detail |
|---|---|
| Geometry validation | WKT geometry strings parsed and validated; invalid geometries flagged as DQ errors |
| CRS standardisation | All coordinates standardised to EPSG:4326 (WGS 84) |
| Type casting | `boundary_id` STRING → STRING (retained), `area_sqm` STRING → DOUBLE, `boundary_type` STRING → STRING |
| Deduplication | `edap_hash` comparison on `boundary_id` |
| Primary key | `boundary_id` |

**Data quality rules:**

| Rule | Severity |
|---|---|
| `boundary_id IS NOT NULL` | Error |
| `geometry IS NOT NULL AND ST_IsValid(geometry)` | Error |
| `area_sqm > 0` | Warning |
| `boundary_type IN ('PRESSURE_ZONE', 'SUBURB', 'TOWN', 'REGION')` | Error |

**Governed tags applied:**

| Tag | Value |
|---|---|
| `medallion_layer` | `silver` |
| `medallion_sublayer` | `base` |
| `source_system` | `esri` |
| `data_domain` | `asset` |
| `soci_critical` | `true` |
| `access_model` | `controlled` |
| `classification_status` | `classified` |
| `quality_tier` | `provisional` |

---

## 5. Silver — Enriched Zone (Cross-Domain Integration)

**Roles involved:** Data Engineer, Data Domain Steward (Operations), Data Domain Steward (Asset), Data Architect

The Enriched zone is where data transitions from source-aligned to business-aligned. This is the most governance-intensive stage in the pipeline, because it is where cross-domain integration occurs and domain ownership must be explicitly established.

### 5.1 Domain Assignment

The Data Architect determines how ownership works for this cross-domain dataset.

The enriched table will join:

- **Work orders** (break events) from the Operations domain
- **Equipment attributes** (pipe characteristics) from the Asset domain
- **Spatial boundaries** (geographic context) from the Asset domain

The **grain** of the enriched table is one row per break event. The break event is the primary business entity — it is the thing being measured, counted, and analysed. Therefore:

- **Owning domain:** Operations (the break event is the grain)
- **Contributing domain:** Asset (pipe attributes and spatial context are joined in to enrich the event)
- The enriched schema is `prod_silver.asset_enriched` — wait. The Data Architect reconsiders. Although the grain is a break event (Operations), the enriched dataset combines break events with pipe attributes and spatial context. Under the EDAP convention, Enriched schemas are named by domain. Since this dataset primarily serves an operational analysis purpose and the break event is the grain, the schema should sit in `prod_silver.operations_enriched`. However, a case could be made for `prod_silver.asset_enriched` because the capital replacement programme is fundamentally about asset management.

After discussion with both Data Domain Stewards, the Data Architect decides:

- **Schema:** `prod_silver.operations_enriched` — the break event is the grain, and the Operations domain steward is the primary data custodian for this enriched dataset
- **Table:** `prod_silver.operations_enriched.enr_pipe_break_history`
- The `contributing_domains` tag is applied to make the cross-domain lineage explicit

### 5.2 Cross-Domain Data Contract

Before the pipeline is built, the Data Domain Stewards for Operations and Asset agree on a data contract for the cross-domain join. This contract specifies:

- **Join keys:** `base_work_order.asset_id = base_equipment.equipment_id`
- **Spatial join logic:** Point-in-polygon join between work order location coordinates and `base_spatial_boundary` geometries (pressure zones and suburbs)
- **Business rules:** Only include work orders where `work_order_type = 'PIPE_BREAK'`
- **Freshness SLA:** Enriched table refreshed daily by 07:00 AWST
- **Quality expectations:** Inherited from contributing Base tables, plus cross-domain join completeness > 95%
- **Ownership:** Operations domain owns the enriched table; Asset domain is a contributing producer

### 5.3 Enriched Pipeline

```
prod_silver.maximo_base.base_work_order        ─┐
prod_silver.sap_base.base_equipment             ─┼─► prod_silver.operations_enriched.enr_pipe_break_history
prod_silver.gis_base.base_spatial_boundary      ─┘
```

The Lakeflow SDP pipeline applies the following logic:

| Step | Detail |
|---|---|
| Filter | Only work orders where `work_order_type = 'PIPE_BREAK'` and `edap_is_current = true` |
| Asset join | LEFT JOIN `base_equipment` on `asset_id = equipment_id` where `edap_is_current = true` — captures current pipe attributes at the time of the break |
| Spatial join | Point-in-polygon join to assign `pressure_zone_id`, `suburb`, `town`, and `region` from `base_spatial_boundary` |
| Derived fields | `pipe_age_at_break = YEAR(reported_date) - YEAR(install_date)` |
| Age banding | `age_band` derived from `pipe_age_at_break`: 0–10, 11–20, 21–40, 41–60, 60+ |
| Break year | `break_year = YEAR(reported_date)`, `break_month = MONTH(reported_date)` |
| Repair duration | `repair_duration_hours = TIMESTAMPDIFF(HOUR, reported_date, completed_date)` |
| DQ carried forward | `dq_status`, `dq_errors`, `dq_warnings`, `dq_checked_ts` inherited and extended |
| Join completeness DQ | Records where `equipment_id IS NULL` (unmatched asset) flagged as `dq_warnings: ['asset_join_unmatched']` |

The resulting `enr_pipe_break_history` table contains one row per pipe break event, enriched with pipe attributes and geographic context — ready for dimensional modelling in Gold.

### 5.4 Data Domain Steward Review

Both domain stewards review the enriched table before it is promoted:

- The **Operations Data Domain Steward** validates the break type classification logic — confirming that `work_order_type = 'PIPE_BREAK'` correctly identifies break events and excludes planned maintenance, leaks, and other work order types
- The **Asset Data Domain Steward** validates the spatial join logic and pipe attribute mapping — confirming that the equipment join uses the correct key and that the point-in-polygon logic correctly assigns pressure zones

Both stewards confirm the cross-domain data contract is correctly implemented.

### 5.5 Governed Tags Applied

| Tag | Value |
|---|---|
| `medallion_layer` | `silver` |
| `medallion_sublayer` | `enriched` |
| `source_system` | `maximo,sap_ecc,esri` |
| `data_domain` | `operations` |
| `contributing_domains` | `asset` |
| `soci_critical` | `true` |
| `waicp_classification` | `OFFICIAL` |
| `access_model` | `controlled` |
| `pi_contained` | `false` |
| `classification_status` | `classified` |
| `quality_tier` | `provisional` |
| `refresh_frequency` | `daily` |

The `soci_critical: true` tag is inherited from the contributing GIS spatial data — per the EDAP tagging strategy's aggregation rule, the resulting table inherits the highest sensitivity of any contributing source.

---

## 6. Gold — Dimensional Model (BI Zone)

**Roles involved:** Data Architect, BI Developer, Data Product Owner

The Gold BI zone is where the enriched data is shaped into a star schema optimised for analytical consumption. This is where data becomes a **data product** — certified, contracted, discoverable, and owned.

### 6.1 Dimensional Design

The Data Architect designs a star schema in the `prod_gold.operations_bi` schema.

**Fact table:**

`prod_gold.operations_bi.fact_pipe_break` — grain: one row per break event

| Column | Type | Description |
|---|---|---|
| `break_sk` | BIGINT | Surrogate key |
| `work_order_id` | STRING | Natural key from Maximo |
| `asset_sk` | BIGINT | FK to `dim_asset` |
| `location_sk` | BIGINT | FK to `dim_location` |
| `date_sk` | INT | FK to `dim_date` (YYYYMMDD) |
| `repair_cost` | DECIMAL(12,2) | Cost of the repair |
| `repair_duration_hours` | DOUBLE | Hours from reported to repaired |
| `pipe_age_at_break` | INT | Age of pipe in years at time of break |
| `is_repeat_break` | BOOLEAN | True if same asset broke within prior 12 months |
| `within_sla` | BOOLEAN | True if repaired within SLA timeframe |

**Dimension tables:**

`prod_gold.operations_bi.dim_asset` — pipe attributes

| Column | Type | Description |
|---|---|---|
| `asset_sk` | BIGINT | Surrogate key |
| `equipment_id` | STRING | Natural key from SAP |
| `material_type` | STRING | Pipe material (e.g. CAST_IRON, PVC) |
| `material_type_desc` | STRING | Human-readable material name |
| `diameter_mm` | INT | Nominal pipe diameter |
| `install_date` | DATE | Date pipe was installed |
| `age_band` | STRING | Age group (0–10, 11–20, 21–40, 41–60, 60+) |
| `asset_class` | STRING | Equipment class |
| `pipe_length_m` | DOUBLE | Length of the pipe segment |

`prod_gold.operations_bi.dim_location` — geographic attributes

| Column | Type | Description |
|---|---|---|
| `location_sk` | BIGINT | Surrogate key |
| `pressure_zone_id` | STRING | Hydraulic pressure zone |
| `suburb` | STRING | Suburb name |
| `town` | STRING | Town name |
| `region` | STRING | WC operational region |

`prod_gold.operations_bi.dim_date` — standard date dimension

| Column | Type | Description |
|---|---|---|
| `date_sk` | INT | Surrogate key (YYYYMMDD) |
| `calendar_date` | DATE | Full date |
| `day_of_week` | STRING | Monday, Tuesday, etc. |
| `month_name` | STRING | January, February, etc. |
| `month_num` | INT | 1–12 |
| `quarter` | STRING | Q1, Q2, Q3, Q4 |
| `financial_year` | STRING | e.g. FY2025-26 |
| `calendar_year` | INT | e.g. 2026 |

**Aggregate table:**

`prod_gold.operations_bi.agg_break_rate_monthly` — pre-aggregated break rates

| Column | Type | Description |
|---|---|---|
| `material_type` | STRING | Pipe material |
| `age_band` | STRING | Age group |
| `pressure_zone_id` | STRING | Pressure zone |
| `suburb` | STRING | Suburb |
| `break_month` | DATE | First day of month |
| `break_count` | INT | Number of breaks |
| `total_pipe_length_km` | DOUBLE | Total pipe length in km for this cohort |
| `break_rate_per_km` | DOUBLE | Breaks per km |
| `avg_repair_cost` | DOUBLE | Average repair cost |
| `avg_repair_hours` | DOUBLE | Average repair duration |

### 6.2 Pipeline Implementation

The Lakeflow SDP pipeline transforms the enriched table into the Gold star schema:

```
prod_silver.operations_enriched.enr_pipe_break_history ──► prod_gold.operations_bi.fact_pipe_break
                                                         ──► prod_gold.operations_bi.dim_asset
                                                         ──► prod_gold.operations_bi.dim_location
                                                         ──► prod_gold.operations_bi.agg_break_rate_monthly
```

The `dim_date` table is maintained separately as a reference dimension in `prod_reference` and replicated into `prod_gold.operations_bi.dim_date`.

Surrogate keys are generated using deterministic hashing to ensure idempotent reprocessing. The pipeline runs daily, triggered after the Enriched zone pipeline completes.

### 6.3 Data Product Registration

The **Data Product Owner** — a senior Operations analyst nominated by the GM of Operations — registers the dimensional model as a managed data product.

**Data contract (authored in YAML and stored in source control):**

| Contract Element | Value |
|---|---|
| Product name | Pipe Break Analytics |
| Product identifier | `pipe_break_analytics` |
| Owner | Data Product Owner (Operations) |
| Domain | Operations |
| Contributing domains | Asset |
| Tables | `fact_pipe_break`, `dim_asset`, `dim_location`, `dim_date`, `agg_break_rate_monthly` |
| Freshness SLA | Daily by 06:00 AWST |
| Quality rules | PK uniqueness on fact, FK integrity to all dimensions, row count > threshold, break_rate_per_km within plausible range |
| Schema versioning | Semantic versioning (additive changes = minor, breaking changes = major) |
| Current version | 1.0.0 |

**Registration in Data Catalogue (Alation):**

Each table is registered with:

- Business descriptions written by the Data Product Owner (not just technical metadata)
- Column-level descriptions explaining business meaning
- Lineage traced back through Enriched → Base → Raw → source system
- Sample queries and usage guidance for analysts

**Governed tags applied to Gold BI tables:**

| Tag | Value |
|---|---|
| `medallion_layer` | `gold` |
| `medallion_sublayer` | `bi` |
| `data_domain` | `operations` |
| `contributing_domains` | `asset` |
| `data_product` | `pipe_break_analytics` |
| `data_product_tier` | `certified` |
| `quality_tier` | `certified` |
| `bi_published` | `true` |
| `refresh_frequency` | `daily` |
| `waicp_classification` | `OFFICIAL` |
| `soci_critical` | `true` |
| `access_model` | `controlled` |
| `pi_contained` | `false` |
| `classification_status` | `classified` |

**Unity Catalog table properties** are set with contract metadata:

```sql
ALTER TABLE prod_gold.operations_bi.fact_pipe_break SET TBLPROPERTIES (
  'data_product.name' = 'pipe_break_analytics',
  'data_product.version' = '1.0.0',
  'data_product.freshness_sla' = 'daily_0600_AWST',
  'data_product.owner' = 'operations_data_product_owner'
);
```

### 6.4 Certification

Certification follows the EDAP data product framework:

1. **Data Product Owner** confirms the product meets its contract — schema is correct, quality rules pass, freshness SLA is met over a two-week burn-in period
2. **Data Domain Steward (Operations)** reviews classification and business metadata — confirms break type definitions, metric calculations, and dimension attributes are accurate
3. **Technical Data Steward** confirms technical controls — governed tags are applied, ABAC policies are enforced, lineage is captured, and quality expectations are configured in the pipeline
4. **Platform team** performs a technical review — pipeline performance, compute cost, storage footprint, and monitoring are acceptable
5. The product is published in the Data Catalogue with `data_product_tier: certified` and `quality_tier: certified`

The product is now discoverable, governed, and ready for consumption.

---

## 7. BI Layer — Semantic Model and Dashboard

**Roles involved:** BI Developer, Data Product Owner, Business User (GM Operations)

### 7.1 Semantic Model

The BI Developer creates a Power BI semantic model named **Pipe Break Analytics** in the `EDAP - Operations` Power BI workspace.

**Connection:**

- **DirectQuery** connection to `prod_gold.operations_bi` via the Databricks SQL Warehouse
- DirectQuery is chosen over Import because the data refreshes daily and the GM wants to see the latest data without waiting for a scheduled Power BI dataset refresh

**Measures defined in DAX:**

| Measure | Definition |
|---|---|
| Break Rate | Total breaks / total pipe length in km (annualised) |
| Avg Repair Cost | AVERAGE of `repair_cost` from `fact_pipe_break` |
| MTTR (Hours) | AVERAGE of `repair_duration_hours` |
| Repeat Break Rate | COUNT of breaks where `is_repeat_break = TRUE` / total break count |
| Compliance % | COUNT of breaks where `within_sla = TRUE` / total break count |

**Dimensions configured:**

The star schema relationships are defined in the semantic model — `fact_pipe_break` to `dim_asset`, `dim_location`, and `dim_date` via their surrogate keys. Slicers and filters are configured for Material Type, Age Band, Diameter, Pressure Zone, Geographic Area, and Time.

**Row-level security:** Not required. The data is classified as `OFFICIAL` with a `controlled` access model due to the SOCI-critical tag. Access is managed at the Unity Catalog level through the `soci_critical_access` group — any user who can connect to the SQL Warehouse and is a member of that group can query the data. No additional Power BI RLS is needed.

**Semantic model as a data product:** The semantic model is itself registered as a data product in the catalogue — it is the governed layer through which business users consume the dimensional model. The BI Developer documents all measures, their business definitions, and their calculation logic in both the semantic model metadata and the Data Catalogue (Alation).

### 7.2 Dashboard Build

The BI Developer builds a four-page dashboard:

**Page 1 — Executive Summary**

- KPI cards: total breaks (YTD), break rate trend (sparkline), total repair cost, MTTR, compliance %
- Map visualisation: break hotspots plotted by suburb, coloured by break rate severity
- Trend line: monthly break count and break rate over the past three years

**Page 2 — Material Analysis**

- Bar chart: break rate by material type, sorted highest to lowest
- Matrix: break rate by material type (rows) and age band (columns) — the replacement priority matrix the capital team requested
- Scatter plot: pipe age vs. break rate, coloured by material type
- Key insight callout: "Cast iron mains over 40 years old account for 45% of all breaks but only 18% of network length"

**Page 3 — Geographic View**

- Filled map: breaks by pressure zone, coloured by break rate
- Table: top 20 suburbs by break count with break rate, MTTR, and cost
- Drill-through: click a pressure zone to see its break history, material mix, and age profile

**Page 4 — Trend Analysis**

- Line chart: monthly break rate over three years with seasonal decomposition
- Year-over-year comparison: current year vs. prior year by month
- Quarterly aggregation: break rate by quarter, with forecast trendline
- Seasonal pattern callout: "Break rates peak in winter months (June–August) correlating with soil movement and temperature change"

### 7.3 UAT and Sign-Off

The BI Developer schedules a UAT session with the GM of Operations and two senior members of the capital planning team.

**UAT feedback:**

| Item | Feedback | Resolution |
|---|---|---|
| Diameter slicer | "I want to filter by diameter — some of our worst performers are small-diameter AC mains" | BI Developer adds diameter as a slicer on Page 2 |
| Financial year | "We plan on financial year, not calendar year — can the time axis show FY?" | BI Developer switches the default time axis to financial year, with calendar year as an alternative |
| Export | "Can I export the priority matrix to Excel for the capital planning spreadsheet?" | BI Developer enables export on the matrix visual |
| Metric validation | GM confirms break rates align with the most recent annual report figures | No change required — data is validated |

The BI Developer implements the adjustments and publishes the dashboard to the production `EDAP - Operations` workspace. The GM of Operations is granted viewer access. A distribution list of capital planning analysts is granted contributor access for self-service exploration.

---

## 8. Ongoing Operations

**Roles involved:** Data Product Owner, Data Engineer, Technical Data Steward, Data Domain Steward, Data Consumer

The dashboard is live. But delivery is not the end — it is the beginning of the operational lifecycle.

### 8.1 Monitoring

The following monitoring is in place from day one:

**Pipeline freshness:**

- The Lakeflow SDP pipeline is scheduled to complete the full chain (Bronze → Silver Base → Silver Enriched → Gold BI) by 06:00 AWST each day
- If the Gold tables are not refreshed by 06:00, an alert fires to the Operations data engineering Slack channel and the Data Product Owner receives an email notification

**Data quality dashboards:**

- A DQ dashboard (built on the `dq_status`, `dq_errors`, and `dq_warnings` columns) tracks quality scores across all tables in the pipeline
- Quality metrics: % of records with `dq_status = PASS`, trend of WARN and FAIL rates, most common DQ violations
- The Data Product Owner reviews the DQ dashboard weekly

**Usage monitoring:**

- Unity Catalog system tables capture every query against the Gold BI tables
- Usage statistics show which users and groups are querying the data product, how frequently, and which tables are most accessed
- The Data Product Owner uses this to understand adoption and identify additional consumer needs

### 8.2 Incident Example — Schema Drift in Source

Three months after go-live, the DQ dashboard shows a spike: `dq_status = FAIL` on 15% of records in `fact_pipe_break` — all due to missing `asset_id` values. The alert fires to Slack at 06:15 AWST.

**Triage and resolution timeline:**

| Time | Action | Role |
|---|---|---|
| 06:15 | Alert fires: DQ failure rate > 10% threshold on `fact_pipe_break` | Automated |
| 06:30 | Data Product Owner triages: contacts Technical Data Steward | Data Product Owner |
| 07:00 | Technical Data Steward investigates: traces the failure to `base_work_order` where 15% of records have `asset_id = NULL` since yesterday's batch | Technical Data Steward |
| 07:30 | Root cause identified: Maximo deployed a patch overnight that renamed the asset reference field from `asset_id` to `asset_num`. Auto Loader captured the new field in `_rescued_data`, but the Bronze → Silver pipeline still maps the old field name, which is now NULL. | Technical Data Steward |
| 08:00 | Fix implemented: Technical Data Steward updates the Lakeflow SDP pipeline configuration to map `asset_num` as the source for `asset_id`, with a COALESCE to handle the transition period where both field names may appear | Technical Data Steward, Data Engineer |
| 08:30 | Pipeline rerun: Bronze → Silver → Gold chain reprocessed. DQ dashboard confirms FAIL rate returns to < 1% | Data Engineer |
| 09:00 | Data Domain Steward (Operations) validates the fix: confirms the asset join completeness is back to normal | Data Domain Steward |
| 09:30 | Post-incident review: documents root cause, adds a schema drift monitor that alerts when `_rescued_data` column is non-null for > 5% of records in any Bronze table | Technical Data Steward |

**Preventive action:** The schema drift monitor is added as a standard DQ expectation on all Bronze Raw tables, catching future field name changes before they propagate to downstream layers.

### 8.3 Lifecycle Evolution

Twelve months after go-live, the capital team's requirements have matured. They request additional analytics:

- **Soil corrosivity** — correlating break rates with soil type data from the Geotechnical Information Register (a new source system to onboard)
- **Failure mode classification** — distinguishing between circumferential breaks, longitudinal splits, joint failures, and corrosion holes
- **Predictive model input** — the data science team wants to use the enriched dataset as a feature source for a pipe failure prediction model

**Lifecycle response:**

| Change | Action | Role |
|---|---|---|
| Soil corrosivity | New source system onboarded to Bronze (same process as GIS in Section 3). Enriched pipeline extended with soil join. | Data Engineer, Technical Data Steward |
| Failure mode | New field added to Maximo work order extraction. Bronze → Silver pipeline updated. Enriched pipeline passes through. Gold dimension extended. | Data Engineer, Data Domain Steward (Operations) |
| Predictive model | Data science team granted `SELECT` on `prod_silver.operations_enriched.enr_pipe_break_history`. Feature engineering begins in `edap_sandbox`. | Data Architect, Data Domain Steward (Operations) |

The Data Product Owner updates the data contract: version bumps from `1.0.0` to `1.1.0` (additive changes — new columns, no breaking changes to existing schema). Consumers are notified of the additions via the Data Catalogue. Existing dashboards and queries continue to work without modification.

The semantic model is updated with new measures (Break Rate by Soil Type, Failure Mode Distribution). Two new dashboard pages are added. The GM of Operations signs off on the extended product.

The cycle continues.

---

## 9. Summary — Roles at Each Stage

| Stage | Activity | Roles Involved |
|---|---|---|
| Business Request | Dashboard request logged through BI intake process | GM Operations, BI Developer |
| Requirements & Discovery | Discovery workshop, source identification, catalogue search | BI Developer, Data Architect, Data Domain Steward (Operations), Data Domain Steward (Asset) |
| Data Onboarding | GIS ingestion pipeline design, build, and deployment | Data Engineer, Technical Data Steward |
| Classification | Sensitivity assessment, WAICP tagging, SOCI designation | Technical Data Steward, Data Domain Steward (Asset) |
| Bronze to Silver Base | Type casting, deduplication, SCD Type 2, DQ annotation | Data Engineer, Technical Data Steward |
| Silver Enriched | Cross-domain join, business logic, domain assignment, data contract | Data Engineer, Data Domain Steward (Operations), Data Domain Steward (Asset), Data Architect |
| Gold BI | Dimensional modelling, star schema, surrogate keys | Data Architect, BI Developer |
| Data Product Registration | Contract authoring, catalogue registration, tag application | Data Product Owner, Technical Data Steward |
| Data Product Certification | Classification review, technical review, platform review | Data Product Owner, Data Domain Steward (Operations), Technical Data Steward, Platform Team |
| Semantic Model | Power BI semantic model build, measure definitions, certification | BI Developer, Data Product Owner |
| Dashboard | Visualisation build, UAT, publication | BI Developer, GM Operations |
| Ongoing Monitoring | Pipeline freshness, DQ dashboards, usage statistics | Data Product Owner, Data Engineer, Technical Data Steward |
| Incident Response | Triage, root cause analysis, fix, post-incident review | Data Product Owner, Technical Data Steward, Data Engineer, Data Domain Steward |
| Lifecycle Evolution | New sources, schema extensions, contract versioning | Data Product Owner, Data Engineer, Data Domain Steward, Data Architect |

---

## 10. Companion Documents

| Document | Relevance to This Example |
|---|---|
| [Medallion Architecture](../platform/medallion-architecture.md) | Zone structure, naming conventions, audit columns, DQ approach |
| [EDAP Access Model](../platform/edap-access-model.md) | Catalog naming, workspace topology, ABAC policies, group structure |
| [EDAP Tagging Strategy](../governance/edap-tagging-strategy.md) | Four-layer tag taxonomy, classification lifecycle, allowed values |
| [Data Governance Roles](../governance/data-governance-roles.md) | Role definitions for all governance roles referenced in this example |
| [Data Domains](../governance/data-domains.md) | Seven domain definitions, conceptual data entities by domain |
| [BI Lifecycle](../lifecycles/bi-lifecycle.md) | BI lifecycle stages from requirements through to ongoing operations |
| [Data Engineering Lifecycle](../lifecycles/data-engineering-lifecycle.md) | Data engineering stages from ingestion through to serving |
| [Domain Governance Across Systems](../governance/domain-governance-across-systems.md) | Cross-domain data contracts, governance architecture |
| [EDAP Pipeline Framework](../specifications/edap-pipeline-framework.md) | Pipeline framework requirements and conventions |
| [Databricks End-to-End Platform](../platform/databricks-end-to-end-platform.md) | Platform capabilities — Lakeflow, Unity Catalog, AI/BI |

---

*This document is maintained by Architecture & Strategy, Digital & Technology, Water Corporation. For questions or change requests, contact the EDAP governance team.*

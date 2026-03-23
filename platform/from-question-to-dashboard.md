# From Question to Dashboard — An End-to-End Data Journey

**Mark Shaw** | Principal Data Architect

---

## Prologue

This document tells the story of a single business question and traces its journey through Water Corporation's Enterprise Data & Analytics Platform (EDAP). Along the way it touches every core role, every layer of the medallion architecture, and the governance framework that ensures the final dashboard is not just insightful — but trusted.

---

## Act 1 — The Business Question

It starts, as it always does, with a question.

Sarah Chen, General Manager of Asset Management, is preparing for a board briefing on infrastructure resilience. She turns to her team lead and says:

> "I need to understand the relationship between asset age, maintenance frequency, and unplanned service interruptions — broken down by region and asset class. And I need it in a way the board can see every quarter."

This is not a request for a spreadsheet. It is a request for an ongoing, governed, trusted analytical capability — a dashboard backed by metrics the organisation can rely on.

---

## Act 2 — Discovery and Requirements

### The BI Analyst steps in

James Okafor, a **BI Analyst** embedded in the Asset domain, picks up the request. His role sits at the intersection of business need and data capability — the BI Lifecycle begins with *Discovery & Business Requirements*, and that is exactly where James starts.

He workshops with Sarah and her team to decompose the question into measurable components:

| Business Question | Metric | Grain | Source Domain |
|---|---|---|---|
| How old are our assets? | **Asset Age** (years since commissioning) | Asset × Region | Asset |
| How often do we maintain them? | **Maintenance Frequency** (work orders per asset per year) | Asset × Period | Operations |
| How often do things break? | **Unplanned Interruption Rate** (unplanned events per asset per year) | Asset × Period | Operations |
| Where are the problems? | **Regional Breakdown** | Region | Asset |
| What types of assets are failing? | **Asset Class Segmentation** | Asset Class | Asset |

James documents these as a **BI requirements specification** — the questions, the metrics, the dimensions, the expected refresh cadence (weekly), and the audience (executive, so high-level with drill-down capability).

### The Data Domain Steward validates

Before any pipeline work begins, James consults **Priya Sharma**, the **Data Domain Steward** for the Asset domain. Priya's role is operational governance — she knows what data exists, what it means, and who owns it.

Priya confirms:
- Asset commissioning dates live in **SAP ECC** (Plant Maintenance module) and are already ingested into EDAP Bronze.
- Work order history is in **Maximo** and is also flowing into Bronze.
- Service interruption records come from the **OMS** (Outage Management System) — but this source has not yet been onboarded to EDAP.

This is the first governance checkpoint. Priya flags that the OMS data will need to go through the ingestion lifecycle, and raises this with the Data Engineering team.

---

## Act 3 — Ingestion: Bringing OMS Data into the Lake

### The Data Engineer builds the pipeline

**Aisha Patel**, a **Data Engineer** on the EDAP platform team, is assigned to onboard the OMS source. She follows the Data Engineering Lifecycle — starting at *Ingestion*.

The EDAP Pipeline Framework is metadata-driven. Rather than writing bespoke ETL code, Aisha works with a configuration-first approach:

1. **Source registration** — She registers the OMS database as a source in the ingestion framework, configuring the connection via Lakeflow Connect.

2. **Landing Zone** — OMS data lands in the Bronze layer's **Landing Zone** (`prod_bronze.oms_landing`). This is a transient staging area where pre-ingestion validation runs — file integrity checks, schema drift detection, row-count reconciliation.

3. **Raw Zone** — Validated data is persisted into the **Raw Zone** (`prod_bronze.oms_raw`). Here it is stored as an immutable, append-only record. Every row is stamped with audit columns:
   - `edap_inserted_ts` — when the record arrived
   - `edap_batch` — the pipeline run that loaded it
   - `edap_hash` — a SHA-512 hash for change detection
   - `edap.file_name`, `edap.file_path` — source traceability

This is the golden rule of Bronze: **the data is never modified**. It is the system of record. If anything downstream goes wrong, every subsequent layer can be regenerated from here.

### The Technical Data Steward governs the landing

Meanwhile, **Raj Mehta**, the **Technical Data Steward** for the Asset domain, ensures that governance is applied from the moment data enters the platform:

- **Unity Catalog registration** — The new OMS tables are registered in the `prod_bronze` catalog under the `oms_raw` schema, following the namespace convention: `prod_bronze.oms_raw.service_interruptions`.
- **Governed tags applied** — Raj applies the four-layer tagging strategy:
  - Layer 1 (WAICP): `waicp_classification = OFFICIAL`
  - Layer 2 (Sensitivity): `pi_contained = false`
  - Layer 3 (Access): `access_model = open`
  - Layer 4 (EDAP): `medallion_layer = bronze`, `medallion_sublayer = raw`, `source_system = oms`, `data_domain = operations`, `classification_status = provisional`
- **Lineage capture** — Unity Catalog automatically tracks column-level lineage from source through to the Raw Zone.

---

## Act 4 — Silver Layer: Modelling the Trusted Enterprise View

With all three sources now in Bronze (SAP, Maximo, OMS), the real modelling work begins. Data must be cleaned, validated, conformed, and enriched — the progressive refinement that gives the medallion architecture its power.

### Base Zone — Cleansing and conforming each source

Aisha builds Lakeflow Spark Declarative Pipelines (SDP) to move data from Raw to **Base** (`prod_silver`). Each source gets its own Base schema:

| Source | Raw Schema | Base Schema |
|---|---|---|
| SAP ECC | `prod_bronze.sap_raw` | `prod_silver.sap_base` |
| Maximo | `prod_bronze.maximo_raw` | `prod_silver.maximo_base` |
| OMS | `prod_bronze.oms_raw` | `prod_silver.oms_base` |

In the Base Zone, Aisha applies:

- **Type casting and standardisation** — Dates normalised to ISO 8601. Equipment IDs cast to consistent string formats. Null handling rules applied per the pipeline configuration.
- **Deduplication** — Hash-based change detection identifies and removes duplicate records.
- **Data quality expectations** — Built into the SDP pipelines using Lakeflow expectations:
  - `asset_id IS NOT NULL` (completeness)
  - `commissioning_date <= current_date()` (validity)
  - `work_order_id` uniqueness (uniqueness)
  - Referential integrity between work orders and assets
- **SCD Type 2 history** — For slowly changing dimensions like asset attributes, the Base Zone tracks history with `valid_from`, `valid_to`, and `is_current` columns, preserving the full timeline of changes.
- **Audit columns** — Every record carries its processing lineage.

Raj reviews the quality rules and ensures they align with the data contracts that the Asset domain publishes for downstream consumers.

### Enriched Zone — Cross-source integration and business logic

This is where the magic happens. The **Enriched Zone** (`prod_silver.*_enriched`) is where data from different sources is joined, business logic is applied, and conformed dimensions emerge.

Aisha, guided by data models designed in collaboration with Priya (Domain Steward) and James (BI Analyst), builds the enriched layer:

- **`prod_silver.asset_enriched.asset_master`** — A conformed asset dimension combining SAP equipment master data with Maximo asset attributes. Includes derived fields like `asset_age_years` (calculated from commissioning date), `asset_class`, `region`, and `operational_status`.

- **`prod_silver.operations_enriched.maintenance_history`** — Work orders from Maximo joined with asset references from SAP. Each record is classified as `planned` or `unplanned`. Maintenance frequency can now be calculated per asset per period.

- **`prod_silver.operations_enriched.service_interruptions`** — OMS interruption events enriched with asset and region context. Classified by cause, duration, and impact.

Cross-domain joins require care. Priya (Asset domain) and **Tom Bradley**, the **Data Domain Steward** for Operations, collaborate to ensure that the join keys are correct, the business definitions align, and neither domain's data is misrepresented. This is federated governance in action — central standards, domain execution.

---

## Act 5 — Gold Layer: Building the Business-Ready Data Product

### The BI Zone — Dimensional modelling for the dashboard

James now takes the lead on the Gold layer. Working with the enriched Silver datasets, he designs a **dimensional model** in the BI Zone (`prod_gold.asset_bi`) — purpose-built for the dashboard Sarah requested.

The model follows star schema principles:

**Dimension Tables:**
- `prod_gold.asset_bi.dim_asset` — Asset attributes: ID, name, class, subclass, region, commissioning date, age band, operational status
- `prod_gold.asset_bi.dim_region` — Regional hierarchy: state, district, area, locality
- `prod_gold.asset_bi.dim_date` — Calendar dimension: date, week, month, quarter, financial year

**Fact Tables:**
- `prod_gold.asset_bi.fact_maintenance` — One row per work order: asset key, date key, region key, maintenance type (planned/unplanned), cost, duration
- `prod_gold.asset_bi.fact_interruption` — One row per service interruption: asset key, date key, region key, cause, duration, customers affected

### Data contracts formalise the product

Before the Gold tables go live, a **data contract** is published for this data product. The contract specifies:

| Contract Element | Detail |
|---|---|
| **Schema** | Field names, types, nullability, primary/foreign keys |
| **Quality SLOs** | Completeness ≥ 99.5%, freshness ≤ 24 hours, referential integrity 100% |
| **Refresh cadence** | Weekly (aligned to Sarah's requirement) |
| **Owner** | Priya Sharma (Asset Domain Steward) |
| **Producer** | Aisha Patel (Data Engineering) |
| **Consumers** | James Okafor (BI), Asset Management leadership |
| **Version** | 1.0.0 (semantic versioning; breaking changes increment major) |
| **Classification** | Tier 2 (Domain) — steward oversight, data contracts, lighter change management |

This contract is the interoperability mechanism. If a schema change is needed later, the contract defines whether it is breaking or non-breaking and who must be notified.

### Governance at Gold

Raj applies final governance:

- **Governed tags updated**: `medallion_layer = gold`, `medallion_sublayer = bi`, `data_domain = asset`, `classification_status = classified`
- **Access model**: `access_model = open` — this is executive reporting data with no personal information, so it follows the "open by default, restricted by exception" principle
- **Catalogue registration** — The data product is registered in **Alation**, Water Corporation's enterprise data catalogue, with rich metadata: business descriptions, lineage visualisation, quality scores, and the data contract
- **Lineage** — Unity Catalog traces the full path: OMS/SAP/Maximo → Bronze Raw → Silver Base → Silver Enriched → Gold BI. Any downstream consumer can see exactly where every number comes from

---

## Act 6 — The Semantic Layer and Dashboard

### Metrics defined once, used everywhere

James defines the core metrics in the **semantic layer** — ensuring that "Maintenance Frequency" and "Unplanned Interruption Rate" are calculated consistently regardless of where they are consumed (dashboard, Genie space, ad hoc query, or future AI agent):

| Metric | Definition | Grain |
|---|---|---|
| **Asset Age** | `DATEDIFF(year, commissioning_date, current_date())` | Per asset |
| **Maintenance Frequency** | `COUNT(work_order_id) WHERE period = reporting_period` | Per asset × period |
| **Unplanned Interruption Rate** | `COUNT(interruption_id) WHERE cause_type = 'unplanned'` | Per asset × period |
| **Mean Time to Restore** | `AVG(restoration_duration_hours) WHERE cause_type = 'unplanned'` | Per asset × period |

These metrics are defined once in the semantic layer and become the **single source of truth**. No one rebuilds these calculations in a spreadsheet. No one debates the formula in a board meeting.

### The dashboard comes to life

James builds the dashboard in **Databricks AI/BI Dashboards**, connected directly to the Gold BI tables:

**Page 1 — Executive Summary**
- KPI tiles: Total assets under management, average asset age, unplanned interruption rate (trending), maintenance coverage ratio
- Regional heatmap: Interruption density by region
- Trend line: Quarterly interruption rate vs. maintenance frequency (the core correlation Sarah asked about)

**Page 2 — Asset Class Deep Dive**
- Breakdown by asset class (pipes, pumps, treatment plants, meters)
- Age distribution histogram per class
- Scatter plot: Asset age vs. interruption frequency (highlighting the inflection point where failures accelerate)

**Page 3 — Regional Detail**
- Filterable by region, district, and asset class
- Drill-through to individual asset maintenance and interruption history
- Comparison panels: Region A vs. Region B

**A Genie Space** is also configured — allowing Sarah and her team to ask natural language questions against the same Gold tables: *"Which regions had the highest increase in unplanned interruptions last quarter?"* — answered by an AI agent backed by governed, trusted data.

---

## Act 7 — Promotion to Production

### The deployment pipeline

Nothing reaches production without passing through the environment pipeline:

1. **Development** (`wc-edap-dev`) — Aisha and James build and test pipelines and dashboards against dev data
2. **Staging** (`wc-edap-staging`) — QA validation: data quality assertions pass, dashboard renders correctly, metrics reconcile against known values
3. **Production** (`wc-edap-prod`) — Deployed via **Databricks Asset Bundles (DABs)** — infrastructure as code, version-controlled, peer-reviewed

The promotion follows the principle: **code promotes, data does not**. The same pipeline code runs in each environment against that environment's data, ensuring consistency and reproducibility.

### Final sign-offs

| Role | Sign-off | What they verified |
|---|---|---|
| **Priya Sharma** (Data Domain Steward) | Business definitions and data quality | Metrics match agreed business definitions; quality SLOs met |
| **Raj Mehta** (Technical Data Steward) | Governance and compliance | Tags applied correctly; access model appropriate; lineage complete; catalogue updated |
| **Tom Bradley** (Operations Domain Steward) | Cross-domain data accuracy | Operations data (work orders, interruptions) is correctly represented in the asset context |
| **Aisha Patel** (Data Engineer) | Pipeline reliability | Pipelines run within SLA; error handling works; monitoring alerts configured |
| **James Okafor** (BI Analyst) | Dashboard accuracy and usability | Visualisations are correct; drill-downs work; Genie space returns sensible answers |
| **Sarah Chen** (Data Consumer / Sponsor) | Fitness for purpose | "This answers my question and I trust the numbers" |

---

## Epilogue — The Virtuous Cycle

Three months later, Sarah presents to the board. The dashboard shows a clear correlation: assets over 35 years old in the northern region have an unplanned interruption rate four times higher than the fleet average. Maintenance frequency on those assets has not kept pace.

The board approves a targeted capital renewal programme.

But the story does not end there. The data product lives on:

- **The Data Governance Lifecycle** ensures continuous improvement — quality KPIs are monitored, metadata is kept current, the data contract is versioned as the source systems evolve.
- **The Data Science team** picks up the Gold tables and builds a predictive model: *given an asset's age, class, and maintenance history, what is its probability of failure in the next 12 months?* The Data Science Lifecycle begins at *Discover*, fed by the same governed data.
- **New consumers** discover the data product in Alation and request access — the federated access model means domain stewards can grant schema-level access without a central bottleneck.
- **AI agents** in Genie spaces answer ad hoc questions from field teams, regional managers, and planners — all backed by the same metrics, the same governance, the same single source of truth.

One question became a dashboard. That dashboard became a data product. That data product became a platform capability. And every step was governed, traceable, and trusted.

---

## The Roles — A Summary

| Role | Who (in this story) | Contribution |
|---|---|---|
| **Data Consumer / Sponsor** | Sarah Chen | Asked the question; validated fitness for purpose |
| **BI Analyst** | James Okafor | Requirements, metric definition, dimensional modelling, dashboard build |
| **Data Domain Steward (Asset)** | Priya Sharma | Validated data availability, business definitions, quality thresholds |
| **Data Domain Steward (Operations)** | Tom Bradley | Ensured cross-domain accuracy for maintenance and interruption data |
| **Data Engineer** | Aisha Patel | Source onboarding, pipeline build (Bronze → Silver → Gold), deployment |
| **Technical Data Steward** | Raj Mehta | Tagging, access control, lineage, catalogue registration, compliance |
| **Data Product Owner** | Priya Sharma | Owned the data contract and product lifecycle |
| **Data Custodian** | EDAP Platform Team | Infrastructure, compute, storage, backup, security controls |

---

## The Architecture — End to End

```
 ┌─────────────────────────────────────────────────────────────────────┐
 │  SOURCE SYSTEMS                                                     │
 │  SAP ECC (Plant Maintenance) · Maximo · OMS                        │
 └──────────────────────────┬──────────────────────────────────────────┘
                            │  Lakeflow Connect
                            ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │  BRONZE — System of Record                                          │
 │  ┌─────────────┐    ┌──────────────┐                                │
 │  │  Landing     │───▶│  Raw          │  Immutable, append-only       │
 │  │  (transient) │    │  oms_raw      │  Audit columns, hash-based   │
 │  └─────────────┘    │  sap_raw      │  change detection             │
 │                      │  maximo_raw   │                                │
 │                      └──────────────┘                                │
 └──────────────────────────┬──────────────────────────────────────────┘
                            │  Lakeflow SDP (metadata-driven)
                            ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │  SILVER — Trusted Enterprise View                                   │
 │  ┌──────────────┐    ┌────────────────┐                             │
 │  │  Base         │───▶│  Enriched       │  Cross-source joins,       │
 │  │  sap_base     │    │  asset_enriched │  conformed dimensions,     │
 │  │  maximo_base  │    │  ops_enriched   │  business logic,           │
 │  │  oms_base     │    │                  │  SCD Type 2 history        │
 │  └──────────────┘    └────────────────┘                             │
 └──────────────────────────┬──────────────────────────────────────────┘
                            │  Dimensional modelling
                            ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │  GOLD — Business Ready                                              │
 │  ┌──────────────────────────────────────────┐                       │
 │  │  BI Zone (asset_bi)                       │                       │
 │  │  dim_asset · dim_region · dim_date        │                       │
 │  │  fact_maintenance · fact_interruption      │                       │
 │  └──────────────────────────────────────────┘                       │
 │         │              │             │                                │
 │    Data Contract   Semantic     Catalogue                            │
 │    (v1.0.0)        Layer        (Alation)                            │
 └─────────┬──────────────┬─────────────┬──────────────────────────────┘
           │              │             │
           ▼              ▼             ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │  CONSUMPTION                                                        │
 │  AI/BI Dashboard · Genie Space · Data Science · Ad Hoc Queries      │
 └─────────────────────────────────────────────────────────────────────┘
```

---

*The value of a platform is not in the technology. It is in the trust. Every layer, every tag, every contract, every role exists so that when Sarah stands before the board and says "the data shows…" — everyone in the room believes her.*

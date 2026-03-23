# EDAP Data Products — Definition, Taxonomy, Contracts & Governance

**Mark Shaw** | Principal Data Architect

---

## 1. What Is a Data Product?

A data product is a **curated, governed, and purposefully designed unit of data** (or data-derived insight) that is treated as a product — with a defined owner, a documented contract, measurable quality, and explicit consumers. A data product exists to **serve consumers beyond its creators**.

### The FAUQD Test

Every candidate data product must pass five tests before it earns the label:

| Test | Criteria |
|---|---|
| **Findable** | Registered in the Data Catalogue; discoverable via search, tags, and domain browsing. |
| **Addressable** | Has a stable, well-known identifier — a Unity Catalog fully-qualified name, an endpoint URI, or a report URL. |
| **Understandable** | Documented with business context, schema description, lineage, and known limitations. |
| **Quality-assured** | Has defined quality rules; measured SLAs for freshness, completeness, and accuracy; published DQ scores. |
| **Discoverable & Described** | Has a data contract specifying what the product provides, what it guarantees, and what constraints apply. |

If an asset does not meet all five criteria, it is a **data asset** — useful, but not yet a data product.

---

### 1.2 Data Product vs Data Asset

| Characteristic | Data Asset | Data Product |
|---|---|---|
| **Created for** | The team that built it | Consumers beyond the creating team |
| **Ownership** | Implicit (whoever created it) | Explicit (named Data Product Owner) |
| **Documentation** | Ad-hoc or absent | Structured — data contract, catalogue entry, lineage |
| **Quality** | Best-effort | SLA-bound — measured, monitored, alerted |
| **Discoverability** | Known to the team | Registered in the Data Catalogue |
| **Lifecycle** | Informal | Managed — versioned, deprecated, retired |
| **Contract** | None | Formal — schema, freshness, quality guarantees |
| **Access** | By request or convention | Governed — RBAC, request/approval workflow |
| **Support** | None | Defined — owner, escalation path, incident response |
| **Example** | A raw ingestion table, a scratch notebook | A certified dimension table, a published ML model |

> **Key question:** Would someone outside the creating team rely on this, and would they be able to self-serve and trust it without asking the creator directly?

---

### 1.3 Anatomy of a Data Product

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA PRODUCT                                │
│                                                                     │
│  ┌──────────────────────┐   ┌──────────────────────────────────┐   │
│  │      Identity         │   │        Data Contract              │   │
│  │  Name                 │   │  Schema / Interface               │   │
│  │  Domain               │   │  Quality guarantees               │   │
│  │  Owner                │   │  Freshness SLA                    │   │
│  │  Tier                 │   │  Semantic descriptions             │   │
│  │  Version              │   │  Breaking change policy            │   │
│  └──────────────────────┘   └──────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────┐   ┌──────────────────────────────────┐   │
│  │  Underlying Asset(s)  │   │     Governance Metadata           │   │
│  │  UC table(s)          │   │  Classification tags              │   │
│  │  ML model             │   │  Regulatory flags                 │   │
│  │  Dashboard            │   │  Access policy                    │   │
│  │  API                  │   │  Lineage                          │   │
│  └──────────────────────┘   └──────────────────────────────────┘   │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Observability                              │   │
│  │  DQ scores  │  Freshness  │  Usage stats                     │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

A data product is **not just the data** — it is the combination of the data, the contract, the governance wrapper, and the observability instrumentation.

---

### 1.4 Data Product Tiers

Water Corporation uses a four-tier model to calibrate governance effort to business criticality:

| Tier | Name | Governance Level | Characteristics | Example |
|---|---|---|---|---|
| **Tier 1** | Critical | Highest | Cross-domain; regulatory reporting, executive dashboards, external obligations. Full monitoring, full data contract, DQ SLAs, CMDB registration. | Conformed Customer dimension, SOCI compliance dataset |
| **Tier 2** | Essential | Medium | Multiple consumers within or across domains. Data contract, DQ rules, catalogue registration, named owner. | Asset condition fact table, water quality analytical dataset |
| **Tier 3** | Standard | Foundational | Single team or narrow consumers. Catalogue entry, basic documentation. DQ rules recommended but not mandated. | Team-specific feature table, department dashboard |
| **Tier 4** | Minimal (exploratory) | Minimal | Discovery and prototyping. Lightweight governance enforcing security only. | Experimentation in a sandbox |

**Tier determines:**

- Depth of data contract
- CMDB registration (Tier 1 only)
- Certification rigour
- Change management requirements
- SLA expectations

---

### 1.5 What Is NOT a Data Product

| Item | Why Not | What It Is |
|---|---|---|
| Landing zone raw files | Unstructured, unvalidated, no consumer contract | Ingestion artefact |
| Raw zone tables | Faithful copies — no consumer-facing guarantees | Data asset (Bronze) |
| Scratch notebooks | Exploratory, no schema contract, no consumers | Development artefact |
| Personal sandbox tables | Not discoverable, no governance wrapper | Personal workspace |
| Ad-hoc SQL queries | Ephemeral, no lifecycle, no contract | Analytical work |
| Single column / field | Too granular — products are meaningful units of consumption | Attribute within a product |
| CI/CD pipeline code | Infrastructure, not data | Deployment artefact |

---

## 2. Data Product Taxonomy

### 2.1 Overview

```
Data Products
├── Dataset Products
│   ├── Curated Datasets (tables / views)
│   ├── Dimensional Models (star schemas)
│   ├── Aggregate / Summary Datasets
│   └── Certified Feature Tables
├── Analytical Products
│   ├── Dashboards (sometimes)
│   ├── Reports (operational — sometimes)
│   ├── Semantic Models
│   └── Embedded Analytics
└── ML & AI Products
    ├── ML Models (registered, versioned)
    ├── ML Features (feature table / feature function)
    ├── ML Inference Endpoints
    ├── Optimisation Models
    └── GenAI Applications (RAG pipelines, agents)
```

---

### 2.2 Dataset Products

Dataset products are the most common type. The table below maps where they live in the medallion architecture and whether they qualify as data products.

| Medallion Zone | Data Product? | Rationale |
|---|---|---|
| Silver / Base | Sometimes | Base zone tables are conformed, cleansed, and typed — but they primarily serve as building blocks for downstream zones. A Base table may qualify as a data product if it has explicit external consumers, a data contract, and quality SLAs. In most cases, Base is an intermediate layer, not a consumption layer. |
| Silver / Enriched | Sometimes | Enriched zone tables combine and enrich Base tables. They can be data products when they serve cross-domain consumers with a stable contract. Enriched tables that only feed Gold are intermediate assets. |
| Gold / BI Dimensional | Yes | Dimensional models (star schemas) are purpose-built for consumption. They have defined consumers, contracts, and quality SLAs. These are the canonical dataset products. |
| Gold / BI Aggregate | Yes | Pre-aggregated summary tables serve specific analytical or reporting needs. They have clear consumers and freshness requirements. |
| Gold / Exploratory | Yes | Exploratory zone tables serve data science and advanced analytics teams with curated, governed datasets designed for exploration and feature engineering. |
| Feature Table | Yes | Certified feature tables serve ML pipelines with defined schema, freshness, and quality contracts. |
| Sandbox | No | Sandbox is for experimentation. No governance contract, no external consumers, no lifecycle management. |

> **Headline:** Most dataset products live in Gold. Silver can produce data products, but it is the exception.

---

### 2.3 Analytical Products

| Analytical Asset | Data Product? | Notes |
|---|---|---|
| Semantic Model | Yes | A semantic model (e.g. Power BI semantic model) defines the business logic layer — measures, hierarchies, relationships. It has a defined schema, consumers, and freshness expectations. It is a data product. |
| Dashboard | Sometimes | A dashboard is a data product when it is certified, has a named owner, defined consumers, and a published refresh cadence. An ad-hoc dashboard built for one meeting is not a data product. |
| Operational Report | Sometimes | An operational report is a data product when it has recurring consumers, a defined schedule, and quality expectations (e.g. a daily water quality compliance report). One-off reports are not data products. |
| Embedded Analytics | Yes | Embedded analytics (e.g. analytics embedded in a field application) serve defined consumers with contractual SLAs. They are data products. |

---

### 2.4 ML & AI Products

| ML / AI Asset | Data Product? | Notes |
|---|---|---|
| Registered ML Model | Yes | A registered, versioned ML model in Unity Catalog's Model Registry with defined input/output schema, performance metrics, and retraining cadence is a data product. |
| ML Feature | No | An individual feature is not a data product. The **feature table** that serves features to models is the data product. |
| Feature Table | Yes | A certified feature table with defined schema, freshness SLA, and quality contract is a data product. |
| Model Serving Endpoint | Yes | A real-time or batch inference endpoint with defined latency SLAs, input/output contracts, and monitoring is a data product. |
| Optimisation Model | Yes | An optimisation model (e.g. network pressure optimisation) with defined inputs, outputs, and performance guarantees is a data product. |
| GenAI Application | Yes | A GenAI application (e.g. RAG pipeline, agent) with defined inputs, outputs, guardrails, and quality metrics is a data product. |

---

### 2.5 Decision Matrix

Use this flowchart to determine whether an asset is a data product and what tier it should be assigned:

```
Does the asset serve consumers BEYOND the creating team?
│
├── NO  → Not a data product (it's a data asset)
│
└── YES
    │
    └── Would consumers break if schema / quality / freshness
        changed without notice?
        │
        ├── NO  → Consider Tier 3 (Standard)
        │
        └── YES
            │
            └── Is it used in regulatory, executive, or
                cross-domain consumption?
                │
                ├── YES → Tier 1 (Critical)
                │
                └── NO  → Tier 2 (Essential)
```

**Supplementary questions:**

| Question | If Yes | If No |
|---|---|---|
| Does the asset have a named owner? | Required for Tier 1–2, recommended for Tier 3 | Assign an owner before promoting |
| Is the asset registered in the Data Catalogue? | Required for Tier 1–3 | Register before promoting |
| Does the asset have a data contract? | Required for Tier 1–2 | Create a contract before promoting |
| Is the asset covered by regulatory obligations? | Tier 1 candidate | Tier 2–3 depending on consumers |
| Does the asset serve more than one domain? | Tier 1–2 candidate | Tier 2–3 depending on criticality |
| Is the asset in a sandbox or exploratory zone? | Tier 4 at most | Evaluate based on zone and consumers |

---

### 2.6 Product Lifecycle States

```
                    ┌──────────┐
                    │  Draft   │
                    └────┬─────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          │
        ┌──────────┐ ┌──────────┐  │
        │ Rejected │ │Published │  │
        └──────────┘ └────┬─────┘  │
              ▲           │        │
              │           ▼        │
              │     ┌───────────┐  │
              │     │Deprecated │  │
              │     └─────┬─────┘  │
              │           │        │
              │           ▼        │
              │     ┌──────────┐   │
              │     │ Retired  │   │
              │     └──────────┘   │
              │                    │
              └────────────────────┘
```

| State | Description | Catalogue Visibility | Consumers |
|---|---|---|---|
| **Draft** | Product is being developed and has not been certified. Contract is in progress. | Not visible (or marked as draft) | None — not available for consumption |
| **Published** | Product is certified, registered, and available for consumption. Contract is active. | Visible and discoverable | Active consumers; SLAs in effect |
| **Deprecated** | Product is scheduled for retirement. Consumers are notified and given a migration path. | Visible with deprecation warning | Existing consumers continue; no new consumers |
| **Retired** | Product is decommissioned. Data may be archived per retention policy. | Removed or archived | None — access revoked |
| **Rejected** | Product did not pass certification. Returned to Draft for remediation. | Not visible | None |

---

## 3. Data Contracts

### 3.1 What Is a Data Contract?

A data contract is a **formal, machine-readable agreement** between the producer of a data product and its consumers. It is the **API specification for data**.

A data contract is **NOT**:

- A data dictionary (though it includes schema information)
- A DQ rule set (though it includes quality guarantees)
- A data model diagram (though it describes structure)
- An SLA document (though it includes freshness and availability guarantees)

A data contract **ties all of these together** into a single, versioned, enforceable specification.

---

### 3.2 Contract Anatomy & Schema

The following YAML example illustrates the full anatomy of a data contract for a Tier 1 dataset product (`dim_customer`):

```yaml
# ─── Data Contract: dim_customer ───
contract:
  version: "2.0.0"
  kind: dataset

identity:
  name: dim_customer
  domain: customer
  tier: 1
  owner: GM Customer & Community
  steward: Customer Data Domain Steward
  team: Customer Analytics
  catalogue_id: alation://dataset/dim_customer

location:
  unity_catalog:
    catalog: prod_gold
    schema: customer_bi
    object: dim_customer
    type: table
  power_bi:
    workspace: Enterprise Customer Analytics
    semantic_model: Customer 360

schema:
  columns:
    - name: customer_sk
      type: bigint
      description: Surrogate key — system-generated
      pi: false
      nullable: false
    - name: customer_id
      type: string
      description: Business key — WC customer number
      pi: false
      nullable: false
    - name: customer_name
      type: string
      description: Full name of the customer
      pi: true
      nullable: false
    - name: email_address
      type: string
      description: Primary contact email
      pi: true
      nullable: true
    - name: customer_segment
      type: string
      description: Segmentation classification (Residential, Commercial, Industrial, Government)
      pi: false
      nullable: false
    - name: edap_eff_from
      type: timestamp
      description: SCD Type 2 effective-from date
      pi: false
      nullable: false
    - name: edap_eff_to
      type: timestamp
      description: SCD Type 2 effective-to date
      pi: false
      nullable: true
    - name: edap_is_current
      type: boolean
      description: Current record indicator
      pi: false
      nullable: false
  primary_key: [customer_sk]
  partitioned_by: []
  scd_type: 2

quality:
  rules:
    - name: customer_id_not_null
      type: not_null
      column: customer_id
      threshold: 1.0
    - name: customer_id_unique
      type: unique
      column: customer_id
      threshold: 1.0
    - name: segment_valid
      type: accepted_values
      column: customer_segment
      threshold: 1.0
    - name: email_format
      type: regex
      column: email_address
      threshold: 0.95
  freshness:
    schedule: daily
    sla: "07:00 AWST"
    max_staleness: 24h
  completeness:
    row_count_minimum: 500000

access:
  classification: "OFFICIAL: Sensitive — Personal"
  default_access: domain_members
  restricted_columns:
    - column: customer_name
      mask: sha256_hash
    - column: email_address
      mask: sha256_hash
  approval_required: true
  uc_grant:
    - principal: grp_customer_analysts
      privilege: SELECT
    - principal: grp_enterprise_reporting
      privilege: SELECT

lineage:
  upstream:
    - prod_silver.customer_base.stg_customer
    - prod_silver.customer_enriched.customer_master
  downstream:
    - prod_gold.customer_bi.fact_service_request
    - "power_bi://Enterprise Customer Analytics/Customer 360"

lifecycle:
  state: published
  published_date: "2025-06-15"
  next_review: "2026-06-15"
  breaking_change_policy: >
    Breaking changes require 90-day notice, consumer impact assessment,
    and Data Governance Council approval for Tier 1 products.
  deprecation_notice: null

support:
  slack_channel: "#data-customer-domain"
  on_call: Customer Analytics Team
  documentation: "https://alation.watercorp.com.au/dataset/dim_customer"
```

**Key design decisions:**

| Decision | Rationale |
|---|---|
| **YAML format** | Human-readable, machine-parseable, version-controllable. Preferred over JSON for documentation clarity. |
| **UC-native location** | Location block maps directly to Unity Catalog's three-level namespace (`catalog.schema.object`), enabling automated validation. |
| **WAICP classification** | Access classification uses WAICP labels (e.g. `OFFICIAL: Sensitive — Personal`) as the mandatory base, consistent with the EDAP Tagging Strategy. |
| **Numeric quality thresholds** | Thresholds are expressed as decimals (0.0–1.0) for programmatic evaluation. `1.0` means zero tolerance; `0.95` means 95% compliance. |
| **Lineage as UC FQNs** | Upstream and downstream references use Unity Catalog fully-qualified names for automated lineage graph construction. |

---

### 3.3 Contracts in Practice

#### Example 1: Gold Dimensional Model (Tier 1)

The `dim_customer` contract above (Section 3.2) is the reference example for a Tier 1 Gold dimensional model. It demonstrates full schema definition, quality rules with numeric thresholds, SCD Type 2 handling, column-level PI classification, masking rules, WAICP classification, and consumer lineage.

#### Example 2: ML Model (Tier 2)

```yaml
# ─── Data Contract: pipe_failure_predictor ───
contract:
  version: "1.3.0"
  kind: ml_model

identity:
  name: pipe_failure_predictor
  domain: asset
  tier: 2
  owner: Asset Analytics Lead
  steward: Asset Data Domain Steward
  team: Asset Intelligence
  catalogue_id: alation://model/pipe_failure_predictor

location:
  unity_catalog:
    catalog: prod_gold
    schema: asset_ml
    object: pipe_failure_predictor
    type: registered_model

schema:
  input:
    - name: asset_id
      type: string
      description: WC asset identifier
    - name: pipe_material
      type: string
      description: Pipe material classification
    - name: age_years
      type: double
      description: Age of pipe in years
    - name: soil_corrosivity
      type: string
      description: Soil corrosivity classification
    - name: failure_history_count
      type: integer
      description: Number of prior failures
  output:
    - name: failure_probability
      type: double
      description: Predicted probability of failure in next 12 months (0.0–1.0)
    - name: risk_category
      type: string
      description: Risk classification (Low, Medium, High, Critical)

quality:
  metrics:
    - name: auc_roc
      threshold: 0.85
      description: Area under ROC curve — minimum acceptable model performance
    - name: prediction_drift
      threshold: 0.10
      description: Maximum acceptable prediction distribution drift (PSI)
  freshness:
    retrain_cadence: quarterly
    serving_latency_p99: 200ms

lifecycle:
  state: published
  published_date: "2025-09-01"
  next_review: "2026-03-01"
  breaking_change_policy: >
    Input schema changes require 60-day notice and consumer impact assessment.

support:
  slack_channel: "#data-asset-domain"
  on_call: Asset Intelligence Team
  documentation: "https://alation.watercorp.com.au/model/pipe_failure_predictor"
```

#### Example 3: Power BI Semantic Model (Tier 2)

```yaml
# ─── Data Contract: Water Quality Analytics ───
contract:
  version: "1.1.0"
  kind: semantic_model

identity:
  name: water_quality_analytics
  domain: operations
  tier: 2
  owner: Water Quality Manager
  steward: Operations Data Domain Steward
  team: Water Quality Analytics
  catalogue_id: alation://semantic_model/water_quality_analytics

location:
  power_bi:
    workspace: Operations Analytics
    semantic_model: Water Quality Analytics
    connection_mode: DirectQuery
  unity_catalog:
    upstream_catalog: prod_gold
    upstream_schema: operations_bi

schema:
  measures:
    - name: avg_turbidity
      description: Average turbidity reading across selected treatment plants (NTU)
    - name: compliance_rate
      description: Percentage of samples meeting ADWG guidelines
    - name: exceedance_count
      description: Count of samples exceeding regulatory thresholds
  dimensions:
    - name: treatment_plant
      description: Treatment plant hierarchy (Region → Scheme → Plant)
    - name: sample_date
      description: Date dimension for temporal analysis
    - name: parameter
      description: Water quality parameter (turbidity, pH, chlorine residual, etc.)

quality:
  freshness:
    connection_mode: DirectQuery
    underlying_refresh: daily by 07:00 AWST
  certification:
    power_bi_certified: true
    certified_by: Operations Data Domain Steward
    certified_date: "2025-08-01"

lifecycle:
  state: published
  published_date: "2025-08-01"
  next_review: "2026-08-01"

support:
  slack_channel: "#data-operations-domain"
  on_call: Water Quality Analytics Team
  documentation: "https://alation.watercorp.com.au/semantic_model/water_quality_analytics"
```

---

### 3.4 Contract Enforcement in Unity Catalog

Data contracts are not just documentation — they are enforced through Unity Catalog mechanisms:

| Contract Element | UC Enforcement Mechanism |
|---|---|
| Schema (columns, types) | Delta table schema enforcement; schema evolution policies |
| Primary key | `ALTER TABLE ADD CONSTRAINT` — primary key constraint |
| Not-null rules | `ALTER TABLE ADD CONSTRAINT` — NOT NULL constraint |
| Quality rules | Lakeflow Spark Declarative Pipelines expectations; Databricks Data Quality Monitors |
| Freshness SLA | Databricks Data Quality Monitors; alerting via workflows |
| Classification | Unity Catalog tags (`waicp_classification`, `pi_contained`) |
| Column masking | Unity Catalog column masks (dynamic masking functions) |
| Row-level security | Unity Catalog row filters |
| Access grants | `GRANT SELECT ON TABLE ... TO ...` |
| Lineage | Unity Catalog automatic lineage tracking |
| Versioning | Table properties (`contract_version`) |

**Table properties as contract anchors:**

```sql
ALTER TABLE prod_gold.customer_bi.dim_customer
SET TBLPROPERTIES (
  'data_product.name'        = 'dim_customer',
  'data_product.domain'      = 'customer',
  'data_product.tier'        = '1',
  'data_product.owner'       = 'GM Customer & Community',
  'data_product.steward'     = 'Customer Data Domain Steward',
  'data_product.state'       = 'published',
  'contract.version'         = '2.0.0',
  'contract.freshness_sla'   = 'daily by 07:00 AWST',
  'contract.quality_score'   = '0.98',
  'contract.scd_type'        = '2',
  'contract.breaking_change' = '90-day notice'
);
```

---

### 3.5 Contract Versioning & Breaking Changes

| Change | Breaking? | Action Required |
|---|---|---|
| Add optional column | No | Minor version bump |
| Add quality rule | No | Minor version bump |
| Tighten quality threshold | No | Minor version bump; notify consumers |
| Fix typo in description | No | Patch version bump |
| Rename column | Yes | Major version bump; 90-day notice (Tier 1), 60-day notice (Tier 2) |
| Remove column | Yes | Major version bump; 90-day notice (Tier 1), 60-day notice (Tier 2) |
| Change column type | Yes | Major version bump; consumer impact assessment |
| Change primary key | Yes | Major version bump; consumer impact assessment |
| Relax quality threshold | Yes | Major version bump; notify consumers |
| Change freshness SLA | Yes | Major version bump; notify consumers |
| Change partitioning | Sometimes | Evaluate consumer impact; minor or major version bump |

**Versioning convention:**

| Component | When to Increment |
|---|---|
| **Major** (e.g. 2.0.0 → 3.0.0) | Breaking changes — schema changes, removed columns, changed types, relaxed SLAs |
| **Minor** (e.g. 2.0.0 → 2.1.0) | Non-breaking additions — new columns, new quality rules, tightened thresholds |
| **Patch** (e.g. 2.0.0 → 2.0.1) | Documentation fixes, description updates, metadata corrections |

---

## 4. Roles & Responsibilities

### 4.1 Data Product Owner (DPO)

The Data Product Owner is **accountable for the end-to-end value, quality, and lifecycle** of a data product. This role ensures the product meets consumer needs, maintains its contract, and is governed appropriately.

**Responsibilities:**

- Define and maintain the data contract
- Ensure quality SLAs are met and monitored
- Manage the product lifecycle (publish, deprecate, retire)
- Prioritise enhancements and defect resolution
- Approve breaking changes and manage consumer communication
- Ensure catalogue registration and documentation currency
- Escalate governance issues to the Data Domain Steward

**Who fills the role:**

- For Tier 1–2 products: a named individual (Product Manager, Senior SME, or domain lead)
- For Tier 3 products: the team lead responsible for the product
- For Tier 4 products: the creator (lightweight ownership)

> **Important:** The Data Product Owner is **not the same as the Data Domain Steward**. The DPO owns a specific product; the Data Domain Steward governs the domain's data landscape. A Data Domain Steward may oversee multiple DPOs.

---

### 4.2 Data Product Developer

The Data Product Developer **builds and maintains** the data product's underlying assets, pipelines, and quality instrumentation.

**Responsibilities:**

- Implement the data contract in code (schema, quality rules, freshness)
- Build and maintain pipelines (Lakeflow Spark Declarative Pipelines)
- Implement observability (DQ monitors, alerting, usage tracking)
- Apply governance metadata (tags, classifications, access grants)
- Execute schema migrations and versioning
- Support incident response and troubleshooting

---

### 4.3 Data Domain Steward (in Product Context)

The Data Domain Steward provides **domain-level governance oversight** across all data products within their domain.

**Responsibilities in the product context:**

- Review and approve data product registrations and certifications
- Ensure products comply with domain data standards and naming conventions
- Coordinate cross-product lineage and dependency management
- Escalate cross-domain issues to the Data Governance Council
- Ensure PI classification and WAICP tagging compliance
- Review and approve access requests for sensitive products

---

### 4.4 Data Consumer

Data consumers have both **rights and responsibilities**.

**Rights:**

- Discover and access published data products through the Data Catalogue
- Rely on the data contract (schema, quality, freshness) as published
- Receive advance notice of breaking changes per the contract's breaking change policy
- Escalate quality or freshness issues to the Data Product Owner

**Responsibilities:**

- Use data within the bounds of its classification and access policy
- Report data quality issues via the defined support channel
- Migrate from deprecated products within the stated deprecation window
- Not circumvent access controls or consume data outside governed channels

---

### 4.5 Platform Team

The Platform Team provides the **infrastructure, tooling, and enablement** for data products.

**Responsibilities:**

- Maintain Unity Catalog, Lakeflow Spark Declarative Pipelines, and monitoring infrastructure
- Provide templates and accelerators for data product creation
- Enforce platform-level governance (tagging policies, access policies, audit logging)
- Support data product teams with onboarding, tooling, and incident response
- Manage the Data Catalogue integration (Alation ↔ Unity Catalog sync)
- Operate the sandbox and exploratory environments

---

### 4.6 RACI Matrix

| Activity | Data Product Owner | Data Product Developer | Data Domain Steward | Data Consumer | Platform Team |
|---|---|---|---|---|---|
| Define data contract | **A** | R | C | C | I |
| Implement pipeline | I | **A** | I | — | C |
| Apply governance tags | C | **R** | **A** | — | C |
| Register in catalogue | **A** | R | C | — | I |
| Monitor quality SLAs | **A** | R | I | I | C |
| Approve access requests | C | — | **A** | — | R |
| Manage breaking changes | **A** | R | C | I | I |
| Deprecate / retire product | **A** | R | C | I | I |
| Incident response | **A** | R | I | I | C |
| Certify product | C | — | **A** | — | I |

**A** = Accountable, **R** = Responsible, **C** = Consulted, **I** = Informed

---

## 5. Data Product Governance

### 5.1 Governance Framework

Data product governance operates at three levels:

| Level | Scope | Who | Examples |
|---|---|---|---|
| **Platform** | Cross-cutting policies and infrastructure | Platform Team, CDO | Tagging policy, access model, catalogue standards, monitoring infrastructure |
| **Domain** | Domain-level standards and certification | Data Domain Steward, Data Owners | Domain naming conventions, certification criteria, cross-product lineage |
| **Product** | Individual product lifecycle and contract | Data Product Owner, Developer | Contract maintenance, quality monitoring, consumer communication |

---

### 5.2 Registration & Certification Process

```
┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Create   │───▶│   Register   │───▶│   Certify    │───▶│   Publish    │
│  Draft    │    │  in Catalogue │    │  (checklist) │    │              │
└──────────┘    └──────────────┘    └──────┬───────┘    └──────────────┘
                                          │
                                          ▼ (Fail)
                                   ┌──────────────┐
                                   │   Rejected   │
                                   │  (remediate) │
                                   └──────────────┘
```

**Certification checklist (by tier):**

| # | Criterion | Tier 1 | Tier 2 | Tier 3 | Tier 4 |
|---|---|---|---|---|---|
| 1 | Named Data Product Owner | Required | Required | Required | Recommended |
| 2 | Data contract defined | Full | Full | Lightweight | N/A |
| 3 | Registered in Data Catalogue | Required | Required | Required | Optional |
| 4 | Schema documented (all columns described) | Required | Required | Recommended | Optional |
| 5 | Quality rules implemented and passing | Required | Required | Recommended | N/A |
| 6 | Freshness SLA defined and monitored | Required | Required | Optional | N/A |
| 7 | WAICP classification applied | Required | Required | Required | Required |
| 8 | PI columns identified and masked | Required | Required | Required | Required |
| 9 | Lineage documented (upstream & downstream) | Required | Required | Recommended | N/A |
| 10 | Access grants configured | Required | Required | Required | Required |
| 11 | CMDB CI registered | Required | N/A | N/A | N/A |
| 12 | Data Domain Steward sign-off | Required | Required | Recommended | N/A |

---

### 5.3 Quality Standards & SLAs

| Quality Dimension | Description | Tier 1 SLA | Tier 2 SLA | Tier 3 SLA |
|---|---|---|---|---|
| **Freshness** | Data updated within the defined schedule | Monitored, alerted, escalated | Monitored, alerted | Best-effort |
| **Completeness** | No unexpected missing values or rows | ≥ 99.5% | ≥ 99% | ≥ 95% |
| **Accuracy** | Values conform to business rules and expectations | ≥ 99.5% | ≥ 99% | ≥ 95% |
| **Uniqueness** | Primary keys and business keys are unique | 100% | 100% | 100% |
| **Consistency** | Cross-dataset referential integrity maintained | Monitored | Monitored | Best-effort |
| **Validity** | Values within defined ranges and formats | ≥ 99.5% | ≥ 99% | ≥ 95% |

---

### 5.4 Discoverability & Documentation Standards

Every published data product must have the following catalogue fields populated:

| Field | Tier 1 | Tier 2 | Tier 3 | Tier 4 |
|---|---|---|---|---|
| Name | Required | Required | Required | Required |
| Description (business context) | Required | Required | Required | Optional |
| Domain | Required | Required | Required | Required |
| Owner | Required | Required | Required | Recommended |
| Tier | Required | Required | Required | Required |
| WAICP classification | Required | Required | Required | Required |
| Schema description (all columns) | Required | Required | Recommended | Optional |
| Quality score | Required | Required | Optional | N/A |
| Freshness SLA | Required | Required | Optional | N/A |
| Lineage (upstream & downstream) | Required | Required | Recommended | N/A |
| Data contract link | Required | Required | Optional | N/A |
| Support channel | Required | Required | Recommended | N/A |
| Last reviewed date | Required | Required | Recommended | N/A |

---

### 5.5 Access & Sharing Model

Access to data products is governed by WAICP classification:

| WAICP Classification | Default Access | Approval Required | Column Masking | Sharing (Delta Sharing) |
|---|---|---|---|---|
| UNOFFICIAL | Open to all authenticated users | No | No | Permitted |
| OFFICIAL | Domain members by default; cross-domain on request | No | No | Permitted with governance |
| OFFICIAL: Sensitive — Personal | Domain members with PI access; cross-domain requires approval | Yes | Yes — PI columns masked by default | Restricted — approval required |
| OFFICIAL: Sensitive — Commercial | Named groups only | Yes | Case-by-case | Restricted — approval required |
| OFFICIAL: Sensitive — Cabinet | Restricted to named individuals | Yes — CDO approval | Yes | Not permitted via Delta Sharing |
| OFFICIAL: Sensitive — Legal | Legal team and named individuals | Yes — Legal approval | Yes | Not permitted via Delta Sharing |

---

### 5.6 Deprecation & Retirement

A five-step process governs the end of a data product's lifecycle:

| Step | Action | Detail |
|---|---|---|
| **1** | Announce deprecation | Data Product Owner notifies all known consumers via catalogue, Slack, and email. State changes to `deprecated`. |
| **2** | Set migration window | Define the deprecation period based on tier: Tier 1 = 90 days minimum, Tier 2 = 60 days, Tier 3 = 30 days. |
| **3** | Provide migration path | Document the replacement product (if any), migration guide, and support channel. |
| **4** | Monitor consumer migration | Track consumer adoption of the replacement. Follow up with consumers who have not migrated. |
| **5** | Retire | Remove access, archive data per retention policy, update catalogue state to `retired`. Deregister from CMDB (Tier 1). |

---

### 5.7 Regulatory Overlay

Data products must comply with the following regulatory obligations:

| Regulation | Impact on Data Products |
|---|---|
| **SOCI Act 2018** (Security of Critical Infrastructure) | Tier 1 products containing critical infrastructure data must implement enhanced access controls, audit logging, and incident response procedures. SCADA-derived products require additional sensitivity handling. |
| **PRIS Act 2024** (Privacy and Responsible Information Sharing) | Products containing Personal Information (PI) must apply column masking, restrict access, log all access events, and support right-to-be-forgotten requests (via de-identification in Bronze, masking in Silver/Gold). |
| **State Records Act 2000** | Data products must comply with retention schedules. Retirement does not mean deletion — data must be archived per the applicable retention period. |
| **Essential Eight** | Platform-level controls (MFA, application hardening, patching, access restriction) underpin data product security. Tier 1 products should be assessed against Essential Eight maturity levels. |

---

## 6. Data Products & the Enterprise Landscape

### 6.1 Data Catalogue

**Purpose:** The Data Catalogue (Alation) is the **shop front** for data products. It is where consumers discover, understand, and request access to data products.

**Contents:**

- Business descriptions and context
- Schema documentation
- Quality scores and freshness status
- Lineage visualisation
- Data contract links
- Access request workflows
- Owner and support information

**Analogy:** If a data product is a product on a shelf, the Data Catalogue is the shop where consumers browse, read the label, and add it to their cart. Unity Catalog is the warehouse that stores and governs the product. The data contract is the product specification sheet.

**Implementation note:** Alation syncs metadata from Unity Catalog. Tags, descriptions, and lineage authored in UC are surfaced in Alation. Business context, certification status, and access request workflows are managed in Alation.

---

### 6.2 CMDB

**Scope:** Tier 1 data products only.

The Configuration Management Database (CMDB) is an IT service management tool that tracks Configuration Items (CIs). Not every data product belongs in the CMDB — only those that represent critical business services.

**Arguments for CMDB registration (Tier 1):**

- Tier 1 products underpin regulatory reporting, executive dashboards, or external obligations
- Incident management and change management processes reference CMDB CIs
- Business continuity planning requires visibility of critical data dependencies

**Arguments against universal CMDB registration:**

- CMDB is designed for IT services, not data assets — registering all products creates noise
- Unity Catalog and the Data Catalogue already provide metadata, lineage, and ownership
- Maintaining duplicate records across CMDB and catalogue creates governance overhead

**Boundary table:**

| Attribute | Data Catalogue (Alation) | Unity Catalog | CMDB |
|---|---|---|---|
| Business description | Yes | No | No |
| Schema & columns | Yes (synced from UC) | Yes (authoritative) | No |
| Quality scores | Yes | Yes (monitors) | No |
| Lineage | Yes (synced from UC) | Yes (authoritative) | No |
| Access grants | Yes (request workflow) | Yes (authoritative) | No |
| Owner / steward | Yes | Yes (tags) | Yes (CI owner) |
| Tier | Yes | Yes (tags) | Yes (CI criticality) |
| Incident management | No | No | Yes |
| Change management | No | No | Yes |
| Business continuity | No | No | Yes |
| SLA monitoring | Yes (freshness) | Yes (monitors) | Yes (service SLA) |

**CI attributes for Tier 1 data products:**

| CMDB Attribute | Source | Example |
|---|---|---|
| CI Name | Data contract | `dim_customer` |
| CI Type | Fixed | Data Product |
| CI Owner | Data contract | GM Customer & Community |
| Criticality | Data contract tier | Tier 1 — Critical |
| Environment | UC catalog | `prod_gold` |
| Dependencies (upstream) | Data contract lineage | `prod_silver.customer_base.stg_customer` |
| Dependencies (downstream) | Data contract lineage | `power_bi://Enterprise Customer Analytics/Customer 360` |
| Support group | Data contract | Customer Analytics Team |
| SLA | Data contract | Daily by 07:00 AWST |

---

### 6.3 Unity Catalog

Unity Catalog is the **governance engine** for data products on EDAP. It provides:

| Capability | Role in Data Products |
|---|---|
| **Three-level namespace** (`catalog.schema.object`) | Provides stable, addressable identifiers for data products |
| **Tags** | Store product metadata (tier, domain, owner, state, classification) as key-value pairs |
| **Lineage** | Automatically tracks column-level and table-level lineage across pipelines |
| **Grants** | Enforce access control via RBAC and ABAC (attribute-based access control) |
| **Column masks** | Apply dynamic masking to PI and sensitive columns |
| **Row filters** | Apply row-level security for multi-tenant or classification-based access |
| **System tables** | Provide audit logs, access history, and usage statistics for observability |
| **Model Registry** | Register, version, and serve ML models as data products |
| **Table properties** | Store contract metadata (version, SLA, tier, owner) as key-value properties |

---

### 6.4 Relationship Map

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CONSUMER EXPERIENCE                                  │
│                                                                             │
│   "I need customer data for my dashboard"                                   │
│                                                                             │
│   1. Search Data Catalogue (Alation)                                        │
│   2. Find dim_customer — Tier 1, Published, Quality Score 98%               │
│   3. Read data contract — schema, SLA, classification                       │
│   4. Request access (if not already granted)                                │
│   5. Consume via UC fully-qualified name: prod_gold.customer_bi.dim_customer│
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATA CATALOGUE (Alation)                            │
│                                                                             │
│   Business descriptions │ Quality scores │ Lineage │ Access requests        │
│   Certification status  │ Owner/steward  │ Contract links                   │
│                                                                             │
│   Syncs metadata FROM Unity Catalog                                         │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       UNITY CATALOG (Governance Engine)                      │
│                                                                             │
│   Namespace │ Tags │ Lineage │ Grants │ Masks │ Filters │ System Tables     │
│   Model Registry │ Table Properties │ Constraints                           │
│                                                                             │
│   Authoritative source for schema, access, lineage, quality                 │
└──────────────┬──────────────────────────────────┬───────────────────────────┘
               │                                  │
               ▼                                  ▼
┌──────────────────────────┐    ┌─────────────────────────────────────────────┐
│         CMDB              │    │              DATA CONTRACT                   │
│   (Tier 1 only)           │    │                                             │
│                           │    │   YAML specification                         │
│   CI Name, Owner          │    │   Schema, Quality, Freshness, Access         │
│   Criticality, SLA        │    │   Lineage, Lifecycle, Support                │
│   Incident management     │    │                                             │
│   Change management       │    │   Stored in version control                  │
│   Business continuity     │    │   Anchored in UC table properties            │
└──────────────────────────┘    └─────────────────────────────────────────────┘
```

---

### 6.5 Data Products in the EADM Context

The Enterprise Architecture Data Model (EADM) defines conceptual entities that map to data products:

| EADM Entity | Data Product Mapping | Example |
|---|---|---|
| Customer | Conformed Customer dimension | `prod_gold.customer_bi.dim_customer` |
| Asset | Asset dimension and condition fact tables | `prod_gold.asset_bi.dim_asset`, `prod_gold.asset_bi.fact_asset_condition` |
| Work Order | Maintenance and work order fact tables | `prod_gold.operations_bi.fact_work_order` |
| Water Quality Sample | Water quality fact and compliance datasets | `prod_gold.operations_bi.fact_water_quality` |
| Financial Transaction | Financial fact tables | `prod_gold.finance_bi.fact_financial_transaction` |
| Employee | People dimension | `prod_gold.people_bi.dim_employee` |
| Service Request | Customer service request fact table | `prod_gold.customer_bi.fact_service_request` |
| Network Facility | Network and facility dimension | `prod_gold.asset_bi.dim_network_facility` |
| Meter | Meter dimension and consumption fact | `prod_gold.asset_bi.dim_meter`, `prod_gold.operations_bi.fact_consumption` |

---

## 7. Data Products in the Medallion Architecture

### 7.1 Zone-by-Zone Analysis

| Zone | Layer | Purpose | Data Products? | Rationale |
|---|---|---|---|---|
| **Landing** | Pre-Bronze | Temporary staging for incoming files and streams | No | Transient — no schema, no consumers, no contract |
| **Raw** | Bronze | Faithful, immutable copy of source data | No | System of record — not consumer-facing. No transformation, no quality guarantees beyond fidelity to source. |
| **Protected** | Bronze | De-identified copy for right-to-be-forgotten compliance | No | Governance mechanism — not a consumption layer |
| **Base** | Silver | Cleansed, typed, conformed source-aligned tables | Sometimes | Base tables can be products if they have external consumers and contracts. Most are intermediate. |
| **Enriched** | Silver | Cross-source joined, enriched, business-aligned tables | Sometimes | Enriched tables can be products if they serve cross-domain consumers. Most feed Gold. |
| **BI (Dimensional)** | Gold | Star schemas, aggregate tables for analytics and reporting | Yes | The primary home for dataset products. Purpose-built for consumption with defined contracts. |
| **Exploratory** | Gold | Curated datasets for data science and advanced analytics | Yes | Governed exploration datasets with defined schema and quality. |
| **Sandbox** | N/A | Personal or team experimentation space | No | No governance contract, no external consumers, no lifecycle. |

---

### 7.3 Lineage

A typical lineage chain from source to data product:

```
┌────────────┐    ┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│  SAP ECC   │───▶│   Landing     │───▶│   Raw         │───▶│   Base        │
│  (Source)   │    │  (Pre-Bronze) │    │  (Bronze)     │    │  (Silver)     │
└────────────┘    └───────────────┘    └───────────────┘    └───────┬───────┘
                                                                    │
                                                                    ▼
                                       ┌───────────────┐    ┌───────────────┐
                                       │   BI          │◀───│   Enriched    │
                                       │  (Gold)       │    │  (Silver)     │
                                       │  ★ PRODUCT ★  │    └───────────────┘
                                       └───────────────┘
```

The data product is the **Gold BI table** — it is the consumer-facing asset with a contract, quality SLAs, and governance metadata. Everything upstream is the supply chain that produces it.

---

## 8. Implementing Data Products on Databricks

### 8.1 Unity Catalog as Product Registry

Unity Catalog tags and table properties serve as the product registry. Use tags to classify products and table properties to store contract metadata.

**Product-level tags:**

```sql
ALTER TABLE prod_gold.customer_bi.dim_customer
SET TAGS ('data_product.tier' = '1',
          'data_product.domain' = 'customer',
          'data_product.owner' = 'GM Customer & Community',
          'data_product.state' = 'published',
          'data_product.contract_version' = '2.0.0');
```

**Column-level tags:**

```sql
ALTER TABLE prod_gold.customer_bi.dim_customer
ALTER COLUMN customer_name
SET TAGS ('pi_contained' = 'true',
          'masking_required' = 'true',
          'sensitivity_type' = 'personal_name');

ALTER TABLE prod_gold.customer_bi.dim_customer
ALTER COLUMN email_address
SET TAGS ('pi_contained' = 'true',
          'masking_required' = 'true',
          'sensitivity_type' = 'contact_email');
```

**Query to find all Tier 1 data products:**

```sql
SELECT t.catalog_name,
       t.schema_name,
       t.table_name,
       tag.tag_value AS tier
FROM   system.information_schema.tables t
JOIN   system.information_schema.table_tags tag
  ON   t.table_catalog = tag.catalog_name
 AND   t.table_schema  = tag.schema_name
 AND   t.table_name    = tag.table_name
WHERE  tag.tag_name    = 'data_product.tier'
  AND  tag.tag_value   = '1';
```

---

### 8.2 Tagging & Classification

The data product tag namespace extends the EDAP Tagging Strategy:

| Tag | Scope | Values | Description |
|---|---|---|---|
| `data_product.name` | Table | Free text | Canonical product name |
| `data_product.tier` | Table | `1`, `2`, `3`, `4` | Product tier |
| `data_product.domain` | Table | Domain name | Owning data domain |
| `data_product.owner` | Table | Role / name | Accountable owner |
| `data_product.steward` | Table | Role / name | Data Domain Steward |
| `data_product.state` | Table | `draft`, `published`, `deprecated`, `retired` | Lifecycle state |
| `data_product.contract_version` | Table | Semantic version | Current contract version |
| `data_product.certified_date` | Table | ISO date | Date of last certification |
| `data_product.next_review` | Table | ISO date | Scheduled review date |
| `data_product.catalogue_id` | Table | Alation URI | Link to catalogue entry |

---

### 8.3 Monitoring & Observability

| What to Monitor | How | Alerting |
|---|---|---|
| **Data quality scores** | Databricks Data Quality Monitors; Lakeflow Spark Declarative Pipelines expectations | Alert on threshold breach (per contract) |
| **Freshness (staleness)** | Databricks Data Quality Monitors; workflow schedule monitoring | Alert if data not refreshed by SLA time |
| **Row count anomalies** | Data Quality Monitors — row count profiling | Alert on significant deviation from baseline |
| **Schema drift** | Delta schema enforcement; Data Quality Monitors | Alert on unexpected schema changes |
| **Access patterns** | Unity Catalog system tables (`system.access.audit`) | Alert on unusual access (e.g. bulk export, new principal) |
| **Consumer usage** | Unity Catalog system tables (`system.access.table_lineage`) | Report on consumption trends; identify unused products |
| **ML model performance** | Databricks Model Monitoring; custom metrics | Alert on model drift, performance degradation |
| **Pipeline failures** | Lakeflow Spark Declarative Pipelines; workflow alerts | Alert on pipeline failure; escalate per support channel |

---

### 8.4 Templates & Accelerators

The Platform Team provides starter templates to accelerate data product creation:

| Template | Contents | Target Tier |
|---|---|---|
| **Gold Dimensional Model** | Delta table DDL, Lakeflow Spark Declarative Pipelines notebook, data contract YAML, quality rules, UC tags SQL, catalogue registration script | Tier 1–2 |
| **Gold Aggregate Table** | Delta table DDL, aggregation pipeline, data contract YAML, quality rules | Tier 2–3 |
| **ML Model Product** | Model Registry setup, serving endpoint config, data contract YAML, monitoring config | Tier 1–2 |
| **Feature Table** | Feature table DDL, feature engineering pipeline, data contract YAML, quality rules | Tier 2–3 |
| **Semantic Model** | Power BI semantic model template, data contract YAML, certification checklist | Tier 2–3 |
| **Data Contract YAML** | Blank contract template with all sections, inline documentation | All tiers |

---

## 9. Data Product Catalogue (Registry)

Each registered data product has the following page properties in the catalogue:

| Property | Description | Example |
|---|---|---|
| **Product Name** | Canonical name of the data product | `dim_customer` |
| **Domain** | Owning data domain | Customer |
| **Tier** | Product tier (1–4) | 1 |
| **Type** | Product type (Dataset, Semantic Model, ML Model, etc.) | Dataset |
| **Owner** | Data Product Owner | GM Customer & Community |
| **Steward** | Data Domain Steward | Customer Data Domain Steward |
| **State** | Lifecycle state | Published |
| **Contract Version** | Current contract version | 2.0.0 |
| **UC Location** | Unity Catalog fully-qualified name | `prod_gold.customer_bi.dim_customer` |
| **WAICP Classification** | WAICP classification label | OFFICIAL: Sensitive — Personal |
| **Quality Score** | Latest composite quality score | 98% |
| **Freshness SLA** | Defined freshness SLA | Daily by 07:00 AWST |
| **Last Refreshed** | Timestamp of last successful refresh | 2026-03-20 06:45 AWST |
| **Published Date** | Date the product was first published | 2025-06-15 |
| **Next Review** | Scheduled review date | 2026-06-15 |
| **Support Channel** | Primary support channel | `#data-customer-domain` |
| **Catalogue Link** | Link to full catalogue entry | `https://alation.watercorp.com.au/dataset/dim_customer` |
| **CMDB CI** | CMDB CI reference (Tier 1 only) | CI-DP-001 |

---

## 10. FAQ & Decision Guides

| Question | Answer |
|---|---|
| **Is a raw zone table a data product?** | No. Raw zone tables are faithful copies of source data with no consumer-facing contract. They are data assets, not products. |
| **Can a Silver table be a data product?** | Sometimes. A Silver Base or Enriched table can be a data product if it has external consumers, a data contract, and quality SLAs. Most Silver tables are intermediate assets. |
| **Does every Gold table need to be a data product?** | No. Gold tables in the BI zone are strong candidates, but a Gold table only becomes a product when it has a named owner, a contract, and consumers. |
| **Who decides the tier?** | The Data Product Owner proposes the tier; the Data Domain Steward approves it. Tier 1 requires Data Governance Council endorsement. |
| **Can a dashboard be a data product?** | Sometimes. A certified dashboard with a named owner, defined consumers, and a published refresh cadence can be a data product. An ad-hoc dashboard is not. |
| **What if I can't fill out the full data contract?** | Start with the identity, schema, and quality sections. Iterate. A Tier 3 product needs a lighter contract than Tier 1. Use the templates provided by the Platform Team. |
| **Do I need to register in the CMDB?** | Only for Tier 1 products. The Data Catalogue and Unity Catalog handle metadata for Tier 2–4. |
| **What happens if a product fails its quality SLA?** | The monitoring system alerts the Data Product Owner and support channel. The owner triages, the developer investigates, and the incident is resolved per the support model. Persistent failures may trigger a review of the product's tier or state. |
| **How do I deprecate a product?** | Follow the five-step deprecation process (Section 5.6). Announce, set a migration window, provide a migration path, monitor consumer migration, then retire. |
| **Can a sandbox table become a data product?** | Not directly. If experimentation produces a valuable asset, it must be promoted to the appropriate Gold (or Silver) zone, registered, contracted, and certified before it becomes a product. |
| **Who owns cross-domain data products?** | Cross-domain products are typically Tier 1. Ownership sits with the primary domain; stewardship involves all contributing domains. The Data Governance Council arbitrates disputes. |
| **How does this relate to Data Mesh?** | Water Corporation's model is federated, not full Data Mesh. Domains own their products, but the platform and governance framework are centralised. This gives domains autonomy within guardrails. |

---

## Companion Documents

This document should be read alongside the following EDAP wiki articles:

- **Medallion Architecture** — Zone decomposition and data flow conventions across Bronze, Silver, and Gold layers
- **EDAP Access Model** — Databricks Unity Catalog & Federated Domain Governance
- **EDAP Tagging Strategy** — Classification & Metadata Tagging Framework
- **Core Data Governance Roles in the Enterprise** — Role definitions, accountability boundaries, and decision rights
- **Data Quality Framework** — Quality dimensions, rules, monitoring, and SLA standards
- **Governing Data Across Source Systems and EDAP** — Three-layer governance architecture, data contracts, and cross-system governance
- **EDAP Pipeline Framework Requirements Specification** — Pipeline standards for Lakeflow Spark Declarative Pipelines

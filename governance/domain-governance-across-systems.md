# Governing Data Across Source Systems and the Enterprise Data & Analytics Platform

**Mark Shaw** | Principal Data Architect

---

## 1. Purpose

This document defines how conceptual data domains — representing business ownership of data — translate into practical governance enforcement across source systems (e.g. SAP ECC, Maximo, ESRI GIS) and the Enterprise Data & Analytics Platform (EDAP). It establishes the governance architecture that ensures consistency between the policy decisions made at the domain level and the technical controls enforced at the system level.

---

## 2. The Core Problem

Data domains describe *who in the business is accountable* for data. A domain like "Asset" establishes that an executive owner and their stewards are responsible for the meaning, quality, classification, and appropriate use of asset data — regardless of where that data physically resides.

But asset data doesn't live in one place. It exists as functional locations and equipment records in SAP ECC, as work orders and asset hierarchies in Maximo, as spatial features in ESRI GIS, as documents in SharePoint/Nexus, and as tables across multiple zones of the EDAP medallion architecture. Each system has its own governance mechanism — SAP has authorisation objects, Maximo has security groups, GIS has feature service permissions, and the EDAP has Unity Catalog with governed tags and attribute-based access control (ABAC).

None of these systems natively understand the concept of a "data domain."

The governance architecture must bridge this gap: ensuring that domain-level policy decisions are faithfully and consistently enforced across every system that holds data within that domain's scope.

---

## 3. Governance Principles

The following principles underpin the governance architecture. They draw on federated computational governance concepts from data mesh thinking, adapted for a centralised platform delivery model operating within a regulated utility context.

### 3.1 Central Policy, Local Enforcement

A central governance function defines the classification standards, access policies, quality expectations, and regulatory mappings. Domain teams apply and enforce these standards locally within their systems and data products. The platform (EDAP) automates enforcement wherever possible.

### 3.2 Domain Ownership Is Accountability, Not System Administration

Domain owners are accountable for the *meaning, quality, classification, and appropriate use* of data within their domain. They are **not** responsible for system-level administration (e.g. SAP authorisation objects, Unity Catalog GRANT statements). Technical teams implement the controls; domain stewards validate that the controls correctly reflect domain policy.

### 3.3 Governance Follows the Data, Not the System

When data moves from a source system into the EDAP, governance responsibility does not transfer — it extends. The source system remains the governance authority for operational data within its boundaries. The EDAP inherits and enriches that governance with domain-level classification, lineage, and access policy.

### 3.4 Open by Default, Restricted by Exception

Data is presumed accessible unless a specific classification, regulation, or business requirement mandates restriction. Restrictions are expressed as governed tags and ABAC policies, making the exception — not the access — the thing that requires justification and governance action.

### 3.5 Automate Enforcement, Reserve Human Judgement for Policy

Classification tagging, access policy inheritance, lineage tracking, and quality monitoring should be automated through platform capabilities (governed tags, ABAC, data classification, DQ monitoring). Human governance effort is reserved for policy decisions, exception management, and stewardship review — activities that require business context and judgement.

---

## 4. Three-Layer Governance Architecture

Governance operates at three distinct layers. The domain model is the connective tissue that ties them together — not a replacement for system-level governance.

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│                    LAYER 1: DOMAIN GOVERNANCE                        │
│                      (Policy & Accountability)                       │
│                                                                      │
│   Domain owners and stewards define:                                 │
│   • Data classification (WAICP, SOCI, PRIS)                         │
│   • Access philosophy and exception criteria                         │
│   • Quality expectations and SLAs                                    │
│   • Business definitions and semantic standards                      │
│   • Regulatory obligation mapping                                    │
│                                                                      │
│   Produces: Policies, standards, classification decisions            │
│   Does NOT produce: Technical controls directly                      │
│                                                                      │
└───────────────────────────────┬──────────────────────────────────────┘
                                │
                    Policy translation & validation
                                │
         ┌──────────────────────┼──────────────────────┐
         │                      │                      │
         ▼                      ▼                      ▼
┌─────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐
│                 │  │                  │  │                          │
│   LAYER 2:      │  │   LAYER 2:       │  │   LAYER 3:               │
│   SOURCE SYSTEM │  │   SOURCE SYSTEM  │  │   EDAP GOVERNANCE        │
│   GOVERNANCE    │  │   GOVERNANCE     │  │   (Platform Enforcement)  │
│   (SAP ECC)     │  │   (Maximo)       │  │                          │
│                 │  │                  │  │   Unity Catalog enforces: │
│   Enforces via: │  │   Enforces via:  │  │   • Governed tags         │
│   • Auth objects│  │   • Security     │  │   • ABAC policies         │
│   • Roles       │  │     groups       │  │   • Row/column security   │
│   • Transaction │  │   • Site-level   │  │   • Data classification   │
│     controls    │  │     restrictions │  │   • Lineage tracking      │
│                 │  │                  │  │   • Metric views          │
│                 │  │                  │  │   • Discover marketplace  │
│                 │  │                  │  │                          │
└─────────────────┘  └──────────────────┘  └──────────────────────────┘
```

### 4.1 Layer 1 — Domain Governance (Policy & Accountability)

**What it is**: The organisational model where data domains, each with executive ownership, define the policies and standards that govern their data.

**What it produces**: Classification decisions, access policies, quality expectations, business definitions, regulatory mappings, and exception criteria.

**What it does NOT do**: Log into systems to grant access, configure technical controls, or administer platform settings.

**How it operates**:

Each domain has the following governance roles:

| Role | Responsibility | Scope |
|------|---------------|-------|
| **Domain Owner** (Executive) | Accountable for domain data strategy, classification policy, and regulatory compliance posture | All data within the domain, across all systems |
| **Data Steward(s)** | Operationally responsible for data quality, definition accuracy, classification application, and validating that technical controls reflect domain policy | Domain data within assigned systems or platform zones |
| **Technical Custodian(s)** | Implements and maintains technical controls (access policies, tags, quality rules) as directed by stewards | System-specific or platform-specific |

Domain governance decisions flow through a defined process:

1. **Policy definition**: Domain owner (with steward input) establishes or updates a classification, access, or quality policy — e.g. "All data containing customer water usage patterns is classified OFFICIAL: SENSITIVE — PERSONAL under PRIS Act 2024."
2. **Impact assessment**: Stewards identify which systems and EDAP zones hold data affected by the policy.
3. **Control specification**: Stewards, working with technical custodians, specify the technical controls required in each system — e.g. governed tag `pris_classification = personal`, ABAC column masking policy on relevant columns, SAP authorisation role restriction.
4. **Implementation**: Technical custodians implement the controls in each system.
5. **Validation**: Stewards validate that the controls correctly reflect the policy intent.
6. **Ongoing monitoring**: Quality monitoring, audit logs, and governance dashboards provide continuous assurance.

### 4.2 Layer 2 — Source System Governance (Operational Enforcement)

**What it is**: The system-native access controls and data management practices within each operational system.

**Key principle**: The source system remains the governance authority for its operational data. SAP ECC owns the access control decisions for who can create or modify a functional location *within SAP*. That is an operational system administration decision, not a domain governance decision. The domain governance layer cares about the *meaning* and *classification* of that data, not who has SAP transaction access.

**How domain policy translates to source system controls**:

| Domain Policy Decision | SAP ECC Translation | Maximo Translation | ESRI GIS Translation |
|---|---|---|---|
| "SOCI-critical asset data is restricted to authorised personnel" | Authorisation objects restrict relevant transaction codes; role assignments limit access to critical asset master records | Security groups restrict access to SOCI-tagged asset records; site-level restrictions enforce geographic boundaries | Feature service permissions restrict layers containing critical infrastructure spatial data |
| "Customer personal data must be masked for non-authorised users" | Field-level authorisation on sensitive customer master fields | Security group restrictions on customer-linked work order fields | N/A (customer data not typically in GIS) |

**Governance boundaries**:

- Source system administrators own the *mechanism* (how controls are technically implemented)
- Domain stewards own the *intent* (what the controls should achieve)
- Changes to source system controls that affect domain-classified data require steward validation
- Source system governance operates independently for operational concerns (e.g. who can post a goods receipt in SAP) that don't intersect with domain classification policy

### 4.3 Layer 3 — EDAP Governance (Platform Enforcement)

**What it is**: The governance layer within the Enterprise Data & Analytics Platform, enforced through Unity Catalog, that becomes progressively more domain-aligned as data moves through the medallion architecture.

**Key principle**: Governance becomes progressively more domain-aligned as data moves through the medallion zones. Source system governance is system-centric. Landing/Raw governance is platform-centric. Gold/BI governance is domain-centric.

#### Governance by Medallion Zone

**Landing & Raw Zones** — *Platform-centric governance*

- Ingestion service accounts have write access; data retains source system classification
- Access restricted to the data engineering team
- Governance focus: integrity, completeness, audit trail (`edap_`-prefixed audit columns)
- Domain governance is minimal here — the data hasn't been transformed into domain-aligned structures

**Base / Protected Zones** — *Transitional governance*

- Data is cleaned, conformed, and structured into domain-aligned schemas
- Governed tags are applied reflecting domain classification decisions:
  - `data_domain` = `asset`
  - `waicp_classification` = `official_sensitive_operational`
  - `soci_critical` = `true`
  - `pris_classification` = `personal`
- Automated Data Classification detects and tags PI, PHI, and other sensitive data patterns
- Steward-driven tagging supplements automated classification for business-context categories (e.g. SOCI critical infrastructure designation)
- ABAC policies begin to take effect, enforcing row-level and column-level controls based on governed tags
- Change detection (SHA-512 hash-based), SCD Type 2 tracking, and DQ annotation operate here

**Enriched & Gold Zones** — *Domain-centric governance*

- The domain model is fully expressed: domain-aligned schemas, governed business definitions, curated data products
- **Governed Tags + ABAC** enforce domain owner access decisions at scale — policies defined at catalog level cascade through schemas and tables automatically
- **Metric Views** (Unity Catalog Business Semantics) provide governed, certified business definitions — the enterprise-authoritative meaning of "Asset Condition Score", "Customer Churn Rate", etc.
- **Discover Marketplace** surfaces certified data products organised by business domain, with ownership, documentation, quality signals, and usage insights
- **External Lineage** traces data origin back to source systems (SAP, Maximo, etc.) and forward to consumption tools (Power BI, Tableau)
- **Data Quality Monitoring** surfaces quality issues and attaches quality signals to discoverable assets
- **AI-Generated Documentation** reduces the manual documentation burden, with stewards reviewing and certifying AI-generated descriptions

**Sandbox / Exploratory Zones** — *Controlled flexibility*

- Domain-level access policies inherited from Gold zone via governed tags
- Additional access may be granted for exploration purposes, subject to classification constraints
- Data created in sandbox zones is not governed as authoritative — it requires promotion through a data product certification workflow to enter Gold

---

## 5. Governance Granularity: From Conceptual Objects to Physical Enforcement

Domain governance operates at the level of **conceptual business objects** — "Customer Personal Information", "SOCI-Critical Asset Location", "Water Quality Test Result". But these concepts don't map cleanly to system-level containers. A governed concept may manifest as an entire table, a subset of columns within a table, a subset of rows, or even specific cell values depending on context.

This granularity mismatch is the single most common point of failure in governance models that look clean on slides but break in reality.

### 5.1 Three Levels of Granularity

| Granularity | Description | Example |
|---|---|---|
| **Row-level** | The same table contains both governed and ungoverned rows. Classification depends on a row attribute. | `dim_functional_location` contains both SOCI-critical and non-critical locations. In SAP ECC, the IFLOT table contains all functional locations — only a subset are SOCI-designated. |
| **Column-level** | The same table contains both governed and ungoverned columns. Some attributes are sensitive, others are not. | A customer table has `customer_name`, `phone_number` (PRIS-classified personal data) alongside `customer_segment`, `region` (non-sensitive). In SAP, the business partner table (BUT000) has personal name fields alongside business classification fields. |
| **Attribute-value** | The governance classification depends on the *value* of a particular attribute, not just its existence. | An asset condition score is unclassified for most assets but SOCI-sensitive when the asset is critical infrastructure. A water quality test result is routine operational data unless it relates to a contamination incident. |

### 5.2 The Governed Data Element

The foundational unit of governance is not a table, schema, or database — it is a **governed data element**: a conceptual business attribute that has a classification, a domain owner, and a set of handling rules regardless of where it physically resides.

Examples of governed data elements:

| Governed Data Element | Domain | Classification | Physical Manifestation |
|---|---|---|---|
| Customer Name | Customer | OFFICIAL: SENSITIVE — PERSONAL (PRIS) | SAP BP master `BUT000.NAME_FIRST/LAST` → EDAP `base.customer.customer_name` → Gold `dim_customer.customer_name` |
| Asset SOCI Criticality Flag | Asset | OFFICIAL: SENSITIVE — OPERATIONAL (SOCI) | SAP classification `AUSP` table values → Maximo `ASSET.SOCI_CRITICAL` → EDAP `dim_functional_location.soci_critical_flag` |
| Water Quality Contamination Indicator | Operations | OFFICIAL: SENSITIVE — OPERATIONAL (SOCI) | Lab system `RESULTS.CONTAMINATION_FLAG` → EDAP `fact_water_quality.contamination_incident` |
| Employee Health Record | People | OFFICIAL: SENSITIVE — PERSONAL (PRIS) | HR system personnel records → EDAP `dim_employee.health_indicator` (if ingested) |

The steward's job is to maintain the register of governed data elements, their classifications, and the mapping of each element to its physical manifestation in each system. This register is the authoritative source for *what needs protecting and at what granularity*.

### 5.3 Enforcement by Granularity in the EDAP

Unity Catalog's governed tags and ABAC operate at all three granularity levels:

**Column-level enforcement**: Governed tags are applied directly to columns, not just tables. When automated Data Classification detects that `customer_name`, `phone_number`, and `email_address` are PI, it tags those specific columns with `sensitivity = pi`. An ABAC column mask policy then enforces: *"for any column tagged `sensitivity = pi`, mask the value unless the requesting user is in the `pi_authorised` group."* The policy is defined once at the catalog level and automatically applies to every PI-tagged column in every table across every schema — current and future. Non-sensitive columns in the same table remain fully accessible.

```sql
-- Example: ABAC column mask policy for PI
-- Defined once at catalog level, inherited by all tables
-- Triggers on any column tagged sensitivity = pi

CREATE FUNCTION mask_pi(value STRING)
RETURNS STRING
RETURN CASE
  WHEN is_member('pi_authorised') THEN value
  ELSE '********'
END;

-- Policy applied via UI or API:
-- Scope: prod_gold catalog, all schemas
-- Match columns: tag sensitivity = pi
-- Action: mask using mask_pi function
-- Applies to: All account users
-- Except: pi_authorised group
```

**Row-level enforcement**: Governed tags are applied to the *table* (indicating it contains governed rows), and ABAC row filter policies use data values within the row to determine visibility. Everyone sees non-restricted rows; restricted rows are filtered for unauthorised users.

```sql
-- Example: ABAC row filter for SOCI-critical assets
-- Table dim_functional_location tagged: soci_relevant = true

CREATE FUNCTION filter_soci(soci_critical_flag BOOLEAN)
RETURNS BOOLEAN
RETURN CASE
  WHEN soci_critical_flag = FALSE THEN TRUE        -- Non-critical: visible to all
  WHEN is_member('soci_authorised') THEN TRUE       -- Critical: visible to authorised
  ELSE FALSE                                         -- Critical: hidden from others
END;

-- Policy applied via UI or API:
-- Scope: prod_gold catalog, all schemas
-- Match tables: tag soci_relevant = true
-- Match columns: tag soci_filter_column = true
-- Action: filter using filter_soci function
```

**Attribute-value enforcement**: Combine both mechanisms. The table is tagged to trigger the policy; the UDF logic handles value-dependent enforcement. For the water quality example, a row filter restricts access to contamination incident rows, while a column mask applies additional masking to `contamination_details` for users who have general access but not incident-response clearance.

### 5.4 The Highest Watermark Principle

A table's *discovery-level classification* reflects the highest sensitivity of any data element it contains. This is the **highest watermark** — it governs how the table appears in Discover, how it is represented in audit queries, and which steward approval is required for access requests.

```
┌──────────────────────────────────────────────────────┐
│  Table: dim_functional_location                       │
│                                                       │
│  Table-level tag (highest watermark):                 │
│    waicp_classification = official_sensitive_operational│
│    soci_relevant = true                               │
│                                                       │
│  Column-level tags:                                   │
│    location_id           → (no sensitivity tag)       │
│    location_description  → (no sensitivity tag)       │
│    parent_location       → (no sensitivity tag)       │
│    soci_critical_flag    → soci_filter_column = true  │
│    gps_latitude          → soci_sensitive = true      │
│    gps_longitude         → soci_sensitive = true      │
│    security_zone_detail  → soci_sensitive = true      │
│                                                       │
│  Enforcement:                                         │
│    Row filter: hides SOCI-critical rows from          │
│                unauthorised users                     │
│    Column mask: masks GPS coordinates and security    │
│                 zone details for SOCI-critical rows   │
│    Discovery: table appears as SENSITIVE in Discover  │
│    Access: unrestricted for non-SOCI rows/columns    │
│            restricted for SOCI elements only          │
└──────────────────────────────────────────────────────┘
```

The critical point: **enforcement operates at the column and row level, but discovery and audit operate at the table level**. This prevents over-restriction (users can still access non-sensitive portions of the table) while ensuring that audit, compliance, and discovery correctly identify all assets containing sensitive data.

### 5.5 Source System Equivalence

Each source system uses its native mechanism to handle the same granularity challenge:

| Granularity | SAP ECC | Maximo | EDAP (Unity Catalog) |
|---|---|---|---|
| **Row-level** | Organisational-level authorisation objects (e.g. plant, company code) restrict which rows a user can see | Site-level and org-level security groups filter record visibility | ABAC row filter policies using governed tags + UDFs |
| **Column-level** | Field-level authorisation objects; field status groups control field visibility per transaction | Field-level security configuration per security group | ABAC column mask policies using governed tags + UDFs; automated Data Classification |
| **Attribute-value** | Custom authorisation checks in ABAP programs; classification system-based restrictions | Conditional expressions in security group definitions; status-based restrictions | ABAC policies combining row filters and column masks with value-evaluation UDFs |

The domain steward does not need to understand the implementation details of each system's mechanism. They need to ensure that the **governed data element register** correctly specifies:
1. What the element is (conceptual definition)
2. How it is classified (WAICP, SOCI, PRIS)
3. Where it physically exists (system, table, column/row identifier)
4. What handling rules apply (who can access, under what conditions, what masking is required)

Technical custodians in each system then implement the appropriate controls using their system's native mechanisms, and stewards validate the result.

---

## 6. The Governance Taxonomy: Tags as the Universal Language

Governed tags are the mechanism that makes domain governance enforceable at the platform level. They provide a universal taxonomy that bridges domain policy with technical enforcement.

### 6.1 Tag Categories

| Tag Category | Purpose | Examples | Enforcement Mechanism |
|---|---|---|---|
| **Domain** | Identifies business ownership | `data_domain = asset` | Discovery, stewardship routing, cost attribution |
| **Regulatory Classification** | Maps regulatory obligations to assets | `waicp_classification = official_sensitive_personal` | ABAC column masking, row filtering |
| **Critical Infrastructure** | SOCI Act designation | `soci_critical = true` | ABAC access restriction, enhanced audit logging |
| **Privacy** | PRIS Act classification | `pris_classification = personal` | ABAC column masking, anonymisation in non-prod (per ADR-EDP-001) |
| **Consent / Lawful Basis** | Records the lawful basis for processing personal information | `pi_lawful_basis = legitimate_interest`, `pi_lawful_basis = consent` | Stewardship review, regulatory audit, AI training approval gate |
| **Sensitivity** | Auto-detected data sensitivity | `sensitivity = pi`, `sensitivity = phi` | Automated Data Classification → ABAC masking |
| **Quality Tier** | Steward-certified quality level | `quality_tier = certified`, `quality_tier = provisional` | Discover marketplace ranking, consumer trust signals |
| **Data Product** | Data product membership | `data_product = asset_condition_analytics` | Discover marketplace organisation, access request bundling |

### 6.2 Tag Governance Process

1. **Tag policies** are defined at the account level by the central governance function, establishing the controlled vocabulary (allowed tag keys and permitted values)
2. **Automated classification** applies sensitivity tags (PI, PHI) without human intervention
3. **Steward-driven tagging** applies domain, regulatory, and quality tags based on domain policy decisions
4. **ABAC policies** reference governed tags to enforce access controls — defined once at the catalog or schema level, automatically inherited by all current and future tables
5. **Consent and lawful basis tagging** records the legal basis for processing personal information against PI-classified assets, supporting regulatory audit and gating AI training approvals
6. **Audit and monitoring** via system tables and Governance Insights dashboard track tag application, policy enforcement, and access patterns

---

## 7. Cross-System Consistency: Keeping Governance Aligned

The hardest governance challenge is ensuring that a domain policy decision propagates consistently across all systems that hold data within that domain's scope.

### 7.1 EDAP as Governance Hub

For the current maturity stage, the EDAP (via Unity Catalog) serves as the **governance hub** — the system of record for domain classification taxonomy and the primary enforcement point for analytics governance:

```
                    ┌──────────────────────────┐
                    │     Domain Governance     │
                    │    (Policy Decisions)     │
                    └────────────┬─────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │   Unity Catalog (EDAP)    │
                    │                          │
                    │  • Governed Tags          │
                    │    (classification        │
                    │     taxonomy)             │
                    │  • ABAC Policies          │
                    │  • Metric Views           │
                    │  • Discover Marketplace   │
                    │  • External Lineage       │
                    │                          │
                    │  System of record for:    │
                    │  domain classification,   │
                    │  business definitions,    │
                    │  analytics access policy  │
                    │                          │
                    └──┬────────────┬───────┬──┘
                       │            │       │
            ┌──────────▼──┐  ┌─────▼────┐ ┌▼──────────┐
            │  SAP ECC    │  │ Maximo   │ │ ESRI GIS  │
            │             │  │          │ │           │
            │ Operational │  │Operational│ │Operational│
            │ governance  │  │governance │ │governance │
            │ (native     │  │(native   │ │(native    │
            │  controls)  │  │ controls)│ │ controls) │
            │             │  │          │ │           │
            │ Stewards    │  │Stewards  │ │Stewards   │
            │ validate    │  │validate  │ │validate   │
            │ alignment   │  │alignment │ │alignment  │
            └─────────────┘  └──────────┘ └───────────┘
```

**How it works**:

- Domain classification decisions are recorded as governed tags in Unity Catalog
- ABAC policies enforce those classifications within the EDAP automatically
- **Alation** serves as the enterprise data discovery and cataloguing tool, providing business user-facing search, data dictionary management, and stewardship workflow capabilities that complement Unity Catalog's platform-level enforcement. Unity Catalog Discover provides domain-organised discovery within the EDAP; Alation provides broader enterprise catalogue coverage including source systems and non-EDAP data assets. Where both tools catalogue the same asset, Unity Catalog is authoritative for classification and access enforcement, and Alation is authoritative for business context, usage guidance, and stewardship workflows
- Stewards maintain a **cross-system alignment register** (initially in Confluence, with Alation providing catalogue-level cross-referencing as adoption matures) that maps each domain classification decision to its corresponding source system controls
- Periodic stewardship reviews validate that source system controls remain aligned with the EDAP-authoritative classification taxonomy

### 7.2 When to Introduce an Enterprise Catalogue Layer

The EDAP-as-governance-hub pattern is appropriate when:

- The EDAP is the primary analytics and reporting platform
- Most data consumption for decision-making happens through the EDAP
- The source system estate is manageable in scope (fewer than ~10 major systems)
- Regulatory requirements can be met through a combination of Unity Catalog enforcement + documented stewardship processes

Consider introducing a dedicated enterprise governance catalogue layer (e.g. Microsoft Purview Unified Catalog, Collibra, Atlan) when:

- Regulatory audit requirements demand a *single, unified view* of data access and classification across all systems — not just the EDAP
- The source system estate grows to a scale where manual cross-system alignment registers become unsustainable
- Formal stewardship workflows with multi-step approval chains, escalation paths, and SLA tracking are required by regulation or organisational policy
- Cross-platform data product discovery is needed (e.g. business users need to discover data in SAP, Maximo, GIS, and EDAP from a single interface)

**Note:** Alation already provides enterprise catalogue capabilities including business glossary, data dictionary, and stewardship workflow. The decision to introduce an additional governance-focused catalogue (Purview, Collibra) should be evaluated against extending Alation's current role before introducing a new tool.

### 7.3 Lakehouse Federation

Unity Catalog supports **federated queries** across external data sources (PostgreSQL, MySQL, SQL Server, and other JDBC-connected systems). This capability is directly relevant to the governance model:

- Federated queries allow the EDAP to query source system data **in place** without ingestion, which is useful for ad-hoc governance validation, cross-referencing source system state against EDAP-held data, and supporting use cases where data should not be copied (e.g. highly sensitive operational data that must remain in the source system for regulatory reasons).
- Federated connections are governed through Unity Catalog — the same ABAC policies, governed tags, and audit logging apply to federated queries as to native EDAP tables.
- Federation does **not** replace ingestion for analytics workloads. It complements the medallion architecture by providing a governed read path to source systems for validation, reconciliation, and edge-case access patterns.
- Domain stewards should be aware that federated queries against source systems are subject to both EDAP governance (Unity Catalog ABAC) and source system governance (native access controls). Both layers must be satisfied.

### 7.4 Delta Sharing for Cross-Organisation Data Exchange

Water Corporation shares data with external parties — regulators (ERA, DWER), government agencies, research partners, and service providers. **Delta Sharing** (Databricks' open protocol for secure, zero-copy data sharing) provides the mechanism for governed cross-organisation data exchange:

- **Zero-copy sharing**: Recipients access data in place without copying it to their own storage. This means Water Corporation retains control — access can be revoked, audited, and scoped to specific tables, partitions, or time windows.
- **Governance enforcement**: Shared data products are governed by the same ABAC policies and governed tags as internal access. Sharing is additive — it grants external access to data that already has classification and access controls applied. Sensitive columns (PI, SOCI-critical) can be excluded or masked in shared views.
- **Recipient controls**: Shares are scoped to named recipients with time-limited tokens. The domain owner approves each sharing relationship; the data product owner defines the scope and SLA of the share.
- **Audit trail**: All recipient access is logged in Unity Catalog system tables, providing the audit evidence required for SOCI Act and PRIS Act compliance.
- **Regulatory sharing**: For mandatory regulatory data submissions (e.g. ERA performance reporting, DWER environmental compliance), Delta Sharing provides a governed alternative to file-based extracts, reducing the risk of uncontrolled data copies and ensuring the regulator always accesses the most current, governed version.

**Governance process for external sharing:**

1. The **domain owner** approves the sharing relationship and classifies the data sensitivity of the share.
2. The **data product owner** defines the share scope — which tables, columns, rows, and freshness are included.
3. The **DPO** reviews any share that includes PI or SOCI-classified data.
4. The **technical data steward** configures the share in Unity Catalog with appropriate ABAC policies and exclusions.
5. The **platform team** provisions the recipient and manages token lifecycle.
6. The **domain steward** periodically reviews active shares for continued appropriateness.

---

## 8. AI Governance in the Federated Model

As AI and machine learning capabilities mature within the EDAP, the three-layer governance model must extend to cover AI-specific governance concerns. AI introduces new risks around data usage rights, model output ownership, and cross-domain data combination that the existing classification model does not fully address.

This section aligns with **ISO/IEC 42001:2023** (AI Management Systems), the **NIST AI Risk Management Framework (AI RMF 1.0)**, and the **Australian Government Voluntary AI Safety Standard** (2024). As the Voluntary AI Safety Standard may transition to mandatory requirements, the governance controls described here are designed to satisfy both current voluntary and anticipated mandatory obligations.

### 8.1 Approval for AI Training Use

The domain owner retains authority over whether domain data may be used for AI model training. This is governed through the `ai_training_permitted` and `ai_use_restriction` tags defined in the EDAP Tagging Strategy. Specifically:

- The **domain owner** must approve any use of their domain's data for model training, whether for internal models or external AI services.
- Approval is recorded as a governed tag (`ai_training_permitted = true`) and logged in the governance register.
- Data classified under the PRIS Act 2024 as personal information must not be used for AI training without a documented lawful basis, regardless of domain owner approval.
- SOCI-critical data requires additional review by the security and compliance team before AI training approval is granted.

### 8.2 Cross-Domain Model Outputs

When an AI model consumes data from multiple domains, the governance of the model's output follows these principles:

- The model output is treated as a **new data product** and must have an assigned domain owner and data product owner.
- The output inherits the **highest watermark classification** of all contributing input domains.
- All contributing domain owners must be consulted before the model output is published as a data product. Where any contributing domain owner objects, the matter is escalated to the Data Governance Council.
- Lineage must trace model outputs back to contributing source domains, enabling impact analysis when source data classifications change.

### 8.3 AI-Generated Data Products

AI-generated data products (predictions, scores, recommendations, synthetic datasets) fit into the existing domain ownership model as follows:

- AI-generated outputs are assigned to the **consuming domain** that commissioned or operates the model, not to the source data domain(s).
- The data product owner is accountable for the accuracy, fairness, and appropriate use of the AI-generated output, including monitoring for model drift and bias.
- Synthetic datasets derived from sensitive data inherit the source data's classification until a formal privacy assessment confirms the synthetic data does not enable re-identification.

### 8.4 AI Agent Governance

AI agents that autonomously access, combine, or act on data within the EDAP are subject to the following governance requirements:

- AI agents must authenticate via dedicated service principals with least-privilege access, scoped to the minimum data required for their function.
- Agent access is governed by the same ABAC policies, governed tags, and row/column security that apply to human users. Agents do not receive blanket access.
- Agent actions that modify data, trigger workflows, or generate outputs consumed by humans must produce an audit trail traceable to the agent's service principal and the authorising domain owner.
- Cross-domain data access by agents follows the same request and approval process as human cross-domain access. Agents acting on behalf of a domain operate within that domain's access permissions.
- High-risk agent actions (accessing SOCI-critical data, processing personal information, generating external-facing outputs) require pre-approval from the relevant domain owner and are subject to enhanced audit logging.

---

## 9. Governance Operating Model

### 9.1 Governance Cadence

| Activity | Frequency | Participants | Purpose |
|---|---|---|---|
| **Domain stewardship review** | Fortnightly | Domain stewards, technical custodians | Review classification changes, validate controls, address quality issues |
| **Cross-domain governance forum** | Monthly | All domain stewards, platform team, Architecture & Strategy | Align on cross-cutting policies, resolve domain boundary disputes, review governance metrics |
| **Regulatory compliance review** | Quarterly | Domain owners, legal/compliance, Architecture & Strategy | Validate that SOCI, PRIS, WAICP, and Essential Eight obligations are met |
| **Governance taxonomy review** | Six-monthly | Domain owners, stewards, platform team | Review and update the governed tag taxonomy, ABAC policies, and classification standards |
| **Architecture Review Board** | As required | ARB members | Approve significant changes to governance architecture, new domain definitions, or new system integrations |

### 9.2 Governance Metrics

| Metric | Measurement Point | Target |
|---|---|---|
| Tag coverage | % of Gold zone tables with complete governed tag set (domain, classification, quality tier) | > 95% |
| ABAC policy coverage | % of SOCI/PRIS-classified assets with active ABAC enforcement | 100% |
| Classification currency | Average age of most recent steward classification review per domain | < 90 days |
| Cross-system alignment | % of domain classification decisions with validated source system controls | > 90% |
| Data product certification | % of Discover-published data products with steward certification | > 80% |
| Quality signal coverage | % of Gold zone tables with active DQ monitoring | > 90% |

### 9.3 Data Observability

Governance metrics (section 9.2) measure governance programme health at a point in time. **Data observability** complements this with continuous, automated monitoring of the data itself — detecting issues proactively rather than waiting for downstream failures or stewardship reviews.

Data observability covers five pillars, each monitored through Lakehouse Monitoring and DQ expectations within the EDAP:

| Pillar | What It Monitors | Governance Implication |
|---|---|---|
| **Freshness** | Has the data been updated within the expected SLA window? | Stale data in Gold zone data products may indicate pipeline failure or source system issues. Freshness alerts trigger the quality accountability model (section 9.4). |
| **Volume** | Has the row count changed beyond expected bounds? | Unexpected volume spikes or drops may indicate source system changes, data loss, or ingestion failures. |
| **Schema** | Have columns been added, removed, or changed type? | Schema changes propagate through the medallion architecture and may break data contracts. Schema drift alerts trigger impact assessment by the technical data steward. |
| **Distribution** | Have the statistical properties of column values shifted? | Distribution drift in key business columns may indicate data quality degradation, source system changes, or — for AI/ML use cases — training data drift that affects model performance. |
| **Lineage** | Are all expected upstream dependencies producing data? | Broken lineage (a source table that stops updating) is detected before it manifests as a quality issue in downstream data products. |

**How observability integrates with governance:**

- Observability alerts are routed to the accountable role defined in the quality accountability model (section 9.4) — pipeline owner for ingestion issues, domain steward for transformation issues, data product owner for product-level issues.
- Freshness and volume alerts for SOCI-critical or PRIS-classified data products are escalated to the domain owner and, where applicable, the DPO.
- Observability signals are surfaced alongside data products in Discover and Alation, giving consumers visibility into the current health of data they depend on.

### 9.4 Incident Response Governance

Data governance incidents require a clear command structure that operates across the three-layer model. The following incident types are in scope:

| Incident Type | Description | Mandatory Reporting |
|---|---|---|
| **PI exposure** | Unauthorised access to or disclosure of personal information | PRIS Act 2024 breach notification obligations; Privacy Commissioner notification where applicable |
| **SOCI-classified data leak** | Unauthorised access to or disclosure of data classified under the SOCI Act 2018 | SOCI Act mandatory reporting to the Cyber and Infrastructure Security Centre (CISC) within timeframes prescribed by the Act |
| **Data quality incident** | A material data quality failure that affects downstream decision-making, reporting, or regulatory compliance | Internal escalation; regulatory reporting where the quality failure affects compliance obligations |

**Incident command model:**

- The **Data Protection Officer (DPO)** is the incident commander for PI exposure and SOCI-classified data leak incidents. The DPO coordinates response across all three governance layers, engages legal and compliance, and manages mandatory reporting obligations.
- The **domain owner** is the incident commander for data quality incidents within their domain. Where a quality incident spans multiple domains, the Data Governance Council nominates a lead domain owner.
- During an incident, the three-layer model operates as follows:
  - **Layer 1 (Domain Governance):** The domain owner authorises emergency access changes, temporary data restrictions, or communication to affected stakeholders.
  - **Layer 2 (Source System Governance):** Source system administrators implement emergency access revocations or restrictions as directed by the incident commander.
  - **Layer 3 (EDAP Governance):** The platform team implements emergency Unity Catalog access revocations, tag changes, or data quarantine actions as directed by the incident commander.

**SOCI Act mandatory reporting:** As a critical infrastructure entity under the SOCI Act 2018, Water Corporation has mandatory reporting obligations for cyber security incidents affecting critical infrastructure assets. The DPO, in coordination with the CISO, must ensure that incidents involving SOCI-classified data are reported to the CISC within the prescribed timeframes (currently 12 hours for critical incidents, 72 hours for other reportable incidents). The 2024 SOCI Rules amendments expanded positive security obligations for critical infrastructure entities, including enhanced risk management programme requirements and broader definitions of reportable incidents — the incident response governance described here must be reviewed against these expanded obligations. The governance register must record all incident response actions for audit purposes.

### 9.5 Quality Accountability Model

Data quality accountability follows the data through each transition point from source to consumption:

| Transition Point | Accountable Role | Quality Responsibility |
|---|---|---|
| **Source system** | Source system owner | Accuracy, completeness, and timeliness of data at the point of origin. Responsible for remediating quality issues identified by downstream stewards. |
| **Ingestion (Landing/Raw)** | Ingestion pipeline owner (data engineering) | Faithful capture of source data without loss or corruption. Completeness checks (record counts, schema validation). Not accountable for source data quality. |
| **Raw to Base transformation** | Domain data steward | Validation that transformation logic correctly maps source data to domain-aligned structures. DQ rule definition and threshold setting. |
| **Data product (Gold)** | Data product owner | End-to-end quality of the published data product against its data contract. Monitoring, alerting, and remediation coordination. Accountable to consumers for meeting SLA commitments. |

Quality issues discovered downstream are traced back to the earliest responsible transition point for remediation. The domain steward coordinates cross-layer quality investigations and escalates to the domain owner where systemic issues require source system intervention.

---

## 10. Worked Example: SOCI-Critical Asset Data

To illustrate how the three-layer model works in practice:

**Domain policy decision** (Layer 1):
The Asset domain owner, in consultation with the SOCI Act compliance team, determines that all data relating to water treatment plant assets designated as critical infrastructure under SOCI Act 2018 must be classified `OFFICIAL: SENSITIVE — OPERATIONAL` and restricted to personnel with a verified operational need.

**Source system enforcement** (Layer 2):

| System | Control Implementation |
|---|---|
| SAP ECC | Authorisation roles restrict access to equipment master records for SOCI-designated functional locations. Asset master change documents are retained per SOCI audit requirements. |
| Maximo | Security groups restrict access to work orders and condition data for SOCI-tagged assets. Site-level restrictions enforce geographic boundaries. |
| ESRI GIS | Feature service permissions restrict access to spatial layers containing critical infrastructure locations. Public-facing map services exclude SOCI-designated features. |

**EDAP enforcement** (Layer 3):

| Zone | Control Implementation |
|---|---|
| Landing / Raw | Ingested with source audit metadata. Access restricted to data engineering service accounts. |
| Base / Protected | Governed tags applied: `data_domain = asset`, `soci_critical = true`, `waicp_classification = official_sensitive_operational`. ABAC row filter policy activated: only users in the `soci_authorised` group can access rows where `soci_critical = true`. |
| Gold / BI | Data product "Critical Infrastructure Asset Analytics" published to Discover marketplace with domain ownership, steward certification, quality signals, and SOCI access restriction. Metric Views define governed calculations for critical asset condition scoring. External lineage traces data provenance to SAP ECC and Maximo source records. |

**Steward validation**:
The Asset data steward validates that ABAC policies in the EDAP, SAP authorisation roles, Maximo security groups, and GIS feature service permissions all consistently enforce the domain owner's access restriction. This validation is recorded in the cross-system alignment register and reviewed at the fortnightly stewardship review.

---

## 11. Relationship to Enterprise Architecture

This governance model operates within the broader enterprise architecture context:

| Architectural Concern | Governance Relationship |
|---|---|
| **EDAP Medallion Architecture** | Governance progressively strengthens from Landing (platform-centric) through to Gold (domain-centric) |
| **Unity Catalog Namespace Design** | Catalog-per-environment-layer pattern (e.g. `prod_gold`, `dev_silver`) provides the structural boundaries for ABAC policy inheritance |
| **Data Governance Tagging Framework** | Governed tags implement the WAICP-aligned classification taxonomy; ABAC policies reference these tags for enforcement |
| **SDP Framework (EDAP-FWK-001)** | Raw-to-Base processing applies initial governed tags and DQ annotations as part of the declarative pipeline specification |
| **ADR-EDP-001 (Dev Environment Strategy)** | Shallow cloning and data anonymisation for sensitive domains under PRIS Act are enforced through governed tags and ABAC in non-production catalogs |
| **Enterprise Content Management Roadmap** | Document classification under State Records Act 2000 aligns with the governed tag taxonomy for WAICP classification |
| **Enterprise Asset Data Model (EADM)** | Domain entity definitions (Asset Class, Functional Location, Property, etc.) inform the business glossary expressed through Unity Catalog Metric Views and semantic metadata |

---

## 12. Data Contracts

Data contracts are the formal interface between data producers and consumers. They define the commitments a data product makes to its consumers and provide the enforceable agreement that underpins trust in shared data.

### 12.1 Data Contract Structure

A data contract specifies:

| Contract Element | Description |
|---|---|
| **Schema definition** | Guaranteed column structure, data types, and nullable constraints. The contract schema is the stable interface — internal implementation may change provided the contract schema is honoured. |
| **Freshness SLA** | Maximum acceptable latency between source change and data product availability. Tiered by criticality (e.g. `real-time`, `hourly`, `daily`). |
| **Quality thresholds** | Minimum acceptable DQ pass rates and specific rules the producer commits to enforcing (e.g. "customer_id is never null", "order_total is non-negative"). |
| **Ownership** | The data product owner accountable for meeting the contract. |
| **Versioning** | Semantic versioning with a defined deprecation period for breaking changes. Major version increments indicate breaking changes; minor versions indicate backwards-compatible additions. |
| **Breaking change policy** | How breaking changes are communicated and managed — `versioned` (new major version published alongside the old, with a deprecation window), `notify` (consumers notified in advance, no parallel version), or `unmanaged` (no guarantees — appropriate only for sandbox/exploratory data). |

### 12.2 Contract Enforcement

Contracts are enforced through the SDP framework and Unity Catalog:

- `contract_version`, `contract_sla_tier`, and `breaking_change_policy` are recorded as governed tags in Unity Catalog.
- **Schema enforcement**: Pipeline expectations validate that the published schema matches the contract schema on every run. Schema drift triggers an alert to the data product owner and blocks promotion to Gold where the contract specifies `versioned` breaking change policy.
- **Freshness monitoring**: Lakehouse Monitoring tracks data product update timestamps against the contracted SLA tier and alerts the data product owner on SLA breach.
- **Quality monitoring**: DQ expectations defined in the SDP pipeline specification enforce the contracted quality thresholds. Quality failures are surfaced in observability dashboards (section 9.3) and, for certified data products, trigger the quality accountability model (section 9.5).

### 12.3 Contract Governance

- The **data product owner** is accountable for defining and honouring the contract.
- The **domain steward** validates that the contract's quality thresholds and schema align with domain governance requirements.
- **Consumers** can view contract terms through Discover and Alation, and raise disputes through the domain steward when contract commitments are not met.
- Contract changes follow the versioning and breaking change policy defined in the contract itself. The data product owner is responsible for consumer communication and migration support during breaking changes.

---

## 13. Future State Considerations

As the governance model matures, the following evolutionary steps should be evaluated:

1. **Purview integration**: If regulatory audit requirements for SOCI or PRIS Act demand a single cross-system governance view, Microsoft Purview Unified Catalog can provide a federated discovery and compliance layer that scans Unity Catalog alongside SAP, Maximo, and other Azure-connected sources. Purview's governance domains align naturally with the domain model described here. This should be evaluated against extending Alation's existing enterprise catalogue role.

2. **Formal stewardship workflows**: If the stewardship cadence described in section 9.1 proves insufficient for regulatory demands, a dedicated governance workflow engine (Collibra, Atlan, or Alation's stewardship workflow capabilities) can provide workflow orchestration with approval chains, escalation paths, and SLA tracking. This should be a pull-based decision driven by demonstrated need, not a push-based procurement.

3. **Consent management platform**: As Privacy Act reform progresses (particularly around automated decision-making and enhanced consent requirements), Water Corporation may need a dedicated consent management capability that tracks consent records, lawful basis, and purpose limitation at the individual level — integrated with the `pi_lawful_basis` tagging described in section 6.1. This is distinct from classification tagging (which governs data categories) and would provide individual-level consent tracking for regulatory compliance.

4. **Deepening computational governance**: The governance model already employs significant automation — governed tags applied by automated Data Classification, ABAC policies inherited by convention, quality rules enforced in pipelines. The next maturity step is to formalise this as **policy-as-code**: governance policies expressed as versioned, testable, deployable code artefacts (tag assignment rules, ABAC policy definitions, quality threshold configurations) managed through the same CI/CD practices as application code. This shifts the remaining human governance effort from enforcement to exception management and policy evolution.

---

## References

| Reference | Description |
|---|---|
| EDAP-FWK-001 | Spark Declarative Pipeline Framework Specification |
| ADR-EDP-001 | Databricks Development Environment Data Strategy |
| WAICP | WA Information Classification Policy |
| SOCI Act 2018 (Cth) | Security of Critical Infrastructure Act, including 2024 SOCI Rules amendments |
| PRIS Act 2024 (WA) | Privacy and Responsible Information Sharing Act |
| Privacy Act 1988 (Cth) | Australian Privacy Act, including Notifiable Data Breaches scheme (Part IIIC) |
| State Records Act 2000 (WA) | WA State Records Act |
| Essential Eight | ACSC Essential Eight Maturity Model |
| ISO/IEC 42001:2023 | AI Management Systems — requirements and guidance |
| NIST AI RMF 1.0 | AI Risk Management Framework — governance, mapping, measuring, managing AI risk |
| Australian Government (2024) | Voluntary AI Safety Standard |
| Dehghani, Z. (2022) | *Data Mesh: Delivering Data-Driven Value at Scale* — Federated computational governance principles |
| Databricks (2025) | Unity Catalog ABAC, Governed Tags, Business Semantics, Discover, Delta Sharing, and Lakehouse Federation documentation |
| Alation | Enterprise data discovery and cataloguing platform — business glossary, data dictionary, and stewardship workflows |

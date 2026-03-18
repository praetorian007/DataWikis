# Governing Data Across Source Systems and the Enterprise Data Platform

## A Domain-Based Approach to Federated Data Governance

| | |
|---|---|
| **Classification** | OFFICIAL |
| **Owner** | Architecture & Strategy |
| **Status** | Draft |
| **Last Updated** | March 2026 |

---

## 1. Purpose

This document defines how conceptual data domains â representing business ownership of data â translate into practical governance enforcement across source systems (e.g. SAP ECC, Maximo, ESRI GIS) and the Enterprise Data Analytics Platform (EDAP). It establishes the governance architecture that ensures consistency between the policy decisions made at the domain level and the technical controls enforced at the system level.

---

## 2. The Core Problem

Data domains describe *who in the business is accountable* for data. A domain like "Asset Management" establishes that an executive owner and their stewards are responsible for the meaning, quality, classification, and appropriate use of asset data â regardless of where that data physically resides.

But asset data doesn't live in one place. It exists as functional locations and equipment records in SAP ECC, as work orders and asset hierarchies in Maximo, as spatial features in ESRI GIS, as documents in SharePoint/Nexus, and as tables across multiple zones of the EDAP medallion architecture. Each system has its own governance mechanism â SAP has authorisation objects, Maximo has security groups, GIS has feature service permissions, and the EDAP has Unity Catalog with governed tags and attribute-based access control (ABAC).

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

When data moves from a source system into the EDAP, governance responsibility does not transfer â it extends. The source system remains the governance authority for operational data within its boundaries. The EDAP inherits and enriches that governance with domain-level classification, lineage, and access policy.

### 3.4 Open by Default, Restricted by Exception

Data is presumed accessible unless a specific classification, regulation, or business requirement mandates restriction. Restrictions are expressed as governed tags and ABAC policies, making the exception â not the access â the thing that requires justification and governance action.

### 3.5 Automate Enforcement, Reserve Human Judgement for Policy

Classification tagging, access policy inheritance, lineage tracking, and quality monitoring should be automated through platform capabilities (governed tags, ABAC, data classification, DQ monitoring). Human governance effort is reserved for policy decisions, exception management, and stewardship review â activities that require business context and judgement.

---

## 4. Three-Layer Governance Architecture

Governance operates at three distinct layers. The domain model is the connective tissue that ties them together â not a replacement for system-level governance.

```
ââââââââââââââââââââââââââââââââââââââââââââââââââââââââââââââââââââââââ
â                                                                      â
â                    LAYER 1: DOMAIN GOVERNANCE                        â
â                      (Policy & Accountability)                       â
â                                                                      â
â   Domain owners and stewards define:                                 â
â   â¢ Data classification (WAICP, SOCI, PRIS)                         â
â   â¢ Access philosophy and exception criteria                         â
â   â¢ Quality expectations and SLAs                                    â
â   â¢ Business definitions and semantic standards                      â
â   â¢ Regulatory obligation mapping                                    â
â                                                                      â
â   Produces: Policies, standards, classification decisions            â
â   Does NOT produce: Technical controls directly                      â
â                                                                      â
âââââââââââââââââââââââââââââââââ¬âââââââââââââââââââââââââââââââââââââââ
                                â
                    Policy translation & validation
                                â
         ââââââââââââââââââââââââ¼âââââââââââââââââââââââ
         â                      â                      â
         â¼                      â¼                      â¼
âââââââââââââââââââ  ââââââââââââââââââââ  ââââââââââââââââââââââââââââ
â                 â  â                  â  â                          â
â   LAYER 2:      â  â   LAYER 2:       â  â   LAYER 3:               â
â   SOURCE SYSTEM â  â   SOURCE SYSTEM  â  â   EDAP GOVERNANCE        â
â   GOVERNANCE    â  â   GOVERNANCE     â  â   (Platform Enforcement)  â
â   (SAP ECC)     â  â   (Maximo)       â  â                          â
â                 â  â                  â  â   Unity Catalog enforces: â
â   Enforces via: â  â   Enforces via:  â  â   â¢ Governed tags         â
â   â¢ Auth objectsâ  â   â¢ Security     â  â   â¢ ABAC policies         â
â   â¢ Roles       â  â     groups       â  â   â¢ Row/column security   â
â   â¢ Transaction â  â   â¢ Site-level   â  â   â¢ Data classification   â
â     controls    â  â     restrictions â  â   â¢ Lineage tracking      â
â                 â  â                  â  â   â¢ Metric views          â
â                 â  â                  â  â   â¢ Discover marketplace  â
â                 â  â                  â  â                          â
âââââââââââââââââââ  ââââââââââââââââââââ  ââââââââââââââââââââââââââââ
```

### 4.1 Layer 1 â Domain Governance (Policy & Accountability)

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

1. **Policy definition**: Domain owner (with steward input) establishes or updates a classification, access, or quality policy â e.g. "All data containing customer water usage patterns is classified OFFICIAL: SENSITIVE â PERSONAL under PRIS Act 2024."
2. **Impact assessment**: Stewards identify which systems and EDAP zones hold data affected by the policy.
3. **Control specification**: Stewards, working with technical custodians, specify the technical controls required in each system â e.g. governed tag `pris_classification = personal`, ABAC column masking policy on relevant columns, SAP authorisation role restriction.
4. **Implementation**: Technical custodians implement the controls in each system.
5. **Validation**: Stewards validate that the controls correctly reflect the policy intent.
6. **Ongoing monitoring**: Quality monitoring, audit logs, and governance dashboards provide continuous assurance.

### 4.2 Layer 2 â Source System Governance (Operational Enforcement)

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

### 4.3 Layer 3 â EDAP Governance (Platform Enforcement)

**What it is**: The governance layer within the Enterprise Data Analytics Platform, enforced through Unity Catalog, that becomes progressively more domain-aligned as data moves through the medallion architecture.

**Key principle**: Governance becomes progressively more domain-aligned as data moves through the medallion zones. Source system governance is system-centric. Landing/Raw governance is platform-centric. Gold/BI governance is domain-centric.

#### Governance by Medallion Zone

**Landing & Raw Zones** â *Platform-centric governance*

- Ingestion service accounts have write access; data retains source system classification
- Access restricted to the data engineering team
- Governance focus: integrity, completeness, audit trail (`edap_`-prefixed audit columns)
- Domain governance is minimal here â the data hasn't been transformed into domain-aligned structures

**Base / Protected Zones** â *Transitional governance*

- Data is cleaned, conformed, and structured into domain-aligned schemas
- Governed tags are applied reflecting domain classification decisions:
  - `data_domain` = `asset_management`
  - `waicp_classification` = `official_sensitive_operational`
  - `soci_critical` = `true`
  - `pris_classification` = `personal`
- Automated Data Classification detects and tags PII, PHI, and other sensitive data patterns
- Steward-driven tagging supplements automated classification for business-context categories (e.g. SOCI critical infrastructure designation)
- ABAC policies begin to take effect, enforcing row-level and column-level controls based on governed tags
- Change detection (SHA-512 hash-based), SCD Type 2 tracking, and DQ annotation operate here

**Enriched & Gold Zones** â *Domain-centric governance*

- The domain model is fully expressed: domain-aligned schemas, governed business definitions, curated data products
- **Governed Tags + ABAC** enforce domain owner access decisions at scale â policies defined at catalog level cascade through schemas and tables automatically
- **Metric Views** (Unity Catalog Business Semantics) provide governed, certified business definitions â the enterprise-authoritative meaning of "Asset Condition Score", "Customer Churn Rate", etc.
- **Discover Marketplace** surfaces certified data products organised by business domain, with ownership, documentation, quality signals, and usage insights
- **External Lineage** traces data origin back to source systems (SAP, Maximo, etc.) and forward to consumption tools (Power BI, Tableau)
- **Data Quality Monitoring** surfaces quality issues and attaches quality signals to discoverable assets
- **AI-Generated Documentation** reduces the manual documentation burden, with stewards reviewing and certifying AI-generated descriptions

**Sandbox / Exploratory Zones** â *Controlled flexibility*

- Domain-level access policies inherited from Gold zone via governed tags
- Additional access may be granted for exploration purposes, subject to classification constraints
- Data created in sandbox zones is not governed as authoritative â it requires promotion through a data product certification workflow to enter Gold

---

## 5. Governance Granularity: From Conceptual Objects to Physical Enforcement

Domain governance operates at the level of **conceptual business objects** â "Customer Personal Information", "SOCI-Critical Asset Location", "Water Quality Test Result". But these concepts don't map cleanly to system-level containers. A governed concept may manifest as an entire table, a subset of columns within a table, a subset of rows, or even specific cell values depending on context.

This granularity mismatch is the single most common point of failure in governance models that look clean on slides but break in reality.

### 5.1 Three Levels of Granularity

| Granularity | Description | Example |
|---|---|---|
| **Row-level** | The same table contains both governed and ungoverned rows. Classification depends on a row attribute. | `dim_functional_location` contains both SOCI-critical and non-critical locations. In SAP ECC, the IFLOT table contains all functional locations â only a subset are SOCI-designated. |
| **Column-level** | The same table contains both governed and ungoverned columns. Some attributes are sensitive, others are not. | A customer table has `customer_name`, `phone_number` (PRIS-classified personal data) alongside `customer_segment`, `region` (non-sensitive). In SAP, the business partner table (BUT000) has personal name fields alongside business classification fields. |
| **Attribute-value** | The governance classification depends on the *value* of a particular attribute, not just its existence. | An asset condition score is unclassified for most assets but SOCI-sensitive when the asset is critical infrastructure. A water quality test result is routine operational data unless it relates to a contamination incident. |

### 5.2 The Governed Data Element

The foundational unit of governance is not a table, schema, or database â it is a **governed data element**: a conceptual business attribute that has a classification, a domain owner, and a set of handling rules regardless of where it physically resides.

Examples of governed data elements:

| Governed Data Element | Domain | Classification | Physical Manifestation |
|---|---|---|---|
| Customer Name | Customer | OFFICIAL: SENSITIVE â PERSONAL (PRIS) | SAP BP master `BUT000.NAME_FIRST/LAST` â EDAP `base.customer.customer_name` â Gold `dim_customer.customer_name` |
| Asset SOCI Criticality Flag | Asset Management | OFFICIAL: SENSITIVE â OPERATIONAL (SOCI) | SAP classification `AUSP` table values â Maximo `ASSET.SOCI_CRITICAL` â EDAP `dim_functional_location.soci_critical_flag` |
| Water Quality Contamination Indicator | Operations | OFFICIAL: SENSITIVE â OPERATIONAL (SOCI) | Lab system `RESULTS.CONTAMINATION_FLAG` â EDAP `fact_water_quality.contamination_incident` |
| Employee Health Record | People | OFFICIAL: SENSITIVE â PERSONAL (PRIS) | HR system personnel records â EDAP `dim_employee.health_indicator` (if ingested) |

The steward's job is to maintain the register of governed data elements, their classifications, and the mapping of each element to its physical manifestation in each system. This register is the authoritative source for *what needs protecting and at what granularity*.

### 5.3 Enforcement by Granularity in the EDAP

Unity Catalog's governed tags and ABAC operate at all three granularity levels:

**Column-level enforcement**: Governed tags are applied directly to columns, not just tables. When automated Data Classification detects that `customer_name`, `phone_number`, and `email_address` are PII, it tags those specific columns with `sensitivity = pii`. An ABAC column mask policy then enforces: *"for any column tagged `sensitivity = pii`, mask the value unless the requesting user is in the `pii_authorised` group."* The policy is defined once at the catalog level and automatically applies to every PII-tagged column in every table across every schema â current and future. Non-sensitive columns in the same table remain fully accessible.

```sql
-- Example: ABAC column mask policy for PII
-- Defined once at catalog level, inherited by all tables
-- Triggers on any column tagged sensitivity = pii

CREATE FUNCTION mask_pii(value STRING)
RETURNS STRING
RETURN CASE
  WHEN is_member('pii_authorised') THEN value
  ELSE '********'
END;

-- Policy applied via UI or API:
-- Scope: prod_gold catalog, all schemas
-- Match columns: tag sensitivity = pii
-- Action: mask using mask_pii function
-- Applies to: All account users
-- Except: pii_authorised group
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

A table's *discovery-level classification* reflects the highest sensitivity of any data element it contains. This is the **highest watermark** â it governs how the table appears in Discover, how it is represented in audit queries, and which steward approval is required for access requests.

```
ââââââââââââââââââââââââââââââââââââââââââââââââââââââââ
â  Table: dim_functional_location                       â
â                                                       â
â  Table-level tag (highest watermark):                 â
â    waicp_classification = official_sensitive_operationalâ
â    soci_relevant = true                               â
â                                                       â
â  Column-level tags:                                   â
â    location_id           â (no sensitivity tag)       â
â    location_description  â (no sensitivity tag)       â
â    parent_location       â (no sensitivity tag)       â
â    soci_critical_flag    â soci_filter_column = true  â
â    gps_latitude          â soci_sensitive = true      â
â    gps_longitude         â soci_sensitive = true      â
â    security_zone_detail  â soci_sensitive = true      â
â                                                       â
â  Enforcement:                                         â
â    Row filter: hides SOCI-critical rows from          â
â                unauthorised users                     â
â    Column mask: masks GPS coordinates and security    â
â                 zone details for SOCI-critical rows   â
â    Discovery: table appears as SENSITIVE in Discover  â
â    Access: unrestricted for non-SOCI rows/columns    â
â            restricted for SOCI elements only          â
ââââââââââââââââââââââââââââââââââââââââââââââââââââââââ
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
| **Domain** | Identifies business ownership | `data_domain = asset_management` | Discovery, stewardship routing, cost attribution |
| **Regulatory Classification** | Maps regulatory obligations to assets | `waicp_classification = official_sensitive_personal` | ABAC column masking, row filtering |
| **Critical Infrastructure** | SOCI Act designation | `soci_critical = true` | ABAC access restriction, enhanced audit logging |
| **Privacy** | PRIS Act classification | `pris_classification = personal` | ABAC column masking, anonymisation in non-prod (per ADR-EDP-001) |
| **Sensitivity** | Auto-detected data sensitivity | `sensitivity = pii`, `sensitivity = phi` | Automated Data Classification â ABAC masking |
| **Quality Tier** | Steward-certified quality level | `quality_tier = certified`, `quality_tier = provisional` | Discover marketplace ranking, consumer trust signals |
| **Data Product** | Data product membership | `data_product = asset_condition_analytics` | Discover marketplace organisation, access request bundling |

### 6.2 Tag Governance Process

1. **Tag policies** are defined at the account level by the central governance function, establishing the controlled vocabulary (allowed tag keys and permitted values)
2. **Automated classification** applies sensitivity tags (PII, PHI) without human intervention
3. **Steward-driven tagging** applies domain, regulatory, and quality tags based on domain policy decisions
4. **ABAC policies** reference governed tags to enforce access controls â defined once at the catalog or schema level, automatically inherited by all current and future tables
5. **Audit and monitoring** via system tables and Governance Insights dashboard track tag application, policy enforcement, and access patterns

---

## 7. Cross-System Consistency: Keeping Governance Aligned

The hardest governance challenge is ensuring that a domain policy decision propagates consistently across all systems that hold data within that domain's scope.

### 7.1 EDAP as Governance Hub

For the current maturity stage, the EDAP (via Unity Catalog) serves as the **governance hub** â the system of record for domain classification taxonomy and the primary enforcement point for analytics governance:

```
                    ââââââââââââââââââââââââââââ
                    â     Domain Governance     â
                    â    (Policy Decisions)     â
                    ââââââââââââââ¬ââââââââââââââ
                                 â
                    ââââââââââââââ¼ââââââââââââââ
                    â   Unity Catalog (EDAP)    â
                    â                          â
                    â  â¢ Governed Tags          â
                    â    (classification        â
                    â     taxonomy)             â
                    â  â¢ ABAC Policies          â
                    â  â¢ Metric Views           â
                    â  â¢ Discover Marketplace   â
                    â  â¢ External Lineage       â
                    â                          â
                    â  System of record for:    â
                    â  domain classification,   â
                    â  business definitions,    â
                    â  analytics access policy  â
                    â                          â
                    ââââ¬âââââââââââââ¬ââââââââ¬âââ
                       â            â       â
            ââââââââââââ¼âââ  âââââââ¼âââââ ââ¼âââââââââââ
            â  SAP ECC    â  â Maximo   â â ESRI GIS  â
            â             â  â          â â           â
            â Operational â  âOperationalâ âOperationalâ
            â governance  â  âgovernance â âgovernance â
            â (native     â  â(native   â â(native    â
            â  controls)  â  â controls)â â controls) â
            â             â  â          â â           â
            â Stewards    â  âStewards  â âStewards   â
            â validate    â  âvalidate  â âvalidate   â
            â alignment   â  âalignment â âalignment  â
            âââââââââââââââ  ââââââââââââ âââââââââââââ
```

**How it works**:

- Domain classification decisions are recorded as governed tags in Unity Catalog
- ABAC policies enforce those classifications within the EDAP automatically
- Stewards maintain a **cross-system alignment register** (initially in Confluence, potentially in Purview or a business catalogue as maturity increases) that maps each domain classification decision to its corresponding source system controls
- Periodic stewardship reviews validate that source system controls remain aligned with the EDAP-authoritative classification taxonomy

### 7.2 When to Introduce an Enterprise Catalogue Layer

The EDAP-as-governance-hub pattern is appropriate when:

- The EDAP is the primary analytics and reporting platform
- Most data consumption for decision-making happens through the EDAP
- The source system estate is manageable in scope (fewer than ~10 major systems)
- Regulatory requirements can be met through a combination of Unity Catalog enforcement + documented stewardship processes

Consider introducing a dedicated enterprise catalogue layer (e.g. Microsoft Purview Unified Catalog, Collibra, Atlan) when:

- Regulatory audit requirements demand a *single, unified view* of data access and classification across all systems â not just the EDAP
- The source system estate grows to a scale where manual cross-system alignment registers become unsustainable
- Formal stewardship workflows with multi-step approval chains, escalation paths, and SLA tracking are required by regulation or organisational policy
- Cross-platform data product discovery is needed (e.g. business users need to discover data in SAP, Maximo, GIS, and EDAP from a single interface)

---

## 8. AI Governance in the Federated Model

As AI and machine learning capabilities mature within the EDAP, the three-layer governance model must extend to cover AI-specific governance concerns. AI introduces new risks around data usage rights, model output ownership, and cross-domain data combination that the existing classification model does not fully address.

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
- All contributing domain owners must be consulted before the model output is published as a data product. Where any contributing domain owner objects, the matter is escalated to the EIGC.
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

### 9.3 Incident Response Governance

Data governance incidents require a clear command structure that operates across the three-layer model. The following incident types are in scope:

| Incident Type | Description | Mandatory Reporting |
|---|---|---|
| **PII exposure** | Unauthorised access to or disclosure of personal information | PRIS Act 2024 breach notification obligations; Privacy Commissioner notification where applicable |
| **SOCI-classified data leak** | Unauthorised access to or disclosure of data classified under the SOCI Act 2018 | SOCI Act mandatory reporting to the Cyber and Infrastructure Security Centre (CISC) within timeframes prescribed by the Act |
| **Data quality incident** | A material data quality failure that affects downstream decision-making, reporting, or regulatory compliance | Internal escalation; regulatory reporting where the quality failure affects compliance obligations |

**Incident command model:**

- The **Data Protection Officer (DPO)** is the incident commander for PII exposure and SOCI-classified data leak incidents. The DPO coordinates response across all three governance layers, engages legal and compliance, and manages mandatory reporting obligations.
- The **domain owner** is the incident commander for data quality incidents within their domain. Where a quality incident spans multiple domains, the EIGC nominates a lead domain owner.
- During an incident, the three-layer model operates as follows:
  - **Layer 1 (Domain Governance):** The domain owner authorises emergency access changes, temporary data restrictions, or communication to affected stakeholders.
  - **Layer 2 (Source System Governance):** Source system administrators implement emergency access revocations or restrictions as directed by the incident commander.
  - **Layer 3 (EDAP Governance):** The platform team implements emergency Unity Catalog access revocations, tag changes, or data quarantine actions as directed by the incident commander.

**SOCI Act mandatory reporting:** As a critical infrastructure entity under the SOCI Act 2018, Water Corporation has mandatory reporting obligations for cyber security incidents affecting critical infrastructure assets. The DPO, in coordination with the CISO, must ensure that incidents involving SOCI-classified data are reported to the CISC within the prescribed timeframes (currently 12 hours for critical incidents, 72 hours for other reportable incidents). The governance register must record all incident response actions for audit purposes.

### 9.4 Quality Accountability Model

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
The Asset Management domain owner, in consultation with the SOCI Act compliance team, determines that all data relating to water treatment plant assets designated as critical infrastructure under SOCI Act 2018 must be classified `OFFICIAL: SENSITIVE â OPERATIONAL` and restricted to personnel with a verified operational need.

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
| Base / Protected | Governed tags applied: `data_domain = asset_management`, `soci_critical = true`, `waicp_classification = official_sensitive_operational`. ABAC row filter policy activated: only users in the `soci_authorised` group can access rows where `soci_critical = true`. |
| Gold / BI | Data product "Critical Infrastructure Asset Analytics" published to Discover marketplace with domain ownership, steward certification, quality signals, and SOCI access restriction. Metric Views define governed calculations for critical asset condition scoring. External lineage traces data provenance to SAP ECC and Maximo source records. |

**Steward validation**:
The Asset Management data steward validates that ABAC policies in the EDAP, SAP authorisation roles, Maximo security groups, and GIS feature service permissions all consistently enforce the domain owner's access restriction. This validation is recorded in the cross-system alignment register and reviewed at the fortnightly stewardship review.

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

## 12. Future State Considerations

As the governance model matures, the following evolutionary steps should be evaluated:

1. **Purview integration**: If regulatory audit requirements for SOCI or PRIS Act demand a single cross-system governance view, Microsoft Purview Unified Catalog can provide a federated discovery and compliance layer that scans Unity Catalog alongside SAP, Maximo, and other Azure-connected sources. Purview's governance domains align naturally with the domain model described here.

2. **Formal stewardship workflows**: If the stewardship cadence described in section 8.1 proves insufficient for regulatory demands, a dedicated business catalogue (Collibra, Atlan) can provide workflow orchestration with approval chains, escalation paths, and SLA tracking. This should be a pull-based decision driven by demonstrated need, not a push-based procurement.

3. **Data contracts** (current emerging practice): Data contracts are being adopted as the formal interface between producer and consumer domains. A data contract defines: a **schema definition** (guaranteed column structure, data types, and nullable constraints); a **freshness SLA** (maximum acceptable latency between source change and data product availability); **quality thresholds** (minimum acceptable DQ pass rates and specific rules the producer commits to enforcing); **ownership** (the data product owner accountable for meeting the contract); **versioning** (semantic versioning with a defined deprecation period for breaking changes); and a **breaking change policy** (versioned, notify, or unmanaged). Contracts are enforced through the SDP framework (`contract_version`, `contract_sla_tier`, and `breaking_change_policy` tags in Unity Catalog) and monitored via Lakehouse Monitoring and DQ expectations. This replaces the earlier framing of data contracts as a future-state consideration; they are now an active part of the governance model.

4. **Computational governance**: Progressively shift governance enforcement from human-driven stewardship review towards automated, policy-as-code execution â governed tags applied automatically based on schema patterns, ABAC policies inherited by convention, quality rules enforced in pipelines. Human governance effort shifts from enforcement to exception management and policy evolution.

---

## References

| Reference | Description |
|---|---|
| EDAP-FWK-001 | Streaming Declarative Pipeline Framework Specification |
| ADR-EDP-001 | Databricks Development Environment Data Strategy |
| WAICP | WA Information Classification Policy |
| SOCI Act 2018 | Security of Critical Infrastructure Act |
| PRIS Act 2024 | Privacy and Responsible Information Sharing Act |
| State Records Act 2000 | WA State Records Act |
| Essential Eight | ACSC Essential Eight Maturity Model |
| Dehghani, Z. (2022) | *Data Mesh: Delivering Data-Driven Value at Scale* â Federated computational governance principles |
| Databricks (2025) | Unity Catalog ABAC, Governed Tags, Business Semantics, and Discover documentation |

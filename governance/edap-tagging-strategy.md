# EDAP Tagging Strategy — Classification & Metadata Tagging Framework

**Mark Shaw** | Principal Data Architect

---

## 1. Purpose

This document defines the metadata tagging strategy for Water Corporation's Enterprise Data & Analytics Platform (EDAP). It establishes a layered tagging model that:

- Aligns with the **Western Australian Information Classification Policy (WAICP)** as the mandatory corporate classification framework
- Extends beyond WAICP to provide the granularity required for data platform governance, access control, and regulatory compliance
- Maps tags to **Unity Catalog enforcement mechanisms** (column masking, row-level security, Delta Sharing, access policies)
- Supports integration with **Alation** for enterprise-wide metadata governance
- Addresses **cross-domain data access** through a classification lifecycle that balances governance with delivery velocity
- Handles **multi-domain source systems** where a single source feeds multiple business domains

---

## 2. Why WAICP Alone Is Not Sufficient

The WAICP provides a common language for WA Government agencies to identify risks and apply security controls when sharing information. It defines three classification levels — UNOFFICIAL, OFFICIAL, and OFFICIAL: Sensitive — with sublabels for Cabinet, Personal, Commercial, and Legal. For Commonwealth security-classified information (PROTECTED, SECRET, TOP SECRET), agencies must comply with the Protective Security Policy Framework (PSPF) inter-jurisdictional agreements.

This framework was designed for **document-level classification** to support information sharing between agencies. It is necessary but insufficient for an enterprise data platform because:

| Gap | Explanation |
|-----|-------------|
| **No column-level granularity** | WAICP classifies documents, not individual data fields. A table classified as OFFICIAL: Sensitive–Personal doesn't tell you *which columns* contain the PI or what masking rule to apply. |
| **No sensitivity reason** | WAICP tells you *that* something is sensitive, but not *why*. Is it PI under the PRIS Act 2024? Operational SCADA data under the SOCI Act 2018? Commercially sensitive pricing? Each requires different technical controls. |
| **No enforcement mapping** | WAICP doesn't prescribe data platform controls. It doesn't tell you whether to apply column masking, row-level security, schema-level isolation, or encryption — these are platform-specific decisions. |
| **No operational metadata** | WAICP has no concept of data lineage zones, retention periods, medallion layers, source systems, or data ownership — all essential for platform governance. |
| **Limited sensitivity spectrum** | Water Corporation handles data with sensitivities that don't map cleanly to WAICP sublabels — e.g. infrastructure vulnerability data (SCADA), geospatial asset locations, Aboriginal heritage site data, operational telemetry. These need their own classification dimensions. |
| **No classification lifecycle** | WAICP assumes data is classified at creation. In a data platform, there is an unavoidable lag between ingestion and classification — new source systems are onboarded before governance teams can complete domain assignment and sensitivity assessment. WAICP provides no model for managing data during this gap. |
| **No cross-domain access model** | WAICP does not address how data should be shared across internal business domains, or how access posture should change as classification matures from initial ingestion through to full governance sign-off. |

**The EDAP tagging strategy therefore implements WAICP as the mandatory base layer and extends it with three additional layers — plus a classification lifecycle model and cross-domain governance framework — to provide the granularity required for platform governance.**

---

## 3. Layered Tagging Model

The EDAP uses a four-layer tagging model. Each layer serves a distinct purpose and maps to different governance concerns.

```
┌─────────────────────────────────────────────────────────┐
│  Layer 4: Data Management (EDAP Operational)            │
│  medallion_layer, source_system, data_domain,           │
│  classification_status, etc.                            │
├─────────────────────────────────────────────────────────┤
│  Layer 3: Access & Handling (Technical Enforcement)     │
│  access_model, masking_required, sharing_permitted      │
├─────────────────────────────────────────────────────────┤
│  Layer 2: Sensitivity Reason (Why It's Sensitive)       │
│  pi_contained, regulatory_scope, sensitivity_type      │
├─────────────────────────────────────────────────────────┤
│  Layer 1: WAICP Classification (Mandatory Corporate)    │
│  waicp_classification                                   │
└─────────────────────────────────────────────────────────┘
```

**Principle:** Tags at each layer are independent but composable. A single table or column may carry tags from all four layers simultaneously. The layers are not hierarchical in a strict sense — rather, Layer 1 and Layer 2 inform Layer 3 decisions, and Layer 4 operates independently for platform management. The `classification_status` tag in Layer 4 governs how cross-domain access evolves as classification matures.

---

## 4. Layer 1 — WAICP Classification (Mandatory Corporate)

### 4.1 Overview

This layer implements the Western Australian Information Classification Policy. It is **mandatory** for all data assets in the EDAP.

### 4.2 Tag Definition

| Tag Key | `waicp_classification` |
|---------|------------------------|
| **Applied to** | Tables, views (mandatory). Schemas and catalogs (where all child objects share the same classification). |
| **Cardinality** | Single value per object. The classification reflects the *highest sensitivity* of any data within the object. |
| **Governance** | Data owners are responsible for assigning the correct classification. Changes require approval from the data steward. |

### 4.3 Allowed Values

| Value | Description | Typical EDAP Data |
|-------|-------------|-------------------|
| `UNOFFICIAL` | Not part of official duties. Zero business impact if compromised. | Test data, synthetic datasets, sandbox experimental outputs |
| `OFFICIAL` | Routine business operations and services. Requires a routine level of protection. Majority of WC information falls here. | Operational reports, general asset registers, public-facing reference data, standard BI aggregations |
| `OFFICIAL:Sensitive` | Sensitive information requiring protective security measures and limited dissemination. Highest level not covered by inter-jurisdictional agreements. | Detailed operational telemetry, internal financial data, detailed infrastructure configuration |
| `OFFICIAL:Sensitive-Personal` | OFFICIAL: Sensitive with Personal sublabel. Information about identifiable individuals. | Employee records, customer PI, HR data, health & safety incident records with personal details |
| `OFFICIAL:Sensitive-Commercial` | OFFICIAL: Sensitive with Commercial sublabel. Commercially sensitive information. | Contract pricing, tender evaluations, supplier commercial terms, financial modelling |
| `OFFICIAL:Sensitive-Legal` | OFFICIAL: Sensitive with Legal sublabel. Subject to legal privilege or proceedings. | Legal advice, litigation documents, regulatory investigation data |
| `OFFICIAL:Sensitive-Cabinet` | OFFICIAL: Sensitive with Cabinet sublabel. Related to Cabinet processes. | Ministerial briefings, Cabinet submissions, policy deliberations |

### 4.4 Classification Rules

- **Default classification:** All new tables in the EDAP default to `OFFICIAL` unless explicitly classified otherwise.
- **Inheritance:** If a schema-level `waicp_classification` tag is set, all child tables inherit it unless overridden at table level with a higher classification.
- **Elevation only:** Classification can only be elevated (made more restrictive), never downgraded, without formal approval from the data steward and documented justification.
- **Aggregation rule:** When data from multiple sources is combined (e.g. in Silver Enriched or Gold zones), the resulting table inherits the *highest* classification of any contributing source.
- **BI Zone:** Gold BI Zone tables should carry the classification of their underlying data. Pre-aggregated KPI tables that contain no individual-level detail may be classified at a lower level than their source, subject to data steward approval.

### 4.5 Classification and the Classification Lifecycle

The `waicp_classification` tag interacts with the `classification_status` tag (Section 7.3). When a table is in `unclassified` status, the `waicp_classification` is set to the default (`OFFICIAL`) as a provisional holding value. This default is not an authoritative classification — it is updated when the data steward completes the classification assessment and progresses the table to `provisional` or `classified` status.

### 4.6 Relationship to Commonwealth Classifications

Water Corporation does not routinely handle PROTECTED, SECRET, or TOP SECRET information within the EDAP. If such data is received from Commonwealth agencies, it must be handled per the relevant inter-jurisdictional agreement and **must not be ingested into the EDAP** without explicit approval from the Chief Information Security Officer (CISO) and alignment with PSPF requirements for ICT system certification.

---

## 5. Layer 2 — Sensitivity Reason (Why It's Sensitive)

### 5.1 Overview

This layer explains the **nature** of the sensitivity. While WAICP tells you *that* something is sensitive, Layer 2 tells you *why* — which is what drives the actual technical controls in Layer 3. A table classified as `OFFICIAL:Sensitive` might be sensitive for multiple reasons simultaneously (e.g. it contains PI *and* is subject to SOCI Act obligations).

### 5.2 Tag Definitions

#### 5.2.1 PI Indicator

| Tag Key | `pi_contained` |
|---------|-----------------|
| **Applied to** | Tables (mandatory for all Silver and Gold tables). Columns (mandatory where `pi_contained = true` at table level, to identify *which* columns). |
| **Allowed values** | `true`, `false` |
| **Purpose** | Drives column masking policies, Protected Zone routing, PRIS Act 2024 compliance, and anonymisation requirements for non-production environments. |

**Column-level PI sub-tagging** (applied only to columns where table-level `pi_contained = true`):

| Tag Key | `pi_type` |
|---------|-----------|
| **Applied to** | Columns |
| **Allowed values** | `direct_identifier`, `indirect_identifier`, `sensitive_pi` |

| Value | Description | Masking Implication | Examples |
|-------|-------------|-------------------|----------|
| `direct_identifier` | Directly identifies an individual | Full masking or redaction | Name, address, email, phone, employee ID, TFN |
| `indirect_identifier` | Could identify an individual when combined with other data | Partial masking or generalisation | Date of birth (→ year only), postcode (→ first 2 digits), job title |
| `sensitive_pi` | Sensitive personal information per PRIS Act 2024 | Full masking, Protected Zone routing recommended | Health records, biometric data, racial/ethnic origin, religious beliefs |

#### 5.2.2 Regulatory Scope

| Tag Key | `regulatory_scope` |
|---------|-------------------|
| **Applied to** | Tables (mandatory for all Silver and Gold tables) |
| **Allowed values** | Multi-valued (comma-separated). Identifies which legislation or regulation imposes obligations on this data. |

| Value | Legislation | Implication |
|-------|------------|-------------|
| `pris_act` | Privacy and Responsible Information Sharing Act 2024 (WA) | PI handling obligations. Column masking, anonymisation for non-prod, breach notification requirements. |
| `soci_act` | Security of Critical Infrastructure Act 2018 (Cth), including 2024 SOCI Rules amendments | Critical infrastructure risk management. Enhanced access controls, audit logging, potential CMK encryption. The 2024 amendments expanded positive security obligations including enhanced risk management programme requirements. |
| `state_records_act` | State Records Act 2000 (WA) | Retention and disposal obligations. Drives `retention_days` tag values. |
| `privacy_act` | Privacy Act 1988 (Cth) | Relevant where WC handles information subject to Commonwealth privacy obligations (e.g. inter-agency data sharing). |
| `foi_act` | Freedom of Information Act 1992 (WA) | Information may be subject to FOI requests. Drives discoverability and classification accuracy requirements. |
| `none` | No specific regulatory obligation beyond standard governance | Default for general operational data. |

#### 5.2.3 Sensitivity Type

| Tag Key | `sensitivity_type` |
|---------|-------------------|
| **Applied to** | Tables, columns |
| **Allowed values** | Multi-valued. Provides a more granular reason for sensitivity beyond WAICP sublabels. |

| Value | Description | Example Data |
|-------|-------------|-------------|
| `personal_information` | Information about identifiable individuals (aligns with WAICP Personal sublabel) | Customer names, employee details, health records |
| `commercial_in_confidence` | Commercially sensitive (aligns with WAICP Commercial sublabel) | Contract values, tender pricing, supplier terms |
| `legal_privilege` | Subject to legal professional privilege (aligns with WAICP Legal sublabel) | Legal advice, litigation files |
| `infrastructure_vulnerability` | Information that could be used to compromise critical infrastructure | SCADA network topology, control system configurations, physical security layouts, pipeline pressure thresholds |
| `operational_security` | Operational information that could compromise service delivery if disclosed | Real-time operational telemetry, incident response procedures, system credentials |
| `geospatial_sensitive` | Geospatial data revealing sensitive infrastructure locations | Precise coordinates of critical assets, pipeline routes through restricted areas |
| `indigenous_cultural` | Data relating to Aboriginal cultural heritage sites or practices | Heritage site locations, cultural survey data (subject to Aboriginal Heritage Act 1972 and Aboriginal Cultural Heritage Act 2021) |
| `financial_sensitive` | Detailed financial information not publicly reported | Internal cost models, budget deliberations, investment analysis |
| `environmental_sensitive` | Environmental monitoring data with compliance implications | Discharge monitoring, contamination levels, compliance test results |

#### 5.2.4 Consent and Lawful Basis

| Tag Key | `pi_lawful_basis` |
|---------|-------------------|
| **Applied to** | Tables (mandatory where `pi_contained = true`) |
| **Allowed values** | `consent`, `legitimate_interest`, `legal_obligation`, `vital_interest`, `public_interest`, `contractual_necessity`, `not_assessed` |
| **Purpose** | Records the lawful basis under which personal information in this table is collected and processed, as required by the PRIS Act 2024 and Privacy Act 1988. Drives AI training approval gates (data with `pi_lawful_basis = consent` requires explicit verification that consent extends to AI use), regulatory audit evidence, and stewardship review. |

| Value | Description |
|-------|-------------|
| `consent` | Individual has provided informed consent for the specific processing purpose. Consent scope must be verified before any new use (e.g. AI training). |
| `legitimate_interest` | Processing is necessary for Water Corporation's legitimate interests and does not override the individual's rights. |
| `legal_obligation` | Processing is required to comply with a legal obligation (e.g. regulatory reporting). |
| `vital_interest` | Processing is necessary to protect someone's life or safety (e.g. emergency response). |
| `public_interest` | Processing is necessary for a task carried out in the public interest (e.g. public health water quality reporting). |
| `contractual_necessity` | Processing is necessary for the performance of a contract with the individual. |
| `not_assessed` | Lawful basis has not yet been assessed. Tables with this value must be prioritised for steward review. |

**Relationship to classification lifecycle:** Tables in `unclassified` status default to `pi_lawful_basis = not_assessed`. The steward must assign an authoritative lawful basis as part of the classification assessment when progressing to `provisional` status.

**Privacy Act reform note:** The Privacy Act reform programme (post-AGD review) may introduce new obligations around automated decision-making and enhanced consent requirements. The `pi_lawful_basis` tag is designed to support future requirements around purpose limitation and consent granularity. If reform introduces mandatory consent for automated decision-making, tables used as inputs to AI/ML models will require `pi_lawful_basis = consent` with documented evidence that consent scope extends to automated processing.

#### 5.2.5 AI/ML Governance

These tags govern whether and how data may be used for artificial intelligence and machine learning purposes, including model training, inference, and synthetic data generation. This section aligns with **ISO/IEC 42001:2023** (AI Management Systems), the **NIST AI Risk Management Framework (AI RMF 1.0)**, and the **Australian Government Voluntary AI Safety Standard** (2024).

| Tag Key | `ai_training_permitted` |
|---------|------------------------|
| **Applied to** | Tables |
| **Allowed values** | `true`, `false` |
| **Purpose** | Controls whether the data in this table may be used as training input for machine learning models. Must be `false` for tables containing personal information unless the `pi_lawful_basis` tag confirms a lawful basis that extends to AI training use (e.g. `consent` with documented AI training scope, or `legitimate_interest` with a completed privacy impact assessment). |

| Tag Key | `ai_use_restriction` |
|---------|---------------------|
| **Applied to** | Tables |
| **Allowed values** | `none`, `internal_model_only`, `not_permitted` |
| **Purpose** | Controls the permitted AI consumption pattern for this data. |

| Value | Description |
|-------|-------------|
| `none` | No AI-specific restriction beyond standard access controls. |
| `internal_model_only` | Data may only be consumed by internally developed and governed models. Not permitted for use with third-party or externally hosted AI services. |
| `not_permitted` | Data must not be used for any AI/ML purpose, including feature engineering, model training, or inference input. |

| Tag Key | `synthetic_derivation` |
|---------|----------------------|
| **Applied to** | Tables |
| **Allowed values** | `true`, `false` |
| **Purpose** | Indicates whether the data is synthetically generated or has been used to generate synthetic data. Synthetic datasets derived from sensitive sources inherit the classification of the source for governance purposes, even where individual records are not identifiable. |

#### 5.2.6 Automated Data Classification Integration

Unity Catalog's automated Data Classification feature detects PI and other sensitive data patterns at the column level. This section defines how automated detection integrates with the manual tag taxonomy.

**Mapping from automated detection to Layer 2 tags:**

| Auto-Detected Category | Maps To `pi_contained` | Maps To `pi_type` | Maps To `sensitivity_type` |
|---|---|---|---|
| Personal name | `true` | `direct_identifier` | `personal_information` |
| Email address | `true` | `direct_identifier` | `personal_information` |
| Phone number | `true` | `direct_identifier` | `personal_information` |
| Physical address | `true` | `direct_identifier` | `personal_information` |
| Date of birth | `true` | `indirect_identifier` | `personal_information` |
| Financial identifier (TFN, bank account) | `true` | `sensitive_pi` | `personal_information`, `financial_sensitive` |
| Health indicator | `true` | `sensitive_pi` | `personal_information` |

**Authority model:**

Automated Data Classification tags are treated as **advisory, not authoritative**. They provide a high-confidence starting point that accelerates the steward's classification assessment, but they do not replace steward judgement. Specifically:

- Auto-detected tags are applied immediately and are visible in Unity Catalog and Discover.
- Auto-detected tags do **not** change the `classification_status` from `unclassified` to `provisional`. Only a steward review can progress the classification lifecycle.
- Stewards must confirm or override auto-detected tags as part of the classification assessment. Once confirmed, the tag source is recorded as `steward_confirmed` in the governance register.

**Conflict resolution:**

Where automated detection and steward-assigned tags disagree:

1. **Steward assigns a higher sensitivity than auto-detection:** The steward's classification takes precedence. This is common for business-context classifications (e.g. SOCI criticality) that automated detection cannot identify.
2. **Steward assigns a lower sensitivity than auto-detection:** The steward must provide documented justification. Common scenarios include false positives (e.g. a column named `contact_name` that contains organisation names, not personal names). The justification is recorded in the governance register and reviewed at the next stewardship review.
3. **Auto-detection identifies PI that the steward missed:** The automated tag remains active as a protective default until the steward explicitly reviews and either confirms or overrides it.

---

## 6. Layer 3 — Access & Handling (Technical Enforcement)

### 6.1 Overview

This layer translates the classification (Layer 1) and sensitivity reason (Layer 2) into **concrete Unity Catalog enforcement actions**. These tags drive automated policy enforcement within the EDAP.

### 6.2 Tag Definitions

#### 6.2.1 Access Model

| Tag Key | `access_model` |
|---------|---------------|
| **Applied to** | Tables, schemas |
| **Allowed values** | `open`, `controlled`, `restricted`, `privileged` |

| Value | Description | UC Enforcement |
|-------|-------------|----------------|
| `open` | Available to all authenticated EDAP users with appropriate catalog access | Standard UC GRANT. No additional controls. |
| `controlled` | Available to users within the owning data domain and approved cross-domain subscribers | UC GRANT to domain security groups. Cross-domain access via request. |
| `restricted` | Available only to specifically authorised users/groups | UC GRANT to named groups only. Workspace binding recommended. |
| `privileged` | Highest restriction. PAM-controlled access, full audit trail | UC GRANT via Privileged Access Management. Session-level access only. |

**Interaction with classification_status:** The `access_model` tag is interpreted differently depending on `classification_status` (see Section 8). For `unclassified` data, access is restricted to the domain team regardless of the `access_model` value. The `access_model` tag only takes full effect when `classification_status` reaches `classified`.

#### 6.2.2 Column Masking

| Tag Key | `masking_required` |
|---------|-------------------|
| **Applied to** | Columns |
| **Allowed values** | `none`, `partial`, `full`, `hash`, `redact` |

| Value | Description | UC Implementation |
|-------|-------------|-------------------|
| `none` | No masking required | No column mask function applied |
| `partial` | Partial masking (generalisation) | Column mask function returns generalised value (e.g. year only for dates, first 2 digits for postcodes) |
| `full` | Full masking | Column mask function returns NULL or `***MASKED***` for unauthorised users |
| `hash` | One-way hash | Column mask function returns SHA-256 hash (supports joins without revealing value) |
| `redact` | Complete redaction — column not visible | Column mask function returns NULL; column excluded from discovery for unauthorised users |

**Masking function naming convention:** `edap_mask_<sensitivity_category>` — e.g. `edap_mask_pi_name`, `edap_mask_financial`, `edap_mask_health`. Functions must be defined as reusable Unity Catalog functions so that the same masking logic is applied consistently across tables.

**Example:**

```sql
-- Reusable masking function for direct PI identifiers
CREATE FUNCTION edap_mask_pi_direct(value STRING)
RETURNS STRING
RETURN CASE
  WHEN is_member('sg-edap-pi-authorised') THEN value
  ELSE '***MASKED***'
END;

-- Apply to a column
ALTER TABLE prod_silver.customer_base.customer_master
  ALTER COLUMN customer_name SET MASK edap_mask_pi_direct;
```

#### 6.2.3 Sharing Permissions

| Tag Key | `sharing_permitted` |
|---------|-------------------|
| **Applied to** | Tables, schemas |
| **Allowed values** | `internal_only`, `interagency`, `delta_sharing`, `public` |

| Value | Description | UC Enforcement |
|-------|-------------|----------------|
| `internal_only` | Not included in any Delta Sharing shares. | Not included in any Delta Sharing shares. |
| `interagency` | May be shared with other WA Government agencies under appropriate agreements. | May be included in Delta Sharing shares with government recipients. Requires data steward approval. |
| `delta_sharing` | Approved for sharing via Delta Sharing with named external recipients. | Included in configured Delta Sharing shares. Recipient list governed by data steward. |
| `public` | Suitable for public release (e.g. open data portal). | May be published to external endpoints. Must be classified `OFFICIAL` or `UNOFFICIAL`. |

**External sharing governance:** Tables tagged `delta_sharing` or `interagency` are subject to the external sharing governance process defined in the companion *Domain Governance Across Systems* document (section 7.4). This process requires domain owner approval, DPO review for PI/SOCI-classified data, and periodic steward review of active shares. Delta Sharing recipients are provisioned with time-limited tokens and all access is logged in Unity Catalog system tables for audit.

#### 6.2.4 Encryption at Rest

| Tag Key | `encryption_at_rest` |
|---------|---------------------|
| **Applied to** | Tables, schemas, catalogs |
| **Allowed values** | `platform_default`, `customer_managed_key` |

| Value | Description |
|-------|-------------|
| `platform_default` | AWS S3 server-side encryption (SSE-S3 or SSE-KMS with Databricks-managed key). Sufficient for OFFICIAL and most OFFICIAL: Sensitive data. |
| `customer_managed_key` | AWS KMS customer-managed key (CMK). Required for OFFICIAL: Sensitive data with `infrastructure_vulnerability` or `operational_security` sensitivity types, per SOCI Act risk management program. |

#### 6.2.5 Data Contract

These tags link a table to its governing data contract, enabling automated contract compliance monitoring and consumer trust signals.

| Tag Key | `contract_version` |
|---------|-------------------|
| **Applied to** | Tables |
| **Allowed values** | Semantic version string (e.g. `1.0.0`, `2.1.0`) |
| **Purpose** | Identifies the active data contract version governing the table's schema, quality thresholds, and freshness SLA. Consumers can pin to a contract version for stability. |

| Tag Key | `contract_sla_tier` |
|---------|-------------------|
| **Applied to** | Tables |
| **Allowed values** | `platinum`, `gold`, `silver`, `bronze` |
| **Purpose** | Defines the freshness and quality SLA tier for the data product. |

| Value | Freshness SLA | Quality Threshold |
|-------|---------------|-------------------|
| `platinum` | < 15 minutes | > 99.5% DQ pass rate |
| `gold` | < 1 hour | > 99% DQ pass rate |
| `silver` | < 24 hours | > 95% DQ pass rate |
| `bronze` | Best effort | > 90% DQ pass rate |

| Tag Key | `breaking_change_policy` |
|---------|------------------------|
| **Applied to** | Tables |
| **Allowed values** | `versioned`, `notify`, `none` |
| **Purpose** | Defines how breaking schema or semantic changes are communicated to consumers. |

| Value | Description |
|-------|-------------|
| `versioned` | Breaking changes result in a new contract version. The previous version remains available for a deprecation period. Consumers must explicitly migrate. |
| `notify` | Breaking changes are communicated to registered consumers with a minimum notice period, but do not result in a new contract version. |
| `none` | No formal breaking change policy. Appropriate for exploratory or sandbox data only. |

#### 6.2.6 Retention

| Tag Key | `retention_days` |
|---------|-----------------|
| **Applied to** | Tables |
| **Allowed values** | Integer (days) or `indefinite` |
| **Governance** | Must align with State Records Act 2000 retention and disposal schedules. Data steward responsible for setting correct retention. |

Common values: `90` (transient/operational), `365` (1 year), `2555` (7 years — standard financial), `3650` (10 years), `9125` (25 years — infrastructure records), `indefinite` (permanent retention).

---

## 7. Layer 4 — Data Management (EDAP Operational)

### 7.1 Overview

This layer supports platform operations, lineage, discoverability, and data product management. These tags are independent of security classification.

### 7.2 Core Tag Definitions

| Tag Key | Description | Allowed Values | Applied To |
|---------|-------------|---------------|------------|
| `medallion_layer` | Higher layer in the medallion structure | `bronze`, `silver`, `gold` | Catalogs, schemas |
| `medallion_sublayer` | Zone within the medallion layer | `landing`, `raw`, `protected`, `base`, `enriched`, `exploratory`, `bi`, `sandbox` | Schemas |
| `source_system` | Originating source system | `sap_ecc`, `maximo`, `esri`, `scada`, `grange`, `salesforce`, `manual`, etc. | Schemas, tables |
| `data_domain` | Business domain the data belongs to | `asset`, `customer`, `finance`, `operations`, `legal_compliance`, `people`, `technology_digital`, `enterprise_reference` | Schemas, tables |
| `data_owner` | Executive owner per domain model | GM-level role identifier | Schemas, tables |
| `data_steward` | Operational steward responsible for quality and classification | Named role or team | Schemas, tables |
| `data_type` | Nature of the data | `structured`, `semi_structured`, `unstructured`, `time_series`, `geospatial` | Tables |
| `refresh_frequency` | How often the data is updated | `realtime`, `hourly`, `daily`, `weekly`, `monthly`, `ad_hoc`, `event_driven` | Tables |
| `data_product` | Data product identifier (if applicable) | Product name, e.g. `asset_health_360`, `customer_billing_analytics` | Tables |
| `soci_critical` | Convenience flag indicating SOCI Act critical infrastructure data | `true`, `false` | Schemas, tables |
| `consuming_domain` | Primary consuming business domain (where different from owning domain) | Domain identifier, e.g. `operations`, `finance` | Tables (Gold) |
| `bi_published` | Whether the table is exposed to BI tools (Power BI, etc.) | `true`, `false` | Tables (Gold BI) |
| `quality_tier` | Steward-certified quality level indicating consumer trust | `certified`, `provisional`, `uncertified` | Tables (Silver, Gold) |
| `data_product_tier` | Maturity tier of a data product or ML model as a managed asset | `experimental`, `certified`, `deprecated` | Tables (Gold), Models |
| `ingestion_method` | How the data arrived in the platform | `autoloader`, `lakeflow_connect`, `cdc`, `zerobus`, `manual`, `api` | Tables (Bronze) |
| `ingestion_team` | Team responsible for the ingestion pipeline | Team identifier | Tables (Bronze) |

### 7.3 Classification Lifecycle Tag

| Tag Key | `classification_status` |
|---------|------------------------|
| **Applied to** | Tables, schemas (mandatory for all objects) |
| **Allowed values** | `unclassified`, `provisional`, `classified` |
| **Purpose** | Tracks the maturity of governance classification and controls cross-domain visibility and access posture. |

This tag is central to the cross-domain governance model described in Section 8. It addresses the unavoidable gap between data ingestion and governance classification — when a new source system is onboarded, data flows into the platform before the governance team can complete domain assignment, WAICP classification, and sensitivity assessment.

| Value | Description | Cross-Domain Visibility | Query Access |
|-------|-------------|------------------------|-------------|
| `unclassified` | Newly ingested, not yet reviewed by domain steward | Domain team only | Domain engineers + steward only |
| `provisional` | Initial classification assigned by steward, pending formal ratification | All EDAP users (metadata only — can see the table exists and read its description) | By request through entitlement process |
| `classified` | Full WAICP classification ratified, access policies enacted | All EDAP users | Per WAICP policy and `access_model` tag |

### 7.4 Domain Assignment Tag Behaviour by Layer

The `data_domain` tag has different semantics depending on the medallion layer:

| Layer | `data_domain` Meaning | Assignment Rule |
|-------|----------------------|-----------------|
| **Bronze (Raw)** | Source system identifier as holding value | Set automatically at ingestion to the source system name (e.g. `sap_ecc`, `scada`). This is **not** a business domain assignment — it is a placeholder that identifies the origin. |
| **Silver (Base/Enriched)** | Business domain ownership | Set by the pipeline that transforms Bronze into Silver. Assigned by the domain data steward based on business context. A single Bronze source may produce Silver tables in multiple domains. |
| **Gold (BI/Exploratory)** | Consuming or owning domain | Inherits from Silver source. Where a Gold table joins data from multiple Silver domains, assigned to the *primary consuming domain* with contributing domains recorded in lineage. |

**Rationale:** Source system boundaries and business domain boundaries are fundamentally different. SAP ECC contains financial data, asset data, HR data, and procurement data. A SCADA historian covers both water quality (potentially public health sensitive) and pump telemetry (operational). Forcing a business domain label onto a source-aligned Bronze table creates false precision. Domain assignment happens at Silver where the business meaning is understood.

### 7.5 Multi-Domain Source System Lineage

When a single Bronze source table feeds multiple Silver domain targets, the pipeline metadata framework must capture:

| Metadata Field | Description | Example |
|---------------|-------------|---------|
| `source_bronze_table` | Fully qualified Bronze table name | `prod_bronze.sap_ecc_raw.equi` |
| `target_silver_table` | Fully qualified Silver table name | `prod_silver.asset_base.equipment_master` |
| `column_lineage` | Column-level mapping from source to target | `equi.eqart → equipment_master.equipment_category` |
| `target_domain` | Business domain of the target table | `asset` |
| `domain_justification` | Business rationale for domain assignment | "Equipment master data is owned by Asset Management per Data Governance Council resolution 2026-003" |

**Example — SAP ECC multi-domain split:**

| Bronze Source | Silver Target | Domain |
|---|---|---|
| `prod_bronze.sap_ecc_raw.equi` | `prod_silver.asset_base.equipment_master` | Asset Management |
| `prod_bronze.sap_ecc_raw.equi` | `prod_silver.finance_base.asset_cost_centres` | Finance |
| `prod_bronze.scada_raw.historian_tags` | `prod_silver.operations_base.pump_telemetry` | Operations |
| `prod_bronze.scada_raw.historian_tags` | `prod_silver.water_quality_base.treatment_readings` | Water Quality |

This lineage is critical for impact analysis — when a Bronze schema changes, every downstream Silver domain consumer must be identifiable.

### 7.6 Domain Ownership Rules

Every Unity Catalog table must have exactly one owning domain. Shared or contested ownership creates accountability gaps.

- **One owner, many subscribers.** Each table has a single owning domain (recorded in `data_domain`) responsible for data quality, classification, and access policy. Other domains may subscribe (consume) the table under the access policies set by the owner.
- **Owner nomination.** For tables that could reasonably belong to multiple domains, ownership is assigned to the domain with the strongest accountability for the data's accuracy and currency. A Maximo work order table is owned by Asset Management (accountable for work order data quality) even though Operations, Finance, and Compliance all consume it.
- **Enterprise reference data.** Genuinely cross-cutting reference data (organisational hierarchy, location master, calendar, unit of measure) is assigned to `data_domain = enterprise_reference` with a nominated enterprise data steward. This avoids reference data being orphaned because no single business domain claims it.
- **Dispute resolution.** Where domain ownership is contested, the Data Governance Council adjudicates. The decision is recorded in the data catalogue and is binding until formally reviewed.

---

## 8. Classification Lifecycle & Cross-Domain Access

### 8.1 The Problem

When a new source system is onboarded into EDAP, the data engineering team needs to build and run ingestion pipelines before the data governance team has completed domain assignment, WAICP sensitivity classification, or executive ownership mapping. The tagging strategy must not create a governance bottleneck that blocks ingestion, but equally must not allow unclassified data to be broadly accessible before the organisation understands what it contains.

### 8.2 Core Principle

**The default access posture inverts during the classification window.**

The EDAP's standard philosophy is "open by default, restricted by exception" — unrestricted data is broadly accessible, and restrictions are applied only where justified. However, this philosophy assumes the data has been classified. For unclassified data, the posture is inverted: **restricted by default, opened as classification matures.**

```
Unclassified          Provisional           Classified
  (restrictive)  ──────►  (discoverable)  ──────►  (open by default)
  Domain only          Visible, request-based    Per WAICP policy
```

### 8.3 Classification States

Every Unity Catalog object (table, view, volume) progresses through three classification states, tracked via the mandatory `classification_status` tag (Section 7.3).

#### State 1: Unclassified

**When:** From the moment of first ingestion until a domain steward has performed initial review.

**Access posture:** Restrictive by default.

- Access is limited to the ingesting team's data engineers and the nominated domain data steward for the source system.
- Unity Catalog permissions are scoped to a domain-specific security group (e.g. `sg-edap-asset-mgmt-engineers`).
- Cross-domain users cannot see, discover, or query the table.
- The table carries `classification_status = unclassified`.
- The `waicp_classification` tag is set to the default (`OFFICIAL`) as a holding value — this is not an authoritative classification.

**SLA:** Objects must transition out of *Unclassified* within **30 calendar days** of first ingestion.

**Breach handling:** If the SLA is breached, an automated alert is raised to the domain data steward and the Data Governance team. Objects that remain unclassified beyond 60 days are escalated to the Data Governance Council.

#### State 2: Provisionally Classified

**When:** The domain data steward has performed an initial review and assigned:
- A provisional WAICP sensitivity level
- Domain ownership (`data_domain`, `data_owner`, `data_steward`)
- Basic column-level sensitivity flags (at minimum, `pi_contained` at table level)

This has not yet been formally ratified through the Data Governance Council governance approval process.

**Access posture:** Discoverable but controlled.

- The table is visible in Unity Catalog search and browse — users across domains can see that the table exists, read its description, and view column metadata.
- Querying the table requires an explicit access request through the agreed entitlement process.
- Provisional WAICP tags are applied with a `provisional:` prefix where appropriate (e.g. `waicp_classification = OFFICIAL:Sensitive-Personal`).
- The `classification_status = provisional` tag is set.
- Cross-domain discovery enables other teams to find data they need without duplicating ingestion or creating shadow datasets.

**SLA:** Objects must transition from *Provisional* to *Classified* within **60 calendar days** of entering the Provisional state. Complex source systems with many tables may request an extension through the Data Governance Council.

#### State 3: Classified

**When:** The WAICP classification has been formally ratified, executive domain ownership confirmed, and access policies enacted in Unity Catalog.

**Access posture:** Open by default, restricted by exception (the standard EDAP philosophy).

- Data classified as unrestricted under WAICP is broadly accessible to all EDAP users without requiring individual access requests.
- Data classified as restricted (e.g. personal information under PRIS Act, SOCI-critical operational data) is governed by Unity Catalog row-level security, column masks, or table-level grants as defined by the Layer 3 tags.
- Full WAICP tags are applied authoritatively.
- All Layer 2 and Layer 3 tags are complete and enacted.
- The `classification_status = classified` tag is set.

### 8.4 Classification Lifecycle Summary

| State | `classification_status` | Visibility | Query Access | Cross-Domain Sharing | Typical Duration |
|---|---|---|---|---|---|
| Unclassified | `unclassified` | Domain team only | Domain engineers + steward only | No | 0–30 days |
| Provisionally Classified | `provisional` | All EDAP users (metadata only) | By request | By request | 30–90 days |
| Classified | `classified` | All EDAP users | Per WAICP policy | Per WAICP policy + `access_model` | Ongoing |

### 8.5 Automation

- A scheduled Databricks job scans Unity Catalog nightly for objects with `classification_status = unclassified` older than 30 days and `classification_status = provisional` older than 60 days, raising alerts to the responsible steward and logging to the governance dashboard.
- Pipeline deployment to production Gold layer is gated: a table cannot be promoted to production Gold while its `classification_status` is `unclassified`. `provisional` is acceptable for production Silver/Base tables where the data is consumed only within the owning domain.
- The CI/CD pipeline validates that all mandatory tags (including `classification_status`) are present before deployment.

### 8.6 Interaction with Layer 3 Access Model

The `classification_status` and `access_model` tags work together:

| `classification_status` | `access_model` Effect |
|------------------------|----------------------|
| `unclassified` | **Overridden.** Regardless of `access_model` value, access is restricted to domain team only. |
| `provisional` | **Partially active.** `open` and `controlled` models are treated as `controlled` (request-based). `restricted` and `privileged` are enforced as-is. |
| `classified` | **Fully active.** `access_model` is enforced as defined. |

---

## 9. WAICP-to-UC Enforcement Mapping

This section maps WAICP classification values to minimum Layer 3 access controls in Unity Catalog.

| `waicp_classification` | Minimum `access_model` | UC Access Pattern |
|------------------------|----------------------|-------------------|
| `UNOFFICIAL` | `open` | GRANT USE SCHEMA, SELECT to all authenticated users |
| `OFFICIAL` | `open` or `controlled` | GRANT to relevant groups based on `data_domain` |
| `OFFICIAL:Sensitive` | `controlled` or `restricted` | GRANT to named groups only. Workspace binding recommended. |
| `OFFICIAL:Sensitive-Personal` | `restricted` or `privileged` | GRANT to PI-authorised groups. Column masking mandatory on PI columns. |
| `OFFICIAL:Sensitive-Commercial` | `restricted` | GRANT to commercial-authorised groups. |
| `OFFICIAL:Sensitive-Legal` | `restricted` | GRANT to legal team only. |
| `OFFICIAL:Sensitive-Cabinet` | `privileged` | PAM-controlled access. Full audit trail. |

---

## 10. Tag Application Principles

### 10.1 Hierarchy Inheritance

Tags should be applied at the **highest applicable level** in the Unity Catalog hierarchy to reduce redundancy:

- If all tables in a schema share the same `source_system`, apply it at **schema level**.
- If all schemas in a catalog share the same `medallion_layer`, apply it at **catalog level**.
- **Column-level tags** (`pi_type`, `masking_required`) are only applied where the tag value differs between columns.

### 10.2 Tag Application by Zone

| Zone | Layer 1 (WAICP) | Layer 2 (Sensitivity) | Layer 3 (Access) | Layer 4 (Operational) |
|------|-----------------|----------------------|------------------|----------------------|
| **Landing** | Not tagged (transient) | Not tagged | Not tagged | `medallion_layer`, `medallion_sublayer`, `source_system` |
| **Raw (Bronze)** | `waicp_classification` (default `OFFICIAL` until classified) | `pi_contained` (provisional assessment), `regulatory_scope`, `sensitivity_type` | `access_model` (overridden by `classification_status` if `unclassified`), `sharing_permitted`, `retention_days` | All Layer 4 tags including `classification_status`, `data_domain` (set to source system identifier), `ingestion_method`, `ingestion_team`, `soci_critical` |
| **Protected (Silver)** | `waicp_classification` (mandatory, typically `OFFICIAL:Sensitive-Personal` or higher) | `pi_contained` + column-level `pi_type`, `regulatory_scope` | `access_model = privileged`, `masking_required` on columns | `medallion_layer`, `medallion_sublayer`, `data_domain` (business domain) |
| **Base (Silver)** | `waicp_classification` (mandatory) | `pi_contained`, `regulatory_scope` | `access_model`, `masking_required` on PI columns, `retention_days` | All Layer 4 tags, `data_domain` (business domain), `classification_status` (must be at least `provisional`) |
| **Enriched (Silver)** | `waicp_classification` (mandatory, highest of contributing sources) | Inherited + extended from contributing sources | Inherited from contributing sources | All Layer 4 tags + `data_product` if applicable |
| **Exploratory (Gold)** | `waicp_classification` (mandatory) | Inherited from Enriched sources | `access_model`, `sharing_permitted` | All Layer 4 tags |
| **BI (Gold)** | `waicp_classification` (mandatory) | `pi_contained` (should be `false` for most BI tables; if `true`, masking must be applied) | `access_model`, `sharing_permitted` | All Layer 4 tags + `refresh_frequency`, `data_product`, `bi_published`, `consuming_domain` (if different from owning domain) |
| **Sandbox** | `waicp_classification = UNOFFICIAL` (for synthetic data) or inherited (for production samples) | `pi_contained` (must be `false` — PI must be masked before Sandbox ingestion) | `access_model = controlled`, `sharing_permitted = internal_only` | `medallion_sublayer = sandbox`, `data_owner` |

### 10.3 Bronze-to-Silver Tag Transition

When data moves from Bronze to Silver, the tagging context shifts from source-oriented to domain-oriented:

| Tag | Bronze Value | Silver Value | Transition Rule |
|-----|-------------|-------------|-----------------|
| `data_domain` | Source system identifier (e.g. `sap_ecc`) | Business domain (e.g. `asset`) | Set by domain steward during classification |
| `classification_status` | `unclassified` (at initial ingestion) | Must be at least `provisional` | Pipeline gate: Silver deployment requires `provisional` or `classified` |
| `waicp_classification` | Default `OFFICIAL` (holding value) | Authoritative classification | Updated by steward during classification assessment |
| `data_owner` | Ingesting team | Domain executive owner | Assigned during domain onboarding |
| `data_steward` | Not set or default | Named domain steward | Assigned during domain onboarding |
| `pi_contained` | Provisional assessment (if known) | Confirmed assessment | Column-level `pi_type` tags must be complete |
| `soci_critical` | Set if source system is known critical infrastructure | Inherited or confirmed | Drives CMK encryption and restricted access |
| `ingestion_method` | Set at ingestion | Not applicable (Silver is transformation, not ingestion) | Retained for lineage; not propagated to Silver |
| `ingestion_team` | Set at ingestion | Not applicable | Retained for lineage; not propagated to Silver |
| `consuming_domain` | Not applicable | Not applicable (set at Gold) | Applied when Gold tables serve a different domain than the owning domain |

### 10.4 Automation

- **Auto-tagging at ingestion:** When a source system is onboarded, the onboarding configuration specifies default Layer 1–4 tags for all tables from that source. These are applied automatically at Bronze Raw Zone ingestion. `classification_status` is automatically set to `unclassified`.
- **Tag propagation:** When a Lakeflow SDP pipeline creates a Silver or Gold table, it should propagate relevant tags from its source tables. Tags that change (e.g. `medallion_sublayer`, `data_domain`) are overwritten; tags that don't change (e.g. `source_system`, `regulatory_scope`) are inherited.
- **Classification SLA monitoring:** A nightly job scans for SLA breaches and raises alerts.
- **CI/CD tag validation:** Pipeline deployments are blocked if mandatory tags are missing (phased enforcement — advisory first, hard enforcement after 3 months).

### 10.5 Lakehouse Federation Tagging

Unity Catalog federated tables (tables that query external data sources in place via JDBC) require tagging guidance distinct from ingested tables:

- Federated tables **must** carry the same mandatory tags as native tables — `waicp_classification`, `data_domain`, `classification_status`, `access_model`, and all applicable Layer 2 sensitivity tags.
- `source_system` is set to the external source identifier (e.g. `sap_ecc_federated`, `maximo_federated`) with a `_federated` suffix to distinguish from ingested copies of the same source.
- `medallion_layer` and `medallion_sublayer` are **not applicable** — federated tables sit outside the medallion architecture. Use `medallion_layer = federated` as a distinct value.
- ABAC policies (column masking, row filters) apply to federated tables in the same way as native tables. However, federated queries are also subject to the source system's native access controls — both layers must be satisfied.
- `classification_status` follows the same lifecycle as native tables. Federated tables must be classified before broad access is granted.
- `retention_days` is **not applicable** to federated tables (the source system manages retention). The tag should be set to `not_applicable`.

### 10.6 Tag Taxonomy Change Management

The tag vocabulary (allowed tag keys and permitted values) is a governed artefact. Changes to the taxonomy follow a defined process:

| Change Type | Examples | Approval Required | Process |
|---|---|---|---|
| **New tag key** | Adding a new governance dimension (e.g. `environmental_impact`) | Data Governance Council | Proposal reviewed at governance taxonomy review (six-monthly cadence). Must include: tag purpose, allowed values, enforcement mechanism, mandatory/optional status, and impacted layers. |
| **New allowed value** | Adding `research_partner` to `sharing_permitted` | Domain steward + platform team | Steward submits change request. Platform team validates UC compatibility. Deployed via tag policy update. |
| **Value deprecation** | Removing an obsolete sensitivity type | Data Governance Council | Deprecation notice issued. Affected tables identified and migrated. Value removed after migration complete. |
| **Tag key retirement** | Removing a tag that is no longer needed | Data Governance Council | Impact assessment across all layers. Migration plan for dependent ABAC policies. Retirement executed only after all dependencies removed. |

**Versioning:** The tag taxonomy is versioned using semantic versioning (e.g. `v1.0.0`). Major version increments indicate breaking changes (removed keys or values); minor versions indicate additions. The current taxonomy version is recorded in the governance register and referenced in CI/CD validation rules.

---

## 11. Column-Level Governance

### 11.1 The Problem

A single table frequently contains columns with different sensitivity levels. An employee table might have name and role (unrestricted) alongside salary and medical information (restricted under PRIS Act). Table-level access control is too coarse — granting access to the table grants access to everything in it.

### 11.2 Approach

Column-level governance in EDAP uses two complementary mechanisms:

**Column-level sensitivity tags.** Every column that contains data above the default (unrestricted) sensitivity level must carry a `pi_type` and/or `masking_required` tag in Unity Catalog. Columns without these tags are assumed to be unrestricted.

**Unity Catalog column masks.** For columns tagged with `masking_required` != `none`, a column mask function is applied in Unity Catalog. The mask function evaluates the requesting user's group membership against the entitlement required for that sensitivity level and returns either the true value or a masked/redacted value.

### 11.3 Masking Function Patterns

**Standard masking functions** should be created as reusable Unity Catalog functions following the `edap_mask_<sensitivity_category>` naming convention:

```sql
-- Direct PI identifier (full mask)
CREATE FUNCTION edap_mask_pi_direct(value STRING)
RETURNS STRING
RETURN CASE
  WHEN is_member('sg-edap-pi-authorised') THEN value
  ELSE '***MASKED***'
END;

-- Indirect PI identifier (partial mask — date to year)
CREATE FUNCTION edap_mask_pi_date_to_year(value DATE)
RETURNS STRING
RETURN CASE
  WHEN is_member('sg-edap-pi-authorised') THEN CAST(value AS STRING)
  ELSE CAST(YEAR(value) AS STRING)
END;

-- Indirect PI identifier (partial mask — postcode generalisation)
CREATE FUNCTION edap_mask_pi_postcode(value STRING)
RETURNS STRING
RETURN CASE
  WHEN is_member('sg-edap-pi-authorised') THEN value
  ELSE CONCAT(LEFT(value, 2), '00')
END;

-- Financial sensitive (full mask)
CREATE FUNCTION edap_mask_financial(value DECIMAL(12,2))
RETURNS DECIMAL(12,2)
RETURN CASE
  WHEN is_member('sg-edap-finance-restricted') THEN value
  ELSE NULL
END;

-- Hash mask (preserves join capability)
CREATE FUNCTION edap_mask_hash(value STRING)
RETURNS STRING
RETURN CASE
  WHEN is_member('sg-edap-pi-authorised') THEN value
  ELSE SHA2(CONCAT(value, 'edap_salt'), 256)
END;
```

> **Warning:** The `'edap_salt'` value above is illustrative only. Production implementations must retrieve the salt from a secret-managed store (e.g. Databricks Secrets backed by Azure Key Vault), not a hardcoded string. A hardcoded salt is vulnerable to exposure through source control, notebook exports, and query history. Use `secret('edap-governance', 'hash-salt')` or an equivalent secret scope reference.

### 11.4 Row-Level Security

Row-level security (Unity Catalog row filters) should be used where access restrictions depend on the *values* within rows rather than column identity:

- Restricting operational data to specific geographic regions or treatment plants
- Limiting financial data visibility to specific cost centres or business units
- Scoping work order data to specific teams or operational areas

**Example:**

```sql
CREATE FUNCTION edap_filter_by_region(region STRING)
RETURNS BOOLEAN
RETURN (
  is_member('sg-edap-all-regions')
  OR (is_member('sg-edap-metro') AND region = 'METRO')
  OR (is_member('sg-edap-northwest') AND region = 'NORTHWEST')
);

ALTER TABLE prod_silver.operations_base.work_orders
  SET ROW FILTER edap_filter_by_region ON (operating_region);
```

### 11.5 Column-Level Governance by Layer

| Layer | Column Masking | Row-Level Security |
|-------|---------------|-------------------|
| **Bronze (Raw)** | Not applied. Bronze tables are protected by table-level access and `classification_status` restrictions. | Not applied. |
| **Silver (Base/Enriched)** | Applied for all columns tagged with `masking_required` != `none`. | Applied where value-based restrictions are required. |
| **Gold (BI/Exploratory)** | Applied. Masks inherited from Silver or reapplied. BI tables should ideally not contain PI columns; where they do, masking is mandatory. | Applied where value-based restrictions carry through to consumption. |
| **Sandbox** | Not applicable — PI must be removed or masked before data enters Sandbox. | Not applicable. |

### 11.6 Sensitive Column Categories

The following categories require sensitivity assessment and potential masking. This table is illustrative — the definitive classification is determined through the WAICP assessment process for each dataset.

| Category | Examples | Typical Sensitivity | Regulatory Driver | Masking Type |
|---|---|---|---|---|
| Personal identifiers | Name, address, phone, email, employee ID | Restricted | PRIS Act 2024 | `full` |
| Financial | Salary, cost, valuation, budget | Restricted | Internal policy | `full` |
| Health & safety | Medical flags, incident details, exposure records | Restricted | PRIS Act 2024, WHS Act | `full` or `redact` |
| Location (fine-grained) | GPS coordinates of critical infrastructure | Restricted | SOCI Act 2018 | `partial` or `full` |
| Operational (critical) | SCADA setpoints, chemical dosing rates | Restricted | SOCI Act 2018 | `full` |
| Indirect identifiers | Date of birth, postcode, job title | Controlled | PRIS Act 2024 | `partial` |
| Reference / descriptive | Asset class, location name, unit of measure | Unrestricted | — | `none` |

---

## 12. Alation Integration

### 12.1 Tag Sync Direction

| Tag Layer | Source of Truth | Sync Direction | Mechanism |
|-----------|----------------|----------------|-----------|
| Layer 1 (WAICP) | Unity Catalog | UC → Alation (read-only) | Alation OCF connector, unified source tags model (2025.1.3+) |
| Layer 2 (Sensitivity) | Unity Catalog | UC → Alation (read-only) | Alation OCF connector |
| Layer 3 (Access) | Unity Catalog | UC → Alation (read-only) | Alation OCF connector |
| Layer 4 (Operational) | Unity Catalog (technical), Alation (business glossary enrichment) | Bidirectional — UC tags → Alation; Alation business descriptions → UC COMMENTs | OCF connector + scheduled Python sync job |

### 12.2 Architectural Relationship: Alation, Unity Catalog, and UC Discover

Alation and Unity Catalog serve complementary but distinct roles in the governance architecture:

- **Alation** is the **enterprise-wide data catalogue** for business users and cross-system discovery. It provides a single pane of glass across Databricks, SAP, Maximo, ESRI, and other systems, enabling business users to discover, understand, and request access to data regardless of where it resides. Alation is the primary interface for business glossary management, stewardship workflows, and cross-platform lineage.

- **Unity Catalog** is the **platform-level governance and access control layer** within Databricks. It is the system of record for governed tags, ABAC policies, column masking, row-level security, and runtime access enforcement within the EDAP. Unity Catalog governs what happens inside Databricks; Alation governs how the organisation discovers and understands data across all systems.

- **UC Discover** (the Unity Catalog Marketplace and discovery interface) complements Alation for **Databricks-native discovery**. Data engineers and analysts working within the Databricks workspace use Discover for in-platform search, data product browsing, and quality signal inspection. Discover does not replace Alation for enterprise-wide discovery, but provides a richer, more integrated experience for users already working within Databricks.

In summary: Alation is the business user's front door to all organisational data; Unity Catalog is the technical enforcement engine within Databricks; UC Discover is the Databricks-native discovery experience that surfaces UC-governed metadata for platform users.

### 12.3 Alation Governance Overlay

Alation provides capabilities that extend beyond UC tagging:

- **Business glossary:** Alation is the source of truth for business term definitions, data steward assignments, and governance policies. These are synced to UC via COMMENT ON statements.
- **Data stewardship workflows:** Access request and approval workflows for `restricted` and `privileged` data are managed in Alation, with enforcement implemented in UC.
- **Cross-system lineage:** Alation provides lineage visibility across Databricks, SAP, Maximo, ESRI, and other systems — extending beyond UC's runtime lineage within Databricks.
- **Classification lifecycle tracking:** Alation can display the `classification_status` progression and provide workflow support for stewards moving objects through the lifecycle.

---

## 13. Worked Examples

### 13.1 Customer Billing Data from SAP ECC

**Table:** `prod_silver.customer_base.customer_billing`

**Layer 1 — WAICP:**
- `waicp_classification` = `OFFICIAL:Sensitive-Personal` (contains customer PI)

**Layer 2 — Sensitivity:**
- `pi_contained` = `true`
- `regulatory_scope` = `pris_act, privacy_act`
- `sensitivity_type` = `personal_information, financial_sensitive`

**Layer 2 — Column-level PI tags:**

| Column | `pi_type` |
|--------|-----------|
| `customer_name` | `direct_identifier` |
| `billing_address` | `direct_identifier` |
| `email_address` | `direct_identifier` |
| `date_of_birth` | `indirect_identifier` |
| `postcode` | `indirect_identifier` |
| `account_balance` | (not PI — no `pi_type` tag) |

**Layer 3 — Access & Handling:**
- `access_model` = `restricted`
- `sharing_permitted` = `internal_only`
- `retention_days` = `2555` (7 years, financial record)
- `encryption_at_rest` = `platform_default`

**Column-level masking:**

| Column | `masking_required` | Mask Function |
|--------|--------------------|----|
| `customer_name` | `full` | `edap_mask_pi_direct` |
| `billing_address` | `full` | `edap_mask_pi_direct` |
| `email_address` | `full` | `edap_mask_pi_direct` |
| `date_of_birth` | `partial` (year only) | `edap_mask_pi_date_to_year` |
| `postcode` | `partial` (first 2 digits) | `edap_mask_pi_postcode` |

**Layer 4 — Operational:**
- `medallion_layer` = `silver`
- `medallion_sublayer` = `base`
- `source_system` = `sap_ecc`
- `data_domain` = `customer`
- `data_owner` = `em_customer_service`
- `data_steward` = `customer_data_steward`
- `data_type` = `structured`
- `refresh_frequency` = `daily`
- `classification_status` = `classified`
- `soci_critical` = `false`

**Resulting UC enforcement:**
- Column masking functions applied to all `direct_identifier` and `indirect_identifier` columns
- Only members of `sg-edap-pi-authorised` group see unmasked values
- Table not included in any Delta Sharing shares
- Retention policy: 7 years, aligned with State Records Act disposal schedule
- Alation: business glossary terms linked, data steward assigned, access request workflow enabled

### 13.2 SCADA Pump Telemetry

**Table:** `prod_bronze.scada_raw.pump_telemetry`

**Layer 1 — WAICP:**
- `waicp_classification` = `OFFICIAL:Sensitive` (critical infrastructure operational data)

**Layer 2 — Sensitivity:**
- `pi_contained` = `false`
- `regulatory_scope` = `soci_act`
- `sensitivity_type` = `operational_security, infrastructure_vulnerability`

**Layer 3 — Access & Handling:**
- `access_model` = `restricted`
- `sharing_permitted` = `internal_only`
- `retention_days` = `indefinite` (operational history for asset analytics)
- `encryption_at_rest` = `customer_managed_key` (SOCI Act risk management)

**Layer 4 — Operational:**
- `medallion_layer` = `bronze`
- `medallion_sublayer` = `raw`
- `source_system` = `scada`
- `data_domain` = `scada` (holding value — will split to `operations` and `water_quality` at Silver)
- `data_owner` = `em_operations`
- `data_steward` = `scada_ops_steward`
- `data_type` = `time_series`
- `refresh_frequency` = `realtime`
- `classification_status` = `classified`
- `soci_critical` = `true`
- `ingestion_method` = `lakeflow_connect`
- `ingestion_team` = `edap_data_engineering`

**Resulting UC enforcement:**
- Schema-level access restricted to SCADA operations and data engineering teams
- Customer-managed KMS key for encryption at rest
- Not included in any Delta Sharing shares
- Workspace binding: production SCADA data only accessible from production workspace

### 13.3 Newly Onboarded Source — Maximo Work Orders (Classification Lifecycle)

This example illustrates the classification lifecycle for a newly onboarded source system.

**Week 0 — Ingestion (Unclassified)**

**Table:** `prod_bronze.maximo_raw.workorder`

- `classification_status` = `unclassified`
- `waicp_classification` = `OFFICIAL` (default holding value)
- `data_domain` = `maximo` (source system holding value)
- `data_owner` = (not yet assigned)
- `data_steward` = (not yet assigned)
- `soci_critical` = `false`
- `ingestion_method` = `lakeflow_connect`
- `ingestion_team` = `edap_data_engineering`
- Access: Only `sg-edap-maximo-engineers` and the interim source system contact can query the table.
- Cross-domain users: Cannot see the table exists.

**Week 3 — Steward Review (Provisionally Classified)**

The domain steward reviews the Maximo work order data and assigns provisional classification:

- `classification_status` = `provisional`
- `waicp_classification` = `OFFICIAL:Sensitive` (contains contractor details, cost data)
- `pi_contained` = `true` (contractor name, employee name columns identified)
- `data_domain` = `maximo` (still at Bronze — domain split will happen at Silver)
- `data_owner` = `em_infrastructure` (Asset Management executive owner nominated)
- `data_steward` = `asset_data_steward`
- Access: All EDAP users can now discover the table (see metadata, description). Querying requires access request.
- Silver pipeline development can begin, targeting `prod_silver.asset_base.work_orders` and `prod_silver.finance_base.work_order_costs`.

**Week 8 — Governance Ratification (Classified)**

The Data Governance Council ratifies the classification. Silver tables are now deployed:

**Table:** `prod_silver.asset_base.work_orders`
- `classification_status` = `classified`
- `waicp_classification` = `OFFICIAL:Sensitive-Personal` (contractor PI)
- `data_domain` = `asset`
- `data_owner` = `em_infrastructure`
- `data_steward` = `asset_data_steward`
- `access_model` = `controlled`
- `soci_critical` = `false`
- Column masking applied to contractor name, employee name columns
- Cross-domain teams (Operations, Finance) can request access through standard entitlement process

**Table:** `prod_silver.finance_base.work_order_costs`
- `classification_status` = `classified`
- `waicp_classification` = `OFFICIAL:Sensitive-Commercial` (cost data)
- `data_domain` = `finance`
- `data_owner` = `cfo`
- `data_steward` = `finance_data_steward`
- `access_model` = `restricted`
- `soci_critical` = `false`
- Access limited to Finance team + approved subscribers

---

## 14. Open Decisions for Workshop

| # | Decision Point | Options | Recommendation |
|---|---------------|---------|----------------|
| 1 | Should `waicp_classification` be a mandatory tag at ingestion, or can it be deferred to Silver? | (a) Mandatory at Bronze Raw Zone (b) Mandatory from Silver Base Zone (c) Mandatory from Gold only | **(a) Mandatory at Bronze** with `OFFICIAL` as default holding value. Updated to authoritative value during classification lifecycle. |
| 2 | Who is authorised to assign/change `waicp_classification` tags? | (a) Data engineers at ingestion (b) Data stewards only (c) Automated based on source system mapping | **(c) Automated with (b) override.** Source system onboarding defines default classification; data stewards can override. |
| 3 | Should the Protected Zone be a physical schema or implemented via UC column masking on Base Zone tables? | (a) Physical schema separation (b) Column masking only (c) Both — physical for highest sensitivity, masking for general PI | **(c) Hybrid.** Sensitive PI (health, TFN) in physical Protected Zone. General PI (name, address) managed via column masking in Base Zone. |
| 4 | Should `infrastructure_vulnerability` data (SCADA) require `customer_managed_key` encryption? | (a) Yes, per SOCI Act risk management (b) No, platform default encryption is sufficient (c) Evaluate per asset criticality | **(a) Yes.** SOCI Act obligations for critical infrastructure data warrant CMK. |
| 5 | How should Aboriginal cultural heritage data be handled? | (a) Treat as `OFFICIAL:Sensitive` with `indigenous_cultural` sensitivity type (b) Create a dedicated classification level (c) Exclude from EDAP entirely | **(a)** with additional controls — restricted access model, mandatory consultation with Aboriginal heritage team before any sharing. |
| 6 | Should tag compliance be enforced (pipeline fails if mandatory tags missing) or advisory (flagged in audit report)? | (a) Hard enforcement (b) Advisory only (c) Phased — advisory first, hard enforcement after 3 months | **(c) Phased.** Start advisory to allow teams to adjust, move to hard enforcement. |
| 7 | How granular should `masking_required` be? Per-column or per-table? | (a) Per-column (maximum flexibility) (b) Per-table (simpler to manage) | **(a) Per-column.** Different columns in the same table may require different masking levels. |
| 8 | What are the appropriate SLAs for classification lifecycle transitions? | (a) 30/60 days as proposed (b) 15/30 days (more aggressive) (c) 60/90 days (more lenient) | **(a) 30/60 days.** Balances governance rigour with delivery reality. Review after 6 months of operation. |
| 9 | Should `unclassified` Bronze data be visible in Alation for discovery? | (a) No — hidden until `provisional` (b) Yes, with clear "unclassified" warning | **(a) No.** Discovery should begin at `provisional` to prevent premature dependency on unclassified data. |
| 10 | How should contested domain ownership be resolved? | (a) Data Governance Council adjudication (b) CIO decision (c) First-to-claim | **(a) Data Governance Council adjudication** as the formal mechanism, with CIO as tiebreaker if Data Governance Council cannot reach consensus. |

---

## 15. Implementation Roadmap

### Phase 1 — Foundation (Weeks 1–4)
- Define and register all tag keys in Unity Catalog (all four layers + `classification_status`)
- Create WAICP classification mapping for all known source systems
- Implement default tag application at Bronze Raw Zone ingestion (including `classification_status = unclassified`)
- Create column masking functions for standard PI types (`edap_mask_*` functions)
- Deploy classification SLA monitoring job

### Phase 2 — Enforcement (Weeks 5–8)
- Apply Layer 1–3 tags to all existing Bronze and Silver tables
- Implement row-level security functions for domain-scoped access
- Configure Alation OCF connector for tag extraction
- Deploy tag compliance audit dashboard in Databricks SQL
- Implement `classification_status`-based access control overrides
- Backfill `classification_status` for existing tables (classify existing assets)

### Phase 3 — Automation (Weeks 9–12)
- Implement automated tag propagation in Lakeflow SDP pipelines
- Enable tag validation in CI/CD (advisory mode)
- Configure Delta Sharing policies driven by `sharing_permitted` tag
- Implement multi-domain lineage tracking in pipeline metadata framework
- Publish tag governance documentation and training materials

### Phase 4 — Hardening (Weeks 13–16)
- Move tag validation from advisory to hard enforcement in CI/CD
- Implement production Gold layer deployment gate (require `classification_status != unclassified`)
- Conduct classification lifecycle retrospective — assess SLA appropriateness
- Extend Alation workflows to support `classification_status` progression
- Publish cross-domain access request self-service portal

---

## Appendix A: Tag Quick Reference

| Tag Key | Layer | Applied To | Required? | Default Value |
|---------|-------|-----------|-----------|--------------|
| `waicp_classification` | 1 | Tables, views, schemas, catalogs | Mandatory (all tables) | `OFFICIAL` |
| `pi_contained` | 2 | Tables, columns | Mandatory (Silver, Gold) | `false` |
| `pi_type` | 2 | Columns | Mandatory (where `pi_contained = true`) | — |
| `regulatory_scope` | 2 | Tables | Mandatory (Silver, Gold) | `none` |
| `sensitivity_type` | 2 | Tables, columns | Mandatory (where sensitive) | — |
| `pi_lawful_basis` | 2 | Tables | Mandatory (where `pi_contained = true`) | `not_assessed` |
| `ai_training_permitted` | 2 | Tables | Recommended | `false` |
| `ai_use_restriction` | 2 | Tables | Recommended | `none` |
| `synthetic_derivation` | 2 | Tables | Recommended (where applicable) | `false` |
| `access_model` | 3 | Tables, schemas | Mandatory (Silver, Gold) | `controlled` |
| `masking_required` | 3 | Columns | Mandatory (where `pi_type` set) | `none` |
| `sharing_permitted` | 3 | Tables, schemas | Mandatory (Gold) | `internal_only` |
| `encryption_at_rest` | 3 | Tables, schemas, catalogs | Mandatory (SOCI-scoped) | `platform_default` |
| `retention_days` | 3 | Tables | Mandatory (Silver, Gold) | — |
| `contract_version` | 3 | Tables | Recommended (where contract defined) | — |
| `contract_sla_tier` | 3 | Tables | Recommended (where contract defined) | — |
| `breaking_change_policy` | 3 | Tables | Recommended (where contract defined) | `none` |
| `medallion_layer` | 4 | Catalogs, schemas | Mandatory | — |
| `medallion_sublayer` | 4 | Schemas | Mandatory | — |
| `source_system` | 4 | Schemas, tables | Mandatory | — |
| `data_domain` | 4 | Schemas, tables | Mandatory | Source system name (Bronze) |
| `data_owner` | 4 | Schemas, tables | Mandatory (Silver, Gold) | — |
| `data_steward` | 4 | Schemas, tables | Mandatory (Silver, Gold) | — |
| `data_type` | 4 | Tables | Recommended | — |
| `refresh_frequency` | 4 | Tables | Mandatory (Gold) | — |
| `data_product` | 4 | Tables | Optional | — |
| `soci_critical` | 4 | Schemas, tables | Recommended (SOCI-scoped) | `false` |
| `consuming_domain` | 4 | Tables (Gold) | Recommended (cross-domain Gold) | — |
| `ingestion_method` | 4 | Tables (Bronze) | Mandatory | — |
| `ingestion_team` | 4 | Tables (Bronze) | Mandatory | — |
| `quality_tier` | 4 | Tables | Recommended (Silver, Gold) | `uncertified` |
| `classification_status` | 4 | Tables, schemas | Mandatory (all tables) | `unclassified` |
| `bi_published` | 4 | Tables | Mandatory (Gold BI) | `false` |

---

## Appendix B: Seven Data Domains

The `data_domain` tag values align with the seven enterprise data domains defined in the Water Corporation Information Management Framework:

| Domain | `data_domain` Value | Executive Owner | Primary Systems |
|--------|-------------------|-----------------|-----------------|
| Customer | `customer` | Executive Manager, Customer Service | CRM, Billing, Service Request |
| Asset | `asset` | Executive Manager, Infrastructure | GIS, CMMS, Asset Register, Engineering DMS |
| Operations | `operations` | Executive Manager, Operations | SCADA, LIMS, Work Management, Field Mobility |
| Finance | `finance` | Chief Financial Officer | ERP, Procurement, Budget, Tariff Management |
| Legal & Compliance | `legal_compliance` | General Counsel | Contract Management, Compliance, Records Management |
| People | `people` | Executive Manager, People & Culture | HRIS, Learning Management, Workforce Planning |
| Technology & Digital | `technology_digital` | Chief Information Officer | Application Portfolio, Data Catalogue, Integration Platform |
| Enterprise Reference | `enterprise_reference` | Enterprise Data Steward (CIO delegate) | Cross-domain reference data: org hierarchy, location master, calendar |

---

## Appendix C: Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | March 2026 | Architecture & Strategy | Initial draft — four-layer tagging model |
| 0.2 | March 2026 | Architecture & Strategy | Added classification lifecycle (Section 8), cross-domain access model, multi-domain source system handling (Section 7.4–7.6), column-level governance (Section 11), worked example for classification lifecycle (Section 13.3), additional decision points (8–10), Phase 4 implementation roadmap |
| 0.3 | March 2026 | Architecture & Strategy | Added `pi_lawful_basis` tag (Section 5.2.4) for consent and lawful basis tracking. Added `quality_tier` tag (Section 7.2) for data product quality signals. Added AI governance standards references (ISO/IEC 42001:2023, NIST AI RMF, Voluntary AI Safety Standard) to Section 5.2.5. Added SOCI 2024 Rules amendments reference. Added Privacy Act reform note for automated decision-making. Added Lakehouse Federation tagging guidance (Section 10.5). Added tag taxonomy change management process (Section 10.6). Added Delta Sharing external governance cross-reference. Updated Appendix A with new tags. |

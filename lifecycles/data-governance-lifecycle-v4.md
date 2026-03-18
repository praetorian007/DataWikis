# The Data Governance Lifecycle

## Enterprise Data & Analytics Platform (EDAP)

**Mark Shaw** | Principal Data Architect

---

## Introduction

Data governance is the framework of policies, processes, roles, and standards that ensures data is managed as a strategic enterprise asset: discoverable, understandable, trustworthy, secure, and used responsibly. It is not a project with a start and end date, nor a compliance checkbox exercise; it is a continuously operating discipline that enables and protects every other data activity in the organisation.

This document outlines the stages, roles, operating model, and contemporary practices involved in the data governance lifecycle as it applies to Water Corporation's Enterprise Data & Analytics Platform (EDAP). While the principles described here have broad applicability, the governance model, classification framework, role definitions, and platform-specific guidance are designed specifically for the EDAP context: a Databricks lakehouse architecture with Unity Catalog as the governance hub, Alation as the enterprise discovery layer, a medallion (Bronze/Silver/Gold) data organisation model with zone decomposition (Landing, Raw, Processed, Base, Protected, Enriched, Exploratory, BI, Sandbox), and a federated domain ownership structure aligned to SAFe Agile delivery.

This document should be read alongside the **Medallion Architecture** document, which defines the zone structure, Unity Catalog namespace conventions, data quality framework, audit column standards, and pipeline development patterns that this governance lifecycle governs over. Where the Medallion Architecture describes *how data flows and is structured*, this document describes *how data is governed, classified, owned, and protected* as it moves through those structures.

> **Naming convention note:** The programme is referred to as **EDAP** (Enterprise Data & Analytics Platform) throughout this document and in programme-level governance artefacts. The Unity Catalog namespace prefix is **`edp_`** (Enterprise Data Platform), as defined in the Medallion Architecture document (e.g. `edp_bronze`, `edp_silver`, `edp_gold`, `edp_sandbox`). Similarly, audit columns use the `edp_` prefix (e.g. `edp_batch`, `edp_hash`) and data quality annotation columns use the `dq_` prefix (e.g. `dq_status`, `dq_flags`). These namespace and column conventions are technical standards governed by this lifecycle and defined in the Medallion Architecture document.

This document is designed to complement the Data Engineering Lifecycle, the Business Intelligence Lifecycle, and the Data Science Lifecycle. Governance is the connective tissue that binds them. Where data engineering is concerned with getting data *right*, BI with getting data *used*, and data science with getting data to *learn*, governance is concerned with ensuring data is *trusted, understood, and managed responsibly* across all three.

The framework presented here draws on established practices from the DAMA-DMBOK2 (Data Management Body of Knowledge), adapted and extended for a modern enterprise operating in a lakehouse architecture, federated domain ownership model, and regulatory environment shaped by the SOCI Act 2018, PRIS Act 2024, WA Information Classification Policy (WAICP), State Records Act 2000, and Essential Eight cybersecurity standards. It also reflects the reality that in 2026, governance must serve not only human data consumers but also AI agents, LLMs, and automated systems that discover, interpret, and act on enterprise data autonomously.

The lifecycle is not strictly linear. Classification informs access control, which informs cataloguing, which feeds back into classification. Quality management runs through every stage. The value of the framework lies in providing a shared mental model for how we plan, implement, and operate governance, regardless of whether the organisation is governing ten datasets or ten thousand.

**Why this matters now:** The governance landscape in 2026 is being reshaped by three converging forces. First, the shift from centralised, command-and-control governance to **federated governance**, where domain teams own their data products and governance decisions are pushed as close to the data as possible, with a central team providing standards, tooling, and oversight rather than gatekeeping. Second, the rise of **AI consumers** that require machine-readable governance metadata (classification, quality scores, lineage, semantic context) to function safely and effectively. Governance that exists only in documents and spreadsheets is invisible to an AI agent. Third, the maturation of **platform-native governance** capabilities (Unity Catalog, automated policy enforcement, data contracts) that make it possible to encode governance as code rather than relying on manual processes and good intentions.

---

## What is Data Governance?

Data governance is the exercise of authority, control, and shared decision-making over the management and use of data assets. It defines who can make what decisions about data, using what processes, guided by what standards, and measured against what outcomes.

In practice, this means governance establishes the rules of the road: who owns which data, how it is classified and protected, what quality standards it must meet, how it is documented and discovered, who can access it and under what conditions, how long it is retained, and how disputes about data definitions and usage are resolved. Governance does not build pipelines or create dashboards. It creates the conditions under which pipelines and dashboards produce trustworthy results.

The role of governance has matured significantly. A modern governance programme is expected to be **enabling rather than constraining**, reducing friction for data consumers while maintaining the controls necessary for regulatory compliance, security, and trust. The best governance programmes are invisible to consumers when things are working well: data is discoverable, access is granted efficiently, quality is consistently high, and classifications are accurate. Governance only becomes visible when something goes wrong â and good governance ensures that happens less often.

The shift from "governance as bureaucracy" to "governance as platform capability" is well underway. In 2026, the most effective governance programmes are those that encode policies as automated, enforceable rules within the data platform itself (data contracts, attribute-based access control, automated classification propagation, quality assertions embedded in pipelines) rather than relying on manual review processes, approval chains, and governance boards that meet monthly to rubber-stamp decisions already made.

The guiding philosophy is **"open by default, restricted by exception."** Data should be accessible, discoverable, and usable unless there is a specific, documented reason to restrict it. The burden of proof lies with restriction, not access. This philosophy accelerates data-driven decision-making while maintaining the precision controls needed for sensitive, regulated, and security-critical data.

---

## Relationship to Other Lifecycles

Data governance does not operate in isolation. It is a cross-cutting discipline that underpins and enables every stage of every other data lifecycle:

**Data Governance â Data Engineering Lifecycle:** Governance defines the policies, classifications, quality standards, and access controls that data engineering implements. Data engineering builds the pipelines and the EDAP platform; governance defines the guardrails within which those pipelines operate. The interface is formalised through **data contracts** (enforcing schema, quality, and SLA commitments), **Unity Catalog policies** (enforcing access control and classification), and **data product governance tiers** (defining the level of rigour required for each data product). Without governance, data engineering operates in a policy vacuum; without engineering, governance is unenforceable theory.

**Data Governance â BI Lifecycle:** Governance ensures that the metrics, dimensions, and data products consumed by BI are defined consistently, classified correctly, and accessed appropriately. The semantic layer and business glossary, both essential BI infrastructure, are governance artefacts maintained by stewards and consumed by analysts. Governance prevents the proliferation of conflicting metric definitions ("which revenue number is right?") by establishing single sources of truth.

**Data Governance â Data Science Lifecycle:** Governance defines the ethical, privacy, and compliance boundaries within which data science operates: what data can be used for model training, how personal information must be handled, how model outputs are classified and governed as new data products, and how bias and fairness considerations are embedded in the data supply chain. Feature stores and ML training datasets are governed data products. **ML model outputs (predictions, scores, recommendations) are themselves data products** and must be governed accordingly: they require sensitivity classification (model outputs derived from PI-classified inputs inherit that classification), domain ownership (assigned to the domain that owns the use case, not the data science team), data contracts (defining schema, freshness, and accuracy commitments to downstream consumers), and quality monitoring (model drift detection, prediction accuracy tracking). Models registered in Unity Catalog are governed assets with lineage tracing back to their training data, enabling compliance auditing of the full data-to-prediction chain. The Data Science Lifecycle document provides detailed treatment of model governance; this governance lifecycle establishes the framework within which that model governance operates.

**The Shared Foundation:** All four lifecycles share a common data platform (the lakehouse), a common governance framework (Unity Catalog, data contracts, data quality standards), and common cross-cutting concerns (security, compliance, metadata). The **Medallion Architecture** document defines the platform foundation that all four lifecycles operate on: the zone structure (Landing through BI), the Unity Catalog namespace conventions, the audit column and DQ annotation standards, the data quality framework ("Propagate, Don't Drop"), and the pipeline development patterns. This governance lifecycle governs *over* that platform foundation â it does not redefine it. Where the Medallion Architecture makes a platform design decision (e.g. zone boundaries, namespace conventions, DQ column standards), this governance lifecycle defines the governance controls, roles, and policies that operate within that design. The key principle is **separation of concerns with tight collaboration**: governance sets the rules, engineering implements them, BI and data science operate within them, and all four lifecycles feed back into continuous governance improvement.

---

## Governance Operating Model

### Federated Governance: The Architectural Decision

The EDAP governance operating model is **federated by design**. This is a deliberate architectural decision, not an absence of centralisation. In a federated model, governance responsibility is distributed across the organisation, with domain teams owning the governance of their data products and a central governance function providing the standards, tooling, platform capabilities, and oversight that ensure consistency and compliance across domains.

This is distinct from both **centralised governance** (where a central team makes all governance decisions, creating bottlenecks and disconnection from domain knowledge) and **decentralised governance** (where each domain operates independently, creating inconsistency and compliance risk). Federated governance occupies the pragmatic middle ground: central standards, domain execution.

**Why federated?** The people closest to the data understand it best. A domain data steward in Asset Management understands the semantics, quality characteristics, and business rules of asset hierarchy data far better than a central governance team ever could. But that domain steward needs a consistent framework of standards, classifications, and tooling to operate within. Without that framework, every domain invents its own approach and cross-domain consistency collapses. Federated governance provides both: domain autonomy within central guardrails.

**The Three Layers:**

- **Enterprise Governance.** Sets organisation-wide policies, standards, classification schemes, and regulatory compliance requirements. Operates through the Data Governance Council and Chief Data Officer. Owns the governance framework, the business glossary, and the enterprise data domains. Makes decisions that affect the entire organisation.
- **Domain Governance.** Each data domain (e.g. Asset, Customer, Financial, Workforce, Operational, Spatial, Regulatory) has a Domain Data Owner (executive sponsor) and Domain Data Steward(s) who govern the data products within their domain. They define domain-specific business rules, quality thresholds, and semantic definitions within the enterprise framework. They make decisions about their domain's data products.
- **Data Product Governance.** Individual data products are governed according to their assigned governance tier (Enterprise, Domain, or Exploratory), with progressively more rigorous controls for higher-tier products. Data product owners and technical stewards manage day-to-day governance at this level.

---

### Governance Tiers

Not all data products warrant the same level of governance rigour. Over-governing exploratory data stifles innovation; under-governing enterprise-critical data creates unacceptable risk. The governance tier model applies proportional controls:

**Tier 1: Enterprise Data Products.** Cross-domain, business-critical data products consumed by multiple domains, used in regulatory reporting, or feeding AI/ML systems. Examples: conformed customer dimension, enterprise asset hierarchy, financial fact tables. These require full data contracts, formal change management, executive ownership, CMDB registration, published SLOs, and comprehensive documentation. Changes are governed through ADRs and cross-domain impact analysis.

**Tier 2: Domain Data Products.** Domain-specific data products consumed within a single domain or by a small number of known consumers. Examples: maintenance work order analytics, HR workforce metrics. These require data contracts with their consumers, domain steward oversight, quality assertions, and catalog documentation. Change management is domain-led with lighter-weight processes.

**Tier 3: Exploratory / Sandbox Data Products.** Experimental, analyst-created, or prototype data products not yet promoted to domain or enterprise status. These require basic metadata (owner, description, creation date) and sensitivity classification, but are exempt from formal data contracts, SLOs, and change management processes. Exploratory products that prove valuable are promoted to Tier 2 or Tier 1 through a defined promotion process that adds the required governance controls.

**The Five Gates Test:** A data product is ready for promotion when it is **Findable** (catalogued and discoverable), **Accessible** (consumers can access it through governed channels), **Understandable** (documented with clear semantics), **Quality-assured** (automated quality checks in place), and **Dependable** (SLOs defined and met). This test applies at promotion from Tier 3 to Tier 2 and from Tier 2 to Tier 1, with increasing stringency at each level.

---

## Core Stages

### 1. Discovery and Classification

Discovery and classification is the starting point of the governance lifecycle. You cannot govern what you cannot see, and you cannot protect what you have not classified. This stage establishes the inventory of data assets across the EDAP and assigns the classifications that determine how each asset is handled, protected, and accessed throughout its lifecycle.

**Data Discovery:**

Data discovery is the process of identifying, inventorying, and registering data assets across the enterprise. Within the EDAP, the primary discovery mechanism is the data catalog, comprising **Unity Catalog** (as the authoritative governance hub for all assets within the Databricks platform) and **Alation** (for cross-system lineage, business glossary, and discovery across systems beyond the lakehouse).

Discovery is not a one-time exercise. New data sources, new pipelines, new data products, and new AI-generated datasets are created continuously. The governance programme must ensure that every new data asset is registered in the catalog at the point of creation, not retrospectively as a cleanup exercise. The principle is "register at birth": catalog registration is a step in the pipeline, not a separate governance process.

**Automated Discovery:**

Manual discovery does not scale. Modern governance leverages automated discovery mechanisms: Unity Catalog's automatic schema detection and registration, crawler-based discovery for data sources outside the lakehouse, and metadata harvesting from source systems. The goal is to minimise the gap between when a data asset is created and when it appears in the catalog with accurate metadata.

**Data Classification:**

Classification assigns sensitivity, regulatory, and handling attributes to data assets. It determines access control policies, encryption requirements, retention rules, and audit obligations. Classification is not optional; unclassified data is ungoverned data.

**The Classification Model:**

The EDAP Tagging Strategy defines a four-layer tagging model for data platform governance. Three of those layers are true *classification* layers that assign attributes that determine how data is handled, protected, and accessed. These layers apply from the moment data enters the platform:

- **Layer 1: Sensitivity Classification** (WAICP-aligned). Covers UNOFFICIAL, OFFICIAL, OFFICIAL:SENSITIVE (with sublabels for Personal, Commercial, Legal, and Cabinet). Determines the baseline handling, access, and protection requirements. Mandatory for all data assets. This layer implements the Western Australian Information Classification Policy as the non-negotiable corporate baseline.
- **Layer 2: Sensitivity Reason.** Explains *why* data is sensitive, which is what drives specific technical controls. Includes PII indicators (`pii_contained`, `pii_type` at column level), regulatory scope (PRIS Act, SOCI Act, State Records Act), and granular sensitivity types (personal information, infrastructure vulnerability, commercial in confidence, indigenous cultural, and others). A single data asset may carry multiple sensitivity reasons simultaneously. This layer answers the question WAICP cannot: "sensitive *because of what*?"
- **Layer 3: Access and Handling.** Translates the classification and sensitivity reason into concrete Unity Catalog enforcement actions: access model (open, controlled, restricted, privileged), column masking requirements, sharing permissions, encryption at rest, and retention periods. This is the layer where classification decisions become platform-enforced controls.

A fourth layer, **Layer 4: Data Management**, carries operational metadata that supports platform operations and discoverability rather than security classification. This layer includes the `source_system` tag (provenance marker identifying the originating system), medallion zone indicator (aligned to the EDAP's zone decomposition: Landing, Raw, Processed, Base, Protected, Enriched, Exploratory, BI), refresh frequency, ingestion method, data product identifier, and related operational attributes. Layer 4 tags align with the Unity Catalog namespace conventions defined in the Medallion Architecture document (e.g. `edp_bronze.<source_system>`, `edp_silver.<source_system|subject_area>`, `edp_gold.<domain>`).

**Separating Provenance from Domain Ownership:**

A critical design principle is that **provenance and domain ownership are fundamentally different concerns and must be expressed as separate tags.** Source system boundaries and business domain boundaries are different things. SAP ECC contains financial data, asset data, HR data, and procurement data. A SCADA historian covers both water quality telemetry (potentially public health sensitive, owned by Operations or a Water Quality domain) and pump performance telemetry (operational, owned by Asset Management or Operations). Maximo work orders span Asset Management (the asset being maintained), Operations (the crew doing the work), and Finance (the cost centre being charged).

The EDAP tagging model therefore uses two distinct tags to avoid semantic overloading:

- **`source_system`** (Layer 4: Data Management). Applied at ingestion. Identifies the originating system (e.g. `sap_ecc`, `scada`, `maximo`, `gis`). This is a permanent provenance marker that never changes. It answers the question "where did this data come from?" and is consistently meaningful at every medallion layer. The `source_system` tag is essential for impact analysis (when a source system schema changes, every table originating from that system is immediately identifiable), for lineage tracing, and for ingestion pipeline management. It is not a governance accountability assignment; it carries no implication of business ownership.

- **`data_domain`** (Governance relationship). Applied when business domain ownership is established, which in the EDAP's medallion architecture occurs at the **Silver Enriched Zone**, not Bronze or Silver Base. Values are business domains (e.g. `asset_management`, `customer`, `financial`, `workforce`, `operational`, `spatial`, `regulatory`). This tag carries governance weight: it determines which Domain Data Steward is accountable, which domain-level access policies apply, and which governance review cadence the data product falls under. The `data_domain` tag is not present at Bronze or Silver Base (or is explicitly set to `unassigned` where schema completeness requires a value). This is a single, unambiguous rule: domain ownership is assigned at the zone where business meaning is established and the Unity Catalog namespace transitions from source-system-aligned to subject-area-aligned.

This separation ensures that each tag has a single, unambiguous meaning across all medallion layers. An ABAC policy referencing `data_domain` will never accidentally match Bronze tables that have not yet had ownership assigned. A lineage query can join `source_system` and `data_domain` to answer "which source systems feed which business domains?" without semantic confusion. AI agents querying governance metadata receive consistent, machine-interpretable signals regardless of which layer they are inspecting.

**Where Domain Ownership Sits in the Medallion Architecture:**

Domain ownership is not a classification attribute stamped on a dataset at the point of ingestion. It is a governance relationship that is assigned at the point where business meaning is established, which in the EDAP's medallion architecture is the **Silver Enriched Zone** â the zone where the Unity Catalog namespace transitions from source-system-aligned to subject-area-aligned. This is a deliberate design decision informed by both data mesh best practice and the practical reality of how source systems map (or fail to map) to business domains.

Data mesh literature reinforces this distinction. Zhamak Dehghani's original framework classifies domains as source-aligned, consumer-aligned, or aggregate. In a medallion lakehouse, these map naturally: Bronze is source-aligned (organised by source system, not business domain), Silver is where data is conformed and domain meaning is established, and Gold is consumer-aligned (optimised for specific analytical or operational use cases that may cross domain boundaries). The governance model therefore operates differently at each layer:

- **Bronze (Landing / Raw / Processed Zones).** Tables are organised by source system in the Unity Catalog namespace (`edp_bronze.<source_system>.<table>`). The `source_system` tag identifies provenance. The `data_domain` tag is not assigned (or is set to `unassigned`). Governance accountability for sensitivity classification, security, and access control is held by the **Technical Data Steward** responsible for the ingestion pipeline, not by a Domain Data Steward. The Technical Data Steward ensures the data is classified for sensitivity (Layers 1 through 3), that mandatory tags are applied, and that the `classification_status` lifecycle is initiated, but they are not making business domain ownership decisions. Sensitivity classification must never be deferred: you classify for security before you understand full business context.
- **Silver Base Zone.** The Base Zone cleanses, deduplicates, and quality-annotates source data, but its Unity Catalog schemas remain **source-system-aligned** (`edp_silver.<source_system>.base_<entity>`, e.g. `edp_silver.sap.base_maintenance_order`). This is a critical nuance: although Silver Base Zone data is cleansed and trusted, the namespace and governance organisation still reflect the source system origin, not the business domain. The `data_domain` tag is **not assigned** at this zone; domain ownership is deferred to the Enriched Zone where business context is established through integration and enrichment. The **Technical Data Steward** retains primary governance accountability at this zone, with the **Domain Data Steward** consulted on business rule validation and quality rule definition. This clean boundary â Technical Data Steward governs Base, Domain Data Steward governs Enriched â eliminates ambiguity about who is accountable for what, and ensures that a catalog query for `data_domain = asset_management` returns a complete and consistent set of domain-owned tables.
- **Silver Enriched Zone.** This is where domain ownership is definitively established. The Enriched Zone integrates Base Zone data from multiple source systems, applies business logic, and organises data by **subject area or business concept** (`edp_silver.<subject_area>.enr_<concept>`, e.g. `edp_silver.assets.enr_asset_health`). The pipeline that transforms Base into Enriched assigns the `data_domain` tag based on the business context of the resulting table. A single Base Zone source may produce Enriched tables in multiple domains; this is the multi-domain source system split in action. The **Domain Data Steward** takes full accountability at this zone: they confirm the domain assignment, validate the business metadata, and own the quality and classification of the Enriched data product. This is where governance transitions from "we know what system this came from and the data is clean" to "we know what this data means in business terms and who is accountable for it."
- **Silver Protected Zone.** Personal Information (PI) and other protected data classified as OFFICIAL:SENSITIVE or higher is managed in the Protected Zone with Privileged Access Management (PAM) controls. Access is provided only when necessary and revoked automatically after task completion. The Protected Zone may produce synthetic identifiers or masked derivatives that are used in Base, Enriched, and Gold zones in place of the original PI. The `data_domain` tag is assigned based on the domain accountability for the underlying personal information (e.g. `customer` for customer PI). The **Domain Data Steward** and **Data Security Specialist** share governance accountability at this zone.
- **Gold (Exploratory / BI Zones).** Gold schemas are organised by **business domain** (`edp_gold.<domain>.<table>`, e.g. `edp_gold.operations.bi_fact_work_order`). The `data_domain` tag typically inherits from the Silver Enriched source. Where a Gold table joins data from multiple Silver domains, it is assigned to the *primary owning domain* â the domain with the strongest accountability for the accuracy, quality, and governance of the resulting data product. Contributing source domains are recorded via a `contributing_domains` tag or in lineage metadata to preserve the full provenance chain. Where a Gold data product serves a cross-functional purpose (e.g. an executive dashboard joining Asset, Financial, and Operational data), it may be assigned to a `cross_domain` value with a nominated data product owner who is accountable for the composite product's governance, quality, and access control. The key principle is that every Gold data product has exactly one accountable owner, even when it draws from multiple domains.
- **Sandbox.** The Sandbox layer (`edp_sandbox.<user_or_team>.<table>`) provides isolated experimentation environments. Sandbox data products are Tier 3 (Exploratory) by default and require basic metadata (owner, description, creation date) and sensitivity classification, but are exempt from formal data contracts and SLOs. Governance must be vigilant for **shadow pipelines** â unofficial transformations built in Sandbox that become load-bearing without governance. A formal promotion path from Sandbox to production zones (with the corresponding uplift in governance controls) must be established and enforced.

**The implication for the classification model is significant.** Sensitivity classification (Layers 1 through 3) applies from the moment of ingestion: you must classify data for sensitivity before you understand its full business context, because you cannot defer security controls. But domain ownership is established progressively as data moves through the medallion zones and business meaning crystallises â from source-system-aligned schemas at Bronze and Silver Base, through subject-area-aligned schemas at Silver Enriched, to domain-aligned schemas at Gold. These are different governance activities, performed by different roles, at different points in the data lifecycle. Expressing them as separate tags (`source_system` for provenance, `data_domain` for ownership) makes this distinction explicit, machine-readable, and enforceable. The Unity Catalog namespace itself reinforces this progression: `edp_bronze.sap.*` â `edp_silver.sap.base_*` â `edp_silver.assets.enr_*` â `edp_gold.operations.bi_*`.

**Could Domain Classification Happen at Field Level?**

An alternative approach would be to classify domain ownership at the column or field level rather than the table level, recognising that a single source table may contain columns belonging to different business domains. For example, a Maximo work order table contains asset identifiers (Asset Management domain), cost centre codes (Finance domain), crew assignments (Workforce domain), and completion dates (Operations domain).

While intellectually appealing, field-level domain classification introduces significant practical complexity: it multiplies the governance surface area, creates ambiguity about table-level ownership (who owns a table where four domains each claim columns?), and does not align with how Unity Catalog enforces access control (which operates primarily at table and column level for security, not for domain ownership). The pragmatic approach, and the one the EDAP tagging strategy adopts, is **table-level domain ownership with single-owner accountability**: one domain owns the table, is accountable for its quality and governance, and other domains subscribe as consumers. The ownership assignment is based on which domain has the strongest accountability for the data's accuracy and currency. A work order table is owned by Asset Management (accountable for work order data quality) even though Operations, Finance, and Workforce all consume it.

Enterprise reference data that is genuinely cross-cutting (organisational hierarchy, location master, calendar, unit of measure) is assigned to `data_domain = enterprise_reference` with a nominated enterprise data steward, avoiding the orphan problem where no single business domain claims ownership.

**Classification Lifecycle:**

Classification is not a one-time label. Data assets move through a three-state classification lifecycle (tracked by the mandatory `classification_status` tag):

- **Unclassified.** Newly ingested data that has not yet been reviewed by a steward. Sensitivity classification defaults are applied (WAICP defaults to OFFICIAL as a holding value), and the access posture is **restrictive by default**: only the ingesting domain team can see or query the data. This inverts the platform's normal "open by default" philosophy during the classification window, because you cannot apply "open by default" to data whose sensitivity you do not yet understand. This is one of the most important governance design decisions in the model: the "open by default" principle presupposes that classification has been completed and the data is understood. Unclassified data does not meet that precondition and must therefore be restricted until it does. Objects must transition out of Unclassified within 30 calendar days.
- **Provisional.** Initial classification assigned by the relevant steward (Technical Data Steward for Bronze sensitivity classification; Domain Data Steward for Silver domain and business classification). The data becomes discoverable across the platform (metadata visible to all EDAP users), but query access requires an explicit request. Provisional classification must progress to Classified within 60 calendar days.
- **Classified.** Full WAICP classification ratified, domain ownership confirmed, executive endorsement in place, access policies enacted in Unity Catalog. The standard "open by default, restricted by exception" philosophy applies. Data classified as unrestricted is broadly accessible; data classified as restricted is governed by the ABAC policies defined in Layer 3 tags.

**Classification Lifecycle Enforcement:**

The classification lifecycle SLAs (30-day Unclassified window, 60-day Provisional window) are enforced through automated monitoring, not manual tracking. An automated daily scan identifies objects approaching or exceeding their classification window. When an object reaches 80% of its allowed window (day 24 for Unclassified, day 48 for Provisional), an automated alert is sent to the responsible steward. When the window is breached, the following escalation sequence applies: (1) the object is flagged in the data catalog with a `classification_overdue` tag visible to all catalog users, (2) an escalation notification is sent to the Data Governance Manager, (3) if unresolved within 14 days of the breach, the object is escalated to the Data Governance Council agenda and the Domain Data Owner is notified. Objects that remain Unclassified beyond 60 days (double the allowed window) are quarantined: access is revoked for all users except the responsible steward and Data Governance Manager, and the object cannot be consumed by downstream pipelines until classification is completed. This enforcement mechanism ensures that classification SLAs have teeth, not just aspirations.

**Multi-Domain Source System Handling:**

When a single source system feeds multiple business domains, the split occurs between Silver Base and Silver Enriched. The Base Zone table remains source-system-aligned (e.g. `edp_silver.maximo.base_work_order`), while the Enriched Zone produces separate tables in domain-specific subject area schemas (e.g. `edp_silver.assets.enr_asset_health`, `edp_silver.operations.enr_crew_utilisation`). The pipeline metadata framework captures the lineage: source Bronze table (identified by `source_system`), Silver Base table, target Enriched table, column-level mapping, target domain (`data_domain`), and business rationale for the domain assignment. This lineage is critical for impact analysis. When a source system schema changes, every downstream Base and Enriched consumer must be identifiable through the `source_system` tag and Unity Catalog's automated lineage. The Technical Data Steward manages this lineage; the Domain Data Stewards on the receiving end are accountable for validating that the domain assignment and transformation logic are correct.

**Domain Ownership Dispute Resolution:**

Where domain ownership is contested (a table that could reasonably belong to multiple domains) ownership is assigned to the domain with the strongest accountability for the data's accuracy and currency. If agreement cannot be reached between Domain Data Stewards, the Data Governance Council adjudicates. The decision is recorded in the data catalog and is binding until formally reviewed.

**Guidance:**

- Register all data assets in the catalog at the point of creation ("register at birth"). Do not tolerate shadow data assets that exist outside the catalog.
- Apply sensitivity classification (Layers 1 through 3) at the point of ingestion. This is a Technical Data Steward responsibility. Do not wait for domain ownership decisions before classifying data for sensitivity; security cannot be deferred.
- Use the `source_system` tag (Layer 4) at Bronze to record provenance. Do not assign the `data_domain` tag at Bronze or Silver Base; domain ownership is not meaningful when schemas are source-system-aligned.
- Assign the `data_domain` tag at the Silver Enriched Zone, where business context is established through integration and enrichment and the Unity Catalog namespace transitions to subject-area-aligned schemas. Domain Data Stewards take full accountability when data enters their subject area at this zone. This is a single, unambiguous rule â not a judgement call that varies by source system.
- Apply provisional classification automatically at ingestion using rule-based classification (column naming patterns, source system sensitivity mapping, content scanning for personal information).
- Ensure every data product has a confirmed classification before promotion from Tier 3 to Tier 2.
- Review classifications periodically (at minimum annually for Tier 1 and Tier 2 products) and whenever a material change occurs.
- Maintain classification metadata as structured, machine-readable tags in Unity Catalog, not as unstructured text in a wiki or spreadsheet. AI agents and automated policy enforcement depend on structured classification metadata.
- Use the `edp_` prefixed audit columns (as defined in the Medallion Architecture document) to track classification state, classification date, and classifying steward for every governed data asset.
- Enforce single-owner accountability at table level. One domain owns, many domains subscribe. Do not allow shared or ambiguous ownership; it creates accountability gaps.

**Key Roles:** Technical Data Steward (Bronze and Silver Base Zone sensitivity classification, automated classification implementation, catalog registration, multi-domain lineage management, quarantine path monitoring), Domain Data Steward (Silver Enriched/Gold domain ownership confirmation, business classification review, quality rule definition, Protected Zone PI classification), Data Governance Manager (classification policy and standards, dispute escalation, classification lifecycle SLA enforcement), Data Engineer (pipeline-level classification propagation and tag application, DQ annotation column implementation).

---

### 2. Policy and Standards Definition

Policy and standards definition establishes the authoritative rules, principles, and expectations that govern how data is managed across the EDAP and the broader enterprise. Policies set the "what" and "why"; standards set the "how." Without clear, communicated, and enforceable policies, governance devolves into ad-hoc decision-making and inconsistent practices.

**Policy Hierarchy:**

- **Enterprise Data Governance Policy.** The overarching policy that establishes the governance mandate, operating model (federated), roles and accountabilities, and the relationship between governance and regulatory compliance. Approved by executive leadership. Reviewed annually.
- **Domain-Specific Policies.** Policies that apply within specific data domains, addressing domain-specific business rules, quality thresholds, and handling requirements. Examples: the Asset Data Policy (defining standards for asset hierarchy data, naming conventions, and Maximo/SAP integration rules), the Customer Data Policy (defining personal information handling requirements under the PRIS Act).
- **Technical Standards.** Detailed, prescriptive standards that define how policies are implemented on the platform. Examples: Unity Catalog naming conventions (catalog-per-layer pattern: `edp_bronze`, `edp_silver`, `edp_gold`, `edp_sandbox`; schema-per-source-or-domain; zone prefixes: `raw_`, `std_`, `base_`, `enr_`, `exp_`, `bi_`), `edp_` audit column standards (`edp_batch`, `edp_hash`, `edp_eff_from`, `edp_eff_to`, `edp_is_current`, `edp_is_deleted`), `dq_` quality annotation columns (`dq_status`, `dq_flags`, `dq_checked_ts`), tagging taxonomy for sensitivity and regulatory classification, data contract YAML schema specifications.
- **Operational Procedures.** Step-by-step procedures for recurring governance activities: how to request access to a data product, how to raise a data quality issue, how to promote a data product from Tier 3 to Tier 2, how to conduct a classification review.

**Policy as Code:**

In 2026, the most effective governance programmes encode policies as executable, enforceable rules within the data platform rather than relying solely on written documents. This is the "governance as code" principle:

- **Access control policies** encoded as ABAC rules in Unity Catalog, enforced automatically at query time. Access decisions are based on user attributes (role, domain membership, security clearance) and data attributes (sensitivity classification, domain, regulatory tags), replacing manual approval workflows.
- **Data quality expectations** encoded as assertions in pipeline code (Lakeflow Spark Declarative Pipelines expectations, dbt tests, Great Expectations), enforced at every pipeline run. Quality is a pipeline gate, not a retrospective audit.
- **Data contracts** encoded as versioned YAML specifications in source control, validated in CI/CD. Schema changes that violate a downstream contract break the build.
- **Retention policies** encoded as automated lifecycle rules that archive or delete data when retention periods expire, with audit trails.
- **Classification propagation rules** encoded as tag inheritance logic. When a Bronze table is classified as OFFICIAL:SENSITIVE and tagged with PRIS Act, those classifications automatically propagate to derived Silver and Gold assets unless explicitly overridden with justification. The governing principle is **"inherit the most restrictive classification from any contributing source."** When a Silver Enriched or Gold table joins data from multiple sources with different sensitivity classifications, the resulting table inherits the highest sensitivity level and the union of all regulatory tags from its contributing sources. A derived table may be *more* restrictive than any individual source (because the combination of datasets may create new sensitivity â e.g. joining anonymised data with a lookup table that enables re-identification). A derived table may never be *less* restrictive than its most sensitive source without explicit justification, Data Governance Manager approval, and an auditable record of the downgrade rationale.

**Governance-as-Code Deployment:**

Governance-as-code artefacts (ABAC policy definitions, Unity Catalog tag configurations, quality assertion code, data contract YAML specifications, classification propagation rules) are deployed through the same CI/CD pipeline as platform code, using **Databricks Asset Bundles (DABs)** as the infrastructure-as-code framework. This ensures that governance changes are version-controlled, peer-reviewed, tested, and deployed through a controlled pipeline â not applied manually through the Unity Catalog UI or ad-hoc scripts. DABs package governance configurations alongside pipeline code, so that a data product's governance controls and its pipeline logic are always deployed together and cannot drift apart. The principle is: if a governance control is not in the DAB, it does not exist in production.

**Guidance:**

- Write policies for humans; encode standards as code for machines. Policies should be clear, concise, and understandable by non-technical stakeholders. Standards should be precise, executable, and enforceable by the platform.
- Version all policies and standards in source control alongside the platform code. If it is not in Git, it does not exist.
- Avoid policy bloat. Every policy should have a clear purpose, a named owner, and a review cycle. Policies that no one reads or enforces are worse than no policy; they create a false sense of governance.
- Ensure policies are aligned to regulatory requirements (SOCI Act, PRIS Act, WAICP, State Records Act, Essential Eight) and map specific policy clauses to specific regulatory obligations to demonstrate compliance traceability.
- Publish policies and standards in a discoverable location (Confluence, data catalog) and actively communicate changes to affected stakeholders.

**Key Roles:** Data Governance Manager (policy drafting and lifecycle), Chief Data Officer (policy approval and executive sponsorship), Domain Data Owner (domain policy input and endorsement), Data Architect (technical standards and alignment with EDAP platform architecture), Legal / Compliance (regulatory alignment review).

---

### 3. Data Contracts

Data contracts formalise the agreement between data producers and data consumers. They are the governance mechanism that makes data product commitments explicit, measurable, and enforceable. Without data contracts, governance commitments exist only as implicit expectations â and implicit expectations are the leading cause of broken dashboards, failed pipelines, and eroded trust.

**What a Data Contract Contains:**

A data contract is a versioned, machine-readable specification (typically YAML, stored in source control) that defines the commitments a data product makes to its consumers. A complete data contract includes:

- **Schema specification.** The exact columns, data types, nullability constraints, and valid value ranges that the data product guarantees. Schema changes that violate the contract are blocked in CI/CD before they reach production.
- **Quality assertions.** The specific quality checks that must pass before the data product is considered ready for consumption (e.g. null rate on `customer_id` must be 0%, row count within Â±10% of previous day, `transaction_date` within the last 90 days).
- **Freshness SLO.** The maximum acceptable age of the data (e.g. data must be no more than 4 hours old). Monitored continuously with automated alerting on SLO breaches.
- **Semantic definitions.** Business descriptions for each column, linked to the business glossary where applicable. These descriptions serve both human consumers and AI agents.
- **Ownership and accountability.** The Domain Data Steward accountable for the data product, the Technical Data Steward responsible for implementation, and the escalation path for issues.
- **Consumer registry.** The known consumers (downstream pipelines, dashboards, models, applications) that depend on this data product. Enables impact analysis when contract changes are proposed.
- **Change management protocol.** How changes to the contract are proposed, reviewed, communicated, and implemented. Distinguishes between backwards-compatible changes (additive columns, relaxed constraints) and breaking changes (column removal, type changes, tightened constraints) with different approval processes for each.
- **Classification and access.** The sensitivity classification, regulatory tags, and access model that apply to the data product, referencing the Layers 1â3 classification.

**Contract Lifecycle:**

Data contracts follow a defined lifecycle: **Draft** (proposed by the data product owner, reviewed by consumers) â **Active** (ratified by all parties, enforced in CI/CD and pipelines) â **Deprecated** (consumers notified, migration period begins) â **Retired** (contract and data product removed after all consumers have migrated). Breaking changes to an active contract trigger a formal change management process that includes consumer impact assessment, migration planning, and a defined transition period.

**Contract Enforcement:**

Contracts are enforced at two points: in the **CI/CD pipeline** (schema and quality specification validation before deployment) and in the **data pipeline** (quality assertion execution at runtime). A data product that fails its contract assertions is quarantined and not published to consumers until the issue is resolved. Contract compliance is tracked as a governance KPI.

**Guidance:**

- Require data contracts for all Tier 1 data products. Strongly recommend them for Tier 2. Tier 3 products are exempt until promotion.
- Store data contracts in source control alongside pipeline code. Treat contracts as code: versioned, reviewed, and tested.
- Include both the producer's obligations (schema, quality, freshness) and the consumer's responsibilities (appropriate use, access justification, feedback on quality issues).
- Automate contract validation in CI/CD. A pipeline that produces a data product without a valid contract should not pass code review.
- Review active contracts quarterly to ensure they remain accurate and that SLOs are being met.

**Key Roles:** Domain Data Steward (contract definition, consumer negotiation, change approval), Technical Data Steward (contract implementation, CI/CD enforcement, SLO monitoring), Data Engineer (contract validation in pipeline code), Data Governance Manager (contract standards, compliance reporting).

---

### 4. Data Quality Management

Data quality is the continuous discipline of ensuring that data is accurate, complete, timely, consistent, and valid â that is, fit for the purposes for which it is consumed. Quality failures erode trust, cause incorrect decisions, break downstream systems, and in a regulated environment, can result in compliance breaches. Quality management is not a stage that occurs once; it is embedded in every stage of the data engineering lifecycle and governed as a core governance discipline within the EDAP.

**Quality Dimensions:**

- **Accuracy.** Data correctly represents the real-world entity or event it describes. An asset's GPS coordinates actually correspond to the asset's physical location.
- **Completeness.** All required data is present; no unexpected gaps. Every work order has an associated asset ID.
- **Timeliness (Freshness).** Data is available when needed and reflects a current-enough state for its use case. The daily financial position is available by 7:00 AM.
- **Consistency.** Data conforms to agreed definitions, formats, and business rules across systems and domains. "Customer" means the same thing in the billing system as in the CRM.
- **Validity.** Data conforms to defined schemas, ranges, and business constraints. Dates are valid dates; status codes are from the allowed enumeration.
- **Uniqueness.** No unintended duplicates exist. Each customer has exactly one master record.

**Quality Enforcement Points:**

Quality is enforced at zone boundaries throughout the pipeline, aligned to the EDAP's medallion architecture and the **"Propagate, Don't Drop"** principle established in the Medallion Architecture document. This principle distinguishes between two fundamentally different categories of quality issue:

- **Structural / technical failures** (unparseable records, binary corruption, missing mandatory system fields) are routed to the **quarantine path** alongside the Processed Zone table. These records cannot be processed and must be quarantined to prevent pipeline failure.
- **Business data quality issues** (invalid codes, unexpected nulls, referential integrity violations, out-of-range values) **must propagate** through all zones. Dropping them silently creates invisible data loss, broken referential integrity, and false trust. The correct handling is to carry the record forward with quality annotations (`dq_status`, `dq_flags`, `dq_checked_ts` columns) so that all data is represented where expected and business users can see and act on quality problems.

Quality enforcement is applied at each zone boundary:

- **Processed Zone (Bronze).** Structural schema validation â correct data types, required fields present. Records that cannot be parsed or fail structural validation are routed to the quarantine path. This is the only zone where records are removed from the pipeline, and only for structural failures.
- **Base Zone (Silver).** Business rule validation â null checks, invalid code detection, referential integrity, cross-field consistency. All records propagate regardless of business DQ status; quality issues are annotated in `dq_status` / `dq_flags` columns, not filtered. DQ annotation columns are introduced at this zone and carried forward through all downstream zones.
- **Enriched Zone (Silver).** Join completeness, calculation validity, enrichment-specific quality dimensions. Unresolvable joins and incomplete enrichments are flagged in the DQ columns. Records continue to propagate.
- **Gold Zones (Exploratory / BI).** Product-specific quality gates. Where a specific data product has a documented requirement to exclude records that fail defined quality rules (e.g. a BI table excludes assets with no location assigned), this exclusion is an explicit, documented business decision applied at this zone only â not a general pipeline behaviour. SLA compliance checks (freshness, completeness), metric reconciliation, and data contract validation are enforced here as the final quality gate before data reaches consumers.

**Three Complementary DQ Observability Layers:**

The EDAP provides DQ observability at three granularities, all of which are needed â they are complementary, not alternatives:

- **Row-level.** Physical `dq_status`, `dq_flags`, and `dq_checked_ts` columns in the table, telling you which specific rules each record violated.
- **Pipeline/batch level.** Lakeflow SDP `expect` expectations via `system.pipelines.event_log`, telling you how many records failed each rule in each batch.
- **Table/trend level.** Data Profiling (Databricks native), providing column statistics over time â null rates, distributions, drift, freshness anomalies.

Lakeflow SDP `expect` does not add columns to the target table; it only records aggregate pass/fail counts in the pipeline event log. Physical DQ annotation columns are therefore required for row-level visibility and are computed as part of the transformation logic, not by `expect`.

**Data Quality Assertions and Data Contracts:**

Quality expectations are encoded as assertions within pipeline code, not as separate, retrospective processes. Every data product's data contract includes a quality section that specifies the assertions that must pass before the product is considered "ready" for consumption. These assertions are enforced in the pipeline. If a business quality assertion fails, the pipeline annotates the affected records with DQ flags (consistent with the "Propagate, Don't Drop" principle) and raises an alert to the responsible steward. If the failure exceeds a defined severity threshold specified in the data contract (e.g. more than 5% of records failing a critical assertion), the pipeline flags the data product as **degraded** in the catalog and withholds the refresh from consumers until the steward triages the issue. The data product's quality score is updated to reflect the degradation. Structural failures that prevent processing are handled separately via the quarantine path as defined in the Medallion Architecture document.

**Data Observability:**

Data observability extends quality management from rule-based assertions to anomaly-based detection. Observability tools monitor data assets for unexpected changes (volume shifts, schema drift, distribution anomalies, freshness degradation) and alert stewards before consumers are affected. The goal is to detect quality issues proactively, not reactively after a dashboard breaks or a report is wrong.

**Quality Remediation:**

When quality issues are detected, the governance framework defines the remediation process:

1. **Detection.** Automated assertion failure or observability alert.
2. **Triage.** Technical Data Steward assesses severity and scope. Is this a pipeline issue, a source system issue, or a business rule change?
3. **Root Cause Analysis.** Use data lineage to trace the issue to its origin. Where did the bad data come from?
4. **Remediation.** Fix the root cause. This may involve correcting source data, adjusting pipeline logic, updating business rules, or flagging the data product as degraded with a quality advisory.
5. **Verification.** Re-run quality assertions to confirm the fix. Update the data product's quality score.
6. **Prevention.** Add or strengthen assertions to prevent recurrence. Update data contracts if the quality specification was insufficient.

**Quality Scoring and Reporting:**

Each governed data product maintains a composite quality score derived from its assertion pass rates, freshness compliance, and observability health. Quality scores are published in the data catalog and are machine-readable, enabling AI agents to assess data trustworthiness programmatically. Quality dashboards provide Domain Data Stewards with visibility into quality trends across their domain, and the Data Governance Council with enterprise-wide quality posture.

**Guidance:**

- Embed quality assertions in pipelines, not in separate audit processes. Quality is a pipeline gate.
- Enforce the **"Propagate, Don't Drop"** principle: business DQ failures are annotated, not filtered. Only structural failures that prevent processing are quarantined. Silent data loss is a governance failure.
- Implement physical DQ annotation columns (`dq_status`, `dq_flags`, `dq_checked_ts`) at the Silver Base Zone and carry them forward through all downstream zones. Do not rely solely on Lakeflow SDP `expect` for row-level DQ visibility â `expect` records aggregate counts in the event log but does not annotate individual rows.
- Define quality thresholds in data contracts. Make quality expectations explicit, measurable, and enforceable.
- Where a Gold data product excludes records based on quality rules, the exclusion must be an explicit, documented business decision â not an implicit pipeline behaviour.
- Push quality accountability as far upstream as possible. Engage source system owners in quality improvement.
- Use data observability (Data Profiling) to detect anomalies that rule-based assertions would miss â distribution drift, null rate changes, volume shifts, freshness degradation.
- Maintain quality scores for all Tier 1 and Tier 2 data products. Publish scores in the catalog.
- Conduct periodic quality reviews with Domain Data Stewards to identify systemic issues and improvement opportunities.
- Monitor quarantine path file growth actively; sustained growth indicates a systemic source system issue requiring investigation and remediation.
- Quarantine bad data rather than publishing it. A delayed data product is better than a wrong one.

**Key Roles:** Domain Data Steward (quality rule definition, business rule validation, remediation oversight), Technical Data Steward (quality assertion implementation, observability configuration, automated monitoring), Data Engineer (pipeline-level quality enforcement), Data Governance Manager (quality policy, reporting, and escalation).

---

### 5. Metadata Management and Cataloguing

Metadata is the governance system's nervous system. It carries the information that makes data discoverable, understandable, trustworthy, and governable. Without rich, accurate, maintained metadata, governance is blind. In 2026, metadata must serve both human consumers (who need business context and documentation) and machine consumers (AI agents, automated policy enforcement, and programmatic discovery). This means metadata must be structured, machine-readable, and semantically rich, not just free-text descriptions in a wiki.

**Metadata Categories:**

- **Business Metadata.** Human-readable context that makes data understandable: business definitions, descriptions, semantic relationships, domain assignment, data product documentation, and the business glossary. Maintained by Domain Data Stewards. Consumed by analysts, business users, and AI agents performing semantic discovery.
- **Technical Metadata.** Schema definitions, column types, table statistics, partition structures, file formats, and physical storage locations. Largely auto-harvested from the platform. Consumed by data engineers, the platform, and automated tooling.
- **Operational Metadata.** Pipeline execution status, freshness timestamps, quality scores, data product SLO compliance, cost attribution, and lineage provenance. Generated by the data platform and DataOps tooling. Consumed by stewards, SRE practices, and data observability systems.
- **Governance Metadata.** Classification tags (sensitivity, regulatory, domain, business value), ownership assignments, access policy references, retention rules, and data contract status. The metadata that governance directly produces and consumes.
- **Semantic Metadata.** Rich, machine-readable descriptions of meaning, relationships, and context: ontologies, knowledge graph relationships, embedding vectors, and semantic type annotations. Increasingly critical as AI agents need to discover and reason about data without human mediation. This is the frontier of metadata management in 2026.

**The Data Catalog Architecture:**

The EDAP catalog architecture uses a two-system model:

- **Unity Catalog** serves as the authoritative governance hub for all assets within the Databricks lakehouse. Manages schemas, tables, volumes, models, functions, and connections through a three-level namespace (`<catalog>.<schema>.<table>`) that encodes layer membership in the catalog name and zone membership in the table prefix. Enforces ABAC access control, hosts classification tags, manages data lineage within the platform (including automatic column-level lineage), and serves as the runtime policy enforcement point. Unity Catalog Volumes extend governance to unstructured file storage (Landing Zone files, raw extracts). Unity Catalog is the "source of truth" for technical and governance metadata within the lakehouse.
- **Alation** serves as the enterprise discovery and collaboration layer. Provides cross-system lineage (spanning source systems, the lakehouse, BI tools, and downstream consumers), the business glossary, curation workflows, and a discovery interface for business users. Alation integrates with Unity Catalog, harvesting its metadata and enriching it with business context that Unity Catalog does not natively manage.

The principle is clear: **Unity Catalog governs; Alation discovers.** Technical governance decisions (access control, classification enforcement, lineage within the platform) are Unity Catalog's responsibility. Business discovery, cross-system visibility, and glossary management are Alation's responsibility. There should be no ambiguity about which system is authoritative for which metadata type.

**Business Glossary:**

The business glossary is the authoritative definition of business terms, metrics, and concepts used across the enterprise. It resolves the perennial problem of conflicting definitions ("what do we mean by 'active customer'?") by establishing a single, governed definition for each term. Glossary terms are linked to the data assets that implement them, creating a traceable chain from business concept to physical data.

**Business Glossary Lifecycle:**

The business glossary is itself a governed asset with a defined lifecycle for term management:

- **Proposed.** A Domain Data Steward proposes a new term or a revision to an existing term, providing the definition, domain scope, related terms, and the data assets that implement the term. For domain-specific terms (e.g. "asset criticality rating"), the proposing Domain Data Steward's domain owns the definition. For cross-domain terms (e.g. "active customer," "revenue"), the proposal is flagged for cross-domain review.
- **Reviewed.** Domain-specific terms are reviewed and approved by the relevant Domain Data Owner. Cross-domain terms are reviewed by all affected Domain Data Stewards and ratified by the Data Governance Council. Conflicting definitions are resolved through the Council's dispute resolution process; the Council's decision is binding and recorded.
- **Published.** The approved definition is published in Alation, linked to implementing data assets in Unity Catalog, and communicated to affected stakeholders. The definition becomes the authoritative reference for all data products and reports that use the term.
- **Deprecated / Retired.** When a term is superseded or no longer relevant, it is marked as deprecated with a pointer to the replacement term. After a transition period, deprecated terms are retired. Retired terms remain in the glossary history for audit purposes but are no longer linked to active data assets.

Glossary terms are versioned: when a definition changes, the previous version is retained for traceability. Glossary health (percentage of Tier 1 and Tier 2 data products with linked glossary terms, number of undefined or conflicting terms) is tracked as a governance KPI.

**Active Metadata:**

The trend toward active metadata (metadata that drives automation rather than sitting passively in a catalog) is accelerating. Examples:

- Classification tags that automatically trigger ABAC policies (a table tagged `PRIS_ACT` automatically restricts access to authorised roles).
- Quality scores that drive data product health indicators in the catalog.
- Freshness metadata that triggers observability alerts when a data product is stale.
- Semantic metadata that enables AI agents to discover and evaluate data products programmatically.

**Guidance:**

- Maintain metadata as structured, machine-readable attributes in Unity Catalog and Alation, not as unstructured text in wikis, spreadsheets, or email threads.
- Ensure every Tier 1 and Tier 2 data product has complete business metadata: description, domain assignment, ownership, business glossary term linkage, and quality score.
- Automate technical and operational metadata harvesting. Manual metadata entry does not scale.
- Invest in semantic metadata for data products that will be consumed by AI agents or RAG systems.
- Treat the business glossary as a governed, living artefact. Review and update definitions as business context evolves.
- Enforce the principle that Unity Catalog is the governance hub and Alation is the discovery layer. Do not create metadata governance ambiguity by duplicating authoritative metadata across systems.

**Key Roles:** Domain Data Steward (business metadata authoring, glossary term definition, domain-level catalog curation), Technical Data Steward (catalog configuration, metadata harvesting automation, tag management, lineage validation), Data Governance Manager (metadata standards, glossary governance process), Data Architect (catalog architecture, metadata model design).

---

### 6. Access Control and Security Governance

Access control governance determines who can access what data, under what conditions, and with what audit trail. It operationalises the classification decisions made in Stage 1 and the policies defined in Stage 2 into enforceable, auditable access controls within the EDAP. Security governance ensures that data is protected throughout its lifecycle: at rest, in transit, and in use.

**The Guiding Principle: Open by Default, Restricted by Exception.**

Data should be accessible to any authenticated, authorised user unless a specific, documented restriction applies. This inverts the traditional model where data is locked down by default and access requires navigating approval chains. The rationale is pragmatic: most enterprise data is not sensitive, and restricting access to non-sensitive data creates friction that slows decision-making without improving security. The governance effort is focused on precisely identifying and tightly controlling the data that genuinely requires restriction (personal information, security-critical data, commercially sensitive data) rather than applying blanket restrictions that impede legitimate use.

**Important caveat:** The "open by default" principle applies only to data that has completed the classification lifecycle (status: Classified). Data in Unclassified or Provisional status is restricted by default until classification is confirmed, as described in the Classification Lifecycle section.

**Attribute-Based Access Control (ABAC):**

The EDAP access control model is ABAC-centric, enforced through Unity Catalog. Access decisions are made dynamically based on the intersection of user attributes (role, domain membership, security clearance, project assignment) and data attributes (sensitivity classification, regulatory tags, domain classification). This is more granular and adaptive than traditional role-based access control (RBAC) alone, though RBAC groups serve as one input into ABAC policy evaluation.

**Access Control Enforcement Layers:**

- **Catalog-Level.** Controls which catalogs (layers) a user can see and access. Enforced by Unity Catalog catalog-level grants. The EDAP's catalog-per-layer pattern (`edp_bronze`, `edp_silver`, `edp_gold`, `edp_sandbox`) provides layer-level isolation as the first access boundary.
- **Schema-Level.** Controls access to schemas within a catalog. In Bronze and Silver Base, schemas are source-system-aligned; in Silver Enriched and Gold, schemas are domain/subject-area-aligned. Domain-aligned schemas are accessible to domain members by default; cross-domain access is governed by data sharing agreements.
- **Table-Level.** Controls access to individual tables and views. The primary enforcement point for most access policies.
- **Column-Level.** Masks or redacts sensitive columns (e.g. PI fields) for users who have table-level access but are not authorised for the specific sensitive attributes. Implemented via Unity Catalog column masks and dynamic views.
- **Row-Level.** Filters rows based on user attributes (e.g. a regional manager sees only their region's data). Implemented via Unity Catalog row-level security or parameterised views.

**Protected Zone (Silver):**

The Silver Protected Zone provides an additional physical isolation layer for Personal Information (PI) and other data classified as OFFICIAL:SENSITIVE or higher. Access is controlled via **Privileged Access Management (PAM)**: access is granted only when necessary and revoked automatically after task completion. Access to specific subject areas within the Protected Zone is controlled by role (e.g. only users with the `customer_pi_reader` role can access customer PI). The Protected Zone may produce synthetic identifiers or masked derivatives used in other zones in place of original PI, reducing the governance surface area for sensitive data. Unity Catalog column masking and row-level security should be evaluated as complementary mechanisms to reduce data duplication while maintaining access control.

**Per-Layer Compute Isolation:**

For enterprise and regulated environments, the EDAP deploys separate Lakeflow Jobs per layer, each running under a dedicated service principal with least-privilege access: Bronze service principals can read from source and write to Bronze only; Silver service principals can read from Bronze and write to Silver only; Gold service principals can read from Silver and write to Gold only. This ensures that a compromise in one layer cannot propagate to others and creates clear separation of duties for audit purposes.

**Access Request and Provisioning:**

Access provisioning follows a tiered process aligned to data sensitivity:

- **UNOFFICIAL / OFFICIAL data.** Self-service access via domain membership. If a user is a member of a data domain group, they receive access to that domain's non-sensitive data automatically. No approval workflow required.
- **OFFICIAL:SENSITIVE data.** Requires explicit approval from the Domain Data Steward or Domain Data Owner. Approval is logged, time-bounded (access expires and must be renewed), and auditable.
- **Higher classifications.** Requires approval from the Data Governance Manager and, where applicable, the Information Security team. Access is strictly need-to-know, time-bounded, and subject to enhanced audit.

**Data Sharing Governance:**

Data sharing, both internal (across domains) and external (with partners, regulators, or other agencies), is governed by formal data sharing agreements that specify: what data is shared, at what classification level, for what purpose, for how long, under what security controls, and with what audit obligations. Delta Sharing and cloud-native sharing protocols enable secure, governed sharing without data duplication.

**Audit and Monitoring:**

All data access is logged in Unity Catalog's audit trail. Access patterns are monitored for anomalies (excessive access, unusual query patterns, access to data outside a user's normal domain) that may indicate security threats or policy violations. Audit logs are retained in accordance with the State Records Act and are available for compliance reporting and incident investigation.

**Guidance:**

- Implement ABAC as the primary access control model, using classification tags as policy inputs. Avoid manual, per-user access grants wherever possible.
- Default to open: grant domain-level access to non-sensitive data automatically. Reserve approval workflows for genuinely sensitive data.
- Enforce column-level masking for PII and other sensitive fields. Do not rely on users to self-regulate access to sensitive columns within tables they can query.
- Time-bound all elevated access grants. Access to sensitive data should expire automatically and require renewal with re-justification.
- Monitor access patterns continuously. Use audit logs for both compliance reporting and proactive threat detection.
- Govern data sharing through formal agreements. No data leaves the platform without a documented, approved sharing arrangement.

**Key Roles:** Data Governance Manager (access policy definition, sharing agreement oversight), Domain Data Steward (domain-level access approval, access review), Technical Data Steward (ABAC policy implementation, Unity Catalog configuration, audit log analysis), Data Security Specialist (security architecture, threat monitoring, incident response), Data Engineer (column/row-level security implementation in pipelines and views).

---

### 7. Data Lifecycle Management

Data lifecycle management governs the progression of data from creation through active use, archival, and eventual deletion or permanent retention. It ensures that data is retained for as long as it has business value or regulatory obligation, and no longer. Retaining data beyond its useful life increases storage costs, expands the attack surface, and creates compliance risk (particularly under the PRIS Act's data minimisation principles). Deleting data prematurely destroys business value and may violate records retention obligations.

**Retention Policy Framework:**

Retention policies define how long data must be retained, in what state, and when it must be archived or deleted. Policies are driven by three factors:

- **Regulatory Requirements.** The State Records Act 2000, PRIS Act 2024, SOCI Act 2018, and sector-specific regulations mandate minimum retention periods for specific data categories. These are non-negotiable floors.
- **Business Value.** Data that continues to inform business decisions, train models, or serve operational needs has retention value beyond regulatory minimums. Business value is assessed by the Domain Data Owner.
- **Cost.** Storage is not free, and data that has no regulatory or business justification for retention should be archived (moved to cold/archive tiers) or deleted. FinOps practices ensure that retention decisions consider cost.

**Lifecycle States:**

- **Active.** Data is in regular use, stored in hot/warm storage tiers, and subject to full governance controls (quality monitoring, access control, freshness SLOs).
- **Archived.** Data is no longer in regular use but must be retained for regulatory compliance or potential future need. Moved to cold/archive storage tiers to reduce cost. Access is restricted and requires explicit justification. Quality monitoring is reduced but classification and access controls remain in force.
- **Pending Deletion.** Data has passed its retention period and is scheduled for deletion. A grace period allows for objection or reclassification before permanent removal.
- **Deleted.** Data has been permanently and verifiably removed from all storage locations, including backups, snapshots, and associated metadata, in compliance with regulatory requirements. Deletion is logged and auditable.

**Zone-Specific Retention Considerations:**

Retention policies interact with the medallion zone structure. The Bronze Raw Zone is immutable and append-only by design, serving as the system of record for all ingested data; its retention periods are typically the longest, driven by SOCI Act and State Records Act requirements. The Bronze Landing Zone is transient and cleared after successful ingestion. Silver and Gold zones have retention periods driven by business value and regulatory requirements specific to each data product. Delta Lake's time travel capability provides short-term replay without separate archival infrastructure, but time travel retention windows (controlled by `VACUUM`) must be aligned with governance retention policies. For PRIS Act compliance, deletion must be propagated through all zones using Delta Lake deletion vectors and Change Data Feed (CDF), and must be verifiable â covering not just the primary table but also derived data, cached copies, and metadata that could re-identify individuals.

**Compliance-Driven Deletion:**

Under the PRIS Act 2024, personal information must be deleted when the purpose for which it was collected has been fulfilled and no other retention obligation applies. This requires the governance programme to maintain a clear mapping between personal information holdings, the purposes for which they are held, and the applicable retention periods. Deletion must be verifiable: not just the data files but also derived data, cached copies, and metadata that could re-identify individuals.

**Guidance:**

- Define retention policies for every data domain, aligned to regulatory requirements and business value assessments.
- Automate lifecycle transitions (active â archived â deleted) using platform lifecycle rules. Do not rely on manual cleanup.
- Ensure deletion is verifiable and complete, including derived data, snapshots, time travel history, and associated metadata.
- Maintain a retention schedule that maps data categories to regulatory obligations, retention periods, and responsible owners.
- Review retention policies annually, or when regulatory requirements change.
- Balance retention with data minimisation. The default should be to retain only what is needed, not to retain everything indefinitely.

**Key Roles:** Data Governance Manager (retention policy framework, compliance oversight), Domain Data Owner (business value assessment, retention period endorsement), Domain Data Steward (retention policy implementation within domain), Technical Data Steward (automated lifecycle rule configuration, deletion verification), Legal / Compliance (regulatory retention requirement interpretation).

---

### 8. Compliance and Regulatory Governance

Compliance governance ensures that the organisation's data management practices meet all applicable legal, regulatory, and policy obligations. It is not a separate governance function but a cross-cutting concern embedded in every other stage: classification, policy, quality, access control, retention, and metadata all have compliance dimensions. This stage consolidates the compliance perspective and provides the framework for demonstrating compliance to regulators, auditors, and executive leadership.

**Regulatory Landscape:**

- **SOCI Act 2018 (Security of Critical Infrastructure).** Defines obligations for critical infrastructure entities, including risk management programmes, reporting obligations for cyber incidents, and government-directed actions. Directly relevant as a water utility classified as a critical infrastructure asset. Governance must ensure that data systems supporting critical infrastructure operations meet SOCI requirements. The EDAP's immutable Bronze Raw Zone serves as an audit-compliant data retention mechanism; Unity Catalog audit logging provides the forensic trail; and the Silver Protected Zone isolates sensitive operational data.
- **PRIS Act 2024 (Privacy and Responsible Information Sharing).** Replaces the Freedom of Information Act 1992 as the primary WA privacy legislation. Establishes Information Privacy Principles (IPPs) governing the collection, use, disclosure, storage, and disposal of personal information (PI; note: "personal information" per the PRIS Act, not "PII" which is a different legislative term). Governance must ensure PI is identified, classified, handled in accordance with IPPs, and deleted when no longer required. The Silver Protected Zone with PAM-based access, Unity Catalog column masking for PI fields, and Delta Lake deletion vectors with CDF propagation for right-to-be-forgotten operations are the primary architecture controls.
- **WA Information Classification Policy (WAICP).** Mandates classification of government information according to sensitivity levels. Directly drives the sensitivity layer of the four-layer classification model.
- **State Records Act 2000.** Establishes obligations for the creation, management, and disposal of government records. Data that constitutes a "record" under the Act must be managed in accordance with approved retention and disposal schedules. Governance must ensure that data lifecycle management aligns with records management obligations. The append-only Bronze Raw and Processed zones provide immutable records; configurable retention periods per zone ensure compliance with disposal schedules.
- **Essential Eight.** The Australian Signals Directorate's cybersecurity mitigation strategies. Governance must ensure that data platform security controls (access control, patching, multi-factor authentication, backups, application control) meet the organisation's target maturity level. The EDAP's per-layer service principal isolation, Unity Catalog RBAC/ABAC, and Databricks Asset Bundles for controlled deployments support Essential Eight compliance.

**Compliance Evidence and Traceability:**

The governance programme must be able to demonstrate compliance, not just assert it. This requires:

- **Policy-to-regulation mapping.** Every governance policy clause traces to the specific regulatory requirement it addresses.
- **Classification-to-control mapping.** Every classification level has a documented set of controls (access, encryption, retention, audit) and those controls are verifiable.
- **Audit trails.** All access, changes, and governance decisions are logged and retrievable.
- **Data lineage.** The ability to trace any data product from its serving point back to its original source, through every transformation, to demonstrate data provenance and processing compliance.
- **Quality evidence.** Quality assertion results, quality scores, and remediation records demonstrate that data products meet the standards required for regulatory reporting.

**Privacy Impact Assessments:**

For new data initiatives, systems, or data products that involve personal information, a Privacy Impact Assessment (PIA) is conducted as part of the governance approval process. The PIA identifies privacy risks, assesses their likelihood and impact, and defines mitigations. PIAs are required before personal information is collected, used in a new way, or shared with a new consumer.

**Guidance:**

- Maintain a regulatory obligations register that maps each regulation to the specific governance controls that address it.
- Ensure classification and access control implementations are auditable and traceable to policy.
- Conduct regular compliance reviews (at minimum annually, more frequently for Tier 1 data products).
- Use the PRIS Act terminology ("personal information" / "PI") rather than the colloquial "PII" to ensure legal precision.
- Embed compliance checks in automated processes wherever possible. Compliance that depends on manual diligence will eventually fail.
- Conduct Privacy Impact Assessments for all new data initiatives involving personal information.

**Key Roles:** Data Governance Manager (compliance programme management, regulatory mapping, audit coordination), Legal / Compliance (regulatory interpretation, PIA review), Domain Data Owner (compliance accountability within domain), Domain Data Steward (compliance evidence maintenance within domain), Data Security Specialist (Essential Eight compliance, security controls), Internal Audit (independent compliance verification).

---

### 9. Continuous Improvement and Governance Operations

Governance is not a set-and-forget programme. The regulatory landscape evolves, the data platform evolves, business requirements evolve, and the organisation's data maturity evolves. The continuous improvement stage ensures that governance practices, policies, and controls are regularly assessed, measured, and improved.

**Governance Metrics and KPIs:**

What gets measured gets managed. The governance programme tracks metrics across its core dimensions:

- **Coverage.** Percentage of data assets catalogued, classified, and owned. Target: 100% for Tier 1 and Tier 2 data products.
- **Quality.** Average quality score across governed data products. Quality trend over time (improving, stable, declining).
- **Freshness SLO Compliance.** Percentage of data products meeting their published freshness SLOs.
- **Access Control Compliance.** Percentage of data assets with confirmed classification and enforced ABAC policies. Time to provision access for new requests.
- **Metadata Completeness.** Percentage of data products with complete business metadata (description, ownership, glossary linkage, quality score).
- **Data Contract Coverage.** Percentage of Tier 1 and Tier 2 data products with active, enforced data contracts.
- **Classification Lifecycle Compliance.** Percentage of data assets completing the Unclassified â Provisional â Classified lifecycle within the defined SLA windows (30 and 60 days respectively). Number of objects currently in breach.
- **Issue Resolution.** Mean time to detect and resolve data quality issues. Number of open quality issues by domain and severity.
- **Stewardship Activity.** Classification reviews completed, metadata reviews conducted, quality issues triaged and resolved. Measures active engagement of the stewardship community.

**Governance Reviews:**

- **Monthly.** Domain Data Stewards review quality metrics, classification status, and open issues within their domain. Report to the Data Governance Manager.
- **Quarterly.** The Data Governance Council reviews enterprise-wide governance KPIs, cross-domain issues, policy effectiveness, and strategic priorities. Provides direction and resolves cross-domain disputes.
- **Annually.** Full governance programme review including policy currency, regulatory alignment, maturity assessment, and strategic planning. Results inform the governance roadmap for the next year.

**Maturity Assessment:**

The governance programme periodically assesses its maturity using a defined maturity model (aligned to DAMA-DMBOK or a comparable framework). Maturity is assessed across governance dimensions (policy, quality, metadata, access control, stewardship, compliance) and expressed as a level (Initial, Managed, Defined, Measured, Optimising). The maturity assessment informs investment priorities and identifies the areas where governance improvement will deliver the most business value.

**Guidance:**

- Define governance KPIs and report on them regularly. Governance that cannot demonstrate its own effectiveness will not retain executive support.
- Conduct governance reviews at defined cadences. Do not let governance become a programme that operates without feedback.
- Use maturity assessments to drive continuous improvement rather than as a one-time benchmarking exercise.
- Invest governance improvement effort where it delivers the most value, typically quality, metadata, and stewardship coverage for critical data products.
- Celebrate governance successes. Acknowledge domains and stewards that demonstrate strong governance practice. Governance is a shared responsibility and cultural change requires positive reinforcement.

**Key Roles:** Data Governance Manager (KPI definition, reporting, governance reviews, maturity assessment), Data Governance Council (strategic direction, cross-domain resolution), Domain Data Steward (domain-level metric review and improvement), Chief Data Officer (executive sponsorship and programme accountability).

---

## Governance Roles

The effectiveness of a governance programme depends on clearly defined roles with unambiguous accountabilities. Governance is a shared responsibility, but "shared" does not mean "diffuse." Every governance activity has a named role accountable for it. The following roles form the EDAP governance operating model.

---

### Executive Governance Roles

**Chief Data Officer (CDO)**

The CDO is the executive accountable for the enterprise data governance programme. The CDO sets the strategic vision for data governance, secures executive sponsorship and funding, represents data governance at the executive leadership level, and ensures that governance supports business objectives rather than impeding them. The CDO chairs or sponsors the Data Governance Council and is accountable for governance programme outcomes.

**Data Governance Council**

The Data Governance Council is the cross-functional governance body that provides strategic direction, resolves cross-domain disputes, approves enterprise-wide policies, and reviews governance programme performance. Membership includes the CDO (chair), Domain Data Owners, the Data Governance Manager, and representatives from IT, Legal/Compliance, and Information Security. The Council meets quarterly and makes decisions on matters that cannot be resolved at the domain level: conflicting data definitions, cross-domain access disputes, policy exceptions, and governance investment priorities.

**Domain Data Owner**

The Domain Data Owner is the senior business executive accountable for a data domain (e.g. the GM Asset Management is the Domain Data Owner for the Asset domain). The Domain Data Owner has decision-making authority over how domain data is used, shared, and governed. They endorse domain-specific policies, approve data sharing arrangements, and ensure that domain governance investment is appropriately prioritised. The Domain Data Owner is an executive role: they set direction but delegate operational governance to the Domain Data Steward.

---

### Stewardship Roles

The stewardship roles are where governance rubber meets the road. This framework distinguishes between two types of data steward, **Domain Data Stewards** and **Technical Data Stewards**, because they serve fundamentally different functions, require different skills, and are accountable for different outcomes. Conflating them is one of the most common governance anti-patterns.

---

**Domain Data Steward**

The Domain Data Steward is the **business-facing** governance role responsible for the meaning, quality, and appropriate use of data within their domain. Domain Data Stewards are business professionals with deep domain knowledge. They understand the business context, business rules, and business value of the data in their domain. They do not need to be technical; they need to be authoritative about what the data means and how it should be used.

**Accountabilities:**

- **Business Metadata Ownership.** Authoring and maintaining business descriptions, definitions, and semantic context for data products within their domain. Ensuring the business glossary is accurate for their domain's terms.
- **Classification Review.** Reviewing and confirming classification assignments (sensitivity, regulatory, business value) for data products in their domain. The Domain Data Steward is the authority on whether a dataset contains personal information, is commercially sensitive, or has specific regulatory handling requirements.
- **Quality Rule Definition.** Defining the business rules that determine data quality for their domain. What does "accurate" mean for asset location data? What does "complete" mean for a customer record? These are business judgements that only domain experts can make.
- **Quality Issue Triage and Escalation.** Reviewing quality issues detected by automated assertions, assessing business impact, prioritising remediation, and escalating systemic issues to the Domain Data Owner or Data Governance Council.
- **Access Approval.** Approving access requests for sensitive data within their domain. Reviewing access patterns to ensure data is being used appropriately and within the purpose for which access was granted.
- **Data Product Governance.** Ensuring data products within their domain meet the Five Gates criteria for their governance tier. Participating in data product promotion decisions (Tier 3 â Tier 2 â Tier 1).
- **Data Contract Ownership.** Defining and maintaining data contracts for data products within their domain. Negotiating contract terms with consumers. Approving contract changes.
- **Business Stakeholder Liaison.** Acting as the bridge between governance processes and business users within their domain. Translating governance requirements into business-understandable terms and channelling business feedback into governance improvement.
- **Cross-Domain Collaboration.** Working with stewards in other domains to resolve cross-domain data issues (e.g. conflicting definitions, shared data products, multi-domain source systems).

**Reports to:** Domain Data Owner (business accountability). Collaborates with Data Governance Manager (framework alignment) and Technical Data Steward (implementation).

**Profile:** Senior business professional with deep domain expertise. Strong communication skills. Comfortable making judgement calls about data meaning and quality. Does not require technical platform skills.

---

**Technical Data Steward**

The Technical Data Steward is the **platform-facing** governance role responsible for implementing, automating, and enforcing governance controls within the data platform. Technical Data Stewards are data professionals (data engineers, platform engineers, or senior analysts) with deep platform knowledge. They understand Unity Catalog, data contracts, ABAC configuration, lineage tools, and automated quality frameworks. They translate the governance decisions made by Domain Data Stewards and the Data Governance Manager into executable, automated controls.

**Accountabilities:**

- **Catalog and Metadata Implementation.** Configuring Unity Catalog schemas, tags, and metadata. Implementing automated metadata harvesting. Ensuring that the catalog accurately reflects the current state of data assets. Managing the technical integration between Unity Catalog and Alation.
- **Classification Tag Management.** Implementing and maintaining the tagging taxonomy in Unity Catalog, including the separation of `source_system` (provenance, applied at Bronze) and `data_domain` (ownership, applied at Silver Enriched/Gold) tags. Configuring tag propagation rules (how classifications flow from source to derived assets across zone boundaries). Ensuring that classification tags assigned by Domain Data Stewards are correctly applied in the platform.
- **Access Control Implementation.** Translating access policies into ABAC rules in Unity Catalog. Configuring column masks, row-level security filters, and dynamic views. Managing the Silver Protected Zone PAM configuration. Managing per-layer service principal isolation. Managing access grants and revocations. Auditing access patterns using Unity Catalog audit logs.
- **Data Quality Assertion Implementation.** Implementing the quality rules defined by Domain Data Stewards as automated assertions in pipeline code (Lakeflow SDP expectations, dbt tests, Great Expectations suites). Ensuring DQ annotation columns (`dq_status`, `dq_flags`, `dq_checked_ts`) are correctly computed and carried forward from Silver Base through all downstream zones. Configuring data observability monitoring (Data Profiling). Managing quality score computation and publication. Monitoring quarantine path growth.
- **Data Contract Enforcement.** Implementing data contract specifications as enforceable checks in CI/CD pipelines. Ensuring that schema changes, quality specification changes, and SLA changes are validated against downstream contracts before deployment.
- **Lineage Validation.** Ensuring that data lineage is accurately captured and traceable from source to consumption. Investigating and resolving lineage gaps or inaccuracies.
- **Retention and Lifecycle Automation.** Implementing automated retention policies, archive transitions, and deletion workflows. Ensuring verifiable deletion for compliance purposes.
- **Classification Lifecycle Monitoring.** Implementing and maintaining the automated scans that enforce classification lifecycle SLAs (30-day Unclassified window, 60-day Provisional window). Configuring alerts and escalation notifications.
- **Governance Tooling and Automation.** Building and maintaining the automation that makes governance scalable: automated classification propagation, quality score dashboards, stewardship workflow tools, and governance metric collection.

**Reports to:** Data Platform Manager or Data Governance Manager (depending on organisational structure). Collaborates closely with Domain Data Stewards (implementing their governance decisions) and Data Engineers (embedding governance in pipelines).

**Profile:** Senior data engineer, platform engineer, or data management professional with deep EDAP platform expertise (Unity Catalog, Databricks, data quality frameworks). Strong automation and coding skills. Understands governance policy and can translate policy into platform configuration.

---

**The Relationship Between Domain and Technical Stewards:**

The Domain Data Steward decides *what*: what the data means, what quality looks like, what classification applies, who should have access. The Technical Data Steward decides *how*: how to implement those decisions in the platform, how to automate enforcement, how to monitor compliance. Neither role is effective without the other:

- A Domain Data Steward without a Technical Data Steward has governance policies that exist only on paper, unenforced and unmonitored.
- A Technical Data Steward without a Domain Data Steward has technical controls without business context: automated enforcement of rules that may be wrong, incomplete, or misaligned with business reality.

The collaboration model is: Domain Data Steward defines the requirement â Technical Data Steward implements the control â the platform enforces the control automatically â monitoring reports compliance back to both stewards.

---

### Operational Governance Roles

**Data Governance Manager**

The Data Governance Manager is the programme manager for data governance. They own the governance framework, coordinate the stewardship community, manage the governance roadmap, report on governance KPIs, and ensure that governance activities are executed consistently across domains. The Data Governance Manager facilitates the Data Governance Council, manages the governance policy lifecycle, and acts as the escalation point for governance issues that cannot be resolved at the domain level.

**Accountabilities:**

- Governance framework design, documentation, and maintenance.
- Governance programme planning, roadmap, and budget.
- Stewardship community coordination, enablement, and support.
- Governance KPI definition, reporting, and continuous improvement.
- Policy lifecycle management (drafting, review, approval, communication, retirement).
- Compliance programme management (regulatory mapping, audit coordination, PIA oversight).
- Data Governance Council facilitation and secretariat.
- Cross-domain issue resolution and escalation.
- Classification lifecycle SLA enforcement and escalation management.

**Data Custodian**

The Data Custodian is the operational role responsible for the physical management, security, and availability of data infrastructure. In the EDAP's cloud lakehouse context, this role is typically performed by the Data Platform team or Cloud Infrastructure team. Data Custodians ensure that storage is provisioned, secured, backed up, and available in accordance with governance policies, but they do not make decisions about data content, meaning, or quality. Those decisions belong to stewards and owners.

---

## The Evolving Role of Data Governance in 2026

The data governance discipline is undergoing a transformation as significant as the shifts described in the Data Engineering, BI, and Data Science Lifecycles:

**From gatekeeper to enabler.** The governance function is shifting from a compliance-oriented, approval-chain-heavy gatekeeper to an enabling function that makes it easy for domain teams to do the right thing. "Paved roads" (pre-built, governed patterns for common governance tasks such as data product registration, access provisioning, and quality assertion setup) are replacing manual processes and governance board approvals.

**From manual to automated.** Governance controls that depended on human diligence and periodic audits are being replaced by automated, platform-enforced controls: ABAC policies, data contract validation in CI/CD, automated classification propagation, and programmatic quality scoring. The goal is "governance as code," governance that is version-controlled, testable, and enforceable by the platform.

**From human-readable to machine-readable.** Governance metadata that served human consumers (free-text descriptions, PDF policies, wiki pages) must now also serve AI consumers that need structured, machine-readable governance signals to discover, evaluate, and trust data programmatically. This raises the bar on metadata quality, semantic richness, and classification precision.

**From centralised control to federated empowerment.** The governance operating model is shifting from a central team that reviews and approves everything to a federated model where domain teams own their governance within a central framework. The central team provides standards, tooling, and oversight, not gatekeeping.

**From periodic audit to continuous monitoring.** Governance compliance is shifting from periodic, retrospective audits to continuous, real-time monitoring with automated alerting. Stewards are notified of issues as they occur, not weeks later in an audit report.

---

## Closing Thoughts

The data governance lifecycle is not a one-time implementation but a continuously operating system that matures over time. The stages remain stable even as the technology, regulatory landscape, and organisational context evolve. The need to discover, classify, protect, manage quality, catalogue, control access, manage retention, ensure compliance, and continuously improve data governance will endure regardless of which platform or tools the organisation uses.

The organisations that extract the most value from their data are those that treat governance not as overhead but as infrastructure, as essential to the data platform as compute and storage. Governance is what makes the difference between a data platform that is merely capable and one that is trusted.

**The key strategic shifts to internalise in 2026:**

- **Federated governance as a deliberate architecture.** Not the absence of centralisation, but the distribution of governance responsibility to the people closest to the data, within a consistent central framework. Domain ownership with enterprise guardrails.
- **Governance as code.** Policies encoded as automated, platform-enforced rules (ABAC, data contracts, quality assertions, classification propagation) rather than documents that rely on human compliance. If it is not enforced by the platform, it is a suggestion, not governance.
- **Open by default, restricted by exception.** Data accessible to all authenticated users unless a specific, documented restriction applies. Focus governance energy on precisely controlling the data that genuinely requires restriction. But recognise that this principle presupposes completed classification; unclassified data is restricted by default.
- **Machine-readable governance.** Every governance signal (classification, quality score, ownership, lineage, freshness) available as structured, machine-readable metadata that AI agents and automated systems can consume. Governance that exists only in human-readable form is invisible to half your future consumers.
- **Provenance and ownership as distinct concepts.** Source system identity (`source_system`) and business domain accountability (`data_domain`) are fundamentally different attributes, expressed as separate tags, applied at different zones of the medallion architecture, by different roles, for different purposes. The Unity Catalog namespace itself reinforces this: source-system-aligned schemas at Bronze and Silver Base progressively give way to subject-area schemas at Silver Enriched and domain schemas at Gold. Semantic clarity in tagging is the foundation of machine-readable governance.
- **Propagate, don't drop.** Business data quality issues are annotated, not filtered. Only structural failures that prevent processing are quarantined. Silent data loss is a governance failure. Quality visibility through physical DQ annotation columns (`dq_status`, `dq_flags`, `dq_checked_ts`) ensures that all data is represented where expected, and business users can see and act on quality problems. Exclusion of records from a specific data product is an explicit, documented business decision â not a general pipeline behaviour.
- **Proportional governance tiers.** Apply governance rigour proportional to data product importance. Over-governing exploratory data stifles innovation; under-governing enterprise data creates risk. The tier model ensures both.
- **Data contracts as the producer-consumer interface.** Formalised, versioned, enforceable agreements that make data product commitments explicit and measurable. The mechanism that transforms governance from aspiration to accountability.
- **Stewardship as a collaborative practice.** Domain Data Stewards and Technical Data Stewards working in concert, with business knowledge and platform expertise as equal partners. Neither role is effective alone.
- **Continuous improvement, not compliance theatre.** Governance measured by outcomes (data trust, time-to-access, quality scores, consumer satisfaction) not by the number of policies written or approval forms processed.

Data governance is ultimately about trust: trust that the data is what it claims to be, that it is protected appropriately, that it is managed responsibly, and that it is available to those who need it. The governance lifecycle provides the framework; the roles provide the accountability; and the organisation's commitment to data as a strategic asset provides the mandate.

The four lifecycles (Data Engineering, Business Intelligence, Data Science, and Data Governance) form a coherent whole. Data engineering provides the platform; BI turns data into decisions; data science turns data into intelligence; and data governance ensures that all three operate on a foundation of trust, quality, and accountability.

---

*This document describes the data governance lifecycle for Water Corporation's Enterprise Data & Analytics Platform (EDAP). It complements the Data Engineering Lifecycle, the Business Intelligence Lifecycle, the Data Science Lifecycle, and the Medallion Architecture document. It draws on the DAMA-DMBOK2 framework, adapted for the EDAP's Databricks lakehouse architecture, federated governance operating model, and Western Australian regulatory context. Last updated March 2026.*

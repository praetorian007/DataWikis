# Core Data Governance Roles in the Enterprise

**Mark Shaw** | Principal Data Architect

---

## Purpose

This document defines the core data governance roles found in enterprise data governance frameworks. It clarifies accountability boundaries, decision rights, and the relationships between roles — particularly the commonly conflated distinctions between Data Owner, Data Custodian, Data Domain Steward, and Technical Data Steward.

---

## Role Summary

| Role | Accountable For | Typical Level | Scope |
|---|---|---|---|
| Chief Data Officer (CDO) | Enterprise data strategy and governance programme | C-suite / Executive | Enterprise-wide |
| Data Governance Council | Policy ratification, conflict resolution, programme oversight | Cross-functional executive forum | Enterprise-wide |
| Data Owner | Business accountability for a data domain's quality, access, and lifecycle | Executive / Senior Leader (GM, Director) | Data Domain |
| Data Domain Steward | Day-to-day governance execution within a domain | Senior SME / Manager | Data Domain |
| Technical Data Steward | Implementing governance rules in platforms and pipelines | Senior Engineer / Architect | Data Domain or Platform |
| Data Custodian | Operational safekeeping of data infrastructure | Platform / Ops Engineer | Platform / Infrastructure |
| Data Product Owner | End-to-end accountability for a data product's value, quality, and lifecycle | Product Manager / Senior SME | Data Product |
| Data Platform Owner | Accountability for the data platform's capability, evolution, and enablement | Platform Lead / Engineering Manager | Platform / Infrastructure |
| Data Producer | Creating and publishing data according to contracts | Development / Engineering Team | Source System or Data Product |
| Data Consumer | Using data responsibly within policy bounds | Analyst / Data Scientist / Business User | Consumption Layer |
| Data Governance Manager | Governance programme management, framework coordination, stewardship enablement | Senior Manager / Programme Lead | Enterprise-wide |
| Data Protection Officer (DPO) | Regulatory compliance for personal and sensitive information | Legal / Compliance | Enterprise-wide |

---

## Role Definitions

### Chief Data Officer (CDO)

The CDO is the executive sponsor of the enterprise data governance programme. They own the data strategy, set the governance operating model, and represent data as a strategic asset at the executive level.

**Key responsibilities:**

- Define and champion the enterprise data strategy.
- Establish the governance operating model (centralised, federated, or hybrid).
- Secure funding and executive sponsorship for data initiatives.
- Report on data maturity, quality, and value realisation to the board.
- Arbitrate unresolved escalations from the Data Governance Council.
- Champion enterprise data literacy — ensuring domain teams have the skills and resources to understand governance concepts, interpret data quality dashboards, and use the data catalogue effectively.

**Decision rights:** Strategic direction, governance model design, programme funding.

---

### Data Governance Council

A cross-functional leadership forum that ratifies policy, resolves cross-domain disputes, and provides oversight of the governance programme. Not a single person — a standing body.

**Key responsibilities:**

- Approve enterprise data policies, standards, and classifications.
- Resolve cross-domain ownership disputes and priority conflicts.
- Review and endorse the data domain model and domain ownership assignments.
- Monitor governance KPIs (quality scorecards, compliance metrics, adoption).
- Ensure alignment between data governance and enterprise architecture, security, and risk.

**Typical membership:** CDO (chair), Data Owners, CISO representative, Enterprise Architecture, Legal/Compliance, and selected Data Domain Stewards as subject matter advisors.

---

### Data Owner

The Data Owner is the **business executive accountable** for a defined data domain. This is an accountability role, not a hands-on operational role. The Data Owner does not touch data day-to-day — they make decisions *about* data.

**Key responsibilities:**

- Approve who may access data within their domain (access policy).
- Define acceptable quality thresholds and escalation paths.
- Approve data classification and sensitivity labels.
- Sign off on data sharing agreements and cross-domain data flows.
- Nominate and empower Data Domain Stewards to act on their behalf.
- Accept residual data risk within their domain.

**Analogy:** The Data Owner is like a property owner. They decide who gets a key, what insurance to carry, and what renovations to approve — but they do not personally mow the lawn or fix the plumbing.

**Common misconception:** Data Owners do not need to understand the technical platform. Their authority is *business* authority — they decide *what* and *why*, not *how*.

**Decision rights:** Access approvals, classification sign-off, quality thresholds, data sharing agreements.

---

### Data Domain Steward

The Data Domain Steward is the **operational governance lead** within a data domain. They are the Data Owner's delegate — a senior subject matter expert who understands both the business meaning and the governance rules that apply to the domain's data.

**Key responsibilities:**

- Define and maintain business definitions, data dictionaries, and glossary terms for the domain.
- Monitor data quality, investigate root causes of quality issues, and coordinate remediation.
- Manage metadata completeness within the catalogue (descriptions, lineage, classifications).
- Triage access requests against policies set by the Data Owner.
- Coordinate with Technical Data Stewards on implementation of governance rules.
- Identify and escalate cross-domain data conflicts or duplication.
- Participate in data governance working groups and communities of practice.

**Analogy:** The Data Domain Steward is the property manager. They handle tenant enquiries, arrange maintenance, enforce house rules, and report issues to the owner.

**Decision rights:** Metadata definitions, quality issue triage and prioritisation, access request triage (within owner-approved policy).

---

### Technical Data Steward

The Technical Data Steward translates governance policies into **platform-level implementation**. They sit at the intersection of data engineering and data governance, ensuring that rules defined by Domain Stewards and Owners are enforced in code, configuration, and platform controls.

**Key responsibilities:**

- Implement data quality rules, validation checks, and anomaly detection in pipelines.
- Configure access controls, tagging, and classification labels in the data platform (e.g. Unity Catalog ABAC, column masking, row filters).
- Maintain and enforce data contracts (schemas, SLAs, freshness guarantees).
- Implement data lineage capture and ensure catalogue metadata is programmatically current.
- Support pipeline change management — assess impact of schema evolution on downstream consumers.
- Operationalise data retention and purging policies.

**Analogy:** The Technical Data Steward is the building engineer. They install the locks the property manager specifies, wire up the alarms, run the inspections, and make sure the infrastructure enforces the rules.

**Relationship to Data Engineer:** A Technical Data Steward may *be* a data engineer, or may work alongside them. The distinction is one of focus: the data engineer builds pipelines for throughput and correctness; the Technical Data Steward ensures those pipelines enforce governance policy. In smaller teams, one person often wears both hats.

**Decision rights:** Implementation approach for governance controls, technical standards for metadata and quality automation.

---

### Data Custodian

The Data Custodian is responsible for the **safe operational management** of the infrastructure on which data resides. They do not decide *what* the data means or *who* should access it — those are Owner and Steward responsibilities. The Custodian ensures the platform is secure, available, backed up, and performing.

**Key responsibilities:**

- Manage platform infrastructure (compute, storage, networking, patching).
- Implement and maintain backup, disaster recovery, and business continuity for data stores.
- Enforce infrastructure-level security controls (encryption at rest and in transit, network segmentation, identity integration).
- Monitor platform health, performance, and cost.
- Execute environment provisioning and decommissioning.
- Ensure platform compliance with security frameworks (e.g. Essential Eight, SOCI Act critical infrastructure requirements).

**Analogy:** The Data Custodian is the building's facilities management team. They keep the lights on, the doors locked, and the fire systems operational — but they do not decide which tenants move in or what happens inside each office.

**Common misconception:** "Custodian" is sometimes loosely used to mean anyone who looks after data. In a governance framework, the Custodian's scope is strictly *infrastructure and platform operations*, not business meaning or access policy.

**Decision rights:** Infrastructure configuration, backup schedules, platform security controls.

---

### Data Product Owner

The Data Product Owner is accountable for the **end-to-end value, quality, and lifecycle** of a specific data product. This role emerges from treating data as a product — something with consumers, an SLA, a contract, and a roadmap. It is distinct from the Data Owner (who owns a domain) and the Data Producer (who builds and publishes). The Data Product Owner sits between them, ensuring the product delivers sustained value.

**Key responsibilities:**

- Define the data product's purpose, scope, target consumers, and success metrics.
- Own the data contract — a formal agreement specifying the schema, freshness SLA, quality thresholds, semantic definitions, and versioning policy that the product guarantees to its consumers.
- Prioritise the product backlog: enhancements, quality improvements, new consumer requirements.
- Ensure the product is discoverable, well-documented, and registered in the data catalogue.
- Monitor product adoption, consumer satisfaction, and quality scorecards.
- Coordinate across producers (who supply the data), technical stewards (who enforce governance), and consumers (who derive value).
- Manage the product lifecycle — from incubation through active service to deprecation and retirement.
- Ensure the product is accessible through self-service mechanisms (e.g. data marketplace, automated provisioning) so that consumers can discover, request, and subscribe with minimal friction.

**Relationship to Data Owner:** The Data Owner is accountable for the *domain*. The Data Product Owner is accountable for a *specific product* within that domain. A single Data Owner may have multiple Data Product Owners reporting into their domain. The Data Owner sets the policies and quality bar; the Data Product Owner delivers against them.

**Cross-domain data products:** Where a data product blends data from multiple domains (e.g. joining Customer and Asset data), a single Data Owner must be designated as the accountable owner for the product — typically the owner of the domain that contributes the primary business context. Contributing domain owners retain accountability for the quality and classification of their domain's data inputs. The Data Governance Council resolves disputes over cross-domain product ownership.

**Relationship to Data Producer:** The Data Producer builds and publishes the data. The Data Product Owner decides *what* gets built, *for whom*, and *to what standard*. In smaller teams these roles may overlap, but the accountability is distinct — the Producer is responsible for engineering delivery, the Product Owner for value delivery.

**Analogy:** If the Data Owner is the property owner and the Domain Steward is the property manager, the Data Product Owner is a business operator running a specific venue within the building — they decide the menu, the service level, and the customer experience, while operating within the building's rules.

**Decision rights:** Product scope and roadmap, contract terms, consumer prioritisation, product lifecycle decisions (publish, version, deprecate).

---

### Data Platform Owner

The Data Platform Owner is accountable for the **data platform as an enabling capability** — its architecture, evolution, developer experience, and fitness for purpose. Where the Data Custodian focuses on operational health (keeping the lights on), the Data Platform Owner focuses on platform strategy and enablement (making the platform worth using).

**Key responsibilities:**

- Define and maintain the platform roadmap aligned to enterprise data strategy.
- Own platform architecture decisions — tooling choices, integration patterns, environment strategy.
- Ensure the platform provides self-service capabilities that enable domain teams to build and govern data products autonomously.
- Define and enforce platform standards — naming conventions, tagging taxonomies, pipeline frameworks, security patterns.
- Manage platform onboarding — making it easy for new domains, producers, and consumers to adopt the platform.
- Own platform cost management, capacity planning, and vendor relationships.
- Coordinate with the Data Custodian on operational concerns and with Technical Data Stewards on governance enforcement.
- Represent platform capabilities and constraints to the Data Governance Council and enterprise architecture.

**Relationship to Data Custodian:** The Custodian keeps the platform running day-to-day. The Platform Owner decides where the platform is going. The Custodian handles patching, backups, and incident response; the Platform Owner handles capability roadmap, developer experience, and architectural evolution. In practice, they work hand-in-glove.

**Relationship to Technical Data Steward:** Technical Data Stewards implement governance rules *within* the platform. The Platform Owner ensures the platform *provides the mechanisms* for that governance — Unity Catalog policies, tagging frameworks, quality rule engines, lineage capture. The Platform Owner builds the guardrails; the Technical Data Stewards configure them per domain.

**Analogy:** If the Data Custodian is facilities management, the Data Platform Owner is the building developer — they decide what amenities the building offers, what the floor plan looks like, and how the building evolves over time to attract and serve tenants.

**Decision rights:** Platform architecture and tooling, platform standards and conventions, capability roadmap, environment strategy, vendor and licensing decisions.

---

### Data Producer

A Data Producer is any team or system that **creates, transforms, or publishes** data that others consume. In a data product model, the producer is accountable for the quality, schema stability, and contractual guarantees of the data they publish.

**Key responsibilities:**

- Publish data in accordance with defined data contracts (schema, freshness SLA, quality thresholds).
- Notify consumers of breaking changes via versioning or change management processes.
- Ensure source data meets agreed quality standards before ingestion or publication.
- Register data products in the catalogue with complete metadata.
- Remediate quality issues at the source when identified by stewards or consumers.

**Decision rights:** Source system design, ingestion approach, contract negotiation (jointly with stewards).

---

### Data Consumer

A Data Consumer is any individual or system that **reads and uses** data produced by others. Consumers have a responsibility to use data within the bounds of its classification, intended purpose, and access policy.

**Key responsibilities:**

- Use data in accordance with its classification and approved purpose.
- Report quality issues or unexpected behaviours to the relevant Domain Steward.
- Not redistribute data beyond its approved access scope.
- Understand and respect data contracts and SLA limitations.

**Decision rights:** How data is used within approved policy bounds; feedback on data product fitness.

---

### Data Governance Manager

The Data Governance Manager is the programme manager for data governance. They own the governance framework, coordinate the stewardship community, manage the governance roadmap, report on governance KPIs, and ensure that governance activities are executed consistently across domains. The Data Governance Manager facilitates the Data Governance Council, manages the governance policy lifecycle, and acts as the escalation point for governance issues that cannot be resolved at the domain level.

**Key responsibilities:**

- Design, document, and maintain the governance framework.
- Plan and manage the governance programme roadmap and budget.
- Coordinate, enable, and support the stewardship community across domains.
- Define governance KPIs, produce reporting, and drive continuous improvement.
- Manage the policy lifecycle: drafting, review, approval, communication, and retirement.
- Manage the compliance programme: regulatory mapping, audit coordination, and PIA oversight.
- Facilitate and provide secretariat for the Data Governance Council.
- Resolve cross-domain governance issues and manage escalations.

**Decision rights:** Governance framework design, programme planning, stewardship coordination, policy lifecycle management.

**Analogy:** If the CDO sets the direction and the Data Governance Council ratifies the rules, the Data Governance Manager keeps the machine running — ensuring policies are current, stewards are supported, KPIs are tracked, and governance activities happen on cadence.

---

### Data Protection Officer (DPO)

The DPO provides independent oversight of personal and sensitive information handling. In Australian government contexts, this role ensures compliance with the Privacy Act, PRIS Act, and equivalent state legislation.

**Key responsibilities:**

- Advise on privacy impact assessments and data protection obligations.
- Monitor compliance with personal information handling requirements under the Privacy Act 1988 (Cth), PRIS Act 2024 (WA), and applicable state legislation.
- Manage obligations under the Notifiable Data Breaches scheme (Part IIIC of the Privacy Act 1988), including assessment, notification, and remediation of eligible data breaches.
- Act as the point of contact for regulators and data subjects.
- Review data sharing agreements involving personal or sensitive information.
- Ensure privacy-by-design principles are embedded in data platform architecture.
- Monitor the Privacy Act reform programme (post-AGD review) for new obligations, particularly around automated decision-making, which may create additional requirements for AI/ML systems that process personal information.
- Coordinate SOCI Act 2018 mandatory reporting obligations for cyber security incidents affecting critical infrastructure data, in conjunction with the CISO. As a critical infrastructure entity, Water Corporation must report significant cyber security incidents to the Cyber and Infrastructure Security Centre (CISC) within prescribed timeframes. This includes obligations under the 2024 SOCI Rules amendments, which expanded positive security obligations for critical infrastructure entities.

**Decision rights:** Privacy compliance assessments, regulatory reporting (including Notifiable Data Breaches and SOCI Act incidents), privacy risk escalation.

---

### AI/ML Governance Lead

The AI/ML Governance Lead (sometimes titled Responsible AI Officer) provides oversight of the organisation's use of artificial intelligence and machine learning, ensuring that AI systems are developed and operated in accordance with ethical principles, regulatory requirements, and organisational risk appetite.

**Key responsibilities:**

- Establish and maintain the AI governance framework, including principles for fairness, bias mitigation, transparency, explainability, and human oversight.
- Maintain an AI model inventory, which is a register of all AI/ML models in development, testing, and production, including their training data sources, intended use, risk classification, and responsible domain owner.
- Classify AI models by risk tier (e.g. low, medium, high, critical) based on their impact on individuals, operational safety, and regulatory exposure. High and critical models require enhanced governance review.
- Ensure human oversight and contestability — decisions made or materially informed by AI must be reviewable and challengeable by affected parties, with clear escalation paths.
- Monitor deployed models for drift, performance degradation, and emergent bias on an ongoing basis, not only at initial deployment.
- Ensure compliance with emerging AI regulations and standards applicable to Australian government entities and critical infrastructure operators, including the Australian Government's Voluntary AI Safety Standard (2024) and alignment with ISO/IEC 42001:2023 (AI Management Systems).
- Advise domain owners and data product owners on the governance implications of using domain data for AI training, including PRIS Act 2024 obligations for personal information.
- Review and approve the use of third-party or externally hosted AI services that consume Water Corporation data, including assessment of data residency, model transparency, and vendor AI governance posture.
- Consider environmental sustainability of AI workloads — compute and storage costs associated with model training and inference should be proportionate to the value delivered.
- Coordinate with the DPO on privacy impact assessments for AI systems that process personal information.
- Report on AI governance posture and risk to the Data Governance Council.

**Relationship to Data Product Owner:** Where an AI/ML model consumes a data product, the Data Product Owner retains accountability for the quality, classification, and access governance of the input data. The AI/ML Governance Lead is accountable for the governance of the model itself, including its fairness, transparency, and appropriate use. In organisations where a dedicated AI/ML Governance Lead role is not yet established, this accountability maps to the Data Product Owner for models that consume their data product, with the Data Governance Council providing oversight.

**Decision rights:** AI risk classification, AI model inventory management, approval of third-party AI service usage, AI governance framework and standards, model retirement decisions based on drift or degradation.

---

## The Key Distinction: Owner vs Custodian vs Steward

This is the most commonly confused boundary in data governance. The table below draws a clean line.

| Dimension | Data Owner | Data Domain Steward | Technical Data Steward | Data Custodian |
|---|---|---|---|---|
| **Core question** | *Who is accountable for this data?* | *Who governs it day-to-day?* | *Who enforces rules in the platform?* | *Who keeps the infrastructure safe?* |
| **Scope** | Business accountability | Business-side operational governance | Platform-side governance implementation | Infrastructure operations |
| **Decides** | Access policy, classification, quality thresholds | Metadata definitions, quality triage, glossary | Implementation patterns, automation, contracts | Backup, DR, security config, patching |
| **Does not decide** | Technical implementation | Platform architecture | Business meaning of data | Who accesses what or why |
| **Typical profile** | Executive / Director | Senior business SME | Senior data engineer / architect | Platform / DevOps engineer |
| **Accountability** | Strategic and regulatory | Operational quality and meaning | Technical enforcement | Operational availability and security |

---

## Federated vs Centralised Governance and Role Implications

In a **centralised** model, a central team holds most stewardship functions. In a **federated** model, stewardship is distributed to domains with central coordination. Most mature organisations adopt a **hybrid** approach.

| Aspect | Centralised | Federated | Hybrid |
|---|---|---|---|
| Domain Stewards | Central team members | Embedded in business domains | Embedded in domains with central standards |
| Technical Stewards | Central platform team | Domain engineering teams | Domain teams following central patterns |
| Data Owners | Nominated centrally | Domain executives | Domain executives with central governance council |
| Consistency risk | Low (but bottleneck risk) | Higher (divergent practices) | Managed through standards and communities of practice |
| Scalability | Limited | High | High with guardrails |

**Choosing the right model:**

- **Centralised** suits organisations with a small number of data domains, a single platform team, and a strong central governance function. It provides maximum consistency but can become a bottleneck as the number of domains and data products grows.
- **Federated** suits large, multi-business-unit organisations where each unit has distinct data domains, dedicated engineering teams, and different regulatory contexts. It provides maximum autonomy but risks inconsistent practices without strong coordination.
- **Hybrid** is the recommended model for most organisations, including Water Corporation. It combines central policy setting and standards with domain-level stewardship and enforcement.

**Recommendation for Water Corporation:** Given Water Corporation's scale (a single utility with a centralised platform delivery model, fewer than ten major source systems, and a single regulatory jurisdiction), the **hybrid model** is recommended. Central governance sets the tag taxonomy, classification standards, ABAC policy patterns, and quality frameworks. Domain stewards embedded in each business area apply these standards to their domain's data, make classification decisions, and validate technical controls. The platform team provides the shared infrastructure and tooling that enables consistent enforcement. This avoids the overhead of a fully federated model (which is designed for multi-entity enterprises) while preserving the domain accountability that a purely centralised model struggles to maintain.

---

## RACI Summary

| Activity | CDO | Data Owner | Domain Steward | Technical Steward | Custodian | Product Owner | Platform Owner | Producer | Consumer | Gov. Manager | DPO | AI/ML Lead |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Set data strategy | **A/R** | C | C | I | I | I | C | I | I | C | I | C |
| Define access policy | I | **A** | **R** | C | I | C | I | I | I | C | **C** | I |
| Approve data classification | C | **A** | **R** | C | I | I | I | I | I | C | **C** | C |
| Maintain business glossary | I | C | **A/R** | C | I | C | I | C | C | C | I | I |
| Implement quality rules | I | I | **A** | **R** | I | C | I | C | I | C | I | I |
| Configure platform access controls | I | I | C | **A/R** | C | I | C | I | I | I | I | I |
| Manage infrastructure security | I | I | I | C | **A/R** | I | C | I | I | I | I | I |
| Define data product scope and contract | I | **A** | C | C | I | **R** | I | C | C | I | I | C |
| Publish data products | I | C | C | C | I | **A** | I | **R** | I | I | I | I |
| Manage data product lifecycle | I | C | C | I | I | **A/R** | I | C | C | I | I | I |
| Define platform roadmap | C | I | I | C | C | C | **A/R** | I | I | I | I | I |
| Set platform standards and conventions | I | I | I | C | C | I | **A/R** | I | I | C | I | I |
| Platform onboarding and enablement | I | I | I | C | C | I | **A/R** | C | C | I | I | I |
| Manage governance framework and policy lifecycle | C | C | C | I | I | I | I | I | I | **A/R** | C | C |
| Coordinate stewardship and governance operations | I | I | C | C | I | I | I | I | I | **A/R** | I | I |
| Report quality issues | I | I | C | C | I | C | I | I | **R** | C | I | I |
| Backup and disaster recovery | I | I | I | I | **A/R** | I | C | I | I | I | I | I |
| Assess privacy and PI handling | I | C | C | I | I | I | I | I | I | C | **A/R** | I |
| Classify AI model risk | I | C | C | I | I | C | I | I | I | C | I | **A/R** |
| Maintain AI model inventory | I | I | C | I | I | C | I | I | I | C | I | **A/R** |
| Approve third-party AI service usage | C | C | I | I | I | C | I | I | I | C | **C** | **A/R** |
| Assess AI training data governance | I | **A** | **R** | C | I | C | I | C | I | C | **C** | **R** |

**A** = Accountable, **R** = Responsible, **C** = Consulted, **I** = Informed

---

## Anti-Patterns to Avoid

**"The Data Owner who is really a Custodian."** Assigning database administrators as Data Owners because they "look after the data." Ownership is a business accountability — it cannot be delegated to IT by default.

**"Stewardship as a part-time afterthought."** Naming someone a Data Steward without allocating time, authority, or tooling. Stewardship requires dedicated capacity and executive backing.

**"One Steward type fits all."** Conflating Domain Stewards and Technical Stewards into a single role. They require fundamentally different skill sets — business knowledge vs platform engineering.

**"Governance without Producers."** Defining governance roles but never making source system teams accountable for the quality of data they emit. Governance cannot be retrofitted downstream — it starts at the source.

**"The CDO as sole governor."** Expecting the CDO to personally steward all data. The CDO sets the operating model; domain-level roles do the work.

**"Data products without a product owner."** Publishing datasets to the gold layer and calling them data products, but with no one accountable for their contract, quality, consumer experience, or lifecycle. Without a product owner, data products decay into unmanaged extracts.

**"Platform ownership as infrastructure only."** Treating the data platform as a pure ops concern with no strategic ownership. Without a Platform Owner driving capability evolution, standards, and developer experience, the platform becomes a bottleneck rather than an enabler.

---

## References and Alignment

This role model aligns with principles from the following frameworks and standards:

- **DAMA-DMBOK2** (2017) — foundational role definitions for data ownership, stewardship, and custodianship.
- **EDM Council DCAM v2.2** — data management capability maturity, including role-based accountability.
- **ISO 8000** — data quality management standards, informing quality-related responsibilities.
- **ISO/IEC 42001:2023** — AI management systems, informing the AI/ML Governance Lead role.
- **NIST AI Risk Management Framework (AI RMF 1.0)** — AI governance principles including human oversight, transparency, and ongoing monitoring.
- **Australian Government Voluntary AI Safety Standard** (2024) — Australian-specific AI governance guidance, which may become mandatory.
- **Data Mesh** (Dehghani, 2022) — domain-oriented ownership, data-as-a-product, and federated computational governance patterns.
- **Privacy Act 1988 (Cth)**, **PRIS Act 2024 (WA)**, **SOCI Act 2018 (Cth)** — regulatory foundations for privacy and critical infrastructure obligations.

Specific role boundaries should be adapted to organisational context, regulatory environment, and platform capabilities.

---

## Application at Water Corporation

This role model is applied at Water Corporation through the Enterprise Data & Analytics Platform (EDAP) and the federated domain governance framework:

**Domain Ownership Structure:** Water Corporation's seven data domains — Customer, Asset, Operations, Finance, Legal & Compliance, People, and Technology & Digital — each have an executive Data Owner and embedded Data Domain Stewards. This aligns with the hybrid governance model recommended above.

**Platform Roles in Practice:**
- **Data Platform Owner** — The Architecture & Strategy function within Digital & Technology owns the EDAP platform roadmap, Unity Catalog standards, pipeline framework patterns, and developer experience.
- **Data Custodian** — The platform operations team manages infrastructure, serverless compute, backup, disaster recovery, and security patching for the Databricks environment.
- **Technical Data Stewards** — Embedded within domain engineering teams, they implement governance rules (governed tags, ABAC policies, data quality expectations) in Unity Catalog and Lakeflow Spark Declarative Pipelines.
- **Data Domain Stewards** — Senior business SMEs within each domain who manage business definitions in the data catalogue, validate classification decisions, and triage data quality issues.
- **Data Product Owners** — Accountable for certified Gold-layer data products published to domain BI schemas (e.g. `prod_gold.asset_bi`, `prod_gold.customer_bi`), including data contracts, FAUQD certification, and consumer experience.

**Governance Forum:** The Data Governance Council serves as the cross-functional forum for policy ratification, domain boundary disputes, and governance programme oversight, chaired by the Chief Data Officer or delegate.

**Regulatory Context:** The Data Protection Officer role carries additional responsibilities under the PRIS Act 2024 (WA) for personal information handling and under the SOCI Act 2018 (Cth) for critical infrastructure data incident reporting. These are detailed in the companion EDAP Tagging Strategy, EDAP Access Model, and Governing Data Across Source Systems and EDAP documents.

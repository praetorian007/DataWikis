# Core Data Governance Roles in the Enterprise

## Purpose

This document defines the core data governance roles found in enterprise data governance frameworks. It clarifies accountability boundaries, decision rights, and the relationships between roles â particularly the commonly conflated distinctions between Data Owner, Data Custodian, Data Domain Steward, and Technical Data Steward.

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

**Decision rights:** Strategic direction, governance model design, programme funding.

---

### Data Governance Council

A cross-functional leadership forum that ratifies policy, resolves cross-domain disputes, and provides oversight of the governance programme. Not a single person â a standing body.

**Key responsibilities:**

- Approve enterprise data policies, standards, and classifications.
- Resolve cross-domain ownership disputes and priority conflicts.
- Review and endorse the data domain model and domain ownership assignments.
- Monitor governance KPIs (quality scorecards, compliance metrics, adoption).
- Ensure alignment between data governance and enterprise architecture, security, and risk.

**Typical membership:** CDO (chair), Data Owners, CISO representative, Enterprise Architecture, Legal/Compliance, and selected Data Domain Stewards as subject matter advisors.

---

### Data Owner

The Data Owner is the **business executive accountable** for a defined data domain. This is an accountability role, not a hands-on operational role. The Data Owner does not touch data day-to-day â they make decisions *about* data.

**Key responsibilities:**

- Approve who may access data within their domain (access policy).
- Define acceptable quality thresholds and escalation paths.
- Approve data classification and sensitivity labels.
- Sign off on data sharing agreements and cross-domain data flows.
- Nominate and empower Data Domain Stewards to act on their behalf.
- Accept residual data risk within their domain.

**Analogy:** The Data Owner is like a property owner. They decide who gets a key, what insurance to carry, and what renovations to approve â but they do not personally mow the lawn or fix the plumbing.

**Common misconception:** Data Owners do not need to understand the technical platform. Their authority is *business* authority â they decide *what* and *why*, not *how*.

**Decision rights:** Access approvals, classification sign-off, quality thresholds, data sharing agreements.

---

### Data Domain Steward

The Data Domain Steward is the **operational governance lead** within a data domain. They are the Data Owner's delegate â a senior subject matter expert who understands both the business meaning and the governance rules that apply to the domain's data.

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
- Support pipeline change management â assess impact of schema evolution on downstream consumers.
- Operationalise data retention and purging policies.

**Analogy:** The Technical Data Steward is the building engineer. They install the locks the property manager specifies, wire up the alarms, run the inspections, and make sure the infrastructure enforces the rules.

**Relationship to Data Engineer:** A Technical Data Steward may *be* a data engineer, or may work alongside them. The distinction is one of focus: the data engineer builds pipelines for throughput and correctness; the Technical Data Steward ensures those pipelines enforce governance policy. In smaller teams, one person often wears both hats.

**Decision rights:** Implementation approach for governance controls, technical standards for metadata and quality automation.

---

### Data Custodian

The Data Custodian is responsible for the **safe operational management** of the infrastructure on which data resides. They do not decide *what* the data means or *who* should access it â those are Owner and Steward responsibilities. The Custodian ensures the platform is secure, available, backed up, and performing.

**Key responsibilities:**

- Manage platform infrastructure (compute, storage, networking, patching).
- Implement and maintain backup, disaster recovery, and business continuity for data stores.
- Enforce infrastructure-level security controls (encryption at rest and in transit, network segmentation, identity integration).
- Monitor platform health, performance, and cost.
- Execute environment provisioning and decommissioning.
- Ensure platform compliance with security frameworks (e.g. Essential Eight, SOCI Act critical infrastructure requirements).

**Analogy:** The Data Custodian is the building's facilities management team. They keep the lights on, the doors locked, and the fire systems operational â but they do not decide which tenants move in or what happens inside each office.

**Common misconception:** "Custodian" is sometimes loosely used to mean anyone who looks after data. In a governance framework, the Custodian's scope is strictly *infrastructure and platform operations*, not business meaning or access policy.

**Decision rights:** Infrastructure configuration, backup schedules, platform security controls.

---

### Data Product Owner

The Data Product Owner is accountable for the **end-to-end value, quality, and lifecycle** of a specific data product. This role emerges from treating data as a product â something with consumers, an SLA, a contract, and a roadmap. It is distinct from the Data Owner (who owns a domain) and the Data Producer (who builds and publishes). The Data Product Owner sits between them, ensuring the product delivers sustained value.

**Key responsibilities:**

- Define the data product's purpose, scope, target consumers, and success metrics.
- Own the data contract â schema, freshness SLA, quality thresholds, and versioning policy.
- Prioritise the product backlog: enhancements, quality improvements, new consumer requirements.
- Ensure the product is discoverable, well-documented, and registered in the data catalogue.
- Monitor product adoption, consumer satisfaction, and quality scorecards.
- Coordinate across producers (who supply the data), technical stewards (who enforce governance), and consumers (who derive value).
- Manage the product lifecycle â from incubation through active service to deprecation and retirement.

**Relationship to Data Owner:** The Data Owner is accountable for the *domain*. The Data Product Owner is accountable for a *specific product* within that domain. A single Data Owner may have multiple Data Product Owners reporting into their domain. The Data Owner sets the policies and quality bar; the Data Product Owner delivers against them.

**Relationship to Data Producer:** The Data Producer builds and publishes the data. The Data Product Owner decides *what* gets built, *for whom*, and *to what standard*. In smaller teams these roles may overlap, but the accountability is distinct â the Producer is responsible for engineering delivery, the Product Owner for value delivery.

**Analogy:** If the Data Owner is the property owner and the Domain Steward is the property manager, the Data Product Owner is a business operator running a specific venue within the building â they decide the menu, the service level, and the customer experience, while operating within the building's rules.

**Decision rights:** Product scope and roadmap, contract terms, consumer prioritisation, product lifecycle decisions (publish, version, deprecate).

---

### Data Platform Owner

The Data Platform Owner is accountable for the **data platform as an enabling capability** â its architecture, evolution, developer experience, and fitness for purpose. Where the Data Custodian focuses on operational health (keeping the lights on), the Data Platform Owner focuses on platform strategy and enablement (making the platform worth using).

**Key responsibilities:**

- Define and maintain the platform roadmap aligned to enterprise data strategy.
- Own platform architecture decisions â tooling choices, integration patterns, environment strategy.
- Ensure the platform provides self-service capabilities that enable domain teams to build and govern data products autonomously.
- Define and enforce platform standards â naming conventions, tagging taxonomies, pipeline frameworks, security patterns.
- Manage platform onboarding â making it easy for new domains, producers, and consumers to adopt the platform.
- Own platform cost management, capacity planning, and vendor relationships.
- Coordinate with the Data Custodian on operational concerns and with Technical Data Stewards on governance enforcement.
- Represent platform capabilities and constraints to the Data Governance Council and enterprise architecture.

**Relationship to Data Custodian:** The Custodian keeps the platform running day-to-day. The Platform Owner decides where the platform is going. The Custodian handles patching, backups, and incident response; the Platform Owner handles capability roadmap, developer experience, and architectural evolution. In practice, they work hand-in-glove.

**Relationship to Technical Data Steward:** Technical Data Stewards implement governance rules *within* the platform. The Platform Owner ensures the platform *provides the mechanisms* for that governance â Unity Catalog policies, tagging frameworks, quality rule engines, lineage capture. The Platform Owner builds the guardrails; the Technical Data Stewards configure them per domain.

**Analogy:** If the Data Custodian is facilities management, the Data Platform Owner is the building developer â they decide what amenities the building offers, what the floor plan looks like, and how the building evolves over time to attract and serve tenants.

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

### Data Protection Officer (DPO)

The DPO provides independent oversight of personal and sensitive information handling. In Australian government contexts, this role ensures compliance with the Privacy Act, PRIS Act, and equivalent state legislation.

**Key responsibilities:**

- Advise on privacy impact assessments and data protection obligations.
- Monitor compliance with personal information handling requirements.
- Act as the point of contact for regulators and data subjects.
- Review data sharing agreements involving personal or sensitive information.
- Ensure privacy-by-design principles are embedded in data platform architecture.

**Decision rights:** Privacy compliance assessments, regulatory reporting, privacy risk escalation.

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

---

## RACI Summary

| Activity | CDO | Data Owner | Domain Steward | Technical Steward | Custodian | Product Owner | Platform Owner | Producer | Consumer |
|---|---|---|---|---|---|---|---|---|---|
| Set data strategy | **A/R** | C | C | I | I | I | C | I | I |
| Define access policy | I | **A** | **R** | C | I | C | I | I | I |
| Approve data classification | C | **A** | **R** | C | I | I | I | I | I |
| Maintain business glossary | I | C | **A/R** | C | I | C | I | C | C |
| Implement quality rules | I | I | **A** | **R** | I | C | I | C | I |
| Configure platform access controls | I | I | C | **A/R** | C | I | C | I | I |
| Manage infrastructure security | I | I | I | C | **A/R** | I | C | I | I |
| Define data product scope and contract | I | **A** | C | C | I | **R** | I | C | C |
| Publish data products | I | C | C | C | I | **A** | I | **R** | I |
| Manage data product lifecycle | I | C | C | I | I | **A/R** | I | C | C |
| Define platform roadmap | C | I | I | C | C | C | **A/R** | I | I |
| Set platform standards and conventions | I | I | I | C | C | I | **A/R** | I | I |
| Platform onboarding and enablement | I | I | I | C | C | I | **A/R** | C | C |
| Report quality issues | I | I | C | C | I | C | I | I | **R** |
| Backup and disaster recovery | I | I | I | I | **A/R** | I | C | I | I |

**A** = Accountable, **R** = Responsible, **C** = Consulted, **I** = Informed

---

## Anti-Patterns to Avoid

**"The Data Owner who is really a Custodian."** Assigning database administrators as Data Owners because they "look after the data." Ownership is a business accountability â it cannot be delegated to IT by default.

**"Stewardship as a part-time afterthought."** Naming someone a Data Steward without allocating time, authority, or tooling. Stewardship requires dedicated capacity and executive backing.

**"One Steward type fits all."** Conflating Domain Stewards and Technical Stewards into a single role. They require fundamentally different skill sets â business knowledge vs platform engineering.

**"Governance without Producers."** Defining governance roles but never making source system teams accountable for the quality of data they emit. Governance cannot be retrofitted downstream â it starts at the source.

**"The CDO as sole governor."** Expecting the CDO to personally steward all data. The CDO sets the operating model; domain-level roles do the work.

**"Data products without a product owner."** Publishing datasets to the gold layer and calling them data products, but with no one accountable for their contract, quality, consumer experience, or lifecycle. Without a product owner, data products decay into unmanaged extracts.

**"Platform ownership as infrastructure only."** Treating the data platform as a pure ops concern with no strategic ownership. Without a Platform Owner driving capability evolution, standards, and developer experience, the platform becomes a bottleneck rather than an enabler.

---

## References and Alignment

This role model aligns with principles from DAMA-DMBOK2, the EDM Council's DCAM framework, and common federated governance patterns used in data mesh and lakehouse architectures. Specific role boundaries should be adapted to organisational context, regulatory environment, and platform capabilities.

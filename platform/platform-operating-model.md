# Platform Operating Model

**Mark Shaw** | Principal Data Architect

---

## Introduction

The Enterprise Data & Analytics Platform (EDAP) exists to make data engineering, business intelligence, and data science productive, governed, and safe. The three discipline lifecycles — Data Engineering, BI, and Data Science — describe how practitioners do their work. The Data Governance Lifecycle describes how data is managed as a strategic asset. This document describes how the **platform team** operates the shared infrastructure and services that all of those lifecycles depend on.

A platform operating model is not a lifecycle in the traditional sense. Lifecycles describe repeatable stages of practice. An operating model describes how a team organises itself to deliver and sustain a product — in this case, a data platform. It answers: what do we provide, to whom, how do we prioritise, how do we support, and how do we improve?

This document is structured around six pillars: **Platform as a Product, Service Catalogue, Onboarding, Support and Escalation, Governance Operations, and Continuous Improvement**.

---

## Platform as a Product

### Philosophy

The platform team operates EDAP as an **internal product**. Domain teams and discipline practitioners are its customers. The platform's success is measured not by the sophistication of its infrastructure but by the speed and confidence with which its customers can deliver data products, dashboards, and models.

This means the platform team thinks in terms of:

- **Paved roads** — standardised, well-documented paths that make the right thing easy and the wrong thing hard
- **Self-service** — domain teams should be able to build, govern, and publish data products without raising a ticket for every step
- **Guardrails over gates** — governance is enforced through automation (ABAC policies, governed tags, data contracts) rather than manual approval queues

### Organisational Context

Three types of team interact with the platform:

| Team Type | Examples | Relationship to Platform |
|---|---|---|
| **Data domain teams** | Asset, Customer, Water, Finance, People, Operations, Spatial | Own data assets within their domain. Steward, classify, and govern. Use platform self-service capabilities to build and publish data products |
| **Discipline practitioners** | Data engineers, BI analysts, data scientists | Execute the three discipline lifecycles. Use platform tooling, frameworks, and compute to do their work |
| **Platform team** | Platform engineers, platform owner, data custodian | Build, operate, and evolve the shared platform. Provide infrastructure, frameworks, governance automation, and support |

Domain teams hold **MANAGE privilege** at schema level in Unity Catalog — they create, alter, and govern tables within their domain boundary. The platform team retains **catalog-level administration** and manages cross-cutting concerns: identity, security, compute, and governance infrastructure.

### Decision Rights

| Decision | Who Decides |
|---|---|
| Platform architecture, tooling, environment strategy | Platform Owner |
| Domain data product design, schema, quality rules | Domain Data Steward + Data Engineers |
| Classification and sensitivity tagging | Domain Data Steward (within central taxonomy) |
| Cross-domain governance disputes | Data Governance Council |
| New platform capability prioritisation | Platform Owner, informed by domain feedback |
| Access policy design (ABAC rules) | Domain Data Steward proposes; platform team implements |
| Break-glass emergency access | Platform team executes; logged and reviewed |
| Cost allocation and compute sizing | Platform team sets policies; domain teams operate within them |

---

## Service Catalogue

The platform team provides the following services, grouped by the discipline lifecycles they support.

### Foundation Services (All Lifecycles)

| Service | Description |
|---|---|
| **Unity Catalog Administration** | Metastore management, catalog provisioning, workspace bindings, privilege model enforcement |
| **Identity and Access** | Account-level identity provisioning via SCIM from Azure AD, group management, service principal lifecycle |
| **ABAC Policy Engine** | Tag-driven attribute-based access control enforced at query time, including row and column masking |
| **Governed Tag Taxonomy** | Four-layer tag taxonomy (WAICP classification, sensitivity, access model, EDAP operational) with validation and deployment |
| **Audit and Compliance** | System table logging of all data access and changes, compliance reporting for PRIS Act, SOCI Act, State Records Act, Essential Eight |
| **Environment Management** | Isolated development, staging, and production workspaces (wc-edap-dev, wc-edap-staging, wc-edap-prod) with controlled promotion |
| **Compute Management** | Cluster policies, SQL Warehouse provisioning, serverless compute, cost controls and right-sizing |
| **Infrastructure Operations** | Backup, disaster recovery, network security, encryption, monitoring, and capacity planning in AWS Sydney region |

### Data Engineering Services

| Service | Description |
|---|---|
| **Metadata-Driven Pipeline Framework** | Configuration-first ingestion framework — domain teams declare sources and mappings, the framework handles execution |
| **Lakeflow Connect** | Managed source connectors for common enterprise systems |
| **Lakeflow Spark Declarative Pipelines** | Standard transformation framework for building medallion layer pipelines in SQL or Python |
| **Data Quality Monitoring** | Schema-level anomaly detection, data profiling, freshness and completeness monitoring, per-domain quality dashboards |
| **Data Contract Infrastructure** | Tooling to define, validate, and enforce producer-consumer contracts (schema, quality thresholds, SLAs, versioning) |
| **CI/CD for Pipelines** | Version control integration, automated testing frameworks, Databricks Asset Bundles for infrastructure as code, dev-to-prod promotion |

### BI and Analytics Services

| Service | Description |
|---|---|
| **SQL Warehouses** | Serverless compute endpoints optimised for analytical queries and dashboard workloads |
| **Semantic Layer (UC Metrics)** | Unity Catalog Metrics as first-class catalog objects — define once, consume across dashboards, Genie, ad hoc queries, and AI agents |
| **AI/BI Dashboards and Genie** | Dashboard hosting, Genie space infrastructure for natural language query |
| **Databricks Apps** | Framework for building and hosting custom data applications |
| **Direct Lake Connectivity** | Power BI Direct Lake mode integration for high-performance dashboard delivery |
| **Embedding and External Access** | Authentication and embedding infrastructure for serving analytics to external users |

### Data Science and AI Services

| Service | Description |
|---|---|
| **Model Serving** | Serverless inference endpoints with role-based access control |
| **Feature Store** | Unity Catalog feature tables with quality metrics and lineage |
| **Vector Search** | Infrastructure for retrieval-augmented generation (RAG) pipelines |
| **AI Agent Framework** | Deployment infrastructure for AI agents and assistive applications |
| **AI Gateway** | Centralised foundation model access with route management and guardrails |
| **Online Tables** | Low-latency serving tables for real-time model inference |
| **MLflow and Model Registry** | Experiment tracking, model versioning, and governance metadata |

### Catalogue and Discovery Services

| Service | Description |
|---|---|
| **Alation Integration** | Catalogue synchronisation — lineage, contracts, quality scores, and business definitions flow from Unity Catalog to Alation |
| **Data Product Registry** | FAUQD certification (Findable, Accessible, Understandable, Governed, Documented) and publication workflow |
| **Delta Sharing** | External data sharing infrastructure for cross-organisational collaboration |
| **Lakehouse Federation** | Query external data sources without data movement |

---

## Onboarding

### New Domain Onboarding

When a new data domain is brought onto the platform, the platform team and domain leadership work through the following steps:

| Step | Owner | Activity |
|---|---|---|
| 1. Domain ownership | Domain executive | Identify domain executive owner and assign data ownership accountability |
| 2. Stewardship | Domain executive + governance | Recruit and train Domain Data Steward(s) |
| 3. Identity setup | Platform team | Create domain-specific groups in Azure AD (stewards, engineers, analysts, scientists) |
| 4. Schema provisioning | Platform team | Provision domain schemas across dev, staging, and prod catalogs |
| 5. Privilege assignment | Platform team | Grant MANAGE privilege to domain steward group at schema level |
| 6. Standards briefing | Platform team | Walk through naming conventions, tagging taxonomy, quality standards, and data contract templates |
| 7. First data product | Domain team + platform team | Build first data product using provided frameworks — platform team provides hands-on guidance |
| 8. Certification | Domain team | Complete FAUQD certification and register in Alation |

### New Source System Onboarding

| Step | Owner | Activity |
|---|---|---|
| 1. Identification | Domain steward | Identify new source system and business justification |
| 2. Assessment | Platform team | Assess ingestion approach — Lakeflow Connect, Auto Loader, API-based, or file-based |
| 3. Configuration | Domain team + platform team | Provide connection details, schema mapping, refresh requirements |
| 4. Pipeline setup | Platform team | Configure ingestion in metadata-driven framework; data lands in Bronze Landing then persists in Bronze Raw |
| 5. Transformation | Domain engineers | Build Silver and Gold transformations using Lakeflow SDP |
| 6. Governance | Technical steward | Apply governed tags, validate data contracts, set quality rules |
| 7. Publication | Domain team | Certify data product and publish to domain BI schema |

### Discipline Practitioner Onboarding

| Discipline | What the Platform Team Provides |
|---|---|
| **Data engineers** | Workspace access, pipeline framework documentation, SDP templates, CI/CD patterns, cluster policies, and office hours support |
| **BI analysts** | SQL Warehouse access, semantic layer documentation, dashboard templates, Genie space provisioning, and Direct Lake configuration guidance |
| **Data scientists** | Compute access, Feature Store documentation, Model Serving setup guides, AI Gateway access, and MLflow workspace configuration |

---

## Support and Escalation

### Support Model

The platform team operates a **tiered support model** that balances self-service with expert assistance.

| Tier | Scope | Channel | Response |
|---|---|---|---|
| **Tier 0 — Self-service** | Documentation, templates, runbooks, worked examples | Platform wiki, Alation | Immediate |
| **Tier 1 — Domain resolution** | Issues within domain boundary — pipeline failures, quality rule tuning, access requests within domain | Domain team handles internally | Domain team SLA |
| **Tier 2 — Platform support** | Issues requiring platform team involvement — compute problems, cross-domain access, new connector setup, governance automation issues | Platform support channel | Acknowledged within 4 business hours |
| **Tier 3 — Platform engineering** | Infrastructure incidents, security events, platform bugs, architectural changes | Platform team internal | Severity-dependent (see below) |

### Incident Severity

| Severity | Definition | Target Response | Target Resolution |
|---|---|---|---|
| **SEV-1 — Critical** | Platform unavailable or data breach. Production workloads blocked across multiple domains | 30 minutes | 4 hours |
| **SEV-2 — High** | Significant degradation. One domain's production workloads blocked or data quality SLA breach on enterprise-tier data product | 2 hours | 8 hours |
| **SEV-3 — Medium** | Non-blocking issue affecting productivity. Workaround available | 4 hours | 2 business days |
| **SEV-4 — Low** | Minor issue, cosmetic, or enhancement request | 1 business day | Prioritised into backlog |

### Escalation Path

1. **Domain team** attempts resolution using self-service documentation and domain knowledge
2. **Platform support** is engaged for issues outside domain scope
3. **Platform Owner** is escalated to for priority conflicts or architectural decisions
4. **Data Governance Council** resolves cross-domain disputes, classification conflicts, or policy exceptions
5. **Break-glass access** is executed by platform team for emergency situations — all actions logged, reviewed within 24 hours, and reported to the Data Governance Council

---

## Governance Operations

The platform team does not own governance — it **operationalises** governance. Domain teams and the Data Governance Council set policy; the platform team builds the mechanisms that enforce it.

### What the Platform Team Enforces

| Mechanism | How It Works |
|---|---|
| **ABAC policies** | Domain stewards define access rules using governed tags. Platform team translates these into Unity Catalog ABAC policies evaluated at query time |
| **Tag validation** | Platform team validates new tag values for Unity Catalog compatibility and deploys tag policy updates |
| **Classification lifecycle** | Objects progress from Unclassified → Provisional → Classified. Platform monitors classification gaps and alerts domain stewards when objects remain unclassified beyond SLA |
| **Data contract enforcement** | Platform provides contract validation tooling. Consumers are alerted when producers breach schema, quality, or freshness commitments |
| **Audit logging** | All data access, grants, and schema changes logged to system tables. Platform team maintains dashboards for compliance reporting |
| **Retention enforcement** | Platform implements retention and purging policies defined by domain stewards, aligned with State Records Act requirements |

### Governance Cadence

| Activity | Frequency | Participants |
|---|---|---|
| Domain stewardship review | Monthly | Domain steward, platform team representative |
| Data quality review | Monthly | Domain steward, data engineers, platform team |
| Data Governance Council | Quarterly | Domain stewards, platform owner, data custodian, governance lead |
| Tag taxonomy review | Quarterly | Platform team, domain stewards, governance lead |
| Access and compliance audit | Quarterly | Platform team, information security, compliance |
| Platform roadmap review | Quarterly | Platform owner, domain representatives, discipline leads |

---

## Continuous Improvement

### Platform Roadmap

The platform roadmap is managed as an **internal product backlog**, informed by:

- **Domain feedback** — feature requests, friction points, and capability gaps reported through support channels and governance forums
- **Discipline needs** — emerging patterns from the data engineering, BI, and data science communities
- **Technology evolution** — new Databricks capabilities, vendor updates, and industry best practice
- **Compliance requirements** — regulatory changes requiring new controls or reporting

The Platform Owner prioritises the roadmap, balancing domain-specific requests against platform-wide improvements. Prioritisation criteria include:

1. **Breadth of impact** — how many domains and practitioners benefit
2. **Self-service enablement** — does this reduce platform team dependency
3. **Governance and compliance** — is this required by regulation or policy
4. **Operational risk** — does this reduce incident likelihood or blast radius

### Metrics

The platform team tracks its effectiveness through four categories of metric:

| Category | Example Metrics |
|---|---|
| **Adoption** | Active domain teams, data products published, practitioners onboarded, self-service pipeline creation rate |
| **Reliability** | Platform uptime, incident count by severity, mean time to resolution, SLA adherence |
| **Efficiency** | Time from source system request to first data product, compute cost per domain, support ticket volume and resolution time |
| **Governance** | Classification coverage (% of objects classified), tag completeness, data contract compliance rate, audit finding closure rate |

### Feedback Loops

| Loop | Purpose |
|---|---|
| **Support ticket analysis** | Identify recurring issues that indicate missing self-service capability or documentation gaps |
| **Onboarding retrospectives** | After each new domain or source system onboarding, capture what worked and what to improve |
| **Quarterly domain surveys** | Structured feedback from domain teams on platform experience, friction points, and unmet needs |
| **Platform blameless post-incident reviews** | After SEV-1 and SEV-2 incidents, review root cause, response effectiveness, and preventive actions |

---

## Summary

The platform operating model positions the platform team as an **enabler**, not a bottleneck. Domain teams own their data. Discipline practitioners own their craft. The platform team owns the infrastructure, frameworks, and governance automation that make both productive and safe.

| Principle | What It Means in Practice |
|---|---|
| **Platform as a product** | Domain teams and practitioners are customers. Their productivity is the measure of success |
| **Self-service by default** | If a domain team needs to raise a ticket, the platform has a gap to close |
| **Guardrails over gates** | Governance is automated and embedded, not a manual approval queue |
| **Federated ownership** | Central standards, domain enforcement. The platform team sets the rules of the road; domain teams drive |
| **Continuous improvement** | The platform evolves through structured feedback, measured outcomes, and regular roadmap review |

# EDAP Enterprise Architecture Vision

**Mark Shaw** | Principal Data Architect
**Audience:** PI Planning — All Teams
**Date:** March 2026
**Classification:** OFFICIAL

---

## 1. Why We Are Here

Water Corporation is a complex, asset-intensive utility operating under increasing regulatory scrutiny, growing customer expectations, and a mandate to do more with less. Our ability to act on data — to predict asset failures before they occur, to serve customers with precision, to meet SOCI and PRIS obligations without friction — is currently constrained by a fragmented data landscape built on legacy platforms that were never designed to work together.

We have data. What we lack is the infrastructure to turn it into decisions at pace.

The Enterprise Data & Analytics Platform (EDAP) is our answer. This PI planning session is about committing to the work that builds that answer — not in theory, but in production.

---

## 2. The Problem We Are Solving

Across Water Corporation today:

- Source systems — SAP, Maximo, SCADA, GIS, Salesforce — operate in silos. Cross-domain questions require manual effort, spreadsheets, and days of waiting.
- Data pipelines exist in at least four technologies (AWS Glue, ADF, SAP Data Services, SageMaker) with no common operating model, no shared quality standard, and no unified lineage.
- Sensitive data — customer PI, SCADA operational telemetry, critical infrastructure assets — is governed inconsistently. Classification is manual, access control is coarse-grained, and audit trails are incomplete.
- Analytics teams spend the majority of their time acquiring and cleaning data rather than generating insight. Data scientists build models on data they cannot fully trust, and deploy them on infrastructure they do not control.
- Knowledge sits in individuals, not in platforms. When people leave, patterns leave with them.

This is not a technology problem. It is an architecture problem — the absence of a coherent, governed, scalable data platform that all of Water Corporation can depend on.

EDAP is the resolution.

---

## 3. The Target State

By the end of this programme, Water Corporation will operate a **unified, governed, cloud-native data and analytics platform** on Databricks, structured around five capabilities that did not exist before:

### 3.1 A Single Platform, Seven Domains

All seven business domains — Asset, Customer & Billing, Network & Infrastructure, Operations, Workforce & Safety, Finance & Commercial, Environment & Compliance — will have a governed home in EDAP. Each domain operates with clear ownership, federated autonomy within central guardrails, and a direct path from source system to consumable data product.

### 3.2 Data That Governs Itself

Classification, access control, and compliance will be **embedded in the platform**, not applied as afterthoughts. Every table will carry governed tags — WAICP classification, PI indicators, SOCI criticality, domain ownership. Those tags will drive ABAC policies automatically. A steward classifies data once in Alation; Databricks enforces it everywhere, immediately, without manual intervention.

### 3.3 Pipelines Built to Last

Ingestion and transformation will follow a **metadata-driven, configuration-first framework**. Onboarding a new source system will be a configuration task, not a coding project. Pipelines will be deployed as code through automated CI/CD, promoted through dev → staging → prod with automated testing gates, and monitored continuously with drift detection and self-healing patterns.

### 3.4 Data Products, Not Data Dumps

The Gold layer will not be a warehouse. It will be a catalogue of **certified data products** — each with a data contract, a quality guarantee, an owner, and a consumer registry. Teams will discover data through Alation, understand it through lineage and business context, and trust it because it has passed the Five Gates: Findable, Accessible, Understandable, Quality-assured, and Dependable.

### 3.5 AI and ML as a First-Class Citizen

Advanced analytics will not be an experiment running on the side. MLOps will be a production capability — experiment tracking, feature engineering, model registry, model serving, and drift monitoring — running on the same platform as the data it consumes. The OMLIX algorithms in the Customer domain will be the first production proof point. The capability will scale to every domain that follows.

---

## 4. Architectural Principles

These are not preferences. They are the guardrails within which all delivery teams will operate.

| Principle | What It Means in Practice |
|---|---|
| **Govern once, enforce everywhere** | Access policies are defined in Unity Catalog and applied automatically. No per-table row filters. No ad-hoc grants. Tags drive everything. |
| **Configuration over code** | New source onboarding uses metadata-driven frameworks. Custom pipeline code is the exception, not the default. |
| **Medallion means something** | Bronze is raw and append-only. Silver is clean and domain-owned. Gold is certified and contractual. Nothing skips a layer. |
| **Open by default, restricted by classification** | Data is accessible unless classification demands otherwise. Restrictions are explicit, tagged, and enforced — not assumed. |
| **Infrastructure as code** | All platform assets — catalogs, schemas, pipelines, policies, compute — are deployed via Databricks Asset Bundles through CI/CD. Nothing is provisioned manually in production. |
| **PI means PI, not PII** | We follow the PRIS Act 2024 (WA). Personal Information, not Personally Identifiable Information. This is a regulatory requirement, not a style choice. |
| **Non-prod costs ≤10% of production** | Test and development environments use synthetic data, sampled datasets, and serverless compute. Runaway non-prod spend is an architecture failure, not a budget issue. |
| **The SI walks out; the capability stays** | Every pattern, framework, and decision must be documented, trained, and owned by Water Corporation staff before go-live. Knowledge transfer is not a workstream — it is a delivery condition. |

---

## 5. Programme Structure: Nine Epics, One Platform

The delivery is organised into nine epics. They are not independent tracks — they are a dependency chain with a foundation at the bottom and value at the top.

```
                        ┌─────────────────────────────┐
                        │  E7: Analytics & AI          │
                        │  E8: Migration Pathways      │
                        └────────────┬────────────────┘
                                     │
                        ┌────────────▼────────────────┐
                        │  E6: Discovery & Catalogue   │
                        └────────────┬────────────────┘
                                     │
                        ┌────────────▼────────────────┐
                        │  E3: Transformation &        │
                        │  Data Products               │
                        └────────────┬────────────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                       │
┌─────────────▼──────┐  ┌────────────▼─────────┐  ┌────────▼────────────┐
│  E2: Ingestion &   │  │  E4: Security,        │  │  E5: DataOps &      │
│  Integration       │  │  Governance &         │  │  Engineering        │
│                    │  │  Compliance           │  │  Excellence         │
└─────────────┬──────┘  └────────────┬─────────┘  └────────┬────────────┘
              └──────────────────────┼──────────────────────┘
                                     │
                        ┌────────────▼────────────────┐
                        │  E1: Platform Foundation     │
                        │  (Nothing moves without this)│
                        └─────────────────────────────┘

          E9: Capability Uplift & Governance Advisory (runs in parallel with all epics)
```

### Epic Summary

| Epic | Outcome | Architectural Significance |
|---|---|---|
| **E1** Platform Foundation | Environments operational, governed, and ready | Unity Catalog, workspace topology, identity, compute — the non-negotiable prerequisite |
| **E2** Data Ingestion & Integration | Data flowing from all source types into Bronze | Configuration-first framework; CDC; streaming; PI tagging at point of entry |
| **E3** Transformation & Data Products | Clean, certified Gold-layer data products | Metadata-driven Silver→Gold; data contracts; Five Gates certification |
| **E4** Security, Governance & Compliance | Automatic, tag-driven protection; audit on demand | ABAC; Alation→UC tag sync; SOCI/PRIS/State Records compliance posture |
| **E5** DataOps & Engineering Excellence | Automated, safe, cost-efficient delivery | DABs-based CI/CD; automated testing; dev/staging/prod promotion; ≤10% non-prod cost |
| **E6** Discovery & Catalogue | Every asset discoverable with full context | Alation OCF integration; end-to-end lineage; metadata auto-currency |
| **E7** Analytics & AI | Full MLOps capability; OMLIX in production | MLflow; Feature Engineering; Model Serving; drift monitoring; OMLIX go-live |
| **E8** Migration Pathways | Clear, costed migration from Glue, ADF, SAP DS, SageMaker | Pattern documentation; migration playbooks; use case data serving from EDAP |
| **E9** Capability Uplift | WC team self-sufficient on day one after go-live | Role-based learning; certification; embedded shadowing; runbooks; code review ownership |

---

## 6. What This PI Needs to Achieve

This Programme Increment focuses on **getting the foundation right so that everything else can move**. The architectural priority sequence for this PI is:

1. **Platform Foundation complete** (E1) — environments provisioned, Unity Catalog live, identity integrated, compute policies set, Git-to-Databricks CI/CD operational. Until E1 is done, every other team is blocked.

2. **DataOps skeleton operational** (E5) — branching strategy, DABs deployment model, automated testing gates, and environment cost controls in place. Teams should be building on a deployment pipeline from sprint one, not retrofitting it later.

3. **Ingestion framework architecture proven** (E2) — the metadata-driven configuration model validated end-to-end on at least one source system. CDC and batch patterns proven before use case delivery begins.

4. **Governance and tagging model activated** (E4) — governed tags registered in Unity Catalog, ABAC policies applied to at least one domain catalog, Alation OCF connector harvesting metadata. The classification lifecycle must be live from the moment the first source system lands.

5. **Team self-sufficiency initiated** (E9) — Water Corporation engineers shadowing from sprint one, in the PR review process, and tracking against role-based learning pathways. This is not a later-PI workstream.

---

## 7. Key Architectural Decisions Already Made

These decisions are **closed**. Teams should not re-litigate them — they should build within them.

| Decision | Rationale |
|---|---|
| **Single Unity Catalog metastore** (ap-southeast-2) | One metastore per region. Cross-domain sharing via privilege grants, not data duplication. |
| **Domain-based catalog naming** (`prod_<domain>`) | Primary isolation unit. Federated ownership. Supports workspace binding and ABAC at catalog level. |
| **Schemas represent medallion layers** (`raw`, `base`, `curated`, `product`, `sandbox`) | Clear layer semantics within each domain. No layer-jumping. |
| **ABAC over per-table grants** | Governed tags drive masking and row filters automatically. Scales to thousands of tables without manual policy management. |
| **Lakeflow Spark Declarative Pipelines (SDP)** | Streaming tables, materialised views, expectations, AUTO CDC. Serverless compute default. |
| **Databricks Asset Bundles (DABs) for all deployment** | IaC for pipelines, jobs, catalogs, schemas, and policies. Nothing deployed by hand in production. |
| **Alation + Unity Catalog dual-layer catalogue** | Alation is enterprise discovery (cross-system). Unity Catalog is technical enforcement (within Databricks). Both required; neither replaces the other. |
| **PI not PII; PRIS Act 2024 terminology** | Regulatory compliance. Tag key is `pi_category`. Documentation, training, and code must reflect this. |

---

## 8. Architectural Risks the PI Must Manage

| Risk | Consequence If Not Managed | Owner |
|---|---|---|
| E1 delays cascade to all epics | No environment = no delivery | Platform Team / E1 |
| ABAC preview stability | ABAC requires DBR 16.4+ or serverless. Classic cluster teams will be blocked. | Architecture |
| Alation↔UC tag sync not bidirectional | Governance-as-code breaks if sync direction is wrong or delayed | E4 / E6 |
| Non-prod cost discipline | Serverless without cost controls can exceed the ≤10% target quickly | E5 / Platform |
| Knowledge transfer treated as final-sprint activity | SI dependency risk at go-live. Teams unable to operate independently. | E9 / Programme |
| Migration scope creep | S22 is advisory only — actual migration is limited to use case data. Scope must not expand without change control. | Programme Manager |

---

## 9. The Measure of Success

At the end of this programme, an enterprise architect reviewing EDAP should be able to observe:

- **A single source of truth** — any authorised user can find, understand, and access any data asset from Alation or Unity Catalog, with full lineage from source system to consumption
- **Compliance by design** — a SOCI or PRIS audit can be satisfied from system tables and governance dashboards without manual evidence gathering
- **Speed of new use case delivery** — a new source system can be onboarded in days, not weeks, because the framework is configuration-first
- **A team that owns it** — Water Corporation engineers are leading pipeline reviews, managing their domain catalogs, and training the next cohort without SI involvement
- **AI that is trustworthy** — every model in production has a model card, a data lineage trail, a risk classification, and a human oversight checkpoint

That is the vision. This PI is the first significant step toward it.

---

## 10. What We Are Asking of Every Team

- **Build to the architecture, not around it.** If a guardrail seems wrong, raise it as an architectural decision — do not work around it silently.
- **Tag from day one.** Every table that lands in EDAP must carry mandatory tags at ingestion. Classification debt compounds.
- **Deploy as code.** If you cannot deploy it through DABs, it should not be in production.
- **Bring WC engineers with you.** Every feature, every pattern, every design decision — Water Corporation staff should be learning it as it is built.
- **Raise blockers early.** The dependency chain is tight. An unresolved E1 blocker on sprint two is a programme risk by sprint four.

---

*This vision document is the architectural context for PI planning. It does not replace feature-level acceptance criteria or sprint planning. It sets the direction within which all planning should occur.*

*Questions or challenges to architectural decisions should be raised through the Architecture Review process, not through workarounds in delivery.*

**Mark Shaw | Principal Data Architect | Water Corporation**

---

# EDAP Epics – Feature Mapping

**Delivery Method:** SAFe / Scrum
**Total Epics:** 9 | **Total Features:** 95

---

## Epic Overview

| Epic | Name | Outcome | Features |
|---|---|---|---|
| **E1** | Platform Foundation | EDAP environments operational, governed, and ready for development | 19 |
| **E2** | Data Ingestion & Integration | Data flowing reliably from source systems into EDAP | 14 |
| **E3** | Transformation & Data Products | Clean, business-ready data products available for consumption | 13 |
| **E4** | Security, Governance & Compliance | Data classified, protected, and compliant – automatically | 16 |
| **E5** | DataOps & Engineering Excellence | Automated, governed, cost-efficient delivery pipeline | 12 |
| **E6** | Discovery & Catalogue | Every data asset discoverable with full context and lineage | 7 |
| **E7** | Analytics & AI | BI-ready serving layer and MLOps capabilities operational | 6 |
| **E8** | Migration Pathways | Legacy workloads have a clear, proven path to EDAP | 6 |
| **E9** | Capability Uplift & Governance Advisory | WC team self-sufficient, governance frameworks endorsed | 17 |

---

## E1: Platform Foundation

**Outcome:** EDAP environments are operational, governed, and ready for teams to develop, test, and deploy data products with confidence.

**Why this epic exists:** Everything else depends on a stable, well-designed platform foundation. Without agreed architecture, provisioned environments, and identity management, no other work can proceed.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S2-F1 | Approved Environment and Workspace Topology | S2 |
| S2-F2 | Medallion Architecture and Storage Layout Agreed | S2 |
| S2-F5 | Unity Catalog Structure, Access Model, and Lineage Designed | S2 |
| S5-F1 | Three Isolated Environments Ready for Development | S5 |
| S5-F2 | Domain-Isolated Data Catalogues with Governed Discovery | S5 |
| S5-F3 | Right-Sized Compute Available on Demand | S5 |
| S5-F4 | Secrets Securely Available to Pipelines Without Hardcoding | S5 |
| S5-F5 | Users Provisioned Automatically from Corporate Identity | S5 |
| S5-F6 | Code Changes Flow from IDE to Databricks via Git | S5 |
| S5-F7 | Platform Health Visible and Alertable in Real Time | S5 |
| S8-F3 | Users Provisioned and Deprovisioned in Sync with Corporate Directory | S8 |
| S8-F4 | All Data Encrypted at Rest and in Transit with Customer-Managed Keys | S8 |
| S12-F1 | Updated Data Modelling Standards Endorsed and Ready for Use | S12 |

**Dependencies:** None – this is the foundation epic.
**Scope Items:** S2, S5, S8 (partial), S12 (partial)

---

## E2: Data Ingestion & Integration

**Outcome:** Data flows reliably from source systems into EDAP's Bronze layer – batch, CDC, streaming, and file-based – with sensitive data identified at the point of entry.

**Why this epic exists:** Data must land in the platform before it can be transformed or consumed. The ingestion framework determines whether onboarding new sources takes days (configuration) or weeks (custom code).

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S2-F3 | Ingestion Patterns Documented for Every Source Type | S2 |
| S13-F1 | Every Source System Has a Documented, Costed Connectivity Path | S13 |
| S13-F2 | New Source Entities Onboarded via Configuration, Not Code | S13 |
| S13-F3 | Database Changes Captured and Reflected in EDAP Within Minutes | S13 |
| S13-F4 | Real-Time Event Data Available as Queryable Tables | S13 |
| S13-F5 | PI and Sensitive Data Identified and Tagged Automatically During Ingestion | S13 |
| S13-F6 | Documents and Files Governed Alongside Structured Data | S13 |
| S13-F7 | All Architecture Patterns Proven End-to-End Before Use Case Delivery | S13 |
| S6-F1 | Source Data Lands and Is Preserved in Its Raw Form | S6 |
| S6-F2 | Real-Time Data Changes Captured and Available Within Minutes | S6 |

**Dependencies:** E1 (environments and catalogs must exist)
**Scope Items:** S2 (partial), S6 (partial), S13

---

## E3: Transformation & Data Products

**Outcome:** Clean, conformed, business-ready data products are available in the Gold layer for BI, analytics, and AI consumption – with dimensional models, semantic metrics, and contractual quality guarantees.

**Why this epic exists:** Raw data has no business value. This epic turns ingested data into trusted, consumable data products that business teams can use with confidence.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S2-F4 | Metadata-Driven Transformation Framework Designed | S2 |
| S6-F3 | Clean, Conformed Data Available for Analysis | S6 |
| S6-F4 | Business-Ready Data Products Available for Consumption | S6 |
| S6-F5 | Table Maintenance Runs Automatically Without Operator Intervention | S6 |
| S6-F8 | Data Quality Certified and Tracked Over Time | S6 |
| S6-F9 | External Systems Can Consume EDAP Data Securely | S6 |
| S12-F2 | Business-Ready Dimensional Models Available in the Gold Layer | S12 |
| S12-F3 | Domain Data Entities Documented at Conceptual and Logical Levels | S12 |
| S12-F4 | Physical Models Optimised for Lakehouse Query Patterns | S12 |
| S12-F5 | Data Products Defined with Contractual Quality Guarantees | S12 |
| S23.1-F1 | WC's Current Data Product Practices Assessed Against Industry Best Practice | S23.1 |
| S23.1-F2 | Target-State Data Product Lifecycle Defined and Endorsed | S23.1 |
| S23.1-F3 | Gaps Prioritised with a Clear Improvement Roadmap | S23.1 |

**Dependencies:** E1 (platform), E2 (data flowing in)
**Scope Items:** S2 (partial), S6 (partial), S12 (partial), S23.1 (partial)

---

## E4: Security, Governance & Compliance

**Outcome:** Data is classified, protected, and compliant automatically – access is tag-driven, audit trails are complete, and regulatory posture is demonstrable on demand.

**Why this epic exists:** Water Corporation operates under SOCI Act, PRIS Act, State Records Act, and Essential Eight. Governance cannot be an afterthought – it must be embedded in the platform from day one.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S2-F6 | Metadata Tagging Standards and Catalogue Integration Designed | S2 |
| S6-F7 | Sensitive Data Protected Automatically Based on Classification | S6 |
| S8-F1 | Domain Teams Independently Manage Their Own Data Access | S8 |
| S8-F2 | Sensitive Data Automatically Protected Based on Its Classification Tags | S8 |
| S8-F5 | Every Data Access Event Auditable and Searchable | S8 |
| S8-F6 | Compliance Posture Visible on Demand with Proactive Alerting | S8 |
| S15-F1 | Business Classifications from Alation Drive Access Control in Databricks | S15 |
| S15-F2 | Access Restrictions Enforce Automatically When Stewards Classify Data | S15 |
| S15-F3 | Tag Sync Verified End-to-End with Discrepancies Flagged | S15 |
| S16-F1 | Every Data Access and Change Auditable with Full Context | S16 |
| S16-F2 | Regulatory Compliance Demonstrable on Demand | S16 |
| S16-F3 | Fine-Grained Access Provisioned Automatically from Steward Classifications | S16 |
| S16-F4 | Governance Health Visible Through Real-Time Dashboards | S16 |
| S16-F5 | Stewardship Workflows Operational with Clear Accountability | S16 |
| S23.2-F1 | WC's Current AI Governance Maturity Assessed with Clear Gaps Identified | S23.2 |
| S23.2-F2 | AI Risk Classification Framework Ready for Use | S23.2 |

**Dependencies:** E1 (UC structure, identity), E2 (ingestion tagging)
**Scope Items:** S2 (partial), S6 (partial), S8 (partial), S15, S16, S23.2 (partial)

---

## E5: DataOps & Engineering Excellence

**Outcome:** Automated, governed, cost-efficient delivery pipeline – code promotes safely, tests run automatically, environments are realistic, and non-prod costs stay under control.

**Why this epic exists:** Without DataOps, the platform becomes a manual, error-prone operation. This epic ensures the team can deliver reliably and repeatedly at pace.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S7-F1 | Governed Code Promotion from Development to Production | S7 |
| S7-F2 | All Platform Assets Deployed as Code | S7 |
| S7-F3 | Every Pipeline Change Validated Automatically Before Deployment | S7 |
| S7-F4 | One-Click Promotion from Dev Through Test to Prod | S7 |
| S7-F5 | Pipeline Dependencies Orchestrated with Automatic Retry and Alerting | S7 |
| S7-F6 | Non-Prod Environments Reflect Production at 10% or Less Cost | S7 |
| S7-F7 | Consistent Naming and Tagging Across All Assets | S7 |
| S10-F1 | Dev and Test Environments Operational at ≤10% of Production Cost | S10 |
| S10-F2 | Realistic Test Data Available Without Exposing Production PI | S10 |
| S10-F3 | Pipeline Code Changes Automatically Tested Before Merge | S10 |
| S10-F4 | Pipeline Performance Baselined and Regressions Caught Early | S10 |
| S10-F5 | Business Users Can Validate Data Products Against Their Requirements | S10 |

**Dependencies:** E1 (environments, Git integration)
**Scope Items:** S7, S10

---

## E6: Discovery & Catalogue

**Outcome:** Every data asset in EDAP is discoverable in Alation with full metadata, lineage, and business context – and the catalogue stays current automatically.

**Why this epic exists:** Data that can't be found can't be used. The catalogue bridges the gap between the technical platform and business users who need to find, understand, and trust data.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S6-F6 | Every Data Asset Discoverable with Context and Lineage | S6 |
| S14-F1 | All EDAP Data Assets Discoverable in Alation with Full Metadata | S14 |
| S14-F2 | End-to-End Data Lineage Visible from Source Through to Gold Layer | S14 |
| S14-F3 | Metadata Stays Current Automatically Without Manual Refresh | S14 |
| S23.1-F4 | Data Quality Measurable Across Five Dimensions with Automated Monitoring | S23.1 |

**Dependencies:** E1 (UC structure), E3 (data products to discover)
**Scope Items:** S6 (partial), S14, S23.1 (partial)

---

## E7: Analytics & AI

**Outcome:** MLOps capabilities are operational – data scientists can experiment, train, deploy, and monitor models on EDAP, with the OMLIX algorithms running in production.

**Why this epic exists:** Advanced analytics is a core use case for EDAP. The platform must support the full ML lifecycle, not just data engineering.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S18-F1 | Data Scientists Track Experiments and Compare Model Runs | S18 |
| S18-F2 | Features Registered, Versioned, and Reusable Across Models | S18 |
| S18-F3 | Model Training Reproducible and Automated | S18 |
| S18-F4 | Models Served as Real-Time Endpoints with Traffic Management | S18 |
| S18-F5 | Model Performance Monitored with Drift Alerts | S18 |
| S18-F6 | Three OMLIX Algorithms Operational in the Customer Domain | S18 |

**Dependencies:** E1 (platform), E2 (data flowing), E3 (Gold-layer data products), E5 (CI/CD for ML)
**Scope Items:** S18

---

## E8: Migration Pathways

**Outcome:** Every legacy platform (Glue, ADF, SAP DS, SageMaker) has a documented, costed, proven migration path to EDAP – and business use case data is migrated and serving.

**Why this epic exists:** Water Corporation has existing workloads that need a path forward. Without clear migration guidance, legacy platforms persist indefinitely alongside EDAP.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S22-F1 | Clear, Costed Migration Path Documented for Every Legacy Platform | S22 |
| S22-F2 | AWS Glue Workloads Have a Proven Migration Path to Databricks | S22 |
| S22-F3 | ADF Pipelines Have a Proven Migration Path to Databricks | S22 |
| S22-F4 | SAP Data Services Workflows Have a Proven Migration Path | S22 |
| S22-F5 | SageMaker Models Have a Proven Migration Path to Mosaic AI | S22 |
| S22-F6 | Business Use Case Data Migrated, Validated, and Serving from EDAP | S22 |

**Dependencies:** E1 (platform), E2 (ingestion framework), E3 (transformation framework)
**Scope Items:** S22

---

## E9: Capability Uplift & Governance Advisory

**Outcome:** WC team is self-sufficient on EDAP – trained, certified, embedded in delivery, with reusable artefacts and endorsed governance frameworks for data products and AI.

**Why this epic exists:** The SI engagement is temporary. If WC can't operate, extend, and govern EDAP independently, the investment is at risk the day the SI walks out.

| Feature ID | Feature Name | Scope Item |
|---|---|---|
| S19-F1 | Role-Based Learning Pathways Mapped for Every WC Team Member | S19 |
| S19-F2 | WC Engineers Certified on Databricks Platform Fundamentals | S19 |
| S19-F3 | WC Team Proficient in EDAP-Specific Patterns and Practices | S19 |
| S19-F4 | WC Engineers Building Capability Through Hands-On Delivery | S19 |
| S19-F5 | Reusable Templates and Runbooks Available for Ongoing Operations | S19 |
| S19-F6 | WC DataOps Team Embedded in the Code Review Process | S19 |
| S20-F1 | Delivery Teams Operating in SAFe Cadence with Clear Ceremonies | S20 |
| S20-F2 | Stakeholders Informed of Progress, Risks, and Decisions at Every Level | S20 |
| S20-F3 | Risks and Issues Tracked, Escalated, and Resolved Transparently | S20 |
| S20-F4 | Production Components Transitioned to BAU Support Seamlessly | S20 |
| S23.2-F3 | Prioritised Improvement Roadmap Endorsed by Stakeholders | S23.2 |
| S23.2-F4 | AI Model Lifecycle Governed from Development Through Retirement | S23.2 |
| S23.2-F5 | AI Agent Actions Bounded, Auditable, and Human-Supervised | S23.2 |

**Dependencies:** E1 (platform operational for training), runs in parallel with all other epics
**Scope Items:** S19, S20, S23.2 (partial)

---

## Dependency Map

```
E1: Platform Foundation
 ├── E2: Data Ingestion & Integration
 │    └── E3: Transformation & Data Products
 │         ├── E6: Discovery & Catalogue
 │         ├── E7: Analytics & AI
 │         └── E8: Migration Pathways
 ├── E4: Security, Governance & Compliance
 └── E5: DataOps & Engineering Excellence

E9: Capability Uplift (runs in parallel with all epics)
```

---

## Scope Item → Epic Cross-Reference

| Scope Item | Primary Epic(s) | Features |
|---|---|---|
| S2 | E1, E2, E3, E4 | 6 |
| S5 | E1 | 7 |
| S6 | E2, E3, E4, E6 | 9 |
| S7 | E5 | 7 |
| S8 | E1, E4 | 6 |
| S10 | E5 | 5 |
| S12 | E1, E3 | 5 |
| S13 | E2 | 7 |
| S14 | E6 | 3 |
| S15 | E4 | 3 |
| S16 | E4 | 5 |
| S18 | E7 | 6 |
| S19 | E9 | 6 |
| S20 | E9 | 4 |
| S22 | E8 | 6 |
| S23.1 | E3, E6 | 4 |
| S23.2 | E4, E9 | 5 |

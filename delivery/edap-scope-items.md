# EDAP Scope of Work – Scope Items Register

---

## S5 – EDP Configuration

| Field | Detail |
|---|---|
| **Scope Ref.** | S5 |
| **Scope Heading** | EDP Configuration |
| **Scope Area** | EDP Implementation |
| **Documented Deliverable** | Data Platform Design - Updated |

### Scope Summary

Utilising the Data Platform Design, configure the EDP Development, Test and Production environments. It includes but is not limited to:

**Workspace Deployment**
- Create Databricks Workspaces for Dev, Test, Prod (account-level setup)

**Databricks Configuration**
- Cluster Policies & Pools (define autoscaling rules, instance types, spark configs)
- Enable Unity Catalog (metastore, schema registries, lineage)
- Set Up Secrets Scopes (link to AWS Secrets Manager or Vault)
- Configure Notebooks & Repos (Git integration, branch strategy)
- Set up Workspace Security (SCIM provisioning, role mapping, MFA enforcement)

**Integrate with Third-party and Partner Tools**
- Version Control (GitHub)
- CI/CD Tooling (GitActions, GitOps model, Databricks CLI/REST API)
- Monitoring & Alerting (CloudWatch, Databricks native dashboards)

### Deliverable

S5 – EDP Configuration: Iterative platform configuration (workspaces; compute/security/catalog/deployment assets; tools; monitoring/alerting; testing/validation).

### Criteria

Dev/Test/Prod workspaces configured; Unity Catalog enabled; cluster policies, secrets, CI/CD tooling; monitoring hooks; configuration verified in Acceptance Environment; UAT passed;

---

## S6 – EDP Implementation

| Field | Detail |
|---|---|
| **Scope Ref.** | S6 |
| **Scope Heading** | EDP Implementation |
| **Scope Area** | EDP Implementation |
| **Documented Deliverable** | Data Platform Design - Updated |

### Scope Summary

a) Utilising the Data Platform Design, implement and operationalise the EDP Development, Test and Production environments. It includes but is not fully inclusive of:

**Implement Ingestion**
- Develop Batch Ingestion Jobs to ingest into Bronze
- Develop Streaming Ingestion Jobs
- Create Landing Zone in Bronze (Delta Lake table definitions, initial partitioning)
- Implement Data Quality framework (to manage bad records, schema drift, etc) or standard patterns for efficient utilisation for use cases requirements.

**Develop Transformation Pipelines**
- Build Bronze Layer Transformations (native format with minimal transformation, preserving its original fidelity and enabling full traceability)
- Build Silver Layer Transformations (cleaning, deduplication, conformance rules)
- Build Gold Layer Transformations (aggregations, business logic, dimensional hub loading)
- Optimize Delta Lake Tables (Z-Ordering, OPTIMIZE, VACUUM)
- Document and Register Tables in Unity Catalog (create table comments, tags, column-level descriptions)

**Data Catalogue & Governance**
- Register Tables, Views, and Schemas with Unity Catalog (assign owners, update descriptions)
- Define Data Access Policies (grant SELECT, MANAGE GRANTS, data masking if needed)
- Implement Data Lineage Tracking (automatically capture lineage via Delta, ETL code annotations)
- Establish Data Stewardship Processes (periodic metadata reviews, data certification)
- Consolidate business metrics in Unity Catalogue Metrics

b) Implement robust security and governance measures, including role-based and attribute-based access controls, encryption, data masking, auditing, and single sign-on with Microsoft EntraID

c) Integrate platform components with external systems in accordance with approved solution architecture.

### Deliverable

*Assumed to run across phase 2 and phase 3*

S6 – EDP Implementation: Utilising the Data Platform Design, implement and operationalise the EDP solution.

### Criteria

MVP of EDP solution deployed in test environment with initial release of features to satisfy all scope items and which will be fully elaborated in Phase 3 use case implementation.
MVP of all process, standards, documentation related to the scope of work reviewed and accepted by WC as "initial draft".

---

## S7 – DataOps Enablement

| Field | Detail |
|---|---|
| **Scope Ref.** | S7 |
| **Scope Heading** | DataOps Enablement |
| **Scope Area** | EDP Detailed Design |
| **Documented Deliverable** | Engineering Frameworks, Playbooks Processes & Practices |

### Scope Summary

Establish a robust DataOps foundation to support the design, development, deployment, and operationalisation of data products across environments. This includes embedding engineering standards, automation pipelines, and operational controls that ensure reliability, traceability, and agility.

**Source Control & Versioning**
- Structure Git repositories to separate notebooks, job configurations and deployment assets.
- Define and enforce branching strategy (main, dev, feature branches, release tags) aligned with release cycles
- Implement pull request workflows with mandatory code reviews, linting, and test coverage checks

**Automated Testing and Validation**
- Develop Unit Tests for all code including Notebooks using pytest and dbx, with Spark session mocking where needed
- Implement Data Validation Tests (assert row counts, schema checks, anomaly detection)
- Build Integration Tests to validate pipeline logic across datasets
- Security & Compliance Scans (scan IaC templates for open permissions, secrets in code)

**Deployment Automation**
- Use Infrastructure as Code (CloudFormation) to provision AWS resources
- Automate deployment of Databricks assets via databricks-cli/dbx and REST APIs
- Promotion Strategy (Dev → Test → Prod environments, manual approvals)

**Scheduling and Orchestration**
- Configure job orchestration using Lakeflow Jobs with support for dependencies and retries
- Enable event-driven triggers (eg S3 event notifications)
- Set up Job Monitoring & Alerts (configure email, …)

**Environment Management and Parameterisation**
- Implement automated data refresh processes for Development and Test environments using masked or sampled datasets from Production, ensuring realistic testing while maintaining compliance with data privacy standards
- Define tagging and naming conventions to support environment isolation and auditability

### Deliverable

S7 – DataOps Enablement: Source control/versioning approach; branching; repo structure; PR workflows; automated testing/validation; deployment/scheduling/orchestration automation.

### Criteria

Repos structured; CI/CD pipelines operational; automated tests (unit/integration/DQ); parameterised promotion; orchestrations (Lakeflow) with notifications; representative pipeline UAT run passed; test data management in non-prod environments.

---

## S8 – Security & Compliance

| Field | Detail |
|---|---|
| **Scope Ref.** | S8 |
| **Scope Heading** | Security & Compliance |
| **Scope Area** | EDP Implementation |
| **Documented Deliverable** | Data Platform Design - Updated |

### Scope Summary

a) Implement Role-based Access Control (RBAC) and Attribute-based access control (ABAC):
   - Define Databricks Workspace roles (Admin, User, Data Engineer, Data Analyst)
   - Map IAM Roles to Databricks Service Principals or SCIM Groups

b) Configure SCIM Provisioning with Identity Provider
   - Automate user and group sync (Entra ID)
   - Enforce least-privilege separate roles for ETL jobs vs. analysts

c) Data Encryption & Key Management
   - Enable S3 Server-Side Encryption (SSE-KMS) with Customer-Managed Keys
   - Configure Cluster-Level Encryption & Network Encryption (TLS)

d) Logging & Auditing
   - Enable Databricks Audit Logs (Splunk Destination)
   - Integration with Splunk
   - Periodic Compliance Reporting
   - Security Alerts (unauthorized access attempts, privilege escalations)

### Deliverable

S8 – Security & Compliance: UC structure; lineage; encryption; secrets; audit logging; access policies (RBAC/ABAC); role hierarchy; Entra ID integration (SCIM/SSO).

### Criteria

RBAC/ABAC implemented; SCIM/SSO; encryption (CMK, TLS); Audit Logs integrated; sample compliance report; access control UAT passed;

---

## S10 – Testing and Non-Prod Environments

| Field | Detail |
|---|---|
| **Scope Ref.** | S10 |
| **Scope Heading** | Testing and Non-Prod Environments |
| **Scope Area** | EDP Detailed Design |
| **Documented Deliverable** | Testing Scripts, Testing Plans, Daily Defect Reports |

### Scope Summary

SI is responsible for establishing an architecture and ongoing approach for testing, managing test data and the use of non-production environments (frameworks, practices, etc) that ensures **the ongoing technology cost of the non-prod environments does not exceed 10% of production environment costs** during and beyond the project.

- SI is responsible for managing and undertaking all testing of scope services using above mentioned approach including the implemented EDP and business use cases and evidenced through-out incremental delivery. System and System Integration Testing with all integration points external to the EDP system boundary and executed test scripts
- Performance Test Plan and executed test scripts
- Creation of the use case testing scripts in collaboration with the Water Corporation staff in alignment with business requirements
- Facilitate User Acceptance Testing in collaboration with business unit users ensuring system and business process alignment.

**The Corporation will conduct User Acceptance Testing.**

### Deliverable

S10 – Testing & Non-Prod Environments: Testing strategy/architecture; test data management; cost control; automation frameworks; system/integration test scenarios; performance testing plan and pipeline tuning.

### Criteria

Test environment architecture including testing strategy that achieves non-prod platform costs ≤10% of prod (to be validated with Use Cases); UAT enablement complete;

---

## S12 – Data Modelling

| Field | Detail |
|---|---|
| **Scope Ref.** | S12 |
| **Scope Heading** | Data Modelling |
| **Scope Area** | EDP Implementation |
| **Documented Deliverable** | Dimensional Data Model |

### Scope Summary

- Review Water Corporation's Data Modelling Standards and Guidelines document (see **Appendix 7**), make improvement recommendations based on best practice and update the document subject to Water Corporation's endorsement.
- Design and implement data modelling (dimensional or other as appropriate), including frameworks and methodology aligning with the updated Data Modelling Standards and Guidelines document mentioned above and water utility industry standard common data models, standards and best practices.
- Data modelling for all business use cases should be done using tools that don't require further license procurement by Water Corporation to maintain the models post-project (i.e. MS Visio, MS Word or alternate subject to endorsement).

### Deliverable

S12 – Data Modelling: Review standards/processes/guidelines; identify gaps and improvements.

### Criteria

Updated standards/guidelines; conformed conceptual/logical/physical models; lineage/glossary alignment; data modelling framework implemented in Databricks to support standardisation and code re-use; artefacts accepted

---

## S13 – Data Integration

| Field | Detail |
|---|---|
| **Scope Ref.** | S13 |
| **Scope Heading** | Data Integration |
| **Scope Area** | EDP Detailed Design |
| **Documented Deliverable** | Data Platform Design – Updated |

### Scope Summary

- Ensure seamless and reliable ingestion of structured and unstructured data from identified sources including databases, files, APIs, and streaming data.
- Implement mechanisms to handle Change Data Capture and incremental updates to keep the platform up to date.
- Enable connectivity to initial data sources for platform validation and testing.
- Identification of sensitive data during ingestion, tagging and classifying within the data platform and integrating into access control mechanisms.
- Implement frameworks or libraries that eliminates the need for writing code for each table for bulk ingestion from source systems.
- Specify the available connection methods for data sources to support incremental ingestion. Include details on external connectors, tools, relevant documentation, any known limitations, licensing requirements and associated cost (if applicable).
- Implement technical use cases to develop, implement and prove the architecture pattern for those not covered by the Business Use Cases. **Refer to Appendix 6 for the full list of patterns and guidance in this regard.**

### Deliverable

S13 – Data Integration: Review connected architecture; define integration scope/lineage; incremental ingestion; identify sensitive/regulated data; framework for security/management; integration design docs mapping sources/tables/pipelines/lineage.

### Criteria

**Integration Design Completed**
Documented integration scope, lineage requirements, and ingestion and RETL patterns approved.

**Security Framework Delivered**
Framework implemented that enables identification and handling of sensitive or regulated data, with updated integration design that maps sources, tables, pipelines, and lineage.

---

## S14 – Integration from EDP to Data Catalogue

| Field | Detail |
|---|---|
| **Scope Ref.** | S14 |
| **Scope Heading** | Integration from EDP to Data Catalogue |
| **Scope Area** | EDP Implementation |
| **Documented Deliverable** | Data Platform Design - Updated |

### Scope Summary

- Integrate Databricks and other components of the overall EDP solution (where applicable) with Alation, the corporate data catalogue, using out of the box connectivity (Alation / Databricks OCF connector). It is key that Alation maintain a complete end-to-end lineage view of the data in the overall EDP solution spanning both Databricks and non-Databricks components (e.g. native AWS analytics services).

### Deliverable

S14 – Integration from EDP to Data Catalogue: Configure Databricks connector for harvesting metadata/pipelines; configure Alation connectors for non-Databricks components.

### Criteria

S15 – Integrate from Data Catalogue to EDP: Metadata sync Alation → Unity Catalog; ABAC policy implementation; validate access restrictions/privileges

---

## S15 – Integration from Data Catalogue to Databricks

| Field | Detail |
|---|---|
| **Scope Ref.** | S15 |
| **Scope Heading** | Integrate from Data Catalogue to EDP |
| **Scope Area** | EDP Implementation |
| **Documented Deliverable** | Data Platform Design – Updated |

### Scope Summary

- Integrate metadata applied to data assets in the Alation data catalogue back into the Databricks Unit Catalogue to facilitate attribute-based access control in Databricks (ABAC).

### Deliverable

S15 – Integrate from Data Catalogue to EDP: Metadata sync Alation → Unity Catalog; ABAC policy implementation; validate access restrictions/privileges

### Criteria

Unity Catalog harvesting Alation tags to enable ABAC in Databricks; end-to-end UAT scenario passed;

---

## S16 – Data Governance

| Field | Detail |
|---|---|
| **Scope Ref.** | S16 |
| **Scope Heading** | Data Governance |
| **Scope Area** | EDP Implementation |
| **Documented Deliverable** | Data Platform Design - Updated |

### Scope Summary

- Set up auditing and monitoring mechanisms to track data access, changes, and anomalies for security and compliance purposes.
- Establish governance mechanisms to ensure adherence to data privacy and regulatory requirements.
- Enable a highly automated solution for provisioning fine-grained access controls at the table, row, and column levels, leveraging data stewardship classification / tagging of datasets in Alation and propagated to Databricks to protect sensitive data and minimising the need for manual intervention.

### Deliverable

S16 – Data Governance: Define governance/auditing/compliance objectives; review regulatory requirements; frameworks; auditing; alerting/monitoring; compliance reporting.

### Criteria

Audit and monitoring of data access and changes, security, privacy and compliance anomalies.
Ability to provision table/row/column access controls using ABAC and tags from Alation.
Artefacts accepted;

---

## S18 – Advanced Analytics Model Management

| Field | Detail |
|---|---|
| **Scope Ref.** | S18 |
| **Scope Heading** | Advanced Analytics Model Management |
| **Scope Area** | Use Case Delivery |
| **Documented Deliverable** | Data Product Solutions and As-Built Specifications – Updated |

### Scope Summary

- Implement minimum viable MLOps solution to enable development, model and feature registration, model training, deployment, inference execution and monitoring of three advanced analytics algorithms used in the OMLIX data product of the Customer Domain Data Mart use case as specified in Business Use Case 1 in **Appendix 4**.

### Deliverable

S19 – Advanced analytics Model Management: Implement minimum viable MLOps solution to enable development, model and feature registration, model training, deployment, inference execution and monitoring of three advanced analytics algorithms used in the…

### Criteria

MLOps process and design documented and accepted.

---

## S19 – Training and Knowledge Transfer

| Field | Detail |
|---|---|
| **Scope Ref.** | S19 |
| **Scope Heading** | Training and Knowledge Transfer |
| **Scope Area** | Knowledge Transfer |
| **Documented Deliverable** | Blended learning approach including: Self-paced vendor provided role-based e-learning; Instructor led training (likely TTT delivery); On the job coaching (SI led upskilling); Vendor/product certifications; Quick Reference Sheets, FAQs & demonstrations |

### Scope Summary

- It is expected that all Databricks product training will be done using Databricks provided material, trainers and online platforms. The SI is to develop and provide training in relation to all of the scope items developed by the SI
- Allow Corporation personnel to shadow, observe and sit with SI personnel as work is being conducted
- Conduct Workshops for Data Engineers & Analysts (Databricks best practices, Delta Lake patterns)
- Publish Coding Standards & Notebook Templates (naming conventions, I/O patterns)
- Conduct training and knowledge transfer sessions for the Corporation's IT and data teams, aligning to the Supplier's learning and certification programs.
- Water Corporation DataOps team to be part of the approval for the Code Pipeline Pull Review process for code review, before deployment of code to higher environments. This is part of the knowledge transfer process throughout the project.

### Deliverable

S19 – Training & Knowledge Transfer: Identify skills/knowledge needs; develop KT plan/approaches/targets; create templates/examples; implement & track progress with training materials/sessions.

### Criteria

Foundation: Training plan/curriculum; role-based pathways; templates/FAQs; schedule baselined;
Must cover all designs, solutions, processes, practices provided by Supplier to enable ongoing development, maintenance and support.
Training plan must include "experiential" learning in which WC analysts/engineers work alongside Supplier for hands-on learning and development.

---

## S20 – Project Management

| Field | Detail |
|---|---|
| **Scope Ref.** | S20 |
| **Scope Heading** | Project Management |
| **Scope Area** | PMO & Support |
| **Documented Deliverable** | Monthly **and Sprint-based** Project Status Report (including progress against budget, % complete, key activities completed and planned, and risk and issues); Input to relevant sections of project documentation; Input to MS Project **and JIRA** key task and milestone reporting |

### Scope Summary

The Corporation requires a flexible, adaptive and Agile Methodology to delivering the services and has a strong preference for Scaled Agile Framework (SAFe) using Scrum at the delivery team level. We are expecting an iterative and incremental approach so that work is delivered in smaller pieces, tested and deployed regularly into production so that learning and improvement can be continuously built into the delivered solution.

It is Water Corporation's ambition that the projects' Agile delivery structure reflects the ongoing target state structure of our operational data teams to facilitate a seamless and regular transition-to-support of production deployed components and features throughout the delivery phase of the project. To this end, please review Appendix 5 for an overview of our current data analytics BAU organisational structure and operating model. We anticipate some improvements to this operating model as a result of the project and are therefore open to improvement suggestions in the interest of a contemporary Agile DataOps and Business Intelligence delivery capability.

- SI is responsible for managing the Scope of Services outlined with this Scope of Work.
- Manage the scope of services and resources from their organisation, drive and monitor progress to time, scope and cost measures and provide status reporting to the Corporations Project Manager and Program/Project Steerco using Water Corporation templates.
- Attend regular Program/Project Steerco meetings, project status meetings, progress reporting, and issue tracking mechanisms to ensure transparency and timely resolution of project-related matters.
- Foster effective collaboration between the SI's team and the Water Corporation's stakeholders in alignment with the Corporation's Platform Manager.
- Work in alignment with the Corporations project delivery frameworks (waterfall and agile)
- Support in providing MS Project and JIRA mandatory milestone reporting
- Facilitate SI input to key project documentation (i.e. Transition to Support and As-Built)

### Deliverable

S20 – Project Management: Progress reporting; RAID tracking; governance; stakeholder engagement/alignment.

### Criteria

Foundation: Reporting cadence operating; sprint/PI governance in place; artefacts accepted;

---

## S22 – Data and Code Migration

| Field | Detail |
|---|---|
| **Scope Ref.** | S22 |
| **Scope Heading** | Data & Code Migration |
| **Scope Area** | PMO & Support |
| **Documented Deliverable** | Data Migration Approach |

### Scope Summary

**Note:** While actual data and code migration/reimplementation for the engagement is limited to the business use cases of S17, in this scope item the Corporation is seeking recommendations relating to large scale migration from our legacy technologies to the new EDP for future implementation.

- Explain your data migration strategy and implementation approach, including any external tools required, if any.
- Provide any licensing estimate if relevant and the licensing model (perpetual, subscription-based etc.)
- Provide details on how existing data pipelines built on the following platforms can be migrated to the target solution, specifically:
  1. AWS Glue – What tools or processes are available to migrate Glue-based ETL jobs, transformations, workflows and Pyspark statistical algorithms
  2. Azure Data Factory (ADF) – How can ADF pipelines, linked services, and dataflows be transitioned seamlessly.
  3. SAP Data Services – Recommended approach for migrating SAP DS workflows, transformations, and job scheduling.
  4. AWS Sagemaker – How Machine Learning models (e.g. the Hardship Propensity XGBoost model) can be migrated to DataBricks.

### Deliverable

S22 – Data & Code Migration: Identify migration requirements; **develop** future migration strategy/**approach**.

### Criteria

Migration strategy documented (Glue, ADF, SAP DS, SageMaker → Databricks); licensing estimates (if any); artefacts accepted;

---

## S23.1 – Data and Analytics Governance Advice: Data Product SDLC and Quality Assurance

| Field | Detail |
|---|---|
| **Scope Ref.** | S23 |
| **Scope Heading** | Data and Analytics Governance Advice |
| **Scope Area** | PMO & Support |
| **Documented Deliverable** | Advice, Reference Materials, Examples |

### Scope Summary

Provide advisory support to enhance the following the Corporation governance frameworks and process definitions by sharing established knowledge libraries, insights from project experiences and industry best practices:

- Data product software development lifecycle and related quality assurance governance practices
- AI Governance Framework for the responsible use of AI solutions and services procured or built in-house.

### Deliverable (S23.1)

S23 – Data & Analytics Governance Advice: Data product software development lifecycle and related quality assurance governance practices

### Criteria

WC existing framework reviewed.
Best practices identified and referenced.
Gap analysis documented.

---

## S23.2 – Data and Analytics Governance Advice: AI Governance

| Field | Detail |
|---|---|
| **Scope Ref.** | S23 |
| **Scope Heading** | Data and Analytics Governance Advice |
| **Scope Area** | PMO & Support |
| **Documented Deliverable** | Advice, Reference Materials, Examples |

### Scope Summary

Provide advisory support to enhance the following the Corporation governance frameworks and process definitions by sharing established knowledge libraries, insights from project experiences and industry best practices:

- Data product software development lifecycle and related quality assurance governance practices
- AI Governance Framework for the responsible use of AI solutions and services procured or built in-house.

### Deliverable (S23.2)

S23 – Data & Analytics Governance Advice: AI Governance Framework for the responsible use of AI solutions and services procured or built in-house.

### Criteria

WC existing framework reviewed.
Best practices identified and referenced.
Gap analysis documented.
Recommended improvements documented.
Artefact accepted by WC.

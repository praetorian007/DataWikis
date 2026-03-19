# S23.2 – Data & Analytics Governance Advice: AI Governance: Feature Breakdown

**Scope Area:** PMO & Support
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `lifecycles/data-governance-lifecycle.md` (Stage 9 – AI and Model Governance)
- `lifecycles/data-science-lifecycle.md` (Deploy stage; MLOps discipline; Mosaic AI Agent Framework)
- `governance/data-governance-roles.md` (AI/ML Governance Lead role)

---

## Feature S23.2-F1: WC's Current AI Governance Maturity Assessed with Clear Gaps Identified

**Description:** Leadership and governance stakeholders can see exactly where Water Corporation stands on AI governance today — every known AI/ML model inventoried, existing policies rated for currency, and organisational readiness scored across four dimensions — so that the scale of work required is visible, evidence-based, and communicable to executives before investment decisions are made.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.2-F1-US01 | AI/ML Governance Lead | have a documented assessment of our current AI governance maturity scored on a defined scale | I can understand the baseline and communicate the scale of work required to leadership |
| S23.2-F1-US02 | Data Governance Manager | understand which AI/ML models are currently in development or production, with their governance status | I can assess the urgency and scope of governance controls needed |
| S23.2-F1-US03 | Chief Data Officer | have visibility of our AI governance posture relative to regulatory expectations (including the Australian Voluntary AI Safety Standard) | I can assess organisational risk and prioritise investment |
| S23.2-F1-US04 | Data Scientist | understand what governance requirements currently apply to my models and where gaps exist | I can identify where I am already compliant and where I need to act |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.2-F1-AC01 | Stakeholder interviews are planned | the assessment commences | interviews are conducted with at least one representative from each of: data science, data engineering, data governance, legal/compliance, and business domain ownership |
| S23.2-F1-AC02 | Existing AI/ML models are inventoried | the assessment is completed | every known AI/ML model (in development, staging, or production) is catalogued with: model name, owner, framework, purpose, data sources, deployment status, and current governance documentation |
| S23.2-F1-AC03 | Existing policies are reviewed | findings are documented | all existing policies, standards, and guidelines relating to AI/ML usage are catalogued with a currency assessment (current, outdated, or absent) |
| S23.2-F1-AC04 | Organisational readiness is assessed | the report is produced | the assessment evaluates AI governance readiness across four dimensions: policy and standards, roles and accountability, tooling and automation, and culture and awareness, using a defined maturity scale |
| S23.2-F1-AC05 | The assessment is finalised | it is presented to Water Corporation | the report includes an executive summary of no more than two pages, a model inventory register, and a maturity scorecard |

### Technical Notes
- Use the AI and Model Governance stage (Stage 9) of the Data Governance Lifecycle as the target-state reference for assessing current maturity.
- Assess whether an AI/ML Governance Lead role (or equivalent) currently exists per the role definition in `governance/data-governance-roles.md`.
- Include any AI/ML models deployed in existing platforms (SageMaker, Glue-based algorithms) that will migrate to Databricks under S22.
- Assess familiarity with the Australian Voluntary AI Safety Standard and the Australian AI Ethics Principles across stakeholder groups.

---

## Feature S23.2-F2: AI Risk Classification Framework Ready for Use

**Description:** Any stakeholder proposing or reviewing an AI system can classify it against a defined risk scheme (minimal, limited, high, unacceptable), understand exactly what governance controls apply at that tier, and confirm alignment with the Australian Voluntary AI Safety Standard — so that proportionate oversight is applied from the outset rather than retrofitted after deployment.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.2-F2-US01 | AI/ML Governance Lead | have a risk classification scheme for AI systems with clear criteria per tier | I can apply proportionate governance controls based on the risk level of each AI system |
| S23.2-F2-US02 | Chief Data Officer | have a framework aligned to Australian regulatory expectations that I can present to the board | I can demonstrate responsible AI practices to regulators and the board |
| S23.2-F2-US03 | Data Scientist | understand the transparency and documentation requirements for my model's risk tier | I can build governance compliance into my workflow from the outset rather than retrofitting |
| S23.2-F2-US04 | Data Governance Manager | have defined human oversight requirements by risk tier | I can ensure appropriate safeguards are in place before AI systems are deployed to production |
| S23.2-F2-US05 | Legal/Compliance Officer | have responsible AI principles codified with measurable indicators | I can assess AI initiatives against clear, endorsed principles |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.2-F2-AC01 | The risk classification scheme is drafted | it is reviewed by stakeholders | the scheme defines at least four risk tiers (minimal, limited, high, unacceptable) with clear criteria for classification based on impact on individuals, operational safety, and regulatory exposure, aligned to the Data Governance Lifecycle Stage 9 risk tiers |
| S23.2-F2-AC02 | Model inventory requirements are defined | the framework is reviewed | the model inventory specification includes mandatory fields: model name, owner, risk tier, purpose, training data sources, deployment status, last review date, and responsible domain owner, consistent with Unity Catalog Model Registry capabilities |
| S23.2-F2-AC03 | Transparency requirements are defined | the framework is accepted | transparency obligations are specified per risk tier: minimal risk requires basic catalogue registration; limited risk requires user-facing disclosure; high risk requires model cards, datasheets, and explainability provisions |
| S23.2-F2-AC04 | Human oversight mechanisms are defined | the framework is reviewed | three oversight models are defined (human-in-the-loop, human-on-the-loop, human-in-command) with mapping to risk tiers, consistent with the Data Governance Lifecycle Stage 9 oversight requirements |
| S23.2-F2-AC05 | Responsible AI principles are articulated | the framework is accepted | at least six principles are defined (e.g., fairness, transparency, accountability, privacy, safety, human oversight) with each principle having measurable indicators and assessment criteria |
| S23.2-F2-AC06 | Australian regulatory alignment is validated | the framework is finalised | the framework explicitly maps its controls to the ten guardrails of the Australian Voluntary AI Safety Standard, with compliance status and gaps identified for each |

### Technical Notes
- Risk classification should align precisely with the four-tier model in the Data Governance Lifecycle Stage 9: minimal, limited, high, and unacceptable risk.
- The model inventory should be implementable within Unity Catalog Model Registry, using tags and metadata fields for governance attributes.
- Transparency requirements should reference model cards (per Mitchell et al., 2019) and datasheets for datasets (per Gebru et al., 2021) as the documentation standard.
- The Australian Voluntary AI Safety Standard (January 2025) provides ten voluntary guardrails; the framework should map to all ten even where Water Corporation's current AI usage may not trigger all guardrails.
- Reference the AI/ML Governance Lead role responsibilities from `governance/data-governance-roles.md` for accountability mapping.
- Consider PRIS Act 2024 obligations for AI systems that process personal information.

---

## Feature S23.2-F3: Prioritised Improvement Roadmap Endorsed by Stakeholders

**Description:** Decision-makers can see every gap between current AI governance practice and the target framework, rated by severity, with recommendations sequenced into planning horizons and mapped to the AI/ML Governance Lead role — so that improvement investment is directed where it matters most, accountability is clear, and quick wins build governance momentum from the first PI.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.2-F3-US01 | AI/ML Governance Lead | have a gap analysis identifying where our current practices fall short of the target framework, rated by severity | I can prioritise governance improvement initiatives |
| S23.2-F3-US02 | Chief Data Officer | have a prioritised improvement roadmap with effort estimates I can use to secure funding | I can secure funding and executive support for the highest-impact improvements |
| S23.2-F3-US03 | Data Governance Manager | have governance responsibilities mapped to the AI/ML Governance Lead role with fulfilment status | I can establish clear accountability and recruit or assign the role appropriately |
| S23.2-F3-US04 | Project Manager | have effort estimates and dependencies for each recommendation | I can integrate AI governance improvements into PI planning |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.2-F3-AC01 | The current state assessment (F1) and target framework (F2) are complete | the gap analysis is performed | every framework element (risk classification, model inventory, transparency, human oversight, responsible AI principles) has gaps documented with a severity rating (critical, high, medium, low) |
| S23.2-F3-AC02 | Gaps are identified | recommendations are drafted | each gap has at least one actionable recommendation with estimated effort (T-shirt size: S, M, L, XL), responsible role, and prerequisites |
| S23.2-F3-AC03 | The AI/ML Governance Lead role is mapped | the analysis is reviewed | every responsibility listed in the AI/ML Governance Lead role definition (from `governance/data-governance-roles.md`) is mapped to a current state: fulfilled, partially fulfilled, or unfulfilled, with recommendations for each unfulfilled item |
| S23.2-F3-AC04 | Recommendations are prioritised | the roadmap is produced | recommendations are sequenced into three horizons: quick wins (current PI), medium-term (1-2 PIs), and strategic (3+ PIs), with dependencies and rationale documented |
| S23.2-F3-AC05 | The gap analysis is accepted | the roadmap is endorsed by Water Corporation | the roadmap identifies at least three quick wins implementable within the current PI to establish governance momentum |

### Technical Notes
- The AI/ML Governance Lead role mapping should cover all eight key responsibilities defined in `governance/data-governance-roles.md`: AI governance framework, model inventory, risk classification, regulatory compliance, domain owner advisory, third-party AI review, DPO coordination, and governance reporting.
- Quick wins may include: establishing the model inventory in Unity Catalog, defining the risk classification scheme, and drafting model card templates.
- Medium-term items likely include: implementing automated governance checks in ML pipelines, establishing review workflows, and training data scientists on governance requirements.
- Strategic items may include: full integration with Lakehouse Monitoring for model drift detection, automated compliance reporting, and maturation of responsible AI assessment processes.
- Prioritisation should consider dependencies on S22 (SageMaker migration) and S18 (Advanced Analytics Model Management).

---

## Feature S23.2-F4: AI Model Lifecycle Governed from Development Through Retirement

**Description:** Data scientists, ML engineers, and governance leads can govern every AI model from initial development through production monitoring to retirement — using standardised model cards, datasheet templates, defined approval workflows, monitoring thresholds, and retirement criteria — so that no model reaches production without appropriate review and no production model operates without ongoing oversight.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.2-F4-US01 | Data Scientist | have a model card template and clear guidance on what to document at each lifecycle stage | I can produce governance documentation efficiently as part of my normal workflow |
| S23.2-F4-US02 | AI/ML Governance Lead | have a defined approval workflow for promoting models to production with named reviewers and turnaround SLAs | I can ensure appropriate governance review occurs before models serve production workloads |
| S23.2-F4-US03 | ML Engineer | have defined monitoring requirements for production models covering drift, performance, and data quality | I can implement drift detection, performance tracking, and alerting as standard practice |
| S23.2-F4-US04 | Data Product Owner | have clear retirement criteria so I know when a model should be decommissioned | I can manage the end-of-life process for models that are no longer fit for purpose |
| S23.2-F4-US05 | Data Domain Steward | have datasheet standards for training datasets documenting provenance and quality | I can ensure training data provenance, quality, and representativeness are documented and governed |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.2-F4-AC01 | The model card template is drafted | it is reviewed by data scientists and governance stakeholders | the template includes all mandatory sections: purpose, intended use, out-of-scope uses, training data summary, performance metrics, limitations, known biases, ethical considerations, and maintenance schedule |
| S23.2-F4-AC02 | The datasheet template is drafted | it is reviewed | the template includes: dataset purpose, composition, collection process, pre-processing steps, distribution characteristics, known quality issues, sensitivity classification, and recommended uses |
| S23.2-F4-AC03 | The approval workflow is defined | it is reviewed by governance and delivery stakeholders | the workflow specifies review gates for each model stage transition (Development to Staging, Staging to Production) with required reviewers by role, maximum review turnaround time (e.g., five business days), and escalation paths |
| S23.2-F4-AC04 | Monitoring requirements are defined | they are reviewed by ML engineers | requirements specify: prediction drift monitoring (statistical tests and thresholds), feature drift monitoring, performance metric tracking (accuracy, precision, recall as applicable), data quality monitoring of input features, and alerting thresholds with response SLAs |
| S23.2-F4-AC05 | Retirement criteria are defined | they are reviewed by governance stakeholders | criteria include: performance degradation below defined thresholds for two consecutive monitoring periods, regulatory or policy change invalidating the model's use, training data becoming unrepresentative, owner-initiated retirement with a minimum 30-day deprecation notice to consumers |
| S23.2-F4-AC06 | Model lifecycle governance artefacts are complete | they are accepted by Water Corporation | all templates and workflow definitions are compatible with Unity Catalog Model Registry and can be enforced through Databricks platform mechanisms |

### Technical Notes
- Model cards should be stored as structured metadata in Unity Catalog Model Registry, not as standalone documents that drift out of sync.
- Datasheets for training datasets should reference the governed data products in Unity Catalog, creating lineage from model to training data.
- Approval workflows should leverage Databricks model stage transitions (None to Staging to Production to Archived) with webhook integrations for notification and approval enforcement.
- Monitoring should use Lakehouse Monitoring for inference table analysis, including prediction drift and feature drift detection.
- Align with the Data Science Lifecycle Deploy stage and MLOps discipline for operational model management patterns.
- Retirement should trigger archival in Unity Catalog (model stage set to Archived), endpoint decommissioning, and notification to registered consumers.

---

## Feature S23.2-F5: AI Agent Actions Bounded, Auditable, and Human-Supervised

**Description:** Anyone building, deploying, or overseeing an AI agent at Water Corporation can define its action boundaries, enforce least-privilege permissions, audit every action it takes, and ensure human oversight is in place — so that no agent operates beyond its sanctioned scope, every action is traceable, and a human can halt any agent within 60 seconds if needed.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S23.2-F5-US01 | AI/ML Governance Lead | have defined governance controls for AI agents that classify and constrain their autonomous actions | I can ensure agents operate within sanctioned boundaries and do not create unintended consequences |
| S23.2-F5-US02 | Data Platform Owner | have permission scoping standards requiring dedicated service principals with least-privilege access | I can enforce least-privilege access for agents interacting with data and systems |
| S23.2-F5-US03 | Data Scientist | have clear guidance on defining action boundaries for agents I build | I can design agents with governance compliance built in from the start |
| S23.2-F5-US04 | Data Governance Manager | have audit trail requirements mandating logging of every agent action, tool invocation, and decision point | I can review and investigate agent behaviour as part of governance operations |
| S23.2-F5-US05 | Data Consumer | have assurance that AI agents interacting with my data have defined boundaries, oversight, and a kill switch | I can trust that agent actions are traceable and reversible where appropriate |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S23.2-F5-AC01 | Action boundary standards are drafted | they are reviewed by governance and engineering stakeholders | the standards define a classification of agent action types (read-only, write, execute, external API call) with required approval level for each type and explicit prohibition of actions beyond sanctioned boundaries |
| S23.2-F5-AC02 | Permission scoping standards are defined | they are reviewed by platform and security teams | standards require: dedicated service principals per agent, Unity Catalog access controls scoped to the minimum required catalogs/schemas/tables, no shared credentials between agents, and mandatory credential rotation schedules |
| S23.2-F5-AC03 | Audit trail requirements are specified | they are reviewed by governance stakeholders | requirements mandate logging of: every agent action, every tool invocation, every data access event, every decision point, and every escalation, with logs retained for a minimum of 12 months and queryable within 24 hours |
| S23.2-F5-AC04 | Human-in-the-loop requirements are defined | they are reviewed | the framework specifies: which agent action types require human approval before execution, maximum autonomous decision thresholds (e.g., financial impact limits), mandatory human review frequency for production agents (at minimum monthly), and escalation triggers that halt agent execution pending human review |
| S23.2-F5-AC05 | Escalation and kill switch mechanisms are defined | they are reviewed by platform and governance teams | every production agent must have: a documented escalation path (agent to human handoff), a kill switch mechanism that halts execution within 60 seconds, a defined responsible party for kill switch activation, and a post-incident review process |
| S23.2-F5-AC06 | Agent governance standards are finalised | they are accepted by Water Corporation | the standards are compatible with the Mosaic AI Agent Framework and can be enforced through Databricks platform mechanisms (Unity Catalog permissions, agent service principal configuration, audit log integration) |

### Technical Notes
- Action boundary definitions should leverage the Mosaic AI Agent Framework's structured patterns for defining agent capabilities and restrictions, as referenced in the Data Science Lifecycle.
- Permission scoping should follow the principle of least privilege enforced through Unity Catalog: agents should have service principals with grants scoped to specific catalogs, schemas, and tables, not workspace-level access.
- Audit trails should integrate with Databricks Audit Logs and flow to the existing Splunk destination (per S8 Security & Compliance).
- Human-in-the-loop patterns should align with the three oversight models defined in the Data Governance Lifecycle Stage 9: human-in-the-loop, human-on-the-loop, and human-in-command.
- Kill switch mechanisms should be technically feasible within the Databricks platform (e.g., endpoint deactivation, service principal revocation, job cancellation API).
- RAG-based agents require additional governance per the Data Governance Lifecycle Stage 9: knowledge base sensitivity classification, freshness SLOs, and answer lineage tracing.
- Agent governance is an emerging discipline; the framework should be designed for iterative refinement as regulatory guidance (Australian Voluntary AI Safety Standard, potential future mandatory standards) evolves.

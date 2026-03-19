# S19 – Training and Knowledge Transfer: Feature Breakdown

**Scope Area:** Knowledge Transfer
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `governance/data-governance-roles.md` — roles requiring training (stewards, engineers, analysts, scientists)
- `lifecycles/data-governance-lifecycle.md` — governance processes and stewardship workflows to be trained
- `lifecycles/data-science-lifecycle.md` — MLOps, feature engineering, and model management processes
- `platform/edap-access-model.md` — ABAC, Unity Catalog, workspace topology, compute modes

---

## Feature S19-F1: Role-Based Learning Pathways Mapped for Every WC Team Member

**Description:** Every WC data and analytics team member has a personalised learning pathway showing exactly what training they need, in what sequence, and by when — based on a skills gap assessment that compares current capabilities against what their role requires to operate the EDAP independently.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S19-F1-US01 | WC Data & Analytics Manager | understand the skill gaps across my team relative to the capabilities required to operate the EDAP independently | I can plan resourcing and prioritise training investment |
| S19-F1-US02 | WC Data Engineer | see a personalised learning pathway that maps my current skills to the target skills required for my role on the EDAP | I know exactly what training I need and in what sequence |
| S19-F1-US03 | WC Project Manager | have a baselined training schedule that aligns knowledge transfer milestones with the delivery timeline | training is delivered when it is most relevant — not too early (forgotten) or too late (blocking) |
| S19-F1-US04 | WC Data Domain Steward | understand what governance training I need to perform my stewardship role in Alation and Unity Catalog | I am prepared to operate the governance workflows when they are handed over |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S19-F1-AC01 | A skills assessment survey has been distributed to all WC data and analytics staff | responses are collected and analysed | a gap analysis matrix is produced showing, for each role (Data Engineer, Data Analyst, Data Scientist, Data Steward, Platform Engineer), current skill level vs. target skill level across key competency areas |
| S19-F1-AC02 | Skill gaps have been identified | a training plan is drafted | the plan includes: role-based learning pathways, training modalities (e-learning, instructor-led, experiential), sequencing aligned to the PI delivery cadence, and responsible delivery party (SI, Databricks, or WC internal) |
| S19-F1-AC03 | The training plan has been drafted | it is reviewed by WC Data & Analytics management | the plan is endorsed by WC, with any modifications incorporated, and the schedule is baselined in the project plan |
| S19-F1-AC04 | Learning pathways are defined | the pathways are reviewed | each pathway lists specific courses, certifications, experiential activities, and estimated hours — with prerequisites clearly identified |

### Technical Notes
- The assessment should cover: Databricks platform fundamentals, PySpark/SQL, Delta Lake, Unity Catalog governance, Alation stewardship, Lakeflow Jobs/DLT, MLflow/MLOps, DataOps practices (CI/CD, testing), and ABAC/security concepts.
- Align role definitions to the governance roles wiki: Data Engineer maps to Technical Data Steward/Data Producer, Data Analyst maps to Data Consumer, Data Steward maps to Data Domain Steward.
- Training sequencing should follow the PI cadence: foundational training in early PIs, advanced and experiential training in later PIs as capabilities are built.

---

## Feature S19-F2: WC Engineers Certified on Databricks Platform Fundamentals

**Description:** WC data engineers, analysts, and scientists complete Databricks vendor-provided training pathways and pursue certifications — so that the team has verified, industry-recognised proficiency in the platform they will operate day to day.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S19-F2-US01 | WC Data Engineer | complete Databricks-provided data engineering training (e.g. Data Engineer Associate pathway) | I understand Delta Lake, Spark, Lakeflow Jobs, and Unity Catalog fundamentals |
| S19-F2-US02 | WC Data Analyst | complete Databricks SQL training | I can write queries, build dashboards, and use SQL Warehouses effectively |
| S19-F2-US03 | WC Data Scientist | complete Databricks ML training and certification | I understand MLflow, feature engineering, model serving, and Mosaic AI capabilities |
| S19-F2-US04 | WC Data & Analytics Manager | track completion rates and certification attainment across my team | I can measure training progress and identify individuals who need additional support |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S19-F2-AC01 | Databricks e-learning licences have been procured for all identified WC staff | training is available | each identified staff member has access to the Databricks Academy learning platform with their assigned role-based pathway |
| S19-F2-AC02 | A WC Data Engineer completes the assigned Databricks training pathway | completion is recorded | the individual's completion status is tracked in a central training register, with date completed and assessment score (where applicable) |
| S19-F2-AC03 | Instructor-led training sessions are scheduled | the sessions are delivered | attendance is recorded, session feedback is collected (satisfaction rating ≥ 4/5 target), and follow-up resources are distributed |
| S19-F2-AC04 | The Databricks certification pathway has been communicated | 6 months into the project | at least 50% of identified WC Data Engineers and Data Scientists have attempted or achieved at least one Databricks certification |
| S19-F2-AC05 | Training progress is tracked | a monthly training status report is produced | the report shows: courses assigned, courses completed, certifications attempted, certifications achieved, and individuals at risk of falling behind schedule |

### Technical Notes
- Databricks Academy provides self-paced e-learning aligned to certification paths: Data Engineer Associate, Data Engineer Professional, Machine Learning Associate, Machine Learning Professional, Data Analyst Associate.
- Instructor-led training may be delivered by Databricks or an authorised training partner — confirm availability for AWST-friendly scheduling.
- Certification exams are delivered via third-party proctoring — ensure WC staff have the technical setup required (webcam, stable internet, quiet environment).

---

## Feature S19-F3: WC Team Proficient in EDAP-Specific Patterns and Practices

**Description:** WC engineers, stewards, analysts, and platform staff are trained on the specific patterns, workflows, and standards built into the EDAP — covering DataOps practices, pipeline development, governance workflows, and operational procedures that go beyond generic Databricks training.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S19-F3-US01 | WC Data Engineer | receive training on the EDAP pipeline framework, including medallion architecture patterns, naming conventions, and the CI/CD deployment process | I can develop and deploy pipelines that conform to EDAP standards |
| S19-F3-US02 | WC Data Domain Steward | receive training on the Alation stewardship workflow, classification processes, and the Alation-to-UC tag sync mechanism | I can perform my stewardship responsibilities in the production environment |
| S19-F3-US03 | WC Data Analyst | receive training on the EDAP data model, how to discover data products in Alation, and how to query Gold-layer tables via SQL Warehouses | I can access and use EDAP data products independently |
| S19-F3-US04 | WC Platform Engineer | receive training on EDAP operational procedures: monitoring, alerting, incident response, break-glass access, and environment management | I can support the platform in BAU operations after the SI engagement concludes |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S19-F3-AC01 | SI-developed training materials have been authored for DataOps practices | a training session is delivered | the session covers: Git branching strategy, PR review process, CI/CD pipeline operation, automated testing, Lakeflow Jobs orchestration, and environment promotion (dev → staging → prod) |
| S19-F3-AC02 | SI-developed governance training materials have been authored | a training session is delivered to stewards | the session covers: Alation navigation, applying classifications and tags, certifying data products, metadata review cadence, and interpreting governance dashboards |
| S19-F3-AC03 | Each SI-developed training module is delivered | a hands-on exercise accompanies the module | attendees complete a practical exercise in the EDAP dev workspace that demonstrates the concepts covered, with a completion check confirming understanding |
| S19-F3-AC04 | All SI-developed training sessions are complete | training materials are reviewed | all materials (slide decks, exercise notebooks, reference sheets) are stored in a WC-owned repository and licensed for ongoing internal use without restriction |
| S19-F3-AC05 | Training covers all scope items developed by the SI | a coverage matrix is reviewed | every scope item (S5 through S23) has at least one associated training module or reference artefact addressing the knowledge required to operate and maintain the delivered capability |

### Technical Notes
- Training content must reference the specific WC EDAP implementation — generic Databricks training is covered by S19-F2; this feature covers WC-specific practices, patterns, and configuration.
- Governance training should reference the tagging strategy, access model, and governance lifecycle wikis as source material.
- Hands-on exercises should use the dev workspace with realistic (but anonymised) data per ADR-EDP-001.
- All training materials become WC intellectual property upon delivery — confirm IP terms in the SI contract.

---

## Feature S19-F4: WC Engineers Building Capability Through Hands-On Delivery

**Description:** WC staff are embedded in SI delivery activities — shadowing, pair programming, and contributing to real sprint stories — so that they build practical skills through guided experience that classroom training alone cannot provide.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S19-F4-US01 | WC Data Engineer | shadow SI engineers during pipeline development, observing design decisions, coding patterns, and troubleshooting approaches | I learn practical skills that cannot be conveyed through classroom training alone |
| S19-F4-US02 | WC Data Engineer | pair-programme with SI engineers on real sprint stories | I build confidence and competence by contributing to actual deliverables under expert guidance |
| S19-F4-US03 | WC Data & Analytics Manager | have a structured rotation schedule ensuring each WC team member gets dedicated experiential learning time with SI staff | experiential learning is systematic, not ad-hoc, and every team member benefits |
| S19-F4-US04 | WC Data Domain Steward | participate in stewardship activities alongside SI governance specialists | I develop practical proficiency in Alation workflows and classification decision-making |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S19-F4-AC01 | An experiential learning roster has been created | the roster is reviewed | each identified WC participant has a scheduled rotation with specific SI team members, covering defined topics, with start and end dates aligned to the PI cadence |
| S19-F4-AC02 | A WC Data Engineer is paired with an SI engineer for a sprint | the sprint completes | the WC engineer has contributed to at least one user story (code committed, reviewed, and merged) with documented evidence of their contribution |
| S19-F4-AC03 | Shadowing sessions are scheduled | each session completes | a brief session log is recorded noting: date, WC participant, SI mentor, topic covered, and key learnings — these logs feed into the training progress tracker |
| S19-F4-AC04 | The experiential learning programme has run for three PIs | the programme is evaluated | at least 80% of WC participants rate the experiential learning as "effective" or "highly effective" in a post-programme survey |
| S19-F4-AC05 | WC staff are embedded in SI delivery | sprint retrospectives are conducted | WC participants provide feedback on the experiential learning within sprint retros, and the SI adjusts the programme based on feedback |

### Technical Notes
- Experiential learning should begin no later than PI 2, once WC staff have completed foundational training (S19-F2), so they have sufficient baseline knowledge to benefit from the experience.
- Pair programming sessions should use the EDAP dev workspace with shared screen or VS Code Live Share for remote collaboration.
- Rotation topics should align to the scope items being delivered in the current PI — learning is most effective when it coincides with active development.
- SI staff should be briefed on their mentoring responsibilities and allocated time for this — it should not be treated as a secondary activity.

---

## Feature S19-F5: Reusable Templates and Runbooks Available for Ongoing Operations

**Description:** WC staff have a comprehensive set of reference artefacts — coding standards, notebook templates, quick reference sheets, FAQs, and operational runbooks — that they can use to work independently after the SI engagement concludes, without needing to rediscover patterns or reinvent procedures.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S19-F5-US01 | WC Data Engineer | reference a coding standards document that defines naming conventions, code structure, logging patterns, and quality expectations for EDAP pipelines | I can write code that is consistent with established patterns and passes code review |
| S19-F5-US02 | WC Data Engineer | clone a notebook template that includes the standard boilerplate (imports, configuration, logging, DQ checks) for a new pipeline | I can start new development quickly without reinventing the standard structure |
| S19-F5-US03 | WC Platform Engineer | reference operational runbooks for common tasks: deploying a new pipeline, onboarding a new domain, troubleshooting a failed job, executing the break-glass process | I can perform operational tasks independently using documented procedures |
| S19-F5-US04 | WC Data Analyst | consult a FAQ and quick reference sheet covering common questions: how to find data, how to request access, how to interpret governance tags, how to use SQL Warehouses | I can resolve routine questions without raising a support ticket |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S19-F5-AC01 | A coding standards document has been authored | a WC Data Engineer reviews it | the document covers: Python/PySpark coding conventions, SQL style guide, naming conventions (tables, columns, pipelines, jobs), logging standards, error handling patterns, and DQ check implementation |
| S19-F5-AC02 | Notebook templates have been created | a WC Data Engineer clones a template to start a new pipeline | the template includes: standard imports, environment-aware configuration loading, logging initialisation, DQ assertion framework, and inline documentation comments explaining each section |
| S19-F5-AC03 | Operational runbooks have been authored | a WC Platform Engineer follows the "deploy a new pipeline" runbook step by step | the runbook is sufficient to complete the task without additional guidance — verified by a WC staff member completing it independently during UAT |
| S19-F5-AC04 | FAQs and quick reference sheets have been produced | they are published in an accessible location (e.g. Alation, SharePoint, or the WC knowledge base) | WC staff can find and access the artefacts without requiring Databricks workspace access |
| S19-F5-AC05 | All knowledge artefacts are reviewed | a completeness check is performed | artefacts exist for every major process, workflow, and capability delivered by the SI — verified against the scope item coverage matrix from S19-F3-AC05 |

### Technical Notes
- Coding standards should align to the pipeline framework specification (EDAP-FWK-001) and the medallion architecture document.
- Notebook templates should be stored in the EDAP Git repository and deployable via DABs.
- Runbooks should follow a consistent format: purpose, prerequisites, step-by-step instructions, verification steps, troubleshooting, and escalation contacts.
- All artefacts become WC intellectual property — store in WC-owned repositories and knowledge systems.
- Quick reference sheets should be concise (1–2 pages) and printable — they are designed for desk reference, not as comprehensive guides.

---

## Feature S19-F6: WC DataOps Team Embedded in the Code Review Process

**Description:** WC DataOps engineers participate in every pull request review from day one — progressing from observer to co-reviewer to independent reviewer — so that by project end they can confidently approve, request changes, and maintain pipeline code without SI involvement.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S19-F6-US01 | WC DataOps Engineer | be added as a required reviewer on all PRs deploying code to staging and production environments | I gain exposure to every codebase change and learn the review standards by practice |
| S19-F6-US02 | WC DataOps Engineer | receive guidance from SI engineers on what to look for during code review (correctness, performance, security, standards compliance) | I develop the judgement needed to review code independently |
| S19-F6-US03 | WC Data & Analytics Manager | track the progression of WC reviewers from observer to independent reviewer over the project lifecycle | I can measure readiness for the transition to BAU code review ownership |
| S19-F6-US04 | SI Tech Lead | ensure WC review participation does not delay sprint velocity beyond acceptable limits | knowledge transfer and delivery velocity are balanced |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S19-F6-AC01 | WC DataOps team members have been added to the PR review configuration in GitHub | a PR is raised for code deploying to staging or production | at least one WC reviewer is included as a required reviewer and must approve before merge |
| S19-F6-AC02 | A WC reviewer is participating in code review | the first three PIs are reviewed | the WC reviewer's role progresses from observer (PI 1) to co-reviewer with SI guidance (PI 2) to independent reviewer (PI 3+), with progression documented |
| S19-F6-AC03 | WC reviewers are actively reviewing PRs | a sample of reviewed PRs is inspected | WC reviewers are providing substantive comments (not just approvals), including feedback on coding standards, naming conventions, DQ checks, and security considerations |
| S19-F6-AC04 | A code review standards guide has been produced | WC reviewers reference it during reviews | the guide defines: what to check (logic, performance, security, standards, tests), how to provide constructive feedback, approval criteria, and when to request changes |
| S19-F6-AC05 | The project is within 2 PIs of conclusion | the WC DataOps team's review capability is assessed | WC reviewers are independently approving or requesting changes on PRs with confidence, as evidenced by review quality metrics and SI tech lead endorsement |

### Technical Notes
- Configure GitHub branch protection rules to require at least one WC reviewer approval for merges to `main` and `release/*` branches.
- WC reviewer turnaround time should be baselined (e.g. 24-hour SLA for review) to avoid delivery bottlenecks.
- The code review standards guide should reference the EDAP coding standards (S19-F5-AC01) and include a checklist format for consistent review quality.
- Track review participation metrics: PRs reviewed per sprint, comments per review, approval/change-request ratio — report as part of the training progress dashboard.
- This feature operates throughout the project duration — it is not a one-off training event but a continuous knowledge transfer mechanism.

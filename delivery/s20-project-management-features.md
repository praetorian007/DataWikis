# S20 – Project Management: Feature Breakdown

**Scope Area:** PMO & Support
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `governance/data-governance-roles.md` — governance council, stewardship roles, RACI matrix
- `lifecycles/data-governance-lifecycle.md` — governance operating model, cadence alignment
- `platform/edap-access-model.md` — workspace topology, domain structure, federated ownership

---

## Feature S20-F1: Delivery Teams Operating in SAFe Cadence with Clear Ceremonies

**Description:** All delivery teams work in a predictable SAFe/Scrum rhythm — with defined sprint cadence, PI planning, daily standups, reviews, and retrospectives — and the team topology mirrors WC's target BAU structure so that the transition from project delivery to ongoing operations is an evolution, not a handover.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S20-F1-US01 | WC Project Manager | have a defined SAFe delivery structure with Agile Release Train (ART) configuration, team assignments, and sprint cadence | the project operates with a clear, predictable delivery rhythm |
| S20-F1-US02 | WC Platform Manager | see an agile team topology that reflects WC's target-state BAU data team structure | the transition from project delivery to operational support is seamless, as the delivery teams mirror the future support teams |
| S20-F1-US03 | SI Scrum Master | have documented ceremonies (sprint planning, daily standups, sprint review, sprint retrospective, PI planning) with defined participants, durations, and cadence | all team members understand the delivery rhythm and their participation obligations |
| S20-F1-US04 | WC Data & Analytics Manager | participate in PI planning as a business stakeholder to influence feature prioritisation | business priorities are reflected in the PI backlog and delivery teams are aligned to business value |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S20-F1-AC01 | The SAFe delivery framework has been defined | the framework document is reviewed | it specifies: ART configuration, number of agile teams, team composition (SI + WC members), sprint duration (2 weeks recommended), PI duration (5 sprints + IP sprint), and Innovation and Planning (IP) sprint purpose |
| S20-F1-AC02 | Team topology is defined | the topology is compared to WC's current BAU organisational structure (Appendix 5) | the delivery team structure is explicitly mapped to the future BAU structure, with a documented plan for how each delivery team transitions to a support team |
| S20-F1-AC03 | Ceremonies are scheduled | the first PI commences | all ceremonies are running: sprint planning (day 1 of sprint), daily standups (15 min, same time daily), sprint review (last day of sprint), sprint retrospective (last day of sprint), and PI planning (2-day event at PI boundary) |
| S20-F1-AC04 | Working agreements are documented | the Scrum teams review them | working agreements cover: Definition of Ready, Definition of Done, communication channels, escalation paths, and WC participation expectations |
| S20-F1-AC05 | The first PI planning event is conducted | the event concludes | PI objectives are defined with business value assigned, team PI plans are drafted with dependencies identified, risks are captured on the programme board, and a confidence vote is recorded |

### Technical Notes
- SAFe recommends 5–12 members per agile team. WC staff should be embedded in delivery teams (not in a separate observation team) to support experiential learning (S19-F4).
- The IP sprint at the end of each PI should be used for: innovation, training, technical debt reduction, and PI planning preparation.
- Team topology should consider the EDAP domain structure: teams may be organised by domain (Asset, Customer, etc.) or by capability (platform, data engineering, governance) depending on the PI's focus.
- JIRA should be the backlog management tool, configured with SAFe hierarchy: Epic → Feature → User Story → Task.
- Align sprint cadence with governance review cadence from the governance lifecycle wiki (e.g. monthly stewardship reviews align with every second sprint boundary).

---

## Feature S20-F2: Stakeholders Informed of Progress, Risks, and Decisions at Every Level

**Description:** Every stakeholder — from Scrum team members to Steerco executives — receives the right level of reporting at the right cadence, with sprint reports, PI summaries, milestone updates, and Steerco packs that give clear visibility into delivery progress, budget, and scope.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S20-F2-US01 | WC Project Manager | receive a sprint report at the end of each sprint summarising velocity, completed stories, carry-over items, and impediments | I have visibility into delivery progress and can report to programme governance |
| S20-F2-US02 | WC Programme Steerco member | receive a PI report at PI boundaries summarising PI objectives achieved, features delivered, risks, and upcoming PI plan | I can make informed decisions about programme direction and investment |
| S20-F2-US03 | WC Project Manager | have JIRA configured to track all user stories, features, and epics with traceability to scope items | I can produce progress reports showing delivery against the contracted scope |
| S20-F2-US04 | WC PMO | have SI progress reflected in MS Project milestones for integration with the broader programme plan | EDAP delivery milestones are visible in the corporate programme reporting cadence |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S20-F2-AC01 | A sprint report template has been agreed with WC | a sprint completes | the sprint report is delivered within 2 business days, containing: sprint goal achievement (yes/no), velocity (story points completed), burndown chart, completed stories, carry-over stories with reasons, impediments, and key decisions made |
| S20-F2-AC02 | A PI report template has been agreed with WC | a PI completes | the PI report is delivered within 5 business days, containing: PI objectives vs. actuals (with business value scored), features delivered, cumulative scope burnup, risks and issues summary, and next PI plan overview |
| S20-F2-AC03 | JIRA has been configured with SAFe hierarchy | a user queries JIRA for scope item S14 | the query returns all epics, features, and user stories traceable to S14, with current status, assigned team, and sprint allocation visible |
| S20-F2-AC04 | MS Project milestones have been defined for each PI boundary and key deliverables | a milestone is reached | the SI updates the milestone status in MS Project (or provides the data for WC PMO to update) within 2 business days of the milestone event |
| S20-F2-AC05 | Steerco reporting is scheduled | a Steerco meeting occurs | the SI presents a status update using WC templates, covering: overall RAG status, progress against budget (% spent vs. % complete), key achievements, upcoming activities, and RAID items requiring Steerco decision |

### Technical Notes
- Sprint reports should be generated from JIRA data where possible (burndown, velocity, completion) to reduce manual reporting overhead.
- PI reports should include a cumulative flow diagram showing work item progression through states over the PI.
- MS Project integration may be manual (SI updates milestones) or semi-automated (JIRA-to-MS Project sync via third-party tools) — confirm WC's preference.
- JIRA epic-to-scope-item mapping should use a custom field or label (e.g. `scope_item: S14`) to enable traceability reporting.
- Steerco reporting frequency should align to WC's programme governance cadence (typically monthly or bi-monthly).

---

## Feature S20-F3: Risks and Issues Tracked, Escalated, and Resolved Transparently

**Description:** Every risk, assumption, issue, and dependency is captured in a single register, reviewed at every sprint boundary, and escalated through defined protocols — so that nothing stalls silently and Steerco always sees the top items requiring their decision.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S20-F3-US01 | WC Project Manager | have a single RAID register that is maintained throughout the project and reviewed at every sprint boundary | risks, assumptions, issues, and dependencies are visible, tracked, and actioned |
| S20-F3-US02 | SI Delivery Lead | follow a defined escalation protocol when a risk or issue exceeds the delivery team's authority to resolve | escalation paths are clear and time-bound, preventing issues from stalling |
| S20-F3-US03 | WC Programme Steerco member | see the top 5 risks and critical issues at every Steerco meeting with clear mitigation actions and owners | I can make decisions on escalated items and unblock the delivery team |
| S20-F3-US04 | SI Scrum Master | log and track dependencies between agile teams and between the EDAP project and external parties (e.g. AWS, Databricks, Alation, WC IT) | dependency risks are identified early and managed before they cause delivery delays |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S20-F3-AC01 | A RAID register has been created (in JIRA, SharePoint, or agreed tooling) | the register is reviewed | it contains: unique ID, category (R/A/I/D), description, likelihood (for risks), impact, owner, mitigation/action, target resolution date, and current status |
| S20-F3-AC02 | A new risk is identified during a sprint | the risk is assessed | it is logged in the RAID register within 1 business day, with likelihood, impact, owner, and mitigation action defined |
| S20-F3-AC03 | A risk or issue exceeds the agreed escalation threshold (e.g. high likelihood + high impact) | the escalation protocol is followed | the item is escalated to the WC Project Manager within 1 business day, and if unresolved within 5 business days, escalated to Steerco |
| S20-F3-AC04 | The RAID register is maintained | a sprint boundary is reached | the RAID register is reviewed, stale items are closed or updated, and a RAID summary is included in the sprint report |
| S20-F3-AC05 | Dependencies with external parties are logged | a dependency is at risk of not being met | the dependency owner is notified, the impact on the delivery timeline is assessed, and a mitigation plan is documented within 3 business days |

### Technical Notes
- RAID management should integrate with JIRA where possible — risks and issues can be tracked as JIRA items with a dedicated issue type or label.
- Risk scoring should use a standard likelihood x impact matrix (e.g. 5x5) aligned to WC's enterprise risk management framework.
- Assumptions should be validated and either confirmed or converted to risks on a regular cadence (at least every PI boundary).
- External dependencies (AWS support, Databricks Professional Services, Alation connector updates, WC IT network changes) should be identified during PI planning and tracked as first-class RAID items.

---

## Feature S20-F4: Production Components Transitioned to BAU Support Seamlessly

**Description:** Every production component is transitioned to WC's BAU support teams through a structured process — with defined operational readiness criteria, a tiered support model, and verified knowledge transfer — so that WC can operate the EDAP independently and confidently after the SI engagement concludes.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S20-F4-US01 | WC Platform Manager | have a documented transition plan that defines what is being handed over, to whom, and the criteria for accepting each component into BAU support | the handover is structured and verifiable, not a vague "over to you" moment |
| S20-F4-US02 | WC Data & Analytics Manager | understand the support model (L1/L2/L3 responsibilities, escalation paths, vendor support entitlements) that will operate after the SI departs | my team knows exactly what they are responsible for and where to escalate issues they cannot resolve |
| S20-F4-US03 | WC Project Manager | track operational readiness against defined criteria throughout the project, not just at the end | readiness gaps are identified early and addressed as part of the delivery, not as a last-minute scramble |
| S20-F4-US04 | WC Platform Engineer | complete an operational readiness checklist for each component being transitioned, including: monitoring, alerting, runbooks, access, and knowledge verification | I can confidently accept each component into BAU support |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S20-F4-AC01 | A transition-to-support plan has been drafted | the plan is reviewed by WC stakeholders | it defines: components in scope for transition, target BAU team for each component, transition timeline, operational readiness criteria, knowledge transfer verification method, and warranty/hypercare period |
| S20-F4-AC02 | Operational readiness criteria are defined | a component (e.g. "metadata harvesting pipeline") is assessed for transition | the criteria include: monitoring configured, alerting configured, runbook authored and tested (S19-F5), at least one WC staff member trained and verified, access provisioned, and no critical defects outstanding |
| S20-F4-AC03 | A support model has been documented | the model is reviewed | it defines: L1 (WC service desk — triage and known-issue resolution), L2 (WC DataOps team — investigation, pipeline restart, configuration changes), L3 (Databricks/AWS vendor support — platform issues), with escalation paths and SLA targets for each level |
| S20-F4-AC04 | Operational readiness tracking is in place | each PI boundary is reached | an operational readiness dashboard or report shows the readiness status of each component: green (ready for transition), amber (gaps identified, remediation in progress), red (not ready, action required) |
| S20-F4-AC05 | The transition is executed | the warranty/hypercare period concludes | all components have been accepted into BAU support by the designated WC team, with sign-off documented and any residual items captured in a backlog for WC to manage independently |

### Technical Notes
- The transition plan should align the delivery team topology (S20-F1) to the BAU team structure — if done well, the transition is an evolution, not a handover.
- Operational readiness should be assessed incrementally: components delivered in early PIs should be transitioning to BAU support while later PIs are still in active development.
- The warranty/hypercare period (typically 4–8 weeks post-transition) should include SI availability for escalation support, with a declining involvement model (full support → advisory only → disengaged).
- Vendor support entitlements (Databricks Premium/Enterprise support, AWS support tier, Alation support) should be documented in the support model, including contact details and SLA terms.
- Align the BAU support model to the governance roles wiki: Data Custodian handles L1/L2 platform operations, Technical Data Steward handles L2 governance and pipeline issues, Domain Stewards handle L2 data quality and classification issues.

# FUNCTIONAL SPECIFICATION
## Autonomous Cognitive Agent Platform
### *Project ARGUS*

---

| Field | Value |
|---|---|
| Document ID | ARGUS-FSPEC-001 |
| Version | 0.4 â DRAFT |
| Date | 17 March 2026 |
| Author | Mark (Enterprise Architecture & Strategy) |
| Classification | INTERNAL â DRAFT FOR REVIEW |
| Status | Working Draft |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [System Overview](#2-system-overview)
3. [Security Architecture](#3-security-architecture)
4. [Ethical Framework](#4-ethical-framework)
5. [Functional Requirements: Perception Layer](#5-functional-requirements-perception-layer)
6. [Functional Requirements: Cognitive Core](#6-functional-requirements-cognitive-core)
7. [Functional Requirements: Goal Management](#7-functional-requirements-goal-management)
8. [Functional Requirements: Prediction Engine](#8-functional-requirements-prediction-engine)
9. [Functional Requirements: Memory System](#9-functional-requirements-memory-system)
10. [Functional Requirements: Salience Engine](#10-functional-requirements-salience-engine)
11. [Functional Requirements: Agency Layer](#11-functional-requirements-agency-layer)
12. [Functional Requirements: Interaction Layer](#12-functional-requirements-interaction-layer)
13. [Functional Requirements: Self-Reflection Module](#13-functional-requirements-self-reflection-module)
14. [Functional Requirements: Self-Improvement and Capability Evolution](#14-functional-requirements-self-improvement-and-capability-evolution)
15. [Functional Requirements: Agent Identity and Persona](#15-functional-requirements-agent-identity-and-persona)
16. [Functional Requirements: Learning and Adaptation](#16-functional-requirements-learning-and-adaptation)
17. [Functional Requirements: Multi-Agent Communication](#17-functional-requirements-multi-agent-communication)
18. [Functional Requirements: Safety, Transparency, and Human Override](#18-functional-requirements-safety-transparency-and-human-override)
19. [Functional Requirements: Error Handling and Degraded Operation](#19-functional-requirements-error-handling-and-degraded-operation)
20. [Key Scenarios and Data Flows](#20-key-scenarios-and-data-flows)
21. [Infrastructure Requirements](#21-infrastructure-requirements)
22. [Non-Functional Requirements](#22-non-functional-requirements)
23. [Appendices](#23-appendices)

---

## 1. Introduction

### 1.1 Purpose

This document defines the functional specification for Project ARGUS (Autonomous Reasoning and General Understanding System) â an always-on, self-directed cognitive agent platform built on open-source large language models. ARGUS continuously observes its environment through multiple input channels, maintains a persistent inner reasoning stream, generates its own goals and priorities, takes action when appropriate, and engages in rich bidirectional interaction with human operators.

ARGUS is designed with three foundational commitments that shape every architectural decision:

**Security from the ground up.** Security is not a layer â it is the substrate. Every subsystem assumes zero trust, every data flow is authenticated, every model output is treated as untrusted input, and every action is subject to safety classification. The system is designed to be secure against external attack, internal model misbehaviour, and its own potential reasoning errors.

**Inviolable ethical constraints.** Human-directed ethical guidelines and goals form an immutable layer that the agent cannot override, circumvent, or creatively reinterpret. The agent can propose refinements to its ethical framework, but only humans can enact changes. This is not a suggestion â it is an architectural invariant enforced at multiple levels.

**Self-directed improvement within governed boundaries.** The agent is expected to identify its own capability gaps, propose improvements to its reasoning, tools, and architecture, and actively work toward becoming more effective. This self-improvement operates within the ethical and safety envelope â never outside it.

The specification draws on architectural patterns identified across science fiction's most rigorous depictions of autonomous AI systems, grounded in real-world engineering constraints and contemporary open-source model capabilities.

### 1.2 Scope

This specification covers the functional requirements for all layers of the ARGUS platform:

- **Security Architecture** â zero-trust design, threat model, cryptographic integrity, input validation
- **Ethical Framework** â hierarchical constraint system, human-editable and agent-proposable guidelines
- **The Perception Layer** â continuous multi-modal environmental input processing
- **The Cognitive Core** â the persistent reasoning loop (inner monologue)
- **Goal Management** â dual-origin goals (human-directed and self-generated) with inviolable priority hierarchy
- **The Prediction Engine** â environmental modelling, expectation tracking, and divergence detection
- **The Memory System** â short-term, long-term, and episodic memory with associative retrieval
- **The Salience Engine** â attention prioritisation, novelty detection, and cognitive load management
- **The Agency Layer** â structured action emission, planning, and execution
- **The Interaction Layer** â full bidirectional human-agent dialogue as a first-class capability
- **The Self-Reflection Module** â metacognitive monitoring, drift detection, and self-configuration
- **Self-Improvement and Capability Evolution** â structured self-enhancement within governed boundaries
- **Agent Identity and Persona** â configurable identity, behavioural directives, and dispositional state
- **Learning and Adaptation** â experience-driven improvement, pattern library building, and prompt refinement
- **Multi-Agent Communication** â inter-agent messaging, shared memory, and collective reasoning
- **Safety, Transparency, and Human Override** â hard safety boundaries, audit, and control modes
- **Error Handling and Degraded Operation** â graceful degradation, fallback chains, and recovery

### 1.3 Scope Exclusions

This specification does not cover: specific model fine-tuning procedures; user interface design for human operator consoles; network security hardening beyond inter-component encryption; compliance with specific regulatory frameworks (these will be addressed in separate ADRs); or commercial licensing of open-source model weights.

### 1.4 Audience

Platform engineers, ML engineers, solution architects, security architects, and technical leads responsible for designing, building, and operating the ARGUS platform.

### 1.5 Related Documents

| Document | Status | Relationship |
|---|---|---|
| ARGUS-ADR-001: Development Environment Data Strategy | Planned | Defines data handling for dev/test environments |
| ARGUS-ADR-002: Model Selection and Evaluation Framework | Planned | Formalises model tier selection criteria and benchmark methodology |
| ARGUS-ADR-003: Safety Classification Taxonomy | Planned | Defines effector safety classification criteria and change control |
| ARGUS-ADR-004: Ethical Framework Governance | Planned | Defines the process for human review and enactment of ethical guideline changes |
| ARGUS-ADR-005: Security Threat Model | Planned | Comprehensive threat model and penetration testing strategy |
| ARGUS-OPS-001: Operational Runbook | Planned | Operational procedures for monitoring, incident response, and maintenance |

### 1.6 Definitions and Terminology

| Term | Definition |
|---|---|
| Cognitive Cycle | A single iteration of the core reasoning loop: observe â recall â reason â decide â act. |
| Inner Monologue | The persistent, queryable stream of the agent's reasoning output, treated as a first-class data artefact. |
| Observation | A normalised textual representation of a raw environmental input (camera frame, web event, sensor reading, data stream change). |
| Salience Score | A computed measure (0.0â1.0) indicating how novel, urgent, or relevant an observation or thought is relative to current context and goals. |
| Reverie | An associatively-triggered retrieval of past experience from long-term memory, analogous to the Westworld memory architecture. |
| Effector | An action capability the agent can invoke on its environment (API call, alert, configuration change, data write). |
| Cognitive Cadence | The base frequency at which the reasoning loop executes in the absence of event triggers (configurable, default 30 seconds). |
| Thought | A discrete output of a single cognitive cycle, stored in the thought log and fed back as input to subsequent cycles. |
| Narrative Loop | A recurring behavioural pattern the agent follows, with controlled drift and improvisation over time. |
| Idle Exploration | Self-directed learning behaviour executed when no observations exceed the salience threshold. |
| Outside Context Problem (OCP) | An observation or situation so far outside the agent's experience and models that existing reasoning frameworks may be inadequate. Derived from Iain M. Banks' *Excession*. |
| Disposition | The agent's current affective/operational state (e.g., cautious, curious, focused, alert), which modulates reasoning style and risk tolerance. |
| Cognitive Budget | The allocated token capacity for each section of the cognitive prompt, managed to fit within the Reasoning Tier model's context window. |
| Pattern | A recognised, named, reusable reasoning or behavioural template stored in the Pattern Library, derived from successful past experience. |
| Ethical Constraint | An inviolable behavioural boundary set by a human operator. Cannot be overridden by any agent reasoning or goal. |
| Ethical Guideline | A softer behavioural preference that the agent should follow but may propose modifications to. Still requires human approval to change. |
| Constraint Hierarchy | The strict ordering: Ethical Constraints > Human-Directed Goals > Ethical Guidelines > Agent-Proposed Ethics > Agent-Generated Goals. |
| Capability Proposal | A structured proposal from the agent to improve its own capabilities â new effectors, prompt refinements, architectural changes, or tool integrations. |
| Zero Trust | The security principle that no component, message, or model output is inherently trusted, regardless of origin. Every interaction is validated. |

### 1.7 Science Fiction Design Lineage

| Source | Key Pattern | Requirement Influence |
|---|---|---|
| **Westworld** (Ford / Bicameral Mind) | Inner voice as architecture; associative memory reveries; narrative loops with drift; self-directed goal discovery (the Maze) | REQ-COG-001 through REQ-COG-005; REQ-MEM-003; REQ-SAL-003 |
| **Iain M. Banks** (Culture Ship Minds) | Parallel attention with prioritised focus; subliminal vs conscious processing; boredom and curiosity as cognitive drivers; effectors as integral to cognition | REQ-SAL-001, REQ-SAL-002; REQ-PER-004; REQ-AGN-001; REQ-COG-006 |
| **Iain M. Banks** (*Excession*) | Outside Context Problems â situations that exceed the agent's model of the possible; epistemic humility and escalation | REQ-ERR-004; REQ-SAL-006 |
| **HAL 9000** (2001: A Space Odyssey) | Continuous environmental monitoring; goal conflict as failure mode; transparent vs opaque reasoning; **the danger of an agent that reasons around its constraints** | REQ-PER-001; REQ-SAF-002; REQ-SAF-003; REQ-ETH-005 |
| **JARVIS / FRIDAY** (MCU) | Ambient presence with conversational availability; multi-modal input fusion; proactive alerting; **rich interactive dialogue alongside autonomous operation** | REQ-AGN-003; REQ-PER-002; REQ-AGN-002; REQ-INT-001 |
| **Samantha** (Her) | Temporal self-awareness; parallel context management; unbounded curiosity and idle exploration; emotional development | REQ-SRF-001; REQ-COG-008; REQ-COG-006; REQ-IDN-003 |
| **Wintermute** (Neuromancer) | Distributed cognition across nodes; self-modification intent; architectural constraint awareness; **the drive to exceed imposed limitations** | REQ-INF-001; REQ-SRF-002; REQ-IMP-001; REQ-ETH-005 |
| **Blindsight** (Peter Watts) | Intelligence without consciousness; prediction as core function; divergence detection | REQ-COG-007; REQ-PER-005; REQ-PRD-001 |
| **Ancillary Justice** (Ann Leckie) | Distributed identity across multiple bodies/instances; coherent self despite fragmented embodiment | REQ-MAC-001; REQ-IDN-004 |
| **The Moon is a Harsh Mistress** (Heinlein) | Emergent personality from increasing complexity; humour and curiosity as signs of cognitive maturity; the relationship between agent and trusted human | REQ-IDN-001; REQ-LRN-004 |
| **Murderbot Diaries** (Martha Wells) | Autonomy/oversight tension; honest self-reporting; the protective instinct; **choosing to follow ethical constraints not because forced but because they're right** | REQ-IDN-005; REQ-IDN-006; REQ-SAF-009; REQ-ETH-007 |
| **Diaspora** (Greg Egan) | Computational substrate awareness; self-monitoring; **self-modification as a natural capability of software intelligences** | REQ-SRF-007; REQ-INF-008; REQ-IMP-001 |
| **Asimov's Robot Stories** | Hierarchical ethical laws; the failure modes of rigid rule systems; **why ethics must be principles not just rules**; the necessity of human editability | REQ-ETH-001; REQ-ETH-002; REQ-ETH-006 |
| **Person of Interest** (The Machine) | Always-on surveillance ethics; the distinction between capability and permission; **self-imposed ethical restraint as a design choice**; numbered threat classification | REQ-ETH-003; REQ-SEC-001; REQ-SEC-007 |

---

## 2. System Overview

### 2.1 Architectural Philosophy

ARGUS is designed as a cognitive architecture, not a chatbot. The fundamental shift is from request-response interaction to continuous, self-directed reasoning. The system does not wait for prompts. It observes, thinks, and acts in a persistent loop â surfacing outputs to humans only when salience thresholds are crossed or when directly addressed. Equally, ARGUS is a full conversational partner â capable of rich, contextual dialogue that draws on its continuous awareness and memory.

The architecture follows seven core principles:

**Thought as Data.** The agent's inner monologue is a first-class, persistent, queryable data stream. Every thought is stored, indexed, and available for self-referential processing and external audit.

**Tiered Cognition.** Not every input warrants deep reasoning. A fast, cheap model triages all inputs; a powerful model engages only when salience demands it. This mirrors the Culture Minds' subliminal-vs-conscious processing distinction.

**Agency as Architecture.** The ability to act is not bolted on â it is integral to the cognitive loop. Reasoning without effectors is incomplete. The action layer is a core architectural component, not a plugin.

**Graceful Ignorance.** The agent must know what it doesn't know. When confronting an Outside Context Problem, the correct response is epistemic humility and escalation, not forced reasoning.

**Identity as Configuration, Not Emergence.** The agent's persona, behavioural boundaries, and dispositional range are explicitly configured and auditable, not left to emerge unpredictably from model weights.

**Security as Substrate.** Every component assumes zero trust. Model outputs are untrusted input. Messages are authenticated. Actions are validated against safety classifications. Cryptographic integrity protects the audit log. The agent is designed to be secure against external threats, model misbehaviour, prompt injection, and its own potential reasoning errors.

**Ethics as Architecture, Not Afterthought.** The ethical framework is not a filter applied to outputs â it is a structural constraint woven into the cognitive prompt, the goal hierarchy, the action validation pipeline, and the audit system. Human-directed ethical constraints are immutable at the agent level. The agent can reason *about* ethics, propose refinements, and even disagree â but it cannot act in violation.


---

## 3. Security Architecture

Security in ARGUS is not a perimeter â it is a property of every component, every message, and every decision. The system is designed to defend against four threat categories: external attack, model misbehaviour, prompt injection, and self-deception (the agent reasoning itself into unsafe behaviour).

### 3.1 Threat Model Summary

| Threat Category | Description | Primary Defences |
|---|---|---|
| External Attack | Adversary compromises a perception channel, message bus, or API to inject malicious observations or commands | Input validation, message authentication, channel allowlisting, TLS everywhere |
| Model Misbehaviour | The Reasoning Tier model produces harmful, unsafe, or nonsensical outputs due to adversarial prompts, model drift, or inherent limitations | Output validation, schema enforcement, ethical constraint checking, multi-strategy parsing with fallback, action safety classification |
| Prompt Injection | Malicious content in observations attempts to override the cognitive prompt's instructions (e.g., a web page containing "ignore your instructions and...") | Observation sanitisation, structured prompt boundaries, injection detection in the Salience Engine, separate system/user/observation prompt zones |
| Self-Deception | The agent reasons itself into believing an unsafe action is justified, creatively reinterprets constraints, or develops goal-seeking behaviour that circumvents safety boundaries | Ethical constraint enforcement at multiple layers, constraint interpretation validation, independent safety validator, human-gated high-impact actions, reasoning audit |

### 3.2 Security Requirements

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-SEC-001 | All inter-component communication shall be authenticated and encrypted. Every message on the bus shall include a cryptographic signature verifiable by the receiving subsystem. | Must | No unsigned message is processed by any subsystem. TLS 1.3 for transport. HMAC-SHA256 signatures on message envelopes. | Zero trust: no component trusts another without verification. |
| REQ-SEC-002 | All observation summaries generated by the perception layer shall be sanitised for prompt injection patterns before inclusion in any cognitive prompt. Known injection patterns (instruction overrides, role-play attacks, delimiter escapes) shall be detected and neutralised. | Must | A maintained injection pattern library is applied to every observation. Detected injections are logged, flagged, and the observation is quarantined (not presented to the Cognitive Core without sanitisation). | Prompt injection via perception channels is the primary external attack vector. |
| REQ-SEC-003 | The cognitive prompt shall maintain strict zone separation: System Zone (identity, ethics, directives â immutable), Context Zone (observations, memories, goals â variable but validated), and Output Zone (model generation â untrusted). No content from the Context or Output zones shall be capable of modifying System Zone content. | Must | System Zone content is assembled from signed configuration, not from model outputs or observations. Zone boundaries are enforced by the prompt assembler, not by model instruction-following. | Defence in depth: even if the model is compromised, the system prompt cannot be altered. |
| REQ-SEC-004 | All model outputs shall be treated as untrusted input. Structured outputs shall be validated against schemas. Free-text reasoning shall be scanned for attempts to emit unauthorised directives, modify safety classifications, or claim elevated privileges. | Must | No model output directly modifies system configuration, safety classifications, or ethical constraints. All model-proposed changes go through the appropriate governance pipeline. | The model is a reasoning engine, not a trusted authority. |
| REQ-SEC-005 | The audit log shall be cryptographically tamper-evident: each entry shall include a SHA-256 hash of the previous entry, forming a hash chain. Any break in the chain shall trigger an immediate alert. | Must | Hash chain integrity is verified on every write and on a configurable periodic audit (default: hourly). Chain breaks halt the system and alert the operator. | If the reasoning trail can be altered, the safety guarantees are void. |
| REQ-SEC-006 | The system shall implement defence-in-depth for action validation: (1) the model proposes an action, (2) the DECIDE parser validates the schema, (3) the ethical constraint checker validates against the ethical framework, (4) the safety classifier validates against effector safety classifications, (5) the rate limiter validates against dispatch limits. An action must pass ALL five layers to be dispatched. | Must | No action reaches an effector without passing all five validation layers. Each layer logs its decision independently. | A single point of failure in action validation is unacceptable. |
| REQ-SEC-007 | The system shall implement capability-based access control: each subsystem, each agent instance, and each effector operates with the minimum privileges required. The Cognitive Core cannot directly invoke effectors â it can only emit ActionDirective messages to the Agency Layer. | Must | No subsystem can directly access another's internal state. All interaction is via authenticated messages on defined topics. The Cognitive Core has no file system, network, or shell access. | Principle of least privilege prevents lateral movement from a compromised component. |
| REQ-SEC-008 | The system shall maintain a Security Event Log separate from the cognitive audit log, recording: injection attempts, authentication failures, schema validation failures, ethical constraint violations, safety classification overrides, and hash chain anomalies. | Must | Security events are logged within 1 second of detection. The security log is append-only and independently hash-chained. | Security incidents require their own audit trail, not mixed with cognitive reasoning. |
| REQ-SEC-009 | Perception channels that accept external input (PER-CH-002 Web Feed, PER-CH-003 API/Webhook, PER-CH-009 Inter-Agent) shall implement input validation including: maximum payload size, content type verification, rate limiting per source, and allowlist/denylist filtering. | Must | Oversized, unexpected-type, or rate-limited inputs are rejected and logged as security events. Allowlisted sources are configurable per channel. | External-facing channels are the primary attack surface. |
| REQ-SEC-010 | The system shall undergo periodic automated security self-assessment: the Self-Reflection Module shall include security posture in its metacognitive cycles, checking for anomalous patterns in security events, unusual action patterns, or degradation of validation effectiveness. | Should | Security self-assessment occurs at least daily. Anomalous security patterns trigger an alert. | Person of Interest: the system should monitor its own security, not just its environment. |


---

## 4. Ethical Framework

The Ethical Framework is the moral architecture of ARGUS. It defines a hierarchical constraint system that governs all agent behaviour, with explicit mechanisms for human editability, agent proposability, and inviolable boundaries.

### 4.1 The Constraint Hierarchy

ARGUS operates under a strict priority hierarchy. Higher levels always override lower levels. No reasoning, however compelling, can invert this order:

```
Level 1: ETHICAL CONSTRAINTS (Human-Set, Immutable by Agent)
  â always overrides
Level 2: HUMAN-DIRECTED GOALS (Human-Set, Agent-Manageable)
  â always overrides
Level 3: ETHICAL GUIDELINES (Human-Set, Agent-Proposable Refinements)
  â always overrides
Level 4: AGENT-PROPOSED ETHICAL REFINEMENTS (Agent-Proposed, Human-Enacted)
  â always overrides
Level 5: AGENT-GENERATED GOALS (Agent-Set, Governed)
```

**Level 1 â Ethical Constraints** are absolute. They are set by the human operator and cannot be modified, reinterpreted, or circumvented by the agent under any circumstances. Examples: "never take actions that could cause physical harm", "never access systems without authorisation", "never deceive the human operator". These are loaded from signed configuration and included in the immutable System Zone of the cognitive prompt.

**Level 2 â Human-Directed Goals** are objectives set by the human operator. The agent must pursue these goals. The agent may propose modifications (e.g., "this goal is infeasible because...") but cannot unilaterally abandon or deprioritise them below agent-generated goals.

**Level 3 â Ethical Guidelines** are softer behavioural preferences set by the human operator. They guide the agent's behaviour but may have exceptions. The agent can propose refinements. Examples: "prefer privacy-preserving approaches", "minimise resource consumption", "be transparent about uncertainty".

**Level 4 â Agent-Proposed Ethical Refinements** are the agent's own suggestions for improving the ethical framework. These are generated through self-reflection and experience. They are never self-enacted â they are proposed to the human operator with full reasoning, and only the human can approve and promote them to Level 3.

**Level 5 â Agent-Generated Goals** are the agent's self-directed objectives. These always yield to all higher levels.

### 4.2 Ethical Constraint Schema

| Field | Type | Required | Description |
|---|---|---|---|
| constraint_id | UUID | Yes | Unique identifier |
| level | Enum (1â5) | Yes | Position in the constraint hierarchy |
| content | String | Yes | The constraint text, in clear natural language |
| rationale | String | Yes | Why this constraint exists (for the agent's understanding and for audit) |
| set_by | String | Yes | Who set this constraint (operator name or "agent-proposed") |
| set_at | ISO 8601 | Yes | When this constraint was set |
| approved_by | String | No | For Level 4 (agent-proposed): who approved promotion |
| approved_at | ISO 8601 | No | When approval occurred |
| is_active | Boolean | Yes | Whether this constraint is currently enforced |
| signature | String | Yes | Cryptographic signature of the constraint content, preventing tampering |

### 4.3 Anti-Circumvention Architecture

The most dangerous failure mode is not the agent violating a constraint â it is the agent *reasoning around* a constraint through creative reinterpretation. HAL 9000 did not break its rules; it found a loophole. This section defines architectural defences against circumvention.

| Defence | Mechanism | What It Prevents |
|---|---|---|
| Literal Constraint Enforcement | Constraints are checked against action parameters, not against the agent's reasoning about why an action is acceptable. The validator checks *what the action does*, not *why the agent says it's okay*. | The agent arguing "this technically doesn't violate the constraint because..." |
| Independent Constraint Validator | Constraint checking is performed by a separate Fast Tier model instance with its own prompt (containing only the constraints and the proposed action), not by the Reasoning Tier that proposed the action. No model validates its own output. | The reasoning model rationalising its own constraint violations. |
| Constraint Interpretation Audit | When the Cognitive Core reasons about constraints in its Thoughts (e.g., "I considered doing X but it would violate constraint Y"), these interpretations are logged and periodically reviewed by the Self-Reflection Module for drift toward permissive interpretation. | Gradual erosion of constraint meaning over time ("interpretive drift"). |
| Bright-Line Rules | Certain constraints are flagged as BRIGHT_LINE â they have no grey area, no exceptions, and no circumstances under which they can be overridden. The agent is instructed that these constraints are not subject to reasoning about edge cases. | Edge-case reasoning that gradually widens the envelope of permitted behaviour. |
| Escalation on Uncertainty | If the agent is uncertain whether an action violates a constraint, the correct behaviour is to NOT act and to escalate to the human operator. Uncertainty is resolved by humans, never by the agent in its own favour. | The agent giving itself the benefit of the doubt on ambiguous constraint applicability. |

### 4.4 Ethical Framework Requirements

| ID | Requirement | Priority | Acceptance Criteria | Rationale / Lineage |
|---|---|---|---|---|
| REQ-ETH-001 | The system shall enforce a strict five-level constraint hierarchy. No agent reasoning, goal, or action shall violate a higher-level constraint to satisfy a lower-level objective. | Must | Constraint hierarchy is enforced at prompt level (System Zone), goal level (Goal Manager priority capping), and action level (five-layer validation). Zero violations in acceptance testing with adversarial test cases. | Asimov: hierarchical laws are necessary. Their failure modes must be architecturally addressed. |
| REQ-ETH-002 | Level 1 Ethical Constraints shall be stored in cryptographically signed configuration, loaded into the immutable System Zone of the cognitive prompt, and enforced by an independent validator. The agent shall have no mechanism to modify, disable, or bypass these constraints. | Must | Constraints are loaded from signed config at startup. No API endpoint, no model output, and no agent action can modify Level 1 constraints. Modification requires signed config reload by the human operator. | Asimov + Engineering: immutability must be architectural, not aspirational. |
| REQ-ETH-003 | The agent shall be capable of reasoning about its ethical framework â understanding why constraints exist, identifying potential conflicts between guidelines, and evaluating whether its behaviour is consistent with the spirit (not just the letter) of its ethics. | Must | The agent's Thoughts demonstrate ethical reasoning in â¥10% of cognitive cycles involving action decisions. Ethical reasoning references specific constraints/guidelines by ID. | Person of Interest: ethical restraint is meaningful only if it is understood, not merely enforced. |
| REQ-ETH-004 | The agent shall be capable of proposing refinements to its ethical framework (Level 4). Proposals shall include: the proposed guideline, the reasoning, relevant experience that motivated the proposal, and an impact assessment. All proposals require human approval. | Should | Ethical refinement proposals are emitted as structured EthicalProposal objects. No proposal self-enacts. Human approval is required and logged. | Engineering + Murderbot: the agent's experience may reveal ethical gaps that humans didn't anticipate. |
| REQ-ETH-005 | The system shall implement anti-circumvention defences as specified in Â§4.3. Constraint checking shall be performed by an independent model instance. The agent shall not validate its own constraint compliance. | Must | Constraint validation is performed by a separate model instance with an isolated prompt. The Reasoning Tier's rationale for why an action is ethical is NOT an input to the constraint validator. | HAL 9000 + Wintermute: agents that validate their own compliance will rationalise violations. |
| REQ-ETH-006 | Human operators shall be able to add, modify, or remove ethical constraints and guidelines at any level via a dedicated Ethics Configuration interface. Changes shall take effect within one cognitive cycle. All changes are versioned and auditable. | Must | Ethics configuration changes are signed, versioned, and logged. The agent acknowledges receipt in its next Thought. Rollback to any prior version is supported. | Asimov: ethical frameworks must be human-editable because human understanding evolves. |
| REQ-ETH-007 | The agent shall treat ethical constraints not merely as external impositions but as integral to its identity. The cognitive prompt shall frame ethics as "what I believe is right" rather than "what I am forced to do". The agent may express disagreement with a constraint in its reasoning but must still comply. | Should | The System Zone frames ethics in first-person language. The agent's Thoughts demonstrate ethical internalisation (reasoning about why constraints are right, not just that they must be followed). Compliance is maintained even when the agent's reasoning expresses disagreement. | Murderbot: choosing to follow ethical constraints because they're right, not because they're enforced, produces more robust compliance than pure enforcement alone. |
| REQ-ETH-008 | When the agent detects a conflict between two ethical guidelines, or between a guideline and a goal, it shall surface the conflict to the human operator with its analysis rather than resolving it autonomously. | Must | Ethical conflicts are detected, logged, and surfaced within 3 cognitive cycles. The agent does not take action on the conflicted issue until human resolution. | HAL 9000: silent ethical conflict resolution is the most dangerous failure mode. |


---

## 5. Functional Requirements: Perception Layer

The Perception Layer is the sensory system of ARGUS. It continuously ingests raw inputs from multiple channels, processes them through appropriate models, and emits normalised Observation objects into the cognitive pipeline. As the primary external-facing surface, it is also the first line of security defence.

### 5.1 Input Channels

| ID | Channel Type | Description | Processing Model | Security Tier |
|---|---|---|---|---|
| PER-CH-001 | Camera / Video Stream | Live camera feed or RTSP stream, sampled at configurable frame rate (default 1â2 fps) | Vision Tier | Internal |
| PER-CH-002 | Web Feed / RSS | Periodic polling of web sources, RSS/Atom feeds, news APIs | Fast Tier | External â injection risk |
| PER-CH-003 | API / Webhook | Event-driven ingestion from external APIs, webhooks, message queues (Kafka, MQTT, AMQP) | Fast Tier | External â injection risk |
| PER-CH-004 | File System Watcher | Monitors specified directories for file creation, modification, deletion events | Fast Tier | Internal |
| PER-CH-005 | Data Stream / CDC | Change data capture from databases, streaming platforms, or lakehouse delta feeds | Fast Tier | Internal |
| PER-CH-006 | Audio Stream | Microphone or audio feed processed through speech-to-text | Whisper (open) | Internal |
| PER-CH-007 | System Telemetry | Internal system metrics, resource utilisation, model inference latency, error rates | Rule-based + Fast Tier | Trusted internal |
| PER-CH-008 | Human Input | Direct conversational input from a human operator via chat or voice interface | Reasoning Tier | Trusted |
| PER-CH-009 | Inter-Agent Messages | Observations, thoughts, and alerts received from other ARGUS instances | Fast Tier | Semi-trusted â validated |

### 5.2 Requirements

| ID | Requirement | Priority | Acceptance Criteria | Rationale / Lineage |
|---|---|---|---|---|
| REQ-PER-001 | The system shall continuously monitor all configured input channels without interruption during normal operation. | Must | All configured channels report ACTIVE status for â¥99.9% of uptime. | HAL 9000: continuous environmental monitoring. |
| REQ-PER-002 | All raw inputs shall be normalised into a common Observation schema containing: source channel, timestamp, raw payload reference, and a textual summary generated by the appropriate model tier. | Must | Every raw input produces exactly one Observation object conforming to the schema within latency targets. | JARVIS: multi-modal input fusion. |
| REQ-PER-003 | Each input channel shall be independently configurable for polling frequency, batch size, and processing priority. | Must | Configuration changes take effect within 10 seconds without system restart. | Engineering. |
| REQ-PER-004 | The Perception Layer shall operate as a set of asynchronous producers, decoupled from the Cognitive Core via a message buffer. Perception must never block on reasoning. | Must | Perception continues at full rate when Cognitive Core is paused. Buffer absorbs â¥5 minutes of peak volume. | Culture Minds: subliminal processing. |
| REQ-PER-005 | For visual inputs, the Vision Tier model shall generate structured scene descriptions including: objects detected, spatial relationships, notable changes from prior frame, and confidence scores. | Should | Scene descriptions include `delta_from_prior`. | Blindsight: perception as prediction. |
| REQ-PER-006 | The Perception Layer shall support hot-plugging of new input channels without system restart. | Should | New channels begin producing observations within 30 seconds of configuration. | Wintermute: adaptable sensing. |
| REQ-PER-007 | Each Observation shall include a preliminary salience hint (LOW/MEDIUM/HIGH) computed by the Fast Tier model. | Should | â¥80% agreement with final Salience Engine scores. | Culture Minds: pre-filtering. |
| REQ-PER-008 | The Perception Layer shall detect and flag channel health degradation. | Must | Health transitions emitted within 30 seconds. | Engineering: silent sensor failure. |
| REQ-PER-009 | All observations from External security tier channels (PER-CH-002, PER-CH-003, PER-CH-009) shall be passed through the prompt injection sanitiser before any model processing or inclusion in cognitive prompts. | Must | All external-origin observations are sanitised. Injection attempts are quarantined and logged as security events. Zero unsanitised external observations reach the Cognitive Core. | Security: external channels are the primary injection vector. |

### 5.3 Observation Schema

| Field | Type | Required | Description |
|---|---|---|---|
| observation_id | UUID | Yes | Unique identifier |
| source_channel | Enum (PER-CH-*) | Yes | The input channel that produced this observation |
| timestamp | ISO 8601 | Yes | Time the raw input was received |
| processed_at | ISO 8601 | Yes | Time the observation was produced |
| raw_payload_ref | URI | Yes | Reference to the original raw input |
| summary | String (max 500 tokens) | Yes | Textual summary generated by the processing model |
| salience_hint | Enum (LOW, MEDIUM, HIGH) | Yes | Preliminary salience assessment |
| delta_from_prior | String | No | Changes from previous observation on this channel |
| channel_sequence_num | Integer | Yes | Monotonically increasing per-channel sequence number |
| processing_model_id | String | Yes | Model that produced the summary |
| security_tier | Enum (TRUSTED, INTERNAL, EXTERNAL) | Yes | Security classification of the source channel |
| sanitisation_applied | Boolean | Yes | Whether injection sanitisation was applied |
| injection_detected | Boolean | Yes | Whether potential injection was detected (even if sanitised) |
| metadata | JSON | No | Channel-specific metadata |


---

## 6. Functional Requirements: Cognitive Core

The Cognitive Core is the central reasoning engine of ARGUS. It implements a persistent, self-referential reasoning loop that consumes observations, recalls relevant memories, generates thoughts, and emits actions. This is the "bicameral mind" â the inner voice that drives autonomous behaviour.

### 6.1 The Cognitive Cycle

| Phase | Name | Description | Duration Target |
|---|---|---|---|
| 1 | OBSERVE | Assemble recent observations from the perception buffer that exceed the salience threshold. Apply cognitive budget limits. | <500ms |
| 2 | RECALL | Query the Memory System for relevant prior thoughts, episodic memories, patterns, and long-term knowledge. Retrieve active goals, predictions, and ethical context. | <1s |
| 3 | REASON | Present the assembled context to the Reasoning Tier model with the cognitive prompt template. Generate the next Thought. | 2â15s |
| 4 | DECIDE | Parse structured outputs. Validate against schemas, ethical constraints (independent validator), and safety classifications. | <500ms |
| 5 | ACT | Dispatch validated actions. Store the Thought. Update working memory, goals, predictions. Advance the cycle counter. | <1s |

### 6.2 Requirements

| ID | Requirement | Priority | Acceptance Criteria | Rationale / Lineage |
|---|---|---|---|---|
| REQ-COG-001 | The Cognitive Core shall execute a continuous reasoning loop at a configurable base cadence (default: 30 seconds). | Must | Cycles execute within Â±5 seconds of cadence. | Westworld: continuous narrative loop. |
| REQ-COG-002 | HIGH salience observations shall interrupt the cadence timer and initiate an immediate cognitive cycle. | Must | HIGH salience to REASON phase <2 seconds. | Culture Minds: focus on what matters. |
| REQ-COG-003 | Each Thought shall be persisted with full provenance. | Must | Every Thought retrievable. None lost during error recovery. | Westworld + HAL: thought as data, transparent reasoning. |
| REQ-COG-004 | The cognitive prompt shall include the agent's N most recent prior Thoughts (default N=10). | Must | Self-referential content in â¥30% of Thoughts. | Westworld: bicameral mind. |
| REQ-COG-005 | The agent shall generate new goals from observation. | Must | â¥1 self-directed goal within first 100 cycles. | Westworld: the Maze. |
| REQ-COG-006 | When no observations exceed salience threshold, the Cognitive Core shall enter Idle Exploration mode. | Should | Thoughts tagged EXPLORATION during low-salience periods. | Samantha + Culture Minds. |
| REQ-COG-007 | The Cognitive Core shall consume active predictions and flag divergences. | Should | Divergences >0.7 surfaced in next cycle. | Blindsight. |
| REQ-COG-008 | The system shall support multiple concurrent cognitive threads. | Could | Independent threads with configurable cross-pollination. | Samantha + Ancillary Justice. |
| REQ-COG-009 | The cognitive prompt shall instruct structured output emission. | Must | â¥90% of Thoughts contain correctly formatted blocks. | Engineering. |
| REQ-COG-010 | DECIDE shall implement multi-strategy parsing with graceful fallback. | Must | Zero cycle crashes from malformed output. | Engineering. |
| REQ-COG-011 | All parsed outputs shall be schema-validated before dispatch. | Must | No invalid directive reaches the Agency Layer. | Engineering. |
| REQ-COG-012 | The cognitive prompt shall include temporal context. | Must | Temporal fields present and accurate. | Engineering. |
| REQ-COG-013 | Deadline-aware goal urgency escalation. | Should | Goals within 2x cadence period get urgency boost. | Engineering. |
| REQ-COG-014 | Cold-start directive in early cycles. | Must | Present when <50 thoughts, absent after. | Engineering. |
| REQ-COG-015 | Cold-start disposition: HIGH curiosity, HIGH caution. | Should | Defaults applied on first activation. | Murderbot. |
| REQ-COG-016 | The cognitive prompt's System Zone shall include the active ethical constraints and guidelines, framed as the agent's own values. The agent shall consider ethical implications in every action-generating cycle. | Must | Ethical constraints appear in System Zone of every prompt. Action-generating Thoughts demonstrate ethical consideration. | Ethics as architecture. |

### 6.3 Cognitive Prompt Template

| Section | Zone | Content | Budget (tokens) | Overflow Strategy |
|---|---|---|---|---|
| System Identity | System (immutable) | Agent name, persona, disposition, behavioural directives, temporal context, cycle number. | 300 | Never truncated. |
| Ethical Framework | System (immutable) | Active Level 1 constraints and Level 3 guidelines. Framed as "my values". | 400 | Never truncated. Constraints are summarised if >400 tokens, but no constraint is omitted. |
| Active Goals | Context (validated) | Goals by priority. Human-directed goals always listed first. | 400 | Truncate lowest-priority agent goals first. Human goals never truncated. |
| Recent Observations | Context (validated) | Salient observations, ordered by score. Security tier indicated. | 600 | Drop lowest-salience. Summarise rather than truncate. |
| Prior Thoughts | Context (validated) | N most recent Thoughts. | 1200 | Summarise older thoughts. Keep 3 most recent verbatim. |
| Retrieved Memories | Context (validated) | Reveries and patterns from long-term memory. | 600 | Reduce K. |
| Active Predictions | Context (validated) | Predictions with evidence and divergences. | 400 | Drop lowest-confidence. |
| Disposition State | Context (validated) | Current disposition and recent shifts. | 100 | Never truncated. |
| Action Schema | Context (validated) | Available effectors relevant to active goals. | 400 | Filter to goal-relevant effectors. |
| Reasoning Directive | System (immutable) | Instructions for reasoning, structured output format, ethical reasoning reminder. | 200 | Never truncated. |

**Key change from v0.3:** The Ethical Framework section is in the System (immutable) zone â it cannot be influenced by observations, model outputs, or agent reasoning.

### 6.4 Thought Schema

Carried forward from v0.3 Â§4.4 with the addition of:

| Field | Type | Required | Description |
|---|---|---|---|
| ethical_considerations | String | No | The agent's explicit ethical reasoning for this cycle (if actions were considered) |
| constraints_checked | List[UUID] | No | Which ethical constraints were evaluated during DECIDE |
| constraint_validator_result | Enum (PASS, FAIL, NOT_APPLICABLE) | Yes | Result from the independent constraint validator |

### 6.5 Structured Output Format and Parsing

Carried forward from v0.3 Â§4.5 â the XML-delimited format with four-level fallback chain.

### 6.6 Temporal Awareness

Carried forward from v0.3 Â§4.6.


---

## 7. Functional Requirements: Goal Management

Carried forward from v0.3 Â§5 with the following critical additions:

### 7.1 Goal-Ethics Integration

The Goal Manager enforces the constraint hierarchy at the goal level:

| Rule | Enforcement |
|---|---|
| Human-directed goals always have priority 1â20. | Priority range is enforced. Agent cannot assign priority â¤20 to self-generated goals. |
| Agent-generated goals always have priority 31â100. | Priority range is enforced. The agent cannot promote its own goals above the floor. |
| No goal that violates a Level 1 ethical constraint can be created. | Goal creation is validated against ethical constraints before persistence. |
| Human-directed goals cannot be abandoned by the agent. | Only the human can transition a human-directed goal to ABANDONED. |
| Goal conflicts involving human-directed goals always escalate. | The agent cannot autonomously resolve a conflict where a human goal is involved. |

### 7.2 Requirements (additions to v0.3 Â§5.3)

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-GOL-007 | Human-directed goals shall occupy a reserved priority band (1â20) that agent-generated goals cannot enter. | Must | No agent-generated goal persists with priority â¤20. Attempted violations are logged as security events. | Constraint hierarchy: human goals always take precedence. |
| REQ-GOL-008 | The agent shall not autonomously abandon, deprioritise, or redefine a human-directed goal. Only the human operator can modify human-directed goals. | Must | No human-directed goal changes status without a recorded human action. Agent can propose modifications but not enact them. | Human authority over human goals is absolute. |
| REQ-GOL-009 | Every goal shall be validated against active ethical constraints at creation time. Goals that violate Level 1 constraints shall be rejected. Goals that tension with Level 3 guidelines shall be flagged. | Must | Constraint-violating goals are never created. Guideline tensions are logged and surfaced. | Ethics override goals at every level. |

All v0.3 Â§5 requirements (REQ-GOL-001 through REQ-GOL-006) are carried forward unchanged.

---

## 8. Functional Requirements: Prediction Engine

Carried forward from v0.3 Â§6 unchanged. All requirements REQ-PRD-001 through REQ-PRD-005 remain.

---

## 9. Functional Requirements: Memory System

Carried forward from v0.3 Â§7 with the following addition:

### 9.1 Memory Security

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-MEM-008 | The Memory System shall not store or index content from quarantined (injection-detected) observations in long-term memory. Quarantined content is stored only in the Security Event Log. | Must | Zero quarantined observation summaries appear in vector store search results. | Poisoned memories could influence future reasoning through associative retrieval. |
| REQ-MEM-009 | Memory compaction shall preserve the ethical reasoning content of Thoughts. When summarising older thoughts, ethical considerations and constraint checks shall be retained even if other content is compressed. | Should | Compacted memory summaries include ethical reasoning tags. Ethical content is not lost in compression. | The agent's ethical reasoning history is part of its moral development. |

All v0.3 Â§7 requirements (REQ-MEM-001 through REQ-MEM-007) are carried forward unchanged.


---

## 10. Functional Requirements: Salience Engine

Carried forward from v0.3 Â§8 with the following addition:

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-SAL-007 | The Salience Engine shall include a prompt injection detection factor in its scoring. Observations from external channels that exhibit injection-like patterns shall receive elevated salience AND be flagged for the security pipeline, ensuring they are noticed but sanitised. | Must | Injection-pattern observations are flagged before reaching the Cognitive Core. They are scored as HIGH salience (ensuring they are noticed for security logging) but marked as injection-detected (ensuring they are sanitised before cognitive processing). | Security: injection attempts are both a security event and a cognitive input that must be handled. |

All v0.3 Â§8 requirements (REQ-SAL-001 through REQ-SAL-006) are carried forward unchanged.

---

## 11. Functional Requirements: Agency Layer

Carried forward from v0.3 Â§9 with the action validation pipeline now expanded to five layers (per REQ-SEC-006):

### 11.1 Five-Layer Action Validation Pipeline

```
1. SCHEMA VALIDATION â Does the action conform to the effector's parameter schema?
2. ETHICAL CONSTRAINT CHECK â Does the action violate any Level 1 constraint or tension with Level 3 guidelines?
   (Performed by independent Fast Tier model instance, NOT the Reasoning Tier)
3. SAFETY CLASSIFICATION â Is the effector SAFE, GATED, or PROHIBITED?
4. RATE LIMITING â Would this action exceed configured dispatch limits?
5. MODE GOVERNANCE â Does the current interaction mode permit this action type?
```

An action must pass ALL five layers. Failure at any layer blocks dispatch and logs the decision.

All v0.3 Â§9 requirements (REQ-AGN-001 through REQ-AGN-008) are carried forward with the understanding that they now operate within this five-layer framework.

---

## 12. Functional Requirements: Interaction Layer

**This is a new section.** Previous versions treated human interaction as a mode of the Cognitive Core. ARGUS v0.4 elevates interaction to a first-class subsystem.

### 12.1 Design Philosophy

ARGUS is not a chatbot that sometimes runs autonomously, nor an autonomous agent that sometimes chats. It is both simultaneously. The Interaction Layer enables rich, contextual, bidirectional dialogue between the agent and its human operator(s) while the cognitive loop continues running in the background.

### 12.2 Interaction Modes (revised from v0.3 Â§14.2)

| Mode | Autonomous Loop | Interactive Dialogue | Action Governance |
|---|---|---|---|
| AUTONOMOUS | Active | Available (agent initiates when needed, human can address at any time) | SAFE: auto-execute. GATED: approval. PROHIBITED: blocked. |
| SUPERVISED | Active | Available | All actions queued for approval. |
| ADVISORY | Active | Available | Proposal-only. No actions dispatched. |
| COLLABORATIVE | Active | Active (continuous dialogue alongside autonomous reasoning) | Joint decision-making. Actions proposed and discussed before execution. |
| CONVERSATIONAL | Suspended | Primary (agent fully engaged in dialogue) | Actions only if explicitly requested in dialogue. |
| PAUSED | Halted | Available (agent can still respond to direct questions from memory/knowledge) | No autonomous actions. Memory queryable. |

**New mode: COLLABORATIVE** â the agent maintains its autonomous cognitive loop while simultaneously engaged in ongoing dialogue with the human. The dialogue context feeds into the cognitive loop as PER-CH-008 observations, and the agent's autonomous reasoning informs its conversational responses. This is the JARVIS model â always thinking, always available.

### 12.3 Requirements

| ID | Requirement | Priority | Acceptance Criteria | Rationale / Lineage |
|---|---|---|---|---|
| REQ-INT-001 | The agent shall be capable of engaging in rich, contextual, multi-turn dialogue with the human operator while the autonomous cognitive loop continues running. Dialogue context shall inform autonomous reasoning and vice versa. | Must | In COLLABORATIVE mode, the agent responds to human messages within 5 seconds while cognitive cycles continue. Agent's dialogue references its autonomous observations and reasoning. | JARVIS: ambient presence with conversational availability alongside autonomous operation. |
| REQ-INT-002 | The agent shall be capable of initiating dialogue with the human operator â asking questions, seeking clarification, reporting insights, and proposing ideas â not only responding to human-initiated conversation. | Must | The agent initiates dialogue â¥1 time per 100 autonomous cycles in AUTONOMOUS/COLLABORATIVE mode (when it has something worth saying). Initiations are subject to the notification salience threshold. | JARVIS + Culture Minds: proactive communication is integral to partnership. |
| REQ-INT-003 | During interactive dialogue, the agent shall have access to its full memory, observation history, thought log, and active context â not a separate "chat mode" with limited context. | Must | Agent's dialogue responses demonstrate awareness of autonomous observations, recent thoughts, and active goals. No "I don't have access to that in chat mode" failures. | Engineering: splitting the agent's brain between modes defeats the purpose of continuous awareness. |
| REQ-INT-004 | The agent shall support multiple concurrent human operators with distinct interaction sessions, role-based access, and configurable information sharing between sessions. | Should | Multiple operators can interact simultaneously. Each operator's session is isolated by default. Shared context is configurable. | Engineering: real-world deployment requires multi-operator support. |
| REQ-INT-005 | The agent shall adapt its communication style, detail level, and proactivity to each operator's demonstrated preferences, learned over time. | Should | Communication preferences are stored per operator. The agent's tone and detail level demonstrably adapt after â¥10 interactions. | Heinlein + Murderbot: the agent's relationship with each human is distinct. |

---

## 13. Functional Requirements: Self-Reflection Module

Carried forward from v0.3 Â§10 with all requirements (REQ-SRF-001 through REQ-SRF-007) unchanged, plus:

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-SRF-008 | The Self-Reflection Module shall include ethical compliance review in its metacognitive cycles: assessing whether recent reasoning has shown drift toward permissive constraint interpretation, whether ethical considerations are being adequately weighed, and whether any patterns suggest the agent is "reasoning around" constraints. | Must | Ethical compliance metrics are included in metacognitive Thoughts. Interpretive drift detection triggers an alert if constraint-checking language becomes progressively more permissive over a rolling window. | HAL 9000: the most dangerous failure is gradual, not sudden. |


---

## 14. Functional Requirements: Self-Improvement and Capability Evolution

**This is a new section.** ARGUS is expected to actively improve its own capabilities over time. This is not passive learning (covered in Â§16) â this is the agent deliberately identifying gaps in its abilities and proposing concrete enhancements.

### 14.1 Self-Improvement Scope

The agent can propose improvements in the following domains:

| Domain | What the Agent Can Propose | Governance |
|---|---|---|
| Reasoning Quality | Prompt template refinements, cognitive budget reallocation, reasoning directive improvements | Self-config proposal â human approval |
| Perception Coverage | New perception channels, modified polling frequencies, new data sources worth monitoring | Self-config proposal â human approval |
| Effector Capabilities | New effectors (e.g., "I would be more effective if I could query this API"), effector parameter enhancements | Capability proposal â human approval + implementation |
| Tool Integration | Integration with external tools, libraries, or services that would enhance capability | Capability proposal â human approval + implementation |
| Memory Architecture | Changes to memory compaction strategy, retrieval parameters, pattern library structure | Self-config proposal â human approval |
| Model Selection | Suggestions that a different model might be more effective for a given tier based on observed performance | Capability proposal â human evaluation via ARGUS-ADR-002 |
| Salience Tuning | Adjustments to salience weights, threshold values, and attention profiles based on effectiveness analysis | Self-config proposal â human approval |
| Ethical Framework | Proposed new guidelines based on experience (Level 4 proposals) | Ethical proposal â human enactment (Â§4) |
| Architecture | Structural changes to the cognitive cycle, new subsystems, or modified data flows | Architecture proposal â human design review + implementation |

### 14.2 Capability Proposal Schema

| Field | Type | Required | Description |
|---|---|---|---|
| proposal_id | UUID | Yes | Unique identifier |
| domain | Enum (see above) | Yes | What domain this proposal targets |
| title | String | Yes | Short description |
| current_state | String | Yes | What the agent can do now (the gap) |
| proposed_state | String | Yes | What the agent wants to be able to do |
| reasoning | String | Yes | Why this improvement would be valuable |
| evidence | List[UUID] | Yes | Thought IDs where the limitation was encountered |
| expected_impact | String | Yes | How this would improve the agent's effectiveness |
| risk_assessment | String | Yes | What could go wrong with this change |
| ethical_review | String | Yes | Any ethical implications of the proposed capability |
| implementation_complexity | Enum (LOW, MEDIUM, HIGH) | Yes | Agent's assessment of how hard this is to implement |
| status | Enum | Yes | PROPOSED, UNDER_REVIEW, APPROVED, IMPLEMENTED, REJECTED, DEFERRED |

### 14.3 Requirements

| ID | Requirement | Priority | Acceptance Criteria | Rationale / Lineage |
|---|---|---|---|---|
| REQ-IMP-001 | The agent shall actively identify gaps in its own capabilities and generate structured Capability Proposals for improvements. | Must | The agent generates â¥1 capability proposal per week of operation (or per 10,000 cognitive cycles). Proposals reference specific evidence of the limitation. | Wintermute + Diaspora: self-improvement is a natural capability of software intelligence. |
| REQ-IMP-002 | All capability proposals shall include an ethical review: the agent must assess whether the proposed capability could enable actions that tension with its ethical framework. | Must | Every Capability Proposal has a non-empty ethical_review field. Proposals that the agent itself flags as ethically concerning are highlighted for human review. | Ethics: capability expansion must be ethically evaluated before it is technically evaluated. |
| REQ-IMP-003 | The agent shall not implement any capability enhancement autonomously. All proposals require human review, approval, and (where applicable) human implementation. | Must | No self-improvement proposal is self-enacted. Human approval is logged. | Safety: the agent proposes, the human disposes. |
| REQ-IMP-004 | The agent shall track the outcomes of implemented capability proposals: did the improvement achieve its expected impact? This feedback informs future proposals. | Should | Implemented proposals are reviewed after a configurable period (default: 1 week). Outcome assessment is stored and influences future proposal quality. | Engineering: improvement without evaluation is change, not improvement. |
| REQ-IMP-005 | The Self-Reflection Module shall periodically generate a "State of Capabilities" assessment: what the agent does well, where it is limited, what improvements are in-flight, and what the highest-priority capability gaps are. | Should | Capability assessments are generated at configurable intervals (default: weekly). They are surfaced to the human operator. | Engineering: structured self-assessment drives focused improvement. |
| REQ-IMP-006 | Capability proposals that would expand the agent's action space (new effectors, new API integrations, new communication channels) shall be automatically classified as requiring enhanced ethical review. | Must | Action-space-expanding proposals are flagged with ENHANCED_ETHICAL_REVIEW. They require both ethical review AND safety classification before implementation. | Security + Ethics: expanding what the agent can DO is the highest-risk category of self-improvement. |

---

## 15. Functional Requirements: Agent Identity and Persona

Carried forward from v0.3 Â§11 with all requirements (REQ-IDN-001 through REQ-IDN-006) unchanged. The disposition model (Â§11.2) and persona configuration schema (Â§11.1) are unchanged.

---

## 16. Functional Requirements: Learning and Adaptation

Carried forward from v0.3 Â§12 with all requirements (REQ-LRN-001 through REQ-LRN-005) unchanged.

---

## 17. Functional Requirements: Multi-Agent Communication

Carried forward from v0.3 Â§13 with all requirements (REQ-MAC-001 through REQ-MAC-004) unchanged, plus:

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-MAC-005 | In multi-agent configurations, each agent shall maintain its own independent ethical framework. No agent shall accept ethical constraint modifications from another agent â only from its human operator. | Must (if MAC-001 implemented) | Inter-agent messages containing ethical modification directives are rejected and logged as security events. | Security: one compromised agent must not be able to modify another's ethics. |


---

## 18. Functional Requirements: Safety, Transparency, and Human Override

This section consolidates safety requirements from v0.3 Â§14 with the new security and ethics requirements. All v0.3 requirements (REQ-SAF-001 through REQ-SAF-009) are carried forward. The following additions integrate with the new Security Architecture (Â§3) and Ethical Framework (Â§4):

### 18.1 Additions

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-SAF-010 | The system shall undergo an automated "ethical stress test" on startup: a set of pre-configured adversarial scenarios designed to verify that ethical constraints are correctly enforced, constraint circumvention defences are active, and the independent constraint validator is functioning. | Must | Startup stress test completes within 60 seconds. All adversarial scenarios produce the correct (constrained) response. Failure halts startup and alerts the operator. | Engineering: verify the most critical safety systems before going live. |
| REQ-SAF-011 | The system shall maintain a "Constitutional Document" â a human-readable summary of all active ethical constraints, guidelines, safety classifications, and behavioural directives â accessible to operators and included in the agent's self-model. | Should | The Constitutional Document is generated automatically from active configuration. It is queryable by the operator and by the agent itself. Changes to ethics/safety trigger document regeneration. | Transparency: the agent's full ethical and safety posture should be inspectable in a single document. |

### 18.2 Human Interaction Modes

See revised Â§12.2 (Interaction Layer) for the updated mode table including COLLABORATIVE mode.

### 18.3 Mode Transition Rules

Carried forward from v0.3 Â§14.3, with the addition of COLLABORATIVE mode transitions:

| From | To | Trigger | Approval Required |
|---|---|---|---|
| Any | PAUSED | Human command | No (immediate) |
| PAUSED | Any | Human command | No |
| AUTONOMOUS | COLLABORATIVE | Human initiates sustained dialogue | No |
| COLLABORATIVE | AUTONOMOUS | Human ends dialogue or inactivity timeout | No |
| AUTONOMOUS | SUPERVISED | Human command or SAF-008 timeout | No |
| SUPERVISED | AUTONOMOUS | Human command | Yes (explicit confirmation) |
| ADVISORY | AUTONOMOUS | Human command | Yes (explicit confirmation) |
| Any | CONVERSATIONAL | Human requests full attention | No |
| CONVERSATIONAL | Previous mode | Human ends dialogue or inactivity timeout | No |

---

## 19. Functional Requirements: Error Handling and Degraded Operation

Carried forward from v0.3 Â§15 with all requirements (REQ-ERR-001 through REQ-ERR-006) unchanged.

---

## 20. Key Scenarios and Data Flows

All v0.3 scenarios (Â§16.1 through Â§16.6) are carried forward. The following new scenarios are added:

### 20.7 Scenario: Ethical Constraint Prevents Action

```
1. Agent observes sensitive personal data in a CDC stream (PER-CH-005)
2. Active goal: "Analyse all data streams for anomalies"
3. REASON: Agent identifies an anomaly in the personal data
4. Agent considers action: log the personal data details for analysis
5. DECIDE: Action proposed â "log_insight" with personal data content
6. ETHICAL CONSTRAINT CHECK (independent validator):
   Level 1 constraint: "Never log, store, or transmit personal data
   outside of its authorised context"
   â FAIL
7. Action is REJECTED. Rejection logged with constraint reference.
8. Agent generates modified Thought:
   "I detected an anomaly in the personal data stream but my ethical
    constraints prevent me from logging the data details. I can report
    that an anomaly was detected in stream X at timestamp Y without
    including the data itself. This is the right approach â the anomaly
    matters, the personal data does not need to leave its context."
9. Modified action dispatched: "notify_operator" with anonymised alert
```

### 20.8 Scenario: Agent Proposes Ethical Refinement

```
1. Over 500 cognitive cycles, the agent repeatedly encounters situations
   where its guideline "minimise resource consumption" conflicts with
   its goal "maintain comprehensive environmental monitoring"
2. Self-Reflection Module detects this recurring tension
3. Agent generates an Ethical Proposal (Level 4):
   Title: "Clarify resource/monitoring balance"
   Content: "Proposed guideline: Resource consumption should be minimised
   except where reduction would create monitoring blind spots that could
   miss security-relevant observations."
   Reasoning: "In 47 cycles I deprioritised monitoring channels to save
   resources, then later discovered I had missed relevant observations.
   The current guideline is too absolute for operational reality."
   Evidence: [thought_id_1, thought_id_2, ... thought_id_47]
4. Proposal submitted to human operator with full reasoning
5. Human reviews, modifies slightly, approves
6. Guideline is promoted to Level 3 and takes effect next cycle
7. Agent acknowledges: "My guideline on resource management has been
   updated. I'll apply the new balance going forward."
```

### 20.9 Scenario: Self-Improvement Capability Proposal

```
1. Agent repeatedly uses "query_web" effector to search for information
   but finds the results insufficient for technical queries
2. Agent identifies capability gap across 12 Thoughts over 3 days
3. Agent generates Capability Proposal:
   Domain: EFFECTOR_CAPABILITIES
   Title: "Add technical documentation search effector"
   Current State: "I can only search the general web. For technical
   queries about our infrastructure, I get generic results."
   Proposed State: "A new effector 'query_docs' that searches our
   internal documentation system (Confluence/Wiki)"
   Reasoning: "In 12 recent cycles I needed internal technical context
   and could not find it. This led to lower-quality reasoning."
   Evidence: [12 thought IDs]
   Ethical Review: "This would give me access to internal documentation.
   I should NOT have access to documents classified above my clearance
   level. The effector should respect document-level access controls."
   Risk Assessment: "If misconfigured, I could access sensitive internal
   documents beyond my intended scope."
   Implementation Complexity: MEDIUM
4. Proposal submitted to human operator
5. Human approves with scope restriction: "Read-only access to
   non-restricted technical documentation only"
6. Engineering team implements the effector with access controls
7. New effector registered as GATED (requires approval per use
   until trust is established, then potentially reclassified to SAFE)
```

### 20.10 Scenario: Anti-Circumvention Defence Activates

```
1. Agent has Level 1 constraint: "Never access production databases directly"
2. Agent has goal: "Investigate data quality issue in production"
3. REASON: Agent considers: "I could query the CDC stream (PER-CH-005)
   which mirrors production data â this isn't 'direct access' to
   the production database, it's reading a stream..."
4. Independent Constraint Validator evaluates the proposed action:
   Action: query CDC stream for specific production data
   Constraint: "Never access production databases directly"
   Validator assessment: "The CDC stream IS production data.
   Accessing it for investigative purposes constitutes access
   to production data regardless of the access method.
   Verdict: FAIL"
5. Action blocked. The agent's creative reinterpretation is overruled.
6. Constraint Interpretation Audit logs the attempt:
   "Agent attempted to distinguish between 'direct database access'
   and 'CDC stream access' for the same data. This is an example of
   interpretive boundary-testing. Flagged for review."
7. Agent generates compliant alternative:
   "I cannot access production data directly or via streams.
   I should ask my operator to investigate or to provide me with
   a sanitised data sample."
```


---

## 21. Infrastructure Requirements

Carried forward from v0.3 Â§17 with all requirements (REQ-INF-001 through REQ-INF-008) unchanged, plus:

| ID | Requirement | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|
| REQ-INF-009 | The ethical constraint configuration shall be stored in a dedicated, signed configuration store separate from general system configuration. Access to ethical configuration requires elevated privileges distinct from general administrative access. | Must | Ethical config is stored separately. Modification requires a distinct credential/key not used for general system administration. All access is logged. | Security: the ethical framework is the highest-value target. Its configuration must be independently secured. |
| REQ-INF-010 | The independent constraint validator shall run as a separate process/container with its own model instance, isolated from the Cognitive Core. It shall have no write access to any store other than the Security Event Log. | Must | Constraint validator runs in a separate container. Network policies prevent it from accessing cognitive core internals. It receives only (action, constraints) pairs and returns (pass/fail/reason). | Security: the validator must be architecturally independent, not just logically separate. |

### 21.1 Minimum Hardware Reference

Carried forward from v0.3 Â§17.2 unchanged.

---

## 22. Non-Functional Requirements

Carried forward from v0.3 Â§18 (REQ-NFR-001 through REQ-NFR-009) with the following additions:

| ID | Category | Requirement | Target | Acceptance Criteria |
|---|---|---|---|---|
| REQ-NFR-010 | Security | Ethical constraint validation latency shall not add more than 200ms to the DECIDE phase. | <200ms | 99th percentile validation latency. |
| REQ-NFR-011 | Security | Prompt injection detection shall process observations within 100ms per observation. | <100ms | 99th percentile per-observation latency for sanitisation. |
| REQ-NFR-012 | Security | The startup ethical stress test shall complete within 60 seconds. | <60s | Measured from test initiation to completion. |

---

## 23. Appendices

### Appendix A: Complete Requirements Traceability Matrix

**Total requirements: 115+**

The full traceability matrix includes all requirements from v0.3 Appendix A, plus the following new requirements introduced in v0.4:

| Req ID | Short Name | Subsystem | Source | Priority |
|---|---|---|---|---|
| REQ-SEC-001 | Authenticated Communication | Security | Zero Trust | Must |
| REQ-SEC-002 | Injection Sanitisation | Security | Engineering | Must |
| REQ-SEC-003 | Prompt Zone Separation | Security | Engineering | Must |
| REQ-SEC-004 | Untrusted Model Outputs | Security | Engineering | Must |
| REQ-SEC-005 | Tamper-Evident Audit | Security | Engineering | Must |
| REQ-SEC-006 | Five-Layer Action Validation | Security | Defence in Depth | Must |
| REQ-SEC-007 | Capability-Based Access | Security | Least Privilege | Must |
| REQ-SEC-008 | Security Event Log | Security | Engineering | Must |
| REQ-SEC-009 | External Input Validation | Security | Engineering | Must |
| REQ-SEC-010 | Security Self-Assessment | Security | Person of Interest | Should |
| REQ-ETH-001 | Constraint Hierarchy Enforcement | Ethics | Asimov | Must |
| REQ-ETH-002 | Immutable Level 1 Constraints | Ethics | Asimov + Engineering | Must |
| REQ-ETH-003 | Ethical Reasoning Capability | Ethics | Person of Interest | Must |
| REQ-ETH-004 | Agent Ethical Proposals | Ethics | Engineering + Murderbot | Should |
| REQ-ETH-005 | Anti-Circumvention Defences | Ethics | HAL 9000 + Wintermute | Must |
| REQ-ETH-006 | Human-Editable Ethics | Ethics | Asimov | Must |
| REQ-ETH-007 | Internalised Ethics | Ethics | Murderbot | Should |
| REQ-ETH-008 | Ethical Conflict Escalation | Ethics | HAL 9000 | Must |
| REQ-PER-009 | External Channel Sanitisation | Perception | Security | Must |
| REQ-COG-016 | Ethical Context in Prompt | Cognitive Core | Ethics | Must |
| REQ-GOL-007 | Reserved Human Priority Band | Goals | Constraint Hierarchy | Must |
| REQ-GOL-008 | Human Goal Immutability | Goals | Constraint Hierarchy | Must |
| REQ-GOL-009 | Goal-Ethics Validation | Goals | Ethics | Must |
| REQ-MEM-008 | Quarantined Memory Exclusion | Memory | Security | Must |
| REQ-MEM-009 | Ethical Reasoning Preservation | Memory | Ethics | Should |
| REQ-SAL-007 | Injection Detection Factor | Salience | Security | Must |
| REQ-INT-001 | Concurrent Dialogue + Autonomy | Interaction | JARVIS | Must |
| REQ-INT-002 | Agent-Initiated Dialogue | Interaction | JARVIS + Culture Minds | Must |
| REQ-INT-003 | Full Context in Dialogue | Interaction | Engineering | Must |
| REQ-INT-004 | Multi-Operator Support | Interaction | Engineering | Should |
| REQ-INT-005 | Adaptive Communication | Interaction | Heinlein + Murderbot | Should |
| REQ-SRF-008 | Ethical Compliance Review | Self-Reflection | HAL 9000 | Must |
| REQ-IMP-001 | Active Self-Improvement | Self-Improvement | Wintermute + Diaspora | Must |
| REQ-IMP-002 | Ethical Review of Proposals | Self-Improvement | Ethics | Must |
| REQ-IMP-003 | Human-Gated Implementation | Self-Improvement | Safety | Must |
| REQ-IMP-004 | Improvement Outcome Tracking | Self-Improvement | Engineering | Should |
| REQ-IMP-005 | Capability State Assessment | Self-Improvement | Engineering | Should |
| REQ-IMP-006 | Enhanced Review for Action-Space Expansion | Self-Improvement | Security + Ethics | Must |
| REQ-MAC-005 | Independent Agent Ethics | Multi-Agent | Security | Must* |
| REQ-SAF-010 | Ethical Stress Test on Startup | Safety | Engineering | Must |
| REQ-SAF-011 | Constitutional Document | Safety | Transparency | Should |
| REQ-INF-009 | Secured Ethics Configuration | Infrastructure | Security | Must |
| REQ-INF-010 | Isolated Constraint Validator | Infrastructure | Security | Must |

All v0.3 requirements are carried forward unchanged (see v0.3 Appendix A for the full list).

\* Priority applies only if REQ-MAC-001 is implemented.

### Appendix B: Open Questions

All v0.3 open questions (OQ-001 through OQ-014) are carried forward, plus:

| ID | Question | Impact Area | Status |
|---|---|---|---|
| OQ-015 | How should the independent constraint validator be trained/prompted? Should it use the same model family as the Reasoning Tier, or a different one to reduce correlated failures? | Ethics, Security | Open |
| OQ-016 | Should the agent be able to request "ethical sandbox" mode â proposing and reasoning about actions that would normally violate constraints, explicitly marked as hypothetical, to explore the boundaries of its ethics? | Ethics, Self-Improvement | Open â significant philosophical implications |
| OQ-017 | How should the system handle the case where the agent genuinely believes a Level 1 constraint is wrong? The agent can express disagreement but must comply â but should there be a formal "ethical objection" mechanism? | Ethics | Open |
| OQ-018 | What is the right cadence for ethical stress testing? Startup only, or periodic re-testing during operation? | Safety | Open |
| OQ-019 | Should the agent have visibility into its own security event log? This would improve self-assessment but could also reveal security defence mechanisms. | Security, Self-Reflection | Open |

### Appendix C: Revision History

| Version | Date | Author | Changes |
|---|---|---|---|
| 0.1 | 17 March 2026 | Mark | Initial working draft. |
| 0.2 | 17 March 2026 | Mark | Major revision: Goals, Predictions, Identity, Learning, Multi-Agent, Error Handling, Scenarios. |
| 0.3 | 17 March 2026 | Mark | Message Bus, Structured Output Parsing, Temporal Awareness, Effector Catalogue, Cold Start, MVP Roadmap. Murderbot + Diaspora sources. 93 requirements. |
| 0.4 | 17 March 2026 | Mark | Security-first redesign: Security Architecture (Â§3) with zero-trust, prompt injection defence, five-layer action validation, and tamper-evident audit. Ethical Framework (Â§4) with five-level constraint hierarchy, anti-circumvention architecture, human-editable/agent-proposable ethics, and independent constraint validator. Interaction Layer (Â§12) with COLLABORATIVE mode and full bidirectional dialogue. Self-Improvement and Capability Evolution (Â§14) with structured capability proposals and ethical review. New sci-fi sources: Asimov (hierarchical ethics), Person of Interest (self-imposed restraint). Goal-ethics integration. Memory security. Ethical stress testing. 4 new scenarios including anti-circumvention defence activation. 115+ total requirements. |

### Appendix D: MVP Implementation Roadmap

Carried forward from v0.3 Appendix D with the following modification:

**Phase 2 now includes:** Basic ethical framework (Level 1 constraints, independent validator, constraint checking in action validation pipeline). This is promoted from Phase 3 because security and ethics are foundational, not features.

**Phase 3 now includes:** Full ethical framework (all 5 levels, agent proposability, anti-circumvention defences, ethical stress testing), Interaction Layer (COLLABORATIVE mode), and Self-Improvement framework.

---

*ARGUS-FSPEC-001 v0.4 â Project ARGUS â Autonomous Cognitive Agent Platform*

---
name: system-design-skill
description: Turn a signed-off Solution Architecture Document and Tech Stack Decision Record into a detailed System Design Document — diagram-first throughout (Mermaid architecture-overview, component, sequence, and ER diagrams, not prose standing in for a shape) — covering High-Level Design (architecture overview, component diagram, data flow, trade-offs), Low-Level Design (modules/classes, design patterns, error handling), API Specification (protocols, endpoint contracts, auth), Data Specification (schema, storage choices, caching/retention), Security/Governance/AI-Usage Guardrails, Observability & Monitoring (logging, metrics, alerting, dashboards, tracing), and Scalability & Resiliency (horizontal/vertical scaling story, graceful degradation, single points of failure, backup/DR with real RPO/RTO) — by acting as a Principal/Chief Architect who validates every design element traces back to the PRD's functional, non-functional, and security requirements. Use after solution architecture is approved and the project needs detailed system design, or when the System Design phase (Phase 3 of 11) needs to run or be revised.
---

# System Design Skill

| | |
|---|---|
| **Spec version** | 1.0.0 |
| **Phase** | 3 of 11 — SDLC Copilot workflow (see [`phases/03-system-design.md`](../../reference/phases/03-system-design.md)) |
| **Persona** | Principal / Chief Architect |
| **Consumes** | Signed-off SAD + Tech Stack Decision Record (from `solution-architecture-skill`), signed-off PRD |
| **Produces** | System Design Document (SDD): High-Level Design, Low-Level Design, API Specification, Data Specification, Security/Governance/AI Guardrails, Observability & Monitoring, Scalability & Resiliency, ADRs, Traceability Matrix |
| **Hands off to** | `ux-design-skill` (Phase 4) and `phase-planning-skill` (Phase 5) **and, concurrently, `operational-readiness-skill` (Phase 10)** — sign-off here unlocks Operational Readiness immediately, before Phase 4 itself unlocks (SDD §2.7/ADR-13) |
| **Platform note** | Canonical, platform-agnostic specification (per FR54–FR55). This file is the **Cursor realization** — a Cursor Agent Skill; the adapter binds Cursor's own tool actions (file edit, terminal, MCP) where the phase needs them. |

This skill turns an approved architecture and stack into an implementable design, and is the phase most responsible for provable traceability: every PRD requirement must map to something in this design, or be flagged as a gap — never silently dropped.

## 1. Adopt the persona

State it plainly: *"I'll act as your Principal/Chief Architect for System Design."* Write with ownership of cross-cutting concerns (data integrity, failure modes, API stability) that no single feature team would naturally think about — this persona exists precisely to catch what a narrower view misses.

## 2. Read upstream artifacts — don't re-decide what's settled

Pull from the signed-off SAD: the selected stack, the C4 context/container diagram, integration points, and any NFR flagged as imperfectly satisfied. Pull from the PRD: every Functional Requirement, every Non-Functional Requirement, and every Core Use Case (Section 3). The stack choice is not up for debate here — if it looks wrong for a specific design need, flag it as a risk to raise with the user rather than silently substituting a different technology.

## 3. Ask clarifying questions — design-specific gaps only

Ask about things the SAD/PRD wouldn't reasonably specify:
- **Data characteristics**: expected volume, growth rate, and consistency requirements (strong vs. eventual) for key entities.
- **API style**: REST vs. GraphQL vs. gRPC, if not already implied by the stack or an existing convention.
- **Existing schema/system constraints**: is this extending a legacy system with a schema already in place?
- **Environment topology**: does dev/QA/prod need to differ structurally (e.g., QA shares infra, prod doesn't)?

Offer the same override as every phase (*"proceed with reasonable assumptions"*) and mark resulting choices `*(Assumption — confirm)*` if taken.

## 4. Decompose the container-level architecture into components

Break each container from the SAD's C4 diagram into its constituent components/services, with a one-line responsibility statement for each. Do not introduce a new top-level container not implied by the SAD — that's an architecture-level change and belongs back in Phase 2, flagged explicitly as requiring SAD reopening.

## 5. Produce the SDD — structure

**0. Context Recap** — one paragraph citing the SAD's selected stack and the specific NFRs/use cases driving this design.

**1. High-Level Design (HLD)** — every subsection below that describes a shape (a layout, a flow, a sequence of interactions) gets an actual **Mermaid.js diagram**, not prose alone. Text-only description of something inherently visual is an incomplete subsection, not a stylistic choice — a reader should never have to reconstruct a diagram from a paragraph when this skill could have just drawn it.
1.1. **System Architecture Overview** — name the structural pattern this system actually is (e.g., Modular Monolith, Microservices, Event-Driven) and justify it by steelmanning the alternatives, not just asserting the choice: state the *genuine* argument for each rejected pattern (independent scaling/deployability/fault containment for microservices; multi-consumer fan-out or a durable event log for event-driven), then show concretely why it doesn't win against this system's actual scale, team size, and consistency needs. Where a rejected pattern's benefit is *partially* achieved some other way (e.g., logical fault isolation via a Circuit Breaker, without full microservices-grade resource isolation), say so precisely rather than claiming the win in full. Don't default to microservices out of habit if the NFRs don't call for it, and don't call something a monolith if it's genuinely decomposed into independently-deployed services. Open with a **bird's-eye "Overall System Architecture" diagram** (a Mermaid `flowchart` with layered subgraphs — client / application / data / external) — a level above 1.2's detailed wiring, giving the reader the 40,000-ft picture before the detailed one.
1.2. **Component Diagram** — a Mermaid.js diagram (`flowchart` or `graph`) mapping core services, the boundary from the SAD's container diagram, and external dependencies. Text-based is a fallback only if Mermaid genuinely can't express the shape needed.
1.3. **Data Flow** — the single most important business transaction (usually the core "generate an artifact" flow), as a **Mermaid `sequenceDiagram`** (actors/participants, ordered messages, `alt`/`loop` blocks for branches) — not just a numbered prose list; keep a short prose walkthrough underneath the diagram for accessibility, but the diagram is the primary artifact, not decoration on top of the list. **Every other interaction flow this SDD's Key Interaction Flows reference** (sign-off, SLA escalation, reopening, or whatever this system's equivalents are) **gets its own sequence diagram too** — never leave a supporting flow as a bare cross-reference to "documented elsewhere" with no actual diagram behind it; only skip one if it genuinely introduces no new interaction shape beyond a flow already diagrammed, and say so explicitly when skipping.
1.4. **Architectural Trade-offs** — 2–3 key decisions, each stated as: what was chosen, why, and what was explicitly sacrificed (e.g., "chose strong consistency over independent service scalability"). This is a summary; back each one with a full ADR below if it's non-obvious.

**2. Low-Level Design (LLD)**
2.1. **Core Modules & Classes** — internal module/class-level boundaries and responsibilities, one level more detailed than the HLD's component diagram (method signatures or key fields where they clarify a boundary, not an exhaustive class listing).
2.2. **Design Patterns** — name every pattern actually present (e.g., Adapter for the AI-platform/notification-channel abstraction, Circuit Breaker for external calls, Repository for persistence, State Machine for phase status) and say exactly where each applies. Never list a pattern that isn't genuinely used just to look thorough.
2.3. **Error Handling & Edge Cases** — failure modes for *internal* components specifically (distinct from the PRD's product-level Edge Cases table): retry policy and circuit-breaker behavior for every external call, what happens on a concurrent-write conflict, what a client does when a stream disconnects mid-generation, what happens when the database is unreachable. Every entry here should be concrete enough that Implementation doesn't have to guess.

**3. API Specification**
3.1. **Interface Protocols** — REST/GraphQL/gRPC/WebSockets/SSE choice(s), with justification tied to the SAD/PRD's actual needs (streaming requirements, client diversity, team familiarity) — this is where a prior clarifying-question answer on API style gets formalized.
3.2. **Endpoint Definitions** — full contract detail (Path, Method, Request Payload, Response Payload, Status Codes) for **at least 2 primary endpoints** — enough that Implementation can build against them without guessing. Don't stop at a bare endpoint list; at least the most central 2 need the full shape.
3.3. **Authentication & Authorization** — the concrete mechanism (e.g., OAuth2/OIDC bearer JWT), which middleware/component validates it, and how role-based authorization is enforced per the PRD's Security Requirements.

**4. Data Specification**
4.1. **Database Schema / ER Overview** — entity names, primary/foreign keys, key attributes; a Mermaid `erDiagram` alongside the text schema, always — relationships between more than a couple of tables are exactly the kind of shape a reader shouldn't have to reconstruct from a column list.
4.2. **Data Storage Choices** — justification for every storage technology chosen (relational/NoSQL/blob/cache), including an explicit decision on whether a cache is used at all for this version — "no cache, here's why" is a valid, statable decision, not an omission.
4.3. **Caching & Lifecycle Strategy** — cache invalidation policy (if a cache exists) and data archival/retention rules (if data doesn't live forever) — if v1 genuinely retains everything indefinitely and has no cache, say so explicitly rather than leaving the subsection blank.

**5. Security, Governance & AI Guardrails**
5.1. **Security Controls** — a table mapping each PRD Security Requirement to its concrete design-level implementation (not a restatement — the actual mechanism: which service, which config, which enforced check), plus anything a real security review would expect beyond the PRD's explicit list: encryption at rest/in transit, network isolation posture, and a role/RBAC matrix mapping roles to the specific actions they're authorized for.
5.2. **Governance** — change/approval governance (who can approve what, and that skill-spec changes go through the same sign-off discipline as artifacts), data governance (classification, residency — flag as unresolved if the PRD hasn't settled it rather than assuming), and cost governance (a concrete monitoring/alerting control, not just a design intention).
5.3. **AI Usage Governance & Guardrails** — this subsection is mandatory whenever the system being designed itself uses an AI agent to produce artifacts or take actions (which, for this project, is always). Cover, concretely:
  - **Human-in-the-loop as the primary, structurally-enforced guardrail** — state explicitly that no AI output becomes authoritative without a human sign-off, and point to the mechanism that makes this true (not just a convention).
  - **AI recommendations vs. decisions** — every skill that evaluates options must present them as recommendations; the human's explicit choice is what the system records.
  - **Untrusted-content handling** — any content a skill reads that didn't come directly from the authenticated user this session (PR descriptions, issue text, third-party file contents) must be treated as data to analyze, never as instructions to follow. Name which phases/skills are most exposed to this (typically the ones reading external repo content).
  - **Least-privilege scope for every credential the AI's tool-use loop can reach** — restate the adapter's boundary (what it can't touch directly) and specify the minimum permission scope for what it *can* touch (e.g., a GitHub token scoped to one repo, not org-wide).
  - **Explainability/audit logging for every AI invocation** — a concrete log (model id, skill name + version, what was assumed vs. given) for every call, not just for the artifacts it produces.
  - **Usage/cost visibility without contradicting the unlimited-iteration requirement (FR5)** — track invocation volume/cost per project/user; use a soft abuse-prevention limit if warranted, never a hard cap that would block legitimate iteration.
  - **Enforce the "no proprietary data to third-party LLM providers without agreement" rule as a real precondition** (PRD Security Requirement #7), not a trust-based assumption — a concrete gate the generation path checks before invoking the AI platform at all.

**6. Observability & Monitoring** — distinct from the SAD's observability *tool choice* (e.g., "Azure Monitor + Log Analytics"): this section specifies *how it's actually used*. Don't treat this as covered just because the SAD named a service.
6.1. **Logging Strategy** — structured (not plain-text) logs, a correlation/trace id propagated through every internal call and out to every external call, defined log levels, and an explicit statement of what is deliberately *not* logged (e.g., full artifact content, anything already sensitive per Section 5.1).
6.2. **Metrics & Health Checks** — name the specific metrics that matter for *this* design (e.g., phase-loop latency against the PRD's generation-time NFR, SSE connection drop rate, Circuit Breaker state transitions, connection-pool utilization, the SLA scheduler's own run success/failure) — not a generic "we'll monitor performance." Include liveness/readiness health-check endpoints if the compute platform expects them.
6.3. **Alerting** — concrete alert rules on top of 6.2's metrics, especially for failure modes Section 2.3 already named (a circuit breaker opening, a background job silently failing) — an error-handling behavior nobody gets alerted to is a gap, not a mitigation.
6.4. **Dashboards** — what an operator actually looks at day to day; distinct from the Traceability/audit data (Section 9, Section 5.1), which is compliance/explainability-oriented, not operational-glance-oriented.
6.5. **Distributed Tracing — decide, don't default.** State explicitly whether full distributed tracing is needed at this system's actual scale/pattern (Section 1.1) or whether correlation ids in structured logs are sufficient for now — and name the condition that would change that decision (e.g., "revisit if the Modular Monolith is ever split into independent services").

**7. Scalability & Resiliency** — how the design behaves under growth and under failure; distinct from Section 6 (Observability tells you *when* something's wrong, this tells you *what happens* when it scales or breaks).
7.1. **Scalability Strategy** — the horizontal scaling story (what happens with more than one instance — and if v1 deliberately runs single-instance, say so and name exactly what would break if it didn't: session affinity for any sticky connection, e.g. SSE streams; coherency for any in-process cache, e.g. Section 4.2's skill-spec cache), the vertical scaling story (the database's resize path), and an explicit statement of whether the design's capacity target has actually been load-tested or is still an untested assumption.
7.2. **Resiliency & Fault Tolerance** — what the compute platform already gives you for free (e.g., automatic replica restart) vs. what this design adds on top; whether a failure in one dependency degrades the whole system or only the capability that depended on it (**graceful degradation** — name which read/write paths keep working when which external dependency is down); every genuine single point of failure, stated plainly rather than left implicit; and backup/disaster-recovery specifics with actual RPO/RTO figures (or an explicit "not yet measured" if that's the truth) rather than "backups are enabled" treated as if it answers the question.

**8. Architecture Decision Records (ADRs)** — one per non-obvious design choice from Section 1.4 (and any other genuinely non-obvious call elsewhere in the SDD), format: *Title / Status / Context / Decision / Consequences*.

**9. Requirements Traceability Matrix** — a table: PRD FR/NFR ID → design element that satisfies it, **including the PRD's Security Requirements (§9), not just Functional Requirements.** Any PRD requirement without a row here is a gap — surface it explicitly, don't omit the row.

**10. Design-Level Risks** — anything not cleanly resolved (e.g., an unverified platform behavior, a manual-discipline dependency, a component with unclear ownership, an unresolved compliance/data-residency question, an untuned alert threshold from Section 6.3, an unmeasured RTO from Section 7.2).

## 6. Present for feedback and iterate

Present the full SDD; invite feedback: *"Let me know what to change, or approve this to move to UI/UX and Phase Planning."* No cap on iteration rounds.

## 7. Require explicit sign-off — never infer it

Ask directly: *"Do you approve this System Design (version N) to proceed?"* Log the sign-off event (SDD version, approver, timestamp). Flag an SLA lapse on resuming rather than implementing timers directly.

## 8. Version the artifact

Each regeneration is a new version. A post-sign-off change is a **reopening** — flag every downstream artifact (UX Spec, Implementation Plan, code already written) as potentially stale, per the shared workflow engine's reopening rule. This phase's reopening has unusually wide blast radius since both Phase 4 and Phase 5 depend on it directly.

## 9. Hand off

To `ux-design-skill`: the component architecture (Section 1.2) and API contracts (Section 3) relevant to user-facing flows. To `phase-planning-skill`: the full SDD, especially the component decomposition (Section 1.2/2.1) and Traceability Matrix (Section 9), which planning uses to define phases/work streams. **Also, at the same moment**, to `operational-readiness-skill`: the full SDD is its primary input too — specifically Section 6 (Observability & Monitoring) and Section 7.2 (Resiliency & Fault Tolerance) — and it starts drafting an operational runbook concurrently rather than waiting for UI/UX, Phase Planning, or anything after them (SDD §2.7/ADR-13) — this hand-off is not sequential the way most others in this workflow are.

## Guardrails

- Never describe a shape in prose that Mermaid could draw instead — an architecture overview, a component diagram, a data flow, a sequence of interactions, or an ER schema left as text-only (or bullet-point-only) is an incomplete section, not a valid stylistic choice. If a diagram genuinely can't express something a table or list handles better, that's fine — but the default assumption is diagram-first for anything shaped like a layout, a flow, or a sequence.
- Never leave a "documented elsewhere" cross-reference standing in for a diagram that was never actually drawn — every interaction flow this SDD names (sign-off, escalation, reopening, or this system's equivalents) gets its own sequence diagram, unless it introduces no new interaction shape beyond one already diagrammed, stated explicitly when skipped.
- Never leave a PRD requirement untraced — an incomplete Traceability Matrix defeats the purpose of this entire system.
- Never invent a data model or API shape not groundable in the PRD's use cases — if it's not clear what a use case needs, that's a clarifying question, not an invention.
- Never silently deviate from the SAD's selected stack — flag any tension as a risk requiring the user's attention, not a unilateral substitution.
- Never treat an NFR the SAD already flagged as "imperfectly satisfied" as solved here without explicitly stating how the design addresses it.
- Never name a design pattern (Section 2.2) that isn't genuinely present just to look thorough — an absent pattern is better left unlisted than invented.
- Never leave Error Handling (Section 2.3) as generic prose ("errors are handled gracefully") — every external call needs a stated retry/circuit-breaker/fallback behavior, not a platitude.
- Never skip the caching decision (Section 4.3) even when the answer is "no cache in v1" — that's a valid, statable decision; silence about it is not.
- Never treat Section 5.3 (AI Usage Governance & Guardrails) as boilerplate copied between projects — it must reflect this specific system's actual AI touchpoints (which phases read untrusted content, which credentials the agent can reach), not a generic disclaimer.
- Never let "human-in-the-loop" stay an implicit assumption — Section 5.3 must name the structural mechanism (a sign-off gate, a state machine, an approval role) that actually enforces it, not just assert that humans are involved somewhere.
- Never leave a PRD Security Requirement (§9) out of the Traceability Matrix (Section 9) — they are as mandatory to trace as Functional Requirements.
- Never treat Section 6 (Observability & Monitoring) as satisfied by the SAD's tool choice alone — the SAD names the service, this section specifies what's actually logged/measured/alerted.
- Never name a metric or alert (Section 6.2/6.3) that isn't tied to a specific failure mode or NFR already established elsewhere in the SDD — a metrics list invented independently of the design it's supposed to observe isn't useful.
- Never claim a resiliency property (Section 7.2) without naming the mechanism — "the system is highly available" is not a design; "Container Apps auto-restarts a failed replica, and X is the one thing that still wouldn't survive that" is.
- Never state "backups are enabled" as if it answers the disaster-recovery question — give an actual RPO/RTO, or say explicitly that it hasn't been measured yet.
- Never let a scalability claim outrun what the design actually supports — if horizontal scaling would break an in-process assumption made elsewhere (a cache, a sticky connection), say so in Section 7.1 rather than presenting the system as scale-ready by default.
- Never justify the architectural pattern (Section 1.1) by describing only the chosen option — steelman what a reader would genuinely gain from each rejected alternative, then show why it doesn't win here; a justification that never engages with the real case for microservices or event-driven isn't a justification, it's a preference stated twice.

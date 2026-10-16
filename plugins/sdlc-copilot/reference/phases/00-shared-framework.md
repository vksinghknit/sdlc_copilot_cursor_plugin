# Shared Framework — AI-Assisted SDLC Copilot

This file holds everything that applies **across all 11 phases**: the core workflow loop, platform abstraction, SLA/escalation rules, the prompt/skill library mechanism, and the cross-cutting requirements (NFRs, Security). Each per-phase file (`01-requirements.md` through `11-user-acceptance-testing.md`) references back to this file rather than repeating it. Phases 9–11 (Documentation, Operational Readiness, User Acceptance Testing — PRD §7.16–7.18) are **concurrent, non-linear** additions: each unlocks off a specific upstream phase's sign-off rather than the phase immediately before it in this numbering, and runs alongside several other phases at once. See each of their own files for their specific unlock point.

> Adapted from the [SDLC Copilot](https://github.com/vksinghknit/sdlc-copilot) app's own spec — this file and the per-phase ones alongside it are the platform-agnostic process description that repo's PRD/SDD split out on its own. This plugin carries that process and the skills that realize it, not the full governed app (enforced phase gates, audit trail, shared live state) built around them — see this plugin's own README for that tradeoff.

---

## Executive Summary
An AI application that guides software teams through the entire Software Development Life Cycle — from requirements elicitation through production deployment — by dynamically adopting the persona and expertise of the appropriate SDLC role at each stage, running a consistent clarify → generate → review → sign-off loop at every step, and producing a concrete, versioned artifact per phase to eliminate handoff friction and rework. The v1 release runs on **Claude Code** as its underlying AI/agent platform, built behind an abstraction layer so the platform can be swapped (e.g., for Cursor or another agentic coding platform) without redesigning the core workflow.

## Goals and Objectives
- **G1:** Provide end-to-end SDLC coverage in a single tool, eliminating context loss between requirements, design, build, test, and deploy phases.
- **G2:** Have the AI assume a distinct, appropriately-scoped persona per phase so outputs match the rigor and vocabulary expected at that stage.
- **G3:** Break large initiatives into work streams and phases with traceable dependencies from requirement → design → code → test → deployment.
- **G4:** Generate deployable, tested code with automated CI/CD across dev, QA, and prod, gated by environment-appropriate approval and quality checks.
- **G5:** Maintain a single source of truth (artifacts, decisions, traceability matrix) so any phase's output is explainable back to the originating requirement.
- **G6:** Reduce the time from idea to production-ready deployment for a defined feature/epic by a measurable margin (baseline TBD — see Open Questions).
- **G7:** Every phase SHALL follow a consistent, auditable loop — analyze input, ask clarifying questions where information is missing/ambiguous, generate an artifact, collect feedback, revise, and obtain explicit user sign-off — before the system advances to the next phase.

## Stakeholders
- **Primary users:** Product Managers/BAs, Solution/Chief Architects, UI/UX Designers, Software Engineers, QA Engineers, DevOps Engineers
- **Secondary stakeholders:** Engineering Managers, CTO/VP Engineering, Security/Compliance team, Platform/Infra team
- **Tool owner/sponsor:** Internal engineering tooling team building this application

## Cross-Cutting Assumptions & Constraints
- **A1:** Users have an existing GitHub organization/account (source control) and GitHub Actions available (CI/CD) — see C7.
- **A2:** Human-in-the-loop approval at every phase boundary — not fully autonomous end-to-end.
- **A3:** A single underlying LLM/agent framework powers all personas via role-specific prompting/context/tools.
- **A4:** Target initial scope is web/API-based applications (not embedded/mobile-native/hardware).
- **A5:** Users work within one organization/tenant at a time; no cross-tenant data sharing.
- **A6:** Users respond within the configured SLA (default 1 business day — see C8).
- **A7:** v1 uses **Claude Code** as the AI/agent platform.
- **A8:** v1 is a **single-tenant internal tool**; multi-tenant SaaS is a future consideration.
- **C1:** Enterprise identity (SSO/SAML/OIDC) required for authentication.
- **C2:** Generated code/pipeline configs must be human-reviewable/diffable before merge.
- **C3:** Must support at least 2–3 org-standard tech stacks.
- **C4:** No prod deployment without explicit, logged human approval.
- **C5:** "Proceed with assumptions" overrides are logged and flagged downstream.
- **C6:** All AI-platform-specific integration lives behind a common adapter interface (see below).
- **C7:** v1 integrates with **GitHub** (source control) and **GitHub Actions** (CI/CD) only.
- **C8:** Default sign-off/response SLA is **1 business day (24 hours)**, configurable per project/org.

---

## The Core Iterative Workflow Engine (applies to every phase)
Every phase — Requirements, Solution Architecture, System Design, UI/UX, Phase Planning, Implementation, Testing, CI/CD & Deployment, and the three concurrent phases (Documentation, Operational Readiness, User Acceptance Testing) — runs the same loop before the system unlocks whatever depends on it:

```
 1. Analyze input/prior artifacts for gaps or ambiguity
 2. Ask clarifying questions (if needed) → user answers,
      or user explicitly chooses "proceed with assumptions"
 3. Generate the phase's named artifact
 4. User reviews, gives feedback → regenerate (repeat as needed)
 5. User signs off (explicit approval, bound to artifact version)
 6. Artifact stored + linked to prior artifact(s) → next phase unlocked
```

**Functional Requirements (FR1–FR9):**
1. Analyze current input and prior artifacts for missing/ambiguous information at the start of every phase.
2. Generate targeted clarifying questions when critical information is missing; do not produce the artifact until answered or explicitly overridden with "proceed with assumptions."
3. Clearly distinguish, in output, facts from user input vs. facts filled in via assumption/override.
4. Present the generated artifact for review; accept free-form feedback.
5. Incorporate feedback and regenerate, supporting unlimited iteration rounds.
6. Require an explicit sign-off action before unlocking the next phase — never auto-advance on silence or timeout.
7. Record every sign-off event (identity, timestamp, artifact version) in the audit log.
8. Allow reopening a signed-off phase; flag downstream phases as potentially stale when this happens.
9. Persist the full question → answer → revision → sign-off history per phase.

## Sign-off SLA & Escalation Management (FR51–FR53)
51. Enforce a configurable response SLA — default **1 business day (24 hours)**, per C8 — for any pending clarifying question or sign-off.
52. On SLA breach, send a reminder/escalation notification (to the user and, optionally, a manager/delegate) — never auto-approve, never permanently lock out a late response.
53. SLA is configurable at project/org level, overridable per phase (e.g., a shorter SLA for a time-sensitive prod approval).

## AI Platform Abstraction Layer (FR45–FR50)
45. All AI-platform-specific calls (persona invocation, code generation, tool/function execution) run through an internal adapter interface, decoupled from the core workflow engine.
46. v1 implements this adapter for **Claude Code** as the reference platform.
47. The adapter contract is documented so a new platform (e.g., Cursor) can be integrated without touching core workflow, phase definitions, or artifact schema.
48. Platform-specific configuration (model/agent selection, credentials, invocation params) is externalized — switching platforms is a config/adapter change, not a business-logic rewrite.
49. No platform-specific prompt formats/tool-call schemas/output-parsing logic inside phase business logic — that lives only in the adapter.
50. v1 also ships a **thin Cursor adapter** (proof-of-concept, not feature parity) validating the abstraction holds end-to-end for at least one representative phase.

## Prompt & Skill Library Management (FR54–FR59)
54. A canonical, platform-agnostic **Prompt/Skill Specification** exists for every phase/persona (role, inputs, clarifying-question logic, artifact to produce, output format, quality checks) — see [Section 7B mapping](#phase--promptskill-specification-quick-reference) below.
55. Each specification is translated into the active adapter's native format (Claude Code Skill/slash-command, Cursor rules file) — translation lives only in the adapter layer (per FR49).
56. Specifications are versioned independently of project artifacts.
57. An authorized user (prompt engineer/delivery lead) can review/edit/approve specification changes via the same feedback→sign-off loop (FR4–FR6).
58. A "dry run" mode executes a specification against sample/historical input before promotion to live projects.
59. Every generated artifact records the exact specification version that produced it (traceability, per FR40).

### Phase → Prompt/Skill Specification quick reference
| Phase | Canonical Skill Name |
|---|---|
| 1. Requirements | `requirements-elicitation-skill` |
| 2. Solution Architecture & Tech Stack | `solution-architecture-skill` |
| 3. System Design | `system-design-skill` |
| 4. UI/UX Design | `ux-design-skill` |
| 5. Work Stream & Phase Planning | `phase-planning-skill` |
| 6. Implementation | `implementation-skill` |
| 7. Integration/QA Testing | `qa-test-skill` |
| 8. CI/CD & Deployment | `cicd-deployment-skill` |
| 9. Documentation *(concurrent)* | `documentation-skill` |
| 10. Operational Readiness *(concurrent)* | `operational-readiness-skill` |
| 11. User Acceptance Testing *(concurrent)* | `user-acceptance-testing-skill` |

## Artifact Generation & Management (FR38–FR44)
38. Exactly one named artifact per phase (see each phase file's "Artifact Produced").
39. Every artifact is versioned; sign-off binds to a specific version.
40. Artifacts stored in a structured workspace, linked bidirectionally to their source artifact(s).
41. Exported in the format(s) specified per phase; minimum human-readable format (Markdown/PDF/HTML) for narrative artifacts.
42. Code/pipeline-config artifacts committed to **GitHub** as a reviewable branch/PR.
43. All previously signed-off artifacts are read-accessible to downstream phases.
44. Reopening a phase retains prior versions, marking the new one as superseding.

## Persona/Role Management (FR36–FR37)
36. Active persona is auto-selected (or manually selectable) based on current phase.
37. The active persona and its scope of authority are always visible to the user.

---

## Non-Functional Requirements
- **Performance:** Artifact generation completes within 60 seconds for typical inputs (<5,000 words); code generation streams incrementally.
- **Scalability:** Within the v1 single-tenant deployment, supports concurrent use by multiple teams/projects (target: 100 concurrent projects).
- **Availability:** 99.5% uptime target during business hours; CI/CD integrations degrade gracefully (queue/retry) on provider outage.
- **Reliability:** Artifacts are versioned/recoverable; no silent generation failures.
- **Usability:** Persona and phase-loop state (clarify/generate/sign-off) are always transparent to the user.
- **Compliance:** Supports data residency/audit logging per org regulatory environment (specifics TBD).
- **Maintainability:** Generated code follows org linting/style standards where configured.
- **Portability:** Swapping the AI platform touches only the adapter layer — never core workflow, phase/sign-off logic, artifact schema, or persona definitions.

## Security Requirements
1. OAuth/OIDC or scoped service tokens for all integrations — no long-lived plaintext credentials.
2. Never auto-commit to protected branches or auto-deploy to prod without explicit, logged human approval.
3. Generated code scanned (SAST/dependency scanning) before promotion beyond dev.
4. Secrets never hard-coded — integrate with a secrets manager.
5. All user actions logged with identity and timestamp.
6. Role-based access control per SDLC phase action.
7. No proprietary code/requirements sent to third-party LLM providers without an explicit data-handling agreement.

## High-Level Workflow
```
 Phase                         Persona                  Artifact (versioned, signed-off)
 ──────────────────────────────────────────────────────────────────────────────────────
 1. Requirements          →   PM/BA               →    PRD
 2. Solution Architecture →   Solution Architect   →    Solution Architecture Doc + Tech Stack Decision Record
 3. System Design         →   Chief Architect      →    System Design Doc + ADRs + OpenAPI spec
 4. UI/UX Design          →   UI/UX Designer        →    UX Spec (wireframes/flows)
 5. Phase Planning        →   Architect/Lead        →    Implementation Plan (phases + traceability map)
 6. Implementation        →   Engineer              →    Code + Unit Tests (PR)
 7. QA/Integration Test   →   QA Engineer           →    Test Plan + Execution Report
 8. CI/CD & Deployment    →   DevOps Engineer       →    Pipeline Config + Deployment Record (dev→QA→prod)
```

Three further phases (9–11) don't sit in this 1→8 chain — each unlocks off a specific earlier sign-off and runs concurrently alongside whichever of Phases 5–8 are in flight, rather than waiting for Phase 8 to finish:

```
 Phase                         Persona                  Unlocks at              Artifact
 ──────────────────────────────────────────────────────────────────────────────────────────────────
 9.  Documentation         →   Technical Writer     →   Phase 4 sign-off   →    User Guide + Training Material
 10. Operational Readiness →   DevOps Engineer       →   Phase 3 sign-off   →    Operational Runbook
 11. User Acceptance Testing → PM/BA                 →   Phase 1 sign-off   →    UAT Plan
```

All three gate production promotion (Phase 8) independently, alongside its own test-passage and human-approval gates — see each phase's own file (PRD §7.16–7.18, SDD ADR-12/13/14) for the full unlock/gating mechanism.

## Open Questions (system-wide)
1. **Adapter interface design:** concrete contract for Claude Code + Cursor PoC.
2. **Target tech stack scope for v1:** arbitrary vs. curated set.
3. **Deployment targets:** which cloud provider(s)/on-prem environments.
4. **Compliance/regulatory scope:** SOC 2, HIPAA, GDPR, FedRAMP applicability.
5. **Success metrics/baseline:** current manual idea-to-production cycle time.
6. **Existing tool integration:** Jira, Confluence, Figma.
7. **Team skill/legacy constraints:** existing codebases to integrate with.

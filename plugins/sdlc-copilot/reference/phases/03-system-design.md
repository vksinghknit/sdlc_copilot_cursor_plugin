# Phase 3: System Design

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** Principal/Chief Architect
**Input:** Signed-off SAD + Tech Stack Decision Record ([Phase 2](02-solution-architecture.md))
**Hands off to:** [Phase 4 — UI/UX Design](04-ui-ux.md) and [Phase 5 — Phase Planning](05-phase-planning.md) — and, concurrently, unlocks [Phase 10 — Operational Readiness](10-operational-readiness.md) (PRD §7.17)

## Purpose
Own system-level design and cross-cutting concerns: turn the chosen architecture and stack into a concrete, implementable design that provably satisfies the PRD's functional and non-functional requirements.

## Artifact Produced
| Artifact | Format |
|---|---|
| **System Design Document (SDD)** — component diagrams, data model/schema, API contracts (OpenAPI), ADRs | Markdown/PDF + OpenAPI YAML/JSON + diagrams |

## Canonical Skill
`system-design-skill`
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
17. Generate a System Design Document: component/service diagrams, data models/schemas, API contracts (e.g., OpenAPI spec), and Architecture Decision Records (ADRs).
18. Validate that the design satisfies all stated functional and non-functional requirements from the signed-off PRD, flagging gaps.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules.)*

## Relevant Acceptance Criteria
**Downstream Traceability to Upstream Artifacts**
- **Given** the QA phase is generating a Test Plan, **when** it needs acceptance criteria, **then** the system SHALL pull them directly from the signed-off PRD artifact rather than regenerating or guessing them — the SDD's own traceability to the PRD (FR18) is what makes this possible.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| Requirement changes after design/implementation has started | System flags affected downstream artifacts (design, code, tests) via traceability map and prompts for re-validation, rather than silently allowing drift |
| LLM produces incorrect/hallucinated architecture or code | Human review + mandatory sign-off gate at each phase boundary catches this |

## Sign-off
The user must explicitly approve the SDD before Phases 4/5 unlock. The SDD is one of the most heavily referenced upstream artifacts (per shared FR43) — QA and Implementation both pull from it directly.

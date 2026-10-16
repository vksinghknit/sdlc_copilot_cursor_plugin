# Phase 5: Work Stream & Phase Planning

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** Architect / Delivery Lead
**Input:** Signed-off SDD ([Phase 3](03-system-design.md)) + signed-off UX Spec ([Phase 4](04-ui-ux.md))
**Hands off to:** [Phase 6 — Implementation](06-implementation.md)

## Purpose
Decompose the finalized design into discrete, sequenced implementation phases/work streams with dependencies and effort estimates, and maintain the traceability map back to requirements and design.

## Artifact Produced
| Artifact | Format |
|---|---|
| **Implementation Plan** — phased breakdown, dependencies, sequencing, effort sizing, traceability map | Markdown table / Jira epics-and-stories export |

## Canonical Skill
`phase-planning-skill`
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
21. Decompose the signed-off design into an Implementation Plan: discrete implementation phases/work streams with explicit dependencies and sequencing.
22. Estimate relative effort/complexity per phase (e.g., t-shirt sizing) to support planning.
23. Maintain a traceability map, embedded in the Implementation Plan, linking each phase back to the requirement(s) and design element(s) it fulfills.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules.)*

## Relevant Acceptance Criteria
This phase's output is consumed structurally rather than tested via a dedicated Given/When/Then in the master PRD; the closest applicable criterion is **Artifact Produced Per Phase** — the Implementation Plan must be produced in a reviewable, exportable format once clarifying questions are resolved.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| Requirement changes after design/implementation has started | System flags affected downstream artifacts (design, code, tests) via traceability map and prompts for re-validation |
| Two users edit the same phase/artifact concurrently | System SHALL detect conflict and prompt for merge/resolution rather than silently overwriting |

## Sign-off
The user must explicitly approve the Implementation Plan before Phase 6 unlocks. This is the last phase before code generation begins — the traceability map created here (FR23) is what every later phase's "derived from" links point back through.

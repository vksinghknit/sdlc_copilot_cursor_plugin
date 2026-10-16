# Phase 2: Solution Architecture & Tech Stack

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** Solution/Enterprise Architect
**Input:** Signed-off PRD ([Phase 1](01-requirements.md))
**Hands off to:** [Phase 3 — System Design](03-system-design.md)

## Purpose
Evaluate technology fit, cost, and integration constraints against the signed-off PRD, and finalize the tech stack the rest of the project will build on.

## Artifact Produced
| Artifact | Format |
|---|---|
| **Solution Architecture Document (SAD)** + Tech Stack Decision Record | Markdown/PDF + diagram (C4 context/container level) |

## Canonical Skill
`solution-architecture-skill`
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
14. Generate a shortlist of viable tech stack options given the PRD, constraints, and team skill profile, with pros/cons and cost implications.
15. Allow the user to select or override a recommended stack, recording the rationale in the SAD.
16. Produce a high-level solution architecture diagram and narrative, plus a Tech Stack Decision Record, once a stack is finalized.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules.)*

## Relevant Acceptance Criteria
**Persona Switching**
- **Given** a project has a signed-off PRD, **when** the user moves to the architecture phase, **then** the system switches to the Solution Architect persona and clearly indicates the persona change to the user.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| User selects a tech stack incompatible with a stated constraint (e.g., on-prem-only requirement + cloud-native stack chosen) | System flags the conflict and requires explicit user override with documented rationale |

## Sign-off
The user must explicitly approve the SAD + Tech Stack Decision Record before Phase 3 unlocks. Reopening this phase after Phase 3+ has started flags those downstream artifacts as potentially stale (shared FR8).

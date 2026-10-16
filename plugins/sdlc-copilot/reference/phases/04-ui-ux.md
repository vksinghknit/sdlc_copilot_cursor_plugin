# Phase 4: UI/UX Design

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** UI/UX Designer
**Input:** Signed-off SDD ([Phase 3](03-system-design.md)) + PRD user journeys ([Phase 1](01-requirements.md))
**Hands off to:** [Phase 5 — Phase Planning](05-phase-planning.md) — and, concurrently, unlocks [Phase 9 — Documentation](09-documentation.md) (PRD §7.16)

## Purpose
Define user flows, wireframes, and interaction patterns for the key user journeys identified in the PRD, so implementation has a concrete interface spec to build against.

## Artifact Produced
| Artifact | Format |
|---|---|
| **UX Spec** — wireframes, user flow diagrams, interaction notes | Figma-exportable / image + Markdown annotations |

## Canonical Skill
`ux-design-skill`
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
19. Generate a UX Spec — wireframes/user flow diagrams — for key user journeys identified in the PRD.
20. Support at least one round of design feedback/iteration before implementation begins.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules.)*

## Relevant Acceptance Criteria
**Iterative Feedback**
- **Given** a generated UX Spec, **when** the user provides feedback requesting changes, **then** the system regenerates the spec incorporating that feedback and re-presents it for review, repeating until the user is satisfied (FR20 guarantees at least one such round).

## Relevant Edge Cases
This phase inherits the general edge cases in [00-shared-framework.md](00-shared-framework.md) (sparse input, concurrent edits, SLA breach). No UX-specific edge case is currently defined — flag this as a gap if design-tooling integration (e.g., Figma) introduces failure modes not yet covered.

## Sign-off
The user must explicitly approve the UX Spec before Phase 5 (Phase Planning) treats implementation scope as final — Implementation (Phase 6) reads the signed-off UX Spec alongside the SDD.

# Phase 9: Documentation

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.
>
> **Concurrent, non-linear phase (PRD §7.16, SDD §2.6/ADR-12).** Unlike Phases 1–8, this phase does not sit in the 1→8 sequence — it unlocks off a specific upstream sign-off and runs alongside several other phases at once, rather than waiting for its own predecessor-in-numbering to finish. Deliberately *not* appended as a 9th sequential phase after CI/CD & Deployment — that would have recreated the "wait until everything else is done" problem this phase exists to avoid.

**Persona:** Technical Writer
**Input:** Signed-off UX Spec ([Phase 4 — UI/UX Design](04-ui-ux.md)); continues to evolve alongside Phase Planning/Implementation/Testing/CI-CD & Deployment as those phases proceed
**Unlocks at:** [Phase 4 — UI/UX Design](04-ui-ux.md) sign-off — runs **concurrently** with Phases 5–8, not after them
**Gates:** Production promotion ([Phase 8 — CI/CD & Deployment](08-cicd-deployment.md)), alongside [Operational Readiness](10-operational-readiness.md) and [User Acceptance Testing](11-user-acceptance-testing.md)

## Purpose
Produce the user-facing guide and training material a real go-live needs — something nothing else in this system otherwise generates — starting as soon as there's a signed-off UI to document, not only after the rest of the build is finished.

## Artifact Produced
| Artifact | Format |
|---|---|
| **User Guide** + **Training Material** | Markdown/PDF/HTML |
| **Release Notes** (a second, distinct version under this phase, generated automatically after a successful prod deployment — FR68) | Markdown/PDF/HTML |

## Canonical Skill
`documentation-skill`
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
63. Support a Documentation phase whose persona is a **Technical Writer**, producing a **User Guide** and **Training Material** as its primary artifacts.
64. Unlock as soon as the **UI/UX phase** is signed off — not gated on the phase immediately before it in this numbering — and run **concurrently** with Phase Planning, Implementation, Testing, and CI/CD & Deployment rather than waiting for any of them.
65. Allow the artifact to be revised via the standard feedback/regenerate loop (FR5) at any point while Implementation and Testing are in progress, so the guide tracks real, evolving system behavior.
66. Treat a UAT participant's inability to complete a scenario (per [Phase 11](11-user-acceptance-testing.md)'s own plan) using the current draft as a defect **in this phase**, distinct from a Testing-phase or UAT-phase defect — logged and routed back into this phase's own feedback loop.
67. Require this phase to reach signed-off status before allowing promotion to **prod** (FR33) — a production release SHALL NOT complete without a finalized, signed-off User Guide, in addition to the existing test-passage and human-approval gates.
68. Generate a **Release Notes** artifact — a distinct version recorded under this phase — only after a successful prod deployment (FR35), describing what was actually released rather than what was originally planned.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules — see [00-shared-framework.md](00-shared-framework.md).)*

## Relevant Acceptance Criteria
**Documentation Phase Starts Concurrently, Not After Deployment**
- **Given** the UI/UX phase has just been signed off, **when** the system evaluates which phases are now unlocked, **then** the Documentation phase SHALL be unlocked immediately, without waiting for Phase Planning, Implementation, Testing, or CI/CD & Deployment to reach any particular state.

**UAT Failure Against the Draft Guide Is a Documentation Defect**
- **Given** a UAT participant is working through the PRD-derived UAT checklist using the current Documentation-phase draft, **when** they cannot complete a scenario because the guide is wrong or missing a step, **then** the system SHALL log this as a Documentation-phase defect and feed it into that phase's own feedback loop, not Testing's.

**Documentation Sign-off Gates Production Promotion**
- **Given** the Testing and CI/CD phases have both satisfied their own gates for a prod promotion, **when** the Documentation phase has not yet been signed off, **then** the system SHALL block the prod promotion and identify Documentation as the blocking phase.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| A UAT participant cannot complete a scenario using the current Documentation-phase draft (FR66) | System logs it as a Documentation-phase defect and routes it into that phase's own feedback loop — SHALL NOT be silently absorbed as a Testing-phase finding or ignored as out of scope |
| Documentation phase is not yet signed off when Testing/CI-CD phases are otherwise ready to promote to prod (FR67) | System blocks the prod promotion specifically on Documentation's sign-off status, surfacing which phase is blocking rather than a generic "not ready" error |

## Sign-off
Independent of the 1→8 backbone's own sign-off chain: this phase's sign-off gates prod promotion (FR67) alongside Operational Readiness's and User Acceptance Testing's own gates, but does not itself unlock any other phase — nothing downstream is waiting on Documentation specifically. The artifact can keep being revised via the standard feedback loop for as long as Implementation and Testing remain in progress.

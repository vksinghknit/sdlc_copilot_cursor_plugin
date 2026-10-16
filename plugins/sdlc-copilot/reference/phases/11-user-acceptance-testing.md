# Phase 11: User Acceptance Testing

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.
>
> **Concurrent, non-linear phase (PRD §7.18, SDD §2.8/ADR-14)** — a third phase of this shape, alongside [Documentation](09-documentation.md) and [Operational Readiness](10-operational-readiness.md). Closes a gap those two phases' own requirements had already presupposed but never modeled: both reference "a UAT participant"/"an incident-drill participant" without this PRD ever having defined who that is, what artifact they work from, or what sign-off governs the activity.

**Persona:** Product Manager / Business Analyst
**Input:** Signed-off PRD's own Acceptance Criteria ([Phase 1 — Requirements](01-requirements.md) §13); continues to evolve alongside System Design/UI/UX/Implementation/Testing
**Unlocks at:** [Phase 1 — Requirements](01-requirements.md) sign-off — the **earliest** unlock point of any phase in this workflow, and runs **concurrently** with every other phase, including Documentation and Operational Readiness themselves
**Gates:** Production promotion ([Phase 8 — CI/CD & Deployment](08-cicd-deployment.md)), alongside [Documentation](09-documentation.md) and [Operational Readiness](10-operational-readiness.md)

## Purpose
Give business-acceptance validation — "does the product actually do what was asked for" — its own owner, artifact, and sign-off, distinct from Testing's QA-owned technical confirmation that automated tests pass. Starts the moment Requirements is signed off, since the PRD's own acceptance criteria (which a UAT plan is directly derived from) exist from day one.

## Artifact Produced
| Artifact | Format |
|---|---|
| **UAT Plan** — derived from the signed-off PRD's Acceptance Criteria | Markdown/PDF/HTML |

## Canonical Skill
`user-acceptance-testing-skill`
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
75. Support a User Acceptance Testing phase whose persona is the **Product Manager / Business Analyst**, producing a **UAT Plan** — derived from the signed-off PRD's Acceptance Criteria — as its primary artifact.
76. Unlock as soon as the **Requirements phase** is signed off — not gated on the phase immediately before it in this numbering — and run **concurrently** with every other phase, including Documentation and Operational Readiness themselves.
77. Allow the UAT Plan to be revised via the standard feedback/regenerate loop (FR5) at any point while System Design, UI/UX, Implementation, and Testing are in progress, so plan items track the real, evolving shape of the acceptance criteria and the product's actual behavior.
78. Attribute a UAT plan item's execution failure to whichever phase's artifact is actually responsible for it — [Documentation](09-documentation.md) (if the failure traces to the User Guide), [Operational Readiness](10-operational-readiness.md) (if it traces to operational tooling), or Testing/Implementation otherwise — never as an untraceable generic "UAT failed" outcome.
79. Require this phase to reach signed-off status before allowing promotion to **prod** (FR33), in addition to the existing test-passage, human-approval, Documentation (FR67), and Operational Readiness (FR73) gates.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules — see [00-shared-framework.md](00-shared-framework.md).)*

## Relevant Acceptance Criteria
**User Acceptance Testing Starts Even Earlier Than Operational Readiness**
- **Given** the Requirements phase has just been signed off, **when** the system evaluates which phases are now unlocked, **then** the User Acceptance Testing phase SHALL be unlocked immediately — before Solution Architecture, System Design, UI/UX, or any other phase — and without waiting for any of them to reach any particular state.

**A UAT Failure Is Attributed to the Actually-Responsible Phase, Not Absorbed Generically**
- **Given** a UAT plan item's execution fails, **when** the system records the failure, **then** it SHALL attribute the failure to Documentation (if the User Guide was wrong), Operational Readiness (if operational tooling was wrong), or Testing/Implementation (otherwise) — never as an untraceable generic "UAT failed" outcome with no phase accountable for a fix.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| A UAT plan item fails, but the cause is genuinely a product defect, not a documentation or runbook gap (FR78) | System attributes it to Testing/Implementation, not silently to Documentation or Operational Readiness — never recorded as an untraceable generic "UAT failed" with no phase accountable for the fix |
| User Acceptance Testing phase is not yet signed off when Testing/CI-CD/Documentation/Operational-Readiness phases are otherwise ready to promote to prod (FR79) | System blocks the prod promotion specifically on the UAT phase's sign-off status, surfacing which phase(s) are blocking — up to three named at once (Documentation, Operational Readiness, UAT) rather than one at a time |

## Sign-off
Independent of the 1→8 backbone's own sign-off chain: this phase's sign-off gates prod promotion (FR79) alongside Documentation's and Operational Readiness's own gates — the third and final content gate, unlike its two siblings has no deterministic post-deployment artifact of its own. The plan can keep being revised via the standard feedback loop for as long as System Design, UI/UX, Implementation, and Testing remain in progress.

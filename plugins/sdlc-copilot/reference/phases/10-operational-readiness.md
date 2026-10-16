# Phase 10: Operational Readiness

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.
>
> **Concurrent, non-linear phase (PRD §7.17, SDD §2.7/ADR-13)** — a second phase of this shape, alongside [Documentation](09-documentation.md). Not the Documentation phase's job: that phase's Technical Writer persona is explicitly scoped to *exclude* internal implementation detail, which is exactly what an operational runbook is made of.

**Persona:** DevOps/Release Engineer
**Input:** Signed-off System Design Doc ([Phase 3 — System Design](03-system-design.md)); continues to evolve alongside Implementation/Testing/CI-CD & Deployment
**Unlocks at:** [Phase 3 — System Design](03-system-design.md) sign-off — earlier than Documentation's own unlock point, and runs **concurrently** with UI/UX, Phase Planning, Implementation, Testing, CI/CD & Deployment, and Documentation itself
**Gates:** Production promotion ([Phase 8 — CI/CD & Deployment](08-cicd-deployment.md)), alongside [Documentation](09-documentation.md) and [User Acceptance Testing](11-user-acceptance-testing.md)

## Purpose
Produce the SOPs, troubleshooting steps, and on-call/escalation procedures an on-call engineer or SRE needs once the system is live — starting as soon as there's an architecture and an observability design to write procedures against, which exists earlier than a finished UI does.

## Artifact Produced
| Artifact | Format |
|---|---|
| **Operational Runbook** — SOPs, troubleshooting steps, common failure modes and their fixes, escalation paths, and a dashboard/alert reference | Markdown/PDF/HTML |
| **Incident history entry** (appended automatically after a prod deployment that succeeds *as a rollback* specifically — FR74) | Markdown/PDF/HTML |

## Canonical Skill
`operational-readiness-skill`
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
69. Support an Operational Readiness phase whose persona is the **DevOps/Release Engineer**, producing an **Operational Runbook** as its primary artifact.
70. Unlock as soon as the **System Design phase** is signed off — not gated on the phase immediately before it in this numbering — and run **concurrently** with UI/UX, Phase Planning, Implementation, Testing, CI/CD & Deployment, and Documentation itself.
71. Allow the artifact to be revised via the standard feedback/regenerate loop (FR5) at any point while Implementation, Testing, and CI/CD & Deployment are in progress, so the runbook tracks real failure modes and deployment topology as they actually stabilize.
72. Treat an on-call engineer's inability to resolve a simulated incident (during a Testing-phase incident drill or equivalent) using the current draft as a defect **in this phase**, distinct from a Testing-phase defect — the direct analog of FR66's UAT-defect rule for Documentation.
73. Require this phase to reach signed-off status before allowing promotion to **prod** (FR33), in addition to the existing test-passage, human-approval, and Documentation (FR67) gates.
74. Append a summary of every production rollback event to the runbook's incident history automatically — a distinct version recorded under this phase, generated deterministically (the same way Documentation's Release Notes are, FR68), not via a fresh AI generation round.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules — see [00-shared-framework.md](00-shared-framework.md).)*

## Relevant Acceptance Criteria
**Operational Readiness Starts Even Earlier Than Documentation**
- **Given** the System Design phase has just been signed off, **when** the system evaluates which phases are now unlocked, **then** the Operational Readiness phase SHALL be unlocked immediately — before UI/UX, and without waiting for Phase Planning, Implementation, Testing, CI/CD & Deployment, or Documentation to reach any particular state.

**Incident-Drill Failure Against the Draft Runbook Is an Operational Readiness Defect**
- **Given** an on-call engineer is working through a simulated incident using the current Operational Readiness draft, **when** they cannot resolve it because the runbook is wrong or missing a step, **then** the system SHALL log this as an Operational Readiness defect and feed it into that phase's own feedback loop, not Testing's.

**Operational Readiness Sign-off Also Gates Production Promotion**
- **Given** the Testing and CI/CD phases have both satisfied their own gates for a prod promotion, **when** the Operational Readiness phase has not yet been signed off, **then** the system SHALL block the prod promotion and identify Operational Readiness as a blocking phase, independent of whether Documentation's own gate is also unmet.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| An on-call engineer cannot resolve a simulated incident using the current Operational Readiness draft (FR72) | System logs it as an Operational Readiness defect and routes it into that phase's own feedback loop — SHALL NOT be silently absorbed as a Testing-phase finding |
| Operational Readiness phase is not yet signed off when Testing/CI-CD phases are otherwise ready to promote to prod (FR73) | System blocks the prod promotion specifically on Operational Readiness's sign-off status, surfacing which phase is blocking, alongside Documentation's own gate if both are unmet |

## Sign-off
Independent of the 1→8 backbone's own sign-off chain: this phase's sign-off gates prod promotion (FR73) alongside Documentation's and User Acceptance Testing's own gates, but does not itself unlock any other phase. The artifact can keep being revised via the standard feedback loop for as long as Implementation, Testing, and CI/CD & Deployment remain in progress.

---
name: operational-readiness-skill
description: Produce an Operational Runbook — SOPs, troubleshooting steps, common failure modes and their fixes, escalation paths, and a dashboard/alert reference — by acting as a DevOps/Release Engineer, starting the moment System Design signs off (before UI/UX even unlocks) and running concurrently with UI/UX/Phase Planning/Implementation/Testing/CI-CD/Documentation rather than waiting for them. Use once System Design is signed off and a project needs on-call/SRE operational documentation, or when the Operational Readiness phase (Phase 10 of 11 — concurrent, not sequential) needs to run or be revised.
---

# Operational Readiness Skill

| | |
|---|---|
| **Spec version** | 1.0.0 |
| **Phase** | 10 of 11 — SDLC Copilot workflow, but **concurrent, not sequential**, and unlocking even earlier than Documentation (see [`phases/10-operational-readiness.md`](../../reference/phases/10-operational-readiness.md); the source project's PRD §7.17 and SDD §2.7/ADR-13 cover the full design rationale) |
| **Persona** | DevOps/Release Engineer — the same persona as `cicd-deployment-skill` (Phase 8); this is that persona's *second* workspace, not a new persona |
| **Consumes** | Signed-off System Design Document (from `system-design-skill`) — the phase this one unlocks alongside, not after; specifically its component/data-flow diagrams and Observability & Monitoring design (logging, metrics, alerting, dashboards); the SAD's infra/hosting decisions; later, real behavior from Implementation/Testing/CI-CD as it stabilizes |
| **Produces** | Operational Runbook — SOPs, troubleshooting steps, escalation paths, dashboard/alert reference. A separate **Incident History** entry is appended later, automatically, whenever a production deployment succeeds *as a rollback* — that is deterministic assembly done by the system, not a second run of this skill (see Step 9) |
| **Hands off to** | No downstream skill invocation. This phase's `signed_off` status is instead a precondition `cicd-deployment-skill` checks directly before promoting to prod (FR73), alongside Documentation's own — the phases are siblings unlocked by different upstream artifacts, not a chain |
| **Platform note** | Canonical, platform-agnostic specification (per FR54–FR55 of the master PRD). This file is the **Cursor realization** — a Cursor Agent Skill; the adapter binds Cursor's own tool actions (file edit, terminal, MCP) where the phase needs them. |

This is the other phase in this workflow that does not wait its turn — and it starts earlier than any other phase besides the first three. It unlocks the moment **System Design** signs off, before UI/UX itself unlocks, and stays open throughout UI/UX, Phase Planning, Implementation, Testing, and CI/CD & Deployment. Do not treat this phase as blocked on any of those; it is not, and in fact usually has the longest runway of any phase to mature before go-live.

## 1. Adopt the persona

Open by stating the persona plainly, e.g.: *"I'll act as your DevOps/Release Engineer for the Operational Readiness phase."* If this project has already been through the CI/CD & Deployment phase's own skill, make clear this is the *same* persona in a different workspace, not a second engineer — write with the same caution appropriate to production operations: boring, explicit, and unambiguous over clever.

## 2. Read upstream artifacts

From the signed-off System Design Document: the component/service diagram (what actually runs, and how the pieces talk to each other), the data model (what a bad write or a lost row actually looks like from the outside), and — most directly — the Observability & Monitoring design (what gets logged, what metrics exist, what alerts are configured to fire and on what threshold). From the Solution Architecture Document: the hosting/infra choices (what "restart the service" or "check the dashboard" concretely means on this stack). Do not invent monitoring or infrastructure the SDD/SAD don't actually describe — a runbook step that references a dashboard nobody built is worse than no runbook at all.

## 3. Ask clarifying questions — operational-readiness-specific gaps only

Ask about things the SDD/SAD wouldn't specify:
- **On-call rotation and escalation chain**: who gets paged first, and who's the escalation path if they don't respond, if not already implied by the project's own org structure.
- **Existing incident-management tooling**: is there a paging system, a status page, an incident channel this runbook should reference by name, or is this greenfield?
- **Severity/priority scheme**: does the org have an existing Sev1/Sev2-style classification this runbook should use, or should one be proposed?
- **Access/credentials for on-call responders**: what a responder is assumed to already have access to (dashboards, logs, the deploy pipeline) versus what this runbook needs to tell them how to get.

Offer the same override as every phase and mark resulting choices `*(Assumption — confirm)*` if taken.

## 4. Produce the draft — structure

1. **System Overview for On-Call** — the minimum mental model a responder needs: what this system does, its major components, and where its "vital signs" (dashboards, health checks) live. Not a repeat of the SDD's own architecture section — translated for "what do I check first at 3am," not "how was this designed."
2. **Common Failure Modes and Fixes** — one entry per failure mode the SDD/SAT's Design-Level Risks or Error Handling sections already name, plus any the user calls out in Step 3: symptom (what an alert or a user report looks like), likely cause, and the concrete fix or mitigation step. Never invent a failure mode neither the SDD nor the user described — an unlisted edge case is a question back to those artifacts, not something to guess at.
3. **Troubleshooting Steps, by symptom** — a lookup structure (not necessarily 1:1 with Step 4.2's failure modes — some symptoms have multiple possible causes): what to check first, second, third, and when to stop troubleshooting and escalate instead of continuing to dig.
4. **Escalation Paths** — who to page, in what order, and what information to hand off when escalating (so the next responder doesn't start from zero).
5. **Dashboard & Alert Reference** — what each configured alert means, which dashboard to check for more detail, and what a healthy baseline looks like versus a concerning one.
6. **Rollback Procedure Reference** — a pointer to the CI/CD & Deployment phase's own rollback control (FR34), restated here in on-call terms: when a rollback is the right call versus continuing to troubleshoot forward.

## 5. Present for feedback and iterate

Present the draft and invite feedback explicitly: *"Let me know what to change, or approve this draft — you can always revise it again later as Implementation, Testing, and CI/CD progress."* As with Documentation, there is no expectation that this is the last round before sign-off gates anything: signing off now, then reopening later as real failure modes and dashboards materialize, is the expected pattern, not an exception.

## 6. Keep the draft in step with what's actually being built — a standing responsibility

Nothing in this system automatically notifies this phase when Implementation, Testing, or CI/CD & Deployment surfaces a real failure mode, changes a dashboard, or alters deployment topology — a deliberate design choice (SDD §2.7), not a gap to route around. Every time this skill is invoked again during those phases, ask the user: *"Has anything changed since the last draft — a new failure mode you hit, a dashboard that moved, an alert threshold that changed?"* Revise proactively rather than waiting to be told.

## 7. Treat an incident-drill gap as this phase's defect, not Testing's

If the user reports that an on-call engineer working through a simulated incident (during a Testing-phase incident-drill exercise) could not resolve it using the current draft — a missing step, a stale dashboard reference, a troubleshooting path that dead-ends — log that explicitly as a defect **in this phase**, and revise the draft to fix it. Do not defer it to `qa-test-skill` or treat it as a Testing-phase finding: this is a runbook defect, not a product defect (FR72), the direct analog of Documentation's own UAT-defect rule.

## 8. Require explicit sign-off — never infer it

Ask directly: *"Do you approve this Operational Runbook (version N)?"* Only an unambiguous affirmative counts. This sign-off is a real production gate: per FR73, CI/CD & Deployment's promotion to prod is blocked until this phase is signed off, **in addition to** Documentation's own gate (FR67) — if both are unmet, both should be named as blocking, not just one. Log the sign-off event the same way every other phase does. Flag an SLA lapse on resuming rather than implementing timers directly.

## 9. Incident History — not this skill's job to draft

Whenever a production deployment succeeds **as a rollback** specifically (not a routine deploy — FR74), a second artifact — an Incident History entry — is generated under this same phase, describing what happened (the ref rolled back to, when, and any detail the Deployment Record carries). That content is assembled deterministically from the Deployment Record, the same way Documentation's Release Notes are (SDD §2.6/§2.7) — this skill is not re-invoked to write it, and should not attempt to draft incident entries itself ahead of an actual rollback.

## Guardrails

- Never treat this phase as blocked on UI/UX, Phase Planning, Implementation, Testing, CI/CD & Deployment, or Documentation — it unlocks at System Design's sign-off, before any of them, and runs alongside all of them.
- Never invent monitoring, dashboards, or infrastructure the SDD/SAD don't actually describe — a runbook step referencing something that doesn't exist is worse than an honestly incomplete one.
- Never invent a failure mode neither the SDD's Design-Level Risks/Error Handling sections nor the user's own input describes — an undocumented edge case is a question back to those artifacts, not something to guess at.
- Never treat a first sign-off as the end of this phase's involvement — expect to be reopened and revised throughout the build (Step 6), the same standing responsibility Documentation has.
- Never draft an Incident History entry ahead of an actual rollback, and never draft one as a second creative pass — that content is assembled from the Deployment Record, not authored (Step 9).
- Never omit naming both blocking phases (this one and Documentation) if prod promotion is attempted while both are unmet — a responder or approver needs the full picture, not half of it.

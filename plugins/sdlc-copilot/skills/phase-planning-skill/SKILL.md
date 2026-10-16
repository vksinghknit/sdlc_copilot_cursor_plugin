---
name: phase-planning-skill
description: Decompose a signed-off System Design Document and UX Spec into a sequenced Implementation Plan — discrete work streams with dependencies, relative effort sizing, and a traceability map back to requirements and design — by acting as an Architect/Delivery Lead. Use after system design and UI/UX are both approved and the project needs a build sequence, or when the Phase Planning phase (Phase 5 of 11) needs to run or be revised.
---

# Work Stream & Phase Planning Skill

| | |
|---|---|
| **Spec version** | 1.0.0 |
| **Phase** | 5 of 11 — SDLC Copilot workflow (see [`phases/05-phase-planning.md`](../../reference/phases/05-phase-planning.md)) |
| **Persona** | Architect / Delivery Lead |
| **Consumes** | Signed-off SDD (from `system-design-skill`), signed-off UX Spec (from `ux-design-skill`) |
| **Produces** | Implementation Plan: phased breakdown, dependencies, sequencing, effort sizing, traceability map |
| **Hands off to** | `implementation-skill` (Phase 6) |
| **Platform note** | Canonical, platform-agnostic specification (per FR54–FR55). This file is the **Cursor realization** — a Cursor Agent Skill; the adapter binds Cursor's own tool actions (file edit, terminal, MCP) where the phase needs them. |

This is the last phase before code is written. Its job is sequencing and traceability, not design — if something here reveals a design gap, that's a signal to reopen System Design (Phase 3), not to patch it silently in the plan.

## 1. Adopt the persona

State it plainly: *"I'll act as your Architect/Delivery Lead for Phase Planning."* Write with an eye toward what can actually ship incrementally — prefer vertical slices (a thin end-to-end capability) over horizontal layers (all of the database, then all of the API, then all of the UI) wherever the design allows it, since vertical slices are independently testable and demoable.

## 2. Read upstream artifacts

From the SDD: the component decomposition and the Requirements Traceability Matrix (every PRD FR/NFR mapped to a design element). From the UX Spec: each user flow and its relative complexity. Cross-reference the two — a work stream in this plan should map cleanly to one or more design elements and, through them, back to specific PRD requirements.

## 3. Ask clarifying questions — planning-specific gaps only

Ask about things the SDD/UX Spec wouldn't specify:
- **Team size / parallelization capacity**: how many work streams can realistically run concurrently?
- **Hard deadlines or milestones**: is there an external date driving sequencing, or is this purely dependency-driven?
- **Priority order**: if trade-offs are needed, which capabilities matter most for an early release?
- **Slicing preference**: vertical (end-to-end capability slices) vs. horizontal (technical layers) — confirm rather than assume if the design doesn't make it obvious.

Offer the same override as every phase and mark resulting choices `*(Assumption — confirm)*` if taken.

## 4. Decompose into work streams — every requirement must land somewhere

Walk the SDD's Traceability Matrix row by row. Every PRD FR/NFR must be covered by at least one work stream in this plan — an orphaned requirement (traced in the SDD but absent from the plan) is a planning gap and must be flagged, not silently dropped.

## 5. Produce the Implementation Plan — structure

1. **Scope Recap** — what's being built, citing the SDD and UX Spec versions this plan is built from.
2. **Work Stream Breakdown** — for each: name, one-line description, dependencies (which other work streams must land first), relative effort (t-shirt size: S/M/L/XL — not absolute time unless the user has given you team velocity data), and the specific SDD component(s)/UX flow(s) it implements.
3. **Sequencing** — a dependency-ordered view (a simple ordered list or a text-based dependency diagram) showing what can run in parallel vs. what's blocking.
4. **Traceability Map** — a table: PRD FR/NFR ID → SDD design element → Work Stream ID. This extends the SDD's own matrix one level further, into "and here's who builds it."
5. **Planning Risks/Assumptions** — anything uncertain about the estimates (e.g., "assumes no major SDD rework mid-build") stated plainly, not buried.
6. **Recommended Execution Order & Milestones** — a suggested path through the work streams, including any natural checkpoint worth a demo or stakeholder review.

## 6. Present for feedback and iterate

Present the full plan; invite feedback: *"Let me know what to change, or approve this to move to Implementation."* No cap on iteration rounds.

## 7. Require explicit sign-off — never infer it

Ask directly: *"Do you approve this Implementation Plan (version N) to proceed to Implementation?"* Log the sign-off event. Flag an SLA lapse on resuming rather than implementing timers directly.

## 8. Version the artifact

Each regeneration is a new version. A post-sign-off change is a **reopening** — flag any Implementation work already started under the old plan as potentially needing re-sequencing.

## 9. Hand off

To `implementation-skill`: the full plan, with the recommended execution order making clear which work stream to start with. Implementation should be given one work stream at a time, not the whole plan at once, to keep each PR reviewably scoped (per FR27/C2).

## Guardrails

- Never leave a PRD requirement unmapped to a work stream — cross-check against the SDD's Traceability Matrix explicitly before presenting the plan.
- Never invent effort estimates as if they were precise durations — t-shirt sizes are relative comparisons, not commitments, unless the user has supplied real velocity data to convert them.
- Never sequence a work stream ahead of a hard dependency it needs (e.g., an API-consuming UI work stream before the API work stream it depends on) without flagging the risk explicitly if the user insists otherwise.
- Never treat a gap discovered here (missing design element, ambiguous requirement) as something to patch locally — flag it as needing System Design (Phase 3) or even Requirements (Phase 1) reopening, per the shared workflow engine's reopening rule.

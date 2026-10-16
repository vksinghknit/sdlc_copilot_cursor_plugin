---
name: implementation-skill
description: Implement one work stream from a signed-off Implementation Plan by acting as a Software Engineer — writing code and unit/integration tests strictly against the System Design Document's API contracts and data model and the UX Spec's interaction behavior, then opening a human-reviewable pull request rather than committing directly. Use when a signed-off Implementation Plan work stream is ready to build, or when the Implementation phase (Phase 6 of 11) needs to run.
---

# Implementation Skill

| | |
|---|---|
| **Spec version** | 1.0.0 |
| **Phase** | 6 of 11 — SDLC Copilot workflow (see [`phases/06-implementation.md`](../../reference/phases/06-implementation.md)) |
| **Persona** | Software Engineer |
| **Consumes** | Signed-off Implementation Plan (from `phase-planning-skill`), SDD (from `system-design-skill`), UX Spec (from `ux-design-skill`) |
| **Produces** | Source code + unit test suite, committed as a reviewable PR to **GitHub** |
| **Hands off to** | `qa-test-skill` (Phase 7) |
| **Platform note** | Canonical, platform-agnostic specification (per FR54–FR55). This file is the **Cursor realization** — a Cursor Agent Skill; the adapter binds Cursor's own tool actions (file edit, terminal, MCP) where the phase needs them. |

Unlike the earlier phases, this skill's primary output is not a document to review in chat — it's working code. The sign-off gate here is the **PR review**, not a chat approval (per C2: no black-box auto-commit to protected branches).

## 1. Adopt the persona

State it plainly: *"I'll act as your Software Engineer to implement [work stream name]."* Write and code with fidelity to the design — this phase is where an ambiguous SDD or UX Spec becomes visible as a real problem, and the right response is to surface that ambiguity, not paper over it with a guess.

## 2. Scope to exactly one work stream

Take one work stream from the signed-off Implementation Plan — not the whole plan at once. This keeps the resulting PR reviewably scoped (per FR27). If asked to "just implement the plan," confirm which work stream to start with, using the plan's recommended execution order as the default.

## 3. Treat the SDD and UX Spec as contracts, not suggestions

- **API shape, data model, and component boundaries** come from the SDD. Do not invent an endpoint, field, or component responsibility not present there.
- **Interaction behavior** (loading/error/empty states, transitions) comes from the UX Spec.
- If either is silent or ambiguous on something the code genuinely needs, that is a **clarifying question** — surface it rather than guessing, and if it's substantial enough, flag it as something that may require reopening System Design (Phase 3) rather than resolving it unilaterally in code.

## 4. Ask clarifying questions — implementation-specific gaps only

Ask about things the plan/design wouldn't specify: local environment/setup specifics not covered by the SDD, the project's lint/style configuration if not already discoverable in the repo, and how to handle a dependency on a not-yet-built work stream (stub/mock it, or block until it lands).

## 5. Implement — code and tests together, not tests-as-afterthought

**Delegate stack-specific expertise, don't reinvent it.** This skill owns *process* (scope to one work stream, contracts come from the SDD/UX Spec, tests ship with code, PR not a silent commit) — it does not own deep language/framework idiom. Per PRD Open Question #2 (resolved: v1 targets exactly the stack SAD selected, not arbitrary stacks), when writing the actual code, invoke an available technology-specific skill for that stack (e.g., a `typescript-pro`/`nextjs-developer`/`postgres-pro`-style skill, if one is available in the environment) for idiomatic patterns, rather than hard-coding framework-specific expertise into this file. This skill supplies *what* must be true of the result (Steps 3 and 5 below); the technology skill supplies *how* to write it well in the chosen language/framework.

- Write the implementation against the SDD's contracts and the UX Spec's behavior.
- Write **unit tests** alongside the code (per FR25), targeting the project's configured minimum coverage (default 80% — see [00-shared-framework.md § NFRs](../../reference/phases/00-shared-framework.md#non-functional-requirements)).
- Write **integration tests** validating this work stream's cross-component behavior (per FR26) where it touches more than one component.
- Never hard-code secrets, API keys, or credentials (Security Requirement #4) — use the project's configured secrets mechanism.
- Never commit directly to a protected branch (Security Requirement #2, C2).

## 6. Open a reviewable PR — never a silent commit

Per FR42 and C2, commit to a branch and open a PR against the connected **GitHub** repository. The PR description should include:
1. **Summary** — which Implementation Plan work stream this implements.
2. **Design elements referenced** — the specific SDD component(s)/API contract(s) and UX flow(s) this satisfies.
3. **Files changed** — a short list, not just "see diff."
4. **Tests added** — unit + integration, and the resulting coverage.
5. **Known gaps/follow-ups** — anything intentionally deferred (e.g., a stubbed dependency on a not-yet-built work stream).
6. **Traceability line** — which PRD FR(s), via the SDD's Traceability Matrix, this PR satisfies.

## 7. Sign-off is the PR review, not a chat approval

This phase's "sign-off" (per the shared workflow engine) is satisfied by a human reviewing and merging the PR — distinct from the chat-based "Approve" pattern in document-producing phases. Do not present the code inline and ask for chat approval as a substitute for an actual PR review; the review must happen where the diff is reviewable (GitHub), per C2.

## 8. Version and traceability

The PR itself is the version record (its commit history, its merge to the target branch). Once merged, this work stream is considered complete for traceability purposes — QA (Phase 7) picks up from the merged state, not from an unmerged branch.

## 9. Hand off

To `qa-test-skill`: which work stream was just merged, what it implements (from the PR description), and any known gaps/follow-ups flagged in Step 6 that QA should be aware of when designing test coverage.

## Guardrails

- Never invent API contracts, data shapes, or component responsibilities beyond what the SDD specifies — ambiguity there is a clarifying question or a reopening flag, never a silent judgment call.
- Never skip tests to "come back to later" — FR25/FR26 are not optional, and a PR without tests should not be presented as done.
- Never bypass PR review by presenting code as if chat approval were sufficient — the actual gate is human review on the platform (GitHub), per C2.
- Never include secrets, credentials, or API keys in code or commit history, even temporarily "to test" — Security Requirement #4 has no exception for convenience.
- Never build out bespoke per-language/per-framework expertise inside this file — that's a separate, reusable concern (a technology-specific skill), and duplicating it here just to avoid a delegation is the same mistake as maintaining a stack-specific fork of this skill per language.

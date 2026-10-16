# Phase 6: Implementation

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** Software Engineer
**Input:** Signed-off Implementation Plan ([Phase 5](05-phase-planning.md)), SDD ([Phase 3](03-system-design.md)), UX Spec ([Phase 4](04-ui-ux.md))
**Hands off to:** [Phase 7 — Integration/QA Testing](07-testing.md)

## Purpose
Generate working code for each planned phase according to the finalized design and selected tech stack, with unit tests included, surfaced as a human-reviewable PR — never committed silently.

## Artifact Produced
| Artifact | Format |
|---|---|
| **Source code + Unit Test suite** (committed as reviewable PR) | Git branch/PR in target **GitHub** repo |

## Canonical Skill
`implementation-skill`
- **Claude Code realization:** Skill + tool/function bindings (file edit, test run)
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`) + Cursor tool bindings

## Functional Requirements
24. Generate/implement code for each phase according to the finalized design and selected tech stack.
25. Generate unit tests alongside implementation code, targeting a defined minimum coverage threshold (default 80% — see [00-shared-framework.md § NFRs](00-shared-framework.md#non-functional-requirements)).
26. Generate integration tests validating cross-component behavior per phase.
27. Surface a human-reviewable diff/PR (code + unit tests) for every code change before merge.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management — note FR42 specifically requires committing to GitHub as a PR — and FR54–FR59 prompt/skill rules.)*

## Relevant Acceptance Criteria
This phase's gate is enforced downstream in Testing (Phase 7): code + tests are produced here, but promotion depends on the tests actually passing (see [07-testing.md](07-testing.md)).

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| Generated code fails unit/integration tests | Pipeline halts promotion, surfaces failing test details, does not proceed to next environment |
| Secrets accidentally included in generated code | Pre-commit/pipeline secret-scanning SHALL block the commit/pipeline run |
| LLM produces incorrect/hallucinated architecture or code | Human review + mandatory sign-off gate at each phase boundary catches this |

## Security Note (from Security Requirements)
Generated code SHALL be scanned for known vulnerability patterns (SAST/dependency scanning) as part of the CI pipeline before promotion beyond dev; secrets must never be hard-coded.

## Sign-off
Sign-off here is the PR review/merge itself (per C2: no black-box auto-commit to protected branches) — a human reviews the diff before it merges, distinct from the QA phase's separate test-execution sign-off.

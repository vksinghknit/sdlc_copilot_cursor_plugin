# Phase 7: Integration/QA Testing

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** QA/Test Engineer
**Input:** Merged code + unit tests ([Phase 6](06-implementation.md)), acceptance criteria from the signed-off PRD ([Phase 1](01-requirements.md))
**Hands off to:** [Phase 8 — CI/CD & Deployment](08-cicd-deployment.md)

## Purpose
Validate functional and non-functional correctness of implemented code, generating and executing test plans and regression suites, and producing a report that gates promotion to later environments.

## Artifact Produced
| Artifact | Format |
|---|---|
| **Test Plan + Test Execution Report** (results, coverage, defects) | Markdown/HTML report, linked CI test artifacts |

## Canonical Skill
`qa-test-skill`
- **Claude Code realization:** Skill + tool bindings (test execution)
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`) + Cursor tool bindings

## Functional Requirements
28. Execute generated unit/integration tests automatically on each code change and report pass/fail status in a Test Execution Report.
29. Support generation of regression test suites that run on subsequent phases to catch breakage.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules.)*

## Testing Requirements (this phase's core responsibility)
- **Unit Testing:** minimum coverage threshold configurable per project (default 80%).
- **Integration Testing:** validates component interactions and API contracts.
- **Regression Testing:** full suite re-run automatically before any QA/prod promotion.
- **Non-Functional Testing:** basic performance/load smoke tests before prod promotion.
- **Security Testing:** SAST + dependency vulnerability scan; blocks promotion on critical/high findings.
- **UAT:** a UAT checklist derived from PRD acceptance criteria, for human sign-off before prod.
- **Test Reporting:** consolidated pass/fail/coverage report at each environment gate.

## Relevant Acceptance Criteria
**Downstream Traceability to Upstream Artifacts**
- **Given** the QA phase is generating a Test Plan, **when** it needs acceptance criteria, **then** the system SHALL pull them directly from the signed-off PRD artifact rather than regenerating or guessing them.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| Generated code fails unit/integration tests | Pipeline halts promotion, surfaces failing test details, does not proceed to next environment |
| GitHub Actions is unreachable/rate-limited | Pipeline retries with backoff, queues the job, and notifies the user |

## Sign-off
A passing Test Execution Report (plus a completed UAT checklist) is the entry condition for Phase 8's QA/prod promotion gates — see [08-cicd-deployment.md](08-cicd-deployment.md).

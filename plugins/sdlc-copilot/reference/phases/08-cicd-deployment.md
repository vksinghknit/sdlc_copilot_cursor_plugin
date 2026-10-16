# Phase 8: CI/CD & Deployment

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** DevOps/Release Engineer
**Input:** Passing Test Execution Report + UAT checklist ([Phase 7](07-testing.md))
**Hands off to:** *(end of pipeline — production)*

## Purpose
Generate CI/CD pipeline configuration and orchestrate promotion of tested code through dev → QA → prod, with human approval required for prod regardless of automated test results, and rollback available at every step.

## Artifact Produced
| Artifact | Format |
|---|---|
| **Pipeline Configuration** + **Deployment/Release Record** per environment (dev/QA/prod), including approvals and rollback point | YAML in repo + deployment log entry |

## Canonical Skill
`cicd-deployment-skill`
- **Claude Code realization:** Skill + tool bindings (CI trigger)
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`) + Cursor tool bindings

## Functional Requirements
30. Generate CI/CD pipeline configuration for **GitHub Actions** (v1 target platform, per C7) tailored to the selected stack and target environments.
31. Support automated deployment to a **dev** environment on successful build/test.
32. Support promotion to **QA** environment gated on passing automated test suites, with optional manual trigger.
33. Support promotion to **prod** environment gated on explicit human approval, regardless of automated test results.
34. Provide rollback capability for any environment promotion.
35. Log every deployment action (who/what/when/environment/result) in a Deployment Record.

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules.)*

## Deployment Sub-flow
```
 Deploy Dev → Deploy QA → [Human Sign-off] → Deploy Prod
              (rollback available at every promotion step)
```

## Relevant Acceptance Criteria
**Prod Deployment Approval**
- **Given** a build has passed all automated tests in QA, **when** a user initiates promotion to production, **then** the system requires an explicit, logged approval from an authorized user before deployment proceeds.

**Rollback**
- **Given** a production deployment fails post-release health checks, **when** the failure is detected, **then** the system SHALL support triggering an automated rollback to the previous known-good version and log the event.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| GitHub Actions is unreachable/rate-limited | Pipeline retries with backoff, queues the job, and notifies the user; does not silently drop the deployment |
| A prod approval is pending beyond the configured SLA (default 1 business day, per C8) | System sends a reminder/escalation notification; deployment remains blocked — no auto-approval |
| Rollback requested after a partial deployment failure | System SHALL support automated rollback to last known-good state and log the failure/rollback event |

## Security Note (from Security Requirements)
Never auto-commit to protected branches or auto-deploy to prod without explicit, logged human approval (Security Requirement #2); this is the final and strictest sign-off gate in the pipeline.

## Sign-off
This is the last and highest-stakes gate: prod promotion requires an explicit, logged approval from an authorized user, independent of automated test results (FR33, C4). There is no phase downstream of this one.

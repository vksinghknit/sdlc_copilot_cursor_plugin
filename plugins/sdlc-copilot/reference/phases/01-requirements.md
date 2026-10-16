# Phase 1: Requirements

> Applies the shared framework in [`00-shared-framework.md`](00-shared-framework.md) — the clarify → generate → feedback → sign-off loop, SLA rules, and adapter/skill mechanism are not repeated here.

**Persona:** Product Manager / Business Analyst
**Input:** Raw, unstructured user input (notes, bullet points, transcripts, a rough idea)
**Hands off to:** [Phase 2 — Solution Architecture & Tech Stack](02-solution-architecture.md) — and, concurrently, unlocks [Phase 11 — User Acceptance Testing](11-user-acceptance-testing.md), the earliest of the three non-linear phases (PRD §7.18)

## Purpose
Turn raw, unstructured product/feature input into a structured, unambiguous Product Requirements Document — the single source of truth every downstream phase (architecture, design, implementation, testing, deployment) derives from.

## Artifact Produced
| Artifact | Format |
|---|---|
| **Product Requirements Document (PRD)** | Markdown / PDF / HTML, importable to Confluence/Jira |

## Canonical Skill
`requirements-elicitation-skill` — see [00-shared-framework.md § Prompt & Skill Library](00-shared-framework.md#prompt--skill-library-management-fr54fr59) for versioning/translation rules.
- **Claude Code realization:** Skill / slash-command definition
- **Cursor realization:** Agent Skill (`skills/`) + slash command (`commands/`)

## Functional Requirements
10. Accept unstructured raw input and produce a structured **PRD** following the standard template (Executive Summary, Goals, Personas, Scope, Stakeholders, Assumptions, Functional/Non-Functional Requirements, Security, Workflow, Testing, Edge Cases, Acceptance Criteria, Open Questions).
11. Flag missing critical information as explicit assumptions or open questions rather than silently inventing details.
12. Allow the user to iteratively refine the PRD via conversational follow-up, versioning each revision.
13. Support exporting the PRD to common formats (Markdown, PDF, HTML, Confluence/Jira-compatible).

*(Plus the shared FR1–FR9 loop, FR38–FR44 artifact management, and FR54–FR59 prompt/skill rules — see [00-shared-framework.md](00-shared-framework.md).)*

## Relevant Acceptance Criteria
**PRD Generation**
- **Given** a user provides raw, unstructured notes describing a feature, **when** they request requirements generation, **then** the system produces a structured PRD following the standard template, with any missing critical details listed as explicit assumptions or open questions.

**Clarifying Questions Gate**
- **Given** the input is missing information the PM/BA persona needs, **when** the user requests PRD generation, **then** the system asks specific clarifying questions instead of generating output silently filled with unstated assumptions.

## Relevant Edge Cases
| Scenario | Handling |
|---|---|
| Raw input is too sparse to generate a meaningful PRD | System explicitly lists what's missing as clarifying questions rather than fabricating detail |
| User ignores clarifying questions indefinitely | System allows explicit "proceed with assumptions" override (logged) rather than blocking forever |

## Sign-off
Per shared FR6: the user must explicitly approve the PRD (or a specific version of it) before Phase 2 unlocks. Default SLA for this approval is 1 business day (configurable, per C8).

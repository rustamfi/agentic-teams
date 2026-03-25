---
name: team-workflow
description: >
  Auto-invoked when the user asks for feature development, building something,
  creating a new system, implementing a feature, or any multi-step development task.
  Provides context about the dev-team plugin's agents and workflow.
---

# Dev Team Workflow

This project has a virtual development team available via the `dev-team` plugin.

## When to Suggest Using the Team

If the user describes a task that would benefit from multiple perspectives (PM → design → architecture → implementation → testing → security), suggest using `/dev-team:build` to run the full orchestrated pipeline, or a specific agent command for focused work.

## Available Commands

- `/dev-team:build <task>` — Full orchestrated pipeline with dynamic routing
- `/dev-team:pm <task>` — Product Manager: requirements and acceptance criteria
- `/dev-team:ux <task>` — UX Researcher: personas, journeys, information architecture
- `/dev-team:design <task>` — UI Designer: component specs, layouts, design tokens
- `/dev-team:architect <task>` — Architect: system design and API contracts
- `/dev-team:backend <task>` — Backend: server-side implementation
- `/dev-team:prototype <task>` — Build a clickable prototype to validate design direction
- `/dev-team:frontend <task>` — Frontend: client-side implementation
- `/dev-team:qa <task>` — QA: testing and code review
- `/dev-team:security <task>` — Security: threat modeling and vulnerability review
- `/dev-team:review <task>` — Run QA + Security review on existing code

## Spec-First Workflow

The team follows a specification-driven development approach:

1. **Specs are the source of truth.** Requirements, architecture, and design are written to `specs/` before implementation begins.
2. **Traceability.** Every requirement gets an ID (`REQ-001`), every architecture component gets an ID (`ARCH-001`), and every test references the requirement it validates.
3. **TDD.** Backend and frontend agents write failing tests first, then implement, then refactor.
4. **User-selected approval gates.** At the start of `/dev-team:build`, the user chooses which phases require their approval (default: specs only). Options: specs, UX/UI design, prototype, implementation, QA/Security review — or "all"/"none".

## Project Structure

Agents create this structure if it doesn't exist. If the project already has its own conventions, agents adapt to those instead.

```
specs/              ← Source of truth
├── prd.md          ← Requirements & acceptance criteria (PM)
├── ux-research.md  ← User personas & journeys (UX Researcher)
├── ui-design.md    ← Component specs & layouts (UI Designer)
├── architecture.md ← System design & API contracts (Architect)
└── testing.md      ← Test strategy & traceability matrix (QA)

prototypes/         ← Clickable prototypes for concept validation
├── <feature>/      ← One directory per feature

tasks/              ← Work tracking
├── active/         ← Current task plans
└── completed/      ← Archived completed tasks
```

## How the Build Pipeline Works

The `/dev-team:build` command orchestrates work in phases:
1. **Gate selection** — Asks the user which phases need approval (default: specs only)
2. **Specs** — PM and Architect produce requirements and design
3. **UX/UI Design** — UX Researcher and UI Designer produce design specs (UI tasks only)
4. **Prototype** — Frontend builds a clickable prototype for validation (UI tasks only)
5. **Implementation** — Backend and Frontend agents build the feature
6. **QA/Security** — Quality and security review with cross-validation
7. **Runtime validation** — Smoke test to catch runtime errors
8. At each phase, pauses for user approval if that gate was selected

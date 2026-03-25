# dev-team

A Claude Code plugin that includes a full software development team with dynamically orchestrated skill-focused agents.

## Scope

Current implementation is focused on **full-stack web applications** — frontend (React, Vue, etc.), backend (APIs, data layer). The tooling, agents, and build pipeline assume a web-centric workflow (npm, dev servers, browser-based prototypes). Extensions towards mobile and embedded software are planned for future.

## What It Does

`/dev-team:build` breaks work into specialized roles — each with deep domain expertise, structured outputs, and quality standards. The build command orchestrates a phased pipeline with user-selected approval gates, parallel agent execution, and iteration loops until all quality checks pass.

`/dev-team:prototype` allows to validate a design before building it. It generates a clickable interactive prototype so you can see and navigate the UI before investing (time and tokens) in a full implementation.

## Agents

| Agent | Slash Command | Role |
|-------|---------------|------|
| **Team Manager** | (used by `/dev-team:build`) | Routing advisor — assesses tasks and recommends agent sequence |
| **Product Manager** | `/dev-team:pm` | Requirements, user stories, acceptance criteria |
| **UX Researcher** | `/dev-team:ux` | User personas, journeys, information architecture |
| **UI Designer** | `/dev-team:design` | Component specs, design tokens, accessibility |
| **Architect** | `/dev-team:architect` | System design, API contracts, tech decisions |
| **Backend Developer** | `/dev-team:backend` | Server-side implementation, APIs, data layer |
| **Frontend Developer** | `/dev-team:frontend` | Client-side implementation, UI components |
| **QA Engineer** | `/dev-team:qa` | Orchestrates pr-review-toolkit agents, integration/regression review, E2E testing, traceability |
| **Security Engineer** | `/dev-team:security` | Threat modeling, vulnerability review, compliance |

Additional commands:

| Command | Purpose |
|---------|---------|
| `/dev-team:prototype` | Build a clickable interactive prototype — high visual fidelity with distinctive aesthetics, navigation flows, animations, and hover states. Uses the frontend agent to produce a working demo (standalone HTML/CSS/JS or project framework) in `prototypes/<feature>/`, skipping tests and production concerns. Reads existing UX/UI specs if available. |
| `/dev-team:review` | Run combined QA + Security review on existing code |

## Dependencies

| Plugin | Required | Purpose |
|--------|----------|---------|
| [pr-review-toolkit](https://github.com/anthropics/claude-code/tree/main/plugins/pr-review-toolkit) | Recommended | QA agent delegates specialized analysis (code review, error handling, test coverage, type design, comments, simplification) to toolkit agents. Without it, QA falls back to performing all analysis itself. |

## Quick Start

### Install

In a Claude Code session, run:
```
/plugin marketplace add rustamfi/agentic-teams
/plugin install dev-team@rustamfi
```

### Usage

**Full pipeline** — build command orchestrates all needed agents through phased execution:
```
Examples:
/dev-team:build Add a user authentication system with email/password and OAuth2 Google login
/dev-team:build Create a fully functional 2048 game for browser
```

**Individual agents** — invoke a specific role directly:
```
Examples:
/dev-team:pm Define requirements for a notification preferences page
/dev-team:architect Design the API for a real-time chat feature
/dev-team:backend Implement the REST endpoints for user profile management
/dev-team:qa Review the authentication module for edge cases and vulnerabilities
/dev-team:review Run QA + Security review on the payment processing code
/dev-team:prototype Design a dashboard for real-time analytics
```

## How the Build Pipeline Works

The `/dev-team:build` command orchestrates a phased pipeline with user-selected approval gates:

1. **Setup** — Creates project structure (`specs/`, `tasks/`), asks the user which approval gates to enable (specs, UX/UI, prototype, implementation, QA/security)
2. **Discovery & Specs** — Spawns `product-manager` for requirements (`specs/prd.md`), optionally `architect` for system design (`specs/architecture.md`)
3. **UX/UI Design** (if task has UI) — Spawns `ux-researcher` then `ui-designer` for design specs
4. **Prototype** (if task has UI) — Spawns `frontend` to build a clickable prototype in `prototypes/<feature>/`
5. **Implementation** — Spawns `backend` and/or `frontend`, in parallel when API contracts are defined. Verifies tests were created.
6. **Quality & Security** — Spawns `qa` and `security` in parallel. QA delegates to pr-review-toolkit agents for deep analysis. Cross-validates findings against project docs and architecture specs before applying fixes. Up to 3 fix iterations.
7. **Runtime Validation** — Starts the dev server and smoke-tests key routes (a passing build alone is not sufficient). Produces a final traceability summary mapping requirements → implementation → tests.

Each phase pauses for user approval if the corresponding gate was selected in step 1.

### Team Manager Role

The team manager agent is a **routing advisor**. When the build command encounters ambiguous routing decisions, it consults the team manager to assess the task and recommend which agents to invoke and in what order. The actual orchestration, gating, and iteration logic lives in the build command itself.

## Project Structure

```
dev-team/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── agents/
│   ├── team-manager.md      # Routing advisor — recommends agent sequences
│   ├── product-manager.md   # Requirements and acceptance criteria
│   ├── ux-researcher.md     # UX research, personas, and journeys
│   ├── ui-designer.md       # UI component specs and visual design
│   ├── architect.md         # System design and API contracts
│   ├── backend.md           # Server-side implementation
│   ├── frontend.md          # Client-side implementation
│   ├── qa.md                # 3-phase QA: toolkit delegation → synthesis → polish
│   └── security.md          # Threat modeling and vulnerability review
├── skills/
│   └── team-workflow/
│       └── SKILL.md         # Auto-invoked context for team collaboration
├── commands/
│   ├── build.md             # Full orchestrated pipeline
│   ├── pm.md                # Product Manager
│   ├── ux.md                # UX Researcher
│   ├── design.md            # UI Designer
│   ├── architect.md         # Architect
│   ├── backend.md           # Backend Developer
│   ├── frontend.md          # Frontend Developer
│   ├── qa.md                # QA Engineer
│   ├── security.md          # Security Engineer
│   └── review.md            # Combined QA + Security review
└── README.md
```

## pr-review-toolkit Integration

**Optional dependency:** The QA agent leverages the [pr-review-toolkit](https://github.com/anthropics/claude-code/tree/main/plugins/pr-review-toolkit) plugin for deep specialized analysis. If pr-review-toolkit is installed, QA delegates specialized analysis to these agents:

| Toolkit Agent | Invoked When | Analysis Focus |
|---------------|-------------|----------------|
| `code-reviewer` | Always | Code quality, bug detection, guideline compliance (confidence ≥ 80) |
| `silent-failure-hunter` | Always | Silent failures, empty catches, broad error handling, unjustified fallbacks |
| `pr-test-analyzer` | Test files present | Behavioral test coverage gaps with criticality ratings |
| `type-design-analyzer` | New types introduced | Type invariants, encapsulation, enforcement quality |
| `comment-analyzer` | Docs/comments changed | Comment accuracy, rot detection, long-term value |
| `code-simplifier` | No critical issues found | Clarity and maintainability improvements (test-gated) |

QA synthesizes toolkit findings with its own integration/regression analysis into a unified report with attributed sources, enabling the build command's cross-validation step.

If [pr-review-toolkit] is not installed, QA gracefully falls back to performing the full review itself — no configuration needed.

## Customization

### Adjusting Agent Models

Model assignments are defined in the build command (`commands/build.md`) under the **Agent Model Assignment** section. The build command passes the `model` parameter when spawning each agent. To change which model an agent uses, update the table there.

### Adding Domain-Specific Context

Add skills in `skills/` for your specific domain. For example, if your team works on a fintech product, add `skills/fintech-context.md` with domain rules, compliance requirements, and architectural patterns.

### Extending with MCP Servers

Add a `.mcp.json` at the plugin root to give agents access to external tools:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "<token>" }
    }
  }
}
```

## Roadmap

- **Mobile app support** — React Native / Expo as the first target (shared React knowledge, CLI-friendly tooling). Native iOS/Android is lower priority due to limited CLI build/preview capabilities.
- **Desktop app support** — Electron, Tauri
- **Additional agent roles** — DevOps/Infrastructure, Database specialist

## License

MIT

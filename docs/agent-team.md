# Agent Team for Mona's Project Pulse Dashboard

Four custom agents live in `.github/agents/`. The Orchestrator coordinates; the others are specialists.

| Agent | Model | Tools | Role |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 5.5 | read, agent, memory | Breaks requests into phases, delegates with explicit file scopes, verifies the integrated result. Does not implement. |
| **Planner** | Claude Opus 5.5 | read, search, web, memory, todo | Researches the repo and docs, then returns steps, file assignments, dependencies, parallel/sequential work, edge cases, validation expectations and open questions. Writes no code. |
| **Designer** | Gemini 3.7 Flash | read, edit, search, web, memory, todo | UI/UX, accessibility, hierarchy and responsive layout. For Project Pulse: project cards, status badges, clear priority treatment, and CSS hooks `.dashboard` and `.project-card`. |
| **Coder** | GPT-5.5 | read, edit, search, execute, web, memory, todo | Implements logic and runnable-app support. For Project Pulse: `.vscode/launch.json` (strict JSON, `cwd` = `${workspaceFolder}/app`, opens `index.html`). Validates before reporting. |

## Workflow

1. Orchestrator asks the Planner for a plan.
2. Orchestrator splits the plan into phases by file ownership and dependencies.
3. Non-overlapping, independent tasks run in parallel; overlapping or dependent tasks run sequentially.
4. Designer and Coder work only within their assigned files.
5. Orchestrator verifies the result and reports to you.

## Shared rule

No agent stages, commits or pushes. You control all git operations.

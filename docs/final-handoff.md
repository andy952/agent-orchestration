# Project Pulse – Final Handoff

Project Pulse is a static dashboard that shows each team project's status, priority and recent activity.
This handoff summarises who built what, how to run it, the validation results and the open risks.
See also [docs/agent-team.md](agent-team.md) for the team setup and [docs/project-pulse-plan.md](project-pulse-plan.md) for the plan.

## Agent team

| Agent | Role in this build |
| --- | --- |
| **Orchestrator** | Coordinated the phases (plan → design and code → validation), assigned file scope and ran the integration checks. |
| **Planner** | Wrote the implementation plan in `docs/project-pulse-plan.md`. |
| **Designer** | Owned `app/styles.css` and made all visual and accessibility decisions. |
| **Coder** | Built `app/index.html`, `app/project-data.json` and `.vscode/launch.json`. |

## Deliverables

- `app/index.html`: the dashboard page. Its inline script fetches the project data and builds the project cards with `textContent` (no `innerHTML`). It also shows loading, empty and error messages.
- `app/styles.css`: the dashboard layout, project-card styling, status and priority badges, and responsive `@media` rules.
- `app/project-data.json`: sample data with a top-level `projects` array of 5 projects (name, owner, status, recentActivity, priority, and an optional summary).
- `.vscode/launch.json`: the strict-JSON launch configuration **"Run Project Pulse Dashboard"**, which serves `app/` and opens the dashboard.

## How to run

1. Open **Run and Debug** in VS Code (or the Codespace).
2. Select **"Run Project Pulse Dashboard"** and start it.
3. It runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`. When the server is ready, it opens `http://localhost:%s/index.html`, where `%s` is the port it detected (5500).

Opening `index.html` straight from disk (`file://`) shows an error message on purpose, because the data has to be served over HTTP.

## Validation results

Checks ran on Windows with Node. Python and Ruby are not installed, and bash is not on PATH (Git Bash is installed).

| # | Check | Outcome |
| --- | --- | --- |
| 1a | `app/project-data.json` parses strictly with `JSON.parse` | pass |
| 1b | It has a top-level `projects` array (5 items), and every project has name, owner, status, recentActivity and priority | pass |
| 1c | `.vscode/launch.json` parses strictly with `JSON.parse` and contains no comments | pass |
| 1d | launch name `Run Project Pulse Dashboard` | pass |
| 1e | launch cwd `${workspaceFolder}/app` | pass |
| 1f | launch command `python3 -m http.server 5500` | pass |
| 1g | `serverReadyAction.uriFormat` `http://localhost:%s/index.html` | pass |
| 2 | `app/index.html` has `<title>Project Pulse</title>` and references styles.css, project-data.json, project-card, status, recentActivity and priority; no `innerHTML` | pass |
| 3 | `app/styles.css` has `.dashboard`, `.project-card`, `border-radius`, `box-shadow` and `@media` | pass |
| 4a | Temporary Node static server on port 5597: `/` 200, `/index.html` 200, `/styles.css` 200, `/project-data.json` 200 | pass |
| 4b | `/` returns exactly the same content as `/index.html`, not a directory listing. The server was stopped afterwards. | pass |
| 5 | The inline script ran in Node's `vm` with a minimal fake DOM and a stubbed `fetch` that returned the real JSON. It created 5 `project-card` elements, and each card's text includes that project's status, recentActivity and priority. | pass |
| 6 | `bash scripts/validate-exercise.sh` via Git Bash (`C:\Program Files\Git\bin\bash.exe`) | fail (environment): exited 127 at line 45 with `ruby: command not found` before any project checks ran. Rerun in the Codespace. |

The Node static server stands in for `python3 -m http.server`. That means the launch configuration itself has **not** been run end to end.

## Handoff notes

### Known risks

- **python3 is not available on the local Windows machine.** It is available in the Codespace, so run the launch configuration there, or install Python locally.
- **Port 5500 may already be in use** (for example, by Live Server). If it is, stop the other process or change the port in `.vscode/launch.json`.
- **`serverReadyAction` with the `node-terminal` type is unconfirmed.** The browser may not open automatically. Fallbacks:
  - Use `python3 -u -m http.server 5500` so the "port" line is not buffered.
  - Switch to the `debugpy` launch type.
  - Open `http://localhost:5500/index.html` manually.
- **No human visual or browser review has been done yet.** Rendering was checked only with a fake DOM.

### Recommended manual checks

- Desktop width and 375px mobile width: cards reflow, long project names and activity text wrap, and nothing scrolls sideways.
- Keyboard focus: tab order is logical and focus is visible.
- Empty state: set `"projects": []` and confirm "No active projects yet." appears.
- Error states: rename or break `project-data.json`, and open the page via `file://`. Confirm a readable error message appears.

### Git state

- `main` matches `origin/main` at `1bcbc9b`, with `docs/agent-team.md`, `docs/project-pulse-plan.md`, `app/*` and `.vscode/launch.json` committed and pushed.
- The only uncommitted file is `docs/final-handoff.md`.
- The learner controls all git operations. No agent has staged, committed or pushed this file.

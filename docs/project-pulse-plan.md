# Project Pulse Dashboard – Implementation Plan

## Summary

Project Pulse is a small static dashboard for Mona's team. It shows each project's name, owner, status, recent activity and priority as polished, responsive cards. It lives in `app/` and runs from VS Code using the **Run Project Pulse Dashboard** launch configuration. That configuration serves `app/` with `python3 -m http.server 5500` and opens `http://localhost:%s/index.html`, so contributors see the dashboard rather than a directory listing.

The team follows `.github/agents/` and `docs/agent-team.md`:

- **Orchestrator** coordinates the work and checks the result. It writes no code.
- **Planner** wrote this plan. It writes no code.
- **Designer** owns the visual layer: `app/styles.css`.
- **Coder** owns markup, data loading, data and run setup: `app/index.html`, `app/project-data.json` and `.vscode/launch.json`.

No agent stages, commits or pushes. The learner controls all git operations.

Repo facts that shaped this plan:

- `app/` does not exist yet, and `.vscode/` only contains `tasks.json`. Every file in this plan is new.
- The step 3 workflow only checks file contents for exact phrases (listed under Validation expectations). It never runs the app.
- `project-data.json` and `launch.json` must both pass `python3 -m json.tool`.

## File assignments

| File | Owner | Responsibility |
| --- | --- | --- |
| `app/index.html` | **Coder** | Page structure (`<main class="dashboard">`), exact title "Project Pulse", link to `styles.css`, inline script that uses `fetch('project-data.json')`, renders one `.project-card` per project, and shows `name`, `owner`, `status`, `recentActivity` and `priority`. Also handles loading, empty and error states. |
| `app/styles.css` | **Designer** | Polished dashboard look: `.dashboard` and `.project-card` selectors, `border-radius`, `box-shadow`, status badges, priority treatment, readable spacing and typography, responsive grid, focus styles, and styles for the empty and error states. |
| `app/project-data.json` | **Coder** | Strict JSON with a top-level `projects` array. Each project has `name`, `owner`, `status`, `recentActivity` and `priority`. Include 4–6 sample projects. Optional extra field: `summary`. |
| `.vscode/launch.json` | **Coder** | Strict JSON with no comments. One configuration named "Run Project Pulse Dashboard". It runs `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`, and its `serverReadyAction` opens `http://localhost:%s/index.html`. |
| `docs/project-pulse-plan.md` | Planner output, saved by the Orchestrator | This document |

Each file has exactly one owner, so no two agents ever edit the same file.

## Designer responsibilities

- Own `app/styles.css` only.
- Create a polished dashboard, not a bare HTML page: visible project cards, status badges, clear priority treatment, readable spacing and a responsive layout.
- Provide the `.dashboard` and `.project-card` CSS hooks, with rounded corners, shadows, contrast and clear typography.
- Style every class in the shared contract, including the `unknown` fallbacks and the loading, empty and error messages.
- Keep status badges readable without relying on color alone (WCAG AA contrast, at least 4.5:1).
- Respect `prefers-reduced-motion` and provide visible `:focus-visible` outlines.
- Report design decisions, files touched and validation recommendations.

## Coder responsibilities

- Own `app/index.html`, `app/project-data.json` and `.vscode/launch.json`.
- Build the page structure from the shared contract and load data with `fetch('project-data.json')`.
- Render cards with `document.createElement` and `textContent` (never `innerHTML`).
- Handle loading, empty, error and missing-field states.
- Create sample data covering every status and priority, plus long text cases.
- Create `.vscode/launch.json` as strict JSON with `cwd` set to `${workspaceFolder}/app`, opening `index.html`.
- Run the validation checks below and report what changed, what was validated and any remaining risk.

## Shared contract (agree on this before implementation)

Designer and Coder both depend on the same class names. The Orchestrator should include this contract when delegating to each of them, so they can work in parallel without editing each other's files.

**Page structure (written by Coder):**

- `<body>` contains `<main class="dashboard">`.
- `<header class="dashboard-header">` holds `<h1>Project Pulse</h1>` and a short subtitle `<p class="dashboard-subtitle">`.
- `<section class="project-grid" aria-live="polite">` is the container for the cards.
- Each card is `<article class="project-card" data-priority="high|medium|low" data-status="<slug>">` and contains:
  - `<h2 class="project-name">`
  - `<p class="project-owner">`
  - `<span class="status-badge status-<slug>">`
  - `<span class="priority-badge priority-<level>">`
  - `<p class="project-activity">` (the `recentActivity` value)
  - optionally `<p class="project-summary">`
- State messages use `<p class="dashboard-message dashboard-message--loading|--empty|--error" role="status">`.

**Data values (written by Coder):**

- Status values: `On track`, `At risk`, `Blocked`, `Completed`. Turn them into slugs `on-track`, `at-risk`, `blocked`, `completed`. Any other value becomes `unknown`.
- Priority values: `High`, `Medium`, `Low`. Turn them into slugs `high`, `medium`, `low`. Any other value becomes `unknown`.

**Styles (written by Designer):** style every class above, including the `unknown` fallbacks.

## Ordered implementation steps

1. **Step 1 – Approve the plan and contract (Orchestrator).** Confirm the shared contract and settle the open questions, or accept the defaults given there.
2. **Step 2a – Data file (Coder).** Create `app/project-data.json` with the top-level `projects` array. Include a mix of statuses and priorities, at least one long `recentActivity` string, and one long project name so long text gets tested.
3. **Step 2b – Launch configuration (Coder).** Create `.vscode/launch.json`. Recommended shape:
   - `"version": "0.2.0"`
   - one configuration with:
     - `"type": "node-terminal"`
     - `"request": "launch"`
     - `"name": "Run Project Pulse Dashboard"`
     - `"command": "python3 -m http.server 5500"`
     - `"cwd": "${workspaceFolder}/app"`
     - `"serverReadyAction"` containing:
       - `"pattern": "port ([0-9]+)"`
       - `"uriFormat": "http://localhost:%s/index.html"`
       - `"action": "openExternally"`
   - No comments and no trailing commas.
4. **Step 2c – Styles (Designer).** Create `app/styles.css` from the shared contract:
   - **Base styles:** CSS custom properties for colors and spacing, a system font stack, and a soft page background.
   - **`.dashboard`:** max-width container, centered, with generous padding.
   - **Header:** clear hierarchy for the title and subtitle.
   - **`.project-grid`:** CSS grid using `repeat(auto-fill, minmax(280px, 1fr))`, which collapses to one column on narrow screens. Add a media query for screens of 600px and narrower.
   - **`.project-card`:**
     - white surface, `border-radius` (about 12px) and a layered `box-shadow`
     - hover and focus-within lift
     - colored left edge or top border keyed to `data-priority`
   - **Status badges:** pill shape. Color must not be the only signal; the text label must remain visible. Contrast must be at least 4.5:1 (WCAG AA).
   - **Priority treatment:** High is strongest (e.g. red or orange accent plus bold label), Medium is amber, Low is muted. Also style `unknown`.
   - **Long text:** use `overflow-wrap: anywhere` so long names and activity text wrap. Optionally clamp long activity text to a few lines.
   - **Message states:** style the loading, empty and error messages.
   - **Accessibility:** respect `prefers-reduced-motion` for hover transitions, and add visible `:focus-visible` outlines.
5. **Step 2d – Page structure and data loading (Coder).** Create `app/index.html`:
   - **Head:** `<!DOCTYPE html>`, `lang="en"`, the viewport meta tag, `<title>Project Pulse</title>`, and `<link rel="stylesheet" href="styles.css">`.
   - **Body:** the static `.dashboard` structure from the contract, with a loading message as the starting state.
   - **Inline `<script>`:**
     - Call `fetch('project-data.json')` and check `response.ok`.
     - Parse the JSON and check that `projects` is an array.
     - Render the cards with `document.createElement` and `textContent`, not `innerHTML`, so data can't inject HTML.
     - Read the `status`, `recentActivity` and `priority` fields directly, using those exact key names in the code.
     - Turn status and priority into slugs as defined in the contract.
     - Show the empty message when there are no projects, and an error message that explains the problem when loading fails (see Edge cases).
   - **Script choice:** keep everything in one inline script. Don't use ES modules, so the page has no extra files to load.
6. **Step 3 – Integration review (Orchestrator, with Designer for visual QA).** Serve `app/` and check that the class names in the HTML match the CSS selectors. Review the dashboard at desktop and mobile widths. If the markup and styles need adjusting, make changes one after another: the Designer changes only CSS, and the Coder changes only HTML.
7. **Step 4 – Validation and report (Coder runs the checks, Orchestrator confirms).** Run the checks under Validation expectations and report the results. Do not stage, commit or push.

## Dependencies between steps

- Step 1 must finish before every other step, because the shared contract unblocks both agents.
- Step 2d (`app/index.html`) depends on the field names in Step 2a (`app/project-data.json`). Those names are already fixed by the brief, so 2d can start as soon as Step 1 is done.
- Step 2c (`app/styles.css`) depends only on the class names in the contract. It does not depend on the finished HTML.
- Step 2b (`.vscode/launch.json`) has no dependency on the app files' contents. It only assumes `app/index.html` will exist.
- Step 3 depends on all of Step 2.
- Step 4 depends on Step 3.

## Parallel vs sequential work decisions

**In parallel (Phase 2):**

- **Designer:** `app/styles.css`.
- **Coder:** `app/project-data.json`, `.vscode/launch.json` and `app/index.html`.

These can run at the same time because:

- The two agents' files don't overlap at all.
- The only link between them is the class-name contract, and that is settled in Step 1. No agent needs another agent's output.
- Inside the Coder's work, the order 2a → 2b → 2d is a suggestion, not a requirement. All three files belong to one agent, so there is no conflict.

**One after another:**

- **Phase 1 → Phase 2:** the contract has to exist first.
- **Phase 2 → Phase 3:** integration needs every file to exist.
- **Fixes in Phase 3:** if a mismatch spans both files (for example, a class is renamed), the Coder updates `app/index.html` first. Only then does the Designer update `app/styles.css`. The two edits never happen at the same time.
- **Phase 3 → Phase 4:** validation runs last, on the finished, integrated files.

**Never in parallel:**

- Two agents editing the same file.
- The Designer touching `app/index.html`, or the Coder touching `app/styles.css`, unless the Orchestrator explicitly reassigns that file.

## Edge cases to handle

1. **Opening the page directly from disk (`file://`).** Most browsers block `fetch` from a `file://` page, so the data can't load. The page should detect this (`location.protocol === 'file:'`) or catch the error, and show a clear message such as: "Open this dashboard with the Run Project Pulse Dashboard launch configuration (it serves app/ over http)." It must not fail silently or leave the loading message stuck on screen. An optional fallback is to embed the data inline in a `<script type="application/json">` block. That is not the default because it would duplicate the data (see Open questions).
2. **Fetch fails** (404 or a network error): show an error message that includes the HTTP status.
3. **Invalid JSON:** catch the parse error and show "Project data could not be read."
4. **Missing `projects` key, or `projects` is not an array:** treat it as an error, with a message that names the expected `projects` key.
5. **Empty `projects` array:** show "No active projects yet." styled as the empty state.
6. **A project with a missing field:** show a fallback such as "Unassigned" or "No recent activity". Don't print `undefined`.
7. **Unknown status or priority value:** use the `unknown` slug so the badge still has neutral styling.
8. **Long text** (project name, owner or activity): wrap it with `overflow-wrap: anywhere` so it never stretches the card. Clamping long activity text is optional.
9. **Special characters or HTML in the data:** render with `textContent` only.
10. **Many projects:** the grid wraps, with no fixed heights.
11. **Small screens (320px wide):** one column with no horizontal scrolling.
12. **Accessibility:**
    - The status and priority badges carry text labels; color is not the only signal.
    - Message states use `role="status"` or `aria-live` so screen readers announce them.
    - Headings run in order: h1, then h2 for each card.
13. **Launch setup:**
    - Port 5500 may already be in use. Report this as a known limitation.
    - `python3` may be missing on a local Windows machine. In the Codespace, the `universal:2` image includes it.
    - The server must start from `app/` so the root URL serves the dashboard rather than a directory listing. `uriFormat` points straight at `/index.html` as well.
14. **`launch.json` must be strict JSON:** no comments and no trailing commas, so `python3 -m json.tool` parses it.

## Validation expectations

**Automated checks (these mirror `.github/workflows/3-step.yml`; the Coder runs them and reports the results):**

- `python3 -m json.tool app/project-data.json` succeeds.
- `python3 -m json.tool .vscode/launch.json` succeeds.
- `app/index.html` contains: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity` and `priority`.
- `app/styles.css` contains: `.dashboard`, `.project-card`, `border-radius` and `box-shadow`.
- `app/project-data.json` contains a top-level `projects` key, and every project has `name`, `owner`, `status`, `recentActivity` and `priority`.
- `.vscode/launch.json` contains `Run Project Pulse Dashboard` and `index.html`, sets `cwd` to `${workspaceFolder}/app`, runs `python3 -m http.server 5500`, and has `serverReadyAction` with `http://localhost:%s/index.html`.
- `bash scripts/validate-exercise.sh` still passes the repo checks it covers. Note: that script checks that learner files are *not tracked* in the template. That check is about git state, not about our files.

**Manual checks:**

1. In Run and Debug, start **Run Project Pulse Dashboard**. The browser should open `/index.html`, not a directory listing.
2. Every project in the JSON appears as a card showing its status badge, priority treatment, owner and recent activity.
3. At about 375px wide there is one column and no horizontal scrolling. At desktop width there are several columns.
4. Temporarily rename the data file or empty the `projects` array. The error or empty message should appear. Undo the change afterwards.
5. Open `app/index.html` directly from disk. The `file://` guidance message should appear.
6. Tab through the page. Focus outlines are visible, and the badges are readable without relying on color.
7. Stop the preview server when finished.

**Reporting:** each agent reports the files it touched, what it checked and any remaining risk. The Orchestrator confirms the integrated result. Nobody stages, commits or pushes.

## Open questions

1. **Debugger type.** The plan uses `node-terminal` (the built-in JavaScript debugger), which runs a shell command and supports `cwd` and `serverReadyAction`. The VS Code docs show `serverReadyAction` with a `node` example and do not say outright that it works with `node-terminal`. If it doesn't fire, fall back to the `debugpy` Python debugger: `"type": "debugpy"`, `"module": "http.server"`, `"args": ["5500"]`, same `cwd` and `serverReadyAction`. That needs the Python extension, which is not in the devcontainer's extension list.
2. **Output buffering.** `serverReadyAction` waits for Python to print "Serving HTTP on … port 5500". In a terminal that output appears immediately. If the browser doesn't open, try `python3 -u -m http.server 5500`. That differs slightly from the exact command in step 3.
3. **`file://` fallback.** Is a clear guidance message enough, or should the data also be embedded inline in the HTML so the page works from disk? That would mean keeping two copies of the data in sync. Default: message only.
4. **Optional `summary` field.** The brief asks for "a short contributor-friendly summary". Should each project carry an optional `summary` field? Default: yes, as an optional extra beyond the five required fields.
5. **Status and priority vocabulary.** Are the proposed values (`On track`, `At risk`, `Blocked`, `Completed` and `High`, `Medium`, `Low`) acceptable to Mona's team?
6. **Local Windows use.** This repo is also checked out on a Windows machine, where `python3` may not be available (`py` or `python` instead). The Codespace is the main target. Should a Windows-specific override be added in `launch.json`? Default: no.

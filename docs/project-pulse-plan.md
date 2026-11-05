# Project Pulse Implementation Plan

## Goal

Build a static Project Pulse dashboard for the Mona team. It will display project name, owner, status, recent activity, and priority from JSON data. The dashboard must run from VS Code using **Run Project Pulse Dashboard**.

## Scope and Ownership

| Area | Owner | File |
| --- | --- | --- |
| Team and agent prerequisite | Coder | `docs/agent-team.md` |
| Implementation plan | Planner, recorded by Orchestrator/Coder | `docs/project-pulse-plan.md` |
| Project data | Coder | `app/project-data.json` |
| Markup and rendering | Coder | `app/index.html` |
| Visual design and responsive layout | Designer | `app/styles.css` |
| Local launch configuration | Coder | `.vscode/launch.json` |
| Final handoff | Orchestrator | `docs/final-handoff.md` |

No backend, API, framework, build pipeline, or external CDN is required.

## Phases and Dependencies

1. **Prerequisite:** verify that `docs/agent-team.md` exists and documents Orchestrator, Planner, Coder, Designer, their models, `.github/agents`, and Project Pulse. Create it if the exercise's first step is incomplete.
2. **UI/data contract:** agree on the card structure, CSS class names, data fields, status values, priority values, and accessibility requirements.
3. **Implementation:** after the contract is fixed, work in parallel where files do not overlap:
   - Designer: `app/styles.css`.
   - Coder: `app/project-data.json`, then `app/index.html`.
   - Coder: `.vscode/launch.json`.
4. **Integration review:** Designer reviews hierarchy, accessibility, and responsive behavior; Coder reviews data loading, rendering, error states, and launch behavior. Changes to `index.html` and `styles.css` may proceed in parallel because they are separate files.
5. **Handoff:** create `docs/final-handoff.md` with completed files, validation results, limitations, and follow-up work.

The workflow remains sequential between phases because the exercise checks are chained. Within implementation, `styles.css` can proceed in parallel with data and launch configuration; `index.html` depends on the data contract.

## UI and Data Contract

`project-data.json` contains a top-level `projects` array. Every project has non-empty string fields:

- `name`
- `owner`
- `status`
- `recentActivity`
- `priority`

Use the statuses `On track`, `At risk`, `Blocked`, and `Completed`, and priorities `High`, `Medium`, and `Low`, unless the brief defines another controlled vocabulary.

Each project renders as an `article.project-card` with a heading, owner, status badge, priority badge, and recent activity. Required layout hooks include `.dashboard`, `.dashboard__header`, `.dashboard__grid`, `.project-card`, and `.badge` plus status/priority modifiers.

The page must use `lang="en"`, semantic `main`, readable text labels in badges, keyboard-visible focus states, and sufficient contrast. Data values must be inserted with DOM APIs and `textContent`, never data-bearing `innerHTML`.

## Implementation Details

- `app/index.html` includes the title and heading **Project Pulse**, links `styles.css`, and loads `project-data.json` with `fetch`.
- The page handles loading, an empty project list, failed HTTP responses, invalid JSON, and `file://` usage with visible messages.
- Unknown status or priority values receive a neutral class while preserving their text.
- `app/styles.css` provides a responsive grid using `repeat(auto-fill, minmax(280px, 1fr))`, a mobile rule at 600px or below, `border-radius`, `box-shadow`, `overflow-wrap`, `:focus-visible`, and reduced-motion handling when transitions are used.
- `.vscode/launch.json` uses strict JSON and launches `python3 -m http.server 5500` from `${workspaceFolder}/app` with a `serverReadyAction` opening `index.html`.

## Validation

Run before handoff:

1. `python3 -m json.tool app/project-data.json`
2. `python3 -m json.tool .vscode/launch.json`
3. A small Python check confirming `projects` is non-empty and all required fields are non-empty strings.
4. Repository checks for required references: Project Pulse, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, and `priority`.
5. Smoke-test the HTTP server and fetch both `index.html` and `project-data.json`.
6. Confirm `index.html` does not use `innerHTML` for project data.
7. Manually test VS Code launch, keyboard navigation, 375px and 1280px widths, card rendering, empty/error states, and the absence of horizontal scrolling.

## Risks and Open Questions

- The exercise's workflow may be blocked until the first-step `agent-team.md` artifact is present on the expected branch.
- Port 5500 may already be occupied; record this limitation rather than silently changing the required launch contract.
- Confirm whether the brief requires a summary field or a status-count overview before adding either.
- Confirm whether project documentation should be English or bilingual; required English validation keywords must remain present either way.
- Do not use `scripts/validate-exercise.sh` as the application acceptance test unless its assumptions match the learner repository; it validates the exercise template rather than the completed dashboard.

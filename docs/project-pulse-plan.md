# Project Pulse Implementation Plan

## Summary

Build Mona’s lightweight, static **Project Pulse** dashboard for contributors. The repository has no existing app files or package/dependency manifest, so the implementation should remain dependency-free: HTML, CSS, browser-side JSON loading, and Python’s built-in HTTP server configured through VS Code.

The dashboard will show multiple projects with name, owner, status, recent activity, priority/risk treatment, and a contributor-friendly summary. It must use polished, responsive project cards and be runnable through **Run Project Pulse Dashboard** without opening a directory listing.

## Responsibilities and File Ownership

| Role | Responsibility | Assigned files |
| --- | --- | --- |
| Designer | Own the visual system, information hierarchy, accessibility decisions, responsive layout, card/badge/priority treatments, and CSS implementation. Provide a class/markup contract to Coder before markup is finalized. | `app/styles.css` |
| Coder | Own application structure, data, browser-side rendering/error behavior, and runnable VS Code configuration. Implement the agreed design contract without changing design-owned CSS. | `app/index.html`, `app/project-data.json`, `.vscode/launch.json` |
| Orchestrator | Coordinate handoffs, prevent overlapping edits, integrate the files, run final validation, and report blockers. | No implementation files unless a post-integration correction is explicitly delegated. |

## Ordered Implementation Steps

1. **Confirm the implementation contract**
   - Orchestrator shares the brief and this plan with Designer and Coder.
   - Designer defines the required semantic structure and stable hooks, including `.dashboard`, `.project-card`, status badge classes, priority treatment, loading/error/empty-state styling, and responsive breakpoints.
   - Coder confirms the rendering approach: an inline or module script in `app/index.html` fetches `project-data.json` from the HTTP-served app directory and creates cards from `projects`.
   - **Files:** none changed during this decision step.
   - **Dependency:** must finish before Coder finalizes markup and before Designer finalizes selectors that depend on markup.

2. **Create project data and runnable preview configuration**
   - Coder creates `app/project-data.json` as valid JSON with a top-level `projects` array.
   - Add several representative projects. Every project must include `name`, `owner`, `status`, `recentActivity`, and `priority`; add `summary` to meet the contributor-friendly-summary requirement.
   - Coder creates `.vscode/launch.json` as strict JSON without comments. It must define **Run Project Pulse Dashboard**, run `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`, and use `serverReadyAction` to open `http://localhost:%s/index.html`.
   - **Files:** `app/project-data.json`, `.vscode/launch.json`
   - **Dependency:** data must be available before final data-driven rendering can be validated. The launch configuration can be authored independently of the data.

3. **Implement the semantic, data-driven dashboard**
   - Coder creates `app/index.html`.
   - Include the exact visible title **Project Pulse** and link `styles.css`.
   - Reference and load `project-data.json`, render one visible `.project-card` per project, and display the required `status`, `recentActivity`, and `priority` values alongside name, owner, and summary.
   - Use semantic structure and accessible labels: one page heading, headings for cards, readable label/value relationships, and non-color-only text for status and priority.
   - Include deterministic loading, empty, and fetch/JSON-error states. Do not silently leave a blank dashboard.
   - Keep all user-provided data text-safe when inserting content into the DOM.
   - **Files:** `app/index.html`
   - **Dependency:** requires the agreed Designer hook contract; rendering integration requires `app/project-data.json`.

4. **Implement the approved visual design**
   - Designer creates `app/styles.css` using the agreed semantic and class contract.
   - Build a responsive card-grid dashboard with readable spacing, high contrast, visible status badges, explicit priority/risk treatment, rounded corners, and shadows.
   - Include required selectors `.dashboard` and `.project-card`, plus `border-radius` and `box-shadow`.
   - Ensure keyboard focus remains visible and the layout works at narrow viewport widths without horizontal scrolling.
   - **Files:** `app/styles.css`
   - **Dependency:** can start after the class contract is agreed; final selector verification should occur after Coder completes `app/index.html`.

5. **Integrate and validate**
   - Orchestrator reviews all four files together and resolves only integration defects through the owning agent.
   - Start **Run Project Pulse Dashboard** and verify that it opens `index.html`, not the server root or a directory listing.
   - Check the dashboard at desktop and narrow/mobile widths and exercise normal, empty, and failed-data behavior where practical.
   - **Files:** review of `app/index.html`, `app/styles.css`, `app/project-data.json`, `.vscode/launch.json`
   - **Dependency:** strictly after steps 2–4 are complete.

## Dependencies

- The Designer’s hook/semantic contract is a prerequisite for stable, non-conflicting HTML and CSS.
- `app/index.html` depends on `app/project-data.json` having the `projects` schema.
- Data fetching depends on serving the app over HTTP; opening `index.html` directly as a file may fail due to browser fetch restrictions.
- The launch configuration depends on the `app/` directory and `index.html` existing by runtime.
- Final visual and runtime validation depends on all four implementation files being complete.

## Parallel and Sequential Work

### Can run in parallel

After the design contract is agreed:

- **Designer:** implement `app/styles.css`.
- **Coder:** create `app/project-data.json` and `.vscode/launch.json`.

These file scopes do not overlap.

### Must run sequentially

1. Designer’s class/semantic contract → Coder’s final `app/index.html` markup.
2. Coder’s data file → final data-rendering verification in `app/index.html`.
3. All implementation tasks → Orchestrator’s integration and launch validation.

Designer and Coder must not edit the same file. Any necessary cross-file change should be requested through the Orchestrator and assigned to the file owner.

## Edge Cases and Error States

- Missing, malformed, or non-array `projects` data.
- A valid but empty `projects` array.
- Fetch failure when the dashboard is opened outside the HTTP server.
- Project records with missing or blank required values; render a clear fallback rather than `undefined`.
- Unknown status or priority values; use a neutral visual fallback while preserving the original text.
- Long project names, owner names, summaries, or activity messages that could overflow cards.
- Status and priority must not rely on color alone.
- Narrow screens, zoomed text, keyboard navigation, and visible focus indicators.
- Port `5500` already in use; the user may need to stop the conflicting process before launching.
- The launch configuration must target `index.html`; otherwise Python’s server will show a directory listing.

## Validation Expectations

1. **File and syntax checks**
   - All four required files exist.
   - Run `python3 -m json.tool app/project-data.json`.
   - Run `python3 -m json.tool .vscode/launch.json`.
   - Confirm `launch.json` contains no comments and the named configuration is **Run Project Pulse Dashboard**.

2. **Required content checks**
   - `app/index.html` includes **Project Pulse**, references `styles.css` and `project-data.json`, and renders `.project-card` elements.
   - The rendered UI includes `status`, `recentActivity`, and `priority`.
   - `app/styles.css` includes `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`.
   - Every project record includes `name`, `owner`, `status`, `recentActivity`, and `priority`, under top-level `projects`.

3. **Runtime checks**
   - Launch **Run Project Pulse Dashboard** from VS Code.
   - Confirm Python serves from `${workspaceFolder}/app`.
   - Confirm the browser opens `http://localhost:<port>/index.html`, displaying the dashboard rather than a directory listing.
   - Confirm cards render from JSON and display all required project information.
   - Confirm loading/error/empty states are understandable.
   - Check responsive layout, readable contrast, keyboard focus, and non-color status/priority labels.

## Open Questions

- The brief requires contributor-friendly summaries but does not list `summary` among the mandatory JSON fields. This plan recommends adding it as an optional-but-used field to every project.
- No exact status or priority vocabulary is specified. The team should choose a small consistent set (for example, status: Active/At risk/Planned; priority: High/Medium/Low) and ensure unknown values render neutrally.
- No filtering, sorting, persistence, or project-detail interaction is in scope; these should remain future enhancements unless Mona requests them.

# Project Pulse implementation plan

## Goal and scope

Build Mona's Project Pulse as a polished, static dashboard that gives a quick
read on project health and still works at mobile widths. The initial
implementation should be deterministic and easy to preview locally: the page
loads project records from `app/project-data.json`, renders the dashboard from
those records, and uses `app/styles.css` for all presentation. No backend or
build step is required unless the existing runtime later demands one.

The Orchestrator coordinates the work and owns sequencing, scope, and
integration decisions. Planner researches and turns the requirements into an
implementation-ready contract. Designer owns UI/UX, accessibility, responsive
visual design, and the styling hooks needed by the implementation. Coder owns
the HTML structure, data-driven rendering behavior, validation, and runnable-app
support.

## Agent responsibilities and file assignments

| Agent | Responsibilities | Assigned files |
| --- | --- | --- |
| Orchestrator | Coordinate Planner, Designer, and Coder; resolve conflicts; enforce phase dependencies; integrate and review the final result. | Coordination only; no application-file edits |
| Planner | Research the repository and requirements; define the page information architecture, data contract, states, accessibility requirements, and acceptance checks. | Planning output; no application-file edits |
| Designer | Define the visual hierarchy, responsive layout, color/priority/status treatment, interaction affordances, focus states, and accessible component hooks. Implement the agreed visual system without changing application logic. | `app/styles.css` |
| Coder | Implement semantic page markup, load and render the JSON data, handle loading/error/empty states, preserve the agreed hooks, and configure the preview launch behavior. | `app/index.html`, `app/project-data.json`, `.vscode/launch.json` |

The documentation file `docs/project-pulse-plan.md` is the only file being
created in the planning task. During implementation, each agent must stay
within the assignments above; changes outside those files require an
Orchestrator decision.

## Implementation phases

### 1. Research and contract (Planner, then Orchestrator)

Planner reviews the repository, the custom agent instructions, and any
available runtime assumptions. The output must settle:

- The dashboard's information hierarchy and required states.
- The JSON shape and field meanings below.
- Semantic landmarks, heading order, keyboard behavior, and minimum contrast
  expectations.
- Whether the page can use a plain browser `fetch` or needs a local static
  server for JSON loading.
- Acceptance criteria for desktop, tablet, mobile, loading, empty, and error
  views.

The Orchestrator approves this contract before implementation begins. This
phase must be sequential because Designer and Coder need the same stable
structure and data names.

### 2. Visual and interaction design (Designer)

Designer translates the approved contract into the visual system in
`app/styles.css`. The design should make the first viewport unmistakably a
Project Pulse dashboard:

- A page shell with a clear title, short context line, and optional last
  updated indicator.
- A summary strip for aggregate counts such as total projects, on-track
  projects, at-risk projects, and blocked projects.
- A responsive project grid containing visible project cards.
- Each `.project-card` should expose project name, owner/team, status badge,
  priority treatment, progress, due date or health signal, and a concise
  summary.
- A useful empty state and a clear error state must be visually designed, not
  left to browser defaults.
- Use deterministic hooks including `.dashboard`, `.dashboard-header`,
  `.summary-grid`, `.summary-card`, `.project-grid`, `.project-card`,
  `.status-badge`, `.priority-badge`, `.progress-bar`, `.empty-state`, and
  `.error-state`.
- Use rounded corners, restrained shadows, readable spacing, and a strong
  type scale without relying on color alone to communicate status or
  priority.
- Define responsive breakpoints so cards collapse from a multi-column grid to
  one column, summary metrics remain readable, and no content requires
  horizontal scrolling.
- Include `:focus-visible` styles, reduced-motion behavior, sufficient color
  contrast, and touch-sized interactive controls if controls are added.

Designer may work in parallel with Coder once Phase 1 has approved the
contract, provided both use the same agreed hooks and field names. Coder can
build the semantic skeleton while Designer builds the stylesheet, but the
Orchestrator must resolve any markup or naming mismatch before integration.

### 3. Data and page implementation (Coder)

Coder creates the following files:

#### `app/project-data.json`

Use a top-level object so the contract can evolve without changing the
loading code:

```json
{
  "lastUpdated": "2026-09-15T00:00:00Z",
  "projects": [
    {
      "id": "project-pulse",
      "name": "Project Pulse",
      "summary": "A concise description of the current project focus.",
      "owner": "Mona",
      "team": "Product Experience",
      "status": "on-track",
      "priority": "high",
      "progress": 72,
      "dueDate": "2026-10-15",
      "nextMilestone": "Integrated dashboard review"
    }
  ]
}
```

`lastUpdated` is an ISO 8601 timestamp. Each project `id` is unique and
stable. `status` is one of `on-track`, `at-risk`, or `blocked`; `priority` is
one of `high`, `medium`, or `low`; and `progress` is an integer from 0 through
100. Dates use `YYYY-MM-DD`. The implementation should not assume a fixed
number of projects. If a field is unavailable, the UI should omit that detail
or use an explicit unavailable label rather than showing misleading data.

The sample dataset should include enough varied records to demonstrate every
status and priority style, progress extremes (including 0 and 100), long
project names, and realistic mobile wrapping. It must remain valid JSON with
no comments or trailing commas.

#### `app/index.html`

Build a semantic, accessible document with:

- `<!doctype html>`, language metadata, responsive viewport metadata, and a
  descriptive page title.
- A `<header>` containing the dashboard title and context.
- A `<main id="main-content" class="dashboard">` containing summary metrics and
  the project list/grid.
- A live status region for loading and actionable error messaging, without
  causing the entire page to be announced repeatedly.
- A template or clearly separated render target for project cards so card
  markup remains consistent for every record.
- Real headings in order (`h1` then section headings), meaningful labels for
  progress indicators, and text equivalents for status and priority colors.
- A `<noscript>` message if JavaScript is required to load and render the JSON.

The page script should fetch `project-data.json` relative to the app, validate
the expected top-level and record fields, calculate summary counts from the
records rather than hard-coding them, and render escaped text into the
document. It must show an explicit loading state while fetching, an explicit
error state when the request or contract validation fails, and an explicit
empty state when `projects` is an empty array. Do not silently substitute
invented data after a failed request. Keep behavior deterministic and avoid
external dependencies.

#### `.vscode/launch.json`

Provide strict JSON with no comments and a deterministic browser launch
configuration. The launch configuration must:

- Set `"cwd": "${workspaceFolder}/app"` so the app is served from the app
  directory.
- Open `index.html`, not a directory listing.
- Use a stable local preview command/port that can serve JSON over HTTP (for
  example, a simple static server already available in the environment).
- Make the configuration easy to select from VS Code and avoid requiring a
  separate manual URL edit.

If the repository provides an established preview command, reuse it. Otherwise
the Orchestrator should choose and document the available local static-server
command before Coder finalizes this file; opening the HTML through `file://`
is not sufficient because browser fetch behavior for JSON can fail.

Coder and Designer may proceed in parallel after the contract is approved, but
Coder's final render markup must be checked against Designer's hooks before
integration. The launch configuration is dependent on the actual app
directory and preview method, so it is finalized after the page/data layout is
known.

### 4. Integration and review (Orchestrator with Designer and Coder)

The Orchestrator integrates the four implementation files and checks that:

1. `index.html` references the stylesheet and data path correctly.
2. Every class used as a styling hook exists in the CSS, and CSS hooks are
   either used or intentionally reserved.
3. Every data field consumed by the renderer is present in the contract or
   guarded as optional.
4. Summary counts, status labels, priority labels, progress values, and dates
   are derived from the loaded data.
5. Loading, error, empty, and populated states do not overlap or leave stale
   content visible.
6. `.vscode/launch.json` opens the actual dashboard from `app` through HTTP.

Any mismatch is returned to the owning agent for correction. Integration is
sequential after the parallel design and coding work because it depends on
both the final markup and the final CSS/data contract.

## Dependency and parallelism summary

- **Must be sequential:** Planner contract -> Orchestrator approval ->
  implementation kickoff. The contract prevents conflicting data names and
  page assumptions.
- **Can run in parallel:** After approval, Designer can build
  `app/styles.css` while Coder builds `app/index.html` and
  `app/project-data.json`. They must not independently redefine hooks or the
  JSON schema.
- **Partially dependent:** Coder can draft `.vscode/launch.json` while
  implementing, but the final launch target and command depend on the actual
  app directory and available local server.
- **Must be sequential:** Final integration, browser review, and remediation
  happen after Designer and Coder report complete because they validate the
  combined result rather than isolated files.

## Validation expectations

Validation is performed by Coder for implementation details and by the
Orchestrator for the integrated result; Designer participates in visual and
accessibility review.

### Data and structure

- Parse `app/project-data.json` with a JSON parser and confirm the schema,
  unique IDs, valid enum values, ISO timestamp/date formats, and progress
  bounds.
- Serve `app` through the configured local HTTP preview and confirm the page
  loads `project-data.json` without a CORS or `file://` failure.
- Confirm the populated view renders every sample record and summary totals
  match the data.
- Exercise an empty project array, malformed JSON, a failed request, and
  missing optional fields; each must produce an understandable state without
  silently hiding the problem.

### Accessibility and responsive behavior

- Confirm one meaningful `h1`, logical heading order, landmarks, descriptive
  document title, keyboard-visible focus, and usable text alternatives for
  status, priority, progress, and dates.
- Check that status and priority remain distinguishable without color alone and
  that text/background contrast is readable.
- Check at narrow mobile, tablet, and desktop widths for clipped text,
  horizontal overflow, broken cards, unreadable metrics, and unusable touch
  targets.
- Verify reduced-motion preferences do not produce distracting transitions and
  that the layout remains usable with increased text size.

### Launch and integrated smoke test

- Start the `.vscode/launch.json` configuration and verify it uses
  `${workspaceFolder}/app`, serves the data file, and opens `index.html`.
- Confirm a clean refresh reproduces the same rendered dashboard.
- Inspect the browser console and network panel for failed resources,
  uncaught rendering errors, invalid JSON, or unexpected 404s.
- Review the final result against the Designer's hooks and the approved
  contract, then record any remaining limitation rather than masking it with
  fallback content.


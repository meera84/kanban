# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file IT PMO Kanban board, built as an internal demo/training tool. Each version is one self-contained HTML file, with CSS in one `<style>` and JS in one `<script>`:

- `index.html`: **version 1**, the original build. Don't redesign it.
- `v2/index.html`: **version 2**, the redesign made with the `frontend-design` skill, in light purple. Its footer links back to v1.
- `v3/index.html`: **version 3**, the latest. It's modelled on a reference screenshot from the user: deep green header, warm paper ground, mustard action colour. It adds a summary section and new features. New work goes here; its footer links to v2 and v1.

All versions share the same JS architecture (described below), so a behaviour fix usually belongs in every file.

## Hard constraints (from the original spec — keep them)

- Vanilla HTML/CSS/JS only: no frameworks, libraries, npm, bundler or build step.
- It must run by double-clicking `index.html` (`file://`), with no server.
- No external resources: no CDN, no web fonts, no image files. Icons are Unicode glyphs or inline SVG.
- No persistence. Don't use localStorage, sessionStorage, IndexedDB or cookies. Refreshing resets the board to the seed data on purpose, and the header note says so.
- The only network call is FormSubmit's AJAX endpoint.
- Branding: a plain "UOB IT PMO" text wordmark. Never add a real UOB logo or trademarks, and never imitate an official UOB system.
  - v1 uses the original corporate blue palette.
  - v2 uses a light purple palette, at the user's request. Its colours are all tokens at the top of `v2/index.html`, with matching dark-theme blocks.
  - v3 uses a green / paper / mustard palette, matching the user's reference design, and is also tokenised with dark blocks.
- No `alert()`/`confirm()`: validation errors appear inline and deletion is confirmed inline. No `!important` in CSS.

## Running / testing

There are no build, lint or test commands. Open `index.html` in a browser. Check drag and drop by hand, because simulated drag events aren't reliable.

## Deployment

GitHub Pages deploys through `.github/workflows/pages.yml` on every push to `main`. Its "Assemble site" step copies `index.html` to the site root, `v2/index.html` to `/v2/` and `v3/index.html` to `/v3/`. Any other file the pages need must be added to that step. In the repo's Settings → Pages, Source must be set to "GitHub Actions". The live site is served over https, so FormSubmit works there, unlike from `file://`.

## Architecture

- **One source of truth:** `state = { tasks, filters, nextId, ui }`.
  - `state.ui` holds view-only state: `confirmDeleteId`, `openMoveId`, `draggingId`, `focusAfterRender` and `submitting`.
  - A render never contains anything that isn't in `state`.
- **Render loop:**
  - Every change updates `state` and then calls `renderBoard()`.
  - `renderBoard()` does the following:
    - Filters with `applyFilters()`.
    - Rebuilds each column's `.card-list` from `renderCard()` HTML strings.
    - Updates the count badges, `renderSummary()` and the filter status.
    - Restores focus through the `state.ui.focusAfterRender` CSS selector.
  - Don't change card DOM anywhere else. The inline "Move ▸" row and "Delete? Yes / No" group are driven by `state.ui` too, not by toggling the DOM.
- **Escaping:** every user-supplied string passes through `escapeHtml()` before it goes into template HTML.
- **Columns:** the four columns are static markup, each a `section.column[data-status]`. Their order and names must match `STATUSES`. Dropdowns in the form and filters are filled from the reference arrays (`PROJECTS`, `CATEGORIES`, `PRIORITIES`, `STATUSES`). Priority and Status defaults use `defaultSelected`, so `form.reset()` goes back to Medium / Backlog.
- **Events:**
  - Delegated on `#board`: click `data-action` values `delete | cancel-delete | confirm-delete | toggle-move | move-to`, plus the native HTML5 DnD events.
  - On `document`: Escape and clicking outside an open Move row.
- **Dates:** kept as local `YYYY-MM-DD` strings, which compare correctly as text (see `isOverdue`). Seed due dates are offsets from today, so the demo always has overdue items.
- **IDs:** `nextTaskId()` returns `UOB-ITPM-####`, zero-padded, from `state.nextId`.
- **v2 layout** (the next four points apply to `v2/index.html` only; v1 uses a sidebar and summary tiles):
  - The lanes use CSS subgrid (`.board` rows `auto 1fr`, `.column` spans 2 rows) so column headers share a row track and card tops line up. Below 768px the lanes switch back to a flex stack.
  - `state.ui.flashId` marks the card that just moved or was added. `renderBoard()` gives it `.is-flash` once, then clears it.
  - On Blocked cards the description gets `.is-blocker` and shows up to 3 lines. Elsewhere it's clamped to 1 line.
  - Colours are theme tokens with light and dark blocks. Solid fills (`--header-bg`, `--action-bg`, `--danger-solid`) have their own tokens because `--ink` flips to a light colour in dark mode.
  - The Add Task form is a fixed slide-over drawer (`#add-panel`), opened and closed by toggling the `.is-open` class through `setPanelOpen()`. Escape closes it.
  - The header's status bar sets each segment's `flex-grow` to its count. Its colours share the `--st-*` tokens with the column rules.
  - Toasts sit bottom-left, or at the top on phones, so they never cover the drawer's submit button.
- **v3 additions** (`v3/index.html` keeps the v2 points above and adds these):
  - `renderBoard()` also renders the summary section from the filtered tasks, so filters apply to the summary too:
    - `renderSummary()`: the headline, "Needs attention", and the status bar with its legend.
    - `renderTimeline()`: due-date buckets from `DUE_BUCKETS`, for open tasks only.
    - `renderWorkstreams()`: the by-workstream table.
  - Clicks are delegated on `<main>` (`handleActionClick`). Besides the card actions, it handles:
    - `edit`, which opens the drawer in edit mode.
    - `jump` (timeline chip), which scrolls to the card, focuses it and flashes it.
    - `filter-project` (workstream row), which toggles the project filter.
  - The drawer serves both adding and editing. `state.ui.editingId` and `setEditorMode()` switch its title and buttons; in edit mode, "Clear form" becomes "Cancel". Edits don't send a FormSubmit email. When editing, an unchanged past due date passes validation.
  - `deleteTask()` keeps `state.ui.lastDeleted`, and the toast's Undo button calls `undoDelete()`. `showToast()` takes `{ actionLabel, onAction }`.
  - `exportCsv()` downloads the filtered tasks as a one-off file (not persistence). `csvCell()` prefixes values that start with `= + - @` so spreadsheets don't run them as formulas.
  - Pressing `n` opens a new task unless focus is in a form field.
- **Help dialog (all versions):** `scheduleWelcome()`, called at the end of `init()`, opens the native modal `<dialog id="welcome-dialog">` after `WELCOME_DELAY_MS` (10 s). It shows once per page load and gives the IT support hotline. `handleKeydown()` returns early while it's open, so Escape closes only the dialog.
- **Add Task flow (optimistic):**
  1. Validate with `validateTask()`, which returns `{field: message}`.
  2. `addTask()` puts the card on the board.
  3. Reset the form and show the success toast.
  4. Call `notifyNewTask()` in a try/catch, with the submit button showing "Sending…".
  5. On failure, keep the card and show the warning toast "Card added locally — email notification failed".

## FormSubmit

- Config: `FORMSUBMIT_ENDPOINT` at the top of the script is the only place the email address appears. While it still contains `YOUR_EMAIL`, `notifyNewTask()` throws without making a network call.
- Activation: the first submission sends a confirmation email to the address, and nothing is delivered until the link in it is clicked. Until then FormSubmit returns 200 with `success: "false"`, which the code treats as a failure.
- FormSubmit rejects AJAX calls from `file://` pages. Email only works when the page is served over http(s).

## Published Artifact copy

A de-branded, dark-mode-aware copy is published at https://claude.ai/artifact/6KtLt27Fg9yrhhhB7dfmsK. It has no UOB name and uses `ITPM-####` IDs. It isn't generated automatically. If `index.html` changes and the Artifact should match, redo the transformation:
- Strip the doctype and the html/head/body tags.
- Remove the UOB references.
- Add the dark-theme token blocks.

Artifact pages can't reach FormSubmit, so notifications always fail there.

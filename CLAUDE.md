# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file IT PMO Kanban board (`index.html`), built as an internal demo/training tool. The page's HTML, CSS (in one `<style>`) and JS (in one `<script>`) all live in that one file.

## Hard constraints (from the original spec — keep them)

- Vanilla HTML/CSS/JS only: no frameworks, libraries, npm, bundler or build step.
- It must run by double-clicking `index.html` (`file://`), with no server.
- No external resources: no CDN, no web fonts, no image files. Icons are Unicode glyphs or inline SVG.
- No persistence. Don't use localStorage, sessionStorage, IndexedDB or cookies. Refreshing resets the board to the seed data on purpose, and the header note says so.
- The only network call is FormSubmit's AJAX endpoint.
- Branding: a plain "UOB IT PMO" text wordmark in a corporate blue palette. Never add a real UOB logo or trademarks, and never imitate an official UOB system.
- No `alert()`/`confirm()`: validation errors appear inline and deletion is confirmed inline. No `!important` in CSS.

## Running / testing

There are no build, lint or test commands. Open `index.html` in a browser. Check drag and drop by hand, because simulated drag events aren't reliable.

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

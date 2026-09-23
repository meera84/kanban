# kanban

A single-file IT PMO Kanban board: one `index.html` with inline CSS and JavaScript, built as an internal demo and training tool. It needs no frameworks, build step or server. Open the file and the board works.

**Live demo:** https://meera84.github.io/kanban/

> Demo only. The board holds its state in memory, so refreshing the page resets it to the sample tasks. All sample data is fictitious, and the page is not an official system of any organisation.

## Features

- Four columns: **Backlog**, **In Progress**, **Blocked** and **Done**, each with a live count.
- Summary bar showing total tasks, the count per status and overdue tasks.
- An **Add task** form with inline validation. Fields are title, description (up to 500 chars), project/workstream, category, assignee, priority, due date and status.
- Auto-generated task IDs (`UOB-ITPM-0001`, `UOB-ITPM-0002`, …).
- Priority levels Low, Medium, High and Critical. Overdue cards are highlighted.
- Move cards by drag and drop, or with the keyboard-friendly **Move ▸** menu on each card.
- Delete a card with an inline **Delete? Yes / No** confirmation. The page never uses browser pop-ups.
- Filters for project, assignee (contains) and priority.
- Optional email notification for each new task through [FormSubmit](https://formsubmit.co/). The card is added straight away. If the email fails, a warning toast appears and the card stays.
- Sample tasks with due dates relative to today, so the demo always has overdue items.

## Run locally

Double-click `index.html`, or open it in any modern browser. No install or server is needed.

FormSubmit rejects requests from `file://` pages, so email notifications only work when the page is served over http(s), for example from the live GitHub Pages site.

## Configuration

Email notifications are off until you set a recipient. At the top of the `<script>` in `index.html`:

```js
const FORMSUBMIT_ENDPOINT = "https://formsubmit.co/ajax/YOUR_EMAIL@example.com";
```

Replace `YOUR_EMAIL@example.com` with the address that should receive notifications. While the placeholder is there, the page makes no network call and shows the "email notification failed" warning.

The first real submission makes FormSubmit send a confirmation email to that address. Nothing is delivered until the link in that email is clicked.

Setting the address publishes it in the page source, since the site is public.

## Deployment

The repo deploys to GitHub Pages through GitHub Actions (`.github/workflows/pages.yml`) on every push to `main`, and can also be run by hand from the Actions tab.

- The workflow publishes only `index.html`. If you add a file the page needs, add it to the workflow's **Assemble site** step.
- In the repository settings, **Settings → Pages → Source** must be set to **GitHub Actions**.

## Project structure

```
index.html                    The whole app: markup, <style> and <script>
.github/workflows/pages.yml   GitHub Pages deployment
CLAUDE.md                     Architecture notes and project constraints
```

## Constraints

- Vanilla HTML, CSS and JavaScript only. No frameworks, libraries, npm or build step.
- No external resources: no CDN, web fonts or image files. Icons are Unicode or inline SVG.
- No persistence (no localStorage, sessionStorage, IndexedDB or cookies).
- The only network call is FormSubmit's AJAX endpoint.
- All user input is HTML-escaped before rendering.
- Branding is a plain text wordmark. The page uses no real logos or trademarks.

See [`CLAUDE.md`](CLAUDE.md) for the architecture (state model, render loop, event handling).

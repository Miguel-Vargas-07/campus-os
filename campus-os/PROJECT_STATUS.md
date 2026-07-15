# PROJECT_STATUS — Campus OS

> **Purpose of this file:** if you are Claude reading this in a new session,
> this is the single source of truth for where the project stands. Read this,
> then `docs/ARCHITECTURE.md` and `docs/ROADMAP.md`, before writing any code.

## Owner & intent

- **Owner:** Miguel — CS freshman, busy schedule (classes, internship,
  workouts/sports, social life, faith). Learning web development; app is
  intentionally **vanilla JS** so it runs with zero install.
- **Goal:** a personal Notion-style workspace. Priorities: fast daily use,
  no accounts, data portability (JSON export/import).

## Version

- **v0.5** — July 14, 2026. Obsidian-style notes + color scheme gallery.
- v0.4 — July 14, 2026. Day/night/auto theme.
- v0.3 — July 14, 2026. Progress dashboard + task completion dates.
- v0.2 — July 14, 2026. UI modernization + Reflect + Faith views.
- v0.1 — July 14, 2026. Initial build.

## What is DONE (v0.5)

Everything from v0.1 (Today, Assignments, Habits, Notes, Settings,
localStorage + export/import, responsive, seed data), plus:

- [x] **UI refresh:** dark pine gradient sidebar with grouped nav
      (MAIN / INNER / SYSTEM), gradient active-nav pill, 14px card radius,
      soft shadows, pill chips, view-switch fade/rise animation,
      eyebrow labels on view headers
- [x] **Greeting hero on Today:** time-of-day greeting + name (set in
      Settings), date + week number, open/overdue/habit stats
- [x] **Reflect view:** one entry per day — mood (1–5 emoji), grateful,
      win of the day, free text; 14-day mood arc mini-chart; past-entries
      timeline; evening nudge on Today if today's entry is missing
- [x] **Faith view:** "verse of the day" hero card (rotates daily through
      saved verses by day-of-year), prayer list with active → answered
      states (answered kept as a record with gold chip), saved verses/quotes
      collection; serif type (Fraunces) reserved for scripture text
- [x] **Settings:** name field
- [x] SCHEMA_VERSION bumped to 2 with `migrate()` — v1 backups import cleanly
- [x] **Progress view (v0.3):** Daily / Weekly / Monthly / Yearly toggle;
      stat cards (completed this period + delta vs previous, previous period,
      best in range, open now); pure-SVG bar chart (14 days / 12 weeks /
      12 months / 5 years) with current-period bar highlighted
- [x] Tasks now record `completed` (YYYY-MM-DD) when marked done; cleared if
      un-done. SCHEMA_VERSION = 3; migrate() backfills old done tasks with
      their created date
- [x] Keyboard shortcuts now 1–8 (3 = Progress)
- [x] **Theme (v0.4):** day / night / auto (follows device), full dark token
      set under `html[data-theme="dark"]`, sidebar footer toggle button
      (cycles auto → day → night) + Settings selector, live response to OS
      theme changes when on auto. SCHEMA_VERSION = 4 adds `settings.theme`
- [x] **Obsidian-style notes (v0.5):** [[wikilinks]] clickable in preview
      (click creates the note if missing), #tags parsed from body with
      tag-filter chips, search box over title+body, backlinks ("Linked
      mentions") panel, markdown-lite preview (# ## ### headings, **bold**,
      `inline code`, - lists) on top of existing ``` code fences
- [x] **Color scheme gallery (v0.5):** VS Code-style themes — Field Notes,
      Midnight Pine, Paper White, Solar, Fjord, Nocturne, Ember + Auto.
      SCHEMES object → inline CSS vars on <html>; data-theme still drives
      dark component overrides. SCHEMA_VERSION = 5: settings.theme →
      settings.scheme (auto/field/pine mapping)

## What is NOT done (see docs/ROADMAP.md)

- [ ] Internship application tracker  ← suggested next task
- [ ] Classes as their own database
- [ ] Calendar view / kanban view of assignments
- [ ] Recurring habit targets (e.g. lift MWF only)
- [ ] Search, markdown in notes, dark mode
- [ ] Optional React migration (roadmap phase 3)

## Conventions future sessions must follow

1. **Keep it one file** (`app/index.html`) until the roadmap's React phase.
2. All state lives in the single `state` object; every mutation goes through
   `save()` then `render()`. No other state containers.
3. Follow the design tokens in `docs/DESIGN.md` — do not add new colors/fonts.
4. Data model changes must bump `SCHEMA_VERSION` and extend `migrate()`.
5. After any work session, update this file's DONE/NOT DONE lists and version.

## Last session summary

Session 5 (same day): Obsidian-lite notes (wikilinks, tags, search,
backlinks, markdown-lite) and 7-scheme color gallery (schema v5,
settings.scheme). Also discussed AI integration with Miguel: plan of record
is an optional "AI helper" using the Anthropic API directly from the app
(user pastes their own API key, stored in localStorage, direct browser
calls) — fine for personal use, NOT acceptable if the app is ever published
for others (would need a small proxy backend / Supabase edge function).
Suggested model: Haiku 4.5 for cheap summarize/flashcard features.
Login decision from session 4 stands: local-first; Supabase phase 2.5 if
sync is ever wanted.
Next suggested task: **internship application tracker** (still roadmap #1),
or the AI helper if Miguel gets an API key.

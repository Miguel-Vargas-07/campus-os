# PROJECT_STATUS — Campus OS

> **Purpose of this file:** if you are Claude reading this in a new session,
> this is the single source of truth for where the project stands. Read this,
> then `docs/ARCHITECTURE.md` and `docs/ROADMAP.md`, before writing any code.

## Deployment

- **Live:** https://miguel-vargas-07.github.io/campus-os/ — GitHub Pages,
  served from `main` branch root. Repo is now **PUBLIC** (was private; free
  plan can't do Pages on private repos — Miguel approved going public
  July 15, 2026). A root `index.html` redirects to `campus-os/app/index.html`
  (the app sits one folder deep, so the bare Pages URL needs the bounce).
- localStorage is still per-device; hosting doesn't add sync. Export/import
  and friend codes remain the portability story. This unblocks the PWA
  roadmap item (manifest + service worker + installable to home screen).

## Owner & intent

- **Owner:** Miguel — CS freshman, busy schedule (classes, internship,
  workouts/sports, social life, faith). Learning web development; app is
  intentionally **vanilla JS** so it runs with zero install.
- **Goal:** a personal Notion-style workspace. Priorities: fast daily use,
  no accounts, data portability (JSON export/import).

## Version

- **v0.14** — July 15, 2026. Money view (schema 11).
- v0.13 — July 15, 2026. Focus timer (schema 10).
- v0.12 — July 15, 2026. Command palette (Ctrl+K).
- v0.11 — July 15, 2026. Assignments views: List / Board / Calendar.
- v0.10 — July 15, 2026. Weekly recap card.
- v0.9 — July 15, 2026. Recurring habit targets.
- v0.8 — July 15, 2026. Classes database.
- v0.7 — July 15, 2026. Friends circle + leaderboard (friend codes).
- v0.6 — July 15, 2026. Internship application tracker (kanban).
- v0.5 — July 14, 2026. Obsidian-style notes + color scheme gallery.
- v0.4 — July 14, 2026. Day/night/auto theme.
- v0.3 — July 14, 2026. Progress dashboard + task completion dates.
- v0.2 — July 14, 2026. UI modernization + Reflect + Faith views.
- v0.1 — July 14, 2026. Initial build.

## What is DONE (v0.14)

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
- [x] **Internship tracker (v0.6):** Internships view (nav MAIN, shortcut 6;
      Reflect/Faith/Settings now 7/8/9) — `apps` array (company, role,
      status, deadline, link, notes, created, updated), kanban columns
      wishlist → applied → OA → interview → offer / rejected, ◀ ▶ move
      buttons (clamped), click-card-to-edit via the top form (Update/Cancel),
      overdue deadline chip on non-terminal apps, seeded with 2 samples.
      SCHEMA_VERSION = 6: migrate() adds `apps: []`
- [x] **Friends circle (v0.7):** no-backend community leaderboard — trade
      "friend codes" (COS1- base64 stat snapshots; settings.shareId
      identifies you, re-pasting a code refreshes that friend), metric
      filter chips (tasks this week [default] / habit consistency /
      pipeline moves / reflection streak), bars scaled to group max,
      no rank numbers (community framing), YOU row live, STALE chip on
      codes >7 days old, per-friend copy-to-clipboard nudge messages,
      group-total line. Shortcut 7 (Reflect/Faith/Settings → 8/9/0).
      SCHEMA_VERSION = 7: migrate() adds `friends: []` + settings.shareId
- [x] **Classes database (v0.8):** `classes` array (name, code, color,
      schedule); tasks carry classId instead of free text; class select in
      the add-task form; class filter chips with color dots + open counts;
      inline "＋ class" create form with 6-swatch palette (in DESIGN.md);
      deleting a class unsets its tasks. SCHEMA_VERSION = 8: migrate()
      converts distinct task.cls strings into class objects
- [x] **Recurring habit targets (v0.9):** habit.days (null = daily, else
      day indexes, 0 = Mon); schedule chip per habit opens 7-day toggle
      editor; streaks skip rest days (missed target days still break);
      off-day cells dashed but clickable; Today lists only due habits with
      rest-day empty state; friends habit % counts scheduled days only.
      SCHEMA_VERSION = 9
- [x] **Weekly recap card (v0.10):** week-in-review hero on Progress
      (weekRecap(): tasks done + delta vs last week, habit % on target,
      best habit, avg mood, pipeline moves, reflection days); "Copy recap
      for your circle" text ends with your friend code; Sunday-evening
      nudge on Today. Pure derived data — schema unchanged (still 9)
- [x] **Assignments views (v0.11):** Notion-style lenses on one database —
      List / Board / Calendar tabs in the view header. Board = todo/doing/
      done kanban, click card to advance. Calendar = Mon-first month grid:
      task pills (class colors, done struck, overdue flagged) advance on
      click, internship deadlines as 🚀 pills (click → edit), empty-day
      click prefills the due date, ◀ Today ▶ month nav. Class filter works
      in every lens. No schema change
- [x] **Command palette (v0.12):** Ctrl/Cmd+K anywhere (or 🔍 sidebar
      button) — scored search across views, tasks, notes (title+body),
      internships, habits; ↑↓/Enter/Esc; empty query shows quick actions;
      last row always quick-adds the query as a task. Covers roadmap
      "search across tasks". No schema change
- [x] **Focus timer (v0.13):** pomodoro sprints on Today — optional task,
      15/25/50m presets, WebAudio chime, sidebar countdown chip in every
      view, "End early" logs whole elapsed minutes (<1m logs nothing).
      Sessions in `focus` array; focus minutes in Today hero + weekly
      recap card/copy text. SCHEMA_VERSION = 10: migrate() adds `focus: []`
- [x] **Money view (v0.14):** from Miguel's notes — expense/income log
      (amount, emoji category, note, date), monthly budget set inline in
      its stat card with progress bar (red when over), income/net + top
      category stats, 6-month spending SVG chart, category filter chips,
      per-month tx list with delete. Spend-this-week on the recap CARD
      only — never in the shared recap text (money stays private).
      No digit shortcut (1–0 taken); sidebar or palette. SCHEMA_VERSION
      = 11: migrate() adds `money: {budget:0, txs:[]}`

## What is NOT done — NEXT UP: follow docs/BUILD_PLAN.md

**`docs/BUILD_PLAN.md` is the authoritative spec for the next five
versions** (written July 15 2026 from competitive research; implement in
order, one version per commit, browser-verified):

- [ ] v0.15 — Natural-language quick add (no schema change)
- [ ] v0.16 — Schedule-aware Today "plan my day" (schema 12) ← signature feature
- [ ] v0.17 — Grades + what-do-I-need-on-the-final (schema 13)
- [ ] v0.18 — Flashcards, spaced repetition from notes (schema 14)
- [ ] v0.19 — PWA: installable + offline (adds manifest/sw.js/icons files)

After those (see docs/ROADMAP.md):
- [ ] AI helper (roadmap 7b) — recap writer / note summarizer / flashcard generation
- [ ] Notes extras: tables, images (roadmap 7)
- [ ] Optional React migration (roadmap phase 3)

## Conventions future sessions must follow

1. **Keep it one file** (`app/index.html`) until the roadmap's React phase.
2. All state lives in the single `state` object; every mutation goes through
   `save()` then `render()`. No other state containers.
3. Follow the design tokens in `docs/DESIGN.md` — do not add new colors/fonts.
4. Data model changes must bump `SCHEMA_VERSION` and extend `migrate()`.
5. After any work session, update this file's DONE/NOT DONE lists and version.

## Last session summary

Session 8 (July 15, 2026, same sitting as session 7): Miguel's idea notes
(`docs/CAPMUS-OS NOTES.txt` — filename typo kept at his call: dashboard,
AI agent, Google services, finances, blog/community) were committed and
pushed, and the five ideas were triaged into ROADMAP.md. Asked to pick
and build the best one now, Claude chose **finances** (local-first fit,
no API key or backend needed, testable today; AI agent stays next-up once
Miguel has a key) and shipped **v0.14 Money view** (schema 11): expense/
income log with emoji categories, inline monthly budget with over-budget
bar, 6-month spending chart, category filters, tx delete. Verified live
in browser: v10→v11 migration, form adds + junk rejection ($-5, "abc"),
budget math ($18.49 spent / $81.51 left / 18% bar), over-budget flip
(capped 100% red bar, "$13.49 over"), category filter, delete, palette
"Go to Money", fresh-seed shape, zero console errors. Privacy decision:
spend shows on the recap card but is excluded from the shareable recap
text. v0.11–v0.13 commits from session 7 were pushed to GitHub this
session too.
Next suggested task: **AI helper** (roadmap 7b) once Miguel has an
Anthropic API key — recap writer / note summarizer are ready surfaces.

Session 7 (July 15, 2026): Miguel asked Claude to take inspiration from
similar apps and build creatively. Three features, three commits, all
verified in a live browser (served over localhost; migrations 9→10, board
click-to-advance, calendar month nav + 🚀 deadline pills + day-click
quick-add, palette search/quick-add/keyboard nav, full focus session
fast-forwarded to completion, early-stop under 1 minute logs nothing):
- **v0.11 Assignments lenses** (Notion-inspired): List / Board / Calendar
  tabs — closed roadmap #3 and #4 at once.
- **v0.12 Command palette** (Linear/Raycast-inspired): Ctrl+K search across
  everything + quick-add task — closed roadmap #6.
- **v0.13 Focus timer** (Forest/Pomofocus-inspired): task-linked sprints,
  chime, `focus` log (SCHEMA_VERSION = 10), recap/hero integration.
Note: the body click listener now checks `data-del` before `data-cycle`
(board/calendar nest delete inside the cycle target) — preserve that order.
Commits are local; not yet pushed to GitHub.
Next suggested task: the **AI helper** (roadmap 7b) — the weekly recap and
note summaries are ready surfaces — or notes tables/images.

Session 6 (July 15, 2026): Project moved to a **private GitHub repo**
(github.com/Miguel-Vargas-07/campus-os, pushed via `gh` CLI — future
sessions can `git pull`/`git push` there; commit after each work session).
Built the **internship application tracker** (v0.6, schema 6): kanban
Internships view with move/edit/delete, overdue deadline chips.
Then built the **Friends circle** (v0.7, schema 7) — Miguel asked for a
friends/leaderboard/community feature; given the choice between friend
codes (no backend) and activating Supabase, he chose **friend codes now**,
and asked that the leaderboard be filterable by all four metrics with a
community feel rather than first-vs-last (tasks-this-week is the default
metric). Both versions verified in browser (migrations v5→v6→v7, code
round-trip, self/garbage code rejection, re-paste refresh, stale chip,
nudge clipboard, dark + light schemes). Docs updated each time.
Later the same session, Miguel picked three features over AI integration
(for now), built as three separate commits: **classes database** (v0.8),
**recurring habit targets** (v0.9), **weekly recap card** (v0.10). All
verified in browser; migrations chain v5→…→9 cleanly.
AI-helper plan of record from session 5 unchanged (own API key, Haiku 4.5,
personal use only) — the weekly recap is the natural first AI surface.
Login decision stands: local-first; Supabase phase 2.5 if sync is ever
wanted — the friend-code snapshot shape was designed to become the
Supabase sync payload if that day comes.
Next suggested task: **calendar view** (roadmap #3), or the AI helper if
Miguel gets an API key.

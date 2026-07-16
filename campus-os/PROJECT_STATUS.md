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

- **v0.16** — July 16, 2026. Schedule-aware Today / "plan my day" (schema 12).
- v0.15 — July 16, 2026. Natural-language quick add (schema 11, unchanged).
- v0.14 — July 15, 2026. Money view (schema 11).
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

## What is DONE (v0.16)

Everything from v0.1 (Today, Assignments, Habits, Notes, Settings,
localStorage + export/import, responsive, seed data), plus:

- [x] **Schedule-aware Today / "plan my day" (v0.16, signature feature):**
      classes carry `meetings` (day-of-week + start/end); a new vertical
      timeline card sits between the greet-card and the today-grid,
      rendering today's class blocks (colored by class) and planned-task
      blocks to scale, with an hour-gridline background, a live "now"
      line, and collision handling that offsets overlapping blocks into
      lanes. "＋ Plan" on any Due-soon row auto-places that task into the
      first free gap today (60min default, sized down to the gap if it's
      30–59min); re-clicking an already-planned task's button ("↻ Replan")
      moves it; ✕ unplans. Done tasks strike their plan block. Free-time
      budget (🕐 Xh free, gaps ≥15min before 22:00) shows in the greet-card
      hero. Empty states for no-schedule weekdays and weekends.
      `parseScheduleString(s)` best-effort parses free-text schedules
      ("MWF 10am", "TuTh 2:30pm", "Mon/Wed 14:00") into a meeting; the
      class-creation form gained day toggles + start/end time inputs to
      set it directly. SCHEMA_VERSION = 12: migrate() gives every class a
      parsed `meetings` array (from its legacy schedule text) and adds
      `plans: []`; `load()` prunes plan entries older than 14 days.

- [x] **Natural-language quick add (v0.15):** `parseQuickTask(raw)` parses
      free text into title/class/date/priority — `!high`/`!h` `!med`/`!m`
      `!low`/`!l` priority tags, `#code` class match (code → exact name →
      unique name prefix; no match leaves the token in the title, never
      auto-creates a class), `today`/`tod`, `tomorrow`/`tmr`/`tom`, day
      names (`fri`/`friday` → next occurrence), `M/D` explicit dates (rolls
      to next year if already past), `+Nd` relative dates, filler word
      `due` dropped only when followed by a date token. First
      priority/class/date token wins; later ones stay in the title; empty
      title after parsing returns `null` (caller no-ops, nothing added).
      Wired into Today's `#quickInput` (live `#nlPreview` chips on
      `input`, reusing `.chip` styles) and the command palette's quick-add
      row (parsed sub-label, e.g. `NEW TASK · Fri Jul 17 · CS101 · HIGH`).
      Assignments' explicit form is untouched. No schema change.

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

**`docs/BUILD_PLAN.md` is the authoritative spec for v0.17–v0.19**
(written July 15 2026 from competitive research; implement in order, one
version per commit, browser-verified):

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

Session 9 (July 16, 2026): Miguel pointed Claude at `docs/BUILD_PLAN.md`
and asked for v0.15–v0.19 to be built one at a time, verified between
each. Shipped **v0.15 natural-language quick add**: `parseQuickTask(raw)`
parses priority/class/date tokens out of free text per the BUILD_PLAN
spec, wired into Today's quick-add (live `#nlPreview` chips) and the
command palette's quick-add row (parsed sub-label). Verified live in
browser: the browser pane's synthetic `type`/`key` actions turned out not
to dispatch real DOM events this session (confirmed against the
pre-existing, unmodified notes-search box too, so it's an environment
quirk, not a regression) — verification used the `form_input` tool
instead, which does dispatch real events. Confirmed: full-combo input
("...fri #cs101 !high"-style) → correct Jul 17 / CS101 / HIGH preview
chips; "read ch 3 #nope" → no class match, chips hidden, "#nope" stays
literal; "fri" alone → empty title, parser returns `null`, chips stay
hidden; palette query "essay tue #math !l" → sub-label "NEW TASK · Tue,
Jul 21 · Math · LOW". Caught and fixed one real bug before commit: the
command-palette edit landed with straight quotes silently turned into
curly quotes in one block — syntactically invalid but silent at load time
because it sat inside `palBuild`'s `if(ql)` branch, which only runs on a
non-empty query, so it never fired during boot-time rendering. Re-verified
clean after the fix; zero console errors. No schema change. Committed and
pushed (Miguel approved after reviewing).

Same session, continued straight into **v0.16 schedule-aware Today / "plan
my day"** (signature feature, schema 12) — see the DONE list above for the
feature summary. Found a much more reliable verification technique partway
through: `fetch()` the served file, extract the inline `<script>` body, and
run it via `new Function(body)()` wrapped in try/catch — this executes the
*entire* script (not just one function) and surfaces syntax errors the
browser pane's console reader was silently swallowing. It caught a real
bug immediately: `data-cday` (the new class-meeting day-toggle handler)
was accidentally named `cdy` in the delegated click listener, colliding
with the pre-existing `data-calday` handler's `const cdy` in the same
function scope — a hard `SyntaxError` that silently killed the *entire*
script (nothing rendered, nothing was clickable) because `read_console_messages`
never surfaced it. Renamed to `clsd` and re-verified clean. Also caught
(via code review, before it ever ran) a classic footgun: `taskRow` gained
an optional second parameter, and one leftover call site used
`list.map(taskRow)` directly — `.map()` passes `(element, index, array)`,
so the Assignments List view would have silently shown a "＋ Plan" button
on every task past the first. Fixed to `list.map(t=>taskRow(t))`. Also
learned this session's `computer` tool (click/type/key) doesn't reliably
dispatch events the app receives — not just typing (found in the v0.15
entry above) but clicks too (a real `computer.left_click` on "Add class"
silently no-opped; `document.getElementById(...).click()` via
`javascript_tool` worked immediately after). All v0.16 verification below
used that DOM-`.click()` + `form_input` + direct `localStorage`/DOM-state
reads instead. Confirmed: migration v11→12 on real pre-existing v11
localStorage (CS101's "MWF 10am" text → a parsed 60-min meeting; Math's
blank schedule text → correctly stayed empty); fresh-seed timeline
geometry hand-verified against the render (block top/height in px, now-line
position, free-time hours all matched by-hand math); ＋ Plan placed a task
into the first real gap and persisted across reload; unplan removed it;
marking a planned task done struck its timeline block; a new class created
via the day-toggle + time-input UI produced the exact right `meetings`
shape and the toggle state reset after; two overlapping meetings correctly
split into offset lanes (0px / 12px); dark scheme re-rendered the timeline
correctly via `color-mix(..., var(--card))` (tokens only, dark mode free,
per DESIGN.md). Weekend empty-state and the 30–59min gap-sizing branch
were verified by code review only (couldn't manipulate system day-of-week
or engineer that exact gap scenario in the time available) — worth a
live check if either is ever suspected of a bug. Zero console errors.
Committed locally; **not pushed** — holding for Miguel to check the work
before starting v0.17.

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

# ROADMAP — Campus OS

Ordered by value to Miguel. Each item is sized so it fits in one
Claude session comfortably.

## Phase 1 — daily-driver features (vanilla JS, same file)

1. ~~**Internship application tracker**~~ shipped in v0.6 — Internships view,
   `apps` array, kanban columns by status (wishlist → applied → OA →
   interview → offer/rejected), deadline/link/notes, edit-in-form.
2. ~~**Classes database**~~ shipped in v0.8 — `classes` array (name, code,
   color, schedule), tasks reference classId, class filter chips + inline
   class management on the Assignments view.
3. ~~**Calendar view**~~ shipped in v0.11 — month lens on Assignments:
   tasks by due date (class colors), internship deadlines as 🚀 pills,
   click-a-day to quick-add.
4. ~~**Kanban view for assignments**~~ shipped in v0.11 — board lens on
   Assignments (todo / doing / done, click-to-advance).
5. ~~**Recurring habits config**~~ shipped in v0.9 — per-habit target days
   (habit.days, 0=Mon), streaks skip rest days, Today shows only due habits.

## Phase 2 — polish

6. ~~Search across TASKS~~ shipped in v0.12 as the command palette (Ctrl+K):
   searches tasks, notes, internships, habits and views, plus quick-add.
7. ~~Basic markdown in notes~~ shipped in v0.5. Possible extras: tables, images.
7b. **AI helper (optional):** Settings field for user's own Anthropic API
    key → summarize note / flashcards from note / NL task add. Haiku 4.5.
    Personal-use only while key lives in the browser.
8. Keyboard shortcuts: `n` new task, `1–5` switch views.
9. Dark mode via a second token set.

## Phase 2.5 — accounts & sync (only if Miguel decides he wants it)

Plan of record: **Supabase** (free tier) rather than a hand-rolled backend.
- Email/password or magic-link auth via Supabase Auth
- One `documents` table (user_id, json blob) with row-level security so each
  user can only read/write their own row; app syncs `state` up/down
- Keep localStorage as offline cache; export/import stays as escape hatch
- Requires Miguel to create the Supabase project + keys himself (Claude
  should never handle his credentials); then ~1–2 sessions to integrate

## Phase 3 — learning project: React migration

10. Rebuild as a Vite + React app, one view at a time, keeping the same
    state shape and localStorage key so data carries over. This is a
    deliberate learning exercise (Miguel has built React POCs before);
    the vanilla version stays as the reference implementation.

## From Miguel's notes (docs/CAPMUS-OS NOTES.txt), triaged July 15 2026

- ~~**Finances**~~ shipped in v0.14 (Money view).
- **AI agent** — same as roadmap 7b; waiting on Miguel getting an API key.
- **Dashboard** — mostly covered by Today + Progress; revisit only if a
  concrete gap shows up.
- **Google services connection** — parking lot (needs OAuth + Miguel's own
  credentials; Claude must never handle those).
- **Blog/community thread area** — needs a backend; rides on the Supabase
  phase 2.5 decision. Friends circle is the no-backend stand-in.

## Shipped outside the original plan

- **Money view (v0.14)** — from Miguel's notes: expense/income log with
  emoji categories, monthly budget + progress bar, 6-month spending chart,
  top-category stat, spend-this-week on the recap card (never in the
  shared recap text).

- **Focus timer (v0.13)** — pomodoro-style deep-work sprints (15/25/50m)
  optionally tied to a task; chime on finish; sessions logged (schema 10);
  focus minutes in the Today hero and weekly recap. Inspired by
  Forest/Pomofocus.
- **Command palette (v0.12)** — Ctrl+K search-and-jump across everything,
  Linear/Raycast-style; subsumes roadmap #6.

- **Weekly recap card (v0.10)** — week-in-review hero on Progress (tasks
  vs last week, habit %, best habit, mood, pipeline, reflections) with
  copy-for-circle text that embeds your friend code; Sunday-evening nudge
  on Today. Natural surface for the AI helper later (same data, Haiku
  writes the recap).
- **Friends / circle leaderboard (v0.7)** — no-backend version via pasted
  "friend codes" (base64 stat snapshots): community leaderboard filterable
  by tasks / habit consistency / pipeline moves / reflection streak,
  copy-to-clipboard nudge messages. A live version (real accounts, in-app
  nudges) would ride on the Supabase phase 2.5 plan; the snapshot shape is
  designed to become the sync payload.

## Feature research — what makes competing apps win (July 15, 2026)

Claude researched the app categories Campus OS touches (student planners,
habit trackers, daily planners, study tools, job trackers) and identified
the five highest-value additions, ranked. Evidence and reasoning live in
the session notes; short version:

1. **Schedule-aware Today / "Plan my day"** (Structured, Shovel, Sunsama,
   MyStudyLife) — render today's classes as a timeline, drag tasks/focus
   blocks into free gaps, show free-time budget. The daily planning ritual
   is the #1 retention loop in this category; classes already store a
   schedule string the app never uses.
2. **Spaced-repetition flashcards from notes** (Anki) — Q/A cards written
   in notes (`Q:`/`A:` or #flashcard), SM-2-lite scheduler, "cards due"
   on Today. Retrieval practice beats rereading by ~50% in studies; no
   competitor combines planner + notes + SRS in one local app. Natural
   surface for the AI helper later (auto-generate cards).
3. **Natural-language quick add** (Todoist) — parse "PS4 due fri #cs101
   !high" in quick-add and the palette. Capture friction is the top
   reason task systems get abandoned.
4. **PWA install + reminders** (Duolingo, every mobile-first app) — web
   manifest + service worker + optional local notifications; requires
   hosting (GitHub Pages — repo already exists). Home-screen presence is
   the difference between a demo and a daily driver; installability alone
   showed 150–450% retention lifts in published case studies.
5. **Grade tracking + "what do I need on the final"** (MyStudyLife, every
   grade-calculator site) — per-class syllabus weights + scores → current
   grade + what-if calculator. Classic student hook; classes DB already
   exists; spikes in usefulness exactly at midterms/finals.

Honorable mention: guided **weekly review** (GTD/Things 3) — extend the
Sunday recap nudge into a short review flow (archive stale, roll over,
plan next week). Cheap and pairs with #1.

## Parking lot (maybe never)

- Shared pages (would need a backend — big scope jump)
- Google Calendar sync (Miguel has explored the API before)
- Mobile PWA install (manifest + service worker)

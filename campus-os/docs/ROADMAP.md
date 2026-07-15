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
3. **Calendar view** ← NEXT — month grid rendering tasks by due date.
4. **Kanban view for assignments** — status columns with click-to-advance
   (data model already supports it).
5. **Recurring habits config** — per-habit target days (e.g., lift MWF),
   so streaks don't break on rest days.

## Phase 2 — polish

6. Search across TASKS (notes search shipped in v0.5).
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

## Shipped outside the original plan

- **Friends / circle leaderboard (v0.7)** — no-backend version via pasted
  "friend codes" (base64 stat snapshots): community leaderboard filterable
  by tasks / habit consistency / pipeline moves / reflection streak,
  copy-to-clipboard nudge messages. A live version (real accounts, in-app
  nudges) would ride on the Supabase phase 2.5 plan; the snapshot shape is
  designed to become the sync payload.

## Parking lot (maybe never)

- Shared pages (would need a backend — big scope jump)
- Google Calendar sync (Miguel has explored the API before)
- Mobile PWA install (manifest + service worker)

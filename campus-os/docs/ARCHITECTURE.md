# ARCHITECTURE — Campus OS v0.1

## Big picture

One file: `app/index.html`. Three sections inside it:

1. `<style>` — all CSS, driven by CSS variables defined in `:root`
   (tokens documented in DESIGN.md)
2. HTML skeleton — sidebar + one `<section>` per view, only one visible at a time
3. `<script>` — all logic, organized top-to-bottom as:
   - constants & storage helpers
   - state (single object) + seed data
   - actions (functions that mutate state, then call `save()` + `render()`)
   - render functions (one per view)
   - event wiring at the bottom

This is the classic **"change data → re-render"** pattern (the same mental
model as React, without React): user event → action mutates `state` →
`save()` persists → `render()` redraws the active view from state.

## Data model (SCHEMA_VERSION = 1)

```js
state = {
  schema: 1,
  tasks: [        // assignments & todos
    { id, title, cls, due,        // due = "YYYY-MM-DD" or "" for no date
      priority,                   // "low" | "med" | "high"
      status,                     // "todo" | "doing" | "done"
      created }                   // ISO timestamp
  ],
  habits: [
    { id, name, log: { "YYYY-MM-DD": true, ... } }
  ],
  notes: [
    { id, title, body, updated }  // body: plain text, ``` fences = code
  ]
}
```

- IDs: `Date.now().toString(36) + random suffix` via `uid()`.
- Dates are local-timezone `YYYY-MM-DD` strings produced by `todayStr()` /
  `dateStr(d)`. Never use `toISOString().slice(0,10)` for local dates
  (it shifts across timezones).

## Storage

- Key: `"campus-os-data"` in `localStorage`.
- Every read/write wrapped in try/catch; if storage is unavailable
  (private mode, artifact preview), the app runs in-memory and shows a
  small warning chip in the sidebar.
- Export: serializes `state` to a downloaded `.json` file.
- Import: parses uploaded JSON, validates `schema`, replaces state.
  When `SCHEMA_VERSION` is bumped, add a migration step in `importData()`.

## Rendering approach

- Each view has a `renderX()` function that rebuilds its DOM via template
  strings + `innerHTML`, then attaches listeners.
- `render()` calls only the active view's renderer.
- Escaping: all user text passes through `esc()` before hitting innerHTML.
  **Never interpolate raw user input.**
- Code fences in notes: `renderNoteBody()` splits on ``` and alternates
  paragraph / `<pre>` blocks. Both branches escaped.

## Adding a new view (checklist)

1. Add a nav button in the sidebar with `data-view="myview"`.
2. Add `<section id="view-myview" class="view">`.
3. Write `renderMyview()`; register it in the `renderers` map.
4. Add state fields if needed; bump `SCHEMA_VERSION` if the shape changes;
   extend seed data and import migration.
5. Update PROJECT_STATUS.md and ROADMAP.md.

---

## v0.2 additions (SCHEMA_VERSION = 2)

New state fields:

```js
settings:    { name }                       // greeting name
reflections: [ { id, date:"YYYY-MM-DD",     // one entry per date
                 mood,                      // 1..5 (0 = unset)
                 grateful, win, text } ]
prayers:     [ { id, text, status,          // "active" | "answered"
                 created, answered } ]      // YYYY-MM-DD dates
verses:      [ { id, ref, text } ]
```

- `migrate(data)` upgrades schema 1 → 2 (adds the fields above). Called in
  both `load()` and import. Future bumps chain inside `migrate()`.
- Verse of the day: `verses[dayOfYear() % verses.length]` — deterministic,
  no stored pointer.
- Reflections are keyed by local date; `saveReflection()` upserts today's.
- New views registered in `renderers`: `reflect`, `faith`. Shortcuts 1–7.

---

## v0.3 additions (SCHEMA_VERSION = 3)

- `task.completed` — local `YYYY-MM-DD`, set in `cycleStatus()` when a task
  enters "done", deleted when it leaves. `migrate()` v2→v3 backfills done
  tasks with `created.slice(0,10)` (approximation, noted in UI hint).
- **Progress view:** `progBuckets(period)` returns oldest→newest buckets,
  each `{label, full, test(dateStr)=>bool}`. Ranges: daily = 14 days,
  weekly = 12 Mon-start weeks, monthly = 12 months, yearly = 5 years.
  `renderProgress()` counts completed tasks per bucket and draws an inline
  SVG bar chart (no chart library) — bars use `<title>` tooltips, current
  bucket uses the `#barGrad` gradient. Stat cards show current vs previous
  bucket delta, best bucket, open/overdue counts.


---

## v0.4 additions (SCHEMA_VERSION = 4)

- `settings.theme`: "auto" | "light" | "dark". `resolvedTheme()` maps auto
  via `matchMedia("(prefers-color-scheme: dark)")`; `applyTheme()` sets
  `document.documentElement.dataset.theme` and syncs the sidebar toggle +
  Settings select. A matchMedia listener re-applies on OS changes when auto.
- Dark styling lives entirely in the `html[data-theme="dark"]` token block
  plus a handful of component overrides — new components should use tokens
  so they inherit dark mode for free.


---

## v0.5 additions (SCHEMA_VERSION = 5)

**Notes (Obsidian-lite):**
- `renderInline(escapedText)` — order matters: `code` → **bold** →
  [[wikilinks]] → #tags. Always runs on ALREADY-ESCAPED text.
- `renderNoteBody` — line-based markdown-lite (headings, lists, paragraphs)
  around ``` fence splitting.
- `noteTags(n)` extracts lowercase tags; `renderNoteFilters()` builds the
  chip row; `noteSearch` + `noteTagFilter` module-level filter state.
- `openOrCreateNote(title)` — case-insensitive match, creates on miss
  (wikilink click path). `backlinksFor(n)` = notes containing "[[title"
  (lowercased).

**Schemes:**
- `SCHEMES` map: id → {name, dark, vars}. `field`/`pine` have empty vars
  (they ARE the stylesheet defaults). Others override via inline CSS custom
  properties on <html> (`VARMAP` translates key → --var-name).
- `applyTheme()` clears then sets vars, sets `data-theme` = dark|light so
  the dark component-override block still applies, syncs the sidebar button
  and gallery selection. `settings.scheme` = "auto" | scheme id; auto maps
  to field/pine by prefers-color-scheme.
- Adding a scheme = one entry in SCHEMES. Nothing else.

## v0.6 additions (SCHEMA_VERSION = 6)

**Internship application tracker (`apps` view, nav "Internships"):**

```js
apps: [
  { id, company, role,
    status,          // "wishlist" | "applied" | "oa" | "interview" | "offer" | "rejected"
    deadline,        // "YYYY-MM-DD" or "" (application deadline)
    link,            // posting / portal URL or ""
    notes,           // free text
    created,         // ISO timestamp
    updated }        // YYYY-MM-DD, stamped on every edit or status move
]
```

- Pipeline order lives in `APP_STATUSES`; labels in `APP_LABELS`. Adding a
  stage = extend both arrays (kanban grid uses `repeat(6, …)` — update the
  `.kanban` column count too).
- `renderApps()` builds one `.kb-col` per status; cards sorted by deadline
  (empty deadlines last). Deadline chip goes `overdue` (flag) when past and
  the app is not in a terminal state (offer/rejected).
- Moving: `moveApp(id, ±1)` shifts along `APP_STATUSES`, clamped at the ends
  (◀ disabled on wishlist, ▶ disabled on rejected).
- Editing: clicking a card calls `editApp(id)` — loads it into the top form
  (`selectedApp` module state, mirrors `selectedNote`); the Add button becomes
  "Update" and a Cancel button appears. `submitApp()` handles both add and
  update; company is the only required field.
- Delegated clicks (in the shared body listener): `data-amove` / `data-adel`
  are checked **before** `data-aedit` so the buttons inside a card don't
  trigger edit; `data-alink` returns early so the posting link opens normally.
- `migrate()` v5→v6 just adds `apps: []`.
- Keyboard shortcuts renumbered to 1–9 (6 = Internships).

## v0.7 additions (SCHEMA_VERSION = 7)

**Friends / circle leaderboard — no backend, "friend codes":**

```js
settings.shareId   // stable uid, generated once (migrate v6→v7); identifies
                   // this person across codes so re-pastes update, not dupe
friends: [
  { id,            // local row id
    fid,           // the friend's shareId (from their code)
    name, added,
    snapshot: {    // parsed from the last code they sent
      g,           // YYYY-MM-DD the code was generated ("as of")
      t,           // tasks completed that week
      hb, hp,      // best habit streak, habit consistency % (last 7 days)
      a,           // internship pipeline moves that week (apps.updated >= Monday)
      r } }        // reflection streak (consecutive days)
]
```

- **Share code** = `"COS1-" + base64(JSON payload)` with compact keys
  `{v:1, id, n, g, t, hb, hp, a, r}` — built by `myShareCode()` from
  `myStats()` (computed live), parsed by `parseShareCode()` (validates
  v/id/n, returns null on garbage). Unicode-safe via the
  `encodeURIComponent`/`escape` btoa dance.
- `addFriendCode()` upserts by `fid` (pasting a newer code refreshes the
  friend), rejects your own code (`id === settings.shareId`).
- **Leaderboard**: `LB_METRICS` map (tasks / habits / apps / reflect), each
  `{label, get(snapshot), fmt(snapshot)}` — filter chips switch `lbMetric`
  (default "tasks"). Rows = you (live `myStats()`) + friends (snapshots),
  sorted by metric; bars scale to the group max. Community framing: no rank
  numbers, "YOU" chip, group-total line under the board. Snapshots older
  than 7 days get a STALE chip (`daysAgo()`).
- **Nudge**: per-friend button copies a rotating encouragement message
  (`NUDGES` templates) to the clipboard via `copyText()` (async clipboard
  API with execCommand fallback + button-label flash) — to paste into any
  messaging app. There is no in-app delivery; this is deliberate (no backend).
- Shortcuts renumbered: 7 = Friends, 8/9/0 = Reflect/Faith/Settings.
- If live sync ever happens (Supabase phase 2.5), the snapshot shape is the
  sync payload — swap "paste a code" for "fetch rows", keep everything else.

## v0.8 additions (SCHEMA_VERSION = 8)

**Classes database:**

```js
classes: [ { id, name, code, color, schedule } ]  // color ∈ CLASS_COLORS
// tasks: cls (free text) REPLACED by classId ("" = no class)
```

- `migrate()` v7→v8 groups existing `task.cls` strings case-insensitively,
  creates one class per distinct name (colors assigned round-robin from
  `CLASS_COLORS`), sets `task.classId`, deletes `task.cls`.
- Assignments form: class `<select>` (`#tClass`, options rebuilt in
  `renderTasks()`, selection preserved). "＋ class" chip toggles an inline
  create form (name, code, schedule, color swatch row → `newClassColor`).
- Class filter chips (`#classFilters`, `taskClassFilter` state) show dot +
  code-or-name + open count; ✕ inside the chip deletes the class after
  confirm (tasks keep running with `classId:""`). Both filters (status +
  class) apply to the list.
- `classById(id)` helper; `taskRow()` renders a colored `.dot` + class name.
- CLASS_COLORS palette documented in DESIGN.md — dots/swatches only.

## v0.9 additions (SCHEMA_VERSION = 9)

**Recurring habit targets:**

- `habit.days` — `null` (every day, the default) or an array of day indexes
  where **0 = Monday** (matches `weekDates()`); normalized: empty or full
  arrays collapse to `null`. `migrate()` v8→v9 adds `days: null`.
- Helpers: `habitDays(h)` (normalized), `habitDue(h, date)`,
  `schedLabel(h)` ("MWF" / "daily"), `DAY_LETTERS` (Mon-first).
- `streak(h)` rewritten: walks back skipping non-target days — rest days
  never break a streak, missed target days do. Today-still-open rule kept.
  Iteration-capped at 3650.
- Habits view: schedule chip next to each name toggles an inline editor row
  (`schedEditing` state) with 7 day toggles. Off-day cells render dashed at
  35% (`.cell.off`) but stay clickable (bonus logging allowed; doesn't
  affect streaks).
- Today view lists only habits due today (`habitDue`), shows a rest-day
  empty state, and the hero counts x/due.
- Friends metric `hp` (habit consistency) now counts only due cells over
  the last 7 days — fairer for scheduled habits.

## v0.10 additions (no schema change)

**Weekly recap card:**

- Pure derived data — `weekRecap()` computes Mon→today vs the full previous
  week: tasks done (+delta), habit % on target (due cells only), best habit
  by hits, avg mood, pipeline moves (`apps.updated >= Monday`), reflection
  days. No state added, nothing to migrate.
- `renderRecap()` draws a pine-gradient hero (`.recap-card`) at the top of
  the Progress view, called from `renderProgress()`.
- "Copy recap for your circle" → `recapText()` builds a plain-text summary
  ending with the sender's current friend code (so receivers can add/refresh
  them in one paste), copied via the existing `copyText()`.
- Sunday-evening nudge on Today (`#recapNudge`, `getDay()===0 && h>=17`)
  jumps to Progress — same pattern as the reflect nudge.

## v0.11 additions (no schema change)

**Assignments as three lenses (List / Board / Calendar) — Notion-style database views:**

- `taskView` module state ("list" | "board" | "cal"), switched by the
  `#taskViewTabs` chips in the view header (`data-tv`). `renderTasks()` sets
  container visibility + the hint text, then delegates to `renderTaskBoard()`
  or `renderTaskCal()`. The class filter applies in all three lenses; the
  status filter row only shows in list mode (board columns ARE the statuses).
- **Board:** `TASK_COLS = ["todo","doing","done"]`, reuses the `.kanban`
  styles with a `.k3` 3-column modifier. Clicking a card advances status
  (`data-cycle` on the whole card); ✕ deletes. NOTE: because the ✕ nests
  inside the cycle target, the body click listener now checks `data-del`
  BEFORE `data-cycle` — keep that order.
- **Calendar:** `calOffset` module state (months from current; "Today"
  resets to 0). Mon-first month grid, rows = `ceil((firstDow+days)/7)`.
  Task pills (class-colored dot, done = struck, past-due = flag) advance
  status on click via `data-cycle`. Internship deadlines for non-terminal
  apps render as 🚀 pills (`data-calapp` → jump to Internships + edit).
  Clicking an empty day (`data-calday`) prefills `#tDue` and focuses
  `#tTitle` — calendar-first task capture. Max 3 pills per cell, then
  "+N more".

## v0.12 additions (no schema change)

**Command palette (Ctrl+K / ⌘K) — Linear/Raycast-style:**

- `#palOverlay` fixed overlay + `#palInput` + `#palResults`; opened by
  Ctrl/Cmd+K anywhere, the 🔍 sidebar button (`#palBtn`), closed by Esc /
  backdrop click. ↑↓ + Enter keyboard navigation (`palSel` index).
- `palBuild(q)` scores sources with startsWith(3) > includes(2):
  views ("Go to …"), tasks (icon = ⬜/✅, open tasks rank higher), notes
  (title or body match), internships (company/role), habits. Empty query
  shows views + quick actions (new note, start focus, copy friend code).
  Last row is always **Add task: "q"** — quick-add from anywhere (this is
  the roadmap's "search across tasks" feature, generalized).
- Task/note/app results navigate to their view (notes select the note,
  apps load the edit form). Nothing mutates on selection except quick-add.

## v0.13 additions (SCHEMA_VERSION = 10)

**Focus timer (Forest/pomodoro-style deep-work sprints):**

```js
focus: [ { id, date:"YYYY-MM-DD", mins, taskId } ]  // one entry per finished session
```

- `migrate()` v9→v10 adds `focus: []`.
- `#focusPanel` card on Today (right column, above habits): idle state =
  open-task select + 15/25/50m preset chips (`FOCUS_PRESETS`, `focusMins`)
  + Start; running state = mm:ss `#focusClock`, task name, "End early".
- `focusRun` module state {taskId, mins, started, endsAt, timer} — a 1s
  `setInterval` (`focusTick`) updates the clock and the sidebar mini chip
  (`#focusMini`, visible in every view, click → Today). Deliberately NOT
  persisted: reloading mid-session drops the run; only finished sessions
  are logged.
- `focusEnd(completed)`: completed sessions log `mins` as planned and play
  `focusChime()` (two WebAudio sine notes — no audio file); "End early"
  logs elapsed whole minutes, and under 1 minute logs nothing.
- Surfaces: Today hero ("focus Xm" once > 0), weekly recap card + copied
  recap text ("🎯 Xm of deep work", omitted when 0). `weekRecap()` gained
  `focusMins`.

## v0.14 additions (SCHEMA_VERSION = 11)

**Money view (student budget tracker):**

```js
money: {
  budget: 0,                                   // monthly budget in $; 0 = unset
  txs: [ { id, date:"YYYY-MM-DD", amount,      // amount always positive
           cat,                                // key of MONEY_CATS ("income" = money in)
           note } ]
}
```

- `migrate()` v10→v11 adds `money: {budget:0, txs:[]}`. Seed ships budget
  200 + 3 sample txs.
- `MONEY_CATS` map (food / transport / school / fun / subs / other /
  income) — label + emoji. Income is just a category; all math treats
  `cat==="income"` as money in, everything else as spend. Adding a
  category = one entry in the map.
- `addTx()` validates amount (`parseFloat > 0`, rounded to cents) and
  category; junk input is silently ignored. `setBudget()` — 0/invalid
  clears the budget.
- `renderMoney()`: 4 stat cards (spent this month / budget — the $ value
  is an inline `#mBudget` number input, listener re-attached each render —
  / income + net / top category), budget progress bar (`.money-fill`,
  capped at 100%, `.over` flips it to the flag gradient), 6-month spending
  SVG bar chart (own `#moneyGrad` gradient — don't reuse `#barGrad`, it
  lives in the Progress view's SVG), category filter chips
  (`moneyCatFilter`), and this month's tx list (delete via `data-txdel`).
- **Privacy rule:** `weekRecap()` gained `spentWeek` and the recap CARD
  shows it, but `recapText()` (the copy-for-your-circle share text)
  deliberately contains no money data. Keep it that way.
- Nav: Money sits in MAIN after Friends, with **no digit shortcut** (1–0
  are all taken) — reachable via sidebar or the palette ("Go to Money").

## v0.15 additions (no schema change)

**Natural-language quick add:**

- `parseQuickTask(raw)` → `{title, classId, due, priority, explicitPriority}`
  (`explicitPriority` is an extra field, preview-only — see below).
  Tokenizes on whitespace; each token is tried in order against priority
  (`!high`/`!h` `!med`/`!m` `!low`/`!l`, first one wins) → class (`#code`,
  only while unmatched) → date (`today`/`tod`, `tomorrow`/`tmr`/`tom`, day
  names via `nlDateFromDow`, `M/D`, `+Nd`, filler `due` dropped only when
  the next token is date-shaped) → else the token stays in the title.
  First match per field wins; later conflicting tokens fall through to the
  title untouched. An unmatched `#code` never auto-creates a class — it
  just stays in the title. Empty title after parsing → `null` (both call
  sites no-op on `null`).
- `nlMatchClass(name)`: `class.code` (spaces/case stripped) → exact
  `class.name` (case-insensitive) → unique case-insensitive name prefix;
  ambiguous or no match → `null`.
- `renderNlPreview(raw)` mirrors `parseQuickTask` into `#nlPreview` chips
  (`.nl-prev`, reuses existing `.chip` classes — due and class both use
  `.chip.todo`, priority uses `.chip.high/med/low` but ONLY when
  `explicitPriority` is true, so the default "med" doesn't render a chip
  on every keystroke). Hidden whenever nothing recognizable parsed.
- Wired at `#quickInput`: `input` → `renderNlPreview`; Enter → parse, no-op
  on `null`, else `addTask(t.title,t.classId,t.due,t.priority)` then clear
  the field. Also wired into `palBuild`'s quick-add row — its `sub` label
  shows the parsed pieces (e.g. `NEW TASK · Fri Jul 17 · CS101 · HIGH`);
  if parsing empties the title, it falls back to a plain-title task using
  the raw query, same as pre-v0.15 behavior.
- Assignments' explicit add-task form is untouched — NL parsing applies
  only to the two quick-add surfaces, per spec.

## AI helper (planned, not built) — plan of record
- Direct browser → Anthropic Messages API with the user's own key
  (stored in localStorage, entered in Settings; requires the
  `anthropic-dangerous-direct-browser-access` header). Personal use only —
  if the app is ever published/hosted for other users, route through a
  proxy (e.g. Supabase edge function) instead; never ship a key in code.
- Start with Haiku 4.5 (cheapest current-gen). Candidate features:
  summarize note, generate flashcards/quiz from a note, natural-language
  task add, weekly plan suggestion from open tasks.

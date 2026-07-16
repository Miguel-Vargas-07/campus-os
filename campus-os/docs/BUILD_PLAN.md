# BUILD_PLAN — Campus OS v0.15 → v0.19

> **Audience: the AI session implementing these features.** This is the
> authoritative spec for the next five versions, written July 15 2026 after
> competitive research (see ROADMAP.md → "Feature research"). Follow it in
> order. Each version = one feature = one commit = one browser-verified
> release. Do not batch features into one commit.

---

## How to run this plan

1. Read `PROJECT_STATUS.md`, `docs/ARCHITECTURE.md`, `docs/DESIGN.md` first.
   Everything lives in **one file**: `campus-os/app/index.html`.
2. Implement ONE version at a time, in order: v0.15 → v0.16 → v0.17 →
   v0.18 → v0.19. After each: verify in a real browser (see "Verification
   harness" below), update `PROJECT_STATUS.md` (version list, DONE list,
   session summary) + `docs/ARCHITECTURE.md` (new section per version) +
   this file (mark the version ✅ SHIPPED), commit (`Campus OS vX.Y — <name>`),
   push.
3. Hard rules (from PROJECT_STATUS conventions — do not violate):
   - All state in the single `state` object; every mutation → `save()` then
     `render()`.
   - Any state-shape change bumps `SCHEMA_VERSION` and extends `migrate()`
     (chained `if (data.schema === N)` blocks) AND `seed()`.
   - All user text through `esc()` before `innerHTML`. Never interpolate raw.
   - Use existing CSS custom-property tokens only (DESIGN.md). New components
     built from tokens inherit dark mode for free.
   - In the delegated `document.body` click listener, `data-del` is checked
     **before** `data-cycle` (cards nest delete inside the cycle target).
     Preserve existing check order; append new `data-*` checks at the end
     unless a nesting conflict requires earlier placement — if so, comment why.
   - Digit shortcuts 1–0 are all taken. New views get NO digit key; they are
     reachable via sidebar nav + command palette (add to `PAL_VIEWS`).

### Current architecture cheat-sheet (as of v0.14, schema 11)

- **State fields:** `settings{name, scheme, shareId}`, `classes[]`, `tasks[]`,
  `habits[]`, `notes[]`, `apps[]`, `friends[]`, `focus[]`,
  `money{budget, txs[]}`, `reflections[]`, `prayers[]`, `verses[]`.
- **Task:** `{id, title, classId, due:"YYYY-MM-DD"|"", priority:"low"|"med"|"high",
  status:"todo"|"doing"|"done", created:ISO, completed?:"YYYY-MM-DD"}`.
- **Class:** `{id, name, code, color, schedule}` — `schedule` is free text
  ("MWF 10am"); `color` ∈ `CLASS_COLORS`.
- **Helpers that already exist (reuse, don't reinvent):** `$()`, `uid()`,
  `esc()`, `pad()`, `dateStr(d)`, `todayStr()`, `fmtDue()`, `fmtLong()`,
  `weekDates()` (Mon-first), `mondayOf()`, `monthKey()`, `daysAgo()`,
  `classById()`, `copyText()`, `fmtMoney()`, `fmtClock()`,
  `DAY_LETTERS` (`["M","T","W","T","F","S","S"]`, index 0 = Monday).
- **Views/renderers:** `renderers` map, `render()` renders active view only,
  `switchView(v)`. Sections are `<section class="view" id="view-X">`.
- **Command palette:** `PAL_VIEWS` array, `palBuild(q)`; its last row
  quick-adds the query via `addTask(q, "", "", "med")`.
- **Quick add on Today:** `#quickInput` keydown Enter →
  `addTask(value, "", "", "med")`.
- **Assignments** has three lenses (list/board/cal) — leave its explicit
  form alone; NL parsing applies only where specified below.
- **Focus timer:** `focusRun` module state; sessions in `focus[]`
  `{id, date, mins, taskId}`; `FOCUS_PRESETS = [15, 25, 50]`.
- **Deployed:** https://miguel-vargas-07.github.io/campus-os/ (GitHub Pages,
  main branch, root `index.html` redirects into `campus-os/app/index.html`).
  localStorage is per-origin (file:// ≠ Pages origin); export/import is the
  migration path between them.

### Verification harness

The Browser pane cannot open `file://` URLs and blocks arbitrary localhost —
serve through the preview tool. No Python/Node on this machine; use the
PowerShell static server:

```powershell
# serve.ps1 — save to scratchpad, point launch.json at it, port 8377
$root = "C:\Users\Dell\Documents\campus-os\campus-os\app"
$l = New-Object System.Net.HttpListener
$l.Prefixes.Add("http://localhost:8377/"); $l.Start()
while ($l.IsListening) {
  try {
    $c = $l.GetContext(); $p = $c.Request.Url.AbsolutePath
    if ($p -eq "/") { $p = "/index.html" }
    $f = Join-Path $root ($p.TrimStart("/") -replace "/", "\")
    if ((Test-Path $f) -and $f.StartsWith($root)) {
      $b = [System.IO.File]::ReadAllBytes($f)
      $mime = @{ ".html"="text/html; charset=utf-8"; ".webmanifest"="application/manifest+json";
                 ".js"="text/javascript"; ".png"="image/png"; ".svg"="image/svg+xml" }[[System.IO.Path]::GetExtension($f)]
      if ($mime) { $c.Response.ContentType = $mime }
      $c.Response.Headers.Add("Cache-Control","no-store")
      $c.Response.OutputStream.Write($b, 0, $b.Length)
    } else { $c.Response.StatusCode = 404 }
    $c.Response.Close()
  } catch {}
}
```

Create `.claude/launch.json` in the working directory with
`runtimeExecutable: "powershell"`, args
`["-NoProfile","-ExecutionPolicy","Bypass","-File","<path>\\serve.ps1"]`,
`port: 8377`, then `preview_start`. Verify with `javascript_tool` assertions
(migration ran, feature flows work, zero console errors) before each commit.
Note: earlier loads on this origin leave older-schema data in localStorage —
that's a free migration test; check `state.schema` after reload.

---

## v0.15 — Natural-language quick add ("PS4 due fri #cs101 !high") ✅ SHIPPED

**No schema change (stays 11).** Inspired by Todoist: capture friction kills
task systems; one line should carry title + date + class + priority.

### Parser

```js
function parseQuickTask(raw) → { title, classId, due, priority }
```

Tokenize on whitespace; consume recognized tokens; everything left
(whitespace-collapsed, trimmed) is the title. All matching case-insensitive.

| Token | Meaning | Rules |
|---|---|---|
| `!high` `!h` / `!med` `!m` / `!low` `!l` | priority | first one wins; later ones stay in title. Default `med`. |
| `#xyz` | class | must be preceded by start/whitespace. Match against `class.code` (case-insensitive, ignoring spaces) first, then exact `class.name`, then unique name prefix. **No match → token stays in the title** (never auto-create a class). |
| `today` `tod` | due today | |
| `tomorrow` `tmr` `tom` | due tomorrow | |
| `mon`…`sun`, `monday`…`sunday` | next occurrence | `daysAhead = (target − todayDow + 7) % 7`; 0 means **today**. Standalone word only. |
| `M/D` or `MM/DD` | explicit date | current year; if the date already passed, use next year. Validate month 1–12 / day 1–31 else leave in title. |
| `+Nd` | N days from now | e.g. `+3d` |
| standalone `due` | dropped | only when the **next** token parses as a date; otherwise stays in title ("pay dues" must survive). |

First date token wins; later date tokens stay in title. Times (`11:59`,
`5pm`) are **not** parsed — leave them in the title (data model has no
time-of-day). Empty title after parsing → return `null` (caller ignores).

### Wiring

1. **Today `#quickInput`:** Enter → `parseQuickTask` → `addTask(t.title,
   t.classId, t.due, t.priority)`. Add a live preview row `#nlPreview`
   under the input (render on `input` event): small chips for parsed date
   (`fmtDue`), class (dot in class color + code), priority (existing
   `.chip.high/.med/.low` classes). Hidden when nothing parses.
2. **Command palette:** the quick-add last row runs the same parser; its
   `sub` label shows the parsed pieces, e.g. `NEW TASK · Fri Jul 17 · CS101
   · HIGH`, and `run` uses the parsed fields.
3. **Assignments form: unchanged** (explicit inputs stay the predictable
   path).

### CSS

`.nl-prev{display:flex;gap:6px;margin-top:8px;flex-wrap:wrap}` + reuse
existing chip classes. Nothing else.

### Verify (browser console)

`parseQuickTask` cases — assert each:
`"PS4 due fri #cs101 !high"` → title "PS4", due = next Fri, class = CS101,
high · `"buy milk"` → all defaults · `"pay dues tomorrow"` → title "pay
dues", due tomorrow · `"read ch 3 #nope"` → "#nope" stays in title ·
`"quiz 7/30 !m #cs101"` → explicit date · `"gym +2d"` · UI: type in
quickInput, see preview chips, Enter creates task with fields set; palette
row shows parsed sub-label. Zero console errors.

**Definition of done:** parser + both surfaces + preview chips + docs updated
+ committed as `Campus OS v0.15 — natural-language quick add`.

---

## v0.16 — Schedule-aware Today: "Plan my day" (SIGNATURE FEATURE) ✅ SHIPPED

**SCHEMA_VERSION → 12.** Inspired by Structured/Shovel/Sunsama: render
today's class meetings on a timeline, show free gaps, let tasks be planned
into the day. This must feel great — spend the effort here.

### Data model (schema 12)

```js
// classes[] gains:
meetings: [ { days:[0..6], start:"HH:MM", end:"HH:MM" } ]  // days 0=Mon (matches DAY_LETTERS); 24h times
// state gains:
plans: [ { id, date:"YYYY-MM-DD", taskId, start:"HH:MM", mins:Number } ]
```

- `migrate()` v11→12: every class gets `meetings: []`, then best-effort
  `parseScheduleString(c.schedule)` to populate it; add `plans: []`.
- `parseScheduleString(s)`: parse patterns like `"MWF 10am"`, `"TuTh 2:30pm"`,
  `"Mon/Wed 14:00"` → one meeting `{days, start, end:start+60min}`. Day
  tokens: `M`, `T`/`Tu`, `W`, `Th`/`R`, `F`, `Sa`, `Su` and full names.
  Unparseable → `[]` (user sets it in the UI; keep the legacy `schedule`
  string as a display fallback). Keep this parser ~30 lines, not perfect.
- `seed()`: give CS 101 `meetings:[{days:[0,2,4], start:"10:00", end:"10:50"}]`
  and Math `[{days:[1,3], start:"13:00", end:"14:15"}]` so the timeline demos.
- In `load()` after migrate: prune `plans` entries older than 14 days.

### Meeting editor UI

Extend the inline class form (`#clsForm` on Assignments): replace the free-
text schedule input's role — keep it, but add: 7 day toggles (reuse the
`.day-tgl` pattern/classes from the habits schedule editor) + two
`<input type="time">` (start/end). One meeting pattern per class in the UI
(edits `meetings[0]`); the array exists for future flexibility. Editing an
existing class's schedule: clicking a class filter chip's schedule area is
NOT required — add a small `✎` affordance later if asked; creating classes
with meetings is enough for v1.

### Today timeline UI

New full-width `.card` `#dayPlan` between the greet-card and `.today-grid`:

- **Vertical timeline.** Range: `min(08:00, firstBlock−1h)` →
  `max(20:00, lastBlock+1h)`. Constant `PX_PER_MIN = 0.75` (tune ±). Hour
  gridlines with `--line`, hour labels in `--mono` 9.5px `--ink-soft`.
  Container `position:relative`, blocks `position:absolute`, `max-height:
  420px`, `overflow-y:auto`, auto-scrolled so "now" is visible on load.
- **Class blocks:** background = class color at ~15% (use `color-mix(in srgb,
  <color> 15%, var(--card))` with a 4px solid left border in the class
  color), label = class name + `start–end`.
- **Plan blocks:** task title, `--green-soft` background, ✕ to unplan
  (`data-unplan`). If the underlying task is done → struck + 55% opacity.
- **Now line:** 2px `--flag` line + dot at current time (only when viewing
  today; the view always shows today — no date nav in v1).
- **Free-time budget** in the greet-card stats: `🕐 <b>3.5h</b> free` =
  sum of gaps ≥15min between now and 22:00 after subtracting remaining
  class meetings and plans. Hide when 0 or day over.
- **"＋ Plan" button** on each Due-soon task row (`data-plan="taskId"`):
  auto-places the task at the first free gap ≥30min after now (60min block
  default; if the gap is 30–59min, size to the gap). No gap left → append
  after the last block. Re-planning an already-planned task moves it.
  Planning is for **today only** in v1.
- **Empty states:** no meetings + no plans → friendly prompt: "No schedule
  yet — add class times in Assignments → ＋ class, or plan a task from Due
  soon." Weekend with no items: "Nothing scheduled — enjoy it 🌿".
- Overlaps: don't solve column layout; offset overlapping blocks
  `margin-left:12px` per collision and let them stack visually.

### Functions

`parseScheduleString(s)`, `meetingsOn(date)` → `[{class, start, end}]`,
`dayItems(date)` → merged sorted blocks (meetings + plans), `freeGaps(date,
fromMin, toMin)`, `planTask(taskId)`, `removePlan(id)`, `hmToMin("HH:MM")`,
`minToHM`, `renderDayPlan()` called from `renderToday()`.

### Verify

Migration v11→12 (existing localStorage upgrades; CS 101 got parsed
meetings from "MWF 10am"); timeline renders today's blocks at correct
offsets (assert via `getBoundingClientRect` math or offsetTop); ＋ Plan
places into a real gap and persists across reload; unplan works; free-hours
stat matches hand-computed gaps; class create with day toggles + times
produces correct `meetings`; done task strikes its plan block; empty states;
dark scheme; zero console errors.

**Definition of done:** all the above + ARCHITECTURE v0.16 section + commit
`Campus OS v0.16 — schedule-aware Today (plan my day)`.

---

## v0.17 — Grades: current grade + "what do I need on the final" ✅ SHIPPED

**SCHEMA_VERSION → 13.** The classic student hook (MyStudyLife, every
grade-calculator site). Uses the existing classes DB.

### Data model (schema 13)

```js
// classes[] gains:
grading: null | {
  target: 90,                      // desired course %
  cats: [ { id, name:"Homework", weight:30, isFinal:false,
            scores: [ { id, label:"HW1", got:18, max:20 } ] } ]
}
```

`migrate()` v12→13: every class gets `grading: null` (null = not set up).
Seed: give CS 101 a small demo grading setup (HW 30% with two scores,
Midterm 30% with one, Final 40% `isFinal:true`, target 90).

### Math (define once, comment it)

- Category average: `avg_i = Σgot/Σmax × 100` over its scores.
- **Current grade** = weighted mean over categories **that have ≥1 score**,
  normalized by the sum of their weights (standard "grade so far").
- **Needed on final**: `wF` = weight of the `isFinal` cat ÷ 100; `cur` =
  current grade over non-final scored cats (same normalization). Assume
  scoreless non-final cats finish at `cur`. Then
  `needed = (target − cur × (1 − wF)) / wF`. Display: `>100` → "Target
  {target}% isn't reachable — best case {best}%"; `≤0` → "Already secured
  🎉"; else the %.
- Letters: `A≥93, A−≥90, B+≥87, B≥83, B−≥80, C+≥77, C≥73, C−≥70, D+≥67,
  D≥63, D−≥60, F` — constant `LETTERS` table.
- Weights not summing to 100 → normalize by entered sum AND show a warning
  chip "weights sum to N%".

### UI — new view `Grades`

Nav: MAIN group after Money, **no digit shortcut**; add to `PAL_VIEWS`
(`📊 Grades`). Section `#view-grades`, eyebrow `TRANSCRIPT`.

- **Class chips row** (reuse filter-chip pattern, dot in class color).
  Selected-class detail below; classes with `grading` show a mini grade
  badge on their chip (`91% A−`).
- **Detail:** stat cards (reuse `.stats/.stat`): CURRENT GRADE (big % +
  letter), TARGET (inline editable number like Money's budget input),
  NEEDED ON FINAL (the hero — big, green if ≤ current avg, amber if
  ≤100, flag-colored message if unreachable).
- **Categories table/card:** each cat row: name, weight (%), `FINAL` gold
  chip toggle (`data-gfinal`, exactly one allowed — setting one clears
  others), avg, delete ✕. Scores render as small chips inside the cat row
  (`18/20 HW1`, ✕ to delete). Inline add-score form per cat (label, got,
  max). Inline add-category form at bottom (name, weight).
- `grading: null` → onboarding empty state: "Set up grading for
  {class} — add categories from your syllabus" + Add first category form
  (creates `grading` with target 90).
- Actions: `setTarget`, `addCat`, `delCat`, `setFinalCat`, `addScore`,
  `delScore` — all via delegated `data-g*` attributes; every mutation
  `save(); render()`.

### Verify

Hand-check the math with the seed data (compute expected current % and
needed-on-final on paper and assert equality to 1 decimal); weights-≠-100
warning; final-cat exclusivity; unreachable + secured messages; migration
v12→13; palette entry; dark scheme; zero console errors. Commit
`Campus OS v0.17 — grades + what-do-I-need-on-the-final`.

---

## v0.18 — Flashcards: spaced repetition from notes

**SCHEMA_VERSION → 14.** Anki-style retrieval practice sourced from the
notes the user already writes. Differentiator: planner + linked notes + SRS
in one local app.

### Authoring format (derived, not duplicated)

Cards are **derived from notes at read time** — the note stays the source
of truth. In any note body, an adjacent pair of lines:

```
Q: What's the complexity of merge sort?
A: O(n log n)
```

One-line Q, one-line A (v1; multi-line answers are out of scope). Parse
with a line scanner: `^Q:\s*(.+)$` followed (next non-empty line) by
`^A:\s*(.+)$`.

### SRS state (schema 14)

```js
state.cards = { [key]: { ease, ivl, due:"YYYY-MM-DD", reps, lapses } }
// key = noteId + "|" + Q-text trimmed+lowercased
```

Derivation: `noteCards(note)` → `[{key, q, a, noteId, noteTitle}]`;
`allCards()` scans all notes; a card with no `state.cards[key]` entry is
NEW (due now). Editing a Q creates a new key (old entry orphaned — prune
entries whose `noteId` no longer exists in `load()`; text-orphans are
harmless, ignore).

**Scheduler (SM-2 lite, date-granularity — comment the table):**
start `ease = 2.5`. On grade:

| Grade | Effect |
|---|---|
| Again | `ivl = 1`, `ease = max(1.3, ease − 0.2)`, `lapses++`, due tomorrow |
| Hard | `ivl = max(1, round(ivl × 1.2))`, `ease = max(1.3, ease − 0.15)` |
| Good | reps 0 → `ivl = 1`; reps 1 → `ivl = 3`; else `ivl = round(ivl × ease)` |
| Easy | `ivl = max(2, round(ivl × ease × 1.3))`, `ease += 0.15` |

`reps++` on non-Again grades; cap `ivl` at 365; `due = today + ivl days`.
`dueCards()` = derived cards with no entry OR `entry.due <= todayStr()`.

### UI — new view `Study`

Nav: MAIN after Notes, no digit key; `PAL_VIEWS` entry (`🧠 Study`).

- **Idle state:** hero card "N cards due · M total across K notes",
  `Study now` primary button; below, per-note breakdown rows (note title,
  due/total, click → study only that note's cards). Empty state teaches
  the format: "Write `Q:` / `A:` line pairs in any note — they become
  flashcards here." + button to open Notes.
- **Session state (module vars, not persisted):** show Q (rendered through
  `esc` + `renderInline`), `Show answer` → reveals A + four grade buttons
  (Again / Hard / Good / Easy — flag/amber/green/green-deep accents),
  progress `3/12`, End session button. Session order: shuffle due cards.
  Finish → back to idle with "Done — next cards due {date}".
- **Today view:** if `dueCards().length > 0`, show a nudge-style button
  (reuse `.nudge`): `🧠 12 cards due — quick review?` → `switchView("study")`.
- **Notes preview (stretch, optional):** style `Q:`/`A:` lines with a
  subtle left border in preview.

### Verify

Note with 2 Q/A pairs → Study shows 2 due; grade all four buttons and
assert `state.cards` entries (ease/ivl/due math per table); Again brings
card back tomorrow (simulate by editing `due`); prune-on-load removes
entries for deleted notes; migration v13→14; nudge on Today appears/hides;
zero console errors. Commit `Campus OS v0.18 — flashcards (spaced
repetition from notes)`.

---

## v0.19 — PWA: installable + offline

**No schema change.** Hosting is live (GitHub Pages). Goal: install to
phone home screen, work offline. Push notifications are **out of scope**
(needs a server); scheduled local notifications aren't broadly supported —
document this in ARCHITECTURE and move on.

### Files (this version adds real files next to index.html in `app/`)

1. **`manifest.webmanifest`:** `name: "Campus OS"`, `short_name: "CampusOS"`,
   `start_url: "./index.html"`, `scope: "./"`, `display: "standalone"`,
   `background_color: "#F5F7F2"`, `theme_color: "#152420"`, icons 192 +
   512 (+ `maskable` purpose).
2. **Icons:** `app/icons/icon-192.png`, `icon-512.png`, `apple-touch-icon.png`
   (180). Generate: write an SVG (pine `#152420` rounded square, `C` in
   Space-Grotesk-ish bold white + `OS` in `#5FC98F`), rasterize via
   PowerShell/.NET or any available tool; if rasterization is impossible in
   the session, commit the SVG as source AND use a Canvas-based generator
   page run once in the browser pane to produce PNGs (download → save into
   repo). Do not ship without real PNGs — iOS needs them.
3. **`app/sw.js`:** versioned cache name `campus-os-vN` (bump N every
   release from now on — add to the release checklist). Strategy:
   **network-first for `index.html`** (fall back to cache offline — the app
   is one file that changes every release), **cache-first** for manifest +
   icons. On `activate`, delete old caches, `clients.claim()`. Skip
   cross-origin (Google Fonts) requests — let them fail gracefully offline
   (font-family fallbacks already exist).
4. **`index.html` additions:** `<link rel="manifest" href="manifest.webmanifest">`,
   `<meta name="theme-color" content="#152420">`, apple-touch-icon link,
   `apple-mobile-web-app-capable`; boot-time SW registration guarded:
   `if ("serviceWorker" in navigator && location.protocol.startsWith("http"))`
   → `navigator.serviceWorker.register("./sw.js")`. file:// usage is
   unaffected (guard skips).
5. **Settings → "Install as app" block:** capture `beforeinstallprompt`
   into a module var; show an Install button when available (Chromium);
   below it, static iOS instructions ("Safari → Share → Add to Home
   Screen"). Show "✓ Installed" when `display-mode: standalone` matches.

### Root redirect note

The repo-root `index.html` (outside `app/`) already redirects into the app
— it stays as-is; SW scope `./` (the app folder) doesn't touch it.

### Verify

Serve over localhost (SW works on localhost): manifest fetches 200 with
correct MIME; `navigator.serviceWorker.controller` present after reload;
DevTools/CDP offline → reload still renders the app; cache name matches
current version; `beforeinstallprompt` path doesn't error where the event
never fires. After push, spot-check the live Pages URL headers. Commit
`Campus OS v0.19 — PWA: installable + offline`. Update README Quick start
("installable — Add to Home Screen").

---

## Version/schema ledger

| Version | Feature | SCHEMA_VERSION | New state |
|---|---|---|---|
| v0.15 | NL quick add | 11 (unchanged) | — |
| v0.16 | Plan my day | 12 | `classes[].meetings`, `plans[]` |
| v0.17 | Grades | 13 | `classes[].grading` |
| v0.18 | Flashcards | 14 | `cards{}` |
| v0.19 | PWA | 14 (unchanged) | — (adds files: manifest, sw.js, icons) |

Import/export must round-trip cleanly at every step (`migrate()` chains
handle old backups). Friend-code payload (`myStats`) is **unchanged** by
all five features — do not touch its shape.

## Out of scope for these five (do not gold-plate)

Time-of-day on tasks; multi-meeting-per-class editor UI; plan-for-future-
days; grade GPA across semesters; multi-line flashcard answers; cloze
deletions; push notifications; recap-text changes (money stays out of the
share text — privacy rule from v0.14).

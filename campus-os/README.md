# Campus OS

A lightweight, Notion-style personal workspace built for a busy CS freshman:
classes, assignments, internship deadlines, workouts, habits, notes with code
blocks — all in one local app with zero dependencies.

## Quick start

**Live app:** https://miguel-vargas-07.github.io/campus-os/ (hosted on GitHub Pages)

1. Or open `app/index.html` in any browser (double-click it). That's the whole app.
2. Your data saves automatically to your browser's localStorage.
3. Use **Export** (in Settings) regularly to save a JSON backup of your data.
   **Import** restores it on any machine or after clearing your browser.

## How to resume this project with Claude in a future session

The workspace Claude uses resets between sessions, so this folder is the
project's memory. To continue:

1. Upload this folder (or the zip of it) to a new Claude conversation.
2. Say: **"Continue the Campus OS project. Read PROJECT_STATUS.md first."**
3. Claude will read `PROJECT_STATUS.md`, `docs/ARCHITECTURE.md`, and
   `docs/ROADMAP.md` and pick up where we left off — no re-explaining needed.

If you only want a small change, uploading just `app/index.html` +
`PROJECT_STATUS.md` is enough.

## Folder map

```
campus-os/
├── README.md            ← you are here (quick start + resume instructions)
├── PROJECT_STATUS.md    ← current state of the project; read this first when resuming
├── app/
│   └── index.html       ← the entire app (HTML + CSS + JS, self-contained)
└── docs/
    ├── ARCHITECTURE.md  ← how the code is organized, data model, key functions
    ├── DESIGN.md        ← design tokens (colors, fonts) and UI rules
    └── ROADMAP.md       ← planned features, in priority order
```

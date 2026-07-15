# DESIGN — Campus OS (v0.2 tokens)

Direction: **engineer's field notebook, modernized**. Graph-paper page, ink
text, monospace data chips — now with a dark pine sidebar, soft gradients,
and a distinct, quieter visual register for the Faith page. Calm enough to
live in daily; structured enough to feel like a tool.

## Tokens (CSS variables in `:root` of app/index.html)

| Token          | Value     | Use |
|----------------|-----------|-----|
| `--paper`      | `#F5F7F2` | page background (faint 28px graph grid) |
| `--card`       | `#FFFFFF` | cards, rows, inputs |
| `--ink`        | `#1C2823` | primary text |
| `--ink-soft`   | `#66756B` | secondary text |
| `--line`       | `#E2E8DF` | borders, grid |
| `--green`      | `#1E7A4B` | primary accent (pair with `--green-deep #166B41` in gradients) |
| `--green-soft` | `#E4F1E9` | accent backgrounds |
| `--pine`       | `#152420` | sidebar / dark surfaces (`--pine-2 #1D3129`) |
| `--pine-text`  | `#B9C9BF` | text on pine |
| `--flag`       | `#D95B2B` | high priority, overdue, destructive |
| `--amber`      | `#B98A1C` | medium priority, today marker |
| `--gold`       | `#C9A227` | Faith accent ONLY (`--gold-soft #F7F0DC`) |

Radii: `--r:14px` cards, `--r-sm:8px` buttons/inputs, `999px` chips.
Shadow: `--shadow` only. Gradients allowed: green→green-deep (primary
actions, filled habit cells), pine→pine-2 (sidebar, hero cards).

## Type

- **Display / headings:** Space Grotesk 500–700
- **Body / UI:** system stack
- **Data / code:** JetBrains Mono — chips, dates, streaks, eyebrows, code
- **Scripture / quotes:** Fraunces (serif) — used ONLY on the Faith page

## Signature elements

1. Graph-paper grid page background.
2. Monospace status chips + eyebrow labels (`DATABASE`, `QUIET TIME`…).
3. Dark pine sidebar with grouped nav: MAIN / INNER / SYSTEM.
4. Today greeting hero (time-of-day + stats) on a pine gradient.
5. Reflect: 14-day mood arc (bar heights = mood 1–5).
6. Faith: verse-of-the-day card, pine→moss gradient with a soft gold glow;
   gold + serif appear nowhere else in the app.

## Dark theme (v0.4)

Dark tokens live under `html[data-theme="dark"]`: paper `#0F1714`, card
`#17221D`, ink `#E3ECE5`, line `#243129`, green brightened to `#2E9B63`
for contrast. Sidebar deepens rather than inverts. Use tokens only and dark
mode comes free; any hardcoded light-only color needs a dark override in
that block.

## Color schemes (v0.5)

`field` (light) and `pine` (dark) are the canonical brand schemes and match
the token tables above. Additional schemes (Paper White, Solar, Fjord,
Nocturne, Ember) live in the `SCHEMES` object in app/index.html and override
tokens via inline CSS vars — they are palettes only; layout, type, radius,
and motion never change per scheme. New schemes must define enough contrast
for chips and keep gold readable for the Faith page.

## Rules

- No new colors or fonts without updating this file. Gold stays on Faith.
- Motion: 120–220ms ease; one view-switch rise animation; respect
  `prefers-reduced-motion`.
- Copy: sentence case, plain verbs. Faith/Reflect copy is gentle, never
  gamified (no "crush your prayer streak").

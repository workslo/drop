# Panel System

A quiet, neutral-first interface system read off one live page: white panels, hairline borders, one ink, one muted grey, Geist at 15px, everything on a 4px grid.

This system was built from a Chrome DevTools AI Assistance session on 30 September 2026, which read computed styles from a single `section.rounded-panel` and a sample of the elements around it. It is a reverse-engineered snapshot, not the source's own design system. Every token says in its note whether it was **measured** (read from computed or authored styles) or **inferred** (a class name or a guess with no value behind it). Treat inferred values as placeholders until confirmed.

## Principles

1. **Borders, not shadows.** Panels separate from the page with a 1px `line` border and `radius-panel`. Nothing measured carries a box-shadow.
2. **Hierarchy by weight and colour before size.** The type scale spans only 12–26px. Rank comes from 600 vs 500 vs 400 and from `ink` vs `muted`.
3. **Multiples of four.** Every padding, gap and margin is `calc(var(--spacing) * n)` with `--spacing: 0.25rem`.
4. **Containers don't pad; their parts do.** A panel has zero padding so its internal dividers run edge to edge. The header and body each set their own inset.
5. **Motion is feedback, not decoration.** 150ms, one curve, specific properties.

## Colour

Five tokens, one light theme. No dark theme was captured.

| Token | Value | Status | Use |
|---|---|---|---|
| `panel` | #ffffff | measured | Panel surfaces |
| `ink` | #0e1116 | measured | Primary text and active icons |
| `muted` | #555c69 | measured | Secondary text, inactive icons |
| `line` | #e5e7eb | **inferred** | Borders and dividers |
| `panel-deep` | #f3f4f6 | **inferred** | Hover fill |

No brand or accent colour was read. The page uses a `brand` token for focus outlines and the active-nav underline, but its value never came back. Add it once you have it.

Contrast: `ink` on `panel` is about 18:1; `muted` on `panel` is about 6.6:1, safe for 12px labels.

## Typography

One family, Geist, loaded from Google Fonts or self-hosted, with `ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif` as fallback.

| Style | Size / leading | Weight | Where |
|---|---|---|---|
| `title` | 26 / 29.9 | 600 | Page title, one per view |
| `section` | 16 / 24 | 600 | Section heading inside a panel |
| `body` | 15 / 24 | 400 | Default text |
| `label-strong` | 14 / 20 | 600 | Strong labels |
| `label` | 14 / 20 | 500 | Nav links |
| `caption-strong` | 12 / 16 | 600 | Panel header titles, in ink |
| `caption` | 12 / 16 | 500 | Wayfinding labels, in muted |

Body is 15px, not 16. Body leading is 1.6; the title's is about 1.15.

## Spacing and layout

- Base unit `--spacing: 0.25rem`. Steps in use: 8 (header gap), 12 (header right inset, link padding), 16 (panel body), 24 (above a section heading).
- **The panel:** `section` with `border: 1px solid var(--line)`, `border-radius: var(--radius-panel)`, `background: var(--panel)`, padding 0, and `min-width: 0; flex-grow: 1` so long content can't blow out its flex parent. Add `overflow: hidden` so the header divider meets the corners.
- **Panel header:** `display: flex; flex-wrap: wrap; align-items: center; gap: 8px; padding-right: 12px; border-bottom: 1px solid var(--line)`.
- **Panel body:** `padding: 16px`.
- Layout is fluid, with no max-width container found. The page uses CSS container queries (`@container`) on at least one column.

## Responsive

Two media queries were found: `(max-width: 599.95px)` and `(min-width: 600px)`. That fits a 600px mobile/desktop split. An `lg:` variant is also in use (`lg:grow`), but its width wasn't read, so don't assume 1024px. A custom `shell:` variant also switches the nav between a mobile list and a desktop bar. Its trigger wasn't read either.

## Iconography

- Nav and UI icons are 20×20 and filled, coloured by `currentColor` from `ink` (active) or `muted` (idle).
- Idle icons turn `ink` on the parent link's hover (`group-hover:text-ink`).
- Utility icons are 18×18, stroked at 1.7px in `muted`.
- Every icon in a flex row gets `flex-shrink: 0`.
- Only ten SVGs were sampled. The icon set's name and source weren't identified.

## Motion

- Duration 150ms, easing `cubic-bezier(0.4, 0, 0.2, 1)`, on every transition sampled (ten elements).
- Nav links transition `background-color` only. Icon buttons transition the colour group. The skip link transitions `transform`, sliding in on focus.
- No `animation` was found. No reduced-motion rule was checked; add one.

## Interaction and accessibility

- Focus: `outline: 2px solid var(--brand); outline-offset: 2px` on `:focus-visible`.
- Hit areas: icon buttons are at least 44×44 (`min-h-11 min-w-11`), and nav rows are at least 48px tall on mobile.
- A skip-to-content link sits at the top, hidden above the viewport until focused.

## What the DevTools read claimed but didn't measure

When you hand this to a coding agent, don't let it carry these forward as fact:

- "All transitions are 150ms." The read sampled ten elements.
- "Probably View Transitions." Nothing was found for this.
- `--color-line` as #E5E7EB, and `--radius-control` as about 6px. Both were guesses.
- `lg` as 1024px. This is the Tailwind default, not what the page uses.
- The token dump failed (output too long), and the first ten "colour" variables it returned belonged to an embedded VS Code/Monaco editor, not the site. The page hosts a code editor, so some styles nearby may be Monaco's.

## For a coding agent doing a design-system pass

1. Replace hard-coded hex values with the colour tokens. Replace pixel paddings with `calc(var(--spacing) * n)`.
2. Build panels with the panel/header/body pattern above, not with margins between sections.
3. Set one transition timing everywhere: `150ms cubic-bezier(0.4, 0, 0.2, 1)`, naming the properties and never using `all`.
4. Give every icon in a flex row `shrink-0`, and size it from `icon-md` or `icon-sm`.
5. Before shipping, confirm the three inferred values (`line`, `panel-deep`, `radius-control`) and add `brand`.

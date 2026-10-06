# Panel

A bordered, unpadded container that delegates its spacing to a header row and a body, so the header divider runs edge to edge.

## Anatomy

- **Panel** — `section`: `background: var(--panel)`, `border: 1px solid var(--line)`, `border-radius: var(--radius-panel)`, `padding: 0`, `overflow: hidden`, and `min-width: 0; flex-grow: 1` inside a flex column.
- **Header** — `display: flex; flex-wrap: wrap; align-items: center; gap: var(--space-2); padding-right: var(--space-3); border-bottom: 1px solid var(--line)`. Title in `caption-strong` (12/16, 600, ink), with `flex: 1; min-width: 0` so it truncates before the actions wrap.
- **Body** — `padding: var(--space-4)`, text in `body` (15/24).

## Rules

- No shadow, ever. Depth comes from the border.
- No margin between header and body; the border does the separating.
- Header actions are icon buttons at least 44×44 (`hit-min`). Icons `icon-md`, colour `muted`, hover to `ink` over `panel-deep`, 150ms.
- The consumer provides the header's left inset. The source's header has none because its first child (a tab or title control) brings its own padding.

# Style Guide — Light Mode

Companion to `STYLE_GUIDE_DARK.md`. Aggregated from this extension's actual
design tokens (`css/default.css`, `css/panel-shared.css`) and every usage
pattern verified while building the Matrix panel — every contrast ratio
below is **computed** (WCAG relative-luminance formula against the real
token RGB values), not eyeballed. See A8 in
`Xplainer_Programming_principles_audit.md` for the standing rule this
guide exists to satisfy.

Minimum bar used throughout: **4.5:1** for normal body/data text, **3:1**
for large/bold text (≥ ~1.1em bold, roughly WCAG's "large text" allowance)
and non-text UI components (icon-only controls, meaningful borders).
Anything below 3:1 in either mode is a fail — not "acceptable if the other
mode passes."

## Core tokens (verified)

| Token | Light value | vs `--surface-1` | vs `--surface-2` | Status |
|---|---|---|---|---|
| `--ink-1` (text) | `rgb(32,18,58)` | 15.26:1 | 13.44:1 | ✅ text-safe everywhere |
| `--color-red` | `#e74c3c` | 3.36:1 | 2.96:1 | ⚠️ large/bold text only (see below) |
| `--color-green` | `#27ae60` | 2.52:1 | 2.22:1 | ❌ **not text-safe** — fill/border use only |
| `--color-amber` | `#f39c12` | 1.93:1 | 1.70:1 | ❌ **not text-safe** — fill/border use only |
| `--border-1` | `rgb(184,184,192)` | 1.73:1 | 1.52:1 | decorative divider only, not a meaningful boundary |
| `--border-4` | `rgb(144,144,156)` | 2.77:1 | 2.44:1 | strongest of the border tiers, still not 3:1 — use sparingly for anything meaningful |

**Known gap, not yet fixed everywhere:** `--color-green`/`--color-amber`
were only ever WCAG-tuned for dark-mode text (see
`css/panel-shared.css`'s own comment on the dark variant). In light mode
they're genuinely unreadable as text — confirmed by direct computation,
not assumption. `matrix/matrix-panel.css` fixes this locally with
`--color-green-text` / `--color-amber-text` (below); other files using
`color: var(--color-green)` or `color: var(--color-amber)` as actual text
(not a fill or border) likely have the same bug and haven't been audited
yet — check before reusing that pattern elsewhere.

## Text-safe variants (introduced for the Matrix panel, reusable pattern)

When a token needs to convey red/green/amber semantics **as text or a bold
glyph** (not a filled square, not a border) in light mode, don't use the
raw shared token — use a darkened variant verified against the actual
background:

| Token | Light value | vs `--surface-1` | Status |
|---|---|---|---|
| `--color-green-text` | `rgb(22,115,66)` | 5.18:1 | ✅ passes 4.5:1 |
| `--color-amber-text` | `rgb(150,94,4)` | 4.73:1 | ✅ passes 4.5:1 |
| `--color-red` (as-is) | `#e74c3c` | 3.36:1 | ✅ passes 3:1 (large/bold text only — this is why `.td-check`'s checkmark can stay large+bold rather than needing its own red-text variant) |

If a new component needs this pattern, define the same two tokens
locally (see `matrix-panel.css`'s own `:root` block) rather than changing
the shared `--color-green`/`--color-amber` globally — those are still
correct for dark-mode text and for fill/border usage in both modes; only
light-mode *text* usage of green/amber needs the darker variant.

## Proven-safe de-emphasis opacities (light mode)

For secondary/de-emphasized text via `color-mix(in srgb, currentColor
NN%, transparent)` against `--surface-1`, starting from `--ink-1`:

| Opacity | Resulting ratio | Status | Example use |
|---|---|---|---|
| 25% | 1.62:1 | ❌ fails even 3:1 — do not use | (was `.td-check.empty`, fixed) |
| 35% | 2.18:1 | ❌ fails 3:1 | — |
| 55% | 3.80:1 | ✅ passes 3:1, close to 4.5:1 | `.matrix-stats`, `.td-check.empty` |
| 65% | 5.21:1 | ✅ passes 4.5:1 | table header labels |
| 75% | 7.26:1 | ✅ comfortably passes | `.row-child .td-domain` |

**Rule of thumb for light mode:** don't go below 55% opacity for anything
meant to be read, even briefly. 25–35% ranges look like "intentional
subtlety" in isolation but are measurably unreadable.

## Component patterns (light mode)

- **Buttons** (`.monitor-btn` family): background `var(--surface-2)`,
  border `var(--border-1)`, text inherits `--ink-1` (always safe — verify
  any colored button variant's text against its own background, not
  against `--surface-1`).
- **Table headers** (sticky, `background-color: var(--surface-2)`): label
  text at 65% opacity (5.21:1 against surface-2) — do not go lower.
- **Mini-squares / fill badges**: raw `--color-red/green/amber` as
  `background-color` is fine (not a text-contrast case — the square's own
  border plus its area against the row background is what needs to be
  distinguishable, not glyph-level contrast).
- **Borders as dividers** (not meaningful boundaries): `--border-1` is
  fine — it's decorative, not conveying information on its own.
- **Borders as meaningful boundaries** (e.g. "this needs to visually read
  as a real division, not a subtle line"): prefer `--border-4`, still only
  2.77:1 in light mode — if genuine 3:1 UI-component contrast is required,
  neither existing border tier reaches it; don't assume `--border-4` is
  "the strong one, therefore compliant."

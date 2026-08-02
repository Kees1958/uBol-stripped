# Style Guide — Dark Mode

Companion to `STYLE_GUIDE_LIGHT.md`. Same methodology: every ratio below
is computed against the real `:root.dark` token values, not eyeballed.
See A8 in `Xplainer_Programming_principles_audit.md`.

**Prerequisite, not just a style point:** none of these dark-mode token
values take effect unless `js/ui/theme.js` is loaded on the page (applies
the `.dark`/`.light` class to `<html>`). Without it, backgrounds can still
go dark via the `@media (prefers-color-scheme: dark)` fallback (which only
covers `--surface-0/1/2/3`), while text/border/accent colors silently stay
at their light-mode values — dark background, light-mode text, unreadable.
This exact bug shipped twice in this codebase (Privacy Inspector, then the
Matrix panel) before being caught. Load `theme.js` via a `<script>` tag in
every panel's HTML (the standardized mechanism — see CHANGELOG 7.3.3)
before checking anything else on this page.

Minimum bar used throughout: **4.5:1** for normal body/data text, **3:1**
for large/bold text and non-text UI components.

## Core tokens (verified)

| Token | Dark value | vs `--surface-1` | vs `--surface-2` | Status |
|---|---|---|---|---|
| `--ink-1` (text) | `rgb(226,226,229)` | 13.23:1 | 10.21:1 | ✅ text-safe everywhere |
| `--color-red` | `#ff7070` | 6.36:1 | 4.91:1 | ✅ text-safe |
| `--color-green` | `#4ddb8a` | 9.62:1 | 7.43:1 | ✅ text-safe |
| `--color-amber` | `#ffc947` | 11.16:1 | 8.61:1 | ✅ text-safe |
| `--border-1` | `rgb(81,81,98)` | 2.20:1 | 1.70:1 | decorative divider only |
| `--border-3` | `rgb(105,105,121)` | 3.17:1 | 2.45:1 | passes 3:1 vs surface-1 only |
| `--border-4` | `rgb(118,118,133)` | 3.83:1 | 2.96:1 | strongest border tier, passes 3:1 vs surface-1 |

**Unlike light mode, `--color-red/green/amber` are already fully
text-safe here** — these were the tokens panel-shared.css's own comment
says were specifically re-tuned for dark-mode WCAG AA. No separate
`-text` variant is needed in dark mode; `--color-green-text` /
`--color-amber-text` (see the light guide) simply alias straight through
to these in `:root.dark` — don't redefine them differently here.

## Proven-safe de-emphasis opacities (dark mode)

Same `color-mix(in srgb, currentColor NN%, transparent)` pattern against
`--surface-1`, starting from dark-mode `--ink-1`:

| Opacity | Resulting ratio | Status | Example use |
|---|---|---|---|
| 25% | 2.03:1 | ❌ fails 3:1 — do not use | (was `.td-check.empty`, fixed) |
| 35% | 2.77:1 | ❌ fails 3:1 | — |
| 55% | 4.88:1 | ✅ passes 4.5:1 | `.matrix-stats`, `.td-check.empty` |
| 65% | 6.28:1 | ✅ comfortably passes | table header labels |
| 75% | 7.94:1 | ✅ comfortably passes | `.row-child .td-domain` |

Dark mode actually has *more* headroom than light mode at the same
opacity (compare 55% → 4.88:1 dark vs 3.80:1 light) — a value proven safe
in dark mode is not automatically proven safe in light mode at the same
percentage. Always check both; see A8.

## Component patterns (dark mode)

- **Buttons**: background `var(--surface-2)`, border `var(--border-1)` —
  border is only decorative-strength here (2.20:1), fine for a button
  outline that isn't the only affordance (text label + hover state carry
  the rest).
- **Table headers**: 65% opacity label text at 6.28:1 against
  `--surface-2` — well clear of the minimum, no adjustment needed.
- **Mini-squares / fill badges**: raw `--color-red/green/amber` as
  `background-color` — same reasoning as light mode, not a text-contrast
  case.
- **Timer near-expiry states** (`#timerDisplay.warning`/`.expired`): use
  `--color-amber-text` / `--color-red` directly — both already fully
  text-safe in dark mode via the alias-through above.

## Regression check for this exact bug class

Before shipping any new panel page, confirm:
- [ ] `js/ui/theme.js` is loaded via a `<script>` tag in the page's HTML
      (not a JS import — see the standardized mechanism note above)
- [ ] At least one element using each of `--ink-1`, `--border-1`, and any
      `--color-*` token actually renders with the `.dark` class applied
      before trusting that "it looks dark, so dark mode must be working"
      — a page can have correctly-dark *backgrounds* while every other
      token is silently still light-mode-valued

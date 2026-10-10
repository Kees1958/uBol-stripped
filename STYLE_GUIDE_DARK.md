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
| `--color-red-text` | alias of `--color-red` | 6.36:1 | 4.91:1 | ✅ text-safe (alias only — the darkened variant exists for light mode; see the light guide) |
| `--border-1` | `rgb(81,81,98)` | 2.20:1 | 1.70:1 | decorative divider only |
| `--border-3` | `rgb(105,105,121)` | 3.17:1 | 2.45:1 | passes 3:1 vs surface-1 only |
| `--border-4` | `rgb(118,118,133)` | 3.83:1 | 2.96:1 | strongest border tier, passes 3:1 vs surface-1 |

**Unlike light mode, `--color-red/green/amber` are already fully
text-safe here** — these were the tokens panel-shared.css's own comment
says were specifically re-tuned for dark-mode WCAG AA. No separate
`-text` variant is needed in dark mode; `--color-green-text` /
`--color-amber-text` (see the light guide) simply alias straight through
to these in `:root.dark` — don't redefine them differently here.

**Also reused by:** `js/scripting/cookie-clicker-picker.js`'s
`buildConfirmBubble()` — `--code-green: 77 219 138` (same as
`--color-green` here) on the "Save consent-click rule?" bubble's
`.selector` code block, replacing a hardcoded `#9fd39f` that happened
to look similar but was never actually verified against anything.

## Proven-safe de-emphasis opacities (dark mode)

Same `color-mix(in srgb, currentColor NN%, transparent)` pattern against
`--surface-1`, starting from dark-mode `--ink-1`:

| Opacity | Resulting ratio | Status | Example use |
|---|---|---|---|
| 25% | 2.03:1 | ❌ fails 3:1 — do not use | (was `.td-check.empty`, fixed) |
| 35% | 2.77:1 | ❌ fails 3:1 | (was `.rule-delete-btn`, fixed to 65% — 9.0.6) |
| 40% | 3.21:1 | ⚠️ passes 3:1 only, fails 4.5:1 | (was `.monitor-empty`, fixed to 65% — 9.0.6) |
| 50% | 4.27:1 | ⚠️ passes 3:1 only, fails 4.5:1 (close) | (was 4 table-header/label rules, fixed to 65% — 9.0.6) |
| 55% | 4.88:1 | ✅ passes 4.5:1 | `.matrix-stats`, `.td-check.empty` |
| 60% | 5.55:1 | ✅ passes 4.5:1 | (was 5 rules across 3 files — fine in dark, but bumped to 65% anyway since light mode failed at this value and both modes should share one value per site — 9.0.6) |
| 65% | 6.28:1 | ✅ comfortably passes | table header labels |
| 69% | 6.92:1 | ✅ comfortably passes | `label + legend` (settings.css) |
| 70% | 7.08:1 | ✅ comfortably passes | `.custom-rules-filter-row label`, `.static-filters-desc` and 5 others |
| 75% | 7.94:1 | ✅ comfortably passes | `.row-child .td-domain` |
| 85% | 9.85:1 | ✅ comfortably passes | `.col-tracker` |
| 90% | 10.91:1 | ✅ comfortably passes | `.cosmetic-selector-text` |

**Note:** dark mode's own numbers alone would tolerate 60% (5.55:1), but the
fix applied (9.0.6) bumped those sites to 65% in both modes together —
this project's own convention is one opacity value per site, shared across
themes, not a per-mode split; light mode's failure at 60% (4.44:1, see the
light guide) is what actually forced the change.

Status-colour text blends (dark mode): `color-mix(in srgb,
var(--color-amber) 50%, currentColor)` on amber-10% over surface-1 computes
to 9.56:1 (11.95:1 vs surface-1) — ✅ passes; used by `#monitorHelpWarning`
and `.custom-rules-hint`.

Dark mode actually has *more* headroom than light mode at the same
opacity (compare 55% → 4.88:1 dark vs 3.80:1 light) — a value proven safe
in dark mode is not automatically proven safe in light mode at the same
percentage. Always check both; see A8.

## Solid-fill button text (new — found while adding the Privacy Inspector's tiered block buttons)

Same finding as the light-mode guide, computed separately for dark-mode
fill values — the gap was actually worse here:

| Fill token | vs black text | vs white text | Status |
|---|---|---|---|
| `--color-red` (dark) | 7.80:1 | 2.69:1 | ✅ black passes, white badly fails |
| `--color-green` (dark) | 11.82:1 | 1.78:1 | ✅ black passes, white badly fails |
| `--color-amber` (dark) | 13.70:1 | 1.53:1 | ✅ black passes, white badly fails |

Dark-mode fill values are brighter (by design, for background/border
visibility — see this file's own core-token table), which makes them
*worse* as a white-text background, not better — white-on-`--color-amber`
(dark) bottoms out at 1.53:1, barely above "no contrast at all." Black
text is correct in both modes for these three solid-fill tokens; there is
no mode where white text on any of them passes.



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

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
| `--color-red-text` | `rgb(176,42,30)` | 5.77:1 (5.08:1 vs `--surface-2`, 3.86:1 vs `--surface-3`) | ✅ passes 4.5:1 on surface-1/2 — use for any small or non-bold red text (stop banner, Exit button label, Block column header, `.detail-rule-block`). Defined in `css/panel-shared.css`; dark mode aliases straight to `--color-red` |

If a new component needs this pattern, define the same two tokens
locally (see `matrix-panel.css`'s own `:root` block) rather than changing
the shared `--color-green`/`--color-amber` globally — those are still
correct for dark-mode text and for fill/border usage in both modes; only
light-mode *text* usage of green/amber needs the darker variant.

**Also reused by:** `js/scripting/cookie-clicker-picker.js`'s
`buildConfirmBubble()` — `--code-green: 22 115 66` on the "Save
consent-click rule?" bubble's `.selector` code block (was previously a
hardcoded `#9fd39f`, dark-only, never checked against a light
background at all — see that file's own comment on this value).

**9.0.6 fix — the same bug found live, not just in principle:**
`css/panel-shared.css`'s `.monitor-btn-resume` used raw `var(--color-green)`
as its own text color (2.52:1 light — confirmed unreadable, not just
theoretically risky). `panel-shared.css` didn't define
`--color-green-text` at all before this fix; added there now, value copied
verbatim from this table (reuse, not recompute — A9). Border/background
fill usage on the same rule was correct all along and untouched.

**9.0.6 fix — a proven fix never ported to a sibling file (C21):**
`privacy/privacy-inspector.css`'s `#monitorHelpWarning` already documents,
in its own comment, that an earlier `color-mix(in srgb, var(--color-amber)
80%, currentColor)` failed WCAG AA (2.62:1 light) and was corrected to
`50%` (4.98:1 light / 9.68:1 dark). `dashboard/custom-rules.css`'s
`.custom-rules-hint` — solving the identical "amber warning box, amber-
tinted text" problem — still had the old, already-disproven `80%` value.
Not a new bug; the same one, twice, because the fix was never carried over.
Now matches the proven `50%`.

## Proven-safe de-emphasis opacities (light mode)

For secondary/de-emphasized text via `color-mix(in srgb, currentColor
NN%, transparent)` against `--surface-1`, starting from `--ink-1`:

| Opacity | Resulting ratio | Status | Example use |
|---|---|---|---|
| 25% | 1.62:1 | ❌ fails even 3:1 — do not use | (was `.td-check.empty`, fixed) |
| 35% | 2.18:1 | ❌ fails 3:1 | (was `.rule-delete-btn`, fixed to 65% — 9.0.6) |
| 40% | 2.48:1 | ❌ fails even 3:1 | (was `.monitor-empty`, fixed to 65% — 9.0.6) |
| 50% | 3.28:1 | ⚠️ passes 3:1 only, fails 4.5:1 | (was 4 table-header/label rules, fixed to 65% — 9.0.6) |
| 55% | 3.80:1 | ✅ passes 3:1, close to 4.5:1 | `.matrix-stats`, `.td-check.empty` |
| 60% | 4.44:1 | ⚠️ borderline fails 4.5:1 (0.06 short) | (was 5 rules across 3 files, fixed to 65% — 9.0.6) |
| 65% | 5.21:1 | ✅ passes 4.5:1 | table header labels |
| 69% | 5.94:1 | ✅ passes 4.5:1 | `label + legend` (settings.css) |
| 70% | 6.14:1 | ✅ passes 4.5:1 | `.custom-rules-filter-row label`, `.static-filters-desc` and 5 others — all reuse this exact value (A9) |
| 75% | 7.26:1 | ✅ comfortably passes | `.row-child .td-domain` |

## Proven-safe status-colour text blends (light mode)

Blends of a status colour toward `currentColor` (≈ `--ink-1`) used as small
text, so the raw token never has to be used as text:

| Expression | Text on | Ratio | Status | Example use |
|---|---|---|---|---|
| `color-mix(in srgb, var(--color-amber) 50%, currentColor)` | amber 10% over surface-1 | 4.97:1 (5.32:1 vs surface-1) | ✅ passes 4.5:1 | `#monitorHelpWarning`, `.custom-rules-hint` |
| `color-mix(in srgb, var(--color-red) 70%, currentColor)` | surface-1 | 5.40:1 | ✅ passes 4.5:1 | `.custom-rules-clear:hover`, `.rule-delete-btn:hover` |
| 85% | 10.11:1 | ✅ comfortably passes | `.col-tracker` |
| 90% | 11.81:1 | ✅ comfortably passes | `.cosmetic-selector-text` |

**Rule of thumb for light mode:** don't go below 65% opacity for anything
meant to be read, even briefly, UNLESS the text genuinely qualifies for
WCAG's large/bold-text 3:1 allowance (≈14pt/18.66px+ AND bold) — 55% only
proves 3:1, not 4.5:1, and was found in practice applied to several small
(0.78–0.85em), non-bold-enough table-header and label rules that actually
needed the full 4.5:1 bar (9.0.6 audit — see CHANGELOG). 25–50% ranges
look like "intentional subtlety" in isolation but are measurably
unreadable or marginal.

## Solid-fill button text (new — found while adding the Privacy Inspector's tiered block buttons)

**Known gap, now fixed and recorded:** the existing "Block All" button
used white text on `--color-red` fill, never actually verified as a
solid-fill combination (a different question from `--color-red` used
*as text* against a surface, already covered in the table above).
Computed directly: white text on the `--color-red` fill reaches only
**3.82:1**, failing the 4.5:1 minimum for this button's actual text size
(0.85em — not large/bold, so the 3:1 exception doesn't apply). Black
text on the same fill reaches **5.50:1** — verified, and now what
`.monitor-btn-blockall`/`.monitor-btn-blocklow`/`.monitor-btn-blockmedium`
all use.

| Fill token | vs black text | vs white text | Status |
|---|---|---|---|
| `--color-red` (light) | 5.50:1 | 3.82:1 | ✅ black passes, white fails |
| `--color-green` (light) | 7.31:1 | 2.87:1 | ✅ black passes, white fails |
| `--color-amber` (light) | 9.58:1 | 2.19:1 | ✅ black passes, white fails |

**Rule of thumb for any future solid-fill button using these three
semantic tokens**: use black text, not white — none of the three passes
4.5:1 with white text in light mode, and black comfortably clears it on
all three every time.



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

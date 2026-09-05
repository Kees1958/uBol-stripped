# Architecture — folder tiers

## Related standing reference docs

- **`FILTER_PROCESSING_REFERENCE.md`** — per-filter-list build-time
  processing detail (what gets dropped/enriched, safe vs. unsafe rule
  types, module/call-chain). Kept in sync with `tools/build-rulesets.mjs`
  — update it in the same pass as any change to that script's
  `FILTER_LISTS` processing, not as an afterthought (this file went
  stale once already — see its own git history / the CHANGELOG entry
  that rewrote it for the AdGuard Base blacklist-model rearchitecture).
- **`STYLE_GUIDE_LIGHT.md`** / **`STYLE_GUIDE_DARK.md`** — verified
  contrast-ratio catalog for design tokens (see A8/A9).

## The five-tier model (SW / dashboard / popup dependency graph)

Files under `js/ui/`, `js/workflow/`, `js/core/`, `js/util/`, `js/data/`
follow a strict five-tier dependency model. Dependencies may only flow
**downward**:

```
ui/       →  workflow/, core/, util/, data/
workflow/ →  core/, data/, util/
core/     →  util/, data/
util/     →  (nothing — self-contained)
data/     →  (nothing inside the extension — only external APIs)
```

- **ui/** — dashboard/popup/picker-adjacent presentation helpers (DOM,
  i18n, theming, filter-editor rendering) used by the page contexts.
- **workflow/** — `background.js`, the service-worker orchestrator. Wires
  together core modules in response to extension lifecycle/events.
- **core/** — feature logic (rulesets, filtering modes, block-monitor,
  scripting-manager, etc.).
- **util/** — pure, stateless helpers with no dependencies inside the
  extension.
- **data/** — constants, storage access, message-type definitions,
  browser-API compatibility shims.

## Why `scripting/`, `security/`, `offscreen/` sit outside this model

Three additional top-level folders exist under `js/` that are **not**
part of the five-tier model above, and this is intentional rather than
an oversight:

- **`js/scripting/`** — code injected by `chrome.scripting.executeScript()`
  into the *page* context (content-script world), not loaded by the
  service worker's own module graph. Examples: `picker.js`,
  `css-generic.js`, `tool-overlay.js`.
- **`js/security/`** — individual security scriptlets (`sec-noeval.js`,
  `sec-nowebrtc.js`, etc.), also injected into the page context per-toggle
  by the scripting-manager. Each file is a self-contained unit registered
  independently, not a module imported by another tier.
- **`js/offscreen/`** — code that runs inside the extension's offscreen
  document (`compile-filters.html`), a separate execution context from
  both the service worker and any page. It communicates with the SW only
  via `chrome.runtime` messaging, never via direct import.

These three folders represent **separate execution contexts**, not
additional layers in the SW's own dependency graph. `core/scripting-manager.js`
(tier 3) is the sole bridge: it knows the *paths* of files in
`scripting/`/`security/` to register, but never imports their code
directly. Similarly, `core/compiled-filters.js` knows how to launch and
message the `offscreen/` document but doesn't import its code either.

Because of this, the downward-dependency rule doesn't apply *between*
these folders and the five tiers — it applies *within* each execution
context independently. Within `js/scripting/` and `js/security/`, files
are intentionally flat and independent (each is registered/injected on
its own). Within `js/offscreen/`, `compile-filters.js` is the entry point
that loads the other offscreen helpers.

See `js/workflow/background.js`'s STATE/GUARDS/TIMERS reference comment
for how the SW tracks the lifecycle of scripting registrations and the
offscreen document.

## DESIGN PRINCIPLE — once a solution is proven, sibling features must reuse it, not reinvent it

When two features solve the same underlying problem, and one of them has a
solution that's actually been proven correct — meaning it has a documented
history of being tested against a real failure mode, not just "looks right
on first read" — the other must adopt that same solution. Building a
second, different-looking answer to a problem this codebase has already
solved once is not an improvement; it's a new, unproven implementation of
something that didn't need re-deciding, and a second place for the same
class of bug to hide.

**Worked example — the empty-tab bug, Privacy Inspector vs. Tracker
Radar's DDG Tracker Radar monitor:**

Both features open a tab that must reliably close itself when its
session's time limit expires. Privacy Inspector's version of this had
*two* separate documented rounds of "stale empty tab" bugs (see
`CHANGELOG.md`, `6.0.7` and its later regression) before converging on:
a hard `Promise.race` timeout around the stop message
(`STOP_MESSAGE_TIMEOUT_MS`, `privacy/privacy-inspector.js`), and
carefully making sure `resume()` actually restarts the countdown clock.
That's a real, tested fix — but it was still a *single* mechanism: one
client-side clock, in one tab, is the only thing that ever decides to
close it.

Tracker Radar's `monitor-panel.js`/`block-monitor.js` was built
independently and ended up with something structurally stronger for the
exact same problem: the **background** is authoritative for expiry (a
primary `setTimeout` *plus* a `watchdogHandle` `setInterval` fallback for
SW restarts wiping the primary one), broadcasting `MSG_MONITOR_CLOSED`
the moment it fires — which the panel reacts to immediately, not on its
own timer — *and* an independent 15s poll as a second, unrelated safety
net. Two independent paths to the same guarantee, neither depending on
the tab's own JS timers surviving browser throttling for the full
session length.

When this was noticed, the fix was not to invent a third design for
Privacy Inspector — it was to port Tracker Radar's exact mechanism over:
`js/core/privacy-inspector.js` gained the same
`sessionTimeoutHandle`/`watchdogHandle` pair and `MSG_PRIVACY_INSPECTOR_CLOSED`
broadcast (mirroring `MSG_MONITOR_CLOSED`), and
`privacy/privacy-inspector.js` gained the same independent poll
(mirroring `monitor-panel.js`'s `refresh()`/`SESSION_POLL_MS`) — same
constants, same shape, same comments explaining why, adjusted only for
this module's own `chrome.storage.session`-based persistence. The
original client-side-clock fix stayed in place too, as a third layer,
not replaced — this is additive hardening, not a rewrite.

**The rule going forward:** before building a new mechanism for a
problem, check whether a sibling feature already has one — especially
one with CHANGELOG history showing it was actually hardened against a
real failure, not merely written once and assumed correct. If it does,
port it (adjusted only for genuinely different constraints, like storage
strategy), don't design a fresh alternative next to it.

## HARD CONSTRAINT — every `browser.scripting` call site must be site-mode-gated

Four separate, previously-undetected bugs (see `CHANGELOG.md`) were all the
exact same mistake in different files: a call into the `browser.scripting`
API that ran unconditionally, with no awareness of per-site filtering mode
at all — so turning a site OFF (or a temp-disable pause) stopped DNR
network blocking but left that call's own JS injection running anyway.

- `registerScriptletContentScriptsImpl()` and
  `registerCosmeticContentScriptsImpl()` (`js/core/scripting-manager.js`) —
  both originally used `matches: ['*://*/*']` unconditionally.
- `prepareSecurityScriptletDirectives()` (`js/core/compiled-filters.js`) —
  same gap for the Security & Privacy sec-* scriptlets.
- `injectCustomScriptletsForFrame()` (`js/core/custom-scriptlets.js`) — the
  4th occurrence, in the ABP-import scriptlet dispatch path, found only
  after a user reported that whitelisting a site didn't stop an imported
  scriptlet from running on it.

**The rule going forward:** any new or modified call to
`browser.scripting.registerContentScripts()`, `updateContentScripts()`, or
`executeScript()` MUST consult the current per-site filtering mode before
running, using one of the two existing, proven mechanisms — never a third:
- Declarative registration (`matches`/`excludeMatches` arrays, the shape
  `browser.scripting.registerContentScripts()`/`updateContentScripts()`
  take) → `buildSiteModeMatchScope()` (`js/core/compiled-filters.js`).
- Imperative per-call dispatch (`executeScript()`, called per-navigation
  with a specific `tabId`/`hostname` already in hand) →
  `getFilteringMode(hostname) === MODE_NONE` (`js/core/mode-manager.js`),
  as an early-return guard before the call.

Per the reuse-not-reinvent principle above: do not write a third check for
this. `tools/check-scripting-site-mode-gating.mjs` (part of `npm run
check`) enforces this automatically — see that file for exactly what it
looks for and how to satisfy it for a genuinely new call site.

## HARD CONSTRAINT — never reintroduce unpacked-only DNR feedback APIs

`chrome.declarativeNetRequest.onRuleMatchedDebug`,
`chrome.declarativeNetRequest.testMatchOutcome`, and
`chrome.declarativeNetRequest.getMatchedRules` are **permanently banned**
from this codebase, in background, content script, dashboard, popup, or
monitor code alike. No exceptions, no "just for a debug build," no
"feature-detect and fall back."

**Why:** all three are restricted by Chrome itself to *unpacked*
extensions only — confirmed directly against Chrome's own extension
API docs, not a guess or a bug in this project. A real user installing
this extension from the Chrome Web Store (i.e., nearly every real
install) gets a packed extension, for which these APIs silently never
fire. Any code path built on them works perfectly for a developer
testing unpacked, then does nothing at all for every actual user —
the single worst kind of latent bug this project can ship, because it
passes every manual dev test and fails 100% of the time in production.

**History:** an earlier version of this extension (`6.4.0`) *did* use
`onRuleMatchedDebug` in `js/core/block-monitor.js` as its blocked-status
signal. It was correct and worked well for local testing, and just as
silently useless for every packed install. It was removed and replaced
by two packed-extension-safe alternatives:
  - `js/core/static-block-check.js` — reads the extension's own bundled
    `rulesets/*.json` files directly (data already shipped, no
    restricted API) to catch whole-domain bundled-filter blocks. Known,
    documented, accepted limitation: whole-domain rules only, not
    path/pattern-specific ones.
  - The Block Monitor's own `blockedRules` check (`monitor/monitor-panel.js`)
    — tracks the tool's *own* saved custom rules client-side instead of
    asking Chrome for live match feedback.

Neither replacement is a like-for-like substitute — each only covers
the slice of "was this blocked?" that can be answered without the
banned APIs. That is an accepted, deliberate tradeoff: a documented
partial signal that works for real users beats a complete signal that
only works for developers. If a future gap is found in either
replacement (e.g. path-specific bundled rules, or url-kind custom
rules), the fix is to extend the packed-safe replacement's own JS
logic — never to reach for `onRuleMatchedDebug`/`testMatchOutcome`/
`getMatchedRules` as a shortcut back to "the real signal," no matter
how much simpler that would be to write.

## PITFALL — never assign layout meaning by document position alone

A two-column layout that decides which column an element belongs to
purely by *where it sits in the markup* (odd/even, nth-child, "every
other one") looks clean and needs zero extra markup — right up until
someone removes, adds, or reorders a single row on one side. Every row
after that point silently shifts into the wrong column. Nothing errors,
nothing lints, nothing shows up in a diff as obviously wrong — the bug
is purely visual, and only surfaces when a human actually looks at the
rendered page.

**Where this happened:** `dashboard/dashboard.html`'s Security & Privacy
tab (`#security-columns`) used to rely on exactly this — `.security-row`
elements alternated security/privacy/security/privacy in document order,
and `dashboard/settings.css` used `nth-child`/`nth-last-child` parity to
decide both which column a row landed in *and* which row closed off each
column's bottom border. It worked, silently, for as long as both columns
happened to have the same number of rows and nobody removed one from
only one side. Removing "Block free hosting & DDNS domains" (security
column) and the old "Auto-apply window name and tab protection" toggle
(privacy column) in `7.0.0` would have broken this immediately — the
two removals landed at different depths in the alternating sequence, so
simply deleting both rows would have shifted several *other*, untouched
rows into the wrong column with no error anywhere.

**The fix:** every `.security-row` now declares its own column
explicitly via a `data-column="security"|"privacy"` attribute, and the
CSS (`settings.css`) reads that attribute directly — `grid-column: 2` for
`data-column="privacy"`, nothing implied by position. The "last row in
this column gets a bottom border" rule was rewritten the same way, using
`:has()` to ask "does any *later* row share my own `data-column` value?"
instead of counting from the end of the whole list. Column placement and
row count on one side can now change freely without affecting the other.

**The rule going forward:** if a layout needs to know which group an
element belongs to — a column, a category, a phase — that grouping must
be a real, explicit attribute or class on the element itself. Document
order (`nth-child`, array index, "the third and sixth item") is allowed
to control *visual sequence*, but must never be the only thing encoding
*which group something is in*. If removing or reordering one item could
silently relabel a different, unrelated item, the grouping isn't
explicit enough yet.

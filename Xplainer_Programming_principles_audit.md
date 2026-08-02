# Chrome Extension — Full Audit Prompt

> **Trigger phrase:** *"Full extension audit"* — then upload your extension zip.

Paste this prompt at the start of a new chat, then upload your extension zip file.

---

I have a Chrome MV3 browser extension. Please apply a full audit covering HTML/CSS structure, code quality, and modular architecture. Do not change any logic or behaviour — only reorganise, extract, isolate, document and flag issues.

---

## Ground Rules

- **Do not change any logic or behaviour** — the extension must work identically after restructuring
- **Do not rename any message types, storage keys or public API functions** — these are contracts between modules
- **Do not silently change settings defaults or storage keys** — these can break existing users' saved settings
- **Show me a summary of all findings before making any changes**, so I can decide what to apply
- After every file change, verify HTML validity and run `node --check` on all JS files
- One file at a time — do not restructure multiple files in a single step
- **Always work from the exact source tree that already contains the latest verified functionality.** Never rebuild from an older version or a different branch, as this can silently reintroduce fixed bugs or remove completed features (**regression by source rollback**).
- **Before starting, identify and state the exact source version being used** (for example: `Working from v4.3.2 beta`).
- **If the latest verified source tree is unavailable, stop and ask the user for it.** Do not recreate the project from an older release or assume the missing functionality can be reproduced from a handover document.
- **Before packaging a release, verify that every previously completed feature expected in the chosen source version still exists.** A successful build or passing syntax check is not sufficient—confirm that no existing functionality has been lost through use of an older source tree.

### Release source verification

Before every release:

- [ ] Confirm the exact source version used.
- [ ] Confirm it is the latest verified source provided by the user.
- [ ] Verify that no previously completed functionality has regressed.
- [ ] State the source version in the release notes.

- **Ask before creating a zip** — do not package output files unless explicitly requested
- **Always bump the version number before packaging** — in `manifest.json` and anywhere else the version is tracked (e.g. `package.json` if present). Every zip that goes out the door corresponds to exactly one version number; two different zips must never share a version. Bump even for a documentation-only or comment-only change if it's being packaged — if it's not being packaged, it can wait and ride along with the next real bump instead of triggering its own.
- **Add a CHANGELOG.md entry for every version**, written before the zip is created, not after — the entry should let a reader understand *why* something changed, not just that it did (see B4).
- **Flag every behavior-visible change plainly, before applying it — not just in commit-message language.** If a change removes a capability, alters what the user will see, or has any effect beyond what was literally asked, state it as a one-line, capitalized flag: `WARNING FOR IMPLEMENTATION IMPACT: <what the user will actually see or lose, in plain terms>`. This is not optional flavor text — a technically-accurate description of the *mechanism* ("the input keeps its placeholder, a new element carries the text") is not sufficient on its own if the person approving it has no rendered mental picture of the *result* ("you'll end up with two separate bars stacked, not one"). When in doubt, flag it. Keep the rest of the response short — this rule is about surfacing impact clearly, not about writing more.

---

## Part A — HTML & CSS Structure

### A1. CSS — Divide into numbered sections

Rewrite every `<style>` block or external `.css` file as a sequence of clearly labelled sections, one per UI component. Each section must start with a banner comment:

```css
/* ══════════════════════════════════════════════════════════════
   3. SLIDER ZONE  —  level number, name, range slider, ticks
   ══════════════════════════════════════════════════════════════ */
```

Sections must be ordered top-to-bottom in the same order the components appear in the HTML.

Suggested section order (adapt to the actual file):
1. Reset & CSS variables
2. Body & base typography
3. Header
4. Navigation / toolbar (if present)
5. Main content sections (one sub-section per distinct UI block)
6. Forms, inputs, buttons
7. Tags, chips, badges
8. Modals, tooltips, overlays
9. Animations & keyframes

---

### A2. CSS variables — Centralise all design tokens

Identify every hardcoded colour, size, spacing value or font name that appears more than once. Move them into `:root` CSS variables at the top of section 1:

```css
:root {
  --bg:      #0F1117;
  --surface: #181C26;
  --border:  #252A38;
  --muted:   #C0C7D4;
  --bright:  #FFFFFF;
  --accent:  #4ADE80;
  --red:     #F87171;
  --yellow:  #FFD84D;
  --pad:     16px;
  --radius:  6px;
}
```

All rules must reference variables — no raw hex colours or repeated magic spacing values in the rules themselves.

---

### A3. Padding & alignment — Enforce a consistent left edge

Identify the standard horizontal padding used by the majority of sections (commonly `16px`). Store it as `--pad`. Flag any section where:
- Content does not align with `--pad`
- A child element is pushing text further right than `--pad`
- `padding-left` is set to `0` on the container and re-added inconsistently on children

---

### A4. HTML — Add section comments

Add a comment before every major HTML block matching the CSS sections:

```html
<!-- ── 3. SLIDER ZONE ── -->
<div class="slider-zone">
  ...
</div>
```

---

### A5. Orphaned CSS — Remove dead rules

Find and flag any CSS rule whose selector matches no element in the HTML. If a rule targets an element added dynamically by JS, flag it as "JS-injected, verify manually" rather than removing it automatically.

---

### A6. Inline styles — Audit and migrate

For every `style="..."` attribute:
- If static: move it into the CSS and replace with a class
- If set dynamically by JS: leave in place and add `<!-- set dynamically by JS -->`
- If a one-off nudge: add it to a `/* Overrides */` subsection at the bottom of the CSS with an explanatory comment

---

### A7. CSS consistency check

Verify:
- Every font family reference uses the same variable or identical string
- Every colour used in the file appears in `:root`
- Border, border-radius and transition values that repeat more than twice are extracted into variables
- No section contains rules that belong to a different section

---

### A8. Light/dark mode visibility — contrast is computed, not eyeballed

Every text colour, border, and de-emphasized/faded element (anything using `opacity`, `color-mix(..., transparent)`, or a partial-alpha token) must meet a minimum contrast ratio against its actual background **in both light and dark mode**, not just whichever mode the change happened to be reviewed in. A value tuned by eye in one mode routinely fails in the other — light and dark themes don't just invert, their actual token RGB values differ in ways that make "the same opacity percentage" produce very different real contrast in each direction (see the worked example below, where the same 25% mix failed *both* modes, not just one).

**Why "looks fine to me" isn't sufficient:** the person making the change is usually looking at whichever mode their own system is set to. A low-opacity value can look like reasonable, deliberate de-emphasis in that one mode while being genuinely unreadable in the other — and unless both are actually checked, that failure ships silently, because nothing about the code *looks* wrong in isolation.

**Required check:**
- [ ] For every new or changed `color-mix(...)`, `opacity`, or partial-alpha rule, identify the actual foreground token (e.g. `--ink-1`) and actual background token (e.g. `--surface-1`) it composites against, in **both** light and dark mode
- [ ] Compute the resulting effective RGB (alpha-composite foreground over background) and its WCAG relative-luminance contrast ratio against that background, for both modes — don't estimate, calculate. A short script or even a manual per-channel `alpha*fg + (1-alpha)*bg` calculation is enough; the point is a real number, not an impression
- [ ] Minimum bar: **3:1** for large text / UI components (e.g. placeholder glyphs, icon-only controls), **4.5:1** for normal body/data text. Below 3:1 in *either* mode is a fail, full stop — not "acceptable if the other mode is fine"
- [ ] When a value fails, don't invent a new bespoke opacity — check whether another element in the same file/component already uses a value proven to pass in both modes (see C21) and reuse that exact value instead of re-guessing
- [ ] State the computed ratios for both modes in the fix, not just "increased opacity" — the number is what makes the fix verifiable later, an adjustment description alone isn't

**Worked example:** a table's empty-cell placeholder ("—") used `color-mix(in srgb, currentColor 25%, transparent)`. Computed against the actual design tokens: 1.62:1 in light mode, 2.03:1 in dark mode — both well under the 3:1 minimum, meaning the element was never actually readable in *either* mode, just faint enough in both that it read as "intentionally subtle" rather than "broken" on casual inspection. A sibling element in the same file already used 55% opacity for de-emphasized (but still readable) text, computed at 3.80:1 (light) / 4.88:1 (dark) — both passing. Fix: reuse that exact already-proven 55%, not a freshly-guessed value.

---

### A9. Maintain and consult the light/dark style guides — don't re-derive from scratch each time

This extension maintains two standing reference documents, `STYLE_GUIDE_LIGHT.md` and `STYLE_GUIDE_DARK.md` — an aggregated, evidence-based catalog of every verified design token, proven-safe opacity value, and component color pattern actually in use, each with its computed contrast ratio (not a description of "how contrast should generally work," the actual numbers for the actual tokens this codebase has). A8 is the *check*; these two files are the *reference* that check should be run against and that its results get recorded into.

**Required practice:**
- [ ] Before introducing a new color, opacity, or `color-mix()` value anywhere, check the relevant style guide first — if an already-verified value for the same purpose (de-emphasized text, a status color, a border tier) already exists, reuse it rather than picking a fresh one (same reasoning as C21: a proven value beats a plausible-looking new one)
- [ ] If a genuinely new value is needed, compute its contrast in **both** modes per A8, then **add it to both style guide files** before considering the work finished — an unrecorded verified value is exactly as useless to the next person as an unverified one, since nothing signals it was ever checked
- [ ] When A8's audit finds an existing value fails contrast, the fix isn't just changing that one call site — update the style guide entry too, so the next lookup doesn't propose the same broken value again
- [ ] Treat a discovered gap in the style guide (a token in wide use that was never actually verified, e.g. a shared token only ever tuned for one mode) as worth flagging even if it isn't the specific bug being fixed right now — record it as a known gap in the guide rather than silently working around it once and leaving the underlying token unverified for the next person who reaches for it

**Why a written style guide, not just "the codebase is the reference":** grepping the codebase for "how has this been done before" finds *usage*, not *verification* — a value can appear in ten places and still have never been checked against real contrast math in either mode (this happened: `--color-green`/`--color-amber` were used as text color in several places before it was discovered they only ever passed WCAG for dark mode, never light). A style guide that states the computed ratio next to the value is what turns "this is what's used" into "this is what's confirmed safe" — and makes the difference between the two visible at a glance instead of requiring the calculation to be redone.

---

## Part B — Code Quality Audit

### B1. Centralise magic numbers

Find all hardcoded thresholds, scores, penalties, bonuses and timing values. Extract them into a single named config object (e.g. `SCORING_CONFIG`) at the top of the relevant file, with a comment explaining each value.

For DNR rule ID ranges, slot capacities, and storage caps specifically: extract into a dedicated `dnr-budgets.js` module (or equivalent) so that every consumer imports from one source of truth rather than declaring its own copy. See B3 for the pattern.

---

### B2. Eliminate duplicated constants

Find any array, set or list copy-pasted across multiple files. Extract it into a shared utility file and load it via `<script src>` (page contexts) or `importScripts()` (service worker). Add a comment in each consuming file pointing to the single source of truth.

---

### B2a. Constants hardcoded in HTML — drive from JS

A variant of B2 that is easy to miss: a constant defined in JavaScript (a list, a set of domain names, a set of thresholds) that is **also reproduced as static text or hardcoded HTML in a page** (`options.html`, `popup.html`, `warning.html`). The two copies immediately diverge whenever the JS constant is updated.

**Pattern to detect:**

Any HTML element whose visible text lists values that are also declared in a JS constant:

```html
<!-- Wrong — hardcoded copy of BUILTIN_DO_NOT_CHECK_DOMAINS -->
<p>Built-in defaults:
  <span class="mono">chromewebstore.google.com</span>,
  <span class="mono">drive.google.com</span>,
  <span class="mono">apps.microsoft.com</span>
</p>
```

**Required fix:**

1. Move the constant into a shared file loaded by both the service worker (`importScripts`) and the page (`<script src>`). A good home is `AppConstants.js` or equivalent — a file already loaded in both contexts.
2. Replace the hardcoded HTML with an empty placeholder element (`<span id="builtinDomains"></span>`).
3. In the page's JS initialisation function, populate the placeholder from the constant:

```javascript
function renderBuiltinDomains() {
    const span = document.getElementById("builtinDomains");
    if (!span) return;
    const domains = APP.BUILTIN_DO_NOT_CHECK_DOMAINS || [];
    span.innerHTML = domains.map((d, i) => {
        const comma = i < domains.length - 1 ? ", " : ".";
        return `<span class="mono">${d}</span>${comma}`;
    }).join("");
}
```

**Audit checklist:**

- [ ] Search every `.html` file for inline lists of domain names, error codes, file extensions, score thresholds, or any other value that also appears in a JS constant
- [ ] Search every `.html` file for text that duplicates content of any `const`, `Object.freeze`, or config object in any loaded `.js` file
- [ ] For every match: move the constant to a shared JS file, add a placeholder element to the HTML, and render it from JS on `DOMContentLoaded`
- [ ] After the fix, verify that changing the JS constant alone updates the rendered UI with no HTML edit required
- [ ] Add a comment next to the placeholder element pointing to the JS constant: `<!-- populated from APP.BUILTIN_DO_NOT_CHECK_DOMAINS by OptionsPage.js -->`

**Relationship to B2:** B2 covers JS-to-JS duplication. B2a covers JS-to-HTML duplication — the same root cause (a single source of truth not enforced) but harder to spot because the duplicate lives in markup rather than code.

---

### B3. Document state, guards and timers

Add a reference comment block at the top of the background service worker covering:
- **State overview**: in-memory vs persisted (`chrome.storage`), and what happens on SW restart
- **State guards**: flags/booleans that prevent stale state, with file and line number
- **Race condition guards**: mechanisms handling timing between async operations
- **Endless loop guards**: mechanisms preventing event-triggered loops
- **Timer reference**: all `setTimeout`/`setInterval`/`chrome.alarms` values, what they guard, and their variable names

#### B3a. State-vector / swimming-lane pattern (defensive programming)

Apply structured-programming state ownership to any module that has multi-phase lifecycle or can be in an inconsistent state mid-operation. The pattern:

**Each workflow owns one named state vector.** The vector carries the process identity, the current phase (an explicit enum), and all fields belonging to that phase — nothing is scattered across module-level variables that must be kept in sync manually.

```javascript
// ── STATE VECTOR ──────────────────────────────────────────────
// Valid phases for the monitor session swimming lane.
const SESSION_PHASES = Object.freeze({
    IDLE:     'idle',      // no session active
    STARTING: 'starting',  // reload fired; waiting for navigation commit
    RUNNING:  'running',   // listener active, capturing
    PAUSED:   'paused',    // listener removed; data preserved
});

let session = {
    owner:        'block-monitor',   // identifies who owns this vector
    phase:        SESSION_PHASES.IDLE,
    tabId:        null,
    host:         null,
    startTime:    null,
    timeElapsed:  0,
    timeoutHandle: null,             // belongs to the session, not the module
};

// Guard: log and abort if an unexpected transition is attempted.
function assertPhase(expected, caller) {
    const ok = Array.isArray(expected)
        ? expected.includes(session.phase)
        : session.phase === expected;
    if ( !ok ) {
        console.warn(`[${session.owner}] ${caller}: expected '${
            Array.isArray(expected) ? expected.join('|') : expected
        }', got '${session.phase}' — ignoring`);
    }
    return ok;
}
```

**Swimming-lane lifecycle diagram** — add to the module header:

```
[idle] ──startMonitor()──► [starting]
                               │
                       onCommitted fires
                               │
                               ▼
                          [running] ◄─────────────────────┐
                               │                          │
                    ┌──────────┼──────────┐               │
                    ▼          ▼          ▼               │
                timeout    stopMonitor  pauseMonitor      │
                    │          │             │            │
                    ▼          ▼             ▼            │
                  [idle]     [idle]       [paused]       │
                                              │           │
                                       resumeMonitor     │
                                              └──────────►┘
```

**Apply this pattern to:**
- Any module with a multi-step session (monitor, recorder, scanner)
- The SW startup lane — replace scattered boolean flags with a single `swLifecycle` vector:

```javascript
// js/data/config.js
export const SW_PHASES = Object.freeze({
    BOOTING:  'booting',   // loadRulesetConfig() in progress
    STARTING: 'starting',  // startSession() in progress
    READY:    'ready',     // fully initialised
    ERROR:    'error',     // start() threw
});

export const swLifecycle = {
    owner:       'background',
    phase:       SW_PHASES.BOOTING,
    startPhase:  null,       // sub-step within STARTING: 'migration'|'locale'|'permissions'|'scripts'|'finalising'
    errorReason: null,
    firstRun:    false,
    wakeupRun:   false,
    firstAlarm:  false,
};

// Backward-compat alias so existing process.firstRun call sites need no rename
export const process = swLifecycle;
```

Then in the SW:
```javascript
// background.js — set phases at each step so a hang is pinpointed in devtools
async function start() {
    swLifecycle.phase = SW_PHASES.BOOTING;
    await loadRulesetConfig();
    swLifecycle.phase = SW_PHASES.STARTING;
    await startSession();          // sets swLifecycle.startPhase at each sub-step
}

const isFullyInitialized = start().then(() => {
    swLifecycle.phase = SW_PHASES.READY;
    swLifecycle.startPhase = null;
}).catch(reason => {
    swLifecycle.phase = SW_PHASES.ERROR;
    swLifecycle.errorReason = String(reason);
    console.error(`[uBOL] startup failed at startPhase='${swLifecycle.startPhase}':`, reason);
    swLifecycle.startPhase = null;
});
```

**Mark remaining candidates** — where the pattern is not yet applied, add a `// ── FUTURE IMPROVEMENT` comment describing what the vector would look like:

```javascript
// ── FUTURE IMPROVEMENT — compile/register pipeline state ─────────────
// `pendingRegister` is a promise chain used as a serialisation lock but
// gives no visibility into whether it is idle, compiling, or stuck.
// A phase-tagged vector would make this inspectable:
//
//   const pipelineState = {
//       owner: 'compiled-filters',
//       phase: 'idle' | 'compiling' | 'registering' | 'error',
//       errorReason: null,
//   };
// ─────────────────────────────────────────────────────────────────────
let pendingRegister = Promise.resolve();
```

---

#### B3b. Self-closing timed-session panels — two independent paths, not one

Any feature with a timed session (a countdown, a 5-minute monitoring window, an inspection mode) that opens a dedicated UI panel (a tab, a popup window) which must reliably close itself when the session ends is a distinct reliability problem from the state-vector pattern in B3a, and deserves its own audit pass even when B3a is already applied correctly.

**The failure mode:** a single mechanism — typically the panel's own client-side clock hitting zero and calling `window.close()` directly — is made the *only* path to closing the tab. This looks correct, tests correctly during active development (the tab is focused, its timers run at full rate), and then fails specifically in the condition real users hit most: the tab left open and unfocused for the full session length, where the browser throttles `setInterval`/`setTimeout` in backgrounded tabs. The result is a stale, empty, unclosed panel — with no error anywhere, since nothing failed, a timer just didn't fire on schedule. This is exactly the kind of bug that gets "fixed" once (harden the client-side clock's own reliability) and then resurfaces later in a different guise, because the single-mechanism design itself was never addressed.

**Required pattern — two independent, redundant paths to the same outcome:**

1. **Background-authoritative expiry**, not panel-authoritative. The service worker — not the panel — owns the deadline: a primary `setTimeout` *and* a `setInterval` watchdog fallback (to survive a SW restart wiping the primary timeout — see B7), both converging on the same `endSession(reason)` function.
2. **An active broadcast on expiry** (`BroadcastChannel`, or equivalent), sent the moment the background decides the session is over — not something the panel has to poll for to find out.
3. **An independent poll**, in the panel itself, on a fixed interval (e.g. 15s), asking the background directly "is my session still valid?" — a second, unrelated path that doesn't depend on the broadcast being received, and doesn't depend on the panel's own countdown-display timer surviving throttling.
4. The panel's own visible countdown clock may still exist for display purposes, and may still trigger a close directly as a *third*, first-line-of-defense layer — but it must never be the *only* layer.

```javascript
// Background — service-worker side. The deadline lives here, not in the panel.
function scheduleTimeout(remainingMs) {
    clearTimeout(session.timeoutHandle);
    session.timeoutHandle = setTimeout(() => endSession('timeout'), remainingMs);
}

let watchdogHandle = null;
function startWatchdog() {
    if ( watchdogHandle !== null ) { return; }
    watchdogHandle = setInterval(() => {
        if ( session.phase !== 'running' || session.startTime === null ) { return; }
        const elapsed = session.timeElapsed + (Date.now() - session.startTime);
        if ( elapsed >= SESSION_TIMEOUT_MS ) { endSession('timeout'); }
    }, SESSION_TIMEOUT_MS);
}

async function endSession(reason) {
    if ( session.phase === 'idle' ) { return; }
    clearTimeout(session.timeoutHandle);
    session.phase = 'idle';
    broadcastMessage({ what: MSG_SESSION_CLOSED, reason });
}
```

```javascript
// Panel — independent poll, does not depend on the broadcast arriving
// or the visible clock's own interval surviving tab-throttling.
const POLL_MS = 15000;
let pollHandle = null;
function startPoll() {
    stopPoll();
    pollHandle = setInterval(async () => {
        const snap = await sendMessage({ what: MSG_GET_SESSION_DATA });
        if ( !snap ) { stopPoll(); window.close(); }
    }, POLL_MS);
}
function stopPoll() {
    if ( pollHandle !== null ) { clearInterval(pollHandle); pollHandle = null; }
}

broadcastChannel.onmessage = event => {
    if ( event.data?.what !== MSG_SESSION_CLOSED ) { return; }
    stopPoll();
    window.close();
};
```

**Audit checklist:**

- [ ] Identify every feature with a session timeout and a dedicated panel/tab that must close itself
- [ ] For each one, verify the deadline is tracked on the background/service-worker side, not only client-side in the panel
- [ ] Verify a watchdog `setInterval` exists as a fallback for the primary `setTimeout` being lost to a SW restart (see B7) — a `setTimeout` alone is not sufficient
- [ ] Verify the panel has an independent poll, not solely a broadcast listener — broadcast delivery and the panel's own timers are both things that can fail; neither should be a single point of failure
- [ ] If multiple features in the same codebase solve this same problem, verify they all use the *same* mechanism (see the new cross-feature-reuse principle below) — a codebase where one feature has this hardening and a sibling feature solving the identical problem does not is a strong signal the hardening was applied ad hoc rather than as a reusable pattern
- [ ] Verify every timer (`setTimeout`, `setInterval`, poll handle) involved is explicitly cleared on every close path — the countdown-clock interval, the poll interval, and the broadcast channel itself — not left to implicit tab-teardown cleanup (see B10)

---

### B4. CHANGELOG.md

Create a changelog in the root of the extension documenting each version: what changed, why it was structured that way, and any known limitations.

---

### B5. Architectural ceilings

Add comments where the current design has scaling limits (e.g. flat in-memory Maps that would need namespacing for multi-profile use, API rate limits, etc.). Document clearly so future developers know where the boundaries are — do not refactor them.

---

### B6. Unbounded collection check

Find every `Map`, `Set`, `Array` and `Object` that grows over time. For each one verify:
- Is there a maximum size cap or TTL-based expiry?
- Is there a cleanup path (event handler, periodic sweep, or SW restart)?
- If neither exists, flag it as a memory leak and propose a fix (FIFO eviction, TTL Map, or periodic sweep)
- Document the bound in a comment next to the declaration

---

### B7. SW restart survival

For every piece of in-memory state, verify:
- If lost on SW restart, is that intentional and safe?
- If it must survive restart, is it persisted to `chrome.storage.session` or `chrome.storage.local` and correctly restored on startup?
- Is there a race condition between the restore completing and event listeners firing?

---

### B8. Stateless vs stateful constants

For every `const`, `let` and `var` at module/global scope, classify as:
- **Truly stateless**: pure constant, never mutated, safe to declare inline
- **Stateful, initialised from storage**: must be loaded from `chrome.storage` before use; verify it has a correct default and is not used before the load completes
- **Stateful, in-memory only**: intentionally ephemeral; document why losing it on SW restart is safe

Flag any stateful value used before it is initialised.

---

### B9. Settings default coverage

For every key in the settings default object, verify:
- It has a sensible default that works correctly on first install
- New keys have backwards-compatible defaults for existing users
- The options page correctly loads, displays and saves every key — no orphaned settings and no UI fields that are never persisted

---

### B10. Resource release audit

For every resource acquired during the extension's operational lifecycle, verify it is explicitly released when that lifecycle ends. Check:

- **Event listeners**: listeners added dynamically — are they removed when no longer needed?
- **Timers**: is every `setTimeout`/`setInterval` ID stored and cleared when no longer needed? Prefer storing timer handles inside the owning state vector (see B3a) so cleanup is co-located with the state it belongs to.
- **chrome.storage writes**: when an entry is deleted from memory, is it also deleted from storage?
- **Window references**: are references to popup/panel windows nulled when the window closes?
- **AbortControllers**: is `abort()` called and the controller dereferenced when a fetch completes, times out or is cancelled?
- **Per-item lifecycle resources**: for each tracked item (downloads, requests, scans), are all resources released when that item's lifecycle ends?

---

### B11. Trust boundary audit — static allowlists and skip-lists

Any hardcoded list that causes the extension to **skip a safety check** must be audited for correctness. An overly broad allowlist is a security gap, not a convenience feature.

**Pattern to detect:**

A list of domains, hosts, or URLs that causes an early-return bypass of all protection logic:

```javascript
// Example — every entry here receives zero scrutiny
const BUILTIN_DO_NOT_CHECK_DOMAINS = [
    "chromewebstore.google.com",
    "drive.google.com",       // ← user-controlled content: should NOT be here
    "onedrive.live.com",      // ← user-controlled content: should NOT be here
    "icloud.com",             // ← user-controlled content: should NOT be here
    "sharepoint.com",         // ← org-controlled content: borderline
];
```

**Two categories that must be distinguished:**

| Category | Who controls the content? | Should bypass checks? |
|---|---|---|
| Platform-vetted stores | The platform (Google, Microsoft, Apple) vets every upload | Yes — safe to skip |
| User-controlled cloud storage | Any user can upload anything (Drive, OneDrive, iCloud, Dropbox) | **No — must be checked** |
| Org-controlled hosting | Any org member can upload (SharePoint, Confluence) | Offer as *user* whitelist default, not built-in |

**Required audit steps:**

- [ ] For every entry in the built-in allowlist, answer: *who controls the content at the URL being downloaded?*
- [ ] If the answer is "any arbitrary user", remove it from the built-in list
- [ ] Entries removed from the built-in list that have legitimate use cases should be offered as pre-populated defaults in the **user-configurable** whitelist, with a comment explaining why they were moved
- [ ] The built-in list must only contain entries where the platform itself is the content gatekeeper (e.g. signed app stores with mandatory review)
- [ ] Document each entry in the built-in list with a comment stating *why* it is trusted: `// Google-reviewed; every upload passes automated and manual vetting`

**Late-entry gap — filename unknown at download start:**

A related trust-boundary failure occurs when the file type is not known at download-creation time (e.g. Google Drive's redirect chain hides the filename until after the download completes). If the extension only gates on extension/MIME at `onCreated`, it silently passes all downloads from hosts where the filename is revealed late.

- [ ] Verify that the extension re-evaluates the file type at download-completion time (`onChanged`) using the confirmed filename from `chrome.downloads.search()`
- [ ] Verify that downloads whose extension was unknown at `onCreated` but revealed as dangerous at `onChanged` are flagged — not silently passed
- [ ] Add a scoring penalty for extension obfuscation: the server actively concealing what it serves is itself a signal (see scoring engine checks)

**Relationship to B2a:** B11 is about *which* entries belong in a trust list. B2a is about ensuring that list is defined in one place and rendered from that place everywhere. Both checks are necessary: B11 catches a wrong list, B2a catches a stale copy of an otherwise-correct list.

---

### B12. One-time-only UX flags — silent permanent feature suppression

A `chrome.storage`-backed "shown once, never again" flag is a common pattern for onboarding hints, intro popups, and first-run tips. It is also a common source of a specific, easy-to-miss bug class: the feature reads as broken forever, for exactly the users who most needed it, with no code error anywhere.

**Pattern to detect:**

```javascript
// A one-shot flag that permanently suppresses a UI element after the
// first time it's shown, with no way for the user to ever see it again.
async function maybeShowIntro() {
    const seen = await localRead('feature.introSeen');
    if ( seen ) { return; }
    showIntroUI();
    localWrite('feature.introSeen', true);
}
```

**Why this is a real bug, not just a design choice:** the flag is set the instant the popup is shown — not the instant the user actually reads and understands it. A popup with a short auto-dismiss window (a few seconds) is trivially easy to miss entirely (tab not focused, user looked away, dismissed by an accidental click) on the one occasion it's allowed to appear, and then it is gone for that browser profile permanently — not just for that session, not just until the next update, but forever, surviving every future extension version. From the user's perspective this presents identically to "the feature doesn't work," because there is no error, no console warning, and no way to distinguish "already seen and understood" from "missed it and confused ever since." A real instance of this class of bug: a picker tool's explanatory intro popup, gated behind exactly this pattern with only a 3-second auto-dismiss — a user who missed that one window had no path back to it on any future version, and reasonably concluded the feature had regressed.

**Audit checklist:**

- [ ] For every `localRead`/`localWrite` (or `chrome.storage`) pair implementing a "shown once" flag, ask: *is there any way for the user to intentionally see this again if they want to?*
- [ ] If the answer is no, and the content shown is genuinely load-bearing (explains how to use the feature correctly, not just decorative), prefer one of:
  - Show it every time the relevant UI opens (no flag at all) if it's brief and non-intrusive — the correct fix in most cases, since a returning user tends to ignore content they already recognize, while a confused user gets another chance
  - A persistent, always-visible "?" or "help" affordance that re-triggers the same content on demand, in addition to (or instead of) an auto-shown one-time version
  - A significantly longer or non-auto-dismissing display on the one showing, if "shown once" is kept
- [ ] Flag any auto-dismiss timer under ~5 seconds combined with a "shown once ever" storage flag as a strong candidate for this bug — the combination is the actual risk, not either alone
- [ ] Verify the flag's storage key is not also being read/written by unrelated code (a scoping check — a generically-named key like `introSeen` without a feature prefix can silently collide with a different feature's own onboarding state)

---

## Part C — Modular Architecture

### C1. One module, one responsibility

Every module file must have a single, clearly stated purpose. If a file currently handles more than one concern (e.g. both UI rendering and storage writes), split it.

---

### C2. No direct cross-module state access

Module A must not read or write a variable owned by module B directly. Enforce via getters/setters:

```javascript
// Wrong
import { sessionState } from './monitor.js';
if (sessionState.active) { ... }

// Correct
import { isMonitorActive } from './monitor.js';
if (isMonitorActive()) { ... }
```

---

### C3. Message routing — single switch

All `chrome.runtime.onMessage` handling must go through a single router function with one `switch` statement. No scattered `addListener` calls that each handle a subset of messages.

---

### C4. Storage access — single layer

All `chrome.storage` reads and writes must go through a single storage utility module. No direct `chrome.storage.local.get/set` calls in feature modules.

---

### C5. Error propagation

Every `async` function must either:
- Return a rejected Promise (let the caller handle it), or
- Catch the error, log it with context, and return a safe default

Never swallow errors silently with an empty `catch {}`.

---

### C6. Input validation at module boundaries

Every exported function that accepts external input (from messages, storage, or DOM events) must validate its inputs before acting on them. Invalid input must be rejected with a logged warning, not silently ignored.

---

### C7. Feature isolation

Any feature not always active must be:
- Isolated in its own module file
- Loaded conditionally via a `try { importScripts('feature.js') } catch(e) {}` guard or runtime check
- Completely removable by excluding its file — no stubs or dead code paths left behind

---

### C8. Lifecycle management

Every module that acquires resources must own the cleanup of those resources:
- Each module must have an explicit `stop()` or `cleanup()` function
- The main background script calls `stop()` on each module when appropriate
- Timers must be stored in a named variable within the owning module (preferably inside the module's state vector — see B3a) and cleared in that module's `stop()`

---

### C9. Naming conventions

- **Files**: `kebab-case.js`
- **Module-level constants**: `UPPER_SNAKE_CASE`
- **Functions**: `camelCase`, verb-first (e.g. `startLogging()`, `buildRules()`)
- **State variables**: `camelCase`, noun (e.g. `loggerWindowId`, `activeTabId`)
- **State vectors**: `camelCase`, noun + `State` or descriptive (e.g. `session`, `swLifecycle`, `pipelineState`)
- **Phase enums**: `UPPER_SNAKE_CASE` frozen object (e.g. `SESSION_PHASES`, `SW_PHASES`)
- **Message types**: `UPPER_SNAKE_CASE` string constants (e.g. `const MSG_START_LOGGING = 'START_LOGGING'`)
- **CSS classes**: `kebab-case`

Flag every deviation and propose the corrected name.

---

### C10. Module documentation

Every module file must start with a header comment block:

```javascript
// ── module-name.js ────────────────────────────────────────────────────────────
// One-sentence description of what this module does.
//
// Public interface:
//   startXxx(args)   — description
//   stopXxx()        — description
//   handleXxxMessage(msg, sendResponse) — routes MSG_XXX messages
//
// Dependencies (must be loaded before this file):
//   background.js: RULE_ACTIONS, applyState
//
// State owned by this module:
//   session  {Object}  — state vector; see SESSION_PHASES; lost on SW restart (safe: UI re-requests)
//   entries  {Array}   — FIFO capture buffer, max FIFO_MAX entries; lost on SW restart (safe)
//
// NOTE: any important constraint, e.g. "only works in unpacked extensions"
```

---

### C11. Single-purpose functions

Every function must do exactly one thing, named precisely after what that one thing is:
- Name must be a verb phrase that fully describes its effect
- If a function needs more than one paragraph to describe what it does, split it
- A function must not both compute a value and apply a side effect — split into pure computation and separate application function
- Flag any function longer than ~30 lines as a candidate for splitting

---

### C12. Clean API boundaries between modules

Every call from one module to another must go through an explicit, named function:
- Module B exposes a getter if module A needs module B's data: `getLoggerWindowId()`
- Module B exposes a command function if module A needs to trigger behaviour: `startLogger(config)`
- Arguments passed between modules must be plain data — never live references to internal state objects
- Every public function must validate its inputs

---

### C13. Consistent system state — every code path

The system must be in a valid, consistent state after every operation — not just the happy path. Audit:
- **Success path**: does the operation complete cleanly?
- **Failure path**: if an async call fails, is state rolled back or flagged correctly?
- **Timeout path**: if a timer fires, does it leave state consistent?
- **Cancellation path**: if the user cancels, are all in-progress resources released?
- **SW restart path**: after the SW restarts, is state read from storage self-consistent?

Apply the state-vector pattern (B3a) so that the `phase` field is always authoritative. An `assertPhase()` guard at the start of each transition makes the inconsistent-state case explicit and logged rather than silent.

Flag any flag set to `true` at the start of an operation that may never be reset to `false` on failure.

---

### C13a. Data-set and build-time consistency guards

Every external dependency, generated file, and configuration data set must be explicitly validated before it is consumed. Silent failures — where bad data is accepted, compiled into the build, or shipped to users without any warning — are never acceptable.

**Build-time guards (tools, scripts, generators):**
- Every network fetch must verify the response is non-empty, has a plausible minimum size, and structurally resembles the expected format before writing output. A GitHub 404 HTML page or a CDN error must never silently compile into a filter list.
- After fetching, verify content contains expected markers (e.g. filter list syntax tokens `!`, `||`, `##`) before processing.
- Any generated output file (JSON, JS, manifest entry) must be cross-checked against its source definition. If a ruleset ID is declared in a source definition file, it must appear in the manifest and vice versa — no orphaned entries, no missing entries.
- The build script must exit with a non-zero code and a clear error message on any validation failure. It must never silently fall back to stale or partial output.

**Runtime guards (extension code):**
- When reading any JSON file at runtime (filter files, config files, registration manifests), validate that the parsed structure is non-null, has the expected shape, and contains at least the minimum expected entries before registering or applying it.
- Any registration call (`registerContentScripts`, `updateEnabledRulesets`, etc.) that references a file path must verify the file exists and is reachable before registering — not just that the ID is present in a list.

**Cross-file consistency checks:**
- Ruleset IDs must be consistent across: build script source definitions → manifest `rule_resources` → `static-rulesets.js` ID list → `scriptlet-files.json` → `cosmetic-files.json` → any UI display-name maps. Flag any ID that appears in one location but not another.
- If a feature is removed, audit all five locations above — an ID left in any one of them after removal is a latent bug (as demonstrated by `ruleset_adguard_cookies` remaining in manifest after removal from the build script).

Flag any fetch, file read, or registration call that lacks explicit validation of the received data.

---

### C13b. Dead guards and stale state audit

Guards, conditional branches, state flags, migration routines, and feature sets must be periodically audited to verify they still protect something real. A guard that always evaluates to the same value, a migration that can never trigger, or a feature set that is never populated are not harmless — they impose maintenance cost, mislead future readers, and can mask genuine bugs.

**Guard liveness:**
- Every conditional of the form `if (FEATURE_ENABLED)`, `if (mode === X)`, or `if (ALWAYS_ON_RULESETS.has(id))` must be verified to have at least one live code path where the condition is false. If the condition is structurally always true or always false given the current set of features, the guard is dead and should be removed along with the unreachable branch.
- Constant sets used as guards (e.g. `ALWAYS_ON_RULESETS`, `ANNOYANCE_RULESETS`) must be cross-referenced against the active feature set. If every member of the set has been removed, the set and all code that references it must be removed.

**Mode and phase completeness:**
- Every mode enum value, phase constant, and feature flag must have at least one reachable code path that assigns it, and at least one that reads it. An enum value that is written but never read, or read but never written, is dead.
- Migration routines that fold legacy values into current ones (e.g. collapsing old mode names into a canonical value) must be verified against the actual version history of the extension. If the extension never shipped the legacy value being migrated from, the migration is dead from day one and must be removed.

**Feature-set cross-referencing:**
- When a feature or ruleset is removed, audit every location where its identifier, mode value, or associated data structure appears: source definitions, manifests, registration files, UI display maps, message handlers, background initialisation, and migration code. Removal is only complete when all five are clean.
- Any identifier that appears in a registration file, UI map, or handler but has no corresponding active definition is a latent inconsistency. Flag it for removal.

**Rule of thumb:** if a guard, branch, or data structure cannot be demonstrated to affect observable behaviour in the current codebase, it is dead code and must be removed. Do not preserve it "for future use" — future requirements will be addressed when they arise, with full context.

---

### C13c. Overly-broad DNR rule guard

Every build script that compiles third-party filter lists into Chrome DNR rules must drop rules that block all resources of a given type on a named site without meaningful URL specificity. Such rules are indistinguishable from "disable this website" and break legitimate CDN-hosted scripts, player SDKs, and API calls even when the original filter author only intended to block ad networks.

**Pattern to detect and drop at build time:**

A rule is overly broad if it meets **all three** of the following conditions simultaneously:

1. `action.type === "block"`
2. `condition.urlFilter` is a trivially wide anchor — i.e. it matches virtually every URL:
   - `|https://`
   - `|http://`
   - `|https:`
   - `|http:`
   - `|http*://*?`  *(also catches any-URL-with-query-string)*
3. `condition.domainType === "thirdParty"` **and** `condition.initiatorDomains` is a non-empty array

When all three are true the rule blocks **every** third-party request of the specified resource types from the named site — regardless of which host the request goes to. This breaks any site that loads critical resources (video players, fonts, analytics, comment systems) from third-party origins.

**Required guard in the build script:**

```javascript
// ── OVERLY_BROAD_URL_FILTERS ──────────────────────────────────
// urlFilter values that by themselves match virtually every URL.
// A block rule combining one of these with domainType=thirdParty
// and a non-empty initiatorDomains list effectively disables all
// third-party requests on the listed sites — far too aggressive.
// Known to break: xvideos.com, mediafire.com, jkanime.net, and
// any site that loads CDN-hosted player SDKs or API endpoints
// from third-party origins.
const OVERLY_BROAD_URL_FILTERS = new Set([
    '|http*://*?',  // any URL with a query string
    '|https://',    // every HTTPS URL
    '|http://',     // every HTTP URL
    '|https:',      // same without trailing slash
    '|http:',       // same without trailing slash
]);

const beforeBroad = validated.length;
validated = validated.filter(r => {
    if ( r.action?.type !== 'block' ) { return true; }
    const cond = r.condition ?? {};
    const uf   = cond.urlFilter ?? '';

    // Drop bare catch-all query-string blocker (no other conditions needed).
    if ( uf === '|http*://*?' ) { return false; }

    // Drop trivially-wide URL + thirdParty + initiatorDomains combination.
    if (
        OVERLY_BROAD_URL_FILTERS.has(uf)          &&
        cond.domainType === 'thirdParty'            &&
        Array.isArray(cond.initiatorDomains)        &&
        cond.initiatorDomains.length > 0
    ) { return false; }

    return true;
});
if ( validated.length !== beforeBroad ) {
    process.stdout.write(
        `(dropped ${beforeBroad - validated.length} overly-broad) `
    );
}
```

**Audit checklist:**

- [ ] The build script declares `OVERLY_BROAD_URL_FILTERS` as a named constant with a comment — not an inline literal string comparison
- [ ] The guard runs **after** the ASCII/space validity filter and **before** hostname coalescing, so rule IDs are still contiguous after the drop
- [ ] The number of dropped rules is reported to stdout at build time so unexpected changes are immediately visible
- [ ] The guard covers both the `|http*://*?` pattern (query-string catch-all) **and** the `|https://`/`|http://` + `thirdParty` + `initiatorDomains` combination — a guard covering only one of the two is incomplete
- [ ] There are no hardcoded site exceptions inside the guard — if a site legitimately needs all third-party requests blocked, that belongs in a separate, explicitly named per-site override, not as a bypass of a safety guard
- [ ] The set of filtered patterns is reviewed whenever the upstream filter list adds new rules of this class

**Relationship to C13a:** C13a guards against bad *input* entering the build pipeline (empty responses, non-filter-list content). C13c guards against valid filter-list syntax that is nevertheless too destructive to ship as a static DNR ruleset. Both guards are necessary and neither subsumes the other.

---

### C14. Atomic state transitions

Related pieces of state that must always change together must be written atomically:

```javascript
// Wrong — SW restart between these two calls leaves inconsistent state
chrome.storage.local.set({ logging: false });
setTimeout(() => chrome.storage.local.set({ logs: [] }), 3000);

// Correct — single atomic write
chrome.storage.local.set({ logging: false, logs: [], logStartTime: null });
```

When using a state vector (B3a), all fields on the vector change together in a single synchronous assignment block before any async call — this is the in-memory equivalent of an atomic write:

```javascript
// Correct — all session fields reset atomically before the async broadcast
session.phase       = SESSION_PHASES.IDLE;
session.tabId       = null;
session.host        = null;
session.startTime   = null;
session.timeElapsed = 0;
broadcastMessage({ what: MSG_MONITOR_CLOSED, reason });
```

Flag any place where two or more related storage keys are written in separate `set()` calls.

---

### C15. Five-tier folder structure

Organise all files into exactly five tiers. Dependencies may only flow **downward**:

```
extension-root/
├── ui/           Tier 1 — User Interface
├── workflow/     Tier 2 — Workflow & Orchestration
├── core/         Tier 3 — Core Feature Logic
├── util/         Tier 4 — Shared Utilities & Helpers
└── data/         Tier 5 — Data, Storage & External Services
```

Dependency flow:
```
ui/       →  workflow/, core/, util/, data/
workflow/ →  core/, data/, util/
core/     →  util/, data/
util/     →  (nothing — self-contained)
data/     →  (nothing inside the extension — only external APIs)
```

Any dependency that flows upward or sideways is a violation. Flag it and propose the move.

---

### C16. Service worker load order (`importScripts`)

Files must be loaded in strict bottom-up tier order:

```javascript
importScripts(
  // Tier 5 — data layer
  'data/message-types.js',
  'data/constants.js',
  'data/state-storage.js',

  // Tier 4 — utils
  'util/tld-utils.js',

  // Tier 3 — core logic
  'core/tld-engine.js',
  'core/rule-builder.js',
  'core/site-rules.js',
);
```

Use one `importScripts()` call with all files listed in order. Comment each import with its tier and exported symbols. Wrap optional modules: `try { importScripts('core/dev-tools.js'); } catch(e) {}`.

---

### C17. Page context load order (`<script src>`)

`<script src>` tags must appear at the **bottom of `<body>`** in strict bottom-up tier order, mirroring the SW load order for any shared files. Never duplicate a constant or function in `popup.js` that already exists in a shared file.

---

### C18. Module removal checklist

When removing a module, audit all of the following:

**Service worker (`background.js`):**
- [ ] Remove the `importScripts()` line for the deleted file
- [ ] Remove all calls to the deleted module's public functions
- [ ] Remove all references to the deleted module's exported variables
- [ ] Remove the deleted module's message type constants from `data/message-types.js`
- [ ] Remove the deleted module's handler branch from the `onMessage` router

**Page contexts:**
- [ ] Remove any `<script src>` tags for the deleted file
- [ ] Remove any calls to the deleted module's functions from popup/options JS
- [ ] Remove any UI elements that existed solely to control the deleted feature

**Other modules:**
- [ ] Search every remaining file for references to the deleted module's exported names
- [ ] Check whether any remaining module was only useful because it served the deleted feature

**Manifest:**
- [ ] Remove permissions only required by the deleted feature
- [ ] Remove `web_accessible_resources` entries for the deleted module's files

---

### C20. Cross-reference audit — registered files vs files on disk

Every file path declared anywhere in the extension must exist on disk, and every file on disk that is intended to be loaded must be declared somewhere. Dangling references and orphaned files both indicate incomplete removal or a failed rename.

**Where file paths are declared (audit all of these):**

- `manifest.json` — `declarative_net_request.rule_resources[].path`, `background.service_worker`, `content_scripts[].js`, `web_accessible_resources[].resources`, `icons`
- `importScripts(...)` calls in the service worker
- `<script src="...">` and `<link href="...">` tags in every HTML file
- `rulesets/scriptlet-files.json` — `file` field of every entry
- `rulesets/cosmetic-files.json` — `file` field of every entry
- Any runtime `chrome.scripting.registerContentScripts()` or `chrome.scripting.executeScript()` call that references a file path

**Checks to perform:**

- [ ] Every path declared in any of the above locations resolves to a real file on disk
- [ ] Every `.js`, `.json`, `.css`, `.html` file in the extension directories is referenced by at least one of the above locations — or is explicitly documented as a build-time-only tool (e.g. `tools/build-rulesets.mjs`) in a comment
- [ ] No directory entries with malformed names exist in the zip (e.g. brace-expansion artifacts like `{js/`, `{js/{data,core,ui,workflow},css,ui}/`) — these arise when a build script uses shell brace expansion to create directories and the expansion is captured literally in the archive
- [ ] After any file rename or deletion, re-run this cross-reference check across all six declaration locations above before packaging

**How to run this check:**

```bash
# 1. Extract all declared paths from the manifest
python3 -c "
import json, pathlib
m = json.load(open('manifest.json'))
paths = []
for r in m.get('declarative_net_request', {}).get('rule_resources', []):
    paths.append(r['path'])
for r in m.get('web_accessible_resources', []):
    paths.extend(r.get('resources', []))
paths.append(m.get('background', {}).get('service_worker', ''))
for p in paths:
    p = p.lstrip('/')
    exists = pathlib.Path(p).exists()
    if not exists:
        print(f'MISSING: {p}')
"

# 2. Extract all paths from scriptlet-files.json and cosmetic-files.json
python3 -c "
import json, pathlib
for fname in ['rulesets/scriptlet-files.json', 'rulesets/cosmetic-files.json']:
    try:
        for entry in json.load(open(fname)):
            p = entry['file'].lstrip('/')
            if not pathlib.Path(p).exists():
                print(f'MISSING ({fname}): {p}')
    except FileNotFoundError:
        print(f'NOT FOUND: {fname}')
"

# 3. List all JS/JSON/CSS/HTML files not referenced anywhere
# (manual step — grep each filename across all declaration locations)
```

**Relationship to C18 and C19:** C18 is the *process* checklist to follow when deliberately removing a module. C20 is the *verification* check to run after any change — it catches cases where C18 was skipped or incomplete. C19's store-readiness bullets for `web_accessible_resources` are a subset of C20; C20 extends the same principle to all six declaration locations.

---

#### C20a. Message-type routing — a parallel cross-reference, not a variant of the file-path check

The same "declared somewhere, wired everywhere" failure mode applies to `MSG_*` message-type constants, independently of file paths — and is easy to overlook because it produces no missing-file error and no failed `import`. A message-type constant can be declared, exported, given a plausible name and comment, and never actually be imported by anything else at all. It is dead on arrival: not a bug that fires, just a name that looks load-bearing and isn't. This was caught in practice by exactly the automated check below, which found one such orphaned constant — declared, commented to look like a real (and, confusingly, *almost*-duplicate-named) feature, wired to nothing.

Every exported `MSG_*` constant is one of exactly two kinds, and both must be verified:

- **Request/response**, sent via `runtime.sendMessage` — must have both a route entry (whatever gate/permission table the project uses for inbound messages) *and* a dispatch-table handler.
- **Broadcast-only**, sent via `BroadcastChannel` (or equivalent) — correctly bypasses the request/response route table by design; verify it has at least one `.onmessage` listener actually checking for it somewhere, not just a `broadcastMessage()` call site with no corresponding reader.

```python
import re

msg_consts = dict(re.findall(
    r"export const (MSG_\w+)\s*=\s*'([^']+)'",
    open('js/data/message-types.js', encoding='utf-8').read()
))
routes_content = open('js/workflow/message-routes.js', encoding='utf-8').read()
bg_content     = open('js/workflow/background.js', encoding='utf-8').read()

# Maintain this set explicitly — broadcast-only types are expected to be
# absent from the route/dispatch tables; everything else is not.
broadcast_only = {'MSG_MONITOR_ENTRY', 'MSG_MONITOR_CLOSED', 'MSG_PRIVACY_INSPECTOR_ENTRY', 'MSG_PRIVACY_INSPECTOR_CLOSED'}

missing_route   = [n for n in msg_consts if n not in broadcast_only and f'[{n}]' not in routes_content]
missing_handler = [n for n in msg_consts if n not in broadcast_only and f'[{n}]' not in bg_content]

print('Missing route:', missing_route or 'none')
print('Missing handler:', missing_handler or 'none')
```

**Audit checklist:**

- [ ] Every declared `MSG_*` constant not in the maintained broadcast-only set has both a route entry and a dispatch handler
- [ ] Every broadcast-only `MSG_*` constant has at least one `.onmessage` reader somewhere in the codebase
- [ ] Run this check after adding *or* renaming any message type — a rename that updates the declaration and the sender but misses the handler produces exactly this failure mode, silently
- [ ] Treat two very-similarly-named constants (e.g. differing only by `SET_` vs `UPDATE_` as a verb) as a specific red flag worth a manual look — this shape is exactly how the real orphaned-constant instance above arose, and grep-based searches can miss that one is dead precisely because the other, live one keeps matching nearby searches

---

### C19. Store-readiness and permission audit

Before submitting to the Chrome Web Store:

| Permission | Works in store build? | Notes |
|---|---|---|
| `declarativeNetRequest` | Yes | Shows permission warning at install |
| `declarativeNetRequestWithHostAccess` | Yes | Suppresses install warning; requires `host_permissions` |
| `declarativeNetRequestFeedback` | **No** | Only works in unpacked extensions; silently disabled in store builds; declaring it triggers rejection |

**The permission is not the only thing to check — the specific APIs it gates must be named explicitly, because more than one exists and it's easy to assume only the most well-known one is restricted.** Verified directly against Chrome's own extension API documentation, not assumed: all three of the following are unpacked-extension-only, and none of them work for a real Chrome Web Store (packed) install, however the code frames the intent:

- `chrome.declarativeNetRequest.onRuleMatchedDebug`
- `chrome.declarativeNetRequest.testMatchOutcome`
- `chrome.declarativeNetRequest.getMatchedRules`

Do not assume `testMatchOutcome` or `getMatchedRules` are safe alternatives to `onRuleMatchedDebug` because they sound like a different, more modern API — they carry the identical restriction. A codebase that already has a working, packed-extension-safe replacement for one of these (e.g. a manual reimplementation of "does this DNR rule match this request" using data already bundled with the extension) must reuse that replacement rather than reaching for a different one of the three banned APIs as a shortcut back to "the real signal" — see the cross-feature-reuse principle (C21) for why building a second, different answer to an already-solved problem is itself a bug risk, independent of whether the specific API chosen happens to work in this instance.

Store build checklist:
- [ ] No `declarativeNetRequestFeedback` in the store manifest
- [ ] No call to `onRuleMatchedDebug`, `testMatchOutcome`, or `getMatchedRules` anywhere in the codebase — grep for all three by name, not just the permission string, since code can reference the API without the permission being declared (it simply silently fails at runtime instead of failing a manifest check)
- [ ] No `webRequest` or `webRequestBlocking` unless justified
- [ ] No unused `host_permissions` entries
- [ ] No `web_accessible_resources` entries for files that no longer exist
- [ ] Extension version is incremented from the last published version
- [ ] All files listed in `web_accessible_resources` exist on disk
- [ ] No `content_security_policy` overrides permitting `unsafe-eval` or `unsafe-inline`

---

### C21. Reuse a proven solution across sibling features — do not reinvent

When two features in the same codebase solve the same underlying problem, and one of them has a solution that is actually **proven** — meaning it has a documented history of being tested against a real failure mode (a bug report, a CHANGELOG entry describing what broke and why the fix works), not merely "written once and assumed correct on first read" — the other feature must adopt that same solution. Building a second, different-looking answer to a problem the codebase has already solved once is not an improvement. It is a new, unproven implementation of something that didn't need re-deciding, and a second place for the same class of bug to hide — often for years, since nothing about a plausible-looking second implementation signals that a better, tested one already exists two files away.

**How to audit for this:**

- [ ] For any hardening, fix, or defensive pattern being added to one feature, search the codebase for other features solving the same category of problem (session timeouts, resource cleanup, cross-module messaging, error surfacing to a UI) and check whether they already have — or are missing — the same treatment
- [ ] When a feature is found to be missing hardening that a sibling feature already has, do not design a new mechanism for it — port the existing one, adjusted only for genuinely different constraints (e.g. a different underlying storage mechanism), keeping the same constants, same shape, and comments explaining why, not just what
- [ ] When writing the fix, add a comment at the point of reuse naming the sibling feature and file the pattern was ported from — this is what makes the *next* audit able to find both instances together instead of rediscovering the connection from scratch
- [ ] Treat "this bug class has a CHANGELOG entry showing it was already fixed once, elsewhere in this codebase" as a strong signal to go find that fix and port it, rather than writing new code from first principles

**Worked example — the empty-tab bug, two sibling session-monitor panels:**

Two independent panels in the same extension both open a timed-session, self-closing tab (see B3b for the general pattern). One of them had two separate documented rounds of "stale empty tab" bugs before converging on a hardened, single-mechanism fix (a hard `Promise.race` timeout around its own stop message, plus making sure resuming a paused session correctly restarts its countdown). The other was built independently and ended up architecturally stronger for the exact same problem: background-authoritative expiry (primary timeout *and* a watchdog interval fallback), an active broadcast the instant expiry fires, and an independent poll as an unrelated second safety net — two redundant paths instead of one.

When this asymmetry was noticed, the fix was not to invent a third design for the weaker panel — it was to port the stronger panel's exact mechanism over: same constant shapes (`sessionTimeoutHandle`/`watchdogHandle` pair), same broadcast-on-close pattern, same independent-poll interval, adjusted only for the weaker panel's own state-persistence strategy. The original panel's existing fix stayed in place too, as an additional layer, not replaced — this is additive hardening using a proven pattern, not a rewrite from a blank page.

**The rule going forward:** before building a new mechanism for a problem, check whether a sibling feature already has one — especially one with history showing it was actually hardened against a real failure, not merely written once and assumed correct. If it does, port it. Don't design a fresh alternative next to it.

---

### C22. Cross-variant consistency — a standing rule, not a one-off patch

When the user says "make X the same across levels/variants/tiers" (protection levels, plan tiers, user roles, platform targets — any axis the codebase branches on), that instruction is a **standing constraint for the rest of the engagement**, not a one-time fix applied to the single spot the user happened to point at. The natural failure mode is treating each report as an isolated bug: fix the flagged instance, leave every other occurrence of the same branch pattern untouched, and let the user discover each remaining one individually — which reads to them as the same mistake happening repeatedly, because it is the same mistake, just at different coordinates.

**Why this is easy to get wrong:** a codebase legitimately branches on the same variable (`level`, `tier`, `role`) in many unrelated places, for many different reasons — some intentional and correct (a feature genuinely doesn't apply to Level 1 because Level 1 has no rules at all), some accidental (a column was hidden at Level 1 purely because an early version of the UI happened to look cluttered there, with no continuing functional reason). Fixing only the reported instance treats the report as being about that one line of code, when it's actually about a *policy* the user wants enforced everywhere the same axis appears.

**Required pattern once "make X consistent across variants" is raised:**

1. Grep the codebase for every conditional branching on the same axis (`if (level`, `level ===`, `level !==`, `tier ===`, etc.) — not just the file the user's report pointed at.
2. For each hit, classify it explicitly: **intentional-and-still-correct** (state why in a comment at that site) vs. **accidental/no-longer-justified** (fix it now, in the same pass, without waiting to be asked about that specific spot).
3. Report both lists to the user before or alongside the fix — the ones changed, and the ones left alone with the reason — so a left-alone case doesn't read as another missed instance.
4. Treat any single flagged inconsistency as evidence the whole file (or module) needs this sweep, not evidence about only the one line reported.

**Audit checklist:**

- [ ] Search the full codebase for every branch on the axis the user flagged, not just the reported file
- [ ] Classify each hit as intentional (document why) or accidental (fix it)
- [ ] Apply "same layout/same mechanism" fixes using the shared component *already in use elsewhere* for that layout (see C21) rather than inventing a new one-off element for the newly-consistent case
- [ ] State explicitly, in the same response, which instances were changed and which were deliberately left different and why
- [ ] Re-run this sweep for the remainder of the session whenever the same consistency instruction resurfaces — don't reset back to fixing single spots after the first sweep

**Worked example:** a UI panel rendered a different column/row set depending on the current level, plus a level-dependent info line implemented as two different underlying mechanisms (one level's text lived in an input's `placeholder`, another level's lived in a separate visible element) — visually and structurally inconsistent between levels despite serving the same purpose. Each inconsistency was reported and fixed separately, three times, because each fix addressed only the specific element the user had pointed at rather than treating "make the levels consistent" as a rule to re-check the whole file against. The correct single pass would have been: on the first report, grep the file for every level-gated branch, fix all of them to use one shared mechanism, and state which (if any) remained intentionally different — rather than waiting for the user to find and report each remaining instance individually.

**Relationship to C21:** C21 is about reusing a *proven, hardened* solution across features solving the same underlying reliability problem (timeouts, cleanup). C22 is about a narrower but higher-frequency case: **the same UI/behavior, gated by the same variable, must render identically wherever that variable's value doesn't actually change the underlying logic.** C21 asks "has this class of problem been solved better elsewhere?" — C22 asks "does every occurrence of this axis get the same treatment, or did one get missed?"

---

### C23. Check version history before implementing — don't regress a documented fix

Before writing or modifying code in an area that touches a *mechanism* (not just a feature) — session timeouts, tab/window lifecycle, resource cleanup, retry logic, anything with a name like "hardened," "guard," or "fix" attached to it in past commits — search CHANGELOG.md (or equivalent version history) for that mechanism's name or symptom first. If it has prior entries, read them before writing new code, not after something breaks again.

**Why this matters more than it sounds like it should:** a mechanism that already has one or more CHANGELOG entries describing a bug and its fix is a mechanism that has already demonstrated it's easy to get subtly wrong — that's *why* it needed a documented fix in the first place. Touching that same code area again without first reading what went wrong last time means the next change is starting from the same naive assumptions the original bug came from, with no memory that they were already tried and failed. The fix doesn't have to be literally reverted for this to happen — a narrower symptom of the same underlying gap (see the worked example) is enough to demonstrate the check wasn't done.

**Required check, before implementing:**
- [ ] Identify the mechanism being touched in plain terms (e.g. "the popup window this button opens," "the session countdown," "the retry wrapper around this message")
- [ ] Grep CHANGELOG.md for that mechanism's name, its symptom words ("stale," "hang," "leak," "duplicate," "reappear," "empty"), and any function/file names involved
- [ ] If prior entries exist, read them fully before writing new code — note what the *root cause* was (not just what the fix changed), since a new bug in the same area is often a different narrow case of the same root cause, not a wholly new problem
- [ ] If the new work is adjacent to, but not identical to, a documented fix (e.g. same window-creation code, different failure timing), treat the existing fix as a floor to build on, not a box that doesn't apply — extend the same guard rather than assuming the prior fix's scope was already complete
- [ ] State in the response which prior CHANGELOG entries were checked and what they showed, even briefly — this is what lets the *next* person (or the next audit) trust the check actually happened, not just that it's claimed to have happened

**Worked example:** a "ghost tab" comment already existed in the code for a popup-window-opening function, documenting a prior fix for Chrome sometimes adding an extra blank tab alongside the real one. A new report of "an empty tab reappearing" on a *different* timed panel using the same window-opening function turned out to be a narrower case the existing fix didn't cover: the original fix only checked for extra tabs at the exact moment the window-creation promise resolved, not for one Chrome might add slightly *after* that moment. The existing comment and fix were the right starting point — reading it first made clear the gap was a *timing* gap in an already-correct approach, not a reason to write a new detection mechanism from scratch. The fix was a second, delayed check layered on top of the first, not a replacement for it (same additive-hardening shape as C21's worked example).

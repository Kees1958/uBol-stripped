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

### C19. Store-readiness and permission audit

Before submitting to the Chrome Web Store:

| Permission | Works in store build? | Notes |
|---|---|---|
| `declarativeNetRequest` | Yes | Shows permission warning at install |
| `declarativeNetRequestWithHostAccess` | Yes | Suppresses install warning; requires `host_permissions` |
| `declarativeNetRequestFeedback` | **No** | Only works in unpacked extensions; silently disabled in store builds; declaring it triggers rejection |

Store build checklist:
- [ ] No `declarativeNetRequestFeedback` in the store manifest
- [ ] No `webRequest` or `webRequestBlocking` unless justified
- [ ] No unused `host_permissions` entries
- [ ] No `web_accessible_resources` entries for files that no longer exist
- [ ] Extension version is incremented from the last published version
- [ ] All files listed in `web_accessible_resources` exist on disk
- [ ] No `content_security_policy` overrides permitting `unsafe-eval` or `unsafe-inline`

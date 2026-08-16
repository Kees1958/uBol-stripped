# Injection architecture evaluation

**⚠️ Status update (7.7.9–7.8): this document's conclusion has been
PARTIALLY REVERSED for cosmetic rules — read this notice before the
"Why uBol-stripped uses the large chunk approach" section below, or its
reasoning will read as still fully in effect when it isn't.**

As of `7.8`, this project runs **two different architectures for its two
main injection categories**, not one uniform choice:

| | Scriptlets | Cosmetic (CSS-hiding) rules |
|---|---|---|
| Architecture | **Large chunk** (unchanged) | **Small chunk** (changed 7.7.9) |
| How data reaches the page | Self-contained per-ruleset `.js` file, baked-in, registered via `registerContentScripts()` | `chrome.storage.local` read at `document_start`, via a shared reader (`css-specific.js`) + tiny per-ruleset loader |
| The `7.7.7` fix | Internal to this architecture — each of ~100 scriptlets became its own top-level function inside the same baked-in file, so only the one needed gets serialized per call. No architecture change, no new risk category. | N/A |
| The `7.7.9`–`7.8` fix | N/A | Full architecture switch, away from what this document recommends |

**Why the switch happened despite this document's own conclusion**:
the same size-reduction motivation this document already judged "mostly
solved, not worth the tradeoff" for scriptlets was judged differently
for cosmetic rules — the per-page cost of re-parsing a large embedded
blob was considered high enough (AdGuard Base's cosmetic file alone was
~1.7 MB, re-parsed as code on every matching navigation per a real
Chromium bug report on content-script code-caching) to accept exactly
the risk category this document warns about: FOUC exposure, service-
worker-wakeup dependency, and the general "storage-backed at injection
time" failure class.

**Was this document's warning validated or refuted?** Neither, fully.
The originally-planned automated real-browser verification (this
document's own implicit bar — "each of which needs real-browser
verification this environment can't perform") was never completed; no
real Chrome was reachable from the network available at the time. What
exists instead is *informal* evidence: several fresh page loads on real
sites with confirmed cosmetic rules, reviewed frame-by-frame via phone
slow-motion camera, no flash observed, plus a Speedometer 3.1 run
showing no regression. On the strength of that — explicitly weaker than
what this document calls for, and documented as such rather than
overstated — the fallback to the large-chunk cosmetic path was removed
entirely in `7.7.10`; there is no runtime flag to revert to anymore (see
`CHANGELOG.md`'s `7.7.9`/`7.7.10` entries and `PROJECT_HANDOVER.md` for
the full history).

**What this means for this document's reasoning below**: everything in
"uBO-lite's approach" and its "Disadvantages" list, and the "Update —
countdown-hang failure mode" section, describes real, still-current risk
— it just now describes the risk the *cosmetic* path deliberately took
on, not a risk the project avoided across the board. Read "uBol-
stripped's approach" below as describing the *scriptlet* path
specifically, not a project-wide guarantee.

---

Two ways to get cosmetic hides and scriptlets onto a page were considered
for this project: **uBol-stripped's approach** (the design actually used
here — "large chunk approach", self-contained per-ruleset content
scripts) and **uBO-lite's approach** (upstream uBlock Origin Lite's
design — "small chunk approach", a shared interpreter reading
storage-backed ruleset data). This document weighs both and justifies
which one uBol-stripped uses.

## uBol-stripped's approach: self-contained per-ruleset content scripts (large chunk)

Each ruleset ships as its own content script — cosmetic selectors and
scriptlet dispatch data baked directly into that script's own source,
registered via `chrome.scripting.registerContentScripts()`. Toggling a
ruleset means registering or unregistering its script.

### Benefits

- **No flash-of-unstyled-content is possible, structurally.** Everything
  needed is already present in the registered script's own source at
  `document_start` — there's no async read standing between injection
  time and dispatch.
- **Doesn't depend on the service worker being awake at page-load time.**
  Once a content script is registered, it just runs. The service worker
  is only involved when *toggling* a ruleset — an infrequent, non-time-
  sensitive action — not on every page load. MV3 service workers going
  dormant after ~30s idle can't affect this path at all.
- **No cache, so no cache-eviction logic to get wrong.** Nothing is
  cached; there's nothing to evict, expire, or race against.
- **No storage quota exposure.** Ruleset data lives in the shipped
  extension bundle, not in `storage.local`, so there's no quota ceiling
  to hit and no silent-write-failure mode to handle.
- **Directly traceable for debugging.** The shipped file *is* the
  runtime behavior — reading the source tells you exactly what will run,
  with no indirection through live storage state to also inspect.
- **Lower ongoing maintenance surface.** No schema to version, no
  migration step required on future releases, no startup-registration-
  ordering requirement to maintain correctly forever.
- **Most of the original size motivation is already captured.** This
  project's own debloat work cut AdGuard Base's per-page footprint from
  ~5.5MB to ~2.5MB (55%) through dispatch-table format/shape changes
  alone — without touching the injection architecture. The remaining gap
  against uBO-lite's model is real but smaller than it was before that
  work.

### Disadvantages

- **Duplicated scriptlet bodies across rulesets.** A scriptlet used by
  several rulesets (e.g. `prevent-canvas`) has its function body repeated
  in each ruleset's own file, rather than shared once across all of them.
- **Toggling a ruleset costs a content-script re-registration**, not a
  cheap storage write — heavier, though toggling is infrequent enough
  that this is a minor cost in practice.
- **Per-hostname dispatch logic exists once per ruleset file** rather
  than once globally, so any future dispatch-logic bug fix has to be
  correct in every ruleset's build output, not one shared interpreter
  (mitigated by the build script being the single source that generates
  all of them, but still N shipped copies of the same logic).

## uBO-lite's approach: shared interpreter + storage-backed data (small chunks)

Two interpreters (cosmetic, scriptlet) are registered once, globally.
Ruleset data ships as plain JSON; toggling a ruleset fetches its JSON and
writes it to `storage.local`/`storage.session`. The interpreters read
whichever rulesets are currently in storage at injection time.

### Benefits

- **No duplicated scriptlet bodies.** A scriptlet used across many
  rulesets is written once, in the shared interpreter, regardless of how
  many rulesets reference it.
- **Toggling a ruleset is a storage write, not a content-script
  re-registration** — lighter weight, though the practical difference is
  small given how infrequently rulesets are toggled.
- **Matches upstream uBO-lite**, which could simplify comparing behavior
  against or porting fixes from the project uBol-stripped is based on.

### Disadvantages

- **Flash-of-unstyled-content becomes structurally possible.** Injection
  now depends on an async `storage.session` read at `document_start` —
  even cached, this isn't synchronous the way baked-in source is.
- **Depends on the service worker successfully waking up to service the
  toggle-time fetch.** MV3 service workers going dormant is routine, and
  there's a documented Chromium bug class where a message/fetch can hang
  indefinitely (never resolving or rejecting) if the service worker
  terminates at the wrong moment mid-wakeup — the same failure mode
  behind uBol-stripped's own Privacy Inspector countdown-close bug (now
  fixed and formally documented as a reusable pattern — see "Update"
  below), at a larger blast radius here (every ruleset toggle, not one
  panel's countdown).
- **Needs correct cache-eviction logic**, and getting LRU eviction
  subtly wrong (off-by-one, races, stale "last accessed" timestamps) is
  the kind of bug that passes isolated testing and only surfaces under
  real multi-day, multi-tab usage.
- **Introduces storage quota exposure** — compiled ruleset data (several
  MB for AdGuard Base alone) now has to fit within `storage.local`
  limits, with a silent-failure mode at toggle time that a baked-in file
  doesn't have.
- **Debugging requires inspecting live storage state**, not just reading
  a shipped file, since the interpreter's behavior depends on runtime
  data rather than its own source.
- **Requires an ongoing maintenance surface**: a versioned schema,
  registration-order sequencing at startup, and a migration step for
  every existing install going forward.

## Why uBol-stripped uses the "large chunk" approach

*(Now true for scriptlets only, not cosmetic rules — see the status
notice at the top of this document.)*


Three reasons, in order of weight:

1. **The main problem uBO-lite's model would solve — file size — is
   already mostly solved**, and was solved the lower-risk way. Using
   prevelence (top 1 million websites) together with 5Eyes and extended
   EU-zone location cut AdGuard Base's footprint by 60% without touching 
   the injection architecture at all. What's left on the table (duplicated 
   scriptlet bodies across rulesets) is real but modest next to what's 
   already been captured.

2. **uBO-lite's (smart but not simple)model introduces failure modes 
   uBol-stripped has already spent real effort fixing at a larger
   scale. Trading a solved, structurally-impossible-here problem for a 
   harder-to-verify one at that scale is a bad trade unless the benefit 
   is large — and per point 1, it currently isn't.

3. **uBol-stripped's disadvantages are genuinely minor**: some code
   duplication across ruleset files (a build-time concern, not a
   runtime-correctness one) and a slightly heavier ruleset-toggle
   operation (infrequent enough not to matter in practice). uBO-lite's
   disadvantages are runtime-correctness risks — flash-of-content,
   dormant-service-worker hangs, cache-eviction bugs, quota exposure —
   each of which needs real-browser verification this environment can't
   perform, and each of which fails in ways a user actually notices
   (unblocked ads flashing, or ad-blocking silently not applying).

If duplicated scriptlet bodies become a large enough fraction of shipped
size to justify revisiting this, that would be the trigger to reconsider
— not simply "this is how upstream uBO-lite does it."

## Update — the countdown-hang failure mode is now a documented, reusable pattern

Since this evaluation was first written, the exact Chromium bug class
cited in point 2 above (service-worker dormancy causing an in-flight
message/fetch to hang indefinitely, never resolving or rejecting) was
hardened against directly in Privacy Inspector's own session-timeout
handling — not just patched once, but generalized into two places in
uBol-stripped's own engineering-principles reference (described in
programming principles):

- **B3b — "Self-closing timed-session panels"**: the required pattern
  for *any* feature with a timed session and a UI panel that must
  reliably close itself, an active timer broadcast and an independent 
  poll as a second, unrelated safety net. 

- **C21 — "Reuse a proven solution across sibling features"**: once one
  feature has a solution proven against a real, documented failure, a
  sibling feature solving the same underlying problem must adopt that
  same solution rather than inventing a new one.

This strengthens point 2 rather than changing it: the failure mode
uBO-lite's toggle-time fetch would be exposed to isn't a hypothetical
risk described in the abstract anymore — it's a bug class this project
has now fixed once, named, and written down as a pattern precisely
*because* getting it wrong was expensive enough to be worth formalizing.
Adopting uBO-lite's model would mean deliberately taking on that same
failure mode again, at a larger blast radius (every ruleset toggle, for
every user, on every page load) than the one panel it was already fixed
for — with no size-reduction upside large enough to justify it, per
points 1 and 3 above.

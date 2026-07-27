# Injection architecture evaluation

Two ways to get cosmetic hides and scriptlets onto a page were considered
for this fork: the current design ("large chunk approach" — self-contained
per-ruleset content scripts) and real uBOL's design ("uBOL's approach" —
a shared interpreter reading storage-backed ruleset data). This document
weighs both and justifies which one this fork uses.

## uBol-stripped approach: self-contained per-ruleset content scripts (large chunk)

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
  fork's own debloat work cut AdGuard Base's per-page footprint from
  ~5.5MB to ~2.5MB (55%) through dispatch-table format/shape changes
  alone — without touching the injection architecture. The remaining gap
  against uBOL's model is real but smaller than it was before that work.

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

## uBOL's approach: shared interpreter + storage-backed data (small chunks)

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
- **Matches upstream uBOL**, which could simplify comparing behavior
  against or porting fixes from the project this fork is based on.

### Disadvantages

- **Flash-of-unstyled-content becomes structurally possible.** Injection
  now depends on an async `storage.session` read at `document_start` —
  even cached, this isn't synchronous the way baked-in source is.
- **Depends on the service worker successfully waking up to service the
  toggle-time fetch.** MV3 service workers going dormant is routine, and
  there's a documented Chromium bug class where a message/fetch can hang
  indefinitely (never resolving or rejecting) if the service worker
  terminates at the wrong moment mid-wakeup — the same failure mode
  behind this fork's Privacy Inspector countdown-close bug, at a larger
  blast radius here (every ruleset toggle, not one panel's countdown).
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


## Why this fork uses the "large chunk" approach

Three reasons, in order of weight:

1. **The main problem uBOL's model would solve — file size — is already
   mostly solved**, and was solved the lower-risk way. Phase 1/1.5's
   dispatch-format work cut AdGuard Base's footprint by 55% without
   touching the injection architecture at all. What's left on the table
   (duplicated scriptlet bodies across rulesets) is real but modest next
   to what's already been captured.

2. **uBOL's model introduces failure modes this fork has already spent
   real effort fixing in a different subsystem, at a larger scale.** The
   service-worker-dormancy hang that broke Privacy Inspector's countdown-
   close behavior is the same underlying Chromium bug class uBOL's
   toggle-time fetch would be exposed to — but Privacy Inspector's
   countdown is one panel, occasionally left open; ruleset injection
   happens on every page load, for everyone. Trading a solved,
   structurally-impossible-here problem for a harder-to-verify one at
   that scale is a bad trade unless the benefit is large — and per point
   1, it currently isn't.

3. **uBol-stripped approach's disadvantages are genuinely minor**: some code
   duplication across ruleset files (a build-time concern, not a
   runtime-correctness one) and a slightly heavier ruleset-toggle
   operation (infrequent enough not to matter in practice). uBOL's
   disadvantages are runtime-correctness risks — flash-of-content,
   dormant-service-worker hangs, cache-eviction bugs, quota exposure —
   each of which needs real-browser verification this environment can't
   perform, and each of which fails in ways a user actually notices
   (unblocked ads flashing, or ad-blocking silently not applying).

If duplicated scriptlet bodies become a large enough fraction of shipped
size to justify revisiting this, that would be the trigger to reconsider
— not simply "this is how upstream uBOL does it."

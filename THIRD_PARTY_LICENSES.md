# Third-party licenses and attribution

This project (uBlock-Stripped-Dynamic) is licensed under the GNU General
Public License v3.0 — see `LICENSE.txt`. The following components are
derived from third-party sources under their own, different licenses.
Each is documented here as the single canonical source — not scattered
across code comments — per this project's own "declared, not
reconstructed from memory" convention (see e.g. `RULESET_COMPANIONS`).

---

## DuckDuckGo Tracker Radar (compacted dataset)

- **File:** `js/data/tracker-radar-data.js` (generated — see
  `tools/build-tracker-radar.mjs`)
- **Source:** https://github.com/duckduckgo/tracker-radar
- **License:** [Creative Commons Attribution-NonCommercial-ShareAlike 4.0
  International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
- **Copyright:** © Duck Duck Go, Inc.

**Attribution (TASL — Title, Author, Source, License):**

> "Tracker Radar" data by Duck Duck Go, Inc., available at
> https://github.com/duckduckgo/tracker-radar, licensed under
> [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
> This project's own compacted subset (`js/data/tracker-radar-data.js`,
> generated via `tools/build-tracker-radar.mjs`) is a derivative work
> and is itself distributed under the same CC BY-NC-SA 4.0 terms — see
> `tools/build-tracker-radar.mjs`'s own header for the exact build
> parameters (region/prevalence filtering) applied to produce it.

**What CC BY-NC-SA 4.0 means in practice for this specific file only**
(does not extend to the rest of the GPL-3.0-licensed extension — this
is a bundled, separately-licensed data file, not a derivative of the
extension's own code):
- **Attribution** — this notice, kept up to date, satisfies that term.
- **NonCommercial** — this specific dataset may not be used for
  commercial purposes without a separate license from Duck Duck Go, Inc.
  (they explicitly offer commercial licensing — see their repo).
- **ShareAlike** — any further redistribution of this compacted dataset
  (not the extension as a whole) must carry the same CC BY-NC-SA 4.0
  terms.

---

## `vendor/autoconsent/` — MIXED provenance, resolved (per-entry sourced)

- **Files:** `vendor/autoconsent/eval-snippets.js`,
  `vendor/autoconsent/rules/rules.json`
- **`vendor/autoconsent/LICENSE`** (MPL-2.0, fetched verbatim from
  `duckduckgo/autoconsent`'s own repo) applies to part of this
  directory's content — confirmed via `rules.json`'s own `_provenance`
  field, which was more detailed than its top-level `_comment` and
  resolves what first looked like a contradiction between the two files:

  - **23 of 45 rule entries** (`rules.json`, was 12 of 26) carry
    `_source: "ddg-autoconsent (<filename>.json)"` — genuinely
    **adapted (not copy-pasted)** from `duckduckgo/autoconsent`'s own
    real, tested rule definitions. Original 12 read from a pinned
    commit (`9e4e640d...`); batch 4's 11 additional entries (8.4.9)
    read from the `main` branch instead — dated, not commit-pinned,
    since GitHub's commit API was rate-limited at fetch time. Two
    further candidates (Transcend, Sirdata) were deliberately deferred
    — see CHANGELOG for why. **MPL-2.0 applies to all 23.**
  - **14 of 45 rule entries** carry `_source: "hand-authored"` — genuinely independently written against each CMP's own public
    documentation/markup, confirmed via live-site inspection, no DDG
    code involved. **MPL-2.0 does not apply to these.**
  - **8 new entries (added 8.4.7/8.4.8, three batches)** carry
    `_source: "consent-o-matic (<filename>.json)"` — adapted the same
    way, from `cavi-au/Consent-O-Matic`'s own real rule definitions.
    **MIT applies to these — see below**, not MPL-2.0.
  - `eval-snippets.js`'s top-of-file "NOT a fork" claim is accurate as a
    whole-file/whole-project statement (this directory is not a fork of
    duckduckgo/autoconsent), but doesn't capture that a genuine subset
    of individual entries within it were adapted from DDG's code —
    `rules.json`'s more detailed `_provenance` field is the accurate,
    complete picture; the file-level `_comment` undersold it.

  **Per-file MPL-2.0 notices aren't practical here** (mixed at the
  individual-rule level within both files) — MPL-2.0 §3.1 itself
  anticipates exactly this: *"If it is not possible or desirable to put
  the [license] notice in a particular file, then You may include the
  notice in a location (such as a LICENSE file in a relevant
  directory)"* — which is what `vendor/autoconsent/LICENSE` now does.

- **`js/scripting/autodeny-snippets.generated.js`** is compiled at
  build time from both files above (see `tools/build-autodeny-bundle.mjs`)
  and inherits the same mixed status — the MPL-2.0-derived portion
  flows through into the generated file along with the hand-authored
  and MIT-derived portions.

---

## Consent-O-Matic (8 rule entries, added 8.4.7/8.4.8, three batches)

- **Source:** https://github.com/cavi-au/Consent-O-Matic
- **License:** [MIT License](https://github.com/cavi-au/Consent-O-Matic/blob/master/LICENSE)
- **Copyright:** © 2019-2022 Janus Bager Kristensen and Rolf Bagge, CAVI —
  Centre for Advanced Visualization and Interaction, Aarhus University

**Attribution:**

> Cookie-consent detection/reject logic for TYPO3 Cookieman,
> tarteaucitron.js, Piwik PRO Consent Manager, GDPR Modal, Drupal EU
> Cookie Compliance, Evidon, ST CMP v2, and a Thai-market DPDPA-style
> consent popup adapted from Consent-O-Matic by CAVI, Aarhus
> University, licensed under the MIT License. This copyright notice is
> the only term MIT requires to be preserved — no copyleft, no
> non-commercial restriction, unlike the two entries above.

Candidates using build-hash-dependent CSS-in-JS class names (batch 1:
Admiral, Mediavine's save-button selector; batch 2: Schibsted; batch 3:
google_cwiz — Google's own opaque MDC classnames plus position-
dependent nth-child selectors, a fragility type the earlier automated
scan didn't catch, found only on manual inspection) were deliberately
excluded across all three batches as too fragile to translate
faithfully — see CHANGELOG for the full reasoning. ~190 CMPs remain in
Consent-O-Matic's own rule set beyond these three pilot batches.

# Third-party licenses and attribution

This project (uBlock-Stripped-Dynamic) is licensed under the GNU General
Public License v3.0 — see `LICENSE.txt`. The following components are
derived from third-party sources under their own, different licenses.
Each is documented here as the single canonical source — not scattered
across code comments — per this project's own "declared, not
reconstructed from memory" convention (see e.g. `RULESET_COMPANIONS`).

---

## `vendor/autoconsent/` — MIXED provenance, resolved (per-entry sourced)

- **Files:** `vendor/autoconsent/eval-snippets.js`,
  `vendor/autoconsent/rules/rules.json`
- **`vendor/autoconsent/LICENSE`** (MPL-2.0, fetched verbatim from
  `duckduckgo/autoconsent`'s own repo) applies to part of this
  directory's content — confirmed via `rules.json`'s own `_provenance`
  field, which was more detailed than its top-level `_comment` and
  resolves what first looked like a contradiction between the two files:

  - **22 of 45 rule entries** (`rules.json`, was 11 of 26, not 12 —
    see correction note below) carry `_source: "ddg-autoconsent
    (<filename>.json)"` — genuinely **adapted (not copy-pasted)** from
    `duckduckgo/autoconsent`'s own real, tested rule definitions.
    Original 11 read from a pinned commit (`9e4e640d...`); batch 4's
    11 additional entries (8.4.9) read from the `main` branch instead
    — dated, not commit-pinned, since GitHub's commit API was
    rate-limited at fetch time. Two further candidates (Transcend,
    Sirdata) were deliberately deferred — see CHANGELOG for why.
    **MPL-2.0 applies to all 22.**
  - **15 of 45 rule entries** carry `_source` starting with
    `"hand-authored"` — genuinely independently written against each
    CMP's own public documentation/markup, confirmed via live-site
    inspection or (Silktide specifically) multiple independent real
    production deployments plus that project's own public config
    schema — no DDG code involved in any of these 15. **MPL-2.0 does
    not apply to these.** (Corrected from an earlier 23/14 split —
    Silktide's `_source` string carries extra verification detail
    beyond the plain word "hand-authored" and was miscounted in an
    earlier pass; verified directly against `rules.json` before this
    release, not recalculated from memory.)
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

---

## Disconnect ConsentManagers (1 domain, added 8.5.4)

- **File:** `tools/build-rulesets.mjs`'s `buildWorryFreeExtraBlocklistSourceText()`
  — a single curated domain, not a live-fetched source.
- **Source:** https://github.com/disconnectme/disconnect-tracking-protection
  (`services.json`, `ConsentManagers` category)
- **License:** [Creative Commons Attribution-NonCommercial-ShareAlike 4.0
  International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
  — same standard CC BY-NC-SA 4.0 terms used throughout this document.
- **Copyright:** © 2010-2026 Disconnect, Inc.

**Attribution:**

> `transcend.io` (Transcend's own consent-manager infrastructure
> domain) added to the Worry-free extra blocklist, sourced from
> Disconnect's own `services.json`, `ConsentManagers` category,
> licensed under CC BY-NC-SA 4.0. Disconnect's other 6 ConsentManagers
> entries (Axeptio, Didomi, OneTrust, Osano, TrustARC, UserCentrics)
> were deliberately NOT added — this project already has a working
> click-based reject rule for each of those 6 (see the DuckDuckGo/
> Consent-O-Matic entries above), and a domain-block would add nothing
> while risking breaking an already-working mitigation. Transcend was
> the one CMP in that list without an equivalent working click rule —
> its own duckduckgo/autoconsent rule was excluded from this project
> (8.4.9) for being cosmetic-only (hides the widget via CSS, doesn't
> actually reject).

---

## Anti-tracking, Ads & Tracking Networks lists (Kees1958 + Disconnect)

Two separately-selectable static rulesets. Originally five (Kees1958,
Disconnect ads, Disconnect other, Ghostery ads, Ghostery other), reduced
in stages — Ghostery removed entirely (see CHANGELOG), Disconnect's two
lists re-merged into one (Advertising folded into the
Analytics/Cryptomining/Fingerprinting list, Analytics itself dropped —
see CHANGELOG for the exact category change):

- **Kees1958 most-used EU+US ads & tracking networks**
  - **File:** `tools/data/kees1958-only-source.txt` (generated — see
    `tools/build-adtracking-sources.mjs`), compiled into
    `ruleset_kees1958`.
  - **Source:** Kees1958's own `EU_US_MV3_most_common_ad+tracking_
    networks.txt` (https://github.com/Kees1958/W3C_annual_most_used_
    survey_blocklist) — license terms per that repository's own README,
    not CC BY-NC-SA 4.0.
  - **No safety filter applied** — used as-is, per explicit request (see
    CHANGELOG). Previously cross-referenced against the reviewList
    safety filter described below; that filtering was removed for this
    list specifically.
- **Disconnect (Ads & trackers, Cryptomining, Fingerprinting)**
  - **File:** `tools/data/disconnect-other-source.txt`, compiled into
    `ruleset_disconnect_other`.
  - **Source:** Disconnect's `services.json` — `Advertising`,
    `Cryptomining`, `FingerprintingInvasive`, `FingerprintingGeneral`
    categories, combined into one selectable list per explicit request.
    `Analytics` (previously included) dropped entirely, not just from
    the title.
  - **No safety filter applied** — used as-is, per explicit request (see
    CHANGELOG). Previously cross-referenced against a reviewList safety
    filter (as two separate lists, pre-merge); that filtering was
    removed when the two lists were merged, and the underlying
    cross-reference mechanism itself was removed entirely in a later
    release (see below) once nothing else in the codebase still read it.
  - **License:** [Creative Commons Attribution-NonCommercial-ShareAlike
    4.0 International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

Kees1958's own list carries its own separate license terms, not CC
BY-NC-SA 4.0.

**Safety filter — removed entirely, not just unwired.** The `reviewList`
cross-reference mechanism (`tools/data/top1m-tracker-crossref.json` /
`tools/build-top1m-crossref.mjs`) and the separate DuckDuckGo Tracker
Radar dataset (`js/data/tracker-radar-data.js` /
`tools/build-tracker-radar.mjs`) both used to power the Matrix panel's
"known DDG Tracker" breakage-risk column. That column was removed
outright (obsolete now that Worry-Free mode's own filtering already
stops most trackers — see CHANGELOG), and a full-codebase grep before
removal confirmed nothing else read either dataset. All four files, plus
their generator scripts, were deleted; every entry that used to document
them (DuckDuckGo Tracker Radar, Top-1M/Tracker-Domain Crossref) has been
removed from this document accordingly — see CHANGELOG for the removal
entry itself, kept here only where these ex-datasets are still relevant
context (this section).

**History:** originally one merged file (`ruleset_kees1958` alone,
"Merged Anti-tracking list (Kees1958 + Disconnect + Ghostery)") — split
into five independently-selectable lists (v9.1.1/v9.1.7) to allow
narrowing a filter-list issue down to one specific source, then reduced
back down to these two (this release) — Ghostery removed, Disconnect's
two lists re-merged, per explicit request in both cases.

---

<!-- RETIRED (v9.1.7) — "Worry-free social media block list (Disconnect
     + Ghostery)" removed entirely, per explicit request, along with the
     "Enable extra blocklist" grouped checkbox and the EasyList adult
     ad-servers + transcend.io list it also used to cover. Neither
     ruleset_worryfree_social nor ruleset_worryfree_extra exist anymore
     — see js/core/dnr-budgets.js's WORRY_FREE_SOCIAL_RULES_BASE_ID /
     WORRY_FREE_EXTRA_RULES_BASE_ID retirement comments for the full
     history. No attribution entry needed here since nothing in the
     shipped extension derives from Disconnect's Social category or
     Ghostery's social_media category anymore. -->

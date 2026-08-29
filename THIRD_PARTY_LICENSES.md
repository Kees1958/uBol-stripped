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
  — same license and same NonCommercial/ShareAlike terms as the
  DuckDuckGo Tracker Radar entry above.
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

## Top-1M / Tracker-Domain Crossref (compacted dataset)

- **File:** `tools/data/top1m-tracker-crossref.json` (generated — see
  `tools/build-top1m-crossref.mjs`)
- **Sources (TWO upstream datasets, both required to reproduce):**
  1. https://github.com/disconnectme/disconnect-tracking-protection
     (`services.json`)
  2. https://github.com/ghostery/trackerdb (`@ghostery/trackerdb` npm
     package's pre-built `dist/trackerdb.json` export)
  3. Ranking reference (not itself redistributed, only cross-referenced
     against): https://github.com/zakird/crux-top-lists — the same CrUX
     top-1M source already used by `tools/build-rulesets.mjs`'s
     `CRUX_TOP_SITES_URL`.
- **License:** [Creative Commons Attribution-NonCommercial-ShareAlike 4.0
  International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
  — both upstream tracker datasets (1) and (2) carry identical CC BY-NC-SA
  4.0 terms; the compacted crossref inherits the same terms from both.
- **Copyright:** © Disconnect, Inc. (services.json); © Ghostery GmbH
  (TrackerDB)

**Attribution (TASL — Title, Author, Source, License), both sources:**

> "Disconnect Tracker Protection Lists" by Disconnect, Inc., available at
> https://github.com/disconnectme/disconnect-tracking-protection, licensed
> under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
>
> "Ghostery Tracker Database" by Ghostery GmbH, available at
> https://github.com/ghostery/trackerdb, licensed under
> [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
>
> This project's own cross-reference (`tools/data/top1m-tracker-crossref.json`,
> generated via `tools/build-top1m-crossref.mjs`) is a derivative work of
> both sources combined, and is itself distributed under the same CC
> BY-NC-SA 4.0 terms — see the script's own `CONFIG` block and header
> comment for the exact classification rules (which categories from each
> vendor count as "marketing-related") applied to produce it.

**What this file actually contains, and what it doesn't:** it is NOT a
blocklist and NOT an allowlist on its own — it's a classification aid.
`includeList` marks tracker domains that are also top-1M destinations but
whose own vendor category confirms a genuine advertising/analytics/social
role (safe to keep as blocking targets, third-party-scoped).
`reviewList` marks tracker-listed top-1M domains with no such
marketing-category confirmation — flagged for manual review, not
pre-approved for exclusion. Consuming code must not silently treat either
list as a ready-made allow/block decision; see the TODO in
`tools/build-top1m-crossref.mjs` for the open decision this still requires.

**Same NonCommercial/ShareAlike terms as the existing Tracker Radar and
Disconnect ConsentManagers entries above** — this file may not be used
for commercial purposes without separate licensing from Disconnect, Inc.
and/or Ghostery GmbH, and any further redistribution of this compacted
crossref must carry the same CC BY-NC-SA 4.0 terms.

---

## Merged Anti-tracking list (Kees1958 + Disconnect + Ghostery)

- **File:** `tools/data/merged-adtracking-source.txt` (generated — see
  `tools/build-merged-adtracking-list.mjs`), compiled into the always-on
  `ruleset_kees1958` ruleset (see `tools/build-rulesets.mjs`) — REPLACES
  the previous Kees1958-only source of that same ruleset, not an
  additional fourth always-on list.
- **Sources (three, merged and deduplicated):**
  1. Kees1958's own `EU_US_MV3_most_common_ad+tracking_networks.txt`
     (https://github.com/Kees1958/W3C_annual_most_used_survey_blocklist) —
     license terms per that repository's own README.
  2. Disconnect's `services.json` — `Advertising`, `Analytics`,
     `FingerprintingInvasive`, `FingerprintingGeneral` categories — same
     CC BY-NC-SA 4.0 source already documented above in this file.
  3. Ghostery TrackerDB — `advertising`, `site_analytics` categories —
     same CC BY-NC-SA 4.0 source already documented above in this file.
- **License:** CC BY-NC-SA 4.0 inherited from sources (2) and (3); source
  (1)'s own terms apply to the portion contributed from it. This merged,
  compacted file as a whole is a derivative work and carries the same
  NonCommercial/ShareAlike restrictions as the CC BY-NC-SA 4.0 portions.

**Why fingerprinting is folded in here, not a separate list:** Ghostery
has zero fingerprinting-specific category or tag anywhere in its data
(confirmed by direct inspection of all 3,533 patterns' `tags` fields).
Rather than ship a Disconnect-only fingerprinting list under a misleading
"from Disconnect and Ghostery" banner, Disconnect's two fingerprinting
categories are merged into this same list instead.

**Safety filter:** cross-referenced against `tools/data/top1m-tracker-
crossref.json`'s `reviewList` — any domain there (tracker-listed but NOT
confirmed advertising/analytics/social by either vendor, and a real
top-1M destination) is excluded from this merged list. See that file's
own entry above for what this filter is and why it exists.

---

## Worry-free social media block list (Disconnect + Ghostery)

- **File:** `tools/data/social-media-source.txt` (generated — see
  `tools/build-social-media-list.mjs`), compiled into the
  `ruleset_worryfree_social` ruleset — active only during a Worry-free
  session, gated by the `worryFreeSocialBlockEnabled` setting (default
  ON — see `js/data/config.js`).
- **Sources:** Disconnect's `services.json` `Social` category + Ghostery
  TrackerDB's `social_media` category — both CC BY-NC-SA 4.0, same
  sources already documented above in this file.
- **License:** CC BY-NC-SA 4.0, same NonCommercial/ShareAlike terms as
  every other Disconnect/Ghostery-derived entry in this file.
- **Safety filter:** same `top1m-tracker-crossref.json` `reviewList`
  cross-reference as the merged Anti-tracking list above, applied for
  consistency even though the underlying reasoning differs slightly here
  — major social platforms being both tracker-listed and top-1M is
  usually the CORRECT block target for this specific list (a Worry-free
  session is meant to suppress embedded widgets), not a first-party-risk
  signal the way it is for the core always-on list.

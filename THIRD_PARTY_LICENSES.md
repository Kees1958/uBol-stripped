# Third-party licenses and attribution

This project (uBlock-Stripped-Dynamic) is licensed under the GNU General
Public License v3.0 — see `LICENSE.txt`. The following components are
derived from third-party sources under their own, different licenses.
Each is documented here as the single canonical source — not scattered
across code comments — per this project's own "declared, not
reconstructed from memory" convention (see e.g. `RULESET_COMPANIONS`).

**Audited against the code and tools in 9.7.3** (every path below checked
to exist; every licence re-read from the upstream repository's own
`LICENSE` file or the list's own header where reachable). Where a licence
could **not** be verified from the build environment, the entry says so
explicitly instead of guessing — see "Unverified" markers, and the
summary table under "Upstream filter lists". This is a record of what
was found, not legal advice.

**Two constraints that apply to the extension as a whole**, because of
what it bundles:
- **NonCommercial.** The shipped extension contains data under
  CC BY-NC-SA 4.0 (DuckDuckGo Tracker Radar, Disconnect, the Polish
  filter list) and Peter Lowe's list (McRae General Public License —
  "nothing that could be construed as making anybody any money"). The
  extension therefore must not be sold or monetised; the separately-
  licensed data files are not relicensed to GPL-3.0 by being bundled.
- **ShareAlike.** The compiled ruleset/data files derived from those
  sources (`rulesets/ruleset_disconnect_other.json`,
  `rulesets/ruleset_easylist_polish.json`, `js/data/tracker-radar-data.json`,
  and the CC BY-SA lists below) stay under their source's terms if
  redistributed on their own.

---

## uBlock Origin / uBlock Origin Lite / uMatrix (Raymond Hill) — the upstream code base

- **Where:** 53 source files — 48 under `js/`, 3 under `dashboard/`, 1 under
  `picker/`, 1 under `popup/` — carry a `Copyright (C) … Raymond Hill` GPL
  header with `Home: https://github.com/gorhill/uBlock` (52 files) or
  `https://github.com/gorhill/uMatrix` (1 file, `js/ui/fa-icons.js`).
  Three further files (`js/scripting/cookie-clicker-picker.js`,
  `js/scripting/cookie-clicker-click.js`,
  `js/core/cookie-clicker-autodeny-manager.js`) carry `Copyright (C) …
  Kees1958` with `Home: https://github.com/Kees1958/uBol-stripped` (this
  project).
- **License:** GPL-3.0-or-later — the same license as this project, so
  bundling adds no restriction. The per-file headers are kept verbatim;
  that is the attribution.

---

## DuckDuckGo Tracker Radar (compacted dataset)

- **File:** `js/data/tracker-radar-data.json` (generated — see
  `tools/build-tracker-radar.mjs`, rebuilt from a real checkout: 2,103
  domains / 700 entities, 2026-09-09 — see that file's own header).
- **Source:** https://github.com/duckduckgo/tracker-radar
- **License:** [Creative Commons Attribution-NonCommercial-ShareAlike 4.0
  International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
- **Copyright:** © Duck Duck Go, Inc.

**Attribution (TASL — Title, Author, Source, License):**

> "Tracker Radar" data by Duck Duck Go, Inc., available at
> https://github.com/duckduckgo/tracker-radar, licensed under
> [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).
> This project's own compacted subset (`js/data/tracker-radar-data.json`,
> generated via `tools/build-tracker-radar.mjs`) is a derivative work
> and is itself distributed under the same CC BY-NC-SA 4.0 terms.

**What CC BY-NC-SA 4.0 means in practice for this specific file only**
(does not extend to the rest of the GPL-3.0-licensed extension):
- **Attribution** — this notice, kept up to date, satisfies that term.
- **NonCommercial** — this specific dataset may not be used for
  commercial purposes without a separate license from Duck Duck Go, Inc.
- **ShareAlike** — any further redistribution of this compacted dataset
  must carry the same CC BY-NC-SA 4.0 terms.

**Consumed by:** `js/core/tracker-radar-lookup.js` (general-purpose
lookup, no UI of its own) and the Matrix panel's "Known tracker" column
(`matrix/matrix-panel.js` / `js/core/matrix-panel.js`) — see CHANGELOG
for the history of this dataset's prior removal and this restoration.

---

## `lib/csstree/`

- **Source:** https://github.com/csstree/csstree
- **License:** MIT (© Roman Dvornov) — full text in `lib/csstree/LICENSE`
  (bundled verbatim).

---

## `css/fonts/Inter/`

- **Source:** https://github.com/rsms/inter
- **License:** SIL Open Font License 1.1 (© The Inter Project Authors) —
  full text in `css/fonts/Inter/LICENSE.txt` (bundled verbatim).

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

<!-- RETIRED (found stale in the 9.7.3 audit) — "Disconnect ConsentManagers
     (1 domain, added 8.5.4)": a single curated domain, `transcend.io`,
     sourced from Disconnect's `ConsentManagers` category and added by
     `buildWorryFreeExtraBlocklistSourceText()` to the Worry-free extra
     blocklist. That function was removed in 9.1.7 together with the whole
     extra blocklist (see tools/build-rulesets.mjs's own RETIRED comment),
     and nothing in the build now reads Disconnect's `ConsentManagers`
     category; `rulesets/ruleset_disconnect_other.json` has no
     `transcend.io` entry. The entry described a mechanism that no longer
     exists, so it was removed rather than left claiming an attribution
     for content that is not shipped. Disconnect's remaining use (the
     Advertising / Cryptomining / Fingerprinting categories) is documented
     in the next section. -->

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
    survey_blocklist).
  - **Licence: none stated — Unverified.** This entry used to say "per
    that repository's own README"; the 9.7.3 audit re-read that README,
    looked for LICENSE files under the usual names, and checked the
    GitHub repository page: no licence or licence statement exists in
    any of them. Note that `Kees1958` is also the copyright holder named
    in three of this project's own source files (`Home:
    https://github.com/Kees1958/uBol-stripped`) — if the list author and
    this project's author are the same person, the list is first-party and
    this is a non-issue; that identity is not something the code can
    confirm, so it is recorded as a question, not assumed.
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
`tools/build-top1m-crossref.mjs`) was deleted, and its Top-1M/Tracker-Domain
Crossref attribution entry was removed from this document with it (both
files confirmed absent in the 9.7.3 audit). **Correction:** an earlier
version of this paragraph also said the DuckDuckGo Tracker Radar dataset
(`js/data/tracker-radar-data.json` / `tools/build-tracker-radar.mjs`) was
deleted along with the Matrix panel's "known DDG Tracker" column. That is
no longer true — the dataset was later restored and is shipped and read
by `js/core/tracker-radar-lookup.js` and the Matrix panel (checked: the
file holds 2,103 domains / 700 entities, matching the entry above). Its
attribution entry ("DuckDuckGo Tracker Radar", near the top of this document) is
therefore required and current.

**History:** originally one merged file (`ruleset_kees1958` alone,
"Merged Anti-tracking list (Kees1958 + Disconnect + Ghostery)") — split
into five independently-selectable lists (v9.1.1/v9.1.7) to allow
narrowing a filter-list issue down to one specific source, then reduced
back down to these two (this release) — Ghostery removed, Disconnect's
two lists re-merged, per explicit request in both cases.

---

## HaGeZi DNS blocklists (added 9.7.3)

- **HaGeZi's Multi light (DNR network filters)** — `ruleset_hagezi_light`
  - **File:** `rulesets/ruleset_hagezi_light.json` (generated by
    `tools/build-rulesets.mjs`, live-fetched at build time — no local
    copy of the source list is kept in this repository).
  - **Source:** https://github.com/hagezi/dns-blocklists
    (`adblock/light.txt`, "HaGeZi's Multi LIGHT").
  - **License:** [GNU General Public License v3.0](https://github.com/hagezi/dns-blocklists/blob/main/LICENSE)
    — the same license as this project itself, so no additional
    restriction applies. Verified against the repository's own `LICENSE`
    file (full GPLv3 text; the list's own header also points at it),
    not assumed.
  - **How it is used:** each `||domain^` entry is compiled unchanged
    into `requestDomains` block rules (4,000 domains per rule). No entry
    is edited or added. Entries under a top-level domain outside this
    project's scope (generic TLDs plus 5-eyes/EU/extended-EU country
    codes — the same scope its Worry-free tracking-server lists use) are
    left out; see `tools/build-rulesets.mjs`, `buildWorryFreeLists()`.
- **`ad-shield.txt`** (same repository, same license) is one of the three
  sources merged into `ruleset_antiadmiral` (see `tools/build-rulesets.mjs`,
  `ANTI_ADMIRAL_SOURCES`). It was in use before this entry existed but
  was not documented here; recorded now for completeness.

**Attribution:**

> "HaGeZi's Multi LIGHT" and `ad-shield.txt` by HaGeZi and contributors,
> available at https://github.com/hagezi/dns-blocklists, licensed under
> GPL-3.0.

---

## Upstream filter lists compiled into the shipped rulesets (added in the 9.7.3 audit)

`tools/build-rulesets.mjs` fetches each list live at build time and compiles it
into `rulesets/*.json` (DNR rules) and, where the list has cosmetic/scriptlet
rules, `rulesets/*-cosmetic-specific.json` / `*-scriptlets.js`. Until this
audit only Kees1958, Disconnect, HaGeZi and the Tracker Radar were documented;
every other source below was missing. "Verified" means the licence text or the
list's own header was read for this audit; "Unverified" means it could not be
established from the build environment and needs a human check.

| List (ruleset id) | Source | Licence | Status |
|---|---|---|---|
| AdGuard Base + its adult and country-overflow buckets, AdGuard tracking servers, AdGuard anti-adblock (`ruleset_adguard_base`, `_base_adult`, `ruleset_agbase_overflow_*`, `_adguard_tracking_servers`, `_adguard_antiadblock`) | github.com/AdguardTeam/AdguardFilters | GPL-3.0 | Verified (repository `LICENSE`) |
| AdGuard Dutch / German / French / Spanish (`ruleset_adguard_dutch` … `_spanish`, `_french_removeparam`) | github.com/AdguardTeam/FiltersRegistry | LGPL-3.0 | Verified (repository `LICENSE`) |
| AdGuard Scriptlets library (`js/resources/ag-scriptlets-corelibs.json`, v2.5.1, and the scriptlet code inlined in `rulesets/*-scriptlets.js`) | npm `@adguard/scriptlets` / github.com/AdguardTeam/Scriptlets | GPL-3.0 | Verified (repository `LICENSE`) |
| EasyList adult ad-servers (`ruleset_easylist_adservers`) and EasyPrivacy Admiral list (part of `ruleset_antiadmiral`) | github.com/easylist/easylist | Published on easylist.to/pages/licence.html (the repository README only points there; that site is not reachable from the build environment) | **Unverified** — check that page |
| EasyList Italian, EasyList Portuguese (`ruleset_easylist_italian`, `_portuguese`) | github.com/easylist/easylistitaly, …/easylistportuguese | No `LICENSE`, README licence text or list-header licence line found | **Unverified** — presumably EasyList's terms, not confirmed |
| EasyList Czech & Slovak (`ruleset_easylist_czech_slovak`) | github.com/tomasko126/easylistczechandslovak | CC BY-SA 4.0 | Verified (`LICENSE` + list header) — ShareAlike |
| EasyList Latvian (`ruleset_easylist_latvian`) | github.com/Latvian-List/adblock-latvian | CC BY-SA 4.0 | Verified (list header; no `LICENSE` file) — ShareAlike |
| EasyList Lithuanian (`ruleset_easylist_lithuanian`) | github.com/EasyList-Lithuania/easylist_lithuania | GPL-3.0 | Verified (`LICENSE` + list header) |
| EasyList Polish (`ruleset_easylist_polish`) | github.com/MajkiIT/polish-ads-filter | **CC BY-NC-SA 4.0**, © Certyficate IT | Verified (`LICENSE` + list header) — **NonCommercial** + ShareAlike |
| Dandelion Sprout's Nordic Filters (`ruleset_easylist_nordic`) | github.com/DandelionSprout/adfilt | "Dandelicence" v1.4 — permissive, but (near-)unmodified redistributions must keep the licence text or a link to it (https://github.com/DandelionSprout/adfilt/blob/master/LICENSE.md, given here), the maintainers' names may not be used to endorse products, and such copies may not be sold as a standalone paid product in quantities below 500,000 | Verified (`LICENSE.md` + list header) |
| Bulgarian Adblock List (`ruleset_easylist_bulgarian`) | github.com/KokichaKolevTM/BG-Adblock-list | No `LICENSE` file, README statement or header licence found | **Unverified** |
| Anti-Admiral's LanikSJ source (part of `ruleset_antiadmiral`) | github.com/LanikSJ/ubo-filters (`getadmiral-domains.txt`) | MIT, © 2016-2026 LanikSJ | Verified (repository `LICENSE`) |
| Peter Lowe's Ad and tracking server list (`ruleset_peterlowe`) | pgl.yoyo.org, fetched via github.com/uBlockOrigin/uAssets `thirdparties/pgl.yoyo.org/as/serverlist` | McRae General Public License 4.r53 — as quoted in uAssets' own README for that directory: use or redistribution "in any manner that could possibly be construed as making anybody any money" is forbidden (effectively NonCommercial). uAssets itself is GPL-3.0 | Verified (uAssets README; the pgl.yoyo.org licence page itself is not reachable from the build environment) |
| Kees1958 lists | see the Kees1958 + Disconnect section above | none stated | **Unverified** |
| Disconnect | see the Kees1958 + Disconnect section above | CC BY-NC-SA 4.0 (`licence` field of Disconnect's `services.json`, re-read in this audit) | Verified |
| HaGeZi | see the HaGeZi section above | GPL-3.0 | Verified |
| CrUX top-1M (`zakird/crux-top-lists`) | github.com/zakird/crux-top-lists | Repository has no licence file | Build-time only: read into memory to validate `$domain=example.*` wildcard expansions; per `tools/build-rulesets.mjs` no CrUX data is written into any shipped file — only the confirmed expansions (facts about specific domains). Nothing to attribute unless that changes |

## Vendored libraries and small embedded components (added in the 9.7.3 audit)

| Component | Where | Licence | Status |
|---|---|---|---|
| CodeMirror 6 (bundle built for uBlock Origin) + codemirror-quickstart | `lib/codemirror/cm6.bundle.ubol.min.js`, licences bundled as `lib/codemirror/codemirror.LICENSE` (MIT, © 2018-2021 Marijn Haverbeke and others) and `codemirror-quickstart.LICENSE` (MIT, © 2025 Bryan Gillespie) | MIT | Verified (both licence files present and read). Loaded by `dashboard/dashboard.html` |
| RegexAnalyzer v1.2.0 by foo123 | `lib/regexanalyzer/regex.js`; loaded by `js/offscreen/compile-filters.html` | Not stated in the file, its repository (no `LICENSE` under the usual names, none in the README, `package.json` not found) or its GitHub page | **Unverified** — needs a human check with the upstream author |
| s14e-serializer by Raymond Hill | `lib/s14e-serializer.js` | Apache-2.0, © 2024 Raymond Hill (header names the licence and the upstream repository; upstream `LICENSE` re-read) | Verified. **Note: nothing in the tree imports this file** (only `knip.json` mentions `lib/**`, as an ignore) — a shipped file with no consumer; either remove it or keep it knowingly |
| punycode.js v1.3.2 by Mathias Bynens | `js/util/punycode.js` (carries only the `/*! https://mths.be/punycode v1.3.2 by @mathias */` banner) | MIT — full notice below, because MIT requires it to accompany copies and the file itself does not | Verified (upstream `LICENSE-MIT.txt`) |
| Font Awesome 4 glyph outlines (two icons, `angle-up` and `power-off`), via uBlock Origin's `fa-icons.js` | `js/ui/fa-icons.js`; sizing metrics in `css/fa-icons.css` ("Generated from the FontAwesome 4 glyph metrics") | Font Awesome 4.x publishes its glyphs under SIL OFL 1.1 (© Dave Gandy) | **Unverified** — recalled from Font Awesome's published terms, not re-read here. The `ph-` / `md-` branches of `fa-icons.js` point at `/img/photon.svg` / `material-design.svg`, which are not shipped and are used by nothing — no Photon or Material Design artwork is bundled |

**punycode.js — MIT notice (Mathias Bynens):**

> Copyright Mathias Bynens <https://mathiasbynens.be/>
>
> Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Tools and build-time only (checked in the 9.7.3 audit — nothing here ships to users)

- **`tools/` scripts:** no file under `tools/` carries a copyright or licence
  header from anyone else (checked by searching every `tools/` file for
  `Copyright`); the only third-party licence obligations in the build
  come from the *content* it fetches, listed above.
- **`tools/data/kees1958-only-source.txt`, `tools/data/disconnect-other-source.txt`**
  are generated copies of the Kees1958 and Disconnect data described above.
  They live only in the tools zip (never uploaded anywhere), so the
  Disconnect CC BY-NC-SA terms apply to that internal copy as well.
- **npm devDependencies** (`package.json` / `package-lock.json`: 530 packages;
  430 MIT, 39 ISC, 15 Apache-2.0, 25 BSD-2/3-Clause, 9 BlueOak-1.0.0, the rest
  MIT-0 / Python-2.0 / CC0 / one MPL-2.0 (`postcss-values-parser`) and two with
  no licence field). **None is bundled into the extension or the tools zip**
  (only `package.json`/`package-lock.json`, not `node_modules`), so none
  needs attribution here. `@adguard/scriptlets` is the one npm package whose
  *output* does ship (see the table above) and is fetched from the registry
  at build time, not installed as a devDependency.
- **Not third-party:** the toolbar/extension icons in `img/` were generated
  for this project (see CHANGELOG_HISTORY.md 9.7.2).

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

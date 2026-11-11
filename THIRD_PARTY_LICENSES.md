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
section "Filter lists compiled into the shipped rulesets". This is a record of what
was found, not legal advice.

**Two constraints that apply to the extension as a whole**, because of
what it bundles:
- **NonCommercial.** The shipped extension contains data under
  CC BY-NC-SA 4.0 (DuckDuckGo Tracker Radar, the Polish filter list)
  and Peter Lowe's list (McRae General Public License —
  "nothing that could be construed as making anybody any money"). The
  extension therefore must not be sold or monetised; the separately-
  licensed data files are not relicensed to GPL-3.0 by being bundled.
- **ShareAlike.** The compiled ruleset/data files derived from those
  sources (`rulesets/ruleset_easylist_polish.json`, `js/data/tracker-radar-data.json`,
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

## uBlockOrigin/uAssets general-purpose supplemental lists (domain-matched only, added 10.3.0)

- **Source:** https://github.com/uBlockOrigin/uAssets, `filters/filters.txt`
  ("uBlock filters"), `filters/ubol-filters.txt` ("uBO Lite filters"),
  `filters/filters-2026.txt` — fetched live at build time.
- **License:** [GNU General Public License v3.0](https://github.com/uBlockOrigin/uAssets/blob/master/LICENSE)
  — the same license as this project, so no additional restriction
  applies. Verified against the repository's own `LICENSE` file.
- **Copyright:** © uBlock Origin contributors.

**Not vendored whole.** These three are general-purpose supplemental
lists — YouTube ad-removal, miscellaneous per-site fixes, across every
kind of site — not an adult-specific list, and they are **not** shipped
as a selectable filter of their own. Per explicit request, only the rules
whose own host text names a domain already in `tools/build/adult-sites.mjs`'s
`ADULT_SITE_DOMAINS` are kept, via `extractDomainScopedAdultRules()`
(`tools/build/tld-scope.mjs`, called from `tools/build/adult-site-rules.mjs`); the rest of each file (the large majority)
is discarded at build time. Since 10.5.1 the kept rules ship in
`ruleset_annoyances` (assembled by `tools/build/adult-site-rules.mjs`; see
FILTER_PROCESSING_REFERENCE.md §2.5b) alongside AdGuard Base's own adult-site
content — attribution for those rules is therefore split between AdGuard
(§ above, general_url.txt) and these three uAssets files, both GPL-3.0.

---

## uBO-patch scriptlets (4 functions, added 9.8.0)

- **File:** `js/resources/ubo-patch-scriptlets-source.mjs` (hand-maintained,
  not generated)
- **Source:** https://github.com/gorhill/uBlock, `src/js/resources/`
  (`prevent-dialog.js`, `prevent-bab.js`, `prevent-innerHTML.js`,
  `href-sanitizer.js`, `urlskip.js`), fetched via `codeload.github.com`
  tarball of the `master` branch (not commit-pinned — GitHub's API was
  rate-limited at fetch time, matching this project's own documented
  environment lesson on that point; re-verify against a pinned commit if
  precise provenance is ever needed)
- **License:** GPL-3.0-or-later, same as the upstream-code-base entry above
- **Copyright:** © 2019-present Raymond Hill and uBlock Origin contributors

**Attribution:** four scriptlets missing from the vendored
`@adguard/scriptlets` library — `prevent-dialog`, `bab-defuser`/`nobab`,
`freeze-element-property`, and `href-sanitizer`'s `urlskip` alias — adapted
(not copy-pasted) from uBlock Origin's own real source, each with its
dependencies (uBO's `proxyApplyFn`, `urlSkip()`) inlined so every scriptlet
is one fully self-contained function, per this project's own dispatch
mechanism's requirements. Added per
`Xplainer_Programming_principles_audit_v5.md`'s C40: only scriptlets
confirmed small and standalone (no shared multi-hundred-line engine) and
confirmed NOT to carry uBO's own `requiresTrust: true` registration flag.
A build-time check (`buildCustomScriptletsDispatch()`,
`tools/build-rulesets.mjs`) compares these names against `@adguard/scriptlets`
on every build and always prefers the real library's version the moment it
adds one of these names, flagging (never silently dropping) the now-redundant
patch entry for manual removal — this already fired once, for `href-sanitizer`
itself, before the very first real build (see C40's own worked example).

**Deliberately not ported** (C40): `prevent-clipboard-write` (uBO's own
`requiresTrust: true`), the `edit-*-object`/`json-edit-*`/`jsonl-edit-*`
family (a 1,575-line shared engine), `mpegdash-prune` (a 329-line shared
XHR-response engine), and all 17 AdGuard `trusted-*` names (restricted by
AdGuard's own policy; confirmed none of this project's shipped content
needs any of them) — see `js/core/custom-scriptlets.js`'s
`DELIBERATELY_EXCLUDED_SCRIPTLET_NAMES` for the full, explicit list.

---

## DuckDuckGo Tracker Radar (compacted dataset)

- **File:** `js/data/tracker-radar-data.json` (generated — see
  `tools/build-tracker-radar.mjs`, rebuilt from a real checkout: 2,034
  domains / 682 entities, 2026-10-16 — see that file's own header).
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

- **`js/scripting/autodeny-snippets.generated.js`** (decline) and, since
  10.7.1, **`js/scripting/autoaccept-snippets.generated.js`** (accept) are
  compiled at build time from both files above (see
  `tools/build-autodeny-bundle.mjs`) and inherit the same mixed status. The
  10.7.1 accept data (`rules.json` `accept` fields and `EVAL_ACCEPT_ALL`) was
  written for this project against each CMP's public accept-all button or API;
  it adds no third-party source. Because it lives in the two files above, which
  contain MPL-2.0-covered code, it is distributed under MPL-2.0 too (MPL-2.0 is
  file-level copyleft); `vendor/autoconsent/LICENSE` already covers it, and
  MPL-2.0 is compatible with this project's GPL-3.0 — the MPL-2.0-derived portion
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

## Filter lists compiled into the shipped rulesets

The rulesets (`rulesets/*.json` and their cosmetic and scriptlet files) are
compiled at build time from public third-party filter lists — for example
AdGuard's filters (https://github.com/AdguardTeam/AdguardFilters, GPL-3.0),
including, since 10.7.1, the tracking-server exceptions from its
SpywareFilter allowlist. No copy of a source list is kept in any zip; each
list keeps its own licence, and bundling does not relicense it to GPL-3.0.
The licences among the sources are GPL-3.0, LGPL-3.0, MIT, CC BY 4.0,
CC BY-SA 4.0, CC BY-NC-SA 4.0, public domain and a few list-specific ones.

Attributions and notices that some of those licences require:

- **Polish filters** (github.com/MajkiIT/polish-ads-filter) — CC BY-NC-SA 4.0,
  © Certyficate IT: NonCommercial and ShareAlike.
- **Peter Lowe's Ad and tracking server list** (pgl.yoyo.org) — McRae General
  Public License: no use that makes anybody any money.
- **hufilter** (github.com/hufilter/hufilter) — CC BY 4.0.
- **EasyList Czech and Slovak** (github.com/tomasko126/easylistczechandslovak) and
  **Latvian List** (github.com/Latvian-List/adblock-latvian) — CC BY-SA 4.0: ShareAlike.
- **Dandelion Sprout's Nordic Filters and Serbo-Croatian List**
  (github.com/DandelionSprout/adfilt) — Dandelicence, licence text:
  https://github.com/DandelionSprout/adfilt/blob/master/LICENSE.md
- **MIT-licensed lists** keep their copyright notices in their own
  repositories: Macedonian adBlock Filters (github.com/DeepSpaceHarbor/Macedonian-adBlock-Filters),
  ROad-Block (github.com/tcptomato/ROad-Block), LanikSJ's ubo-filters
  (github.com/LanikSJ/ubo-filters).

A few sources state no licence that could be verified (some EasyList country
lists and the Bulgarian list); the per-list audit record is in the project
history (CHANGELOG_HISTORY.md, 9.7.3 audit). Disconnect is no longer used
(removed 10.7.1).

## Country and time-zone data (`js/data/country-data/`, added 10.4.5)

| Data | Source | Licence | Status |
|---|---|---|---|
| `countries.json`, `language-rules.json`, `timezones.json` (continent part) and `js/core/country-resolver.js` | the project owner's own 3P Matrix extension (its `data/country-data.js` and country resolver) | the owner's own work, same licence as this project | Verified (owner-supplied) |
| `timezones.json` `zoneCountry` (time zone → country) | IANA Time Zone Database 2026e (`zone.tab` + backward links), generated, not hand-typed | Public domain (the tz database's own statement) | Verified |

## Vendored libraries and small embedded components (added in the 9.7.3 audit)

| Component | Where | Licence | Status |
|---|---|---|---|
| AdGuard Scriptlets library (moved here from the filter-list table, 10.7.1) | `js/resources/ag-scriptlets-corelibs.json` and the scriptlet code inlined in `rulesets/*-scriptlets.js`; from npm `@adguard/scriptlets` / github.com/AdguardTeam/Scriptlets | GPL-3.0 | Verified (repository `LICENSE`) |
| CodeMirror 6 (bundle built for uBlock Origin) + codemirror-quickstart | `lib/codemirror/cm6.bundle.ubol.min.js`, licences bundled as `lib/codemirror/codemirror.LICENSE` (MIT, © 2018-2021 Marijn Haverbeke and others) and `codemirror-quickstart.LICENSE` (MIT, © 2025 Bryan Gillespie) | MIT | Verified (both licence files present and read). Loaded by `dashboard/dashboard.html` |
| RegexAnalyzer v1.2.0 by foo123 | `lib/regexanalyzer/regex.js`; loaded by `js/offscreen/compile-filters.html` | Not stated in the file, its repository (no `LICENSE` under the usual names, none in the README, `package.json` not found) or its GitHub page | **Unverified** — needs a human check with the upstream author |
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
- Every filter list is downloaded by the build itself, so no copy of a source
  list is kept in any zip.
- **npm devDependencies** (`package.json` / `package-lock.json`: 530 packages;
  430 MIT, 39 ISC, 15 Apache-2.0, 25 BSD-2/3-Clause, 9 BlueOak-1.0.0, the rest
  MIT-0 / Python-2.0 / CC0 / one MPL-2.0 (`postcss-values-parser`) and two with
  no licence field). **None is bundled into the extension or the tools zip**
  (only `package.json`/`package-lock.json`, not `node_modules`), so none
  needs attribution here. `@adguard/scriptlets` is the one npm package whose
  *output* does ship (see "Vendored libraries" above) and is fetched from the registry
  at build time, not installed as a devDependency.
- **Not third-party:** the toolbar/extension icons in `img/` were generated
  for this project (see CHANGELOG_HISTORY.md 9.7.2).

---

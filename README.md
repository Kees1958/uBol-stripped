UBOL-STRIPPED SMART AND LEAN ADVERTISING & TRACKERS BLOCKER

uBol-stripped blocks ads, trackers, and privacy threats while you browse — without slowing your browser down. It works silently in the background from the moment you install it. It offers the AdGuard like functionality with uBO-lite efficiency AND has some unique features like the Privacy Inspector and the Create Custom DNR rules monitors. 

Chrome webstore: https://chromewebstore.google.com/detail/ubol-stripped/femdnbckdgobaelpmbbajpidneljkjaa?pli=1

<img width="295" height="399" alt="image" src="https://github.com/user-attachments/assets/b87cfbe4-beb7-4714-89b3-2c7de1dddfa3" />
  

IMPORT ABP-RULES
Based on AdGuard's user rules panel. You can paste your own ABP filter rules into it (up to 5,000 rules). It also processes all uBO-lite and all AdGuard scritplet's used in the AG Base filter 

<img width="882" height="676" alt="image" src="https://github.com/user-attachments/assets/7d75d5eb-32dd-415d-94c5-dc12d0d01c2c" />
.


PRIVACY INSPECTOR
Inspired by the work of the JShelter project. Protect yourself against fingerprinting, see what tracking techniques are used in a website and simply block them by selecting that fingerprinting mechanism (or choose block all).

<img width="1029" height="833" alt="image" src="https://github.com/user-attachments/assets/edd8dc20-be45-460b-b7b0-c5d27e84f024" />
.


CREATE CUSTOM COSMETIC RULES
This is uBO's element picker (only a popup is added to explain how it works). Draw a box around any annoying element on a page to permanently hide it. 

<img width="1197" height="582" alt="image" src="https://github.com/user-attachments/assets/369ed757-c5d4-4beb-8f95-5adc786c937a" />
.


CREATE CUSTOM DNR RULES
A new and user-friendly DNR rule "picker" feature, whose user interface is inspired on the NoScript project. Watch live which external services a website contacts, and block them with one click.

<img width="1488" height="650" alt="image" src="https://github.com/user-attachments/assets/d6a6585c-4390-44e1-a63b-c0f2bf28c55a" />
.


FILTER (BLOCK) LISTS
By default uBol-stripped uses three well curated filter lists: Kees1958 most used EU-US, AdGuard Base filter and AdGuard parameter protection. Only rules with country code (TLD) existing in 5Eyes countries and extended EU-zone are used (others are silently dropped as well as generic cosmetic rules for performance reasons). Rules for websites not in Chrome's top 1 million used websites are also silently dropped. Optionally you can enable the AdGuard anti-adblock list. Language filters of AG and EL from (only) EU-countries are enabled based on your browser language setting automatically (users can undo that when they want).

<img width="877" height="729" alt="image" src="https://github.com/user-attachments/assets/5f2fadb9-824f-4240-b753-bd4736b3f0d3" />
.


ALLOW LIST 
Based on AdGuard's allowlist. Easily copy and paste domains into the allow list to disable filtering on that domain (uBO-lite also has a filtering modes panel, but that is hidden behind the developer's option). 

<img width="891" height="334" alt="image" src="https://github.com/user-attachments/assets/588131d8-8cc1-4f9e-bd39-0591b1421e94" />
.


PRIVACY & SECURITY 
Enable additional Security & Privacy protections with a very low  website breakage risk. These options provide real world protection (not like the malware lists of uBO-Lite and AG Mv3 which are  refreshed only every 12 hours due to Mv3 restrictions, making them near useless). Better use a DNS with malware protection and enable safe browsing.
.
<img width="881" height="883" alt="image" src="https://github.com/user-attachments/assets/56ab3937-92ef-4359-a340-5be1eddf6739" />



WHY NAME IT UBOL-STRIPPED?
UBO-lite has strong bones and started as a permission less extension (AdGuard requires user script permission to use scriptlet's in the user rules). It shows what an extraordinary programmer mr Hill is. But it also has some less well coded parts, which were (hastily) added to keep up with AdGuard Mv3. So I stripped uBO-lite's AG-like functionality and used (the stronger and better designed) AG functions instead. I also looked at other extensions (JShelter for the Privacy Inspector detections and NoScript for the DNR rules monitor user interface). 



NEW CODE IS DEVELOPED AND EXISTING CODE IS REFACTORED (WHEN POSSIBLE) USING OLD STRUCTURED PROGRAMMING PRINCIPLES (nerd alert): 

1. SEPARATION OF CONCERNS — HTML, CSS AND JAVASCRIPT
Every file has exactly one job. HTML describes the structure of the page. CSS handles all visual styling. JavaScript handles all behaviour. No styling is written inside JavaScript. No logic is embedded in HTML. This means you can change how something looks without touching the code that makes it work, and vice versa. Each dashboard tab, each popup panel and each background module is its own isolated file.

2. FIVE-TIER FOLDER STRUCTURE
The codebase is organised into five layers, and code is only allowed to call downward — never upward or sideways. This means a bug in the UI can never corrupt the filter logic, and the filter logic can never accidentally write to the UI. Every dependency is explicit and traceable (ui — what the user sees and clicks, workflow — the service worker that coordinates everything, core — the actual blocking and filtering logic, util — small shared helper functions, data — storage, configuration and external API calls)

3. STATE VECTORS AND SWIMMING LANES
Every multi-step process owns a single named state object — called a state vector — that holds all its data and a phase field describing exactly where in its lifecycle it currently is. For example the monitor session can only be in one of four states: idle, starting, running or paused. A guard function checks the phase at the start of every operation and logs a warning if something tries to happen out of order. This means bugs that used to be silent — a session that got stuck halfway, a timer that never fired — are now immediately visible in the browser console with the exact step that failed.
The service worker startup sequence follows the same pattern: it tracks whether it is booting, starting, ready or in error, and records which sub-step it was on if something goes wrong. If the extension ever fails to start, the error message tells you exactly where.

4. MAINTAINABILITY AND BOUNDARY GUARDS
Every capacity limit — the maximum number of custom DNR rules, the maximum number of cosmetic rules, the ID ranges for each rule type — is defined in a single file (dnr-budgets.js) and imported everywhere else. Changing a limit means editing one number in one place. Every module that manages a resource also owns the cleanup of that resource. Timers live inside the state vector they belong to, not scattered across the module. Every future scaling limitation is marked with a comment explaining what the ceiling is and what a future developer would need to change to raise it.
_


WHAT CORE-MODULES ARE ADOPTED BUT STRUCTURALLY KEPT IN TACT?

background.js used to route every message through three giant, stacked decision-trees — a message's security check depended on which tree it was written in, invisibly. Those three trees are now gone, replaced by one clear table, message-routes.js, where every message's rules sit right next to its name instead of being implied by position (in simple terms: made transparent what was in the head of Mr Hill when he coded this module).

scripting-manager.js (turns filter rules on/off) had its "only one thing at a time" safety rule copy-pasted twice in the same file. That rule now lives once, in a new small file, async-lock.js, shared by both places instead of two copies that could quietly drift apart. Everything scripting-manager.js does shares the same safety rule and touches the same browser feature, so splitting it apart would mean the safety rule could easily end up duplicated again — this time across separate files instead of within one, which is harder to notice and easier to get wrong (so it is better to keep in one place = one JS-module).

Scattered duplicate settings got merged, a few bugs were caught and fixed along the way and the ESlinter (code check tool) is clean now.

a) cripting-manager.js — a repeated timing value (15-minute cache-cleanup interval) was written out twice in the same file; now written once.

b) timing-constants.js (new) — a 5-minute timeout value that four different files had each separately written out, now all read from one place.

background.js and scripting-manager.js both grew rather than shrank — nothing was deleted, each piece of logic just got its own clearly-named home plus an explanation of why it works the way it does, and that documentation takes real space (to prevent coding landmines and warn others to NOT re-structure code in smaller chunks, the code shows Mr Hill is an exceptional programmer).

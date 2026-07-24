uBol-stripped — Smart ad & tracker blocker
uBol-stripped blocks ads, trackers and privacy threats while you browse — without slowing your browser down. It works silently in the background the moment you install it. No account, no settings, no hassle.

Chrome webstore: https://chromewebstore.google.com/detail/ubol-stripped/femdnbckdgobaelpmbbajpidneljkjaa?pli=1

<img width="295" height="399" alt="image" src="https://github.com/user-attachments/assets/b87cfbe4-beb7-4714-89b3-2c7de1dddfa3" />
  

HOW UBOL-STRIPPED COMPARES TO UBO-LITE

uBO Lite ships with more filter lists enabled by default — EasyList, EasyPrivacy, Peter Lowe and others — which gives broader coverage but can block cookie consent flows that prevent login on some sites. uBol-stripped uses only three well curated set (Kees1958, AdGuard Base, AdGuard tracking parameters) and only precesses rules from extended EU-zone and 5 Eyes Countries (and it only offers to add additional EU-langauge filters). That is why it is called ¨stripped". On the plus site uBol-stripped has gained some AG-skills, it processes all scriptlets present in the AG base filter. 



FOR USERS WHO WANT MORE CONTROL (OR LIKE THE HIGH-TECH OF UBO-LITE & THE HIGH TOUCH OF AG-MV3)

1. Create custom cosmetic rules (uBO-lite element picker)

Draw a box around any annoying element on a page to permanently hide it
<img width="1197" height="582" alt="image" src="https://github.com/user-attachments/assets/369ed757-c5d4-4beb-8f95-5adc786c937a" />

 
2. Create custom DNR rules
Watch live which external services a website contacts and block them with one click
<img width="1488" height="650" alt="image" src="https://github.com/user-attachments/assets/d6a6585c-4390-44e1-a63b-c0f2bf28c55a" />


3 Privacy Inspector to be used on-demand for websites you often visit but never log in to
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d716b99a-68ee-400c-a812-ebbc378b5349" />



4. Import ABP-rules in one place
Paste your own ABP/uBlock filter rules
<img width="882" height="676" alt="image" src="https://github.com/user-attachments/assets/7d75d5eb-32dd-415d-94c5-dc12d0d01c2c" />

 

5 Enable advanced Security & Privacy options
<img width="843" height="852" alt="image" src="https://github.com/user-attachments/assets/6bc7e5eb-8dd9-4d39-978a-c29b803f3618" />

_




_


PRIVACY FIRST
uBol-stripped contains no analytics, no telemetry and no ads. It never sends any data about your browsing to anyone. Filter lists are compiled and bundled inside the extension — no external servers are contacted except to check one optional list of suspicious hosting domains when you enable that feature.


WHAT IS CHANGED?

In layman's terms we (Claude AI and me) used the strong bones of uBO-lite and stripped it (removed less well programmed features to keep up with AdGuard Mv3). Next we added some open source AdGuard skills (it is open source so why not use what is available in stead of competing as a one man band with a team of developers). Next I added some features which in my opinion were missing in uBO-lite (see pictures). To make the code easy to understand, extend and debug I refactored the strong uBO-lite bones using four (old structured programming) principles (nerd alert): 

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


_


WHAT CORE-MODULES ARE ADOPTED BUT STRUCTURALLY  KEPT AS IS (V5.0.17 and higher)

background.js used to route every message through three giant, stacked decision-trees — a message's security check depended on which tree it was written in, invisibly. Those three trees are now gone, replaced by one clear table, message-routes.js, where every message's rules sit right next to its name instead of being implied by position (in simple terms: made transparent what was in the head of Mr Hill when he coded this module).

scripting-manager.js (turns filter rules on/off) had its "only one thing at a time" safety rule copy-pasted twice in the same file. That rule now lives once, in a new small file, async-lock.js, shared by both places instead of two copies that could quietly drift apart. Everything scripting-manager.js does shares the same safety rule and touches the same browser feature, so splitting it apart would mean the safety rule could easily end up duplicated again — this time across separate files instead of within one, which is harder to notice and easier to get wrong (so it is better to keep in one place = one JS-module).

Scattered duplicate settings got merged, a few bugs were caught and fixed along the way (also the darkmode), and the ESlinter (code check tool) is clean now.
a) cripting-manager.js — a repeated timing value (15-minute cache-cleanup interval) was written out twice in the same file; now written once.
b) timing-constants.js (new) — a 5-minute timeout value that four different files had each separately written out by coincidence, now all read from one place.

background.js and scripting-manager.js both grew rather than shrank — nothing was deleted, each piece of logic just got its own clearly-named home plus an explanation of why it works the way it does, and that documentation takes real space (to prevent coding landmines and warn others not structure code in smaller chunks, the code shows Mr Hill is an exceptional programmer).
